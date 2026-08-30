# 3.3.5 ヘルプ（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.3.5「ヘルプ」（Help）
- レベル **AAA**
- 原則3「理解可能（Understandable）」> ガイドライン 3.3「入力支援」に属する
- **文脈に応じたヘルプ**（context-sensitive help）が利用できること
- 現在実行している操作・入力に関連する説明・支援を提供する
- ラベルだけで機能が十分に説明できる場合は、追加ヘルプは不要
- ヘルプの**存在が分かり**、必要なときに**取得できる**こと

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.5 Help](https://www.w3.org/TR/WCAG22/#help)
> - [Understanding SC 3.3.5 Help](https://www.w3.org/WAI/WCAG22/Understanding/help.html)

## 達成基準の原文（要約）

- 文脈に応じたヘルプが利用できること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.5](https://www.w3.org/TR/WCAG22/#help)

## なぜ必要か

- 認知・学習・読み書きの障害がある人は、入力ミスを起こしやすい
- 高齢者も、テキスト入力やマウス操作でつまずきやすい
- 操作手順が分からないと、タスクを途中で諦めやすい
- 文脈に沿ったヘルプがあれば、**今やっていること**を見失わずに支援を受けられる
- ミスを減らし、フォームや手続きの完了率を上げられる

> **参照**
> - [Understanding SC 3.3.5 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/help.html#intent)
> - [Understanding SC 3.3.5 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/help.html#benefits)

## 文脈に応じたヘルプとは

- **現在実行している機能**に関連するヘルプテキスト
- フォーム全体の説明、各項目横のヘルプリンク、入力形式の例、スペルチェックなど
- **明確なラベル**自体が文脈に応じたヘルプとして機能しうる
- 作者がヘルプを提供しても、ユーザーエージェントがプログラム的に提供してもよい

```text
文脈に応じたヘルプの例：
  求職フォームの「志望動機」横の「記入のヒント」リンク
  パスポート番号欄の形式説明（AA1234567）
  フォーム冒頭の入力手順

文脈に応じたヘルプではない例：
  サイト全体の「お問い合わせ」（3.2.6 のヘルプ機構）
  別タスクの FAQ ページへの一般リンクだけ
  ラベルで十分説明できている単純な「氏名」欄
```

> **参照**
> - [Understanding SC 3.3.5 — Key Terms: context-sensitive help](https://www.w3.org/WAI/WCAG22/Understanding/help.html)

## 3.3.2・3.2.6 との関係

| 達成基準 | レベル | 焦点 |
|---|---|---|
| 3.3.2 ラベルまたは説明 | A | 入力項目に**見える**ラベル・説明 |
| 3.2.6 一貫したヘルプ | A | サイト横断の**ヘルプ機構**の位置 |
| 3.3.5 ヘルプ | AAA | ラベルでは足りないときの**文脈別**支援 |

- 3.3.2 は最低限のラベル・説明（レベル A）
- 3.3.5 は、ラベルだけでは機能を説明しきれないときの追加ヘルプ（レベル AAA）
- 3.2.6 は FAQ・お問い合わせなど**サイト全体**のヘルプの一貫性
- 3.3.5 は**今のフォーム・今の項目**に紐づくヘルプ

```text
3.3.2 だけで足りる例：
  <label for="name">氏名</label>
  <input id="name" name="name" type="text">

3.3.5 も必要になりやすい例：
  「社会保障番号（SSN）」— 初見では何をどう書くか分からない
  「志望動機（500字以内）」— 書き方のガイドが必要
  「法人番号」— 形式・取得方法の説明が必要
```

> **参照**
> - [wcag/3-3-2-labels-or-instructions.md — 3.3.2 ラベルまたは説明](./3-3-2-labels-or-instructions.md)
> - [wcag/3-2-6-consistent-help.md — 3.2.6 一貫したヘルプ](./3-2-6-consistent-help.md)

## 状況別の十分な手法

### 状況A：フォームがテキスト入力を要求する

- **G71**：各 Web ページにヘルプリンクを提供
- **G193**：ページ内アシスタント（チャットボット等）でヘルプを提供
- **G194**：スペルチェックと入力候補の提供
- **G184**：フォームまたはフィールド群の冒頭に、必要な入力の説明

### 状況B：フォームが決まったデータ形式の入力を要求する

- **G89**：期待するデータ形式と例の提供
- **G184**：フォームまたはフィールド群の冒頭に、必要な入力の説明

> **参照**
> - [Understanding SC 3.3.5 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/help.html#techniques)

## 手法1：フォーム冒頭に入力手順を示す（G184）

- フォームまたは関連フィールド群の**最初**に、必要な入力内容を説明
- 必須項目、形式、文字数制限、添付ファイルの条件など
- 3.3.2 の説明を拡張する形で、3.3.5 も満たしうる

```html
<!-- 良い例：フォーム冒頭の手順 -->
<form action="/apply" method="post">
  <h1>求人応募フォーム</h1>
  <div id="form-intro">
    <p>
      すべての必須項目（<abbr title="必須">*</abbr>）に入力してください。
      履歴書は PDF 形式、5MB 以内で添付してください。
      分からない項目は各設問横の「ヘルプ」を参照してください。
    </p>
  </div>
  <!-- 入力フィールド -->
</form>
```

```html
<!-- 悪い例：説明なしでいきなり入力 -->
<form action="/apply" method="post">
  <label for="resume">履歴書 *</label>
  <input id="resume" name="resume" type="file">
  <!-- 形式・サイズ・必須の説明がない -->
</form>
```

> **参照**
> - [Technique G184: Providing text instructions at the beginning of a form or set of fields that describes the necessary input](https://www.w3.org/WAI/WCAG22/Techniques/general/G184)

## 手法2：期待する形式と例を示す（G89）

- 日付、電話番号、ID 番号など、**決まった形式**が求められる入力向け
- 形式の説明と具体例を、項目の近くに置く
- 3.3.2 の説明、3.3.3 のエラー提案と重なることもある

```html
<!-- 良い例：形式と例 -->
<label for="passport">パスポート番号</label>
<input
  id="passport"
  name="passport"
  type="text"
  aria-describedby="passport-hint"
>
<p id="passport-hint">
  2文字のアルファベット + 7桁の数字。例：AB1234567
</p>
```

```html
<!-- 悪い例：形式が分からない -->
<label for="passport">パスポート番号</label>
<input id="passport" name="passport" type="text">
```

> **参照**
> - [Technique G89: Providing expected data format and example](https://www.w3.org/WAI/WCAG22/Techniques/general/G89)

## 手法3：項目ごとにヘルプリンクを置く（G71）

- 複雑な設問の横に、**その項目専用**のヘルプリンク
- リンク先は同一ページ内のアンカー、モーダル、別ページでもよい
- 存在が分かり、必要なときに取得できることが重要

```html
<!-- 良い例：設問横のヘルプ -->
<div class="field">
  <label for="motivation">
    志望動機（500字以内）
    <a href="#help-motivation" id="motivation-help-link">ヘルプ</a>
  </label>
  <textarea id="motivation" name="motivation" aria-describedby="help-motivation"></textarea>
</div>
<section id="help-motivation" aria-labelledby="help-motivation-title">
  <h2 id="help-motivation-title">志望動機の書き方</h2>
  <p>
    なぜこの職種・この会社に応募するのかを具体的に書いてください。
    過去の経験と将来の目標を結びつけると分かりやすいです。
  </p>
</section>
```

```html
<!-- 悪い例：ヘルプが見つからない -->
<label for="motivation">志望動機</label>
<textarea id="motivation" name="motivation"></textarea>
<!-- 500字制限も書き方も不明。ヘルプへの導線もない -->
```

> **参照**
> - [Technique G71: Providing a help link on every web page](https://www.w3.org/WAI/WCAG22/Techniques/general/G71)
> - [Understanding SC 3.3.5 — Examples: on-line job application](https://www.w3.org/WAI/WCAG22/Understanding/help.html#examples)

## 手法4：ページ内アシスタントで支援する（G193）

- チャットボット、仮想アシスタント、対話型ヘルプ
- **現在のページ・フォーム**に関連する質問に答えられること
- 汎用 FAQ だけでは文脈に応じたヘルプにならない場合がある

```html
<!-- 良い例：フォーム文脈を理解するアシスタント -->
<button type="button" aria-controls="form-assistant" aria-expanded="false">
  入力のサポート
</button>
<div id="form-assistant" hidden>
  <p>このフォームについて質問できます。例：「履歴書の形式は？」</p>
  <!-- アシスタント UI -->
</div>
```

```html
<!-- 悪い例：サイト全体 FAQ へのリンクだけ -->
<a href="/faq">FAQ</a>
<!-- 今の設問に関する具体的な支援ではない -->
```

> **参照**
> - [Technique G193: Providing help by an assistant in the web page](https://www.w3.org/WAI/WCAG22/Techniques/general/G193)

## 手法5：スペルチェックと入力候補（G194）

- テキスト入力のスペルチェック、オートコンプリート、入力候補
- 書字障害・読み書き障害のある人の入力支援
- ブラウザや OS の機能、またはアプリ側の実装

```html
<!-- 良い例：スペルチェック有効 -->
<label for="cover-letter">カバーレター</label>
<textarea
  id="cover-letter"
  name="cover-letter"
  spellcheck="true"
  lang="ja"
></textarea>
```

```html
<!-- 良い例：オートコンプリートで入力を補助 -->
<label for="address">住所</label>
<input
  id="address"
  name="address"
  type="text"
  autocomplete="street-address"
>
```

```html
<!-- 悪い例：支援なしの長文入力 -->
<textarea id="cover-letter" spellcheck="false"></textarea>
<!-- スペルチェックも候補もなし -->
```

> **参照**
> - [Technique G194: Providing spell checking and suggestions for text input](https://www.w3.org/WAI/WCAG22/Techniques/general/G194)

## 推奨（必須ではない）手法

- **H89**：`title` 属性で文脈に応じたヘルプを提供
- キーボード・スクリーンリーダー利用者には `aria-describedby` や可視テキストの方が望ましい
- `title` だけに頼る設計は避ける

```html
<!-- 推奨：aria-describedby で関連付け -->
<label for="tax-id">マイナンバー</label>
<input
  id="tax-id"
  name="tax-id"
  type="text"
  aria-describedby="tax-id-hint"
>
<p id="tax-id-hint">12桁の個人番号。ハイフンなし。</p>
```

```html
<!-- 非推奨：title のみ -->
<input id="tax-id" name="tax-id" type="text" title="12桁の個人番号">
```

> **参照**
> - [Technique H89: Using the title attribute to provide context-sensitive help](https://www.w3.org/WAI/WCAG22/Techniques/html/H89)

## よくある失敗例

- ラベルが曖昧なのに、追加ヘルプがない
- ヘルプはあるが、**今の操作**と無関係な一般 FAQ だけ
- ヘルプリンクの存在が分からない（アイコンのみ、コントラスト不足）
- ヘルプを開くと別タブに飛び、フォームの文脈を見失う
- 複雑な設問なのに、形式・例・文字数制限の説明がない
- `title` 属性だけでヘルプを提供（多くのユーザーが気づけない）
- 3.2.6 のサイトヘルプはあるが、フォーム内の文脈別ヘルプがない

```html
<!-- 失敗例 -->
<label for="employer-id">EIN</label>
<input id="employer-id" name="employer-id" type="text">
<!-- 略語の意味・形式・取得方法が不明 -->
```

```html
<!-- 改善例 -->
<label for="employer-id">雇用者識別番号（EIN）</label>
<input
  id="employer-id"
  name="employer-id"
  type="text"
  aria-describedby="ein-hint"
>
<p id="ein-hint">
  米国の雇用者向け税務 ID。XX-XXXXXXX 形式。例：12-3456789
  <a href="#help-ein">EIN の確認方法</a>
</p>
```

## テスト・確認方法

### 手動テスト

- テキスト入力や特定形式が求められるフォームを洗い出す
- 各項目について、ラベルだけで機能が説明できるか確認する
- 説明不足の項目に、文脈に応じたヘルプがあるか確認する
- ヘルプの存在が視覚的・プログラム的に分かるか確認する
- ヘルプを開いても、現在のタスクの文脈を見失わないか確認する
- スペルチェック・形式例・フォーム冒頭説明が適切か確認する

```text
チェックリスト：
1. テキスト入力フォームを特定したか
2. ラベルだけで足りない項目を特定したか
3. 文脈に応じたヘルプ（説明・リンク・例・アシスタント）があるか
4. ヘルプの存在がユーザーに分かるか
5. 必要なときにヘルプを取得できるか
6. 決まった形式の入力に、形式と例があるか（G89）
7. フォーム冒頭に全体説明があるか（G184）
8. title 属性だけに頼っていないか
9. 3.3.2 のラベル・説明と整合しているか
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.3.5 は、**文脈に応じたヘルプ**が利用できる達成基準（レベル AAA）
- 現在実行している操作・入力に関連する支援を提供する
- ラベルだけで十分な項目には追加ヘルプは不要
- フォーム冒頭説明（G184）、形式と例（G89）、項目別ヘルプ（G71）、アシスタント（G193）、スペルチェック（G194）が典型
- 3.3.2（ラベル・説明）や 3.2.6（一貫したヘルプ）と役割が異なる
- ヘルプの存在が分かり、必要なときに取得できることが重要
