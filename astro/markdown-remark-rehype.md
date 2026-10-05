# Markdownのカスタマイズ — remarkプラグイン, rehypeプラグイン

## Markdown 処理パイプライン

- Astro の Markdown は **Unified** エコシステムで処理される
- 大きく2段階に分かれる

```text
Markdown テキスト
    ↓ remark（Markdown AST を処理）
Markdown AST
    ↓ 変換
HTML AST
    ↓ rehype（HTML AST を処理）
HTML 文字列
```

- **remark プラグイン** — Markdown 段階（見出し、リンク、コードブロックなど）
- **rehype プラグイン** — HTML 段階（属性追加、要素変換など）
- `astro.config.mjs` の `markdown` 設定でカスタマイズ

> 参照: [Astro — Markdown configuration](https://docs.astro.build/en/reference/configuration-reference/#markdown-options)

## 設定の基本形

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  markdown: {
    remarkPlugins: [],
    rehypePlugins: [],
  },
});
```

- Astro 4 以前は `remarkPlugins` / `rehypePlugins` が主流
- Astro 5 以降は `markdown.processor: unified({ ... })` 形式も推奨される場合がある
- プラグインの依存関係によっては `@astrojs/markdown-remark` から import する

```js
import { unified, rehypeHeadingIds } from '@astrojs/markdown-remark';

export default defineConfig({
  markdown: {
    processor: unified({
      rehypePlugins: [rehypeHeadingIds],
    }),
  },
});
```

> 参照: [Astro — markdown.processor](https://docs.astro.build/en/reference/configuration-reference/#markdownprocessor)

## よく使う例1：見出し ID（アンカーリンク）

- 見出しに `id` 属性を付け、目次リンクや `#section` ジャンプを可能にする
- Astro 組み込みの `rehypeHeadingIds` を使う方法が簡単

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import { unified, rehypeHeadingIds } from '@astrojs/markdown-remark';

export default defineConfig({
  markdown: {
    processor: unified({
      rehypePlugins: [
        rehypeHeadingIds,
        // 他の rehype プラグインは rehypeHeadingIds の後に置く
      ],
    }),
  },
});
```

- 生成例：`## Picture コンポーネント` → `<h2 id="picture-">...</h2>`
- 特殊文字を含む見出しは ID の末尾に `-` が付く場合がある（Astro v6 の仕様変更に注意）

```markdown
## `<Picture />`
<!-- id="picture-" になる -->
```

```markdown
[Picture コンポーネント](/guides/images/#picture-)
```

### 悪い例：手動で ID を本文に書く

```markdown
<h2 id="intro">はじめに</h2>
```

- Markdown としては動くが、執筆者の手間が増える
- 見出し変更時に ID も更新が必要

### 良い例：プラグインで自動付与

- 執筆者は通常の `## 見出し` だけ書けばよい
- 目次コンポーネントは `render()` の `headings` も併用できる

> 参照: [Astro — rehypeHeadingIds](https://docs.astro.build/en/guides/markdown-content/)

## よく使う例2：外部リンクの属性追加

- 外部リンクに `target="_blank"`、`rel="noopener noreferrer"` を自動付与
- `rehype-external-links` が定番

```bash
npm install rehype-external-links @astrojs/markdown-remark
```

```js
// astro.config.mjs
import { unified } from '@astrojs/markdown-remark';
import { defineConfig } from 'astro/config';
import rehypeExternalLinks from 'rehype-external-links';

export default defineConfig({
  markdown: {
    processor: unified({
      rehypePlugins: [
        [
          rehypeExternalLinks,
          {
            target: '_blank',
            rel: ['noopener', 'noreferrer'],
            content: { type: 'text', value: ' ↗' },
          },
        ],
      ],
    }),
  },
});
```

### 悪い例：執筆者に毎回 HTML を書かせる

```markdown
<a href="https://example.com" target="_blank" rel="noopener noreferrer">Example</a>
```

- Markdown の簡潔さが失われる
- 付け忘れが発生しやすい

### 良い例：プラグインで一括適用

```markdown
[Example](https://example.com)
```

- 内部リンク（同一ドメイン）は通常そのまま
- 外部リンクだけ属性とアイコンを付与

> 参照: [Astro — Add links to external sites recipe](https://docs.astro.build/en/recipes/external-links/)

## remark プラグインの例

- `remark-toc` — `## 目次` の位置に目次を自動挿入
- `remark-gfm` — GitHub Flavored Markdown（表・打ち消し線など）

## プラグインの指定方法

### プラグイン名だけ

```js
remarkPlugins: [remarkToc]
```

### オプション付き（配列形式）

```js
rehypePlugins: [
  [rehypeExternalLinks, { target: '_blank' }],
]
```

- 第1要素：プラグイン関数
- 第2要素：オプションオブジェクト

## remark と rehype の使い分け

| やりたいこと | 段階 | 例 |
|---|---|---|
| Markdown 記法の拡張 | remark | `remark-gfm`（表・打ち消し線） |
| 目次の挿入 | remark | `remark-toc` |
| 見出しへの id 付与 | rehype | `rehypeHeadingIds` |
| 外部リンク属性 | rehype | `rehype-external-links` |
| コードハイライト属性 | rehype | `rehype-pretty-code` |
| 画像の lazy loading | rehype | カスタム rehype プラグイン |

- **Markdown の構造を変える** → remark
- **HTML の属性・要素を変える** → rehype

## MDX / Content Collections との関係

- `markdown` 設定は `.md` ファイルと Content Collections の Markdown に適用される
- MDX ファイルは MDX 独自の処理も通る
- MDX 内のコンポーネント差し替えは [MDXの活用](./mdx-in-astro.md) の `components` prop が別レイヤー

```text
Markdown 設定（remark/rehype）  →  全 .md / コレクション
MDX components prop             →  MDX 内の要素差し替え
```

## プラグインの順序

- rehype プラグインは **順序が重要**
- `rehypeHeadingIds` は、ID に依存する他プラグインより **先** に置く

```js
rehypePlugins: [
  rehypeHeadingIds,           // 1. 先に ID を付ける
  otherPluginUsingHeadingIds, // 2. ID を使うプラグイン
]
```

- 順序を逆にすると、ID 依存プラグインが空振りする

## よくある間違い

| 間違い | 対処 |
|---|---|
| プラグインをインストールし忘れ | `npm install` を確認 |
| remark と rehype を取り違える | 処理段階を確認 |
| プラグイン順序が逆 | ID 付与を先に |
| MDX の components と混同 | 用途が異なる |

## 設計チェックリスト

- [ ] 外部リンク方針を rehype プラグインで統一した
- [ ] 見出し ID を自動付与している（目次・ジャンプリンク向け）
- [ ] プラグインの順序を確認した
- [ ] 執筆者がプレーン Markdown だけ書けばよい状態にした
- [ ] MDX の `components` prop との役割分担を理解した

## まとめ

- Astro の Markdown は remark（Markdown 段階）と rehype（HTML 段階）の2段パイプライン
- `astro.config.mjs` の `markdown.remarkPlugins` / `rehypePlugins`（または `processor: unified()`）で拡張
- 見出し ID は `rehypeHeadingIds`、外部リンクは `rehype-external-links` が定番
- 執筆者の手間を減らし、サイト全体のルールを設定で一括適用するのが目的
- MDX の `components` prop とは別レイヤー。用途に応じて使い分ける

> 参照: [Content Collections](./content-collections.md) / [MDXの活用](./mdx-in-astro.md)
