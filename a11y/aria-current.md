# aria-current — ナビゲーションの現在位置を示す

## aria-currentとは

- 一連の要素の中から「現在のもの」を示すためのARIA属性
- ナビゲーションメニューで「今いるページ」を支援技術に伝える最も適切な方法
- 視覚的なハイライト（太字、下線、色の変化）だけでは支援技術に現在位置が伝わらない
- `aria-current` を指定すると、スクリーンリーダーが「現在の○○」と読み上げる

> **参照**
> - [W3C - aria-current (state)](https://www.w3.org/TR/wai-aria-1.2/#aria-current)
> - [MDN - aria-current](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-current)

## aria-currentの値

| 値 | 意味 | 使用場面 |
|---|---|---|
| `page` | 現在のページ | ナビゲーションリンク |
| `step` | 現在のステップ | ステッパー / ウィザード |
| `location` | 現在の位置 | フローチャート / サイトマップ |
| `date` | 現在の日付 | カレンダー |
| `time` | 現在の時刻 | タイムテーブル |
| `true` | 汎用的な「現在の項目」 | 上記に当てはまらない場合 |
| `false`（デフォルト） | 現在の項目ではない | 明示的に否定する場合 |

> **参照**
> - [W3C - WAI-ARIA 1.2 - aria-current](https://www.w3.org/TR/wai-aria-1.2/#aria-current)

## 基本的な使い方

### ナビゲーションメニュー（aria-current="page"）

```html
<!-- 悪い例：視覚的なハイライトのみでスクリーンリーダーには伝わらない -->
<nav aria-label="メインナビゲーション">
  <ul>
    <li><a href="/" class="active">ホーム</a></li>
    <li><a href="/products">製品</a></li>
    <li><a href="/about">会社概要</a></li>
  </ul>
</nav>
<!-- スクリーンリーダー：「ホーム、リンク」→ 現在のページかどうか分からない -->

<!-- 良い例：aria-current="page"で現在のページを明示 -->
<nav aria-label="メインナビゲーション">
  <ul>
    <li><a href="/" aria-current="page">ホーム</a></li>
    <li><a href="/products">製品</a></li>
    <li><a href="/about">会社概要</a></li>
  </ul>
</nav>
<!-- スクリーンリーダー：「ホーム、現在のページ、リンク」 -->
```

### CSSでのスタイリング

- `aria-current` は属性セレクタでスタイリングに使える
- `.active` クラスの代わりに `[aria-current="page"]` を使うことで、スタイルとセマンティクスを一致させられる

```css
/* 悪い例：.activeクラスで視覚的にのみ区別 */
.nav-link.active {
  font-weight: bold;
  border-bottom: 2px solid currentColor;
}

/* 良い例：aria-current属性セレクタでスタイリング */
.nav-link[aria-current="page"] {
  font-weight: bold;
  border-bottom: 2px solid currentColor;
}
```

- この方法の利点：
  - スタイルとアクセシビリティが常に同期する（片方だけ付け忘れることがない）
  - `.active` クラスの追加・削除が不要になる
  - CSSだけ見てもアクセシビリティ対応であることが分かる

> **参照**
> - [Adrian Roselli - aria-current](https://adrianroselli.com/2017/10/aria-current-with-css.html)

## 実践的なパターン

### パンくずリスト

```html
<nav aria-label="パンくずリスト">
  <ol>
    <li><a href="/">ホーム</a></li>
    <li><a href="/products">製品</a></li>
    <li><a href="/products/widget" aria-current="page">ウィジェット</a></li>
  </ol>
</nav>
```

### ステッパー / ウィザード

```html
<!-- 3ステップの注文フロー、現在はステップ2 -->
<nav aria-label="注文の進捗">
  <ol>
    <li>
      <a href="/cart">カート確認</a>
    </li>
    <li>
      <span aria-current="step">配送先入力</span>
    </li>
    <li>
      <span>お支払い</span>
    </li>
  </ol>
</nav>
<!-- スクリーンリーダー：「配送先入力、現在のステップ」 -->
```

### サイドバーナビゲーション

```html
<nav aria-label="ドキュメント">
  <ul>
    <li>
      <span>はじめに</span>
      <ul>
        <li><a href="/docs/install">インストール</a></li>
        <li><a href="/docs/quickstart" aria-current="page">クイックスタート</a></li>
      </ul>
    </li>
    <li>
      <span>ガイド</span>
      <ul>
        <li><a href="/docs/components">コンポーネント</a></li>
        <li><a href="/docs/routing">ルーティング</a></li>
      </ul>
    </li>
  </ul>
</nav>
```

### カレンダー

```html
<table role="grid" aria-label="2026年3月">
  <thead>
    <tr>
      <th scope="col" abbr="日">日</th>
      <th scope="col" abbr="月">月</th>
      <!-- ... -->
    </tr>
  </thead>
  <tbody>
    <tr>
      <!-- ... -->
      <td><button aria-current="date">29</button></td>
      <td><button>30</button></td>
      <td><button>31</button></td>
    </tr>
  </tbody>
</table>
<!-- スクリーンリーダー：「29、現在の日付」 -->
```

> **参照**
> - [W3C APG - Breadcrumb Example](https://www.w3.org/WAI/ARIA/apg/patterns/breadcrumb/examples/breadcrumb/)

## フレームワークでの実装

### React

```jsx
function NavLink({ href, children }) {
  const pathname = usePathname();
  const isCurrent = pathname === href;

  return (
    <a
      href={href}
      aria-current={isCurrent ? "page" : undefined}
    >
      {children}
    </a>
  );
}

function Navigation() {
  return (
    <nav aria-label="メインナビゲーション">
      <ul>
        <li><NavLink href="/">ホーム</NavLink></li>
        <li><NavLink href="/products">製品</NavLink></li>
        <li><NavLink href="/about">会社概要</NavLink></li>
      </ul>
    </nav>
  );
}
```

### Next.js（App Router）

```jsx
"use client";
import Link from "next/link";
import { usePathname } from "next/navigation";

function NavLink({ href, children }) {
  const pathname = usePathname();
  const isCurrent = pathname === href;

  return (
    <Link
      href={href}
      aria-current={isCurrent ? "page" : undefined}
    >
      {children}
    </Link>
  );
}
```

> **参照**
> - [Next.js - usePathname](https://nextjs.org/docs/app/api-reference/functions/use-pathname)

## aria-selectedとの違い

| 属性 | 目的 | 主な使用場面 |
|---|---|---|
| `aria-current` | 一連の中で「現在のもの」を示す | ナビゲーション、パンくず、ステッパー |
| `aria-selected` | 選択可能な要素の中で「選択中のもの」を示す | タブ、リストボックス、グリッドセル |

```html
<!-- aria-current：ナビゲーションの現在ページ -->
<nav>
  <a href="/" aria-current="page">ホーム</a>
  <a href="/about">概要</a>
</nav>

<!-- aria-selected：タブの選択状態 -->
<div role="tablist">
  <button role="tab" aria-selected="true">タブ1</button>
  <button role="tab" aria-selected="false">タブ2</button>
</div>
```

- `aria-current` はページ遷移を伴うナビゲーションに使用
- `aria-selected` はページ内のウィジェットの選択状態に使用
- 両者は目的が異なるため混同しない

> **参照**
> - [W3C - aria-selected](https://www.w3.org/TR/wai-aria-1.2/#aria-selected)
> - [Léonie Watson - Using aria-current](https://tink.uk/using-the-aria-current-attribute/)

## よくある間違い

### 1. クラス名だけで現在位置を示す

```html
<!-- 間違い：.activeクラスのみ -->
<a href="/" class="active">ホーム</a>

<!-- 正しい：aria-currentを併用 -->
<a href="/" class="active" aria-current="page">ホーム</a>

<!-- さらに良い：aria-currentをスタイルに利用し、クラスを廃止 -->
<a href="/" aria-current="page">ホーム</a>
```

### 2. aria-current="true" を page の代わりに使う

```html
<!-- 不適切：意味が曖昧 -->
<a href="/" aria-current="true">ホーム</a>

<!-- 適切：ナビゲーションにはpageを使う -->
<a href="/" aria-current="page">ホーム</a>
```

- `true` は汎用的な値で動作はするが、`page` / `step` / `date` など具体的な値のほうがスクリーンリーダーがより適切に読み上げる

### 3. 全ページでaria-currentを付けっぱなしにする

```html
<!-- 間違い：全てのリンクにaria-currentが付いている -->
<a href="/" aria-current="page">ホーム</a>
<a href="/about" aria-current="page">概要</a>

<!-- 正しい：現在のページにのみ付与 -->
<a href="/" aria-current="page">ホーム</a>
<a href="/about">概要</a>
```

## まとめ

- `aria-current` はナビゲーションの「現在位置」を支援技術に伝えるための属性
- 値は用途に応じて `page`（ページ）、`step`（ステップ）、`date`（日付）などを使い分ける
- 視覚的なハイライト（.activeクラス）の代わりに `[aria-current="page"]` セレクタでスタイリングすると、アクセシビリティとスタイルが常に同期する
- `aria-selected` とは目的が異なる — ナビゲーションには `aria-current`、ウィジェットの選択状態には `aria-selected`
- SPAフレームワークではルーティングの状態に基づいて `aria-current` を動的に切り替える
