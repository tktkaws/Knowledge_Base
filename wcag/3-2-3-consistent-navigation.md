# 3.2.3 一貫したナビゲーション（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.2.3「一貫したナビゲーション」（Consistent Navigation）
- レベル **AA**
- 原則3「理解可能（Understandable）」> ガイドライン 3.2「予測可能」に属する
- **Webページの集合内**で繰り返されるナビゲーション機構は、毎回**同じ相対順序**であること
- 例外：ユーザーが順序の変更を開始した場合
- サブナビの追加や一部リンクの増減は、相対順序が保たれれば許される

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.3 Consistent Navigation](https://www.w3.org/TR/WCAG22/#consistent-navigation)
> - [Understanding SC 3.2.3 Consistent Navigation](https://www.w3.org/WAI/WCAG22/Understanding/consistent-navigation.html)

## 達成基準の原文（要約）

- Webページの集合内で複数の Web ページに繰り返されるナビゲーション機構は、ユーザーが変更を開始した場合を除き、繰り返されるたびに同じ相対順序で現れること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.3](https://www.w3.org/TR/WCAG22/#consistent-navigation)

## なぜ必要か

- 画面拡大利用者は、ページ境界や視覚的な位置で繰り返しコンテンツを探す
- 空間記憶やデザインの手がかりでナビを見つける人も、順序が変わると迷う
- スクリーンリーダー利用者は、ソース順で順番に読む。順序が毎回違うと予測できない
- 認知・学習・知的障害のある人は、同じ場所にあると覚えた操作が使えなくなる
- 2回目以降の訪問で、目的のリンクを素早く見つけやすくなる

> **参照**
> - [Understanding SC 3.2.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/consistent-navigation.html#benefits)

## ナビゲーション機構とは

- ユーザーが別の Web ページへ移動するための仕組み
- 例：グローバルナビ、フッターリンク、パンくず、サイドバーの主要リンク
- 繰り返される部品全体（ロゴ、検索、ナビ）の**相対順序**も対象
- 1ページだけのサイトでは、適用場面が限られる

```text
対象になりやすい例：
  全ページ共通のヘッダーナビ
  フッターの会社情報リンク群
  各ページ先頭の「メインコンテンツへ」リンク
  サイト内検索ボックス（毎回同じ位置）

対象外になりやすい例：
  そのページ固有の本文内リンク
  1回だけ現れるプロセス内のステップ表示
  別集合（別言語版サイト）のナビ
```

> **参照**
> - [Understanding SC 3.2.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/consistent-navigation.html#intent)

## 同じ相対順序とは

- 他の繰り返し部品に対する**相対的な位置**が同じこと
- 間に別の項目が**挿入・削除**されても、残った項目同士の順序が変わらなければよい
- サブナビが展開されて項目が増えても、主要リンクの順序は保たれる
- 絶対位置（画面上の座標）ではなく、DOM 上の相対順序が重要

```text
許容される例：
  ページA：ホーム → 製品 → 会社概要 → お問い合わせ
  ページB：ホーム → 製品 → [製品A] [製品B] → 会社概要 → お問い合わせ
  → 主要4項目の相対順序は同じ

  ページA：ロゴ → 検索 → ナビ
  ページB：ロゴ → ナビ
  → 検索がなくても、残った部品の順序は同じ

失敗例：
  ページA：ホーム → 製品 → 会社概要
  ページB：製品 → ホーム → 会社概要
  → ホームと製品の相対順序が逆
```

> **参照**
> - [Understanding SC 3.2.3 — Key Terms: same relative order](https://www.w3.org/WAI/WCAG22/Understanding/consistent-navigation.html#dfn-same-relative-order)
> - [Technique G61: Presenting repeated components in the same relative order each time they appear](https://www.w3.org/WAI/WCAG22/Techniques/general/G61)

## Webページの集合

- 共通の目的を持ち、同じ作者・グループ・組織が作成した Web ページの集まり
- 例：同一コーポレートサイト、ドキュメントサイト全体
- 言語が異なる版は、別の集合として扱う
- 2.4.5（複数の手段）と同じ「集合」の考え方

> **参照**
> - [wcag/2-4-5-multiple-ways.md — 2.4.5 複数の手段](./2-4-5-multiple-ways.md)

## 手法：繰り返し部品を同じ相対順序で出す（G61）

- **G61**：繰り返されるコンポーネントを、毎回同じ相対順序で提示する
- 共通レイアウト（ヘッダー、フッター、サイドバー）をテンプレート化する
- フレームワークでは、ルートレイアウトや共通コンポーネントで順序を固定する

```html
<!-- 良い例：全ページで同じ順序のヘッダー -->
<header>
  <a href="/"><img src="/logo.svg" alt="株式会社サンプル"></a>
  <form role="search" action="/search">
    <label for="q">検索</label>
    <input id="q" name="q" type="search">
    <button type="submit">検索</button>
  </form>
  <nav aria-label="メインナビゲーション">
    <ul>
      <li><a href="/">ホーム</a></li>
      <li><a href="/products">製品</a></li>
      <li><a href="/about">会社概要</a></li>
      <li><a href="/contact">お問い合わせ</a></li>
    </ul>
  </nav>
</header>
```

```html
<!-- 良い例：製品セクションではサブリンクが増えるが、主要順序は同じ -->
<nav aria-label="メインナビゲーション">
  <ul>
    <li><a href="/">ホーム</a></li>
    <li>
      <a href="/products" aria-current="page">製品</a>
      <ul>
        <li><a href="/products/a">製品A</a></li>
        <li><a href="/products/b">製品B</a></li>
      </ul>
    </li>
    <li><a href="/about">会社概要</a></li>
    <li><a href="/contact">お問い合わせ</a></li>
  </ul>
</nav>
```

```html
<!-- 悪い例：ページごとに主要リンクの順序が違う -->
<!-- トップページ -->
<nav>
  <a href="/">ホーム</a>
  <a href="/products">製品</a>
  <a href="/about">会社概要</a>
</nav>

<!-- 製品ページ -->
<nav>
  <a href="/products">製品</a>
  <a href="/">ホーム</a>
  <a href="/about">会社概要</a>
</nav>
```

```jsx
// 良い例：Next.js で共通 Header を1か所に
const navItems = [
  { href: "/", label: "ホーム" },
  { href: "/products", label: "製品" },
  { href: "/about", label: "会社概要" },
  { href: "/contact", label: "お問い合わせ" },
];

export function SiteHeader() {
  return (
    <header>
      <Logo />
      <SearchForm />
      <nav aria-label="メインナビゲーション">
        <ul>
          {navItems.map((item) => (
            <li key={item.href}>
              <a href={item.href}>{item.label}</a>
            </li>
          ))}
        </ul>
      </nav>
    </header>
  );
}
```

> **参照**
> - [Technique G61: Presenting repeated components in the same relative order each time they appear](https://www.w3.org/WAI/WCAG22/Techniques/general/G61)

## スキップリンクとの組み合わせ

- 各ページ先頭の「メインコンテンツへ」リンクも、繰り返されるナビ機構
- 毎回同じ位置（先頭）に置くと、キーボード利用者が予測しやすい
- 2.4.1（ブロックスキップ）と合わせて実装する

```html
<!-- 良い例：全ページで先頭に同じスキップリンク -->
<body>
  <a href="#main" class="skip-link">メインコンテンツへ</a>
  <header><!-- 共通ヘッダー --></header>
  <main id="main"><!-- ページ固有コンテンツ --></main>
</body>
```

> **参照**
> - [wcag/2-4-1-bypass-blocks.md — 2.4.1 ブロックスキップ](./2-4-1-bypass-blocks.md)

## ユーザーによる順序変更

- ユーザーが適応型ユーザーエージェントや設定で順序を変えるのは、作者の失敗ではない
- 例：ユーザー設定でナビを非表示にする、読み上げ順を変える
- 作者がページごとに勝手に順序を入れ替えるのは対象

## 3.2.4 との関係

| 達成基準 | レベル | 対象 |
|---|---|---|
| 3.2.3 一貫したナビゲーション | AA | 繰り返されるナビの**相対順序** |
| 3.2.4 一貫した識別性 | AA | 同じ機能の部品の**見た目・ラベル・名前** |

- 3.2.3 は「どこにあるか（順序）」
- 3.2.4 は「同じ機能が同じ見た目・名前か」
- 両方そろうと、サイト全体が予測しやすくなる

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.4 Consistent Identification](https://www.w3.org/TR/WCAG22/#consistent-identification)

## よくある失敗例（F66）

- ページごとにグローバルナビのリンク順を変える
- A/B テストで、ナビの並びだけをページごとに変える
- セクションごとに、主要リンクの優先順位を入れ替える
- レスポンシブで、モバイルとデスクトップで**主要リンクの相対順序**が変わる

```html
<!-- 失敗例：F66 — ページ2で Brazil と Canada の順序が逆 -->
<!-- Page 1 -->
<nav>
  <a href="/brazil">Brazil</a>
  <a href="/canada">Canada</a>
  <a href="/germany">Germany</a>
</nav>

<!-- Page 2 -->
<nav>
  <a href="/canada">Canada</a>
  <a href="/brazil">Brazil</a>
  <a href="/germany">Germany</a>
</nav>
```

```css
/* 失敗例：flex の order で、ページごとに視覚順だけ変える */
/* DOM 順は同じでも、支援技術の読み順と画面の順がズレる */
.nav--home .item-products { order: 1; }
.nav--products .item-products { order: -1; }
```

> **参照**
> - [Failure F66: presenting navigation links in a different relative order on different pages](https://www.w3.org/WAI/WCAG22/Techniques/failures/F66)

## テスト・確認方法

### 手動テスト

- サイト内の複数ページで、繰り返されるナビを洗い出す
- 主要リンクの相対順序が同じか、ページ間で比較する
- ヘッダー内のロゴ・検索・ナビの順序も確認する
- サブナビが増えても、主要項目の順序が保たれているか確認する
- モバイル／デスクトップで順序が変わっていないか確認する

```text
チェックリスト：
1. 集合内の複数ページで、同じナビが繰り返されているか
2. 主要リンクの相対順序はページ間で同じか
3. ロゴ・検索・ナビなど、繰り返し部品全体の順序も同じか
4. サブリンクの追加・削除だけなら、主要順序は保たれているか
5. CSS の order 等で、読み順と見た目の順がページごとに変わっていないか
6. スキップリンクは毎回同じ位置か
```

```javascript
// Console：各ページの nav 内リンク順を比較する手がかり
[...document.querySelectorAll("nav a")].map((a) => a.textContent.trim());
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.2.3 は、Webページの集合内で繰り返されるナビを**同じ相対順序**で出す達成基準（レベル AA）
- 主要リンクの順序をページごとに入れ替えない
- サブナビの追加や一部リンクの増減は、相対順序が保たれれば許される
- 共通レイアウト・共通コンポーネントで順序を固定するのが実務的
- 順序（3.2.3）と識別性（3.2.4）を合わせると、サイト全体が予測しやすくなる
