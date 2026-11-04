# EmDashについて調べてみた — WordPressと比べた運用の差

## EmDashとは

- EmDash は、Astro のアプリに管理画面を足すオープンソースの CMS — ライセンスは MIT で、ソフトウェア自体の利用料は 0 円
- 公開ページと管理画面は同じデプロイに載る — 管理画面のパスは `/_emdash/admin/`
- 編集者は管理画面で入稿し、開発者は Astro のページとコンポーネントで HTML を書く
- 本文は Portable Text という構造化 JSON で保存する — WordPress のように HTML 文字列として持たない
- 公開は Astro の Live Content Collections 経由 — サーバー描画のページは、次のリクエストで公開済みの変更を読める。プリレンダーしたページは再ビルドまで静的のまま
- Cloudflare が 2026年4月1日に発表し、2026年9月28日に 1.0 を安定版として出している
- 向くのは、Astro で作る新規サイト、編集者に Git を触らせたくないサイト、WordPress からフロントを Astro で作り直す移行
- Astro 以外のフロント、複数の無関係なアプリで共有するヘッドレス CMS、WooCommerce 級の EC を本体に求める案件には向かない

> 参照: [Why EmDash](https://docs.emdashcms.com/why-emdash/) / [EmDash 1.0: the stable CMS with a secure plugin registry](https://blog.cloudflare.com/emdash-cms-plugin-registry/)

## WordPressとの比較

- 似ている概念は残る — 投稿タイプはコレクション、カスタムフィールドは型付きフィールド、カテゴリとタグはタクソノミー、メニュー、メディア、リビジョンがある
- テーマ階層は無い — `single.php` に相当するのは `src/pages/` のルートファイルで、URL はファイル配置で自分で決める
- 管理画面は `wp-admin` の見た目の複製ではない — Gutenberg のブロックは Portable Text になり、画面はサイトに定義したコレクションに従う
- WordPress の PHP テーマとプラグインはそのまま動かない — 持ち込めるのはコンテンツで、見た目と独自機能は Astro と EmDash 向けに書き直す
- コンテンツの取り込みは管理画面の Import WordPress から — WXR（ツール → エクスポート）か、WordPress 側に EmDash Exporter を入れてアプリケーションパスワードでつなぐ
- 公開ステータスは `publish` だけが EmDash の published になる — draft、pending、private、future、trash は下書きになる
- 入稿基盤は別サービスのドメインに分けず、サイトと同じプロセスが管理画面を出す
- 置き換えではなく、概念が近い別の CMS として読む — EC、ページビルダー、既存プラグイン資産を前提にしたサイトは移行コストが大きい

| 観点 | WordPress（自前インストール） | EmDash |
|---|---|---|
| 実行環境 | PHP とデータベースを公開サーバーに置く構成が多い | Astro と TypeScript。DB は Cloudflare D1、SQLite、libSQL、PostgreSQL |
| 画面 | テーマ（PHP テンプレート） | Astro コンポーネント。実行時にテーマを差し替える仕組みは無い |
| 管理画面 | `/wp-admin/`、`/wp-login.php` | `/_emdash/admin/`。同じデプロイ |
| 本文 | HTML。ブロックはコメント付き HTML | Portable Text（構造化 JSON） |
| 反映 | 保存後、同一サーバーで即時 | SSR は次のリクエスト。プリレンダーは再ビルドまで静的 |
| 拡張 | プラグインが PHP プロセスと DB を共有する | サンドボックスプラグインは宣言した権限だけ。ネイティブプラグインは同一プロセス |
| ライセンス | GPL。ソフトウェアは 0 円 | MIT。ソフトウェアは 0 円 |

```astro
---
import { getEmDashCollection } from "emdash";

const { entries: posts, error } = await getEmDashCollection("posts", {
  orderBy: { published_at: "desc" },
  limit: 10,
});
if (error) return new Response("Could not load posts", { status: 500 });
---
```

> 参照: [EmDash for WordPress Developers](https://docs.emdashcms.com/coming-from/wordpress/) / [Why EmDash](https://docs.emdashcms.com/why-emdash/)

## 維持費

- 比べる単位は CMS の月額だけにしない — 配信、データベース、メディア、更新に使う時間を同じ表に載せる
- 以下の金額は 2026年10月時点の公開料金 — Cloudflare は米ドル。円換算はしない
- 制作費、デザイン、WordPress からの画面の作り直しは含まない — ここにあるのは運用で毎月かかる分と、自分の手元に残る作業
- ドメインはどちらも別契約になりやすい — 月額の差の本体にはなりにくい

### 料金が乗る場所

| 費用 | WordPress（自前インストール） | EmDash |
|---|---|---|
| CMS の利用料 | 0 円（GPL） | 0 円（MIT） |
| サーバー・配信 | 共有レンタルサーバーが要る。PHP と MySQL の世代を自分で合わせる | Cloudflare Workers に載せるか、Node.js 用のサーバーを自分で持つ |
| Workers の下限 | — | 無料プランは 1 日 **10 万** リクエスト、CPU はリクエストあたり **10ms**。有料は月額 **5 ドル**からで、月 **1,000 万** リクエストと **3,000 万** CPU ミリ秒が含まれる |
| データベース | レンタルサーバーの MySQL に含まれることが多い | D1 の無料枠は保存 **5GB**、行読み取り 1 日 **500 万**、行書き込み 1 日 **10 万**。有料は先頭 5GB が含まれ、超過は **0.75 ドル / GB・月** |
| メディア | 同じサーバーのディスク | R2 の無料枠は保存 **10GB・月**、Class A **100 万**、Class B **1,000 万**。超過分の保存は **0.015 ドル / GB・月**。egress は無料 |
| プラグインをレジストリから入れる | 追加のプランは不要。有料プラグインは別 | Cloudflare では Worker Loader が要り、Workers Paid が条件。Node.js では `workerd` を自分のサーバーで動かす |
| バックアップ | サーバー機能かプラグイン。世代管理は自分 | JSON の定期書き出しは設定できるが、復元には使えない。復旧は D1 の Time Travel、SQL ダンプ、SQLite のファイルコピーと、メディアの別コピー |
| 更新 | コア、プラグイン、テーマ、PHP | npm の依存とデプロイ。スキーマ変更と破壊的なフィールド削除は自分でリハーサルする |

- ソフトウェアが無料であることと、運用が安いことは別 — WordPress の更新漏れと同じように、EmDash もデプロイ、マイグレーション、バックアップの時間は残る
- レジストリのプラグインを Cloudflare で使うなら、下限は Workers Paid の月額 **5 ドル** — 無料枠だけで本番を賄えるかは、SSR が **10ms** の CPU に収まるかの実測が先
- Node.js に載せる場合の月額は、選んだ VPS かレンタルサーバーの料金 — D1 と R2 の無料枠は使わない
- 暗号化されたプラグイン設定を戻すには `EMDASH_ENCRYPTION_KEY` のローテーション一覧を別に保管する — データベースのダンプには入らない

> 参照: [Cloudflare Workers の料金](https://developers.cloudflare.com/workers/platform/pricing/) / [Cloudflare D1 の料金](https://developers.cloudflare.com/d1/platform/pricing/) / [Cloudflare R2 の料金](https://developers.cloudflare.com/r2/pricing/) / [Backups and recovery](https://docs.emdashcms.com/guides/backups/) / [Configure the plugin sandbox](https://docs.emdashcms.com/deployment/plugin-sandbox/)

## セキュリティ

- 攻撃面は、インターネットから届くリクエストがコードを実行する場所 — EmDash でも管理画面は公開オリジンの `/_emdash/` に残る
- WordPress と違うのは PHP が無いことと、サンドボックスプラグインがデータベース全体を最初から持たないこと
- WordPress のプラグインは同じ PHP プロセスで DB、ファイル、ネットワークに届きうる — 脆弱性の多くがプラグイン側に出る、という前提で EmDash は権限を分けている
- サンドボックスプラグインは、自分の KV とストレージ以外はマニフェストの capability が無いと API が渡されない — `content:read` が無ければコンテンツ API は無い。`network:request` は `allowedHosts` のホストに限る
- インストール前に、レジストリがその capability を運営者へ見せる — 承認した権限だけが実行時に通る
- ネイティブプラグインはこの隔離の外 — Astro と同じプロセスで動き、React の管理画面や公開 HTML の差し込みができる。信頼してデプロイするコードとして扱う
- `sandbox: false` はデバッグ用 — 本番に置くと、プラグインがサーバープロセスの権限を受け取る。Cloudflare Workers ではこの設定自体が起動を拒否する
- ログインの標準はパスキー（WebAuthn） — パスワードをネットワークに送らない。GitHub、Google、Microsoft のログインを追加できる。メールのマジックリンクは **15 分**・単回
- ロールは Subscriber、Contributor、Author、Editor、Admin の 5 つ — ナビを隠すことではなく、サーバーが操作ごとに権限を見る
- Cloudflare では、本番の保護ルートを Cloudflare Access に寄せるモードがある — その場合、EmDash のログイン画面ではなく Access が認証の主体になる
- 残る作業は、アカウント、デプロイの秘密情報、データベースとメディアの復旧点、ネイティブプラグインのレビュー — CMS を替えても、この境界は自分のまま

```jsonc
{
  "slug": "plugin-hello",
  "capabilities": ["content:read", "network:request"],
  "allowedHosts": ["api.example.com"]
}
```

> 参照: [Capabilities and security](https://docs.emdashcms.com/plugins/creating-plugins/capabilities/) / [Configure the plugin sandbox](https://docs.emdashcms.com/deployment/plugin-sandbox/) / [Authentication](https://docs.emdashcms.com/guides/authentication/) / [Admin panel](https://docs.emdashcms.com/concepts/admin-panel/)

## 編集のしやすさ

- 編集者の作業場所は管理画面 — コレクションから生成されたフォームで、下書き、公開、メディア、タクソノミー、メニュー、ウィジェット領域を権限の範囲で触る
- 短いテキストは入力欄、`portableText` はリッチテキスト、画像フィールドはメディアピッカー、参照フィールドは別エントリの選択になる
- リッチテキストで扱えるのは見出し、リスト、引用、コード、リンク、画像、登録済みのカスタムブロック
- 下書きを有効にしたコレクションでは、保存と公開が分かれる — 保存は下書き、Publish で公開クエリに出る。予約公開もできる
- 編集ロックを有効にすると、開いているあいだ他の書き手は上書きできない — 読み込み後に誰かが更新していた保存も拒否され、新しい版を読み直す
- メディアはグリッドかリスト、検索、差し替え、複数選択 — 対応するストレージでは、ブラウザが短命の署名付き URL で直接アップロードする
- メニュー項目やウィジェットを管理画面で足しても、Astro 側が `getMenu()` や `WidgetArea` で描かなければ公開ページは変わらない
- 見た目の自由配置はページビルダーではない — 編集者が並べ替えられるのは、開発者が定義したブロックまで
- ロールの目安は、Author が自分のエントリの編集と公開、Editor が他人のコンテンツとメニュー、Admin がコンテンツモデル、ユーザー、プラグイン、設定

> 参照: [Admin panel](https://docs.emdashcms.com/concepts/admin-panel/) / [Build pages with blocks](https://docs.emdashcms.com/guides/blocks/)

## カスタマイズのしやすさ

- 公開面のカスタマイズは Astro のコード — `src/pages/`、`src/layouts/`、`src/components/` を直接編集する。テーマの実行時切り替えは無い
- 公式テンプレートは `create-astro` のひな形 — blog、starter、portfolio、marketing があり、Node 用と Cloudflare 用が分かれている。生成後のファイルはサイトの所有物になる
- コンテンツモデルは管理画面の Content Types でコレクションとフィールドを足せる — 開発者はシードファイルへ書き出し、型定義を生成できる
- ローカル開発ではモデル変更時に `emdash-env.d.ts` が更新される — `emdash types` を使う流れでは、スキーマ変更のあとに再生成する
- ブロックは、開発者が型と Astro コンポーネントを用意し、編集者が順番を決める — Hero や機能グリッドのような部品を `blocks` フィールドに登録する形
- フィールド削除は全エントリから値を消す。コレクション削除はその中身ごと消える — 本番の破壊的変更の前にバックアップとリハーサルが要る
- 管理画面の画面追加や公開 HTML への差し込みはネイティブプラグイン — サンドボックスプラグインは管理画面のページ、設定、API、フックまでに留まりやすい
- 型がクエリからテンプレートまで通るのが、WordPress の PHP テーマとの差 — 画面の自由度はコードを書けるかどうかで決まる

> 参照: [Themes](https://docs.emdashcms.com/themes/overview/) / [Build pages with blocks](https://docs.emdashcms.com/guides/blocks/) / [Why EmDash](https://docs.emdashcms.com/why-emdash/)

## プラグインの有無

- プラグインはある — 1.0 でレジストリが公開され、管理画面から探してインストールできる
- 形式は 2 つ — サンドボックス（隔離ランタイム、レジストリから入れられる）とネイティブ（npm 経由で `astro.config` に書き、デプロイが要る）
- サンドボックスができないことのうち、React の管理画面、Portable Text の描画コンポーネント、公開ページへの HTML 差し込みはネイティブ側
- レジストリはカタログとパッケージの所有者を分けている — 配布記録は作者の Atmosphere アカウント（AT Protocol）にあり、EmDash は検索とインストールの窓口
- インストール時に、署名、チェックサム、パッケージ名、バージョン、要求する権限を突き合わせる — カタログに載った説明文をそのまま信用する仕組みではない
- 現時点のレジストリは無料プラグイン — 有料の販売は今後の予定として書かれている
- WordPress.org のような数と年数は無い — Cloudflare の 1.0 発表では、Lexington Themes の EmDash 対応テーマや、Urumi による EC プラグインがエコシステムの例として挙がっている
- WordPress プラグインの移植は、フックと API の対応を自分で書く作業 — 公式は概念の対応と、エージェント向けの移植手順をドキュメントに置いている
- 入れたあとも更新は自分のサイトの操作 — サンドボックスはランナーが無いと `SANDBOX_NOT_AVAILABLE` でインストールできない

> 参照: [Plugins](https://docs.emdashcms.com/plugins/overview/) / [Plugin registry（EmDash 1.0 発表）](https://blog.cloudflare.com/emdash-cms-plugin-registry/) / [Porting WordPress Plugins](https://docs.emdashcms.com/migration/porting-plugins/)

## まとめ

- EmDash は Astro と同じデプロイで動く CMS で、ソフトウェアは MIT の 0 円
- WordPress の投稿、タクソノミー、メニュー、メディアに近い概念はあるが、PHP テーマとプラグインはそのまま動かない
- コンテンツは WXR か Exporter で移せる。画面と独自機能は作り直し
- 維持費の本体はホスティングと運用時間 — Cloudflare の無料枠から始められる余地はあるが、レジストリのプラグインは Workers Paid（月額 **5 ドル**から）が要る
- 復旧は JSON 書き出しではなく、データベースの時点復旧とメディアの別コピー
- セキュリティの差は、PHP が無いことと、サンドボックスプラグインの capability — ネイティブプラグインと管理画面そのものは自分で守る
- 編集はフォームと Portable Text、下書きと公開の分離、予約、編集ロック — 自由な見た目の組み立ては、定義済みブロックの並べ替えまで
- カスタマイズは Astro のコードと型付きのコンテンツモデル — テーマの実行時切り替えは無い
- プラグインレジストリは 1.0 で始まっている。数は WordPress より少なく、必要な機能がプラグインで足りるかは移行前に確認する
