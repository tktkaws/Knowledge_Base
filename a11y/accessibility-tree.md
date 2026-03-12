# アクセシビリティツリーの仕組み

## アクセシビリティツリーとは

- ブラウザがDOMツリーから生成する、もう一つのツリー構造
- 支援技術（スクリーンリーダーなど）がWebページの内容を理解するためのデータ構造
- DOMツリーの「意味的な情報」だけを抽出・再構成したもの
- 視覚的な見た目（CSS）ではなく、要素の「役割」「名前」「状態」を保持する構造
- ユーザーの目に見えないが、スクリーンリーダーが読み上げる内容の源泉

> 参照: [MDN — Accessibility tree](https://developer.mozilla.org/en-US/docs/Glossary/Accessibility_tree)

## DOMツリーとアクセシビリティツリーの関係

- DOMツリーはHTMLを解析した結果のすべての要素を含むツリー
- アクセシビリティツリーはDOMツリーから派生する並行的なツリー
- DOMの全要素がアクセシビリティツリーに含まれるわけではない
- 装飾目的の要素（`<div>`のラッパーなど）は省略されることがある
- 逆に、DOMに存在しない情報（ARIAで付与した名前や状態）が追加されることもある

```
DOMツリー                    アクセシビリティツリー
─────────                    ──────────────────
<html>                       document
  <body>                       ├─ banner (header)
    <header>                   │    └─ navigation (nav)
      <nav>                    │         ├─ link "ホーム"
        <a href="/">ホーム</a>  │         └─ link "概要"
        <a href="/about">概要</a>├─ main
      </nav>                   │    ├─ heading "記事タイトル" (level 1)
    </header>                  │    └─ paragraph "本文テキスト..."
    <main>                     └─ contentinfo (footer)
      <h1>記事タイトル</h1>           └─ link "お問い合わせ"
      <p>本文テキスト...</p>
    </main>
    <footer>
      <a href="/contact">お問い合わせ</a>
    </footer>
  </body>
</html>
```

- DOMツリーでは `<html>`, `<body>`, `<header>` など構造要素がすべて存在
- アクセシビリティツリーでは意味のある情報（ロール、名前）だけに整理される

> 参照: [W3C — The Accessibility Tree](https://www.w3.org/TR/accname-1.2/)

## アクセシビリティツリーに含まれる4つの情報

アクセシビリティツリーの各ノードは、以下の4つの情報を持つ。

### 1. ロール（Role）

- 要素が「何であるか」を示す情報
- 例: `button`, `link`, `heading`, `navigation`, `textbox`
- HTML要素の暗黙のロール、または `role` 属性で明示的に指定されたロール

### 2. 名前（Accessible Name）

- 要素を識別するためのテキストラベル
- 例: ボタンの「送信」、リンクの「ホームに戻る」、画像の代替テキスト
- テキストコンテンツ、`aria-label`、`aria-labelledby`、`<label>` などから算出

### 3. 状態（State）

- 要素の現在の状態を示す情報
- 例: `checked`（チェック済み）、`expanded`（展開済み）、`disabled`（無効）、`selected`（選択済み）
- HTML属性やARIA属性から取得

### 4. プロパティ（Property）

- 要素の追加情報を示すもの
- 例: `required`（必須）、`readonly`（読み取り専用）、`aria-describedby`（説明の参照先）
- 状態と異なり、ユーザー操作で頻繁に変化しない情報

```html
<!-- この要素のアクセシビリティツリー上の情報 -->
<input
  type="checkbox"
  id="agree"
  checked
  aria-describedby="agree-desc"
/>
<label for="agree">利用規約に同意する</label>
<span id="agree-desc">同意しないと登録できません</span>

<!--
  ロール: checkbox
  名前: "利用規約に同意する"（labelから算出）
  状態: checked（チェック済み）
  プロパティ: describedby="同意しないと登録できません"
-->
```

> 参照: [W3C — WAI-ARIA 1.2 — Accessibility Semantics](https://www.w3.org/TR/wai-aria-1.2/#accessibility_tree)

## ブラウザがアクセシビリティツリーを構築するプロセス

アクセシビリティツリーが支援技術に届くまでの流れは以下の通り。

```
HTML
  ↓ パース
DOM ツリー
  ↓ ブラウザが意味的情報を抽出
アクセシビリティツリー
  ↓ プラットフォーム固有のAPIに変換
OS のアクセシビリティ API
  ↓ 支援技術がAPIを通じて情報を取得
スクリーンリーダー等の支援技術
```

### 各段階の詳細

1. **HTML → DOM ツリー**: ブラウザがHTMLを解析し、DOMツリーを構築
2. **DOM → アクセシビリティツリー**: ブラウザがDOM要素のセマンティクス（暗黙のロール、ARIA属性、テキストコンテンツなど）を解析し、アクセシビリティツリーを構築
3. **アクセシビリティツリー → OS のアクセシビリティ API**: ブラウザがアクセシビリティツリーの情報を、OS固有のアクセシビリティAPIに変換して公開
   - Windows: UI Automation (UIA) / MSAA
   - macOS: NSAccessibility
   - Linux: AT-SPI
4. **OS のアクセシビリティ API → 支援技術**: スクリーンリーダー（VoiceOver, NVDA, JAWSなど）がOSのAPIを通じて情報を取得し、ユーザーに読み上げ・点字表示などで伝達

> 参照: [MDN — Accessibility tree](https://developer.mozilla.org/en-US/docs/Glossary/Accessibility_tree), [W3C — Core Accessibility API Mappings](https://www.w3.org/TR/core-aam-1.2/)

## アクセシビリティツリーに影響を与えるHTML要素とARIA属性

### セマンティックHTML要素

- HTML要素は仕様上「暗黙のロール」を持っている
- 適切なHTML要素を使うだけで、アクセシビリティツリーに正しい情報が反映される

| HTML要素 | アクセシビリティツリーでのロール |
|---|---|
| `<button>` | `button` |
| `<a href="...">` | `link` |
| `<h1>` ~ `<h6>` | `heading`（level 1〜6） |
| `<nav>` | `navigation` |
| `<main>` | `main` |
| `<header>`（body直下） | `banner` |
| `<footer>`（body直下） | `contentinfo` |
| `<input type="text">` | `textbox` |
| `<input type="checkbox">` | `checkbox` |
| `<select>` | `combobox` / `listbox` |
| `<table>` | `table` |
| `<img alt="...">` | `img` |

### ARIA属性がアクセシビリティツリーに与える影響

- `role` 属性: 要素のロールを上書きまたは明示
- `aria-label`: アクセシブルな名前を直接指定
- `aria-labelledby`: 別の要素のテキストを名前として参照
- `aria-describedby`: 補足説明を関連付け
- `aria-hidden`: アクセシビリティツリーから要素を除外
- `aria-expanded`, `aria-checked`, `aria-selected`: 要素の状態を伝達
- `aria-required`, `aria-readonly`: 要素のプロパティを伝達

```html
<!-- role属性でアクセシビリティツリー上のロールを変更 -->
<div role="alert">エラーが発生しました</div>
<!-- アクセシビリティツリー: ロール=alert, 名前="エラーが発生しました" -->

<!-- aria-labelでアクセシブルな名前を付与 -->
<button aria-label="メニューを開く">
  <svg><!-- ハンバーガーアイコン --></svg>
</button>
<!-- アクセシビリティツリー: ロール=button, 名前="メニューを開く" -->
```

> 参照: [W3C — WAI-ARIA 1.2](https://www.w3.org/TR/wai-aria-1.2/), [MDN — ARIA](https://developer.mozilla.org/ja/docs/Web/Accessibility/ARIA)

## アクセシビリティツリーから除外される要素

以下の条件に当てはまる要素は、アクセシビリティツリーに含まれない。

### `display: none`

- 要素が視覚的にもアクセシビリティツリー上も完全に非表示
- スクリーンリーダーからも認識されない

```css
.hidden {
  display: none; /* アクセシビリティツリーから除外 */
}
```

### `visibility: hidden`

- 要素のレイアウト領域は維持されるが、視覚的に非表示
- アクセシビリティツリーからも除外される

```css
.invisible {
  visibility: hidden; /* アクセシビリティツリーから除外 */
}
```

### `aria-hidden="true"`

- 視覚的には表示されたまま、アクセシビリティツリーからのみ除外
- 装飾用のアイコンなど、支援技術に伝える必要がない要素に使用
- **子孫要素もすべて除外される**ため、フォーカス可能な要素を含めてはいけない

```html
<!-- 装飾アイコンをスクリーンリーダーから隠す -->
<button>
  <span aria-hidden="true">★</span>
  お気に入りに追加
</button>

<!-- 危険な使い方：フォーカス可能な要素をaria-hiddenの中に入れる -->
<!-- 悪い例 -->
<div aria-hidden="true">
  <button>このボタンはフォーカスできるがスクリーンリーダーで認識されない</button>
</div>
```

### `role="presentation"` / `role="none"`

- 要素自体のセマンティクスをアクセシビリティツリーから除去
- ただし子要素のセマンティクスは保持される
- `aria-hidden="true"` とは異なり、子孫を丸ごと隠すわけではない

```html
<!-- テーブルのセマンティクスを除去（レイアウト目的のテーブル） -->
<table role="presentation">
  <tr>
    <td>レイアウト用のセル</td>
  </tr>
</table>
```

### 視覚的に隠すがアクセシビリティツリーには残す方法

- `display: none` や `aria-hidden="true"` と異なり、視覚的にだけ隠したい場合がある
- スクリーンリーダー専用のテキスト（「メニューを開く」など）を提供するケース

```css
/* visually-hiddenパターン：視覚的に隠すがアクセシビリティツリーには残る */
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

```html
<a href="/home">
  <svg aria-hidden="true"><!-- 家のアイコン --></svg>
  <span class="visually-hidden">ホームに戻る</span>
</a>
```

> 参照: [MDN — aria-hidden](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-hidden), [A11Y Project — Visually Hidden](https://www.a11yproject.com/posts/how-to-hide-content/)

## セマンティックHTMLがアクセシビリティツリーに与える影響

- セマンティックHTMLを正しく使うだけで、アクセシビリティツリーに適切な情報が自動的に反映される
- `<div>` や `<span>` だけで構成されたページは、アクセシビリティツリー上ほぼ空の構造になる
- これが「ARIAの第一ルール：ネイティブHTML要素を使えるなら使う」の根拠

### 悪い例：div乱用によるアクセシビリティツリーの欠落

```html
<!-- 悪い例：divだけで構成されたページ -->
<div class="header">
  <div class="logo">サイト名</div>
  <div class="nav">
    <div class="nav-item" onclick="location.href='/'">ホーム</div>
    <div class="nav-item" onclick="location.href='/about'">概要</div>
  </div>
</div>
<div class="main">
  <div class="title" style="font-size: 24px; font-weight: bold;">記事タイトル</div>
  <div class="content">本文テキスト</div>
  <div class="btn" onclick="handleSubmit()">送信</div>
</div>
<div class="footer">
  <div class="link" onclick="location.href='/contact'">お問い合わせ</div>
</div>

<!--
  アクセシビリティツリー:
    generic         ← divはすべて"generic"ロール。意味なし
      generic
      generic
        generic     ← リンクとして認識されない
        generic
    generic
      generic       ← 見出しとして認識されない
      generic
      generic       ← ボタンとして認識されない
    generic
      generic       ← リンクとして認識されない

  → スクリーンリーダーはページ構造を把握できない
  → ランドマークナビゲーション不可
  → Tab キーでのナビゲーション不可（フォーカスできない）
-->
```

### 良い例：セマンティックHTMLによる豊かなアクセシビリティツリー

```html
<!-- 良い例：セマンティックHTMLを使用 -->
<header>
  <a href="/" class="logo">サイト名</a>
  <nav aria-label="メインナビゲーション">
    <ul>
      <li><a href="/">ホーム</a></li>
      <li><a href="/about">概要</a></li>
    </ul>
  </nav>
</header>
<main>
  <h1>記事タイトル</h1>
  <p>本文テキスト</p>
  <button onclick="handleSubmit()">送信</button>
</main>
<footer>
  <a href="/contact">お問い合わせ</a>
</footer>

<!--
  アクセシビリティツリー:
    banner                              ← ランドマーク
      link "サイト名"                    ← リンクとして認識
      navigation "メインナビゲーション"    ← ランドマーク
        list
          listitem
            link "ホーム"
          listitem
            link "概要"
    main                                ← ランドマーク
      heading "記事タイトル" (level 1)    ← 見出しとして認識
      paragraph "本文テキスト"
      button "送信"                      ← ボタンとして認識（Tab可、Enter/Space可）
    contentinfo                          ← ランドマーク
      link "お問い合わせ"

  → スクリーンリーダーがページ全体の構造を正確に把握
  → ランドマークジャンプ、見出しジャンプが可能
  → Tab キーで全インタラクティブ要素にアクセス可能
-->
```

> 参照: [W3C — Using ARIA](https://www.w3.org/TR/using-aria/), [MDN — HTML elements reference](https://developer.mozilla.org/ja/docs/Web/HTML/Element)

## ARIA属性の正しい使い方と間違った使い方

### 悪い例：不要なARIAの追加

```html
<!-- 悪い例：ネイティブ要素にすでにある意味を二重に指定 -->
<button role="button">送信</button>
<nav role="navigation">...</nav>
<a href="/" role="link">ホーム</a>

<!-- 良い例：ネイティブ要素をそのまま使う -->
<button>送信</button>
<nav>...</nav>
<a href="/">ホーム</a>
```

### 悪い例：ARIAだけに頼ってキーボード操作を忘れる

```html
<!-- 悪い例：role="button"を付けたがキーボード操作を実装していない -->
<div role="button">送信</div>
<!-- → Tabキーでフォーカスできない、Enter/Spaceで動作しない -->

<!-- 良い例：button要素を使えばすべて組み込み済み -->
<button>送信</button>
```

### 良い例：ネイティブ要素で表現できないUIにARIAを使用

```html
<!-- タブUI：ネイティブ要素が存在しないためARIAが必要 -->
<div role="tablist" aria-label="設定カテゴリ">
  <button role="tab" aria-selected="true" aria-controls="panel-general" id="tab-general">
    一般
  </button>
  <button role="tab" aria-selected="false" aria-controls="panel-display" id="tab-display">
    表示
  </button>
</div>
<div role="tabpanel" id="panel-general" aria-labelledby="tab-general">
  一般設定の内容...
</div>
<div role="tabpanel" id="panel-display" aria-labelledby="tab-display" hidden>
  表示設定の内容...
</div>
```

> 参照: [W3C — Using ARIA — First Rule of ARIA Use](https://www.w3.org/TR/using-aria/#firstrule)

## アクセシブルな名前（Accessible Name）の計算方法

- アクセシビリティツリーの各ノードは「アクセシブルな名前」を持つ
- スクリーンリーダーが要素を読み上げる際の識別名として使われる
- 名前の算出は優先度の高い順に以下のルールで行われる

### 名前の算出優先順位

1. **`aria-labelledby`**: 指定されたIDの要素のテキストを名前として使用（最優先）
2. **`aria-label`**: 属性値をそのまま名前として使用
3. **`<label>` 要素**: フォーム要素の場合、関連付けられた `<label>` のテキスト
4. **テキストコンテンツ**: 要素内のテキスト（`<button>送信</button>` の「送信」）
5. **`title` 属性**: 他に名前がない場合のフォールバック
6. **`placeholder` 属性**: 他に名前がない場合のフォールバック（非推奨）

```html
<!-- 1. aria-labelledby（最優先） -->
<h2 id="dialog-title">確認</h2>
<div role="dialog" aria-labelledby="dialog-title">
  <!-- アクセシブルな名前: "確認" -->
</div>

<!-- 2. aria-label -->
<button aria-label="閉じる">
  <svg><!-- Xアイコン --></svg>
</button>
<!-- アクセシブルな名前: "閉じる" -->

<!-- 3. label要素 -->
<label for="email">メールアドレス</label>
<input type="email" id="email" />
<!-- アクセシブルな名前: "メールアドレス" -->

<!-- 4. テキストコンテンツ -->
<button>送信する</button>
<!-- アクセシブルな名前: "送信する" -->

<!-- 5. title属性（フォールバック） -->
<input type="text" title="検索キーワード" />
<!-- アクセシブルな名前: "検索キーワード" -->
```

### アクセシブルな名前がない場合の問題

```html
<!-- 悪い例：アクセシブルな名前がないボタン -->
<button>
  <svg viewBox="0 0 24 24">
    <path d="M3 18h18v-2H3v2zm0-5h18v-2H3v2zm0-7v2h18V6H3z"/>
  </svg>
</button>
<!-- スクリーンリーダー: "ボタン"としか読み上げない。何のボタンか不明 -->

<!-- 良い例：aria-labelで名前を付与 -->
<button aria-label="メニューを開く">
  <svg viewBox="0 0 24 24" aria-hidden="true">
    <path d="M3 18h18v-2H3v2zm0-5h18v-2H3v2zm0-7v2h18V6H3z"/>
  </svg>
</button>
<!-- スクリーンリーダー: "メニューを開く ボタン" -->
```

> 参照: [W3C — Accessible Name and Description Computation 1.2](https://www.w3.org/TR/accname-1.2/), [MDN — Accessible name](https://developer.mozilla.org/en-US/docs/Glossary/Accessible_name)

## Chrome DevTools でアクセシビリティツリーを確認する方法

### 方法1: Elements パネルの Accessibility ペイン

1. Chrome DevTools を開く（F12 または Cmd+Opt+I）
2. **Elements** パネルを選択
3. 確認したいHTML要素を選択
4. 右側のペインから **Accessibility** タブを選択
5. 以下の情報を確認可能
   - **Computed Properties**: ロール、名前（Accessible Name）、キーボードフォーカス可否
   - **ARIA Attributes**: 要素に設定されたARIA属性
   - **Source of Name**: アクセシブルな名前がどこから算出されたか

### 方法2: アクセシビリティツリーの全体表示

1. Chrome DevTools を開く
2. **Elements** パネルを選択
3. Elements パネル内で右クリック
4. **「Enable full-page accessibility tree」** をクリック（または Elements パネル左上のアクセシビリティアイコンをクリック）
5. DOMツリーの表示がアクセシビリティツリー表示に切り替わる
6. 各ノードのロール・名前・状態が一覧で確認可能

### 確認すべきポイント

- 各要素が適切なロールを持っているか
- ボタンやリンクにアクセシブルな名前が付いているか
- フォーム要素にラベルが関連付けられているか
- ランドマークが正しく構成されているか
- `aria-hidden="true"` で意図せず重要な情報を隠していないか

> 参照: [Chrome Developers — The Accessibility Tab](https://developer.chrome.com/docs/devtools/accessibility/reference)

## まとめ

- アクセシビリティツリーはDOMツリーから生成される並行的なツリー構造で、支援技術がページを理解する基盤
- 各ノードはロール・名前・状態・プロパティの4つの情報を持つ
- HTML → DOM → アクセシビリティツリー → OS API → 支援技術 の流れで情報が伝達される
- セマンティックHTMLを使うだけで、アクセシビリティツリーに適切な情報が自動反映される
- `div` の乱用はアクセシビリティツリーをほぼ空にし、支援技術でのページ理解を困難にする
- `display: none`、`visibility: hidden`、`aria-hidden="true"` はアクセシビリティツリーから要素を除外する
- ARIAはネイティブHTML要素で表現できない場合にのみ使用する
- Chrome DevTools でアクセシビリティツリーを確認し、実装の正しさを検証できる
