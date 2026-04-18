# HTMLのコンテンツモデル — フローコンテンツ・フレージングコンテンツなど

## コンテンツモデルとは

- HTML要素を「どのカテゴリに属するか」で分類する仕組み
- 各要素が「どの要素の子になれるか」「どの要素を子に持てるか」を定義する
- HTML5で導入された概念（HTML4までは「ブロック要素/インライン要素」の2分類だった）
- 1つの要素が複数のカテゴリに同時に属することがある

> 参照: [MDN - Content categories](https://developer.mozilla.org/ja/docs/Web/HTML/Content_categories)

## ブロック要素/インライン要素との違い

- HTML4以前は要素を「ブロック要素」と「インライン要素」の2つに分類していた
- HTML5ではより細かいコンテンツカテゴリに再分類された
- CSSの `display: block` / `display: inline` とHTML仕様のカテゴリは別の概念

```html
<!-- HTML4の考え方 -->
<!-- ブロック要素: div, p, h1, ul ... -->
<!-- インライン要素: span, a, strong, em ... -->

<!-- HTML5のコンテンツモデル -->
<!-- フローコンテンツ、フレージングコンテンツ、セクショニングコンテンツ... -->
<!-- 1要素が複数カテゴリに属する -->
```

> 参照: [HTML Standard - Kinds of content](https://html.spec.whatwg.org/multipage/dom.html#kinds-of-content)

## 7つのコンテンツカテゴリ

### 1. フローコンテンツ（Flow content）

- `<body>` 内で使用できるほとんどの要素が該当する最も広いカテゴリ
- 他のカテゴリの多くはフローコンテンツのサブセット

**代表的な要素:**
`<div>`, `<p>`, `<h1>`〜`<h6>`, `<ul>`, `<ol>`, `<table>`, `<form>`, `<section>`, `<article>`, `<header>`, `<footer>`, `<a>`, `<span>`, `<strong>`, `<em>`, `<img>`, `<input>` など

### 2. フレージングコンテンツ（Phrasing content）

- テキストとテキストレベルのマークアップを表す要素
- HTML4での「インライン要素」に近い概念
- フローコンテンツのサブセット

**代表的な要素:**
`<span>`, `<a>`, `<strong>`, `<em>`, `<code>`, `<br>`, `<img>`, `<input>`, `<button>`, `<select>`, `<textarea>`, `<label>`, `<time>`, `<mark>`, `<abbr>` など

**ポイント:** テキストそのもの（Text node）もフレージングコンテンツに含まれる

### 3. セクショニングコンテンツ（Sectioning content）

- 文書のアウトライン（章立て構造）を定義する要素
- 各セクショニング要素はそれぞれ独自の見出し階層を持てる

**該当する要素（4つのみ）:**
`<article>`, `<aside>`, `<nav>`, `<section>`

### 4. ヘディングコンテンツ（Heading content）

- セクションの見出しを定義する要素

**該当する要素:**
`<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`, `<hgroup>`

### 5. インタラクティブコンテンツ（Interactive content）

- ユーザーの操作を受け付ける要素

**代表的な要素:**
`<a>`（href属性あり）, `<button>`, `<input>`（type="hidden"以外）, `<select>`, `<textarea>`, `<details>`, `<label>`

### 6. エンベデッドコンテンツ（Embedded content）

- 外部リソースを文書に埋め込む要素

**代表的な要素:**
`<img>`, `<video>`, `<audio>`, `<canvas>`, `<iframe>`, `<svg>`, `<picture>`

### 7. メタデータコンテンツ（Metadata content）

- 文書のメタ情報や外部リソースとの関係を定義する要素
- 主に `<head>` 内で使用される

**代表的な要素:**
`<meta>`, `<link>`, `<title>`, `<style>`, `<script>`, `<base>`

> 参照: [MDN - Content categories](https://developer.mozilla.org/ja/docs/Web/HTML/Content_categories)

## カテゴリの重複

- 1つの要素が複数のカテゴリに同時に属する

| 要素 | 属するカテゴリ |
|------|---------------|
| `<a>` | フロー、フレージング、インタラクティブ |
| `<img>` | フロー、フレージング、エンベデッド |
| `<input>` | フロー、フレージング、インタラクティブ |
| `<button>` | フロー、フレージング、インタラクティブ |
| `<div>` | フロー |
| `<span>` | フロー、フレージング |
| `<section>` | フロー、セクショニング |
| `<h1>` | フロー、ヘディング |

> 参照: [HTML Standard - Kinds of content](https://html.spec.whatwg.org/multipage/dom.html#kinds-of-content)

## コンテンツモデルによるネスト制約

- 各要素の仕様には「許可されるコンテンツ」が定義されている
- この制約に違反すると、ブラウザが自動的にDOMを修正する（意図しない表示になる）

### p要素の制約

- `<p>` の中にはフレージングコンテンツしか入れられない

```html
<!-- 悪い例: p要素の中にdiv（フレージングコンテンツでない） -->
<p>
  テキスト
  <div>ブロック要素</div>
</p>
<!-- ブラウザがpを自動的に閉じてしまい、意図しないDOMになる -->
<!-- 実際のDOM: <p>テキスト</p><div>ブロック要素</div><p></p> -->

<!-- 良い例: p要素の中はフレージングコンテンツのみ -->
<p>
  テキスト
  <strong>重要なテキスト</strong>
</p>
```

### a要素の制約

- `<a>` の中にインタラクティブコンテンツは入れられない

```html
<!-- 悪い例: a要素の中にbutton（インタラクティブコンテンツの入れ子） -->
<a href="/page">
  <button>クリック</button>
</a>

<!-- 良い例: どちらか一方を使う -->
<a href="/page">リンクテキスト</a>
<!-- または -->
<button onclick="location.href='/page'">クリック</button>
```

### button要素の制約

- `<button>` の中にインタラクティブコンテンツは入れられない

```html
<!-- 悪い例: button要素の中にa要素 -->
<button>
  <a href="/page">リンク</a>
</button>

<!-- 良い例: buttonの中はフレージングコンテンツ（インタラクティブ以外） -->
<button>
  <span>送信</span>
</button>
```

### ul/ol要素の制約

- `<ul>` / `<ol>` の直接の子は `<li>` のみ

```html
<!-- 悪い例: ulの直接の子にdiv -->
<ul>
  <div>
    <li>項目1</li>
    <li>項目2</li>
  </div>
</ul>

<!-- 良い例: ulの直接の子はliのみ -->
<ul>
  <li>項目1</li>
  <li>項目2</li>
</ul>
```

> 参照: [MDN - 各要素のPermitted content](https://developer.mozilla.org/ja/docs/Web/HTML/Element/p)

## トランスペアレントコンテンツモデル

- 一部の要素は「透過的（transparent）」なコンテンツモデルを持つ
- 透過的な要素自体を取り除いても、子要素が親に対して有効なコンテンツであること

### 代表的な透過的要素

- `<a>` — リンクを除去しても中身が親に対して妥当であれば有効
- `<del>` / `<ins>` — 削除・挿入マークを除去しても構造が妥当であること

```html
<!-- 良い例: aを取り除いても、divの子にpは妥当 -->
<div>
  <a href="/page">
    <p>段落テキスト</p>
  </a>
</div>

<!-- 悪い例: aを取り除いてもpの子にdivは不正 -->
<p>
  <a href="/page">
    <div>ブロック要素</div>
  </a>
</p>
```

> 参照: [HTML Standard - Transparent content models](https://html.spec.whatwg.org/multipage/dom.html#transparent-content-models)

## バリデーションで確認する

- コンテンツモデル違反はブラウザがエラーを出さず自動修正するため、気づきにくい
- W3Cのバリデーターで定期的にチェックするのが有効

**ツール:**
- [W3C Markup Validation Service](https://validator.w3.org/)
- ブラウザの開発者ツール（Elementsパネルで実際のDOMを確認）

```html
<!-- pの中にdivを書いた場合のDOM（開発者ツールで確認） -->
<!-- 記述したHTML -->
<p>テキスト<div>中身</div></p>

<!-- ブラウザが解釈した実際のDOM -->
<p>テキスト</p>
<div>中身</div>
<p></p>
<!-- 空のp要素が生成されてしまう -->
```

> 参照: [W3C Validator](https://validator.w3.org/)

## まとめ

- HTML5のコンテンツモデルは7つのカテゴリで要素を分類する
- 「ブロック要素/インライン要素」はCSS上の概念であり、HTMLの仕様上は別の分類
- 各要素には「許可されるコンテンツ」が定義されており、違反するとブラウザがDOMを自動修正する
- `<p>` の中にはフレージングコンテンツのみ、`<a>` の中にインタラクティブコンテンツは不可
- バリデーターや開発者ツールで実際のDOMを確認する習慣が大切
