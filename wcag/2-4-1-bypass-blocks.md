# 2.4.1 ブロックスキップ（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.1「ブロックスキップ」（Bypass Blocks）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- **複数の Web ページで繰り返されるコンテンツのブロック**をスキップする手段があること
- キーボードやスクリーンリーダーで、メインコンテンツへ素早く到達できるようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.1 Bypass Blocks](https://www.w3.org/TR/WCAG22/#bypass-blocks)
> - [Understanding SC 2.4.1 Bypass Blocks](https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks.html)

## 達成基準の原文（要約）

- 複数の Web ページで繰り返されるコンテンツのブロックをバイパスする**メカニズム**が利用できること

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.1](https://www.w3.org/TR/WCAG22/#bypass-blocks)

## 繰り返されるブロックとは

- サイト内の複数ページで共通して現れるまとまったコンテンツ
- 例：グローバルナビ、ヘッダー、広告フレーム、共通サイドバー
- **単語・短いフレーズ・単一リンク**程度の小さな繰り返しは「ブロック」に含めない
- 対象は、関連するページ集合（同じ作者・組織が作った共通目的のページ群）を想定
- 無関係な2ページ間の偶然の重複まで避ける必要はない

```text
対象の例：
  全ページ共通のグローバルナビゲーション
  共通ヘッダー・フッター内のリンク群
  検索結果の前に毎回ある長いフィルター一覧
  広告やサービスのサイドバー

対象外になりやすい例：
  ページ内の同じ単語の繰り返し
  単一の「ホーム」リンクだけ
  互いに無関係なサイト間の似たヘッダー
```

> **参照**
> - [Understanding SC 2.4.1 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks.html#intent)

## なぜ必要か

- マウス利用者はメイン付近をクリックすればよいが、逐次ナビでは先頭から順に進む
- スクリーンリーダー利用者は、毎回ヘッダーや数十のナビリンクを聞かされる
- キーボード利用者は、メイン到達までに何十回も Tab が必要になることがある
- 一部の利用者にとっては、その操作が長時間化したり身体的負担になったりする
- 画面拡大利用者も、毎回ヘッダーを探してメインの開始位置を探す負担がある

> **参照**
> - [Understanding SC 2.4.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks.html#benefits)

## 適合の主な経路

| 経路 | 内容 | 代表的な手法 |
|---|---|---|
| スキップリンク | 繰り返すブロックを飛ばすリンクを置く | G1 / G123 / G124 |
| 構造でスキップ可能にする | ランドマーク・見出しなどで領域をまとめる | ARIA11 / H69 / H64 / SCR28 |

- スキップリンクとランドマーク・見出しは**併用が望ましい**
- ユーザーエージェント機能と冗長な手段まで必須ではない（例：ページ最下部のナビだけなら、ブラウザの先頭へ戻る機能で足りる場合がある）
- 単一ページ内の繰り返しブロックへのスキップは推奨だが、この基準の必須ではない

```text
推奨の組み合わせ：
1. ページ先頭の「メインコンテンツへ」スキップリンク
2. <main> / <nav> などのランドマーク
3. セクション先頭の見出し
```

## 手法1：メインへスキップするリンク（G1）

- 各ページの先頭に、メインコンテンツ領域へ直接行くリンクを置く
- キーボード利用者が最初に到達できる要素にする
- フォーカス時に見えるようにする（常時非表示のままにしない）

```html
<!-- 良い例：ページ先頭のスキップリンク -->
<body>
  <a href="#main-content" class="skip-link">メインコンテンツへスキップ</a>

  <header>
    <nav aria-label="グローバル">
      <ul>
        <li><a href="/">ホーム</a></li>
        <li><a href="/products">製品</a></li>
        <!-- 多数のリンク... -->
      </ul>
    </nav>
  </header>

  <main id="main-content" tabindex="-1">
    <h1>本日のニュース</h1>
    <p>メインの記事...</p>
  </main>
</body>
```

```css
/* 良い例：フォーカス時に見えるスキップリンク */
.skip-link {
  position: absolute;
  left: -9999px;
  top: 0;
  z-index: 1000;
}

.skip-link:focus {
  left: 0;
  padding: 0.75rem 1rem;
  background: #000;
  color: #fff;
}
```

```html
<!-- 悪い例：スキップ手段がなく、毎回ナビを全部 Tab する -->
<body>
  <header>
    <nav>
      <!-- 40個のリンク -->
    </nav>
  </header>
  <div class="content">
    <h1>本日のニュース</h1>
  </div>
</body>
```

> **参照**
> - [Technique G1: Adding a link at the top of each page that goes directly to the main content area](https://www.w3.org/WAI/WCAG22/Techniques/general/G1)
> - [a11y/skip-link.md — スキップリンクの実装と意義](../a11y/skip-link.md)

## 手法2：ブロックの前後にスキップリンクを置く（G123 / G124）

- **G123**：繰り返すブロックの先頭に、そのブロックの末尾へ飛ぶリンクを置く
- **G124**：ページ先頭に、各コンテンツ領域へ飛ぶリンク群を置く
- 長いフィルター一覧や複数サイドバーがあるページで有効

```html
<!-- 良い例：長いフィルターをスキップ（G123） -->
<a href="#product-results">フィルターをスキップして商品一覧へ</a>

<aside id="filters" aria-label="絞り込み">
  <!-- 多数のチェックボックス・リンク -->
</aside>

<section id="product-results">
  <h2>検索結果</h2>
  <!-- 商品一覧 -->
</section>
```

```html
<!-- 良い例：ページ先頭から各領域へ（G124） -->
<nav aria-label="ページ内ジャンプ">
  <ul>
    <li><a href="#main-content">本文へ</a></li>
    <li><a href="#search">検索へ</a></li>
    <li><a href="#related">関連情報へ</a></li>
  </ul>
</nav>
```

> **参照**
> - [Technique G123: Adding a link at the beginning of a block of repeated content to go to the end of the block](https://www.w3.org/WAI/WCAG22/Techniques/general/G123)
> - [Technique G124: Adding links at the top of the page to each area of the content](https://www.w3.org/WAI/WCAG22/Techniques/general/G124)
> - [Understanding SC 2.4.1 — e-commerce filters example](https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks.html#examples)

## 手法3：ARIA ランドマークで領域をまとめる（ARIA11）

- `<main>` / `<nav>` / `<header>` / `<footer>` / `<aside>` などで領域を識別する
- スクリーンリーダーのランドマーク一覧から、繰り返す領域を飛ばして移動できる
- 同じ種類のランドマークが複数ある場合は、accessible name で区別する
- 1.3.1「情報および関係性」でも十分な手法として使われる

```html
<!-- 良い例：セマンティックなランドマーク -->
<header>
  <p>サイト名</p>
</header>

<nav aria-label="グローバル">
  <!-- 共通ナビ -->
</nav>

<main>
  <h1>記事タイトル</h1>
  <p>本文...</p>
</main>

<aside aria-label="広告">
  <!-- サイドバー -->
</aside>

<footer>
  <!-- フッター -->
</footer>
```

```html
<!-- 悪い例：すべて div で、領域ジャンプできない -->
<div class="header">...</div>
<div class="nav">...</div>
<div class="content">...</div>
<div class="sidebar">...</div>
```

> **参照**
> - [Technique ARIA11: Using ARIA landmarks to identify regions of a page](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA11)
> - [wcag/1-3-1-info-and-relationships.md — 1.3.1 情報および関係性](./1-3-1-info-and-relationships.md)

## 手法4：見出しでセクションを始める（H69）

- 各セクションの先頭に見出し要素（`h1`〜`h6`）を置く
- スクリーンリーダーの見出しジャンプで、繰り返すブロックを越えられる
- ページ単体の構造マークアップとしても重要（1.3.1 / 2.4.6 / 2.4.10 とも関連）

```html
<!-- 良い例：セクション先頭に見出し -->
<nav aria-label="グローバル">
  <h2>サイトメニュー</h2>
  <ul>...</ul>
</nav>

<main>
  <h1>今日のトップ記事</h1>
  <p>...</p>
</main>
```

> **参照**
> - [Technique H69: Providing heading elements at the beginning of each section of content](https://www.w3.org/WAI/WCAG22/Techniques/html/H69)

## その他の手法

| 手法 | 内容 |
|---|---|
| H64 | `iframe` に `title` を付け、フレーム単位でスキップしやすくする |
| SCR28 | 折りたたみ可能なメニューで、展開前にブロックをバイパスできる |
| H97（参考） | 関連リンクを `nav` でグループ化する |

```html
<!-- 良い例：iframe に title（H64） -->
<iframe
  src="/ads/banner.html"
  title="広告：キャンペーン情報"
  width="300"
  height="250"
></iframe>
```

```html
<!-- 良い例：折りたたみメニュー（SCR28） -->
<button type="button" aria-expanded="false" aria-controls="global-nav">
  メニュー
</button>
<nav id="global-nav" hidden>
  <ul><!-- 多数のリンク --></ul>
</nav>
```

> **参照**
> - [Technique H64: Using the title attribute of the iframe element](https://www.w3.org/WAI/WCAG22/Techniques/html/H64)
> - [Technique SCR28: Using an expandable and collapsible menu to bypass block of content](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR28)

## 実装時の注意点

- スキップリンクのジャンプ先には `id` を付け、必要なら `tabindex="-1"` でフォーカスを受け取れるようにする
- `display: none` だけだとフォーカスできない実装に注意する
- スキップ後、キーボードフォーカスが実際にメイン側へ移ることを確認する
- ランドマークだけに頼ると、ランドマーク非対応のキーボード利用者には不十分な場合がある
- そのため、**スキップリンク + ランドマーク + 見出し**の併用が実務的

```html
<!-- 良い例：ジャンプ先がフォーカスを受け取れる -->
<main id="main-content" tabindex="-1">
  <h1>本文</h1>
</main>
```

```css
/* 注意：フォーカス不能な非表示は避ける */
.skip-link-bad {
  display: none; /* フォーカスできない */
}
```

## 1.3.1・2.4.6 などとの関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 1.3.1 情報および関係性 | 構造・関係をマークアップで示す | ランドマーク・見出しが共通手法 |
| 2.4.1 ブロックスキップ | 繰り返すブロックを飛ばす | スキップリンクや構造で実現 |
| 2.4.6 見出しおよびラベル | 見出し・ラベルの説明性 | 見出しの質 |
| 2.4.10 セクション見出し | セクション見出しの提供（AAA） | ページ内構造の強化 |

- 2.4.1 は「飛ばせるか」、1.3.1 は「構造が伝わるか」
- 構造マークアップは 2.4.1 の意図としても強く推奨される

> **参照**
> - [Understanding SC 2.4.1 — relation to 1.3.1](https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks.html#intent)

## テスト・確認方法

### 手動テスト

- キーボードのみでページ先頭から操作する
- 最初の Tab でスキップリンクが見えるか確認する
- スキップリンクを実行し、メイン側へフォーカスが移るか確認する
- スクリーンリーダーのランドマーク一覧・見出し一覧で領域ジャンプできるか確認する
- 複数ページで同じヘッダー／ナビが繰り返されるか確認する

```text
チェックリスト：
1. 複数ページで繰り返すナビ／ヘッダーなどがあるか
2. それをスキップする手段があるか（リンク・ランドマーク・見出し等）
3. スキップリンクはフォーカス時に見えるか
4. ジャンプ後のフォーカス位置は適切か
5. メインコンテンツへ少ない操作で到達できるか
```

### 開発者ツールでの確認

```bash
# スキップリンク・main・ランドマークまわりを確認
rg "skip|main-content|<main|<nav|role=\"main\"|role=\"navigation\"" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.php"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)
> - [a11y/skip-link.md — スキップリンクの実装と意義](../a11y/skip-link.md)

## まとめ

- 2.4.1 は、複数ページで繰り返されるコンテンツのブロックを**スキップする手段**を求める達成基準（レベル A）
- キーボード・スクリーンリーダー・画面拡大利用者の負担軽減が目的
- 代表的な手段はスキップリンク（G1 など）と、ランドマーク・見出しによる構造化（ARIA11 / H69 など）
- 実務ではスキップリンクとセマンティックなページ構造の併用が安全
- 1.3.1（構造）や 2.4.6 / 2.4.10（見出し）と併せて設計する
