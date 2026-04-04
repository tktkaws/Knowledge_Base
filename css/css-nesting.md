# ネスティング(CSS Nesting)の書き方

## CSS Nestingとは

- セレクタの内側に別のセレクタを**入れ子（ネスト）** で書けるCSSのネイティブ機能
- SassやLessなどのプリプロセッサで人気だったネスト記法が、標準CSSで使えるようになったもの
- 関連するスタイルをグループ化でき、**可読性の向上**と**セレクタの繰り返し削減**が主なメリット
- CSS Nesting Module Level 1として仕様策定

> 参照: [MDN - CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)

## 基本構文

```css
/* 従来の書き方：セレクタを繰り返す */
.card {
  padding: 1rem;
}
.card .title {
  font-size: 1.5rem;
}
.card .title span {
  color: #666;
}

/* CSS Nesting：入れ子で書く */
.card {
  padding: 1rem;

  .title {
    font-size: 1.5rem;

    span {
      color: #666;
    }
  }
}
```

- 親セレクタのブロック `{}` の中に子セレクタのルールを直接記述する
- ネストされたセレクタは暗黙的に**子孫結合子**（スペース）で結合される
- 上記の例では `.card .title` と `.card .title span` が自動的に生成される

> 参照: [MDN - CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)

## &（ネスティングセレクタ）の使い方

- `&` は**親セレクタそのもの**を表す特別なセレクタ
- 擬似クラスや擬似要素を付ける場合、結合セレクタを使う場合などに使用する

### 擬似クラス・擬似要素との組み合わせ

```css
.button {
  background-color: #0066cc;
  color: #fff;

  /* &なしでも動作するが、明示的に書くと意図が明確 */
  &:hover {
    background-color: #0052a3;
  }

  &:focus-visible {
    outline: 2px solid #0066cc;
    outline-offset: 2px;
  }

  &::after {
    content: "→";
    margin-left: 0.5em;
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}
```

- `:hover` や `::after` など、親セレクタに直接つなげたい場合に `&` を使う
- `&:hover` は `.button:hover` に展開される

### &を使ったセレクタの結合

```css
.card {
  padding: 1rem;

  /* .card.featured（同じ要素にクラスを追加） */
  &.featured {
    border: 2px solid gold;
  }

  /* .card#hero（IDとの結合） */
  &#hero {
    font-size: 2rem;
  }

  /* .card[data-size="large"]（属性セレクタとの結合） */
  &[data-size="large"] {
    padding: 2rem;
  }
}
```

- `&.featured` は `.card.featured`（スペースなしの複合セレクタ）になる
- `&` がないと `.card .featured`（子孫セレクタ）になるので意味が変わる

> 参照: [MDN - Nesting selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Nesting_selector)

## &を後ろに置く（親の文脈を切り替える）

```css
.card {
  padding: 1rem;

  /* .dark-theme .card */
  .dark-theme & {
    background-color: #1a1a1a;
    color: #e0e0e0;
  }

  /* article > .card */
  article > & {
    margin-bottom: 2rem;
  }
}
```

- `&` をセレクタの後ろに置くと「親の親」や「別の文脈での親」を表現できる
- `.dark-theme &` は `.dark-theme .card` に展開される
- テーマやコンテキスト違いのスタイルを、対象要素のブロック内にまとめて書ける

> 参照: [MDN - Nesting selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Nesting_selector)

## 結合子との組み合わせ

```css
.list {
  display: flex;
  flex-direction: column;

  /* .list > li（子結合子） */
  > li {
    padding: 0.5rem;
  }

  /* .list + .list（隣接兄弟結合子） */
  + .list {
    margin-top: 1rem;
  }

  /* .list ~ p（一般兄弟結合子） */
  ~ p {
    font-size: 0.875rem;
  }
}
```

- `>`、`+`、`~` の結合子はネスト内でそのまま使える
- `> li` は `.list > li` に展開される
- `&` を省略しても結合子から始まるセレクタは正しく解釈される

> 参照: [MDN - CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)

## メディアクエリや@ルールのネスト

```css
/* 従来の書き方：セレクタを繰り返す */
.grid {
  display: grid;
  grid-template-columns: 1fr;
}
@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* CSS Nesting：まとめて書ける */
.grid {
  display: grid;
  grid-template-columns: 1fr;

  @media (min-width: 768px) {
    grid-template-columns: repeat(2, 1fr);
  }

  @media (min-width: 1024px) {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

- `@media`、`@container`、`@supports`、`@layer` などの@ルールをセレクタ内にネストできる
- ネストされた@ルール内のスタイルは自動的に親セレクタに適用される
- **コンポーネント単位でレスポンシブスタイルをまとめられる**のが大きな利点

### @containerとの組み合わせ

```css
.card {
  padding: 1rem;

  @container (min-width: 400px) {
    display: grid;
    grid-template-columns: 200px 1fr;
  }
}
```

> 参照: [MDN - CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)

## 実践的な活用例

### コンポーネントスタイルをまとめる

```css
.nav {
  display: flex;
  gap: 1rem;
  background-color: #fff;

  .nav-link {
    text-decoration: none;
    color: #333;

    &:hover {
      color: #0066cc;
    }

    &.active {
      color: #0066cc;
      font-weight: bold;
      border-bottom: 2px solid currentColor;
    }
  }

  @media (max-width: 767px) {
    flex-direction: column;

    .nav-link {
      padding: 0.75rem 1rem;
    }
  }
}
```

- 1つのコンポーネントに関するすべてのスタイル（状態、レスポンシブ含む）を1ブロックにまとめられる

### リストコンポーネントの状態管理

```css
.todo-item {
  display: flex;
  align-items: center;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #e0e0e0;

  &:last-child {
    border-bottom: none;
  }

  &.completed {
    .todo-text {
      text-decoration: line-through;
      color: #999;
    }
  }

  &:hover {
    background-color: #f5f5f5;
  }

  .todo-text {
    flex: 1;
  }

  .delete-button {
    opacity: 0;
    transition: opacity 0.2s;
  }

  &:hover .delete-button {
    opacity: 1;
  }
}
```

> 参照: [web.dev - CSS nesting](https://developer.chrome.com/docs/css-ui/css-nesting)

## Sassのネストとの違い

### 文字列の連結はできない

```scss
/* Sass：&で文字列を連結できる */
.card {
  &__title { /* .card__title（BEM記法） */
    font-size: 1.5rem;
  }
  &--featured { /* .card--featured */
    border-color: gold;
  }
}
```

```css
/* CSS Nesting：&は文字列連結ではなくセレクタ全体を参照する */
/* 悪い例：意図通りにならない */
.card {
  &__title { /* ❌ .card__title にはならない */
    font-size: 1.5rem;
  }
}

/* 良い例：BEM記法にはCSS Nestingを使わない */
.card__title {
  font-size: 1.5rem;
}
.card--featured {
  border-color: gold;
}
```

- Sassの `&` は親セレクタの**テキスト**をそのまま連結するが、CSS Nestingの `&` は**セレクタオブジェクト**として参照する
- BEMのような命名規則で `&__element` や `&--modifier` のパターンはCSS Nestingでは使えない

### セレクタリストでの&の挙動

```css
.card, .panel {
  padding: 1rem;

  .title {
    font-size: 1.5rem;
    /* → .card .title, .panel .title に展開 */
  }

  &.featured {
    border-color: gold;
    /* → .card.featured, .panel.featured に展開 */
  }
}
```

- 親がセレクタリスト（カンマ区切り）の場合、`&` はリスト全体を参照する
- `:is()` を使って展開される（`.card, .panel` → `:is(.card, .panel)`）

> 参照: [MDN - Nesting selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Nesting_selector)

## 詳細度（Specificity）への影響

```css
/* このネストの結果は :is(.card, .panel) .title になる */
.card, .panel {
  .title {
    font-size: 1.5rem;
  }
}
```

- 親がセレクタリストの場合、内部的に `:is()` で展開される
- `:is()` の詳細度は引数の中で**最も詳細度が高いセレクタ**に合わせられる

```css
/* 注意が必要な例 */
#header, .card {
  .title {
    /* :is(#header, .card) .title に展開 */
    /* :is()の詳細度は #header に引っ張られて (1, 0, 0) */
    /* .card .title の意図でも (1, 0, 1) になる */
    color: red;
  }
}
```

- IDセレクタと混在したセレクタリストをネストの親にすると、詳細度が意図せず高くなる場合がある
- 可能であればセレクタリストの要素は同じ詳細度のものにそろえるとよい

> 参照: [MDN - CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)

## ネストの深さに注意する

```css
/* 悪い例：深すぎるネスト */
.page {
  .container {
    .section {
      .card {
        .card-body {
          .title {
            font-size: 1.5rem;
            /* → .page .container .section .card .card-body .title */
            /* 詳細度が高くなりすぎ、上書きが困難になる */
          }
        }
      }
    }
  }
}

/* 良い例：2〜3階層までに抑える */
.card {
  padding: 1rem;

  .card-body {
    .title {
      font-size: 1.5rem;
    }
  }
}
```

- ネストが深くなるほど**生成されるセレクタの詳細度が高く**なる
- **2〜3階層**までに抑えるのがベストプラクティス
- Sassでも同様だが、CSS Nestingでも深いネストは避けるべき

> 参照: [web.dev - CSS nesting](https://developer.chrome.com/docs/css-ui/css-nesting)

## ブラウザサポート

- Chrome 120+、Safari 17.2+、Firefox 117+ で全主要ブラウザがサポート
- 2024年以降、プロダクションで使えるレベルのサポート状況
- 初期仕様では要素型セレクタ（`div`, `p` など）のネストに `&` が必須だったが、**現在の仕様（relaxed parsing）ではすべてのセレクタで `&` なしのネストが可能**

### フォールバック

```css
/* CSS Nestingをサポートしないブラウザ向けのフォールバック */
.card {
  padding: 1rem;
}
.card .title {
  font-size: 1.5rem;
}

/* CSS Nestingをサポートするブラウザのみ */
@supports (selector(&)) {
  .card {
    padding: 1rem;

    .title {
      font-size: 1.5rem;
    }
  }
}
```

- `@supports (selector(&))` でCSS Nestingの機能検出が可能
- 現在は主要ブラウザすべてがサポート済みのため、新規プロジェクトではフォールバック不要なケースが多い

> 参照: [Can I Use - CSS Nesting](https://caniuse.com/css-nesting)

## まとめ

- CSS Nestingはセレクタを入れ子で記述できるネイティブCSS機能
- `&` は親セレクタを参照する特別なセレクタで、擬似クラスや複合セレクタの結合に使う
- `@media` や `@container` もネスト可能で、コンポーネント単位でレスポンシブスタイルをまとめられる
- Sassの `&` とは異なり、文字列連結（BEMの `&__element`）はできない
- セレクタリストの親は `:is()` で展開されるため、詳細度の意図しない上昇に注意
- ネストの深さは**2〜3階層まで**に抑えるのがベストプラクティス
- 全主要ブラウザでサポート済み、プロダクションで利用可能

## 参照元

- [MDN - CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)
- [MDN - Nesting selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Nesting_selector)
- [web.dev - CSS nesting](https://developer.chrome.com/docs/css-ui/css-nesting)
- [Can I Use - CSS Nesting](https://caniuse.com/css-nesting)
