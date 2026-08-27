# 3.3.3 エラー修正の提案（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.3.3「エラー修正の提案」（Error Suggestion）
- レベル **AA**
- 原則3「理解可能（Understandable）」> ガイドライン 3.3「入力支援」に属する
- **入力エラー**が自動検出され、**修正の提案が分かっている**場合、その提案をユーザーに示すこと
- 例外：コンテンツの**セキュリティ**や**目的**を損なう場合
- 3.3.1（エラーの特定）の上に乗る。1つの文で両方を満たすこともある

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.3 Error Suggestion](https://www.w3.org/TR/WCAG22/#error-suggestion)
> - [Understanding SC 3.3.3 Error Suggestion](https://www.w3.org/WAI/WCAG22/Understanding/error-suggestion.html)

## 達成基準の原文（要約）

- 入力エラーが自動検出され、修正の提案が分かっている場合、コンテンツのセキュリティや目的を損なう場合を除き、その提案をユーザーに提供すること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.3](https://www.w3.org/TR/WCAG22/#error-suggestion)

## なぜ必要か

- エラーが分かっても、**どう直すか**が分からないとフォームを諦めやすい
- 認知・学習障害のある人は、修正方法の手がかりがないと入力を繰り返しにくい
- 視覚障害のある人は、正しい形式や許容値を推測しにくい
- 運動障害のある人は、試行回数を減らしたい
- 提案があれば、修正にかかる時間と負担が減る

> **参照**
> - [Understanding SC 3.3.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/error-suggestion.html#benefits)

## 3.3.1 との関係

| 達成基準 | レベル | 焦点 |
|---|---|---|
| 3.3.1 エラーの特定 | A | **どの項目**がエラーか、**何が**問題か |
| 3.3.3 エラー修正の提案 | AA | **どう直すか**の提案（分かっている場合） |

```text
3.3.1 だけ満たす例：
  「メールアドレスが無効です」

3.3.1 と 3.3.3 の両方を満たす例：
  「name@domain.com の形式で有効なメールアドレスを入力してください」

3.3.3 が不要な例：
  修正方法が分からない（サーバー内部エラーなど）
  提案するとセキュリティが損なわれる（パスワードの正解を示すなど）
```

> **参照**
> - [wcag/3-3-1-error-identification.md — 3.3.1 エラーの特定](./3-3-1-error-identification.md)

## 適用される2つの状況

### 状況A：特定のデータ形式が必要

- 日付、メール、電話番号、郵便番号など、決まった形式を求める項目
- 正しい形式の例、説明、似た入力からの変換提案を示す

### 状況B：限られた値の集合から選ぶ

- 都道府県、月名、商品サイズなど、許容値が有限の項目
- 許容値の一覧、最も近い候補の提案を示す

```text
両方に当てはまる例：
  必須の都道府県セレクト（必須 + 有限集合）
  必須の生年月日（必須 + 特定形式）
```

> **参照**
> - [Understanding SC 3.3.3 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/error-suggestion.html)

## 例外：提案しない場合

- **セキュリティ**：パスワードの正解を示す、認証情報を漏らす
- **目的**：提案自体がコンテンツの目的を損なう
- 修正方法が**分からない**場合は、この基準の対象外（3.3.1 は別途必要）

```text
提案しない例：
  「正しいパスワードは ○○ です」
  内部サーバーエラーで原因不明

提案する例：
  「@ を含むメール形式で入力してください」
  「12 と入力されました。『12月』の意味ですか？」
```

## 手法1：形式エラーに正しい例・説明を示す（G85）

- **G85**：形式・値が要求外のとき、正しい入力の例・説明・類似値を示す
- 3.3.1 も同時に満たすことが多い

```html
<!-- 良い例：形式の説明と例 -->
<label for="email">メールアドレス</label>
<input
  id="email"
  name="email"
  type="email"
  aria-invalid="true"
  aria-describedby="email-error"
>
<p id="email-error" role="alert">
  name@domain.com の形式で有効なメールアドレスを入力してください。
  例：taro@example.com
</p>
```

```html
<!-- 悪い例：エラーは分かるが、直し方が分からない -->
<p id="email-error" role="alert">メールアドレスが無効です。</p>
```

```html
<!-- 良い例：日付形式 -->
<label for="birth">生年月日</label>
<input id="birth" name="birth" type="text" aria-describedby="birth-error">
<p id="birth-error" role="alert">
  YYYY-MM-DD の形式で入力してください。例：1990-04-01
</p>
```

> **参照**
> - [Technique G85: Providing a text description when user input falls outside the required format or values](https://www.w3.org/WAI/WCAG22/Techniques/general/G85)

## 手法2：許容値外の入力に候補を示す（G84）

- **G84**：許容値リスト外の入力に、許容値であることと、可能なら一覧や最も近い候補を示す
- 月名に「12」と入力 → 「December の意味ですか？」または月名一覧

```html
<!-- 良い例：有限集合から選ぶ -->
<label for="month">月</label>
<input id="month" name="month" type="text" aria-describedby="month-error">
<p id="month-error" role="alert">
  次のいずれかを入力してください：
  January, February, March, April, May, June,
  July, August, September, October, November, December。
  「12」と入力されました。『December』の意味ですか？
</p>
```

```html
<!-- より良い例：候補から選ばせる -->
<label for="state">都道府県</label>
<input id="state" name="state" type="text" aria-describedby="state-error">
<p id="state-error" role="alert">
  「東京」と入力されました。次のいずれかを選んでください：
</p>
<ul>
  <li><button type="button">東京都</button></li>
  <li><button type="button">東京（廃止）</button></li>
</ul>
```

```html
<!-- 悪い例：許容値外だとだけ言う -->
<p id="state-error">無効な都道府県です。</p>
```

> **参照**
> - [Technique G84: Providing a text description when the user provides information that is not in the list of allowed values](https://www.w3.org/WAI/WCAG22/Techniques/general/G84)

## 手法3：修正候補のテキストを提案する（G177）

- **G177**：正しいテキストが推定できるとき、修正候補を提示する
- フィールド横、フォーム先頭、リンク先一覧など、近くに置く
- チェックボックス・ラジオで候補を選ばせる方法もある

```html
<!-- 良い例：期間のあいまいな入力 -->
<label for="duration">期間</label>
<input id="duration" name="duration" type="text" value="6" aria-describedby="duration-error">
<p id="duration-error" role="alert">
  エラーを検出しました。次のいずれかを意図していましたか？
</p>
<fieldset>
  <legend>期間の候補</legend>
  <input id="dur-days" name="duration-fix" type="radio" value="6 days">
  <label for="dur-days">6 days</label>
  <input id="dur-weeks" name="duration-fix" type="radio" value="6 weeks">
  <label for="dur-weeks">6 weeks</label>
  <input id="dur-months" name="duration-fix" type="radio" value="6 months">
  <label for="dur-months">6 months</label>
</fieldset>
```

```html
<!-- 良い例：スペルチェック風の提案 -->
<p role="alert">
  「Kohl」の検索結果：
  <a href="/search?q=Kohl+Center">Kohl Center</a>,
  <a href="/search?q=Kohl%27s+East">Kohl's Dept. Store-East</a>
</p>
```

> **参照**
> - [Technique G177: Providing suggested correction text](https://www.w3.org/WAI/WCAG22/Techniques/general/G177)

## 3.3.2 との関係

- 3.3.2（ラベルまたは説明）は、**入力前**の手がかり
- 3.3.3 は、**エラー検出後**の修正提案
- 事前に形式・必須を説明しておくと、エラー自体を減らせる
- それでもエラーになったとき、3.3.3 で直し方を示す

```html
<!-- 入力前（3.3.2） -->
<label for="email">メールアドレス</label>
<input id="email" type="email" aria-describedby="email-hint">
<p id="email-hint">例：taro@example.com</p>

<!-- エラー後（3.3.3） -->
<p id="email-error" role="alert">
  name@domain.com の形式で入力してください。例：taro@example.com
</p>
```

> **参照**
> - [wcag/3-3-2-labels-or-instructions.md — 3.3.2 ラベルまたは説明](./3-3-2-labels-or-instructions.md)

## よくある失敗例

- 「入力エラーがあります」だけで、直し方がない
- 赤枠・アイコンだけで、具体的な提案がない
- 許容値があるのに、一覧や候補を示さない
- パスワードの正解を示してしまう（セキュリティ例外）
- 提案がエラー項目から遠く、関連付けがない
- ブラウザ既定の汎用メッセージだけに頼る

```html
<!-- 失敗例 -->
<p role="alert">入力内容に誤りがあります。</p>
```

```html
<!-- 改善例 -->
<p id="zip-error" role="alert">
  郵便番号は 123-4567 の形式で入力してください。
  例：100-0001
</p>
```

```jsx
// 良い例：React で具体的な提案
function EmailError({ value }) {
  return (
    <p id="email-error" role="alert">
      「{value}」は有効なメール形式ではありません。
      name@domain.com の形式で入力してください。
    </p>
  );
}
```

## テスト・確認方法

### 手動テスト

- 形式が決まっている項目に、意図的に不正な値を入力して送信する
- 有限集合の項目に、リスト外の値を入力して送信する
- エラーメッセージに、具体的な修正提案があるか確認する
- 提案が分からないケースでは、無理に提案していないか確認する
- セキュリティ上、提案してはいけない項目（パスワード等）を確認する

```text
チェックリスト：
1. エラー時、3.3.1 の特定・説明があるか
2. 修正方法が分かるなら、具体的な提案があるか
3. 形式エラーなら、正しい例・説明があるか
4. 許容値外なら、一覧や最も近い候補があるか
5. 提案はエラー項目の近くにあるか
6. セキュリティ・目的の例外に当たらないか
7. 事前説明（3.3.2）と重複していても問題ないか
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.3.3 は、入力エラーが検出され、修正提案が分かっている場合に、その提案を示す達成基準（レベル AA）
- 3.3.1 は「何が」問題か、3.3.3 は「どう直すか」
- 形式エラーは正しい例・説明、許容値外は一覧や候補の提示が典型
- 提案が分からない、またはセキュリティ・目的上示せない場合は対象外
- 1つのエラーメッセージで 3.3.1 と 3.3.3 の両方を満たすことも多い
