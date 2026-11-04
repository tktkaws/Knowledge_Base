# WPのバックアッププラグインの比較

## バックアッププラグインとは

- WordPressのバックアッププラグインは、サイトのデータを保存し、障害や操作ミスが起きたときに復元するためのプラグインである
- 通常のWordPressサイトの復元には、投稿・設定などを持つデータベースと、画像・テーマ・プラグインなどのファイルの両方が必要になる<sup><a href="#ref-1">[1]</a></sup>
- バックアップは同じサイトを以前の状態に戻すための保存、移行は別のサーバーやドメインへ移す作業である — 両方を扱える製品もある
- 比較対象はAll-in-One WP Migration、WPvivid、UpdraftPlus、Duplicator、Jetpack VaultPress Backup、BlogVaultの6製品とする
- 仕様の確認日は2026年10月28日である — 無料版で使える機能、容量制限、有料版で増える機能を公式情報で比較する

参照

<ol start="1">
  <li id="ref-1"><a href="https://developer.wordpress.org/advanced-administration/security/backup/">WordPress Developer Resources — Backups</a></li>
</ol>

## 容量制限の見方

- 「容量無制限」と書かれていても、作成・アップロード・保存のどの段階を指すかで意味が変わる
- プラグインに固定の上限がなくても、サーバーの空き容量、PHPのメモリ、実行時間などが原因で失敗することはある
- 比較するときは、次の制限を分けて確認する

| 比較項目 | 確認する内容 |
|---|---|
| バックアップ全体の容量 | サイト全体や1回のバックアップに、製品側の固定上限があるか |
| 復元時のアップロード容量 | ブラウザーから送るファイルが、PHP・Webサーバーなどの上限を超えないか |
| 分割ファイルの容量 | 大きなバックアップを何MBずつに分けるか — 総容量の上限とは異なる |
| クラウドの保存容量 | 保存先の契約容量に、複数世代のバックアップが収まるか |
| 保存期間・世代数 | 何日前まで、または何回分まで戻せるか |

- PHPの `upload_max_filesize` は1ファイルのアップロード上限、`post_max_size` は1回のPOST全体の上限である — All-in-One WP Migrationの表示値はこれらの小さい方が基準になる<sup><a href="#ref-2">[2]</a></sup>
- たとえば毎回3GBの完全バックアップを7世代残すなら、単純計算で21GBを使う — 増分保存や圧縮などを考慮しない例である
- 増分バックアップは前回から変更されたデータを保存する方式である — 保存量を抑えられるが、必要な容量は変更量と保存期間によって変わる
- 「無料でクラウドへ転送できる」は「保存先の容量や利用料金も無料」という意味ではない

参照

<ol start="2">
  <li id="ref-2"><a href="https://blog.servmask.com/wordpress-upload-limit/">ServMask — Why All-in-One WP Migration Shows an Upload Limit</a></li>
</ol>

## 無料で使える4製品の比較

- 無料版では容量だけでなく、自動実行、復元、外部保存まで使えるかを確認する
- 表のクラウド連携は接続機能の有無を示す — 保存先の契約や使用容量は別に確認する

| 製品 | バックアップ容量の制限 | 定期バックアップ | 無料での復元 | 外部保存 |
|---|---|---|---|---|
| All-in-One WP Migration | エクスポートの総容量に固定上限なし。インポートはホストのアップロード制限を受ける | 基本の無料版にはなし | ファイルをダウンロードして「インポート」から復元 | 基本はPCへのダウンロード。クラウド連携は有料拡張 |
| WPvivid Backup & Migration | 公式説明に一律の総容量上限の記載なし。分割設定・サーバー環境の影響を受ける | 対応 | 管理画面から復元 | Google Drive、Dropbox、OneDrive、Amazon S3、FTP・SFTPなど |
| UpdraftPlus | 総容量の固定上限なし。標準の400MBは分割サイズであり、サイト全体の上限ではない | 対応 | 管理画面から復元 | Google Drive、Dropbox、Amazon S3、FTPなど |
| Duplicator Lite | 5.0以降は製品側の固定容量上限なし。サーバー環境の影響は残る | Proで対応 | 管理画面からのワンクリック復元に対応 | Duplicator Cloudと連携。ストレージ契約は別途必要 |

### All-in-One WP Migration

- サイトを `.wpress` ファイルに書き出し、別のWordPressへ読み込む操作が中心である
- 現在の公式説明では、無料版に固定の512MBインポート上限はない — 表示されるアップロード上限はホスト側の設定による<sup><a href="#ref-3">[3]</a></sup>
- 無料で復元する場合は、保存したファイルを「インポート」から読み込む — 「バックアップ」一覧のワンクリック復元はUnlimited Extensionの機能である
- ファイルを作成できても、復元先へアップロードできるとは限らない — 利用するサーバーの上限まで確認する

### WPvivid Backup & Migration

- 無料版で手動・定期バックアップ、復元、サイト移行、外部ストレージへの保存を扱える<sup><a href="#ref-4">[4]</a></sup>
- 無料でOneDriveやSFTPを使いたい場合も比較候補になる
- 容量は一律に「何GBまで」と判断せず、分割サイズと大きなファイルの除外設定も確認する — 除外条件に合うファイルはバックアップに入らない<sup><a href="#ref-5">[5]</a></sup>
- 無料版にもサーバーの実行時間・メモリなどの制約は残る — 容量上限の記載がないことは、どの環境でも復元できる保証ではない

### UpdraftPlus

- 無料版でバックアップの定期実行、Google Driveなどへの外部保存、復元に対応する<sup><a href="#ref-6">[6]</a></sup>
- 大きなバックアップは標準で400MBごとに分割される — 400MBを超えるサイトをバックアップできないわけではない<sup><a href="#ref-7">[7]</a></sup>
- 現在は無料版でも、バックアップを移行先へアップロードして復元する移行に対応する — サイト間の直接転送などはPremiumで機能が増える
- OneDriveやSFTPへの保存は有料版の機能であり、WPvividの無料版とは対応範囲が異なる

### Duplicator Lite

- 公式案内では、5.0以降の無料版は有料版と同じバックアップエンジンを使い、容量上限を設けていない<sup><a href="#ref-8">[8]</a></sup>
- 現行の無料版はワンクリック復元やDuplicator Cloud連携にも対応する — 古い比較記事の「復元やクラウド連携はすべて有料」をそのまま当てはめない
- 旧版向けの資料に残る「DupArchiveは500MBまで」という説明も、現行版の条件と区別する
- 定期実行やGoogle Drive・Amazon S3などの外部ストレージ連携はProの比較項目になる

参照

<ol start="3">
  <li id="ref-3"><a href="https://wordpress.org/plugins/all-in-one-wp-migration/">WordPress.org — All-in-One WP Migration and Backup</a></li>
  <li id="ref-4"><a href="https://wordpress.org/plugins/wpvivid-backuprestore/">WordPress.org — WPvivid — Backup, Migration &amp; Staging</a></li>
  <li id="ref-5"><a href="https://docs.wpvivid.com/advanced-settings-free.html">WPvivid Documentation — Advanced Settings (Free)</a></li>
  <li id="ref-6"><a href="https://wordpress.org/plugins/updraftplus/">WordPress.org — UpdraftPlus: WP Backup &amp; Migration Plugin</a></li>
  <li id="ref-7"><a href="https://teamupdraft.com/docs/updraftplus/backing-up/explanations/backup-limits-explained/">TeamUpdraft Documentation — Are there any backup limits?</a></li>
  <li id="ref-8"><a href="https://duplicator.com/duplicator-review/">Duplicator — 無料版の機能・5.0以降の容量制限に関する公式説明</a></li>
</ol>

## 有料版・有料サービスの比較

- 有料製品はバックアップを自分の保存先へ送るプラグインと、クラウド保存を含むサービスを分けて比較する
- All-in-One WP MigrationのUnlimited Extension、WPvivid Pro、UpdraftPlus Premium、Duplicator Proは既存プラグインの機能を増やす選択肢である
- Jetpack VaultPress BackupとBlogVaultは、契約にクラウド保存を含むサービスである — バックアップ機能を使う契約と、プラグイン自体の無料インストールを混同しない

| 製品・サービス | 容量制限・保存容量 | 主な有料機能・確認点 |
|---|---|---|
| All-in-One WP Migration Unlimited Extension | ホストのアップロード制限を回避する転送に対応。保存先の空き容量は別途必要 | バックアップ一覧からの復元、サーバー上のファイルからのインポート。クラウド連携・定期実行は必要な拡張を別途確認 |
| WPvivid Backup & Migration Pro | 無料版の容量上限を解除することが主な違いではない。保存先とサーバーの条件を確認 | 増分バックアップ、詳細なスケジュール、除外・対象指定、更新前の自動バックアップなど |
| UpdraftPlus Premium | バックアップ全体の固定上限なし。UpdraftVaultの付属容量や追加契約はプランを確認 | 増分バックアップ、複数保存先、時刻指定、更新前の自動バックアップ、OneDrive・SFTPなど |
| Duplicator Pro | 無料版と同様に製品側の固定容量上限なし。Duplicator Cloudの容量は別契約 | 定期実行、Google Drive・Amazon S3などへの連携、移行機能の拡張。保存先数・サイト数はプランで異なる |
| Jetpack VaultPress Backup | 基本のバックアップ保存容量は10GBから。追加ストレージあり | リアルタイムのクラウドバックアップと復元。保存容量と保存期間を組み合わせて確認 |
| BlogVault | 対象サイト容量はPersonalが20GB、Businessが50GB、WooCommerceが100GBまで | 保存期間は順に30日・90日・365日。サイト容量の上限であり、Jetpackの総保存容量とは比較する対象が異なる |

- Unlimited Extensionは、無料版の「固定512MB制限を外す製品」と説明するより、ホストの制限に対応した転送と復元機能を追加する製品として捉える — 現行ライセンスには利用サイト数の条件もある<sup><a href="#ref-9">[9]</a></sup>
- WPvivid Proは増分バックアップや詳細なスケジュールなどが主な違いであり、無料版で定期実行と外部保存が足りるなら、容量だけを理由に有料化する必要はない<sup><a href="#ref-10">[10]</a></sup>
- UpdraftPlus Premiumも、容量上限の解除より、保存先・自動実行・復元対象などの機能差で判断する<sup><a href="#ref-11">[11]</a></sup>
- Duplicator Proのライセンス料と、Duplicator Cloudのストレージ料金は分けて確認する — 外部ストレージを使う場合も、その契約容量と料金が必要になる<sup><a href="#ref-12">[12]</a></sup>
- Jetpackの10GBは「10GBのサイトを何世代でも保存できる」という意味ではない — 複数世代の保存で容量を消費し、上限に達すると古い世代の削除や、新規バックアップの停止が起こり得る<sup><a href="#ref-13">[13]</a></sup>
- BlogVaultは対象サイトの容量と保存期間をプラン別に示している — 100GBを超えるサイトは個別相談の対象である<sup><a href="#ref-14">[14]</a></sup>
- 費用は初年度割引だけでなく、更新料金、利用サイト数、外部ストレージ代まで含めて比較する

参照

<ol start="9">
  <li id="ref-9"><a href="https://servmask.com/products/unlimited-extension">ServMask — Unlimited Extension</a></li>
  <li id="ref-10"><a href="https://wpvivid.com/pricing/join">WPvivid — 無料版・Proの機能とプラン</a></li>
  <li id="ref-11"><a href="https://teamupdraft.com/updraftplus/free-vs-premium/">TeamUpdraft — Free vs Premium UpdraftPlus</a></li>
  <li id="ref-12"><a href="https://duplicator.com/pricing/">Duplicator — ProとDuplicator Cloudのプラン</a></li>
  <li id="ref-13"><a href="https://jetpack.com/support/backup/jetpack-vaultpress-backup-storage-and-retention/">Jetpack — VaultPress Backup: Storage and Retention</a></li>
  <li id="ref-14"><a href="https://blogvault.net/pricing/">BlogVault — プラン別のサイト容量と保存期間</a></li>
</ol>

## 用途に合わせた選び方と復元確認

- 無料で日常の定期バックアップと外部保存を始めるなら、WPvividとUpdraftPlusを候補にする — 利用中の保存先に無料版で接続できるかで選ぶ
- `.wpress` ファイルで手動の書き出し・読み込みを行いたいなら、All-in-One WP Migrationを候補にする — 復元先のアップロード上限を先に確認する
- バックアップとサイト移行をまとめて扱い、後から定期実行や保存先を増やすなら、Duplicator LiteとProを比較する
- クラウド保存を含む契約でリアルタイムのバックアップを使うならJetpack、サイト容量と保存期間でプランを選ぶならBlogVaultを候補にする — BlogVaultでリアルタイム保存が必要な場合はWooCommerceプランの条件を確認する
- 採用前に、実際のサイトと同程度のデータでバックアップと復元を試す — ファイルが作成できたことだけで判断しない
- 復元テストは本番サイトとは別の環境で行い、ログイン、投稿、画像、フォームなどが戻るかを確認する
- 管理画面に入れなくなった場合に備え、プラグインの再インストール、外部管理画面、復旧用の機能など、製品ごとの復元経路も確認する
- バックアップは本番サーバー以外にも保存する — 同じサーバーにしかコピーがないと、サーバー障害と同時に失う可能性がある<sup><a href="#ref-15">[15]</a></sup>

参照

<ol start="15">
  <li id="ref-15"><a href="https://developer.wordpress.org/advanced-administration/security/backup/">WordPress Developer Resources — バックアップの保存と復元</a></li>
</ol>

## まとめ

- 容量制限はバックアップ全体、アップロード、分割サイズ、クラウド保存容量を分けて比較する
- All-in-One WP Migrationの無料版を固定512MB上限とする説明は、現在の公式仕様に合わない
- WPvividとUpdraftPlusは、無料で定期バックアップ・復元・外部保存を始める候補になる
- UpdraftPlusの標準400MBは分割サイズであり、サイト全体の容量上限ではない
- Duplicatorは5.0以降の無料版に固定容量上限がなく、古いバージョンの500MB制限と区別する
- 有料版は増分保存、自動実行、対応する保存先、クラウドの容量・保存期間などで選ぶ
- 導入前に、実データに近い条件で復元できることと、本番サーバー以外にも保存されることを確認する
