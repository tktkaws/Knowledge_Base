# ACFブロック

## ACFブロックとは何か

### ブロックエディタ（Gutenberg）のおさらい

WordPressの投稿編集画面では、「ブロックエディタ」を使ってコンテンツを組み立てます。
テキスト・画像・見出しなど、それぞれが「ブロック」という単位で管理されています。

### ACFブロックとは

ACFブロックとは、**ACF PROの機能を使って自分で作ったオリジナルのブロック**のことです。
たとえば、こんなものをブロックとして作れます。

- スタッフ紹介カード（写真・名前・役職・一言コメント）
- バナーリンク（画像・テキスト・リンク先URL）
- お知らせボックス（種類・タイトル・本文）

### 通常のブロックとの違い

| | 標準ブロック | ACFブロック |
|---|---|---|
| 作るのに必要なもの | 不要（最初から使える） | ACF PRO ＋ テーマファイル |
| 入力UI | ブロック内で直接編集 | サイドバーにACFのフィールドが表示される |
| デザインの自由度 | テーマに依存 | PHPテンプレートで自由に組める |
| 向いている用途 | 汎用的なコンテンツ | サイト固有の繰り返し使うパーツ |

### ACFブロックを使うメリット

- **クライアントが迷わない**：入力項目が明確なので、誰でも同じ形で入力できます
- **デザインが崩れにくい**：自由記述を減らせるので、意図したレイアウトを保てます
- **コードで管理できる**：テンプレートファイルとして保存するため、Gitで管理できます

## ファイル構成

ACFブロックは、主に以下のファイルで構成されます。
`block.json` は使用せず、すべての設定を `functions.php` に記述します。

```text
themes/
└── your-theme/
    ├── functions.php            ← ブロックの登録と設定をすべてここに書く
    └── blocks/
        └── staff-card/
            └── staff-card.php  ← 表示用テンプレート
```

> `blocks/` フォルダの場所や命名はテーマによって異なる場合があります。
> 既存テーマの構成を確認してから作業してください。

## ステップ1：フィールドグループの設定

フィールドグループとは、ブロックに紐づける**入力項目のセット**です。
管理画面から設定します。

### 設定手順

1. WordPressの管理画面 → **「カスタムフィールド」→「フィールドグループ」→「新規追加」**
2. フィールドグループに名前をつける（例：スタッフ紹介ブロック）
3. 「フィールドを追加」でフィールドを追加する
4. 「表示ルール」でブロックに紐づける設定をする
5. 保存する

### 表示ルールの設定（重要）

フィールドグループを特定のACFブロックに紐づけるには、表示ルールを設定します。

```text
表示ルール
  ブロック ＝ [作成するブロックの名前]
```

> ブロックを先に登録しないと、この選択肢に出てきません。
> 先にブロックを登録してから表示ルールを設定するか、後から設定し直してください。

## ステップ2：ブロックを functions.php に登録する

`acf_register_block_type()` という関数を使って、ブロックの定義と設定をすべて `functions.php` に直接記述します。

### サンプル：スタッフ紹介ブロック

```php
add_action('acf/init', 'register_acf_blocks');
function register_acf_blocks() {

    if (!function_exists('acf_register_block_type')) return;

    acf_register_block_type([
        'name'            => 'staff-card',
        'title'           => 'スタッフ紹介カード',
        'description'     => 'スタッフの写真・名前・役職を表示するブロックです',
        'category'        => 'common',
        'icon'            => 'admin-users',
        'keywords'        => ['staff', 'スタッフ', '紹介'],
        'render_template' => get_template_directory() . '/blocks/staff-card/staff-card.php',
        'mode'            => 'preview',
        'supports'        => [
            'align' => false,
        ],
    ]);

}
```

### 各パラメータの説明

| パラメータ | 説明 |
|---|---|
| `name` | ブロックの識別名。英小文字・ハイフンのみ使用可。内部的に `acf/` が自動で付きます |
| `title` | エディタに表示されるブロック名 |
| `description` | ブロックの説明文。エディタのツールチップなどに表示されます |
| `category` | ブロックのカテゴリ（`common` / `formatting` / `layout` など） |
| `icon` | ブロックのアイコン（[Dashicons](https://developer.wordpress.org/resource/dashicons/) から選ぶ） |
| `keywords` | ブロック検索時のキーワード。配列で複数指定できます |
| `render_template` | 表示に使うPHPテンプレートの**絶対パス** |
| `mode` | エディタ上の表示モード。`preview`（プレビュー表示）か `edit`（フィールド表示） |
| `supports` | WordPress標準のブロック機能のON/OFF。`align` は整列ボタンの表示制御 |

### render_template のパスについて

`render_template` には**絶対パス**を指定します。
`get_template_directory()` はテーマフォルダの絶対パスを返してくれるため、これを使って結合します。

```php
// 正しい書き方（絶対パス）
'render_template' => get_template_directory() . '/blocks/staff-card/staff-card.php',

// 間違い（相対パスは動きません）
'render_template' => 'blocks/staff-card/staff-card.php',
```

> 子テーマを使っている場合は `get_template_directory()` ではなく `get_stylesheet_directory()` を使ってください。

### 複数のブロックを登録する場合

`acf_register_block_type()` を並べて書きます。

```php
add_action('acf/init', 'register_acf_blocks');
function register_acf_blocks() {

    if (!function_exists('acf_register_block_type')) return;

    // スタッフ紹介カード
    acf_register_block_type([
        'name'            => 'staff-card',
        'title'           => 'スタッフ紹介カード',
        'category'        => 'common',
        'icon'            => 'admin-users',
        'render_template' => get_template_directory() . '/blocks/staff-card/staff-card.php',
        'mode'            => 'preview',
    ]);

    // バナーリンク
    acf_register_block_type([
        'name'            => 'banner',
        'title'           => 'バナーリンク',
        'category'        => 'common',
        'icon'            => 'format-image',
        'render_template' => get_template_directory() . '/blocks/banner/banner.php',
        'mode'            => 'preview',
    ]);

    // お知らせボックス
    acf_register_block_type([
        'name'            => 'notice-box',
        'title'           => 'お知らせボックス',
        'category'        => 'common',
        'icon'            => 'info',
        'render_template' => get_template_directory() . '/blocks/notice-box/notice-box.php',
        'mode'            => 'preview',
    ]);
}
```

## ステップ3：表示テンプレート（PHP）を作る

ブロックの見た目を作るPHPファイルです。
`get_field()` でACFのフィールド値を取得して、HTMLに出力します。

### サンプル：スタッフ紹介カード（staff-card.php）

```php
<?php
// フィールドの値を取得する
$name     = get_field('name');       // テキスト：名前
$position = get_field('position');   // テキスト：役職
$photo    = get_field('photo');      // 画像：写真（配列で返ってくる）
$comment  = get_field('comment');    // テキストエリア：一言コメント
?>

<div class="staff-card">
  <?php if ($photo) : ?>
    <div class="staff-card__photo">
      <img src="<?php echo esc_url($photo['url']); ?>" alt="<?php echo esc_attr($photo['alt']); ?>">
    </div>
  <?php endif; ?>

  <div class="staff-card__body">
    <?php if ($name) : ?>
      <p class="staff-card__name"><?php echo esc_html($name); ?></p>
    <?php endif; ?>

    <?php if ($position) : ?>
      <p class="staff-card__position"><?php echo esc_html($position); ?></p>
    <?php endif; ?>

    <?php if ($comment) : ?>
      <p class="staff-card__comment"><?php echo esc_html($comment); ?></p>
    <?php endif; ?>
  </div>
</div>
```

### ポイント：出力時はエスケープ処理を忘れずに

ユーザーが入力した値をそのままHTMLに出力すると、XSS（クロスサイトスクリプティング）という脆弱性につながる可能性があります。
WordPressが用意している関数でエスケープ処理をしましょう。

| 関数 | 使いどころ |
|---|---|
| `esc_html()` | テキストを出力するとき |
| `esc_url()` | URLを出力するとき |
| `esc_attr()` | タグの属性値に出力するとき |
| `wp_kses_post()` | HTMLタグを含む可能性があるテキストのとき |

## 参考リンク

- [ACF公式ドキュメント（acf_register_block_type）](https://www.advancedcustomfields.com/resources/acf_register_block_type/)
- [Dashiconsアイコン一覧](https://developer.wordpress.org/resource/dashicons/)
- [WordPressエスケープ関数リファレンス](https://developer.wordpress.org/themes/theme-security/data-sanitization-escaping/)
