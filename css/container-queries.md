# コンテナクエリ(@container)の使い方

## コンテナクエリとは

- **親要素（コンテナ）のサイズ**に基づいてスタイルを切り替える仕組み
- メディアクエリがビューポート（画面全体）のサイズを参照するのに対し、コンテナクエリはコンポーネントの親要素のサイズを参照する
- コンポーネントの「配置場所」に応じたスタイル変更が可能になる
- 2023年2月に全主要ブラウザで対応（Baseline 2023）

> 参照: [MDN - CSS container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries)

## なぜ必要か

### メディアクエリの限界

```css
/* メディアクエリ：ビューポート幅に基づいてスタイルを変更 */
.card {
  display: flex;
  flex-direction: column;
}

@media (min-width: 768px) {
  .card {
    flex-direction: row;
  }
}
```

- 同じ `.card` コンポーネントがサイドバー（狭い領域）にもメインコンテンツ（広い領域）にも配置される場合、ビューポート幅だけでは適切なスタイルを決められない
- ビューポートが768px以上でも、サイドバー内のカードは狭いので縦並びにしたい

### コンテナクエリで解決

```css
/* コンテナクエリ：親要素の幅に基づいてスタイルを変更 */
.card-wrapper {
  container-type: inline-size;
}

.card {
  display: flex;
  flex-direction: column;
}

@container (min-width: 400px) {
  .card {
    flex-direction: row;
  }
}
```

- `.card` は配置場所の幅が400px以上なら横並び、未満なら縦並び
- ビューポートサイズに関係なく、コンポーネント自身が適切にレスポンシブになる

> 参照: [MDN - CSS container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries)

## 基本構文

### 1. コンテナの定義（container-type）

- コンテナクエリの対象となる要素に `container-type` を指定する
- コンテナ自身ではなく、その**子孫要素**のスタイルを変更する

| 値 | 説明 |
|---|---|
| `normal`（デフォルト） | コンテナクエリの対象にならない |
| `inline-size` | インライン方向（通常は幅）のサイズに基づくクエリが可能 |
| `size` | 幅と高さの両方に基づくクエリが可能 |

```css
/* 良い例：inline-size が最も一般的 */
.sidebar {
  container-type: inline-size;
}

/* 悪い例：container-type を指定せずに @container を使う */
/* → クエリが機能しない */
```

- `inline-size` が最もよく使われる
- `size` は高さのクエリも必要な場合のみ使用（パフォーマンスコストがやや高い）

### 2. コンテナの命名（container-name）

- 複数のコンテナが入れ子になっている場合に、どのコンテナを参照するか名前で指定できる

```css
.sidebar {
  container-type: inline-size;
  container-name: sidebar;
}

.main {
  container-type: inline-size;
  container-name: main-content;
}

/* sidebar コンテナのサイズに基づくクエリ */
@container sidebar (min-width: 300px) {
  .widget { font-size: 1.1rem; }
}

/* main-content コンテナのサイズに基づくクエリ */
@container main-content (min-width: 600px) {
  .article { columns: 2; }
}
```

### 3. container ショートハンド

```css
/* container: <name> / <type> */
.sidebar {
  container: sidebar / inline-size;
}

/* 名前なし */
.wrapper {
  container-type: inline-size;
}
```

### 4. @container ルール

```css
/* 名前なし：最も近い祖先コンテナを参照 */
@container (min-width: 400px) {
  .card { flex-direction: row; }
}

/* 名前指定：特定のコンテナを参照 */
@container sidebar (max-width: 300px) {
  .nav-item { font-size: 0.875rem; }
}

/* 複数条件 */
@container (min-width: 400px) and (max-width: 800px) {
  .card { grid-template-columns: 1fr 1fr; }
}
```

> 参照: [MDN - container-type](https://developer.mozilla.org/en-US/docs/Web/CSS/container-type)

## コンテナクエリの単位

- コンテナのサイズに相対的な単位が使える

| 単位 | 説明 |
|---|---|
| `cqw` | コンテナ幅の1% |
| `cqh` | コンテナ高さの1% |
| `cqi` | コンテナのインラインサイズの1% |
| `cqb` | コンテナのブロックサイズの1% |
| `cqmin` | `cqi` と `cqb` の小さい方 |
| `cqmax` | `cqi` と `cqb` の大きい方 |

```css
.sidebar {
  container-type: inline-size;
}

/* コンテナ幅に対する相対的なフォントサイズ */
.sidebar .title {
  font-size: clamp(1rem, 4cqi, 2rem);
}
```

> 参照: [MDN - Container query length units](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries#container_query_length_units)

## 実践パターン

### 1. レスポンシブカードコンポーネント

```css
.card-container {
  container-type: inline-size;
}

.card {
  display: grid;
  gap: 16px;
  padding: 16px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
}

/* 狭い場合：縦並び */
.card .image {
  aspect-ratio: 16 / 9;
}

/* 広い場合：横並び */
@container (min-width: 500px) {
  .card {
    grid-template-columns: 200px 1fr;
  }
  .card .image {
    aspect-ratio: 1;
  }
}

/* さらに広い場合 */
@container (min-width: 800px) {
  .card {
    grid-template-columns: 300px 1fr;
  }
}
```

### 2. サイドバーのナビゲーション

```css
.sidebar {
  container: sidebar / inline-size;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 12px;
}

.nav-item .label {
  display: block;
}

/* サイドバーが狭い場合：アイコンのみ表示 */
@container sidebar (max-width: 80px) {
  .nav-item {
    justify-content: center;
  }
  .nav-item .label {
    display: none;
  }
}
```

### 3. ダッシュボードのウィジェット

```css
.widget-area {
  container-type: inline-size;
}

.widget {
  padding: 16px;
}

.widget .stats {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

@container (min-width: 400px) {
  .widget .stats {
    flex-direction: row;
    justify-content: space-between;
  }
}

@container (min-width: 600px) {
  .widget .stats {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
  }
}
```

### 4. メディアクエリとの併用

```css
/* ページ全体の構造はメディアクエリで制御 */
.page {
  display: grid;
  grid-template-columns: 1fr;
}

@media (min-width: 1024px) {
  .page {
    grid-template-columns: 280px 1fr;
  }
}

/* コンポーネントの内部はコンテナクエリで制御 */
.main {
  container-type: inline-size;
}

@container (min-width: 600px) {
  .card-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@container (min-width: 900px) {
  .card-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

> 参照: [web.dev - Container queries](https://web.dev/blog/cq-stable)

## 注意点と制約

### 1. コンテナ自身のスタイルは変更できない

```css
.wrapper {
  container-type: inline-size;
}

/* 悪い例：コンテナ自身を @container で変更しようとする → 効かない */
@container (min-width: 500px) {
  .wrapper {
    background: red; /* コンテナ自身には適用されない */
  }
}

/* 良い例：コンテナの子孫要素のスタイルを変更する */
@container (min-width: 500px) {
  .wrapper .child {
    background: red;
  }
}
```

### 2. container-type によるレイアウトへの影響

- `container-type: inline-size` を指定すると、その要素にはインライン方向の**サイズ封じ込め（containment）**が適用される
- ブロック要素なら通常通り親幅いっぱいに広がるため、ほとんどの場合問題ない
- インライン要素やフロート要素では意図しない挙動が起きる可能性がある

```css
/* 注意：container-type を指定した要素は BFC を形成する */
.wrapper {
  container-type: inline-size;
  /* → 新しいブロック整形コンテキスト（BFC）が作られる */
  /* → 子要素のマージン相殺が発生しない */
}
```

### 3. 名前なし @container は最も近い祖先を参照する

```css
.outer {
  container-type: inline-size;
}

.inner {
  container-type: inline-size;
}

/* .inner のコンテナ（最も近い祖先）を参照する */
@container (min-width: 400px) {
  .content { /* ... */ }
}

/* .outer を参照したい場合は名前を使う */
.outer { container: outer / inline-size; }

@container outer (min-width: 800px) {
  .content { /* ... */ }
}
```

## スタイルクエリ（Style Queries）

- コンテナのサイズだけでなく、**カスタムプロパティの値**に基づくクエリも可能
- `@container style()` で記述する
- 2024年時点で一部ブラウザのみ対応（Chrome 111+）

```css
.card {
  --variant: default;
}

.card.featured {
  --variant: featured;
}

/* カスタムプロパティの値に基づくスタイル変更 */
@container style(--variant: featured) {
  .card-title {
    font-size: 1.5rem;
    color: gold;
  }
}
```

- スタイルクエリはまだ実験的機能のため、プロダクション利用は慎重に判断する

> 参照: [MDN - Container style queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries#container_style_queries)

## ブラウザ対応

- サイズクエリ（`container-type: inline-size`）: Baseline 2023（Chrome 105+, Firefox 110+, Safari 16+）
- スタイルクエリ: Chrome 111+ のみ（2024年時点）

> 参照: [Can I Use - CSS Container Queries](https://caniuse.com/css-container-queries)

## まとめ

- コンテナクエリは**親要素のサイズ**に基づいてスタイルを切り替える仕組み
- メディアクエリの「ビューポート依存」という制限を解消し、真にレスポンシブなコンポーネントが作れる
- `container-type: inline-size` でコンテナを定義し、`@container` でクエリを記述する
- コンテナ自身のスタイルは変更できない（子孫要素のみ対象）
- メディアクエリとの併用が一般的（ページ構造はメディアクエリ、コンポーネントはコンテナクエリ）
- コンテナクエリ単位（`cqi`, `cqw` など）でコンテナサイズに相対的な値も指定できる

## 参照元

- [MDN - CSS container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries)
- [MDN - container-type](https://developer.mozilla.org/en-US/docs/Web/CSS/container-type)
- [web.dev - Container queries](https://web.dev/blog/cq-stable)
- [Can I Use - CSS Container Queries](https://caniuse.com/css-container-queries)
