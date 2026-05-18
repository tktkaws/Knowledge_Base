# HTTP→HTTPSリダイレクトの正しい設定方法

## なぜHTTPからHTTPSへリダイレクトするのか

- HTTPS化しただけでは、HTTPでアクセスするユーザーは暗号化されないまま
- ブックマーク、外部リンク、直接入力で`http://`にアクセスされる可能性がある
- HTTPへのアクセスをすべてHTTPSに自動転送（リダイレクト）することで、常に暗号化通信を保証する
- SEO上、HTTPとHTTPSの両方にコンテンツがあると重複コンテンツとみなされるため、正規URLの統一にもなる

## リダイレクトの基本

### 301リダイレクト（恒久的）

```
HTTP/1.1 301 Moved Permanently
Location: https://example.com/page

意味: このURLは恒久的に移動した
効果: ブラウザがリダイレクト先をキャッシュする。検索エンジンがインデックスを移行する
```

- HTTP→HTTPSのリダイレクトには**301（恒久的リダイレクト）**を使う
- 302（一時的リダイレクト）は使わない。検索エンジンが元のHTTP URLのインデックスを維持してしまう

### リダイレクトの流れ

```
ブラウザ → http://example.com/page
           ↓ 301リダイレクト
ブラウザ → https://example.com/page
           ↓ 正常なレスポンス
           200 OK（暗号化通信）
```

## Nginxでの設定

### 基本的なリダイレクト設定

```nginx
# HTTPからHTTPSへの301リダイレクト
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

# HTTPS側の設定
server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # ... サイトの設定
}
```

### よくある間違い

```nginx
# 悪い例: rewriteを使う（不要に複雑）
server {
    listen 80;
    server_name example.com;
    rewrite ^(.*)$ https://example.com$1 permanent;
}

# 良い例: returnを使う（シンプルで高速）
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

- `return 301`は`rewrite`より処理が軽く、意図も明確
- `$host`はリクエストのHostヘッダをそのまま使うため、複数ドメインに対応

### wwwあり/なしの統一も同時に行う場合

```nginx
# wwwなしに統一する場合
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://example.com$request_uri;
}

server {
    listen 443 ssl;
    server_name www.example.com;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    return 301 https://example.com$request_uri;
}

server {
    listen 443 ssl;
    server_name example.com;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    # ... サイトの設定
}
```

## Apacheでの設定

### .htaccessでのリダイレクト

```apache
# .htaccess に記述
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

| ディレクティブ | 意味 |
|---|---|
| `RewriteEngine On` | mod_rewriteを有効化 |
| `RewriteCond %{HTTPS} off` | HTTPSでない場合のみ適用 |
| `RewriteRule ^(.*)$` | すべてのURLパスに一致 |
| `https://%{HTTP_HOST}%{REQUEST_URI}` | 同じホスト・パスのHTTPS URLに転送 |
| `[R=301,L]` | 301リダイレクト、以降のルールは処理しない |

### VirtualHostでのリダイレクト

```apache
# httpd.conf または sites-available/example.conf
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    Redirect permanent / https://example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    # ... サイトの設定
</VirtualHost>
```

### よくある間違い

```apache
# 悪い例: RewriteCondなしでHTTPSでもリダイレクトしてしまう
RewriteEngine On
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
# → HTTPSアクセスもリダイレクトしようとしてループする

# 良い例: RewriteCondでHTTPの場合のみリダイレクト
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

## ロードバランサー・リバースプロキシ環境での設定

### CDN/ロードバランサーがSSLを終端する場合

```
ブラウザ → HTTPS → CDN/LB → HTTP → オリジンサーバー
```

- オリジンサーバーから見ると通信は常にHTTPに見える
- `%{HTTPS}`や`$scheme`が常に`http`になるため、通常のリダイレクト設定ではループが発生する

### X-Forwarded-Protoヘッダを使った判定

```nginx
# Nginx（ロードバランサー背後）
server {
    listen 80;
    server_name example.com;

    # X-Forwarded-Proto がhttpの場合のみリダイレクト
    if ($http_x_forwarded_proto = "http") {
        return 301 https://$host$request_uri;
    }

    # ... サイトの設定
}
```

```apache
# Apache（ロードバランサー背後）
RewriteEngine On
RewriteCond %{HTTP:X-Forwarded-Proto} =http
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

### AWSのALBでのリダイレクト

```
ALB側でHTTP→HTTPSリダイレクトを設定可能（Webサーバーの設定不要）:

1. ALBのリスナー設定を開く
2. HTTPリスナー（ポート80）のルールを編集
3. アクションを「リダイレクト先」に設定:
   - プロトコル: HTTPS
   - ポート: 443
   - ステータスコード: 301
```

- ALBでリダイレクトするとオリジンサーバーの負荷がゼロ
- CloudFrontでも同様の設定が可能（Viewer Protocol Policy: Redirect HTTP to HTTPS）

## HSTS（HTTP Strict Transport Security）

### HSTSとは

- ブラウザに「このサイトには常にHTTPSでアクセスせよ」と指示するHTTPレスポンスヘッダ
- HSTS設定後、ブラウザは自動的にHTTP→HTTPSの変換を行い、HTTPリクエストを送信しない
- 初回の301リダイレクトすら不要になる（初回アクセス以降）

### リダイレクトだけでは不十分な理由

```
301リダイレクトの場合:
  1. ブラウザが http://example.com にHTTPリクエストを送信
  2. サーバーが 301 で https://example.com にリダイレクト
  3. ブラウザが https://example.com にHTTPSリクエストを送信

問題: ステップ1のHTTPリクエストは暗号化されていない
→ 中間者攻撃でリダイレクトを書き換えられる可能性がある（SSLストリッピング攻撃）

HSTSの場合:
  1. ブラウザが内部で http:// → https:// に変換（307 Internal Redirect）
  2. ブラウザが https://example.com にHTTPSリクエストを送信

→ HTTPリクエストが外部に送信されないため、SSLストリッピング攻撃を防止
```

### Nginxでの設定

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    # HSTSヘッダを追加（有効期間1年、サブドメイン含む）
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # ... SSL設定
}
```

### Apacheでの設定

```apache
<VirtualHost *:443>
    ServerName example.com

    # HSTSヘッダを追加
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

    # ... SSL設定
</VirtualHost>
```

### HSTSの各パラメータ

| パラメータ | 意味 |
|---|---|
| `max-age=31536000` | HSTSの有効期間（秒）。31536000 = 1年 |
| `includeSubDomains` | すべてのサブドメインにもHSTSを適用 |
| `preload` | ブラウザのHSTSプリロードリストへの登録を申請 |

### HSTSプリロードリスト

```
通常のHSTS:
  初回アクセス時はHTTPリクエストが発生する（この時点では未設定）

HSTSプリロード:
  ブラウザにドメインが事前登録されるため、初回アクセスからHTTPSを強制
```

- [hstspreload.org](https://hstspreload.org/)で登録申請できる
- Chrome、Firefox、Safari、Edgeなど主要ブラウザに反映される
- 一度登録すると解除が困難なため、HTTPS化が完全に完了してから申請する

### HSTSの段階的導入

```nginx
# ステップ1: 短い有効期間で試す（5分）
add_header Strict-Transport-Security "max-age=300" always;

# ステップ2: 1週間に延長
add_header Strict-Transport-Security "max-age=604800" always;

# ステップ3: 1ヶ月に延長
add_header Strict-Transport-Security "max-age=2592000; includeSubDomains" always;

# ステップ4: 1年に設定（本番運用）
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

# ステップ5: プリロードリストへの登録（任意）
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
```

- いきなり長いmax-ageを設定すると、問題発生時にHTTPに戻せなくなる
- 短い期間から始めて段階的に延長する

- 参照: [MDN — Strict-Transport-Security](https://developer.mozilla.org/ja/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security)

## リダイレクト設定後の確認

### curlでリダイレクトを確認

```bash
# HTTPアクセスで301が返ることを確認
curl -I http://example.com

# 期待される出力:
# HTTP/1.1 301 Moved Permanently
# Location: https://example.com/

# リダイレクトを追跡して最終URLを確認
curl -IL http://example.com

# HSTSヘッダの確認
curl -I https://example.com | grep -i strict
# Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### リダイレクトループの確認

```bash
# リダイレクト回数が多すぎないか確認（最大10回で停止）
curl -ILs -o /dev/null -w "%{url_effective}\n%{redirect_url}\n%{num_redirects}\n" http://example.com

# ブラウザで「ERR_TOO_MANY_REDIRECTS」が表示された場合:
# - ロードバランサー環境でX-Forwarded-Protoの判定が抜けている
# - HTTPSのVirtualHostでもHTTPSへリダイレクトしている
# - .htaccessとVirtualHostの両方でリダイレクトが設定されている
```

### SSLテストツール

```
SSL Labs（Qualys）:
  https://www.ssllabs.com/ssltest/
  → SSL/TLS設定の総合テスト。A+評価を目指す

Security Headers:
  https://securityheaders.com/
  → HSTSを含むセキュリティヘッダの確認
```

## よくある間違い

### 1. 302リダイレクトを使ってしまう

```nginx
# 悪い例: 302（一時的リダイレクト）
return 302 https://$host$request_uri;

# 良い例: 301（恒久的リダイレクト）
return 301 https://$host$request_uri;
```

- 302だと検索エンジンがHTTP版のインデックスを維持し続ける

### 2. リダイレクトチェーン（多段リダイレクト）

```
悪い例: 3回もリダイレクトする
http://www.example.com → https://www.example.com → https://example.com

良い例: 1回でリダイレクト
http://www.example.com → https://example.com
```

```nginx
# 良い例: HTTPとwwwの両方を1回のリダイレクトで処理
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://example.com$request_uri;
}

server {
    listen 443 ssl;
    server_name www.example.com;
    return 301 https://example.com$request_uri;
}
```

### 3. 内部リンクのHTTPを放置

```html
<!-- 悪い例: サイト内リンクがHTTPのまま -->
<a href="http://example.com/about">About</a>
<img src="http://example.com/image.png">

<!-- 良い例: プロトコル相対URLまたは相対パスを使用 -->
<a href="/about">About</a>
<img src="/image.png">
```

- リダイレクトされるとはいえ、無駄な301リクエストが発生する
- サイト内リンクは相対パスに統一する

## まとめ

- HTTP→HTTPSのリダイレクトには301（恒久的リダイレクト）を使う
- Nginxでは`return 301 https://$host$request_uri;`がシンプルで推奨
- ロードバランサー環境では`X-Forwarded-Proto`ヘッダで判定し、リダイレクトループを防ぐ
- HSTSヘッダでブラウザにHTTPS接続を強制し、SSLストリッピング攻撃を防止する
- HSTSは短いmax-ageから段階的に延長する。プリロードリストへの登録は慎重に判断
- リダイレクトチェーン（多段リダイレクト）を避け、1回のリダイレクトで最終URLに到達させる
- サイト内リンクは相対パスに統一し、不要なリダイレクトを削減する
