# 日本の祝日情報を取得する方法 — JSONで使うための3つの案

## 祝日データをカレンダーで使うための前提

- カレンダーに日本の祝日を表示するため、日付と祝日名をJSONで取得する方法を整理する
- 対象は「国民の祝日」と、振替休日・祝日に挟まれた休日 — 土日や会社独自の休業日は別に管理する
- 手軽に導入するならHolidays JP API、Googleカレンダーとの連携がすでにあるならGoogle Calendar API、取得元を公的な情報にそろえるなら内閣府のCSVが候補になる
- 安定して表示したい場合は、取得したデータを検証して自サイトに保存し、そのJSONをカレンダーから読む構成にする — 外部サービスへの通信を画面表示のたびに行わずに済む
- 以下では、取得元にかかわらず日付を`YYYY-MM-DD`、値を名称にそろえる — 日付ごとの判定やカレンダー用イベントへの変換がしやすい
- 掲載内容の確認日は2026年10月20日

```json
{
  "2026-01-01": "元日",
  "2026-01-12": "成人の日"
}
```

> 参照: [内閣府 — 「国民の祝日」について](https://www8.cao.go.jp/chosei/shukujitsu/gaiyou.html)

## 案01：Holidays JP APIからJSONを取得する

- Holidays JP APIは、日本の祝日一覧をJSON・CSVで公開しているサービス — 内閣府が提供するAPIではない
- Googleが用意しているGoogleカレンダーの情報から、データを自動生成している
- APIキーは不要で、`fetch()`から取得できる
- `https://holidays-jp.github.io/api/v1/date.json`で一覧、`https://holidays-jp.github.io/api/v1/2026/date.json`で年別のデータを取得する
- 公式サイトが案内している対象範囲は去年・今年・来年 — 年別URLがあるからといって、任意の過去・未来の年を取得できる前提にはしない
- JSONは日付をキーにしたオブジェクトなので、取得後にそのまま祝日判定に使える

```js
async function fetchHolidays(year) {
  const response = await fetch(
    `https://holidays-jp.github.io/api/v1/${year}/date.json`,
    { signal: AbortSignal.timeout(10000) },
  );

  if (!response.ok) {
    throw new Error(`祝日データの取得に失敗: ${response.status}`);
  }

  return response.json();
}

// モジュール内、またはasync関数内で実行する
const holidays = await fetchHolidays(2026);
console.log(holidays['2026-01-01']); // 元日
```

### 採用するときに確認すること

- 導入時の作業が少なく、Google Cloudの設定やCSVの変換処理を自分で用意せずに済む
- 一方、データの更新と配信を外部プロジェクトに依存する — 利用年のデータがあるか、更新が続いているかを確認する
- HTTPエラーやタイムアウトを「祝日なし」に変換しない — 呼び出し側でエラーを受け取り、保存済みデータの利用や取得失敗の表示を判断する
- Google由来の情報なので、公的な休日だけを厳密に扱う用途では内閣府の一覧と照合する

> 参照: [Holidays JP API](https://holidays-jp.github.io/) / [ソースコード — holidays-jp/api](https://github.com/holidays-jp/api)

## 案02：Google Calendar APIから自分で取得する

- Googleが公開している日本の祝日カレンダーを、Google Calendar APIの`events.list`で取得する方法
- Holidays JP APIと取得元は共通だが、APIの設定、取得、整形、保存を自分で管理する点が異なる
- 自前で取得しても、Google側の祝日データへの依存は残る — 案01と案02を切り替えても、取得元そのものを二重化したことにはならない
- すでにGoogle Calendar APIを使っている場合や、ほかの公開カレンダーも同じ処理で取り込みたい場合に検討しやすい

### 準備するもの

- Google Cloudでプロジェクトを用意し、Google Calendar APIを有効にする
- 公開データを読むためのAPIキーを作成し、利用できるAPIと実行元を制限する — サーバーで実行する場合は、その環境に合う制限を設定する
- 取得対象の公開カレンダーIDを確認する — Googleカレンダーの設定にある「カレンダーの統合」などで確認し、古い記事のIDを無条件に転用しない
- 以下の例はNode.js 22以降のサーバー・ビルド処理向け — `GOOGLE_CALENDAR_API_KEY`と`GOOGLE_HOLIDAY_CALENDAR_ID`を環境変数に設定する
- 公開カレンダーの読み取りと、利用者の非公開カレンダーへのアクセスは別 — 後者にはOAuthなどの認可が必要になる

> 参照: [Google Workspace — Create access credentials](https://developers.google.com/workspace/guides/create-credentials#api-key) / [Googleカレンダー — 他のユーザーとカレンダーを共有する](https://support.google.com/calendar/answer/37082?hl=ja)

### 年単位で取得し、日付と名称に整形する

- `timeMin`と`timeMax`で取得期間を指定し、日本時間の年初から翌年の年初までを対象にする
- `singleEvents=true`で繰り返し予定を個別の予定として取得する
- `nextPageToken`があれば次のページも読む — 1回のレスポンスだけで取得が完了したと判断しない
- 以下は終日予定の日付と名称を取り出す例であり、法律上の祝日を判定する処理は含まない

```js
async function fetchPublicCalendarDays(year) {
  const apiKey = process.env.GOOGLE_CALENDAR_API_KEY;
  const calendarId = process.env.GOOGLE_HOLIDAY_CALENDAR_ID;
  if (!apiKey || !calendarId) {
    throw new Error('APIキーと公開カレンダーIDを設定する');
  }

  const days = {};
  let pageToken;

  do {
    const url = new URL(
      `https://www.googleapis.com/calendar/v3/calendars/${encodeURIComponent(calendarId)}/events`,
    );
    url.search = new URLSearchParams({
      key: apiKey,
      timeMin: `${year}-01-01T00:00:00+09:00`,
      timeMax: `${year + 1}-01-01T00:00:00+09:00`,
      timeZone: 'Asia/Tokyo',
      singleEvents: 'true',
      orderBy: 'startTime',
    }).toString();
    if (pageToken) url.searchParams.set('pageToken', pageToken);

    const response = await fetch(url, {
      signal: AbortSignal.timeout(10000),
    });
    if (!response.ok) {
      throw new Error(`カレンダーの取得に失敗: ${response.status}`);
    }

    const data = await response.json();
    for (const event of data.items ?? []) {
      const date = event.start?.date;
      if (event.status === 'cancelled') continue;
      if (!date?.startsWith(`${year}-`) || !event.summary) continue;
      if (Object.hasOwn(days, date)) {
        throw new Error(`同日の予定が重複しているため要確認: ${date}`);
      }
      days[date] = event.summary;
    }
    pageToken = data.nextPageToken;
  } while (pageToken);

  return days;
}

const days = await fetchPublicCalendarDays(2026);
console.log(days);
```

- Googleの祝日カレンダーには、祝日のほかに行事を扱う設定もある — 画面上の表示設定がAPIの結果にも反映されると決めつけず、取得内容を確認する
- 終日予定であることや、名称に「祝日」が含まれることだけでは、公的な休日かどうかを判定できない
- 祝日用の1日単位のデータを想定した例なので、複数日にまたがる予定を持つ一般のカレンダーにはそのまま流用しない
- サーバーで取得した結果をJSONとして保存し、フロントエンドには整形済みのデータを渡す

> 参照: [Google Calendar API — Events: list](https://developers.google.com/workspace/calendar/api/v3/reference/events/list) / [Eventsリソース](https://developers.google.com/workspace/calendar/api/v3/reference/events) / [Googleカレンダー — 祝日を追加または削除する](https://support.google.com/calendar/answer/13748345?hl=ja)
>
> 関連記事: [JavaScriptのGoogle Calendar APIで祝日を判定する方法](https://iwb.jp/javascript-google-calendar-api-credentials-holiday/) / [PHPでGoogleカレンダーから日本の祝日一覧を取得する](https://www.aura-office.co.jp/blog/php-get-holiday/)

## 案03：内閣府のCSVをJSONに変換する

- 内閣府は「国民の祝日」についてのページで、祝日・休日の一覧をCSVとして公開している
- 取得URLは`https://www8.cao.go.jp/chosei/shukujitsu/syukujitsu.csv` — JSONではないため、自分で変換する
- 確認時点の掲載範囲は1955年から2027年まで — 未掲載の年を「祝日なし」と扱わない
- 振替休日や祝日に挟まれた休日も収録される — 名称が「休日」の行を除外しない
- 公的な情報を元にできる一方、文字コード、CSVの解析、JSONの生成、更新処理は自分で管理する

### Node.jsでCSVからJSONを生成する

- 以下はNode.js 22以降で実行するビルド用スクリプトの例 — ブラウザーから内閣府へ直接アクセスする構成にはしない
- 作業用のNode.jsプロジェクトで`npm install csv-parse`を実行し、次のコードを`generate-holidays.mjs`として保存する
- CSVはShift_JISとして読み、日付を`YYYY-MM-DD`へ変換する — UTF-8を前提にした`response.text()`を使うと日本語が文字化けする
- CSVの解析には`csv-parse`を使う — 単純な`split(',')`では、引用符や値に含まれるカンマを扱えない
- ヘッダーと日付、重複、対象年のデータの有無を確認してから、`public/holidays-2026.json`へ書き出す

```js
// generate-holidays.mjs
import { mkdir, writeFile } from 'node:fs/promises';
import { parse } from 'csv-parse/sync';

const year = 2026;
const response = await fetch(
  'https://www8.cao.go.jp/chosei/shukujitsu/syukujitsu.csv',
  { signal: AbortSignal.timeout(10000) },
);
if (!response.ok) {
  throw new Error(`CSVの取得に失敗: ${response.status}`);
}

const csv = new TextDecoder('shift_jis', { fatal: true }).decode(
  await response.arrayBuffer(),
);
const [header, ...rows] = parse(csv, {
  bom: true,
  skip_empty_lines: true,
});
if (
  header?.[0] !== '国民の祝日・休日月日' ||
  header?.[1] !== '国民の祝日・休日名称'
) {
  throw new Error('CSVのヘッダーが想定と異なる');
}

const holidays = {};
for (const [rawDate, name] of rows) {
  const match = /^(\d{4})\/(\d{1,2})\/(\d{1,2})$/.exec(rawDate);
  if (!match || !name?.trim()) throw new Error('CSVの値が不正');

  const [, y, m, d] = match;
  const date = `${y}-${m.padStart(2, '0')}-${d.padStart(2, '0')}`;
  const parsed = new Date(`${date}T00:00:00Z`);
  if (
    Number.isNaN(parsed.getTime()) ||
    parsed.toISOString().slice(0, 10) !== date
  ) {
    throw new Error(`日付が不正: ${rawDate}`);
  }
  if (Number(y) !== year) continue;
  if (Object.hasOwn(holidays, date)) {
    throw new Error(`日付が重複している: ${date}`);
  }
  holidays[date] = name.trim();
}

if (Object.keys(holidays).length === 0) {
  throw new Error(`${year}年の祝日データが見つからない`);
}

await mkdir('public', { recursive: true });
await writeFile(
  `public/holidays-${year}.json`,
  JSON.stringify(holidays, null, 2) + '\n',
  'utf8',
);
```

- `node generate-holidays.mjs`で生成し、`public`内のファイルを静的ファイルとして配信する構成を想定している — 配置先は利用するフレームワークに合わせる
- このコードは取得と変換の例 — 本番では生成結果を一時ファイルやビルド成果物として確認し、成功した場合だけ公開中のJSONを置き換える
- 出典を追えるよう、配信先のページやデータの説明に内閣府へのリンクと取得日を残す

> 参照: [内閣府 — 「国民の祝日」について](https://www8.cao.go.jp/chosei/shukujitsu/gaiyou.html) / [祝日・休日一覧CSV](https://www8.cao.go.jp/chosei/shukujitsu/syukujitsu.csv) / [CSV Parse — Sync API](https://csv.js.org/parse/api/sync/) / [MDN — TextDecoder](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoder)
>
> 関連記事: [日本の祝日データを自前で持つ — 内閣府のCSVをJSONにする](https://zenn.dev/kodomo_news/articles/12-japanese-holidays) / [JavaScriptで祝日対応のカレンダー作成](https://nyanblog2222.com/programming/javascript/2835/)

## 他の方法と、安定して表示するための運用

### ライブラリや手動管理のJSONを使う

- npmパッケージの`@holiday-jp/holiday_jp`を使う方法もある — 指定期間の祝日を取得し、用途に合うJSONへ変換できる
- パッケージをアプリに組み込む構成なら、画面表示時に外部の祝日APIへ通信せずに済む
- パッケージが収録している年の範囲と更新状況は確認する — インストールすれば将来の祝日にも無条件に対応できるわけではない
- 表示する年が限られる小規模なサイトなら、内閣府の一覧を確認してJSONを手動で管理する方法もある — 年の追加と変更時の確認を作業として残す

> 参照: [holiday-jp/holiday_jp-js](https://github.com/holiday-jp/holiday_jp-js)

### 取得処理と画面表示を分ける

- 取得元への接続を、サーバーの定期処理やビルド時にまとめる — 例えば月1回の取得に加え、翌年分の公表や制度変更を確認したときに再取得する
- 取得に失敗した場合は、最後に正常取得したJSONを保持し、更新の失敗を運用者へ通知する
- 保存済みデータを使う場合も、対象年を収録しているか確認する — 保存されていない翌年を平日だけのカレンダーとして表示しない
- 更新日時と対象年を記録し、取得自体が止まった場合にも気づけるようにする
- 更新前後の日付や件数の差分を確認する — HTTPの成功やJSONの解析成功だけでは、必要な休日がそろっているかは分からない
- 2026年を確認するなら、通常の祝日に加え、5月6日の振替休日と9月22日の祝日に挟まれた休日も確認対象にする
- 外部サービスの停止に備えることと、祝日データの正確さを確認することは別の作業として扱う

> 参照: [内閣府 — 2026年・2027年の国民の祝日・休日](https://www8.cao.go.jp/chosei/shukujitsu/gaiyou.html)

### FullCalendarでは終日イベントに変換する

- 保存したJSONを取得し、日付を`start`、名称を`title`に変換すると、FullCalendarのイベントとして渡せる
- 以下はFullCalendar本体と使用する表示用プラグインを読み込み済みで、`#calendar`要素がある場合の例
- 日付を不要に日時へ変換せず、`YYYY-MM-DD`の文字列と`allDay: true`で渡す — UTCへの変換で日付がずれる処理を避けられる
- 祝日の背景色だけを変える場合でも、名称をテキストで確認できるようにする

```js
const response = await fetch('/holidays-2026.json');
if (!response.ok) throw new Error('祝日データを読み込めない');
const holidays = await response.json();

const calendar = new FullCalendar.Calendar(
  document.querySelector('#calendar'),
  {
    initialView: 'dayGridMonth',
    initialDate: '2026-01-01',
    events: Object.entries(holidays).map(([date, name]) => ({
      title: name,
      start: date,
      allDay: true,
    })),
  },
);
calendar.render();
```

- この例のデータは2026年分だけなので、ほかの年への移動に対応する場合は、その年のJSONも取得してイベントを更新する
- 既存の予定がある場合は、通常の予定用データと祝日用データをイベントソースとして追加するなど、両方を表示する構成にする

> 参照: [FullCalendar — Event Parsing](https://fullcalendar.io/docs/event-parsing) / [events as an array](https://fullcalendar.io/docs/events-array)

## まとめ

- 手軽にJSONを取得するならHolidays JP API — APIキーなしで利用でき、公式の案内範囲は去年・今年・来年
- Google Calendar APIから自前で取得する場合は、認証情報、カレンダーの選択、ページ送り、データの確認を自分で管理する
- Holidays JP APIとGoogle Calendar APIは取得元が共通なので、両方を使っても祝日情報の独立した照合先にはならない
- 公的な休日を元にJSONを用意するなら内閣府のCSV — 文字コードの変換と更新処理が必要になる
- 外部APIへの通信を画面表示のたびに行わず、検証済みのJSONを自サイトで配信すると、取得元の停止中も表示を続けやすい
- 今回のように日本の祝日をカレンダーへ表示する用途なら、導入の手軽さを優先する場合は案01、公的な情報を元に自分で管理する場合は案03を選ぶ
