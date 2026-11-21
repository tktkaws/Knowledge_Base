# SPF / DKIM / DMARCの仕組み — なりすましメールを防ぐ3つの認証技術

## なぜメール認証が必要か

- メール（SMTP）は仕様上、送信元アドレスを自由に詐称できる
- フィッシングメールやスパムの多くが送信元を偽装している
- SPF・DKIM・DMARCの3つを組み合わせることで、なりすましを検知・防止する
- 2024年2月以降、GmailやYahoo!は大量送信者にこれら3つの設定を義務化

- 参照: [Google — メール送信者のガイドライン](https://support.google.com/a/answer/81126)

## 3つの認証技術の関係

```
送信サーバー → 受信サーバー
                  ├─ SPF認証:   送信元IPアドレスは許可されているか？
                  ├─ DKIM認証:  メール内容は改ざんされていないか？
                  └─ DMARC認証: SPF/DKIMの結果に基づき、どう処理するか？
```

| 技術 | 検証対象 | 仕組み |
|---|---|---|
| SPF | 送信元IPアドレス | DNSのTXTレコードで許可IPを宣言 |
| DKIM | メール本文・ヘッダの改ざん | 電子署名で完全性を検証 |
| DMARC | SPF/DKIMの結果 + ドメイン一致 | ポリシーに基づく処理とレポート |

- SPFだけ、DKIMだけでは不十分。3つ揃えて初めて効果を発揮する

## SPF（Sender Policy Framework）

### 概要

- 送信元IPアドレスがそのドメインから送信を許可されているかを検証する仕組み
- ドメインのDNSにTXTレコードとして設定する
- 受信サーバーがメール受信時にDNSを参照して検証

### 仕組み

```
1. 送信者が user@example.com からメールを送信
2. 受信サーバーが example.com のDNS TXTレコードを参照
3. SPFレコードに記載されたIPアドレスと、実際の送信元IPを比較
4. 一致 → SPF Pass / 不一致 → SPF Fail
```

### DNSレコードの設定例

```dns
; 基本的なSPFレコード
example.com.  IN  TXT  "v=spf1 ip4:203.0.113.0/24 include:_spf.google.com -all"
```

| 構文 | 意味 |
|---|---|
| `v=spf1` | SPFバージョン1（必須・先頭に記述） |
| `ip4:203.0.113.0/24` | このIPv4アドレス範囲からの送信を許可 |
| `ip6:2001:db8::/32` | このIPv6アドレス範囲からの送信を許可 |
| `include:_spf.google.com` | Google Workspaceの送信サーバーを許可 |
| `a` | ドメインのAレコードのIPを許可 |
| `mx` | ドメインのMXレコードのIPを許可 |
| `-all` | 上記以外はすべて拒否（Hard Fail） |
| `~all` | 上記以外はソフトに失敗扱い（Soft Fail） |

### よくある設定パターン

```dns
; Google Workspace を使用
example.com.  IN  TXT  "v=spf1 include:_spf.google.com -all"

; 自社サーバー + 外部メール配信サービスを併用
example.com.  IN  TXT  "v=spf1 ip4:203.0.113.10 include:sendgrid.net include:_spf.google.com -all"

; メールを送信しないドメイン（なりすまし防止）
no-mail.example.com.  IN  TXT  "v=spf1 -all"
```

### 注意点

- SPFレコードはドメインに1つだけ設定する（複数あるとエラー）
- `include`の参照が10回を超えるとPermerrorになる（DNS参照回数の上限）
- SPFはエンベロープFrom（MAIL FROM）を検証する。ヘッダFromとは異なる場合がある

- 参照: [RFC 7208 — Sender Policy Framework](https://datatracker.ietf.org/doc/html/rfc7208)

## DKIM（DomainKeys Identified Mail）

### 概要

- メールに電子署名を付与し、送信後に本文やヘッダが改ざんされていないことを証明する仕組み
- 公開鍵暗号方式を使用
- 送信サーバーが秘密鍵で署名し、受信サーバーがDNSに公開された公開鍵で検証

### 仕組み

```
1. 送信サーバーがメールのヘッダ・本文からハッシュを生成
2. 秘密鍵でハッシュを暗号化し、DKIM-Signatureヘッダとして付与
3. 受信サーバーがDNSからDKIM公開鍵（TXTレコード）を取得
4. 公開鍵で署名を復号し、メールのハッシュと比較
5. 一致 → DKIM Pass（改ざんなし） / 不一致 → DKIM Fail
```

### DNSレコードの設定例

```dns
; セレクタ名._domainkey.ドメイン に公開鍵を設定
selector1._domainkey.example.com.  IN  TXT  "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQ..."
```

| フィールド | 意味 |
|---|---|
| `v=DKIM1` | DKIMバージョン1 |
| `k=rsa` | 鍵の暗号化方式（RSA） |
| `p=MIGf...` | 公開鍵（Base64エンコード） |
| `selector1` | セレクタ名（鍵のローテーションに使用） |

### DKIM-Signatureヘッダの例

```
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
  d=example.com; s=selector1;
  h=from:to:subject:date:message-id;
  bh=2jUSOH9NhtVGCQWNr9BrIAPreKQjO6Sn7XIkfJVOzv8=;
  b=AuUoFEfDxTDkHlLXSZEpZj79LICEps6eda7W3deTVFOk...
```

| フィールド | 意味 |
|---|---|
| `a` | 署名アルゴリズム（`rsa-sha256`推奨） |
| `c` | 正規化方式（`relaxed/relaxed`が一般的） |
| `d` | 署名ドメイン |
| `s` | セレクタ名（DNS参照時に使用） |
| `h` | 署名対象のヘッダフィールド |
| `bh` | 本文のハッシュ値 |
| `b` | 署名値 |

### 注意点

- 鍵長は2048ビット以上を推奨（1024ビットは脆弱）
- セレクタを使って鍵をローテーションできる（旧鍵と新鍵を併存可能）
- メーリングリストや転送時にメール本文が変更されるとDKIM検証が失敗する場合がある

- 参照: [RFC 6376 — DomainKeys Identified Mail](https://datatracker.ietf.org/doc/html/rfc6376)

## DMARC（Domain-based Message Authentication, Reporting & Conformance）

### 概要

- SPFとDKIMの認証結果を統合し、認証失敗時の処理ポリシーを宣言する仕組み
- 加えて、認証結果のレポートを受け取ることで、自ドメインのなりすまし状況を監視できる
- ヘッダFrom（受信者に表示される送信元）とのドメイン一致（アライメント）を検証する

### 仕組み

```
1. 受信サーバーがSPFとDKIMの認証を実施
2. ヘッダFromのドメインとSPF/DKIMのドメインが一致するか確認（アライメント）
3. DMARCポリシーに従い、失敗時の処理を決定
4. 認証結果のレポートをドメイン管理者に送信
```

### アライメントとは

- SPFアライメント: エンベロープFromのドメインとヘッダFromのドメインが一致するか
- DKIMアライメント: DKIM署名の`d=`ドメインとヘッダFromのドメインが一致するか
- DMARC Passの条件: SPFまたはDKIMのどちらかが「認証Pass + アライメント一致」を満たすこと

```
ヘッダFrom: user@example.com

SPFアライメント:
  エンベロープFrom: bounce@example.com    → 一致（example.com）✓
  エンベロープFrom: bounce@mail.other.com → 不一致 ✗

DKIMアライメント:
  DKIM d=example.com  → 一致 ✓
  DKIM d=other.com    → 不一致 ✗
```

### DNSレコードの設定例

```dns
; _dmarc.ドメイン にポリシーを設定
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=reject; rua=mailto:dmarc-reports@example.com; ruf=mailto:dmarc-forensic@example.com; adkim=r; aspf=r"
```

| タグ | 意味 | 値の例 |
|---|---|---|
| `v` | バージョン（必須） | `DMARC1` |
| `p` | ポリシー（必須） | `none` / `quarantine` / `reject` |
| `rua` | 集約レポート送信先 | `mailto:dmarc@example.com` |
| `ruf` | フォレンジックレポート送信先 | `mailto:forensic@example.com` |
| `adkim` | DKIMアライメントモード | `r`（relaxed） / `s`（strict） |
| `aspf` | SPFアライメントモード | `r`（relaxed） / `s`（strict） |
| `pct` | ポリシー適用率 | `100`（デフォルト） |
| `sp` | サブドメインのポリシー | `none` / `quarantine` / `reject` |

### ポリシーの段階的導入

```dns
; ステップ1: まず監視のみ（メールの配信には影響しない）
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=none; rua=mailto:dmarc@example.com"

; ステップ2: 失敗メールを迷惑メールフォルダへ（25%に適用）
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=quarantine; pct=25; rua=mailto:dmarc@example.com"

; ステップ3: 失敗メールを迷惑メールフォルダへ（100%に適用）
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"

; ステップ4: 失敗メールを拒否
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=reject; rua=mailto:dmarc@example.com"
```

- いきなり`p=reject`にすると正当なメールがブロックされるリスクがある
- `p=none`で集約レポートを確認し、問題ないことを確認してから段階的に強化する

### 集約レポート（RUA）の読み方

- XML形式で毎日送られてくる
- 主な確認ポイント:
  - 自社ドメインから送信しているIPアドレスの一覧
  - 各IPのSPF/DKIM/DMARC認証結果
  - 認証失敗しているIPアドレス（なりすましの可能性）

```xml
<record>
  <row>
    <source_ip>203.0.113.10</source_ip>
    <count>150</count>
    <policy_evaluated>
      <disposition>none</disposition>
      <dkim>pass</dkim>
      <spf>pass</spf>
    </policy_evaluated>
  </row>
</record>
```

- 参照: [RFC 7489 — DMARC](https://datatracker.ietf.org/doc/html/rfc7489)

## 3つの認証を設定する手順

### 1. SPFの設定

```dns
; 使用中のメールサービスに合わせてincludeを追加
example.com.  IN  TXT  "v=spf1 include:_spf.google.com -all"
```

### 2. DKIMの設定

- メールサービス提供元からDKIM公開鍵を取得
- 指定されたセレクタ名でDNS TXTレコードを追加

```dns
google._domainkey.example.com.  IN  TXT  "v=DKIM1; k=rsa; p=MIIBIjANBgkqh..."
```

### 3. DMARCの設定

```dns
; まずは監視モードから開始
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=none; rua=mailto:dmarc@example.com"
```

### 4. 確認

```bash
# SPFレコードの確認
dig example.com TXT +short

# DKIMレコードの確認
dig selector1._domainkey.example.com TXT +short

# DMARCレコードの確認
dig _dmarc.example.com TXT +short
```

- 設定後にテストメールを送信し、メールヘッダの`Authentication-Results`を確認する

## よくある間違い

### 1. SPFレコードを複数設定してしまう

```dns
; 悪い例：TXTレコードに複数のSPFが存在する
example.com.  IN  TXT  "v=spf1 include:_spf.google.com -all"
example.com.  IN  TXT  "v=spf1 include:sendgrid.net -all"

; 良い例：1つのレコードにまとめる
example.com.  IN  TXT  "v=spf1 include:_spf.google.com include:sendgrid.net -all"
```

### 2. SPFのDNS参照回数が上限を超える

```dns
; 悪い例：includeが多すぎて10回の上限を超える可能性
example.com.  IN  TXT  "v=spf1 include:a.com include:b.com include:c.com include:d.com include:e.com include:f.com -all"

; 良い例：不要なincludeを整理する、またはSPFフラットニングツールを使用
example.com.  IN  TXT  "v=spf1 ip4:203.0.113.0/24 include:_spf.google.com -all"
```

### 3. DMARCをいきなりrejectにする

```dns
; 悪い例：事前確認なしでrejectを設定
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=reject"

; 良い例：noneから始めてレポートで状況を把握
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=none; rua=mailto:dmarc@example.com"
```

### 4. 外部メール配信サービスのDKIM設定を忘れる

- SPFだけ設定してDKIMを設定しないと、DMARCアライメントが失敗する可能性がある
- SendGrid、Amazon SES、Mailchimpなど外部サービス利用時は各サービスのDKIM設定手順に従う

## まとめ

- SPFは送信元IPアドレスの正当性を、DKIMはメール内容の完全性を、DMARCはこれらを統合してポリシーを適用する仕組み
- 3つを組み合わせることで、なりすましメールを効果的に防止できる
- DMARCは`p=none`（監視）から段階的に`p=reject`（拒否）へ強化する
- SPFレコードはドメインに1つだけ、DNS参照は10回以内に収める
- DKIM鍵は2048ビット以上を使用し、定期的にローテーションする
- Gmailガイドライン対応のため、メール送信を行うすべてのドメインで設定が必須
