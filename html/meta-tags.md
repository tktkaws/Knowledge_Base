# メタタグの種類と使い分け — charset, viewport, og:tags

## メタタグとは

- `<meta>` 要素でHTML文書のメタ情報を定義する仕組み
- ブラウザ、検索エンジン、SNSなどに向けた情報を提供する
- `<head>` 内に記述する
- ページの表示内容には直接影響しない（間接的に影響するものはある）

> 参照: [MDN - \<meta\>](https://developer.mozilla.org/ja/docs/Web/HTML/Element/meta)

## 必須のメタタグ

### charset（文字エンコーディング）

```html
<meta charset="UTF-8">
```

- 文書の文字エンコーディングを宣言する
- **UTF-8がほぼ唯一の選択肢**（HTML5仕様で推奨）
- head内の最初の要素として記述する（先頭1024バイト以内に必要）
- 省略や誤指定すると文字化けの原因になる

```html
<!-- 悪い例: 古い書き方 -->
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">

<!-- 良い例: HTML5の書き方 -->
<meta charset="UTF-8">
```

> 参照: [MDN - charset](https://developer.mozilla.org/ja/docs/Web/HTML/Element/meta#charset)

### viewport（表示領域の制御）

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

- モバイルブラウザでの表示領域を制御する
- レスポンシブデザインに必須
- `width=device-width` — ビューポート幅をデバイス幅に合わせる
- `initial-scale=1.0` — 初期ズーム倍率を等倍にする

```html
<!-- 悪い例: ユーザーのズームを無効化 -->
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

<!-- 良い例: ズームを許可（アクセシビリティに配慮） -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

- `user-scalable=no` や `maximum-scale=1.0` はアクセシビリティ上の問題がある
- 弱視のユーザーがズームできなくなる

> 参照: [MDN - viewport meta tag](https://developer.mozilla.org/ja/docs/Web/HTML/Viewport_meta_tag)

## SEO関連のメタタグ

### description（ページの説明文）

```html
<meta name="description" content="このページはCSSグリッドレイアウトの基本的な使い方を解説します。">
```

- 検索結果のスニペット（説明文）として表示される可能性がある
- 120〜160文字程度が推奨
- ページごとにユニークな内容にする

```html
<!-- 悪い例: 全ページ同じ説明文 -->
<meta name="description" content="私のウェブサイトです">

<!-- 良い例: ページ固有の説明文 -->
<meta name="description" content="CSS Gridの基本プロパティ（grid-template-columns, grid-gap等）をコード例付きで解説します。">
```

### robots（クロール制御）

```html
<!-- デフォルト（省略してもこの挙動） -->
<meta name="robots" content="index, follow">

<!-- 検索結果に表示しない -->
<meta name="robots" content="noindex">

<!-- リンクをたどらない -->
<meta name="robots" content="nofollow">

<!-- 検索結果に表示せず、リンクもたどらない -->
<meta name="robots" content="noindex, nofollow">
```

| 値 | 意味 |
|----|------|
| `index` | 検索インデックスに登録する（デフォルト） |
| `noindex` | 検索インデックスに登録しない |
| `follow` | ページ内のリンクをたどる（デフォルト） |
| `nofollow` | ページ内のリンクをたどらない |

> 参照: [Google - robots メタタグ](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag)

## OGP（Open Graph Protocol）

- SNS（X、Facebook、LINEなど）でURLをシェアしたときの表示を制御する
- `<meta property="og:xxx">` の形式で記述する
- `name`ではなく`property`属性を使う点に注意

### 基本的なOGPタグ

```html
<meta property="og:title" content="CSSグリッドの基本">
<meta property="og:description" content="CSS Gridの基本的な使い方を解説します。">
<meta property="og:type" content="article">
<meta property="og:url" content="https://example.com/css-grid">
<meta property="og:image" content="https://example.com/images/css-grid-og.png">
<meta property="og:site_name" content="フロントエンド知識ベース">
<meta property="og:locale" content="ja_JP">
```

| プロパティ | 必須 | 説明 |
|-----------|------|------|
| `og:title` | ○ | コンテンツのタイトル |
| `og:type` | ○ | コンテンツの種類（`website`, `article`など） |
| `og:url` | ○ | コンテンツの正規URL |
| `og:image` | ○ | サムネイル画像のURL |
| `og:description` | - | コンテンツの説明文 |
| `og:site_name` | - | サイト名 |
| `og:locale` | - | ロケール（`ja_JP`など） |

### og:typeの主な値

| 値 | 用途 |
|----|------|
| `website` | トップページ、一般的なページ |
| `article` | ブログ記事、ニュース記事 |
| `profile` | ユーザープロフィールページ |

> 参照: [Open Graph Protocol 公式](https://ogp.me/)

## X（旧Twitter）カード

```html
<!-- カードタイプの指定 -->
<meta name="twitter:card" content="summary_large_image">

<!-- 以下はOGPタグがあれば省略可能だが、明示推奨 -->
<meta name="twitter:title" content="CSSグリッドの基本">
<meta name="twitter:description" content="CSS Gridの基本的な使い方を解説します。">
<meta name="twitter:image" content="https://example.com/images/css-grid-og.png">
```

| カードタイプ | 表示 |
|-------------|------|
| `summary` | 小さいサムネイル付きカード |
| `summary_large_image` | 大きい画像付きカード |

- OGPタグが設定されていれば、Xは自動的にそれをフォールバックとして使う
- `twitter:card` だけは必ず指定する（フォールバックがないため）

> 参照: [X Cards Documentation](https://developer.x.com/en/docs/x-for-websites/cards/overview/abouts-cards)

## その他の実用的なメタタグ

### テーマカラー

```html
<!-- ブラウザのアドレスバーやタブの色を制御（モバイル） -->
<meta name="theme-color" content="#3b82f6">

<!-- ダークモード対応 -->
<meta name="theme-color" content="#3b82f6" media="(prefers-color-scheme: light)">
<meta name="theme-color" content="#1e3a5f" media="(prefers-color-scheme: dark)">
```

### カラースキームの宣言

```html
<!-- ダークモード対応をブラウザに伝える -->
<meta name="color-scheme" content="light dark">
```

### http-equiv

```html
<!-- 5秒後に別ページへリダイレクト -->
<meta http-equiv="refresh" content="5;url=https://example.com/new-page">

<!-- CSPの設定（HTTPヘッダーでの設定が推奨） -->
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">
```

- `http-equiv="refresh"` はアクセシビリティの問題があるため、サーバーサイドのリダイレクト（301/302）を優先すべき

> 参照: [MDN - http-equiv](https://developer.mozilla.org/ja/docs/Web/HTML/Element/meta#http-equiv)

## 実践的なテンプレート

```html
<head>
  <!-- 基本 -->
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ページタイトル | サイト名</title>
  <meta name="description" content="ページの説明文（120〜160文字）">
  <meta name="theme-color" content="#3b82f6">

  <!-- OGP -->
  <meta property="og:title" content="ページタイトル">
  <meta property="og:description" content="ページの説明文">
  <meta property="og:type" content="article">
  <meta property="og:url" content="https://example.com/page">
  <meta property="og:image" content="https://example.com/images/og.png">
  <meta property="og:site_name" content="サイト名">
  <meta property="og:locale" content="ja_JP">

  <!-- X Card -->
  <meta name="twitter:card" content="summary_large_image">

  <!-- ファビコン -->
  <link rel="icon" href="/favicon.ico">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">

  <!-- CSS -->
  <link rel="stylesheet" href="/css/style.css">
</head>
```

> 参照: [web.dev - HTML metadata](https://web.dev/learn/html/metadata)

## まとめ

- `charset` と `viewport` は全ページに必須
- `description` はSEOに影響するのでページごとにユニークに設定する
- OGPタグを設定するとSNSシェア時の見栄えを制御できる
- `user-scalable=no` はアクセシビリティに問題があるため避ける
- `twitter:card` はOGPのフォールバックがないため明示的に指定する
