# メール配信の基礎 — MTA・SMTP・DNSの関係を理解する

## メールが届くまでの全体像

- メールの送受信は複数のサーバーとプロトコルが連携して実現される
- Web開発者がメール配信の問題に対処するには、この仕組みの基本を理解する必要がある

```
送信者（MUA）
  ↓  SMTP（ポート587）
送信側MTA（SMTPサーバー）
  ↓  DNS（MXレコード参照）
  ↓  SMTP（ポート25）
受信側MTA（SMTPサーバー）
  ↓  ローカル配信
受信側MDA（メールボックス）
  ↓  POP3 / IMAP
受信者（MUA）
```

## 主要な登場人物

| 略称 | 正式名称 | 役割 | 具体例 |
|---|---|---|---|
| MUA | Mail User Agent | メールの読み書きをするクライアント | Gmail、Outlook、Thunderbird |
| MTA | Mail Transfer Agent | メールを中継・転送するサーバー | Postfix、Sendmail、Exim |
| MDA | Mail Delivery Agent | メールをユーザーのメールボックスに配信 | Dovecot、Procmail |
| MSA | Mail Submission Agent | MUAからの送信を受け付ける | MTAが兼ねることが多い |

- 参照: [RFC 5321 — Simple Mail Transfer Protocol](https://datatracker.ietf.org/doc/html/rfc5321)

## SMTP（Simple Mail Transfer Protocol）

### 概要

- メールを送信・中継するためのプロトコル
- テキストベースのコマンドとレスポンスでやり取りする
- 1982年に策定された古いプロトコルで、元々は認証や暗号化の仕組みがなかった

### ポート番号

| ポート | 用途 | 暗号化 |
|---|---|---|
| 25 | MTA間のメール中継 | STARTTLS（オプション） |
| 587 | クライアントからの送信（Submission） | STARTTLS（必須） |
| 465 | クライアントからの送信（SMTPS） | 暗黙的TLS |

- ポート25はMTA同士の通信に使用。ISPによってブロックされていることが多い（OP25B）
- クライアントからの送信にはポート587（推奨）または465を使用

### SMTPの通信の流れ

```
クライアント                    サーバー
    |                              |
    | ←── 220 smtp.example.com ──  |  サーバーの挨拶
    | ── EHLO client.example.com → |  クライアントの自己紹介
    | ←── 250-... (機能一覧) ────  |
    | ── AUTH LOGIN ─────────────→ |  認証
    | ←── 235 認証成功 ──────────  |
    | ── MAIL FROM:<a@example.com>→|  送信元アドレス（エンベロープFrom）
    | ←── 250 OK ────────────────  |
    | ── RCPT TO:<b@example.com> → |  宛先アドレス（エンベロープTo）
    | ←── 250 OK ────────────────  |
    | ── DATA ───────────────────→ |  メール本文の開始
    | ←── 354 開始してください ──  |
    | ── (ヘッダ + 本文) ────────→ |
    | ── . ──────────────────────→ |  メール本文の終了
    | ←── 250 OK ────────────────  |
    | ── QUIT ───────────────────→ |  接続終了
    | ←── 221 さようなら ────────  |
```

### エンベロープとヘッダの違い

- エンベロープ: SMTPの`MAIL FROM`と`RCPT TO`で指定されるアドレス。実際の配送に使用
- ヘッダ: メール本文中の`From:`や`To:`フィールド。受信者に表示される
- この2つは異なる値を持つことがある（メーリングリスト、転送、なりすましなど）

```
MAIL FROM:<bounce@mail.example.com>     ← エンベロープFrom（実際の送信経路）
RCPT TO:<user@example.org>              ← エンベロープTo（実際の配送先）

From: info@example.com                  ← ヘッダFrom（受信者に表示される）
To: user@example.org                    ← ヘッダTo（受信者に表示される）
Reply-To: support@example.com           ← 返信先（Fromとは別に指定可能）
```

- 参照: [RFC 5321 — SMTP](https://datatracker.ietf.org/doc/html/rfc5321)

## DNSとメール配信

### MXレコード

- MX（Mail Exchanger）レコードは、そのドメイン宛のメールを受け取るサーバーを指定する
- 優先度の数値が小さいほど優先して使用される

```dns
example.com.  IN  MX  10  mail1.example.com.
example.com.  IN  MX  20  mail2.example.com.
```

- 上記の場合、まず`mail1`に配送を試み、失敗したら`mail2`に配送する
- MXレコードが存在しない場合、Aレコード（またはAAAAレコード）にフォールバックする

### メール配信に関わるDNSレコード

| レコード | 役割 |
|---|---|
| MX | メール受信サーバーの指定 |
| A / AAAA | サーバーのIPアドレス解決 |
| TXT（SPF） | 送信許可IPの宣言 |
| TXT（DKIM） | DKIM公開鍵の公開 |
| TXT（DMARC） | 認証ポリシーの宣言 |
| PTR | IPアドレスからドメイン名への逆引き |

### 逆引きDNS（PTRレコード）

- IPアドレスからドメイン名を引く仕組み
- 多くの受信サーバーが送信元IPの逆引きを確認する
- 逆引きが設定されていない、または一致しない場合、スパム判定されやすい

```
正引き: mail.example.com → 203.0.113.10
逆引き: 203.0.113.10 → mail.example.com   ← 一致していることが重要
```

- 参照: [RFC 5321 Section 2.3.5](https://datatracker.ietf.org/doc/html/rfc5321#section-2.3.5)

## メール配信のエラーとバウンス

### SMTPステータスコード

| コード | 分類 | 意味 |
|---|---|---|
| 2xx | 成功 | 要求が正常に処理された |
| 3xx | 中間応答 | データ入力の継続を要求 |
| 4xx | 一時エラー | 一時的な問題。再送で成功する可能性あり |
| 5xx | 永続エラー | 恒久的な問題。再送しても成功しない |

### よくあるエラーコード

| コード | 意味 | 対処 |
|---|---|---|
| 421 | サービスが一時的に利用不可 | 時間をおいて再送 |
| 450 | メールボックスが一時的に利用不可 | 時間をおいて再送 |
| 550 | メールボックスが存在しない | 宛先アドレスの確認 |
| 551 | ユーザーがローカルでない | 転送先アドレスの確認 |
| 552 | メールボックスの容量超過 | 受信者側の対応が必要 |
| 553 | メールボックス名が不正 | 宛先アドレスの確認 |
| 554 | トランザクション失敗 | メール内容やポリシーの確認 |

### バウンスメール

- ハードバウンス: 永続的なエラー（宛先不明など）。即座にリストから削除すべき
- ソフトバウンス: 一時的なエラー（サーバー負荷、容量超過など）。時間をおいて再送される

```
宛先不明の場合のバウンスメール例:

From: MAILER-DAEMON@mail.example.com
To: sender@example.com
Subject: Undelivered Mail Returned to Sender

The email account that you tried to reach does not exist.
550 5.1.1 <unknown@example.com>: Recipient address rejected
```

## STARTTLS — 通信の暗号化

- SMTP通信を途中からTLSで暗号化する仕組み
- 平文で接続を開始し、`STARTTLS`コマンドで暗号化に切り替える
- MTA間の通信（ポート25）で広く使われている

```
クライアント                    サーバー
    | ←── 220 ... ──────────────  |
    | ── EHLO ──────────────────→ |
    | ←── 250-STARTTLS ─────────  |  STARTTLSに対応していることを通知
    | ── STARTTLS ──────────────→ |  暗号化を要求
    | ←── 220 Ready for TLS ────  |
    |  ~~~ TLSハンドシェイク ~~~   |  ここから暗号化通信
    | ── EHLO ──────────────────→ |  暗号化後に再度EHLO
    | ...                          |
```

- 暗黙的TLS（ポート465）は接続開始時からTLSで暗号化する方式
- 現在はポート587 + STARTTLSが推奨されているが、ポート465も再標準化されている

- 参照: [RFC 3207 — SMTP over TLS](https://datatracker.ietf.org/doc/html/rfc3207)

## メール配信サービスの活用

### 自前のMTAを運用するリスク

- IPレピュテーション（信頼度）の維持が難しい
- スパムリストに載ると他のメールも届かなくなる
- バウンス処理、送信レート制御、暗号化対応などの運用コストが高い
- SPF・DKIM・DMARCの設定と維持が必要

### 主な外部メール配信サービス

| サービス | 特徴 |
|---|---|
| SendGrid | APIベースのメール配信。無料枠あり |
| Amazon SES | AWSの低コストメール配信サービス |
| Mailgun | 開発者向けAPIが充実 |
| Postmark | トランザクションメールに特化 |

### APIを使った送信の例（SendGrid）

```js
// SendGridのWeb APIでメール送信（Node.js）
const sgMail = require('@sendgrid/mail');
sgMail.setApiKey(process.env.SENDGRID_API_KEY);

const msg = {
  to: 'recipient@example.com',
  from: 'sender@example.com',   // 送信元ドメインの認証が必要
  subject: 'テスト件名',
  text: 'プレーンテキスト本文',
  html: '<p>HTML本文</p>',
};

try {
  await sgMail.send(msg);
  console.log('送信成功');
} catch (error) {
  console.error('送信失敗:', error.response?.body);
}
```

- SMTPリレーではなくAPIを使った送信が推奨される（信頼性・管理性が高い）

## よくある間違い

### 1. OP25Bを知らずにポート25で送信しようとする

```
悪い例: アプリケーションからポート25で直接外部にメールを送信
 → ISPにブロックされて届かない

良い例: ポート587でメールサービスのSMTPサーバーに接続する
 → または外部メール配信サービスのAPIを使用する
```

- OP25B（Outbound Port 25 Blocking）: ISPが一般ユーザーのポート25発信を遮断する施策
- スパム防止のため、ほとんどのISPが実施している

### 2. エンベロープFromとヘッダFromの違いを理解していない

```
問題: SPFを設定したのにDMARCが失敗する

原因: SPFはエンベロープFromを検証するが、DMARCはヘッダFromとの
      アライメント（一致）を求める。外部サービス経由で送信すると
      エンベロープFromがサービスのドメインになりアライメントが失敗する

対策: 外部サービスでカスタムエンベロープFromを設定するか、
      DKIMでアライメントを通す
```

### 3. バウンスメールを無視する

- ハードバウンスのアドレスに送り続けると送信元のレピュテーションが低下する
- バウンス率が高いとスパム判定されやすくなる
- 定期的にバウンスアドレスをリストから除外する運用が必要

## まとめ

- メール配信はMUA → MTA → MTA → MDA → MUAの経路で行われる
- SMTPはメール送信・中継のプロトコル。クライアントからはポート587を使用する
- DNSのMXレコードでメール受信サーバーを指定する
- エンベロープ（SMTP）とヘッダ（メール本文）は別物で、異なる値を持つことがある
- STARTTLSまたは暗黙的TLSで通信を暗号化する
- 自前のMTA運用はコストが高い。外部メール配信サービスの利用が現実的
- バウンスメールを適切に処理し、送信元レピュテーションを維持する
