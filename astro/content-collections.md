# Content Collections — Markdownコンテンツの型安全な管理

## Content Collections とは

- Markdown / MDX などのコンテンツを **型付きコレクション** として管理する仕組み
- フロントマターを Zod スキーマで検証し、TypeScript の型を自動生成
- `getCollection` / `getEntry` / `render` でビルド時に安全に取得・描画
- Astro 5 以降は **Content Layer API** が標準（`src/content.config.ts`）

> 参照: [Astro — Content collections](https://docs.astro.build/en/guides/content-collections/)

## Astro 5 の構成

```text
src/
├── content/
│   └── blog/
│       ├── post-1.md
│       └── post-2.mdx
├── content.config.ts   ← コレクション定義
└── pages/
    └── blog/
        └── [id].astro  ← 動的ルート
```

- 旧 API（`src/content/config.ts` + `defineCollection` の legacy モード）は Astro 6 で削除予定
- 新規プロジェクトは `src/content.config.ts` を使う

## content.config.ts の定義

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
    updatedDate: z.coerce.date().optional(),
    draft: z.boolean().default(false),
    tags: z.array(z.string()).optional(),
  }),
});

export const collections = { blog };
```

### ポイント

- `defineCollection` — コレクション1つ分の設定
- `loader: glob(...)` — ファイルパターンで Markdown を読み込む
- `schema` — Zod でフロントマターの型と必須項目を定義
- `export const collections` — プロジェクト内の全コレクションを登録

> 参照: [Astro — defineCollection](https://docs.astro.build/en/reference/modules/astro-content/#definecollection)

## Zod スキーマの例

```ts
import { z } from 'astro/zod';

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.md' }),
  schema: z.object({
    title: z.string(),
    description: z.string().max(160),
    pubDate: z.coerce.date(),
    heroImage: z.string().optional(),
    tags: z.array(z.string()).default([]),
  }),
});
```

### 悪い例：スキーマなし

```ts
const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.md' }),
  // schema 無し → typo や型不一致にビルド時に気づけない
});
```

### 良い例：必須項目と型を明示

- `title` 欠落 → ビルドエラー
- `pubDate: "2026-01-01"` → `z.coerce.date()` で Date 型に変換
- エディタで `post.data.title` の補完が効く

- スキーマ変更後は dev サーバー再起動 or `s + Enter`（コンテンツ同期）が必要な場合がある

> 参照: [Astro — Defining the collection schema](https://docs.astro.build/en/guides/content-collections/#defining-the-collection-schema)

## Markdown ファイルの書き方

```md
---
title: 'はじめての Content Collections'
description: 'Zod スキーマで型安全に管理する'
pubDate: 2026-01-15
tags:
  - astro
  - markdown
---

本文はここに書く。
```

- フロントマターはスキーマに従う
- ファイルパスが `id` になる（例：`post-1.md` → `id: "post-1"`）

## getCollection — 一覧取得

```astro
---
// src/pages/blog/index.astro
import { getCollection } from 'astro:content';

const allPosts = await getCollection('blog');
const posts = allPosts
  .filter((post) => !post.data.draft)
  .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
---
<ul>
  {posts.map((post) => (
    <li>
      <a href={`/blog/${post.id}/`}>{post.data.title}</a>
      <time datetime={post.data.pubDate.toISOString()}>
        {post.data.pubDate.toLocaleDateString('ja-JP')}
      </time>
    </li>
  ))}
</ul>
```

- 第2引数でフィルタ関数を渡せる

```ts
const publishedPosts = await getCollection('blog', ({ data }) => {
  return data.draft !== true;
});
```

## getEntry — 単一取得

```ts
import { getEntry } from 'astro:content';

const post = await getEntry('blog', 'post-1');
if (!post) {
  throw new Error('Post not found');
}
// post.data.title など型付きでアクセス
```

- Astro 5 では `getEntryBySlug` は非推奨 → `getEntry` を使う

## render — 本文の描画

```astro
---
import { getEntry, render } from 'astro:content';

const post = await getEntry('blog', 'post-1');
if (!post) throw new Error('Not found');

const { Content, headings, remarkPluginFrontmatter } = await render(post);
---
<article>
  <h1>{post.data.title}</h1>
  <Content />
</article>
```

- `Content` — コンパイル済みの MD/MDX コンポーネント
- `headings` — 見出し一覧（目次生成向け）
- MDX の場合は [MDXの活用](./mdx-in-astro.md) の `components` prop も使える

> 参照: [Astro — render()](https://docs.astro.build/en/reference/modules/astro-content/#render)

## 動的ルートとの連携

```astro
---
// src/pages/blog/[id].astro
import { getCollection, render } from 'astro:content';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { id: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---
<h1>{post.data.title}</h1>
<p>{post.data.description}</p>
<Content />
```

- `getStaticPaths` + `getCollection` でビルド時に全記事ページを生成（SSG）
- `post.id` が URL の `[id]` に対応（`post-1.md` → `/blog/post-1/`）

### スラッグに `/` が含まれる場合

```astro
// src/pages/blog/[...slug].astro
export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }));
}
```

- 複数階層の `id` には rest parameter（`[...slug]`）を使う

> 参照: [Astro — Generating Routes from Content](https://docs.astro.build/en/guides/content-collections/#generating-routes-from-content)

- スキーマ定義により `post.data.*` に TypeScript 補完が効き、typo はビルドで検出できる

## よくある間違い

| 間違い | 対処 |
|---|---|
| `src/content/config.ts` に書いている | Astro 5 では `src/content.config.ts` |
| スキーマとフロントマターのキー不一致 | ビルドエラーメッセージで修正 |
| `getEntryBySlug` を使っている | `getEntry('blog', id)` に移行 |
| draft 記事が公開される | `getCollection` でフィルタ |

## 設計チェックリスト

- [ ] `src/content.config.ts` でコレクションを定義した
- [ ] Zod スキーマで必須フィールドを指定した
- [ ] 一覧ページで `getCollection`、詳細で `getEntry` + `render` を使っている
- [ ] `getStaticPaths` で動的ルートを生成している
- [ ] draft や非公開記事のフィルタを入れている

## まとめ

- Content Collections は Markdown コンテンツを型安全に管理する Astro の中核機能
- Astro 5 では `src/content.config.ts` + `defineCollection` + Zod スキーマが標準
- `getCollection` で一覧、`getEntry` + `render` で詳細描画
- `getStaticPaths` と組み合わせてビルド時に全記事ページを生成できる
- スキーマを書くことでフロントマターの typo や型不一致をビルド時に検出できる

> 参照: [MDXの活用 — Markdown内でコンポーネントを使う](./mdx-in-astro.md) / [Markdownのカスタマイズ](./markdown-remark-rehype.md)
