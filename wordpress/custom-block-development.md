docs(wordpress):コアブロックの種類と使い分け# カスタムブロックの作り方（@wordpress/scripts）

## 概要

- コアブロックで対応できない独自のUIやコンテンツ構造が必要なとき、カスタムブロックを作る
- `@wordpress/scripts` はWordPress公式のビルドツールパッケージ
- 内部的にwebpackを使用し、JSX・ESNext・SCSSのトランスパイル・バンドルを行う
- `@wordpress/create-block` コマンドで雛形を生成できる

### 参照元

- [WordPress Block Editor Handbook – Block Development](https://developer.wordpress.org/block-editor/getting-started/devenv/)
- [@wordpress/scripts – npm](https://www.npmjs.com/package/@wordpress/scripts)

---

## 開発環境の準備

### 必要なツール

- Node.js（LTS推奨、v18以上）
- npm（Node.jsに同梱）
- WordPressのローカル開発環境（wp-env、Local、Docker等）

### wp-envによるローカル環境の起動

```bash
# wp-envをグローバルインストール
npm install -g @wordpress/env

# プラグインディレクトリに移動して起動
cd wp-content/plugins/my-block-plugin
npx wp-env start

# → http://localhost:8888 でWordPressが起動する
# → 管理画面: http://localhost:8888/wp-admin（ID: admin / PW: password）
```

### 参照元

- [@wordpress/env – npm](https://www.npmjs.com/package/@wordpress/env)
- [WordPress Block Development Environment](https://developer.wordpress.org/block-editor/getting-started/devenv/)

---

## @wordpress/create-blockでプロジェクトを作成

### 基本コマンド

```bash
# プラグインディレクトリで実行
cd wp-content/plugins

# 雛形を生成（プラグインごと作成される）
npx @wordpress/create-block my-custom-block

# 生成されたディレクトリに移動
cd my-custom-block

# 開発サーバーを起動（ファイル変更の監視 + 自動ビルド）
npm start
```

### 生成されるディレクトリ構造

```
my-custom-block/
├── build/                  # ビルド後のファイル（自動生成）
├── src/
│   ├── block.json          # ブロックのメタデータ定義
│   ├── edit.js             # エディタ側の表示（編集画面）
│   ├── save.js             # フロント側の表示（保存時のHTML出力）
│   ├── index.js            # ブロック登録のエントリーポイント
│   ├── editor.scss         # エディタ専用のスタイル
│   └── style.scss          # エディタ・フロント共通のスタイル
├── my-custom-block.php     # プラグインのメインPHPファイル
├── package.json
└── readme.txt
```

### 参照元

- [@wordpress/create-block – npm](https://www.npmjs.com/package/@wordpress/create-block)

---

## block.json — ブロックのメタデータ定義

- ブロックの名前、カテゴリ、属性、スクリプト、スタイルなどを定義するファイル
- WordPress 5.8以降、ブロック登録の標準的な方法
- PHPとJavaScriptの両方からこのファイルを参照してブロックを登録する

### 基本構造

```json
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "my-plugin/my-custom-block",
  "version": "1.0.0",
  "title": "カスタムブロック",
  "category": "design",
  "icon": "smiley",
  "description": "カスタムブロックの説明文",
  "supports": {
    "html": false,
    "color": {
      "background": true,
      "text": true
    },
    "spacing": {
      "margin": true,
      "padding": true
    }
  },
  "attributes": {
    "content": {
      "type": "string",
      "default": ""
    },
    "alignment": {
      "type": "string",
      "default": "left"
    }
  },
  "textdomain": "my-custom-block",
  "editorScript": "file:./index.js",
  "editorStyle": "file:./index.css",
  "style": "file:./style-index.css"
}
```

### 主要プロパティ

| プロパティ | 説明 |
|-----------|------|
| `name` | ブロックの一意な識別子。`namespace/block-name` の形式 |
| `title` | エディタに表示されるブロック名 |
| `category` | ブロック挿入パネルのカテゴリ（text, media, design, widgets, theme, embed） |
| `icon` | Dashiconsのアイコン名、またはSVG |
| `supports` | ブロックがサポートする機能（色、余白、配置など） |
| `attributes` | ブロックが保持するデータの定義 |
| `editorScript` | エディタ側で読み込むJSファイル |
| `editorStyle` | エディタ専用のCSSファイル |
| `style` | エディタ・フロント共通のCSSファイル |

### 参照元

- [WordPress Block API – block.json](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-metadata/)

---

## index.js — ブロック登録のエントリーポイント

```jsx
import { registerBlockType } from '@wordpress/blocks';
import './style.scss';
import Edit from './edit';
import save from './save';
import metadata from './block.json';

// block.jsonのメタデータを使ってブロックを登録
registerBlockType(metadata.name, {
  edit: Edit,
  save,
});
```

### 登録の仕組み

- `registerBlockType` に `block.json` の `name` とコンポーネントを渡す
- `edit`: エディタ画面での表示・操作を担当するReactコンポーネント
- `save`: 保存時にデータベースに書き込まれるHTML構造を返す関数

### 参照元

- [WordPress Block API – Registration](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-registration/)

---

## edit.js — エディタ側の表示

- エディタ画面でブロックがどう見えるか・どう操作できるかを定義する
- Reactコンポーネントとして実装する
- `@wordpress/block-editor` の `useBlockProps` で必須のprops・クラスを付与する

### 基本例

```jsx
import { __ } from '@wordpress/i18n';
import { useBlockProps, RichText } from '@wordpress/block-editor';
import './editor.scss';

export default function Edit({ attributes, setAttributes }) {
  const blockProps = useBlockProps();

  return (
    <div {...blockProps}>
      <RichText
        tagName="p"
        value={attributes.content}
        onChange={(content) => setAttributes({ content })}
        placeholder={__('テキストを入力…', 'my-custom-block')}
      />
    </div>
  );
}
```

### よく使うコンポーネント

| コンポーネント | インポート元 | 用途 |
|---------------|-------------|------|
| `useBlockProps` | `@wordpress/block-editor` | ブロックのラッパー要素に必須のpropsを付与 |
| `RichText` | `@wordpress/block-editor` | リッチテキスト編集（太字・リンクなど対応） |
| `InspectorControls` | `@wordpress/block-editor` | 右サイドバーに設定パネルを追加 |
| `BlockControls` | `@wordpress/block-editor` | ブロックツールバーにボタンを追加 |
| `MediaUpload` | `@wordpress/block-editor` | メディアライブラリの呼び出し |
| `TextControl` | `@wordpress/components` | テキスト入力フィールド |
| `ToggleControl` | `@wordpress/components` | ON/OFFトグル |
| `SelectControl` | `@wordpress/components` | ドロップダウン選択 |
| `PanelBody` | `@wordpress/components` | サイドバーの折りたたみセクション |

### サイドバー設定の追加例

```jsx
import { __ } from '@wordpress/i18n';
import {
  useBlockProps,
  RichText,
  InspectorControls,
} from '@wordpress/block-editor';
import {
  PanelBody,
  SelectControl,
  ToggleControl,
} from '@wordpress/components';

export default function Edit({ attributes, setAttributes }) {
  const { content, alignment, showBorder } = attributes;
  const blockProps = useBlockProps({
    className: showBorder ? 'has-border' : '',
  });

  return (
    <>
      {/* 右サイドバーの設定パネル */}
      <InspectorControls>
        <PanelBody title={__('表示設定', 'my-custom-block')}>
          <SelectControl
            label={__('配置', 'my-custom-block')}
            value={alignment}
            options={[
              { label: '左寄せ', value: 'left' },
              { label: '中央', value: 'center' },
              { label: '右寄せ', value: 'right' },
            ]}
            onChange={(value) => setAttributes({ alignment: value })}
          />
          <ToggleControl
            label={__('ボーダーを表示', 'my-custom-block')}
            checked={showBorder}
            onChange={(value) => setAttributes({ showBorder: value })}
          />
        </PanelBody>
      </InspectorControls>

      {/* エディタでの表示 */}
      <div {...blockProps}>
        <RichText
          tagName="p"
          style={{ textAlign: alignment }}
          value={content}
          onChange={(value) => setAttributes({ content: value })}
          placeholder={__('テキストを入力…', 'my-custom-block')}
        />
      </div>
    </>
  );
}
```

### 参照元

- [WordPress Block Editor Handbook – Edit and Save](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-edit-save/)
- [WordPress Components Reference](https://developer.wordpress.org/block-editor/reference-guides/components/)

---

## save.js — フロント側の表示

- 投稿を保存するときに生成されるHTMLを定義する
- `save` 関数の戻り値がデータベースに保存される
- `useBlockProps.save()` で必須のクラスやdata属性を付与する

### 基本例

```jsx
import { useBlockProps, RichText } from '@wordpress/block-editor';

export default function save({ attributes }) {
  const { content, alignment, showBorder } = attributes;
  const blockProps = useBlockProps.save({
    className: showBorder ? 'has-border' : '',
  });

  return (
    <div {...blockProps}>
      <RichText.Content
        tagName="p"
        style={{ textAlign: alignment }}
        value={content}
      />
    </div>
  );
}
```

### editとsaveの違い

| 項目 | edit | save |
|------|------|------|
| 実行タイミング | エディタ画面を開いたとき | 投稿を保存したとき |
| 用途 | 編集UI（入力・操作） | フロントエンドに出力するHTML |
| Reactの状態管理 | 使える（useState等） | 使えない（静的なHTML出力） |
| イベントハンドラ | 使える（onChange等） | 使えない |
| useBlockProps | `useBlockProps()` | `useBlockProps.save()` |
| RichText | `<RichText ... />` | `<RichText.Content ... />` |

### save関数の注意点

- save関数は純粋関数（同じattributesに対して常に同じHTMLを返す）でなければならない
- save関数のHTML出力が変わると、既存の投稿で「ブロックリカバリー」エラーが発生する
- 動的な出力が必要な場合はsaveで `null` を返し、PHPの `render_callback` を使う（ダイナミックブロック）

```jsx
// 良い例：saveは純粋にattributesからHTMLを生成
export default function save({ attributes }) {
  return <p {...useBlockProps.save()}>{attributes.content}</p>;
}

// 悪い例：saveで現在時刻など動的な値を使う
export default function save() {
  return <p {...useBlockProps.save()}>{new Date().toLocaleString()}</p>;
}
```

### 参照元

- [WordPress Block Editor Handbook – Edit and Save](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-edit-save/)

---

## ダイナミックブロック（PHPレンダリング）

- `save` 関数で `null` を返し、フロント側のHTMLをPHPで動的に生成する方式
- 最新の投稿一覧、ログインユーザーの情報など、表示時に動的なデータが必要な場合に使う
- ブロックリカバリーエラーが起きないため保守性が高い

### block.jsonに render を追加

```json
{
  "name": "my-plugin/dynamic-block",
  "title": "ダイナミックブロック",
  "render": "file:./render.php"
}
```

### render.php

```php
<?php
/**
 * ダイナミックブロックのフロント出力
 *
 * @param array    $attributes ブロックの属性値
 * @param string   $content    InnerBlocksのコンテンツ
 * @param WP_Block $block      ブロックインスタンス
 */

$wrapper_attributes = get_block_wrapper_attributes();
$recent_posts = wp_get_recent_posts([
    'numberposts' => $attributes['postsToShow'] ?? 3,
    'post_status' => 'publish',
]);
?>

<div <?php echo $wrapper_attributes; ?>>
    <ul>
        <?php foreach ($recent_posts as $post) : ?>
            <li>
                <a href="<?php echo esc_url(get_permalink($post['ID'])); ?>">
                    <?php echo esc_html($post['post_title']); ?>
                </a>
            </li>
        <?php endforeach; ?>
    </ul>
</div>
```

### save.jsはnullを返す

```jsx
// ダイナミックブロックのsave
export default function save() {
  return null;
}
```

### 参照元

- [WordPress Block Editor Handbook – Dynamic Blocks](https://developer.wordpress.org/block-editor/how-to-guides/block-tutorial/creating-dynamic-blocks/)

---

## PHPでブロックを登録する

### プラグインのメインPHPファイル

```php
<?php
/**
 * Plugin Name: My Custom Block
 * Description: カスタムブロックのサンプル
 * Version: 1.0.0
 * Requires at least: 6.1
 * Requires PHP: 7.0
 * Author: Your Name
 * Text Domain: my-custom-block
 */

if (! defined('ABSPATH')) {
    exit;
}

function my_custom_block_init() {
    // build/ディレクトリ内のblock.jsonを読み込んでブロックを登録
    register_block_type(__DIR__ . '/build');
}
add_action('init', 'my_custom_block_init');
```

### 仕組み

- `register_block_type()` に `build/` ディレクトリのパスを渡す
- `build/block.json` の `editorScript`、`editorStyle`、`style` を自動的にエンキューしてくれる
- 手動で `wp_enqueue_script` / `wp_enqueue_style` を書く必要がない

### 参照元

- [WordPress Block Editor Handbook – Block Registration (PHP)](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-registration/)

---

## @wordpress/scripts のビルドコマンド

### 主要コマンド

| コマンド | 説明 |
|---------|------|
| `npm start` | 開発用ビルド（watchモード、ファイル変更を監視して自動リビルド） |
| `npm run build` | 本番用ビルド（圧縮・最適化あり） |

### package.jsonのscripts

```json
{
  "scripts": {
    "build": "wp-scripts build",
    "start": "wp-scripts start",
    "format": "wp-scripts format",
    "lint:css": "wp-scripts lint-style",
    "lint:js": "wp-scripts lint-js"
  }
}
```

### ビルドの出力

- `src/` ディレクトリのファイルがトランスパイル・バンドルされて `build/` に出力される
- `build/index.js` — バンドルされたJavaScript
- `build/index.css` — エディタ用CSS
- `build/style-index.css` — フロント・エディタ共通CSS
- `build/index.asset.php` — 依存関係とバージョンハッシュ（自動生成）
- `build/block.json` — `src/block.json` のコピー

### 参照元

- [@wordpress/scripts – npm](https://www.npmjs.com/package/@wordpress/scripts)

---

## 属性（attributes）の定義と使い方

### attributesの型

| 型 | 説明 | 例 |
|---|------|-----|
| `string` | 文字列 | テキスト、URL |
| `number` | 数値 | カウント、サイズ |
| `boolean` | 真偽値 | ON/OFFフラグ |
| `array` | 配列 | リスト項目 |
| `object` | オブジェクト | 複合データ |

### 定義例（block.json）

```json
{
  "attributes": {
    "title": {
      "type": "string",
      "default": ""
    },
    "count": {
      "type": "number",
      "default": 3
    },
    "isVisible": {
      "type": "boolean",
      "default": true
    },
    "items": {
      "type": "array",
      "default": [],
      "items": {
        "type": "object"
      }
    }
  }
}
```

### 属性の読み書き

```jsx
// edit.js での使い方
export default function Edit({ attributes, setAttributes }) {
  const { title, count, isVisible } = attributes;

  // 良い例：setAttributesでオブジェクトを渡して更新
  const updateTitle = (newTitle) => {
    setAttributes({ title: newTitle });
  };

  // 良い例：複数の属性を同時に更新
  const resetSettings = () => {
    setAttributes({
      title: '',
      count: 3,
      isVisible: true,
    });
  };

  // 悪い例：attributesを直接書き換える
  // attributes.title = 'new title'; // ← これはNG
}
```

### 参照元

- [WordPress Block API – Attributes](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-attributes/)

---

## 開発のワークフロー

### 全体の流れ

```
1. npx @wordpress/create-block でプロジェクト生成
2. npm start で開発サーバーを起動
3. src/ 以下のファイルを編集（自動ビルドが走る）
4. ブラウザでエディタを開いてブロックの動作を確認
5. 問題なければ npm run build で本番ビルド
6. プラグインとしてデプロイ
```

### テンプレートをカスタマイズしたい場合

```bash
# インタラクティブモードで細かく設定
npx @wordpress/create-block --interactive

# TypeScript対応テンプレートを使う
npx @wordpress/create-block my-block --variant dynamic

# 既存プラグイン内にブロックを追加
npx @wordpress/create-block my-block --no-plugin
```

### 参照元

- [WordPress Block Editor Handbook – Quick Start Guide](https://developer.wordpress.org/block-editor/getting-started/tutorial/)

---

## よくあるトラブルと対処法

### ブロックリカバリーエラーが出る

- 原因：`save` 関数の出力HTMLが変更されたが、既存の投稿には古いHTMLが残っている
- 対処法：
  - 開発中は「ブロックのリカバリーを試行」で更新
  - 本番で頻繁に変更が見込まれるならダイナミックブロックにする
  - `deprecated` プロパティで旧バージョンのsaveを定義してマイグレーションする

### ブロックが挿入パネルに表示されない

- `block.json` の `name` の形式が正しいか確認（`namespace/block-name`）
- `register_block_type()` のパスが `build/` ディレクトリを指しているか確認
- `npm run build` が成功しているか確認（`build/` にファイルが出力されているか）

### スタイルが反映されない

- `npm start` が動いているか確認（SCSSの変更が自動ビルドされているか）
- ブラウザのキャッシュをクリアする
- `editor.scss` と `style.scss` の使い分けを確認（エディタだけか、フロントにも反映するか）

### 参照元

- [WordPress Block Editor Handbook – Troubleshooting](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-deprecation/)
