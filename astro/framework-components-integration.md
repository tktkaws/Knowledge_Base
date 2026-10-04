# React / Vue / Svelteコンポーネントの統合

## なぜフレームワークコンポーネントを使うか

- Astro は UI フレームワーク非依存（UI-agnostic）
- 静的な部分は `.astro`、インタラクティブな部分だけ React などで書ける
- 既存の React / Vue / Svelte コンポーネント資産を部分的に再利用できる
- ページ全体を SPA 化せず、必要な島だけフレームワークを載せる

> 参照: [Astro — Framework components](https://docs.astro.build/en/guides/framework-components/)

## 対応フレームワーク

| フレームワーク | インテグレーション | ファイル拡張子 |
|---|---|---|
| React | `@astrojs/react` | `.jsx` / `.tsx` |
| Preact | `@astrojs/preact` | `.jsx` / `.tsx` |
| Vue | `@astrojs/vue` | `.vue` |
| Svelte | `@astrojs/svelte` | `.svelte` |
| Solid | `@astrojs/solid-js` | `.jsx` / `.tsx` |

- 1プロジェクト内で複数フレームワークを混在できる
- ただし、混在しすぎるとバンドルと設計が複雑になる

## インストール手順（React の例）

```bash
npx astro add react
```

- ウィザードが `@astrojs/react` のインストールと `astro.config` の更新を行う
- 手動で行う場合は以下

```bash
npm install @astrojs/react react react-dom
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';

export default defineConfig({
  integrations: [react()],
});
```

### Vue / Svelte の例

```bash
npx astro add vue
# または
npx astro add svelte
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import vue from '@astrojs/vue';
import svelte from '@astrojs/svelte';

export default defineConfig({
  integrations: [vue(), svelte()],
});
```

> 参照: [Astro — @astrojs/react](https://docs.astro.build/en/guides/integrations-guide/react/)

## Astro ページでの使い方

```astro
---
// src/pages/index.astro
import Counter from '../components/Counter.jsx';
import Greeting from '../components/Greeting.vue';
import Toggle from '../components/Toggle.svelte';
---
<html lang="ja">
  <body>
    <h1>Framework components in Astro</h1>
    <Counter />
    <Greeting name="Astro" />
    <Toggle />
  </body>
</html>
```

- `.astro` ファイル内で import し、通常のHTMLタグのように使う
- Astro コンポーネントと同じページに混在できる

## client: 無し = 静的HTML

- **重要：`client:*` ディレクティブを付けない限り、フレームワークコンポーネントも静的HTMLとして出力される**
- クライアントJSは送られない
- 見た目は再現されるが、クリックや入力などのイベントは動かない

```astro
---
import LikeButton from '../components/LikeButton.jsx';
---

<!-- 静的HTMLのみ。JSは送られない -->
<LikeButton />

<!-- 島になる。インタラクティブに動く -->
<LikeButton client:load />
```

### 悪い例：動くはずのUIに client:* を付け忘れる

```astro
---
import SearchBox from '../components/SearchBox.jsx';
---
<!-- 入力しても反応しない。JSが無い -->
<SearchBox placeholder="検索..." />
```

### 良い例：インタラクティブな部品だけ client:* を付ける

```astro
---
import ArticleBody from '../components/ArticleBody.astro';
import SearchBox from '../components/SearchBox.jsx';
---
<ArticleBody />
<SearchBox client:idle placeholder="検索..." />
```

- 静的で足りる部分は `.astro` コンポーネントのまま
- イベントが必要な部分だけ `client:*` を付ける
- 詳細は [client:ディレクティブ](./client-directives.md) を参照

> 参照: [Astro — Hydrating interactive components](https://docs.astro.build/en/guides/framework-components/#hydrating-interactive-components)

## props の渡し方

```astro
---
import UserCard from '../components/UserCard.jsx';

const user = { name: 'Taro', role: 'Editor' };
---
<UserCard user={user} client:load />
```

- Astro からフレームワークコンポーネントへ props を渡せる
- `client:*` 付きの場合、props は **シリアライズ可能な値** に限定される
- 関数やクラスインスタンスはクライアント側で使えない

## 混在時の注意点

### 1. フレームワーク実行時の重複

- 同じフレームワークの島が複数あっても、実行時は1回だけ送られる（最適化される）
- **異なるフレームワーク** を混在させると、それぞれの実行時が必要になる

```astro
---
import ReactWidget from '../components/ReactWidget.jsx';
import VueWidget from '../components/VueWidget.vue';
---
<!-- React 実行時 + Vue 実行時の両方が必要 -->
<ReactWidget client:load />
<VueWidget client:load />
```

- 可能なら1フレームワークに統一する方がバンドルは軽い

### 2. スタイルのスコープ

- Astro の `<style>` スコープは `.astro` コンポーネントに効く
- React / Vue コンポーネント内のスタイルは各フレームワークの仕組みに従う
- グローバルCSSや Tailwind などで統一するのが現実的

### 3. SSR と client:only

- `client:only` はサーバー描画をスキップし、クライアントのみで描画する
- ブラウザAPI前提のコンポーネント向け
- 使えるなら SSR ありの `client:load` / `client:visible` を優先する

```astro
<!-- ブラウザAPI前提。SSRできない -->
<Chart client:only="react" />
```

### 4. .astro 内にフレームワークコンポーネントをネスト

- `.astro` → React → `.astro` のような入れ子は **できない**
- フレームワークコンポーネントの中に Astro コンポーネントは置けない
- レイアウトの骨格は `.astro`、葉のインタラクションだけフレームワーク、という分担が基本

```astro
<!-- 良い例：Astro が骨格、React が葉 -->
---
import Layout from '../layouts/Layout.astro';
import Counter from '../components/Counter.jsx';
---
<Layout>
  <h1>タイトル</h1>
  <Counter client:load />
</Layout>
```

## フレームワークコンポーネント内での Astro コンポーネント

- **不可** — React コンポーネント内で `.astro` を import して使うことはできない
- 逆方向（Astro → フレームワーク）は可能
- 共通UIは `.astro` でラップするか、フレームワーク側で完結させる

## 選び方の目安

| 状況 | おすすめ |
|---|---|
| 静的な見出し・本文・カード | `.astro` |
| 既存 React 資産の再利用 | `@astrojs/react` + `client:*` |
| 軽量なインタラクション | Preact や Solid も検討 |
| チームが Vue 統一 | `@astrojs/vue` に寄せる |
| ページ全体がアプリ的 | Astro より Next.js 等を検討 |

## 設計チェックリスト

- [ ] 必要なインテグレーションを `astro.config` に追加した
- [ ] インタラクティブな部品だけ `client:*` を付けている
- [ ] 静的で足りる部分をフレームワークコンポーネントにしていない
- [ ] 複数フレームワーク混在の理由が説明できる
- [ ] props に関数など非シリアライズ可能な値を渡していない

## まとめ

- `@astrojs/react` などのインテグレーションで React / Vue / Svelte を Astro に統合できる
- `client:*` 無しではフレームワークコンポーネントも静的HTMLになり、JSは送られない
- インタラクティブにしたい部品だけ `client:*` を付けるのが基本
- 複数フレームワーク混在は可能だが、バンドルと設計が複雑になりやすい
- 骨格は `.astro`、葉の操作はフレームワーク、という分担が Astro の定石

> 参照: [Astroとは何か](./what-is-astro.md) / [Astroのアイランドアーキテクチャ](./islands-architecture.md)
