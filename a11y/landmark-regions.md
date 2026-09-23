# ランドマークロールとページ構造の設計

## ランドマークとは

- ページの主要な領域をプログラム的に識別する仕組み
- スクリーンリーダー利用者は、ランドマーク一覧からヘッダー・ナビ・メインなどへ直接移動できる
- 視覚的な配置・色・余白で伝わる「塊」を、支援技術にも同じ粒度で伝える
- HTMLのセクショニング要素が、多くのランドマークを暗黙的に作る

> 参照: [WAI-ARIA Authoring Practices — Landmark Regions](https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/)

## HTML要素と暗黙のランドマーク

| HTML要素 | 暗黙のロール | 備考 |
|---|---|---|
| `<header>`（`body` 直下） | `banner` | サイト全体のヘッダー |
| `<nav>` | `navigation` | ナビゲーション |
| `<main>` | `main` | そのページの主コンテンツ |
| `<aside>` | `complementary` | 補足・関連情報 |
| `<footer>`（`body` 直下） | `contentinfo` | サイト全体のフッタ |
| `<section>`（名前あり） | `region` | 名前がないとランドマークにならない |
| `<form>`（名前あり） | `form` | 名前がないとランドマークにならない |
| `<search>` | `search` | 検索用の領域 |

- `header` / `footer` が `article` / `section` / `main` / `nav` / `aside` の中にある場合、`banner` / `contentinfo` にはならない
- 同等のHTMLがあるなら `role="banner"` などよりネイティブ要素を使う

> 参照: [WCAG Technique H101](https://www.w3.org/WAI/WCAG22/Techniques/html/H101)

## 設計の基本方針

1. 知覚できるコンテンツは、適切なランドマークの中に入れる
2. ページのランドマークはおおよそ7つ以下を目安にする
3. `banner` / `main` / `complementary` / `contentinfo` はトップレベルにする
4. 同じ種類が複数あるときは、それぞれ一意の名前を付ける
5. ラベルに「ナビゲーション」などロール名を重複して入れない

> 参照: [APG — Landmarks Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/landmarks/)

## 各ランドマークの使い方

### banner（サイトヘッダー）

- ロゴ、サイト名、全体検索など、サイト共通の導入部
- ページに原則1つ。`body` 直下の `<header>` で作る

```html
<body>
  <header>
    <a href="/">サイト名</a>
    <!-- 全体検索やユーティリティリンクなど -->
  </header>
  ...
</body>
```

### navigation（ナビ）

- 主要な移動用リンクのまとまり
- メインナビ・パンくず・フッターリンクなど、目的ごとに分けてよい
- 複数ある場合は必ず名前を付ける

```html
<nav aria-label="メイン">
  <ul>
    <li><a href="/">ホーム</a></li>
    <li><a href="/products">製品</a></li>
  </ul>
</nav>

<nav aria-label="パンくず">
  <ol>...</ol>
</nav>
```

```html
<!-- 悪い例：ラベルにロール名を含める -->
<nav aria-label="サイトナビゲーション">
  <!-- 「サイトナビゲーション、ナビゲーション」のように重複して読まれやすい -->
</nav>

<!-- 良い例 -->
<nav aria-label="サイト">
```

### main（メイン）

- そのページ固有の主コンテンツ
- 原則1つ。スキップリンクの飛び先にもなる
- SPAでも、主内容のコンテナとして維持する

```html
<main id="main">
  <h1>製品一覧</h1>
  ...
</main>
```

### complementary（補足）

- メインと関連しつつ、切り離しても意味が通る補足情報（関連記事、目次、広告枠など）
- トップレベルに置くのが推奨
- 複数ある場合は名前を付ける。1つでも名前があると目的が伝わりやすい

```html
<aside aria-labelledby="related-heading">
  <h2 id="related-heading">関連記事</h2>
  ...
</aside>
```

### contentinfo（サイトフッタ）

- 著作権、プライバシーポリシー、問い合わせなどサイト共通のフッタ情報
- 原則1つ。`body` 直下の `<footer>` で作る

```html
<footer>
  <p><small>&copy; 2026 Example Inc.</small></p>
  <nav aria-label="フッター">
    <ul>
      <li><a href="/privacy">プライバシー</a></li>
    </ul>
  </nav>
</footer>
```

### region（名前付きセクション）

- 上記に当てはまらないが、重要なまとまりとしてジャンプさせたい領域
- **アクセシブルネームが必須**（`aria-labelledby` / `aria-label`）
- `<section>` に見出しを付け、`aria-labelledby` で紐付けるのが分かりやすい

```html
<section aria-labelledby="pricing-heading">
  <h2 id="pricing-heading">料金プラン</h2>
  ...
</section>
```

```html
<!-- 悪い例：名前がなく、ただのグループ扱い -->
<section>
  <div>料金プラン</div>
</section>
```

### search / form

- 検索UIは `<search>`、または名前付きの `<form>`
- 一般的な入力フォームをランドマークにするなら、見える見出しと名前を付ける

```html
<search>
  <form action="/search" method="get">
    <label for="q">サイト内検索</label>
    <input id="q" type="search" name="q">
    <button type="submit">検索</button>
  </form>
</search>
```

```html
<form aria-labelledby="contact-heading" action="/contact" method="post">
  <h2 id="contact-heading">お問い合わせ</h2>
  ...
</form>
```

## ページ全体の構成例

```html
<body>
  <a href="#main" class="skip-link">メインコンテンツへスキップ</a>

  <header>
    <a href="/">Example</a>
    <search>...</search>
  </header>

  <nav aria-label="メイン">...</nav>

  <main id="main">
    <h1>ドキュメント</h1>

    <nav aria-label="パンくず">...</nav>

    <article>
      <h2>はじめに</h2>
      <p>...</p>
    </article>

    <section aria-labelledby="faq-heading">
      <h2 id="faq-heading">FAQ</h2>
      ...
    </section>
  </main>

  <aside aria-labelledby="toc-heading">
    <h2 id="toc-heading">目次</h2>
    ...
  </aside>

  <footer>
    <p><small>&copy; Example Inc.</small></p>
  </footer>
</body>
```

### 構成のポイント

- スキップリンクで `main` へ飛べるようにする
- メインナビとパンくずは、どちらも `nav` だがラベルで区別する
- `article` 自体はランドマークではないが、文書構造としては有用
- すべてを `region` にしない — 本当にジャンプさせたい塊だけにする

## 名前の付け方

| 優先 | 方法 | 向く場合 |
|---|---|---|
| 1 | `aria-labelledby` | 見える見出しがある |
| 2 | `aria-label` | 見える見出しがないが区別が必要 |

```html
<!-- 推奨：見える見出しを名前にする -->
<aside aria-labelledby="news-heading">
  <h2 id="news-heading">お知らせ</h2>
</aside>

<!-- 見出しがない場合 -->
<nav aria-label="パンくず">...</nav>
```

- 同じ種類のランドマークが1つだけなら、`main` / `banner` / `contentinfo` の名前は任意
- `navigation` / `complementary` は名前があると分かりやすい
- `region` / 名前で初めてランドマークになる `section`・`form` は名前が必須

## 見出し構造との関係

- ランドマークは「大きな領域」、見出しは「領域内の目次」
- どちらも必要。ランドマークだけでは細部の構造は伝わらない
- 領域の先頭に適切な見出しを置くと、視覚・非視覚の両方で分かりやすい

```html
<main>
  <h1>設定</h1>
  <section aria-labelledby="profile-heading">
    <h2 id="profile-heading">プロフィール</h2>
    ...
  </section>
  <section aria-labelledby="security-heading">
    <h2 id="security-heading">セキュリティ</h2>
    ...
  </section>
</main>
```

## よくある間違い

### 1. main が複数ある / ない

```html
<!-- 間違い -->
<div class="content">...</div>
<div class="content">...</div>

<!-- 正解：ページ固有の主内容は main にまとめる -->
<main>...</main>
```

### 2. header / footer の入れ子で banner / contentinfo を期待する

```html
<!-- article 内の header は banner にならない -->
<article>
  <header>
    <h2>記事タイトル</h2>
  </header>
</article>
```

- サイト共通ヘッダーだけを `body` 直下の `<header>` にする

### 3. ランドマークの付けすぎ

```html
<!-- 悪い例：小さな塊まで全部 region -->
<section aria-label="ボタン領域">...</section>
<section aria-label="余白">...</section>
```

- 多すぎると一覧の価値が下がる。目安は7つ以下

### 4. コンテンツがランドマークの外に置かれている

- ランドマーク間だけ移動する利用者は、外の情報を見落とすことがある
- 知覚できる内容はいずれかのランドマークに含める

### 5. div + role でネイティブ要素を再現する

```html
<!-- 避ける -->
<div role="navigation" aria-label="メイン">...</div>
<div role="main">...</div>

<!-- 使う -->
<nav aria-label="メイン">...</nav>
<main>...</main>
```

### 6. トップレベルであるべきランドマークを入れ子にする

```html
<!-- 避ける：main の中に banner 相当を置く -->
<main>
  <header>サイトロゴ</header>
</main>
```

- サイトヘッダー・メイン・フッタは兄弟関係で並べる

## 実装チェックリスト

- [ ] `main` が1つある
- [ ] サイトヘッダーが `body` 直下の `header`（または同等）になっている
- [ ] サイトフッタが `body` 直下の `footer`（または同等）になっている
- [ ] ナビが `nav` で、複数時は一意の名前がある
- [ ] 重要な補足は `aside`（必要なら名前付き）
- [ ] `section` / `form` をランドマークにするなら名前がある
- [ ] ランドマークが多すぎない（目安7つ以下）
- [ ] 知覚できるコンテンツがランドマーク外に残っていない
- [ ] スキップリンクが `main` など主内容へ到達できる
- [ ] ラベルにロール名を重複させていない

## まとめ

- ランドマークは、ページの大きな構造を支援技術に伝えるための領域識別
- まずは `<header>` / `<nav>` / `<main>` / `<aside>` / `<footer>` などのHTMLで暗黙ロールを使う
- `main` は1つ、同種が複数なら名前で区別、`region` には必ず名前を付ける
- 数を絞り、内容をランドマーク内に収め、見出し構造と併用する
- `div role="..."` よりセマンティックHTMLを優先する

> 参照: [セマンティックHTMLの原則](./semantic-html.md)、[スキップリンクの実装と意義](./skip-link.md)
