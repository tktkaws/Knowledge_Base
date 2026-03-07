# aria-live — 動的コンテンツの変更を通知する

## aria-liveとは

- ページの再読み込みなしに変化するコンテンツ領域を「ライブリージョン」として定義する属性
- ライブリージョン内のテキストが更新されると、スクリーンリーダーが自動的に読み上げる
- フォーカスの移動なしに、画面上の変更を支援技術に伝える唯一の手段

```html
<div aria-live="polite">
  <!-- ここの内容が変更されるとスクリーンリーダーが読み上げる -->
</div>
```

- 参照: [WAI-ARIA 1.2 — aria-live](https://www.w3.org/TR/wai-aria-1.2/#aria-live)

## なぜ必要か

- スクリーンリーダーは通常、フォーカスのある要素しか読み上げない
- ページの別の場所で内容が変わっても、ユーザーはその変化に気づけない
- 視覚的には見える変化（トースト通知、検索結果件数の更新など）を非視覚的にも伝える必要がある

### aria-liveがないと困る例

```
ユーザーがフォームを送信
  → フォーカスは送信ボタンのまま
  → 画面上部に「保存しました」のメッセージが表示される
  → スクリーンリーダーは何も読み上げない
  → ユーザーは成功したのか失敗したのかわからない
```

## aria-liveの3つの値

| 値 | 動作 | 用途 |
|---|---|---|
| `off` | 変更を通知しない（デフォルト） | ライブリージョンを無効化する場合 |
| `polite` | 現在の読み上げが終わってから通知 | ほとんどのケースで推奨 |
| `assertive` | 現在の読み上げを中断して即座に通知 | 緊急性の高いエラーや警告のみ |

### polite

- 現在の読み上げを中断しない、礼儀正しい(polite)通知
- ユーザーの作業を妨げずに情報を伝えるため、大半のケースで適切
- ステータスメッセージ、検索結果の更新、保存完了通知などに使用

```html
<!-- 検索結果の件数表示 -->
<p aria-live="polite">検索結果: 42件</p>

<!-- 保存完了メッセージ -->
<div aria-live="polite" id="status-message"></div>

<script>
  document.getElementById('status-message').textContent = '保存しました';
  // → スクリーンリーダーが現在の読み上げ終了後に「保存しました」と読み上げ
</script>
```

### assertive

- 現在の読み上げを即座に中断して通知する、緊急性の高い通知
- ユーザーの作業を中断するため、本当に緊急な場合のみ使用
- セッション切れの警告、致命的なエラー、時間制限の通知などに限定

```html
<!-- セッションタイムアウト警告 -->
<div aria-live="assertive">
  セッションが60秒後に切れます。延長しますか？
</div>
```

### off

- ライブリージョンの通知を無効にする
- 一時的に通知を止めたい場合に使用

```html
<!-- 一括更新中は通知を抑制 -->
<ul aria-live="off" id="log">
  <!-- 大量のログ行が追加される間は通知しない -->
</ul>
```

- 参照: [MDN — aria-live](https://developer.mozilla.org/ja/docs/Web/Accessibility/ARIA/Attributes/aria-live)

## 関連属性

### aria-atomic

- `true`：ライブリージョン全体を読み上げる
- `false`（デフォルト）：変更された部分のみ読み上げる

```html
<!-- 時刻表示：分だけ変わっても全体を読み上げたい -->
<div aria-live="polite" aria-atomic="true">
  現在時刻: <span>14</span>時<span>30</span>分
</div>
<!-- aria-atomic="true"の場合：「現在時刻: 14時31分」 -->
<!-- aria-atomic="false"の場合：「31」だけ読み上げ -->
```

### aria-relevant

- ライブリージョン内のどの種類の変更を通知するか指定する属性
- デフォルト値: `additions text`

| 値 | 通知する変更 |
|---|---|
| `additions` | 子要素の追加 |
| `removals` | 子要素の削除 |
| `text` | テキスト内容の変更 |
| `all` | すべての変更（`additions removals text`と同等） |

```html
<!-- チャットログ：メッセージの追加のみ通知 -->
<div aria-live="polite" aria-relevant="additions">
  <p>ユーザーA: こんにちは</p>
  <!-- 新しいメッセージが追加されると読み上げ -->
</div>

<!-- 参加者リスト：追加と削除の両方を通知 -->
<ul aria-live="polite" aria-relevant="additions removals">
  <li>田中さん</li>
  <li>鈴木さん</li>
</ul>
```

### aria-busy

- `true`：ライブリージョンが更新中であることを示す
- 複数回の連続した更新がある場合に、最終状態のみ通知するために使用
- 更新完了時に`false`に戻すと、その時点の内容が通知される

```html
<div aria-live="polite" aria-busy="true" id="results">
  <!-- データ読み込み中... -->
</div>

<script>
  const results = document.getElementById('results');
  // 読み込み開始
  results.setAttribute('aria-busy', 'true');

  fetchData().then(data => {
    results.innerHTML = `<p>${data.length}件の結果</p>`;
    // 読み込み完了 → この時点で内容が読み上げられる
    results.setAttribute('aria-busy', 'false');
  });
</script>
```

- 参照: [WAI-ARIA 1.2 — aria-atomic, aria-relevant, aria-busy](https://www.w3.org/TR/wai-aria-1.2/#aria-atomic)

## 暗黙のライブリージョンロール

- 一部のARIAロールは暗黙的にaria-liveの動作を持つ
- これらのロールを使う場合、aria-live属性の明示的な指定は不要

| ロール | 暗黙のaria-live値 | 用途 |
|---|---|---|
| `alert` | `assertive` | 重要な警告メッセージ |
| `status` | `polite` | 状態の変化を示す情報 |
| `log` | `polite` | ログ情報（チャット、エラーログなど） |
| `timer` | `off` | タイマー表示 |
| `marquee` | `off` | 繰り返し変化する情報 |

```html
<!-- role="status" は aria-live="polite" と同等 -->
<div role="status">保存しました</div>

<!-- role="alert" は aria-live="assertive" と同等 -->
<div role="alert">セッションが切れました</div>
```

- 参照: [WAI-ARIA 1.2 — Live Region Roles](https://www.w3.org/TR/wai-aria-1.2/#live_region_roles)

## 実装パターン

### パターン1: フォームのバリデーションメッセージ

```html
<form>
  <label for="email">メールアドレス</label>
  <input type="email" id="email" aria-describedby="email-error">
  <p id="email-error" role="alert"></p>
</form>

<script>
  const input = document.getElementById('email');
  const error = document.getElementById('email-error');

  input.addEventListener('blur', () => {
    if (!input.validity.valid) {
      error.textContent = '有効なメールアドレスを入力してください';
      // role="alert"により即座に読み上げられる
    } else {
      error.textContent = '';
    }
  });
</script>
```

### パターン2: トースト通知

```html
<!-- トーストのコンテナを事前にDOMに配置しておく -->
<div id="toast-container" aria-live="polite" role="status"></div>

<script>
  function showToast(message) {
    const container = document.getElementById('toast-container');
    container.textContent = message;
    // → スクリーンリーダーが「設定を更新しました」と読み上げ

    setTimeout(() => {
      container.textContent = '';
    }, 5000);
  }

  showToast('設定を更新しました');
</script>
```

### パターン3: 検索結果の件数更新

```html
<input type="search" aria-label="商品を検索">
<p aria-live="polite" id="search-count"></p>
<ul id="search-results"></ul>

<script>
  function updateResults(items) {
    document.getElementById('search-count').textContent =
      `${items.length}件の商品が見つかりました`;
    // → politeで読み上げ
  }
</script>
```

### パターン4: ローディング状態の通知

```html
<div aria-live="polite" id="content-area">
  <p>データを読み込んでいます...</p>
</div>

<script>
  const area = document.getElementById('content-area');

  // 読み込み開始時
  area.setAttribute('aria-busy', 'true');
  area.innerHTML = '<p>データを読み込んでいます...</p>';

  // 読み込み完了時
  fetchData().then(data => {
    area.innerHTML = renderData(data);
    area.setAttribute('aria-busy', 'false');
    // → 完了時点の内容が読み上げられる
  });
</script>
```

### Reactでの使用例

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    search(query).then(setResults);
  }, [query]);

  return (
    <>
      {/* aria-liveを持つ要素は初回レンダリング時からDOMに存在させる */}
      <p role="status">
        {results.length > 0
          ? `${results.length}件の結果`
          : '結果が見つかりませんでした'}
      </p>
      <ul>
        {results.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </>
  );
}
```

- 参照: [ARIA APG — Alert Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/alert/)

## よくある間違い

### 1. ライブリージョンを動的に追加する

```html
<!-- 悪い例：要素の追加と内容の設定を同時に行う -->
<script>
  const alert = document.createElement('div');
  alert.setAttribute('role', 'alert');
  alert.textContent = 'エラーが発生しました';
  document.body.appendChild(alert);
  // → 追加と同時にテキストが設定されるため、読み上げられない場合がある
</script>

<!-- 良い例：ライブリージョンを事前にDOMに配置し、テキストだけ変更する -->
<div role="alert" id="error-area"></div>
<script>
  document.getElementById('error-area').textContent = 'エラーが発生しました';
  // → 確実に読み上げられる
</script>
```

### 2. assertiveを多用する

```html
<!-- 悪い例：すべての通知をassertiveにする -->
<div aria-live="assertive">3件の新着メッセージがあります</div>
<div aria-live="assertive">自動保存しました</div>
<div aria-live="assertive">検索結果: 15件</div>
<!-- ユーザーの作業が常に中断される -->

<!-- 良い例：緊急性に応じて使い分ける -->
<div aria-live="polite">3件の新着メッセージがあります</div>
<div aria-live="polite">自動保存しました</div>
<div aria-live="polite">検索結果: 15件</div>
```

### 3. 大量のテキストをライブリージョンに流し込む

```html
<!-- 悪い例：長文がすべて読み上げられてしまう -->
<div aria-live="polite">
  <article>
    <h2>記事タイトル</h2>
    <p>長い本文が続く...</p>
    <p>さらに続く...</p>
  </article>
</div>

<!-- 良い例：ステータス情報だけをライブリージョンに入れる -->
<div aria-live="polite">記事を読み込みました</div>
<article>
  <h2>記事タイトル</h2>
  <p>長い本文が続く...</p>
</article>
```

### 4. ライブリージョンの中にフォーカス可能な要素を入れる

```html
<!-- 悪い例：ライブリージョン内にボタンを配置 -->
<div aria-live="polite">
  保存に失敗しました <button>再試行</button>
</div>

<!-- 良い例：通知とアクションを分離する -->
<div aria-live="polite">保存に失敗しました</div>
<button>再試行</button>
```

- ライブリージョン内のインタラクティブ要素にはフォーカスが移動しないため、ユーザーが操作できない可能性がある

## polite vs assertive の判断基準

```
その通知はユーザーの作業を中断してまで伝える必要があるか？
  ├── YES（安全やデータ損失に関わる）→ assertive
  │     例：セッション切れ、未保存データの警告、致命的エラー
  │
  └── NO（知っておくと便利な情報）→ polite
        例：保存完了、検索結果更新、新着通知
```

## まとめ

- `aria-live`はフォーカス移動なしにコンテンツの変化を支援技術に伝える属性
- ほとんどのケースで`polite`を使用、`assertive`は緊急時のみ
- ライブリージョンの要素は事前にDOMに配置し、テキストの変更で通知を発火させる
- `role="status"`（polite相当）や`role="alert"`（assertive相当）で暗黙的にライブリージョンになる
- 関連属性`aria-atomic`・`aria-relevant`・`aria-busy`で通知の挙動を細かく制御可能
- 大量のテキストやインタラクティブ要素はライブリージョンに含めない
