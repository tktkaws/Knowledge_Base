# 2.4.8 現在位置（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.8「現在位置」（Location）
- レベル **AAA**（最高の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- **Webページの集合内**で、ユーザーの現在位置を示す情報が利用できること
- サイトやアプリのどこにいるかを把握しやすくする

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.8 Location](https://www.w3.org/TR/WCAG22/#location)
> - [Understanding SC 2.4.8 Location](https://www.w3.org/WAI/WCAG22/Understanding/location.html)

## 達成基準の原文（要約）

- Webページの集合内におけるユーザーの位置についての情報が利用できること

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.8](https://www.w3.org/TR/WCAG22/#location)

## Webページの集合とは

- 共通の目的を持ち、同じ作者・グループ・組織が作成した Webページの集まり
- 例：同一サイト内の部門ページ・製品カテゴリ・ドキュメント階層
- 言語が異なるバージョンは、別の集合として扱う
- 単一ページだけのサイトでは、この基準の適用場面が限られる

```text
集合の例：
  大学サイト内の研究室ページ群
  ポータル内のカテゴリ階層
  ドキュメントサイトの章立てページ

別集合になりやすい例：
  日本語版と英語版
  別組織が運営する関連サイト
```

> **参照**
> - [Understanding SC 2.4.8 — Key Terms: set of web pages](https://www.w3.org/WAI/WCAG22/Understanding/location.html#dfn-set-of-web-pages)

## なぜ必要か

- 注意力が続きにくい人は、長いナビ経路の途中で「今どこか」を見失いやすい
- 深い階層へ直接リンクしてきた場合、周囲の文脈が分からないことがある
- 現在位置が分かると、上位階層へ戻ったり、関連情報を探したりしやすい
- 認知障害のあるユーザーの混乱を減らせる

> **参照**
> - [Understanding SC 2.4.8 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/location.html#benefits)

## 適合の主な経路

| 手法 | 内容 |
|---|---|
| G65 | パンくずリストを提供する |
| G63 | サイトマップを提供する |
| G128 | ナビゲーションバーで現在位置を示す |
| ARIA26 | `aria-current` で現在項目を識別する |
| G127 | ページと大きな集合との関係を識別する |

- 実務では、パンくず + ナビの現在位置表示の組み合わせが多い
- サイトマップは「全体の地図」、パンくずは「今いる経路」を示す
- ページタイトルで集合内の位置を示す方法（G127）も有効

```text
よくある組み合わせ：
  パンくず + グローバルナビの現在位置
  サイトマップ + ページタイトルの階層表現
  ナビの aria-current + パンくず
```

> **参照**
> - [Understanding SC 2.4.8 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/location.html#techniques)

## 手法1：パンくずリスト（G65）

- ホームから現在ページまでの階層経路を示す
- 各階層へ戻れるリンクがあると、位置把握と移動の両方に役立つ
- 現在ページはリンクにしない、または `aria-current="page"` を付ける

```html
<!-- 良い例：パンくずで現在位置を示す -->
<nav aria-label="パンくず">
  <ol>
    <li><a href="/">ホーム</a></li>
    <li><a href="/products">製品</a></li>
    <li><a href="/products/audio">オーディオ</a></li>
    <li aria-current="page">ワイヤレスイヤホン X1</li>
  </ol>
</nav>
```

```html
<!-- 悪い例：現在位置が分からない -->
<!-- 深い商品ページだけがあり、上位階層への手がかりがない -->
<main>
  <h1>ワイヤレスイヤホン X1</h1>
</main>
```

> **参照**
> - [Technique G65: Providing a breadcrumb trail](https://www.w3.org/WAI/WCAG22/Techniques/general/G65)
> - [Understanding SC 2.4.8 — breadcrumb example](https://www.w3.org/WAI/WCAG22/Understanding/location.html#examples)

## 手法2：ナビバーで現在位置を示す（G128 / ARIA26）

- グローバルナビやローカルナビで、今いるページ／セクションを視覚的に示す
- `aria-current="page"` または `aria-current="true"` で支援技術にも伝える
- 見た目だけ（色や太字）に頼らず、プログラム上でも識別できるようにする

```html
<!-- 良い例：ナビで現在ページを示す -->
<nav aria-label="グローバル">
  <ul>
    <li><a href="/">ホーム</a></li>
    <li><a href="/products" aria-current="page">製品</a></li>
    <li><a href="/support">サポート</a></li>
    <li><a href="/about">会社情報</a></li>
  </ul>
</nav>
```

```css
/* 視覚的にも現在位置が分かるようにする */
[aria-current="page"] {
  font-weight: bold;
  text-decoration: none;
  border-bottom: 2px solid currentColor;
}
```

```html
<!-- 悪い例：見た目だけで、支援技術に伝わらない -->
<nav>
  <a href="/">ホーム</a>
  <a href="/products" class="active">製品</a>
  <!-- class="active" だけでは不十分な場合がある -->
</nav>
```

> **参照**
> - [Technique G128: Indicating current location within navigation bars](https://www.w3.org/WAI/WCAG22/Techniques/general/G128)
> - [Technique ARIA26: Using aria-current to identify the current item in a set](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA26)
> - [a11y/aria-current.md — aria-current の使い方](../a11y/aria-current.md)

## 手法3：サイトマップ（G63）

- サイト全体の構造を一覧できるページを用意する
- ユーザーが「全体のどこにいるか」を把握しやすくなる
- 2.4.5（複数の手段）でも十分な手法として使われる

```html
<!-- 良い例：サイトマップで全体位置を把握できる -->
<main>
  <h1>サイトマップ</h1>
  <ul>
    <li><a href="/">ホーム</a></li>
    <li>
      <a href="/products">製品</a>
      <ul>
        <li><a href="/products/audio">オーディオ</a></li>
        <li><a href="/products/camera">カメラ</a></li>
      </ul>
    </li>
  </ul>
</main>
```

> **参照**
> - [Technique G63: Providing a site map](https://www.w3.org/WAI/WCAG22/Techniques/general/G63)

## 手法4：上位組織・集合との関係を示す（G127）

- ページが、より大きな集合のどこに属するかを示す
- 例：研究室ページから学科・大学ホームへのリンク
- ページタイトルで `個別ページ | セクション | サイト名` と示す方法もある

```html
<!-- 良い例：上位組織へのリンクで位置関係が分かる -->
<header>
  <p>
    <a href="https://www.example-univ.ac.jp/">サンプル大学</a>
    /
    <a href="https://www.example-univ.ac.jp/science/">理学部</a>
    /
    <span aria-current="page">高橋研究室</span>
  </p>
  <h1>高橋研究室</h1>
</header>
```

```html
<!-- 良い例：title で集合内の位置を示す -->
<title>インストール | ドキュメント | サンプルアプリ</title>
```

> **参照**
> - [Technique G127: Identifying a web page's relationship to a larger collection of web pages](https://www.w3.org/WAI/WCAG22/Techniques/general/G127)
> - [Understanding SC 2.4.8 — research group example](https://www.w3.org/WAI/WCAG22/Understanding/location.html#examples)
> - [wcag/2-4-2-page-titled.md — 2.4.2 ページタイトル](./2-4-2-page-titled.md)

## 2.4.5・2.4.2 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.2 ページタイトル | ページの話題・目的の識別 | タブや検索結果での識別 |
| 2.4.5 複数の手段 | ページへの到達手段が複数 | どう行くか |
| 2.4.8 現在位置 | 集合内での現在位置 | 今どこにいるか |

- 2.4.5 は「目的ページへの行き方」、2.4.8 は「今いる場所の把握」
- パンくずやサイトマップは、両方に役立つことが多い
- ページタイトルの階層表現は、2.4.2 と 2.4.8（G127）の両方を補強する

```text
2.4.5 では適合しうるが 2.4.8 では弱い例：
  検索とグローバルナビはあるが、現在位置の表示がない

2.4.8 に適合しやすい例：
  パンくずがある
  ナビに aria-current がある
  タイトルが「ページ | セクション | サイト」になっている
```

> **参照**
> - [wcag/2-4-5-multiple-ways.md — 2.4.5 複数の手段](./2-4-5-multiple-ways.md)

## 実装時の注意点

- パンくずは視覚的に分かるだけでなく、リスト構造とリンクでマークアップする
- 現在ページを色だけで示さず、`aria-current` も併用する
- SPA ではルート変更時にパンくず・`aria-current`・`document.title` を更新する
- 深い階層へランディングしたユーザーでも、上位へ戻れるようにする
- サイトマップは最新の情報構造と一致させる

```text
実務チェック：
  下層ページで「今どこか」が分かるか
  上位階層へ戻れるか
  ナビの現在位置が支援技術にも伝わるか
  直接リンクで来ても迷わないか
```

## テスト・確認方法

### 手動テスト

- サイトの下層ページを開く
- パンくず、ナビの現在位置、タイトルなどから位置が分かるか確認する
- 上位ページへ戻れるか確認する
- スクリーンリーダーで `aria-current` が伝わるか確認する
- 深い URL へ直接アクセスしても、位置情報が得られるか確認する

```text
チェックリスト：
1. Webページの集合内のページか
2. 現在位置を示す情報があるか
3. パンくず／ナビ／サイトマップ／タイトルなどのどれか（または複数）か
4. 見た目だけでなく、プログラム上でも現在位置が分かるか
5. 上位階層や関連ページへ移動できるか
```

### 開発者ツールでの確認

```bash
# パンくず・aria-current・サイトマップまわりを探す
rg "aria-current|breadcrumb|パンくず|sitemap|サイトマップ" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.php" -i
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.8 は、ページ集合内で**ユーザーの現在位置が分かる情報**を求める達成基準（レベル AAA）
- 代表的な手段は、パンくず（G65）、ナビの現在位置表示（G128 / ARIA26）、サイトマップ（G63）、集合との関係の明示（G127）
- 深い階層へ直接来たユーザーや、認知的な負荷が高いユーザーの迷いを減らす
- 2.4.5（どう行くか）と対になる「今どこにいるか」の基準
- 見た目の強調だけでなく、`aria-current` や構造化マークアップで支援技術にも伝える
