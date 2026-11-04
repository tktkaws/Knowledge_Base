# WPの移行をプラグインを使用せずに行う方法 — ファイルとDBを手動で移す

## WordPressの移行とは

- WordPressの移行はサイトを動かすファイルとデータベースを別の環境へ移す作業である
- ファイルにはWordPress本体、テーマ、既存のプラグイン、アップロード画像、設定ファイルが含まれる
- データベースには投稿、固定ページ、ユーザー、コメント、各種設定などが保存される — ファイルだけをコピーしてもサイト全体は移行できない<sup><a href="#ref-1">[1]</a></sup>
- 対象は通常の単一サイト構成とする — マルチサイトや、WordPress本体だけを別ディレクトリに置く構成は追加の設定が必要になる
- 作業にはサーバーの管理画面、SFTP（暗号化されたファイル転送）、phpMyAdmin（ブラウザーで使うDB管理ツール）を使う
- URLを変更する場合は、WP-CLI（WordPressをコマンドで操作するツール）も使う — 移行用プラグインのインストールは不要である<sup><a href="#ref-2">[2]</a></sup>

参照

<ol start="1">
  <li id="ref-1"><a href="https://developer.wordpress.org/advanced-administration/security/backup/">WordPress Developer Resources — Backups</a></li>
  <li id="ref-2"><a href="https://make.wordpress.org/cli/handbook/guides/installing/">WP-CLI Handbook — Installing</a></li>
</ol>

## 1. 移行先とバックアップを準備する

- 移行先に対象ドメインを登録し、公開ディレクトリ、PHP・MySQLまたはMariaDBの互換性、空き容量、HTTPSの証明書を確認する
- 移行と同時にWordPressやテーマのバージョンを変えず、まず同じ構成で動作を確認する
- 移行先に専用の空のデータベースと接続用ユーザーを作り、そのDBに必要な権限を与える
- DB名、ユーザー名、パスワード、接続先ホスト名を控える — DBを別のサーバーへ移す場合も、移行先で使う接続情報が必要である
- ドメイン・`http`と`https`・サブディレクトリが変わるか確認する — URLが完全に同じなら、後述するURL置換は通常不要である
- ファイルとDBのバックアップを、移行作業用のコピーとは別に保管する — 移行完了まで旧サーバーも残す
- 最終コピーの前に、旧サイトへの投稿・画像追加・コメント・注文などの書き込みを止める — ホスティング側のアクセス制限などを使い、外部連携や定期処理による更新も止める
- 試しにコピーして検証した場合も、切り替え前に書き込みを止めて最新のファイルとDBを取り直す — コピー後の更新は移行先へ自動では反映されない<sup><a href="#ref-3">[3]</a></sup>

参照

<ol start="3">
  <li id="ref-3"><a href="https://developer.wordpress.org/advanced-administration/upgrade/migrating/#moving-to-a-new-server">WordPress Developer Resources — Moving to a New Server</a></li>
</ol>

## 2. 旧サーバーからファイルとDBを取得する

### ファイルのダウンロード

- SFTPでWordPressの設置ディレクトリを開き、配下のファイルとディレクトリをダウンロードする
- `wp-admin`、`wp-includes`、`wp-content`と、直下の`index.php`や`wp-config.php`などを取得する
- `.htaccess`のような隠しファイルも確認する — Webサーバーの設定や、公開ディレクトリの外にある独自ファイルは別途控える
- `wp-content/uploads`の画像と、利用中のテーマ・プラグインがそろっているか確認する

### phpMyAdminでDBをエクスポートする

- 旧サーバーのphpMyAdminを開き、`wp-config.php`の`DB_NAME`に指定されているDBを選択する
- 「エクスポート」でSQL形式を選び、対象サイトのテーブルの構造とデータを保存する
- WordPress専用のDBなら全テーブルを対象にする — 他のサイトとDBを共有している場合は、独自テーブルも含めて対象サイトが使う範囲を確認する
- 管理画面の「ツール → エクスポート」で取得する投稿用XMLは、DB全体のバックアップの代わりにはならない
- SQLファイルはユーザー情報などを含むため、Webから取得できる公開ディレクトリには置かない<sup><a href="#ref-4">[4]</a></sup>

### WP-CLIでDBを書き出す場合

- SSH（サーバーへの暗号化された接続）とWP-CLIが使える環境なら、コマンドでもSQLを書き出せる
- 以下はLinuxサーバー上の例である — パスは実際の設置先に置き換え、保存先は公開ディレクトリの外に作る
- `wp db export`には`mysqldump`が必要で、接続情報は`wp-config.php`から読み込まれる
- この例はWordPress専用DBを前提とし、DB内の全テーブルを書き出す<sup><a href="#ref-5">[5]</a></sup>

```bash
mkdir -p /home/account/wp-migration
cd /home/account/public_html
wp db export /home/account/wp-migration/site.sql
```

参照

<ol start="4">
  <li id="ref-4"><a href="https://developer.wordpress.org/advanced-administration/security/backup/database/">WordPress Developer Resources — Backing Up Your Database</a></li>
  <li id="ref-5"><a href="https://developer.wordpress.org/cli/commands/db/export/">WP-CLI — wp db export</a></li>
</ol>

## 3. 移行先へアップロードしてDBを復元する

### ファイルと接続設定

- 取得したWordPressのファイルを、移行先の公開ディレクトリへSFTPでアップロードする
- 既存サイトのファイルへ混ぜず、移行用に用意した場所へ配置する — WordPressの新規インストール操作は不要である
- 移行先の`wp-config.php`を編集し、DBの接続情報を移行先の値へ変更する
- 以下は既存の定義を書き換える箇所の例である — 同じ定数の定義を追加しない
- `$table_prefix`はインポートするテーブル名の接頭辞と一致させる — 元が`abc_`なら、理由なく`wp_`へ変更しない
- 独自の絶対パス、キャッシュ接続先、`WP_HOME`や`WP_SITEURL`の固定値がある場合も、移行先に合っているか確認する<sup><a href="#ref-6">[6]</a></sup>

```php
define('DB_NAME', 'new_database');
define('DB_USER', 'new_database_user');
define('DB_PASSWORD', 'replace_with_new_password');
define('DB_HOST', 'replace_with_provider_db_host');
```

### DBのインポート

- 移行先のphpMyAdminで、用意した空のDBを選択する
- 「インポート」から旧サイトのSQLファイルを指定し、処理が完了したことを確認する
- インポート先の選択を確認する — SQLの内容によっては同名テーブルが上書きされる
- 容量や実行時間の制限で失敗する場合は、ホスティングのDB復元機能やSSH経由のインポートを使う
- WP-CLIを使う場合は、SQLファイルを公開ディレクトリの外へ転送し、以下を移行先サーバーで実行する
- `wp db import`は接続先DBを作成しない — 空のDBを作ってユーザーに必要な権限を与え、MySQLクライアントも使える状態にする<sup><a href="#ref-7">[7]</a></sup>

```bash
cd /home/account/public_html
wp db import /home/account/wp-migration/site.sql
```

参照

<ol start="6">
  <li id="ref-6"><a href="https://developer.wordpress.org/advanced-administration/wordpress/wp-config/">WordPress Developer Resources — Editing wp-config.php</a></li>
  <li id="ref-7"><a href="https://developer.wordpress.org/cli/commands/db/import/">WP-CLI — wp db import</a></li>
</ol>

## 4. URLを調整して公開を切り替える

### URLが変わる場合の置換

- ドメイン、HTTPSの有無、設置パスが変わる場合は、移行先DB内の旧URLを新URLへ置換する
- WordPressの設定には、文字列の長さも記録するシリアライズ形式のデータがある — SQLファイルの単純な文字列置換やDB全体への`REPLACE()`は、このデータを壊す原因になる
- WP-CLIの`wp search-replace`はシリアライズされたデータを扱えるため、移行先で置換に使う<sup><a href="#ref-8">[8]</a></sup>
- WP-CLIが未導入なら公式手順で用意するか、ホスティング事業者に利用方法を確認する — phpMyAdminでの単純な一括置換を代用にしない
- 以下は`https://old.example.com`から`https://new.example.com`への移行例である — URLと実行場所を確認し、移行先DBの置換前バックアップを作る
- `--dry-run`で対象テーブルと件数を確認し、意図した範囲なら`--dry-run`を外して実行する
- `--all-tables-with-prefix`は同じ接頭辞の独自テーブルも対象にする — 対象外のテーブルや、処理されないテーブルが残っていないか実行結果を確認する
- `--skip-columns=guid`で投稿の識別子を置換対象から外す — `guid`に旧URLが残っていても、同じ投稿の識別子として維持する<sup><a href="#ref-9">[9]</a></sup>

```bash
wp db export /home/account/wp-migration/before-url-replace.sql

wp search-replace 'https://old.example.com' 'https://new.example.com' --all-tables-with-prefix --skip-columns=guid --dry-run

# dry-runの対象と件数を確認したあとに実行する
wp search-replace 'https://old.example.com' 'https://new.example.com' --all-tables-with-prefix --skip-columns=guid
```

- `http`や`www`の有無など旧URLの表記が複数ある場合は、残っている値を確認して個別に処理する
- `home`はサイトの公開URL、`siteurl`はWordPress本体のURLである — 通常のルート設置では両方が新URLになっているか確認する
- `WP_HOME`や`WP_SITEURL`が`wp-config.php`で定義されている場合は、DBとは別にその定義も更新する
- テーマや独自コードに直書きしたURL、ファイル内の絶対パスはDB置換の対象にならないため、別に確認する

```bash
wp option get home
wp option get siteurl
```

### 公開前の確認と切り替え

- URLを変えずサーバーだけを移す場合は、hosts設定などで手元の端末だけ移行先IPを参照させ、同じドメインで検証する — 新サーバーのHTTPS証明書も必要である
- 投稿、固定ページ、画像、管理画面へのログイン、フォーム送信、必要なら注文処理を確認する
- 管理画面の「設定 → パーマリンク設定」で設定を保存し、個別記事が表示されるか確認する
- Apacheでは`.htaccess`の内容と書き込み権限、Nginxではサーバー側の転送ルールを確認する — Nginxは`.htaccess`を読み込まない<sup><a href="#ref-10">[10]</a></sup>
- サーバーやCDNのキャッシュを削除し、CSS・画像の読み込み失敗、旧URLへの転送、HTTPSページ内のHTTP参照を確認する
- 検証後にDNS（ドメイン名と接続先を対応させる仕組み）の向き先を変更する — ドメインも変える場合は、旧URLから対応する新URLへの恒久転送も設定する
- DNSの切り替え中は旧サーバーへ接続する利用者もいるため、旧側の書き込み停止を維持し、移行先で更新を受け付ける
- 切り替え後も動作とエラーログを確認し、問題があれば切り替えを戻す — 移行先で受け付けた注文や投稿は旧DBへ自動では戻らない
- 確認用のhosts設定を戻し、不要になった転送用SQLファイルを削除する — 復旧用のバックアップは非公開の場所へ保管する

参照

<ol start="8">
  <li id="ref-8"><a href="https://developer.wordpress.org/cli/commands/search-replace/">WP-CLI — wp search-replace</a></li>
  <li id="ref-9"><a href="https://developer.wordpress.org/advanced-administration/upgrade/migrating/">WordPress Developer Resources — Migrating WordPress</a></li>
  <li id="ref-10"><a href="https://developer.wordpress.org/advanced-administration/server/web-server/nginx/">WordPress Developer Resources — Nginx</a></li>
</ol>

## まとめ

- WordPressの移行には、ファイルとデータベースの両方が必要である
- 移行前にバックアップを作り、最終コピー中は旧サイトへの書き込みを止める
- ファイルはSFTPで転送し、DBはSQL形式でエクスポート・インポートする
- 移行先の`wp-config.php`でDB接続情報とテーブルの接頭辞を確認する
- URLが変わる場合は、シリアライズ形式に対応したWP-CLIで置換する
- DNS切り替え前に表示やフォームを検証し、切り替え後も旧側の書き込みを止めておく
- 旧サーバーとバックアップを残し、切り戻す場合は移行先で増えたデータも確認する
