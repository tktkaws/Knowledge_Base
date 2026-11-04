# 内閣府の祝日CSVをJSONに変換する — WordPressのFullCalendarに祝日を表示する

## WordPressで祝日データを扱う構成

- WordPress上にFullCalendarでカレンダーを作り、内閣府が公開するCSVから取得した祝日・休日を表示する
- PHPでCSVを取得してUTF-8へ変換し、日付と名称をJSONにまとめる — ブラウザーはWordPressからJSONを取得する
- 変換したJSONはWordPressのデータベースに保存し、独自のREST APIから配信する構成とする
- CSVの取得はプラグインの有効化時と定期更新時に行う — カレンダーを開くたびに内閣府へアクセスしない
- 取得や変換に失敗したときは、前回のJSONを残す
- 対象は国民の祝日と、振替休日・祝日に挟まれた休日 — 土日、会社独自の休業日、年末年始の休暇は別に扱う
- 以下の例はPHP 8.2以降と`mbstring`拡張を使用する — FullCalendarは6.1.22にバージョンを固定し、v6のAPIで記述する
- 内容の確認日は2026年10月20日 — この時点の内閣府CSVには1955年から2027年までのデータが掲載されている

```text
内閣府のCSV
  ↓ PHPで取得・文字コード変換・CSV解析
JSONをWordPressのデータベースへ保存
  ↓ WordPressのREST APIから配信
ブラウザーのJavaScriptでJSONを取得
  ↓ 日付と名称を終日イベントへ変換
FullCalendarに祝日・休日を表示
```

- 実装は小さな独自プラグインにまとめる — PHPのコードブロックは、掲載順に同じ`jp-holiday-calendar.php`へ記述する
- JavaScriptとCSSを含む3ファイルを用意してから、WordPress管理画面でプラグインを有効化する

```text
wp-content/plugins/jp-holiday-calendar/
├── jp-holiday-calendar.php
├── calendar.js
└── calendar.css
```

> 参照: [内閣府 — 「国民の祝日」について](https://www8.cao.go.jp/chosei/shukujitsu/gaiyou.html) / [FullCalendar v6 — Initialize with Script Tags](https://legacy.fullcalendar.io/v6/initialize-globals)

## PHPでCSVを解析し、JSONとして保存する

### CSVの文字コードと日付をそろえる

- CSVの取得URLは`https://www8.cao.go.jp/chosei/shukujitsu/syukujitsu.csv`
- CSVはShift_JIS系の文字コードなので、`mb_convert_encoding()`でUTF-8へ変換する
- CSVの解析には`fgetcsv()`を使う — カンマや改行で単純に分割する方法では、引用符付きの値を正しく扱えない
- `2026/1/1`のような日付を`2026-01-01`にそろえる — `checkdate()`で存在する日付か確認する
- 内閣府CSVで名称が「休日」の行も残す — 振替休日や祝日に挟まれた休日が含まれるため、名前だけで除外しない
- 最初に次のコードを`jp-holiday-calendar.php`へ記述する — PHPの閉じタグは付けない

```php
<?php
/**
 * Plugin Name: JP Holiday Calendar
 * Description: 内閣府のCSVから取得した祝日をFullCalendarに表示する
 * Version: 1.0.0
 * Requires PHP: 8.2
 */

defined('ABSPATH') || exit;

const JPH_CSV_URL = 'https://www8.cao.go.jp/chosei/shukujitsu/syukujitsu.csv';

function jph_parse_csv(string $bytes): array {
    if (!function_exists('mb_convert_encoding')) {
        throw new RuntimeException('mbstring拡張が必要');
    }
    if (!mb_check_encoding($bytes, 'SJIS-win')) {
        throw new RuntimeException('CSVの文字コードを確認できない');
    }

    $csv = mb_convert_encoding($bytes, 'UTF-8', 'SJIS-win');
    $stream = fopen('php://temp', 'w+');
    if ($stream === false) {
        throw new RuntimeException('CSV用のストリームを開けない');
    }

    try {
        fwrite($stream, $csv);
        rewind($stream);
        $header = fgetcsv($stream, 0, ',', '"', '');
        if ($header !== ['国民の祝日・休日月日', '国民の祝日・休日名称']) {
            throw new RuntimeException('CSVのヘッダーが想定と異なる');
        }

        $holidays = [];
        while (($row = fgetcsv($stream, 0, ',', '"', '')) !== false) {
            if ($row === [null]) continue;
            if (count($row) !== 2) {
                throw new RuntimeException('CSVの列数が想定と異なる');
            }

            [$raw_date, $name] = $row;
            if (!preg_match('~^(\d{4})/(\d{1,2})/(\d{1,2})$~', $raw_date, $m)) {
                throw new RuntimeException('日付の形式が不正');
            }
            $year = (int) $m[1];
            $month = (int) $m[2];
            $day = (int) $m[3];
            if (!checkdate($month, $day, $year) || trim($name) === '') {
                throw new RuntimeException('日付または名称が不正');
            }

            $date = sprintf('%04d-%02d-%02d', $year, $month, $day);
            if (isset($holidays[$date])) {
                throw new RuntimeException('日付が重複している');
            }
            $holidays[$date] = trim($name);
        }
    } finally {
        fclose($stream);
    }

    if ($holidays === []) {
        throw new RuntimeException('祝日データが空');
    }
    ksort($holidays);

    $years = array_unique(array_map(
        static fn($date) => (int) substr($date, 0, 4),
        array_keys($holidays)
    ));
    sort($years);
    if ($years !== range(min($years), max($years))) {
        throw new RuntimeException('祝日データに年単位の欠落がある');
    }

    return [
        'first_year' => min($years),
        'last_year' => max($years),
        'holidays' => $holidays,
    ];
}
```

> 参照: [PHP — mb_convert_encoding](https://www.php.net/manual/ja/function.mb-convert-encoding.php) / [PHP — fgetcsv](https://www.php.net/manual/ja/function.fgetcsv.php) / [内閣府の祝日・休日一覧CSV](https://www8.cao.go.jp/chosei/shukujitsu/syukujitsu.csv)

### 全件の解析が成功した場合だけJSONを更新する

- WordPressのHTTP APIでCSVを取得し、通信エラーとHTTPステータスを確認する
- すべての行を解析した後で`wp_json_encode()`を実行し、Options APIでJSON文字列を保存する
- 有効期限で消えるキャッシュにはせず、次の更新が失敗しても最後に取得できたデータが残るようにする
- 出典、取得日時、収録年もJSONに含める — フロントエンドで表示可能な範囲を判断するために使う
- 続けて、同じPHPファイルへ次のコードを追記する

```php
function jph_refresh() {
    $response = wp_remote_get(JPH_CSV_URL, [
        'timeout' => 15,
        'limit_response_size' => 1024 * 1024,
    ]);
    if (is_wp_error($response)) return $response;
    if (wp_remote_retrieve_response_code($response) !== 200) {
        return new WP_Error('jph_http', 'CSVの取得に失敗');
    }

    try {
        $data = jph_parse_csv(wp_remote_retrieve_body($response));
        $data['source'] = JPH_CSV_URL;
        $data['fetched_at'] = gmdate('c');
        $json = wp_json_encode($data, JSON_UNESCAPED_UNICODE);
        if ($json === false) {
            throw new RuntimeException('JSONへの変換に失敗');
        }
    } catch (Throwable $e) {
        return new WP_Error('jph_parse', $e->getMessage());
    }

    update_option('jph_data_json', $json, false);
    if (get_option('jph_data_json') !== $json) {
        return new WP_Error('jph_save', 'JSONの保存に失敗');
    }
    return true;
}
```

- ヘッダーや日付の検証は、形式の崩れを見つけるためのもの — 個々の祝日の欠落まで検出できるわけではないため、運用では更新前後の件数や日付も確認する

> 参照: [WordPress — wp_remote_get](https://developer.wordpress.org/reference/functions/wp_remote_get/) / [wp_json_encode](https://developer.wordpress.org/reference/functions/wp_json_encode/) / [update_option](https://developer.wordpress.org/reference/functions/update_option/)

## 定期更新とJSONを返すREST APIを用意する

### 有効化時と1日1回の更新を登録する

- プラグインの有効化時に初回の取得を実行し、WP-Cronへ1日1回の更新を登録する
- 失敗内容は`jph_last_error`へ保存する — 更新に失敗しても`jph_data_json`は削除しない
- 無効化時には定期処理の予約を解除する
- 次のコードも同じPHPファイルへ追記する

```php
function jph_run_refresh(): void {
    $result = jph_refresh();
    update_option(
        'jph_last_error',
        is_wp_error($result) ? $result->get_error_message() : '',
        false
    );
}

add_action('jph_refresh_event', 'jph_run_refresh');

register_activation_hook(__FILE__, function () {
    if (!wp_next_scheduled('jph_refresh_event')) {
        wp_schedule_event(time() + DAY_IN_SECONDS, 'daily', 'jph_refresh_event');
    }
    jph_run_refresh();
});

register_deactivation_hook(__FILE__, function () {
    wp_clear_scheduled_hook('jph_refresh_event');
});
```

- WP-Cronは通常、アクセスをきっかけに実行される — 「daily」は毎日決まった時刻の実行を保証するものではない
- アクセスが少ないサイトやWP-Cronを無効にしている環境では、サーバー側のcronなどで実行機会を用意する

> 参照: [WordPress — Scheduling WP Cron Events](https://developer.wordpress.org/plugins/cron/scheduling-wp-cron-events/) / [WP-Cronの仕組み](https://developer.wordpress.org/plugins/cron/)

### 保存済みデータだけをJSONとして配信する

- `/wp-json/jp-holiday-calendar/v1/holidays`から保存済みデータを返す
- 公開の祝日データを読み取るAPIなので、`permission_callback`には`__return_true`を指定する
- APIの呼び出しではCSVを取得しない — ブラウザーからのアクセスが増えても内閣府へのリクエストは増えない
- 初回取得に成功していない場合は、空の祝日一覧の代わりにHTTP 503を返す
- 保存したJSON文字列を一度配列に戻して返す — WordPressがレスポンスをJSON化するため、JSON文字列の二重エンコードを避ける

```php
add_action('rest_api_init', function () {
    register_rest_route('jp-holiday-calendar/v1', '/holidays', [
        'methods' => 'GET',
        'permission_callback' => '__return_true',
        'callback' => function () {
            $data = json_decode(get_option('jph_data_json', ''), true);
            if (!is_array($data) || empty($data['holidays'])) {
                return new WP_Error(
                    'jph_unavailable',
                    '祝日データをまだ取得できていない',
                    ['status' => 503]
                );
            }

            $response = rest_ensure_response($data);
            $response->header('Cache-Control', 'no-store');
            return $response;
        },
    ]);
});
```

- 返却形式は次のようなオブジェクト — `holidays`の中身は抜粋で、実際には収録範囲の全件を含む

```json
{
  "first_year": 1955,
  "last_year": 2027,
  "holidays": {
    "2026-01-01": "元日",
    "2026-05-06": "休日",
    "2026-09-22": "休日"
  },
  "source": "https://www8.cao.go.jp/chosei/shukujitsu/syukujitsu.csv",
  "fetched_at": "2026-10-20T00:00:00+00:00"
}
```

> 参照: [WordPress — register_rest_route](https://developer.wordpress.org/reference/functions/register_rest_route/) / [rest_ensure_response](https://developer.wordpress.org/reference/functions/rest_ensure_response/)

## FullCalendarでJSONから祝日を表示する

### ショートコードとスクリプトの読み込みを追加する

- 次のコードを同じPHPファイルの末尾へ追記する
- 固定ページのショートコードブロックに`[jp_holiday_calendar]`を置く構成 — 投稿本文にこのショートコードがあるページだけでJSとCSSを読み込む
- FullCalendar本体、日本語ロケール、独自のJavaScriptの順に依存関係を設定する
- REST APIのURLは`rest_url()`で生成する — WordPressの設置パスやパーマリンク設定に合わせたURLを渡せる
- カレンダーの下に内閣府への出典リンクを表示する

```php
add_action('wp_enqueue_scripts', function () {
    $post = get_post();
    if (!is_singular() || !$post ||
        !has_shortcode($post->post_content, 'jp_holiday_calendar')) {
        return;
    }

    wp_enqueue_script(
        'jph-fullcalendar',
        'https://cdn.jsdelivr.net/npm/fullcalendar@6.1.22/index.global.min.js',
        [], '6.1.22', true
    );
    wp_enqueue_script(
        'jph-locale',
        'https://cdn.jsdelivr.net/npm/@fullcalendar/core@6.1.22/locales/ja.global.min.js',
        ['jph-fullcalendar'], '6.1.22', true
    );
    wp_enqueue_script(
        'jph-calendar',
        plugins_url('calendar.js', __FILE__),
        ['jph-locale'], '1.0.0', true
    );
    wp_enqueue_style(
        'jph-calendar', plugins_url('calendar.css', __FILE__), [], '1.0.0'
    );
});

add_shortcode('jp_holiday_calendar', function () {
    $url = esc_url(rest_url('jp-holiday-calendar/v1/holidays'));
    return '<div class="jph-calendar" data-holidays-url="' . $url . '">'
        . '<p class="jph-status" role="status">祝日データを読み込み中</p>'
        . '<div class="jph-view"></div>'
        . '<p>出典：<a href="https://www8.cao.go.jp/chosei/shukujitsu/gaiyou.html">'
        . '内閣府「国民の祝日」について</a></p></div>';
});
```

- テーマや別のプラグインですでにFullCalendarを読み込んでいる場合は、本体の読み込みを一本化する
- この条件分岐は通常の投稿・固定ページの本文を対象とする — テンプレート内の`do_shortcode()`やページビルダーで使う場合は、その表示条件に合わせて読み込み処理を変更する

> 参照: [WordPress — wp_enqueue_script](https://developer.wordpress.org/reference/functions/wp_enqueue_script/) / [add_shortcode](https://developer.wordpress.org/reference/functions/add_shortcode/) / [FullCalendar v6 — locale](https://legacy.fullcalendar.io/v6/locale)

### 日付と名称を終日イベントに変換する

- 次のコードを`calendar.js`へ記述する
- JSONを取得し、`holidays`をFullCalendarのイベント配列へ変換する
- 日付は`YYYY-MM-DD`の文字列のまま`start`へ渡し、`allDay: true`を指定する — 祝日に時刻やUTC変換を持ち込まない
- `validRange`で収録年の範囲に移動を制限する — 未収録の年を、祝日のない年として表示しない
- 現在の年が収録範囲外なら範囲内の年を開き、その旨をメッセージで伝える

```js
document.addEventListener('DOMContentLoaded', () => {
  document.querySelectorAll('.jph-calendar').forEach(async (root) => {
    const status = root.querySelector('.jph-status');
    try {
      if (typeof FullCalendar === 'undefined') {
        throw new Error('FullCalendarを読み込めない');
      }

      const response = await fetch(root.dataset.holidaysUrl, {
        signal: AbortSignal.timeout(10000),
      });
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      const data = await response.json();
      if (!data.holidays || !Number.isInteger(data.first_year) ||
          !Number.isInteger(data.last_year) || data.first_year > data.last_year) {
        throw new Error('祝日JSONの形式が不正');
      }

      const currentYear = new Date().getFullYear();
      const inRange = currentYear >= data.first_year && currentYear <= data.last_year;
      const initialYear = Math.min(Math.max(currentYear, data.first_year), data.last_year);
      const events = Object.entries(data.holidays).map(([date, name]) => ({
        id: `holiday-${date}`,
        title: name,
        start: date,
        allDay: true,
        classNames: ['jph-holiday'],
      }));

      const calendar = new FullCalendar.Calendar(root.querySelector('.jph-view'), {
        locale: 'ja',
        initialView: 'dayGridMonth',
        initialDate: inRange ? new Date() : `${initialYear}-01-01`,
        validRange: {
          start: `${data.first_year}-01-01`,
          end: `${data.last_year + 1}-01-01`,
        },
        headerToolbar: { left: 'prev,next today', center: 'title', right: '' },
        eventSources: [{ events }],
        height: 'auto',
      });
      calendar.render();

      const updated = new Date(data.fetched_at);
      const age = Date.now() - updated.getTime();
      status.textContent = `${data.first_year}〜${data.last_year}年の祝日・休日を表示`;
      if (!inRange) status.textContent += '／現在の年のデータは未収録';
      if (!Number.isFinite(age) || age > 7 * 24 * 60 * 60 * 1000) {
        status.textContent += '／取得日時が古いため更新状況の確認が必要';
      }
    } catch (error) {
      status.textContent = '祝日カレンダーを読み込めないため、時間をおいて再度確認してください';
      console.error('祝日カレンダー:', error);
    }
  });
});
```

- このAPIは収録年などの情報を持つ独自形式なので、URLをそのままFullCalendarの`events`へ渡さず、`holidays`からイベント配列を作る
- 通常の予定も表示する場合は、`eventSources`へ既存の予定用ソースを追加する — 祝日用ソースと一緒に表示できる
- `end`は含まない境界なので、最終年の翌年1月1日を指定する

> 参照: [FullCalendar v6 — Event Object](https://legacy.fullcalendar.io/v6/event-object) / [eventSources](https://legacy.fullcalendar.io/v6/eventSources) / [validRange](https://legacy.fullcalendar.io/v6/validRange)

### 祝日名を見分けやすくする

- 次のコードを`calendar.css`へ記述する
- 色に加えて祝日・休日の名称も表示する — 内閣府CSVで「休日」とされている名称は、そのまま表示する
- FullCalendar v6のグローバルバンドルは必要なスタイルをJavaScriptから追加するため、ここでは独自の装飾だけを指定する

```css
.jph-calendar .jph-holiday {
  --fc-event-bg-color: #fff2f2;
  --fc-event-border-color: #9f2222;
  --fc-event-text-color: #7b1616;
  color: #7b1616;
}

.jph-calendar .jph-holiday .fc-event-title {
  white-space: normal;
}
```

> 参照: [FullCalendar v6 — Upgrading from v5](https://legacy.fullcalendar.io/v6/upgrading-from-v5)

## 表示と更新を確認する

### 有効化後に確認する項目

- WordPress管理画面のプラグイン一覧で「JP Holiday Calendar」を有効化する
- 固定ページにショートコードブロックを追加し、`[jp_holiday_calendar]`を入力してプレビューする
- ブラウザーの開発者ツールで、祝日APIがHTTP 200を返し、`holidays`に日付と名称が入っているか確認する
- 2026年なら1月1日の元日、5月6日の振替休日、9月22日の祝日に挟まれた休日を確認する — 後ろの2日はこの実装では「休日」と表示される
- 収録範囲より先の年へ移動できないことと、通常の予定を追加した場合に祝日と併存することを確認する
- 読み込みに失敗した場合は、画面のメッセージ、ブラウザーのコンソール、REST APIのレスポンスを確認する

> 参照: [内閣府 — 2026年・2027年の国民の祝日・休日](https://www8.cao.go.jp/chosei/shukujitsu/gaiyou.html)

### 更新が続いているか確認する

- WP-CLIを利用できる環境なら、次のコマンドで定期処理を手動実行し、直近の取得エラーを確認できる
- `jph_last_error`は成功時に空文字になる — 保存済みJSONの`fetched_at`と合わせて確認する
- 更新失敗時は過去のデータが表示され続けるため、運用では取得日時を監視する — 例では7日以上古い場合に画面へメッセージを追加している
- 翌年分の公開後や祝日の変更を確認した際にも再取得し、内閣府の一覧と照合する
- キャッシュプラグインやCDNを使う場合は、このREST APIに古いレスポンスが残らない設定を確認する
- 例ではFullCalendarをCDNから取得している — JavaScriptの配信も自サイトで管理する場合は、本体と日本語ロケールをプラグイン内へ配置して読み込み先を変更する

```sh
wp cron event run jph_refresh_event
wp option get jph_last_error
```

- 定期処理の実行が成功しても、CSVの取得が成功したとは限らない — コールバックがエラーを記録する構成なので、上の2コマンドと`fetched_at`を合わせて確認する

> 参照: [WP-CLI — wp cron event run](https://developer.wordpress.org/cli/commands/cron/event/run/) / [wp option get](https://developer.wordpress.org/cli/commands/option/get/)

## まとめ

- 内閣府CSVをWordPressのPHPで取得し、文字コードと日付の形式をそろえてJSONへ変換する
- JSONはWordPressのデータベースへ保存し、REST APIでブラウザーへ渡す
- 取得処理をプラグイン有効化時と定期更新時にまとめ、画面表示のたびに内閣府へアクセスしない
- FullCalendarでは日付と名称を`start`・`title`へ変換し、`allDay: true`で表示する
- 取得に失敗した場合は保存済みデータを維持し、未収録の年は表示範囲に含めない
- 更新日時とエラーを確認する運用を用意し、JSONが取得できることと祝日情報が更新されていることを分けて確認する
