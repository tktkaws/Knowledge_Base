# メディアクエリの基本とブレイクポイント設計

## メディアクエリとは

- **ビューポートやデバイスの特性**に基づいてCSSを条件付きで適用する仕組み
- レスポンシブデザインの基盤となる技術
- 画面幅だけでなく、解像度・配色・操作方法などさまざまな条件でスタイルを切り替えられる

> 参照: [MDN - Using media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries)

## 基本構文

### @media ルール

```css
/* 基本形 */
@media (条件) {
  /* 条件に一致した場合に適用されるスタイル */
}

/* メディアタイプ + 条件 */
@media screen and (min-width: 768px) {
  .container { max-width: 720px; }
}
```

### メディアタイプ

| タイプ | 対象 |
|---|---|
| `all`（デフォルト） | すべてのデバイス |
| `screen` | 画面 |
| `print` | 印刷 |

```css
/* 印刷時のスタイル */
@media print {
  .no-print { display: none; }
  body { font-size: 12pt; }
}
```

- `screen` は省略されることが多い（`all` がデフォルトのため）
- 実務では `screen` と `print` 以外を使うことはほぼない

### メディア特性（Media Features）

#### サイズ関連

| 特性 | 説明 |
|---|---|
| `width` / `min-width` / `max-width` | ビューポートの幅 |
| `height` / `min-height` / `max-height` | ビューポートの高さ |
| `aspect-ratio` | ビューポートのアスペクト比 |
| `orientation` | `portrait`（縦長） or `landscape`（横長） |

```css
@media (min-width: 768px) { /* 幅768px以上 */ }
@media (max-width: 767px) { /* 幅767px以下 */ }
@media (orientation: landscape) { /* 横長画面 */ }
```

#### ユーザー設定関連

| 特性 | 説明 |
|---|---|
| `prefers-color-scheme` | ダーク/ライトモードの設定 |
| `prefers-reduced-motion` | アニメーション軽減の設定 |
| `prefers-contrast` | コントラストの設定 |
| `prefers-reduced-transparency` | 透明度軽減の設定 |

```css
@media (prefers-color-scheme: dark) {
  :root { --bg: #0f172a; --text: #e2e8f0; }
}

@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}
```

#### デバイス特性関連

| 特性 | 説明 |
|---|---|
| `hover` | ホバーが可能か（`hover` or `none`） |
| `pointer` | ポインタの精度（`fine`, `coarse`, `none`） |
| `resolution` | 画面の解像度（dpi, dppx） |

```css
/* タッチデバイス：ホバー不可・ポインタが粗い */
@media (hover: none) and (pointer: coarse) {
  .tooltip { display: none; }
  button { min-height: 44px; min-width: 44px; }
}

/* 高解像度ディスプレイ */
@media (resolution >= 2dppx) {
  .logo { background-image: url('logo@2x.png'); }
}
```

> 参照: [MDN - Media features](https://developer.mozilla.org/en-US/docs/Web/CSS/@media#media_features)

## 論理演算子

### and — すべての条件を満たす

```css
@media (min-width: 768px) and (max-width: 1023px) {
  /* 768px〜1023px の範囲 */
}
```

### or（カンマ区切り） — いずれかの条件を満たす

```css
@media (max-width: 767px), (orientation: portrait) {
  /* 幅767px以下、または縦長画面 */
}
```

### not — 条件を否定

```css
@media not print {
  /* 印刷以外のすべてのメディア */
}
```

### Range構文（Level 4）

- 比較演算子を使った直感的な記法
- 全主要ブラウザ対応（2023年〜）

```css
/* 良い例：Range構文（モダン） */
@media (width >= 768px) { /* 768px以上 */ }
@media (768px <= width <= 1023px) { /* 768px〜1023px */ }

/* 従来の記法 */
@media (min-width: 768px) { /* 768px以上 */ }
@media (min-width: 768px) and (max-width: 1023px) { /* 768px〜1023px */ }
```

> 参照: [MDN - Using media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries)

## ブレイクポイント設計

### ブレイクポイントとは

- メディアクエリでレイアウトを切り替える画面幅の境界値
- デバイスの種類ではなく、**コンテンツが崩れるポイント**で設定するのが理想

### よく使われるブレイクポイント

```css
/* 一般的なブレイクポイント例 */
/* sm:  640px  — スマートフォン横向き */
/* md:  768px  — タブレット縦向き */
/* lg:  1024px — タブレット横向き / 小さいノートPC */
/* xl:  1280px — デスクトップ */
/* 2xl: 1536px — 大画面デスクトップ */
```

### 主要フレームワークのブレイクポイント比較

| | Bootstrap 5 | Tailwind CSS | Material UI |
|---|---|---|---|
| xs / sm | 576px | 640px | 600px |
| md | 768px | 768px | 900px |
| lg | 992px | 1024px | 1200px |
| xl | 1200px | 1280px | 1536px |
| xxl | 1400px | 1536px | — |

### モバイルファースト vs デスクトップファースト

#### モバイルファースト（推奨）

- **基本スタイルをモバイル向けに書き**、`min-width` で拡張していく
- パフォーマンスに有利（モバイルでは不要なスタイルを読み込まない）
- 現在の主流アプローチ

```css
/* 良い例：モバイルファースト */
.container {
  padding: 16px;
}

@media (min-width: 768px) {
  .container {
    padding: 24px;
    max-width: 720px;
    margin: 0 auto;
  }
}

@media (min-width: 1024px) {
  .container {
    max-width: 960px;
  }
}
```

#### デスクトップファースト

- 基本スタイルをデスクトップ向けに書き、`max-width` で縮小していく
- レガシーサイトの改修時に使われることがある

```css
/* デスクトップファースト（非推奨だが理解しておく） */
.container {
  max-width: 1200px;
  padding: 32px;
}

@media (max-width: 1023px) {
  .container {
    max-width: 720px;
    padding: 24px;
  }
}

@media (max-width: 767px) {
  .container {
    padding: 16px;
  }
}
```

```css
/* 悪い例：min-width と max-width を混在させる */
@media (min-width: 768px) {
  .sidebar { width: 250px; }
}
@media (max-width: 1023px) {
  .sidebar { width: 200px; }
}
/* どちらが適用されるか混乱しやすい */
```

> 参照: [web.dev - Responsive design](https://web.dev/learn/design/media-queries)

## ブレイクポイント設計のベストプラクティス

### 1. デバイスではなくコンテンツに合わせる

```css
/* 悪い例：特定のデバイスに合わせた値 */
@media (min-width: 375px) { } /* iPhone SE */
@media (min-width: 390px) { } /* iPhone 14 */
@media (min-width: 414px) { } /* iPhone 14 Plus */

/* 良い例：コンテンツが崩れるポイントで設定 */
@media (min-width: 600px) { } /* コンテンツが1列→2列に切り替わる幅 */
@media (min-width: 900px) { } /* サイドバーを表示できる幅 */
```

### 2. ブレイクポイントの数を最小限にする

```css
/* 悪い例：ブレイクポイントが多すぎる */
@media (min-width: 320px) { }
@media (min-width: 480px) { }
@media (min-width: 640px) { }
@media (min-width: 768px) { }
@media (min-width: 1024px) { }
@media (min-width: 1280px) { }
@media (min-width: 1440px) { }

/* 良い例：必要な分だけ（通常2〜4個） */
@media (min-width: 768px) { }
@media (min-width: 1024px) { }
```

### 3. CSS変数でブレイクポイントごとに値を切り替える

```css
:root {
  --container-padding: 16px;
  --grid-columns: 1;
  --font-size-heading: 1.5rem;
}

@media (min-width: 768px) {
  :root {
    --container-padding: 24px;
    --grid-columns: 2;
    --font-size-heading: 2rem;
  }
}

@media (min-width: 1024px) {
  :root {
    --container-padding: 32px;
    --grid-columns: 3;
    --font-size-heading: 2.5rem;
  }
}

/* コンポーネントは変数を参照するだけ */
.container { padding: var(--container-padding); }
.grid { grid-template-columns: repeat(var(--grid-columns), 1fr); }
h1 { font-size: var(--font-size-heading); }
```

### 4. メディアクエリが不要な場面を知る

```css
/* メディアクエリ不要でレスポンシブにできる例 */

/* Grid の auto-fit + minmax */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
}

/* clamp() によるフルイドタイポグラフィ */
h1 {
  font-size: clamp(1.5rem, 4vw, 3rem);
}

/* flexbox の折り返し */
.nav {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

/* min() / max() による制約 */
.container {
  width: min(100% - 32px, 1200px);
  margin-inline: auto;
}
```

> 参照: [web.dev - Learn Responsive Design](https://web.dev/learn/design)

## 実践パターン

### 1. レスポンシブナビゲーション

```css
/* モバイル：ハンバーガーメニュー */
.nav-links {
  display: none;
}

.nav-toggle {
  display: block;
}

/* デスクトップ：横並びメニュー */
@media (min-width: 768px) {
  .nav-links {
    display: flex;
    gap: 24px;
  }
  .nav-toggle {
    display: none;
  }
}
```

### 2. レスポンシブグリッドレイアウト

```css
.page {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
  padding: 16px;
}

@media (min-width: 768px) {
  .page {
    grid-template-columns: 250px 1fr;
    padding: 24px;
  }
}

@media (min-width: 1024px) {
  .page {
    grid-template-columns: 250px 1fr 200px;
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

### 3. 印刷スタイル

```css
@media print {
  /* ナビゲーション・フッターを非表示 */
  nav, footer, .sidebar, .no-print {
    display: none;
  }

  /* リンクURLを表示 */
  a[href]::after {
    content: " (" attr(href) ")";
    font-size: 0.8em;
  }

  /* 背景色を無効化 */
  body {
    color: black;
    background: white;
  }

  /* ページ分割を制御 */
  h2, h3 {
    page-break-after: avoid;
  }
}
```

### 4. ダークモード対応

```css
:root {
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
  --color-surface: #f8fafc;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0f172a;
    --color-text: #e2e8f0;
    --color-surface: #1e293b;
  }
}

body {
  background: var(--color-bg);
  color: var(--color-text);
}
```

## よくある間違い

### 1. ブレイクポイントの1px問題

```css
/* 悪い例：隙間が生じる */
@media (max-width: 768px) { /* 〜768px */ }
@media (min-width: 769px) { /* 769px〜 */}
/* 768.5px のような小数値で両方にマッチしない可能性 */

/* 良い例：min-width だけで統一（モバイルファースト） */
/* 基本スタイル = モバイル */
@media (min-width: 769px) { /* 769px〜 */ }

/* または Range 構文 */
@media (width < 769px) { /* 〜768px */ }
@media (width >= 769px) { /* 769px〜 */ }
```

### 2. メディアクエリの順序ミス

```css
/* 悪い例：大きい方が先に来ている */
@media (min-width: 1024px) {
  .container { max-width: 960px; }
}
@media (min-width: 768px) {
  .container { max-width: 720px; }
  /* 1024px以上でもこちらが後に書かれているので上書きしてしまう */
}

/* 良い例：小さい方から順に書く（モバイルファースト） */
@media (min-width: 768px) {
  .container { max-width: 720px; }
}
@media (min-width: 1024px) {
  .container { max-width: 960px; }
}
```

### 3. コンテナクエリとの使い分けを忘れる

```css
/* 悪い例：コンポーネントの内部レイアウトにメディアクエリを使う */
@media (min-width: 768px) {
  .card { flex-direction: row; }
}
/* サイドバーに配置された.cardはビューポートが広くても狭い */

/* 良い例：コンテナクエリを使う */
.card-wrapper { container-type: inline-size; }

@container (min-width: 400px) {
  .card { flex-direction: row; }
}
```

## メディアクエリ vs コンテナクエリの使い分け

| | メディアクエリ | コンテナクエリ |
|---|---|---|
| 参照するサイズ | ビューポート | 親要素 |
| 用途 | ページ全体のレイアウト | コンポーネント内部のレイアウト |
| 適している場面 | グリッド列数の変更、ナビの表示切替 | カードの縦横切替、ウィジェットの適応 |

## まとめ

- メディアクエリはビューポートやデバイス特性に基づいてスタイルを切り替える仕組み
- **モバイルファースト**（`min-width`）で書くのが推奨
- ブレイクポイントはデバイスではなく**コンテンツが崩れるポイント**で設定する
- ブレイクポイントの数は最小限に（通常2〜4個）
- CSS変数と組み合わせるとブレイクポイントごとの値管理が簡潔になる
- `clamp()` や `auto-fit` などメディアクエリ不要のレスポンシブ手法も活用する
- コンポーネント内部のレイアウトにはコンテナクエリの方が適している場合がある

## 参照元

- [MDN - Using media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries)
- [MDN - @media](https://developer.mozilla.org/en-US/docs/Web/CSS/@media)
- [web.dev - Learn Responsive Design](https://web.dev/learn/design)
- [Can I Use - Media Queries Level 4](https://caniuse.com/css-media-range-syntax)
