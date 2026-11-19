# Webサイトのセキュリティヘッダー設定 — CSP・X-Frame-Options・HSTS

## セキュリティヘッダーとは

- HTTPレスポンスヘッダーの中で、ブラウザのセキュリティ機能を制御するもの
- サーバーからブラウザに「このページはこういうルールで表示してください」と指示する仕組み
- 適切に設定するだけでXSS・クリックジャッキング・プロトコルダウングレードなどの攻撃を緩和できる
- 設定コストが低い割に効果が高く、「設定しない理由がない」ヘッダーが多い

> 参照: [OWASP — HTTP Security Response Headers](https://owasp.org/www-project-secure-headers/)

---

## Content-Security-Policy（CSP）

### 概要

- ページ内で読み込み可能なリソースの出所（オリジン）を制限するヘッダー
- XSS攻撃の被害を大幅に軽減する最も強力なセキュリティヘッダー
- 許可されていないソースからのスクリプト・スタイル・画像等の読み込みをブラウザがブロックする

### 基本構文

```
Content-Security-Policy: ディレクティブ 値1 値2; ディレクティブ 値;
```

### 主要なディレクティブ

| ディレクティブ | 制御対象 |
|--------------|---------|
| `default-src` | 他のディレクティブが未指定の場合のフォールバック |
| `script-src` | JavaScriptファイル・インラインスクリプト |
| `style-src` | CSSファイル・インラインスタイル |
| `img-src` | 画像（`<img>`、CSS `background-image`） |
| `font-src` | Webフォント |
| `connect-src` | XHR / Fetch / WebSocket の接続先 |
| `frame-src` | `<iframe>`で埋め込むコンテンツ |
| `frame-ancestors` | このページを`<iframe>`で埋め込める親ページ |
| `form-action` | `<form>`の送信先 |
| `base-uri` | `<base>`タグで指定できるURL |
| `object-src` | `<object>`・`<embed>`・`<applet>` |

### ソース値の種類

| 値 | 意味 |
|---|------|
| `'self'` | 同一オリジンのみ許可 |
| `'none'` | すべて拒否 |
| `'unsafe-inline'` | インラインのscript / styleを許可（非推奨） |
| `'unsafe-eval'` | `eval()`等の動的コード実行を許可（非推奨） |
| `'nonce-ランダム値'` | 指定nonceを持つインライン要素のみ許可 |
| `'strict-dynamic'` | 信頼済みスクリプトが読み込んだスクリプトも許可 |
| `https:` | HTTPS経由のリソースのみ許可 |
| `data:` | `data:`URIを許可 |
| `具体的なドメイン` | 指定ドメインからのリソースを許可 |

### 設定例

```
# 最小限の厳格な設定
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; object-src 'none'; frame-ancestors 'none';

# Google Analyticsとフォントを許可する実用的な設定
Content-Security-Policy: default-src 'self'; script-src 'self' https://www.googletagmanager.com; style-src 'self' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: https://www.google-analytics.com; object-src 'none';
```

### nonceベースのCSP

- インラインスクリプトを安全に許可する方法
- サーバーがリクエストごとにランダムなnonceを生成し、ヘッダーとHTML両方に埋め込む

```html
<!-- サーバーが毎回異なるnonceを生成 -->
<!-- レスポンスヘッダー: Content-Security-Policy: script-src 'nonce-abc123' -->

<!-- 許可される（nonceが一致） -->
<script nonce="abc123">
  console.log('このスクリプトは実行される');
</script>

<!-- ブロックされる（nonceがない） -->
<script>
  console.log('このスクリプトはブロックされる');
</script>
```

### Report-Onlyモード

```
# 違反をレポートするだけで、実際にはブロックしない（テスト用）
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report;
```

```
# 悪い例：いきなり厳格なCSPを本番に適用 → サイトが壊れるリスク
Content-Security-Policy: default-src 'self';

# 良い例：まずReport-Onlyで違反を収集し、問題がないことを確認してから適用
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report;
```

- Report-Onlyで1〜2週間様子を見てから本番適用するのが安全

> 参照: [MDN — Content-Security-Policy](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Content-Security-Policy)

---

## X-Frame-Options

### 概要

- 自サイトのページが`<iframe>`内に埋め込まれることを制限するヘッダー
- クリックジャッキング攻撃の防止が主な目的
- CSPの`frame-ancestors`ディレクティブと役割が重複するが、古いブラウザ対応のために併用が推奨

### クリックジャッキングとは

```
攻撃の仕組み:

1. 攻撃者が自分のページに透明なiframeで標的サイトを埋め込む
2. 標的サイトの「削除」ボタンの上に「当選おめでとう！クリック！」を重ねる
3. ユーザーは「当選」ボタンをクリックしたつもりが、実際には標的サイトの「削除」ボタンを押す

攻撃者のページ:
┌──────────────────────────────┐
│  当選おめでとう！          │
│  ┌────────────────────┐      │
│  │ ここをクリック！    │ ← 見えるボタン
│  └────────────────────┘      │
│  ┌────────────────────┐      │
│  │ アカウント削除      │ ← 透明なiframeの中（見えない）
│  └────────────────────┘      │
└──────────────────────────────┘
```

### 設定値

| 値 | 動作 |
|---|------|
| `DENY` | すべてのiframe埋め込みを拒否 |
| `SAMEORIGIN` | 同一オリジンからの埋め込みのみ許可 |

```
# 悪い例：X-Frame-Optionsを設定しない → クリックジャッキングのリスク

# 良い例：iframe埋め込みを完全に禁止
X-Frame-Options: DENY

# 良い例：同一オリジンからの埋め込みだけ許可（自サイト内でiframeを使う場合）
X-Frame-Options: SAMEORIGIN
```

### CSP frame-ancestorsとの関係

```
# X-Frame-Options（古い仕様、制御が限定的）
X-Frame-Options: SAMEORIGIN

# CSP frame-ancestors（新しい仕様、より柔軟な制御が可能）
Content-Security-Policy: frame-ancestors 'self' https://trusted.example.com;
```

- `frame-ancestors`は特定のドメインを複数指定できる（X-Frame-Optionsにはない機能）
- 両方を設定した場合、CSPが優先される
- 古いブラウザ対応のため、両方を設定するのがベストプラクティス

> 参照: [MDN — X-Frame-Options](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/X-Frame-Options)

---

## Strict-Transport-Security（HSTS）

### 概要

- ブラウザに対して「以後のアクセスはすべてHTTPSで行うように」と指示するヘッダー
- 初回のHTTP→HTTPSリダイレクト後、ブラウザが指定期間中HTTPSを強制する
- 中間者攻撃（MITM）によるHTTPへのダウングレード攻撃を防止

### SSLストリッピング攻撃

```
HSTSなしの場合:
ユーザー → http://example.com → 攻撃者が傍受 → 偽のHTTPページを返す

HSTSありの場合:
ユーザー → ブラウザが自動的にhttps://に変換 → 攻撃者が介入できない
```

### 設定値

| パラメータ | 意味 |
|-----------|------|
| `max-age=秒数` | HSTSを有効にする期間（秒） |
| `includeSubDomains` | サブドメインにもHSTSを適用 |
| `preload` | ブラウザのプリロードリスト登録を申請する意思表示 |

### 設定例

```
# 基本的な設定（1年間、サブドメイン含む）
Strict-Transport-Security: max-age=31536000; includeSubDomains

# プリロードリスト登録用（最も強力）
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

### 段階的な導入

```
# 悪い例：いきなりmax-ageを1年に設定 → 設定ミス時に長期間アクセス不能になるリスク

# 良い例：段階的にmax-ageを伸ばす
# ステップ1: 5分（テスト）
Strict-Transport-Security: max-age=300

# ステップ2: 1週間（様子見）
Strict-Transport-Security: max-age=604800

# ステップ3: 1ヶ月
Strict-Transport-Security: max-age=2592000; includeSubDomains

# ステップ4: 1年（本番運用）
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### HSTSプリロードリスト

- ブラウザに事前登録されたHSTSサイトのリスト
- 初回アクセスからHTTPSが強制される（通常のHSTSは初回アクセス時は保護されない）
- 登録は https://hstspreload.org/ から申請
- 登録の取り消しには時間がかかるため、慎重に判断すること

### 注意点

- HTTPS対応が完全でない状態でHSTSを有効にすると、サイトにアクセスできなくなる
- サブドメインを含める場合、すべてのサブドメインがHTTPS対応であること
- `max-age=0`でHSTSを無効化できるが、すでにブラウザにキャッシュされた情報は`max-age`期間まで残る

> 参照: [MDN — Strict-Transport-Security](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Strict-Transport-Security)

---

## X-Content-Type-Options

### 概要

- ブラウザのMIMEタイプスニッフィング（推測）を防止するヘッダー
- `nosniff`を設定すると、サーバーが返した`Content-Type`以外でのリソース解釈を禁止する
- 設定値は`nosniff`の1つだけ

### MIMEスニッフィング攻撃

```
攻撃の仕組み:
1. 攻撃者がプロフィール画像として悪意のあるHTMLファイルをアップロード
2. サーバーはContent-Type: image/pngとして配信
3. X-Content-Type-Optionsがないと、ブラウザがHTML/JSとして解釈する可能性がある
4. スクリプトが実行され、XSSが成立

対策:
X-Content-Type-Options: nosniff
→ ブラウザはContent-Typeを信頼し、スニッフィングしない
```

### 設定

```
# 常に設定すべき（設定しない理由がない）
X-Content-Type-Options: nosniff
```

> 参照: [MDN — X-Content-Type-Options](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/X-Content-Type-Options)

---

## Referrer-Policy

### 概要

- ページ遷移やリソース読み込み時に送信されるリファラー（参照元URL）情報を制御するヘッダー
- URLにセッションIDやトークンが含まれる場合、外部サイトに漏洩するリスクがある
- 適切に設定して不要な情報の流出を防ぐ

### 主要な設定値

| 値 | 動作 |
|---|------|
| `no-referrer` | リファラーを一切送信しない |
| `origin` | オリジンのみ送信（パスを含めない） |
| `same-origin` | 同一オリジン間のみリファラーを送信 |
| `strict-origin-when-cross-origin` | 同一オリジンには完全URL、クロスオリジンにはオリジンのみ送信（推奨） |
| `no-referrer-when-downgrade` | HTTPSからHTTPへの遷移時のみリファラーを送信しない |

```
# 悪い例：Referrer-Policyを設定しない → デフォルト動作はブラウザ依存

# 良い例：推奨設定
Referrer-Policy: strict-origin-when-cross-origin

# 厳格な設定（外部サイトに一切情報を送りたくない場合）
Referrer-Policy: no-referrer
```

### URLにセンシティブ情報が含まれるケース

```
# リファラーで漏洩する可能性があるURL例
https://example.com/reset-password?token=abc123
https://example.com/admin/users/12345

# strict-origin-when-cross-originなら、外部サイトには以下のみ送信
https://example.com
（パスやクエリパラメータは含まれない）
```

> 参照: [MDN — Referrer-Policy](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Referrer-Policy)

---

## Permissions-Policy（旧Feature-Policy）

### 概要

- ブラウザAPIの使用許可をページ単位で制御するヘッダー
- カメラ、マイク、位置情報などのAPIを不要なページで無効化できる
- iframe内でのAPI利用を制限することもできる

### 主要な機能

| 機能名 | 対象API |
|-------|---------|
| `camera` | カメラ |
| `microphone` | マイク |
| `geolocation` | 位置情報 |
| `fullscreen` | フルスクリーン |
| `payment` | Payment Request API |
| `autoplay` | 自動再生 |
| `display-capture` | 画面キャプチャ |

### 設定例

```
# すべて無効化して必要なものだけ許可する方針
Permissions-Policy: camera=(), microphone=(), geolocation=(self), fullscreen=(self), payment=()

# 特定のiframe（外部サービス）にだけカメラを許可
Permissions-Policy: camera=(self "https://video-chat.example.com"), microphone=(self "https://video-chat.example.com")
```

```
# 悪い例：Permissions-Policyを設定しない → 埋め込んだiframeがAPIを自由に使える

# 良い例：使わないAPIは明示的に無効化
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
```

> 参照: [MDN — Permissions-Policy](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Permissions-Policy)

---

## サーバーごとの設定方法

### Nginxの場合

```nginx
server {
    # CSP
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; object-src 'none'; frame-ancestors 'none';" always;

    # クリックジャッキング対策
    add_header X-Frame-Options "DENY" always;

    # HTTPS強制
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # MIMEスニッフィング防止
    add_header X-Content-Type-Options "nosniff" always;

    # リファラー制御
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # ブラウザAPI制限
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=(self)" always;
}
```

- `always`を付けないとエラーレスポンス（4xx、5xx）にヘッダーが付かない

### Apacheの場合

```apache
<IfModule mod_headers.c>
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; object-src 'none'; frame-ancestors 'none';"
    Header always set X-Frame-Options "DENY"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Permissions-Policy "camera=(), microphone=(), geolocation=(self)"
</IfModule>
```

### Next.jsの場合

```js
// next.config.js
const securityHeaders = [
  { key: 'Content-Security-Policy', value: "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; object-src 'none'; frame-ancestors 'none';" },
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'Strict-Transport-Security', value: 'max-age=31536000; includeSubDomains' },
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
  { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=(self)' },
];

module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: securityHeaders,
      },
    ];
  },
};
```

---

## セキュリティヘッダーの確認方法

### ブラウザの開発者ツール

1. F12（またはCommand+Option+I）で開発者ツールを開く
2. Networkタブを選択
3. ページをリロードしてHTMLドキュメントのリクエストを選択
4. Headersセクションの「Response Headers」を確認

### コマンドラインで確認

```bash
# レスポンスヘッダーを確認
curl -I https://example.com

# 特定のヘッダーだけ抽出
curl -sI https://example.com | grep -i "content-security-policy\|x-frame-options\|strict-transport"
```

### オンラインツール

- [Security Headers](https://securityheaders.com/) — URLを入力するとセキュリティヘッダーの設定状況をA〜Fで評価
- [Mozilla Observatory](https://observatory.mozilla.org/) — Mozillaが提供するWebサイトセキュリティ診断ツール

---

## セキュリティヘッダー一覧と推奨設定

| ヘッダー | 目的 | 推奨設定 | 必須度 |
|---------|------|---------|--------|
| `Content-Security-Policy` | XSS防止 | `default-src 'self'`を基本に調整 | 強く推奨 |
| `X-Frame-Options` | クリックジャッキング防止 | `DENY` or `SAMEORIGIN` | 強く推奨 |
| `Strict-Transport-Security` | HTTPS強制 | `max-age=31536000; includeSubDomains` | HTTPS対応サイトは必須 |
| `X-Content-Type-Options` | MIMEスニッフィング防止 | `nosniff` | 必須 |
| `Referrer-Policy` | リファラー情報の制御 | `strict-origin-when-cross-origin` | 推奨 |
| `Permissions-Policy` | ブラウザAPI制限 | 使わないAPIを`()`で無効化 | 推奨 |

---

## まとめ

- セキュリティヘッダーはサーバー設定に数行追加するだけで、多くの攻撃を緩和できる
- CSPはXSS対策の要。Report-Onlyモードで段階的に導入する
- X-Frame-Optionsでクリックジャッキングを防止。CSP `frame-ancestors`との併用が理想
- HSTSはHTTPS対応サイトでは必須。段階的に`max-age`を伸ばして導入する
- X-Content-Type-Optionsの`nosniff`は常に設定する
- Security Headersなどのツールで定期的に設定状況を確認する
