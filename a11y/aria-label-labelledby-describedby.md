# aria-label / aria-labelledby / aria-describedby の使い分け

## 3つの属性の概要

| 属性 | 役割 | 値の形式 |
|---|---|---|
| `aria-label` | 要素にアクセシブルな名前を直接指定 | テキスト文字列 |
| `aria-labelledby` | 別の要素のテキストを参照してアクセシブルな名前を設定 | 要素のID(スペース区切りで複数可) |
| `aria-describedby` | 要素に補足説明を関連付け | 要素のID(スペース区切りで複数可) |

## アクセシブルな名前(Accessible Name)とは

- スクリーンリーダーが要素を読み上げる際に使う「名前」
- ボタンなら「送信ボタン」の「送信」の部分
- 入力欄ならlabel要素で紐づけたテキスト
- 要素が何であるかを識別するための情報

## aria-label

### 基本

- 要素にアクセシブルな名前を直接文字列で指定する属性
- 画面上に見えるラベルテキストがない場合に使用
- 値は翻訳されない可能性があるため、多言語対応時は注意が必要

```html
<!-- アイコンだけのボタンにラベルを付与 -->
<button aria-label="メニューを開く">
  <svg><!-- ハンバーガーアイコン --></svg>
</button>

<!-- 閉じるボタン -->
<button aria-label="閉じる">✕</button>

<!-- 検索フォーム -->
<input type="search" aria-label="サイト内検索">
```

### 使いどころ

- アイコンのみのボタン（ハンバーガーメニュー、閉じるボタンなど）
- ラベルが視覚的に不要だが支援技術には必要な入力欄
- 同じ種類のランドマークが複数存在する場合の区別

```html
<!-- 同じnavが複数ある場合の区別 -->
<nav aria-label="メインナビゲーション">...</nav>
<nav aria-label="フッターナビゲーション">...</nav>
```

### 注意点

- **画面上に見えるラベルテキストが既にある場合は使わない**
- aria-labelは要素の既存のテキストコンテンツを上書きする
- 見えるテキストと読み上げが食い違う原因になる

```html
<!-- 悪い例：見えるテキストと読み上げが一致しない -->
<button aria-label="フォームを送信する">送信</button>
<!-- スクリーンリーダー：「フォームを送信する ボタン」 -->
<!-- 画面上の表示：「送信」 -->

<!-- 良い例：テキストがあるならaria-labelは不要 -->
<button>送信</button>
```

## aria-labelledby

### 基本

- 画面上に表示されている別の要素のテキストをアクセシブルな名前として参照する属性
- 値はラベルとなる要素のID
- 複数のIDをスペース区切りで指定し、名前を結合できる

```html
<h2 id="billing-heading">お支払い情報</h2>
<form aria-labelledby="billing-heading">
  <!-- フォーム内容 -->
</form>
```

### 使いどころ

- 画面上に既に見えるテキストをラベルとして再利用したい場合
- 見出しとセクションの関連付け
- ダイアログのタイトルとダイアログ要素の紐付け
- 複数の要素のテキストを組み合わせてラベルにしたい場合

```html
<!-- ダイアログに見出しをラベルとして紐付け -->
<dialog aria-labelledby="dialog-title">
  <h2 id="dialog-title">アカウント削除の確認</h2>
  <p>本当にアカウントを削除しますか？</p>
  <button>削除</button>
  <button>キャンセル</button>
</dialog>

<!-- 複数要素のテキストを結合してラベルにする -->
<span id="name-label">名前</span>
<span id="required-text">（必須）</span>
<input type="text" aria-labelledby="name-label required-text">
<!-- スクリーンリーダー：「名前 （必須） テキスト入力」 -->
```

### aria-label との優先順位

- aria-labelledbyはaria-labelよりも優先度が高い
- 両方指定された場合、aria-labelledbyの値が使われる
- アクセシブルな名前の計算優先順位:
  1. `aria-labelledby`
  2. `aria-label`
  3. HTML要素に紐づいたラベル（`<label>`など）
  4. 要素のテキストコンテンツ

```html
<!-- aria-labelledbyが優先される -->
<span id="label-text">設定画面</span>
<button aria-labelledby="label-text" aria-label="設定">
  <svg><!-- 歯車アイコン --></svg>
</button>
<!-- スクリーンリーダー：「設定画面 ボタン」 -->
```

### 注意点

- 参照先のIDが存在しない場合、ラベルが空になる
- `hidden`や`display: none`の要素でも参照可能（aria-labelledby独自の特徴）
- 参照先のテキストが変更されると、ラベルも自動的に変わる

```html
<!-- 非表示の要素でもaria-labelledbyで参照できる -->
<span id="secret-label" hidden>ダッシュボードに戻る</span>
<a href="/dashboard" aria-labelledby="secret-label">
  <svg><!-- 戻るアイコン --></svg>
</a>
```

## aria-describedby

### 基本

- 要素に**補足説明**を関連付ける属性
- aria-label / aria-labelledbyが「名前」を付けるのに対し、aria-describedbyは「説明」を付ける
- スクリーンリーダーは名前の読み上げ後に短い間を置いて説明を読み上げる

```html
<label for="password">パスワード</label>
<input
  type="password"
  id="password"
  aria-describedby="password-hint"
>
<p id="password-hint">8文字以上、英数字と記号を含めてください</p>
<!-- スクリーンリーダー：「パスワード テキスト入力 ... 8文字以上、英数字と記号を含めてください」 -->
```

### 使いどころ

- フォーム入力欄のヒントテキストや入力規則
- エラーメッセージの関連付け
- ボタンやリンクに対する補足情報
- ツールチップの内容との紐付け

```html
<!-- エラーメッセージの関連付け -->
<label for="email">メールアドレス</label>
<input
  type="email"
  id="email"
  aria-describedby="email-error"
  aria-invalid="true"
>
<p id="email-error" role="alert">有効なメールアドレスを入力してください</p>

<!-- ヒントとエラーの両方を関連付け -->
<label for="username">ユーザー名</label>
<input
  type="text"
  id="username"
  aria-describedby="username-hint username-error"
>
<p id="username-hint">半角英数字3〜20文字</p>
<p id="username-error" role="alert">このユーザー名は既に使用されています</p>

<!-- 削除ボタンに補足説明 -->
<button aria-describedby="delete-warning">アカウントを削除</button>
<p id="delete-warning">この操作は取り消せません</p>
```

### 名前(label)と説明(description)の違い

- **名前**：要素を識別するための短い情報（「送信」「ユーザー名」など）
- **説明**：要素を理解するための補足情報（「8文字以上」「この操作は取り消せません」など）
- スクリーンリーダーの読み上げ順：名前 → 役割 → 状態 → **説明**
- 説明は名前の後に読み上げられるため、まず名前で要素を特定し、追加情報として説明を伝える構造

## 3つの属性の使い分けフローチャート

```
要素にラベルを付けたい
  ├── 画面上にラベルテキストが見えている？
  │     ├── YES → aria-labelledby（既存テキストを参照）
  │     └── NO  → aria-label（直接テキストを指定）
  │
  └── ラベルに加えて補足説明も付けたい？
        └── YES → aria-describedby（名前とは別の補足情報を紐付け）
```

## 組み合わせの実例

### フォーム入力欄の完全な例

```html
<div>
  <label for="phone">電話番号</label>
  <input
    type="tel"
    id="phone"
    aria-describedby="phone-format phone-error"
    aria-invalid="true"
  >
  <p id="phone-format">例：090-1234-5678</p>
  <p id="phone-error" role="alert">電話番号の形式が正しくありません</p>
</div>
<!-- 名前：「電話番号」（labelから取得） -->
<!-- 説明：「例：090-1234-5678 電話番号の形式が正しくありません」 -->
```

### モーダルダイアログの例

```html
<dialog
  aria-labelledby="modal-title"
  aria-describedby="modal-desc"
>
  <h2 id="modal-title">ファイルの削除</h2>
  <p id="modal-desc">「report.pdf」を完全に削除します。この操作は元に戻せません。</p>
  <button>削除する</button>
  <button>キャンセル</button>
</dialog>
<!-- 名前：「ファイルの削除」 -->
<!-- 説明：「report.pdfを完全に削除します。この操作は元に戻せません。」 -->
```

### 同じ「詳しく見る」リンクが複数ある場合

```html
<article>
  <h3 id="article-1">Reactの基礎</h3>
  <p>Reactの基本的な概念を解説...</p>
  <a href="/react-basics" aria-labelledby="article-1 read-more-1">
    <span id="read-more-1">を読む</span>
  </a>
</article>
<!-- スクリーンリーダー：「Reactの基礎 を読む リンク」 -->

<article>
  <h3 id="article-2">CSS Gridの使い方</h3>
  <p>CSS Gridの基本的なレイアウト手法...</p>
  <a href="/css-grid" aria-labelledby="article-2 read-more-2">
    <span id="read-more-2">を読む</span>
  </a>
</article>
<!-- スクリーンリーダー：「CSS Gridの使い方 を読む リンク」 -->
```

## よくある間違い

### 1. 見えるテキストがあるのにaria-labelで上書き

```html
<!-- 悪い例 -->
<a href="/contact" aria-label="お問い合わせページへ">お問い合わせ</a>

<!-- 良い例：リンクテキストだけで十分 -->
<a href="/contact">お問い合わせ</a>
```

### 2. aria-describedbyを名前の代わりに使う

```html
<!-- 悪い例：名前がなく説明だけがある -->
<input type="text" aria-describedby="field-label">
<span id="field-label">ユーザー名</span>

<!-- 良い例：名前はlabelで、説明はdescribedbyで -->
<label for="username">ユーザー名</label>
<input type="text" id="username" aria-describedby="username-hint">
<span id="username-hint">半角英数字3〜20文字</span>
```

### 3. 存在しないIDを参照

```html
<!-- 悪い例：IDが存在しないためラベルが空になる -->
<button aria-labelledby="nonexistent-id">
  <svg><!-- アイコン --></svg>
</button>
```

## まとめ

- `aria-label`：画面に見えるラベルがない場合に直接テキストを指定
- `aria-labelledby`：画面に見えるテキストを参照してラベルにする（最優先）
- `aria-describedby`：ラベルとは別の補足説明を紐付ける
- まずHTML標準の仕組み（`<label>`, テキストコンテンツ）で対応できないか検討
- 名前(label系)と説明(describedby)は役割が異なるため、適切に使い分ける
