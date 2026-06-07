# 1.3.5 入力目的の特定（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.3.5「入力の目的の特定」（Identify Input Purpose）
- レベル **AA**（標準的な適合レベル）
- WCAG **2.1** で追加された達成基準
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.3「適応可能」に属する
- **ユーザー自身**に関する情報を収集する入力欄の**目的**を、プログラム的に特定可能にする
- 主な実装手段は HTML の **`autocomplete` 属性**

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.5 Identify Input Purpose](https://www.w3.org/TR/WCAG22/#identify-input-purpose)
> - [Understanding SC 1.3.5 Identify Input Purpose](https://www.w3.org/WAI/WCAG22/Understanding/identify-input-purpose.html)

## 達成基準の原文（要約）

- **ユーザーに関する情報**を収集する各入力欄について、次の条件を満たすこと
  - 入力欄の目的が WCAG 2.2 **第7章「入力目的」** に定義されたものである
  - 入力データの期待される意味を特定できる技術で実装されている
- 技術が対応していれば、**プログラム的に目的を特定**できる必要がある

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.5](https://www.w3.org/TR/WCAG22/#identify-input-purpose)
> - [WCAG 2.2 — Section 7: Input Purposes for User Interface Components](https://www.w3.org/TR/WCAG22/#input-purposes)

## 対象となる入力欄

### 対象

- **フォームを入力しているユーザー自身**に関する情報を収集する欄
- 具体例
  - 氏名、メールアドレス、電話番号
  - 住所（請求先・配送先）
  - クレジットカード情報
  - 生年月日、ユーザー名、パスワード
- `<input>` だけでなく `<select>`・`<textarea>` も含む

### 対象外

- **他者**に関する情報（緊急連絡先の氏名、贈り先の住所等）
- ユーザーに関係ない入力（検索キーワード、商品数量、コメント本文）
- `transaction-amount` は Input Purposes リストにあるが、**ユーザー自身の情報**でなければ要件なし

```html
<!-- 対象：ログインユーザー自身のメールアドレス -->
<label for="email">メールアドレス</label>
<input type="email" id="email" name="email" autocomplete="email">

<!-- 対象外：他人のメールアドレス（紹介者等） -->
<label for="referrer-email">紹介者のメールアドレス</label>
<input type="email" id="referrer-email" name="referrer-email">
<!-- autocomplete は不要（または off） -->

<!-- 対象外：検索 -->
<label for="search">サイト内検索</label>
<input type="search" id="search" name="q">
```

> **参照**
> - [Understanding SC 1.3.5 — Intent (scoped to user)](https://www.w3.org/WAI/WCAG22/Understanding/identify-input-purpose.html#intent)

## なぜ必要か

- 認知障害・記憶障害のあるユーザーが、ラベルだけでは**入力目的**を理解しにくい場合がある
- `autocomplete` トークンにより、支援技術が**アイコン**等で目的を視覚的に補助できる
- ブラウザの**オートフィル**で入力負担を軽減（運動障害・時間が必要なユーザーにも有益）
- トークンは**言語非依存**。ラベルが母語でなくても目的を一貫して特定可能

> **参照**
> - [Understanding SC 1.3.5 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/identify-input-purpose.html#benefits)

## `type` 属性だけでは足りない理由

- `type="email"` は入力の**種類**（メール形式）を示すが、**目的**（誰のメールか）は示さない
- `type="text"` では氏名・住所・会社名など区別できない
- 1.3.5 では WCAG 第7章の**細かいトークン**で目的を特定する

| 属性 | 示すもの | 例 |
|---|---|---|
| `type="email"` | 入力形式 | メール形式の文字列 |
| `autocomplete="email"` | 入力目的 | **ユーザー自身**のメールアドレス |
| `type="text"` | 汎用テキスト | 区別不可 |
| `autocomplete="given-name"` | 入力目的 | **ユーザー自身**の名（ファーストネーム） |

> **参照**
> - [Understanding SC 1.3.5 — Intent (type vs autocomplete)](https://www.w3.org/WAI/WCAG22/Understanding/identify-input-purpose.html#intent)

## 実装方法（Technique H98）

- HTML の **`autocomplete` 属性**に、WCAG / HTML 仕様で定義された**有効なトークン**を指定
- ラベルの意味と `autocomplete` トークンが**一致**していること

```html
<!-- 悪い例：autocomplete なし -->
<label for="fname">名</label>
<input type="text" id="fname" name="fname">

<!-- 良い例：目的を autocomplete で特定 -->
<label for="fname">名</label>
<input type="text" id="fname" name="fname" autocomplete="given-name">

<label for="lname">姓</label>
<input type="text" id="lname" name="lname" autocomplete="family-name">

<label for="email">メールアドレス</label>
<input type="email" id="email" name="email" autocomplete="email">

<label for="tel">電話番号</label>
<input type="tel" id="tel" name="tel" autocomplete="tel">
```

### よく使う autocomplete トークン

| カテゴリ | トークン | 用途 |
|---|---|---|
| 氏名 | `name` | 氏名全体 |
| 氏名 | `given-name` | 名 |
| 氏名 | `family-name` | 姓 |
| 氏名 | `honorific-prefix` | 敬称（Mr. 等） |
| 認証 | `username` | ユーザー名 |
| 認証 | `new-password` | 新規パスワード |
| 認証 | `current-password` | 現在のパスワード |
| 連絡先 | `email` | メールアドレス |
| 連絡先 | `tel` | 電話番号 |
| 住所 | `street-address` | 番地まで |
| 住所 | `address-level1` | 都道府県 |
| 住所 | `postal-code` | 郵便番号 |
| 住所 | `country-name` | 国名 |
| 支払い | `cc-name` | カード名義 |
| 支払い | `cc-number` | カード番号 |
| 支払い | `cc-exp` | 有効期限 |
| 支払い | `cc-csc` | セキュリティコード |
| その他 | `bday` | 生年月日 |
| その他 | `organization` | 会社名 |
| その他 | `organization-title` | 役職 |

> **参照**
> - [Technique H98: Using HTML autocomplete attributes](https://www.w3.org/WAI/WCAG22/Techniques/html/H98)
> - [MDN — HTML autocomplete attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete)

## 請求先・配送先（section トークン）

- `shipping` / `billing` をトークン先頭に付けて**セクション**を区別
- 同一フォーム内に請求先と配送先の両方がある場合に使用

```html
<!-- 良い例：請求先と配送先を区別 -->
<fieldset>
  <legend>請求先住所</legend>
  <label for="billing-street">番地</label>
  <input id="billing-street" autocomplete="billing street-address">
  <label for="billing-postal">郵便番号</label>
  <input id="billing-postal" autocomplete="billing postal-code">
</fieldset>

<fieldset>
  <legend>配送先住所</legend>
  <label for="shipping-street">番地</label>
  <input id="shipping-street" autocomplete="shipping street-address">
  <label for="shipping-postal">郵便番号</label>
  <input id="shipping-postal" autocomplete="shipping postal-code">
</fieldset>
```

> **参照**
> - [HTML — Autofill detail tokens](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#autofill-detail-tokens)

## 氏名・住所の分割

- 日本語 UI でも、HTML 仕様のトークンに**対応する欄**があれば指定する
- 姓・名を分けるフォーム → `family-name` / `given-name`
- 郵便番号・都道府県・市区町村・番地 → 対応する `address-level*` / `postal-code` 等

```html
<!-- 良い例：日本の住所フォーム -->
<label for="postal">郵便番号</label>
<input id="postal" autocomplete="postal-code">

<label for="pref">都道府県</label>
<select id="pref" autocomplete="address-level1">
  <option value="">選択してください</option>
  <option value="東京都">東京都</option>
</select>

<label for="city">市区町村</label>
<input id="city" autocomplete="address-level2">

<label for="street">番地・建物名</label>
<input id="street" autocomplete="street-address">
```

## 複数の目的が混在する入力欄

- ユーザー名とメールアドレスを**1つの欄**にまとめる場合等
- 技術上複数トークンを指定できない場合、**いずれか一方**を指定するか、**省略**してよい（Understanding の Note）

```html
<!-- ユーザー名またはメールの統合欄：username または email のいずれか -->
<label for="login">ユーザー名またはメールアドレス</label>
<input id="login" autocomplete="username">
```

> **参照**
> - [Understanding SC 1.3.5 — Intent (multiple purposes)](https://www.w3.org/WAI/WCAG22/Understanding/identify-input-purpose.html#intent)

## 代表的な失敗例（Failure F107）

- **無効な** `autocomplete` 値（仕様にない文字列）
- ラベルの意味と**不一致**のトークン
- カスタム文字列を `autocomplete` に使う（`autocomplete="full-name-jp"` 等）

```html
<!-- F107：無効な autocomplete 値 -->
<input autocomplete="fullname">
<input autocomplete="user-email-address">

<!-- F107：ラベルとトークンが不一致 -->
<label for="email">メールアドレス</label>
<input id="email" autocomplete="tel">

<!-- 良い例：仕様どおりのトークン -->
<label for="email">メールアドレス</label>
<input id="email" autocomplete="email">
```

> **参照**
> - [Failure F107: Incorrect autocomplete attribute values](https://www.w3.org/WAI/WCAG22/Techniques/failures/F107)

## `autocomplete="off"` の使いどころ

- **ユーザー自身の情報**を収集する欄では `off` にしない
- `off` が適切な例
  - スパム対策の honeypot 欄
  - 他者に関する情報
  - 検索・コメント等、Input Purposes に該当しない欄
  - ワンタイム認証コード（保存・自動入力不要）

```html
<!-- honeypot：autocomplete off は問題なし -->
<input type="text" name="website" tabindex="-1" autocomplete="off" aria-hidden="true">

<!-- ワンタイムコード -->
<label for="otp">認証コード</label>
<input id="otp" autocomplete="one-time-code">
<!-- one-time-code は HTML 仕様の有効トークン -->
```

> **参照**
> - [other/web-form-email-security.md — honeypot](../other/web-form-email-security.md)

## セキュリティに関する考慮（H98）

- `autocomplete` は**同一ブラウザ・同一ユーザー**のローカル保存が前提
- ユーザーが**オプトイン**で保存を選択する
- フォームは**自動送信されない**。送信前にユーザーが内容を確認できる
- パスワードマネージャー等は `autocomplete` なしでも推測するが、**正確なトークン**で精度が上がる
- 他人の情報を収集する欄に `autocomplete` を付けない（H98 の注意）

> **参照**
> - [Technique H98 — Security considerations](https://www.w3.org/WAI/WCAG22/Techniques/html/H98)

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.3.1 情報および関係性（A） | `<label>` と入力欄の関連付け |
| 3.3.2 ラベルまたは説明（A） | **可視ラベル**の提供（1.3.5 とは別要件） |
| 4.1.2 名前・役割・値（A） | accessible name（ラベル） |

- 1.3.5 は**入力目的のプログラム的特定**（`autocomplete`）
- 3.3.2 は**人間が読めるラベル・説明**
- **両方**必要。ラベルだけでは 1.3.5 未達、`autocomplete` だけでは 3.3.2 未達

```html
<!-- 1.3.1 + 3.3.2 + 1.3.5 を満たす例 -->
<label for="email">メールアドレス</label>
<input type="email" id="email" name="email" autocomplete="email" required>
```

> **参照**
> - [wcag/1-3-1-info-and-relationships.md — 1.3.1](./1-3-1-info-and-relationships.md)
> - [a11y/accessible-form.md — フォームのアクセシビリティ](../a11y/accessible-form.md)

## 将来のメタデータ（WAI-Adapt）

- 支援技術の対応が進めば、WAI-Adapt 等の**シンボル・メタデータ**も利用可能
- 現時点では **HTML `autocomplete`** が主要な sufficient technique（H98）
- ラベルをシンボルに置き換える等、将来の拡張が想定されている

> **参照**
> - [WAI-Adapt: Symbols Module](https://www.w3.org/WAI/WAI-Adapt/)

## テスト・確認方法

### 自動テスト

- ユーザー情報入力欄の `autocomplete` 有無・値の妥当性
- ACT Rule：[Autocomplete attribute has valid value](https://www.w3.org/WAI/standards-guidelines/act/rules/73f2c2/proposed/)
- axe DevTools 等で検出可能

### 手動テスト

- 氏名・メール・電話・住所・クレカ等、**ユーザー自身**の欄に適切なトークンがあるか
- ラベルの意味と `autocomplete` トークンが**一致**しているか
- 他者情報・検索欄に**誤って** `autocomplete` が付いていないか
- ブラウザのオートフィルで**正しい欄**に値が入るか（参考）

```bash
# autocomplete 属性の一覧を確認（開発者ツール Console）
[...document.querySelectorAll('input,select,textarea')].map(el => ({
  tag: el.tagName,
  id: el.id,
  autocomplete: el.autocomplete,
  type: el.type
}));
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.3.5 は**ユーザー自身**の情報を収集する入力欄の**目的**をプログラム的に特定する達成基準（レベル AA、WCAG 2.1 追加）
- 主な実装は **`autocomplete` 属性**（H98）。WCAG 第7章 / HTML 仕様の有効トークンを使用
- `type` 属性だけでは不十分。`given-name` / `email` / `postal-code` 等の**目的トークン**が必要
- 請求先・配送先は `billing` / `shipping` セクション付きトークン
- 無効・不一致な値は **Failure F107**
- 3.3.2（ラベル）・1.3.1（関連付け）と**併せて**確認。`autocomplete` はラベルの代替ではない
