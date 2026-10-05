# Astroのスコープ付きCSS

## スコープ付きCSSとは

- Astroコンポーネント内の `<style>` タグに書いたCSSが、**そのコンポーネントだけ**に効く仕組み
- ビルド時にクラスや属性セレクタへ一意の `data-astro-cid-*` が付与される
- 他コンポーネントの同名クラスと衝突しにくい
- React の CSS Modules や Vue の scoped CSS に近い考え方

> 参照: [Astro — Styling in Astro](https://docs.astro.build/en/guides/styling/)

## 自動スコープの仕組み

- コンポーネント内の `<style>` は、デフォルトでスコープ付き
- セレクタに `data-astro-cid-xxxxx` が自動付与される
- コンポーネント外の要素には当たらない

```astro
<!-- コンポーネント内 -->
<style>
  h1 {
    color: red;
  }
  .text {
    color: blue;
  }
</style>

<h1>タイトル</h1>
<p class="text">本文</p>
```

```css
/* ビルド後の出力イメージ */
h1[data-astro-cid-hhnqfkh6] {
  color: red;
}
.text[data-astro-cid-hhnqfkh6] {
  color: blue;
}
```

- 追加設定なしでコンポーネント単位のスタイル分離ができる
- グローバルCSSファイルを増やさずに済むことが多い

## スコープ付きCSSの基本

- コンポーネントの見た目は、その `.astro` ファイル内の `<style>` に書く
- 親子関係のあるコンポーネントでも、子のルート要素には親のスコープ属性が付かない
- 子コンポーネントへスタイルを渡すには `class` props を使う

```astro
---
// Card.astro
const { class: className, ...rest } = Astro.props;
---
<div class:list={['card', className]} {...rest}>
  <slot />
</div>

<style>
  .card {
    border: 1px solid #ccc;
    padding: 1rem;
  }
</style>
```

```astro
---
// Page.astro
import Card from '../components/Card.astro';
---
<Card class="featured" />
```

- 子の `.card` は子コンポーネント内のスコープに閉じる
- 親から `class="featured"` を渡しても、親の `<style>` では `.featured` を直接当てられない

> 参照: [Astro — Passing classes to child components](https://docs.astro.build/en/guides/styling/#passing-classes-to-child-components)

## :global() — スコープ内からグローバル指定

- スコープ付き `<style>` の中で、一部だけグローバルに当てたいときに `:global()` を使う
- slot 経由で入る Markdown や CMS コンテンツへのスタイル当てに向く
- コンポーネント全体をグローバル化するより、対象を限定する方が安全

```astro
<!-- 良い例：slot 内の見出しだけグローバル指定 -->
<style>
  h1 {
    color: red; /* このコンポーネントの h1 のみ */
  }
  article :global(h1) {
    color: blue; /* slot 内の h1 に適用 */
  }
</style>

<h1>ページタイトル</h1>
<article><slot /></article>
```

```astro
<!-- 悪い例：意図せず広範囲に漏れる -->
<style>
  :global(a) {
    color: hotpink; /* サイト全体のリンク色を変えてしまう */
  }
</style>
```

- `:global()` は強力だが、デバッグが難しくなるケースがある
- 「このコンポーネントの子孫だけ」という意図を明確にする

> 参照: [Astro — Mix scoped and global styles](https://docs.astro.build/en/guides/styling/#mix-scoped-and-global-styles)

## is:global — style タグ全体をグローバル化

- `<style is:global>` で、その style ブロック全体のスコープを無効化
- リセットCSSや typographic ベースラインなど、サイト全体に効かせたいときに使う
- コンポーネント内に書いても、出力はグローバルCSSになる

```astro
<!-- 良い例：レイアウトコンポーネントでベーススタイルを1か所に -->
<style is:global>
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }
</style>
```

```astro
<!-- 悪い例：各コンポーネントで is:global を乱用 -->
<style is:global>
  .button { background: blue; }
</style>
<!-- 同名 .button が他コンポーネントと衝突しやすい -->
```

- `is:global` は `:global()` より影響範囲が広い
- 可能なら `src/styles/global.css` への切り出しも検討する

> 参照: [Astro — Global Styles](https://docs.astro.build/en/guides/styling/#global-styles)

## class:list — 条件付きクラス名

- `class:list` ディレクティブで、配列・オブジェクト・文字列を組み合わせてクラスを動的付与
- スコープ付きCSSのクラス名と相性が良い
- 三項演算子で `class` を連結するより読みやすい

```astro
---
const { variant, disabled } = Astro.props;
---

<!-- 良い例 -->
<button
  class:list={[
    'btn',
    { 'btn-primary': variant === 'primary' },
    { 'btn-disabled': disabled },
  ]}
>
  <slot />
</button>

<style>
  .btn { padding: 0.5rem 1rem; }
  .btn-primary { background: blue; color: white; }
  .btn-disabled { opacity: 0.5; pointer-events: none; }
</style>
```

```astro
---
const { variant, disabled } = Astro.props;
const classes = `btn ${variant === 'primary' ? 'btn-primary' : ''} ${disabled ? 'btn-disabled' : ''}`;
---

<!-- 悪い例：余分な空白や typo が入りやすい -->
<button class={classes}><slot /></button>
```

> 参照: [Astro — Combining classes with class:list](https://docs.astro.build/en/guides/styling/#combining-classes-with-classlist)

## CSS Modules との関係

- `.module.css` ファイルを import すると、CSS Modules として扱われる
- クラス名がビルド時に一意化されたオブジェクトとして返る
- Astro コンポーネントでも React / Vue コンポーネントでも利用可能

```astro
---
import styles from './Alert.module.css';
const { type = 'info' } = Astro.props;
---

<div class:list={[styles.alert, styles[type]]}>
  <slot />
</div>
```

```css
/* Alert.module.css */
.alert {
  padding: 1rem;
  border-radius: 4px;
}
.info {
  background: #e0f2fe;
}
.error {
  background: #fee2e2;
}
```

- Astro の `<style>` スコープと CSS Modules は別系統
- どちらも「スコープを切る」手段だが、使い分けの目安がある

| 手段 | 向く場面 |
|---|---|
| `<style>`（自動スコープ） | `.astro` コンポーネント専用のスタイル |
| CSS Modules | 複数フレームワークで共有するUI、クラス名を厳密に管理したい |
| グローバルCSS | リセット、タイポグラフィ、デザイントークン |

```astro
<!-- 悪い例：同じ見た目を二重管理 -->
<style>
  .alert { padding: 1rem; }
</style>
<!-- かつ Alert.module.css にも .alert がある -->
```

> 参照: [Astro — CSS Modules](https://docs.astro.build/en/guides/imports/#css-modules)

## スコープ付きCSSの落とし穴

### 1. 子コンポーネントへ親のセレクタが届かない

```astro
<!-- 悪い例：Card 内部の .title を親から指定しようとする -->
<style>
  .card .title {
    font-size: 2rem; /* Card 内の .title には当たらない */
  }
</style>
<Card />
```

- 子の DOM は別スコープ
- 子側でスタイルを書くか、`class` props で修飾クラスを渡す

### 2. :global() / is:global の使いすぎ

- スコープの恩恵が薄れ、発生源の追跡が難しくなる
- レイアウト / global.css / is:global の役割分担を決めておく

## まとめ

- Astro の `<style>` はデフォルトでコンポーネントスコープになる
- 子要素や slot 向けには `:global()`、サイト全体には `is:global` または global.css を使う
- `class:list` で条件付きクラスを安全に組み立てられる
- CSS Modules は `.module.css` import で利用でき、フレームワーク横断のUIに向く
- スコープ付きを基本に、グローバルは必要最小限に留めるのが要点
