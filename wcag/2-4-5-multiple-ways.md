# 2.4.5 複数の手段（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.5「複数の手段」（Multiple Ways）
- レベル **AA**
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- **Webページの集合内**で、あるページを見つける手段が**2つ以上**あること
- 例外：そのページがプロセスの結果、またはプロセスの一部である場合

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.5 Multiple Ways](https://www.w3.org/TR/WCAG22/#multiple-ways)
> - [Understanding SC 2.4.5 Multiple Ways](https://www.w3.org/WAI/WCAG22/Understanding/multiple-ways.html)

## 達成基準の原文（要約）

- Webページの集合内で、ある Webページを見つける手段が複数あること
- ただし、その Webページがプロセスの結果、またはプロセスの一部である場合は除く

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.5](https://www.w3.org/TR/WCAG22/#multiple-ways)

## Webページの集合とは

- 共通の目的を持ち、同じ作者・グループ・組織が作成した Webページの集まり
- 例：同一サイト内の会社案内・製品一覧・お問い合わせ
- 言語が異なるバージョンは、別の集合として扱う
- 単一ページだけで完結するサイトには、この基準の適用場面が限られる

```text
集合の例：
  コーポレートサイト全体
  ドキュメントサイトの各ページ
  レシピサイトの各レシピページ

別集合になりやすい例：
  日本語版サイトと英語版サイト
  まったく別ドメインの関連プロモーションサイト
```

> **参照**
> - [Understanding SC 2.4.5 — Key Terms: set of web pages](https://www.w3.org/WAI/WCAG22/Understanding/multiple-ways.html#dfn-set-of-web-pages)

## なぜ必要か

- ユーザーによって、探しやすいナビ手段が異なる
- 視覚障害のあるユーザーは、大きなナビをスクロールするより検索の方が早いことがある
- 認知障害のあるユーザーは、階層ナビよりサイトマップや目次の方が把握しやすいことがある
- 順番に読んで理解したいユーザーもいる
- 1つの手段だけでは、一部のユーザーが目的のページにたどり着けない

> **参照**
> - [Understanding SC 2.4.5 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/multiple-ways.html#benefits)

## 適合の考え方：2つ以上の手段を組み合わせる

- 十分な手法は「次のうち**2つ以上**を使う」こと
- グローバルナビだけ、検索だけ、では不足しやすい
- 小規模サイトでも、何らかのオリエンテーション手段は必要

| 手法 | 内容 |
|---|---|
| G125 | 関連ページへのリンクを提供する |
| G64 | 目次を提供する |
| G63 | サイトマップを提供する |
| G161 | 検索機能を提供する |
| G126 | 他の全ページへのリンク一覧を提供する |
| G185 | ホームからサイト内の全ページへリンクする |

```text
よくある組み合わせ：
  グローバルナビ + 検索
  ホームのページ一覧 + 各ページ間リンク
  サイトマップ + 検索
  目次 + 前後ページリンク
```

> **参照**
> - [Understanding SC 2.4.5 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/multiple-ways.html#techniques)

## 手法1：検索機能（G161）

- サイト内検索で、目的のページやコンテンツを直接探せるようにする
- 大きなサイトでは特に有効
- カテゴリ選択など、別の探し方と併用するとよい

```html
<!-- 良い例：検索 + カテゴリの2手段 -->
<form role="search" action="/search" method="get">
  <label for="q">レシピを検索</label>
  <input id="q" name="q" type="search">
  <button type="submit">検索</button>
</form>

<label for="category">カテゴリから探す</label>
<select id="category" name="category">
  <option value="soup">スープ</option>
  <option value="salad">サラダ</option>
  <option value="dessert">デザート</option>
</select>
```

```text
良い例：
  「スープ」と検索する
  カテゴリ「スープ」を選ぶ
  → どちらでも同じレシピ一覧へ到達できる
```

> **参照**
> - [Technique G161: Providing a search function to help users find content](https://www.w3.org/WAI/WCAG22/Techniques/general/G161)
> - [Understanding SC 2.4.5 — search mechanism example](https://www.w3.org/WAI/WCAG22/Understanding/multiple-ways.html#examples)

## 手法2：サイトマップ（G63）

- サイト内の主要ページを一覧できる専用ページを用意する
- 階層構造が一目で分かると、認知負荷を下げやすい
- フッターやヘルプから到達できるようにする

```html
<!-- 良い例：サイトマップ -->
<main>
  <h1>サイトマップ</h1>
  <ul>
    <li><a href="/">ホーム</a></li>
    <li>
      製品
      <ul>
        <li><a href="/products/a">製品A</a></li>
        <li><a href="/products/b">製品B</a></li>
      </ul>
    </li>
    <li><a href="/about">会社概要</a></li>
    <li><a href="/contact">お問い合わせ</a></li>
  </ul>
</main>
```

> **参照**
> - [Technique G63: Providing a site map](https://www.w3.org/WAI/WCAG22/Techniques/general/G63)

## 手法3：目次（G64）

- 長い文書やドキュメントサイトで、各セクションへの目次を提供する
- EPUB などのデジタル出版物でも有効
- 読者が位置を把握しやすくなる

```html
<!-- 良い例：ドキュメントの目次 -->
<nav aria-labelledby="toc-heading">
  <h2 id="toc-heading">目次</h2>
  <ol>
    <li><a href="#intro">はじめに</a></li>
    <li><a href="#install">インストール</a></li>
    <li><a href="#usage">使い方</a></li>
    <li><a href="#api">API</a></li>
  </ol>
</nav>
```

> **参照**
> - [Technique G64: Providing a Table of Contents](https://www.w3.org/WAI/WCAG22/Techniques/general/G64)

## 手法4：関連ページへのリンク（G125）

- 前後ページ、関連カテゴリ、パンくずなど、関連ページへ移動できるリンクを置く
- 単独では不十分なことが多く、別手段との併用が前提

```html
<!-- 良い例：前後リンク + パンくず -->
<nav aria-label="パンくず">
  <ol>
    <li><a href="/">ホーム</a></li>
    <li><a href="/docs">ドキュメント</a></li>
    <li aria-current="page">インストール</li>
  </ol>
</nav>

<nav aria-label="前後のページ">
  <a href="/docs/intro">前へ：はじめに</a>
  <a href="/docs/usage">次へ：使い方</a>
</nav>
```

> **参照**
> - [Technique G125: Providing links to navigate to related web pages](https://www.w3.org/WAI/WCAG22/Techniques/general/G125)

## 手法5：小規模サイトでの全ページリンク（G126 / G185）

- ページ数が少ないサイトでは、ホームから全ページへリンクするだけで足りる場合がある
- 各ページに他ページへのリンク一覧を置く方法もある
- Understanding では、3〜4ページ程度の小規模サイトの例が示されている

```html
<!-- 良い例：各ページに他ページへのリンク一覧がある -->
<nav aria-label="サイト内ページ">
  <ul>
    <li><a href="/">ホーム</a></li>
    <li><a href="/services" aria-current="page">サービス</a></li>
    <li><a href="/price">料金</a></li>
    <li><a href="/access">アクセス</a></li>
    <li><a href="/contact">お問い合わせ</a></li>
  </ul>
</nav>

<!-- 加えて前後リンクもあると、手段が2つになる -->
<nav aria-label="前後のページ">
  <a href="/">前へ：ホーム</a>
  <a href="/price">次へ：料金</a>
</nav>
```

> **参照**
> - [Technique G126: Providing a list of links to all other web pages](https://www.w3.org/WAI/WCAG22/Techniques/general/G126)
> - [Technique G185: Linking to all of the pages on the site from the home page](https://www.w3.org/WAI/WCAG22/Techniques/general/G185)
> - [Understanding SC 2.4.5 — hair salon example](https://www.w3.org/WAI/WCAG22/Understanding/multiple-ways.html#examples)

## 例外：プロセスの結果・途中ページ

- プロセスの結果としてしか存在しないページは、複数手段で見つける必要がない
- プロセスの途中ステップも例外になりうる
- ユーザーがそのプロセスを完了しないと到達できないページが対象

```text
例外の例：
  振込完了後にだけ表示される確認ページ
  検索を実行した結果ページ
  カート確認 → 配送先 → 支払い、の途中ステップ

例外にならない例：
  会社概要ページ
  製品詳細ページ
  ヘルプ記事
  静的な料金ページ
```

```html
<!-- 例外：振込確認は振込完了後にしか到達できない -->
<main>
  <h1>振込が完了しました</h1>
  <p>受付番号：A-12345</p>
  <a href="/accounts">口座一覧へ戻る</a>
</main>
```

> **参照**
> - [Understanding SC 2.4.5 — process examples](https://www.w3.org/WAI/WCAG22/Understanding/multiple-ways.html#examples)
> - [Understanding SC 2.4.5 — Key Terms: process](https://www.w3.org/WAI/WCAG22/Understanding/multiple-ways.html#dfn-process)

## 2.4.1・2.4.8 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.1 ブロックスキップ | 繰り返すブロックを飛ばす | ページ内の移動効率 |
| 2.4.5 複数の手段 | ページ集合内でページを見つける手段が複数 | ページ間の到達手段 |
| 2.4.8 現在位置 | 集合内での現在位置が分かる（AAA） | どこにいるかの把握 |

- 2.4.5 は「目的のページへどう行くか」
- 2.4.8 は「今どこにいるか」
- パンくずは、2.4.5 の関連リンクとしても、2.4.8 の現在位置把握にも役立つ

> **参照**
> - [wcag/2-4-1-bypass-blocks.md — 2.4.1 ブロックスキップ](./2-4-1-bypass-blocks.md)
> - [WCAG 2.2 — Success Criterion 2.4.8 Location](https://www.w3.org/TR/WCAG22/#location)

## 実装時の注意点

- 大規模サイトは「ナビ + 検索」または「サイトマップ + 検索」が実務的
- 小規模サイトは「ホームからの全リンク + 各ページ間リンク」で足りることが多い
- 検索結果ページ自体は例外だが、検索機能は他ページを見つける手段になる
- SPA でも、ルート一覧・検索・目次など、同等の複数手段を提供する
- サイトマップや検索が壊れている・空だと、手段として数えにくい

```text
実務チェック：
  任意の下層ページへ、異なる2つの経路で到達できるか
  プロセス結果ページを「例外」と誤って拡大解釈していないか
  検索・サイトマップ・目次が実際に使えるか
```

## テスト・確認方法

### 手動テスト

- サイト内の代表的な下層ページを選ぶ
- そのページへ到達する手段を2つ以上列挙できるか確認する
- プロセス結果ページ（確認画面・検索結果）は例外として切り分ける
- 小規模サイトなら、ホームと各ページの相互リンクで足りるか確認する

```text
チェックリスト：
1. 対象は Webページの集合内のページか
2. プロセスの結果／途中ページではないか
3. 到達手段が2つ以上あるか
4. それらの手段は実際に動作するか
5. ナビだけ／検索だけ、になっていないか
```

### 開発者ツールでの確認

```bash
# 検索・サイトマップ・目次まわりを探す
rg "role=\"search\"|sitemap|目次|site map|table of contents" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.php" -i
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.5 は、ページ集合内で目的のページを見つける手段を**2つ以上**求める達成基準（レベル AA）
- 検索・サイトマップ・目次・関連リンク・全ページリンクなどを組み合わせる
- プロセスの結果や途中ステップのページは例外
- 小規模サイトでも、ホームと各ページを相互に結ぶ程度の複数手段は必要
- 2.4.1（ページ内スキップ）や 2.4.8（現在位置）と併せて、サイト全体の迷いやすさを減らす
