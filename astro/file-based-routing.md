# ファイルベースルーティングの基本

## ファイルベースルーティングとは

- `src/pages/` 内のファイルパスがそのまま URL になる仕組み
- ルート定義ファイル（React Router の routes 配列など）を別途書かなくてよい
- Astro のデフォルトのページ生成方式
- 1ファイル = 1ルート（動的ルートを除く）という対応が基本

> 参照: [Astro — Routing](https://docs.astro.build/en/guides/routing/)

## 基本ルール

| ファイル | URL |
|---|---|
| `src/pages/index.astro` | `/` |
| `src/pages/about.astro` | `/about` |
| `src/pages/contact.astro` | `/contact` |

- 拡子子 `.astro` は URL に含まれない
- ファイル名がパスセグメントになる

```text
src/pages/
├── index.astro       → /
├── about.astro       → /about
└── team.astro        → /team
```

## ネスト（ディレクトリ）

- サブディレクトリは URL の `/` に対応
- フォルダ階層 = パス階層

```text
src/pages/
├── index.astro           → /
├── blog/
│   ├── index.astro       → /blog
│   └── first-post.astro  → /blog/first-post
└── docs/
    └── getting-started.astro → /docs/getting-started
```

```text
URL: /blog/first-post
     │    └─ first-post.astro
     └─ blog/ ディレクトリ
```

- `index.astro` はそのディレクトリの **インデックスページ**（末尾 `/` の有無は設定・ホスト依存）

> 参照: [Astro — Pages](https://docs.astro.build/en/basics/astro-pages/)

## index.astro の役割

- ディレクトリの「トップ」を担当
- `src/pages/index.astro` → サイトトップ `/`
- `src/pages/blog/index.astro` → ブログ一覧 `/blog`

```astro
---
// src/pages/blog/index.astro
const posts = [
  { slug: 'hello', title: 'Hello' },
  { slug: 'world', title: 'World' },
];
---

<h1>ブログ一覧</h1>
<ul>
  {posts.map((post) => (
    <li><a href={`/blog/${post.slug}`}>{post.title}</a></li>
  ))}
</ul>
```

- 一覧ページと詳細ページ（`[slug].astro`）をセットで使うパターンが多い

## 対応ファイル形式

| 拡張子 | 用途 |
|---|---|
| `.astro` | 通常のページ |
| `.md` / `.mdx` | Markdown ページ |
| `.html` | 静的 HTML |
| `.js` / `.ts` | API エンドポイント |

- Markdown ページにもレイアウトを指定できる（[レイアウトコンポーネント](./layout-components.md) 参照）

## `_` プレフィックス — ルートから除外

- ファイル / ディレクトリ名が `_` で始まるものは **ルーター対象外**
- ビルド出力（`dist/`）にも含まれない
- 部分テンプレートやルート生成用のヘルパーを pages 近くに置きたいときに使う

```text
src/pages/
├── index.astro
├── blog/
│   ├── index.astro
│   ├── [slug].astro
│   └── _draft-template.astro   ← URL にならない
└── _components/                ← ディレクトリごと除外
    └── Card.astro
```

```text
<!-- 悪い例：pages 内に URL 不要な部品を _ なしで置く -->
<!-- src/pages/partials/Header.astro → /partials/Header になってしまう -->
```

- 再利用 UI は原則 `src/components/` に置く方が分かりやすい
- `_` は「pages 配下だけどルートにしたくない」例外用途

> 参照: [Astro — Excluding pages](https://docs.astro.build/en/guides/routing/#excluding-pages)

## 末尾スラッシュ

- `trailingSlash` 設定（`astro.config.mjs`）で `/about` と `/about/` の扱いを制御
- ホスティング先（Netlify、Cloudflare Pages など）のリダイレクト設定とも整合を取る

```js
// astro.config.mjs
export default defineConfig({
  trailingSlash: 'always', // /about/ を正とする
});
```

## 404 ページ

- `src/pages/404.astro` または `src/pages/404.md` でカスタム 404
- 静的ホストでは `404.html` として出力される

## 静的エンドポイント（API Routes）

- `.js` / `.ts` ファイルで JSON などを返すエンドポイントを定義できる
- SSG 時はビルド時に生成、SSR 時はリクエストごとに実行

```ts
// src/pages/api/hello.ts
import type { APIRoute } from 'astro';

export const GET: APIRoute = () => {
  return new Response(
    JSON.stringify({ message: 'Hello' }),
    { headers: { 'Content-Type': 'application/json' } },
  );
};
```

- URL: `/api/hello`
- 動的パラメータ `[id].ts` や `getStaticPaths` も pages と同様に使える

```ts
// src/pages/api/users/[id].ts
import type { APIRoute } from 'astro';

export const GET: APIRoute = ({ params }) => {
  return new Response(JSON.stringify({ id: params.id }));
};
```

> 参照: [Astro — Endpoints](https://docs.astro.build/en/guides/endpoints/)

## ルーティングと .astro ファイルの関係

```astro
---
// src/pages/about.astro
// このファイルが /about のエントリ
import Layout from '../layouts/Layout.astro';
---

<Layout title="About">
  <h1>About us</h1>
</Layout>
```

- pages 内の `.astro` は通常 [レイアウト](./layout-components.md) でラップする
- コンポーネントスクリプトで `Astro.url` を参照すると現在の URL 情報が取れる

```astro
---
const pathname = Astro.url.pathname; // "/about"
---
```

## 動的ルートとの境界

- 固定パス → 本記事のファイルベースルーティング
- `[slug].astro` など → [動的ルーティングと getStaticPaths](./dynamic-routing-getstaticpaths.md)

| パターン | 例 | 記事 |
|---|---|---|
| 固定 | `about.astro` | 本記事 |
| 動的1段 | `[slug].astro` | 動的ルーティング |
| キャッチオール | `[...slug].astro` | 動的ルーティング |

## よくある間違い

### 1. pages 以外に置いて URL にならない

- `src/components/About.astro` は `/about` にならない
- 公開 URL が必要なら `src/pages/about.astro`

### 2. 大文字ファイル名

- `About.astro` は `/About` になる（ホストによっては大小文字を区別）
- ケバブケース小文字（`about.astro`）を推奨

### 3. リンクのパスを間違える

```astro
<!-- 悪い例：ファイルパスをそのまま href に -->
<a href="src/pages/about.astro">About</a>

<!-- 良い例：URL パス -->
<a href="/about">About</a>
```

### 4. index の有無を忘れる

- `/blog` に一覧を出したい → `blog/index.astro` が必要
- `blog.astro` だけだと `/blog` にはならず `/blog` という **ファイル名** のルートにもならない（`blog.astro` → `/blog` だが `blog/` ディレクトリ構造とは別）

## ルーティング設計のヒント

- URL 設計を先に決め、フォルダ構造を後から合わせる
- 一覧（`index`）と詳細（`[slug]`）は同じディレクトリにまとめる
- API は `src/pages/api/` 以下に集約
- ルートにしないファイルは `components` か `_` プレフィックス

## 学習の進め方（このリポジトリ内）

1. [プロジェクト構成](./project-structure.md)
2. ファイルベースルーティング（本記事）
3. [動的ルーティングと getStaticPaths](./dynamic-routing-getstaticpaths.md)
4. [レイアウトコンポーネント](./layout-components.md)

## まとめ

- `src/pages/` のファイルパスが URL になるファイルベースルーティング
- ディレクトリネスト = パスネスト、`index.astro` がディレクトリのトップ
- `_` プレフィックスでルート生成から除外できる
- `.ts` / `.js` で静的 / 動的 API エンドポイントも定義可能
- 可変パスは `[slug].astro` など動的ルートで扱う
