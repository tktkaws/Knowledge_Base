# CSRF対策の基礎 — トークンによるリクエスト検証

## CSRFとは

- CSRF（Cross-Site Request Forgery: クロスサイトリクエストフォージェリ）は、ユーザーが意図しないリクエストを外部サイトから送信させる攻撃
- 「リクエスト偽造」とも呼ばれる
- ユーザーがログイン中のサイトに対して、攻撃者のサイトから不正なリクエストを送信させる
- ブラウザが自動的にCookieを付与する仕組みを悪用する

> 参照: [OWASP — Cross-Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf)

---

## CSRFの攻撃の仕組み

### 攻撃の流れ

```
1. ユーザーがbank.example.comにログイン中（セッションCookieがブラウザに保存されている）

2. ユーザーが攻撃者のサイト（evil.example.com）にアクセス

3. 攻撃者のサイトに埋め込まれたフォームやスクリプトが
   bank.example.comに対してリクエストを自動送信

4. ブラウザはbank.example.comへのリクエストにセッションCookieを自動付与

5. bank.example.comはCookieが有効なため、正規のリクエストとして処理してしまう

ユーザー ──→ evil.example.com（攻撃者のサイト）
   │
   │  攻撃者のページが自動的にリクエストを送信
   ↓
bank.example.com ← Cookieが自動付与されるため認証済みとして処理
```

### 攻撃の具体例

```html
<!-- 攻撃者のサイトに設置されたページ -->

<!-- 例1: 隠しフォームで自動送信 -->
<form action="https://bank.example.com/transfer" method="POST" id="csrf-form">
  <input type="hidden" name="to" value="attacker-account">
  <input type="hidden" name="amount" value="1000000">
</form>
<script>document.getElementById('csrf-form').submit();</script>

<!-- 例2: 画像タグでGETリクエストを送信（GETで状態変更する設計の場合） -->
<img src="https://bank.example.com/transfer?to=attacker&amount=1000000">

<!-- 例3: fetchで送信 -->
<script>
  fetch('https://bank.example.com/transfer', {
    method: 'POST',
    credentials: 'include',  // Cookieを含める
    body: new URLSearchParams({ to: 'attacker', amount: '1000000' }),
  });
</script>
```

### CSRFとXSSの違い

| 項目 | CSRF | XSS |
|------|------|-----|
| 攻撃の起点 | 攻撃者のサイト（外部サイト） | 標的サイト内 |
| 悪用する信頼 | サーバーがユーザーを信頼すること | ユーザーがサイトを信頼すること |
| 実行される操作 | 既存の機能（送金、設定変更など） | 任意のスクリプト |
| Cookie | 自動送信されることを悪用 | Cookie自体を読み取れる |

---

## CSRF対策の方法

### 1. CSRFトークン（Synchronizer Token Pattern）

- サーバーがランダムなトークンを生成し、フォームに埋め込む
- リクエスト受信時にトークンの一致を検証する
- 攻撃者は正規のトークンを知ることができないため、リクエストが拒否される

```
仕組み:

1. サーバーがページ表示時にランダムなトークンを生成
2. トークンをセッションに保存 + HTMLフォームに埋め込み
3. ユーザーがフォーム送信時にトークンも一緒に送信
4. サーバーがセッションのトークンと送信されたトークンを比較
5. 一致すれば正規のリクエスト、不一致なら拒否

攻撃者のサイトから:
→ トークンを知らないため、正しいトークンを含むリクエストを作れない
→ 同一オリジンポリシーにより、標的サイトのHTMLを読み取れない
```

#### Express + csurfでの実装例

```js
import express from 'express';
import csrf from 'csurf';
import cookieParser from 'cookie-parser';

const app = express();
app.use(cookieParser());
app.use(express.urlencoded({ extended: false }));

const csrfProtection = csrf({ cookie: true });

// フォーム表示（GETリクエスト）
app.get('/transfer', csrfProtection, (req, res) => {
  // トークンをフォームに埋め込む
  res.send(`
    <form action="/transfer" method="POST">
      <input type="hidden" name="_csrf" value="${req.csrfToken()}">
      <input type="text" name="to" placeholder="送金先">
      <input type="number" name="amount" placeholder="金額">
      <button type="submit">送金</button>
    </form>
  `);
});

// フォーム送信（POSTリクエスト）
app.post('/transfer', csrfProtection, (req, res) => {
  // csrfミドルウェアがトークンを自動検証
  // トークンが不一致なら403エラーを返す
  // ここに到達 = トークン検証成功
  res.send('送金が完了しました');
});

// CSRFエラーハンドリング
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    return res.status(403).send('不正なリクエストです');
  }
  next(err);
});
```

#### SPAでのCSRFトークン

```js
// サーバー側: APIエンドポイントでトークンを返す
app.get('/api/csrf-token', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// クライアント側: トークンを取得してリクエストに含める
async function transfer(to, amount) {
  // まずトークンを取得
  const tokenRes = await fetch('/api/csrf-token', { credentials: 'include' });
  const { csrfToken } = await tokenRes.json();

  // トークンをヘッダーに含めてリクエスト
  const res = await fetch('/api/transfer', {
    method: 'POST',
    credentials: 'include',
    headers: {
      'Content-Type': 'application/json',
      'CSRF-Token': csrfToken,
    },
    body: JSON.stringify({ to, amount }),
  });

  return res.json();
}
```

### 2. SameSite Cookie属性

- Cookieに`SameSite`属性を設定し、クロスサイトリクエスト時のCookie送信を制限する
- 最も手軽なCSRF対策。CSRFトークンと併用が推奨

| 値 | 動作 |
|---|------|
| `Strict` | クロスサイトリクエストでは一切Cookieを送信しない |
| `Lax` | クロスサイトのトップレベルナビゲーション（リンククリック）のGETのみCookieを送信 |
| `None` | 常にCookieを送信（`Secure`属性が必須） |

```
# 悪い例：SameSite属性なし → ブラウザのデフォルト動作に依存
Set-Cookie: session_id=abc123; Path=/;

# 良い例：Lax（推奨。リンク遷移ではCookieが送信されるのでUXを損なわない）
Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure; SameSite=Lax;

# より厳格：Strict（外部サイトからのリンク遷移でもCookieが送信されない）
Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure; SameSite=Strict;
```

```
SameSite=Laxの動作:

外部サイトのリンクをクリック → GET https://example.com/ → Cookie送信される（○）
外部サイトのフォーム送信     → POST https://example.com/ → Cookie送信されない（×）
外部サイトのfetch            → POST https://example.com/ → Cookie送信されない（×）
外部サイトのiframe           → GET https://example.com/  → Cookie送信されない（×）
```

- 現在の主要ブラウザのデフォルトは`Lax`だが、明示的に設定するのがベストプラクティス
- `SameSite=Lax`だけではGETリクエストによる状態変更を防げないため、CSRFトークンとの併用が望ましい

### 3. Originヘッダーの検証

- ブラウザはPOSTリクエスト等にOriginヘッダーを自動付与する
- サーバー側でOriginヘッダーを検証し、許可されたオリジンからのリクエストのみ受け付ける

```js
// Originヘッダーの検証
function validateOrigin(req, res, next) {
  const origin = req.headers.origin;
  const referer = req.headers.referer;

  // Originヘッダーがある場合（POST等）
  if (origin) {
    if (origin !== 'https://example.com') {
      return res.status(403).send('不正なオリジンです');
    }
    return next();
  }

  // Originがない場合はRefererを確認（GETからの遷移等）
  if (referer) {
    const refererUrl = new URL(referer);
    if (refererUrl.origin !== 'https://example.com') {
      return res.status(403).send('不正なリファラーです');
    }
    return next();
  }

  // Origin / Refererが両方ない場合の判断はサービスによる
  return res.status(403).send('オリジン情報がありません');
}

app.post('/transfer', validateOrigin, (req, res) => {
  // 送金処理
});
```

- Originヘッダーはブラウザが自動付与するため、JavaScriptから偽装できない
- ただし一部のリクエストでOriginが送信されないケースがあるため、CSRFトークンとの併用が安全

### 4. カスタムリクエストヘッダーの検証

- XMLHttpRequestやfetchで送信されるカスタムヘッダーは、CORSのプリフライトなしにはクロスオリジンから送信できない
- カスタムヘッダーの存在をチェックすることでCSRF対策になる

```js
// クライアント側: カスタムヘッダーを付与
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest',  // カスタムヘッダー
  },
  body: JSON.stringify({ to: 'user', amount: 1000 }),
});

// サーバー側: カスタムヘッダーの存在を検証
function requireAjax(req, res, next) {
  if (req.headers['x-requested-with'] !== 'XMLHttpRequest') {
    return res.status(403).send('不正なリクエストです');
  }
  next();
}

app.post('/api/transfer', requireAjax, (req, res) => {
  // 送金処理
});
```

- HTML formやimgタグからはカスタムヘッダーを送信できないため、これらの攻撃を防げる
- ただしCORS設定が甘い場合（`Access-Control-Allow-Headers: *`）は回避される可能性がある

---

## GETリクエストで状態を変更してはいけない理由

```
# 悪い例：GETで状態変更（削除、更新など）
GET /delete-account?confirm=true
GET /transfer?to=attacker&amount=1000

# → imgタグやリンクで攻撃可能（CSRFトークンが効きにくい）
<img src="https://example.com/delete-account?confirm=true">
```

- GETリクエストはキャッシュやプリフェッチの対象になるため、意図しない実行のリスクがある
- 状態を変更する操作はPOST/PUT/DELETEで実装する（REST原則）
- SameSite=LaxでもGETリクエストはCookieが送信されるため、GETでの状態変更は危険

---

## フレームワーク別のCSRF対策

### Next.js（App Router）

```js
// Server Actionsは自動的にCSRF対策が組み込まれている
// Origin / Hostヘッダーの検証が行われる

// app/transfer/page.jsx
export default function TransferPage() {
  async function transfer(formData) {
    'use server';
    const to = formData.get('to');
    const amount = formData.get('amount');
    // Server Actionは自動的にCSRFから保護される
  }

  return (
    <form action={transfer}>
      <input name="to" placeholder="送金先" />
      <input name="amount" type="number" placeholder="金額" />
      <button type="submit">送金</button>
    </form>
  );
}
```

```js
// APIルート（Route Handlers）では自前のCSRF対策が必要
// app/api/transfer/route.js
export async function POST(request) {
  // Originヘッダーの検証
  const origin = request.headers.get('origin');
  if (origin !== process.env.ALLOWED_ORIGIN) {
    return new Response('Forbidden', { status: 403 });
  }

  const body = await request.json();
  // 送金処理
}
```

### Expressの一般的なパターン

```js
import express from 'express';
import csrf from 'csurf';
import cookieParser from 'cookie-parser';

const app = express();
app.use(cookieParser());
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

// CSRF保護を全ルートに適用
const csrfProtection = csrf({ cookie: { httpOnly: true, secure: true, sameSite: 'lax' } });

// GETリクエストにはCSRF不要（状態変更しない前提）
app.get('/form', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// POST/PUT/DELETEにCSRF保護を適用
app.post('/api/data', csrfProtection, (req, res) => {
  // CSRFトークン検証済み
});
```

---

## Double Submit Cookie パターン

- セッションを使わずにCSRF対策を実装するパターン
- Cookieとリクエストボディやヘッダーに同じトークンを設定し、一致を検証する
- ステートレスなAPIサーバーで有用

```
仕組み:

1. サーバーがランダムなトークンをCookieに設定
   Set-Cookie: csrf_token=random123; Path=/; SameSite=Lax;

2. クライアントがCookieからトークンを読み取り、リクエストヘッダーにも設定
   Cookie: csrf_token=random123
   X-CSRF-Token: random123

3. サーバーがCookieのトークンとヘッダーのトークンを比較
   → 一致すれば正規のリクエスト

なぜ安全か:
→ 攻撃者はCookieは自動送信させられるが、ヘッダーにトークンを設定するには
  JavaScriptでCookieを読む必要がある
→ 同一オリジンポリシーにより、攻撃者のサイトからは標的サイトのCookieを読めない
```

```js
// サーバー側の実装
import crypto from 'crypto';

// トークン設定
app.get('/api/csrf-token', (req, res) => {
  const token = crypto.randomBytes(32).toString('hex');
  res.cookie('csrf_token', token, {
    httpOnly: false,  // JSから読み取り可能にする必要がある
    secure: true,
    sameSite: 'lax',
  });
  res.json({ ok: true });
});

// トークン検証ミドルウェア
function verifyDoubleCsrf(req, res, next) {
  const cookieToken = req.cookies.csrf_token;
  const headerToken = req.headers['x-csrf-token'];

  if (!cookieToken || !headerToken || cookieToken !== headerToken) {
    return res.status(403).json({ error: 'CSRFトークンが無効です' });
  }
  next();
}

app.post('/api/data', verifyDoubleCsrf, (req, res) => {
  // CSRF検証済み
});
```

```js
// クライアント側
function getCsrfToken() {
  const match = document.cookie.match(/csrf_token=([^;]+)/);
  return match ? match[1] : '';
}

fetch('/api/data', {
  method: 'POST',
  credentials: 'include',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': getCsrfToken(),
  },
  body: JSON.stringify({ key: 'value' }),
});
```

---

## CSRF対策チェックリスト

| チェック項目 | 対策内容 |
|------------|---------|
| CSRFトークン | 状態変更を伴うフォーム・APIにトークンを導入 |
| SameSite Cookie | セッションCookieに`SameSite=Lax`以上を設定 |
| GETの安全性 | GETリクエストで状態変更を行わない |
| Originヘッダー検証 | サーバー側でリクエスト元を検証 |
| Cookie属性 | `HttpOnly; Secure; SameSite=Lax`を設定 |
| フレームワーク | フレームワーク組み込みのCSRF対策を有効化 |
| CORS設定 | `Access-Control-Allow-Origin`を適切に制限 |

---

## まとめ

- CSRFはブラウザがCookieを自動送信する仕組みを悪用した攻撃
- 最も一般的な対策はCSRFトークン。サーバーが生成したランダムなトークンをリクエストに含めて検証する
- `SameSite=Lax`Cookieは手軽で効果的な対策だが、GETによる状態変更には無力
- 状態を変更する操作は必ずPOST/PUT/DELETEで実装する
- CSRFトークン + SameSite Cookie + Origin検証の多層防御が推奨
- Next.jsのServer Actionsなど、フレームワークが自動的にCSRF対策を行うケースも増えている
