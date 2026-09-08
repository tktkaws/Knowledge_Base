# 3.3.7 冗長な入力（A）

## 達成基準の概要

- WCAG 2.2 達成基準 3.3.7「冗長な入力」（Redundant Entry）
- レベル **A**
- WCAG **2.2 で新規追加**（`[New]`）
- 原則3「理解可能（Understandable）」> ガイドライン 3.3「入力支援」に属する
- **同一プロセス内**で、以前に入力・提供された情報を再度求めないこと
- 再入力が必要な場合は、次のいずれかで対応する
  1. **自動入力（auto-populated）**
  2. **ユーザーが選択できる形で提示（available for the user to select）**
- ブラウザのオートコンプリートだけに頼るのは不十分（**コンテンツ側**が前回入力を提供する必要がある）

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.7 Redundant Entry](https://www.w3.org/TR/WCAG22/#redundant-entry)
> - [Understanding SC 3.3.7 Redundant Entry](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html)

## 達成基準の原文（要約）

- 同一プロセス内で、以前にユーザーが入力した、またはユーザーに提供された情報を再度入力させる場合、次のいずれかであること
  - 自動入力される
  - ユーザーが選択できる
- 次の場合は例外
  - 再入力が**本質的（essential）**である
  - コンテンツの**セキュリティ**のために必要である
  - 以前に入力した情報が**もはや有効でない**

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.7](https://www.w3.org/TR/WCAG22/#redundant-entry)

## なぜ必要か

- 認知・学習障害のある人は、直前に入力した情報を思い出しにくい
- 多段階プロセスでは誰でも疲労が進み、短期記憶への負荷が高まる
- 再入力を強いると、途中離脱や誤入力が増える
- スイッチ操作・音声入力など、テキスト入力コストが高い人も負担が減る
- 同じ情報を二度書かせないだけで、完了率が上がる

> **参照**
> - [Understanding SC 3.3.7 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#intent)
> - [Understanding SC 3.3.7 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#benefits)

## 「同一プロセス」とは

- **プロセス**：目的を完了するために必要な一連のユーザー操作
- 例：商品選択 → 配送情報 → 支払い → 注文確定
- セッションを閉じて戻ってきた場合や、別活動に移った場合は対象外
- 一方、チェックアウト中の**サードパーティ決済**など、ドメインをまたいでも同一プロセスなら対象になりうる
- **セッション間での永続保存**までを要求するものではない

```text
対象になる例：
  会員登録のステップ1〜3で同じ住所を何度も手入力させる
  配送先入力後、請求先で同じ住所を白紙から再入力させる
  エラー再表示時に、入力済みフィールドをすべてクリアする

対象外になりやすい例：
  昨日入力した内容を、今日の別セッションで覚えている必要
  履歴書ファイルのアップロード（別手段での提供）
  別タスクとして後日やり直す申し込み
```

> **参照**
> - [Understanding SC 3.3.7 — Intent（process）](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#intent)
> - [WCAG 2.2 — Glossary: process](https://www.w3.org/TR/WCAG22/#dfn-processes)

## 「選択できる」とは

- 自動入力が難しい場合の代替手段
- Understanding では次のような形が想定されている
  - ドロップダウンから前回値を選ぶ
  - ページ上に表示されたテキストをコピーして入力する
  - 「配送先と同じ」チェックで請求先を埋める
- 選択可能なデータは**同じページ上**にある必要がある
- 初期表示で見えていて入力欄の近くにあるのが理想
- show/hide コンポーネント内にあってもよい

```text
acceptable な提示例：
  前回の社員 ID をデフォルト表示し、変更も可能
  「請求先は配送先と同じ」チェックボックス
  直前ステップの会社名・住所を一覧から選択

不十分になりやすい例：
  ブラウザのオートフィルだけに依存
  別ページに戻らないと前回値が分からない
  「さっき入力した内容を思い出して書いてください」だけ
```

> **参照**
> - [Understanding SC 3.3.7 — Intent（available to select）](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#intent)

## 例外

| 例外 | 内容 | 例 |
|---|---|---|
| **本質的（essential）** | 再入力しないと機能の意味が崩れる | 記憶ゲームで前回の答えを自動入力しない |
| **セキュリティ** | 表示・コピーを防ぐ必要がある情報 | パスワード作成時の再入力確認 |
| **無効になった情報** | 以前の値がもう使えない | 期限切れクーポン、変更済み住所の再確認 |

- パスワードの再入力確認は、セキュリティ例外として許容されることが多い
- 3.3.8（アクセシブルな認証）とは別基準。パスワードのオートフィル許可は 3.3.8 側の話
- プライバシー例外は明記されていないが、一時保存時は個人情報保護に配慮する

> **参照**
> - [Understanding SC 3.3.7 — Exceptions](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#exceptions)

## 十分な手法

| 手法 | 内容 |
|---|---|
| **G221** | プロセスの前ステップのデータを提供する |
| （将来の手法候補） | 同じ情報を二度求めない |

> **参照**
> - [Understanding SC 3.3.7 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#techniques)
> - [Technique G221: Provide data from a previous step in a process](https://www.w3.org/WAI/WCAG22/Techniques/general/G221)

## 手法1：チェックで前回値を自動入力する（G221）

- EC の「請求先は配送先と同じ」が典型
- ユーザー操作をトリガーに、前ステップの値を埋める

```html
<!-- 良い例：配送先と同じ住所をコピー -->
<fieldset>
  <legend>請求先住所</legend>
  <input
    id="same-as-shipping"
    name="same_as_shipping"
    type="checkbox"
  >
  <label for="same-as-shipping">配送先住所と同じ</label>

  <label for="billing-address">住所</label>
  <input id="billing-address" name="billing_address" type="text">
  <!-- チェック時に配送先の値をコピー（JS） -->
</fieldset>
```

```html
<!-- 悪い例：白紙から再入力 -->
<fieldset>
  <legend>請求先住所</legend>
  <label for="billing-address">住所</label>
  <input id="billing-address" name="billing_address" type="text">
  <!-- 直前に入力した配送先を再利用できない -->
</fieldset>
```

> **参照**
> - [Technique G221 — Example 1](https://www.w3.org/WAI/WCAG22/Techniques/general/G221)
> - [Understanding SC 3.3.7 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#examples)

## 手法2：前ステップの値をデフォルト表示する（G221）

- 確認・変更が必要な場面で、前回値を初期値にする
- 変更は可能にしておく

```html
<!-- 良い例：前ステップの社員 ID をデフォルト表示 -->
<label for="employee-id">利用者の社員 ID</label>
<input
  id="employee-id"
  name="employee_id"
  type="text"
  value="A12345"
  aria-describedby="employee-id-hint"
>
<p id="employee-id-hint">
  ステップ1で入力した ID を初期表示しています。必要なら変更できます。
</p>
```

```html
<!-- 悪い例：確認ステップで空欄 -->
<label for="employee-id">利用者の社員 ID を再入力してください</label>
<input id="employee-id" name="employee_id" type="text">
```

```html
<!-- 良い例：検索結果ページで検索語を保持 -->
<label for="q">検索</label>
<input id="q" name="q" type="search" value="ワイヤレスイヤホン">
```

> **参照**
> - [Technique G221 — Example 2](https://www.w3.org/WAI/WCAG22/Techniques/general/G221)
> - [Understanding SC 3.3.7 — Examples: search results](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#examples)

## 手法3：前ステップの情報を選択できる形で示す（G221）

- 自動入力が難しい場合、同じページ上で選べるようにする

```html
<!-- 良い例：直前に作成した事業情報を選択 -->
<fieldset>
  <legend>ユーザープロフィールに紐づける事業</legend>
  <label>
    <input type="radio" name="business" value="biz-1" checked>
    山田商事（東京都千代田区… / 法人番号 1234567890123）
  </label>
  <label>
    <input type="radio" name="business" value="other">
    別の事業を入力する
  </label>
</fieldset>
```

```html
<!-- 悪い例：作成直後なのに再度すべて手入力 -->
<label for="corp-id">法人番号</label>
<input id="corp-id" name="corp_id" type="text">
<label for="corp-name">会社名</label>
<input id="corp-name" name="corp_name" type="text">
<!-- ステップ1で入力済みの情報を思い出せ、という設計 -->
```

> **参照**
> - [Technique G221 — Example 3](https://www.w3.org/WAI/WCAG22/Techniques/general/G221)

## 手法4：エラー後も入力内容を保持する

- 送信エラーでページが戻っても、入力済み値を消さない
- Understanding の例でも、カード番号などをクリアしないことが示されている

```html
<!-- 良い例：エラー後も値を保持 -->
<form action="/checkout" method="post">
  <div role="alert">カード番号の形式が正しくありません</div>

  <label for="card">カード番号</label>
  <input
    id="card"
    name="card"
    type="text"
    inputmode="numeric"
    value="4111111111111111"
    aria-invalid="true"
  >

  <label for="name">名義</label>
  <input id="name" name="name" type="text" value="TARO YAMADA">
  <!-- 他の項目も保持 -->
</form>
```

```html
<!-- 悪い例：エラーで全クリア -->
<form action="/checkout" method="post">
  <div role="alert">入力エラーがあります。最初から入力し直してください。</div>
  <input id="card" name="card" type="text" value="">
  <input id="name" name="name" type="text" value="">
</form>
```

> **参照**
> - [Understanding SC 3.3.7 — Examples: incorrect credit card](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#examples)

## 3.3.8・autocomplete との関係

| トピック | 関係 |
|---|---|
| **3.3.8 アクセシブルな認証** | 認証時の認知機能テスト制限。パスワードのブラウザ自動入力は 3.3.8 の手法 |
| **3.3.7 冗長な入力** | **同一プロセス内**で、サイト側が前回入力を再利用させる |
| **`autocomplete` 属性** | ブラウザ支援として有用だが、3.3.7 単体の十分条件ではない |

- 3.3.7 は「サイトが前回値を渡す／選べるようにする」こと
- ブラウザが勝手に埋めてくれることだけに依存しない
- パスワード再入力はセキュリティ例外になりうる一方、認証全体は 3.3.8 を別途確認する

> **参照**
> - [Understanding SC 3.3.7 — Intent（Accessible Authentication）](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#intent)
> - [wcag/1-3-5-identify-input-purpose.md — 1.3.5 入力目的の特定](./1-3-5-identify-input-purpose.md)

## プライバシーへの注意

- 基準自体に PII 専用の例外はない
- 自動入力のために一時保存するなら、漏洩しない実装にする
- 冗長入力を減らすことと、余計な個人情報を残すことは切り分けて設計する

```text
安全寄りの実装例：
  同一セッション・同一プロセス内だけ保持
  完了後に一時データを破棄
  画面表示時は必要な項目だけ再提示

リスクのある実装例：
  URL に個人情報を載せて引き継ぐ
  共有端末で前回入力が残り続ける
```

> **参照**
> - [Understanding SC 3.3.7 — Intent（privacy）](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html#intent)

## よくある失敗例

- 多段階フォームで、前ステップの内容を記憶して再入力させる
- 配送先と請求先で同じ住所を二度手入力させる
- バリデーションエラー後に全フィールドをクリアする
- ブラウザのオートフィルだけを根拠に適合宣言する
- 確認画面なのに、値を見せず再入力だけ求める
- サードパーティ決済画面でも、渡せる情報を渡さず再入力させる
- パスワード以外の通常項目まで「セキュリティなので再入力必須」にする

```html
<!-- 失敗例 -->
<p>ステップ2：ステップ1で入力したメールアドレスを再入力してください</p>
<input name="email_confirm" type="email">
<!-- 表示も自動入力も選択もない -->
```

```html
<!-- 改善例 -->
<label for="email">メールアドレス</label>
<input id="email" name="email" type="email" value="taro@example.com">
<p>ステップ1の入力内容を表示しています。必要なら修正できます。</p>
```

## テスト・確認方法

### 手動テスト

- 会員登録、購入、申し込みなど、複数ステップのプロセスを洗い出す
- 各ステップで、以前に入力した情報を再度求めているか確認する
- 再入力がある場合、自動入力または選択可能かを確認する
- エラー再表示時に、入力内容が消えていないか確認する
- ブラウザオートフィルを無効にして、サイト側の支援だけで通るか確認する
- 例外（本質的・セキュリティ・無効化）に該当するか切り分ける

```text
チェックリスト：
1. 同一プロセス内の再入力箇所を洗い出したか
2. 再入力がある場合、自動入力または選択手段があるか
3. 選択可能なデータが同じページ上にあるか
4. エラー後も入力値が保持されているか
5. ブラウザオートフィルなしでも達成できるか
6. 例外（essential / security / no longer valid）を誤用していないか
7. セッションをまたぐ永続保存まで過剰実装していないか（必須ではない）
8. 一時保存時のプライバシー配慮があるか
```

> **参照**
> - [Technique G221 — Tests](https://www.w3.org/WAI/WCAG22/Techniques/general/G221)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.3.7 は、同一プロセス内で同じ情報を二度手入力させない達成基準（レベル A・WCAG 2.2 新規）
- 再入力が必要なら、**自動入力**するか、**選択できる形**で前回値を提供する
- ブラウザのオートコンプリートだけに頼るのは不十分
- 例外は、本質的な再入力・セキュリティ・情報が無効になった場合
- G221（前ステップのデータを提供）が代表的な十分な手法
- エラー後の入力保持や「配送先と同じ」チェックが実務で効きやすい
