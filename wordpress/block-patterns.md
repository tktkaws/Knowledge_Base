# ブロックパターンの登録と活用

## 概要

- 複数のブロックを組み合わせた「テンプレート」を登録し、ワンクリックで挿入できる仕組み
- WordPress 5.5で導入、WordPress 6.0以降で大幅に機能強化
- コアパターン（WordPress標準）、テーマパターン、プラグインパターン、ユーザー作成パターンがある
- パターンはPHPまたはブロックエディタのUIから登録できる

### 参照元

- [WordPress Block Patterns](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-patterns/)
- [WordPress Pattern Directory](https://wordpress.org/patterns/)

---

## パターンの種類

### 同期パターン（Synced Patterns）

- 旧「再利用ブロック」。WordPress 6.3で名称変更
- 1か所を編集すると、使用しているすべての場所に反映される
- CTAバナー、免責事項、お問い合わせ情報など共通コンテンツに適する
- データベースの `wp_block` カスタム投稿タイプとして保存される

### 非同期パターン（Non-synced Patterns）

- 挿入時にブロックがコピーされ、以後は独立して編集できる
- レイアウトのテンプレートとして使う（ヒーローセクション、料金表など）
- 挿入後に内容を変えても他の場所には影響しない

### 使い分け

```
共通コンテンツを一括管理したい → 同期パターン
レイアウトの雛形として使いたい → 非同期パターン
```

### 参照元

- [WordPress Patterns – Synced and Unsynced](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-patterns/)

---

## エディタUIからパターンを作成する

### 同期パターンの作成

1. パターンにしたいブロックを選択（複数選択可）
2. ブロックツールバーの「…」→「パターンを作成」
3. パターン名を入力
4. 「同期」トグルをONにして保存

### 非同期パターンの作成

1. 同じ手順で「パターンを作成」を選択
2. 「同期」トグルをOFFにして保存

### パターンの管理

- 管理画面 →「外観」→「パターン」でパターン一覧を表示
- カテゴリごとに分類可能
- エクスポート（JSON）して他のサイトに持っていける

### 参照元

- [WordPress Documentation – Block Pattern](https://wordpress.org/documentation/article/block-pattern/)

---

## PHPでパターンを登録する（register_block_pattern）

### 基本構文

```php
<?php
function my_theme_register_patterns() {
    register_block_pattern(
        'my-theme/hero-section',  // パターン名（namespace/name）
        [
            'title'       => __('ヒーローセクション', 'my-theme'),
            'description' => __('ページ上部に配置するヒーローエリア', 'my-theme'),
            'categories'  => ['featured'],
            'keywords'    => ['ヒーロー', 'バナー', 'メインビジュアル'],
            'content'     => '<!-- wp:cover {"url":"https://example.com/image.jpg","dimRatio":50} -->
<div class="wp-block-cover"><span class="wp-block-cover__background has-background-dim"></span><img class="wp-block-cover__image-background" src="https://example.com/image.jpg" alt="" /><div class="wp-block-cover__inner-container">
<!-- wp:heading {"textAlign":"center","level":1} -->
<h1 class="wp-block-heading has-text-align-center">サイトタイトル</h1>
<!-- /wp:heading -->
<!-- wp:paragraph {"align":"center"} -->
<p class="has-text-align-center">サブタイトルのテキストがここに入ります</p>
<!-- /wp:paragraph -->
</div></div>
<!-- /wp:cover -->',
        ]
    );
}
add_action('init', 'my_theme_register_patterns');
```

### register_block_patternのパラメータ

| パラメータ | 型 | 説明 |
|-----------|------|------|
| `title` | string | パターン挿入パネルに表示される名前 |
| `description` | string | パターンの説明文 |
| `content` | string | ブロックマークアップ（HTMLコメント形式） |
| `categories` | array | パターンカテゴリのスラッグ配列 |
| `keywords` | array | 検索用キーワード |
| `blockTypes` | array | パターンが挿入可能なブロックタイプ |
| `inserter` | bool | インサーターに表示するか（デフォルト: true） |
| `viewportWidth` | int | プレビューの表示幅（px） |

### 参照元

- [WordPress Developer Resources – register_block_pattern](https://developer.wordpress.org/reference/functions/register_block_pattern/)

---

## パターンカテゴリの登録

### 独自カテゴリの追加

```php
<?php
function my_theme_register_pattern_categories() {
    register_block_pattern_category(
        'my-theme-layouts',
        [
            'label'       => __('My Theme レイアウト', 'my-theme'),
            'description' => __('テーマ独自のレイアウトパターン', 'my-theme'),
        ]
    );
}
add_action('init', 'my_theme_register_pattern_categories');
```

### コアで用意されているカテゴリ

| スラッグ | 表示名 |
|---------|--------|
| `featured` | 注目 |
| `text` | テキスト |
| `columns` | カラム |
| `header` | ヘッダー |
| `footer` | フッター |
| `gallery` | ギャラリー |
| `call-to-action` | CTA |
| `banner` | バナー |

### 参照元

- [WordPress Developer Resources – register_block_pattern_category](https://developer.wordpress.org/reference/functions/register_block_pattern_category/)

---

## ファイルベースのパターン登録（WordPress 6.0以降）

- テーマの `patterns/` ディレクトリにPHPファイルを置くだけで自動登録される
- `register_block_pattern()` を書く必要がない
- ファイル先頭のコメントヘッダーでメタデータを定義する

### ディレクトリ構造

```
my-theme/
├── patterns/
│   ├── hero-section.php
│   ├── pricing-table.php
│   └── faq-accordion.php
├── style.css
├── functions.php
└── ...
```

### パターンファイルの例（patterns/hero-section.php）

```php
<?php
/**
 * Title: ヒーローセクション
 * Slug: my-theme/hero-section
 * Categories: featured, banner
 * Keywords: ヒーロー, バナー, メインビジュアル
 * Viewport Width: 1200
 * Block Types: core/post-content
 * Inserter: yes
 */
?>

<!-- wp:cover {"dimRatio":50,"minHeight":400,"minHeightUnit":"px"} -->
<div class="wp-block-cover" style="min-height:400px">
    <span class="wp-block-cover__background has-background-dim"></span>
    <div class="wp-block-cover__inner-container">
        <!-- wp:heading {"textAlign":"center","level":1} -->
        <h1 class="wp-block-heading has-text-align-center">見出しテキスト</h1>
        <!-- /wp:heading -->

        <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
        <div class="wp-block-buttons">
            <!-- wp:button -->
            <div class="wp-block-button">
                <a class="wp-block-button__link wp-element-button">詳しく見る</a>
            </div>
            <!-- /wp:button -->
        </div>
        <!-- /wp:buttons -->
    </div>
</div>
<!-- /wp:cover -->
```

### コメントヘッダーで使えるフィールド

| フィールド | 説明 |
|-----------|------|
| `Title` | 必須。パターン名 |
| `Slug` | 必須。一意な識別子（`namespace/name` 形式） |
| `Categories` | カテゴリスラッグ（カンマ区切り） |
| `Keywords` | 検索用キーワード（カンマ区切り） |
| `Viewport Width` | プレビューの幅 |
| `Block Types` | 挿入可能なブロックタイプ |
| `Inserter` | `yes` / `no`（インサーターに表示するか） |
| `Post Types` | 使用可能な投稿タイプ |

### 参照元

- [WordPress Theme Handbook – Patterns](https://developer.wordpress.org/themes/features/block-patterns/)

---

## パターンでの動的コンテンツ

- ファイルベースのパターンはPHPファイルなので、動的な値を埋め込める

### 画像パスにテーマのURLを使う例

```php
<?php
/**
 * Title: ヒーローセクション（テーマ画像）
 * Slug: my-theme/hero-with-theme-image
 * Categories: featured
 */
$image_url = esc_url(get_theme_file_uri('assets/images/hero.jpg'));
?>

<!-- wp:cover {"url":"<?php echo $image_url; ?>","dimRatio":50} -->
<div class="wp-block-cover">
    <span class="wp-block-cover__background has-background-dim"></span>
    <img class="wp-block-cover__image-background" src="<?php echo $image_url; ?>" alt="" />
    <div class="wp-block-cover__inner-container">
        <!-- wp:heading {"textAlign":"center"} -->
        <h2 class="wp-block-heading has-text-align-center">ようこそ</h2>
        <!-- /wp:heading -->
    </div>
</div>
<!-- /wp:cover -->
```

### register_block_patternとファイルベースの比較

```php
// 良い例：ファイルベース（WordPress 6.0以降のテーマ開発）
// patterns/hero.php にファイルを置くだけで自動登録

// 悪い例：プラグインではないのにregister_block_patternで長いHTMLを書く
// → メンテナンスが困難、エスケープ処理も面倒
register_block_pattern('my-theme/hero', [
    'title'   => 'ヒーロー',
    'content' => '<!-- wp:cover ... 長大なHTML ... -->',
]);
```

### 参照元

- [WordPress Theme Handbook – Patterns](https://developer.wordpress.org/themes/features/block-patterns/)

---

## WordPress.orgパターンディレクトリ

- [wordpress.org/patterns](https://wordpress.org/patterns/) で公開されているパターンをコピーして使える
- エディタの挿入パネル →「パターン」タブ →「パターンを探す」からアクセス
- 気に入ったパターンをクリックするとブロックマークアップがコピーされる
- WordPress 6.0以降、テーマがパターンディレクトリのパターンをバンドルできる

### theme.jsonでリモートパターンを参照

```json
{
  "version": 3,
  "patterns": [
    "short-text-surrounded-by-round-images",
    "suspended-cafe-background-image"
  ]
}
```

### 参照元

- [WordPress Pattern Directory](https://wordpress.org/patterns/)

---

## コアパターンの無効化

### すべてのコアパターンを無効にする

```php
<?php
// コアのリモートパターン（パターンディレクトリ）を無効化
remove_theme_support('core-block-patterns');
```

### 特定のコアパターンだけ削除する

```php
<?php
function my_theme_remove_patterns() {
    unregister_block_pattern('core/query-standard-posts');
    unregister_block_pattern('core/query-medium-posts');
}
add_action('init', 'my_theme_remove_patterns');
```

### 参照元

- [WordPress Developer Resources – remove_theme_support](https://developer.wordpress.org/reference/functions/remove_theme_support/)

---

## パターン設計のベストプラクティス

### 命名規則

- パターン名は `namespace/descriptive-name` 形式にする
- テーマなら `my-theme/hero-section`、プラグインなら `my-plugin/pricing-table`

### コンテンツの書き方

- ダミーテキストは実際のユースケースに近い内容にする（「Lorem ipsum」は避ける）
- 画像はプレースホルダーURLまたはテーマ同梱の画像を使う
- ユーザーが編集しやすい構造にする（過度にネストしない）

### パフォーマンス

- パターンが多すぎるとインサーターの表示が遅くなる
- 使わないコアパターンは `remove_theme_support('core-block-patterns')` で無効化する
- 頻繁に使うパターンだけ登録する

```
良いパターン設計：
  ✔ 1つの目的に特化したパターン
  ✔ 汎用的に使えるレイアウト
  ✔ 明確なカテゴリ分類

悪いパターン設計：
  ✘ 1つのパターンに詰め込みすぎる
  ✘ ほぼ同じパターンを微妙な違いで大量登録
  ✘ ハードコードされた固有コンテンツ
```

### 参照元

- [WordPress Block Editor Handbook – Patterns](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-patterns/)
