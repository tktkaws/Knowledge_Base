# CSS変数(カスタムプロパティ)の基本と活用パターン

## カスタムプロパティとは

- CSSネイティブの変数機能（CSS Custom Properties for Cascading Variables）
- `--` プレフィックスで宣言し、`var()` 関数で参照する
- CSSのカスケードと継承のルールに従う
- JavaScriptから動的に読み書きできる
- プリプロセッサ変数(Sass/Less)とは根本的に異なる仕組み

> 参照: [MDN - Using CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)

## 基本構文

### 宣言と参照

- `--` プレフィックス + 任意の名前で宣言
- `var()` 関数で値を参照
- 大文字・小文字を区別する（`--color` と `--Color` は別のプロパティ）

```css
/* 良い例：:rootで宣言し、var()で参照 */
:root {
  --primary-color: #3b82f6;
  --font-size-base: 16px;
  --spacing-unit: 8px;
}

.button {
  background-color: var(--primary-color);
  font-size: var(--font-size-base);
  padding: calc(var(--spacing-unit) * 2);
}
```

```css
/* 悪い例：var()が使えない場所で使おうとする */

/* NG: プロパティ名にvar()は使えない */
.box {
  var(--property): 10px;
}

/* NG: セレクタ内でvar()は使えない */
var(--my-selector) {
  color: red;
}

/* NG: メディアクエリの条件にvar()は使えない */
@media (max-width: var(--breakpoint)) {
  /* ... */
}
```

### 命名規則の推奨パターン

- ケバブケースを使用（CSS既存プロパティとの一貫性）
- 用途や意味を反映した名前を付ける
- カテゴリをプレフィックスにすると管理しやすい

```css
/* 良い例：カテゴリ別に整理された命名 */
:root {
  /* 色 */
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --color-error: #ef4444;

  /* タイポグラフィ */
  --font-size-sm: 0.875rem;
  --font-size-md: 1rem;
  --font-size-lg: 1.25rem;

  /* スペーシング */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 32px;
}
```

```css
/* 悪い例：意味が不明瞭な命名 */
:root {
  --c1: #3b82f6;
  --a: 16px;
  --x: 8px;
}
```

> 参照: [MDN - Using CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)

## スコープと継承

### カスケードの仕組み

- カスタムプロパティは通常のCSSプロパティと同様にカスケードに従う
- 宣言されたセレクタのスコープ内で有効
- 子孫要素に継承される
- より詳細度の高いセレクタや、DOMツリーの下位で再宣言すると上書きされる

```css
/* 良い例：スコープを活かした宣言 */

/* グローバルスコープ */
:root {
  --text-color: #1a1a1a;
  --bg-color: #ffffff;
}

/* コンポーネントスコープで上書き */
.dark-section {
  --text-color: #f5f5f5;
  --bg-color: #1a1a1a;
}

/* どちらのセクションでも同じプロパティ名で参照できる */
.card {
  color: var(--text-color);
  background-color: var(--bg-color);
}
```

```html
<!-- DOMの構造によって適用される値が変わる -->
<body>
  <!-- --text-color は #1a1a1a（:rootの値） -->
  <div class="card">通常のカード</div>

  <section class="dark-section">
    <!-- --text-color は #f5f5f5（.dark-sectionで上書き） -->
    <div class="card">暗い背景のカード</div>
  </section>
</body>
```

### :root vs html

- `:root` は `html` 要素を指すが、詳細度が異なる
- `:root` は擬似クラスセレクタ → 詳細度 `0-1-0`
- `html` はタイプセレクタ → 詳細度 `0-0-1`
- グローバル変数の宣言には `:root` を使うのが慣例

> 参照: [MDN - :root](https://developer.mozilla.org/en-US/docs/Web/CSS/:root)

## フォールバック値

### var() の第2引数

- `var()` の第2引数にフォールバック値を指定できる
- カスタムプロパティが未定義の場合にフォールバック値が使われる
- フォールバックは入れ子にできる
- カンマ以降の全てがフォールバック値として解釈される

```css
/* 良い例：フォールバック値の指定 */
.button {
  /* --accent-color が未定義なら blue を使う */
  background-color: var(--accent-color, blue);

  /* フォールバックの入れ子 */
  color: var(--button-text, var(--text-color, #ffffff));

  /* カンマ以降全てがフォールバック値
     → フォールバックは "1px solid black" */
  border: var(--button-border, 1px solid black);
}
```

```css
/* 悪い例：フォールバックなしで未定義の変数を使う */
.button {
  /* --accent-color が未定義の場合、backgroundプロパティが無効になる */
  background-color: var(--accent-color);
}
```

### 無効な値の扱い（IACVT: Invalid At Computed Value Time）

- カスタムプロパティの値がプロパティに対して不正な場合の挙動
- `var()` のフォールバック値ではなく、プロパティの継承値または初期値が使われる
- よくある混乱の原因

```css
:root {
  --text-color: 16px; /* 色として無効な値 */
}

p {
  color: blue;
}

p.special {
  /* --text-color は "16px" で色としては無効
     → var()のフォールバック red は使われない
     → colorプロパティの初期値または継承値が適用される */
  color: var(--text-color, red);
}
```

- `var()` のフォールバックは「変数が未定義」の場合のみ有効
- 値が存在するが型として不正な場合はフォールバックが発動しない

> 参照: [MDN - Invalid custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties#invalid_custom_properties)

## 活用パターン

### 1. テーマの切り替え（ダーク/ライトモード）

```css
/* 良い例：カスタムプロパティでテーマを実装 */
:root {
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
  --color-primary: #3b82f6;
  --color-surface: #f8fafc;
  --color-border: #e2e8f0;
}

/* メディアクエリでダークモードに切り替え */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0f172a;
    --color-text: #e2e8f0;
    --color-primary: #60a5fa;
    --color-surface: #1e293b;
    --color-border: #334155;
  }
}

/* data属性によるテーマ切り替え（JSで制御） */
[data-theme="dark"] {
  --color-bg: #0f172a;
  --color-text: #e2e8f0;
  --color-primary: #60a5fa;
  --color-surface: #1e293b;
  --color-border: #334155;
}

/* コンポーネントは変数を参照するだけ */
body {
  background-color: var(--color-bg);
  color: var(--color-text);
}

.card {
  background-color: var(--color-surface);
  border: 1px solid var(--color-border);
}
```

### 2. レスポンシブデザイン

```css
/* 良い例：ブレイクポイントで変数の値を切り替える */
:root {
  --container-width: 100%;
  --grid-columns: 1;
  --font-size-heading: 1.5rem;
  --spacing-section: 24px;
}

@media (min-width: 768px) {
  :root {
    --grid-columns: 2;
    --font-size-heading: 2rem;
    --spacing-section: 48px;
  }
}

@media (min-width: 1024px) {
  :root {
    --container-width: 960px;
    --grid-columns: 3;
    --font-size-heading: 2.5rem;
    --spacing-section: 64px;
  }
}

.grid {
  display: grid;
  grid-template-columns: repeat(var(--grid-columns), 1fr);
  gap: var(--spacing-section);
  max-width: var(--container-width);
}
```

### 3. コンポーネントバリアント

```css
/* 良い例：内部変数でバリアントを制御 */
.button {
  --_bg: var(--button-bg, #3b82f6);
  --_color: var(--button-color, #ffffff);
  --_size: var(--button-size, 1rem);

  background-color: var(--_bg);
  color: var(--_color);
  font-size: var(--_size);
  padding: calc(var(--_size) * 0.5) calc(var(--_size) * 1);
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* バリアントは変数を上書きするだけ */
.button--danger {
  --button-bg: #ef4444;
}

.button--outline {
  --button-bg: transparent;
  --button-color: #3b82f6;
}

.button--large {
  --button-size: 1.25rem;
}
```

```css
/* 悪い例：バリアントごとに全プロパティを再宣言 */
.button--danger {
  background-color: #ef4444;
  color: #ffffff;       /* 重複 */
  font-size: 1rem;       /* 重複 */
  padding: 0.5rem 1rem;  /* 重複 */
  border: none;           /* 重複 */
  border-radius: 4px;     /* 重複 */
}
```

### 4. calc() との組み合わせ

```css
/* 良い例：単位なしの値とcalc()で柔軟な計算 */
:root {
  --spacing-unit: 8;
}

.card {
  padding: calc(var(--spacing-unit) * 2 * 1px);    /* 16px */
  margin-bottom: calc(var(--spacing-unit) * 3 * 1px); /* 24px */
}
```

> 参照: [MDN - var()](https://developer.mozilla.org/en-US/docs/Web/CSS/var)

## プリプロセッサ変数(Sass)との違い

| 特徴 | カスタムプロパティ(CSS) | Sass変数 |
|---|---|---|
| 処理タイミング | ブラウザ実行時（ランタイム） | ビルド時（コンパイル時） |
| スコープ | DOMツリーに基づくカスケード | ファイル/ブロックスコープ |
| 継承 | DOMツリーに沿って子孫に継承 | なし |
| 動的変更 | JavaScriptで動的に変更可能 | 不可（コンパイル後は固定値） |
| メディアクエリ内での切替 | 可能 | 不可 |
| 型チェック | `@property` で可能 | なし |
| DevToolsでの確認 | 変数として確認・編集可能 | コンパイル後の値のみ表示 |

```scss
/* Sassの場合：ビルド時に固定値に置換される */
$primary: #3b82f6;

.button {
  background: $primary; /* コンパイル後: background: #3b82f6; */
}
/* メディアクエリで$primaryを切り替えることはできない */
```

```css
/* CSSカスタムプロパティ：ランタイムで値が解決される */
:root {
  --primary: #3b82f6;
}

.button {
  background: var(--primary);
}

/* メディアクエリで動的に切り替え可能 */
@media (prefers-color-scheme: dark) {
  :root {
    --primary: #60a5fa;
  }
}
```

- 両者は競合しない
- 静的な定数はSass変数、動的に変化する値はCSSカスタムプロパティと使い分けるのが実用的

> 参照: [CSS-Tricks - Difference between types of CSS variables](https://css-tricks.com/difference-between-types-of-css-variables/)

## JavaScriptからの動的更新

### 読み取り・書き込み

```javascript
// 良い例：getComputedStyleでカスタムプロパティの値を取得
const root = document.documentElement;
const primaryColor = getComputedStyle(root)
  .getPropertyValue('--primary-color')
  .trim(); // 先頭にスペースが入る場合があるのでtrim()

// 良い例：setPropertyでカスタムプロパティの値を変更
root.style.setProperty('--primary-color', '#ef4444');
```

```javascript
// 悪い例：style属性から直接読もうとする
const color = element.style.getPropertyValue('--primary-color');
// → CSSファイルで宣言された値は取得できない（インラインスタイルのみ）
```

### テーマ切り替えの実装例

```javascript
const toggleTheme = () => {
  const currentTheme = document.documentElement.dataset.theme;
  document.documentElement.dataset.theme =
    currentTheme === 'dark' ? 'light' : 'dark';
};

// ユーザーの選択をlocalStorageに保存
const savedTheme = localStorage.getItem('theme');
if (savedTheme) {
  document.documentElement.dataset.theme = savedTheme;
}
```

### マウス位置の追跡

```javascript
document.addEventListener('mousemove', (e) => {
  document.documentElement.style.setProperty('--mouse-x', `${e.clientX}px`);
  document.documentElement.style.setProperty('--mouse-y', `${e.clientY}px`);
});
```

```css
.spotlight {
  background: radial-gradient(
    circle at var(--mouse-x) var(--mouse-y),
    rgba(255, 255, 255, 0.2),
    transparent 80%
  );
}
```

> 参照: [MDN - CSSStyleDeclaration.setProperty()](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/setProperty)

## @property ルールによる型付きカスタムプロパティ

### 概要

- CSS Houdini仕様の一部
- カスタムプロパティに型・初期値・継承の有無を定義できる
- 2024年7月にBaseline（全主要ブラウザ対応）に到達
- 通常のカスタムプロパティではできないアニメーションが可能になる

### 構文

```css
@property --property-name {
  syntax: "<type>";       /* 必須：許容する値の型 */
  inherits: true | false; /* 必須：継承するかどうか */
  initial-value: value;   /* syntax が "*" 以外の場合は必須 */
}
```

### syntax に指定できる主な型

| 型 | 説明 | 例 |
|---|---|---|
| `"<color>"` | 色 | `red`, `#ff0000`, `rgb(255, 0, 0)` |
| `"<length>"` | 長さ | `10px`, `2rem`, `50vh` |
| `"<percentage>"` | パーセント | `50%` |
| `"<number>"` | 数値 | `1.5`, `0`, `100` |
| `"<angle>"` | 角度 | `45deg`, `0.5turn` |
| `"<integer>"` | 整数 | `1`, `3`, `100` |
| `"*"` | 任意の値 | 何でも可 |

### アニメーションへの活用

- 通常のカスタムプロパティはアニメーション不可（中間値を補間できない）
- `@property` で型を指定するとブラウザが中間値を計算できるようになる

```css
/* 良い例：@propertyでグラデーションをアニメーション */
@property --gradient-angle {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}

.animated-gradient {
  --gradient-angle: 0deg;
  background: linear-gradient(
    var(--gradient-angle),
    #3b82f6,
    #a855f7
  );
  transition: --gradient-angle 0.5s ease;
}

.animated-gradient:hover {
  --gradient-angle: 180deg;
}
```

```css
/* 悪い例：@propertyなしではグラデーション角度をアニメーションできない */
.animated-gradient {
  --gradient-angle: 0deg;
  background: linear-gradient(
    var(--gradient-angle),
    #3b82f6,
    #a855f7
  );
  /* 中間値を補間できないため、即座に切り替わってしまう */
  transition: --gradient-angle 0.5s ease;
}
```

### @keyframes との組み合わせ

```css
@property --progress {
  syntax: "<percentage>";
  inherits: false;
  initial-value: 0%;
}

.progress-bar {
  background: linear-gradient(
    to right,
    #22c55e var(--progress),
    #e5e7eb var(--progress)
  );
  animation: fill 2s ease forwards;
}

@keyframes fill {
  to {
    --progress: 100%;
  }
}
```

> 参照: [MDN - @property](https://developer.mozilla.org/en-US/docs/Web/CSS/@property)

## よくある落とし穴

### 1. フォールバック値と無効値の混同

```css
:root {
  --size: red; /* lengthとしては無効 */
}

.box {
  /* redはwidthとして無効だが、変数自体は"定義済み"
     → フォールバック 200px は使われない
     → widthの初期値 auto が適用される */
  width: var(--size, 200px);
}
```

### 2. 単位の結合

```css
/* 悪い例：文字列結合はできない */
:root {
  --size: 16;
}

.box {
  /* NG: "16px" にはならず無効になる */
  font-size: var(--size)px;
}
```

```css
/* 良い例：calc()で単位を付ける */
:root {
  --size: 16;
}

.box {
  font-size: calc(var(--size) * 1px);
}
```

### 3. 循環参照

```css
/* 悪い例：循環参照は無効になる */
:root {
  --a: var(--b);
  --b: var(--a);
}

.box {
  color: var(--a); /* 無効 → プロパティの初期値が使われる */
}
```

### 4. メディアクエリの条件には使えない

```css
/* 悪い例 */
:root {
  --breakpoint: 768px;
}

@media (min-width: var(--breakpoint)) {
  /* 動作しない */
}
```

```css
/* 代替案：メディアクエリ内で変数の値を変更する */
:root {
  --columns: 1;
}

@media (min-width: 768px) {
  :root {
    --columns: 2;
  }
}

.grid {
  grid-template-columns: repeat(var(--columns), 1fr);
}
```

## ベストプラクティス

### デザイントークンとして整理する

```css
/* 良い例：プリミティブ → セマンティック の2層構造 */

/* 1. プリミティブトークン（生の値） */
:root {
  --blue-50: #eff6ff;
  --blue-500: #3b82f6;
  --blue-900: #1e3a5a;
  --gray-100: #f3f4f6;
  --gray-900: #111827;
}

/* 2. セマンティックトークン（用途を表す名前） */
:root {
  --color-bg-primary: var(--gray-100);
  --color-text-primary: var(--gray-900);
  --color-accent: var(--blue-500);
}

/* コンポーネントはセマンティックトークンを参照 */
.card {
  background: var(--color-bg-primary);
  color: var(--color-text-primary);
}
```

```css
/* 悪い例：プリミティブ値をコンポーネントで直接使う */
.card {
  background: var(--gray-100);
  color: var(--gray-900);
}
/* テーマ切り替え時に全コンポーネントを修正する必要がある */
```

### プライベート変数の慣例

```css
/* 良い例：_ プレフィックスで内部変数を示す */
.button {
  /* 内部でのみ使用（プライベート） */
  --_bg: var(--button-bg, #3b82f6);
  --_color: var(--button-color, #ffffff);

  background-color: var(--_bg);
  color: var(--_color);
}
```

### 必ずフォールバック値を設定する

```css
/* 良い例 */
.component {
  color: var(--component-text, var(--color-text-primary, #1a1a1a));
  gap: var(--component-gap, 16px);
}
```

## ブラウザ対応

- カスタムプロパティ (`--*` / `var()`)：全モダンブラウザ対応（IE11は非対応）
- `@property` ルール：2024年7月にBaseline到達（Chrome 85+, Firefox 128+, Safari 16.4+）

> 参照: [Can I Use - CSS Variables](https://caniuse.com/css-variables), [Can I Use - @property](https://caniuse.com/mdn-css_at-rules_property)

## まとめ

- カスタムプロパティは `--` プレフィックスで宣言し `var()` で参照する
- DOMツリーに沿って継承され、セレクタのスコープ内で有効
- フォールバック値は「変数が未定義」の場合のみ適用される（型が不正な場合は発動しない）
- テーマ切り替え・レスポンシブ対応・コンポーネントバリアントの実装に有用
- Sassとは処理タイミングが異なり、ランタイムで値が解決される
- JavaScriptから `setProperty()` / `getPropertyValue()` で動的に操作可能
- `@property` ルールで型・初期値・継承を制御でき、アニメーションも可能になる
- デザイントークンを「プリミティブ → セマンティック」の2層で整理するのがベストプラクティス

## 参照元

- [MDN - Using CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)
- [MDN - var()](https://developer.mozilla.org/en-US/docs/Web/CSS/var)
- [MDN - @property](https://developer.mozilla.org/en-US/docs/Web/CSS/@property)
- [MDN - CSSStyleDeclaration.setProperty()](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/setProperty)
- [Can I Use - CSS Variables](https://caniuse.com/css-variables)
- [CSS-Tricks - Difference between types of CSS variables](https://css-tricks.com/difference-between-types-of-css-variables/)
