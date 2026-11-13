# カード全体をクリック可能にする — 入れ子になったリンクを避ける

## クリック可能なカードとは

カードは、画像、見出し、概要、メタ情報、操作などを一つのまとまりとして表示するUI。記事一覧、商品一覧、検索結果、ダッシュボードなどに使われる。

カード全体をクリック可能にするとポインターの操作範囲は広がるが、HTML構造を誤ると次の問題が起きる。

- リンクの中へ別のリンクやボタンを入れてしまう
- `div`のクリックイベントだけで遷移し、キーボードから使えない
- 同じ行き先のリンクがカード内に何度も現れる
- リンク名がカード内の全テキストになり、長く分かりにくい
- 副操作を押したつもりで詳細ページへ移動する
- テキストを選択しようとすると遷移する
- フォーカス位置がカードの見た目から分からない

最初に、カードの目的と操作数を整理する。

| カードの内容 | 基本方針 |
|---|---|
| 行き先が一つで、副操作がない | カードの内容を一つの`a`で包める |
| 主な行き先と副操作がある | 主リンクと副操作を兄弟要素にする |
| ページ遷移ではなく選択・実行する | `button`、チェックボックスなど目的に合う要素を使う |
| 情報を表示するだけ | カード自体を操作可能にしない |

> 参照: [WHATWG HTML Standard — The `a` element](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-a-element)

## 最初にリンクかボタンかを決める

```html
<!-- ページへ移動する -->
<a href="/articles/accessible-forms">記事を読む</a>

<!-- 保存状態を切り替える -->
<button type="button">お気に入りに追加</button>
```

- URLへ移動する、ダウンロードする場合は`a[href]`
- お気に入り、削除、開閉、選択など現在のページで動作する場合は`button`
- フォームの選択肢ならチェックボックスやラジオボタン

「カード風の見た目」だけで要素を決めない。

## 避ける構造

### リンクの中にリンクとボタンを入れる

```html
<!-- 不正な構造 -->
<a href="/articles/card-design" class="card">
  <h2>カードUIの設計</h2>
  <p>カード全体をクリック可能にする方法を解説します。</p>
  <a href="/categories/css">CSS</a>
  <button type="button">あとで読む</button>
</a>
```

`a`の子孫には、別の`a`、ボタンなどのインタラクティブコンテンツ、`tabindex`を指定した要素を置けない。ブラウザがDOMを意図しない形へ修正したり、クリックとフォーカスの対象が不明確になったりする。

### クリック可能な`div`だけを使う

```html
<!-- 避ける -->
<div class="card" onclick="location.href='/articles/card-design'">
  <h2>カードUIの設計</h2>
</div>
```

この`div`にはリンクの意味も標準のキーボード操作もない。次のリンク機能も失われる。

- Tabキーで移動する
- Enterキーで開く
- 新しいタブやウィンドウで開く
- リンク先URLをコピーする
- コンテキストメニューを使う
- ステータスバーでURLを確認する
- 訪問済みリンクとして区別する

`role="link"`と`tabindex="0"`を追加して自作するより、`a href`を使う。

## パターン1: 行き先が一つのカード

カード内に他のリンク、ボタン、入力欄がない場合は、内容全体を一つのリンクで包める。

```html
<ul class="card-grid">
  <li>
    <a class="card card--link" href="/articles/card-design">
      <article>
        <img
          src="card-design.jpg"
          alt=""
          width="640"
          height="360"
        />
        <div class="card__body">
          <h2>カードUIの設計</h2>
          <p>カード全体をクリック可能にする方法を解説します。</p>
          <p>2026年11月14日</p>
        </div>
      </article>
    </a>
  </li>
</ul>
```

`a`は、内部に別の操作要素がなければ、見出し、段落、画像などのフローコンテンツを含められる。

この例の画像は記事タイトルと同じ行き先を補助するサムネイルのため、`alt=""`にしてリンク名の重複を避けている。画像がタイトルから分からない重要情報を伝えるなら、簡潔な代替テキストを付ける。

### リンク名が長くなりすぎないようにする

リンクのアクセシブルネームは、基本的にリンク内のテキストから作られる。カード全体を包むと、タイトル、説明、日付などが一つの長いリンク名として伝わる場合がある。

```text
リンク: カードUIの設計 カード全体をクリック可能にする方法を解説します。2026年11月14日
```

内容が長い、メタ情報が多い、リンク一覧からタイトルだけを素早く探してほしい場合は、カード全体を`a`で包まず、次の主リンクパターンを使う。

`aria-label`でタイトルだけに上書きすると、画面に見える説明と支援技術が受け取るリンク名の関係が弱くなる。まずHTML構造と表示内容を簡潔にする。

## パターン2: 主リンクと副操作があるカード

タグリンク、お気に入りボタン、メニューなどがある場合は、カードをリンクで包まない。主リンクと副操作を兄弟要素にする。

```html
<article class="card">
  <img
    class="card__image"
    src="card-design.jpg"
    alt=""
    width="640"
    height="360"
  />

  <div class="card__body">
    <h2 class="card__title">
      <a class="card__primary-link" href="/articles/card-design">
        カードUIの設計
      </a>
    </h2>

    <p>カード全体をクリック可能にする方法を解説します。</p>

    <div class="card__actions">
      <a href="/categories/css">CSSの記事</a>
      <button type="button" aria-pressed="false">
        あとで読む
      </button>
    </div>
  </div>
</article>
```

この構造では、Tabキーで次の順に移動する。

1. 「カードUIの設計」への主リンク
2. 「CSSの記事」へのリンク
3. 「あとで読む」ボタン

カード自体へ`tabindex="0"`を付けない。カードと主リンクの二重のTab停止位置を作らず、実際に操作する要素だけへフォーカスを移す。

## CSSの疑似要素で主リンクを広げる

DOMでは主リンクと副操作を分けたまま、主リンクの`::after`をカード全体へ広げられる。

```css
.card {
  position: relative;
  display: grid;
  gap: 1rem;
  padding: 1rem;
  border: 0.125rem solid #777;
  border-radius: 0.5rem;
  background: Canvas;
  color: CanvasText;
}

.card__primary-link::after {
  content: "";
  position: absolute;
  inset: 0;
  border-radius: 0.5rem;
}

.card__actions {
  position: relative;
  z-index: 1;
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.card:hover {
  border-color: currentColor;
  box-shadow: 0 0.25rem 0.75rem rgb(0 0 0 / 20%);
}

.card:has(.card__primary-link:focus-visible) {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}
```

疑似要素は主リンクのクリック領域になる。HTML上はリンクの入れ子にならず、キーボードフォーカスは見出し内の主リンク一つだけ。

副操作へ`position: relative`と高い`z-index`を付け、主リンクの疑似要素より前面で操作できるようにする。リンク、ボタン、入力欄などすべての副操作で確認する。

### `:has()`を使わないフォールバック

```css
.card:focus-within {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}
```

`:focus-within`では、副操作へフォーカスしたときもカード全体にアウトラインが付く。副操作自身のフォーカス表示を消さず、現在位置を区別できるようにする。

## 疑似要素パターンの注意点

カード全体を覆う疑似要素にはトレードオフがある。

- カード本文のドラッグ選択を妨げることがある
- 本文の一部に別の操作を追加したら、前面へ出すCSSが必要になる
- ブラウザのリンクメニューがカードの広い範囲で表示される
- 要素の重なり順が複雑になる
- 埋め込み動画、地図、フォームなどと相性が悪い

説明文をコピーする需要が高いカードや、操作要素が多いカードでは、カード全体をクリック可能にしない。タイトルリンクや「記事を読む」リンクの余白を広げるだけでも十分な場合がある。

```css
.card__primary-link {
  display: inline-block;
  padding-block: 0.5rem;
}
```

クリック範囲を広げることより、意図しない遷移を防ぐことを優先する。

## JavaScriptでカードの余白クリックを補助する

テキスト選択を残したい場合、カードの余白を通常クリックしたときだけ主リンクへ委譲する方法もある。意味とキーボード操作は実際の`a`が担当する。

```html
<article class="card" data-clickable-card>
  <h2>
    <a href="/articles/card-design" data-card-primary-link>
      カードUIの設計
    </a>
  </h2>
  <p>カード全体をクリック可能にする方法を解説します。</p>
  <button type="button">あとで読む</button>
</article>
```

```js
const interactiveSelector = [
  'a',
  'button',
  'input',
  'select',
  'textarea',
  'label',
  'summary',
  '[contenteditable="true"]',
  '[data-card-ignore]',
].join(',');

document.addEventListener('click', (event) => {
  const card = event.target.closest('[data-clickable-card]');
  if (!card || event.defaultPrevented || event.button !== 0) return;

  if (event.metaKey || event.ctrlKey || event.shiftKey || event.altKey) return;
  if (event.target.closest(interactiveSelector)) return;

  const selection = window.getSelection();
  const selectedInsideCard = selection && !selection.isCollapsed && [
    selection.anchorNode,
    selection.focusNode,
  ].some((node) => node && card.contains(node));

  if (selectedInsideCard) return;

  const primaryLink = card.querySelector('[data-card-primary-link]');
  primaryLink?.click();
});
```

このJavaScriptは通常の左クリックだけを補助する。新しいタブで開く、URLをコピーするなどのリンク機能は主リンクから利用する。

カードへキーボードイベントを追加しない。Tabキーで主リンクへ移動し、Enterキーで開く標準操作を保つ。

JavaScriptが失敗しても主リンクと副操作は使える。カードの余白クリックはプログレッシブエンハンスメントとして扱う。

## CSSとJavaScriptのどちらを選ぶか

| 方法 | 長所 | 注意点 |
|---|---|---|
| 一つの`a`で包む | 単純でカード全体が本物のリンク | 副操作を置けない、リンク名が長くなり得る |
| `::after`で主リンクを広げる | 修飾クリックやリンクメニューをカード全体で使える | テキスト選択や重なりに注意 |
| JavaScriptで余白クリックを委譲 | テキスト選択と除外条件を制御しやすい | 修飾クリックやコンテキストメニューは主リンクのみ |
| 主リンクだけにする | 最も予測しやすく保守しやすい | カード全体はクリックできない |

カードの情報量、操作数、テキスト選択の必要性から選ぶ。すべてのカードを同じ方法にする必要はないが、同じ一覧内では操作を一貫させる。

## 同じ行き先のリンクを重複させない

```html
<!-- 避ける：同じ行き先が3回Tab順序に現れる -->
<article>
  <a href="/articles/card-design">
    <img src="card-design.jpg" alt="カードUIの設計" />
  </a>
  <h2><a href="/articles/card-design">カードUIの設計</a></h2>
  <a href="/articles/card-design">続きを読む</a>
</article>
```

同じカード内で画像、タイトル、「続きを読む」が同じURLなら、一つのリンクにまとめるか、主リンク一つを広げる。

重複リンクがあると、キーボード利用者は同じ行き先で何度も停止し、スクリーンリーダーのリンク一覧にも似た項目が増える。

画像とタイトルを一つのリンクにする例:

```html
<article class="card">
  <a class="card__primary-link" href="/articles/card-design">
    <img src="card-design.jpg" alt="" width="640" height="360" />
    <h2>カードUIの設計</h2>
  </a>
  <p>カード全体をクリック可能にする方法を解説します。</p>
</article>
```

## リンクの目的を明確にする

カードが並ぶページで「詳細を見る」だけのリンクが繰り返されると、リンク一覧から行き先を区別しにくい。

```html
<!-- 推奨：タイトル自体をリンクにする -->
<h2><a href="/articles/card-design">カードUIの設計</a></h2>
```

デザイン上「詳細を見る」が必要なら、視覚的に隠したタイトルを補足できる。

```html
<a href="/articles/card-design">
  詳細を見る<span class="visually-hidden">: カードUIの設計</span>
</a>
```

```css
.visually-hidden {
  position: absolute;
  inline-size: 1px;
  block-size: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}
```

`title`属性だけでリンクの目的を補わない。タッチ操作やキーボード操作では見つけにくく、支援技術への伝わり方にも差がある。

> 参照: [Understanding WCAG 2.2 SC 2.4.4 — Link Purpose (In Context)](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-in-context.html)

## 画像の代替テキスト

カードの画像とタイトルが同じリンク内にある場合、画像の役割を判断する。

### タイトルと同じ内容を繰り返すだけ

```html
<a href="/products/headphones">
  <img src="headphones.jpg" alt="" />
  <h2>ワイヤレスヘッドホン XM5</h2>
</a>
```

空の`alt`で重複を避ける。

### タイトルにない重要情報がある

```html
<a href="/properties/101">
  <img
    src="room-101.jpg"
    alt="南向きの窓と木製の床があるリビング"
  />
  <h2>桜町マンション 101号室</h2>
</a>
```

物件選びに画像の内容が重要なら、判断に必要な情報を簡潔に伝える。

### 画像だけがリンク

画像の`alt`がリンク名になる。画像の見た目だけでなく、行き先を表す名前にする。

## カードの構造

独立した記事や商品は`article`で表せる。単純なナビゲーション項目なら`li`と`a`だけでもよい。

```html
<section aria-labelledby="latest-articles-heading">
  <h2 id="latest-articles-heading">新着記事</h2>
  <ul class="card-grid">
    <li>
      <article class="card">
        <h3>
          <a href="/articles/card-design">カードUIの設計</a>
        </h3>
        <p>カード全体をクリック可能にする方法を解説します。</p>
      </article>
    </li>
    <li>
      <article class="card">
        <h3>
          <a href="/articles/form-design">フォームUIの設計</a>
        </h3>
        <p>入力エラーを伝える方法を解説します。</p>
      </article>
    </li>
  </ul>
</section>
```

- 一覧を`ul`でまとめる
- 一覧全体の見出しを置く
- ページの見出し階層に合わせてカード見出しのレベルを決める
- すべてのカードを`region`やランドマークにしない
- 見た目のためだけに`article`を使わない

## 副操作を区別する

お気に入りボタンのように状態が変わる操作は、カードの主リンクから独立させる。

```html
<button
  type="button"
  class="card__favorite"
  aria-pressed="false"
  aria-label="カードUIの設計をあとで読む"
>
  <span aria-hidden="true">☆</span>
</button>
```

状態が変わったら`aria-pressed`と見た目を同じ状態から更新する。

```js
document.addEventListener('click', (event) => {
  const button = event.target.closest('.card__favorite');
  if (!button) return;

  const pressed = button.getAttribute('aria-pressed') === 'true';
  const nextPressed = !pressed;
  const icon = button.querySelector('[aria-hidden="true"]');

  button.setAttribute('aria-pressed', String(nextPressed));
  if (icon) icon.textContent = nextPressed ? '★' : '☆';
});
```

アイコンだけのボタンでは、対象のカード名と操作をアクセシブルネームに含める。複数カードの「お気に入り」ボタンを区別できるようにする。

主リンクの疑似要素を広げている場合、このボタンが実際に前面でクリックできることを確認する。

## Reactで実装する

### 行き先が一つ

```jsx
function ArticleCard({ article }) {
  return (
    <a className="card card--link" href={article.href}>
      <article>
        <img
          src={article.image.src}
          alt=""
          width={article.image.width}
          height={article.image.height}
        />
        <h2>{article.title}</h2>
        <p>{article.summary}</p>
      </article>
    </a>
  );
}
```

`article.href`がない場合はクリック可能な`div`へ切り替えず、情報カードとして別コンポーネントにする。リンクと非リンクで見た目が同じでも、操作可能かどうかを見分けられるデザインにする。

### 副操作がある

```jsx
function ArticleCard({ article, saved, onSavedChange }) {
  return (
    <article className="card">
      <img
        className="card__image"
        src={article.image.src}
        alt=""
        width={article.image.width}
        height={article.image.height}
      />

      <div className="card__body">
        <h2 className="card__title">
          <a className="card__primary-link" href={article.href}>
            {article.title}
          </a>
        </h2>
        <p>{article.summary}</p>

        <div className="card__actions">
          <a href={article.category.href}>{article.category.name}の記事</a>
          <button
            type="button"
            aria-pressed={saved}
            aria-label={`${article.title}をあとで読む`}
            onClick={() => onSavedChange(!saved)}
          >
            <span aria-hidden="true">{saved ? '★' : '☆'}</span>
          </button>
        </div>
      </div>
    </article>
  );
}
```

カード本体へ`onClick`と`tabIndex`を追加して主リンクを再実装しない。主な遷移は常に`a href`へ任せる。

クライアントサイドルーターのリンクコンポーネントを使う場合も、最終的に`a`と`href`が出力され、新しいタブ、URLコピー、JavaScript無効時の移動を利用できることを確認する。

## クリック範囲とターゲットサイズ

カード全体をリンクにする目的の一つは、ポインターで押しやすくすること。ただし、副操作との境界が曖昧になるほど広げない。

- WCAG 2.2レベルAAでは、原則としてターゲットを24 × 24 CSS px以上にするか、十分な間隔を確保する
- 主リンクと副操作が重ならないようにする
- アイコンボタンにも十分な幅と高さを持たせる
- カード間に誤操作を防ぐ間隔を置く
- 拡大しても隣のカードへクリック範囲がはみ出さないようにする

```css
.card__actions a,
.card__actions button {
  min-inline-size: 2.75rem;
  min-block-size: 2.75rem;
  padding: 0.5rem;
}
```

44 × 44 CSS pxは、より操作しやすい目安として検討できる。最低値を満たすことだけでなく、利用場面と操作頻度からサイズを決める。

> 参照: [Understanding WCAG 2.2 SC 2.5.8 — Target Size (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)

## ホバーとフォーカスをそろえる

ホバーだけでカードのクリック可能性を示さない。キーボードフォーカスでも同等の手がかりを示す。

```css
.card:hover,
.card:has(.card__primary-link:focus-visible) {
  border-color: currentColor;
  box-shadow: 0 0.25rem 0.75rem rgb(0 0 0 / 20%);
}

.card:has(.card__primary-link:focus-visible) {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

@media (forced-colors: active) {
  .card:has(.card__primary-link:focus-visible) {
    outline-color: Highlight;
  }
}
```

- フォーカス表示を`outline: none`だけで消さない
- ホバーだけで現れる重要情報を置かない
- カード全体の変化とフォーカス対象自体の位置が分かるようにする
- 強制カラーモードでも境界とフォーカスを確認する
- タッチ環境ではホバーがない前提で設計する

## テキスト選択とドラッグ

カード本文には、住所、商品番号、引用などコピーしたい情報が含まれることがある。

- テキストをドラッグした後に遷移しない
- 画像をドラッグしたときの挙動を確認する
- 横スクロール領域でカードクリックが誤発火しないようにする
- カード内の選択可能テキストへ`user-select: none`を安易に付けない
- スワイプやドラッグだけで実行する機能を作らない

本文選択が重要なら、カード全体のリンク化より、主リンクだけを十分なサイズにする。

## よくある失敗

### `a`の中へ別の操作要素を入れる

主リンクと副操作を兄弟要素にする。カード全体を包めるのは、行き先が一つで他の操作がない場合だけ。

### カードへ`role="link"`を付ける

実際の`a href`を使う。リンクの標準動作を再実装しない。

### カードと主リンクの両方をTab停止位置にする

カード自体に`tabindex`を付けない。主リンクと副操作だけをTab順序へ含める。

### 同じURLのリンクを何度も置く

画像、タイトル、「続きを読む」を一つの主リンクへまとめる。

### すべてのリンクが「詳細を見る」

タイトル自体をリンクにするか、リンク名へ対象タイトルを含める。

### 疑似要素が副操作を覆う

副操作を前面へ出し、マウス、タッチ、キーボードで個別に操作できるか確認する。

### JavaScriptで内側の操作も遷移させる

リンク、ボタン、入力欄、ラベル、テキスト選択をカードクリックの対象から除外する。

### カードの見た目だけがクリック可能性を示す

リンクテキスト、下線、フォーカス表示など、ホバーしなくても操作可能と分かる手がかりを残す。

### 非リンクカードも手の形のカーソルになる

操作できないカードへ`cursor: pointer`やリンクと同じホバー効果を付けない。

## テストのチェックリスト

### HTMLと意味

- [ ] ページ移動には`a href`、動作には`button`を使っている
- [ ] `a`の中に別の`a`、ボタン、入力欄、`tabindex`要素がない
- [ ] カード自体を`role="link"`で再実装していない
- [ ] カード一覧が必要に応じて`ul`で構造化されている
- [ ] カード見出しがページの見出し階層に合っている
- [ ] 同じ行き先のリンクを不必要に重複させていない
- [ ] 情報カードを操作可能に見せていない

### キーボードとリンク機能

- [ ] Tabキーで主リンクと副操作へ論理的な順序で移動できる
- [ ] Enterキーで主リンクを開ける
- [ ] SpaceキーとEnterキーでボタンを操作できる
- [ ] カードと主リンクの二重フォーカスがない
- [ ] 新しいタブで開く、URLコピー、コンテキストメニューを利用できる
- [ ] JavaScriptを無効にしても主リンクを使用できる
- [ ] フォーカス位置をカードの見た目から確認できる

### ポインターと副操作

- [ ] カードのクリック範囲が隣のカードへはみ出していない
- [ ] 主リンクの範囲が副操作を覆っていない
- [ ] タグリンク、メニューボタン、お気に入りを個別に操作できる
- [ ] 24 × 24 CSS px以上または必要な間隔を確認した
- [ ] タッチ操作で隣のターゲットを誤って押しにくい
- [ ] 修飾クリックや中クリックの挙動を確認した

### 名前と画像

- [ ] 主リンクの名前から行き先を予測できる
- [ ] 「詳細を見る」だけのリンクが繰り返されていない
- [ ] アイコンボタンの名前に操作と対象がある
- [ ] 状態ボタンの`aria-pressed`と見た目が一致する
- [ ] タイトルと同じ内容の画像で代替テキストを重複させていない
- [ ] 重要な画像には判断に必要な代替テキストがある

### 表示と操作の競合

- [ ] テキストを選択しても意図せず遷移しない
- [ ] 200%・400%拡大でもカードと副操作を使用できる
- [ ] 狭い画面でリンクとボタンが重ならない
- [ ] 強制カラーモードでカード境界とフォーカスを確認できる
- [ ] ホバーなしでも操作可能な場所が分かる
- [ ] スクリーンリーダーのリンク一覧で各カードを区別できる

## 関連して読むとよいテーマ

- [セマンティックHTML — 正しいHTML要素を使う](./semantic-html.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス表示の設計 — :focus-visibleとWCAG 2.4.7 / 2.4.11](./focus-visible.md)
- [画像のalt属性の書き方 — 装飾画像と情報画像](./image-alt-text.md)
- [アクセシブルなパンくずリストの実装](./accessible-breadcrumb.md)
- [アクセシブルなディスクロージャーの実装 — 一部だけを開閉する](./accessible-disclosure.md)
- [手動テストのチェックリスト — 最低限確認すべき項目](./manual-testing-checklist.md)

## まとめ

- ページ移動には実際の`a href`を使う
- 行き先が一つで副操作がなければ、カードを一つのリンクで包める
- 副操作があるカードでは、主リンクと副操作を兄弟要素にする
- `a`の中へ別のリンク、ボタン、入力欄を入れない
- 主リンクの疑似要素を広げる場合、副操作とテキスト選択を確認する
- JavaScriptによる余白クリックは標準リンクを残した補助機能にする
- カードと主リンクの二重のTab停止位置を作らない
- 同じ行き先のリンクをカード内で繰り返さない
- リンク名へカードのタイトルを含め、行き先を区別できるようにする
- ホバーだけでなくフォーカスでもクリック可能性を示す
- クリック範囲、副操作、テキスト選択をマウス、タッチ、キーボードで確認する
