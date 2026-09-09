# 3.3.8 アクセシブルな認証（最低限）（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.3.8「アクセシブルな認証（最低限）」（Accessible Authentication (Minimum)）
- レベル **AA**
- WCAG **2.2 で新規追加**（`[New]`）
- 原則3「理解可能（Understandable）」> ガイドライン 3.3「入力支援」に属する
- 認証プロセスの各ステップで、**認知機能テスト**を必須にしないこと
- 認知機能テストを使う場合は、次のいずれかが必要
  1. **代替（Alternative）**：認知機能テストに頼らない別の認証方法
  2. **支援機構（Mechanism）**：テスト完了を助ける仕組み（パスワードマネージャ、コピー＆ペーストなど）
  3. **オブジェクト認識（Object Recognition）**：物体の認識である
  4. **個人コンテンツ（Personal Content）**：ユーザーがサイトに提供した非テキストコンテンツの識別である
- 対象は**既存アカウントへのログイン・認証**。アカウント新規作成そのものは主眼ではない

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.8 Accessible Authentication (Minimum)](https://www.w3.org/TR/WCAG22/#accessible-authentication-minimum)
> - [Understanding SC 3.3.8 Accessible Authentication (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html)

## 達成基準の原文（要約）

- 認証プロセスのいずれのステップでも、認知機能テスト（パスワードの記憶やパズルの解答など）を必須にしない
- ただし、そのステップが次のいずれかを提供する場合を除く
  - **代替**：認知機能テストに依存しない別の認証方法
  - **支援機構**：認知機能テストの完了を支援する仕組み
  - **オブジェクト認識**：物体を認識するテストである
  - **個人コンテンツ**：ユーザーがサイトに提供した非テキストコンテンツを識別するテストである
- 注：オブジェクト認識・個人コンテンツは画像・映像・音声で表されてもよい
- 注：支援機構の例には、パスワードマネージャによる入力支援、再入力負担を減らすコピー＆ペーストがある

> **参照**
> - [WCAG 2.2 — Success Criterion 3.3.8](https://www.w3.org/TR/WCAG22/#accessible-authentication-minimum)

## なぜ必要か

- サイト固有のパスワードを覚えることは、認知障害のある人にとって非常に高い負担になる
- ワンタイムコードの書き写し、パズル CAPTCHA、パスフレーズの一部抽出なども障壁になる
- 記憶・転写・計算・パズルは、一部のユーザーをログインから排除しうる
- パスワードマネージャやコピー＆ペーストを許せば、記憶に頼らず認証できる
- セキュリティを下げずに、認知負荷を減らせる

> **参照**
> - [Understanding SC 3.3.8 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html#intent)
> - [Understanding SC 3.3.8 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html#benefits)

## 認知機能テストとは

- ユーザーに**記憶・操作・転写**を求めるタスク
- Understanding の定義例：
  - **記憶**：ユーザー名・パスワード・文字列・画像・パターンの暗記
  - **転写**：文字の書き写し
  - **正しいスペリング**
  - **計算**
  - **パズルの解答**
- **対象外**：氏名・メール・電話番号など、個人に固有でサイト横断で一貫している識別子
- タッチ画面のスワイプパターンも、記憶を求めるなら認知機能テストになりうる

```text
認知機能テストの例：
  サイト固有パスワードを暗記して手入力
  歪んだ文字 CAPTCHA を書き写す
  SMS の 6 桁コードを目で見て手入力（ペースト不可）
  「パスワードの 2・6・末尾文字だけ入力」

認知機能テストではない例：
  メールアドレスの入力（個人に固有で一貫）
  指紋・顔認証・ハードウェアキーのボタン押下
  「ログイン試行を承認する」通知への確認
  QR コードをアプリでスキャンして確認
```

> **参照**
> - [Understanding SC 3.3.8 — Key Terms: cognitive function test](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html)

## 4つの例外パス

| パス | 内容 | 実務例 |
|---|---|---|
| **代替** | 認知機能テストなしの別認証 | マジックリンク、WebAuthn、OAuth |
| **支援機構** | テストを助ける仕組み | パスワードマネージャ自動入力、ペースト許可 |
| **オブジェクト認識** | 物体の認識 | 画像から信号機を選ぶ系 CAPTCHA |
| **個人コンテンツ** | ユーザー提供の非テキスト識別 | 自分で登録した写真の選択 |

- オブジェクト認識・個人コンテンツは例外として認められるが、認知アクセシビリティには十分ではない
- より包摂的にするなら **3.3.9（高度）** を目指し、これらにも頼らない
- 多要素認証では、**すべてのステップ**に認知機能テストなしの経路が必要

> **参照**
> - [Understanding SC 3.3.8 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html#intent)

## ログインフォームでのポイント

- ユーザー名／メール + パスワードでも、次を満たせば達成しうる
  - ブラウザ・パスワードマネージャがフィールドを埋められる
  - ユーザーのコピー＆ペーストをブロックしない
- 一般に、**1.3.5 入力目的の特定**と**4.1.2 名前・役割・値**を満たすマークアップが有効
- スクリプトで自動入力を妨害したり、ペーストを禁止したりすると不合格になりやすい
- アカウント復旧・パスワード変更も認証の一部。そこでも認知機能テストだけの経路にしない

```text
合格しやすい条件：
  autocomplete が適切
  accessible name が明確
  paste 可能
  パスワードマネージャが動作する

不合格になりやすい条件：
  paste 禁止
  自動入力を JS でクリア
  文字ごとに分割入力して一括ペースト不可
  「○文字目だけ入力」形式
```

> **参照**
> - [Understanding SC 3.3.8 — Login forms](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html#intent)
> - [wcag/1-3-5-identify-input-purpose.md — 1.3.5 入力目的の特定](./1-3-5-identify-input-purpose.md)

## 十分な手法

| 手法 | 内容 |
|---|---|
| **G218** | メールリンク認証（マジックリンク） |
| **H100** | 適切にマークアップしたメール・パスワード入力 |
| （将来手法候補） | WebAuthn をユーザー名／パスワードの代替にする |
| （将来手法候補） | OAuth によるサードパーティログイン |
| （将来手法候補） | 2つの手法を組み合わせた二要素認証 |

> **参照**
> - [Understanding SC 3.3.8 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html#techniques)

## 手法1：適切にマークアップしたログイン入力（H100）

- `label`・`autocomplete`・ペースト許可が要点
- ユーザーエージェントとパスワードマネージャがフィールドを認識できるようにする

```html
<!-- 良い例：認識・自動入力・ペースト可能なログイン -->
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
<!-- 悪い例：ペースト禁止 + 曖昧な名前 -->
<form method="post" action="/login">
  <input name="u" type="text" placeholder="ID">
  <input
    name="p"
    type="password"
    placeholder="PASS"
    onpaste="return false"
  >
  <button type="submit">ログイン</button>
</form>
```

> **参照**
> - [Technique H100: Providing properly marked up email and password inputs](https://www.w3.org/WAI/WCAG22/Techniques/html/H100)

## 手法2：メールリンク認証（G218）

- メールアドレスを送り、届いたリンクをクリックしてログイン
- パスワード記憶が不要な**代替**経路になる
- 既存のパスワードログインに加えて提供してもよい

```html
<!-- 良い例：マジックリンクの入口 -->
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
<p>届いたメールのリンクを開くとログインできます。パスワードの記憶は不要です。</p>
```

```text
流れ：
  1. ユーザーがメールアドレスを送信
  2. サイトが期限付きトークン付きリンクをメール送信
  3. ユーザーがリンクを開く
  4. サイト側でログイン完了
```

> **参照**
> - [Technique G218: Email link authentication](https://www.w3.org/WAI/WCAG22/Techniques/general/G218)

## 手法3：WebAuthn / OAuth などの代替認証

- 指紋・顔・PIN・セキュリティキーなど、端末の認証を使う（WebAuthn）
- Google / Apple 等のサードパーティログイン（OAuth）
- サイト側が特定の生体情報を強制する必要はない。ユーザーが自分に合う方法を使えることが重要

```html
<!-- 良い例：パスワード以外の選択肢を明示 -->
<section aria-labelledby="login-title">
  <h1 id="login-title">ログイン</h1>
  <!-- メール + パスワード（H100） -->
  <button type="button">パスキーでログイン</button>
  <button type="button">Google でログイン</button>
  <a href="/login/email-link">メールリンクでログイン</a>
</section>
```

> **参照**
> - [Understanding SC 3.3.8 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html#examples)
> - [Web Authentication API（MDN）](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API)

## 二要素認証（2FA）での注意

- パスワード + 確認コードでも、**各ステップ**に支援または代替が必要
- 確認コードは、少なくとも**ペースト可能**で、自動入力を妨げないこと
- 桁ごとに分かれた入力で、全体ペーストができないと失敗しやすい（F109）
- 次は認知機能テストになりにくい
  - USB キーのボタン押下
  - 端末への承認通知
  - QR コードのスキャン確認
  - OS の生体認証

```html
<!-- 良い例：ワンタイムコードを一括ペースト可能 -->
<label for="otp">認証アプリの 6 桁コード</label>
<input
  id="otp"
  name="otp"
  type="text"
  inputmode="numeric"
  autocomplete="one-time-code"
  maxlength="6"
>
<!-- ペースト可。分割フィールドでも先頭への一括ペーストで全体が埋まる実装なら可 -->
```

```html
<!-- 悪い例：桁分割で一括ペースト不可（F109） -->
<fieldset>
  <legend>6 桁コードを1文字ずつ入力</legend>
  <input maxlength="1" inputmode="numeric">
  <input maxlength="1" inputmode="numeric">
  <input maxlength="1" inputmode="numeric">
  <input maxlength="1" inputmode="numeric">
  <input maxlength="1" inputmode="numeric">
  <input maxlength="1" inputmode="numeric">
</fieldset>
<!-- 全体を一度に貼れない -->
```

```html
<!-- 悪い例：パスワードの一部だけ手入力 -->
<p>パスワードの 2・6・末尾の文字を入力してください</p>
<input maxlength="1">
<input maxlength="1">
<input maxlength="1">
```

> **参照**
> - [Understanding SC 3.3.8 — Two-factor authentication](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html#intent)
> - [Failure F109: Preventing password or code re-entry in the same format](https://www.w3.org/WAI/WCAG22/Techniques/failures/F109)

## CAPTCHA との関係

- 認証プロセス内の CAPTCHA も、認知機能テストになりうる
- 歪んだ文字の書き写しは不合格になりやすい
- 音声 CAPTCHA でも、聞き取って書き写す必要があれば**代替例外を満たさない**
- オブジェクト認識 CAPTCHA は 3.3.8 の例外に入りうるが、可能なら避ける
- より良いのは、認証そのものに認知テストを入れない設計（3.3.9）

```text
CAPTCHA の整理：
  文字書き写し（視覚 / 音声）→ 認知機能テスト。代替や支援が必要
  物体認識（信号機を選ぶ等）→ 3.3.8 では例外になりうる
  個人が登録した画像の選択 → 個人コンテンツ例外になりうる
  そもそも CAPTCHA なし / 別認証 → 最も望ましい
```

> **参照**
> - [Understanding SC 3.3.8 — Object Recognition](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum.html#intent)

## 3.3.7・3.3.9 との関係

| 達成基準 | レベル | 焦点 |
|---|---|---|
| 3.3.7 冗長な入力 | A | 同一プロセス内の再入力回避 |
| 3.3.8 アクセシブルな認証（最低限） | AA | 認証で認知機能テストを必須にしない（例外あり） |
| 3.3.9 アクセシブルな認証（高度） | AAA | オブジェクト認識・個人コンテンツ例外も使わない |

- 3.3.7 はフォーム全般の再入力。3.3.8 は**認証**に特化
- パスワードのブラウザ自動入力は、3.3.8 では有効な支援機構
- 3.3.9 は 3.3.8 より厳しく、認識系 CAPTCHA 例外に頼れない

> **参照**
> - [wcag/3-3-7-redundant-entry.md — 3.3.7 冗長な入力](./3-3-7-redundant-entry.md)

## よくある失敗例

- パスワード欄で `paste` を禁止する
- パスワードマネージャの自動入力をスクリプトで消す
- OTP を桁分割し、一括ペーストできない（F109）
- 「パスワードの○文字目だけ入力」を唯一の手段にする
- 文字書き写し CAPTCHA だけでログインできる
- 音声 CAPTCHA があるが、結局転写が必要
- 2FA の第2要素だけが認知機能テストで、代替がない
- アカウント復旧がパズルや暗記だけ

```html
<!-- 失敗例 -->
<input
  type="password"
  onpaste="return false"
  autocomplete="off"
>
<!-- 記憶して手入力するしかない -->
```

```html
<!-- 改善例 -->
<input
  id="password"
  type="password"
  autocomplete="current-password"
>
<!-- マネージャ入力・ペースト可。さらにパスキー等の代替があるとより良い -->
```

## テスト・確認方法

### 手動テスト

- ログイン・2FA・アカウント復旧の経路を洗い出す
- 各ステップに認知機能テストがあるか確認する
- ある場合、代替・支援機構・オブジェクト認識・個人コンテンツのいずれかか確認する
- パスワード／OTP 欄でペーストできるか試す
- パスワードマネージャやブラウザ自動入力が動くか試す
- 桁分割 OTP で、全体を一度に貼れるか確認する
- CAPTCHA がある場合、書き写し以外の経路があるか確認する

```text
チェックリスト：
1. 認証・2FA・復旧の全ステップを洗い出したか
2. 認知機能テストがあるステップに、4例外パスのいずれかがあるか
3. パスワード／コードを元の形式のまま入力・ペーストできるか
4. autocomplete と accessible name が適切か（H100）
5. 自動入力・ペーストをブロックしていないか
6. 文字書き写し CAPTCHA だけになっていないか
7. 2FA に、転写不要な選択肢（通知承認・セキュリティキー等）があるか
8. 可能ならマジックリンク / WebAuthn / OAuth などの代替があるか
```

> **参照**
> - [Failure F109 — Tests](https://www.w3.org/WAI/WCAG22/Techniques/failures/F109)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.3.8 は、認証で認知機能テストを必須にしない達成基準（レベル AA・WCAG 2.2 新規）
- 使う場合は、代替・支援機構・オブジェクト認識・個人コンテンツのいずれかが必要
- 実務では、適切なマークアップ + ペースト許可（H100）と、マジックリンク（G218）が基本
- パスワードの一部入力や、一括ペースト不可の分割 OTP は代表的な失敗（F109）
- オブジェクト認識 CAPTCHA は例外になりうるが、可能なら避け、3.3.9 を意識する
- 多要素認証では、認知機能テストなしで通れる経路を全ステップに用意する
