# JSON-LDとは何か — 構造化データの基礎

## JSON-LDとは

- JSON for Linking Data の略称
- Webページの内容を検索エンジンや機械が理解できる形式で記述するためのフォーマット
- JSON形式で構造化データを表現する仕様
- W3Cが標準化した仕様（2014年勧告、2020年に1.1へ更新）
- Googleが推奨する構造化データの記述方法

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "JSON-LDの基礎",
  "author": {
    "@type": "Person",
    "name": "山田太郎"
  }
}
</script>
```

- 参照: [W3C — JSON-LD 1.1](https://www.w3.org/TR/json-ld11/)

## なぜ構造化データが必要か

- 検索エンジンはHTMLを「テキスト」として読み取るが、その意味までは理解しにくい
- 構造化データを記述することで「これは記事である」「著者は誰か」「評価は何点か」を明確に伝えられる
- Googleの検索結果にリッチリザルト（リッチスニペット）として表示される可能性がある

### リッチリザルトの例

- レシピ：調理時間、カロリー、評価の星が検索結果に表示
- FAQ：質問と回答が検索結果に展開表示
- 商品：価格、在庫状況、レビュー数が検索結果に表示
- パンくずリスト：ページ階層が検索結果に表示
- イベント：日時、場所、チケット情報が検索結果に表示

- 参照: [Google — リッチリザルトの種類](https://developers.google.com/search/docs/appearance/structured-data/search-gallery)

## 構造化データの記述方式の比較

| 方式 | 記述場所 | 特徴 |
|---|---|---|
| JSON-LD | `<script>`タグ内 | HTMLと分離して記述。Googleが推奨 |
| Microdata | HTML属性 | HTMLタグに直接属性を追加。HTMLと密結合 |
| RDFa | HTML属性 | Microdataに似た属性ベースの方式 |

- **JSON-LDが推奨される理由**: HTMLの構造に依存しないため、テンプレートの変更に強い

```html
<!-- JSON-LD：HTMLと完全に分離 -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "ワイヤレスイヤホン"
}
</script>

<!-- Microdata：HTMLに直接埋め込み -->
<div itemscope itemtype="https://schema.org/Product">
  <span itemprop="name">ワイヤレスイヤホン</span>
</div>
```

- 参照: [Google — 構造化データの仕組み](https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data)

## JSON-LDの基本構文

### 必須キーワード

| キーワード | 意味 |
|---|---|
| `@context` | 語彙の定義元を指定（通常は`https://schema.org`） |
| `@type` | データの型を指定（`Article`, `Product`, `Person`など） |

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "サンプル株式会社",
  "url": "https://example.com"
}
```

### よく使うキーワード

| キーワード | 意味 |
|---|---|
| `@id` | エンティティの一意な識別子(URI) |
| `@graph` | 複数のエンティティをまとめて記述 |

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Organization",
      "@id": "https://example.com/#org",
      "name": "サンプル株式会社"
    },
    {
      "@type": "WebSite",
      "publisher": { "@id": "https://example.com/#org" }
    }
  ]
}
```

- 参照: [Schema.org](https://schema.org/)

## Schema.orgの主要な型

### Webサイト全般でよく使う型

| 型 | 用途 |
|---|---|
| `WebSite` | Webサイト自体の情報 |
| `WebPage` | 個別のページ情報 |
| `Organization` | 組織情報 |
| `Person` | 人物情報 |
| `BreadcrumbList` | パンくずリスト |
| `SiteNavigationElement` | サイトナビゲーション |

### コンテンツ系

| 型 | 用途 |
|---|---|
| `Article` | 記事 |
| `BlogPosting` | ブログ記事（Articleのサブタイプ） |
| `FAQPage` | FAQページ |
| `HowTo` | ハウツー・手順 |
| `Event` | イベント情報 |

### ECサイト・ビジネス系

| 型 | 用途 |
|---|---|
| `Product` | 商品情報 |
| `Offer` | 価格・在庫情報 |
| `Review` | レビュー |
| `AggregateRating` | 総合評価 |
| `LocalBusiness` | 店舗・拠点情報 |

## 実装パターン

### パターン1: 企業サイトのトップページ

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "サンプル株式会社",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png",
  "sameAs": [
    "https://twitter.com/example",
    "https://www.facebook.com/example"
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "telephone": "+81-3-1234-5678",
    "contactType": "customer service",
    "availableLanguage": "Japanese"
  }
}
</script>
```

### パターン2: ブログ記事

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "CSS Gridの基本的な使い方",
  "description": "CSS Gridの基本概念とレイアウト手法を解説します",
  "image": "https://example.com/images/css-grid.png",
  "datePublished": "2025-01-15",
  "dateModified": "2025-03-01",
  "author": {
    "@type": "Person",
    "name": "山田太郎",
    "url": "https://example.com/authors/yamada"
  },
  "publisher": {
    "@type": "Organization",
    "name": "サンプル Tech Blog",
    "logo": {
      "@type": "ImageObject",
      "url": "https://example.com/logo.png"
    }
  }
}
</script>
```

### パターン3: パンくずリスト

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "ホーム",
      "item": "https://example.com"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "ブログ",
      "item": "https://example.com/blog"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "CSS Gridの基本"
    }
  ]
}
</script>
```

### パターン4: FAQページ

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "送料はいくらですか？",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "全国一律500円です。5,000円以上のご購入で送料無料となります。"
      }
    },
    {
      "@type": "Question",
      "name": "返品は可能ですか？",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "商品到着から7日以内であれば返品可能です。"
      }
    }
  ]
}
</script>
```

### パターン5: 商品ページ

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "ワイヤレスノイズキャンセリングヘッドホン",
  "image": "https://example.com/images/headphone.png",
  "description": "高性能ノイズキャンセリング搭載のワイヤレスヘッドホン",
  "brand": {
    "@type": "Brand",
    "name": "SampleAudio"
  },
  "offers": {
    "@type": "Offer",
    "price": "29800",
    "priceCurrency": "JPY",
    "availability": "https://schema.org/InStock",
    "url": "https://example.com/products/headphone"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.5",
    "reviewCount": "128"
  }
}
</script>
```

- 参照: [Google — 構造化データの各タイプのドキュメント](https://developers.google.com/search/docs/appearance/structured-data)

## HTMLへの配置方法

- `<head>`内または`<body>`内のどちらに配置しても有効
- Googleは`<head>`内への配置を推奨
- 1ページに複数の`<script type="application/ld+json">`を配置可能
- `@graph`で1つのscriptタグにまとめることも可能

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>ページタイトル</title>

  <!-- JSON-LDはheadに配置 -->
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "WebPage",
    "name": "ページタイトル"
  }
  </script>
</head>
<body>
  ...
</body>
</html>
```

## Next.js / Reactでの実装

```jsx
// Next.js App Routerでの実装例
export default function ArticlePage({ article }) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'BlogPosting',
    headline: article.title,
    datePublished: article.publishedAt,
    author: {
      '@type': 'Person',
      name: article.author,
    },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <article>
        <h1>{article.title}</h1>
        <p>{article.body}</p>
      </article>
    </>
  );
}
```

## テスト・検証ツール

| ツール | 用途 |
|---|---|
| [リッチリザルトテスト](https://search.google.com/test/rich-results) | Googleのリッチリザルト対応状況の確認 |
| [Schema Markup Validator](https://validator.schema.org/) | Schema.org準拠のバリデーション |
| Google Search Console | 実際のインデックス状況とエラーの確認 |

- JSON-LDを記述したら必ずリッチリザルトテストで検証する
- Search Consoleの「拡張」セクションで構造化データのエラーを継続的に監視

- 参照: [Google — リッチリザルトテスト](https://search.google.com/test/rich-results)

## よくある間違い

### 1. ページの実際の内容と一致しないデータを記述

```json
// 悪い例：ページに表示されていない評価を構造化データに含める
{
  "@type": "Product",
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "5.0"
  }
}
```

- Googleのガイドラインでは、ページに表示されている情報のみを構造化データに含めることを求めている

### 2. @contextの指定漏れ

```json
// 悪い例：@contextがない
{
  "@type": "Article",
  "headline": "タイトル"
}

// 良い例
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "タイトル"
}
```

### 3. 必須プロパティの不足

- 各型にはGoogleが求める必須・推奨プロパティがある
- Articleなら`headline`、Productなら`name`が必須
- リッチリザルトテストで不足を確認する

## まとめ

- JSON-LDはWebページの内容を機械可読にするための構造化データ記述フォーマット
- `<script type="application/ld+json">`でHTMLと分離して記述
- Schema.orgの語彙を使って型（`@type`）とプロパティを定義
- 正しく実装するとGoogleの検索結果にリッチリザルトとして表示される可能性がある
- ページの実際の内容と一致するデータのみを記述する
- リッチリザルトテストで必ず検証する
