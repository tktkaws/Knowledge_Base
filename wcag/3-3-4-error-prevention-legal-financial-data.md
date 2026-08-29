# 3.3.4 エラー回避（法的・金融・データ）（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.3.4「エラー回避（法的・金融・データ）」（Error Prevention (Legal, Financial, Data)）
- レベル **AA**
- 原則3「理解可能（Understandable）」> ガイドライン 3.3「入力支援」に属する
- **法的拘束力**、**金融取引**、**ユーザーが操作できるデータ**の変更・削除、**テスト回答の送信**を伴うページでは、次のいずれかを満たすこと
  1. **可逆（Reversible）**：送信を取り消せる
  2. **検証（Checked）**：入力エラーをチェックし、修正の機会を与える
  3. **確認（Confirmed）**：確定前に内容を確認・修正できる仕組みがある
- 単なるドキュメントの保存や、1件ずつの通常編集すべてに確認を求める意図ではない
- 3.3.1〜3.3.3 と組み合わせて、重大なミスを防ぐための**最後の安全装置**

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.4 Error Prevention (Legal, Financial, Data)](https://www.w3.org/TR/WCAG22/#error-prevention-legal-financial-data)
> - [Understanding SC 3.3.4 Error Prevention (Legal, Financial, Data)](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html)

## 達成基準の原文（要約）

- 次のいずれかを引き起こす Web ページについて、少なくとも次のいずれかが真であること
  - ユーザーに法的拘束力のあるコミットメントが発生する
  - ユーザーの金融取引が発生する
  - データストレージ上の**ユーザーが操作できるデータ**を変更または削除する
  - ユーザーのテスト回答を送信する
- 少なくとも次のいずれかが真であること
  - **可逆**：送信は取り消し可能である
  - **検証**：ユーザーが入力したデータは入力エラーについてチェックされ、修正の機会が与えられる
  - **確認**：確定前に情報を確認・修正できる仕組みがある

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.4](https://www.w3.org/TR/WCAG22/#error-prevention-legal-financial-data)

## なぜ必要か

- 誤送信の結果が**取り消せない**・**高額**・**法的に拘束力がある**場合、一度のミスが深刻な被害になる
- 読解障害のある人は、数字や文字の入れ替えが起きやすい
- 運動障害のある人は、意図しないキー入力や誤クリックが起きやすい
- 視覚障害のある人は、確認画面の情報を見落としやすい
- 取り消し・検証・確認のいずれかがあれば、重大な結果の前にミスを防げる

> **参照**
> - [Understanding SC 3.3.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html#intent)
> - [Understanding SC 3.3.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html#benefits)

## 対象となる4つの状況

### 状況1：法的拘束力のあるコミットメント

- 商品購入、契約締結、確定申告の送信など
- 一度確定すると法的義務や権利が発生する操作

### 状況2：金融取引

- 株の売買、航空券の購入、振込の実行など
- 即時に実行され、後から変更できない取引

### 状況3：ユーザーが操作できるデータの変更・削除

- アカウントの住所・電話番号の更新
- 請求履歴レコードの削除
- 旅行プロフィール全体の削除など、**大量のデータ喪失**につながる操作
- **対象外**：ユーザーが直接閲覧・操作できないログ、検索エンジン監視データなど

### 状況4：テスト回答の送信

- オンライン試験・評価アプリでの最終提出
- 一度送信すると再提出できないテスト

```text
対象外の例：
  通常のドキュメント編集の「保存」
  1件ずつの軽微なデータ更新
  ユーザーが見られないサーバーログ

対象の例：
  返金不可の航空券購入
  1000株の株注文
  アカウント全データの一括削除
  期末試験の最終提出
```

> **参照**
> - [Understanding SC 3.3.4 — Key Terms: user-controllable, legal commitments](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html)

## 3つの回避手段（いずれか1つで可）

| 手段 | 内容 | 典型例 |
|---|---|---|
| **可逆（Reversible）** | 送信後に取り消し・修正できる | 一定時間内の注文キャンセル、削除データの復元 |
| **検証（Checked）** | 入力エラーをチェックし修正機会を与える | バリデーション + エラー表示 + 再入力 |
| **確認（Confirmed）** | 確定前に内容を確認・修正できる | 注文確認画面、削除前の確認ダイアログ |

- **3つすべて**を満たす必要はない
- **1つでも**満たせば達成基準を満たす
- 実務では複数を組み合わせることが多い

```text
EC サイトの注文：
  確認画面（Confirmed）+ バリデーション（Checked）+ 24時間以内キャンセル（Reversible）
  → 1つ以上あれば OK。3つ揃えるとより安全

データ削除：
  確認ダイアログ（Confirmed）+ ゴミ箱から復元（Reversible）
```

> **参照**
> - [Understanding SC 3.3.4 — Success Criterion](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html)

## 3.3.1〜3.3.3 との関係

| 達成基準 | レベル | 焦点 |
|---|---|---|
| 3.3.1 エラーの特定 | A | どの項目がエラーか |
| 3.3.2 ラベルまたは説明 | A | 入力前の手がかり |
| 3.3.3 エラー修正の提案 | AA | どう直すかの提案 |
| 3.3.4 エラー回避（法的・金融・データ） | AA | **重大な結果**の前にミスを防ぐ |

- 3.3.4 の「検証（Checked）」は、3.3.1〜3.3.3 のエラー処理を含む
- 3.3.4 は**対象が限定**される（法的・金融・ユーザーデータ・テスト）
- 3.3.6（エラー回避・すべて・AAA）は、対象が**すべての送信**に拡大される

> **参照**
> - [wcag/3-3-1-error-identification.md — 3.3.1 エラーの特定](./3-3-1-error-identification.md)
> - [wcag/3-3-3-error-suggestion.md — 3.3.3 エラー修正の提案](./3-3-3-error-suggestion.md)

## 手法1：確定前に内容を確認・修正する（G98 / Confirmed）

- **G98**：送信前に回答・入力内容を確認・修正できるようにする
- 注文確認画面、申込内容のレビュー、テスト提出前の見直しが典型

```html
<!-- 良い例：注文確認画面 -->
<h1>注文内容の確認</h1>
<dl>
  <dt>商品</dt><dd>ワイヤレスイヤホン × 1</dd>
  <dt>お届け先</dt><dd>東京都千代田区…</dd>
  <dt>支払方法</dt><dd>クレジットカード（下4桁 1234）</dd>
  <dt>合計</dt><dd>¥12,800（税込）</dd>
</dl>
<a href="/cart/edit">内容を修正する</a>
<button type="submit">この内容で注文を確定する</button>
```

```html
<!-- 悪い例：入力後すぐに確定 -->
<form action="/checkout/complete" method="post">
  <!-- 入力フィールド -->
  <button type="submit">購入する</button>
  <!-- 確認画面なし。1クリックで取引完了 -->
</form>
```

```html
<!-- 良い例：テスト提出前の見直し -->
<h1>回答の確認</h1>
<p>以下の内容で提出します。修正する場合は各設問に戻ってください。</p>
<ol>
  <li>設問1：B <a href="/test/q1">修正</a></li>
  <li>設問2：A <a href="/test/q2">修正</a></li>
</ol>
<button type="submit">最終提出する</button>
```

> **参照**
> - [Technique G98: Providing the ability for the user to review and correct answers before submitting](https://www.w3.org/WAI/WCAG22/Techniques/general/G98)
> - [Understanding SC 3.3.4 — Examples: Order confirmation](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html#examples)

## 手法2：一定時間内に取り消し・修正できる（G164 / Reversible）

- **G164**：オンラインリクエスト（取引）後、ユーザーが修正・キャンセルできる**明示された時間**を設ける
- 注文確定後24時間以内のキャンセル、クーリングオフ期間の表示

```html
<!-- 良い例：取り消し可能期間の明示 -->
<section aria-labelledby="order-done">
  <h1 id="order-done">ご注文ありがとうございます</h1>
  <p>
    注文番号 #12345 を受け付けました。
    <strong>2026年8月30日 10:00 まで</strong>であれば、
    <a href="/orders/12345/cancel">注文をキャンセル</a>できます。
  </p>
</section>
```

```html
<!-- 悪い例：確定後いっさい変更不可 -->
<p>注文が確定しました。変更・キャンセルはできません。</p>
```

> **参照**
> - [Technique G164: Providing a stated time within which an online request (or transaction) may be amended or canceled by the user after making the request](https://www.w3.org/WAI/WCAG22/Techniques/general/G164)

## 手法3：削除データの復元（G99 / Reversible）

- **G99**：削除した情報を復元できるようにする
- ゴミ箱、論理削除、一定期間の復旧が典型

```html
<!-- 良い例：削除後の復元 -->
<div role="status">
  「2025年請求書.pdf」を削除しました。
  <button type="button">元に戻す</button>
  （30日間復元可能）
</div>
```

```html
<!-- 悪い例：即時完全削除 -->
<button type="submit">完全に削除（元に戻せません）</button>
<!-- 確認も復元もなし -->
```

> **参照**
> - [Technique G99: Providing the ability to recover deleted information](https://www.w3.org/WAI/WCAG22/Techniques/general/G99)

## 手法4：操作前に確認を求める（G168 / Confirmed）

- **G168**：選択した操作を続行する前に確認を求める
- 削除・取引実行・テスト提出など、特に重要な操作に使う

```html
<!-- 良い例：削除前の確認 -->
<dialog id="delete-confirm" open aria-labelledby="delete-title">
  <h2 id="delete-title">アカウントを削除しますか？</h2>
  <p>
    プロフィール、注文履歴、保存した住所がすべて削除されます。
    この操作は取り消せません。
  </p>
  <button type="button" data-action="cancel">キャンセル</button>
  <button type="submit" data-action="confirm">削除する</button>
</dialog>
```

```html
<!-- 悪い例：確認なしの破壊的操作 -->
<button type="submit">削除</button>
<!-- 1クリックで全データ削除 -->
```

```html
<!-- 良い例：時間外取引のリスク説明 + 確認 -->
<dialog open aria-labelledby="after-hours-title">
  <h2 id="after-hours-title">時間外取引の確認</h2>
  <p>
    現在は取引時間外です。時間外の注文はリスクが高く、
    通常と異なる価格で約定する場合があります。
  </p>
  <button type="button">注文をキャンセル</button>
  <button type="submit">リスクを理解した上で注文する</button>
</dialog>
```

> **参照**
> - [Technique G168: Requesting confirmation to continue with selected action](https://www.w3.org/WAI/WCAG22/Techniques/general/G168)
> - [Understanding SC 3.3.4 — Examples: Stock sale](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html#examples)

## 手法5：チェックボックスで明示的同意を求める（G155）

- **G155**：送信ボタンに加え、チェックボックスで明示的な同意を求める
- 誤クリック・誤タップの防止に有効
- 法的取引や削除操作と組み合わせやすい

```html
<!-- 良い例：同意チェック + 送信 -->
<form action="/tax/submit" method="post">
  <!-- 入力フィールド -->
  <fieldset>
    <legend>最終確認</legend>
    <input id="agree" name="agree" type="checkbox" required>
    <label for="agree">
      入力内容を確認し、申告内容に誤りがないことを確認しました
    </label>
  </fieldset>
  <button type="submit" disabled aria-describedby="agree-hint">
    申告書を送信する
  </button>
  <p id="agree-hint">チェックを入れると送信できます</p>
</form>
```

```html
<!-- 悪い例：送信ボタンのみ -->
<button type="submit">申告書を送信</button>
<!-- 誤タップで即送信 -->
```

> **参照**
> - [Technique G155: Providing a checkbox in addition to a submit button](https://www.w3.org/WAI/WCAG22/Techniques/general/G155)

## 状況別の十分な手法

| 状況 | 十分な手法の例 |
|---|---|
| **A：法的取引**（購入、確定申告など） | G164（取り消し期間）、G98（確認画面）、G155（同意チェック） |
| **B：情報の削除** | G99（復元）、G168（確認）、G155（同意チェック） |
| **C：テストアプリ** | G98（回答確認）、G168（提出確認） |

> **参照**
> - [Understanding SC 3.3.4 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html#techniques)

## 推奨（必須ではない）手法

- **SCR18**：クライアント側バリデーションとアラート
- **G199**：送信成功時のフィードバック
- 3.3.4 単体の十分条件ではないが、ミス防止と UX 向上に有用

> **参照**
> - [Technique SCR18: Providing client-side validation and alert](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR18)
> - [Technique G199: Providing success feedback when data is submitted successfully](https://www.w3.org/WAI/WCAG22/Techniques/general/G199)

## よくある失敗例

- 購入ボタン1つで即決済完了（確認・取り消し・検証のいずれもなし）
- 確認画面があっても、内容の修正手段がない
- 削除確認ダイアログがなく、1クリックで全データ消失
- バリデーションはあるが、重大操作には確認画面がない（検証だけでは足りる場合もあるが、確認の方が適切なことも多い）
- 取り消し期間があるが、ユーザーに知らせていない
- 通常の「保存」と同じ UI で、法的・金融操作を扱っている
- テストで「送信」1回押しで即確定、見直し不可

```html
<!-- 失敗例：EC の即時購入 -->
<button type="submit">今すぐ購入 ¥98,000</button>
<!-- 確認なし・キャンセル不可・バリデーションも弱い -->
```

```html
<!-- 改善例：確認 + 修正 + 取り消し -->
<a href="/checkout/review">注文内容を確認する</a>
<!-- 確認画面で修正可能 -->
<!-- 確定後24時間キャンセル可能を明示 -->
```

## テスト・確認方法

### 手動テスト

- 購入・契約・振込・削除・テスト提出など、対象操作を洗い出す
- 各操作について、可逆・検証・確認のいずれかがあるか確認する
- 確認画面で内容修正に戻れるか試す
- 意図的に誤入力して、検証と修正機会があるか試す
- 確定後に取り消し・復元できるか、期限が明示されているか確認する
- 通常の保存操作と、重大操作の UX が区別されているか確認する

```text
チェックリスト：
1. 対象操作（法的・金融・ユーザーデータ・テスト）を特定したか
2. 可逆・検証・確認のいずれか1つ以上があるか
3. 確認画面で全項目を見直せるか
4. 確認画面から修正に戻れるか
5. 削除に確認と（可能なら）復元があるか
6. 取り消し期限がユーザーに分かる形で示されているか
7. 3.3.1〜3.3.3 のエラー処理と整合しているか
8. 通常の保存まで過剰な確認を要求していないか（意図確認）
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.3.4 は、法的拘束・金融取引・ユーザーデータの変更削除・テスト提出で、重大なミスを防ぐ達成基準（レベル AA）
- **可逆**・**検証**・**確認**のいずれか1つを満たせばよい
- 確認画面（G98）、取り消し期間（G164）、復元（G99）、操作確認（G168）、同意チェック（G155）が典型手法
- 通常のドキュメント保存や軽微な編集すべてに確認を求める意図ではない
- 3.3.1〜3.3.3 のエラー処理と組み合わせ、重大な結果の前に安全装置を設ける
