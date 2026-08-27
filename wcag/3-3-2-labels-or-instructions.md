# 3.3.2 ラベルまたは説明（A）

## 達成基準の概要

- WCAG 2.2 達成基準 3.3.2「ラベルまたは説明」（Labels or Instructions）
- レベル **A**（最低限の適合レベル）
- 原則3「理解可能（Understandable）」> ガイドライン 3.3「入力支援」に属する
- ユーザー入力を求めるコンテンツには、**ラベルまたは説明**を提供すること
- 必須・任意を問わず、入力を受け付けるすべてのフォーム項目が対象
- ラベルは**すべてのユーザーに見える**必要がある（`aria-label` だけでは足りないことがある）

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.2 Labels or Instructions](https://www.w3.org/TR/WCAG22/#labels-or-instructions)
> - [Understanding SC 3.3.2 Labels or Instructions](https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html)

## 達成基準の原文（要約）

- コンテンツがユーザー入力を求める場合、ラベルまたは説明を提供すること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.2](https://www.w3.org/TR/WCAG22/#labels-or-instructions)

## なぜ必要か

- 認知・言語・学習障害のある人は、入力すべき内容が分からないと誤入力しやすい
- 期待する形式や必須項目が分かれば、送信エラーを減らせる
- ラジオ・チェック・セレクトは、各選択肢のラベルがないと何を選んでいるか分からない
- すべてのユーザーが、支援技術なしでも目的を把握できる必要がある
- 情報過多も有害。タスクに必要な手がかりに絞る

> **参照**
> - [Understanding SC 3.3.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html#benefits)

## 対象になるもの / ならないもの

- 対象：テキスト入力、チェックボックス、ラジオ、セレクト、ファイル選択など、**データ入力**を伴うコントロール
- 「requires」は必須項目だけを意味しない。入力を受け付ける項目すべてが対象
- 対象外：リンク、開閉ウィジェットなど、データ入力と無関係な操作

```text
対象の例：
  氏名・メール・検索フィールド
  ラジオ・チェックの各選択肢
  日付・電話番号・郵便番号
  必須・任意の両方

対象外の例：
  「詳細を見る」リンク
  アコーディオンの開閉ボタン
  送信・リセットボタン（ラベルは value / テキストで足りる）
```

> **参照**
> - [Understanding SC 3.3.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html#intent)

## 関連する達成基準との違い

| 達成基準 | 焦点 |
|---|---|
| 3.3.2 ラベルまたは説明 | **見える**ラベル・説明があるか |
| 1.3.1 情報及び関係性 | ラベルとコントロールの**関連付け**がプログラムで判定できるか |
| 4.1.2 名前・役割・値 | **アクセシブルネーム**があるか（`aria-label` でも可） |
| 2.4.6 見出し及びラベル | ラベルが**十分に説明的**か |

- 3.3.2 を満たしても、マークアップが不適切なら 1.3.1 は不合格になりうる
- `aria-label` だけで支援技術に名前が届いても、見えるラベルがなければ 3.3.2 は不合格になりうる
- ラベルの質（曖昧さ）は 2.4.6 の範囲

```html
<!-- 4.1.2 は満たしうるが、3.3.2 は不十分な例 -->
<input type="text" aria-label="メールアドレス">
<!-- 画面上にラベルが見えない -->
```

```html
<!-- 3.3.2 と 1.3.1 / 4.1.2 を同時に満たす例 -->
<label for="email">メールアドレス</label>
<input id="email" name="email" type="email">
```

> **参照**
> - [wcag/1-3-1-info-and-relationships.md — 1.3.1 情報及び関係性](./1-3-1-info-and-relationships.md)
> - [wcag/2-4-6-headings-and-labels.md — 2.4.6 見出し及びラベル](./2-4-6-headings-and-labels.md)

## ラベルとは

- ウェブコンテンツ内の部品を識別するために、ユーザーに提示されるテキスト（またはテキストによる代替）
- HTML の `<label>` に限らない
- **ラベル**はすべてのユーザーに提示される。**名前（name）** は支援技術だけに露出する場合がある
- 画像ラベルも可だが、想定読者に広く伝わるか注意する

## 手法1：label 要素で関連付ける（H44 + G131）

- **G131**：説明的なラベルを提供する
- **H44**：`<label for>` とコントロールの `id` を一致させる
- 3.3.2 では、ラベルが**見える**こと
- テキスト入力はラベルを前に、チェック・ラジオはラベルを後に置くのが一般的
- ラベルをクリックしてもフォーカスできるため、運動障害のある人にも有利

```html
<!-- 良い例：テキスト入力 -->
<label for="firstname">名</label>
<input id="firstname" name="firstname" type="text" autocomplete="given-name">
```

```html
<!-- 良い例：チェックボックス -->
<input id="agree" name="agree" type="checkbox">
<label for="agree">利用規約に同意する</label>
```

```html
<!-- 悪い例：プレースホルダーだけ -->
<input type="email" placeholder="メールアドレス">
```

```html
<!-- 悪い例：見た目だけの文字列で関連付けなし -->
<span>メールアドレス</span>
<input type="email" name="email">
```

> **参照**
> - [Technique H44: Using label elements to associate text labels with form controls](https://www.w3.org/WAI/WCAG22/Techniques/html/H44)
> - [Technique G131: Providing descriptive labels](https://www.w3.org/WAI/WCAG22/Techniques/general/G131)

## 手法2：グループに fieldset / legend（H71）

- **H71**：関連するコントロール群を `<fieldset>` + `<legend>` で説明する
- 電話番号を複数欄に分けるとき、括弧やハイフンだけではラベルにならない（F82）
- ラジオボタン群にも有効

```html
<!-- 良い例：電話番号のグループ -->
<fieldset>
  <legend>電話番号</legend>
  <label for="area">市外局番</label>
  <input id="area" name="area" type="text" inputmode="numeric" maxlength="4">
  <label for="local">市内局番</label>
  <input id="local" name="local" type="text" inputmode="numeric" maxlength="4">
  <label for="number">加入者番号</label>
  <input id="number" name="number" type="text" inputmode="numeric" maxlength="4">
</fieldset>
```

```html
<!-- 悪い例：視覚的な整形だけでテキストラベルがない（F82） -->
(
<input name="area" maxlength="3">
)
-
<input name="exchange" maxlength="3">
-
<input name="line" maxlength="4">
```

```html
<!-- 良い例：ラジオ群 -->
<fieldset>
  <legend>配送方法</legend>
  <input id="standard" name="shipping" type="radio" value="standard">
  <label for="standard">通常便</label>
  <input id="express" name="shipping" type="radio" value="express">
  <label for="express">速達</label>
</fieldset>
```

> **参照**
> - [Technique H71: Providing a description for groups of form controls using fieldset and legend elements](https://www.w3.org/WAI/WCAG22/Techniques/html/H71)
> - [Failure F82: visually formatting a set of phone number fields but not including a text label](https://www.w3.org/WAI/WCAG22/Techniques/failures/F82)

## 手法3：形式・必須の説明を付ける（G89 / G184 / H90）

- **G89**：期待するデータ形式と例を示す
- **G184**：フォーム先頭で必要な入力を説明する
- **H90**：必須項目を label / legend で示す
- 特殊な形式や厳しいルールがあるときほど、事前の説明が重要

```html
<!-- 良い例：形式の説明 -->
<label for="date">生年月日</label>
<input id="date" name="date" type="text" aria-describedby="date-hint">
<p id="date-hint">例：1990-04-01（YYYY-MM-DD）</p>
```

```html
<!-- 良い例：必須の明示 -->
<label for="username">ユーザー名（必須）</label>
<input id="username" name="username" type="text" required aria-required="true">
<p>半角英数字とハイフンのみ。3〜20文字。</p>
```

```html
<!-- 悪い例：厳格なルールがあるのに説明なし -->
<label for="username">ユーザー名</label>
<input id="username" name="username" type="text">
<!-- 使える文字が分からず、送信後に初めてエラーになる -->
```

> **参照**
> - [Technique G89: Providing expected data format and example](https://www.w3.org/WAI/WCAG22/Techniques/general/G89)
> - [Technique G184: Providing text instructions at the beginning of a form or set of fields](https://www.w3.org/WAI/WCAG22/Techniques/general/G184)
> - [Technique H90: Indicating required form controls using label or legend](https://www.w3.org/WAI/WCAG22/Techniques/html/H90)

## 手法4：隣接ボタンで目的を示す（G167）

- 検索欄など、隣接するボタンがフィールドの目的を示す場合がある
- 単独の検索アイコンをラベルにする例もあるが、テキストラベルの方が明確

```html
<!-- 許容されうる例：隣接ボタンがラベルの役割 -->
<input id="site-search" name="q" type="search">
<button type="submit">検索</button>
```

```html
<!-- より望ましい例 -->
<label for="site-search">サイト内検索</label>
<input id="site-search" name="q" type="search">
<button type="submit">検索</button>
```

> **参照**
> - [Technique G167: Using an adjacent button to label the purpose of a field](https://www.w3.org/WAI/WCAG22/Techniques/general/G167)

## 氏名を分けるとき

- 「氏名」1ラベルで2つの入力があると、2つ目がラベルなしに見える
- 「姓」「名」など、それぞれに明示的なラベルを付ける

```html
<!-- 良い例 -->
<label for="family">姓</label>
<input id="family" name="family" type="text" autocomplete="family-name">
<label for="given">名</label>
<input id="given" name="given" type="text" autocomplete="given-name">
```

```html
<!-- 悪い例 -->
<label>氏名</label>
<input name="family" type="text">
<input name="given" type="text">
```

## よくある失敗例

- プレースホルダーだけをラベル代わりにする
- `aria-label` だけで、画面上にラベルがない
- 電話番号を括弧・ハイフンの見た目だけで識別させる（F82）
- 「氏名」1つで複数欄をまとめる
- 必須や形式のルールを、送信後のエラーまで隠す
- 情報を詰め込みすぎて、かえって分かりにくくする

```html
<!-- 失敗例 -->
<input type="tel" placeholder="電話番号">
```

```html
<!-- 改善例 -->
<label for="tel">電話番号</label>
<input id="tel" name="tel" type="tel" autocomplete="tel" aria-describedby="tel-hint">
<p id="tel-hint">ハイフンなしの数字で入力してください。</p>
```

## テスト・確認方法

### 手動テスト

- すべての入力コントロールに、見えるラベルまたは説明があるか確認する
- ラジオ・チェックの各選択肢にもラベルがあるか確認する
- 複数欄のグループに、グループ全体の説明があるか確認する
- 特殊な形式・必須がある項目に、事前説明があるか確認する
- プレースホルダーや `aria-label` だけになっていないか確認する

```text
チェックリスト：
1. 各入力に見えるラベル／説明があるか
2. label とコントロールは関連付けられているか（1.3.1）
3. 必須・形式の説明は入力前に分かるか
4. 電話番号などの分割フィールドにグループラベルがあるか
5. プレースホルダーだけ／aria-label だけになっていないか
6. 情報過多でかえって分かりにくくないか
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.3.2 は、ユーザー入力を求めるコントロールに、**見えるラベルまたは説明**を求める達成基準（レベル A）
- 必須・任意を問わず、データ入力を受け付ける項目が対象
- `<label>`、`fieldset` / `legend`、形式・必須の説明が典型
- `aria-label` やプレースホルダーだけでは、多くの場合不十分
- 関連付けは 1.3.1、名前は 4.1.2、ラベルの質は 2.4.6 と役割を分ける
