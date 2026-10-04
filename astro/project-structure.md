# プロジェクト構成 — src/pages, src/components, src/layouts

## Astro プロジェクトの全体像

- Astro は **opinionated（意見のある）** なフォルダ構成を推奨
- `src/` にソースコード、`public/` に加工不要の静的ファイル
- ルート直下に設定ファイル（`astro.config.mjs`、`package.json` など）
- 最小構成でも `src/pages/` は必須（ここが無いとルートが生まれない）

> 参照: [Astro — Project structure](https://docs.astro.build/en/basics/project-structure/)

## 標準ディレクトリ構成

```text
/
├── public/              # 加工なしでそのまま配信
├── src/
│   ├── assets/          # ビルド時に最適化される画像など
│   ├── components/      # 再利用 UI 部品
│   ├── content/         # Content Collections の Markdown 等
│   ├── content.config.ts # コレクション定義（Astro 5）
│   ├── layouts/         # ページ共通レイアウト
│   ├── pages/           # ルート（必須）
│   └── styles/          # グローバル CSS など
├── astro.config.mjs
├── package.json
└── tsconfig.json
```

| ディレクトリ | 必須 | 役割 |
|---|---|---|
| `src/pages/` | **必須** | ファイルベースルーティングの起点 |
| `src/components/` | 推奨 | ボタン、カード、ヘッダーなど部品 |
| `src/layouts/` | 推奨 | 全ページ共通の HTML 骨格 |
| `public/` | 推奨 | favicon、robots.txt など |
| `src/assets/` | 任意 | `astro:assets` で最適化する画像 |

- `components` / `layouts` は慣習的な名前で、厳密な必須ではない
- 小規模プロジェクトでは `layouts` を `components` にまとめることもある

## src/pages — ルートの起点

- ここにある `.astro`、`.md`、`.mdx` などが **URL になる**
- `index.astro` → `/`
- `about.astro` → `/about`
- `[slug].astro` → 動的ルート

```text
src/pages/
├── index.astro          → /
├── about.astro          → /about
├── blog/
│   ├── index.astro      → /blog
│   └── [slug].astro     → /blog/:slug
└── api/
    └── hello.ts         → /api/hello（エンドポイント）
```

- **pages 以外のファイルは URL にならない**
- 詳細は [ファイルベースルーティング](./file-based-routing.md) を参照

> 参照: [Astro — Pages](https://docs.astro.build/en/basics/astro-pages/)

## src/components — 再利用 UI

- ページを構成する部品置き場
- Header、Footer、Card、Navigation など
- `.astro` だけでなく React / Vue コンポーネントも置ける

```astro
---
// src/pages/index.astro
import Hero from '../components/Hero.astro';
import FeatureList from '../components/FeatureList.astro';
---

<Hero title="Welcome" />
<FeatureList />
```

```text
<!-- 悪い例：pages 内に全部直書き -->
<!-- src/pages/index.astro が数百行になり、再利用もテストもしづらい -->
<header>...</header>
<main>...(長大なマークアップ)...</main>
<footer>...</footer>
```

- コンポーネントは [props と slots](./astro-file-structure.md) でデータを受け取る
- URL にはならない点が pages との最大の違い

> 参照: [Astro — Components](https://docs.astro.build/en/basics/astro-components/)

## src/layouts — 共通骨格

- 複数ページで共有する HTML 構造（`<html>`、`<head>`、ヘッダー / フッター）
- `<slot />` で各ページの固有コンテンツを差し込む
- 入れ子にして BaseLayout → BlogLayout のように段階化も可能

```text
src/layouts/
├── BaseLayout.astro     # html/head/body、共通 nav
└── BlogPostLayout.astro # 記事用メタ情報、目次枠
```

- 詳細は [レイアウトコンポーネント](./layout-components.md) を参照

> 参照: [Astro — Layouts](https://docs.astro.build/en/basics/layouts/)

## public/ vs src/assets/

### public/

- ビルド時に **加工されずそのまま** 出力ディレクトリへコピー
- URL は `/` からのパス（`public/favicon.ico` → `/favicon.ico`）
- robots.txt、manifest、固定 favicon、フォント（加工不要なもの）向き

```text
public/
├── favicon.ico          → /favicon.ico
├── robots.txt           → /robots.txt
└── fonts/
    └── NotoSans.woff2   → /fonts/NotoSans.woff2
```

### src/assets/

- import して使うアセット置き場
- `astro:assets` の `<Image />` で最適化・リサイズ可能
- ビルドパイプラインを通るため、参照されないファイルは出力に含まれない

```astro
---
import { Image } from 'astro:assets';
import heroImage from '../assets/hero.jpg';
---

<Image src={heroImage} alt="Hero" width={1200} height={630} />
```

| 観点 | `public/` | `src/assets/` |
|---|---|---|
| 参照方法 | `/path/to/file` の絶対パス | `import` |
| 最適化 | なし（そのまま） | あり（Image コンポーネント等） |
| 向く例 | favicon、robots.txt | 記事サムネ、レスポンシブ画像 |

```astro
<!-- 悪い例：大きな写真を public に置いて最適化なしで配信 -->
<img src="/images/huge-photo.jpg" alt="" />

<!-- 良い例：assets + Image で最適化 -->
<Image src={photo} alt="説明" widths={[400, 800, 1200]} sizes="(max-width: 768px) 100vw, 800px" />
```

> 参照: [Astro — public/ directory](https://docs.astro.build/en/basics/project-structure/#public)
> 参照: [Astro — Images](https://docs.astro.build/en/guides/images/)

## src/content/ と content.config.ts

- Markdown / MDX などコンテンツファイルの置き場（Content Collections）
- Astro 5 では `src/content.config.ts` で `defineCollection` + Zod スキーマを定義
- 型安全な frontmatter 管理と `getCollection` / `getEntry` API

```text
src/
├── content/
│   └── blog/
│       ├── post-1.md
│       └── post-2.md
└── content.config.ts
```

- 動的ルート `[slug].astro` と組み合わせて記事一覧・詳細を生成する典型パターン（[動的ルーティング](./dynamic-routing-getstaticpaths.md) 参照）

> 参照: [Astro — Content Collections](https://docs.astro.build/en/guides/content-collections/)

## ルート直下の設定ファイル

| ファイル | 役割 |
|---|---|
| `astro.config.mjs` | サイト URL、インテグレーション、アダプター、SSR 設定 |
| `package.json` | 依存関係、`dev` / `build` スクリプト |
| `tsconfig.json` | TypeScript 設定（推奨） |
| `env.d.ts` | Astro 型定義の参照 |

```js
// astro.config.mjs（最小例）
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://example.com',
});
```

## その他よく見るディレクトリ

| パス | 用途 |
|---|---|
| `src/styles/` | グローバル CSS、変数定義 |
| `src/lib/` | API クライアント、ユーティリティ（慣習） |
| `src/data/` | JSON など静的データ |
| `src/middleware/` | リクエスト前処理（SSR / オンデマンド） |

- 公式で固定された名前ではないが、チーム内で統一すると探しやすい

## 構成の設計指針

### pages は薄く

- データ取得とレイアウト適用が主役
- 見た目の詳細は components に委譲

### components は用途で分割

- `components/ui/` — ボタン、バッジなど汎用
- `components/blog/` — ブログ専用
- 深くしすぎない（3階層程度まで）

### layouts は HTML 骨格に集中

- ナビ、メタタグ、フッター
- ページ固有の本文は slot に任せる

## よくある間違い

### 1. components を pages 相当で使う

- `src/components/about.astro` を作っても `/about` にはならない
- URL にしたいなら `src/pages/about.astro`

### 2. すべてを public/ に置く

- 最適化や tree-shaking の恩恵を受けられない
- 自作 CSS / JS は `src/`、加工不要静的ファイルだけ `public/`

### 3. pages ディレクトリを省略

- Astro プロジェクトとして成立しない
- 最低1ページ（通常 `index.astro`）が必要

## 最小プロジェクト例

```bash
npm create astro@latest
```

```text
my-site/
├── public/favicon.svg
├── src/
│   ├── components/
│   │   └── Header.astro
│   ├── layouts/
│   │   └── Layout.astro
│   └── pages/
│       └── index.astro
├── astro.config.mjs
└── package.json
```

## 学習の進め方（このリポジトリ内）

1. [.astro ファイルの構造](./astro-file-structure.md)
2. プロジェクト構成（本記事）
3. [ファイルベースルーティング](./file-based-routing.md)
4. [動的ルーティングと getStaticPaths](./dynamic-routing-getstaticpaths.md)
5. [レイアウトコンポーネント](./layout-components.md)

## まとめ

- `src/pages/` がルートの起点で、**必須ディレクトリ**
- `src/components/` は再利用 UI、`src/layouts/` は共通 HTML 骨格
- `public/` は加工なし静的配信、`src/assets/` は import + 最適化向き
- pages を薄く、components / layouts に責務を分けると保守しやすい
- Content Collections は `src/content/` + `content.config.ts` で型安全に管理する
