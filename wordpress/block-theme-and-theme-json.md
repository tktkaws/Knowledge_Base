# ブロックテーマとtheme.json

## 概要

- WordPress 5.9で正式導入されたテーマの新しい形式
- PHPテンプレート（`index.php`、`single.php` 等）の代わりにHTMLテンプレートを使う
- サイト全体をブロックエディタで編集できる（Full Site Editing = FSE）
- `theme.json` でテーマのデザイン設定を一元管理する

### 参照元

- [WordPress Block Theme Handbook](https://developer.wordpress.org/themes/block-themes/)
- [WordPress Full Site Editing](https://developer.wordpress.org/block-editor/getting-started/full-site-editing/)

---

## クラシックテーマとブロックテーマの違い

| 項目 | クラシックテーマ | ブロックテーマ |
|------|-----------------|---------------|
| テンプレートファイル | PHP（`index.php`、`single.php` 等） | HTML（`templates/index.html` 等） |
| テンプレートパーツ | `get_header()` / `get_footer()` | `templates/parts/header.html` 等 |
| レイアウト構築 | PHPでループとテンプレートタグを記述 | ブロックマークアップで構成 |
| カスタマイズ | カスタマイザー | サイトエディタ |
| デザイン設定 | CSS + `functions.php` | `theme.json` |
| ウィジェット | ウィジェットエリア | ブロックで直接配置 |
| メニュー | `wp_nav_menu()` | ナビゲーションブロック |

### 参照元

- [WordPress Theme Handbook – Block Themes vs Classic Themes](https://developer.wordpress.org/themes/block-themes/)

---

## ブロックテーマの最小構成

### 必須ファイル

```
my-block-theme/
├── style.css            # テーマのメタデータ（必須）
├── theme.json           # デザイン設定（実質必須）
└── templates/
    └── index.html       # フォールバックテンプレート（必須）
```

### style.css（テーマヘッダーのみ）

```css
/*
Theme Name: My Block Theme
Theme URI: https://example.com
Author: Your Name
Description: ブロックテーマのサンプル
Version: 1.0.0
Requires at least: 6.0
Tested up to: 6.5
Requires PHP: 7.4
License: GNU General Public License v2 or later
Text Domain: my-block-theme
*/
```

### templates/index.html

```html
<!-- wp:template-part {"slug":"header","area":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
    <!-- wp:query -->
    <div class="wp-block-query">
        <!-- wp:post-template -->
            <!-- wp:post-title {"isLink":true} /-->
            <!-- wp:post-date /-->
            <!-- wp:post-excerpt /-->
        <!-- /wp:post-template -->
        <!-- wp:query-pagination -->
        <div class="wp-block-query-pagination">
            <!-- wp:query-pagination-previous /-->
            <!-- wp:query-pagination-numbers /-->
            <!-- wp:query-pagination-next /-->
        </div>
        <!-- /wp:query-pagination -->
    </div>
    <!-- /wp:query -->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","area":"footer"} /-->
```

### 参照元

- [WordPress Theme Handbook – Block Theme Structure](https://developer.wordpress.org/themes/block-themes/block-theme-setup/)

---

## テンプレート階層（ブロックテーマ版）

- クラシックテーマと同じ階層構造だが、PHPではなくHTMLファイルを使う
- `templates/` ディレクトリに配置する

### 主要なテンプレート

| ファイル名 | 用途 |
|-----------|------|
| `index.html` | フォールバック（必須） |
| `front-page.html` | フロントページ |
| `home.html` | ブログ一覧ページ |
| `single.html` | 個別投稿 |
| `page.html` | 固定ページ |
| `archive.html` | アーカイブ一覧 |
| `category.html` | カテゴリアーカイブ |
| `search.html` | 検索結果 |
| `404.html` | 404エラー |
| `singular.html` | 投稿・固定ページ共通 |

### テンプレートの解決順序（例：カテゴリページ）

```
category-{slug}.html
  → category-{id}.html
    → category.html
      → archive.html
        → index.html
```

### 参照元

- [WordPress Template Hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/)

---

## テンプレートパーツ

- ヘッダー、フッター、サイドバーなど共通部分を分離して管理する仕組み
- `parts/` ディレクトリに配置する
- テンプレートから `<!-- wp:template-part -->` ブロックで参照する

### ディレクトリ構造

```
my-block-theme/
├── templates/
│   ├── index.html
│   ├── single.html
│   └── page.html
└── parts/
    ├── header.html
    ├── footer.html
    └── sidebar.html
```

### parts/header.html の例

```html
<!-- wp:group {"tagName":"header","layout":{"type":"constrained"}} -->
<header class="wp-block-group">
    <!-- wp:group {"layout":{"type":"flex","justifyContent":"space-between"}} -->
    <div class="wp-block-group">
        <!-- wp:site-title /-->
        <!-- wp:navigation /-->
    </div>
    <!-- /wp:group -->
</header>
<!-- /wp:group -->
```

### テンプレートからの参照

```html
<!-- 良い例：areaを指定してテンプレートパーツを参照 -->
<!-- wp:template-part {"slug":"header","area":"header"} /-->

<!-- 良い例：tagNameでセマンティックなHTML要素を指定 -->
<!-- wp:template-part {"slug":"footer","area":"footer","tagName":"footer"} /-->
```

### area属性

| area | 説明 |
|------|------|
| `header` | サイトのヘッダー領域 |
| `footer` | サイトのフッター領域 |
| `uncategorized` | その他（デフォルト） |

### 参照元

- [WordPress Theme Handbook – Template Parts](https://developer.wordpress.org/themes/block-themes/templates-and-template-parts/)

---

## サイトエディタ

- ブロックテーマで使えるサイト全体の編集画面
- 管理画面 →「外観」→「エディター」からアクセス
- テンプレート、テンプレートパーツ、スタイルをGUIで編集できる

### サイトエディタでできること

- テンプレートの編集・新規作成
- テンプレートパーツ（ヘッダー・フッター等）の編集
- グローバルスタイル（色・タイポグラフィ・余白）の変更
- ナビゲーションメニューの編集
- パターンの管理

### サイトエディタでの変更の保存先

- サイトエディタでの変更はデータベースに保存される
- テーマファイル（`templates/` や `parts/`）は変更されない
- データベースの変更がテーマファイルより優先される
- 「カスタマイズをクリア」でテーマファイルの状態にリセットできる

```
テーマファイル（templates/index.html）
  ↓ 基本のテンプレート
データベース（wp_template投稿タイプ）
  ↓ サイトエディタでの変更が上書き
最終的な表示
```

### 参照元

- [WordPress Site Editor](https://developer.wordpress.org/block-editor/getting-started/full-site-editing/)

---

## theme.jsonの役割

- テーマのデザイン設定を一元管理するJSONファイル
- カラーパレット、フォントサイズ、余白、レイアウト幅などを定義
- ブロックごとのデフォルトスタイルを設定できる
- エディタのUIオプション（どの設定をユーザーに見せるか）を制御する

### theme.jsonの最小構成

```json
{
  "$schema": "https://schemas.wp.org/wp/6.5/theme.json",
  "version": 3
}
```

### theme.jsonのトップレベル構造

```json
{
  "$schema": "https://schemas.wp.org/wp/6.5/theme.json",
  "version": 3,
  "settings": {},
  "styles": {},
  "templateParts": [],
  "customTemplates": [],
  "patterns": []
}
```

| プロパティ | 説明 |
|-----------|------|
| `version` | theme.jsonのスキーマバージョン（現在は3） |
| `settings` | エディタで使える色・フォント・余白などの選択肢と有効/無効の制御 |
| `styles` | グローバルなCSS設定（色・タイポグラフィ・余白の初期値） |
| `templateParts` | テンプレートパーツの定義（slug、area等） |
| `customTemplates` | カスタムテンプレートの定義 |
| `patterns` | パターンディレクトリから取得するパターンのスラッグ |

### 参照元

- [WordPress Theme JSON Reference](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/)

---

## functions.phpの最小化

- ブロックテーマでは `functions.php` の役割が大幅に減る
- `add_theme_support()` の多くが `theme.json` に移行済み

### クラシックテーマでは必要だったがブロックテーマでは不要な記述

```php
<?php
// ブロックテーマでは不要（theme.jsonが自動的に有効化する）

// add_theme_support('post-thumbnails');        → theme.jsonで自動有効
// add_theme_support('editor-color-palette', $colors); → theme.json settings.color.palette
// add_theme_support('editor-font-sizes', $sizes);     → theme.json settings.typography.fontSizes
// add_theme_support('custom-spacing');                 → theme.json settings.spacing
// add_theme_support('align-wide');                     → theme.json settings.layout
// add_theme_support('responsive-embeds');              → theme.jsonで自動有効
```

### ブロックテーマのfunctions.phpに残る主な役割

```php
<?php
// テキストドメインの読み込み（翻訳対応）
function my_theme_setup() {
    load_theme_textdomain('my-block-theme', get_template_directory() . '/languages');
}
add_action('after_setup_theme', 'my_theme_setup');

// パターンカテゴリの登録
function my_theme_register_pattern_categories() {
    register_block_pattern_category('my-theme-layouts', [
        'label' => __('レイアウト', 'my-block-theme'),
    ]);
}
add_action('init', 'my_theme_register_pattern_categories');

// 追加のスタイルシート読み込み（Webフォントなど）
function my_theme_enqueue_styles() {
    wp_enqueue_style(
        'my-theme-google-fonts',
        'https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700&display=swap',
        [],
        null
    );
}
add_action('wp_enqueue_scripts', 'my_theme_enqueue_styles');
```

### 参照元

- [WordPress Theme Handbook – Block Theme Setup](https://developer.wordpress.org/themes/block-themes/block-theme-setup/)

---

## ブロックテーマの完成例（ディレクトリ構成）

```
my-block-theme/
├── style.css                  # テーマヘッダー
├── theme.json                 # デザイン設定
├── functions.php              # 最小限のPHP
├── templates/                 # ページテンプレート
│   ├── index.html
│   ├── front-page.html
│   ├── single.html
│   ├── page.html
│   ├── archive.html
│   ├── search.html
│   └── 404.html
├── parts/                     # テンプレートパーツ
│   ├── header.html
│   ├── footer.html
│   └── sidebar.html
├── patterns/                  # パターン
│   ├── hero-section.php
│   └── cta-banner.php
├── assets/                    # 静的ファイル
│   ├── images/
│   └── fonts/
└── languages/                 # 翻訳ファイル
```

### 参照元

- [WordPress Theme Handbook – Block Theme Structure](https://developer.wordpress.org/themes/block-themes/block-theme-setup/)

---

## 公式ブロックテーマ（Twenty Twenty-Fourなど）

- WordPress同梱のデフォルトテーマがブロックテーマの参考になる
- Twenty Twenty-Two（2022年）が初のデフォルトブロックテーマ
- Twenty Twenty-Three、Twenty Twenty-Four、Twenty Twenty-Fiveと毎年進化

### 学習リソースとして活用する

```bash
# Twenty Twenty-Fiveのソースコードを確認する
wp-content/themes/twentytwentyfive/

# 特に参考になるファイル
twentytwentyfive/theme.json        # デザイン設定の実例
twentytwentyfive/templates/        # テンプレートの書き方
twentytwentyfive/parts/            # テンプレートパーツの分割方法
twentytwentyfive/patterns/         # パターンの実装例
```

### 参照元

- [Twenty Twenty-Five – WordPress Theme](https://wordpress.org/themes/twentytwentyfive/)
- [GitHub – WordPress/twentytwentyfive](https://github.com/WordPress/twentytwentyfive)
