# レイアウトコンポーネントの作り方と入れ子構造

## レイアウトコンポーネントとは

- 複数ページで共有する HTML 骨格（`<html>`、`<head>`、共通 nav / footer）
- 通常の `.astro` コンポーネントと同じ形式
- **`<slot />`** で各ページ固有のコンテンツを差し込む
- `src/layouts/` に置くのが慣習（[プロジェクト構成](./project-structure.md) 参照）

> 参照: [Astro — Layouts](https://docs.astro.build/en/basics/layouts/)

## 最小のレイアウト

```astro
---
// src/layouts/Layout.astro
interface Props {
  title: string;
}

const { title } = Astro.props;
---

<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{title}</title>
  </head>
  <body>
    <header>
      <nav><a href="/">Home</a></nav>
    </header>
    <main>
      <slot />
    </main>
    <footer>© 2024</footer>
  </body>
</html>
```

```astro
---
// src/pages/about.astro
import Layout from '../layouts/Layout.astro';
---

<Layout title="About">
  <h1>About us</h1>
  <p>会社概要です。</p>
</Layout>
```

- `<Layout>` タグの **子要素** が `<slot />` の位置に展開される
- ページ側は本文だけ書けばよい

## slot の仕組み

| 概念 | 説明 |
|---|---|
| デフォルト slot | `<slot />` — 子要素全体を受け取る |
| 名前付き slot | `<slot name="sidebar" />` — 特定の部分だけ差し込み |
| フォールバック | `<slot>デフォルト内容</slot>` — 子が無いときの表示 |

```astro
---
// src/layouts/TwoColumnLayout.astro
interface Props {
  title: string;
}
const { title } = Astro.props;
---

<Layout title={title}>
  <div class="grid">
    <aside>
      <slot name="sidebar" />
    </aside>
    <article>
      <slot />
    </article>
  </div>
</Layout>
```

```astro
---
import TwoColumnLayout from '../layouts/TwoColumnLayout.astro';
---

<TwoColumnLayout title="記事">
  <nav slot="sidebar">目次...</nav>
  <h1>本文</h1>
</TwoColumnLayout>
```

- `slot="名前"` 属性で名前付き slot に割り当て
- デフォルト slot には `slot` 属性なしの子が入る

> 参照: [Astro — Slots](https://docs.astro.build/en/basics/astro-components/#slots)

## BaseLayout の入れ子

- レイアウト同士をネストして段階的に構造を足す
- BaseLayout（サイト全体）→ SectionLayout（セクション固有）→ ページ

```astro
---
// src/layouts/BaseLayout.astro
interface Props {
  title: string;
  description?: string;
}
const { title, description = '' } = Astro.props;
---

<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <title>{title}</title>
    {description && <meta name="description" content={description} />}
  </head>
  <body>
    <header><!-- グローバル nav --></header>
    <slot />
    <footer><!-- グローバル footer --></footer>
  </body>
</html>
```

```astro
---
// src/layouts/BlogPostLayout.astro
import BaseLayout from './BaseLayout.astro';

interface Props {
  title: string;
  pubDate: Date;
  description?: string;
}
const { title, pubDate, description } = Astro.props;
---

<BaseLayout title={title} description={description}>
  <article class="prose">
    <header>
      <h1>{title}</h1>
      <time datetime={pubDate.toISOString()}>
        {pubDate.toLocaleDateString('ja-JP')}
      </time>
    </header>
    <slot />
  </article>
</BaseLayout>
```

```astro
---
// src/pages/blog/hello.astro
import BlogPostLayout from '../../layouts/BlogPostLayout.astro';
---

<BlogPostLayout title="Hello" pubDate={new Date('2024-01-01')}>
  <p>記事本文...</p>
</BlogPostLayout>
```

```text
ページ → BlogPostLayout → BaseLayout → HTML 出力
```

- 外側ほど「サイト共通」、内側ほど「ページ種別固有」
- `<html>` / `<head>` は通常 **最外の1レイアウトだけ** に置く

## 入れ子の設計指針

| レイヤー | 担当 |
|---|---|
| BaseLayout | html/head、グローバル nav/footer、OGP 共通 |
| セクション Layout | ブログ、ドキュメントなどカテゴリ固有 UI |
| ページ | 本文コンテンツのみ |

```astro
<!-- 悪い例：各ページで html/head/body を繰り返す -->
<!-- src/pages/about.astro -->
<html>...</html>

<!-- src/pages/contact.astro -->
<html>...</html>
```

- メタタグや nav の変更が全ファイル波及する

## Markdown ページのレイアウト

- `.md` / `.mdx` ページにもレイアウトを適用できる
- frontmatter の `layout` キーで指定
- レイアウト側は `Astro.props.frontmatter` で frontmatter 全体にアクセス

### Markdown ファイル

```md
---
layout: ../layouts/BlogPostLayout.astro
title: '初めての投稿'
author: 'Taro'
pubDate: 2024-06-01
---

# 見出し

本文は Markdown で書く。
```

### レイアウト側

```astro
---
// src/layouts/BlogPostLayout.astro
import BaseLayout from './BaseLayout.astro';

const { frontmatter } = Astro.props;
---

<BaseLayout title={frontmatter.title}>
  <article>
    <h1>{frontmatter.title}</h1>
    <p>by {frontmatter.author}</p>
    <slot />
    <time>{frontmatter.pubDate}</time>
  </article>
</BaseLayout>
```

- `<slot />` に **コンパイル済み Markdown の HTML** が入る
- `.astro` ページと同じ slot 機構

> 参照: [Astro — Markdown layouts](https://docs.astro.build/en/basics/layouts/#markdown-layouts)

## Content Collections + レイアウト

- Astro 5 の Content Collections では `.astro` ページ側でレイアウトを import するパターンが一般的
- `render()` の `<Content />` をレイアウトの slot 相当位置に置く

```astro
---
// src/pages/blog/[slug].astro
import { getCollection, render } from 'astro:content';
import BlogPostLayout from '../../layouts/BlogPostLayout.astro';

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

<BlogPostLayout
  title={post.data.title}
  pubDate={post.data.pubDate}
  description={post.data.description}
>
  <Content />
</BlogPostLayout>
```

- Markdown frontmatter の `layout` キーではなく、動的ルート側でラップ
- 型安全な `post.data` をレイアウト props に渡せる

> 参照: [動的ルーティングと getStaticPaths](./dynamic-routing-getstaticpaths.md)

## props でレイアウトを制御

```astro
---
// src/layouts/Layout.astro
interface Props {
  title: string;
  showSidebar?: boolean;
}
const { title, showSidebar = false } = Astro.props;
---

<BaseLayout title={title}>
  {showSidebar ? (
    <div class="with-sidebar">
      <aside>...</aside>
      <main><slot /></main>
    </div>
  ) : (
    <main><slot /></main>
  )}
</BaseLayout>
```

- 同一レイアウトで表示切り替え
- バリエーションが増えすぎたら別レイアウトファイルに分割

## レイアウト vs 通常コンポーネント

| 観点 | レイアウト | コンポーネント |
|---|---|---|
| 役割 | ページ全体の骨格 | 部分的 UI |
| slot | ほぼ必須 | 任意 |
| html/head | 最外レイアウトが担当 | 通常含まない |
| 置き場所 | `src/layouts/`（慣習） | `src/components/` |

- 技術的には同じ `.astro` コンポーネント
- 役割と置き場所で区別する

## よくある間違い

### 1. slot を忘れる

```astro
<!-- 悪い例：Layout に slot が無い -->
<main>固定コンテンツだけ</main>
<!-- ページ側の子要素が表示されない -->
```

### 2. 複数レイアウトで html を重複

```astro
<!-- 悪い例：BlogPostLayout も BaseLayout も <!DOCTYPE html> を持つ -->
<!-- ネスト時に html が二重になる -->
```

### 3. Markdown layout パス間違い

```md
---
layout: layouts/BlogPostLayout.astro
---
```

- Markdown ファイルからの **相対パス** で指定
- プロジェクト構成に合わせて `../layouts/...` など調整

### 4. frontmatter と props の混同

- Markdown layout → `Astro.props.frontmatter`
- 通常コンポーネント → `Astro.props` に直接定義した props

## 設計チェックリスト

- [ ] `<html>` / `<head>` は BaseLayout に1箇所だけ
- [ ] ページファイルは Layout でラップし、本文だけ書いている
- [ ] ブログ・ドキュメントなど種別ごとに中間レイアウトがある
- [ ] slot / 名前付き slot の使い分けが意図どおり
- [ ] Content Collections 記事は `render()` + レイアウトで統一

## 学習の進め方（このリポジトリ内）

1. [.astro ファイルの構造](./astro-file-structure.md)
2. [プロジェクト構成](./project-structure.md)
3. [ファイルベースルーティング](./file-based-routing.md)
4. レイアウトコンポーネント（本記事）

## まとめ

- レイアウトは slot でページ固有コンテンツを差し込む `.astro` コンポーネント
- BaseLayout → セクション Layout → ページの入れ子で DRY に保つ
- Markdown は frontmatter の `layout` キー、Collections は `.astro` 側でラップ
- 名前付き slot でサイドバーなど部分差し替えが可能
- html/head は最外レイアウト1箇所に集約する
