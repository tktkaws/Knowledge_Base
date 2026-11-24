# WordPressでのJSON-LDとサイトマップの設定方法

## 概要

- WordPressではプラグインを使ってJSON-LD（構造化データ）とサイトマップを設定するのが一般的
- コードを書かずに管理画面から設定可能
- 手動でテーマに記述する方法もある

## サイトマップの設定

### WordPress標準のサイトマップ（5.5以降）

- WordPress 5.5からサイトマップ機能が標準搭載
- 有効化の操作は不要で、デフォルトで自動生成される
- `https://example.com/wp-sitemap.xml` でアクセス可能

#### 標準サイトマップの構成

```
wp-sitemap.xml（インデックス）
├── wp-sitemap-posts-post-1.xml    （投稿）
├── wp-sitemap-posts-page-1.xml    （固定ページ）
├── wp-sitemap-taxonomies-category-1.xml（カテゴリー）
├── wp-sitemap-taxonomies-post_tag-1.xml（タグ）
└── wp-sitemap-users-1.xml         （著者アーカイブ）
```

#### 標準サイトマップの制限

- `lastmod`（最終更新日）が出力されない
- 画像サイトマップに非対応
- 除外URLの細かい制御ができない
- 上記の制限が問題になる場合はプラグインを使用

- 参照: [WordPress Developer Resources — Sitemaps](https://developer.wordpress.org/apis/sitemaps/)

### プラグインでのサイトマップ設定

#### XML Sitemaps（旧 Google XML Sitemaps）

- 最も歴史のあるサイトマップ専用プラグイン
- シンプルで軽量

```
設定手順:
1. プラグイン「XML Sitemaps」をインストール・有効化
2. 設定 → XML-Sitemap を開く
3. サイトマップに含める投稿タイプを選択
4. 更新頻度と優先度を設定
5. 変更を保存
```

- サイトマップURL: `https://example.com/sitemap.xml`

#### Yoast SEO のサイトマップ機能

- SEOプラグインに含まれるサイトマップ機能
- JSON-LD出力機能もあるため、1つのプラグインで両方対応可能

```
設定手順:
1. プラグイン「Yoast SEO」をインストール・有効化
2. Yoast SEO → 設定 → サイトの機能 を開く
3. 「XMLサイトマップ」が有効になっていることを確認
4. 投稿タイプ・タクソノミーごとの表示/非表示を設定
```

- サイトマップURL: `https://example.com/sitemap_index.xml`
- WordPress標準のサイトマップは自動的に無効化される

- 参照: [Yoast — XML sitemaps](https://yoast.com/help/xml-sitemaps-in-the-wordpress-seo-plugin/)

### robots.txtへの反映

- WordPressは仮想的な`robots.txt`を自動生成
- サイトマップのURLは使用するプラグインに応じて自動で追記される
- 手動で変更したい場合はテーマの`functions.php`でフィルターを使用

```php
// functions.php — robots.txtにサイトマップURLを追加
add_filter('robots_txt', function ($output) {
    $output .= "Sitemap: https://example.com/sitemap.xml\n";
    return $output;
}, 10, 1);
```

## JSON-LD（構造化データ）の設定

### プラグインで設定する方法

#### Yoast SEO

- 有効化するだけでJSON-LDが自動出力される
- `Organization` / `Person`、`WebSite`、`BreadcrumbList`、`Article` などを自動生成
- 管理画面から組織名、ロゴ、ソーシャルアカウントを設定可能

```
設定手順:
1. Yoast SEO → 設定 → サイトの基本 を開く
2. サイト名、組織/個人の選択、ロゴを設定
3. ソーシャルアカウントのURLを入力
4. 各投稿・ページの編集画面でSEO設定を個別調整
```

##### Yoast SEOが自動出力するJSON-LDの例

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "WebSite",
      "@id": "https://example.com/#website",
      "url": "https://example.com/",
      "name": "サンプルサイト",
      "publisher": { "@id": "https://example.com/#organization" }
    },
    {
      "@type": "Organization",
      "@id": "https://example.com/#organization",
      "name": "サンプル株式会社",
      "url": "https://example.com/",
      "logo": {
        "@type": "ImageObject",
        "url": "https://example.com/logo.png"
      }
    },
    {
      "@type": "Article",
      "headline": "記事タイトル",
      "datePublished": "2025-03-01",
      "dateModified": "2025-03-05",
      "author": {
        "@type": "Person",
        "name": "山田太郎"
      }
    }
  ]
}
```

- `@graph`を使って複数のエンティティを1つのスクリプトタグにまとめて出力
- `@id`による相互参照でエンティティ間の関係を表現

- 参照: [Yoast — Schema.org structured data](https://developer.yoast.com/features/schema/overview/)

#### Rank Math

- Yoast SEOの代替として人気のSEOプラグイン
- JSON-LDのスキーマタイプを投稿ごとに選択可能（Article, Product, Recipe, FAQなど）
- 無料版でも多くのスキーマタイプに対応

```
設定手順:
1. プラグイン「Rank Math」をインストール・有効化
2. セットアップウィザードで組織情報を入力
3. Rank Math → タイトルとメタ → 投稿タイプ でデフォルトのスキーマタイプを設定
4. 各投稿の編集画面 → Rank Math → Schema タブで個別設定
```

##### 投稿ごとのスキーマ設定例

```
投稿編集画面 → Rank Math → Schema:
  - Schema Type: Article / BlogPosting / NewsArticle
  - FAQ Schema: 質問と回答を追加（FAQPage型を自動生成）
  - HowTo Schema: 手順を追加（HowTo型を自動生成）
```

- 参照: [Rank Math — Schema (Structured Data)](https://rankmath.com/kb/rich-snippets/)

### テーマに手動で記述する方法

- プラグインを使わずにJSON-LDをテーマテンプレートに直接記述
- 出力内容を完全に制御したい場合に有効

#### functions.phpにフックで追加

```php
// functions.php
add_action('wp_head', function () {
    if (is_single()) {
        global $post;
        $schema = [
            '@context'      => 'https://schema.org',
            '@type'         => 'BlogPosting',
            'headline'      => get_the_title($post),
            'datePublished' => get_the_date('c', $post),
            'dateModified'  => get_the_modified_date('c', $post),
            'author'        => [
                '@type' => 'Person',
                'name'  => get_the_author_meta('display_name', $post->post_author),
            ],
            'publisher'     => [
                '@type' => 'Organization',
                'name'  => get_bloginfo('name'),
                'logo'  => [
                    '@type' => 'ImageObject',
                    'url'   => 'https://example.com/logo.png',
                ],
            ],
        ];

        // アイキャッチ画像がある場合
        if (has_post_thumbnail($post)) {
            $schema['image'] = get_the_post_thumbnail_url($post, 'full');
        }

        echo '<script type="application/ld+json">' . wp_json_encode($schema, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES) . '</script>' . "\n";
    }
});
```

#### パンくずリストのJSON-LD出力

```php
// functions.php
add_action('wp_head', function () {
    if (is_single() || is_page()) {
        $items = [];
        $position = 1;

        // ホーム
        $items[] = [
            '@type'    => 'ListItem',
            'position' => $position++,
            'name'     => 'ホーム',
            'item'     => home_url('/'),
        ];

        // カテゴリー（投稿の場合）
        if (is_single()) {
            $categories = get_the_category();
            if (!empty($categories)) {
                $cat = $categories[0];
                $items[] = [
                    '@type'    => 'ListItem',
                    'position' => $position++,
                    'name'     => $cat->name,
                    'item'     => get_category_link($cat->term_id),
                ];
            }
        }

        // 現在のページ
        $items[] = [
            '@type'    => 'ListItem',
            'position' => $position,
            'name'     => get_the_title(),
        ];

        $schema = [
            '@context'        => 'https://schema.org',
            '@type'           => 'BreadcrumbList',
            'itemListElement' => $items,
        ];

        echo '<script type="application/ld+json">' . wp_json_encode($schema, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES) . '</script>' . "\n";
    }
});
```

#### 注意点

- `wp_json_encode()`を使う（`json_encode()`ではなく）
- `JSON_UNESCAPED_UNICODE`で日本語のエスケープを防止
- `JSON_UNESCAPED_SLASHES`でURLのスラッシュを保持
- XSS対策として、ユーザー入力を含む場合は`wp_kses()`でサニタイズ

## プラグイン比較

| 機能 | Yoast SEO | Rank Math | 手動実装 |
|---|---|---|---|
| サイトマップ自動生成 | 対応 | 対応 | 別途必要 |
| JSON-LD自動出力 | 対応 | 対応 | 自分で実装 |
| 投稿ごとのスキーマ変更 | 有料版のみ | 無料版で対応 | 自分で実装 |
| FAQスキーマ | 有料版のみ | 無料版で対応 | 自分で実装 |
| パフォーマンス | 軽量 | やや重い | 最軽量 |
| カスタマイズ性 | フィルターフック | フィルターフック | 完全自由 |
| 学習コスト | 低い | 低い | 高い |

### 選び方の目安

- **Yoast SEO**: シンプルに始めたい場合。実績と安定性を重視
- **Rank Math**: 無料で多機能に使いたい場合。FAQやHowToスキーマを手軽に追加
- **手動実装**: 出力を完全に制御したい場合。不要なコードを排除してパフォーマンス重視

## 検証とトラブルシューティング

### 設定後に確認すべきこと

1. **サイトマップの確認**
   - `https://example.com/sitemap.xml`（または`sitemap_index.xml`）にアクセス
   - XMLが正しく表示されることを確認
   - 含めるべきURLが存在し、除外すべきURLが含まれていないこと

2. **JSON-LDの確認**
   - ページのソースコードで`<script type="application/ld+json">`を検索
   - [リッチリザルトテスト](https://search.google.com/test/rich-results)でURLを入力して検証
   - エラーや警告がないことを確認

3. **Google Search Consoleへの送信**
   - サイトマップURLを送信
   - 「拡張」セクションで構造化データのエラーを確認

### よくあるトラブル

#### サイトマップが404になる

```
原因: パーマリンク設定の問題
対処: 設定 → パーマリンク → 変更を保存（内容を変えなくてもOK）
      → .htaccessのリライトルールが再生成される
```

#### JSON-LDが重複出力される

```
原因: 複数のSEOプラグインが同時に有効になっている
対処: SEOプラグインは1つだけ有効にする
      テーマに手動で記述したJSON-LDとプラグイン出力が重複していないか確認
```

#### リッチリザルトテストでエラーが出る

```
原因: 必須プロパティの不足
対処: エラーメッセージに記載されたプロパティを設定
      例: Article型で "image" が不足 → アイキャッチ画像を設定
```

- 参照: [Google — リッチリザルトテスト](https://search.google.com/test/rich-results)

## まとめ

- WordPressのサイトマップは5.5以降で標準搭載。詳細な制御にはプラグインを使用
- JSON-LDはYoast SEOやRank Mathで自動出力するのが最も手軽
- プラグインを使う場合はSEOプラグインを1つに絞り、重複出力を避ける
- 手動実装は`wp_head`フックと`wp_json_encode()`で`<head>`に出力
- 設定後はリッチリザルトテストとSearch Consoleで必ず検証
