# Let's Encryptで無料SSL証明書を取得・自動更新する方法

## Let's Encryptとは

- 無料でDV（ドメイン認証）SSL証明書を発行する認証局（CA）
- ISRG（Internet Security Research Group）が運営する非営利プロジェクト
- 2015年にサービス開始。2024年時点で全Webサイトの証明書の約50%以上を発行
- ACME（Automatic Certificate Management Environment）プロトコルによる完全自動化が特徴

| 項目 | 内容 |
|---|---|
| 費用 | 無料 |
| 証明書の種類 | DVのみ（OV・EVは非対応） |
| 有効期間 | 90日（自動更新を前提とした短期間） |
| ワイルドカード | 対応（DNS-01チャレンジが必要） |
| レート制限 | 同一ドメインに対して週50枚まで |

- 参照: [Let's Encrypt — About](https://letsencrypt.org/about/)

## Certbotとは

- Let's Encryptの証明書を取得・更新するための公式推奨クライアントツール
- EFF（Electronic Frontier Foundation）が開発・メンテナンス
- Apache、Nginxとの連携機能を内蔵
- 証明書の取得から設定、自動更新までをワンコマンドで実行可能

- 参照: [Certbot — 公式サイト](https://certbot.eff.org/)

## ドメイン認証（チャレンジ）の方式

- Let's Encryptがドメインの所有権を確認する方法は3種類

| 方式 | 仕組み | ワイルドカード | 用途 |
|---|---|---|---|
| HTTP-01 | Webサーバーに検証用ファイルを設置 | 非対応 | 最も一般的。Webサーバーが稼働中の場合 |
| DNS-01 | DNSにTXTレコードを追加 | 対応 | ワイルドカード証明書の取得、Webサーバーが外部公開されていない場合 |
| TLS-ALPN-01 | TLSハンドシェイク中に検証 | 非対応 | ポート443のみ開放している特殊な環境 |

### HTTP-01チャレンジの流れ

```
1. CertbotがLet's Encryptにリクエスト
2. Let's Encryptがトークン（ランダムな文字列）を発行
3. Certbotがトークンを http://example.com/.well-known/acme-challenge/TOKEN に設置
4. Let's EncryptのサーバーがそのURLにアクセスしてトークンを検証
5. 検証成功 → 証明書を発行
```

### DNS-01チャレンジの流れ

```
1. CertbotがLet's Encryptにリクエスト
2. Let's Encryptがトークンを発行
3. Certbotが _acme-challenge.example.com にTXTレコードとしてトークンを設定
4. Let's EncryptがDNSを参照してトークンを検証
5. 検証成功 → 証明書を発行（ワイルドカード対応）
```

## Certbotのインストール

### Ubuntu / Debian

```bash
# snapdでインストール（推奨）
sudo snap install --classic certbot

# certbotコマンドへのシンボリックリンクを作成
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### CentOS / RHEL

```bash
# EPELリポジトリを有効化
sudo dnf install epel-release

# certbotをインストール
sudo dnf install certbot
```

### macOS（開発用途）

```bash
brew install certbot
```

- 参照: [Certbot — Installation](https://certbot.eff.org/instructions)

## Nginxでの証明書取得

### 自動設定（推奨）

```bash
# Nginxプラグイン付きでインストール
sudo snap install --classic certbot
sudo apt install python3-certbot-nginx  # または snap で自動インストール済み

# 証明書の取得とNginx設定の自動変更
sudo certbot --nginx -d example.com -d www.example.com
```

- Certbotが自動的にNginxの設定ファイルを書き換え、SSL設定を追加する
- HTTP→HTTPSリダイレクトの設定も自動で行うか選択できる

### Certbot実行後のNginx設定（自動生成される内容）

```nginx
server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # ... 既存の設定
}

server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;  # HTTPSへリダイレクト
}
```

### 証明書のみ取得（手動設定）

```bash
# 証明書だけ取得し、Nginx設定は自分で行う
sudo certbot certonly --nginx -d example.com -d www.example.com
```

## Apacheでの証明書取得

### 自動設定（推奨）

```bash
# Apacheプラグイン付きでインストール
sudo apt install python3-certbot-apache

# 証明書の取得とApache設定の自動変更
sudo certbot --apache -d example.com -d www.example.com
```

### Certbot実行後のApache設定（自動生成される内容）

```apache
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf

    # ... 既存の設定
</VirtualHost>

<VirtualHost *:80>
    ServerName example.com
    RewriteEngine On
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>
```

## Webサーバーを使わない取得（standaloneモード）

```bash
# Certbot自身が一時的にWebサーバーを起動して認証
# 既存のWebサーバーを事前に停止する必要がある
sudo certbot certonly --standalone -d example.com -d www.example.com
```

- ポート80が空いている必要がある
- Webサーバーが稼働中の場合はこのモードは使えない

## ワイルドカード証明書の取得

```bash
# DNS-01チャレンジでワイルドカード証明書を取得
sudo certbot certonly --manual --preferred-challenges dns \
  -d "*.example.com" -d example.com
```

### 実行時の操作

```
1. Certbotが以下のようなメッセージを表示:
   Please deploy a DNS TXT record under the name:
   _acme-challenge.example.com
   with the following value:
   gfj9Xq...Rg5nTY3QKrSbHF

2. DNSに指定されたTXTレコードを追加して反映を待つ

3. Enterキーを押して検証を続行
```

### DNSプラグインによる自動化

```bash
# Cloudflareの場合
sudo snap install certbot-dns-cloudflare

# APIトークンの設定ファイルを作成
sudo mkdir -p /etc/letsencrypt
cat <<EOF | sudo tee /etc/letsencrypt/cloudflare.ini
dns_cloudflare_api_token = YOUR_API_TOKEN
EOF
sudo chmod 600 /etc/letsencrypt/cloudflare.ini

# 自動でDNS認証を行いワイルドカード証明書を取得
sudo certbot certonly --dns-cloudflare \
  --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini \
  -d "*.example.com" -d example.com
```

- 主要なDNSプロバイダ用のプラグインが公式で提供されている
- Route 53、Google Cloud DNS、Cloudflareなどに対応

## 証明書ファイルの構成

```
/etc/letsencrypt/live/example.com/
├── fullchain.pem   ← サーバー証明書 + 中間証明書（Webサーバーに設定するのはこれ）
├── chain.pem       ← 中間証明書のみ
├── cert.pem        ← サーバー証明書のみ
└── privkey.pem     ← 秘密鍵（厳重に管理）
```

| ファイル | 用途 | Nginx設定 |
|---|---|---|
| `fullchain.pem` | 証明書（中間証明書含む） | `ssl_certificate` |
| `privkey.pem` | 秘密鍵 | `ssl_certificate_key` |

- `privkey.pem`はrootのみ読み取り可能にする（`chmod 600`）
- 実体は`/etc/letsencrypt/archive/`にあり、`live/`はシンボリックリンク

## 自動更新の設定

### Certbotの自動更新（デフォルト）

```bash
# snapでインストールした場合、自動更新のタイマーが設定済み
# systemdタイマーの確認
sudo systemctl list-timers | grep certbot

# 手動で更新テスト（実際には更新しない）
sudo certbot renew --dry-run
```

### systemdタイマーの確認

```bash
# タイマーの状態を確認
sudo systemctl status snap.certbot.renew.timer

# タイマーの内容を確認（1日2回実行）
sudo systemctl cat snap.certbot.renew.timer
```

### cronで手動設定する場合

```bash
# /etc/cron.d/certbot に追加
0 0,12 * * * root certbot renew --quiet --deploy-hook "systemctl reload nginx"
```

- `--quiet`: 更新がない場合は出力を抑制
- `--deploy-hook`: 証明書が実際に更新された場合のみ実行されるコマンド
- 1日2回実行が推奨（失敗した場合のリトライのため）

### 更新時のWebサーバー再読み込み

```bash
# Nginx の場合
sudo certbot renew --deploy-hook "systemctl reload nginx"

# Apache の場合
sudo certbot renew --deploy-hook "systemctl reload apache2"
```

- `--deploy-hook`は証明書が更新された場合のみ実行される
- `--post-hook`はrenewコマンド実行のたびに実行される（更新がなくても）

## 証明書の管理コマンド

```bash
# 取得済み証明書の一覧を表示
sudo certbot certificates

# 出力例:
# Certificate Name: example.com
#   Domains: example.com www.example.com
#   Expiry Date: 2026-07-03 (VALID: 89 days)
#   Certificate Path: /etc/letsencrypt/live/example.com/fullchain.pem
#   Private Key Path: /etc/letsencrypt/live/example.com/privkey.pem

# 特定の証明書を削除
sudo certbot delete --cert-name example.com

# 証明書に新しいドメインを追加（再発行）
sudo certbot certonly --nginx -d example.com -d www.example.com -d api.example.com --expand
```

## レート制限

| 制限 | 上限 |
|---|---|
| 同一ドメインへの証明書発行 | 週50枚 |
| 完全に同じドメインセットの証明書 | 週5枚 |
| 登録アカウント数 | 同一IP・3時間で10アカウント |
| 検証失敗 | 同一アカウント・ドメイン・1時間で5回 |

- テスト時は`--staging`フラグを使ってステージング環境で試すことでレート制限を回避
- ステージング環境の証明書はブラウザで信頼されない（テスト専用）

```bash
# ステージング環境で証明書取得をテスト
sudo certbot certonly --staging --nginx -d example.com
```

- 参照: [Let's Encrypt — Rate Limits](https://letsencrypt.org/docs/rate-limits/)

## よくあるトラブルと対処法

### ポート80がブロックされている

```
Error: Could not connect to port 80

原因: ファイアウォールやセキュリティグループでポート80が閉じている
対処: ポート80を開放するか、DNS-01チャレンジを使用する
```

### Nginx/Apacheの設定エラー

```bash
# 設定ファイルの文法チェック
sudo nginx -t
sudo apachectl configtest

# よくある原因: ssl_certificateのパスが間違っている
# 正しいパス: /etc/letsencrypt/live/ドメイン名/fullchain.pem
```

### 自動更新が失敗する

```bash
# 更新テストで原因を確認
sudo certbot renew --dry-run

# よくある原因:
# - Webサーバーが停止している
# - ポート80がブロックされている
# - ドメインのDNSが変更されている
# - ディスク容量不足
```

### 証明書の期限切れ

```bash
# 証明書の有効期限を確認
sudo certbot certificates

# 手動で即座に更新
sudo certbot renew --force-renewal

# 更新後にWebサーバーを再読み込み
sudo systemctl reload nginx
```

## AWS・クラウド環境での代替手段

### AWS Certificate Manager（ACM）

```
- AWS環境ではACMで無料SSL証明書を取得可能
- ALB（Application Load Balancer）やCloudFrontに直接アタッチ
- 自動更新が完全に管理される
- EC2に直接設置する場合はCertbotが必要
```

### Cloudflare

```
- Cloudflareをプロキシとして使用すると、自動的にSSL証明書が適用される
- オリジンサーバーとの間もCloudflare Origin CA証明書で暗号化可能
- CDNとSSLを同時に導入できる
```

## まとめ

- Let's Encryptは無料のDV証明書を自動で発行する認証局。Certbotが公式推奨ツール
- HTTP-01チャレンジが最も簡単。ワイルドカード証明書にはDNS-01が必要
- Certbotの`--nginx`/`--apache`オプションでWebサーバー設定まで自動化できる
- 証明書は90日有効。自動更新をsystemdタイマーまたはcronで必ず設定する
- `--deploy-hook`で更新時のWebサーバー再読み込みを自動化する
- テスト時は`--staging`でレート制限を回避する
- AWS環境ではACM、Cloudflare利用時はCloudflareのSSLも選択肢になる
