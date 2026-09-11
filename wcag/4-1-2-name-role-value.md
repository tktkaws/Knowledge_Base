# 4.1.2 名前（name）・役割（role）・値（value）（A）

## 達成基準の概要

- WCAG 2.2 達成基準 4.1.2「名前（name）・役割（role）・値（value）」（Name, Role, Value）
- レベル **A**
- 原則4「堅牢（Robust）」> ガイドライン 4.1「互換性」に属する
- すべての**ユーザーインターフェースコンポーネント**について、次を満たすこと
  1. **名前（name）**と**役割（role）**がプログラムで解釈できる
  2. ユーザーが設定できる**状態・プロパティ・値**がプログラムで設定できる
  3. それらの変更通知が、ユーザーエージェント・支援技術に届く
- 対象例：フォームコントロール、リンク、スクリプトで作ったカスタム UI
- **標準 HTML を仕様どおりに使えば、多くの場合すでに満たせる**
- 主に、独自 UI やスクリプトで役割を変えたコンポーネントを作る作者向け

> **参照**
> - [WCAG 2.2 — Success Criterion 4.1.2 Name, Role, Value](https://www.w3.org/TR/WCAG22/#name-role-value)
> - [Understanding SC 4.1.2 Name, Role, Value](https://www.w3.org/WAI/WCAG22/Understanding/name-role-value.html)

## 達成基準の原文（要約）

- すべてのユーザーインターフェースコンポーネントについて
  - 名前と役割がプログラムで解釈できる
  - ユーザーが設定できる状態・プロパティ・値はプログラムで設定できる
  - これらの変更通知がユーザーエージェント（支援技術を含む）に提供される
- 注：主に独自 UI を開発・スクリプトする作者向け。標準 HTML コントロールは仕様どおり使えばこの基準を満たす

> **参照**
> - [WCAG 2.2 — Success Criterion 4.1.2](https://www.w3.org/TR/WCAG22/#name-role-value)

## なぜ必要か

- スクリーンリーダーは、名前・役割・状態・値を読み上げて操作を助ける
- 画面拡大・音声入力なども、正しい役割と名前に依存する
- 見た目だけボタンでも、コード上は `div` のままでは支援技術が理解できない
- フォーカス・チェック・展開などの状態変化が伝わらないと、操作不能に近い
- アクセシビリティ API に正しく露出することが、支援技術との互換の基盤になる

> **参照**
> - [Understanding SC 4.1.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/name-role-value.html#intent)
> - [Understanding SC 4.1.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/name-role-value.html#benefits)

## 名前・役割・値とは

| 用語 | 意味 | 例 |
|---|---|---|
| **名前（name）** | ソフトウェアがコンポーネントを識別するテキスト | 「送信」「メールアドレス」 |
| **役割（role）** | コンポーネントの機能種別 | button、link、checkbox、dialog |
| **状態（state）** | 変化しうる動的な特性 | focus、checked、expanded、pressed |
| **値（value）** | 現在の値 | スライダーの 50、テキスト入力の内容 |

- **名前**は支援技術にだけ見えてもよい。すべてのユーザーに見えるものは**ラベル**
- 多くの場合、ラベルと名前は一致する
- HTML の `name` 属性とは無関係
- 何を伝えるべきかは、WAI-ARIA や各プラットフォームの仕様に従う

```text
スクリーンリーダーが知りたいこと（例：チェックボックス）：
  名前 → 「利用規約に同意する」
  役割 → チェックボックス
  状態 → 未チェック / チェック済み
  操作後 → 状態変更が通知される
```

> **参照**
> - [WCAG 2.2 — Glossary: name / role / state / label](https://www.w3.org/TR/WCAG22/#dfn-name)
> - [Accessible Name and Description Computation 1.2](https://www.w3.org/TR/accname-1.2/)

## 標準 HTML を使う（最優先）

- `button`、`a`、`input`、`select`、`textarea` などは、仕様どおり使えば名前・役割・値を備える
- ARIA の第一ルール：適切なネイティブ要素があるならそれを使う
- カスタム実装は、ネイティブで足りないときだけ

```html
<!-- 良い例：ネイティブの button / label / checkbox -->
<label>
  <input type="checkbox" name="agree">
  利用規約に同意する
</label>
<button type="submit">送信</button>
```

```html
<!-- 悪い例：見た目だけコントロール -->
<div class="checkbox" onclick="toggle()">利用規約に同意する</div>
<div class="btn" onclick="submit()">送信</div>
<!-- role・名前・キーボード・状態通知がない -->
```

> **参照**
> - [Technique H91: Using HTML form controls and links](https://www.w3.org/WAI/WCAG22/Techniques/html/H91)
> - [Technique H88: Using HTML according to spec](https://www.w3.org/WAI/WCAG22/Techniques/html/H88)
> - [a11y/aria-first-rule.md — ARIA の第一ルール](../a11y/aria-first-rule.md)

## 状況別の十分な手法

### 状況A：マークアップの標準 UI（HTML など）

- **G108**：マークアップ機能で名前・役割を露出する
- **H91**：HTML のフォームコントロールとリンク
- **H44**：`label` でフォームコントロールとテキストを関連付け
- **H64**：`iframe` の `title`
- **H65**：`label` が使えないときの `title`（最終手段寄り）
- **ARIA14 / ARIA16**：`aria-label` / `aria-labelledby` で名前を提供

### 状況B：スクリプトで標準 UI の役割を変える

- **ARIA16**：`aria-labelledby` で名前を提供
- 役割変更時は、期待される状態・キーボード操作も実装する

### 状況D：独自 UI コンポーネント

- **G10**：アクセシビリティ API を支援する技術でコンポーネントを作る
- **ARIA4**：`role` で役割を露出
- **ARIA5**：状態・プロパティ属性で状態を露出
- **ARIA16**：名前の提供

> **参照**
> - [Understanding SC 4.1.2 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/name-role-value.html#techniques)

## 手法1：label で名前を付ける（H44）

- 見えるラベルとコントロールをプログラムで関連付ける
- アクセシブルネームの基本

```html
<!-- 良い例：label の関連付け -->
<label for="email">メールアドレス</label>
<input id="email" name="email" type="email" autocomplete="email">
```

```html
<!-- 悪い例：見た目の文言だけで関連付けなし -->
<p>メールアドレス</p>
<input name="email" type="email">
<!-- プログラムで解釈できる名前がない（F68） -->
```

> **参照**
> - [Technique H44: Using label elements to associate text labels with form controls](https://www.w3.org/WAI/WCAG22/Techniques/html/H44)

## 手法2：カスタム UI に role と状態を付ける（ARIA4 / ARIA5）

- 独自ウィジェットでは、役割・状態・値を明示する
- トグル、コンボボックス、ディスクロージャーなどが典型
- 状態変化時に属性を更新し、支援技術へ通知できるようにする

```html
<!-- 良い例：ディスクロージャー -->
<button
  type="button"
  aria-expanded="false"
  aria-controls="faq-1"
  id="faq-1-btn"
>
  配送料はいくらですか？
</button>
<div id="faq-1" hidden>
  全国一律 500 円です。
</div>
```

```html
<!-- 良い例：トグルボタン -->
<button type="button" aria-pressed="false">太字</button>
<!-- 押下で aria-pressed="true" に更新 -->
```

```html
<!-- 悪い例：div をクリック可能にしただけ（F59） -->
<div class="accordion-header" onclick="open()">配送料はいくらですか？</div>
<div class="accordion-panel">全国一律 500 円です。</div>
<!-- role・expanded・キーボード操作なし -->
```

> **参照**
> - [Technique ARIA4: Using a WAI-ARIA role to expose the role of a user interface component](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA4)
> - [Technique ARIA5: Using WAI-ARIA state and property attributes to expose the state of a user interface component](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA5)
> - [a11y/aria-expanded-hidden-controls.md — aria-expanded / aria-controls](../a11y/aria-expanded-hidden-controls.md)

## 手法3：aria-labelledby / aria-label で名前を補う（ARIA16 / ARIA14）

- 見えるラベルを流用するなら `aria-labelledby`
- 見えるラベルを置けないときだけ `aria-label`
- 見えるラベルがあるのにアクセシブルネームが空、は失敗（F111）

```html
<!-- 良い例：複数テキストを名前に使う -->
<span id="qty-label">数量</span>
<span id="qty-hint">（1〜10）</span>
<input
  id="qty"
  type="number"
  min="1"
  max="10"
  aria-labelledby="qty-label qty-hint"
>
```

```html
<!-- 良い例：アイコンのみボタン -->
<button type="button" aria-label="メニューを開く">
  <svg aria-hidden="true" focusable="false"><!-- icon --></svg>
</button>
```

```html
<!-- 悪い例：見える「検索」があるのに名前がない -->
<div class="search">
  <span>検索</span>
  <input type="search">
  <button type="submit">
    <img src="search.svg" alt="">
  </button>
</div>
```

> **参照**
> - [Technique ARIA16: Using aria-labelledby to provide a name for user interface controls](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA16)
> - [Technique ARIA14: Using aria-label to provide an accessible name where a visible label cannot be used](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA14)
> - [a11y/aria-label-labelledby-describedby.md — aria-label / labelledby](../a11y/aria-label-labelledby-describedby.md)

## 手法4：画像リンク・分割フィールドにも名前を付ける

- リンクの中身が画像だけのとき、画像にテキスト代替（またはリンク側の名前）が必要（F89）
- 電話番号の分割入力など、各パートにも名前が必要（F86）

```html
<!-- 良い例：画像のみリンク -->
<a href="/">
  <img src="logo.svg" alt="会社名ホーム">
</a>
```

```html
<!-- 悪い例：alt もリンク名もない -->
<a href="/">
  <img src="logo.svg" alt="">
</a>
```

```html
<!-- 良い例：分割フィールドにも名前 -->
<fieldset>
  <legend>電話番号</legend>
  <label for="tel-area">市外局番</label>
  <input id="tel-area" name="tel_area" type="text" inputmode="numeric">
  <label for="tel-local">市内局番以降</label>
  <input id="tel-local" name="tel_local" type="text" inputmode="numeric">
</fieldset>
```

> **参照**
> - [Failure F89: No accessible name for an image which is the only content in a link](https://www.w3.org/WAI/WCAG22/Techniques/failures/F89)
> - [Failure F86: Not providing names for each part of a multi-part form field](https://www.w3.org/WAI/WCAG22/Techniques/failures/F86)

## フォーカス状態の通知

- フォーカスの有無は特に重要な状態
- プログラムで解釈でき、変化が支援技術に通知される必要がある（F79）
- ネイティブのフォーカス可能要素を使い、独自フォーカス管理では API に露出する

```html
<!-- 良い例：ネイティブのフォーカス可能要素 -->
<button type="button">設定を開く</button>
<a href="/help">ヘルプ</a>
```

```html
<!-- 悪い例：見た目のフォーカスだけでプログラム上は不明 -->
<div class="fake-focus" style="outline: 2px solid blue">項目</div>
<!-- tabindex もなく、支援技術にフォーカス状態が伝わらない -->
```

> **参照**
> - [Failure F79: Focus state not programmatically determinable](https://www.w3.org/WAI/WCAG22/Techniques/failures/F79)
> - [a11y/focus-management.md — フォーカス管理](../a11y/focus-management.md)

## 代表的な失敗（Failures）

| ID | 内容 |
|---|---|
| **F59** | `div` / `span` を UI にしたのに `role` がない |
| **F68** | UI コントロールにプログラムで解釈できる名前がない |
| **F15** | カスタムコントロールがアクセシビリティ API を使わない／不完全 |
| **F42** | リンクをエミュレートしているが名前・役割などが不十分 |
| **F79** | フォーカス状態が解釈できない／変化通知がない |
| **F86** | 分割フォームの各パートに名前がない |
| **F89** | 画像のみリンクにアクセシブルネームがない |
| **F111** | 見えるラベルがあるのにアクセシブルネームがない |
| **F20** | 非テキストの変化にテキスト代替が追随しない |

> **参照**
> - [Understanding SC 4.1.2 — Failures](https://www.w3.org/WAI/WCAG22/Understanding/name-role-value.html#techniques)

## 関連する達成基準

| 達成基準 | 関係 |
|---|---|
| **1.3.1 情報および関係性** | 構造・関係性の露出。4.1.2 は UI コンポーネントの名前・役割・値 |
| **2.1.1 キーボード** | カスタム UI は役割だけでなくキーボード操作も必要 |
| **2.4.4 / 2.4.9 リンクの目的** | リンクの名前が目的を示すか |
| **2.5.3 名前（name）のラベル** | 見えるラベル文字列がアクセシブルネームに含まれるか |
| **3.3.2 ラベルまたは説明** | 入力の手がかり。4.1.2 はプログラムでの名前露出 |
| **4.1.3 ステータスメッセージ** | ステータス更新の通知（AA） |

> **参照**
> - [wcag/1-3-1-info-and-relationships.md — 1.3.1](./1-3-1-info-and-relationships.md)
> - [wcag/2-5-3-label-in-name.md — 2.5.3](./2-5-3-label-in-name.md)

## よくある失敗例

- `div` / `span` をクリック可能にしただけで `role` も名前もない
- アイコンボタンに `aria-label` も画像 `alt` もない
- カスタムアコーディオンで `aria-expanded` を更新しない
- 見える「氏名」ラベルと入力が関連付いていない
- チェックボックスの選択状態が支援技術に伝わらない
- `role="button"` にしたのに Enter / Space やフォーカスがない
- リンク風の `div` で、名前・役割・キーボードが欠落（F42 系）

```html
<!-- 失敗例 -->
<span class="link" onclick="location.href='/docs'">ドキュメント</span>
```

```html
<!-- 改善例 -->
<a href="/docs">ドキュメント</a>
```

## テスト・確認方法

### 手動・支援技術

- すべての操作可能コントロールを洗い出す
- スクリーンリーダーで、名前・役割・状態が読み上げられるか確認する
- 操作後に状態変化（checked / expanded など）が伝わるか確認する
- ブラウザのアクセシビリティツリーで Name / Role / Value を確認する

### 自動チェックの補助

- axe などで「ボタンに名前がない」「role がない」などを検出
- 自動だけではカスタムウィジェットの状態更新漏れは見落としやすい

```text
チェックリスト：
1. 可能な限りネイティブ HTML コントロールを使っているか
2. すべての UI にプログラムで解釈できる名前があるか
3. カスタム UI に適切な role があるか
4. checked / expanded / pressed などが更新されているか
5. フォーカス状態が支援技術に伝わるか
6. 画像のみリンクにアクセシブルネームがあるか
7. 分割フィールドの各パートに名前があるか
8. 見えるラベルがあるコントロールで、ネームが空になっていないか
```

> **参照**
> - [a11y/accessibility-tree.md — アクセシビリティツリー](../a11y/accessibility-tree.md)
> - [a11y/a11y-automated-testing.md — 自動テスト](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 4.1.2 は、UI コンポーネントの**名前・役割・状態／値**を支援技術が使えるようにする達成基準（レベル A）
- 標準 HTML を仕様どおり使えば、多くの場合すでに達成できる
- カスタム UI では ARIA の `role`・状態属性・名前付けが必要
- 状態変化とフォーカス変化の通知も要件に含まれる
- `div` ボタン化、名前なしコントロール、状態未更新が代表的な失敗
- 1.3.1・2.1.1・2.5.3・3.3.2 と組み合わせて、意味・操作・ラベルを一貫させる
