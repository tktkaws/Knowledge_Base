# Tailwind CSS × Astro のセットアップ

## 現行の推奨手順（Tailwind v4）

- Astro 5.2 以降は `astro add tailwind` が公式の最短ルート
- 内部では `@tailwindcss/vite` プラグインが設定される
- 旧来の `@astrojs/tailwind` インテグレーションは Tailwind v4 では非推奨
- CSS 側は `@import "tailwindcss";` 一行が基本

> 参照: [Astro — Tailwind CSS](https://docs.astro.build/en/guides/styling/#tailwind)

## セットアップ手順

### 1. 新規プロジェクトで Tailwind を選ぶ

```bash
npm create astro@latest
# ウィザードで Tailwind を選択
```

- テンプレートに Tailwind が含まれる場合、以降の手順は省略できる

### 2. 既存プロジェクトへ追加（Astro >= 5.2）

```bash
npx astro add tailwind
```

```bash
pnpm astro add tailwind
```

```bash
yarn astro add tailwind
```

- `astro.config.mjs` に Vite プラグインが追加される
- `src/styles/global.css` が生成される（または追記される）

> 参照: [Astro — Add Tailwind 4](https://docs.astro.build/en/guides/styling/#add-tailwind-4)

### 3. Astro 5.2 未満の場合（手動）

```bash
npm install tailwindcss @tailwindcss/vite
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
  },
});
```

- Tailwind 公式の Vite プラグイン手順に沿う
- Astro バージョンが古い場合はアップグレードを優先検討

> 参照: [Tailwind CSS — Install with Vite](https://tailwindcss.com/docs/installation/using-vite)

## global.css の設定

```css
/* src/styles/global.css */
@import "tailwindcss";
```

- Tailwind v3 以前の `@tailwind base;` 等のディレクティブは v4 では不要
- カスタムテーマや `@theme` ブロックもこのファイルに追記する

```css
/* テーマ拡張の例 */
@import "tailwindcss";

@theme {
  --color-brand: #2563eb;
  --font-sans: "Inter", sans-serif;
}
```

## レイアウトでの import

- global.css は各ページではなく、共通レイアウトで1回 import する
- 二重 import すると CSS が重複出力されやすい

```astro
---
// src/layouts/Layout.astro
import '../styles/global.css';

const { title } = Astro.props;
---
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <title>{title}</title>
  </head>
  <body class="bg-slate-50 text-slate-900">
    <slot />
  </body>
</html>
```

```astro
---
// src/pages/index.astro
import Layout from '../layouts/Layout.astro';
---
<Layout title="Home">
  <h1 class="text-3xl font-bold">Hello Tailwind</h1>
</Layout>
```

```astro
---
// 悪い例：ページごとに global.css を import
import '../styles/global.css';
---
<h1 class="text-3xl">Home</h1>
```

> 参照: [Astro — Import global stylesheet in layout](https://docs.astro.build/en/guides/styling/#tailwind)

## astro.config.mjs の確認

```js
// 良い例（Tailwind v4 + @tailwindcss/vite）
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
  },
});
```

```js
// 悪い例（v4 プロジェクトに旧インテグレーションが残っている）
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [tailwind()],
});
```

- v4 移行時は `@astrojs/tailwind` の import と `integrations` エントリを削除する
- `astro add tailwind` 実行後に設定を確認する

> 参照: [Astro — Remove Tailwind integration](https://docs.astro.build/en/guides/styling/#remove-tailwind-integration)

## Astro × Tailwind の注意点

### 1. スコープ付き `<style>` との共存

- Tailwind はユーティリティクラス（グローバル）
- コンポーネント固有の装飾は `<style>` スコープ、レイアウトは Tailwind、と役割分担が分かりやすい

```astro
<div class="flex items-center gap-4 p-4 rounded-lg bg-white shadow">
  <slot />
</div>

<style>
  /* コンポーネント固有の微調整だけ scoped に */
  div:focus-within {
    outline: 2px solid var(--color-brand);
  }
</style>
```

### 2. `@apply` の乱用

- v4 でも `@apply` は使えるが、ユーティリティをそのまま HTML に書く方が追いやすいことが多い
- 同じクラス列を何度も複製するなら Astro コンポーネント化を検討

```css
/* 悪い例：@apply で Tailwind を別名ラッパーだらけに */
.btn {
  @apply px-4 py-2 bg-blue-600 text-white rounded;
}
```

### 3. 動的クラス名

- 完全な文字列連結より、事前に分かるクラスを列挙する

```astro
---
const { size } = Astro.props;
---

<!-- 良い例 -->
<p class:list={[
  'text-base',
  { 'text-sm': size === 'sm' },
  { 'text-lg': size === 'lg' },
]}>
  <slot />
</p>
```

```astro
<!-- 悪い例：ビルド時に検出できない動的クラス -->
<p class={`text-${size}`}><slot /></p>
```

- Tailwind の JIT はソース内の**完全なクラス文字列**をスキャンする
- 動的生成クラスは purge 対象外になり、スタイルが当たらない

### 4. React / Vue コンポーネント内

- UI フレームワークコンポーネントでも Tailwind クラスはそのまま使える
- `client:*` 付きコンポーネントでも global.css を layout で読み込んでいれば問題ない

### 5. Content Collections / MDX

- Markdown 本文に Tailwind クラスを書く場合、typography プラグインや `@tailwindcss/typography` の検討
- prose クラスで本文スタイルを一括適用するパターンが多い

## 動作確認

```bash
npm run dev
```

- ブラウザでユーティリティクラスが効いているか確認
- 本番ビルドでも確認する

```bash
npm run build
npm run preview
```

- dev だけ効いて build で消える場合、動的クラス名や import 漏れを疑う

## トラブルシューティング

| 症状 | よくある原因 |
|---|---|
| クラスが一切効かない | layout で global.css を import していない |
| 一部のクラスだけ効かない | 動的クラス名（`text-${size}` など） |
| 設定エラー | 旧 `@astrojs/tailwind` が残っている |
| CSS が二重に読み込まれる | 複数ページで global.css を import |

## まとめ

- Astro 5.2+ では `astro add tailwind` で Tailwind v4 + `@tailwindcss/vite` が推奨構成
- `src/styles/global.css` に `@import "tailwindcss";` を書き、layout で1回 import する
- 旧 `@astrojs/tailwind` インテグレーションは v4 では外す
- 動的クラス名より `class:list` や列挙型 props を使う
- Tailwind はレイアウト・ユーティリティ、scoped CSS はコンポーネント固有、と役割分担する
