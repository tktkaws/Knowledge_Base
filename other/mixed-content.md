# Mixed Contentの原因と解消方法 — HTTPS移行時の落とし穴

## Mixed Contentとは

- HTTPSページ内でHTTPのリソースを読み込んでいる状態
- HTTPSで暗号化されたページに、暗号化されていないHTTPリソースが混在すること
- ブラウザが警告やブロックを行い、ページの表示や機能に影響を与える

```
https://example.com/page.html
  ├─ https://example.com/style.css     ← OK（HTTPS）
  ├─ https://cdn.example.com/app.js    ← OK（HTTPS）
  ├─ http://example.com/image.png      ← Mixed Content（HTTP）
  └─ http://other.com/script.js        ← Mixed Content（HTTP）
```

- 参照: [MDN — Mixed Content](https://developer.mozilla.org/ja/docs/Web/Security/Mixed_content)

## なぜ問題なのか

### セキュリティ上のリスク

```
HTTPSページ + HTTPリソース の組み合わせ:

ブラウザ ←── HTTPS（暗号化） ──── サーバー（ページ本体）
ブラウザ ←── HTTP（平文）    ──── サーバー（画像/スクリプト）
                ↑
              第三者が盗聴・改ざん可能
```

| リスク | 内容 |
|---|---|
| スクリプトの改ざん | HTTPで読み込まれたJSが改ざんされ、悪意あるコードが実行される |
| コンテンツの差し替え | HTTPで読み込まれた画像やテキストが第三者に差し替えられる |
| Cookie/セッションの漏洩 | HTTPリクエストにCookieが付与され、平文で送信される |
| HTTPS の意味がなくなる | ページ自体が暗号化されていても、混在リソースから攻撃可能 |

## Mixed Contentの種類

### Active Mixed Content（能動的混在コンテンツ）

- ページの動作やDOM構造に影響を与えるリソース
- **ブラウザが自動的にブロックする**（読み込まれない）

| リソース | HTMLでの記述例 |
|---|---|
| JavaScript | `<script src="http://...">` |
| CSS | `<link rel="stylesheet" href="http://...">` |
| iframe | `<iframe src="http://...">` |
| XMLHttpRequest / Fetch | `fetch("http://...")` |
| Web Fonts（CSS内） | `@font-face { src: url("http://...") }` |
| `<object>`, `<embed>` | プラグインコンテンツ |

### Passive Mixed Content（受動的混在コンテンツ）

- ページの表示に影響するが、DOM操作はできないリソース
- **ブラウザが警告を表示するが、読み込みは許可される**（Chrome 80以降はブロック対象が拡大）

| リソース | HTMLでの記述例 |
|---|---|
| 画像 | `<img src="http://...">` |
| 音声 | `<audio src="http://...">` |
| 動画 | `<video src="http://...">` |

### ブラウザの動作まとめ

```
Active Mixed Content（JS, CSS, iframe など）:
  → 自動ブロック。コンソールにエラー表示
  → ページの機能が壊れる可能性が高い

Passive Mixed Content（画像, 音声, 動画）:
  → 警告表示。アドレスバーの鍵アイコンが「安全ではない」に変化
  → 最新のChrome/Firefoxではブロックされる場合もある
```

- 参照: [Chrome — Mixed Content の段階的ブロック](https://blog.chromium.org/2019/10/no-more-mixed-messages-about-https.html)

## Mixed Contentの検出方法

### ブラウザの開発者ツール

```
Chrome DevTools:
  1. F12（または右クリック→検証）で開発者ツールを開く
  2. Consoleタブで Mixed Content の警告・エラーを確認

警告の例:
  Mixed Content: The page at 'https://example.com/' was loaded over HTTPS,
  but requested an insecure image 'http://example.com/image.png'.
  This request has been blocked; the content must be served over HTTPS.

  Mixed Content: The page at 'https://example.com/' was loaded over HTTPS,
  but requested an insecure script 'http://example.com/script.js'.
  This request was automatically upgraded to HTTPS.
```

### Networkタブでの確認

```
Chrome DevTools → Network タブ:
  1. ページを再読み込み
  2. フィルタで "mixed-content:displayed" または "mixed-content:blocked" を検索
  3. ブロックされたリソースは赤字で表示される
```

### Security タブでの確認

```
Chrome DevTools → Security タブ:
  - 「This page is not secure.」と表示される
  - Mixed Contentの詳細が表示される
  - 問題のあるリソースの一覧を確認できる
```

### コマンドラインでの一括検出

```bash
# HTMLファイル内のHTTP参照を検索
grep -rn 'http://' --include='*.html' --include='*.css' --include='*.js' /path/to/site/

# よくあるパターン
grep -rn 'src="http://' --include='*.html' /path/to/site/
grep -rn 'href="http://' --include='*.html' /path/to/site/
grep -rn "url('http://" --include='*.css' /path/to/site/
grep -rn 'url("http://' --include='*.css' /path/to/site/
```

### オンラインツール

```
Why No Padlock:
  https://www.whynopadlock.com/
  → URLを入力するとMixed Contentを検出してくれる

Lighthouse（Chrome DevTools内蔵）:
  → Best Practices の項目でMixed Contentを指摘
```

## Mixed Contentの修正方法

### 1. HTTPをHTTPSに書き換える

```html
<!-- 悪い例: HTTPでリソースを読み込み -->
<img src="http://example.com/image.png">
<script src="http://cdn.example.com/library.js"></script>
<link rel="stylesheet" href="http://fonts.googleapis.com/css?family=Roboto">

<!-- 良い例: HTTPSに変更 -->
<img src="https://example.com/image.png">
<script src="https://cdn.example.com/library.js"></script>
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto">
```

- 外部リソースがHTTPSに対応しているか事前に確認する
- 主要なCDN（Google Fonts、jQuery CDN、cdnjs等）はすべてHTTPS対応済み

### 2. プロトコル相対URLを使う

```html
<!-- プロトコル相対URL（ページのプロトコルに追従） -->
<img src="//example.com/image.png">
<script src="//cdn.example.com/library.js"></script>
```

- プロトコル相対URLはページのプロトコルに合わせてHTTP/HTTPSを自動選択
- ただし現在はHTTPSが標準のため、明示的に`https://`を書く方が望ましい
- Paul Irishが2014年に「プロトコル相対URLはアンチパターン」と指摘

### 3. 相対パスを使う（同一ドメインの場合）

```html
<!-- 悪い例: 絶対URLでHTTPを指定 -->
<img src="http://example.com/images/logo.png">
<a href="http://example.com/about">About</a>

<!-- 良い例: 相対パスを使用 -->
<img src="/images/logo.png">
<a href="/about">About</a>
```

- 同一ドメインのリソースは相対パスにすることで、プロトコルに依存しなくなる
- 最も確実なMixed Content防止策

### 4. CSS内のHTTP参照を修正

```css
/* 悪い例 */
@font-face {
  font-family: 'MyFont';
  src: url('http://example.com/fonts/myfont.woff2');
}

.hero {
  background-image: url('http://example.com/images/hero.jpg');
}

/* 良い例 */
@font-face {
  font-family: 'MyFont';
  src: url('https://example.com/fonts/myfont.woff2');
}

.hero {
  background-image: url('/images/hero.jpg');
}
```

### 5. JavaScript内のHTTP参照を修正

```javascript
// 悪い例
fetch('http://api.example.com/data')
const img = new Image()
img.src = 'http://example.com/image.png'

// 良い例
fetch('https://api.example.com/data')
const img = new Image()
img.src = 'https://example.com/image.png'

// 動的にプロトコルを合わせる場合
const apiUrl = `${window.location.protocol}//api.example.com/data`
```

### 6. CMSやデータベース内のURLを一括修正

```sql
-- WordPressの場合: データベース内のHTTP URLをHTTPSに置換
-- wp_options テーブル
UPDATE wp_options SET option_value = REPLACE(option_value, 'http://example.com', 'https://example.com')
WHERE option_name = 'home' OR option_name = 'siteurl';

-- wp_posts テーブル（記事本文内のURL）
UPDATE wp_posts SET post_content = REPLACE(post_content, 'http://example.com', 'https://example.com');

-- wp_postmeta テーブル
UPDATE wp_postmeta SET meta_value = REPLACE(meta_value, 'http://example.com', 'https://example.com');
```

- SQLを直接実行する前にバックアップを必ず取得する
- WordPressではSearch Replace DBなどのツールを使うのが安全

## Content-Security-Policyによる対策

### upgrade-insecure-requests

```html
<!-- HTTPのリソースをブラウザが自動的にHTTPSに変換して読み込む -->
<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
```

```nginx
# Nginxでレスポンスヘッダとして設定
add_header Content-Security-Policy "upgrade-insecure-requests" always;
```

```apache
# Apacheでレスポンスヘッダとして設定
Header always set Content-Security-Policy "upgrade-insecure-requests"
```

| 動作 | 説明 |
|---|---|
| 対象 | ページ内のすべてのHTTPリクエスト（画像、CSS、JS、フォント等） |
| 変換 | `http://`を`https://`に自動変換してからリクエスト |
| 制限 | 外部リソースがHTTPSに対応していない場合は読み込み失敗 |
| 利点 | HTML/CSS/JSの修正なしにMixed Contentを解消できる応急処置 |

- HTTPS移行の過渡期に有効な対策
- 最終的にはソースコード自体のURLを修正するのが望ましい

### block-all-mixed-content

```html
<!-- すべてのMixed Contentをブロック（Passive含む） -->
<meta http-equiv="Content-Security-Policy" content="block-all-mixed-content">
```

- Passive Mixed Content（画像等）も含めてすべてブロックする
- `upgrade-insecure-requests`の方が実用的なため、こちらはあまり使われない

- 参照: [MDN — Content-Security-Policy: upgrade-insecure-requests](https://developer.mozilla.org/ja/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/upgrade-insecure-requests)

## HTTPS移行時のチェックリスト

### 移行前

```
□ SSL証明書の取得と設置
□ HTTPSでサイトが正常に表示されることを確認
□ ステージング環境でMixed Contentがないか確認
```

### ソースコードの修正

```
□ HTML内のhttp://参照をhttps://または相対パスに変更
□ CSS内のurl()でhttp://参照を修正
□ JavaScript内のhttp://参照を修正
□ 外部リソース（CDN、API）がHTTPS対応していることを確認
□ CMSのサイトURLをHTTPSに変更
□ データベース内のURLを一括置換
```

### サーバー設定

```
□ HTTP→HTTPSの301リダイレクトを設定
□ HSTSヘッダを設定（段階的に）
□ Content-Security-Policy: upgrade-insecure-requests を設定（移行期の安全策）
```

### 確認

```
□ 主要ページでDevToolsのConsoleにMixed Content警告がないか確認
□ SSL Labsテストでグレードを確認
□ Google Search ConsoleにHTTPS版を登録
□ sitemap.xmlのURLをHTTPSに更新
□ canonical URLをHTTPSに更新
□ 外部サービス（Google Analytics、広告タグ等）のURL設定を確認
```

## よくある原因と解消パターン

### 1. 画像のURL（最も多い）

```
原因: CMSで過去に投稿した記事内の画像URLがhttp://のまま
対処: データベースの一括置換 + 新規投稿はHTTPSまたは相対パスで
```

### 2. 外部ウィジェット・埋め込み

```
原因: Google Maps、YouTube、SNSボタンなどの埋め込みコードがHTTP
対処: 各サービスの最新の埋め込みコードに更新（現在はほぼHTTPS対応済み）
```

### 3. 広告タグ

```
原因: 広告ネットワークのスクリプトがHTTP
対処: 広告プロバイダに確認し、HTTPS対応版のタグに更新
```

### 4. カスタムフォント

```
原因: Google FontsなどのCSSリンクがHTTP
対処: https://fonts.googleapis.com/ に変更（以前はHTTPデフォルトだった）
```

### 5. ハードコードされた絶対URL

```
原因: テンプレートや設定ファイルに http://example.com がハードコードされている
対処: 設定ファイルやテンプレートを検索して一括修正
```

## まとめ

- Mixed ContentはHTTPSページ内にHTTPリソースが混在する状態で、セキュリティを損なう
- Active Mixed Content（JS, CSS等）はブラウザが自動ブロック、Passive（画像等）は警告表示
- DevToolsのConsole/Network/Securityタブで検出できる
- 修正の基本は、HTTP参照をHTTPSまたは相対パスに書き換えること
- `upgrade-insecure-requests`は移行期の応急処置として有効
- CMS利用時はデータベース内のURLの一括置換も必要
- 移行時はチェックリストに沿って漏れなく対応し、DevToolsで最終確認する
