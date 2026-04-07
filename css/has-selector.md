# :has()セレクタの使い方と活用例

## :has()セレクタとは

- 特定の子要素や後続要素を**持っている親要素**を選択できるCSS擬似クラス
- 長年CSSに欠けていた「親セレクタ」をついに実現した機能
- 正確には**リレーショナル擬似クラス**と呼ばれ、親だけでなく兄弟要素の条件でも選択できる
- 従来はJavaScriptでしか実現できなかった条件付きスタイリングをCSSだけで記述可能

> 参照: [MDN - :has()](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)

## 基本構文

```css
/* 「imgを子に持つ.card」を選択 */
.card:has(img) {
  /* .cardにスタイルが適用される */
}
```

- `:has()` の引数にはセレクタリスト（カンマ区切りで複数可）を指定する
- **スタイルが適用されるのは `:has()` の前に書いた要素**（この例では `.card`）
- 引数のセレクタに一致する要素が**1つでもあれば**条件を満たす

```css
/* 基本的な書式 */
親セレクタ:has(子・子孫セレクタ) { ... }

/* 複数条件（OR — いずれかに一致） */
.card:has(img, video) { ... }
```

> 参照: [MDN - :has()](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)

## 子要素の有無で親をスタイリングする

### 画像のあるカードとないカードの出し分け

```html
<div class="card">
  <img src="photo.jpg" alt="写真">
  <h2>タイトル</h2>
  <p>本文テキスト</p>
</div>

<div class="card">
  <h2>タイトル</h2>
  <p>本文テキスト</p>
</div>
```

```css
/* 悪い例：JavaScriptでクラスを付与して分岐する */
/* <div class="card card--has-image"> のようにHTMLを変更する必要がある */

/* 良い例：:has()で子要素の有無をCSSだけで判定 */
.card:has(img) {
  display: grid;
  grid-template-columns: 200px 1fr;
  /* 画像があれば横並びレイアウト */
}

.card:not(:has(img)) {
  /* 画像がなければテキストのみのレイアウト */
  padding: 1.5rem;
}
```

- `:has()` と `:not()` を組み合わせることで「持っていない場合」も表現できる
- HTMLにバリエーション用のクラスを追加する必要がなくなる

> 参照: [web.dev - :has()](https://developer.chrome.com/blog/has-m105)

## 直接の子要素に限定する

```css
/* 子孫すべてを探索（デフォルト） */
.card:has(img) { ... }

/* 直接の子要素だけに限定 */
.card:has(> img) { ... }

/* 直接の子が.headerで、その中にimgがある */
.card:has(> .header img) { ... }
```

- `>` 結合子を使うと直接の子要素に限定できる
- 深い階層の要素を意図せずマッチさせたくない場合に有用

## フォームのバリデーション状態に応じたスタイリング

### 入力状態に応じたフィールドグループのスタイル

```css
/* 入力が無効な状態のフィールドグループ全体を強調 */
.form-group:has(:invalid) {
  border-left: 3px solid #dc3545;
  padding-left: 1rem;
}

/* 入力が有効な状態のフィールドグループ */
.form-group:has(:valid) {
  border-left: 3px solid #28a745;
  padding-left: 1rem;
}

/* エラーメッセージの表示切り替え */
.form-group .error-message {
  display: none;
}
.form-group:has(:invalid:not(:placeholder-shown)) .error-message {
  display: block;
  color: #dc3545;
}
```

```html
<div class="form-group">
  <label for="email">メールアドレス</label>
  <input id="email" type="email" placeholder="example@mail.com" required>
  <span class="error-message">有効なメールアドレスを入力してください</span>
</div>
```

- `:invalid`, `:valid`, `:checked` などの状態擬似クラスと組み合わせると強力
- 入力フィールド自体ではなく、その**親要素（ラベルやエラーメッセージを含むグループ全体）** にスタイルを適用できる

### チェックボックスの状態で親を変更

```css
/* チェックされたらカード全体のスタイルを変更 */
.selectable-card:has(input[type="checkbox"]:checked) {
  border-color: #0066cc;
  background-color: #f0f7ff;
  box-shadow: 0 0 0 2px #0066cc;
}
```

> 参照: [MDN - :has()](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)

## 兄弟要素の条件で選択する

- `:has()` は子要素だけでなく、**後続の兄弟要素**の条件でも選択できる

### 隣接兄弟結合子 `+` との組み合わせ

```css
/* 直後にpが続くh2 */
h2:has(+ p) {
  margin-bottom: 0.5rem;
}

/* 直後にpが続かないh2 */
h2:not(:has(+ p)) {
  margin-bottom: 1.5rem;
}
```

### 一般兄弟結合子 `~` との組み合わせ

```css
/* 後にエラーメッセージがあるinput */
input:has(~ .error) {
  border-color: #dc3545;
}
```

- 従来のCSSは「後続の兄弟」しか選択できなかったが、`:has()` により「前の兄弟」の条件に基づくスタイリングが可能になった

> 参照: [CSS-Tricks - :has()](https://css-tricks.com/the-css-has-selector/)

## :has()と他のセレクタの組み合わせ

### :has() + :not()

```css
/* 空のリスト（li を持たない ul） */
ul:not(:has(li)) {
  display: none;
}

/* 画像を持たないカードだけ背景色を変える */
.card:not(:has(img)) {
  background-color: #f5f5f5;
}
```

### :has() + :is() / :where()

```css
/* 画像またはビデオを含むセクション */
section:has(:is(img, video, iframe)) {
  padding: 2rem;
}

/* 詳細度を上げずにスタイルを適用 */
section:has(:where(img, video)) {
  margin-block: 2rem;
}
```

### 複数の:has()を連結（AND条件）

```css
/* 画像もボタンも持つカード */
.card:has(img):has(button) {
  /* 両方の条件を満たす場合のみ適用 */
}

/* :has()内のカンマはOR条件 */
.card:has(img, video) {
  /* 画像またはビデオを持つカード */
}
```

> 参照: [MDN - :has()](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)

## 実践的な活用例

### 空のグリッドセルの自動調整

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

/* 子要素が2つ以下なら2カラムに変更 */
.grid:not(:has(:nth-child(3))) {
  grid-template-columns: repeat(2, 1fr);
}

/* 子要素が1つだけなら1カラム */
.grid:not(:has(:nth-child(2))) {
  grid-template-columns: 1fr;
}
```

### ダークモード切り替えトグル

```css
/* :has()を使ったCSS-onlyダークモード */
html:has(#dark-mode-toggle:checked) {
  color-scheme: dark;
  --bg-color: #1a1a1a;
  --text-color: #e0e0e0;
}
```

```html
<input type="checkbox" id="dark-mode-toggle" hidden>
<label for="dark-mode-toggle">ダークモード切替</label>
```

- JavaScriptなしで `html` 要素のスタイルをチェックボックスの状態で切り替え可能
- `html:has()` でページ全体に影響するスタイル変更を実現

### フォーカス中のフィールドグループをハイライト

```css
.form-group:has(:focus-visible) {
  background-color: #f0f7ff;
  border-radius: 8px;
  outline: 2px solid #0066cc;
  outline-offset: 4px;
}
```

### 特定の子要素数に応じたレイアウト変更

```css
/* リスト項目が5つ以上あれば2カラムにする */
ul:has(li:nth-child(5)) {
  columns: 2;
  column-gap: 2rem;
}

/* リスト項目が10以上なら3カラム */
ul:has(li:nth-child(10)) {
  columns: 3;
}
```

### 画像キャプションの有無でスタイル変更

```css
figure:has(figcaption) {
  border: 1px solid #e0e0e0;
  padding: 1rem;
  border-radius: 8px;
}

figure:not(:has(figcaption)) {
  /* キャプションがなければシンプルに表示 */
}
```

> 参照: [web.dev - :has()](https://developer.chrome.com/blog/has-m105)

## 詳細度（Specificity）

- `:has()` 自体の詳細度は0
- **引数の中で最も詳細度が高いセレクタ**の詳細度が `:has()` の詳細度になる

```css
/* :has(img) → 要素セレクタ1つ分 (0, 0, 1) */
.card:has(img) { ... }
/* .card = (0, 1, 0) + :has(img) = (0, 0, 1) → 合計 (0, 1, 1) */

/* :has(.featured) → クラスセレクタ1つ分 (0, 1, 0) */
.card:has(.featured) { ... }
/* .card = (0, 1, 0) + :has(.featured) = (0, 1, 0) → 合計 (0, 2, 0) */

/* :has(#hero) → IDセレクタ1つ分 (1, 0, 0) */
.card:has(#hero) { ... }
/* .card = (0, 1, 0) + :has(#hero) = (1, 0, 0) → 合計 (1, 1, 0) */
```

- 詳細度を上げたくない場合は `:has(:where(...))` を使うと `:has()` の詳細度を0にできる

```css
/* :where()で詳細度を0にする */
.card:has(:where(.featured, #hero)) { ... }
/* :has()部分の詳細度は (0, 0, 0) */
```

> 参照: [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)

## パフォーマンスの注意点

```css
/* 悪い例：非常に広い範囲を探索するセレクタ */
*:has(img) {
  /* ページ内の全要素に対してimg子孫を探索 → 重い */
}

/* 良い例：対象を限定する */
.card:has(> img) {
  /* .card要素の直接の子だけ確認すればよい → 軽い */
}
```

- `:has()` はブラウザがセレクタの評価を最適化しているため、一般的な使用では問題にならない
- ただし `*:has(...)` のように全要素を対象にすると負荷が高くなる可能性がある
- **対象要素を限定**し、可能であれば `>` で直接の子に絞るとパフォーマンスが良い

> 参照: [web.dev - :has()](https://developer.chrome.com/blog/has-m105)

## ブラウザサポート

- Chrome 105+、Safari 15.4+、Firefox 121+ で全主要ブラウザがサポート
- 2024年以降、プロダクションで安心して使えるレベルのサポート状況

### フォールバック

```css
/* :has()をサポートしないブラウザ向けのフォールバック */
.card {
  /* デフォルトスタイル（:has()なしでも成立するように） */
  padding: 1.5rem;
}

/* :has()をサポートするブラウザのみ適用 */
@supports selector(:has(*)) {
  .card:has(img) {
    display: grid;
    grid-template-columns: 200px 1fr;
  }
}
```

- `@supports selector(:has(*))` で機能検出が可能
- 現在は主要ブラウザすべてがサポートしているため、フォールバックが不要なケースも多い

> 参照: [Can I Use - :has()](https://caniuse.com/css-has)

## まとめ

- `:has()` は「特定の子要素や兄弟要素を持つ要素」を選択できるリレーショナル擬似クラス
- 長年求められていた**親セレクタ**の機能を実現し、JavaScriptに頼らない条件付きスタイリングが可能
- `:not(:has(...))` で「持っていない場合」、`:has()` の連結でAND条件を表現できる
- フォームの状態（`:invalid`, `:checked`）との組み合わせが特に強力
- 全主要ブラウザでサポート済みのため、プロダクションで利用可能
- パフォーマンスを意識して対象要素を限定し、`>` で直接の子に絞るのがベストプラクティス

## 参照元

- [MDN - :has()](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)
- [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)
- [web.dev - :has()](https://developer.chrome.com/blog/has-m105)
- [CSS-Tricks - :has()](https://css-tricks.com/the-css-has-selector/)
- [Can I Use - :has()](https://caniuse.com/css-has)
