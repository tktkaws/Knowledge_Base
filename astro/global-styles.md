# グローバルスタイルの適用方法

## グローバルスタイルとは

- サイト全体、または複数ページ・コンポーネントに共通で効くCSS
- リセットCSS、タイポグラフィ、カラートークン、ユーティリティ（Tailwind など）が典型
- Astro の `<style>` デフォルトはスコープ付きのため、全体に効かせるには別手段が必要

> 参照: [Astro — Styles and CSS](https://docs.astro.build/en/guides/styling/)

## 主な適用方法（3パターン）

| 方法 | 向く用途 |
|---|---|
| `src/styles/global.css` を import | サイト全体のベース、Tailwind、デザイントークン |
| layout で読み込み | 全ページ共通のスタイル配信 |
| `<style is:global>` / `:global()` | コンポーネント内から限定的にグローバル指定 |

- 基本は **global.css + layout import**
- `is:global` は例外用途に留める

## src/styles/global.css の作成

```css
/* src/styles/global.css */

/* リセット・ベース */
*,
*::before,
*::after {
  box-sizing: border-box;
}

html {
  font-family: system-ui, sans-serif;
  line-height: 1.6;
}

body {
  margin: 0;
  color: #1e293b;
  background: #f8fafc;
}

/* デザイントークン（CSS 変数） */
:root {
  --color-brand: #2563eb;
  --space-md: 1rem;
}

a {
  color: var(--color-brand);
}
```

- プロジェクト全体の「土台」になるスタイルを集約
- コンポーネント固有の見た目はここに書きすぎない

> 参照: [Astro — Create global stylesheet](https://docs.astro.build/en/tutorial/2-pages/5/)

## layout での読み込み

- 共通レイアウトコンポーネントで global.css を **1回だけ** import
- その layout を使う全ページにスタイルが届く

```astro
---
// src/layouts/BaseLayout.astro
import '../styles/global.css';

interface Props {
  title: string;
}

const { title } = Astro.props;
---
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{title}</title>
  </head>
  <body>
    <header>...</header>
    <main>
      <slot />
    </main>
    <footer>...</footer>
  </body>
</html>
```

```astro
---
// src/pages/about.astro
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout title="About">
  <h1>About</h1>
</BaseLayout>
```

```astro
---
// 悪い例：各ページで global.css を import
import '../styles/global.css';
---
<h1>About</h1>
```

- ページごとの import は CSS 重複や読み込み順の混乱の原因
- layout 経由に統一する

> 参照: [Astro — Import global stylesheet in layout](https://docs.astro.build/en/guides/styling/#tailwind)

## Tailwind を使う場合

```css
/* src/styles/global.css */
@import "tailwindcss";
```

```astro
---
// src/layouts/BaseLayout.astro
import '../styles/global.css';
---
```

- Tailwind も global.css 経由が標準
- `@theme` ブロックでカスタムトークンを定義できる

> 参照: [Astro — Tailwind CSS](https://docs.astro.build/en/guides/styling/#tailwind)

## is:global と :global() の使い方

### is:global — style タグ全体をグローバル化

```astro
<style is:global>
  /* サイト全体の h1 に効く（layout 相当の影響） */
  h1 {
    font-size: 2rem;
    font-weight: 700;
  }
</style>
```

- コンポーネントファイル内に書いても、出力はグローバルCSS
- 乱用するとスタイルの所在が分からなくなる

### :global() — スコープ内の一部だけグローバル指定

```astro
<style>
  .wrapper {
    padding: var(--space-md);
  }
  /* slot 内（Markdown 等）の見出しだけ */
  .wrapper :global(h2) {
    margin-top: 2rem;
    border-bottom: 1px solid #e2e8f0;
  }
</style>

<div class="wrapper">
  <slot />
</div>
```

- コンポーネントのスコープは維持しつつ、子孫の特定要素だけグローバルに当てる
- CMS や Markdown 本文のスタイル当てに向く

> 参照: [Astro — Global Styles](https://docs.astro.build/en/guides/styling/#global-styles)

## scoped（スコープ付き）との使い分け

| 観点 | グローバル | スコープ付き `<style>` |
|---|---|---|
| 適用範囲 | サイト全体 | そのコンポーネントのみ |
| 向く例 | リセット、フォント、Tailwind | カード、ボタン、固有レイアウト |
| 衝突リスク | 高い（命名に注意） | 低い |
| 保守性 | 1ファイルに集約しやすい | コンポーネントと同居 |

```astro
---
// Card.astro — コンポーネント固有は scoped
---
<article class="card">
  <slot />
</article>

<style>
  .card {
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    padding: 1rem;
  }
</style>
```

```css
/* global.css — サイト共通のベース */
body {
  margin: 0;
  font-family: sans-serif;
}
```

- **グローバル**：変わらない土台・共通ルール
- **scoped**：コンポーネント単位の見た目

> 参照: [Astro — Scoped Styles](https://docs.astro.build/en/guides/styling/#scoped-styles)

## 悪い例・注意点

```css
/* 悪い例：global.css に Card 専用スタイル → コンポーネントと離れて取り残しやすい */
.card-title { font-size: 1.25rem; }
```

```astro
<!-- 悪い例：各コンポーネントが is:global で h2 を上書き → 詳細度競合 -->
<style is:global>
  h2 { color: red; }
</style>
```

- コンポーネント固有スタイルは scoped `<style>` に寄せる
- layout を使わずページ直 import すると追加漏れが起きやすい
- scoped と global で同名クラス（`.btn` 等）を二重定義しない

## まとめ

- グローバルスタイルは `src/styles/global.css` を layout で import するのが基本
- ページごとの import や `is:global` の乱用は避ける
- コンポーネント固有の見た目は scoped `<style>` に閉じる
- `:global()` は slot 内コンテンツなど、限定的なグローバル指定に使う
- 「土台は global、部品は scoped」と役割を分けると保守しやすい
