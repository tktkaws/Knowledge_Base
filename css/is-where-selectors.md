# :is() / :where() セレクタの違いと使い分け

## :is() と :where() とは

- 複数のセレクタをグルーピングして**冗長な記述を減らす**ための擬似クラス関数
- どちらも引数にセレクタリスト（カンマ区切り）を受け取り、いずれかに一致すれば条件を満たす
- 機能的にはほぼ同じだが、**詳細度（Specificity）の扱いが異なる**点が最大の違い
- `:is()` は引数の中で最も高い詳細度を採用する
- `:where()` は常に詳細度が **0** になる

> 参照: [MDN - :is()](https://developer.mozilla.org/en-US/docs/Web/CSS/:is) / [MDN - :where()](https://developer.mozilla.org/en-US/docs/Web/CSS/:where)

## 基本構文

```css
/* :is() — 引数のいずれかにマッチ（詳細度あり） */
:is(h1, h2, h3) {
  color: #333;
}

/* :where() — 引数のいずれかにマッチ（詳細度なし） */
:where(h1, h2, h3) {
  color: #333;
}
```

- 引数にはセレクタリストを指定する（型セレクタ、クラス、ID、擬似クラスなど何でも可）
- ネストして使うこともできる

> 参照: [MDN - :is()](https://developer.mozilla.org/en-US/docs/Web/CSS/:is)

## 冗長なセレクタの簡略化

### :is() / :where() を使わない場合

```css
/* 悪い例：同じスタイルを繰り返し記述する必要がある */
header a:hover,
nav a:hover,
footer a:hover {
  color: #0066cc;
  text-decoration: underline;
}

article h1,
article h2,
article h3 {
  font-weight: bold;
  line-height: 1.3;
}
```

### :is() で簡略化

```css
/* 良い例：:is()でグルーピング */
:is(header, nav, footer) a:hover {
  color: #0066cc;
  text-decoration: underline;
}

article :is(h1, h2, h3) {
  font-weight: bold;
  line-height: 1.3;
}
```

- セレクタの**任意の位置**で使える（先頭、途中、末尾）
- ネストの各レベルで使うことも可能

```css
/* 複数箇所で同時に使う */
:is(header, footer) :is(h1, h2) {
  margin-block: 0.5rem;
}
/* header h1, header h2, footer h1, footer h2 と同等 */
```

> 参照: [MDN - :is()](https://developer.mozilla.org/en-US/docs/Web/CSS/:is)

## 詳細度の違い — :is() vs :where()

- `:is()` → 引数の中で**最も詳細度が高いセレクタ**の値を採用
- `:where()` → 常に詳細度 **0**

```css
/* :is(#id, .class, element) の詳細度は #id の (1, 0, 0) */
:is(#hero, .card, article) {
  padding: 1rem;
}
/* この宣言全体の詳細度は (1, 0, 0) */

/* :where(#id, .class, element) の詳細度は常に (0, 0, 0) */
:where(#hero, .card, article) {
  padding: 1rem;
}
/* この宣言全体の詳細度は (0, 0, 0) */
```

### 詳細度の比較表

| セレクタ | 詳細度 |
|---|---|
| `:is(h1, h2, h3)` | `(0, 0, 1)` ← 最も高い `h1` の詳細度 |
| `:is(.card, .hero)` | `(0, 1, 0)` ← 最も高い `.card` の詳細度 |
| `:is(#main, .card)` | `(1, 0, 0)` ← 最も高い `#main` の詳細度 |
| `:where(h1, h2, h3)` | `(0, 0, 0)` ← 常に0 |
| `:where(#main, .card)` | `(0, 0, 0)` ← 常に0 |

> 参照: [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)

## :is() を使う場面

- 通常のスタイリングで**セレクタの冗長さを減らしたい**とき
- 詳細度が上がっても問題ないとき
- コンポーネント内など、**他との競合を気にしなくてよい**スコープ

```css
/* コンポーネント内のスタイリング */
.card :is(h2, h3, h4) {
  color: #1a1a1a;
  margin-bottom: 0.5rem;
}

.card :is(p, ul, ol) {
  color: #555;
  line-height: 1.7;
}
```

> 参照: [MDN - :is()](https://developer.mozilla.org/en-US/docs/Web/CSS/:is)

## :where() を使う場面

### デフォルトスタイル / リセットCSS

- `:where()` の詳細度が0であることを活かし、**あとから簡単に上書きできるベーススタイル**を定義するのに最適

```css
/* 悪い例：:is()でリセットを書くと詳細度が邪魔になる */
:is(ul, ol) {
  list-style: none;  /* 詳細度 (0, 0, 1) */
  padding: 0;
}
/* 上書きには同等以上の詳細度が必要 */

/* 良い例：:where()でリセットを書けば簡単に上書きできる */
:where(ul, ol) {
  list-style: none;  /* 詳細度 (0, 0, 0) */
  padding: 0;
}
/* どんなセレクタでも上書き可能 */
ul {
  list-style: disc;  /* (0, 0, 1) で簡単に上書きできる */
  padding-left: 1.5rem;
}
```

### CSSライブラリ / デザインシステムのベーススタイル

```css
/* ライブラリ側：:where()で詳細度を最低にする */
:where(.btn) {
  padding: 0.5rem 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: white;
}

:where(.btn:hover) {
  background: #f5f5f5;
}

/* 利用者側：クラス1つで簡単にカスタマイズできる */
.btn {
  background: #0066cc;
  color: white;
  border-color: #0066cc;
}
```

- ライブラリのスタイルを `:where()` で定義しておくと、利用者が**詳細度の戦いをせずに**カスタマイズできる

> 参照: [web.dev - :is() and :where()](https://web.dev/articles/css-is-and-where)

## :is() と :where() の組み合わせ

- 1つのセレクタ内で両方を使い分けることも可能
- **一部は詳細度を残し、一部は詳細度を消す**という細かな制御ができる

```css
/* .card部分は詳細度に含め、要素セレクタ部分は詳細度0にする */
.card :where(h2, h3, h4) {
  font-weight: bold;
}
/* 詳細度: (0, 1, 0) — :where()部分は0 */

/* 比較: :is()を使った場合 */
.card :is(h2, h3, h4) {
  font-weight: bold;
}
/* 詳細度: (0, 1, 1) — :is(h2, h3, h4)が(0, 0, 1)を加算 */
```

> 参照: [MDN - :where()](https://developer.mozilla.org/en-US/docs/Web/CSS/:where)

## 寛容なセレクタリスト

- `:is()` と `:where()` は**寛容なセレクタリスト（forgiving selector list）** を使用する
- 引数の1つが無効でも、**他の有効なセレクタは正常にマッチ**する

```css
/* 従来のセレクタリスト — 1つでも無効だとルール全体が無効 */
h1, h2, :unsupported, h3 {
  color: red;
  /* ブラウザが:unsupportedを認識しない → ルール全体が無視される */
}

/* :is() — 無効なセレクタを無視して残りは有効 */
:is(h1, h2, :unsupported, h3) {
  color: red;
  /* :unsupportedは無視され、h1, h2, h3 にスタイルが適用される */
}
```

- プレフィックス付きセレクタや実験的な機能と併用する際に安全

```css
/* ベンダープレフィックスを安全に混在させる */
:is(::-webkit-slider-thumb, ::-moz-range-thumb) {
  appearance: none;
  width: 20px;
  height: 20px;
}
```

> 参照: [MDN - :is()](https://developer.mozilla.org/en-US/docs/Web/CSS/:is)

## CSS Nestingとの併用

- CSS Nesting内で `:is()` / `:where()` を使うとさらに簡潔に書ける

```css
.card {
  padding: 1rem;

  & :is(h2, h3) {
    color: #1a1a1a;
  }

  & :where(p, span) {
    color: #555;
  }

  &:is(:hover, :focus-within) {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }
}
```

- ネスティングの `&` と組み合わせることで、従来のSassのような書き味で使える

> 参照: [MDN - CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)

## 使い分けの判断フロー

```
セレクタをグルーピングしたい
  │
  ├─ 詳細度を維持してよい → :is() を使う
  │    ├─ コンポーネント内のスタイリング
  │    └─ 通常のスタイル定義
  │
  └─ 詳細度を上げたくない → :where() を使う
       ├─ リセットCSS / ノーマライズ
       ├─ ライブラリのデフォルトスタイル
       └─ 上書きされることを前提としたベーススタイル
```

## ブラウザサポート

- `:is()` — Chrome 88+、Safari 14+、Firefox 78+
- `:where()` — Chrome 88+、Safari 14+、Firefox 78+
- 2024年以降、全主要ブラウザで安定サポート済み

> 参照: [Can I Use - :is()](https://caniuse.com/css-matches-pseudo) / [Can I Use - :where()](https://caniuse.com/mdn-css_selectors_where)

## まとめ

- `:is()` と `:where()` はどちらもセレクタのグルーピングに使い、冗長な記述を減らせる
- **最大の違いは詳細度**：`:is()` は引数の最高詳細度を採用、`:where()` は常に0
- リセットCSSやライブラリのベーススタイルには `:where()` が適している
- コンポーネント内の通常のスタイリングには `:is()` が適している
- どちらも寛容なセレクタリストを採用しており、無効なセレクタがあってもルール全体が壊れない
- CSS Nestingと組み合わせるとさらに表現力が高まる

## 参照元

- [MDN - :is()](https://developer.mozilla.org/en-US/docs/Web/CSS/:is)
- [MDN - :where()](https://developer.mozilla.org/en-US/docs/Web/CSS/:where)
- [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)
- [MDN - CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)
- [web.dev - :is() and :where()](https://web.dev/articles/css-is-and-where)
- [Can I Use - :is()](https://caniuse.com/css-matches-pseudo)
- [Can I Use - :where()](https://caniuse.com/mdn-css_selectors_where)
