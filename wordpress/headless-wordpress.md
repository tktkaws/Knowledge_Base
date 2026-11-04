# WordPressをヘッドレスCMSとして利用する

## ヘッドレスCMSとは

- ヘッドレスCMSとは、記事の管理と、読者に見せるページの表示を分けたCMS（コンテンツ管理システム）である
- WordPressで記事を書き、AstroがREST API（HTTPでデータを取得・操作する仕組み）から記事を取得してHTMLを生成する構成を組める<sup><a href="#ref-1">[1]</a></sup>
- 公開サイトを `www.example.com`、WordPressを `cms.example.com` に分けると、公開先と管理用のサーバーを区別しやすい
- Astroの静的ビルドで生成したファイルは、HTMLやCSSを配信できるレンタルサーバーに置ける — ビルドはGitHub Actionsなどの別環境で実行する<sup><a href="#ref-2">[2]</a></sup>
- 読者へのHTML配信ではWordPressのPHPを実行しないが、CMS用のPHP、データベース、WordPress本体とプラグインの保守は引き続き必要になる
- 静的ページへの反映は再ビルドとアップロードの完了後になる — 下書きのプレビューも、Astro側で別途用意する必要がある

```text
編集者がWordPressで記事を公開・更新
  → WordPressからGitHubへ更新を通知
  → GitHub ActionsでAstroをビルド
  → 生成したHTML・CSS・画像をレンタルサーバーへ転送
  → 読者は公開サイトの静的ページを閲覧
```

参照

<ol start="1">
  <li id="ref-1"><a href="https://docs.astro.build/en/guides/cms/wordpress/">Astro — Headless WordPress &amp; Astro</a></li>
  <li id="ref-2"><a href="https://docs.astro.build/en/guides/deploy/">Astro — Deploy your Astro Site</a></li>
</ol>

## Astroで記事を取得して静的ページを作る

- 通常の投稿は `/wp-json/wp/v2/posts` から取得する — 固定ページやカスタム投稿タイプは、それぞれ対応するエンドポイントを使う
- 記事ごとのページは、Astroの `getStaticPaths()` でURLを列挙して生成する<sup><a href="#ref-3">[3]</a></sup>
- REST APIの一覧にはページ分割がある — `per_page` の上限は100件で、全件取得には `page` とレスポンスヘッダーの `X-WP-TotalPages` を使う<sup><a href="#ref-4">[4]</a></sup>
- APIがエラーを返した場合はビルドを止める — 通信失敗を「記事が0件になった」と扱うと、記事のないサイトを公開しかねない
- 次の例は、Basic認証をかけたCMSから公開済みの投稿を取得する処理である — 認証情報はビルド環境の環境変数に設定し、ブラウザーで動くスクリプトに含めない

```ts
// src/lib/wordpress.ts
import { Buffer } from 'node:buffer';

type Post = {
  id: number;
  title: { rendered: string };
  content: { rendered: string };
};

export async function fetchPosts(): Promise<Post[]> {
  const { WP_ORIGIN, CMS_BASIC_USER, CMS_BASIC_PASSWORD } = import.meta.env;
  if (!WP_ORIGIN || !CMS_BASIC_USER || !CMS_BASIC_PASSWORD) {
    throw new Error('CMSへの接続設定が不足している');
  }

  const token = Buffer.from(
    `${CMS_BASIC_USER}:${CMS_BASIC_PASSWORD}`,
  ).toString('base64');
  const posts: Post[] = [];
  let totalPages = 1;

  for (let page = 1; page <= totalPages; page++) {
    const url = new URL('/wp-json/wp/v2/posts', WP_ORIGIN);
    url.search = new URLSearchParams({
      status: 'publish',
      per_page: '100',
      page: String(page),
      orderby: 'id',
      order: 'asc',
    }).toString();

    const response = await fetch(url, {
      headers: { Authorization: `Basic ${token}` },
    });
    if (!response.ok) {
      throw new Error(`記事の取得に失敗: HTTP ${response.status}`);
    }

    const pages = response.headers.get('X-WP-TotalPages');
    if (pages === null || !/^\d+$/.test(pages)) {
      throw new Error('記事の総ページ数を取得できない');
    }
    totalPages = Number(pages);
    posts.push(...(await response.json() as Post[]));
  }

  return posts;
}
```

- `WP_ORIGIN` は `https://cms.example.com` のようなオリジンを指定する — この例はWordPressがドメイン直下にあり、REST APIの標準URLを使える前提である
- 以下を `src/pages/posts/[id].astro` に置くと、投稿IDに対応するページを生成できる
- `set:html` はHTMLをエスケープせず出力する — 信頼できる編集者が管理する本文を使い、必要に応じてサニタイズする<sup><a href="#ref-5">[5]</a></sup>
- WordPressのテーマのCSSやJavaScriptは自動では付かない — ブロックの見た目や動的な機能はAstro側でも確認する
- ページ分割で取得している間に記事が増減すると、取得内容がそろわない場合がある — 更新が多いサイトでは、取得結果の再確認や再ビルドも考える

```astro
---
import { fetchPosts } from '../../lib/wordpress';

export async function getStaticPaths() {
  const posts = await fetchPosts();
  return posts.map((post) => ({
    params: { id: String(post.id) },
    props: { post },
  }));
}

const { post } = Astro.props;
---

<!doctype html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width" />
    <title>記事 {post.id}</title>
  </head>
  <body>
    <main>
      <article>
        <h1 set:html={post.title.rendered} />
        <div set:html={post.content.rendered} />
      </article>
    </main>
  </body>
</html>
```

参照

<ol start="3">
  <li id="ref-3"><a href="https://docs.astro.build/en/guides/cms/wordpress/">Astro — WordPressの記事からページを生成する</a></li>
  <li id="ref-4"><a href="https://developer.wordpress.org/rest-api/using-the-rest-api/pagination/">WordPress — Pagination</a></li>
  <li id="ref-5"><a href="https://docs.astro.build/en/reference/directives-reference/#sethtml">Astro — set:html</a></li>
</ol>

## WordPressの更新を検知してGitHubへ通知する

- Webhookとは、CMSの更新などをきっかけに、指定したURLへHTTPリクエストを送る仕組みである
- microCMSのGitHub Actions連携では、コンテンツの変更をWebhookで通知し、`repository_dispatch` でビルドを開始する — WordPressでも同じ流れを組める<sup><a href="#ref-6">[6]</a></sup>
- 記事データはWordPressで管理し、GitHubにはAstroのソースコードを置く — 通知を受けたActionsがWordPressのREST APIから最新の記事を取得する
- 編集者はWordPressで公開・更新するだけでよい — 通常の記事更新でGitのコミットやpushを操作する必要はない

```text
WordPressで記事を公開・更新・公開解除・削除
  → WebhookでGitHubのAPIへPOST
  → repository_dispatchを受けたGitHub Actionsが起動
  → WordPressのREST APIから公開記事を取得
  → Astroでビルド
  → レンタルサーバーへ自動転送
```

### WordPress側でWebhookの送信先を設定する

- Webhook送信用のプラグインを利用し、通知する操作と送信先を設定する — 送信イベント、HTTPヘッダー、JSONの本文を変更できるものを選ぶ
- 独自プラグインで実装する場合は、保存後の `wp_after_insert_post` とHTTP送信用の `wp_remote_post()` を利用できる<sup><a href="#ref-7">[7]</a></sup>
- 送信先URLは `https://api.github.com/repos/OWNER/REPO/dispatches` とし、`OWNER/REPO` をAstroのリポジトリに置き換える
- HTTPメソッドは `POST`、`Accept` ヘッダーは `application/vnd.github+json`、`Content-Type` ヘッダーは `application/json` にする
- `Authorization` ヘッダーには `Bearer <GitHubトークン>` を設定する — トークンはWordPress側のサーバーで保持し、公開するコードやブラウザーに渡さない
- fine-grained personal access tokenは対象リポジトリを限定し、APIが要求する `Contents: write` を付与する — 有効期限も管理する<sup><a href="#ref-8">[8]</a></sup>
- JSONの本文は次の形にする — `wordpress_updated` は次のセクションのActions側の `types` と一致させる

```json
{
  "event_type": "wordpress_updated"
}
```

- 通知対象は、記事の新規公開、公開済み記事の更新、公開解除、削除にする — 公開解除や削除も再ビルドし、公開ページへ反映する
- 下書きの保存、自動保存、リビジョンの作成は除外する — 本文やカスタムフィールドなど、ビルドに使うデータの保存完了後に通知する
- 独自実装では、保存後または保存前の `post_status` が `publish` の投稿を通知対象にする — 完全削除やカテゴリー名・画像だけの変更は、対応するイベントでも通知する
- GitHub APIの `204` は通知の受け付け成功を示す — ビルドや公開の完了はActionsの実行結果で確認する
- 送信に失敗した場合のログと再送手段を用意する — 次の例では `workflow_dispatch` による手動実行も残しておく

参照

<ol start="6">
  <li id="ref-6"><a href="https://document.microcms.io/manual/webhook-setting">microCMS — コンテンツのWebhookを設定</a></li>
  <li id="ref-7"><a href="https://developer.wordpress.org/reference/hooks/wp_after_insert_post/">WordPress — wp_after_insert_post</a> / <a href="https://developer.wordpress.org/reference/functions/wp_remote_post/">wp_remote_post()</a></li>
  <li id="ref-8"><a href="https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event">GitHub — Create a repository dispatch event</a></li>
</ol>

## GitHub Actionsでビルドしてレンタルサーバーへ公開する

- WordPressのWebhookを受け取ったら、記事の取得、Astroのビルド、レンタルサーバーへの転送を1つのワークフローで実行する
- `.github/workflows/deploy.yml` をリポジトリのデフォルトブランチに置く — `repository_dispatch` を受け取るワークフローはデフォルトブランチに存在する必要がある<sup><a href="#ref-9">[9]</a></sup>
- 以下はSSHの公開鍵認証と `rsync` に対応したレンタルサーバー向けの例である — サーバー側にも `rsync` が必要で、SFTP・FTPSだけに対応する場合は転送ステップを置き換える
- `package-lock.json` をコミットし、使用するAstroが対応するNode.jsのバージョンを `.nvmrc` に記載しておく
- `WP_ORIGIN` はGitHub ActionsのVariablesに、`CMS_BASIC_USER` と `CMS_BASIC_PASSWORD` はSecretsに登録する
- SSH接続用の `DEPLOY_HOST`、`DEPLOY_USER`、`DEPLOY_KEY`、`DEPLOY_KNOWN_HOSTS` もSecretsに登録する — 公開鍵はサーバー側へ登録し、ホスト鍵は契約先の案内など信頼できる方法で確認する
- SSHのポート番号はVariablesの `DEPLOY_PORT` に設定する — 未設定の場合は22を使う
- コード中の `/home/ACCOUNT/www/site/` は、公開サイト専用のディレクトリへ置き換える — WordPressやバックアップと同じディレクトリを指定しない
- `concurrency` で公開処理を直列に実行する — 転送の途中で中断しないよう、この例は `cancel-in-progress: false` にする<sup><a href="#ref-10">[10]</a></sup>

```yaml
name: Build and deploy Astro

on:
  repository_dispatch:
    types: [wordpress_updated]
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: production-site
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-node@v7
        with:
          node-version-file: .nvmrc
          cache: npm
      - run: npm ci
      - run: npm run build
        env:
          WP_ORIGIN: ${{ vars.WP_ORIGIN }}
          CMS_BASIC_USER: ${{ secrets.CMS_BASIC_USER }}
          CMS_BASIC_PASSWORD: ${{ secrets.CMS_BASIC_PASSWORD }}
      - uses: actions/upload-artifact@v7
        with:
          name: static-site
          path: dist/
          if-no-files-found: error
      - name: Deploy to rental server
        env:
          DEPLOY_HOST: ${{ secrets.DEPLOY_HOST }}
          DEPLOY_USER: ${{ secrets.DEPLOY_USER }}
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
          DEPLOY_KNOWN_HOSTS: ${{ secrets.DEPLOY_KNOWN_HOSTS }}
          DEPLOY_PORT: ${{ vars.DEPLOY_PORT || '22' }}
        run: |
          install -m 700 -d ~/.ssh
          printf '%s\n' "$DEPLOY_KEY" > ~/.ssh/id_ed25519
          printf '%s\n' "$DEPLOY_KNOWN_HOSTS" > ~/.ssh/known_hosts
          chmod 600 ~/.ssh/id_ed25519 ~/.ssh/known_hosts
          rsync -az --delete-delay \
            -e "ssh -p $DEPLOY_PORT -o BatchMode=yes -o StrictHostKeyChecking=yes" \
            dist/ "$DEPLOY_USER@$DEPLOY_HOST:/home/ACCOUNT/www/site/"
```

- Actionsの各設定は公式リポジトリで確認できる — 導入時には使用するバージョンを確認する<sup><a href="#ref-11">[11]</a></sup><sup><a href="#ref-12">[12]</a></sup><sup><a href="#ref-13">[13]</a></sup>
- 転送するのは `dist/` の中身である — 公開先でNode.jsを起動する必要はない
- ビルドが成功した場合だけ転送し、CMSのファイルと公開サイトの転送先を分ける — 同じレンタルサーバーでも、配置先と書き込み権限を区別する
- `rsync` の `--delete-delay` は、転送先にだけ残っているファイルを転送終了時に削除する — 非公開化・削除した記事や変更前のURLのHTMLを公開先から取り除くために使う<sup><a href="#ref-14">[14]</a></sup>
- 公開先にだけ置いた `.htaccess` なども削除対象になる — 公開サイトに必要な設定ファイルはAstroの `public/` などで管理し、`dist/` に含める
- 初回は `rsync` に `--dry-run` を追加し、転送・削除の対象と配置先を確認してから実際の転送を有効にする
- 転送失敗に備えて直前の成果物を保存する — サーバーが対応する場合は、別ディレクトリへの転送完了後に公開先を切り替える
- WordPressの予約公開は、WordPress側で実際に公開処理が動いてから通知する — 管理サイトへのアクセスが少ない場合は、サーバーのcronでWP-Cronを動かす方法も検討する<sup><a href="#ref-15">[15]</a></sup>

参照

<ol start="9">
  <li id="ref-9"><a href="https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#repository_dispatch">GitHub — repository_dispatchの実行条件</a></li>
  <li id="ref-10"><a href="https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/control-workflow-concurrency">GitHub — Control the concurrency of workflows and jobs</a></li>
  <li id="ref-11"><a href="https://github.com/actions/checkout">GitHub — actions/checkout</a></li>
  <li id="ref-12"><a href="https://github.com/actions/setup-node">GitHub — actions/setup-node</a></li>
  <li id="ref-13"><a href="https://github.com/actions/upload-artifact">GitHub — actions/upload-artifact</a></li>
  <li id="ref-14"><a href="https://download.samba.org/pub/rsync/rsync.1">rsync — Manual</a></li>
  <li id="ref-15"><a href="https://developer.wordpress.org/plugins/cron/hooking-wp-cron-into-the-system-task-scheduler/">WordPress — Hooking WP-Cron Into the System Task Scheduler</a></li>
</ol>

## 管理画面のBasic認証と外部フォーム

### Basic認証をかける範囲

- WordPressのログインとは別に、Webサーバー側でBasic認証をかけられる — 通信にはHTTPSを使う<sup><a href="#ref-16">[16]</a></sup>
- 管理画面だけを保護する場合は、`/wp-admin/` に加えて `/wp-login.php` も対象にする — この設定だけではREST APIやアップロード画像は保護されない
- 公開サイトからCMSへのアクセスを不要にする構成なら、`cms.example.com` 全体にBasic認証をかける方法もある — 記事取得時は、前述のようにビルド環境から認証情報を送る
- 次はApacheでCMS全体を保護する場合の設定例である — CMSのドキュメントルートの `.htaccess` に追加し、既存のWordPress用のリライトルールは残す
- `.htaccess` と認証用モジュールが使えることを確認し、`AuthUserFile` は実際の絶対パスに置き換える — `.htpasswd` は公開ディレクトリの外に作成する

```apache
AuthType Basic
AuthName "CMS"
AuthUserFile /home/ACCOUNT/private/.htpasswd
Require valid-user
```

- サーバーのBasic認証と、WordPressのApplication Passwordsは認証する相手が異なる — Application PasswordsはREST APIでWordPressユーザーを認証する機能である<sup><a href="#ref-17">[17]</a></sup>
- サーバーのBasic認証とApplication Passwordsは同じ `Authorization` ヘッダーを使うため、別々の認証情報をそのまま重ねられない — PHPへのヘッダー転送設定によっては競合するため、ビルド環境から公開記事を取得できるか確認する
- CMS全体を保護する場合は、本文中の画像URLにも注意する — WordPressの画像URLを残すと、読者のブラウザーにも認証が要求される
- この場合はビルド時に必要な画像を認証付きで取得し、公開サイトへコピーしてURLを書き換える — `src` に加え、`srcset` やPDFなどのリンクも対象を確認する
- 前述のコード例は画像のコピーまでは行わない — 公開前に追加実装するか、公開用メディアだけを認証不要で配信する構成を選ぶ
- CMS全体を保護するとWP-CronへのHTTPアクセスも認証対象になる — cronの実行方法を合わせて確認する
- ヘッドレス化した後も、WordPressの更新、不要なプラグインの削除、ログイン対策、バックアップを続ける

### 問い合わせフォームを外部サービスに任せる

- フォームのHTMLはAstroで作り、送信先を外部サービスにする — Formspreeでは、フォームの `action` に専用のURLを指定する構成を取れる<sup><a href="#ref-18">[18]</a></sup>
- `YOUR_FORM_ID` は作成したフォームのIDに置き換える — 各入力項目に `name` を付け、送信後の画面と実際の受信を確認する
- 送信上限、スパム対策、通知メール、保存期間、個人情報の取り扱いを確認してから採用する
- Contact Form 7などWordPress側のフォーム機能を使わなければ、公開サイトからWordPressへフォームを送信する処理を設けずに済む

```html
<form action="https://formspree.io/f/YOUR_FORM_ID" method="post">
  <label for="email">メールアドレス</label>
  <input id="email" name="email" type="email" autocomplete="email" required />

  <label for="message">お問い合わせ内容</label>
  <textarea id="message" name="message" required></textarea>

  <button type="submit">送信する</button>
</form>
```

参照

<ol start="16">
  <li id="ref-16"><a href="https://httpd.apache.org/docs/2.4/howto/auth.html">Apache HTTP Server — Authentication and Authorization</a></li>
  <li id="ref-17"><a href="https://developer.wordpress.org/rest-api/using-the-rest-api/authentication/">WordPress — Authentication</a></li>
  <li id="ref-18"><a href="https://formspree.io/html/">Formspree — Simple HTML forms</a></li>
</ol>

## まとめ

- WordPressで記事を管理し、Astroで静的ページを生成してレンタルサーバーに公開する構成を組める
- 公開記事の取得はビルド時に行い、ページ分割とAPIエラーを扱う
- microCMSのWebhookと同じ流れで、WordPressの更新を `repository_dispatch` で通知し、ビルドと公開を自動化できる
- 公開・更新に加え、非公開化や削除も通知し、公開先に残った古いHTMLを削除する
- Basic認証をかける範囲に合わせて、APIへの接続、画像の配信、WP-Cronの実行方法を確認する
- フォームは外部サービスで受信し、送信結果やスパム対策、個人情報の取り扱いを確認する
- WordPressの保守と、ビルド・転送失敗への対応は継続して必要になる
