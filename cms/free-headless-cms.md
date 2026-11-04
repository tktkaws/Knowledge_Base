# 無料枠のあるヘッドレスCMS — Astroで使う候補と繰り返しフィールドの対応

- ヘッドレスCMSは、記事などの管理と公開画面を分け、APIでコンテンツを取得するCMSである
- 今回はmicroCMS、Contentful、Sanity、Hygraph、Storyblokの5つを紹介する — 期限付きのトライアルと、継続して利用できる無料プランは区別する
- 比較する機能はACF（Advanced Custom Fields）のRepeaterに相当する繰り返し入力である — 「質問と回答」のような複数の項目を1組にして、行を追加・並べ替えできるかを確認する<sup><a href="#ref-1">[1]</a></sup>
- 公開環境は、Astroで静的生成し、ソースをGitHubで管理し、GitHub ActionsからレンタルサーバーへFTPでデプロイする構成を想定する — CMSのデータはビルド時に取得し、生成した`dist/`を転送する<sup><a href="#ref-2">[2]</a></sup>
- CMSで記事を更新してもGitHubへのpushは発生しない — 自動反映にはCMSのWebhookを受けて認証付きでGitHub Actionsを起動する中継処理などが必要であり、小規模なら手動実行や定期実行も候補になる<sup><a href="#ref-3">[3]</a></sup>
- CMSの無料枠には、レンタルサーバー、ドメイン、メール、フォーム、GitHub Actionsの利用料は含まれない — FTP接続に使う認証情報はSecretsで管理し、サーバーが対応する場合はFTPSなどの暗号化された転送を使う
- 情報の確認日は2026年10月26日 — 無料枠の機能と用途制限を確認し、API・転送量・編集人数の上限を超える場合の費用も調べる

<ol start="1">
  <li id="ref-1"><a href="https://www.advancedcustomfields.com/resources/repeater/">ACF — Repeater</a></li>
  <li id="ref-2"><a href="https://docs.astro.build/en/guides/on-demand-rendering/">Astro — 静的生成とオンデマンドレンダリング</a></li>
  <li id="ref-3"><a href="https://docs.github.com/en/rest/actions/workflows#create-a-workflow-dispatch-event">GitHub — ワークフローをAPIで起動する</a></li>
</ol>

## microCMS — 日本語の管理画面で繰り返し項目を作る

- 日本語の管理画面で記事を編集し、REST APIで取得するサービスである — 日本語で入稿してもらう小規模サイトの候補にしやすい
- 無料のHobbyプランは最大3人のメンバー、5個のAPI、月20GBのデータ転送量に対応する — API呼び出し回数は無制限でも、転送量を超えるとAPIが停止する<sup><a href="#ref-4">[4]</a></sup>
- ACFのRepeaterに相当する機能は無料で利用できる — カスタムフィールドで「質問と回答」の組を作り、それを繰り返しフィールドへ設定する<sup><a href="#ref-5">[5]</a></sup>
- 編集者は必要な件数だけ項目を追加し、順序を変更できる — APIからは配列として取得できるため、Astroではその配列を順に描画する
- 同じ組の繰り返しに加え、見出し、画像付き本文などの異なるカスタムフィールドを組み合わせられる — この使い方は、ACFのFlexible Contentに近い用途にも対応する
- 繰り返しを使うためだけにTeamへ上げる必要はない — 編集人数、API数、権限など、実際に必要な条件でプランを選ぶ
- Hobbyは法人サイトでも利用できる — 無料枠に収まる企業サイトなら、個人向けプランという名称だけで候補から外す必要はない<sup><a href="#ref-6">[6]</a></sup>
- Astroでビルド時に取得すれば、通常のHTML閲覧ではCMSへ問い合わせずに済む — ただしmicroCMSの画像を直接配信する場合は、その転送量も確認する
- 日本語の編集環境と、ACFのRepeaterに近い入力方法を重視するなら、最初に試したい候補である

<ol start="4">
  <li id="ref-4"><a href="https://microcms.io/pricing">microCMS — 料金プラン</a></li>
  <li id="ref-5"><a href="https://document.microcms.io/manual/repeat-field">microCMS — 繰り返しフィールド</a></li>
  <li id="ref-6"><a href="https://help.microcms.io/ja/knowledge/free-plan-for-corporation">microCMS — Hobbyプランの法人サイトでの利用</a></li>
</ol>

## Contentful — 複数エントリーへの参照で繰り返しを表現する

- 記事を型に沿って管理し、APIから取得するサービスである — コンテンツを他のページや配信先でも再利用する設計と組み合わせやすい
- Freeプランは10ユーザー、2ロケール、月10万回のAPI呼び出し、月50GBのCDN帯域を含む<sup><a href="#ref-7">[7]</a></sup>
- ACFのRepeaterを代替するデータ構造は無料枠でも作れる — たとえば「質問」「回答」を持つFAQ用のコンテンツタイプを作り、親ページから複数のFAQエントリーを参照する<sup><a href="#ref-8">[8]</a></sup>
- ただし、親ページの中に行を直接追加する方式と同じではない — FAQを別のエントリーとして作成・公開し、その参照を親ページへ追加する運用になる
- FAQを複数ページで共有する場合は再利用しやすい一方、ページごとに大量の行を作る用途では編集と管理の作業が増える
- 繰り返しの1行を別エントリーとして管理するため、親ページの数だけでなく、参照先も含めたレコード数を見込む
- 重要な制限として、確認時点の公式Usage limitsではFreeはテスト・学習用途に限られ、商用用途は不可とされている — 「繰り返しを無料で試せる」ことと「企業サイトを無料運用できる」ことは分けて判断する<sup><a href="#ref-9">[9]</a></sup>
- まずはコンテンツ設計やAPI連携を学ぶための候補とし、商用公開を考える場合は有料プランを含めて比較したい

<ol start="7">
  <li id="ref-7"><a href="https://www.contentful.com/pricing/">Contentful — Pricing</a></li>
  <li id="ref-8"><a href="https://www.contentful.com/developers/docs/concepts/links/">Contentful — Entry and asset links</a></li>
  <li id="ref-9"><a href="https://www.contentful.com/help/admin/usage/usage-limit/">Contentful — Usage limits</a></li>
</ol>

## Sanity — オブジェクトの配列をコードで定義する

- 編集用のSanity Studioと、コンテンツを保存・配信する基盤を組み合わせるサービスである — 入力項目をJavaScriptやTypeScriptのスキーマで定義する
- Freeプランには最大20席、公開データセット2個、ドキュメント1万件などが含まれる — 編集画面を開発しながら調整したい場合の候補になる<sup><a href="#ref-10">[10]</a></sup>
- ACFのRepeaterに相当する機能は無料で利用できる — `array`型の中に`object`型を定義し、「質問と回答」の組を複数追加できるようにする<sup><a href="#ref-11">[11]</a></sup>
- 配列の要素を追加・並べ替えできるため、複数の入力項目をまとめた繰り返しとして扱える — この用途のために外部の有料プラグインを導入する必要はない
- 異なるオブジェクト型を配列へ含める設計もできる — 表示側では要素の型を判定し、対応するAstroコンポーネントへ渡す
- 無料プランのデータセットは公開型に限られる — 機密情報の置き場所として扱わず、必要なら非公開データセットを含むプランを検討する<sup><a href="#ref-10">[10]</a></sup>
- 無料プランの権限はAdministratorとViewerであり、人数だけでなく権限の分け方も確認する — 編集だけを任せたい場合に、管理者権限でよいかを判断する<sup><a href="#ref-10">[10]</a></sup>
- microCMSのように管理画面だけで項目を設定する方法と比べ、開発担当者がスキーマを管理する作業が必要になる
- 入力項目や編集画面をコードで管理し、繰り返しの構造を柔軟に作りたい場合に試したい

<ol start="10">
  <li id="ref-10"><a href="https://www.sanity.io/pricing">Sanity — Pricing</a></li>
  <li id="ref-11"><a href="https://www.sanity.io/docs/studio/array-type">Sanity — Array</a></li>
</ol>

## Hygraph — 複数値を許可したコンポーネントを使う

- GraphQLでコンテンツを取得するサービスである — 必要なフィールドを指定したクエリをAstroのビルド時に実行する
- 無料のHobbyプランには3席、1,000件のコンテンツエントリー、月50万回のAPI呼び出し、10個のコンポーネントが含まれる<sup><a href="#ref-12">[12]</a></sup>
- ACFのRepeaterに相当する機能は無料で利用できる — 「質問と回答」のコンポーネントを作り、親のコンポーネントフィールドで`Allow multiple values`を有効にする<sup><a href="#ref-13">[13]</a></sup>
- 同じ型を繰り返す場合はBasicコンポーネントフィールド、複数の型を組み合わせる場合はModularコンポーネントフィールドを使う — どちらも複数の値に対応する
- コンポーネント数の上限と、記事内に追加できる繰り返し項目の数は別の条件である — 「10コンポーネントだから1記事10行まで」とは判断しない
- 公式ドキュメントでは、編集画面で扱えるコンポーネントのインスタンス数や入れ子の深さにも制限が示されている — 大量の繰り返しや深い入れ子を作る場合は、その条件も確認する<sup><a href="#ref-13">[13]</a></sup>
- Hobbyは個人のプロジェクトや試用向けに案内されている — 複数人で業務運用する場合は、権限や契約条件を確認する
- REST APIに慣れている場合はGraphQLの学習も必要になるが、コンテンツの構造と取得する項目を明示して作りたい場合の候補になる

<ol start="12">
  <li id="ref-12"><a href="https://hygraph.com/docs/developer-guides/project/update-billing">Hygraph — プランごとの利用上限</a></li>
  <li id="ref-13"><a href="https://hygraph.com/docs/developer-guides/schema/components">Hygraph — Components</a></li>
</ol>

## Storyblok — ブロックを繰り返してページを組み立てる

- コンテンツをブロックとして管理し、プレビューと結び付けて編集するサービスである — ページを構成する部品を組み合わせる用途で比較したい
- 無料のStarterには1席、月10万回のAPIリクエスト、200個のコンポーネントが含まれる — 上位プランの45日間トライアルと、継続無料のStarterを混同しない<sup><a href="#ref-14">[14]</a></sup>
- ACFのRepeaterに相当する機能は無料で利用できる — 「質問と回答」を持つネスト可能なブロックを作り、親の`Blocks`フィールドへ複数追加する<sup><a href="#ref-15">[15]</a></sup>
- 同じ型を繰り返すだけでなく、見出し、画像付き本文などのブロックを組み合わせられる — ACFのFlexible Contentに近いページ構成にも使える
- `Blocks`フィールドには入力できるブロック数の最小・最大を設定できる — 自由に追加できることと、サイトの表示ルールを保つことを両立させる<sup><a href="#ref-16">[16]</a></sup>
- Astroでは、ブロックの種類に対応するコンポーネントを用意する — CMSのブロックを追加しただけで、新しい見た目まで自動生成されるわけではない
- 静的公開と、編集用のライブプレビューは別に設計する — プレビューを使いたい場合は、そのための表示環境とアクセス制御も用意する
- Starterはテストや個人プロジェクト向けに案内されており、標準の席数も少ない — 編集人数や必要な機能が増えた場合の費用を確認する
- 一人でブロック編集を試す場合や、入稿画面とページの見え方を近づけたい場合の候補になる

<ol start="14">
  <li id="ref-14"><a href="https://www.storyblok.com/pricing">Storyblok — Pricing</a></li>
  <li id="ref-15"><a href="https://www.storyblok.com/docs/concepts/blocks">Storyblok — Blocks</a></li>
  <li id="ref-16"><a href="https://www.storyblok.com/docs/concepts/fields#blocks">Storyblok — Blocksフィールド</a></li>
</ol>

## まとめ

- 日本語で編集し、ACFのRepeaterに近い入力方法を使いたいなら、microCMSのHobbyから試しやすい
- Contentfulは複数エントリーへの参照で代替できるが、Freeの商用利用制限と、行ごとにエントリーを管理する違いがある
- Sanityはオブジェクトの配列で繰り返しを作れ、コードでスキーマを管理したい場合に向く — 公開データセットと権限の条件も確認する
- Hygraphは複数値に対応したコンポーネントで繰り返しを作れ、GraphQLによる取得と組み合わせられる
- Storyblokは`Blocks`で繰り返しを作れ、ブロックを組み合わせるページ編集を試せる
- 繰り返し機能が無料でも、件数、転送量、人数、権限、用途の条件まで同じとは限らない
- Astroの静的生成では、CMSの更新後に再ビルドと転送が必要になる — CMSの更新を契機にビルドを起動する方法も用意する
- CMSの料金に加え、レンタルサーバー、GitHub Actions、プレビュー環境、保守の費用も含めて選びたい
