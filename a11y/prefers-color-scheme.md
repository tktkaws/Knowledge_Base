# prefers-color-scheme — ダークモード対応の基礎

## prefers-color-scheme とは

- ユーザーがOSレベルで設定しているカラーテーマ（ライト/ダーク）を検知するためのCSSメディアクエリ
- CSS Media Queries Level 5 で定義された機能
- 値は `light`（ライトモード）と `dark`（ダークモード）の2種類
- ユーザーの好みに応じてWebサイトの配色を自動的に切り替えることが可能
- ブラウザサポートは主要ブラウザで広く対応済み（Chrome 76+, Firefox 67+, Safari 12.1+, Edge 79+）

### 参照元
- [MDN — prefers-color-scheme](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-color-scheme)
- [W3C — CSS Media Queries Level 5](https://drafts.csswg.org/mediaqueries-5/#prefers-color-scheme)

---

## ダークモードのアクセシビリティ上の意義

- **光過敏症（Photophobia）への配慮**: 明るい画面が頭痛・片頭痛・眼痛を引き起こすユーザーへの対応
- **眼精疲労の軽減**: 長時間のPC作業やスマートフォン使用時の目の負担を軽減
- **暗所での使用**: 暗い環境での画面のまぶしさを抑え、目の快適さを向上
- **認知負荷の低減**: 明度差を適切に調整することで、コンテンツの読みやすさを向上
- **ユーザーの自律性**: OS設定を尊重することで、ユーザーが自分に合った表示を選べる環境を提供
- **WCAG 2.2 ガイドライン 1.4.1（色の使用）**: 色だけに依存しない情報伝達と合わせて考慮すべき要素

### 参照元
- [W3C — Understanding Success Criterion 1.4.1: Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html)
- [WebAIM — Visual Disabilities](https://webaim.org/articles/visual/)

---

## 各OSでの設定方法

| OS | 設定パス |
|---|---|
| **macOS** | システム設定 → 外観 → 「ダーク」を選択 |
| **Windows** | 設定 → 個人用設定 → 色 → 「ダーク」を選択 |
| **iOS** | 設定 → 画面表示と明るさ → 「ダーク」を選択 |
| **Android** | 設定 → ディスプレイ → ダークテーマ を有効化 |

- すべての主要OSがシステムレベルでダークモードをサポート
- OSの設定変更はリアルタイムでブラウザに反映される
- 一部のOSでは時間帯による自動切り替え機能もあり

### 参照元
- [Apple — ダークモード](https://support.apple.com/ja-jp/guide/mac-help/mchl52e1c2d2/mac)
- [Microsoft — Windows のカラーの変更](https://support.microsoft.com/ja-jp/windows/)

---

## CSS での基本的な使い方

### 基本構文

```css
/* ライトモード（デフォルト） */
body {
  background-color: #ffffff;
  color: #1a1a1a;
}

/* ダークモード */
@media (prefers-color-scheme: dark) {
  body {
    background-color: #1a1a2e;
    color: #e0e0e0;
  }
}
```

- `@media (prefers-color-scheme: dark)` でダークモード時のスタイルを定義
- `@media (prefers-color-scheme: light)` でライトモード時のスタイルを明示的に定義することも可能
- 一般的にはライトモードをデフォルトとし、ダークモード用の上書きを記述するアプローチが多い

### 参照元
- [MDN — @media prefers-color-scheme](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-color-scheme)

---

## color-scheme プロパティと meta タグ

### color-scheme CSS プロパティ

```css
/* ライトとダークの両方に対応していることをブラウザに宣言 */
:root {
  color-scheme: light dark;
}
```

- ブラウザのデフォルトUI（スクロールバー、フォーム要素、選択ハイライトなど）をカラーテーマに適応させる
- `light dark` と記述すると、ブラウザが自動的にフォーム要素やスクロールバーのスタイルを切り替える
- ページ全体に適用する場合は `:root` に指定

### meta タグ

```html
<!-- head内に記述 -->
<meta name="color-scheme" content="light dark">
```

- CSSの読み込み前にブラウザへカラースキーム対応を通知
- CSSがまだ読み込まれていない段階でのFOUC（Flash of Unstyled Content）を防止
- ページの初期描画時から適切な背景色が適用される

### 悪い例：color-scheme を設定しない

```html
<!-- meta タグなし -->
<html>
<head>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <!-- CSS読み込み前にライトモードの白背景が一瞬表示され、 -->
  <!-- ダークモードユーザーに画面のちらつきが発生 -->
</body>
</html>
```

### 良い例：color-scheme を設定する

```html
<html>
<head>
  <meta name="color-scheme" content="light dark">
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <!-- ブラウザが初期描画時からダークモードの背景色を適用 -->
  <!-- FOUCが発生しない -->
</body>
</html>
```

### 参照元
- [MDN — color-scheme](https://developer.mozilla.org/ja/docs/Web/CSS/color-scheme)
- [web.dev — color-scheme](https://web.dev/articles/color-scheme)

---

## CSS カスタムプロパティを使ったカラーパレット設計

### 悪い例：色を直接ハードコードする

```css
/* 悪い例：各所に色の値を直接記述 */
.card {
  background-color: #ffffff;
  color: #333333;
  border: 1px solid #cccccc;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

@media (prefers-color-scheme: dark) {
  .card {
    background-color: #2d2d2d;
    color: #e0e0e0;
    border: 1px solid #444444;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
  }
}

/* コンポーネントが増えるたびに同じ色指定を繰り返すことになり、保守性が低下 */
```

### 良い例：CSS カスタムプロパティでカラーパレットを一元管理

```css
/* 良い例：カスタムプロパティでカラートークンを定義 */
:root {
  color-scheme: light dark;

  /* ライトモードのカラーパレット */
  --color-bg-primary: #ffffff;
  --color-bg-secondary: #f5f5f5;
  --color-bg-elevated: #ffffff;
  --color-text-primary: #1a1a1a;
  --color-text-secondary: #555555;
  --color-border: #d0d0d0;
  --color-shadow: rgba(0, 0, 0, 0.1);
  --color-accent: #0066cc;
  --color-accent-text: #ffffff;
  --color-success: #2e7d32;
  --color-error: #c62828;
  --color-warning: #f57f17;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-primary: #121212;
    --color-bg-secondary: #1e1e1e;
    --color-bg-elevated: #2d2d2d;
    --color-text-primary: #e0e0e0;
    --color-text-secondary: #a0a0a0;
    --color-border: #3a3a3a;
    --color-shadow: rgba(0, 0, 0, 0.4);
    --color-accent: #5c9ce6;
    --color-accent-text: #121212;
    --color-success: #66bb6a;
    --color-error: #ef5350;
    --color-warning: #ffb74d;
  }
}

/* コンポーネントではカスタムプロパティを参照するだけ */
.card {
  background-color: var(--color-bg-elevated);
  color: var(--color-text-primary);
  border: 1px solid var(--color-border);
  box-shadow: 0 2px 4px var(--color-shadow);
}

.button-primary {
  background-color: var(--color-accent);
  color: var(--color-accent-text);
}
```

- カラーパレットの変更が `:root` の定義だけで完結
- コンポーネント側のCSSを一切変更せずにテーマ切り替えが可能
- セマンティックな命名（`--color-bg-primary` など）で用途が明確

### 参照元
- [MDN — CSS カスタムプロパティの使用](https://developer.mozilla.org/ja/docs/Web/CSS/Using_CSS_custom_properties)
- [web.dev — Building a color scheme](https://web.dev/articles/building/a-color-scheme)

---

## ライトモード・ダークモードの切り替え実装

### パターン1: CSS のみの実装（メディアクエリ）

```css
:root {
  color-scheme: light dark;
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #121212;
    --color-text: #e0e0e0;
  }
}

body {
  background-color: var(--color-bg);
  color: var(--color-text);
}
```

- 最もシンプルな実装。JavaScriptが不要
- OS設定に完全に連動
- ユーザーがサイト単位で切り替える手段がない点がデメリット

### パターン2: JavaScript によるトグル切り替え + ユーザー設定の保存

```html
<button
  id="theme-toggle"
  type="button"
  aria-label="カラーテーマを切り替える"
  aria-pressed="false"
>
  テーマ切り替え
</button>
```

```css
/* data属性でテーマを管理 */
:root {
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
}

[data-theme="dark"] {
  --color-bg: #121212;
  --color-text: #e0e0e0;
}

body {
  background-color: var(--color-bg);
  color: var(--color-text);
}
```

```js
const toggle = document.getElementById('theme-toggle');

// localStorageから保存済みのテーマを取得
const savedTheme = localStorage.getItem('theme');
if (savedTheme) {
  document.documentElement.setAttribute('data-theme', savedTheme);
  toggle.setAttribute('aria-pressed', savedTheme === 'dark' ? 'true' : 'false');
}

toggle.addEventListener('click', () => {
  const currentTheme = document.documentElement.getAttribute('data-theme');
  const newTheme = currentTheme === 'dark' ? 'light' : 'dark';

  document.documentElement.setAttribute('data-theme', newTheme);
  localStorage.setItem('theme', newTheme);
  toggle.setAttribute('aria-pressed', newTheme === 'dark' ? 'true' : 'false');
});
```

- ユーザーが手動でテーマを選択可能
- `localStorage` で設定が永続化され、再訪問時にも適用
- `aria-label` と `aria-pressed` でスクリーンリーダーにもトグル状態を伝達

### パターン3: システム設定 + 手動切り替えの共存（推奨）

```html
<button
  id="theme-toggle"
  type="button"
  aria-label="カラーテーマを切り替える"
>
  テーマ切り替え
</button>
```

```css
:root {
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
}

/* システム設定によるダークモード */
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --color-bg: #121212;
    --color-text: #e0e0e0;
  }
}

/* 手動切り替えによるダークモード */
[data-theme="dark"] {
  --color-bg: #121212;
  --color-text: #e0e0e0;
}

body {
  background-color: var(--color-bg);
  color: var(--color-text);
  transition: background-color 0.2s ease, color 0.2s ease;
}
```

```js
const toggle = document.getElementById('theme-toggle');

function getSystemTheme() {
  return window.matchMedia('(prefers-color-scheme: dark)').matches
    ? 'dark'
    : 'light';
}

function applyTheme() {
  const savedTheme = localStorage.getItem('theme');

  if (savedTheme) {
    // ユーザーが手動で選択したテーマを優先
    document.documentElement.setAttribute('data-theme', savedTheme);
  } else {
    // 手動選択がなければシステム設定に従う（data-theme を除去）
    document.documentElement.removeAttribute('data-theme');
  }
}

// 初期適用
applyTheme();

// 手動切り替え
toggle.addEventListener('click', () => {
  const currentTheme =
    document.documentElement.getAttribute('data-theme') || getSystemTheme();
  const newTheme = currentTheme === 'dark' ? 'light' : 'dark';

  document.documentElement.setAttribute('data-theme', newTheme);
  localStorage.setItem('theme', newTheme);
});

// OS設定変更のリアルタイム検知
window
  .matchMedia('(prefers-color-scheme: dark)')
  .addEventListener('change', () => {
    // ユーザーが手動選択していない場合のみ反映
    if (!localStorage.getItem('theme')) {
      applyTheme();
    }
  });
```

- OS設定を尊重しつつ、ユーザーがサイト単位で上書き可能
- `localStorage` にテーマが保存されていなければシステム設定に従う
- OS設定変更を `matchMedia` の `change` イベントでリアルタイム検知
- `:root:not([data-theme="light"])` でシステム設定と手動設定の競合を防止

### 参照元
- [MDN — Window.matchMedia()](https://developer.mozilla.org/ja/docs/Web/API/Window/matchMedia)
- [MDN — Web Storage API](https://developer.mozilla.org/ja/docs/Web/API/Web_Storage_API)
- [web.dev — prefers-color-scheme](https://web.dev/articles/prefers-color-scheme)

---

## ダークモードでのコントラスト比の注意点

### 純黒 `#000000` を背景に使うべきでない理由

- 純黒と純白（`#000` と `#fff`）の組み合わせはコントラスト比が 21:1 と極端に高い
- 高すぎるコントラストは逆に眼精疲労を引き起こす（ハレーション現象）
- 特にOLEDディスプレイでは純黒の周囲が「にじむ」ように見える現象が発生
- 推奨される背景色は `#121212`〜`#1a1a2e` 程度のダークグレー

### 悪い例：純黒背景に純白テキスト

```css
/* 悪い例：コントラスト比が高すぎる */
body {
  background-color: #000000;
  color: #ffffff;
}
```

### 良い例：ダークグレー背景にオフホワイトテキスト

```css
/* 良い例：適度なコントラストで目に優しい */
body {
  background-color: #121212;
  color: #e0e0e0; /* コントラスト比 約 13.8:1 */
}
```

### テキストの明度調整

- WCAGが求める最低コントラスト比は通常テキストで **4.5:1**、大きなテキストで **3:1**
- ダークモードでは本文テキストに `#e0e0e0`〜`#d4d4d4` 程度を推奨
- 補助テキストには `#a0a0a0` 程度を使用し、最低 **4.5:1** を確保
- ダークモード用のコントラスト比チェックも必ず実施すること

### 参照元
- [WCAG 2.2 — 1.4.3 Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)
- [Material Design — Dark theme](https://m2.material.io/design/color/dark-theme.html)

---

## 画像・メディアの対応

### filter を使った画像の明度調整

```css
/* ダークモードで画像の明度を少し下げて馴染ませる */
@media (prefers-color-scheme: dark) {
  img:not([src*=".svg"]) {
    filter: brightness(0.85);
  }
}
```

### mix-blend-mode を使った調整

```css
/* ダークモードでロゴ画像を背景に馴染ませる */
@media (prefers-color-scheme: dark) {
  .logo {
    mix-blend-mode: screen;
  }
}
```

### picture 要素の source media でモード別画像を出し分ける

```html
<!-- 良い例：ダークモード用の画像を別途用意 -->
<picture>
  <source
    srcset="chart-dark.png"
    media="(prefers-color-scheme: dark)"
  >
  <img src="chart-light.png" alt="月次売上の推移グラフ">
</picture>
```

- グラフや図解など、背景色が固定されている画像はモード別に用意するのが最適
- `<picture>` 要素の `<source>` にメディアクエリを指定可能
- SVGはCSS変数やcurrentColorを使ってテーマに自動対応させることが可能

### SVG を currentColor で対応する

```css
/* 良い例：SVGアイコンをテキスト色に追従させる */
.icon {
  fill: currentColor;
}
```

```html
<svg class="icon" viewBox="0 0 24 24" aria-hidden="true">
  <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5" />
</svg>
```

### 参照元
- [MDN — picture 要素](https://developer.mozilla.org/ja/docs/Web/HTML/Element/picture)
- [MDN — filter](https://developer.mozilla.org/ja/docs/Web/CSS/filter)
- [web.dev — prefers-color-scheme: media query](https://web.dev/articles/prefers-color-scheme)

---

## よくある失敗パターン

### 1. ハードコードされた色

```css
/* 悪い例：色が直接記述されており、ダークモード対応が困難 */
.alert {
  background-color: #fff3cd;
  color: #856404;
  border: 1px solid #ffc107;
}

/* すべてのコンポーネントで個別にダークモード対応が必要になる */
```

```css
/* 良い例：カスタムプロパティで管理 */
.alert {
  background-color: var(--color-alert-bg);
  color: var(--color-alert-text);
  border: 1px solid var(--color-alert-border);
}
```

### 2. ボックスシャドウの調整忘れ

```css
/* 悪い例：ダークモードで影が見えない */
.card {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* ダーク背景に暗い影を落としても視認できない */
```

```css
/* 良い例：ダークモードでは影の代わりにボーダーやハイライトで奥行きを表現 */
:root {
  --shadow-card: 0 2px 8px rgba(0, 0, 0, 0.1);
}

@media (prefers-color-scheme: dark) {
  :root {
    --shadow-card: 0 2px 8px rgba(0, 0, 0, 0.5);
    /* または影の代わりにボーダーで区切る */
  }

  .card {
    border: 1px solid var(--color-border);
  }
}

.card {
  box-shadow: var(--shadow-card);
}
```

### 3. ステータスカラーの調整忘れ

```css
/* 悪い例：ライトモード用のステータス色をそのまま使用 */
.badge-success {
  background-color: #2e7d32; /* ダーク背景上でコントラスト不足 */
  color: #ffffff;
}

.badge-error {
  background-color: #c62828; /* ダーク背景上でコントラスト不足 */
  color: #ffffff;
}
```

```css
/* 良い例：ダークモードではステータスカラーの明度を上げて視認性を確保 */
:root {
  --color-success-bg: #2e7d32;
  --color-success-text: #ffffff;
  --color-error-bg: #c62828;
  --color-error-text: #ffffff;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-success-bg: #66bb6a;
    --color-success-text: #1a1a1a;
    --color-error-bg: #ef5350;
    --color-error-text: #1a1a1a;
  }
}

.badge-success {
  background-color: var(--color-success-bg);
  color: var(--color-success-text);
}

.badge-error {
  background-color: var(--color-error-bg);
  color: var(--color-error-text);
}
```

### 4. transitionの未設定によるちらつき

```css
/* 悪い例：テーマ切り替え時に色が瞬時に変わり違和感がある */
body {
  background-color: var(--color-bg);
  color: var(--color-text);
}
```

```css
/* 良い例：テーマ切り替え時にスムーズなトランジションを適用 */
body {
  background-color: var(--color-bg);
  color: var(--color-text);
  transition: background-color 0.2s ease, color 0.2s ease;
}

/* ただし、初回読み込み時のトランジションを防ぐために
   JavaScriptでページ読み込み完了後にクラスを付与する工夫が必要 */
```

### 参照元
- [Material Design — Dark theme](https://m2.material.io/design/color/dark-theme.html)
- [web.dev — prefers-color-scheme](https://web.dev/articles/prefers-color-scheme)

---

## まとめ

- `prefers-color-scheme` はユーザーのOS設定を尊重するアクセシビリティ対応の基本
- `<meta name="color-scheme">` と `color-scheme` プロパティでブラウザUIも適応させる
- CSS カスタムプロパティでカラーパレットを一元管理し、保守性を確保
- システム設定 + 手動切り替えの共存パターンが最もユーザーフレンドリー
- 純黒 `#000` は避け、ダークグレー（`#121212` 前後）を背景に使用
- コントラスト比はダークモードでも **WCAG 4.5:1** 以上を維持
- 画像・影・ステータスカラーなど、見落としやすい要素も必ず対応
