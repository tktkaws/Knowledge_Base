# セマンティックHTMLとは何か — div/spanの多用から脱却する

## セマンティックHTMLとは

- HTML要素を「見た目」ではなく「意味（セマンティクス）」で選ぶ考え方
- ブラウザ、検索エンジン、支援技術（スクリーンリーダーなど）がコンテンツの構造と意味を正しく理解できるようになる
- `<div>` や `<span>` は意味を持たない汎用コンテナであり、これらだけでページを構築すると情報の構造が失われる

## なぜセマンティックHTMLが重要か

- **アクセシビリティ**: スクリーンリーダーがランドマーク（`<nav>`, `<main>` など）を認識し、ユーザーがページ内を効率的に移動できる
- **SEO**: 検索エンジンがコンテンツの階層と重要度を正しく判断できる
- **保守性**: コードを読んだだけで構造の意図が伝わり、チーム開発がしやすくなる
- **ブラウザのデフォルトスタイル**: セマンティック要素にはブラウザが適切なデフォルトスタイルを適用する

## 主要なセマンティック要素

### ページ構造

| 要素 | 意味 |
|------|------|
| `<header>` | ページやセクションの導入部分 |
| `<nav>` | ナビゲーションリンクのまとまり |
| `<main>` | ページの主要コンテンツ（1ページに1つ） |
| `<aside>` | メインコンテンツと間接的に関連する補足情報 |
| `<footer>` | ページやセクションのフッター |
| `<section>` | テーマ的にまとまったコンテンツのブロック |
| `<article>` | 自己完結した独立コンテンツ（ブログ記事、ニュース記事など） |

### テキスト・インライン

| 要素 | 意味 |
|------|------|
| `<h1>`〜`<h6>` | 見出しの階層 |
| `<p>` | 段落 |
| `<strong>` | 重要性を示す強調 |
| `<em>` | ニュアンス上の強調（アクセント） |
| `<time>` | 日時 |
| `<mark>` | ハイライト（検索結果の一致箇所など） |
| `<blockquote>` | 引用文 |
| `<figure>` / `<figcaption>` | 図版とそのキャプション |

## 悪い例・良い例

### ページ構造

```html
<!-- 悪い例: divだけで構成 -->
<div class="header">
  <div class="nav">
    <a href="/">ホーム</a>
    <a href="/about">概要</a>
  </div>
</div>
<div class="main">
  <div class="article">
    <div class="title">記事タイトル</div>
    <div class="content">記事の本文...</div>
  </div>
</div>
<div class="footer">© 2026</div>
```

```html
<!-- 良い例: セマンティック要素で構成 -->
<header>
  <nav>
    <a href="/">ホーム</a>
    <a href="/about">概要</a>
  </nav>
</header>
<main>
  <article>
    <h1>記事タイトル</h1>
    <p>記事の本文...</p>
  </article>
</main>
<footer>© 2026</footer>
```

### テキストの強調

```html
<!-- 悪い例: 見た目だけのマークアップ -->
<span style="font-weight: bold;">重要なお知らせ</span>
<span style="font-style: italic;">ここに注意</span>
```

```html
<!-- 良い例: 意味のあるマークアップ -->
<strong>重要なお知らせ</strong>
<em>ここに注意</em>
```

## divとspanを使ってよい場面

- CSSのスタイリングやレイアウトのためだけにラッパーが必要な場合
- JavaScriptのフック（イベントリスナーの対象など）として使う場合
- 適切なセマンティック要素が存在しない場合

```html
<!-- OK: レイアウト目的のラッパー -->
<main>
  <div class="grid-container">
    <article>...</article>
    <article>...</article>
  </div>
</main>
```

## よくある間違い

- `<section>` をスタイル用のラッパーとして使う → `<div>` を使うべき
- `<article>` をあらゆるブロックに使う → 自己完結したコンテンツにのみ使用
- 見出しレベルを飛ばす（`<h1>` の次に `<h3>` など）→ 順序を守る
- `<nav>` をすべてのリンク集に使う → 主要なナビゲーションブロックにのみ使用

## 参照

- [MDN - HTML要素リファレンス](https://developer.mozilla.org/ja/docs/Web/HTML/Element)
- [W3C - HTML Living Standard: Semantics](https://html.spec.whatwg.org/multipage/semantics.html)
- [MDN - セマンティクスとは](https://developer.mozilla.org/ja/docs/Glossary/Semantics)
