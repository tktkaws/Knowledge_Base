# 2026年10月時点で安くて堅牢なCMSとは — WordPress・Drupal・Contentfulなどを比較する

- PHPの脆弱性対応が大変だったことをきっかけに、WordPress以外のCMSも比較してみることにした
- 比較するのはWordPress、Drupal、Contentful、microCMS、Movable Type、EmDash — 製品の知名度だけでなく、費用と自分で管理する範囲から選びたい
- 今回の「安い」は、利用料金に加え、構築・更新確認・障害復旧にかかる作業時間も含めたものとする
- 「堅牢」は、攻撃される入口を減らせること、更新を続けられること、問題が起きても復旧できることの3点で考える
- 以下の向き不向きは、公式の仕様と運用構成からの判断である — 脆弱性の件数だけで製品の安全性を順位付けするものではない
- 情報の確認日は2026年10月23日 — 料金は通貨・税・支払条件が異なるため、表示金額だけで単純比較しない

## WordPress — 導入しやすいが、継続的な更新対応が必要

- 費用：WordPress.orgで配布される本体は無償で利用できるが、サーバー・独自ドメイン・有料テーマ・有料プラグイン・保守費は別に必要になる<sup><a href="#ref-1">[1]</a></sup>
- 強み：記事編集からサイト表示までを一体で用意しやすく、テーマやプラグインで必要な機能を追加できる
- 保守：自分で設置する場合、本体・プラグイン・テーマに加え、PHPやデータベースなどの実行環境も管理対象になる — ホスティング事業者との分担は契約によって異なる
- セキュリティ：公式ガイドでも更新・アクセス制御・バックアップなどを組み合わせた対策が案内されている — 導入後も運用が続く前提で選ぶ<sup><a href="#ref-2">[2]</a></sup>
- PHPの問題：2026年10月21日の重大なPHP脆弱性についても、WordPress公式はWordPress固有の問題ではなく、PHP側の更新が必要と案内している<sup><a href="#ref-3">[3]</a></sup>
- 向く用途：既存のWordPress資産を活用したいサイト、編集担当者が操作に慣れているサイト、プラグインの機能を多く使うサイト
- 今回の評価：初期費用を抑えやすい一方、保守の時間を費用へ含めると、常に最安とは限らない
- 運用を減らしたい場合：マネージドWordPressも比較対象になる — PHP更新、プラグイン更新、互換性確認、復旧作業をどこまで任せられるかを確認する

<ol start="1">
  <li id="ref-1"><a href="https://wordpress.org/about/license/">WordPress — License</a></li>
  <li id="ref-2"><a href="https://developer.wordpress.org/advanced-administration/security/hardening/">WordPress — Hardening WordPress</a></li>
  <li id="ref-3"><a href="https://wordpress.org/news/2026/10/critical-php-security-release/">WordPress — Critical PHP Security Release</a></li>
</ol>

## Drupal — 複雑な編集運用に向くが、PHPの管理は残る

- 費用：Drupal本体は無償で利用できるオープンソースであり、サーバー・開発・保守の費用は別に必要になる<sup><a href="#ref-4">[4]</a></sup>
- 強み：記事の公開状態や承認手順を設定するContent ModerationとWorkflows、多言語管理などの機能が用意されている — 複数の担当者や言語を扱う運用で比較したい<sup><a href="#ref-5">[5]</a></sup><sup><a href="#ref-6">[6]</a></sup>
- 保守：PHPとデータベースを使い、Composerによる依存関係の管理も必要になる — WordPressから移っても、PHPの脆弱性対応そのものはなくならない<sup><a href="#ref-7">[7]</a></sup>
- セキュリティ：本体と追加モジュールのセキュリティ勧告が公式に公開され、リリース予定とサポート対象を確認できる — 対応する担当者と更新手順が必要である<sup><a href="#ref-8">[8]</a></sup><sup><a href="#ref-9">[9]</a></sup>
- 向く用途：承認フロー、多言語、複雑なコンテンツ構造を扱い、継続して開発・保守できる組織のサイト
- WordPressとの比較：編集ルールや権限の設計を重視する場合に検討する価値があるが、単純なブログでは設計・学習・検証の作業が割高になりやすい
- 今回の評価：要件に合えば有力だが、「PHP対応が大変だったので保守を減らしたい」という理由だけで選ぶ移行先にはしにくい
- 注意点：セキュリティへの取り組みがあることと、導入したサイトを放置できることは別である — 採用時には利用する本体・モジュールのサポート状況も確認する

<ol start="4">
  <li id="ref-4"><a href="https://www.drupal.org/about/licensing">Drupal — Licensing</a></li>
  <li id="ref-5"><a href="https://www.drupal.org/docs/8/core/modules/content-moderation/overview">Drupal — Content moderation overview</a></li>
  <li id="ref-6"><a href="https://www.drupal.org/docs/administering-a-drupal-site/multilingual-guide">Drupal — Multilingual guide</a></li>
  <li id="ref-7"><a href="https://www.drupal.org/docs/getting-started/system-requirements">Drupal — System requirements</a></li>
  <li id="ref-8"><a href="https://www.drupal.org/security">Drupal — Security advisories</a></li>
  <li id="ref-9"><a href="https://www.drupal.org/about/core/policies/core-release-cycles/schedule">Drupal — Core release schedule</a></li>
</ol>

## ContentfulとmicroCMS — CMSの基盤運用をサービスへ任せる

### Contentful — 小規模用途では料金と利用条件に注意

- 構成：コンテンツの管理と公開画面を分離し、API経由でデータを利用するサービスである — CMS側のサーバーを自分で設置せずに使える
- 費用：Freeは0米ドル、Liteは月額300米ドル、Enterpriseは個別見積もり — 公開画面の開発・配信費用や追加のSpaceなどは別に考える<sup><a href="#ref-10">[10]</a></sup>
- 無料枠：Freeには10ユーザー、2ロケール、月10万回のAPI呼び出し、月50GBの配信帯域が含まれる<sup><a href="#ref-10">[10]</a></sup>
- 利用条件：確認時点の公式Usage limitsでは、Freeはテスト・学習用途に限られ、商用用途は不可とされている — 無料枠の数値に収まるだけでは採用を判断できない<sup><a href="#ref-11">[11]</a></sup>
- 強み：コンテンツを構造化して扱え、Liteにはコメント・タスク管理・予約公開・共同編集などが含まれる — 多言語や複数の配信先を含む要件で比較したい<sup><a href="#ref-10">[10]</a></sup>
- 保守：CMS基盤の運用はサービス側に任せられるが、公開画面、API連携、アクセス権、トークン管理、デプロイの保守は利用者側に残る
- WordPress・Drupalとの比較：自分でCMSのPHP環境を維持する作業を減らせる一方、テーマを入れるだけで公開サイトが完成する構成ではない
- 今回の評価：チームでのコンテンツ運用を重視するなら候補になるが、小さな商用ブログを安く運営したい場合は、Liteの月額料金が負担になりやすい

<ol start="10">
  <li id="ref-10"><a href="https://www.contentful.com/pricing/">Contentful — Pricing</a></li>
  <li id="ref-11"><a href="https://www.contentful.com/help/admin/usage/usage-limit/">Contentful — Usage limits</a></li>
</ol>

### microCMS — 日本語の編集環境と小規模な構成で比較したい

- 構成：Contentfulと同様に、管理画面とAPIをサービスとして使い、公開画面を別に作る構成である
- 費用：Hobbyは月額0円、Teamは税抜き月額4,900円から — 公開サイトのホスティング費と独自ドメイン代は別になる<sup><a href="#ref-12">[12]</a></sup>
- 無料枠：Hobbyは最大3人のメンバー、5個のAPI、月20GBのデータ転送量 — API呼び出し回数は無制限でも、転送量を超えるとAPIが停止する<sup><a href="#ref-12">[12]</a></sup>
- 強み：日本語の管理画面を使いたい場合に検討しやすく、小規模な構成から始められる
- 保守：CMS基盤を自分で管理する必要はないが、公開画面の実装、下書きプレビュー、更新時のデプロイ、APIキーの権限管理は設計する必要がある
- Contentfulとの比較：確認時点では、有料プランの入口はmicroCMSのTeamのほうが低額である — ただし、編集人数、権限、承認手順、転送量など、必要な機能が含まれるプラン同士で比較する
- WordPressとの比較：プラグインで公開側の機能を追加する方法から、自分で画面や外部サービスとの連携を実装する方法へ変わるため、制作費まで含めて判断する
- 今回の評価：公開画面を開発できる担当者がいて、日本語で記事を編集してもらいたい小規模サイトなら候補になる

<ol start="12">
  <li id="ref-12"><a href="https://microcms.io/pricing">microCMS — 料金プラン</a></li>
</ol>

## Movable Type — 静的生成と有償サポートを重視する場合

- 構成：Movable Type（ムーバブル・タイプ）は、記事を編集して静的HTMLを生成できるCMSである — ソフトウェア版とクラウド版があり、管理する範囲と費用が異なる<sup><a href="#ref-13">[13]</a></sup><sup><a href="#ref-14">[14]</a></sup>
- 費用：ソフトウェア版は税込132,000円で初年度メンテナンスが付属し、継続用の年間メンテナンスは税込44,000円 — サーバーと構築・保守の費用は別になる<sup><a href="#ref-14">[14]</a></sup>
- クラウド版：Movable Type 9の案内は税込月額13,750円からで、ストレージ料金が別途加算される — メンテナンスとしてテクニカルサポートと最新バージョンの提供が含まれる<sup><a href="#ref-14">[14]</a></sup>
- ライセンス：個人無償ライセンスもあるが、個人所有のサーバーで個人的な日記などを書くためのもの — 法人サイトへ無条件に流用できる無料版とは考えない<sup><a href="#ref-14">[14]</a></sup>
- PHPとの関係：本体はPerlで動き、ダイナミックパブリッシングを利用しなければPHPのインストールは不要である — ソフトウェア版ではPerlやデータベースなどの実行環境の管理が残る<sup><a href="#ref-15">[15]</a></sup>
- 堅牢さ：静的HTMLの閲覧では毎回CMSを実行せずに済み、CMSと公開環境を分離する構成も検討できる — 管理画面、検索、フォームなどの動的処理は別に保護する必要がある
- 保守：クラウド版には同じメジャーバージョン内の自動アップデート機能があるが、利用するプラグインなどの動作検証は必要になる<sup><a href="#ref-16">[16]</a></sup>
- 今回の評価：最小の利用料を目指す個人ブログより、静的生成とメーカーのサポートを重視し、予算を確保できる企業サイトで比較したい — PHPを使わない場合でも、本体の更新と復旧の備えは必要である

<ol start="13">
  <li id="ref-13"><a href="https://www.sixapart.jp/movabletype/">Six Apart — Movable Type</a></li>
  <li id="ref-14"><a href="https://www.sixapart.jp/movabletype/license/">Six Apart — Movable Type ライセンスのご案内</a></li>
  <li id="ref-15"><a href="https://www.movabletype.jp/documentation/mt9/start-guide/about-movable-type/system-requirements/">Movable Type 9 — 動作に必要な環境とブラウザの対応</a></li>
  <li id="ref-16"><a href="https://www.movabletype.jp/documentation/cloud/guide/cfg-auto-update.html">Movable Type クラウド版 — 自動アップデート機能</a></li>
</ol>

## EmDash — PHPを使わず、プラグインの権限を制限する構成

- 構成：EmDash（エムダッシュ）はCloudflareが開発する、Astroを基盤にしたオープンソースのCMSである — 2026年9月28日に1.0が正式公開された<sup><a href="#ref-17">[17]</a></sup>
- 費用：CMS本体は無償だが、稼働先のインフラ料金は別になる — Cloudflareへ配置する場合はWorkersやデータベース、画像などのストレージの利用量を確認する<sup><a href="#ref-18">[18]</a></sup>
- 料金の目安：Workers Paidは最低月額5米ドルで、使用量に応じた追加料金がある — これはEmDash全体の定額料金ではなく、必要なサービスを合算して考える<sup><a href="#ref-19">[19]</a></sup>
- 強み：管理画面と公開サイトをAstroの構成で扱え、PHP環境を維持せずにサイトを運用できる — Cloudflare Workersのほか、Node.jsへの配置にも対応する<sup><a href="#ref-20">[20]</a></sup>
- セキュリティ：サンドボックス型プラグインは隔離された実行環境で動き、宣言された権限に基づくAPI利用とリソース制限が適用される — 拡張機能が使える範囲を絞る設計である<sup><a href="#ref-21">[21]</a></sup>
- 制限：ネイティブ型プラグインはAstroサイトと同じプロセスで動き、同じ隔離境界は持たない — すべてのプラグインが自動的に隔離されるとは考えない<sup><a href="#ref-21">[21]</a></sup>
- 保守：Cloudflareの実行基盤を使っても、EmDash本体・テーマ・プラグインの更新、設定、認証情報、データのバックアップと復元確認は運用へ組み込む必要がある
- 他の候補との比較：ContentfulやmicroCMSのように完成したCMS基盤を契約して使う方法と比べ、アプリケーションの配置や更新を自分で扱う範囲が大きい — WordPressのテーマやプラグインも、そのまま使える前提にはしない
- 今回の評価：AstroやCloudflareを扱える担当者がいて、PHPの管理を減らしたい場合に検証する価値がある — 正式版公開から日が浅いため、必要な編集機能、移行、拡張機能、復旧手順を小さなサイトで確認してから採用したい
- 選び方：既存資産を使うならWordPress、複雑な編集運用ならDrupal、CMS基盤を任せるならContentfulやmicroCMS、静的生成と有償サポートならMovable Type、AstroとCloudflareを基盤に自分で運用するならEmDashが比較候補になる
- 移行費用：記事だけでなく、画像、既存URL、リダイレクト、検索、フォーム、編集担当者への説明も含め、既存WordPressの保守を外部へ任せる場合と比較する

<ol start="17">
  <li id="ref-17"><a href="https://blog.cloudflare.com/emdash-cms-plugin-registry/">Cloudflare — EmDash 1.0: the stable CMS with a secure plugin registry</a></li>
  <li id="ref-18"><a href="https://docs.emdashcms.com/deployment/cloudflare/">EmDash — Deploy to Cloudflare</a></li>
  <li id="ref-19"><a href="https://developers.cloudflare.com/workers/platform/pricing/">Cloudflare Workers — Pricing</a></li>
  <li id="ref-20"><a href="https://docs.emdashcms.com/getting-started/">EmDash — Create your first EmDash site</a></li>
  <li id="ref-21"><a href="https://docs.emdashcms.com/plugins/creating-plugins/choosing-a-format/">EmDash — Choosing a plugin format</a></li>
</ol>

## まとめ

- WordPressは既存資産やプラグインを活用しやすいが、PHPを含む更新と復旧の運用が必要になる
- Drupalは承認フローや多言語管理を含む要件で候補になるが、PHPの保守を減らす目的だけでは選びにくい
- ContentfulはCMS基盤を任せられる一方、無料プランの用途制限と月額300米ドルのLiteの費用を確認したい
- microCMSは日本語で編集する小規模サイトの候補であり、料金だけでなく人数・権限・転送量の条件で比較する
- Movable Typeは静的生成と有償サポートを重視する場合の候補であり、ソフトウェア版とクラウド版で費用と保守範囲が異なる
- EmDashはPHPを使わない構成とサンドボックス型プラグインが特徴だが、インフラ料金とアプリケーションの保守を考える必要がある
- EmDashは正式版公開から日が浅いため、必要な機能・移行・復旧を確認してから採用を判断する
- 安くて堅牢な構成は、月額料金だけでは決まらない — 移行費用と保守時間を含め、更新と復旧を続けられるものを選ぶ
