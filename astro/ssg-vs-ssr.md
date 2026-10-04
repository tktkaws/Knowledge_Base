# 静的サイト生成(SSG)とサーバーサイドレンダリング(SSR)

## SSG と SSR とは

- **SSG（Static Site Generation）** — ビルド時に HTML を生成し、CDN 等から配信
- **SSR（Server-Side Rendering）** — リクエストごとにサーバーで HTML を生成して返す
- Astro はデフォルトで SSG 寄り（`output: 'static'`）
- 必要なページだけ SSR に切り替えるハイブリッド運用も可能

> 参照: [Astro — On-demand rendering](https://docs.astro.build/en/guides/on-demand-rendering/)

## Astro の output 設定（現行）

- Astro 5 以降、`output` は **`'static'`** と **`'server'`** の2種類
- 旧 `'hybrid'` は廃止され、`'static'` に統合された
- ハイブリッド＝「基本 static、必要なページだけ SSR」は `prerender` フラグで制御

| output | デフォルト動作 |
|---|---|
| `'static'`（デフォルト） | 全ページをビルド時にプリレンダー |
| `'server'` | 全ページをリクエスト時にサーバー描画 |

> 参照: [Astro — output configuration](https://docs.astro.build/en/reference/configuration-reference/#output)

## output: 'static'（SSG / ハイブリッド）

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  output: 'static', // 省略可能（デフォルト）
});
```

- ビルド時に `dist/` へ静的 HTML が出力される
- ホスティング先は CDN・オブジェクトストレージで足りることが多い
- **一部だけ SSR にしたい** ページは `export const prerender = false` を指定

```astro
---
// src/pages/dashboard.astro
export const prerender = false; // このページだけ SSR

const user = Astro.locals.user;
---
<h1>こんにちは、{user?.name}</h1>
```

- SSR ページを使うには **アダプター** の追加が必要
- Astro v4 の `output: 'hybrid'` は v5 では `'static'` + `prerender = false` に読み替える

> 参照: [Astro — Removed hybrid mode (v5)](https://docs.astro.build/en/guides/upgrade-to/v5/#removed-hybrid-rendering-mode)

## output: 'server'（フル SSR）

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'server',
  adapter: node({ mode: 'standalone' }),
});
```

- すべてのページがデフォルトで SSR
- 静的にしてよいページ（About、プライバシーポリシーなど）は `prerender = true`

```astro
---
// src/pages/about.astro
export const prerender = true; // ビルド時に静的 HTML 化
---
<h1>About</h1>
```

- 高頻度更新・認証・パーソナライズが**サイトの大半**なら `'server'` を検討
- 最初は `'static'` から始め、SSR が必要なページだけ opt-out する方が安全、という公式の推奨もある

> 参照: [Astro — server mode](https://docs.astro.build/en/guides/on-demand-rendering/#server-mode)

## prerender フラグの整理

| 設定 | `output: 'static'` | `output: 'server'` |
|---|---|---|
| デフォルト | プリレンダー（SSG） | SSR |
| `export const prerender = false` | SSR（要アダプター） | （デフォルトと同じ） |
| `export const prerender = true` | （デフォルトと同じ） | SSG |

```astro
---
// API ルートも同様
export const prerender = false;

export async function GET() {
  return new Response(JSON.stringify({ time: Date.now() }));
}
---
```

- ページ（`.astro`）と API エンドポイント（`src/pages/api/`）の両方に指定可能

> 参照: [Astro — Prerender configuration](https://docs.astro.build/en/guides/on-demand-rendering/)

## アダプターとは

- SSR やサーバーレス関数を動かすための**デプロイ先向けプラグイン**
- `output: 'static'` でも `prerender = false` のページがある場合に必要
- 静的サイトのみならアダプター不要

| アダプター | 主なデプロイ先 |
|---|---|
| `@astrojs/vercel` | Vercel |
| `@astrojs/netlify` | Netlify |
| `@astrojs/cloudflare` | Cloudflare Workers / Pages |
| `@astrojs/node` | Node.js サーバー |

```bash
npx astro add vercel
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import vercel from '@astrojs/vercel';

export default defineConfig({
  output: 'static',
  adapter: vercel(),
});
```

- `astro add` でアダプターと設定が追加される
- デプロイ先に合わせて1つ選ぶ

> 参照: [Astro — Adapters](https://docs.astro.build/en/guides/deploy/)

## いつ SSG で足りるか

- ブログ、ドキュメント、コーポレートサイト
- 全ユーザーに同じ HTML でよい
- ビルド時にコンテンツが確定している
- 最速の配信と低コストを優先したい

```js
// 追加設定なしで SSG
export default defineConfig({});
```

- Astro の主戦場
- Vercel / Netlify / Cloudflare Pages への静的デプロイが簡単

## いつ SSR が必要か

- **リクエストごとに内容が変わる** — ログインユーザー名、権限別 UI
- **リアルタイムデータ** — 在庫、為替、ダッシュボード
- **サーバー側でのみ取得できる情報** — 内部 API、DB 直結
- **フォーム送信後の動的レスポンス** — サーバーアクション、認証フロー
- **パーソナライズ** — A/B テスト、地域別コンテンツ

```astro
---
// 悪い例：クライアント fetch だけで機密データを扱う
// SSR でサーバー側取得 + HTML 埋め込みの方が安全なことも
export const prerender = false;

const data = await fetch('https://internal-api.example/items', {
  headers: { Authorization: `Bearer ${import.meta.env.API_TOKEN}` },
}).then((r) => r.json());
---
<ul>
  {data.map((item) => <li>{item.name}</li>)}
</ul>
```

- 秘密鍵は `import.meta.env` 経由でサーバー側のみ参照
- 静的 HTML に埋め込むと漏洩リスクがあるため、公開データかどうかを確認

## SSG vs SSR の比較

| 観点 | SSG | SSR |
|---|---|---|
| 生成タイミング | ビルド時 | リクエスト時 |
| 配信 | CDN 向き | サーバー / サーバーレス必須 |
| TTFB | 一般に速い | サーバー処理分だけ遅れうる |
| 動的データ | ビルド時点で固定 | リクエスト時に最新 |
| コスト | 低い（静的ホスト） | 実行時間・リクエスト課金 |
| 複雑さ | 低い | アダプター・ランタイム管理 |

## 設計の進め方（推奨）

1. まず `output: 'static'`（デフォルト）で始める
2. SSR が必要なページだけ `export const prerender = false`
3. 必要になったらアダプターを追加
4. サイトの大半が SSR 必要なら `output: 'server'` を検討

```astro
<!-- 良い例：静的が大半、ダッシュボードだけ SSR -->
<!-- src/pages/index.astro → デフォルト SSG -->
<!-- src/pages/blog/[slug].astro → getStaticPaths で SSG -->
<!-- src/pages/account.astro → prerender = false -->
```

```js
// 悪い例：最初から全ページ server モード
export default defineConfig({
  output: 'server',
  adapter: vercel(),
});
// 静的で足りる About / Blog まで毎回サーバー描画
```

## よくある間違い

### 1. hybrid 用語だけで v5 を設定する

```js
// 悪い例（Astro v5 では無効）
export default defineConfig({
  output: 'hybrid',
});
```

- v5 では `'static'` + `prerender = false` に移行

### 2. SSR ページがあるのにアダプター未設定

- ビルドまたはデプロイでエラー
- `astro add <adapter>` を実行

### 3. 全部 SSR にしてパフォーマンスを落とす

- コンテンツサイトでは SSG の恩恵が大きい
- 本当に動的なページだけ opt-out

## まとめ

- Astro のデフォルトは SSG（`output: 'static'`）
- v5 以降のハイブリッドは `'static'` + `export const prerender = false` で実現
- フル SSR は `output: 'server'` + アダプター、静的ページは `prerender = true`
- ブログ・ドキュメントは SSG、認証・動的データは SSR が目安
- まず static から始め、必要なページだけ SSR に広げるのが現実的
