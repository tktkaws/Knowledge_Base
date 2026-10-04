# 動的ルーティングとgetStaticPaths

## 動的ルーティングとは

- URL の一部が可変になるルート
- ファイル名を `[param].astro` のように **角括弧** で囲んで定義
- ブログ記事 `/blog/hello-world`、商品 `/products/123` などに向く
- 1つの `.astro` テンプレートから複数 URL を生成する

> 参照: [Astro — Dynamic routes](https://docs.astro.build/en/guides/routing/#dynamic-routes)

## 基本形 — [slug].astro

```text
src/pages/blog/[slug].astro  →  /blog/:slug
```

```astro
---
// src/pages/blog/[slug].astro
export function getStaticPaths() {
  return [
    { params: { slug: 'hello-world' } },
    { params: { slug: 'second-post' } },
  ];
}

const { slug } = Astro.params;
---

<h1>記事: {slug}</h1>
```

- `getStaticPaths()` が **どの slug でページを作るか** をビルド時に宣言
- テンプレート内では `Astro.params.slug` で値を参照

## getStaticPaths の戻り値

各要素は `{ params, props? }` のオブジェクト。

| プロパティ | 意味 |
|---|---|
| `params` | URL パラメータ（`[slug]` → `{ slug: '...' }`） |
| `props` | そのページに渡す追加データ（任意） |

```astro
---
export function getStaticPaths() {
  return [
    {
      params: { slug: 'hello-world' },
      props: { title: 'Hello World', date: '2024-01-01' },
    },
    {
      params: { slug: 'second-post' },
      props: { title: 'Second Post', date: '2024-02-01' },
    },
  ];
}

const { slug } = Astro.params;
const { title, date } = Astro.props;
---

<article>
  <h1>{title}</h1>
  <time>{date}</time>
  <p>slug: {slug}</p>
</article>
```

- `params` は URL に、`props` はコンポーネントに渡る
- SSG では props でデータを渡すと、テンプレート内の再 fetch を減らせる

> 参照: [Astro — getStaticPaths](https://docs.astro.build/en/reference/routing-reference/#getstaticpaths)

## Content Collections + getStaticPaths（Astro 5）

- 実務では Markdown 記事を Content Collections で管理し、`getCollection` から paths を生成するパターンが一般的
- Astro 5 では `src/content.config.ts` + `defineCollection` + Zod

### content.config.ts

```ts
// src/content.config.ts
import { defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';
import { z } from 'astro/zod';

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.{md,mdx}' }),
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
  }),
});

export const collections = { blog };
```

### 動的ページ

```astro
---
// src/pages/blog/[slug].astro
import { getCollection, render } from 'astro:content';
import BlogLayout from '../../layouts/BlogPostLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---

<BlogLayout title={post.data.title} description={post.data.description}>
  <Content />
</BlogLayout>
```

- `post.id` が URL の slug になる（ファイル名ベース）
- `render()` で Markdown 本文を `<Content />` コンポーネントとして描画

> 参照: [Astro — Content Collections](https://docs.astro.build/en/guides/content-collections/)

## 複数パラメータ — [category]/[slug].astro

```text
src/pages/blog/[category]/[slug].astro  →  /blog/:category/:slug
```

```astro
---
export function getStaticPaths() {
  return [
    { params: { category: 'tech', slug: 'astro-intro' } },
    { params: { category: 'life', slug: 'coffee' } },
  ];
}

const { category, slug } = Astro.params;
---

<p>{category} / {slug}</p>
```

- 角括弧の数 = パスセグメント数
- `params` のキー名はファイル名と一致させる

## キャッチオール — [...slug].astro

- 残りのパスを **可変長** で受け取る
- ドキュメントサイトの階層 URL（`/docs/a/b/c`）向き

```text
src/pages/docs/[...slug].astro  →  /docs/*（任意の深さ）
```

```astro
---
export function getStaticPaths() {
  return [
    { params: { slug: 'getting-started' } },
    { params: { slug: 'guides/routing' } },
    { params: { slug: 'reference/api' } },
  ];
}

const { slug } = Astro.params;
// slug は "guides/routing" のような文字列（配列ではない）
---

<h1>Docs: {slug}</h1>
```

- `slug` は `/` 区切りの **1本の文字列**
- 存在しないパスは `getStaticPaths` に含めなければ 404

## オプショナルキャッチオール — [[...slug]].astro

- `[...slug]` に加え、**パスなし**（親ディレクトリ自体）もマッチ
- `src/pages/docs/[[...slug]].astro` → `/docs` も `/docs/a/b` も同一ファイル

```astro
---
export function getStaticPaths() {
  return [
    { params: { slug: undefined } },      // /docs
    { params: { slug: 'getting-started' } },
  ];
}
---
```

- 一覧と階層ページを1ファイルにまとめたいときの選択肢
- 設計が複雑になりやすいので、分離できるなら `index.astro` + `[...slug].astro` の方が読みやすいことも多い

> 参照: [Astro — Rest parameters](https://docs.astro.build/en/guides/routing/#rest-parameters)

## SSG 前提の動作

- Astro デフォルト（`output: 'static'`）では `getStaticPaths` が **必須**
- ビルド時に return された paths 分だけ HTML が生成される
- return に無い URL は存在しない（404）

```astro
---
// 悪い例：getStaticPaths なしの [slug].astro（SSG モード）
// → ビルドエラーまたはルート未定義
const { slug } = Astro.params;
---
```

- 外部 CMS から記事一覧を fetch して paths を返す使い方も同じ

## SSR / オンデマンドとの違い（触り）

| 観点 | SSG（静的） | SSR / オンデマンド |
|---|---|---|
| `getStaticPaths` | 必須 | **使えない** |
| パラメータ取得 | `Astro.params` | `Astro.params` |
| データ渡し | `props` が使える | `props` は使えない |
| ページ生成 | ビルド時 | リクエスト時 |

```astro
---
// SSR モード（getStaticPaths なし）
export const prerender = false;

const { slug } = Astro.params;
const post = await fetchPostBySlug(slug); // リクエストごとに取得

if (!post) {
  return Astro.redirect('/404');
}
---
<h1>{post.title}</h1>
```

- SSR では slug を使って都度データを引く
- `output: 'server'` またはページ単位の `prerender = false` で有効化
- ハイブリッド（一部だけ SSR）も可能

> 参照: [Astro — On-demand rendering](https://docs.astro.build/en/guides/on-demand-rendering/)

## params と props の使い分け

| データ | 推奨 |
|---|---|
| URL に現れる識別子 | `params` |
| 記事本文、メタ情報など | `props`（SSG）または fetch（SSR） |

```astro
---
// 悪い例：params だけ渡してテンプレート内で毎回全記事を再検索
export function getStaticPaths() {
  return [{ params: { slug: 'hello' } }];
}
const posts = await getCollection('blog');
const post = posts.find(p => p.id === Astro.params.slug);
---

// 良い例：getStaticPaths で props に post を渡す
export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }));
}
const { post } = Astro.props;
---
```

## TypeScript で型付け

```astro
---
import type { GetStaticPaths } from 'astro';
import type { CollectionEntry } from 'astro:content';

export const getStaticPaths = (async () => {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }));
}) satisfies GetStaticPaths;

interface Props {
  post: CollectionEntry<'blog'>;
}

const { post } = Astro.props;
---
```

## 404 の扱い

- `getStaticPaths` に含まれない slug → ビルド成果物に存在しない → 404
- SSR では `Astro.redirect('/404')` や `new Response(null, { status: 404 })` で制御

## よくある間違い

### 1. params のキー名不一致

```astro
// ファイル名: [slug].astro
// 悪い例
{ params: { id: 'hello' } }  // slug が undefined になる

// 良い例
{ params: { slug: 'hello' } }
```

### 2. SSR で getStaticPaths を書く

- オンデマンドページでは無視 / エラーになる
- 代わりに `Astro.params` + データ fetch

### 3. 大量 paths の build 時間

- 数千ページ超えるとビルドが重くなる
- ページネーション、ISR、SSR の検討

## 学習の進め方（このリポジトリ内）

1. [ファイルベースルーティング](./file-based-routing.md)
2. 動的ルーティングと getStaticPaths（本記事）
3. [レイアウトコンポーネント](./layout-components.md)
4. Content Collections 記事（今後）

## まとめ

- `[slug].astro` で可変 URL、`getStaticPaths()` で SSG 時の paths を宣言
- `params` は URL パラメータ、`props` はページへ渡すデータ（SSG のみ）
- Content Collections では `getCollection` + `render` と組み合わせるのが定番
- `[...slug]` / `[[...slug]]` で可変長・オプショナルパスに対応
- SSR では `getStaticPaths` は使わず、`Astro.params` で都度データ取得
