# GitHubのcontributionsを取得する

## コントリビューションとは

- GitHubのプロフィールには過去1年間のコントリビューションを日ごとのマスで表すカレンダーがある<sup><a href="#ref-1">[1]</a></sup>
- コントリビューションには条件を満たすコミット、Issueの作成、Pull Requestの作成・レビューなどが含まれる — コミット数だけを表すものではない<sup><a href="#ref-2">[2]</a></sup>
- マスが緑色に並ぶ見た目から、日本の開発者の間では「草」と呼ばれ、活動を記録することを「草を生やす」と表現する
- `618 contributions in the last year` は過去1年間の合計が618件という表示例である — 618日活動したという意味ではない
- 合計と日別件数を公式APIで取得するならGraphQLを使える — 画面から読み取る場合は、SVGかHTMLかを先に確認する

参照

<ol start="1">
  <li id="ref-1"><a href="https://docs.github.com/en/account-and-profile/concepts/contributions-on-your-profile">GitHub Docs — Contributions on your profile</a></li>
  <li id="ref-2"><a href="https://docs.github.com/en/account-and-profile/reference/profile-contributions-reference">GitHub Docs — Profile contributions reference</a></li>
</ol>

## REST APIとGraphQL API

- GitHub REST APIには、プロフィールのコントリビューションカレンダーをそのまま取得する専用エンドポイントがない
- ユーザーのイベントを取得するREST APIも代用には向かない — 取得対象は過去30日以内、最大300イベントであり、1年間のカレンダーを復元するためのデータはそろわない<sup><a href="#ref-3">[3]</a></sup>
- GraphQLは取得したい項目をクエリに書いて問い合わせるAPIの方式である
- GitHub GraphQL APIの `contributionsCollection.contributionCalendar` から、合計の `totalContributions` と日別の `contributionCount` を取得できる<sup><a href="#ref-4">[4]</a></sup>
- `contributionsCollection` の `from` を省略すると1年前、`to` を省略すると現在時刻が基準になる — 次のクエリは期間を指定せずに取得する例である
- クエリは認証して `https://api.github.com/graphql` に送る — アクセストークンはサーバーやビルド環境で管理し、公開するブラウザーのコードには含めない<sup><a href="#ref-5">[5]</a></sup>
- `user` が `null` ならユーザー名を確認し、レスポンスに `errors` があれば、その内容を確認する

```graphql
query {
  user(login: "octocat") {
    contributionsCollection {
      contributionCalendar {
        totalContributions
        weeks {
          contributionDays {
            date
            contributionCount
          }
        }
      }
    }
  }
}
```

参照

<ol start="3">
  <li id="ref-3"><a href="https://docs.github.com/en/rest/activity/events">GitHub Docs — REST API endpoints for events</a></li>
  <li id="ref-4"><a href="https://docs.github.com/en/graphql/reference/users#contributioncalendar">GitHub Docs — ContributionCalendar・ContributionCalendarDay・User</a></li>
  <li id="ref-5"><a href="https://docs.github.com/en/graphql/guides/forming-calls-with-graphql">GitHub Docs — Forming calls with GraphQL</a></li>
</ol>

## SVGから日付と件数を読み取る

### 取得元の形式を確認する

- SVGは、図形や文字をXML形式で記述する画像形式である — `<rect>` は四角形を表す要素である<sup><a href="#ref-6">[6]</a></sup>
- 2026年10月28日に確認した `https://github.com/users/torvalds/contributions` のレスポンスは `text/html` であり、日ごとのマスは `<td>`、件数は `<tool-tip>` に入っていた<sup><a href="#ref-7">[7]</a></sup>
- このURLをSVG画像として解析しても日別件数は取得できない — 現在のHTMLを読むコードは次のセクションに示す
- SVGを読み取る方法が使えるのは、手元のSVGや別の生成元のSVGに、日付と件数が保存されている場合である
- ブラウザーの開発者ツールで「Network」を開き、対象のレスポンスの `Content-Type` と本文を確認する — SVGの標準的なメディアタイプは `image/svg+xml` で、本文には `<svg>` がある
- URLの末尾を `.svg` に変えるだけでは、GitHubがSVGを返すようにはならない

### SVG内のデータを確認する

- 次は日付を `data-date`、件数を `data-count` に持たせた説明用のSVGである — 現在のGitHubのレスポンスを写したものではない
- `data-*` は独自のデータを保持する属性である — `data-date` や `data-count` がすべてのSVGに備わっているわけではない
- `fill` の色は件数そのものではない — 同じ色に複数の件数が対応する場合、色だけから正確な件数は復元できない

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 40 10">
  <rect x="0"  y="0" width="10" height="10"
    data-date="2026-10-26" data-count="0" fill="#ebedf0" />
  <rect x="15" y="0" width="10" height="10"
    data-date="2026-10-27" data-count="2" fill="#9be9a8" />
  <rect x="30" y="0" width="10" height="10"
    data-date="2026-10-28" data-count="5" fill="#40c463" />
</svg>
```

### JavaScriptでSVGを解析する

- 次の関数はブラウザーで実行する — `DOMParser` でSVG文字列をXMLとして解析し、各マスを `{ date, count }` に変換する<sup><a href="#ref-8">[8]</a></sup>
- `parsererror` とルート要素を確認し、壊れたXMLやSVG以外のデータを拒否する
- マスがない場合や件数が不正な場合はエラーにする — 読み取り失敗を「0件」と扱わない
- 集計する前に日付で並べ替える — SVG内の要素が日付順に並んでいるとは限らない

```js
function parseContributionSvg(svgText) {
  const doc = new DOMParser().parseFromString(svgText, 'image/svg+xml');
  if (doc.querySelector('parsererror') || doc.documentElement.localName !== 'svg') {
    throw new Error('有効なSVGではない');
  }

  const cells = [...doc.querySelectorAll('rect[data-date][data-count]')];
  if (cells.length === 0) {
    throw new Error('日付と件数を持つマスが見つからない');
  }

  const days = cells.map((cell) => {
    const date = cell.getAttribute('data-date');
    const rawCount = cell.getAttribute('data-count');
    const count = Number(rawCount);
    if (!/^\d{4}-\d{2}-\d{2}$/.test(date) || !/^\d+$/.test(rawCount)
      || !Number.isSafeInteger(count)) {
      throw new Error('日付または件数の形式が不正');
    }
    return { date, count };
  });

  return days.sort((a, b) => a.date.localeCompare(b.date));
}
```

- SVGのURLがある場合は、次の関数で取得してから解析する — 対象は日付・件数の属性を持つSVGを配信しているURLである
- 別のオリジンから取得する場合は、配信元がCORSによる読み取りを許可している必要がある — 許可されていなければサーバー側で取得する<sup><a href="#ref-9">[9]</a></sup>

```js
async function fetchContributionSvg(url) {
  const response = await fetch(url);
  if (!response.ok) throw new Error(`取得失敗: ${response.status}`);

  const type = response.headers.get('content-type') ?? '';
  if (!type.toLowerCase().startsWith('image/svg+xml')) {
    throw new Error('取得先はSVGを返していない');
  }

  const days = parseContributionSvg(await response.text());
  return {
    days,
    total: days.reduce((sum, day) => sum + day.count, 0),
  };
}
```

- 説明用SVGを `parseContributionSvg()` に渡すと、0件・2件・5件の3日分を取得でき、合計は7件になる
- この合計はSVGに含まれる日付の合計である — 過去1年間の値と呼ぶには、SVGが含む期間も確認する必要がある
- SVGの生成元が属性名を変えたり、件数を省いたりした場合は、解析方法の変更が必要になる

参照

<ol start="6">
  <li id="ref-6"><a href="https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/rect">MDN — &lt;rect&gt;</a></li>
  <li id="ref-7"><a href="https://github.com/users/torvalds/contributions">GitHub — 公開コントリビューションカレンダーのHTML</a></li>
  <li id="ref-8"><a href="https://developer.mozilla.org/en-US/docs/Web/API/DOMParser/parseFromString">MDN — DOMParser: parseFromString()</a></li>
  <li id="ref-9"><a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS">MDN — Cross-Origin Resource Sharing (CORS)</a></li>
</ol>

## 現在のGitHubのHTMLから取得する

### 日付とツールチップを対応させる

- `https://github.com/users/{username}/contributions` から、コントリビューションカレンダーのHTMLを取得できる — 公式APIとして文書化されたエンドポイントではない
- 2026年10月28日の確認では、日付は `<td>` の `data-date`、色の段階は `data-level` に入っていた
- 件数はマスの `id` と同じ値を `for` に持つ `<tool-tip>` のテキストから読み取る — `data-level` は正確な件数を表さない
- 次の断片は実際のレスポンスの構造を示すために属性を省略したものである<sup><a href="#ref-10">[10]</a></sup>

```html
<td
  id="contribution-day-component-0-0"
  data-date="2025-10-26"
  data-level="1"
></td>
<tool-tip for="contribution-day-component-0-0">
  10 contributions on October 26th.
</tool-tip>
```

### ブラウザーで取得してJSONにする

- 次のコードは `https://github.com` のページを開いたブラウザーの開発者ツールのConsoleで実行する
- `username` を対象のユーザー名に置き換える — `credentials: 'omit'` を指定し、ログイン用Cookieを付けずに取得する
- 合計は `#js-contribution-activity-description` の見出しから、日別件数は各ツールチップから読み取る
- `No contributions` は0件、`1 contribution` は1件、`1,234 contributions` は1,234件として扱う
- 説明文が見つからない場合や形式が変わった場合はエラーにする

```js
async function fetchContributionCalendar(username) {
  const url = `https://github.com/users/${encodeURIComponent(username)}/contributions`;
  const response = await fetch(url, { credentials: 'omit' });
  if (!response.ok) throw new Error(`取得失敗: ${response.status}`);

  const doc = new DOMParser().parseFromString(await response.text(), 'text/html');
  const heading = doc.querySelector('#js-contribution-activity-description');
  const totalMatch = heading?.textContent.trim().match(/^([\d,]+)\s+contributions?\b/);
  const cells = [...doc.querySelectorAll('td[data-date]')];
  if (!totalMatch || cells.length === 0) {
    throw new Error('カレンダーが見つからないか、HTMLの構造が変わった');
  }

  const tooltips = new Map(
    [...doc.querySelectorAll('tool-tip[for]')].map((tip) => [
      tip.getAttribute('for'), tip.textContent.trim(),
    ]),
  );

  const days = cells.map((cell) => {
    const date = cell.getAttribute('data-date');
    const text = tooltips.get(cell.id) ?? '';
    const match = text.match(/^(No|\d[\d,]*) contributions?\b/);
    if (!/^\d{4}-\d{2}-\d{2}$/.test(date) || !match) {
      throw new Error(`日付または件数を読み取れない: ${date}`);
    }
    const count = match[1] === 'No' ? 0 : Number(match[1].replaceAll(',', ''));
    if (!Number.isSafeInteger(count)) throw new Error('件数が不正');
    return { date, count };
  });

  return {
    username,
    total: Number(totalMatch[1].replaceAll(',', '')),
    days: days.sort((a, b) => a.date.localeCompare(b.date)),
  };
}

const result = await fetchContributionCalendar('torvalds');
console.log(JSON.stringify(result, null, 2));
```

- 返り値の `total` が画面の見出しの合計、`days` が取得したマスの日付と件数である
- カレンダーの端に表示される日付と、見出しが集計する期間を同一と決めつけない — 画面の合計が必要なら `total` を使う
- 自分のサイトからこのURLを直接 `fetch()` する場合はCORSの制約を受ける — サーバーや静的サイトのビルド時に取得する構成では、HTMLを解析できるライブラリも用意する
- Node.jsの標準環境にはブラウザーの `DOMParser` がないため、このコードをそのままNode.jsで実行することはできない

参照

<ol start="10">
  <li id="ref-10"><a href="https://github.com/users/torvalds/contributions">GitHub — コントリビューションカレンダーのレスポンス</a></li>
</ol>

## 取得結果を使うときの注意点

- 継続して使う機能には、取得項目が公式に定義されているGraphQL APIを選ぶ — HTMLの属性名や英語の説明文は変更される可能性がある
- HTMLから読み取る場合は結果をキャッシュし、ページの表示ごとに同じカレンダーを取得しない
- 通信失敗や解析失敗を0件として保存しない — 直前に取得できた値があれば取得日時とともに保持し、更新失敗と区別する
- 非公開リポジトリの活動が含まれる範囲は、ユーザーの公開設定や取得時の認証・権限に左右される — 匿名で読んだHTMLと、認証したAPIの結果が常に一致するとは限らない<sup><a href="#ref-11">[11]</a></sup>
- GitHubのコントリビューションはUTCを基準に記録される — 日本時間の日別件数と同じ意味で扱わない<sup><a href="#ref-12">[12]</a></sup>

参照

<ol start="11">
  <li id="ref-11"><a href="https://docs.github.com/en/graphql/reference/users#contributionscollection">GitHub Docs — ContributionsCollection</a></li>
  <li id="ref-12"><a href="https://docs.github.com/en/account-and-profile/concepts/contributions-on-your-profile">GitHub Docs — Contributions on your profile</a></li>
</ol>

## まとめ

- コントリビューションの合計はコミット数や活動した日数とは異なる
- REST APIのイベント一覧から、過去1年間のカレンダーをそのまま復元することはできない
- GraphQL APIでは `totalContributions` と日別の `contributionCount` を取得できる
- SVGに日付と件数が保存されていれば解析できるが、色だけから正確な件数は求められない
- 現在のGitHubのカレンダーはHTMLであり、日付の属性とツールチップのテキストを対応させて取得する
- HTMLからの取得は表示構造の変更に影響されるため、エラー検知とキャッシュを用意する
- 合計や日別件数を比較するときは、対象期間、公開設定、認証状態、UTCの基準をそろえる
