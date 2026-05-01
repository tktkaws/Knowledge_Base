# Apache vs Nginx — 特徴の違いと選定の基準

## 概要

- Apache HTTP ServerとNginxは、世界で最も広く使われている2つのWebサーバーソフトウェア
- Apacheは1995年登場、長い歴史と豊富なモジュールが強み
- Nginxは2004年登場、高い同時接続処理能力とリバースプロキシ機能が強み
- どちらが「優れている」ではなく、用途や要件に応じて選定することが重要

> 参照: [Apache HTTP Server Project](https://httpd.apache.org/) / [Nginx公式ドキュメント](https://nginx.org/en/docs/)

---

## アーキテクチャの違い

### Apache — プロセス/スレッドベース

- リクエストごとにプロセスまたはスレッドを割り当てる方式
- MPM（Multi-Processing Module）でモデルを切り替え可能
  - **prefork**: プロセスベース。安定性が高いがメモリ消費大
  - **worker**: スレッドベース。preforkより軽量
  - **event**: worker改良版。Keep-Alive接続を効率的に処理

```
# MPMの確認コマンド
apachectl -V | grep MPM

# 出力例
Server MPM:     event
```

### Nginx — イベント駆動（非同期）

- 少数のワーカープロセスがイベントループで大量の接続を処理
- 1つのワーカーが数千の同時接続を処理可能
- C10K問題（1万同時接続）を解決するために設計された

```
# Nginxのワーカープロセス設定
worker_processes auto;  # CPUコア数に自動設定

events {
    worker_connections 1024;  # 1ワーカーあたりの最大接続数
}
```

> 参照: [Apache MPM Documentation](https://httpd.apache.org/docs/2.4/mpm.html) / [Nginx Architecture](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)

---

## 主要な違いの比較

| 項目 | Apache | Nginx |
|------|--------|-------|
| アーキテクチャ | プロセス/スレッドベース | イベント駆動（非同期） |
| 静的ファイル配信 | 普通 | 非常に高速 |
| 同時接続処理 | 接続数に比例してリソース消費 | 少ないリソースで大量処理 |
| 設定の柔軟性 | `.htaccess`でディレクトリ単位の設定可 | 設定ファイルの集中管理のみ |
| モジュール | 動的ロード可能 | コンパイル時に組み込み（一部動的対応） |
| リバースプロキシ | mod_proxyで対応 | ネイティブで高性能 |
| ドキュメント/情報量 | 非常に豊富 | 豊富（ただしApacheほどではない） |

---

## Apache の特徴と得意なこと

### `.htaccess`によるディレクトリ単位の設定

- ディレクトリごとに設定ファイルを置ける
- サーバー全体の設定を変更せず、個別サイトの挙動を制御可能
- **共有ホスティング（レンタルサーバー）で特に有用**

```apache
# .htaccess の例
RewriteEngine On
RewriteRule ^old-page$ /new-page [R=301,L]

# Basic認証
AuthType Basic
AuthName "Restricted Area"
AuthUserFile /path/to/.htpasswd
Require valid-user
```

### 動的モジュールの読み込み

- 必要なモジュールを動的に有効化/無効化可能
- PHPをmod_phpとして直接組み込める

```apache
# モジュールの有効化（Debian/Ubuntu系）
a2enmod rewrite
a2enmod ssl
a2enmod headers

# 設定ファイルで直接指定
LoadModule rewrite_module modules/mod_rewrite.so
```

### PHPとの親和性

- mod_phpによりPHPをApacheプロセス内で直接実行
- WordPressなどPHPベースのCMSとの相性が良い

> 参照: [Apache Module Index](https://httpd.apache.org/docs/2.4/mod/)

---

## Nginx の特徴と得意なこと

### 静的ファイルの高速配信

- イベント駆動のため、静的ファイル（HTML/CSS/JS/画像）の配信が非常に高速
- メモリ消費が少なく、大量アクセスに強い

```nginx
server {
    listen 80;
    server_name example.com;

    # 静的ファイルの配信
    location /static/ {
        root /var/www;
        expires 30d;              # ブラウザキャッシュ30日
        add_header Cache-Control "public, immutable";
    }
}
```

### リバースプロキシ / ロードバランサー

- バックエンドのアプリケーションサーバー（Node.js、Pythonなど）の前段に配置
- 複数サーバーへのリクエスト分散が容易

```nginx
# リバースプロキシの設定
upstream backend {
    server 127.0.0.1:3000;   # Node.jsアプリ
    server 127.0.0.1:3001;   # 別のインスタンス
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### SSL/TLS終端（SSLターミネーション）

- Nginxでhttpsを処理し、バックエンドにはhttpで転送
- バックエンドアプリの負荷を軽減

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;  # バックエンドはHTTP
    }
}
```

> 参照: [Nginx Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/) / [Nginx Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

---

## 選定の基準

### Apacheが適しているケース

- **レンタルサーバー（共有ホスティング）を使う場合**
  - `.htaccess`が使えるのはApacheのみ
  - ほとんどのレンタルサーバーがApacheを採用
- **WordPress / PHPベースのCMSを運用する場合**
  - mod_phpとの連携が簡単
  - `.htaccess`によるパーマリンク設定が前提になっている
- **ディレクトリ単位で細かく設定を変えたい場合**
  - プロジェクトごとに異なるルールを適用しやすい

### Nginxが適しているケース

- **高トラフィックなサイト / APIサーバーの場合**
  - 同時接続の処理効率が圧倒的に高い
  - メモリ消費が少ない
- **Node.js / Python / Go などのアプリサーバーの前段**
  - リバースプロキシとして最適
  - SSL終端 + 静的ファイル配信をNginxが担当
- **マイクロサービスのゲートウェイとして**
  - ロードバランシング機能が充実
  - ヘルスチェックやフェイルオーバーに対応
- **静的サイト（SSG）のホスティング**
  - 高速な静的ファイル配信がそのまま活かせる

### 両方を組み合わせるケース

- Nginxをフロントのリバースプロキシ、ApacheをバックエンドとしてPHPを処理
- 実際の運用では、この構成も少なくない

```
クライアント → Nginx（SSL終端 + 静的ファイル） → Apache（PHP処理）
```

---

## パフォーマンス比較の目安

### 静的ファイル配信

```
# 簡易ベンチマーク（ab: Apache Bench）
ab -n 10000 -c 100 http://localhost/index.html

# 一般的な傾向
# Nginx: 高スループット、低メモリ消費
# Apache(event MPM): Nginxに近い性能
# Apache(prefork MPM): メモリ消費が大きい
```

### メモリ使用量の傾向

| 同時接続数 | Apache (prefork) | Apache (event) | Nginx |
|-----------|-----------------|----------------|-------|
| 100 | ~100MB | ~30MB | ~5MB |
| 1,000 | ~1GB | ~200MB | ~10MB |
| 10,000 | 実質困難 | ~1.5GB | ~30MB |

※ あくまで傾向を示す概算値。実際の値は環境・設定に依存する

> 参照: [Nginx vs Apache Performance Comparison](https://www.nginx.com/blog/nginx-vs-apache-our-view/)

---

## 設定ファイルの構文比較

### Apacheの設定例

```apache
# httpd.conf または sites-available/*.conf
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Nginxの設定例

```nginx
# /etc/nginx/conf.d/*.conf または sites-available/*
server {
    listen 80;
    server_name example.com;
    root /var/www/html;

    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;
}
```

---

## まとめ

| 判断基準 | 選択肢 |
|---------|--------|
| レンタルサーバーで使う | Apache |
| WordPress / PHP CMS | Apache（またはNginx + PHP-FPM） |
| `.htaccess`が必要 | Apache |
| 高トラフィック・大量同時接続 | Nginx |
| リバースプロキシ・ロードバランサー | Nginx |
| Node.js / Pythonアプリの前段 | Nginx |
| 静的サイトのホスティング | Nginx |
| SSLターミネーション | Nginx |
| 学習コスト重視 | Apache（情報が豊富） |

- 迷ったらNginxを選んでおけば多くのケースに対応できる
- ただしレンタルサーバーやWordPress案件ではApacheの知識が必要になる場面が多い
- 両方の基礎を理解しておくことが実務では重要
