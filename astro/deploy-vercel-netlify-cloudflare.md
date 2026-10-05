# Vercel / Netlify / Cloudflare Pagesへのデプロイ

## 概要

- Astro サイトは静的（SSG）がデフォルトで、3プラットフォームすべてにデプロイ可能
- 静的デプロイならアダプター不要、`npm run build` の `dist/` を公開するだけ
- SSR やサーバーレス API が必要なら各プラットフォーム向け **アダプター** を追加
- ビルドコマンド・公開ディレクトリ・環境変数の設定が要点

> 参照: [Astro — Deploy your site](https://docs.astro.build/en/guides/deploy/)

## 静的デプロイの共通設定

| 項目 | 一般的な値 |
|---|---|
| ビルドコマンド | `npm run build` |
| 公開ディレクトリ | `dist` |
| Node バージョン | 18 以上（プロジェクトに合わせる） |

```json
// package.json
{
  "scripts": {
    "build": "astro build",
    "preview": "astro preview"
  }
}
```

- Git 連携で push 時に自動ビルドが一般的
- ローカルで `astro build` → `dist/` 確認 → `astro preview` で本番同等確認

## Vercel

### 静的サイト（デフォルト）

- Astro プロジェクトは追加設定なしで Vercel に静的デプロイ可能
- Vercel が Astro を自動検出し、ビルド設定を提案することが多い

| 設定 | 値 |
|---|---|
| Build Command | `npm run build` |
| Output Directory | `dist` |
| Install Command | `npm install` |

> 参照: [Astro — Deploy to Vercel (static)](https://docs.astro.build/en/guides/deploy/vercel/#project-configuration)

### SSR / サーバーレス（アダプター）

```bash
npx astro add vercel
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import vercel from '@astrojs/vercel';

export default defineConfig({
  output: 'static', // または 'server'
  adapter: vercel(),
});
```

- `prerender = false` のページや API ルートを Vercel Functions 上で実行
- Vercel ダッシュボードまたは CLI で環境変数を設定

```bash
# ローカル開発用に Vercel の env を pull
vercel env pull .env.local
```

- `.env.local` は gitignore 対象
- `import.meta.env.PUBLIC_*` がクライアント公開、`import.meta.env.*`（PUBLIC なし）はサーバー専用

> 参照: [Astro — Vercel adapter](https://docs.astro.build/en/guides/integrations-guide/vercel/)

### CLI デプロイ

```bash
npm run build
vercel deploy --prebuilt
```

- ビルド済み成果物を直接アップロード
- CI パイプラインで使う場合向け

## Netlify

### 静的サイト

```toml
# netlify.toml（プロジェクトルート）
[build]
  command = "npm run build"
  publish = "dist"
```

- Netlify UI でも同等の設定が可能
- `netlify.toml` をリポジトリに置くと設定がコード化される

> 参照: [Astro — Deploy to Netlify](https://docs.astro.build/en/guides/deploy/netlify/)

### SSR（アダプター）

```bash
npx astro add netlify
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import netlify from '@astrojs/netlify';

export default defineConfig({
  output: 'static',
  adapter: netlify(),
});
```

- Netlify Functions / Edge Functions として SSR ページを実行
- 環境変数は Netlify ダッシュボードの **Site settings → Environment variables**

| スコープ | 用途 |
|---|---|
| Production | 本番ビルド・本番実行 |
| Deploy previews | PR プレビュー |
| Branch deploys | ブランチ別デプロイ |

> 参照: [Astro — Netlify adapter](https://docs.astro.build/en/guides/integrations-guide/netlify/)

## Cloudflare Pages

### 静的サイト

- Cloudflare Pages ダッシュボードから Git 連携
- ビルド設定を手動入力

| 設定 | 値 |
|---|---|
| Framework preset | Astro（または None） |
| Build command | `npm run build` |
| Build output directory | `dist` |

```jsonc
// wrangler.jsonc（静的サイト向け・Pages 直接デプロイ時）
{
  "name": "my-astro-app",
  "compatibility_date": "2026-01-01",
  "assets": {
    "directory": "./dist"
  }
}
```

- `compatibility_date` はデプロイ日付付近に更新
- Wrangler CLI から `wrangler pages deploy dist` も可能

> 参照: [Astro — Deploy to Cloudflare](https://docs.astro.build/en/guides/deploy/cloudflare/)

### SSR（Cloudflare Workers アダプター）

```bash
npx astro add cloudflare
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  output: 'server',
  adapter: cloudflare(),
});
```

```jsonc
// wrangler.jsonc — 環境変数（非機密）
{
  "vars": {
    "PUBLIC_API_URL": "https://api.example.com"
  }
}
```

- 機密値は Cloudflare ダッシュボードの **Variables and Secrets** で管理
- Workers / Pages Functions 上で SSR

> 参照: [Astro — Cloudflare adapter](https://docs.astro.build/en/guides/integrations-guide/cloudflare/)

## 静的 vs SSR — デプロイの選び方

| 構成 | アダプター | 向く例 |
|---|---|---|
| 静的のみ | 不要 | ブログ、ドキュメント、LP |
| 静的 + 一部 SSR | 各 platform adapter | ログイン後ページだけ動的 |
| フル SSR | 各 platform adapter | ダッシュボード中心 |

```js
// 良い例：静的サイトなら adapter なし
export default defineConfig({});
```

```js
// SSR が必要になってから adapter 追加
import vercel from '@astrojs/vercel';
export default defineConfig({
  adapter: vercel(),
});
```

- 不要な adapter はランタイム依存を増やす
- まず静的でデプロイし、必要ページだけ `prerender = false`

## 環境変数の基本

### Astro の env ルール

| プレフィックス | 参照場所 | 例 |
|---|---|---|
| `PUBLIC_` | クライアント + サーバー | `PUBLIC_ANALYTICS_ID` |
| なし | サーバー（ビルド / SSR）のみ | `API_SECRET` |

```astro
---
// サーバー側のみ（SSR ページ、API）
const secret = import.meta.env.API_SECRET;
---
```

```astro
<script define:vars={{ analyticsId: import.meta.env.PUBLIC_ANALYTICS_ID }}>
  // クライアントで PUBLIC_ 変数を利用
</script>
```

- 秘密鍵に `PUBLIC_` を付けない
- `.env` / `.env.local` は git に含めない

> 参照: [Astro — Environment variables](https://docs.astro.build/en/guides/environment-variables/)

### 各プラットフォームでの設定場所

| プラットフォーム | 設定場所 |
|---|---|
| Vercel | Project Settings → Environment Variables |
| Netlify | Site settings → Environment variables |
| Cloudflare | Workers/Pages → Settings → Variables |

- 本番・プレビュー・開発で値を分けられる
- ビルド時に必要な変数（`PUBLIC_*` 含む）はビルド環境にも設定

## デプロイ前の確認とトラブル

| 確認 / 症状 | 対応 |
|---|---|
| ローカル build & preview | `npm run build` → `astro preview` |
| SSR ページあり | 各 platform adapter を追加 |
| 環境変数 undefined | ホスティング側に設定、`PUBLIC_` 付け忘れに注意 |
| 404 on refresh（SSR） | adapter 未設定、rewrite ルール不足 |
| 静的 Astro の選定 | Vercel / Netlify / Cloudflare いずれも同等に簡単 |

## まとめ

- 静的 Astro は `npm run build` → `dist` 公開で3プラットフォームすべてにデプロイ可能
- SSR が必要なら `@astrojs/vercel` / `netlify` / `cloudflare` アダプターを追加
- ビルドコマンドは `npm run build`、公開ディレクトリは `dist` が基本
- 環境変数は `PUBLIC_` の有無でクライアント公開可否が決まる
- まず静的デプロイで動かし、SSR は必要になってから adapter を足す
