# GASを使用したCMSを作成したい — Astroで静的生成し、レンタルサーバーへ公開する

## GASを使ったCMSとは

- GAS（Google Apps Script）はGoogleのサービスをJavaScriptで操作し、処理を自動化する実行環境である
- Googleスプレッドシートを記事の入力画面にし、GASで公開用データをGitHubへ送れば、小規模なCMSを自作できる
- 想定する流れは「スプレッドシートで編集 → GASで公開用JSONを作成 → GitHubにコミット → GitHub ActionsでAstroをビルド → レンタルサーバーへアップロード」である
- GASでGitコマンドを実行する構成にはせず、GitHubのAPIでファイルを更新する — 利用者から見ると、記事の更新がリポジトリに反映される仕組みになる
- Astroはビルド時にHTMLなどを生成し、生成物を公開サーバーへ配置できる — 静的サイトとして構築すれば、閲覧のたびにGASやGoogleスプレッドシートへアクセスする必要はない<sup><a href="#ref-1">[1]</a></sup>
- お知らせなど、入力項目が少なく更新頻度も高すぎないサイトを想定する — 高機能な編集画面や承認フローまで必要なら、作り込む費用も考える
- 以下は構成案であり、実際の公開手順はレンタルサーバーの接続方式や権限に合わせて決める
- 公式情報の確認日は2026年10月26日

<ol start="1">
  <li id="ref-1"><a href="https://docs.astro.build/en/guides/deploy/">Astro — Deploy your Astro Site</a></li>
</ol>

## スプレッドシートの更新を検知する

- 入力項目は記事を識別する`id`、URLに使う`slug`、タイトル、本文、公開日時、公開状態などから始める
- 行番号は並べ替えや行の追加で変わるため、記事の識別には固定の`id`を使う
- 画像も扱うなら、保存先と配信先を別に決める — Driveの共有URLを記入しただけで、Web向けの画像配信と権限管理が完成するわけではない
- セルの編集を検知する方法として編集トリガーがあるが、単純な`onEdit(e)`では認可が必要なサービスを利用できない — GitHubへの外部通信を伴う処理は、認可した関数やインストール型トリガーで実行する<sup><a href="#ref-2">[2]</a></sup><sup><a href="#ref-3">[3]</a></sup>
- セルを編集するたびに公開すると、書きかけの記事を反映したり、短時間に多数のビルドを実行したりする可能性がある
- 最初は編集と公開を分け、公開したい記事を確定してからGASの同期処理を実行する運用が分かりやすい
- 自動化する場合は時間主導型トリガーで公開対象を定期的に読み、前回GitHubへ送れた内容と比較する方法を候補にする<sup><a href="#ref-3">[3]</a></sup>
- JSONを作る際は記事の並び順と日付形式を固定し、その内容のハッシュを比較する — 毎回変わる処理時刻などは、変更判定の対象へ含めない
- シート内の値をスクリプトやAPIで変更しても、編集トリガーは実行されない — 数式の再計算や外部連携まで含めて追いたい場合は、編集トリガーだけに依存しない<sup><a href="#ref-3">[3]</a></sup>
- 定期実行は即時公開を保証するものではない — 予約公開にも使えるが、実行間隔、ビルド、転送にかかる時間を見込む
- 公開を取り消した記事や削除した記事も反映できるよう、公開対象全体のJSONを作り直す構成にすると扱いやすい

<ol start="2">
  <li id="ref-2"><a href="https://developers.google.com/apps-script/guides/triggers">Google — Simple Triggers</a></li>
  <li id="ref-3"><a href="https://developers.google.com/apps-script/guides/triggers/installable">Google — Installable Triggers</a></li>
</ol>

## GitHubへ公開用データを送る

- GitHubのContents APIを使うと、指定したリポジトリのファイルを作成・更新できる — GASからは`UrlFetchApp.fetch()`でHTTPSリクエストを送る<sup><a href="#ref-4">[4]</a></sup><sup><a href="#ref-5">[5]</a></sup>
- 小規模なサイトなら、公開対象を`src/data/posts.json`のような1ファイルにまとめると、1回の更新で記事一覧をそろえられる
- JSONの例は次のようになる — 下書きや内部メモは含めず、公開に必要な項目だけを取り出す

```json
[
  {
    "id": "news-001",
    "slug": "site-renewal",
    "title": "サイトをリニューアルした",
    "body": "新しいサイトの公開についてのお知らせ",
    "publishedAt": "2026-10-26T09:00:00+09:00"
  }
]
```

- APIへはファイルの内容をBase64に変換して渡し、更新時には取得済みのファイルの`sha`も指定する — ここでの`sha`は更新前のファイルを識別する値である<sup><a href="#ref-4">[4]</a></sup>
- 他の更新と競合した場合は最新の内容を取得してやり直す — GAS側でも`LockService`で同期処理の同時実行を抑える<sup><a href="#ref-6">[6]</a></sup>
- GitHubへの送信が成功した後に比較用のハッシュと同期日時を保存する — 失敗した送信も送信済みとして記録すると、次回の再送ができなくなる
- GitHubへ反映できたことと、公開サイトへ反映できたことは分けて記録する — コミット後にAstroのビルドやアップロードが失敗する場合もある
- APIでコミットが作成されると、`push`イベントの実行条件に合うGitHub Actionsワークフローを起動できる — ワークフロー側で対象ブランチやファイルを指定する<sup><a href="#ref-7">[7]</a></sup>
- GitHub Actionsの`GITHUB_TOKEN`で作ったコミットは、通常、新たな`push`ワークフローを起動しない — 今回のGASからの更新では、対象リポジトリに限定した認証情報を別に用意する<sup><a href="#ref-7">[7]</a></sup>

<ol start="4">
  <li id="ref-4"><a href="https://docs.github.com/en/rest/repos/contents">GitHub — REST API endpoints for repository contents</a></li>
  <li id="ref-5"><a href="https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app">Google — Class UrlFetchApp</a></li>
  <li id="ref-6"><a href="https://developers.google.com/apps-script/reference/lock/">Google — Lock Service</a></li>
  <li id="ref-7"><a href="https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow">GitHub — Triggering a workflow</a></li>
</ol>

## Astroでビルドし、レンタルサーバーへ配置する

- Astroは静的生成の設定で利用し、公開用JSONから一覧ページと記事ページを作る — 静的生成だけなら、公開サーバーでNode.jsを動かす必要はない
- 記事データはビルド時にも検証する — 必須項目、日付、`id`と`slug`の重複、不正なURLなどを確認し、問題があれば公開を止める
- AstroのContent Collectionsでは、スキーマによるコンテンツの検証を組み込める<sup><a href="#ref-8">[8]</a></sup>
- GitHub Actionsでは依存関係をインストールしてビルドし、成功した場合だけ生成先の`dist/`を転送する — ソースコード、`.git`、環境変数のファイルは公開しない
- レンタルサーバーを契約するときは、SFTP・SSH・FTPSなどの対応状況と、自動接続に使える認証方法を確認する — 平文FTPを前提にしない
- SSHやSFTPでは接続先のホスト鍵を検証し、接続できればどのサーバーでもよい設定にしない
- 複数の公開処理が同時に同じ場所へ書き込まないよう、デプロイの同時実行を制御する — GitHub Actionsには`concurrency`による制御がある<sup><a href="#ref-9">[9]</a></sup>
- 公開先へ順番に上書きすると、一時的に新旧ファイルが混在する可能性がある — サーバーが対応していれば、別ディレクトリへ転送してから公開先を切り替える方式を検討する
- 切り替え方式が使えない場合も、転送完了の確認、失敗時の復旧、削除済みページの除去を手順に含める — GitHubから記事を消すだけでは、サーバーに残ったHTMLは消えない
- 前回正常に公開した生成物とコミットを記録し、公開に失敗した場合は同じコミットから再実行できるようにする
- GASには実行時間やAPI呼び出しなどの割り当てがあるため、ビルドと転送はGASへ集約せずGitHub Actionsへ任せる — 上限と課金条件は利用する各サービスで確認する<sup><a href="#ref-10">[10]</a></sup>
- ランニングコストにはレンタルサーバー、ドメイン、メール、フォーム、GitHub Actionsの利用料を含める — 既存契約と無料枠で収まる部分があっても、CMS全体が無条件に無料とは限らない

<ol start="8">
  <li id="ref-8"><a href="https://docs.astro.build/en/guides/content-collections/">Astro — Content collections</a></li>
  <li id="ref-9"><a href="https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/control-deployments">GitHub — Deploying with GitHub Actions</a></li>
  <li id="ref-10"><a href="https://developers.google.com/apps-script/guides/services/quotas">Google — Quotas for Google Services</a></li>
</ol>

## セキュリティとフォームの扱い

### GoogleとGitHubの権限を分ける

- 公開側をHTMLなどの静的ファイルだけにすれば、記事を表示するためのPHPやデータベースを自分で稼働させずに済む — 保護する対象はGoogleアカウント、GAS、GitHub、転送先にも残る
- 編集用シートは関係者だけに共有し、同期用GASは編集者を絞ったスタンドアロンスクリプトとして管理する — 広く共有したシートに紐づくスクリプトへ強い認証情報を置かない
- インストール型トリガーは作成者の権限で実行されるため、担当者の退職やアカウント停止、権限変更時の引き継ぎも決める<sup><a href="#ref-11">[11]</a></sup>
- GitHubの認証情報はシートのセルへ書かず、GASのScript Propertiesなどで管理する — Script Propertiesはスクリプト単位で共有される保存先であり、スクリプト編集者から秘密を隔離するものではない<sup><a href="#ref-12">[12]</a></sup>
- fine-grained personal access token（対象と権限を限定できる個人アクセストークン）を使う場合は、対象リポジトリと有効期限を設定し、必要な`Contents: write`を与える — ワークフローを書き換える権限は追加しない<sup><a href="#ref-4">[4]</a></sup>
- `Contents: write`は特定のデータファイルだけに書き込み先を限定する権限ではない — トークンを盗まれた場合の影響を抑えたいなら、記事データとサイトのコードを別リポジトリへ分ける設計も検討する

### 記事データをコードとして実行しない

- GASの送信先と保存するパスは固定し、シートの値から任意のファイルを書き換えられる設計にしない
- 記事本文を`.astro`やMDXのソースコードとして生成する構成は避け、JSONなどのデータとして渡す — 記事の編集権限がビルド時のコード実行につながらないようにする
- HTMLを許可する場合はタグや属性、URLの検証が必要になる — 静的HTMLでも、不正なスクリプトを埋め込んで公開すれば閲覧者へ影響する
- 記事タイトルなどをGitHub Actionsの`run`へ直接埋め込まない — 外部入力がシェルのコードとして実行される問題を避ける<sup><a href="#ref-13">[13]</a></sup>
- サーバーの接続情報はGitHub ActionsのSecretsで管理し、ログへ出力しない — デプロイ用アカウントの権限を絞り、利用する外部Actionも確認する<sup><a href="#ref-14">[14]</a></sup>

### 問い合わせは外部サービスで受け付ける

- 問い合わせフォームはSSGformなどの外部サービスを利用する — 自作HTMLから専用URLへPOSTして受付と通知を任せられる<sup><a href="#ref-15">[15]</a></sup>
- 問い合わせ内容を記事用シートへ保存したり、記事の公開用JSONへ混ぜたりしない
- フォームの公開URLへ送信できることを前提に、スパム対策、件数上限、通知の不達、個人情報の保存・削除方法を確認する
- この構成では、GASを匿名で呼べる公開Webアプリとして設置する必要はない — 公開サイトの閲覧と問い合わせ受付を、CMS用GASの実行から分けられる
- 小規模なお知らせサイトなら実現を検討しやすいが、公開処理の監視と復旧は自分で用意する必要がある — CMSサービスの利用料を減らす代わりに、連携処理の保守を引き受ける構成である

<ol start="11">
  <li id="ref-11"><a href="https://developers.google.com/apps-script/guides/triggers/installable">Google — インストール型トリガーの実行権限</a></li>
  <li id="ref-12"><a href="https://developers.google.com/apps-script/guides/properties">Google — Properties Service</a></li>
  <li id="ref-13"><a href="https://docs.github.com/en/actions/concepts/security/script-injections">GitHub — Script injections</a></li>
  <li id="ref-14"><a href="https://docs.github.com/en/actions/reference/security/secure-use">GitHub — Secure use reference</a></li>
  <li id="ref-15"><a href="https://ssgform.com/">SSGform — 静的サイトでのフォーム設置</a></li>
</ol>

## まとめ

- スプレッドシートを入力画面、GASを同期処理に使う小規模なCMSは構築できる
- 更新の反映にはGitHub APIを使い、Astroのビルドとサーバーへの転送はGitHub Actionsで実行する
- 編集と公開を分け、定期実行で公開対象の変更を検知すれば、編集のたびにビルドする構成を避けられる
- GitHubへの同期成功と公開成功を区別し、競合、再実行、削除済みページ、公開失敗時の復旧を扱う
- 公開側を静的にしても、GoogleとGitHubの権限、認証情報、記事データ、転送先の保護は必要である
- 記事はコードとして実行せず、検証したデータとしてビルドへ渡す
- 問い合わせは外部フォームサービスへ分け、記事の公開データと混在させない
- サービス利用料だけでなく、連携処理の監視と保守にかかる費用も含めて採用を判断する
