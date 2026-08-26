# 3.3.1 エラーの特定（A）

## 達成基準の概要

- WCAG 2.2 達成基準 3.3.1「エラーの特定」（Error Identification）
- レベル **A**（最低限の適合レベル）
- 原則3「理解可能（Understandable）」> ガイドライン 3.3「入力支援」に属する
- **入力エラー**が自動検出された場合、**どの項目がエラーか**を特定し、**テキストで説明**すること
- 色やアイコンだけでは足りない。テキスト（またはテキストによる代替）が必要
- 修正方法の提案は 3.3.3 の範囲。1つの文で両方を満たすこともある

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.1 Error Identification](https://www.w3.org/TR/WCAG22/#error-identification)
> - [Understanding SC 3.3.1 Error Identification](https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html)

## 達成基準の原文（要約）

- 入力エラーが自動検出された場合、エラーになっている項目を特定し、エラーをユーザーにテキストで説明すること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.1](https://www.w3.org/TR/WCAG22/#error-identification)

## なぜ必要か

- 色だけのエラー表示は、色覚特性のある人には伝わらない
- スクリーンリーダー利用者は、テキストでないとエラー内容を把握しにくい
- 送信に失敗したのに、フォームを再表示するだけでは、失敗に気づけない
- 認知・言語・学習障害のある人は、**何が**間違っているか分からないと修正できない
- どの項目が問題か特定できれば、修正に集中できる

> **参照**
> - [Understanding SC 3.3.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html#benefits)

## 入力エラーとは

- ユーザーが入力した情報が、ページ側で**受け付けられない**状態
- **必須項目の未入力**
- **形式・値が要求と合わない**入力（メール形式、数値のみ、範囲外など）

```text
入力エラーの例：
  必須のメールアドレスが空
  存在しない郵便番号
  電話番号欄に英字を入力
  未来の生年月日
  最低入札額を下回る金額

入力エラーではない例：
  送信ボタンを押していない（まだ検証前）
  任意項目が空
```

- ブラウザが値を自動補正した場合も、**変更内容をテキストで伝える**必要がある

> **参照**
> - [Understanding SC 3.3.1 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html#intent)

## 3.3.3 との関係

| 達成基準 | レベル | 焦点 |
|---|---|---|
| 3.3.1 エラーの特定 | A | **どの項目**がエラーか、**何が**問題か（テキスト） |
| 3.3.3 エラー修正の提案 | AA | **どう直すか**の提案（可能な場合） |

```text
3.3.1 だけ満たす例：
  「メールアドレスが無効です」

3.3.1 と 3.3.3 の両方を満たす例：
  「name@domain.com の形式で有効なメールアドレスを入力してください」
```

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.3 Error Suggestion](https://www.w3.org/TR/WCAG22/#error-suggestion)

## 手法1：必須項目の未入力をテキストで示す（G83）

- **G83**：必須項目が未入力のとき、テキストでどの項目か説明する
- フォーム先頭の一覧、各フィールド横のメッセージ、どちらでもよい
- 送信失敗を `<title>` やアラートでも伝えると、スクリーンリーダー利用者に届きやすい
- 以前入力した値は、可能なら再表示する（パスワードなど除く）

```html
<!-- 良い例：フォーム先頭にエラー一覧 -->
<form action="/signup" method="post" novalidate>
  <div role="alert" id="form-errors">
    <h2>入力内容を確認してください</h2>
    <ul>
      <li><a href="#email">メールアドレス：必須項目です</a></li>
      <li><a href="#name">お名前：必須項目です</a></li>
    </ul>
  </div>

  <label for="email">メールアドレス（必須）</label>
  <input id="email" name="email" type="email" aria-invalid="true" aria-describedby="email-error">
  <p id="email-error">必須項目です。</p>

  <label for="name">お名前（必須）</label>
  <input id="name" name="name" type="text" aria-invalid="true" aria-describedby="name-error">
  <p id="name-error">必須項目です。</p>

  <button type="submit">登録</button>
</form>
```

```html
<!-- 悪い例：赤枠だけ、テキスト説明なし -->
<input id="email" name="email" type="email" class="error-border">
```

> **参照**
> - [Technique G83: Providing text descriptions to identify required fields that were not completed](https://www.w3.org/WAI/WCAG22/Techniques/general/G83)

## 手法2：形式・値のエラーをテキストで示す（G85）

- **G85**：形式や許容値外の入力に、テキストで説明する
- どの項目が問題か、何が受け付けられなかったかを書く

```html
<!-- 良い例：形式エラーをテキストで説明 -->
<label for="phone">電話番号（数字のみ）</label>
<input
  id="phone"
  name="phone"
  type="tel"
  inputmode="numeric"
  pattern="[0-9]+"
  aria-invalid="true"
  aria-describedby="phone-error"
>
<p id="phone-error">数字のみ入力してください。</p>
```

```html
<!-- 悪い例：汎用メッセージだけ -->
<p>入力内容に誤りがあります。</p>
<!-- どの項目か分からない -->
```

> **参照**
> - [Technique G85: Providing a text description when user input falls outside the required format or values](https://www.w3.org/WAI/WCAG22/Techniques/general/G85)

## 手法3：aria-invalid と aria-describedby（ARIA21）

- **ARIA21**：`aria-invalid="true"` でエラー項目を示す
- エラーメッセージは `aria-describedby` で関連付ける
- 検証前に `aria-invalid="true"` を付けない
- エラー解消後は `false` に戻すか、属性を外す

```html
<!-- 良い例 -->
<label for="pin">暗証番号（4桁）</label>
<input
  id="pin"
  name="pin"
  type="text"
  maxlength="4"
  aria-invalid="true"
  aria-describedby="pin-error"
>
<p id="pin-error">暗証番号は4桁の数字で入力してください。</p>
```

```html
<!-- 悪い例：aria-invalid だけ、説明テキストなし -->
<input id="pin" aria-invalid="true">
```

```jsx
// 良い例：React でエラー状態を付ける
function EmailField({ error }) {
  return (
    <>
      <label htmlFor="email">メールアドレス</label>
      <input
        id="email"
        name="email"
        type="email"
        aria-invalid={error ? "true" : undefined}
        aria-describedby={error ? "email-error" : undefined}
      />
      {error && (
        <p id="email-error" role="alert">
          {error}
        </p>
      )}
    </>
  );
}
```

> **参照**
> - [Technique ARIA21: Using aria-invalid to Indicate An Error Field](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA21)
> - [Technique ARIA19: Using ARIA role=alert or Live Regions to Identify Errors](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA19)

## HTML ネイティブ検証の注意

- `required` や `type="email"` などのブラウザ既定検証は、**アクセシビリティ対応（accessibility supported）** か慎重に判断する
- 既定メッセージは汎用的で、3.3.3 を満たさないことが多い
- 複数エラーがあると、**最初の1件だけ**が読み上げられることがある
- 拡大表示でも、検証メッセージが小さく見えにくい
- 実務では `novalidate` で既定検証を止め、カスタムのテキストエラーを出すことが多い

```html
<!-- カスタム検証に切り替える例 -->
<form novalidate action="/submit" method="post">
  <!-- 独自のエラーメッセージを DOM に出す -->
</form>
```

> **参照**
> - [Understanding SC 3.3.1 — User agent native HTML form validation](https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html#intent)

## エラー表示の置き方

- 先頭に一覧、インライン、ダイアログ、どれでもよい
- 必須は**テキスト**での説明
- 色・背景・アイコンは**追加**として使ってよい

```text
表示パターン：
  フォーム先頭のエラー一覧 + 各項目のインライン説明（推奨）
  各項目の横だけ
  role="alert" のライブリージョン
  aria-alertdialog（重大なエラー）
```

```html
<!-- 良い例：色 + テキストの併用 -->
<input
  id="zip"
  name="zip"
  aria-invalid="true"
  aria-describedby="zip-error"
  class="field-error"
>
<p id="zip-error" class="error-message">
  存在しない郵便番号です。
</p>
```

## よくある失敗例

- 送信失敗後、フォームだけ再表示してエラー表示がない
- 赤枠・赤文字だけで、テキスト説明がない
- 「入力エラーがあります」だけで、**どの項目**か分からない
- `aria-invalid` だけ付けて、エラーメッセージがない
- ラベル文言だけ変えて、スクリーンリーダーが気づかない
- ブラウザ既定の検証メッセージだけに頼る

```html
<!-- 失敗例 -->
<form>
  <input class="is-invalid" type="email" name="email">
  <button type="submit">送信</button>
</form>
```

```html
<!-- 改善例 -->
<form novalidate>
  <div role="alert">メールアドレス：必須項目です。</div>
  <label for="email">メールアドレス</label>
  <input
    id="email"
    name="email"
    type="email"
    aria-invalid="true"
    aria-describedby="email-error"
  >
  <p id="email-error">必須項目です。</p>
  <button type="submit">送信</button>
</form>
```

## テスト・確認方法

### 手動テスト

- 必須項目を空のまま送信する
- 形式が違う値を入力して送信する
- エラー項目が特定できるテキストがあるか確認する
- 色だけでなく、スクリーンリーダーでも内容が分かるか確認する
- 複数エラーがあるとき、すべて説明されているか確認する

```text
チェックリスト：
1. 送信失敗時、テキストでエラーが伝わるか
2. どの項目がエラーか特定できるか
3. 何が問題か（未入力 / 形式不正など）説明されているか
4. aria-invalid と aria-describedby が適切か
5. 色・アイコンだけに頼っていないか
6. ブラウザ既定検証だけになっていないか
7. 以前の入力値は再表示されているか（パスワード除く）
```

> **参照**
> - [ACT Rule — Error message describes invalid form field value](https://www.w3.org/WAI/standards-guidelines/act/rules/36b590/)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.3.1 は、入力エラーが検出されたら、**項目の特定**と**テキストでの説明**を求める達成基準（レベル A）
- 色や枠線だけでは不十分。テキスト（またはテキストによる代替）が必須
- `aria-invalid` + `aria-describedby` + インライン／一覧のエラーメッセージが典型
- 修正方法の提案は 3.3.3。1文で両方満たすこともある
- ブラウザ既定検証だけに頼るのは、複数エラーや読み上げで不十分になりやすい
