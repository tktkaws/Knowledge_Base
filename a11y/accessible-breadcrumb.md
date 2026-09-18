# アクセシブルなパンくずリストの実装

## パンくずリストとは

- 現在ページまでの階層パスを、上位から順に並べたナビゲーション
- サイト内での位置把握と、上位ページへの移動を助ける
- 通常はメインコンテンツの直前に横並びで配置する
- 専用のキーボード操作は不要 — 通常のリンク操作で足りる

> 参照: [WAI-ARIA Authoring Practices — Breadcrumb Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/breadcrumb/)

## 必要な構造とARIA

| 要素 / 属性 | 役割 |
|---|---|
| `<nav>` | ナビゲーションランドマークにする |
| `aria-label` / `aria-labelledby` | 「パンくず」など用途が分かる名前 |
| `<ol>`（推奨）または `<ul>` | 階層の並びをリストとして示す |
| 祖先ページ | `<a>` リンク |
| 現在ページのリンク | `aria-current="page"` |

- 現在ページがリンクでない（テキストのみ）場合、`aria-current` は任意
- ページ内に複数の `<nav>` があるとき、ラベルで区別する（例: 「パンくず」「メイン」）

> 参照: [APG — Breadcrumb Example](https://www.w3.org/WAI/ARIA/apg/patterns/breadcrumb/examples/breadcrumb/)

## HTML構造

```html
<nav aria-label="パンくず">
  <ol>
    <li><a href="/">ホーム</a></li>
    <li><a href="/products">製品</a></li>
    <li><a href="/products/laptops">ノートPC</a></li>
    <li>
      <a href="/products/laptops/x1" aria-current="page">
        モデル X1
      </a>
    </li>
  </ol>
</nav>
```

### 現在ページをリンクにしない場合

```html
<nav aria-label="パンくず">
  <ol>
    <li><a href="/">ホーム</a></li>
    <li><a href="/products">製品</a></li>
    <li><a href="/products/laptops">ノートPC</a></li>
    <li aria-current="page">モデル X1</li>
  </ol>
</nav>
```

- どちらでも可。リンクにする場合は `aria-current="page"` をリンクに付ける
- テキストのみにする場合は、視覚的にもリンクと区別する

## 区切り記号の扱い

- `/` や `>` をHTMLテキストとして入れると、スクリーンリーダーが毎回読み上げる
- 区切りはCSSで表現し、アクセシビリティツリーに載せない

```css
.breadcrumb {
  list-style: none;
  display: flex;
  flex-wrap: wrap;
  gap: 0;
  padding: 0;
  margin: 0;
}

.breadcrumb li {
  display: flex;
  align-items: center;
}

.breadcrumb li:not(:last-child)::after {
  content: "/";
  margin-inline: 0.5rem;
  speak: never; /* 対応ブラウザ向け。根本対策はCSS装飾であること */
  /* 読み上げ抑制の本命は、区切りをリスト構造外の装飾にすること */
}
```

```css
/* APG例に近い方法：リンクの境界を斜めボーダーで表現し、文字区切りを避ける */
.breadcrumb a {
  display: inline-block;
  position: relative;
  padding-inline-end: 1rem;
}

.breadcrumb li:not(:last-child) a::after {
  content: "";
  position: absolute;
  inset-block: 0.2em;
  inset-inline-end: 0.35rem;
  border-inline-end: 1px solid currentColor;
  transform: skew(-15deg);
}
```

- `content: "/"` でも多くのスクリーンリーダーは読み上げないが、環境差がある
- 文字ではなくボーダー等の装飾にする方が、冗長な読み上げを避けやすい

## React での実装例

```jsx
function Breadcrumb({ items }) {
  return (
    <nav aria-label="パンくず">
      <ol className="breadcrumb">
        {items.map((item, index) => {
          const isLast = index === items.length - 1;

          return (
            <li key={item.href} {...(!item.href && isLast ? { 'aria-current': 'page' } : {})}>
              {isLast ? (
                item.href ? (
                  <a href={item.href} aria-current="page">
                    {item.label}
                  </a>
                ) : (
                  item.label
                )
              ) : (
                <a href={item.href}>{item.label}</a>
              )}
            </li>
          );
        })}
      </ol>
    </nav>
  );
}

// 使用例
<Breadcrumb
  items={[
    { label: 'ホーム', href: '/' },
    { label: '製品', href: '/products' },
    { label: 'ノートPC', href: '/products/laptops' },
    { label: 'モデル X1', href: '/products/laptops/x1' },
  ]}
/>
```

## 構造化データ（任意）

- SEOやリッチリザルト向けに JSON-LD の `BreadcrumbList` を併用することがある
- アクセシビリティ要件とは別レイヤー。見た目のパンくずと内容を一致させる

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "ホーム",
      "item": "https://example.com/"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "製品",
      "item": "https://example.com/products"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "モデル X1",
      "item": "https://example.com/products/laptops/x1"
    }
  ]
}
</script>
```

> 参照: [schema.org — BreadcrumbList](https://schema.org/BreadcrumbList)

## スタイルの注意点

```css
.breadcrumb a[aria-current="page"] {
  font-weight: 700;
  text-decoration: none;
  color: inherit;
  /* 色だけに頼らない。太字など別の手がかりも使う */
}

.breadcrumb a:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

- 現在ページは色差だけに依存しない
- 長い階層では折り返しを許可し、タップ／クリック領域を確保する

## よくある間違い

### 1. nav にラベルがない

```html
<!-- 間違い：他のナビと区別できない -->
<nav>
  <ol>...</ol>
</nav>

<!-- 正解 -->
<nav aria-label="パンくず">
  <ol>...</ol>
</nav>
```

### 2. リストにせず span だけで並べる

```html
<!-- 間違い：項目数や順序が伝わりにくい -->
<nav aria-label="パンくず">
  <a href="/">ホーム</a> / <a href="/a">A</a> / <span>現在</span>
</nav>

<!-- 正解：ol / ul で項目化する -->
```

### 3. 現在位置を aria-current で示さない

```html
<!-- 間違い：見た目の active クラスだけ -->
<li><a href="/current" class="active">現在のページ</a></li>

<!-- 正解 -->
<li><a href="/current" aria-current="page">現在のページ</a></li>
```

### 4. 区切り文字をDOMに入れて読み上げさせる

```html
<!-- 避ける例 -->
<li><a href="/">ホーム</a> /</li>
<li><a href="/a">製品</a> &gt;</li>
```

- 区切りはCSSで表現する

### 5. パンくずなのに見出し階層やメインナビの代替にしてしまう

- パンくずは補助ナビ。ページ固有の見出し（`h1` など）は別途必要
- メインナビゲーションの代わりにはならない

### 6. 階層と一致しないリンクを並べる

- 表示ラベルと実際のURL階層が食い違うと、位置把握の助けにならない
- 省略（`...`）を使う場合も、到達可能な上位ページへの導線を残す

## 実装チェックリスト

- [ ] `<nav>` で囲み、`aria-label`（または `aria-labelledby`）がある
- [ ] 項目が `<ol>` または `<ul>` のリストになっている
- [ ] 祖先ページがリンクになっている
- [ ] 現在ページに `aria-current="page"` がある（リンクの場合は必須）
- [ ] 区切り記号がスクリーンリーダーで冗長に読まれない
- [ ] 現在ページが視覚的にも区別できる（色だけに依存しない）
- [ ] フォーカスインジケーターが見える
- [ ] リンク先が実際の階層と一致している

## まとめ

- パンくずは「名前付きの `nav` + リスト + 現在ページの `aria-current="page"`」が基本
- 追加のキーボード実装は不要。通常のリンク操作で十分
- 区切り記号はCSSで付け、DOM上の `/` や `>` の読み上げを避ける
- 見た目の `active` だけでは不十分。プログラム的に現在位置を示す
- JSON-LD は任意の補強であり、HTMLのアクセシブルなマークアップの代替にはならない

> 参照: [WCAG Technique G65: Providing a breadcrumb trail](https://www.w3.org/WAI/WCAG22/Techniques/general/G65)
