# .htaccessの基本 — リダイレクト・Basic認証・キャッシュ制御

## 概要

- `.htaccess`はApache Webサーバーのディレクトリ単位の設定ファイル
- サーバー全体の設定（httpd.conf）を変更せずに、特定のディレクトリ以下の挙動を制御できる
- レンタルサーバー（共有ホスティング）でよく使われる
- ファイル名はドット始まりのため、OS上では隠しファイル扱い

> 参照: [Apache .htaccess Tutorial](https://httpd.apache.org/docs/2.4/howto/htaccess.html)

---

## .htaccessの基本ルール

### 配置と有効範囲

- `.htaccess`を配置したディレクトリとその全サブディレクトリに設定が適用される
- 下位ディレクトリに別の`.htaccess`があれば、そちらが優先される

```
/var/www/html/
├── .htaccess          ← サイト全体に適用
├── index.html
├── admin/
│   ├── .htaccess      ← admin/以下に適用（上位の設定を上書き）
│   └── index.html
└── images/
    └── logo.png       ← 上位の.htaccessが適用される
```

### 有効化の条件

- Apacheの設定で`AllowOverride`が許可されている必要がある
- `AllowOverride None`の場合、`.htaccess`は完全に無視される

```apache
# httpd.conf での設定
<Directory /var/www/html>
    AllowOverride All      # すべてのディレクティブを許可
</Directory>

# 限定的に許可する場合
<Directory /var/www/html>
    AllowOverride FileInfo AuthConfig Limit
</Directory>
```

### パフォーマンスへの影響

- Apacheはリクエストのたびに`.htaccess`を読み込む
- ディレクトリ階層が深いと、各階層の`.htaccess`をすべてチェックする
- **可能であれば`httpd.conf`に直接記述する方がパフォーマンスは良い**
- レンタルサーバーでは`httpd.conf`を編集できないため`.htaccess`が唯一の選択肢になる

> 参照: [Apache Performance Tuning — .htaccess](https://httpd.apache.org/docs/2.4/misc/perf-tuning.html)

---

## リダイレクト

### 基本のリダイレクト（Redirect）

```apache
# 単純なリダイレクト
Redirect 301 /old-page.html https://example.com/new-page.html
Redirect 302 /temporary.html https://example.com/other.html

# ディレクトリ全体のリダイレクト
Redirect 301 /old-directory/ https://example.com/new-directory/
```

- `301`: 恒久的なリダイレクト（SEO評価を引き継ぐ）
- `302`: 一時的なリダイレクト（SEO評価を引き継がない）

### mod_rewriteによる高度なリダイレクト

```apache
# mod_rewriteの有効化
RewriteEngine On

# HTTP → HTTPSリダイレクト
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

# wwwあり → wwwなしに統一
RewriteCond %{HTTP_HOST} ^www\.example\.com$ [NC]
RewriteRule ^(.*)$ https://example.com/$1 [R=301,L]

# wwwなし → wwwありに統一
RewriteCond %{HTTP_HOST} ^example\.com$ [NC]
RewriteRule ^(.*)$ https://www.example.com/$1 [R=301,L]
```

### RewriteRuleの構文

```apache
RewriteRule パターン 置換先 [フラグ]
```

- **パターン**: リクエストURIに対する正規表現
- **置換先**: リダイレクト先のURL
- **フラグ**: 動作を制御するオプション

| フラグ | 意味 |
|-------|------|
| `[R=301]` | 301リダイレクト |
| `[R=302]` | 302リダイレクト |
| `[L]` | このルールで処理を終了 |
| `[NC]` | 大文字小文字を区別しない |
| `[QSA]` | クエリ文字列を引き継ぐ |

### RewriteCondの条件指定

```apache
# 条件変数の例
RewriteCond %{HTTPS} off              # HTTPSでない場合
RewriteCond %{HTTP_HOST} ^www\.       # wwwで始まるホスト
RewriteCond %{REQUEST_URI} !^/api/    # /api/で始まらないURI
RewriteCond %{REQUEST_FILENAME} !-f   # ファイルが存在しない場合
RewriteCond %{REQUEST_FILENAME} !-d   # ディレクトリが存在しない場合
```

### よくあるリダイレクトパターン

```apache
RewriteEngine On

# 拡張子.htmlを省略可能にする
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME}.html -f
RewriteRule ^(.*)$ $1.html [L]

# 末尾のスラッシュを削除
RewriteRule ^(.*)/$ /$1 [R=301,L]

# 特定のパスをリダイレクト
RewriteRule ^blog/(.*)$ https://blog.example.com/$1 [R=301,L]

# メンテナンスモード（自分のIPアドレス以外をリダイレクト）
RewriteCond %{REMOTE_ADDR} !^123\.456\.789\.
RewriteRule ^(.*)$ /maintenance.html [R=503,L]
```

> 参照: [Apache mod_rewrite Documentation](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)

---

## Basic認証

### 基本的なBasic認証の設定

```apache
# .htaccess
AuthType Basic
AuthName "Restricted Area"
AuthUserFile /path/to/.htpasswd
Require valid-user
```

### .htpasswdファイルの作成

```bash
# 新規作成（-c は新規ファイル作成）
htpasswd -c /path/to/.htpasswd username

# ユーザーの追加（既存ファイルに追記）
htpasswd /path/to/.htpasswd another-user

# ユーザーの削除
htpasswd -D /path/to/.htpasswd username
```

### .htpasswdの配置場所

```apache
# 悪い例：Webからアクセスできる場所に置く
AuthUserFile /var/www/html/.htpasswd

# 良い例：ドキュメントルートの外に置く
AuthUserFile /var/www/.htpasswd
```

- `.htpasswd`はドキュメントルートの外に置く
- Webブラウザからアクセスできない場所に置くのがセキュリティ上重要

### 特定のディレクトリだけ認証をかける

```
/var/www/html/
├── .htaccess          ← 認証なし
├── index.html
└── staging/
    ├── .htaccess      ← Basic認証を設定
    └── index.html
```

### 特定のIPアドレスを認証なしにする

```apache
# Apache 2.4の書き方
AuthType Basic
AuthName "Restricted Area"
AuthUserFile /path/to/.htpasswd

# 社内IPからは認証不要、それ以外は認証必要
<RequireAny>
    Require ip 192.168.1.0/24
    Require valid-user
</RequireAny>
```

### 特定のファイルだけ認証をかける

```apache
# wp-login.phpだけBasic認証をかける（WordPress向け）
<Files wp-login.php>
    AuthType Basic
    AuthName "Admin Login"
    AuthUserFile /path/to/.htpasswd
    Require valid-user
</Files>
```

> 参照: [Apache Authentication and Authorization](https://httpd.apache.org/docs/2.4/howto/auth.html)

---

## キャッシュ制御

### mod_expiresによるブラウザキャッシュ設定

```apache
<IfModule mod_expires.c>
    ExpiresActive On

    # デフォルトのキャッシュ期間
    ExpiresDefault "access plus 1 month"

    # ファイルタイプ別の設定
    ExpiresByType text/html "access plus 0 seconds"
    ExpiresByType text/css "access plus 1 year"
    ExpiresByType application/javascript "access plus 1 year"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType image/svg+xml "access plus 1 year"
    ExpiresByType font/woff2 "access plus 1 year"
</IfModule>
```

- HTMLはキャッシュしない（常に最新を取得）
- CSS/JS/画像/フォントは長期キャッシュ（ファイル名にハッシュを含めてキャッシュバスティング）

### mod_headersによるCache-Control設定

```apache
<IfModule mod_headers.c>
    # HTMLはキャッシュしない
    <FilesMatch "\.(html|htm)$">
        Header set Cache-Control "no-cache, no-store, must-revalidate"
        Header set Pragma "no-cache"
        Header set Expires 0
    </FilesMatch>

    # 静的アセットは長期キャッシュ
    <FilesMatch "\.(css|js|woff2|png|jpg|svg)$">
        Header set Cache-Control "public, max-age=31536000, immutable"
    </FilesMatch>
</IfModule>
```

### ETagの制御

```apache
# ETagを無効化（CDNとの相性が悪い場合に有効）
<IfModule mod_headers.c>
    Header unset ETag
</IfModule>
FileETag None
```

> 参照: [Apache mod_expires](https://httpd.apache.org/docs/2.4/mod/mod_expires.html) / [Apache mod_headers](https://httpd.apache.org/docs/2.4/mod/mod_headers.html)

---

## Gzip圧縮（mod_deflate）

```apache
<IfModule mod_deflate.c>
    # テキストベースのファイルを圧縮
    AddOutputFilterByType DEFLATE text/html
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/json
    AddOutputFilterByType DEFLATE image/svg+xml
    AddOutputFilterByType DEFLATE text/xml
    AddOutputFilterByType DEFLATE application/xml

    # すでに圧縮されたファイルは除外
    SetEnvIfNoCase Request_URI \.(?:gif|jpe?g|png|webp|woff2)$ no-gzip
</IfModule>
```

> 参照: [Apache mod_deflate](https://httpd.apache.org/docs/2.4/mod/mod_deflate.html)

---

## アクセス制御

### IPアドレスによるアクセス制御

```apache
# Apache 2.4の書き方
# 特定のIPのみ許可
<RequireAll>
    Require ip 192.168.1.0/24
    Require ip 10.0.0.1
</RequireAll>

# 特定のIPを拒否
<RequireAll>
    Require all granted
    Require not ip 203.0.113.0/24
</RequireAll>
```

```apache
# 悪い例：Apache 2.2の古い書き方（2.4では非推奨）
Order deny,allow
Deny from all
Allow from 192.168.1.0/24

# 良い例：Apache 2.4の書き方
Require ip 192.168.1.0/24
```

### 特定のファイルへのアクセスを禁止

```apache
# .htaccessや.htpasswdへの直接アクセスを禁止
<FilesMatch "^\.ht">
    Require all denied
</FilesMatch>

# 設定ファイルへのアクセスを禁止
<FilesMatch "\.(env|ini|log|conf)$">
    Require all denied
</FilesMatch>

# wp-config.phpへのアクセスを禁止
<Files wp-config.php>
    Require all denied
</Files>
```

### ディレクトリ一覧表示の無効化

```apache
# ディレクトリにindex.htmlがない場合のファイル一覧表示を無効化
Options -Indexes
```

```apache
# 悪い例：ディレクトリ一覧が表示される状態（セキュリティリスク）
Options +Indexes

# 良い例：ディレクトリ一覧を無効化
Options -Indexes
```

---

## セキュリティヘッダーの設定

```apache
<IfModule mod_headers.c>
    # クリックジャッキング対策
    Header always set X-Frame-Options "SAMEORIGIN"

    # XSS対策
    Header always set X-Content-Type-Options "nosniff"

    # リファラー制御
    Header always set Referrer-Policy "strict-origin-when-cross-origin"

    # HSTS（HTTPSを強制）
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</IfModule>
```

> 参照: [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)

---

## カスタムエラーページ

```apache
# エラーページの指定
ErrorDocument 404 /errors/404.html
ErrorDocument 403 /errors/403.html
ErrorDocument 500 /errors/500.html
ErrorDocument 503 /errors/maintenance.html
```

---

## トラブルシューティング

### 500 Internal Server Error が出る場合

- `.htaccess`の構文エラーが原因であることが多い
- Apacheのエラーログを確認する

```bash
# エラーログの確認
tail -f /var/log/apache2/error.log     # Debian/Ubuntu
tail -f /var/log/httpd/error_log       # CentOS/RHEL
```

### リダイレクトループが発生する場合

```apache
# 悪い例：条件なしでリダイレクトするとループする
RewriteRule ^(.*)$ https://example.com/$1 [R=301,L]

# 良い例：条件を付けてループを防ぐ
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

### mod_rewriteが動作しない場合

1. `mod_rewrite`が有効になっているか確認
2. `AllowOverride All`が設定されているか確認
3. `.htaccess`ファイルのパーミッション（644推奨）を確認

```bash
# mod_rewriteの有効化（Debian/Ubuntu）
a2enmod rewrite
systemctl restart apache2

# .htaccessのパーミッション確認
ls -la .htaccess
# -rw-r--r-- であること
```

---

## まとめ

- `.htaccess`はApache専用の機能、Nginxでは使えない
- リダイレクト、認証、キャッシュ制御、アクセス制限など幅広い用途がある
- レンタルサーバーでは唯一のサーバー設定手段になることが多い
- パフォーマンスの観点からは、可能なら`httpd.conf`への記述が望ましい
- 構文エラーはサイト全体のダウンにつながるため、変更時はバックアップを取ること
- Apache 2.4以降の構文を使うこと（2.2の`Order/Allow/Deny`は非推奨）
