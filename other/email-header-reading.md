# メールのヘッダ情報の読み方 — トラブルシューティングの基本

## メールヘッダとは

- メール本文の前に付与されるメタ情報
- 送信元、経由サーバー、認証結果、タイムスタンプなどが記録される
- 通常はメールクライアントで非表示だが、トラブル時の調査に不可欠

## メールヘッダの表示方法

| メールクライアント | 表示手順 |
|---|---|
| Gmail | メールを開く → 右上の「︙」→「メッセージのソースを表示」 |
| Outlook（Web） | メールを開く → 右上の「…」→「表示」→「メッセージのソースを表示」 |
| Thunderbird | メールを開く → 「表示」→「メッセージのソース」（Ctrl+U） |
| Apple Mail | メールを開く → 「表示」→「メッセージ」→「すべてのヘッダ」 |

## ヘッダの基本構造

```
Return-Path: <bounce@mail.example.com>
Received: from mail2.example.com (mail2.example.com [203.0.113.20])
    by mx.recipient.com with ESMTPS id abc123
    for <user@recipient.com>;
    Thu, 03 Apr 2026 10:30:00 +0900
Received: from mail1.example.com (mail1.example.com [203.0.113.10])
    by mail2.example.com with ESMTP id def456;
    Thu, 03 Apr 2026 10:29:58 +0900
Authentication-Results: mx.recipient.com;
    dkim=pass header.d=example.com header.s=selector1;
    spf=pass (sender IP is 203.0.113.20) smtp.mailfrom=mail.example.com;
    dmarc=pass (p=REJECT dis=NONE) header.from=example.com
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
    d=example.com; s=selector1;
    h=from:to:subject:date:message-id;
    bh=abc123...;
    b=def456...
From: info@example.com
To: user@recipient.com
Subject: テストメール
Date: Thu, 03 Apr 2026 10:29:55 +0900
Message-ID: <unique-id@example.com>
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
```

- ヘッダは下から上に向かって追加される（最下部が最初に付与されたヘッダ）
- `Received`ヘッダを下から順に読むことで、メールの配送経路を追跡できる

## 主要なヘッダフィールド

### 送信者・受信者に関するフィールド

| フィールド | 説明 |
|---|---|
| `From` | 送信者の表示名とアドレス（受信者に見える） |
| `To` | 宛先アドレス |
| `Cc` | カーボンコピーの宛先 |
| `Reply-To` | 返信先アドレス（Fromと異なる場合に設定） |
| `Return-Path` | バウンスメールの返送先（エンベロープFrom） |
| `Sender` | 実際の送信者（Fromと異なる場合） |

### 配送経路に関するフィールド

| フィールド | 説明 |
|---|---|
| `Received` | メールが経由したサーバーの記録（複数存在） |
| `X-Originating-IP` | 送信元のIPアドレス（一部のサーバーが付与） |

### 認証に関するフィールド

| フィールド | 説明 |
|---|---|
| `Authentication-Results` | SPF・DKIM・DMARCの認証結果 |
| `DKIM-Signature` | DKIM署名 |
| `ARC-Authentication-Results` | 転送時の認証チェーン（ARC） |

### メッセージ識別に関するフィールド

| フィールド | 説明 |
|---|---|
| `Message-ID` | メールの一意な識別子 |
| `In-Reply-To` | 返信元メールのMessage-ID |
| `References` | スレッド内の関連メールのMessage-ID一覧 |
| `Date` | メールの送信日時 |

### コンテンツに関するフィールド

| フィールド | 説明 |
|---|---|
| `MIME-Version` | MIMEバージョン（通常`1.0`） |
| `Content-Type` | 本文の形式（`text/plain`、`text/html`、`multipart/mixed`など） |
| `Content-Transfer-Encoding` | エンコーディング方式（`7bit`、`base64`、`quoted-printable`） |

## Receivedヘッダの読み方

### 構造

```
Received: from 送信元サーバー (IPアドレス情報)
    by 受信サーバー with プロトコル id セッションID
    for <宛先>;
    日時
```

### 実例の読み方

```
③ Received: from mail-relay.example.com (mail-relay.example.com [203.0.113.30])
      by mx.recipient.com with ESMTPS id gh789
      for <user@recipient.com>;
      Thu, 03 Apr 2026 10:30:00 +0900

② Received: from internal-mta.example.com (internal-mta.example.com [10.0.0.5])
      by mail-relay.example.com with ESMTP id ef456;
      Thu, 03 Apr 2026 10:29:59 +0900

① Received: from app-server.example.com (app-server.example.com [10.0.0.2])
      by internal-mta.example.com with ESMTP id cd345;
      Thu, 03 Apr 2026 10:29:58 +0900
```

```
配送経路（下から上に読む）:

① app-server.example.com（アプリケーションサーバー）
   ↓
② internal-mta.example.com（内部メールサーバー）
   ↓
③ mail-relay.example.com（外部送信用リレーサーバー）
   ↓
   mx.recipient.com（受信者のメールサーバー）
```

### 注意点

- `Received`ヘッダは各サーバーが自分で追加するため、偽装される可能性がある
- 信頼できるのは受信側サーバーが追加した最上部のヘッダ
- `from`の後のカッコ内にある逆引きDNS名やIPアドレスが実際の接続元情報

## Authentication-Resultsヘッダの読み方

### 構造

```
Authentication-Results: 検証を行ったサーバー;
    dkim=結果 header.d=署名ドメイン header.s=セレクタ;
    spf=結果 smtp.mailfrom=エンベロープFromドメイン;
    dmarc=結果 header.from=ヘッダFromドメイン
```

### 認証結果の値

| 値 | 意味 |
|---|---|
| `pass` | 認証成功 |
| `fail` | 認証失敗 |
| `softfail` | 軽度の失敗（SPFの`~all`の場合） |
| `neutral` | 判定なし（SPFの`?all`の場合） |
| `none` | レコードが設定されていない |
| `temperror` | 一時的なエラー（DNS参照失敗など） |
| `permerror` | 永続的なエラー（レコードの構文エラーなど） |

### 読み方の例

```
; すべて成功のパターン
Authentication-Results: mx.google.com;
    dkim=pass header.d=example.com header.s=selector1 header.b=abc123;
    spf=pass (google.com: domain of info@example.com designates 203.0.113.10 as permitted sender) smtp.mailfrom=info@example.com;
    dmarc=pass (p=REJECT sp=REJECT dis=NONE) header.from=example.com
→ SPF: Pass、DKIM: Pass、DMARC: Pass — 正常

; DKIMが失敗しているパターン
Authentication-Results: mx.google.com;
    dkim=fail (body hash did not verify) header.d=example.com;
    spf=pass smtp.mailfrom=example.com;
    dmarc=pass header.from=example.com
→ DKIMは失敗しているが、SPFがPassかつアライメント一致でDMARC Pass

; すべて失敗のパターン（なりすましの可能性が高い）
Authentication-Results: mx.google.com;
    dkim=fail header.d=example.com;
    spf=fail smtp.mailfrom=attacker.com;
    dmarc=fail header.from=example.com
→ 高確率でなりすましメール
```

## トラブルシューティングのパターン

### パターン1: メールが届かない

```
確認手順:
1. バウンスメールが返ってきていないか確認
2. 受信者の迷惑メールフォルダを確認
3. 送信サーバーのログでSMTPエラーコードを確認
4. DNSレコード（MX, SPF, DKIM, DMARC）の設定を確認
```

```bash
# MXレコードの確認
dig recipient.com MX +short

# SPFレコードの確認
dig example.com TXT +short | grep spf

# DKIMレコードの確認
dig selector1._domainkey.example.com TXT +short
```

### パターン2: 迷惑メールに分類される

```
確認ポイント:
1. Authentication-Resultsヘッダの認証結果を確認
   → SPF, DKIM, DMARCがすべてpassか？
2. 送信元IPの逆引きDNS（PTRレコード）が正しいか
3. 送信元IPがブラックリストに載っていないか
4. メール本文にスパム判定されやすい要素がないか
   → 過剰な画像、短縮URL、特定のキーワードなど
```

```bash
# 逆引きDNSの確認
dig -x 203.0.113.10 +short

# ブラックリストの確認（例: Spamhaus）
dig 10.113.0.203.zen.spamhaus.org +short
# 応答があればブラックリストに登録されている
```

### パターン3: DKIMが失敗する

```
よくある原因:
1. DNS上のDKIM公開鍵が間違っている、または未設定
2. メーリングリストや転送で本文が変更された
3. DKIM署名のセレクタが変更されたのにDNS更新が漏れた
4. 署名対象ヘッダが転送中に変更された

確認方法:
- DKIM-Signatureヘッダの d= と s= を確認
- DNS上の公開鍵が正しく設定されているか確認
```

```
DKIM-Signature: v=1; a=rsa-sha256; d=example.com; s=selector1; ...

確認コマンド:
dig selector1._domainkey.example.com TXT +short

期待する結果:
"v=DKIM1; k=rsa; p=MIIBIjANBgkqh..."
```

### パターン4: DMARCアライメントが失敗する

```
よくある原因:
1. エンベロープFromとヘッダFromのドメインが一致しない
   → 外部メール配信サービスのデフォルト設定
2. DKIM署名のドメイン(d=)とヘッダFromのドメインが一致しない
   → 外部サービスのデフォルトDKIM署名

確認方法:
- Return-Path（エンベロープFrom）のドメインを確認
- DKIM-Signatureの d= のドメインを確認
- From ヘッダのドメインと比較
```

```
; アライメント失敗の例
Return-Path: <bounce@sendgrid.net>         ← SPFドメイン: sendgrid.net
DKIM-Signature: ... d=sendgrid.net ...     ← DKIMドメイン: sendgrid.net
From: info@example.com                     ← ヘッダFrom: example.com
→ SPFもDKIMもsendgrid.netで、example.comとアライメント不一致 → DMARC Fail

; アライメント成功の例
Return-Path: <bounce@em123.example.com>    ← SPFドメイン: example.com（サブドメイン）
DKIM-Signature: ... d=example.com ...      ← DKIMドメイン: example.com
From: info@example.com                     ← ヘッダFrom: example.com
→ DKIMがexample.comでアライメント一致 → DMARC Pass
```

### パターン5: メールの遅延

```
確認手順:
1. Receivedヘッダの各タイムスタンプを比較
2. どのサーバー間で遅延が発生しているか特定

Received: from relay.example.com ...
    by mx.recipient.com ...;
    Thu, 03 Apr 2026 10:35:00 +0900    ← ③ +5分の遅延

Received: from mta.example.com ...
    by relay.example.com ...;
    Thu, 03 Apr 2026 10:30:00 +0900    ← ② ここまでは正常

Received: from app.example.com ...
    by mta.example.com ...;
    Thu, 03 Apr 2026 10:29:58 +0900    ← ① 送信元

→ relay.example.com から mx.recipient.com の間で5分の遅延が発生
```

## ヘッダ解析ツール

| ツール | 用途 |
|---|---|
| [Google Admin Toolbox — Messageheader](https://toolbox.googleapps.com/apps/messageheader/) | Receivedヘッダの可視化・遅延分析 |
| [MXToolbox Header Analyzer](https://mxtoolbox.com/EmailHeaders.aspx) | ヘッダの各フィールドを解析 |
| [Mail Tester](https://www.mail-tester.com/) | メールの到達率・スパムスコアのテスト |

- ヘッダ全文を貼り付けるだけで、配送経路や認証結果を視覚的に確認できる
- トラブルシューティングの初手として活用すると効率的

## Xヘッダ（独自ヘッダ）

- `X-`で始まるヘッダは各サーバーやサービスが独自に追加するフィールド
- 標準ではないが、デバッグ情報として有用

```
X-Mailer: Nodemailer/6.9.1              ← 送信に使用したソフトウェア
X-Google-DKIM-Signature: ...            ← Googleの追加DKIM署名
X-Spam-Status: No, score=-2.0           ← スパムスコア
X-Spam-Flag: NO                         ← スパム判定結果
X-Original-To: user@old-domain.com      ← 転送前の元の宛先
X-MS-Exchange-Organization-SCL: 1       ← Exchange Onlineのスパム信頼度
```

## まとめ

- メールヘッダは下から上に読む。`Received`ヘッダが配送経路を示す
- `Authentication-Results`でSPF・DKIM・DMARCの認証結果を確認できる
- `Return-Path`はエンベロープFrom、`From`はヘッダFrom。この違いがDMARCアライメントに影響する
- メールの遅延は`Received`ヘッダのタイムスタンプ比較で特定する
- トラブルシューティングの第一歩はヘッダの確認。解析ツールを活用すると効率的
- `dig`コマンドでDNSレコード（SPF・DKIM・DMARC・MX）を直接確認できる
