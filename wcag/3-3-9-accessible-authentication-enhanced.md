# 3.3.9 アクセシブルな認証（高度）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.3.9「アクセシブルな認証（高度）」（Accessible Authentication (Enhanced)）
- レベル **AAA**
- WCAG **2.2 で新規追加**（`[New]`）
- 原則3「理解可能（Understandable）」> ガイドライン 3.3「入力支援」に属する
- **3.3.8 と同じ目的**だが、例外がより厳しい
- 認証プロセスで認知機能テストを必須にしないこと
- 使う場合に認められるのは次の2つだけ
  1. **代替（Alternative）**：認知機能テストに頼らない別の認証方法
  2. **支援機構（Mechanism）**：テスト完了を助ける仕組み
- **オブジェクト認識**と**個人コンテンツ**の例外は、3.3.9 では使えない

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.9 Accessible Authentication (Enhanced)](https://www.w3.org/TR/WCAG22/#accessible-authentication-enhanced)
> - [Understanding SC 3.3.9 Accessible Authentication (Enhanced)](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-enhanced.html)

## 達成基準の原文（要約）

- 認証プロセスのいずれのステップでも、認知機能テスト（パスワードの記憶やパズルの解答など）を必須にしない
- ただし、そのステップが次のいずれかを提供する場合を除く
  - **代替**：認知機能テストに依存しない別の認証方法
  - **支援機構**：認知機能テストの完了を支援する仕組み
- 3.3.8 にあった「オブジェクト認識」「個人コンテンツ」の例外はない

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.9](https://www.w3.org/TR/WCAG22/#accessible-authentication-enhanced)

## なぜ必要か

- 3.3.8 の利点と同じく、記憶・転写・パズルに依存しないログインを確保する
- 物体認識 CAPTCHA や「自分で登録した画像を選ぶ」も、認知・知覚処理の負担になりうる
- 失読症・計算障害・記憶の困難がある人は、認識系テストでもつまずきやすい
- AAA では、認証経路から認識系テストごと外すことが求められる
- 認知能力の水準にかかわらず、ログインできる設計を目指す

> **参照**
> - [Understanding SC 3.3.9 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-enhanced.html#intent)
> - [Understanding SC 3.3.9 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-enhanced.html#benefits)

## 3.3.8 との違い

| 項目 | 3.3.8（AA・最低限） | 3.3.9（AAA・高度） |
|---|---|---|
| レベル | AA | AAA |
| 代替 | ○ | ○ |
| 支援機構 | ○ | ○ |
| オブジェクト認識 | ○（例外あり） | **×（例外なし）** |
| 個人コンテンツ | ○（例外あり） | **×（例外なし）** |

```text
3.3.8 では許容されうるが、3.3.9 では不可：
  「車が写っている画像をすべて選べ」
  「あなたが以前アップロードした写真を選べ」
  画像・映像・音声クリップから正解を選ばせる認識テスト

3.3.8 / 3.3.9 の両方で有効：
  パスワードマネージャ・ペースト可能なログイン（支援機構）
  マジックリンク（代替）
  WebAuthn / パスキー（代替）
  OAuth（代替）
  通知承認・セキュリティキー・QR 確認（認知機能テストではない）
```

- 認証の必須ステップで、認識系の選択 UI を出してはならない
- 認識系 CAPTCHA を使うなら、**認知機能テストなしの別経路**を必ず用意する

> **参照**
> - [Understanding SC 3.3.9 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-enhanced.html#intent)
> - [wcag/3-3-8-accessible-authentication-minimum.md — 3.3.8 アクセシブルな認証（最低限）](./3-3-8-accessible-authentication-minimum.md)

## 認知機能テスト（再掲）

- ユーザーに**記憶・操作・転写**を求めるタスク
- 例：サイト固有パスワードの暗記、文字の書き写し、計算、パズル
- **対象外**：氏名・メール・電話番号など、個人に固有でサイト横断で一貫している識別子
- 3.3.9 では、物体認識や個人提供メディアの識別も、例外として救済されない

```text
AAA でも「支援機構」で通せる例：
  適切にマークアップされたパスワード欄 + ペースト可
  パスワードマネージャの自動入力を妨害しない

AAA でも「代替」で通せる例：
  メールリンクログイン
  パスキー / WebAuthn
  サードパーティ OAuth

AAA では通らない例（代替なしの場合）：
  物体認識 CAPTCHA だけがログインの関門
  「登録済みの自分の写真を選べ」だけが 2FA の手段
```

> **参照**
> - [Understanding SC 3.3.9 — Key Terms: cognitive function test](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-enhanced.html)

## 十分な手法

| 手法 | 内容 |
|---|---|
| **G218** | メールリンク認証（マジックリンク） |
| **H100** | 適切にマークアップしたメール・パスワード入力 |
| （将来手法候補） | WebAuthn をユーザー名／パスワードの代替にする |
| （将来手法候補） | OAuth によるサードパーティログイン |
| （将来手法候補） | 2つの手法を組み合わせた二要素認証 |

- 十分な手法自体は 3.3.8 と同じ一覧
- 差は、認識系例外に頼れないこと

> **参照**
> - [Understanding SC 3.3.9 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-enhanced.html#techniques)

## 手法1：適切にマークアップしたログイン入力（H100）

- `label`・`autocomplete`・ペースト許可が要点
- パスワードマネージャとブラウザ自動入力を妨害しない

```html
<!-- 良い例：支援機構として機能するログイン -->
<form method="post" action="/login">
  <div>
    <label for="email">メールアドレス</label>
    <input
      id="email"
      name="email"
      type="email"
      autocomplete="email"
      required
    >
  </div>
  <div>
    <label for="password">パスワード</label>
    <input
      id="password"
      name="password"
      type="password"
      autocomplete="current-password"
      required
    >
  </div>
  <button type="submit">ログイン</button>
</form>
```

```html
<!-- 悪い例：ペースト禁止 -->
<input
  type="password"
  autocomplete="off"
  onpaste="return false"
>
```

> **参照**
> - [Technique H100: Providing properly marked up email and password inputs](https://www.w3.org/WAI/WCAG22/Techniques/html/H100)

## 手法2：メールリンク認証（G218）

- パスワード記憶が不要な**代替**経路
- 認識系 CAPTCHA を認証の必須ステップに入れない

```html
<!-- 良い例：マジックリンク -->
<form method="post" action="/login/email-link">
  <label for="email">メールアドレス</label>
  <input
    id="email"
    name="email"
    type="email"
    autocomplete="email"
    required
  >
  <button type="submit">メールでログインリンクを送る</button>
</form>
```

```text
AAA での注意：
  リンク送信前に物体認識 CAPTCHA を必須にすると、
  そのステップ自体が 3.3.9 に抵触しうる
  → CAPTCHA なし、または認知機能テストなしの別手段を用意する
```

> **参照**
> - [Technique G218: Email link authentication](https://www.w3.org/WAI/WCAG22/Techniques/general/G218)

## 手法3：WebAuthn / OAuth / 転写不要な 2FA

- パスキー、指紋・顔・PIN、セキュリティキー
- サードパーティログイン（OAuth）
- 2FA では、通知承認・QR 確認・USB キー押下など、転写不要な手段を用意する

```html
<!-- 良い例：認識テストなしの選択肢 -->
<section aria-labelledby="login-title">
  <h1 id="login-title">ログイン</h1>
  <!-- メール + パスワード（ペースト可） -->
  <button type="button">パスキーでログイン</button>
  <button type="button">Google でログイン</button>
  <a href="/login/email-link">メールリンクでログイン</a>
</section>
```

```html
<!-- 良い例：2FA が転写不要 -->
<p>スマートフォンに送った通知で「承認」をタップしてください。</p>
<button type="button">認証アプリで QR をスキャン</button>
<button type="button">セキュリティキーを使用</button>
```

> **参照**
> - [Understanding SC 3.3.9 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-enhanced.html#examples)
> - [Web Authentication API（MDN）](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API)

## 認識系 CAPTCHA・個人コンテンツを避ける

- 3.3.9 の核心はここ
- 認証の必須ステップで、次を出さない
  - 特定物体が写った画像の選択
  - ユーザーが以前提供した画像・映像・音声の選択
- ボット対策が必要なら、認証以外の層や、認知機能テストなしの代替と組み合わせる

```html
<!-- 悪い例：3.3.8 では例外になりうるが、3.3.9 では不可 -->
<section aria-labelledby="captcha-title">
  <h2 id="captcha-title">車が写っている画像をすべて選んでください</h2>
  <!-- 物体認識 CAPTCHA。代替なしなら 3.3.9 不合格 -->
</section>
```

```html
<!-- 悪い例：個人コンテンツの識別 -->
<section aria-labelledby="photo-title">
  <h2 id="photo-title">あなたが登録したペットの写真を選んでください</h2>
  <!-- 3.3.8 の個人コンテンツ例外。3.3.9 では使えない -->
</section>
```

```html
<!-- 良い例：認識テストなしでボット耐性も確保しやすい方向 -->
<!-- パスキー / マジックリンク / レート制限 / リスクベース認証など -->
<button type="button">パスキーでログイン</button>
<a href="/login/email-link">メールリンクでログイン</a>
```

> **参照**
> - [Understanding SC 3.3.9 — Intent（CAPTCHA example）](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-enhanced.html#intent)

## 失敗例（F109 ほか）

- パスワードやコードを、作成時と違う形式で再入力させる（F109）
- 桁分割 OTP で一括ペーストできない
- 「パスワードの○文字目だけ入力」
- ペースト禁止・自動入力妨害
- 物体認識 CAPTCHA だけが関門（3.3.9 固有の不合格要因）
- 個人提供メディアの選択だけが 2FA

```html
<!-- 失敗例：F109 -->
<p>パスワードの 2・6・末尾の文字を入力してください</p>
<input maxlength="1">
<input maxlength="1">
<input maxlength="1">
```

```html
<!-- 改善例 -->
<label for="otp">認証アプリの 6 桁コード</label>
<input
  id="otp"
  name="otp"
  type="text"
  inputmode="numeric"
  autocomplete="one-time-code"
  maxlength="6"
>
<!-- ペースト可。さらに通知承認などの代替もあるとよい -->
```

> **参照**
> - [Failure F109: Preventing password or code re-entry in the same format](https://www.w3.org/WAI/WCAG22/Techniques/failures/F109)

## 実務での達成方針

```text
優先順位の目安：
  1. 認知機能テスト自体を認証の必須経路から外す
  2. どうしてもパスワードを使うなら、H100 + ペースト許可
  3. マジックリンク / パスキー / OAuth を代替として用意
  4. 2FA は通知承認・キー・QR など転写不要な手段を含める
  5. 物体認識・個人コンテンツ CAPTCHA を認証必須にしない

3.3.8 達成済みでも、次を点検する：
  □ 物体認識 CAPTCHA に依存していないか
  □ 「登録済み画像を選べ」系がないか
  □ それらの代替として、認知機能テストなし経路があるか
```

## テスト・確認方法

### 手動テスト

- ログイン・2FA・アカウント復旧を洗い出す
- 3.3.8 の観点（ペースト、自動入力、代替）を確認する
- 加えて、物体認識・個人コンテンツのステップがないか確認する
- ある場合、認知機能テストなしの代替で完走できるか確認する
- 代替なしなら 3.3.9 不合格

```text
チェックリスト：
1. 3.3.8 の要件（代替 or 支援機構）を満たしているか
2. 物体認識 CAPTCHA が認証必須になっていないか
3. 個人提供メディアの識別が認証必須になっていないか
4. パスワード／OTP を元の形式のままペーストできるか
5. 自動入力をブロックしていないか
6. 2FA に転写・認識不要な経路があるか
7. アカウント復旧でも認識系テストだけになっていないか
8. AAA を標榜するなら、認識例外に頼る設計を捨てられているか
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)
> - [wcag/3-3-8-accessible-authentication-minimum.md — 3.3.8](./3-3-8-accessible-authentication-minimum.md)

## まとめ

- 3.3.9 は、認証で認知機能テストを必須にしない達成基準（レベル AAA・WCAG 2.2 新規）
- 3.3.8 の強化版で、認められる例外は**代替**と**支援機構**のみ
- 物体認識 CAPTCHA や個人提供コンテンツの識別は、3.3.9 では例外にならない
- 手法は G218・H100・WebAuthn・OAuth など 3.3.8 と同じ方向
- AAA を目指すなら、認識系テストを認証の必須経路から外す
- ペースト不可や一部文字入力は、3.3.8 / 3.3.9 共通の失敗（F109）
