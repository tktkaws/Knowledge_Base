# HTTPヘッダーの基礎 — Cache-Control・Content-Security-Policy・CORS

## 概要

- HTTPヘッダーは、クライアント（ブラウザ）とサーバーの間でやり取りされるメタ情報
- リクエストヘッダー（ブラウザ → サーバー）とレスポンスヘッダー（サーバー → ブラウザ）がある
- キャッシュ制御、セキュリティ、アクセス制御など、Webサイトの動作に大きく関わる
- フロントエンドエンジニアが知っておくべき主要なヘッダーを解説する

> 参照: [MDN — HTTP Headers](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers)

---

## HTTPヘッダーの確認方法

### ブラウザの開発者ツール

1. F12（またはCommand+Option+I）で開発者ツールを開く
2. Networkタブを選択
3. 任意のリクエストをクリック
4. Headersセクションで確認

### コマンドラインで確認

```bash
# レスポンスヘッダーのみ表示
curl -I https://example.com

# リクエスト・レスポンスの両方を表示
curl -v https://example.com
```

---

## Cache-Control — キャッシュ制御

### 役割

- ブラウザやCDNがレスポンスをキャッシュするかどうか、どのくらいの期間キャッシュするかを指定する
- 適切に設定することで表示速度の向上とサーバー負荷の軽減が実現できる

### 主要なディレクティブ

| ディレクティブ | 意味 |
|--------------|------|
| `public` | 誰でもキャッシュ可能（CDN・ブラウザ） |
| `private` | ブラウザのみキャッシュ可能（CDNはキャッシュしない） |
| `no-cache` | キャッシュするが、使用前にサーバーに再検証する |
| `no-store` | 一切キャッシュしない |
| `max-age=秒数` | キャッシュの有効期間（秒） |
| `s-maxage=秒数` | CDN・プロキシ向けのキャッシュ有効期間 |
| `immutable` | キャッシュ期間中は再検証しない |
| `must-revalidate` | キャッシュ期限切れ後は必ずサーバーに問い合わせる |
| `stale-while-revalidate=秒数` | 期限切れ後もバックグラウンドで再検証しつつ古いキャッシュを返す |

### よくある設定パターン

```
# HTMLファイル — キャッシュしない（常に最新を取得）
Cache-Control: no-cache, no-store, must-revalidate

# CSS/JSファイル — 長期キャッシュ（ファイル名にハッシュを含める前提）
Cache-Control: public, max-age=31536000, immutable

# APIレスポンス — ユーザー固有データ
Cache-Control: private, no-cache

# 静的画像 — CDNでもキャッシュ
Cache-Control: public, max-age=604800, stale-while-revalidate=86400
```

### キャッシュバスティング

```html
<!-- 悪い例：ファイル名が同じだとキャッシュが効いて更新が反映されない -->
<link rel="stylesheet" href="/css/style.css">

<!-- 良い例：ハッシュ付きファイル名でキャッシュを確実に制御 -->
<link rel="stylesheet" href="/css/style.a1b2c3d4.css">
```

- ビルドツール（Vite、webpackなど）がファイル名にハッシュを自動付与する
- ファイル内容が変わればハッシュが変わり、新しいキャッシュとして取得される

### no-cacheとno-storeの違い

```
# no-cache：キャッシュは保存するが、使う前に毎回サーバーに確認する
Cache-Control: no-cache

# no-store：キャッシュを一切保存しない（個人情報を含むページなど）
Cache-Control: no-store

# 併用することが多い（最も厳格）
Cache-Control: no-cache, no-store, must-revalidate
```

> 参照: [MDN — Cache-Control](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Cache-Control)

---

## Content-Security-Policy（CSP） — コンテンツセキュリティポリシー

### 役割

- ページ内で読み込み可能なリソースの出所を制限するセキュリティヘッダー
- XSS（クロスサイトスクリプティング）攻撃のリスクを大幅に軽減する
- 許可されていないソースからのスクリプト・スタイル・画像等の読み込みをブロックする

### 基本構文

```
Content-Security-Policy: ディレクティブ 値1 値2; ディレクティブ 値;
```

### 主要なディレクティブ

| ディレクティブ | 制御対象 |
|--------------|---------|
| `default-src` | 他のディレクティブのフォールバック |
| `script-src` | JavaScript |
| `style-src` | CSS |
| `img-src` | 画像 |
| `font-src` | Webフォント |
| `connect-src` | XHR / Fetch / WebSocket |
| `media-src` | 音声・動画 |
| `frame-src` | iframe |
| `frame-ancestors` | このページをiframeで埋め込める親ページ |
| `form-action` | フォームの送信先 |
| `base-uri` | `<base>`タグのURL |

### 値の種類

| 値 | 意味 |
|---|------|
| `'self'` | 同一オリジンのみ許可 |
| `'none'` | すべて拒否 |
| `'unsafe-inline'` | インラインのscript/style を許可（非推奨） |
| `'unsafe-eval'` | eval()を許可（非推奨） |
| `'nonce-ランダム値'` | 特定のnonceを持つインラインスクリプトのみ許可 |
| `'strict-dynamic'` | 信頼済みスクリプトが読み込んだスクリプトも許可 |
| `https:` | HTTPS経由のみ許可 |
| `data:` | data:URIを許可 |
| `具体的なドメイン` | 指定ドメインからの読み込みを許可 |

### 設定例

```
# 厳格な設定
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self';

# Google Analyticsを許可する場合
Content-Security-Policy: default-src 'self'; script-src 'self' https://www.googletagmanager.com https://www.google-analytics.com; img-src 'self' https://www.google-analytics.com;

# Google Fontsを許可する場合
Content-Security-Policy: default-src 'self'; style-src 'self' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com;
```

### nonceを使った安全なインラインスクリプト

```html
<!-- サーバーが毎回ランダムなnonceを生成してヘッダーとHTMLの両方に埋め込む -->

<!-- レスポンスヘッダー -->
<!-- Content-Security-Policy: script-src 'nonce-abc123def456' -->

<!-- HTML -->
<script nonce="abc123def456">
  console.log("このスクリプトは許可される");
</script>

<!-- nonceのないスクリプトはブロックされる -->
<script>
  console.log("このスクリプトはブロックされる");
</script>
```

### Report-Only モード（テスト用）

```
# 違反をレポートするだけで、実際にはブロックしない
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report;
```

- 本番適用前にReport-Onlyモードでテストし、問題がないことを確認してから適用する

```
# 悪い例：いきなり厳格なCSPを本番に適用 → サイトが動かなくなるリスク
Content-Security-Policy: default-src 'self';

# 良い例：まずReport-Onlyで様子を見る
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report;
```

> 参照: [MDN — Content-Security-Policy](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Content-Security-Policy)

---

## CORS — Cross-Origin Resource Sharing

### 役割

- 異なるオリジン間でのリソース共有を制御する仕組み
- デフォルトでは、ブラウザは異なるオリジンへのリクエストを制限する（同一オリジンポリシー）
- CORSヘッダーにより、許可されたオリジンからのリクエストだけを通す

### オリジンとは

```
https://example.com:443/path
└─スキーム─┘└─ホスト──┘└ポート┘

# 同一オリジン
https://example.com/page1  と  https://example.com/page2  → 同一

# 異なるオリジン（以下はすべて異なるオリジン）
https://example.com  と  http://example.com     → スキームが異なる
https://example.com  と  https://api.example.com → ホストが異なる
https://example.com  と  https://example.com:8080 → ポートが異なる
```

### 主要なCORSヘッダー

| ヘッダー | 役割 |
|---------|------|
| `Access-Control-Allow-Origin` | 許可するオリジン |
| `Access-Control-Allow-Methods` | 許可するHTTPメソッド |
| `Access-Control-Allow-Headers` | 許可するリクエストヘッダー |
| `Access-Control-Allow-Credentials` | Cookieの送信を許可するか |
| `Access-Control-Max-Age` | プリフライトの結果をキャッシュする秒数 |
| `Access-Control-Expose-Headers` | JSからアクセス可能なレスポンスヘッダー |

### 単純リクエストとプリフライトリクエスト

```
【単純リクエスト】
GET / POST（条件付き）
→ ブラウザが直接リクエストを送り、レスポンスのCORSヘッダーを確認

【プリフライトリクエスト】
PUT / DELETE / カスタムヘッダー付き
→ ブラウザがまずOPTIONSリクエストを送り、許可を確認してから本リクエストを送る
```

```
# プリフライトの流れ
1. ブラウザ → サーバー: OPTIONS /api/data
   Origin: https://frontend.example.com
   Access-Control-Request-Method: PUT
   Access-Control-Request-Headers: Content-Type, Authorization

2. サーバー → ブラウザ: 200 OK
   Access-Control-Allow-Origin: https://frontend.example.com
   Access-Control-Allow-Methods: GET, POST, PUT, DELETE
   Access-Control-Allow-Headers: Content-Type, Authorization
   Access-Control-Max-Age: 86400

3. ブラウザ → サーバー: PUT /api/data （本リクエスト）
```

### CORS設定例（サーバー側）

#### Nginxの場合

```nginx
location /api/ {
    # 特定のオリジンを許可
    add_header Access-Control-Allow-Origin "https://frontend.example.com" always;
    add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
    add_header Access-Control-Allow-Headers "Content-Type, Authorization" always;
    add_header Access-Control-Max-Age 86400 always;

    # プリフライトリクエスト（OPTIONS）への応答
    if ($request_method = OPTIONS) {
        return 204;
    }

    proxy_pass http://backend;
}
```

#### Expressの場合

```javascript
// corsミドルウェアを使用（推奨）
const cors = require('cors');

app.use(cors({
  origin: 'https://frontend.example.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 86400,
}));
```

### よくあるCORSエラーと対処法

```
# エラー例
Access to fetch at 'https://api.example.com/data' from origin
'https://frontend.example.com' has been blocked by CORS policy:
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

| 原因 | 対処法 |
|------|--------|
| サーバーにCORSヘッダーがない | サーバー側で`Access-Control-Allow-Origin`を設定 |
| Originが一致しない | 許可するオリジンを正確に指定 |
| Credentialsを送りたいのにワイルドカード | `*`ではなく具体的なオリジンを指定 |
| カスタムヘッダーが許可されていない | `Access-Control-Allow-Headers`に追加 |

```
# 悪い例：すべてのオリジンを許可（セキュリティリスク）
Access-Control-Allow-Origin: *

# 良い例：必要なオリジンだけ許可
Access-Control-Allow-Origin: https://frontend.example.com
```

```
# 悪い例：ワイルドカードとCredentialsの併用（ブラウザがブロックする）
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true

# 良い例：具体的なオリジンとCredentialsの併用
Access-Control-Allow-Origin: https://frontend.example.com
Access-Control-Allow-Credentials: true
```

> 参照: [MDN — CORS](https://developer.mozilla.org/ja/docs/Web/HTTP/CORS)

---

## その他の重要なHTTPヘッダー

### X-Frame-Options

- ページをiframe内で表示することを制限する
- クリックジャッキング攻撃の防止

```
X-Frame-Options: DENY            # すべてのiframe埋め込みを拒否
X-Frame-Options: SAMEORIGIN      # 同一オリジンからのみ許可
```

> CSPの`frame-ancestors`の方が新しく、より細かい制御が可能

### X-Content-Type-Options

- ブラウザのMIMEタイプスニッフィング（推測）を防止する

```
X-Content-Type-Options: nosniff
```

- 常に設定すべきヘッダー。設定しない理由がない

### Strict-Transport-Security（HSTS）

- ブラウザに対して、以後のアクセスをすべてHTTPSにするよう指示する

```
# 1年間HTTPSを強制（サブドメイン含む）
Strict-Transport-Security: max-age=31536000; includeSubDomains

# HSTSプリロードリスト登録用
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

> 参照: [MDN — Strict-Transport-Security](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Strict-Transport-Security)

### Referrer-Policy

- リクエスト時に送信されるリファラー情報を制御する

```
# 同一オリジンには完全なURLを、異なるオリジンにはオリジンのみ送信
Referrer-Policy: strict-origin-when-cross-origin

# リファラーを一切送信しない
Referrer-Policy: no-referrer
```

| 値 | 動作 |
|---|------|
| `no-referrer` | リファラーを送信しない |
| `origin` | オリジンのみ送信（パスを含めない） |
| `same-origin` | 同一オリジンのみリファラーを送信 |
| `strict-origin-when-cross-origin` | 推奨。クロスオリジンではオリジンのみ送信 |

### Permissions-Policy（旧Feature-Policy）

- ブラウザAPIの使用を制限する

```
# カメラとマイクを無効化、位置情報は自サイトのみ許可
Permissions-Policy: camera=(), microphone=(), geolocation=(self)
```

> 参照: [MDN — Permissions-Policy](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Permissions-Policy)

---

## レスポンスヘッダーの推奨設定まとめ

### Nginxでの設定例

```nginx
server {
    # セキュリティヘッダー
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=(self)" always;

    # CSP（サイトに合わせて調整）
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self';" always;

    # キャッシュ（HTMLファイル）
    location ~* \.html$ {
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }

    # キャッシュ（静的アセット）
    location ~* \.(css|js|woff2|png|jpg|svg)$ {
        add_header Cache-Control "public, max-age=31536000, immutable";
    }
}
```

### Apacheでの設定例

```apache
<IfModule mod_headers.c>
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set Permissions-Policy "camera=(), microphone=(), geolocation=(self)"
</IfModule>
```

---

## まとめ

| ヘッダー | 用途 | 必須度 |
|---------|------|--------|
| `Cache-Control` | キャッシュ制御 | 必須 |
| `Content-Security-Policy` | XSS防止 | 強く推奨 |
| `Access-Control-Allow-Origin` | CORS制御 | API提供時は必須 |
| `Strict-Transport-Security` | HTTPS強制 | HTTPS対応サイトは必須 |
| `X-Content-Type-Options` | MIMEスニッフィング防止 | 必須 |
| `X-Frame-Options` | クリックジャッキング防止 | 強く推奨 |
| `Referrer-Policy` | リファラー制御 | 推奨 |
| `Permissions-Policy` | ブラウザAPI制限 | 推奨 |

- HTTPヘッダーはWebサイトのパフォーマンスとセキュリティの両方に関わる
- 特にCache-Control、CSP、CORSの3つはフロントエンド開発で頻繁に出会う
- セキュリティヘッダーは「設定しない理由がない」ものがほとんど
- 本番適用前にはブラウザの開発者ツールやReport-Onlyモードで動作確認すること
