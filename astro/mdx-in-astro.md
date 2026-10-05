# MDXの活用 — Markdown内でコンポーネントを使う

## MDX とは

- Markdown に JSX コンポーネントを埋め込める形式
- 記事本文中に Callout、コードデモ、インタラクティブUIなどを差し込める
- Astro では `@astrojs/mdx` インテグレーションで利用
- Content Collections と組み合わせると型安全な MDX ブログが作れる

> 参照: [Astro — MDX integration](https://docs.astro.build/en/guides/integrations-guide/mdx/)

## セットアップ

```bash
npx astro add mdx
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import mdx from '@astrojs/mdx';

export default defineConfig({
  integrations: [mdx()],
});
```

- `.mdx` ファイルが Markdown ページとして扱われる
- Content Collections では loader の pattern に `mdx` を含める

```ts
// src/content.config.ts
loader: glob({ base: './src/content/blog', pattern: '**/*.{md,mdx}' }),
```

## 基本的な MDX ファイル

```mdx
---
title: 'MDX のはじめ方'
description: 'Markdown 内でコンポーネントを使う'
---

import Callout from '../components/Callout.astro';

# はじめに

通常の **Markdown** も書ける。

<Callout type="info">
  これは Astro コンポーネントを埋め込んだ Callout です。
</Callout>

## 次のステップ

- [Content Collections](./content-collections.md) と組み合わせる
- 見出し要素をカスタムコンポーネントに差し替える
```

- フロントマター + import + Markdown + JSX が1ファイルに共存
- Astro コンポーネント（`.astro`）も React コンポーネントも import できる

## MDX 内での import

```mdx
import { Chart } from '../components/Chart.jsx';
import Badge from '../components/Badge.astro';

<Badge label="New" />

<Chart data={[1, 2, 3]} client:visible />
```

### 悪い例：client:* 無しでインタラクティブコンポーネント

```mdx
import Counter from '../components/Counter.jsx';

<!-- 動かない。静的HTMLのみ -->
<Counter />
```

### 良い例：必要なコンポーネントだけ client:*

```mdx
import Counter from '../components/Counter.jsx';
import Note from '../components/Note.astro';

<Note>静的な注釈は .astro で十分</Note>
<Counter client:visible />
```

- MDX 内のフレームワークコンポーネントも [client:ディレクティブ](./client-directives.md) のルールに従う

## components prop — Markdown 要素の差し替え

- MDX 内で `export const components` を定義し、標準 HTML 要素をカスタムコンポーネントに置き換えられる
- レンダリング時に `<Content components={...} />` でも上書きできる

### MDX ファイル内で定義

```mdx
import Blockquote from '../components/Blockquote.astro';

export const components = { blockquote: Blockquote };

> この引用は Blockquote コンポーネントで描画される
```

```astro
---
// src/components/Blockquote.astro
const props = Astro.props;
---
<blockquote {...props} class="border-l-4 border-blue-500 pl-4">
  <slot />
</blockquote>
```

### Astro ページから components を渡す

```astro
---
import { Content, components } from '../content/example.mdx';
import CustomHeading from '../components/CustomHeading.astro';
---
<Content components={{ ...components, h1: CustomHeading }} />
```

- `...components` で MDX 内の定義を引き継ぎつつ、`h1` だけ上書き
- サイト全体で見出しデザインを統一したいときに便利

> 参照: [Astro — Using components in MDX](https://docs.astro.build/en/guides/integrations-guide/mdx/#using-components-in-mdx)

## Content Collections との連携

```astro
---
// src/pages/blog/[id].astro
import { getEntry, render } from 'astro:content';
import CustomHeading from '../../components/CustomHeading.astro';

const post = await getEntry('blog', Astro.params.id);
if (!post) throw new Error('Not found');

const { Content } = await render(post);
---
<article>
  <h1>{post.data.title}</h1>
  <Content components={{ h1: CustomHeading, h2: CustomHeading }} />
</article>
```

- コレクションエントリの MDX 本文を `render()` で取得
- `Content` の `components` prop で見出しやリンクの見た目を制御

## 差し替え可能な要素

| Markdown 記法 | 対応する HTML | components のキー |
|---|---|---|
| `# 見出し` | `<h1>` | `h1` |
| `> 引用` | `<blockquote>` | `blockquote` |
| `` `code` `` | `<code>` | `code` |
| `[text](url)` | `<a>` | `a` |
| など | 標準 HTML 要素 | 要素名（小文字） |

- サイト全体のタイポグラフィを MDX 側で統一できる
- [Markdownのカスタマイズ](./markdown-remark-rehype.md)（remark/rehype）とは別レイヤー

## MDX 最適化（optimize）

- `@astrojs/mdx` は MDX を静的 HTML に最適化する `optimize` オプションを持つ
- `components` prop で渡す動的コンポーネントは `ignoreElementNames` で除外が必要な場合がある

```js
// astro.config.mjs
import mdx from '@astrojs/mdx';

export default defineConfig({
  integrations: [
    mdx({
      optimize: {
        ignoreElementNames: ['h1', 'Callout'],
      },
    }),
  ],
});
```

- カスタムコンポーネントが静的文字列に変換されてしまう問題を防ぐ

## .md と .mdx の使い分け

| 形式 | 向く用途 |
|---|---|
| `.md` | プレーンな文章、シンプルなブログ |
| `.mdx` | コンポーネント埋め込み、デモ、Callout、チャート |

- すべて MDX にする必要はない
- コンポーネントが必要な記事だけ `.mdx` にする方がビルドも管理も軽い

## よくある間違い

### 1. slot を忘れたカスタムコンポーネント

```astro
---
// 悪い例：子コンテンツが表示されない
---
<div class="callout">
  <!-- <slot /> が無い -->
</div>
```

```astro
---
// 良い例
---
<div class="callout">
  <slot />
</div>
```

### 2. MDX 内で Astro コンポーネントに client:* を付け忘れ

- フレームワークコンポーネントと同様、操作が必要なら `client:*` が必要

### 3. components のキー名を間違える

```mdx
export const components = { Blockquote: Blockquote };
<!-- キーは小文字の HTML 要素名：blockquote -->
```

## 設計チェックリスト

- [ ] `@astrojs/mdx` を `astro.config` に追加した
- [ ] Content Collections の loader pattern に `mdx` を含めた
- [ ] インタラクティブな MDX 内コンポーネントに `client:*` を付けた
- [ ] 見出し・引用などの統一に `components` prop を検討した
- [ ] コンポーネント不要な記事は `.md` のままにしている

## まとめ

- MDX は Markdown 内に Astro / React などのコンポーネントを埋め込める拡張形式
- `@astrojs/mdx` で有効化し、Content Collections と組み合わせて型安全に管理できる
- `export const components` または `<Content components={...} />` で HTML 要素を差し替えられる
- インタラクティブな部品には `client:*` が必要（MDX 内も例外ではない）
- 全部 MDX にせず、コンポーネントが要る記事だけ `.mdx` にするのが現実的

> 参照: [Content Collections](./content-collections.md) / [Markdownのカスタマイズ — remarkプラグイン, rehypeプラグイン](./markdown-remark-rehype.md)
