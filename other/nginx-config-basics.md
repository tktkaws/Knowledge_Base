# Nginxの設定ファイルの読み方と基本設定

## 概要

- Nginxの設定ファイルは独自の構文を持つテキストファイル
- ディレクティブ（命令）とコンテキスト（ブロック）で構成される
- `.htaccess`のような分散設定は存在せず、設定ファイルの集中管理が基本
- 設定変更後はリロードまたは再起動が必要

> 参照: [Nginx Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)

---

## 設定ファイルの場所

### 主要な設定ファイル

| パス | 役割 |
|------|------|
| `/etc/nginx/nginx.conf` | メインの設定ファイル |
| `/etc/nginx/conf.d/` | サイトごとの設定ファイル（推奨） |
| `/etc/nginx/sites-available/` | 利用可能なサイト設定（Debian/Ubuntu系） |
| `/etc/nginx/sites-enabled/` | 有効化されたサイト設定（シンボリックリンク） |
| `/etc/nginx/mime.types` | MIMEタイプの定義 |

### Debian/Ubuntu系とCentOS系の違い

```bash
# Debian/Ubuntu系
# sites-available / sites-enabled 方式
ls /etc/nginx/sites-available/
ls /etc/nginx/sites-enabled/

# サイトの有効化（シンボリックリンク作成）
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

# CentOS/RHEL系
# conf.d/ ディレクトリ方式
ls /etc/nginx/conf.d/
```

---

## 設定ファイルの基本構文

### ディレクティブとコンテキスト

```nginx
# 単純ディレクティブ（セミコロンで終了）
worker_processes auto;

# ブロックディレクティブ（コンテキスト）
events {
    worker_connections 1024;
}
```

- **単純ディレクティブ**: `名前 パラメータ;` の形式
- **ブロックディレクティブ**: `名前 { ... }` の形式（中に他のディレクティブを含む）
- コメントは`#`で始まる
- 各ディレクティブの末尾にはセミコロン`;`が必須

### コンテキストの階層構造

```nginx
# グローバルコンテキスト（main）
worker_processes auto;
error_log /var/log/nginx/error.log;

events {
    # eventsコンテキスト
    worker_connections 1024;
}

http {
    # httpコンテキスト
    include mime.types;

    server {
        # serverコンテキスト
        listen 80;

        location / {
            # locationコンテキスト
            root /var/www/html;
        }
    }
}
```

| コンテキスト | 役割 |
|-------------|------|
| `main`（最上位） | ワーカープロセス数、エラーログなどの全体設定 |
| `events` | 接続処理に関する設定 |
| `http` | HTTP通信に関する設定 |
| `server` | バーチャルホスト（サイト）ごとの設定 |
| `location` | URI（パス）ごとの設定 |

> 参照: [Nginx Configuration File Structure](https://nginx.org/en/docs/beginners_guide.html#conf_structure)

---

## nginx.confの全体像

```nginx
# === グローバル設定 ===
user nginx;                          # 実行ユーザー
worker_processes auto;               # ワーカープロセス数（autoでCPUコア数に合わせる）
error_log /var/log/nginx/error.log;  # エラーログの出力先
pid /run/nginx.pid;                  # PIDファイルの場所

# === イベント設定 ===
events {
    worker_connections 1024;         # 1ワーカーあたりの最大同時接続数
    multi_accept on;                 # 複数の接続を一度に受け入れる
}

# === HTTP設定 ===
http {
    include /etc/nginx/mime.types;   # MIMEタイプの読み込み
    default_type application/octet-stream;

    # ログフォーマット
    log_format main '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent"';

    access_log /var/log/nginx/access.log main;

    sendfile on;                     # 効率的なファイル送信
    tcp_nopush on;                   # レスポンスヘッダとファイルを一括送信
    keepalive_timeout 65;            # Keep-Alive接続のタイムアウト
    gzip on;                         # Gzip圧縮の有効化

    # サイトごとの設定ファイルを読み込み
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

---

## serverブロックの設定

### 基本的なサイト設定

```nginx
server {
    listen 80;                       # ポート番号
    server_name example.com www.example.com;  # ドメイン名
    root /var/www/example.com/html;  # ドキュメントルート
    index index.html index.htm;      # デフォルトのインデックスファイル

    location / {
        try_files $uri $uri/ =404;   # ファイルが見つからなければ404
    }
}
```

### server_nameの書き方

```nginx
# 完全一致
server_name example.com;

# 複数ドメイン
server_name example.com www.example.com;

# ワイルドカード
server_name *.example.com;

# 正規表現
server_name ~^(?<subdomain>.+)\.example\.com$;

# すべてのリクエストにマッチ（デフォルトサーバー）
server_name _;
```

### listenディレクティブ

```nginx
listen 80;                    # IPv4のポート80
listen [::]:80;               # IPv6のポート80
listen 443 ssl;               # HTTPS
listen 443 ssl http2;         # HTTPS + HTTP/2
listen 80 default_server;     # デフォルトサーバー
```

> 参照: [Nginx server_name](https://nginx.org/en/docs/http/server_names.html)

---

## locationブロックの設定

### locationの種類と優先順位

```nginx
# 1. 完全一致（最優先）
location = /favicon.ico {
    log_not_found off;
}

# 2. 前方一致（優先的に停止）
location ^~ /images/ {
    root /var/www/static;
}

# 3. 正規表現（大文字小文字を区別）
location ~ \.php$ {
    fastcgi_pass unix:/run/php/php-fpm.sock;
}

# 4. 正規表現（大文字小文字を区別しない）
location ~* \.(jpg|jpeg|png|gif|svg)$ {
    expires 30d;
}

# 5. 前方一致（通常）
location /api/ {
    proxy_pass http://backend;
}

# 6. デフォルト
location / {
    try_files $uri $uri/ =404;
}
```

**優先順位**（上が最優先）:

1. `= /path` — 完全一致
2. `^~ /path` — 前方一致（正規表現より優先）
3. `~ pattern` / `~* pattern` — 正規表現（定義順で最初にマッチしたもの）
4. `/path` — 前方一致（最も長くマッチしたもの）

### try_filesディレクティブ

```nginx
# ファイル → ディレクトリ → 404
location / {
    try_files $uri $uri/ =404;
}

# SPAのフォールバック（React, Vueなど）
location / {
    try_files $uri $uri/ /index.html;
}

# PHPのフォールバック（WordPress）
location / {
    try_files $uri $uri/ /index.php?$args;
}
```

> 参照: [Nginx location Directive](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)

---

## SSL/HTTPS設定

### 基本的なSSL設定

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL設定の最適化
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # セッションキャッシュ
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    root /var/www/example.com/html;
}
```

### HTTP → HTTPSリダイレクト

```nginx
# HTTPをHTTPSにリダイレクト
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://example.com$request_uri;
}
```

```nginx
# 悪い例：rewriteを使ったリダイレクト（非効率）
server {
    listen 80;
    server_name example.com;
    rewrite ^(.*)$ https://example.com$1 permanent;
}

# 良い例：returnを使ったリダイレクト（効率的）
server {
    listen 80;
    server_name example.com;
    return 301 https://example.com$request_uri;
}
```

---

## リバースプロキシの設定

### Node.js / アプリサーバーの前段としての設定

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;

        # 必須のヘッダー設定
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### ロードバランシング

```nginx
upstream backend {
    # ラウンドロビン（デフォルト）
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

# 重み付け
upstream backend_weighted {
    server 127.0.0.1:3000 weight=3;   # 3倍のリクエストを処理
    server 127.0.0.1:3001 weight=1;
}

# IPハッシュ（同じクライアントを同じサーバーに振り分け）
upstream backend_sticky {
    ip_hash;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
    }
}
```

> 参照: [Nginx Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

---

## Gzip圧縮

```nginx
http {
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;              # 圧縮レベル（1-9、6が推奨）
    gzip_min_length 1000;           # 1000バイト未満は圧縮しない
    gzip_types
        text/plain
        text/css
        application/json
        application/javascript
        text/xml
        application/xml
        image/svg+xml;
}
```

---

## キャッシュ制御

```nginx
server {
    listen 80;
    server_name example.com;

    # HTMLはキャッシュしない
    location ~* \.html$ {
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }

    # 静的アセットは長期キャッシュ
    location ~* \.(css|js|woff2|png|jpg|jpeg|gif|svg|ico)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;            # アクセスログを記録しない（負荷軽減）
    }
}
```

---

## セキュリティ設定

```nginx
server {
    # セキュリティヘッダー
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # 隠しファイルへのアクセスを拒否
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # サーバーバージョン情報を隠す
    server_tokens off;
}
```

---

## アクセス制御

### IPアドレスによる制限

```nginx
location /admin/ {
    allow 192.168.1.0/24;          # 社内ネットワークからのみ許可
    allow 10.0.0.1;                # 特定のIPを許可
    deny all;                      # それ以外は拒否
}
```

### Basic認証

```nginx
location /staging/ {
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

```bash
# .htpasswdの作成（apache2-utilsが必要）
htpasswd -c /etc/nginx/.htpasswd username
```

---

## エラーページのカスタマイズ

```nginx
server {
    error_page 404 /errors/404.html;
    error_page 500 502 503 504 /errors/50x.html;

    location /errors/ {
        internal;                   # 直接アクセス不可（エラー時のみ表示）
        root /var/www;
    }
}
```

---

## 設定の確認と反映

### 構文チェック

```bash
# 設定ファイルの構文チェック（必ず反映前に実行する）
nginx -t

# 出力例
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 設定の反映

```bash
# 方法1: リロード（ダウンタイムなし・推奨）
nginx -s reload

# 方法2: systemctlでリロード
systemctl reload nginx

# 方法3: 再起動（接続が切れる）
systemctl restart nginx
```

```bash
# 悪い例：構文チェックせずにリロード
nginx -s reload

# 良い例：構文チェックしてからリロード
nginx -t && nginx -s reload
```

### 現在の設定を確認

```bash
# コンパイルオプションの確認
nginx -V

# 設定ファイルのパス確認
nginx -t 2>&1 | grep "configuration file"
```

---

## よくある設定パターン

### 静的サイト（SSG）

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/example.com/dist;

    index index.html;

    location / {
        try_files $uri $uri/ $uri.html =404;
    }

    location ~* \.(css|js|png|jpg|svg|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### SPA（React / Vue / Angular）

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/example.com/build;

    index index.html;

    # すべてのルートをindex.htmlにフォールバック
    location / {
        try_files $uri $uri/ /index.html;
    }

    # APIリクエストをバックエンドに転送
    location /api/ {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### WordPress（PHP-FPM）

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/example.com;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # .htaccessへのアクセスを拒否
    location ~ /\.ht {
        deny all;
    }
}
```

> 参照: [Nginx Full Example Configuration](https://www.nginx.com/resources/wiki/start/topics/examples/full/)

---

## まとめ

- Nginxの設定はコンテキスト（main → http → server → location）の階層構造
- locationの優先順位を理解することが正確な設定の鍵
- 設定変更前には必ず`nginx -t`で構文チェック
- `.htaccess`に相当する機能はないため、すべて設定ファイルに集約する
- `return`はリダイレクト、`try_files`はファイル検索、`proxy_pass`はリバースプロキシの基本ディレクティブ
- 静的サイト・SPA・WordPress（PHP-FPM）など、用途に応じたテンプレートを使い分ける
