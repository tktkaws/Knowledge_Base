# ボックスモデル — content-box vs border-box の違い

## ボックスモデルとは

- CSSにおいて、すべてのHTML要素は「ボックス（箱）」として扱われる
- ボックスは内側から外側に向かって4つの領域で構成される
- レイアウトの計算、要素のサイズ指定、要素間のスペース調整はすべてこのモデルに基づく
- ボックスモデルの理解はCSS全体の基盤

```
┌─────────────────────────────────────────────┐
│                  margin                     │
│  ┌───────────────────────────────────────┐  │
│  │              border                   │  │
│  │  ┌─────────────────────────────────┐  │  │
│  │  │            padding              │  │  │
│  │  │  ┌───────────────────────────┐  │  │  │
│  │  │  │                           │  │  │  │
│  │  │  │         content           │  │  │  │
│  │  │  │                           │  │  │  │
│  │  │  └───────────────────────────┘  │  │  │
│  │  └─────────────────────────────────┘  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

> 参照: [MDN - The box model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model)

## 4つの領域

### 1. content（コンテンツ領域）

- テキスト・画像などの実際のコンテンツが表示される領域
- `width` と `height` プロパティでサイズを指定
- `box-sizing` の値によって、`width` / `height` が指す範囲が変わる（後述）

### 2. padding（パディング領域）

- コンテンツ領域の外側、ボーダーの内側に位置する余白
- 要素の背景色（`background-color`）はパディング領域にも適用される
- **負の値は指定不可**

```css
/* 一括指定 */
padding: 16px;                    /* 上下左右すべて 16px */
padding: 16px 24px;               /* 上下 16px、左右 24px */
padding: 8px 16px 24px;           /* 上 8px、左右 16px、下 24px */
padding: 8px 16px 24px 32px;      /* 上 8px、右 16px、下 24px、左 32px */

/* 個別指定 */
padding-top: 8px;
padding-right: 16px;
padding-bottom: 24px;
padding-left: 32px;
```

### 3. border（ボーダー領域）

- パディングの外側を囲む線
- `width`（太さ）、`style`（線種）、`color`（色）の3つのプロパティで構成

```css
/* 一括指定 */
border: 2px solid #333;

/* 辺ごとの指定 */
border-top: 1px solid #ccc;
border-bottom: 3px double #000;
```

### 4. margin（マージン領域）

- ボーダーの外側の余白。要素間のスペースを制御
- 要素の背景色は適用されない（常に透明）
- **負の値を指定可能**（要素を重ねる用途に利用）
- **マージンの相殺（margin collapsing）** という特殊な振る舞いがある（後述）

```css
/* 一括指定 */
margin: 16px;                     /* 上下左右すべて 16px */
margin: 0 auto;                   /* 上下 0、左右はブロック要素の中央揃え */

/* 個別指定 */
margin-top: 8px;
margin-bottom: 24px;
```

> 参照: [MDN - The box model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model)

## box-sizing プロパティ

- `box-sizing` は `width` と `height` がボックスのどの範囲を指すかを決定する
- 取りうる値は `content-box`（初期値）と `border-box` の2つ
- **マージンはどちらの場合もサイズ計算に含まれない**

| プロパティ | 初期値 | 継承 |
|---|---|---|
| `box-sizing` | `content-box` | なし |

> 参照: [MDN - box-sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing)

## content-box（初期値）

- CSSの仕様上のデフォルト値
- `width` / `height` は **コンテンツ領域のみ** のサイズを指す
- パディングとボーダーは指定した `width` / `height` の **外側に加算** される

### 計算式

```
要素の実際の幅 = width + padding-left + padding-right + border-left-width + border-right-width
要素の実際の高さ = height + padding-top + padding-bottom + border-top-width + border-bottom-width
```

### 計算例

```css
.box-content {
  box-sizing: content-box; /* デフォルト値 */
  width: 300px;
  height: 200px;
  padding: 20px;
  border: 5px solid #333;
}
```

```
実際の幅 = 300 + 20 + 20 + 5 + 5 = 350px
実際の高さ = 200 + 20 + 20 + 5 + 5 = 250px

※ widthに300pxを指定したが、画面上の要素は350px幅になる
```

## border-box

- `width` / `height` は **コンテンツ + パディング + ボーダーを含めた** サイズを指す
- パディングとボーダーは指定した `width` / `height` の **内側に収まる**
- コンテンツ領域は自動的に縮小される
- 指定した値がそのまま要素の表示サイズになるため、直感的

### 計算式

```
要素の実際の幅 = width（指定値のまま）
コンテンツ領域の幅 = width - padding-left - padding-right - border-left-width - border-right-width
```

### 計算例

```css
.box-border {
  box-sizing: border-box;
  width: 300px;
  height: 200px;
  padding: 20px;
  border: 5px solid #333;
}
```

```
実際の幅 = 300px（指定通り）
コンテンツ領域の幅 = 300 - 20 - 20 - 5 - 5 = 250px

実際の高さ = 200px（指定通り）
コンテンツ領域の高さ = 200 - 20 - 20 - 5 - 5 = 150px
```

> 参照: [MDN - box-sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing)

## 比較表

同じCSSプロパティ値でも `box-sizing` によって実際のサイズが異なる。

```css
.box {
  width: 200px;
  padding: 20px;
  border: 5px solid black;
}
```

| | content-box | border-box |
|---|---|---|
| 要素の実際の幅 | 200 + 40 + 10 = **250px** | **200px** |
| コンテンツ領域の幅 | **200px** | 200 - 40 - 10 = **150px** |
| widthが指す範囲 | コンテンツ領域のみ | コンテンツ + パディング + ボーダー |

## border-box が推奨される理由

### 1. サイズ計算が直感的

- `width: 300px` と指定すれば、実際に300pxとして描画される
- パディングやボーダーを後から追加しても要素の外形サイズが変わらない
- デザインカンプとの照合が容易

### 2. レスポンシブデザインとの相性が良い

- `width: 50%` と指定した場合、`border-box` ならパディングやボーダーを含めて親の50%に収まる
- `content-box` だと50%を超えてしまいレイアウトが崩れやすい

### 3. CSSフレームワークのデファクト

- Bootstrap、Tailwind CSS、Normalize.css など主要なCSSフレームワーク / リセットCSSは `border-box` をグローバルに適用している

> 参照: [CSS-Tricks - Inheriting box-sizing](https://css-tricks.com/inheriting-box-sizing-probably-slightly-better-best-practice/)

## ユニバーサル box-sizing リセット

### 基本パターン

```css
/* すべての要素に border-box を直接適用 */
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

### 推奨パターン（inherit を活用）

```css
html {
  box-sizing: border-box;
}

*,
*::before,
*::after {
  box-sizing: inherit;
}
```

- `inherit` を使うことで、サードパーティのコンポーネントが独自の `box-sizing` を持つ場合にも対応できる
- コンポーネントの独立性を尊重しつつ、デフォルトは `border-box` にできる

### 悪い例 vs 良い例

```css
/* 悪い例：リセットなしでwidth: 100% + paddingを使う */
.container {
  width: 100%;
  padding: 20px;
  /* 実際の幅は 100% + 40px → 横スクロールバーが出る */
}
```

```css
/* 良い例：border-box リセット済みの状態 */
*,
*::before,
*::after {
  box-sizing: border-box;
}

.container {
  width: 100%;
  padding: 20px;
  /* 実際の幅はちょうど 100%。パディングは内側に収まる */
}
```

> 参照: [CSS-Tricks - Inheriting box-sizing](https://css-tricks.com/inheriting-box-sizing-probably-slightly-better-best-practice/)

## パーセント幅・min/max との相互作用

### パーセント幅との組み合わせ

```css
/* 悪い例：content-box + パーセント幅 + padding でレイアウト崩れ */
.sidebar {
  box-sizing: content-box;
  width: 30%;
  padding: 20px;
  /* 実際の幅 = 30% + 40px → 親の30%を超える */
}
.main {
  box-sizing: content-box;
  width: 70%;
  padding: 20px;
  /* 実際の幅 = 70% + 40px → 親の70%を超える */
}
/* sidebar + main = 100% + 80px → はみ出して折り返し発生 */
```

```css
/* 良い例：border-box + パーセント幅 */
.sidebar {
  box-sizing: border-box;
  width: 30%;
  padding: 20px;
  /* 実際の幅 = ちょうど30% */
}
.main {
  box-sizing: border-box;
  width: 70%;
  padding: 20px;
  /* 実際の幅 = ちょうど70% */
}
/* sidebar + main = ちょうど100% → ぴったり収まる */
```

### calc() との組み合わせ

```css
/* 悪い例：content-box で calc() を使って調整（冗長） */
.item {
  box-sizing: content-box;
  width: calc(50% - 40px); /* padding 20px × 2 を差し引く */
  padding: 20px;
}

/* 良い例：border-box なら calc() 不要 */
.item {
  box-sizing: border-box;
  width: 50%;
  padding: 20px;
}
```

> 参照: [MDN - box-sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing)

## よくあるレイアウトバグ

### バグ1: 横スクロールバーの発生

```css
/* 悪い例 */
.full-width-section {
  width: 100%;
  padding: 0 32px;
  /* content-box: 実際の幅 = 100% + 64px → はみ出す */
}

/* 良い例 */
.full-width-section {
  box-sizing: border-box;
  width: 100%;
  padding: 0 32px;
  /* 実際の幅 = ちょうど100% */
}
```

### バグ2: パディング追加後のデザイン崩れ

```css
/* 悪い例：content-box でpaddingを変更するとサイズが変わる */
.card {
  box-sizing: content-box;
  width: 300px;
  padding: 16px;
  /* 実際の幅 = 332px */
}
/* paddingを増やすと... */
.card {
  padding: 32px;
  /* 実際の幅 = 364px に変化 → グリッドからはみ出す */
}

/* 良い例：border-box ならpadding変更しても外形は不変 */
.card {
  box-sizing: border-box;
  width: 300px;
  padding: 16px; /* → 32px に変更しても実際の幅は300pxのまま */
}
```

### バグ3: マージンの相殺（Margin Collapsing）

- 隣接するブロック要素の上下マージンは「合算」ではなく「大きいほうが採用」される
- `box-sizing` とは独立した仕組みだが、ボックスモデルの知識として重要

```css
.heading {
  margin-bottom: 24px;
}
.paragraph {
  margin-top: 16px;
}
/* 間のスペースは 24 + 16 = 40px ではなく、24px（大きいほうが採用） */
```

- マージンの相殺が発生する条件
  - 隣接する兄弟要素の上下マージン
  - 親要素と最初/最後の子要素のマージン（パディングやボーダーで隔てられていない場合）
  - 空のブロック要素の上下マージン
- 相殺を防ぐ方法
  - 親要素に `padding` または `border` を設定する
  - 親要素に `overflow: hidden` を設定する
  - 親要素に `display: flex` または `display: grid` を設定する

> 参照: [MDN - Mastering margin collapsing](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing)

### バグ4: インライン要素の width / height が効かない

- `<span>`、`<a>`、`<strong>` などのインライン要素には `width` / `height` が適用されない
- `box-sizing` を変更しても解決しない

```css
/* 悪い例：インライン要素にwidthを指定しても無効 */
span {
  width: 200px;     /* 無視される */
  height: 100px;    /* 無視される */
}

/* 良い例：display を変更する */
span {
  display: inline-block;
  width: 200px;     /* 適用される */
  height: 100px;    /* 適用される */
}
```

> 参照: [MDN - The box model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model)

## DevTools でボックスモデルを確認する方法

1. DevTools を開く（F12 または Cmd+Opt+I）
2. **Elements** パネルで対象の要素を選択
3. 右側ペインの **Computed** タブを選択
4. ボックスモデルの図が表示される

```
ボックスモデルの図の見方:

┌─ margin（オレンジ）──────────────────────────┐
│  ┌─ border（黄色）──────────────────────┐    │
│  │  ┌─ padding（緑）──────────────┐     │    │
│  │  │  ┌─ content（青）──────┐    │     │    │
│  │  │  │    350 × 150       │    │     │    │
│  │  │  └────────────────────┘    │     │    │
│  │  └────────────────────────────┘     │    │
│  └──────────────────────────────────────┘    │
└──────────────────────────────────────────────┘
```

### 確認すべきポイント

- content領域のサイズ（中央の数値）
- 各辺のmargin / border / padding の値
- `box-sizing` の値（Computed タブのプロパティ一覧）
- ページ上でホバーすると、margin（オレンジ）/ padding（緑）/ content（青）が色分け表示される
- ボックスモデル図の数値はダブルクリックで編集可能

> 参照: [Chrome Developers - CSS reference](https://developer.chrome.com/docs/devtools/css)

## まとめ

- ボックスモデルはCSSレイアウトの基盤。すべての要素は content / padding / border / margin の4層で構成される
- `content-box`（デフォルト）は `width` / `height` がコンテンツ領域のみを指し、padding・borderが外側に加算される
- `border-box` は `width` / `height` が padding・border を含む全体を指し、指定した値がそのまま表示サイズになる
- モダン開発では `border-box` のユニバーサルリセットが標準
- パーセント幅 + padding の組み合わせは `content-box` で特に問題を起こしやすい
- マージンの相殺、インライン要素の挙動は頻出のつまずきポイント
- DevTools の Computed タブでボックスモデルを確認する習慣をつける

## 参照元

- [MDN - The box model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model)
- [MDN - box-sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing)
- [MDN - Mastering margin collapsing](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing)
- [CSS-Tricks - Inheriting box-sizing](https://css-tricks.com/inheriting-box-sizing-probably-slightly-better-best-practice/)
- [Chrome Developers - CSS reference](https://developer.chrome.com/docs/devtools/css)
