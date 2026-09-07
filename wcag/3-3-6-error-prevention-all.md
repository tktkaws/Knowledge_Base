# 3.3.6 エラー回避（すべて）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.3.6「エラー回避（すべて）」（Error Prevention (All)）
- レベル **AAA**
- 原則3「理解可能（Understandable）」> ガイドライン 3.3「入力支援」に属する
- **情報の送信を求めるすべての Web ページ**で、次のいずれかを満たすこと
  1. **可逆（Reversible）**：送信を取り消せる
  2. **検証（Checked）**：入力エラーをチェックし、修正の機会を与える
  3. **確認（Confirmed）**：確定前に内容を確認・修正できる仕組みがある
- 3.3.4 と同じ3手段だが、対象が**法的・金融・データ・テストに限定されない**
- 問い合わせ、会員登録、アンケート、設定変更など、**あらゆる送信フォーム**が対象

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.6 Error Prevention (All)](https://www.w3.org/TR/WCAG22/#error-prevention-all)
> - [Understanding SC 3.3.6 Error Prevention (All)](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-all.html)

## 達成基準の原文（要約）

- ユーザーに情報の送信を求める Web ページについて、少なくとも次のいずれかが真であること
  - **可逆**：送信は取り消し可能である
  - **検証**：ユーザーが入力したデータは入力エラーについてチェックされ、修正の機会が与えられる
  - **確認**：確定前に情報を確認・修正できる仕組みがある

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.6](https://www.w3.org/TR/WCAG22/#error-prevention-all)

## なぜ必要か

- 障害のある人は入力ミスをしやすく、ミスに気づきにくい・回復しにくいことがある
- 読解障害のある人は、数字や文字の入れ替えが起きやすい
- 運動障害のある人は、意図しないキー入力や誤クリックが起きやすい
- 法的・金融以外の送信でも、誤った連絡先・誤った設定・誤送信は実害になる
- 取り消し・検証・確認のいずれかがあれば、送信前または送信後にミスを防げる

> **参照**
> - [Understanding SC 3.3.6 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-all.html#intent)
> - [Understanding SC 3.3.6 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-all.html#benefits)

## 3.3.4 との違い

| 項目 | 3.3.4（AA） | 3.3.6（AAA） |
|---|---|---|
| レベル | AA | AAA |
| 対象 | 法的拘束・金融取引・ユーザーデータの変更削除・テスト提出 | **情報の送信を求めるすべてのフォーム** |
| 回避手段 | 可逆・検証・確認のいずれか | 同じ（可逆・検証・確認のいずれか） |
| 十分な手法 | 状況別に G98 / G99 / G155 / G164 / G168 など | **3.3.4 の十分な手法を、すべての送信フォームに適用** |

```text
3.3.4 のみで足りる例：
  返金不可の航空券購入
  口座振替の確定
  アカウント全削除
  期末試験の最終提出

3.3.6 で追加対象になる例：
  お問い合わせフォーム
  ニュースレター登録
  プロフィールのニックネーム変更
  アンケート送信
  サポートチケット作成
  設定画面の「保存」
```

- 3.3.4 を満たしていても、一般フォームに安全装置がなければ 3.3.6 は未達
- AAA 適合を目指す場合、送信を伴うフォームを網羅的に点検する

> **参照**
> - [wcag/3-3-4-error-prevention-legal-financial-data.md — 3.3.4 エラー回避（法的・金融・データ）](./3-3-4-error-prevention-legal-financial-data.md)
> - [Understanding SC 3.3.6 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-all.html#intent)

## 3つの回避手段（いずれか1つで可）

| 手段 | 内容 | 一般フォームでの典型例 |
|---|---|---|
| **可逆（Reversible）** | 送信後に取り消し・修正できる | 送信後に下書きへ戻す、登録解除リンク |
| **検証（Checked）** | 入力エラーをチェックし修正機会を与える | 必須・形式チェック + エラー表示 + 再入力 |
| **確認（Confirmed）** | 確定前に内容を確認・修正できる | 確認画面、送信前ダイアログ |

- **3つすべて**を満たす必要はない
- **1つでも**満たせば達成基準を満たす
- 実務では「検証」が最も導入しやすく、重要な送信には「確認」を足すことが多い

```text
お問い合わせフォーム：
  必須・メール形式の検証（Checked）だけで達成しうる

会員登録：
  バリデーション（Checked）+ 入力内容の確認画面（Confirmed）

設定の保存：
  保存後に「元に戻す」や再編集（Reversible）
  または保存前の確認（Confirmed）
```

> **参照**
> - [Understanding SC 3.3.6 — Success Criterion](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-all.html#success-criterion)

## 3.3.1〜3.3.4 との関係

| 達成基準 | レベル | 焦点 |
|---|---|---|
| 3.3.1 エラーの特定 | A | どの項目がエラーか |
| 3.3.2 ラベルまたは説明 | A | 入力前の手がかり |
| 3.3.3 エラー修正の提案 | AA | どう直すかの提案 |
| 3.3.4 エラー回避（法的・金融・データ） | AA | **重大な結果**の前にミスを防ぐ |
| 3.3.6 エラー回避（すべて） | AAA | **すべての送信**でミスを防ぐ |

- 3.3.6 の「検証（Checked）」は、3.3.1〜3.3.3 のエラー処理を含むことが多い
- 3.3.4 は対象が限定される。3.3.6 はその対象を**全送信**に拡大した上位基準
- AAA を目指すなら、3.3.4 の手法を一般フォームにも横展開する

> **参照**
> - [wcag/3-3-1-error-identification.md — 3.3.1 エラーの特定](./3-3-1-error-identification.md)
> - [wcag/3-3-3-error-suggestion.md — 3.3.3 エラー修正の提案](./3-3-3-error-suggestion.md)
> - [wcag/3-3-4-error-prevention-legal-financial-data.md — 3.3.4](./3-3-4-error-prevention-legal-financial-data.md)

## 十分な手法

- Understanding では、十分な手法は次の1点に集約されている
  - **3.3.4 の十分な手法を、情報送信を求めるすべてのフォームに適用する**
- つまり G98 / G99 / G155 / G164 / G168 などを、購入・削除だけでなく一般フォームにも使う

| 3.3.4 の手法 | 3.3.6 での使い方の例 |
|---|---|
| **G98** 送信前に内容を確認・修正 | 問い合わせ内容の確認画面 |
| **G164** 一定時間内の取り消し・修正 | 投稿後しばらく編集・削除可能 |
| **G99** 削除情報の復元 | 設定リセット後の復元 |
| **G168** 操作前の確認 | 一括送信・一括変更の確認ダイアログ |
| **G155** 同意チェック + 送信 | 誤タップ防止の最終確認チェック |

> **参照**
> - [Understanding SC 3.3.6 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-all.html#techniques)
> - [Understanding SC 3.3.4 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html#techniques)

## 手法1：入力エラーを検証し修正機会を与える（Checked）

- 一般フォームで最も現実的な達成手段
- 必須漏れ・形式不正を検出し、送信せずに修正できるようにする
- 3.3.1（特定）・3.3.3（提案）とセットで実装する

```html
<!-- 良い例：検証 + 修正機会 -->
<form action="/contact" method="post" novalidate>
  <div role="alert" id="form-errors">
    <p>2件の入力エラーがあります。修正して再度送信してください。</p>
    <ul>
      <li><a href="#email">メールアドレスの形式が正しくありません</a></li>
      <li><a href="#message">お問い合わせ内容は必須です</a></li>
    </ul>
  </div>

  <label for="email">メールアドレス</label>
  <input
    id="email"
    name="email"
    type="email"
    value="user@"
    aria-invalid="true"
    aria-describedby="email-error"
  >
  <p id="email-error">例：name@example.com</p>

  <label for="message">お問い合わせ内容</label>
  <textarea
    id="message"
    name="message"
    aria-invalid="true"
    aria-describedby="message-error"
  ></textarea>
  <p id="message-error">内容を入力してください</p>

  <button type="submit">送信する</button>
</form>
```

```html
<!-- 悪い例：検証なしで即送信 -->
<form action="/contact" method="post">
  <label for="email">メールアドレス</label>
  <input id="email" name="email" type="text">
  <label for="message">内容</label>
  <textarea id="message" name="message"></textarea>
  <button type="submit">送信</button>
  <!-- 空送信・不正形式でも受け付ける -->
</form>
```

> **参照**
> - [Understanding SC 3.3.6 — Checked](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-all.html#success-criterion)
> - [wcag/3-3-1-error-identification.md — 3.3.1 エラーの特定](./3-3-1-error-identification.md)

## 手法2：確定前に内容を確認・修正する（G98 / Confirmed）

- 複数項目・長文・宛先を含む送信に有効
- 確認画面から編集へ戻れることが重要

```html
<!-- 良い例：問い合わせの確認画面 -->
<h1>送信内容の確認</h1>
<dl>
  <dt>お名前</dt><dd>山田 太郎</dd>
  <dt>メールアドレス</dt><dd>taro@example.com</dd>
  <dt>件名</dt><dd>請求書の再発行について</dd>
  <dt>内容</dt><dd>先月分の請求書 PDF を再送してください。</dd>
</dl>
<a href="/contact/edit">内容を修正する</a>
<button type="submit">この内容で送信する</button>
```

```html
<!-- 悪い例：入力後すぐに送信完了 -->
<form action="/contact/send" method="post">
  <!-- 入力フィールド -->
  <button type="submit">送信する</button>
  <!-- 確認画面なし。誤送信を止められない -->
</form>
```

> **参照**
> - [Technique G98: Providing the ability for the user to review and correct answers before submitting](https://www.w3.org/WAI/WCAG22/Techniques/general/G98)

## 手法3：送信後に取り消し・修正できる（G164 / Reversible）

- 掲示板投稿、プロフィール更新、下書き公開など向き
- 取り消し可能であることに加え、**期限や方法が分かる**とよい

```html
<!-- 良い例：送信後の編集・削除 -->
<section aria-labelledby="sent-title">
  <h1 id="sent-title">投稿を公開しました</h1>
  <p>
    「週末イベントの案内」を公開しました。
    <strong>公開から24時間以内</strong>であれば、
    <a href="/posts/123/edit">編集</a>
    または
    <a href="/posts/123/delete">削除</a>
    できます。
  </p>
</section>
```

```html
<!-- 悪い例：一度送ったら変更不可 -->
<p>投稿が公開されました。編集・削除はできません。</p>
```

```html
<!-- 良い例：ニュースレター登録の解除 -->
<p>
  登録完了メールを送信しました。
  購読をやめる場合は、メール内の解除リンクからいつでも停止できます。
</p>
```

> **参照**
> - [Technique G164: Providing a stated time within which an online request (or transaction) may be amended or canceled by the user after making the request](https://www.w3.org/WAI/WCAG22/Techniques/general/G164)

## 手法4：操作前に確認を求める（G168 / Confirmed）

- 一括変更、複数宛先への送信、設定のリセットなど、影響範囲が広い操作向け
- 「確認」は破壊的操作だけでなく、一般の重要な送信にも使える

```html
<!-- 良い例：一括送信前の確認 -->
<dialog id="bulk-send" open aria-labelledby="bulk-title">
  <h2 id="bulk-title">12件のメッセージを送信しますか？</h2>
  <p>選択した12名に同じ内容が送信されます。送信後の一括取り消しはできません。</p>
  <button type="button" data-action="cancel">キャンセル</button>
  <button type="submit" data-action="confirm">送信する</button>
</dialog>
```

```html
<!-- 悪い例：確認なしの一括送信 -->
<button type="submit">選択した宛先に送信</button>
<!-- 1クリックで大量誤送信 -->
```

> **参照**
> - [Technique G168: Requesting confirmation to continue with selected action](https://www.w3.org/WAI/WCAG22/Techniques/general/G168)

## 手法5：チェックボックスで明示的同意を求める（G155）

- 誤クリック・誤タップ防止に有効
- 確認画面やバリデーションと組み合わせやすい

```html
<!-- 良い例：同意チェック + 送信 -->
<form action="/survey" method="post">
  <!-- 回答フィールド -->
  <fieldset>
    <legend>最終確認</legend>
    <input id="review" name="review" type="checkbox" required>
    <label for="review">
      回答内容を確認し、この内容で送信することに同意します
    </label>
  </fieldset>
  <button type="submit">アンケートを送信する</button>
</form>
```

```html
<!-- 悪い例：送信ボタンのみ -->
<button type="submit">アンケートを送信</button>
<!-- 誤タップで即送信 -->
```

> **参照**
> - [Technique G155: Providing a checkbox in addition to a submit button](https://www.w3.org/WAI/WCAG22/Techniques/general/G155)

## 実務での適用方針

- **軽いフォーム**（問い合わせ、簡易登録）：検証（Checked）を最低ラインにする
- **複数項目・誤送信コストが高いフォーム**：確認画面（Confirmed）を追加
- **後から直せるデータ**：送信後の編集・削除（Reversible）を用意
- **すべてに重い確認 UI を付ける必要はない**。手段は1つで足りる
- ただし「検証なし・確認なし・取り消し不可」の送信は AAA では不可

```text
優先度の目安：
  1. すべての送信フォームにバリデーション（Checked）
  2. 誤送信の影響が大きいものに確認画面（Confirmed）
  3. 可能なら送信後の修正・取り消し（Reversible）

過剰になりやすい例：
  検索ボックスの毎回確認ダイアログ
  1文字のフィルタ変更ごとの確認
  → 「情報の送信」でも、軽微な操作は UX を見て手段を選ぶ
```

## よくある失敗例

- お問い合わせが検証なしで空送信・不正メールを受け付ける
- 会員登録で確認画面も取り消しもなく、誤ったメールアドレスで確定
- 設定保存が即反映で、元に戻す手段がない
- 3.3.4 対象（購入・削除）だけ確認があり、一般フォームは無防備
- 確認画面はあるが、修正して戻る手段がない
- 取り消し可能だが、ユーザーに方法・期限が伝わっていない
- 「送信完了」だけで、内容の見直し機会が一度もない

```html
<!-- 失敗例：一般フォームに安全装置なし -->
<form action="/newsletter" method="post">
  <input name="email" type="text" placeholder="メールアドレス">
  <button type="submit">登録</button>
</form>
<!-- 形式チェックなし・確認なし・登録後の解除導線も不明 -->
```

```html
<!-- 改善例：検証 + 解除可能 -->
<form action="/newsletter" method="post">
  <label for="email">メールアドレス</label>
  <input id="email" name="email" type="email" required autocomplete="email">
  <button type="submit">登録する</button>
</form>
<p>登録後は、配信メール内のリンクからいつでも解除できます。</p>
```

## テスト・確認方法

### 手動テスト

- サイト内の「送信」「保存」「登録」「投稿」など、情報送信を伴う画面を洗い出す
- 各画面について、可逆・検証・確認のいずれかがあるか確認する
- 3.3.4 対象だけでなく、問い合わせ・アンケート・設定も含める
- 意図的に空送信・不正形式で、検証と修正機会があるか試す
- 確認画面がある場合、修正に戻れるか試す
- 送信後に編集・取り消し・解除ができるか、案内があるか確認する

```text
チェックリスト：
1. 情報送信を求めるフォームをすべて洗い出したか
2. 各フォームに可逆・検証・確認のいずれか1つ以上があるか
3. 検証がある場合、エラー特定と修正機会があるか（3.3.1系）
4. 確認画面がある場合、全項目の見直しと修正に戻れるか
5. 可逆の場合、方法と（必要なら）期限が分かるか
6. 3.3.4 対象以外の一般フォームもカバーしているか
7. 軽微な操作に過剰な確認を強いていないか（UX の確認）
8. 3.3.4 の手法を横展開できているか
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.3.6 は、**情報送信を求めるすべてのフォーム**でミスを防ぐ達成基準（レベル AAA）
- **可逆**・**検証**・**確認**のいずれか1つを満たせばよい
- 3.3.4 と同じ手段だが、対象が法的・金融・データ・テストに限定されない
- 十分な手法は「3.3.4 の十分な手法をすべての送信フォームに適用すること」
- 実務ではまず全フォームに検証を入れ、影響が大きいものに確認・取り消しを足す
- AAA 適合では、一般の問い合わせや設定保存も点検対象になる
