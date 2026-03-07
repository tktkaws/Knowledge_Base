# ARIAロールの基本 — role属性の種類と使い方

## WAI-ARIAとは

- Web Accessibility Initiative - Accessible Rich Internet Applications の略称
- HTMLだけでは表現しきれない意味や振る舞いを支援技術(スクリーンリーダーなど)に伝えるための仕様
- W3Cが策定した標準仕様

## role属性とは

- HTML要素に「この要素が何であるか」を明示するための属性
- ブラウザがアクセシビリティツリーを構築する際に使用される情報
- スクリーンリーダーが要素の役割をユーザーに伝える手がかり

```html
<div role="navigation">
  <a href="/home">ホーム</a>
  <a href="/about">概要</a>
</div>
```

## ARIAロールの大前則：HTMLのネイティブセマンティクスを優先する

- **適切なHTML要素が存在するなら、roleを使わずその要素を使うべき**
- これを「ARIAの第一ルール」と呼ぶ
- ネイティブHTML要素はrole・キーボード操作・フォーカス管理が組み込み済み
- ARIAはHTMLで表現できない場合の補助手段

```html
<!-- 悪い例：divにroleを付けてボタンを再現 -->
<div role="button" tabindex="0">送信</div>

<!-- 良い例：button要素をそのまま使う -->
<button>送信</button>
```

## ロールのカテゴリ一覧

### 1. ランドマークロール(Landmark Roles)

- ページの大まかな構造を示すロール
- スクリーンリーダーのランドマークナビゲーション機能でジャンプ先になる領域
- 対応するHTML要素がある場合はそちらを使うのが推奨

| ロール | 意味 | 対応するHTML要素 |
|---|---|---|
| `banner` | サイト全体のヘッダー | `<header>`(bodyの直下の場合) |
| `navigation` | ナビゲーションリンクのまとまり | `<nav>` |
| `main` | ページのメインコンテンツ | `<main>` |
| `complementary` | メインコンテンツの補足情報 | `<aside>` |
| `contentinfo` | サイト全体のフッター | `<footer>`(bodyの直下の場合) |
| `search` | 検索機能を持つ領域 | `<search>` |
| `region` | 汎用的なランドマーク領域 | `<section>`(アクセシブルな名前がある場合) |
| `form` | フォーム領域 | `<form>`(アクセシブルな名前がある場合) |

```html
<!-- nav要素を使えばrole="navigation"は不要 -->
<nav aria-label="メインナビゲーション">
  <ul>
    <li><a href="/">ホーム</a></li>
    <li><a href="/products">製品</a></li>
  </ul>
</nav>

<!-- 同じ種類のランドマークが複数ある場合はaria-labelで区別 -->
<nav aria-label="パンくずリスト">...</nav>
<nav aria-label="メインナビゲーション">...</nav>
```

### 2. ウィジェットロール(Widget Roles)

- インタラクティブなUI要素に使用するロール
- ネイティブHTML要素で代替できるものが多い
- カスタムUIを構築する場合に使用

#### よく使うウィジェットロール

| ロール | 意味 | 対応するHTML要素 |
|---|---|---|
| `button` | ボタン | `<button>` |
| `link` | リンク | `<a href="...">` |
| `checkbox` | チェックボックス | `<input type="checkbox">` |
| `radio` | ラジオボタン | `<input type="radio">` |
| `textbox` | テキスト入力欄 | `<input type="text">` / `<textarea>` |
| `slider` | スライダー | `<input type="range">` |
| `switch` | ON/OFF切替 | なし(カスタム実装が必要) |
| `tab` | タブ | なし |
| `tabpanel` | タブパネル | なし |
| `dialog` | ダイアログ | `<dialog>` |
| `alertdialog` | 警告ダイアログ | なし |
| `tooltip` | ツールチップ | なし |
| `menu` | メニュー | なし(アプリケーションメニュー用) |
| `menuitem` | メニュー内の項目 | なし |

```html
<!-- タブUI：ネイティブ要素がないためroleが必要 -->
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="panel-1">タブ1</button>
  <button role="tab" aria-selected="false" aria-controls="panel-2">タブ2</button>
</div>
<div role="tabpanel" id="panel-1">タブ1の内容</div>
<div role="tabpanel" id="panel-2" hidden>タブ2の内容</div>
```

### 3. 文書構造ロール(Document Structure Roles)

- ページ内のコンテンツの構造を示すロール
- 多くはHTML要素で代替可能

| ロール | 意味 | 対応するHTML要素 |
|---|---|---|
| `heading` | 見出し | `<h1>` ~ `<h6>` |
| `list` | リスト | `<ul>` / `<ol>` |
| `listitem` | リスト項目 | `<li>` |
| `table` | テーブル | `<table>` |
| `row` | テーブル行 | `<tr>` |
| `cell` | テーブルセル | `<td>` |
| `columnheader` | テーブル列ヘッダー | `<th>` |
| `img` | 画像 | `<img>` |
| `article` | 独立した記事 | `<article>` |
| `separator` | 区切り線 | `<hr>` |
| `toolbar` | ツールバー | なし |
| `group` | 関連要素のグループ | `<fieldset>` |

### 4. ライブリージョンロール(Live Region Roles)

- 動的に変化するコンテンツの変更をスクリーンリーダーに通知するロール
- ページ更新やリアルタイム情報の伝達に使用

| ロール | 意味 | 通知の緊急度 |
|---|---|---|
| `alert` | 重要な警告メッセージ | 高（assertive相当） |
| `status` | 状態の変化を示す情報 | 低（polite相当） |
| `log` | ログ情報の追加 | 低（polite相当） |
| `marquee` | 繰り返し変化する情報 | 低 |
| `timer` | タイマー | 低 |

```html
<!-- フォーム送信後のステータス表示 -->
<div role="status">保存しました</div>

<!-- エラーアラート -->
<div role="alert">入力内容にエラーがあります</div>
```

## 暗黙のロール(Implicit Roles)

- HTML要素には仕様上、デフォルトでロールが割り当てられている
- これを「暗黙のロール」と呼ぶ
- 暗黙のロールと同じ値をrole属性で明示的に指定するのは冗長

| HTML要素 | 暗黙のロール |
|---|---|
| `<button>` | `button` |
| `<a href="...">` | `link` |
| `<input type="checkbox">` | `checkbox` |
| `<select>` | `combobox`(単一選択時） / `listbox`(multiple時） |
| `<header>`(body直下) | `banner` |
| `<nav>` | `navigation` |
| `<main>` | `main` |
| `<footer>`(body直下) | `contentinfo` |
| `<aside>` | `complementary` |
| `<h1>` ~ `<h6>` | `heading` |
| `<ul>` / `<ol>` | `list` |
| `<li>` | `listitem` |
| `<table>` | `table` |
| `<form>`(名前付き) | `form` |
| `<section>`(名前付き) | `region` |

```html
<!-- 冗長な例：button要素は暗黙のロールがbuttonなのでrole指定は不要 -->
<button role="button">送信</button>

<!-- 正しい例 -->
<button>送信</button>
```

## role属性を使う際のルール

### roleはアクセシビリティツリー上の意味を変えるだけ

- role属性は見た目や動作を一切変えない
- キーボード操作やフォーカス管理は自分で実装する必要がある
- role="button"を付けただけではEnterキーやSpaceキーで動作しない

```html
<!-- roleだけでは不十分な例 -->
<span role="button">送信</span>

<!-- 最低限必要な実装 -->
<span
  role="button"
  tabindex="0"
  onkeydown="if(event.key==='Enter'||event.key===' ')handleClick()"
  onclick="handleClick()"
>
  送信
</span>

<!-- だからbutton要素を使うべき -->
<button onclick="handleClick()">送信</button>
```

### roleの上書きに関する制約

- HTML要素の暗黙のロールを別のロールに変更できる場合がある
- ただし仕様上許可された組み合わせのみ有効
- 不適切なロールの上書きは支援技術の動作を壊す原因

```html
<!-- 許可されている例：liにtabロールを適用 -->
<ul role="tablist">
  <li role="tab">タブ1</li>
  <li role="tab">タブ2</li>
</ul>

<!-- 禁止されている例：見出しをボタンに変更 -->
<h2 role="button">これはNG</h2>
```

### presentationとnone

- `role="presentation"` と `role="none"` は同義
- 要素のセマンティクスをアクセシビリティツリーから除去する
- レイアウト目的でtableを使う場合などに使用

```html
<!-- レイアウト用テーブルからテーブルセマンティクスを除去 -->
<table role="presentation">
  <tr>
    <td>レイアウト用</td>
    <td>テーブル</td>
  </tr>
</table>
```

## よくある間違い

### 1. ネイティブ要素を使わずdiv + roleで再実装する

```html
<!-- 間違い -->
<div role="button" tabindex="0">クリック</div>

<!-- 正解 -->
<button>クリック</button>
```

- div+roleの場合、キーボード操作・フォーム送信・disabled状態など全て自前実装が必要

### 2. role="menu"をナビゲーションに使う

```html
<!-- 間違い：サイトナビゲーションにmenuロールを使う -->
<ul role="menu">
  <li role="menuitem"><a href="/">ホーム</a></li>
  <li role="menuitem"><a href="/about">概要</a></li>
</ul>

<!-- 正解：navとリストを使う -->
<nav>
  <ul>
    <li><a href="/">ホーム</a></li>
    <li><a href="/about">概要</a></li>
  </ul>
</nav>
```

- `role="menu"` はデスクトップアプリケーションのメニューバーを模したUIのためのロール
- サイトのナビゲーションリンクには `<nav>` を使用

### 3. aria-labelなしでランドマークを複数配置

```html
<!-- 間違い：どのnavか区別できない -->
<nav>...</nav>
<nav>...</nav>

<!-- 正解：aria-labelで区別 -->
<nav aria-label="メインメニュー">...</nav>
<nav aria-label="フッターリンク">...</nav>
```

## まとめ

- ARIAロールは要素の「役割」を支援技術に伝える仕組み
- 最も重要な原則：**ネイティブHTML要素で表現できるならroleは使わない**
- ランドマーク・ウィジェット・文書構造・ライブリージョンの4カテゴリ
- roleは意味だけを変更し、見た目や動作は一切変えない
- カスタムUIにroleを使う場合、キーボード操作やフォーカス管理の自前実装が必須
