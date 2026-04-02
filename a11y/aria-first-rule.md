# WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う

## ARIAの5つのルール

- W3Cは「Using ARIA」ドキュメントでARIAを使う際の5つのルールを定義
- 最も重要なのが第1ルール：**ネイティブHTMLで実現できるなら、ARIAを使わない**

> **参照**
> - [W3C - Using ARIA](https://www.w3.org/TR/using-aria/)

## 第1ルール：ネイティブHTML要素を優先する

- 「要素の意味や振る舞いに対応するネイティブHTML要素またはHTML属性が存在するなら、ARIAのロール・ステート・プロパティを追加して要素をアクセシブルにするのではなく、そのネイティブ要素・属性を使うべき」
- ネイティブHTML要素はブラウザが以下を自動的に提供するため、ARIAよりも堅牢
  - 暗黙のロール（アクセシビリティツリーへの反映）
  - キーボード操作（フォーカス、Enter/Space による操作）
  - フォームの送信・バリデーション
  - 状態管理（disabled、checked、selectedなど）

### 典型的な違反パターンと正しい実装

#### ボタン

```html
<!-- 悪い例：divにARIAを追加してボタンを再現 -->
<div role="button" tabindex="0" onclick="handleClick()"
     onkeydown="if(event.key==='Enter'||event.key===' ')handleClick()">
  送信
</div>
<!-- 必要な実装：role, tabindex, click, keydown, disabled状態の管理... -->

<!-- 良い例：button要素を使う -->
<button onclick="handleClick()">送信</button>
<!-- キーボード操作、フォーカス、ロール、disabled属性が全て組み込み済み -->
```

#### リンク

```html
<!-- 悪い例：spanにrole="link"を付与 -->
<span role="link" tabindex="0" onclick="location.href='/about'"
      onkeydown="if(event.key==='Enter')location.href='/about'">
  概要ページへ
</span>
<!-- 右クリックメニュー、Cmd+クリックで新タブ、ブラウザのリンク機能が全て使えない -->

<!-- 良い例：a要素を使う -->
<a href="/about">概要ページへ</a>
<!-- ブラウザ標準のリンク機能が全て使える -->
```

#### チェックボックス

```html
<!-- 悪い例：divでチェックボックスを自作 -->
<div role="checkbox" tabindex="0" aria-checked="false"
     onclick="toggleCheck(this)"
     onkeydown="if(event.key===' ')toggleCheck(this)">
  利用規約に同意
</div>

<!-- 良い例：input[type="checkbox"]を使う -->
<label>
  <input type="checkbox" name="agree">
  利用規約に同意
</label>
<!-- チェック状態管理、フォーム送信、label連携が全て組み込み済み -->
```

#### 見出し

```html
<!-- 悪い例：divにrole="heading"を付与 -->
<div role="heading" aria-level="2">セクションタイトル</div>

<!-- 良い例：h2要素を使う -->
<h2>セクションタイトル</h2>
```

#### ナビゲーション

```html
<!-- 悪い例：divにrole="navigation"を付与 -->
<div role="navigation">
  <a href="/">ホーム</a>
  <a href="/about">概要</a>
</div>

<!-- 良い例：nav要素を使う -->
<nav>
  <a href="/">ホーム</a>
  <a href="/about">概要</a>
</nav>
```

> **参照**
> - [W3C - Using ARIA - First Rule](https://www.w3.org/TR/using-aria/#firstrule)
> - [MDN - ARIA and HTML](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Guides/ARIA_and_HTML)

## 第2ルール：ネイティブセマンティクスを不必要に変更しない

- HTML要素が持つ暗黙のロールを、不必要に別のロールで上書きしない
- やむを得ない場合を除き、要素の本来の意味を保つ

```html
<!-- 悪い例：見出しをタブに変更 -->
<h2 role="tab">タブ1</h2>

<!-- 良い例：見出しの中にタブ要素を配置 -->
<h2><button role="tab">タブ1</button></h2>

<!-- もしくはtabロール用に別の要素を使う -->
<div role="tablist">
  <button role="tab">タブ1</button>
  <button role="tab">タブ2</button>
</div>
```

> **参照**
> - [W3C - Using ARIA - Second Rule](https://www.w3.org/TR/using-aria/#secondrule)

## 第3ルール：すべてのインタラクティブARIA要素はキーボードで操作可能にする

- `role="button"` を付けた要素はEnterキーとSpaceキーで操作可能にする必要がある
- `role="slider"` を付けた要素は矢印キーで値を変更可能にする必要がある
- ARIAはロールを宣言するだけで、操作はブラウザが自動補完しない

```html
<!-- 悪い例：キーボード操作が実装されていない -->
<div role="button" tabindex="0" onclick="handleClick()">
  送信
</div>
<!-- マウスでは動くがキーボードでは動かない -->

<!-- 悪い例の修正：キーボード操作を追加 -->
<div role="button" tabindex="0" onclick="handleClick()"
     onkeydown="if(event.key==='Enter'||event.key===' '){event.preventDefault();handleClick()}">
  送信
</div>
<!-- ここまでするなら最初からbuttonを使うべき -->
```

> **参照**
> - [W3C - Using ARIA - Third Rule](https://www.w3.org/TR/using-aria/#thirdrule)

## 第4ルール：フォーカス可能な要素にrole="presentation"やaria-hidden="true"を使わない

- フォーカス可能な要素（`<button>`, `<a href>`, `<input>`など）を `aria-hidden="true"` で隠すと、スクリーンリーダーでは読めないがキーボードではフォーカスできる「ゴースト要素」が生まれる
- ユーザーに見えない要素にフォーカスが飛ぶ混乱を引き起こす

```html
<!-- 悪い例：フォーカス可能な要素をaria-hiddenで隠す -->
<button aria-hidden="true">非表示のボタン</button>
<!-- キーボードでフォーカスは当たるが、スクリーンリーダーは無視する -->

<!-- 良い例：本当に非表示にするならdisplayで消す -->
<button style="display: none;">非表示のボタン</button>
<!-- フォーカスも当たらず、スクリーンリーダーからも除外される -->

<!-- 良い例：inert属性で非活性化（モーダル背面など） -->
<div inert>
  <button>この領域のボタンはフォーカスも操作も不可</button>
</div>
```

> **参照**
> - [W3C - Using ARIA - Fourth Rule](https://www.w3.org/TR/using-aria/#fourthrule)

## 第5ルール：すべてのインタラクティブ要素にアクセシブルな名前を付ける

- ボタン、リンク、入力欄などのインタラクティブ要素には必ず名前が必要
- 名前がないと、スクリーンリーダーは「ボタン」「リンク」としか読み上げられない

```html
<!-- 悪い例：アクセシブルな名前がない -->
<button>
  <svg viewBox="0 0 24 24"><path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/></svg>
</button>
<!-- 読み上げ：「ボタン」 -->

<!-- 良い例：aria-labelで名前を付与 -->
<button aria-label="項目を追加">
  <svg aria-hidden="true" viewBox="0 0 24 24"><path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/></svg>
</button>
<!-- 読み上げ：「項目を追加、ボタン」 -->
```

> **参照**
> - [W3C - Using ARIA - Fifth Rule](https://www.w3.org/TR/using-aria/#fifthrule)
> - [W3C - Accessible Name and Description Computation](https://www.w3.org/TR/accname-1.2/)

## ARIAが必要になる正当なケース

- ネイティブHTMLに対応する要素が存在しないウィジェット
  - タブUI（`role="tablist"`, `role="tab"`, `role="tabpanel"`）
  - ツリービュー（`role="tree"`, `role="treeitem"`）
  - ツールバー（`role="toolbar"`）
  - コンボボックス / オートコンプリート（`role="combobox"`）
- 動的コンテンツの状態通知
  - `aria-live` によるライブリージョン
  - `aria-expanded` による展開/折りたたみ状態
  - `aria-busy` によるロード中の状態
- ネイティブ要素だけでは不足する追加情報
  - `aria-describedby` による補足説明の関連付け
  - `aria-current` による現在位置の明示
  - `aria-invalid` / `aria-errormessage` によるエラー状態

```html
<!-- ARIAが必要な例：タブUI -->
<div role="tablist" aria-label="商品情報">
  <button role="tab" aria-selected="true" aria-controls="panel-detail" id="tab-detail">
    詳細
  </button>
  <button role="tab" aria-selected="false" aria-controls="panel-review" id="tab-review">
    レビュー
  </button>
</div>
<div role="tabpanel" id="panel-detail" aria-labelledby="tab-detail">
  詳細の内容...
</div>
<div role="tabpanel" id="panel-review" aria-labelledby="tab-review" hidden>
  レビューの内容...
</div>
```

> **参照**
> - [W3C - ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)

## よくあるアンチパターン

### 冗長なARIA

```html
<!-- 悪い例：暗黙のロールと同じroleを明示 -->
<nav role="navigation">...</nav>
<button role="button">送信</button>
<a href="/about" role="link">概要</a>

<!-- 良い例：ネイティブ要素の暗黙のロールを信頼 -->
<nav>...</nav>
<button>送信</button>
<a href="/about">概要</a>
```

### ARIAの過剰使用

```html
<!-- 悪い例：全てにARIAを付けすぎ -->
<div role="main" aria-label="メインコンテンツ">
  <div role="heading" aria-level="1" aria-label="ページタイトル">ホーム</div>
  <div role="list" aria-label="メニュー">
    <div role="listitem"><div role="link" tabindex="0">リンク1</div></div>
  </div>
</div>

<!-- 良い例：ネイティブHTMLで実現 -->
<main>
  <h1>ホーム</h1>
  <ul>
    <li><a href="/link1">リンク1</a></li>
  </ul>
</main>
```

> **参照**
> - [WebAIM - To ARIA! The Cause of, and Solution to, All Our Accessibility Problems](https://webaim.org/blog/aria-cause-solution/)

## まとめ

- ARIAのファーストルール：ネイティブHTML要素が使えるなら、ARIAを使わない
- ネイティブHTML要素はロール・キーボード操作・状態管理が組み込み済みで、ARIAより堅牢
- ARIAはHTMLだけでは表現できないウィジェットや状態のための補助手段
- 「No ARIA is better than bad ARIA」— 不適切なARIAは、ARIAなしよりもアクセシビリティを悪化させる
- 5つのルールを理解し、ARIAを使う前に「ネイティブHTMLで実現できないか？」を常に確認
