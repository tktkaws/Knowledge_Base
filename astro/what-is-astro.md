# Astroとは何か — コンテンツ重視の静的サイトフレームワーク

## Astroとは

- コンテンツ重視のウェブサイト向けに設計された Web フレームワーク
- ブログ、ドキュメント、マーケティングサイト、ポートフォリオ、LP などに向く
- デフォルトではクライアントに JavaScript を送らず、HTML を中心に配信する
- 必要になった部品だけを「アイランド」としてハイドレーションできる

> 参照: [Astro — Why Astro?](https://docs.astro.build/en/concepts/why-astro/)

## 何が得意か

| 向くサイト | 例 |
|---|---|
| コンテンツを速く届けたい | ブログ、ニュース、ドキュメント |
| 公開情報が中心 | コーポレート、ポートフォリオ、LP |
| 一部だけインタラクティブ | 検索UI、コメント、カートの一部 |

| 向くとは限らないサイト | 例 |
|---|---|
| アプリ的な体験が主 | 管理画面、SNS、複雑なダッシュボード |
| 画面遷移なしの高度なクライアント状態 | Figma 的なツール、常時同期するUI |

- Astro はコンテンツ配信のパフォーマンスを優先する
- アプリ全体がクライアント状態で動くなら Next.js / Remix などの方が合うことが多い

## コアとなる考え方

### 1. Zero JS by default

- UIコンポーネントは、基本的に HTML / CSS として出力される
- クライアントJSは、明示的に付けたときだけ送られる
- 「気づいたらバンドルが重い」状態を防ぎやすい

### 2. Server-first / MPA 寄り

- できるだけサーバー側（ビルド時またはリクエスト時）でHTMLを作る
- 伝統的なサーバーサイドの考え方に近いが、言語は HTML / CSS / JS（TS）のまま
- SPA のように最初から全部をクライアントで描画する前提ではない

### 3. UI-agnostic（UIフレームワーク非依存）

- React / Vue / Svelte / Solid などを同じプロジェクトで使える
- 「全部 React で書く」必要はない
- 静的な部分は `.astro`、インタラクティブな部分だけ React など、という使い方ができる

### 4. Islands Architecture

- ページ全体をSPA化するのではなく、必要な島（コンポーネント）だけを動かす
- 島以外は静的HTMLのまま
- 詳細は別記事「Astroのアイランドアーキテクチャ」で扱う

> 参照: [Astro — Islands architecture](https://docs.astro.build/en/concepts/islands/)

## Next.js などとの違い（ざっくり）

| 観点 | Astro | Next.js など |
|---|---|---|
| 主戦場 | コンテンツサイト | Webアプリ / 高インタラクション |
| デフォルト | 静的HTML寄り、JS少なめ | コンポーネントとクライアントJSが中心になりやすい |
| ページモデル | MPA 寄り | SPA + SSR/SSG のハイブリッドが多い |
| UI | 複数フレームワークを混在しやすい | 通常は React 前提 |

- 「どっちが上」ではなく、サイトの性質で選ぶ
- ドキュメントサイトを最速で出したいなら Astro、会員制の複雑なアプリなら Next.js、という判断が分かりやすい

## 主な機能（全体像）

- **ファイルベースルーティング** — `src/pages` 配下のファイルがルートになる
- **レイアウト / コンポーネント** — `.astro` でページ構造を組み立てる
- **Content Collections** — Markdown などのコンテンツを型付きで管理
- **アダプター** — 静的ホストだけでなく、Node / Vercel / Cloudflare などへSSRも可能
- **インテグレーション** — React、MDX、Sitemap、Partytown などを追加しやすい

## 最小のイメージ

```astro
---
// フロントマター（サーバー側で動く）
const title = 'Hello Astro';
const posts = ['はじめてのAstro', 'Islandsとは'];
---

<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <title>{title}</title>
  </head>
  <body>
    <h1>{title}</h1>
    <ul>
      {posts.map((post) => <li>{post}</li>)}
    </ul>
  </body>
</html>
```

- `---` で囲んだ部分はビルド時 / サーバー側で実行される
- テンプレート部分が HTML として出力される
- このままだとクライアントJSは増えない

```astro
---
import LikeButton from '../components/LikeButton.jsx';
---

<!-- 必要な部品だけクライアントで動かす -->
<LikeButton client:load />
```

## プロジェクトの始め方

```bash
npm create astro@latest
```

```bash
cd プロジェクト名
npm run dev
```

- ウィザードでテンプレートや TypeScript、Git 初期化を選べる
- 公式テンプレートを指定することもできる

```bash
npm create astro@latest -- --template minimal
```

> 参照: [Astro — Install and setup](https://docs.astro.build/en/install-and-setup/)

## どんなときにAstroを選ぶか

### 選ぶとよい例

- ブログやドキュメントを高速に公開したい
- 既存の React コンポーネントを一部だけ再利用したい
- Markdown 中心の執筆フローにしたい
- 初期表示の速さを最優先したい

### 別案を検討する例

- ログイン後の画面が多く、画面内状態が複雑
- リアルタイム更新や高度なクライアントルーティングが主機能
- チーム全体が App Router 前提の React に統一されている

## 学習の進め方（このリポジトリ内）

1. Astroとは何か（本記事）
2. アイランドアーキテクチャ
3. `.astro` ファイルの構造
4. プロジェクト構成（`pages` / `components` / `layouts`）
5. ルーティング、コンポーネント、Content Collections へ進む

## まとめ

- Astro はコンテンツを速く届けるためのフレームワーク
- デフォルトで JS を送らず、必要な島だけをハイドレーションする
- ブログ・ドキュメント・LP に強く、アプリ全体の複雑UIには向かないこともある
- React などを部分採用できるため、既存資産とも共存しやすい
- 「速く読ませるサイト」を作るとき、まず候補に入れる価値がある
