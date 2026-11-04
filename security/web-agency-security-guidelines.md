# Web制作会社のセキュリティガイドライン

作成日・参考資料確認日：2026-10-13  
状態：社内導入用ドラフト  
対象：経験3年未満のフロントエンドエンジニア／静的サイト（HTML・CSS・バニラJavaScript）／WordPress

## 目的と使い方

制作途中のサイトが、テスト環境のBasic認証未設定により外部へ公開され、検索エンジンにインデックス登録されたインシデントを踏まえ、制作・公開・保守で必要なセキュリティ対策を定める。

本書は、[[Web制作会社のコーディングガイドライン]]と併せて利用する。記法や表示品質はコーディングガイドライン、本書は非公開情報の保護・安全な実装・運用を扱う。

「必須」は本書で提案する社内基準。「推奨」は案件に応じて追加する対策。「責任者判断」は制作者だけで決めず、案件の技術責任者・運用責任者が適用範囲を決める項目とする。実施できない必須項目は、理由・代替策・確認者を記録し、責任者の判断を受ける。

サーバー設定は利用環境に合った手順を責任者が用意する。設定権限がない担当者は依頼と結果確認を行う。契約上の保守範囲や担当の有無を、制作開始時に確定する。

## 公開されているセキュリティガイドライン・標準

当社の対象技術に関連する、IPA・WordPress公式・OWASPの一次資料を列挙する。制作会社独自の社内規定に限らず、開発者・運営者向けのガイドラインと検証標準を含む。

1. **IPA：安全なウェブサイトの作り方**【日本語／Web開発・運用】
   - [公開ページ・PDF・チェックリスト](https://www.ipa.go.jp/security/vuln/websecurity/about.html)
   - XSS、SQLインジェクション、CSRF、アクセス制御等の対策を扱う。掲載されている日本語版は改訂第7版第4刷（2021年3月31日公開）。新人教育の基礎資料にする。

2. **WordPress：Hardening WordPress**【英語／WordPress運用】
   - [公式ガイド](https://developer.wordpress.org/advanced-administration/security/hardening/)
   - 更新、認証、ファイル権限、バックアップなどを扱う。管理画面への制限等は機能への影響も確認して適用する。

3. **WordPress：Security — Common APIs Handbook**【英語／テーマ・独自機能の開発】
   - [公式ガイド](https://developer.wordpress.org/apis/security/)
   - 入力値の検証・サニタイズ、出力時のエスケープ等の考え方と個別APIの資料を提供する。

4. **OWASP：Web Frontend Security Cheat Sheet**【英語／フロントエンド】
   - [公開ガイド](https://cheatsheetseries.owasp.org/cheatsheets/Web_Frontend_Security_Cheat_Sheet.html)
   - ブラウザー側の実装で注意するデータの扱いなどを整理している。バニラJavaScriptのレビューに使う。

5. **OWASP：Cross Site Scripting Prevention Cheat Sheet**【英語／XSS対策】
   - [公開ガイド](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
   - HTML・属性・URL等の出力先に応じた安全な処理を扱う。innerHTMLやCMSデータの表示を確認するときに使う。

6. **OWASP：Cross-Site Request Forgery Prevention Cheat Sheet**【英語／CSRF対策】
   - [公開ガイド](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
   - 利用者の認証状態を悪用した操作への対策を扱う。フォームやデータ更新処理の設計に使う。

7. **OWASP：Secrets Management Cheat Sheet**【英語／認証情報・秘密情報】
   - [公開ガイド](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
   - 秘密情報の保管、権限、失効・更新等を扱う。Gitや納品物への認証情報混入を防ぐ運用に使う。

8. **OWASP：HTTP Headers Cheat Sheet**【英語／HTTP設定】
   - [公開ガイド](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)
   - CSP、HSTS、Content-Type、キャッシュ等を扱う。サーバー・CDN側の設定を責任者と検討するときに使う。

9. **OWASP：File Upload Cheat Sheet**【英語／ファイルアップロード】
   - [公開ガイド](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
   - 形式・容量・保存場所・実行権限等を扱う。添付フォームやアップロード機能を追加する案件で使う。

10. **OWASP：Application Security Verification Standard（ASVS）**【英語／検証標準】
    - [公開標準・配布資料](https://owasp.org/projects/asvs)
    - セキュリティ要件と検証項目を定める標準。責任者が案件に必要な項目を選ぶための資料とし、本書の確認だけでASVS準拠をうたわない。

資料の対象・更新時期は異なる。特定のプラグインや旧バージョンの設定をそのまま採用せず、使用中の製品の公式資料と照合する。以下の本文は当社向けに組み立てた運用案であり、公開資料の全文転載ではない。

## 目次

1. 基本原則・担当と判断範囲
2. テスト環境の非公開化と検索登録防止
3. アカウント・認証情報の管理
4. 公開ファイル・データ・転送の管理
5. HTML・バニラJavaScriptの安全な実装
6. フォーム・外部サービス・アップロード
7. WordPressの導入・更新・運用
8. WordPressのテーマ・独自機能の実装
9. HTTPS・HTTPヘッダー・キャッシュ
10. バックアップ・ログ・監視・引き継ぎ
11. 公開前後の確認
12. インシデントへの対応

## 第1章 基本原則・担当と判断範囲【共通】

### 必須ルール

- 制作開始時に、公開してよい情報、保護する情報、公開予定日、テスト環境と本番環境を確認する。
- 制作者、別担当の確認者、公開責任者、サーバー・保守担当、緊急連絡先を案件の管理資料に記録する。
- ブラウザーでボタンや画面を隠す処理を、アクセス制限として扱わない。データ取得・更新の可否はサーバー側で制御する。
- 新しい外部サービス、独自の認証・決済・ファイルアップロード処理は、実装前に技術責任者へ相談する。
- 必須項目の未確認・不合格がある状態では、テストURLの共有や本番公開を進めない。

**確認方法：** 案件の環境一覧と担当一覧を見て、接続先・公開範囲・判断する人を説明できること。項目が対象外の場合は理由を記録する。

参考：[IPA：安全なウェブサイトの作り方](https://www.ipa.go.jp/security/vuln/websecurity/about.html)

## 第2章 テスト環境の非公開化と検索登録防止【共通・最優先】

### 2.1 配置前にアクセス制限を設定する

**必須：** 「環境作成 → HTTPS・アクセス制限設定 → 外部からの確認 → 制作物の配置 → 再確認 → URL共有」の順で進める。制作物の配置後に設定する運用にしない。

- テスト環境はHTTPS上のBasic認証を標準とする。責任者が同等以上の保護として選定したVPN・IP制限・ホスティングの認証機能等も利用できる。
- トップページに加え、下層ページ、画像、PDF、サイトマップ、WordPressのuploads・REST API・フィード等、制作物を取得できる入口を保護する。
- WordPressの管理画面へのログイン制限や投稿単位のパスワード保護を、テストサイト全体の保護として代用しない。
- 別ホスト名、HTTP、CDN、オリジンサーバー等の入口を洗い出し、保護を迂回できない構成にする。確認は責任者が管理する範囲で行う。
- 認証を一時解除したまま作業しない。外部確認ツールが認証に対応しない場合は、保護を維持できる検証方法を責任者が選ぶ。

Basic認証の認証情報は暗号化されないため、HTTPSと組み合わせる。[MDN：HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication)

### 2.2 設定後は未認証で確認する

**合格条件：** 未認証・誤った認証情報では制作物を取得できず、許可された認証情報でのみ閲覧できる。Basic認証の場合、未認証の応答は原則として401と`WWW-Authenticate`を返し、本文に制作中の内容を含めない。

認証済みブラウザーは認証情報を再送する場合があるため、未認証確認には認証情報を付けないHTTPクライアント等を使う。Windowsでの確認例は以下。例示URLは実際のテストURLに置き換える。

```powershell
curl.exe -q --silent --show-error --dump-header - --output NUL "https://staging.example.com/"
```

この例はGET応答のヘッダー確認用。本文の非公開性は別途確認する。301・302等なら転送先も確認し、その応答だけで合格にしない。トップだけでなく、実在する下層・添付ファイル・API等も確認する。[MDN：HTTP認証の応答](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication)

### 2.3 noindexは補助策として使う

- テスト環境のHTMLには`noindex`を設定する。環境側で設定できる場合は、PDF等も対象にできる`X-Robots-Tag: noindex`を検討する。
- WordPressでは「検索エンジンがサイトをインデックスしないようにする」を有効にし、実際のHTML・ヘッダーを確認する。この設定は訪問者の閲覧を制限しない。[WordPress：表示設定](https://wordpress.org/documentation/article/settings-reading-screen/)
- `robots.txt`の`Disallow`だけで非公開・インデックス登録防止を実現しようとしない。Googleは`robots.txt`内の`noindex`指定をサポートしていない。
- 認証や`robots.txt`でクローラーを遮断している間、ページの`noindex`は読まれない。設定追加だけで既存の検索結果が直ちに消えるとは判断しない。
- 本番公開時は、本番の一般公開するページから不要な`noindex`を解除する。テスト環境の保護は維持する。

参考：[Google：noindexによる除外](https://developers.google.com/search/docs/crawling-indexing/block-indexing?hl=ja)、[Google：非公開コンテンツの保護](https://developers.google.com/search/docs/crawling-indexing/control-what-you-share?hl=ja)

**確認記録：** 確認日時、対象URL、認証の有無、応答、確認者を残す。環境作成時、デプロイ後、サーバー・CDN設定変更後に再確認する。

## 第3章 アカウント・認証情報の管理【共通】

### 必須ルール

- 案件・サービスごとに異なる、十分に長いパスワードを承認されたパスワード管理ツールで生成・保管する。使い回さない。
- 個別アカウントを原則とし、管理者権限は必要な人に限定する。共有アカウントが必要な場合は利用者を記録する。
- ホスティング、ドメイン、Git等の管理サービスは、対応している場合は多要素認証を有効にする。WordPressは承認した方式で導入・動作確認する。
- 認証情報をGit、HTML、JavaScript、URL、公開可能なドキュメントに書かない。スクリーンショットやログへの混入も確認する。
- APIキーは、公開可能なクライアント用識別子と秘密鍵を区別する。秘密鍵はサーバー側で管理し、公開用キーも提供元の制限機能を設定する。
- 退職・担当変更・案件終了時は不要な権限を失効させる。漏えいが疑われる認証情報は、削除だけで済ませず失効・再発行する。

**確認方法：** 利用者・権限の一覧、納品物、Gitの差分を別担当者が確認する。漏えいした鍵は失効済みで、新しい鍵への切り替えが確認できること。Git履歴の処理は責任者と行う。

参考：[OWASP：Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)、[WordPress：Hardening](https://developer.wordpress.org/advanced-administration/security/hardening/)

## 第4章 公開ファイル・データ・転送の管理【共通】

### 必須ルール

- 配置先に必要な成果物だけをアップロードする。`.git`、`.env`、DBダンプ、バックアップZIP、ログ、デザイン原本、不要な旧ファイルを公開領域に置かない。
- サーバーで必要な秘密設定は、ホスティングの安全な設定領域等で管理する。WordPressの`wp-config.php`や認証ファイルは外部取得されないことを確認する。
- 777等の広すぎるファイル権限で問題を解決しない。必要な権限はサーバーの実行ユーザー・所有者に合わせて責任者が決める。
- ファイル転送はSFTP等の暗号化された方法を使う。FTPしか使えない環境は責任者へ相談する。
- テストには架空・匿名化済みデータを使う。本番DBの複製が必要な場合は、個人情報・鍵・メール送信先等を確認し、利用者と保管期限を限定する。
- 公開するPDF・画像・ファイルの内容と、コメント・位置情報等の不要なメタデータを確認する。

**確認方法：** 配置ファイル一覧と公開対象一覧が一致すること。機密ファイルがHTTP経由で取得できず、不要なディレクトリ一覧も表示されないこと。ファイル名を変えるだけでは保護完了としない。

参考：[WordPress：ファイル権限・設定の保護](https://developer.wordpress.org/advanced-administration/security/hardening/)、[OWASP：Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)

## 第5章 HTML・バニラJavaScriptの安全な実装【共通】

### 必須ルール

- URLパラメーター、フォーム入力、API、CMS、ブラウザー保存領域のデータを無条件に信用しない。
- 文字列を表示する場合は`textContent`を使う。未検証データを`innerHTML`、`insertAdjacentHTML`、`document.write`に渡さない。
- 外部由来のデータを`eval`や`new Function`等でコードとして実行しない。
- 入力値からリンクを作る場合は、用途に応じたスキーム・ホスト等を許可リストで検証する。文字列を属性に入れるだけではURLの安全性は保証されない。
- HTMLを許可する必要がある場合は、許可する要素・属性と保守されているサニタイザーを責任者が選定する。独自の正規表現だけで除去しない。
- 秘密情報を`localStorage`等に保存しない。ブラウザーから見える値を、サーバーの認可判断に使わない。
- `postMessage`を使う場合は、送信先を限定し、受信側で送信元のoriginとデータ形式を検証する。

**実装例：検索語を文字列として表示する**

```javascript
const keyword = new URLSearchParams(location.search).get("q") ?? "";
const output = document.querySelector("[data-search-keyword]");

if (output) {
  output.textContent = keyword;
}
```

**確認方法：** 検証環境で`<em>test</em>`等を入力し、装飾されたHTMLとして解釈されず文字列として表示されること。これは一例であり、コードレビューでデータの取得元と出力先も確認する。

参考：[OWASP：XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)、[OWASP：Web Frontend Security](https://cheatsheetseries.owasp.org/cheatsheets/Web_Frontend_Security_Cheat_Sheet.html)、[OWASP：HTML5 Security](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)

## 第6章 フォーム・外部サービス・アップロード【使用する案件】

### 必須ルール

- フロントの入力チェックに加えて、送信先のサーバー側でも必須・型・長さ・許可値を検証する。静的HTMLだけではメール送信や保存処理は完結しないため、承認された送信先を使う。
- 送信先・保存先・外部連携先を本番とテストで分け、テスト送信が顧客や一般利用者へ届かないようにする。
- 認証を伴う更新処理はサーバー側で権限を確認し、CSRF対策を実装する。hidden属性の値を信用して権限を決めない。
- CAPTCHAを認証・権限確認の代わりにしない。公開フォームは必要に応じて送信回数制限等も組み合わせる。
- アップロードは、許可形式、内容、容量、保存名、保存先、操作権限をサーバー側で検証する。拡張子やブラウザー申告のMIMEだけで判定しない。
- アップロード領域でスクリプトが実行されない構成を責任者が確認する。SVG等の能動的な内容を含められる形式の許可は個別に判断する。
- 外部スクリプト・タグ・埋め込みは必要性、提供元、取得する情報を確認する。用途の分からないコードを貼り付けない。

**推奨：** 固定配布の外部スクリプトはバージョンを固定し、適用可能ならSRI（配布ファイルの改変検知）を使う。動的に更新されるタグは提供元の仕様に合わせて別途管理する。

**確認方法：** 正常送信だけでなく、未認証・権限不足・不正形式・過大容量等が拒否されること。検証時の送信先と保存先を確認する。

参考：[WordPress：入力値の検証](https://developer.wordpress.org/apis/security/data-validation/)、[OWASP：CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)、[OWASP：File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)、[OWASP：Third Party JavaScript](https://cheatsheetseries.owasp.org/cheatsheets/Third_Party_Javascript_Management_Cheat_Sheet.html)

## 第7章 WordPressの導入・更新・運用【WordPress】

### 必須ルール

- WordPress本体とテーマ・プラグインは公式または正規の配布元から入手する。不正に再配布された有料製品を使わない。
- WordPress・PHP・テーマ・プラグインのバージョンと更新担当を記録する。公開前に既知の脆弱性と対応状況を確認する。
- 使用しないプラグイン・テーマ・テストユーザーを整理する。復旧用に必要なものを除き、不要な製品を残さない。
- 更新時はバックアップと確認手順を用意する。緊急のセキュリティ更新は責任者が優先度・期限・一時遮断の要否を判断する。
- 投稿担当者に不要な管理者権限を与えない。管理画面のファイル編集は原則無効化する。
- テスト用メール設定やデバッグ表示を本番へ持ち込まない。エラー・ログから秘密情報が外部に見えないことを確認する。

**責任者判断：** 多要素認証、ログイン試行制限、WAF、管理画面の追加制限等を選定する。追加制限は、ログイン、ブロックエディター、フォーム、`admin-ajax.php`等の動作も確認する。セキュリティプラグインの導入だけで更新・権限管理を完了としない。

**確認方法：** バージョン・製品・利用者の一覧、更新結果、主要機能の動作結果が記録されていること。

参考：[WordPress：Hardening WordPress](https://developer.wordpress.org/advanced-administration/security/hardening/)

## 第8章 WordPressのテーマ・独自機能の実装【WordPress】

### 8.1 検証・サニタイズ・エスケープを分ける

- **検証：** 入力が仕様に合うか判定し、不正な値を拒否する。型・長さ・許可値等を確認する。
- **サニタイズ：** 用途に応じてデータを整える。`sanitize_text_field()`等を使うが、検証や出力時の処理の代用にしない。
- **エスケープ：** 表示直前に出力先に合わせて処理する。本文は`esc_html()`、属性は`esc_attr()`、URLは`esc_url()`を使う。許可されたHTMLを出す場合は`wp_kses()`等で範囲を限定する。

```php
<a href="<?php echo esc_url( home_url( '/' ) ); ?>">
    <?php echo esc_html( get_bloginfo( 'name' ) ); ?>
</a>
```

この例はHTMLへの出力処理の例。更新処理の入力検証・認可は別途必要となる。JavaScriptへのデータ受け渡しにはJSONの適切な生成・埋め込み方法を使い、HTML用エスケープだけで済ませない。

参考：[WordPress：Validating Data](https://developer.wordpress.org/apis/security/data-validation/)、[Sanitizing Data](https://developer.wordpress.org/apis/security/sanitizing/)、[Escaping Data](https://developer.wordpress.org/apis/security/escaping/)

### 8.2 nonceと権限確認を両方行う

**必須：** 独自の更新・削除処理は、処理前に適切な認証・権限確認とnonce等のCSRF対策を行う。WordPressのnonceは認証・認可の代用にはならず、一回限りのトークンでもない。必要な操作権限は`current_user_can()`等で別途確認する。[WordPress：Nonces](https://developer.wordpress.org/apis/security/nonces/)

- 独自REST APIは`permission_callback`で、操作・対象データに必要な権限を検証する。
- 公開用エンドポイントだけを明示的に公開し、未公開投稿・個人情報等を返さない。
- REST APIの一括遮断やログインURL変更だけで対策完了としない。必要な公開機能と保護対象を区別し、脆弱な実装・製品は修正・更新する。

参考：[WordPress：Adding Custom Endpoints](https://developer.wordpress.org/rest-api/extending-the-rest-api/adding-custom-endpoints/)

### 8.3 SQL・ファイル操作を安全に行う

**必須：** 原則としてWordPressの標準APIを使う。独自SQLが必要な場合は`$wpdb->prepare()`等で値をパラメーター化し、入力を文字列連結しない。列名・並び順等は許可リストで限定する。入力されたパスをそのままファイル読み込み・削除に使わない。

**確認方法：** コードレビューで入力元、対象データの権限、検証、出力先を追う。未認証・低権限・他ユーザーの対象ID等で許可外の取得・変更ができないことをテスト環境で確認する。

参考：[OWASP：SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)、[IPA：安全なウェブサイトの作り方](https://www.ipa.go.jp/security/vuln/websecurity/about.html)

## 第9章 HTTPS・HTTPヘッダー・キャッシュ【共通】

### 必須ルール

- 本番・テストともHTTPSを使用し、証明書の有効性・更新担当を確認する。HTTPからの転送が非公開情報を露出させないことも確認する。
- HTML・CSS・JavaScript等のContent-Typeを適切に返し、原則`X-Content-Type-Options: nosniff`を設定する。
- 非公開・個人情報を含む応答は共有キャッシュへ保存・配信しない。CDNの設定も含め、認証済み応答が未認証ユーザーへ届かないことを確認する。
- CORSは必要な送信元・用途だけを許可する。CORS設定をサーバー側の認証・認可の代用にしない。

### 責任者が案件別に設計する設定

- **CSP：** スクリプト・接続先等を制限する。初期導入はReport-Only等で影響を確認してから強制する。Report-Onlyは遮断しないため、強制適用とは区別して記録する。
- **クリックジャッキング対策：** CSPの`frame-ancestors`等で埋め込み元を制限する。外部での埋め込み要件を先に確認する。
- **Referrer-Policy：** 外部への参照元情報の送信範囲を決める。秘密情報をURLに入れない運用も継続する。
- **HSTS：** HTTPSを強制する設定。`includeSubDomains`やpreloadは、全対象のHTTPS運用・証明書を確認してから判断する。

**確認方法：** ブラウザー開発者ツール等で実際の応答ヘッダーを確認し、ページ表示・フォーム・外部連携・認証・埋め込みが設計どおり動くことを確認する。CSPやWAFは安全な実装を補う対策として扱う。

参考：[OWASP：HTTP Headers](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)、[OWASP：XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

## 第10章 バックアップ・ログ・監視・引き継ぎ【共通／保守】

### 必須ルール

- 公開・更新前に、復元に必要なファイル・DB・設定をバックアップする。ソースのGit管理だけでDBやメディアの保護を済ませない。
- バックアップは公開ディレクトリの外、かつ本番障害・侵害の影響を分離できる保管先に置き、利用者を限定する。
- 復元を隔離された環境で確認し、手順・所要時間を記録する。「バックアップ成功」の表示だけで復旧可能と判断しない。
- 保守契約で許容するデータ損失時間と復旧時間を決め、それに合わせて取得間隔・保存世代・担当を定める。
- 認証失敗、管理者追加、重要設定変更、更新失敗、バックアップ失敗等の確認・通知先を決める。
- ログにパスワード・秘密鍵・不要な個人情報を記録しない。ログの閲覧権限、保管期間、削除方法を決める。
- 納品時に更新担当・バックアップ担当・連絡先を引き継ぐ。不要になったテスト環境、アカウント、DNS・ホスティング設定は管理資料に沿って廃止する。

**確認方法：** バックアップ取得と復元の記録があり、通知が担当者へ届くこと。保守対象外の場合も、以後の担当と引き継ぎ完了を記録する。

参考：[WordPress：バックアップと監視](https://developer.wordpress.org/advanced-administration/security/hardening/)、[OWASP：Logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)

## 第11章 公開前後の確認【共通】

制作者と別担当の確認者がチェックする。各項目に確認者・日時・結果を付け、対象外は理由を残す。未解決の必須項目がある場合は公開責任者に報告する。

### テストURLの共有前

- [ ] HTTPSとアクセス制限を、制作物の配置前に設定した。
- [ ] 未認証・誤認証で拒否され、正しい認証情報でのみ閲覧できる。
- [ ] 下層・添付ファイル・API・フィード・別入口も保護されている。
- [ ] 認証済みキャッシュが未認証ユーザーへ配信されない。
- [ ] 補助策のnoindexとWordPress表示設定を実際の出力で確認した。
- [ ] 秘密ファイル・本番個人情報・誤送信するメール設定が混入していない。
- [ ] 共有先を限定し、別担当者の確認記録を残した。

### 本番公開前

- [ ] 公開するドメイン・サーバー・ディレクトリ・DBと対象成果物を確定した。
- [ ] 認証情報の混入、公開不要ファイル、機密ファイルの外部取得を確認した。
- [ ] 独自機能の入力検証・出力処理・権限確認・CSRF対策を確認した。
- [ ] WordPress・テーマ・プラグイン等の更新と脆弱性対応を確認した。
- [ ] 管理者権限、不要アカウント、デバッグ表示、メール送信先を確認した。
- [ ] HTTPS・ヘッダー・キャッシュの設定と主要機能を確認した。
- [ ] バックアップ、復元手順、切り戻し判断、連絡先を確認した。
- [ ] 別担当者が結果を確認し、公開責任者が公開可否を判断した。

### 本番公開直後

- [ ] 本番の公開対象を未認証で閲覧・操作できることを確認した。
- [ ] 本番の公開対象に不要なnoindex・クロール制限が残っていない。
- [ ] テスト環境のアクセス制限とnoindexが維持されている。
- [ ] CDNを含め、公開前のキャッシュや機密応答が誤配信されていない。
- [ ] フォーム、管理画面、通知、外部連携が正しい接続先で動作する。
- [ ] 公開日時、成果物の版、確認結果、保守担当を記録した。

## 第12章 インシデントへの対応【共通・最優先】

### 12.1 誤公開・検索登録を発見した場合

1. **直ちに社内報告する。** 対象URL、発見日時、見えている情報を責任者へ伝える。
2. **アクセスを遮断する。** テスト環境の認証・アクセス制限を設定・復旧し、未認証で内容を取得できないことを確認する。検索結果の削除を待って遮断を遅らせない。
3. **証跡と影響範囲を確認する。** 公開期間、ページ・添付・API、アクセスログ、キャッシュ、検索結果を記録する。証跡取得に時間がかかる場合も、露出の抑止を進める。
4. **検索結果の非表示を申請する。** 責任者が該当サイトのSearch Console所有権を確認し、削除ツールを使う。本番URLや別案件のURLを巻き込まないよう、対象範囲を別担当者と確認する。
5. **恒久対策を維持する。** 認証の継続や不要URLの削除等を行う。一時的な検索非表示だけで対応完了としない。noindexを読ませる目的で未公開情報を再公開しない。
6. **報告と経過確認を行う。** 顧客への報告は責任者が行い、検索結果・認証状態を継続確認する。他の検索エンジンはそれぞれ対応する。

Search Consoleの削除ツールはGoogle検索での一時的な非表示であり、サイトへのアクセスや他の検索エンジンを遮断するものではない。[Google：Removals tool](https://support.google.com/webmasters/answer/9689846?hl=en)

### 12.2 不正ログイン・改ざん・秘密情報漏えいが疑われる場合

1. 不審なユーザー・ファイル・通知等を発見したら、責任者へ報告し、発見時点の記録を残す。
2. 責任者が公開停止・隔離・アクセス制限の範囲を判断し、被害の拡大を抑える。
3. ログ・ファイル・DB等の証跡を保全する。不審なユーザーやファイルの削除だけで調査を終えない。
4. 漏えいが疑われるパスワード・鍵・セッション等を失効・更新する。復元した設定で古い鍵が再有効化されないよう確認する。
5. 侵入経路を調査し、脆弱な製品・設定・コードを修正する。信頼できる構成・データから復旧し、再侵入できないことを確認する。
6. 復旧後の動作と監視を確認し、責任者が再公開を判断する。調査が難しい場合はホスティング事業者・専門家へ依頼する。

参考：[WordPress：FAQ My site was hacked](https://wordpress.org/documentation/article/faq-my-site-was-hacked/)、[OWASP：Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)

**再発防止：** 個人の注意だけに依存せず、アクセス制限済みの環境テンプレート、配置時の未認証確認、秘密情報の混入検知、更新・バックアップの通知等へ反映する。

## 付録：新人が区別しておく用語

- **認証：** 利用者が誰であるか確認すること。
- **認可：** その利用者が操作・データ取得を許されているか判断すること。
- **クロール：** クローラーがURLへアクセスし情報を取得すること。
- **インデックス：** 検索エンジンが取得した情報を検索用に登録すること。
- **noindex：** 対応する検索エンジンに登録しないよう伝える指定。閲覧を遮断しない。
- **XSS：** 本来データである値がスクリプト等として解釈され、利用者のブラウザーで実行される問題。
- **CSRF：** 利用者の認証状態を悪用し、意図しない操作を送信させる攻撃。
- **nonce：** WordPress等でリクエストの確認に使う値。権限確認の代わりにはならない。
- **WAF：** 不正なHTTPリクエスト等を検査・遮断する仕組み。更新・安全な実装・認可を補う対策。

## 社内導入時に確定する項目

技術責任者は、利用ホスティング別の認証設定手順、承認する秘密情報管理ツール、管理アカウントの認証方式、更新期限、バックアップ取得・保存・復元の基準、監視・緊急連絡先、公開判定の記録先を決める。案件開始時には、その基準と実際の担当・契約範囲を照合する。
