# HTML文書の基本構造 — DOCTYPE, head, bodyの役割

## HTML文書の全体像

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ページタイトル</title>
</head>
<body>
  <!-- ページの表示内容 -->
</body>
</html>
```

- すべてのHTMLファイルはこの基本構造に従う
- 各パートがそれぞれ異なる役割を持つ

> 参照: [MDN - HTML の基本](https://developer.mozilla.org/ja/docs/Learn/Getting_started_with_the_web/HTML_basics)

## DOCTYPE宣言

```html
<!DOCTYPE html>
```

- 文書がHTML5であることをブラウザに伝える宣言
- **必ずファイルの先頭に記述する**（前に空行や文字があってはならない）
- 省略すると「互換モード（Quirks Mode）」でレンダリングされる
- 互換モードではCSSのボックスモデルなどが標準と異なる挙動になる
- HTML5では `<!DOCTYPE html>` の1行だけでよい（以前のバージョンはもっと長かった）

### 互換モードの問題

```html
<!-- 悪い例: DOCTYPE省略 -->
<html>
<head>
  <title>テスト</title>
</head>
<body>
  <!-- 互換モードで描画される可能性がある -->
</body>
</html>
```

```html
<!-- 良い例: DOCTYPEを先頭に記述 -->
<!DOCTYPE html>
<html lang="ja">
<head>
  <title>テスト</title>
</head>
<body>
  <!-- 標準モードで描画される -->
</body>
</html>
```

> 参照: [MDN - DOCTYPE](https://developer.mozilla.org/ja/docs/Glossary/Doctype)

## html要素

```html
<html lang="ja">
  ...
</html>
```

- 文書のルート要素（すべての要素の親）
- `lang`属性でページの主要言語を指定する
- `lang`属性の役割:
  - スクリーンリーダーが正しい発音で読み上げる
  - 検索エンジンがコンテンツの言語を判別する
  - ブラウザの翻訳機能が動作する基準になる

```html
<!-- 悪い例: lang属性なし -->
<html>

<!-- 良い例: lang属性あり -->
<html lang="ja">

<!-- 英語ページの場合 -->
<html lang="en">
```

> 参照: [MDN - \<html\>](https://developer.mozilla.org/ja/docs/Web/HTML/Element/html)

## head要素

- ページの**メタ情報**を格納する領域
- ブラウザの表示領域には直接表示されない
- 検索エンジン、ブラウザ、SNSなどに向けた情報を記述する

### 必須のhead内要素

```html
<head>
  <!-- 文字エンコーディング: 必ず最初に記述 -->
  <meta charset="UTF-8">

  <!-- レスポンシブ対応のviewport設定 -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- ページタイトル: ブラウザタブ・検索結果に表示 -->
  <title>ページタイトル</title>
</head>
```

### よく使われるhead内要素

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ページタイトル</title>

  <!-- ページの説明文（検索結果のスニペットに使われる） -->
  <meta name="description" content="ページの説明文をここに書く">

  <!-- CSS読み込み -->
  <link rel="stylesheet" href="/css/style.css">

  <!-- ファビコン -->
  <link rel="icon" href="/favicon.ico">

  <!-- 外部フォント -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP&display=swap">

  <!-- JavaScript（defer付き） -->
  <script src="/js/main.js" defer></script>
</head>
```

### head内要素の記述順序のベストプラクティス

1. `<meta charset>` — 文字化け防止のため最初に置く
2. `<meta name="viewport">` — レイアウト計算に必要
3. `<title>` — ページの識別
4. `<meta>` タグ群 — description, OGPなど
5. `<link>` タグ群 — CSS, ファビコン, preconnectなど
6. `<script>` タグ群 — defer/asyncで読み込むJS

> 参照: [MDN - \<head\>: 文書メタデータ要素](https://developer.mozilla.org/ja/docs/Web/HTML/Element/head)

## body要素

- ページに**実際に表示されるコンテンツ**を格納する領域
- テキスト、画像、フォーム、リンクなどすべての可視コンテンツが入る
- 1つのHTML文書に `<body>` は1つだけ

```html
<body>
  <header>
    <nav>...</nav>
  </header>

  <main>
    <h1>ページの見出し</h1>
    <p>本文...</p>
  </main>

  <footer>
    <p>© 2026</p>
  </footer>
</body>
```

> 参照: [MDN - \<body\>](https://developer.mozilla.org/ja/docs/Web/HTML/Element/body)

## scriptの読み込み方法

### body末尾に置く（従来の方法）

```html
<body>
  <main>...</main>
  <!-- HTML解析が終わった後にJSが読み込まれる -->
  <script src="/js/main.js"></script>
</body>
```

### head内でdefer属性を使う（推奨）

```html
<head>
  <!-- HTML解析と並行してダウンロード、解析完了後に実行 -->
  <script src="/js/main.js" defer></script>
</head>
```

### defer と async の違い

| 属性 | ダウンロード | 実行タイミング | 実行順序 |
|------|-------------|---------------|---------|
| なし | HTML解析を止めて即座 | ダウンロード直後 | 記述順 |
| `defer` | HTML解析と並行 | HTML解析完了後 | 記述順を保証 |
| `async` | HTML解析と並行 | ダウンロード完了直後 | 順序の保証なし |

```html
<!-- 悪い例: headにそのまま置く（HTML解析をブロック） -->
<head>
  <script src="/js/heavy-library.js"></script>
</head>

<!-- 良い例: deferで並行ダウンロード -->
<head>
  <script src="/js/heavy-library.js" defer></script>
</head>
```

> 参照: [MDN - \<script\>](https://developer.mozilla.org/ja/docs/Web/HTML/Element/script)

## よくある間違い

### charsetをhead内の先頭に置かない

```html
<!-- 悪い例: titleの後にcharset -->
<head>
  <title>テスト</title>
  <meta charset="UTF-8">
</head>

<!-- 良い例: charsetを最初に -->
<head>
  <meta charset="UTF-8">
  <title>テスト</title>
</head>
```

- charsetは先頭1024バイト以内に記述する必要がある（HTML仕様）
- 後に書くとタイトルなどが文字化けする可能性がある

### viewport設定の省略

```html
<!-- 悪い例: viewport未設定 -->
<head>
  <meta charset="UTF-8">
  <title>テスト</title>
</head>

<!-- 良い例: viewport設定あり -->
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>テスト</title>
</head>
```

- viewportがないとモバイルでデスクトップ向けレイアウトが表示される

> 参照: [web.dev - Responsive web design basics](https://web.dev/articles/responsive-web-design-basics)

## まとめ

- `<!DOCTYPE html>` は必ず先頭に書き、標準モードで描画させる
- `<html lang="ja">` で言語を明示する
- `<head>` にはメタ情報を、`<body>` には表示コンテンツを格納する
- `<meta charset="UTF-8">` はhead内の最初に置く
- scriptは `defer` 属性をつけてhead内で読み込むのが現在の推奨
