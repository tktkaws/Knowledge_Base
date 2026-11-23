# Webフォームからのメール送信 — セキュアな実装のポイント

## Webフォームとメール送信の関係

- お問い合わせフォーム、会員登録、パスワードリセットなどでメール送信が必要になる
- フォーム経由のメール送信には特有のセキュリティリスクがある
- 不適切な実装はスパムの踏み台やインジェクション攻撃の原因になる

## メール送信の基本構成

```
ユーザー → Webフォーム → サーバーサイド処理 → メール配信サービス → 受信者
              ↓
         バリデーション
         サニタイズ
         レート制限
         CSRF対策
```

- フロントエンドだけでのバリデーションは不十分。サーバーサイドでの検証が必須
- メール送信処理はサーバーサイドで行う（クライアントからSMTPに直接接続しない）

## メールヘッダインジェクション

### 攻撃の仕組み

- ユーザー入力がメールヘッダにそのまま挿入されると、追加のヘッダを注入できる
- 改行コード（`\r\n`）を含む入力で任意のヘッダフィールドやBCC宛先を追加される

```
悪い例: ユーザー入力をそのままヘッダに使用

入力値:
  名前: "攻撃者\r\nBCC: victim1@example.com, victim2@example.com"

生成されるヘッダ:
  From: 攻撃者
  BCC: victim1@example.com, victim2@example.com  ← 注入されたヘッダ
  Subject: お問い合わせ
```

### 対策

```js
// 悪い例：ユーザー入力を直接ヘッダに設定
const name = req.body.name;
const mailOptions = {
  from: `${name} <noreply@example.com>`,  // 改行が含まれるとインジェクション
  to: 'admin@example.com',
  subject: `お問い合わせ: ${req.body.subject}`,  // ここも危険
};

// 良い例：改行コードを除去してからヘッダに設定
function sanitizeHeader(value) {
  return value.replace(/[\r\n]/g, '').trim();
}

const name = sanitizeHeader(req.body.name);
const subject = sanitizeHeader(req.body.subject);
const mailOptions = {
  from: `${name} <noreply@example.com>`,
  to: 'admin@example.com',
  subject: `お問い合わせ: ${subject}`,
};
```

- Nodemailerなど主要なライブラリは内部でヘッダインジェクション対策を行っている
- ただしライブラリ任せにせず、入力値のサニタイズは自分でも行うべき

- 参照: [OWASP — Mail Header Injection](https://owasp.org/www-community/attacks/Mail_Header_Injection)

## スパムの踏み台にされないための対策

### 宛先の固定

```js
// 悪い例：ユーザー入力で宛先を指定できてしまう
app.post('/contact', (req, res) => {
  sendMail({
    to: req.body.to,  // 任意のアドレスにメールを送れてしまう
    subject: 'お問い合わせ',
    body: req.body.message,
  });
});

// 良い例：宛先をサーバー側で固定する
app.post('/contact', (req, res) => {
  sendMail({
    to: 'contact@example.com',  // 宛先は固定
    subject: 'お問い合わせ',
    body: sanitize(req.body.message),
  });
});
```

### レート制限

```js
// express-rate-limitを使用した例
import rateLimit from 'express-rate-limit';

const contactLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15分間
  max: 5,                     // 最大5回
  message: '送信回数の上限に達しました。しばらく時間をおいてから再度お試しください。',
  standardHeaders: true,
  legacyHeaders: false,
});

app.post('/contact', contactLimiter, (req, res) => {
  // メール送信処理
});
```

- IPアドレスベースのレート制限に加え、セッション単位の制限も検討する
- フォーム送信後のリダイレクト（PRGパターン）で重複送信を防止する

### CAPTCHA / botスクリーニング

```html
<!-- reCAPTCHA v3の実装例 -->
<form action="/contact" method="POST">
  <input type="text" name="name" required>
  <input type="email" name="email" required>
  <textarea name="message" required></textarea>
  <input type="hidden" name="g-recaptcha-response" id="recaptchaResponse">
  <button type="submit">送信</button>
</form>

<script src="https://www.google.com/recaptcha/api.js?render=SITE_KEY"></script>
<script>
  document.querySelector('form').addEventListener('submit', async (e) => {
    e.preventDefault();
    const token = await grecaptcha.execute('SITE_KEY', { action: 'contact' });
    document.getElementById('recaptchaResponse').value = token;
    e.target.submit();
  });
</script>
```

```js
// サーバー側でのreCAPTCHA検証
app.post('/contact', async (req, res) => {
  const recaptchaToken = req.body['g-recaptcha-response'];

  const verifyUrl = 'https://www.google.com/recaptcha/api/siteverify';
  const response = await fetch(verifyUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: `secret=${process.env.RECAPTCHA_SECRET}&response=${recaptchaToken}`,
  });
  const data = await response.json();

  if (!data.success || data.score < 0.5) {
    return res.status(400).json({ error: 'bot判定されました' });
  }

  // メール送信処理
});
```

### ハニーポットフィールド

```html
<!-- CSSで非表示にしたダミーフィールド -->
<form action="/contact" method="POST">
  <!-- ハニーポット: 人間には見えないが、botは入力する -->
  <div style="position: absolute; left: -9999px;" aria-hidden="true">
    <input type="text" name="website" tabindex="-1" autocomplete="off">
  </div>

  <input type="text" name="name" required>
  <textarea name="message" required></textarea>
  <button type="submit">送信</button>
</form>
```

```js
// サーバー側でハニーポットをチェック
app.post('/contact', (req, res) => {
  // ハニーポットに値が入っていたらbot
  if (req.body.website) {
    // ログに記録しつつ、成功レスポンスを返す（botに気づかれないため）
    console.log('bot検知:', req.ip);
    return res.json({ success: true });
  }

  // 通常のメール送信処理
});
```

## CSRF対策

- フォーム送信にはCSRF（Cross-Site Request Forgery）対策が必須
- 外部サイトから自サイトのフォームに対してリクエストを偽造される可能性がある

```js
// csurfミドルウェアを使用した例
import csrf from 'csurf';

const csrfProtection = csrf({ cookie: true });

// フォーム表示時にトークンを埋め込む
app.get('/contact', csrfProtection, (req, res) => {
  res.render('contact', { csrfToken: req.csrfToken() });
});

// 送信時にトークンを検証
app.post('/contact', csrfProtection, (req, res) => {
  // CSRFトークンが一致しなければ403エラー
  // メール送信処理
});
```

```html
<form action="/contact" method="POST">
  <input type="hidden" name="_csrf" value="<%= csrfToken %>">
  <!-- フォームフィールド -->
</form>
```

## 入力バリデーション

### メールアドレスの検証

```js
// 悪い例：正規表現だけで完全な検証を試みる
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;

// 良い例：基本的な形式チェック + ライブラリによる検証
import { isEmail } from 'validator';

function validateEmail(email) {
  if (typeof email !== 'string') return false;
  if (email.length > 254) return false;  // RFC 5321の最大長
  return isEmail(email);
}
```

- メールアドレスの正規表現による完全な検証は非常に複雑
- `validator`ライブラリなど実績のあるツールを使用する
- 最終的な検証は確認メールの送信で行う（メールアドレスの実在確認）

### 本文の検証

```js
function validateContactForm(body) {
  const errors = [];

  // 名前: 1〜100文字
  if (!body.name || body.name.trim().length === 0) {
    errors.push('名前を入力してください');
  } else if (body.name.length > 100) {
    errors.push('名前は100文字以内で入力してください');
  }

  // メールアドレス
  if (!validateEmail(body.email)) {
    errors.push('有効なメールアドレスを入力してください');
  }

  // メッセージ: 1〜5000文字
  if (!body.message || body.message.trim().length === 0) {
    errors.push('メッセージを入力してください');
  } else if (body.message.length > 5000) {
    errors.push('メッセージは5000文字以内で入力してください');
  }

  return errors;
}
```

## 自動返信メールの注意点

### バックスキャッター問題

- 入力されたメールアドレスに自動返信すると、なりすまされた場合に無関係な人にメールが届く
- これを「バックスキャッター」と呼び、スパムの一種と見なされる

```
悪い例:
1. 攻撃者がフォームに victim@example.com を入力
2. サーバーが victim@example.com に自動返信
3. victim は身に覚えのないメールを受信 → スパム報告

対策:
- 自動返信メールの本文にユーザー入力を含めない
- 自動返信の送信レートを制限する
- CAPTCHAで自動送信を防止する
```

### 自動返信メールの安全な実装

```js
// 悪い例：ユーザー入力をそのまま自動返信に含める
const autoReply = {
  to: req.body.email,
  subject: 'お問い合わせを受け付けました',
  html: `
    <p>${req.body.name} 様</p>
    <p>以下のお問い合わせを受け付けました：</p>
    <p>${req.body.message}</p>  <!-- XSSの可能性 + 攻撃者のコンテンツを拡散 -->
  `,
};

// 良い例：ユーザー入力を最小限にし、エスケープする
import escape from 'escape-html';

const autoReply = {
  to: req.body.email,
  subject: 'お問い合わせを受け付けました',
  html: `
    <p>${escape(req.body.name)} 様</p>
    <p>お問い合わせを受け付けました。内容を確認の上、担当者よりご連絡いたします。</p>
    <p>受付番号: ${ticketId}</p>
  `,
};
```

## Nodemailerを使った実装例

```js
import nodemailer from 'nodemailer';
import rateLimit from 'express-rate-limit';
import escape from 'escape-html';
import { isEmail } from 'validator';

// トランスポーターの設定
const transporter = nodemailer.createTransport({
  host: 'smtp.example.com',
  port: 587,
  secure: false,  // STARTTLSを使用
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS,
  },
});

// レート制限
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
});

// ヘッダのサニタイズ
function sanitizeHeader(value) {
  return String(value).replace(/[\r\n]/g, '').trim();
}

// バリデーション
function validate(body) {
  const errors = [];
  if (!body.name || body.name.trim().length === 0 || body.name.length > 100) {
    errors.push('名前が無効です');
  }
  if (!isEmail(body.email || '')) {
    errors.push('メールアドレスが無効です');
  }
  if (!body.message || body.message.trim().length === 0 || body.message.length > 5000) {
    errors.push('メッセージが無効です');
  }
  return errors;
}

app.post('/contact', limiter, async (req, res) => {
  // ハニーポットチェック
  if (req.body.website) {
    return res.json({ success: true });
  }

  // バリデーション
  const errors = validate(req.body);
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }

  const name = sanitizeHeader(req.body.name);
  const email = sanitizeHeader(req.body.email);
  const message = req.body.message;

  try {
    // 管理者へのメール
    await transporter.sendMail({
      from: '"お問い合わせフォーム" <noreply@example.com>',
      to: 'contact@example.com',
      replyTo: email,
      subject: `お問い合わせ: ${name}`,
      text: `名前: ${name}\nメール: ${email}\n\n${message}`,
    });

    // 送信者への自動返信（入力内容は含めない）
    await transporter.sendMail({
      from: '"Example Inc." <noreply@example.com>',
      to: email,
      subject: 'お問い合わせを受け付けました',
      text: `${name} 様\n\nお問い合わせありがとうございます。\n内容を確認の上、担当者よりご連絡いたします。`,
    });

    res.json({ success: true });
  } catch (error) {
    console.error('メール送信エラー:', error);
    res.status(500).json({ error: 'メール送信に失敗しました' });
  }
});
```

## セキュリティチェックリスト

| チェック項目 | 内容 |
|---|---|
| メールヘッダインジェクション対策 | 改行コードの除去 |
| 宛先の固定 | ユーザー入力で宛先を変更できないこと |
| レート制限 | IP・セッション単位で送信回数を制限 |
| CSRF対策 | トークンによるリクエスト検証 |
| CAPTCHA / ハニーポット | bot送信の防止 |
| 入力バリデーション | サーバーサイドでの型・長さ・形式チェック |
| サニタイズ | HTMLエスケープ、ヘッダインジェクション防止 |
| 自動返信の安全性 | ユーザー入力をそのまま返信に含めない |
| 環境変数の管理 | SMTP認証情報をコードに直書きしない |
| エラーハンドリング | SMTPエラーの詳細をユーザーに公開しない |

## まとめ

- Webフォームからのメール送信はサーバーサイドで行い、宛先はサーバー側で固定する
- メールヘッダインジェクションを防ぐため、ユーザー入力から改行コードを除去する
- レート制限・CAPTCHA・ハニーポットでスパム送信の踏み台を防止する
- CSRF対策としてトークンを使用する
- 自動返信メールにはユーザー入力をそのまま含めない（バックスキャッター対策）
- 入力バリデーションはフロントエンドとサーバーサイドの両方で行う
- SMTP認証情報は環境変数で管理し、コードにハードコードしない
