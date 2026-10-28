# アクセシブルなページネーションの実装 — 現在ページと前後への移動

## ページネーションとは

ページネーション（Pagination）は、長い一覧を複数のページに分け、現在位置や前後のページへ移動するためのナビゲーション。検索結果、商品一覧、記事一覧などで使われる。

見た目は数字の並びでも、役割は値を選ぶウィジェットではなく、**一連のページ間を移動するナビゲーション**。

- 全体を名前付きの`nav`で囲む
- ページ項目をリストで表す
- 移動先を実際の`a`要素とURLで提供する
- 現在ページを`aria-current="page"`で示す
- 前後リンクの目的をリンクテキストで伝える
- 動的に更新する場合は、URL、履歴、タイトル、フォーカスも同期する

専用の`role="pagination"`は存在しない。`menu`、`tablist`、`listbox`として実装する必要もない。

> 参照:
> - [W3C Design System — Pagination](https://design-system.w3.org/components/pagination.html)
> - [W3C WAI — ARIA26: Using aria-current to identify the current item in a set](https://www.w3.org/WAI/WCAG21/Techniques/aria/ARIA26)

## 基本のHTML

```html
<nav class="pagination" aria-label="検索結果のページ">
  <ul class="pagination__list">
    <li>
      <a href="?query=accessibility&page=2" rel="prev">
        <span aria-hidden="true">←</span>
        前のページ
      </a>
    </li>

    <li>
      <a href="?query=accessibility&page=1" aria-label="1ページ目">
        1
      </a>
    </li>
    <li aria-hidden="true" class="pagination__ellipsis">…</li>
    <li>
      <a href="?query=accessibility&page=2" aria-label="2ページ目">
        2
      </a>
    </li>
    <li>
      <a
        href="?query=accessibility&page=3"
        aria-label="3ページ目"
        aria-current="page"
      >
        3
      </a>
    </li>
    <li>
      <a href="?query=accessibility&page=4" aria-label="4ページ目">
        4
      </a>
    </li>
    <li aria-hidden="true" class="pagination__ellipsis">…</li>
    <li>
      <a href="?query=accessibility&page=20" aria-label="20ページ目">
        20
      </a>
    </li>

    <li>
      <a href="?query=accessibility&page=4" rel="next">
        次のページ
        <span aria-hidden="true">→</span>
      </a>
    </li>
  </ul>
</nav>
```

この例では、URLへ検索条件とページ番号の両方を残している。ページを再読み込みしても同じ結果を表示でき、リンクを新しいタブで開く、コピーする、ブックマークする、ブラウザの戻る・進むを使うといった標準機能を保てる。

## `nav`に用途が分かる名前を付ける

ページ内には、メインナビゲーション、パンくず、目次、ページネーションなど複数の`nav`が存在することがある。支援技術のランドマーク一覧で区別できる名前を付ける。

```html
<nav aria-label="商品一覧のページ">
  <!-- ページリンク -->
</nav>
```

画面上の見出しを名前に使う場合は`aria-labelledby`を使う。

```html
<nav aria-labelledby="pagination-heading">
  <h2 id="pagination-heading" class="visually-hidden">
    商品一覧のページ
  </h2>
  <!-- ページリンク -->
</nav>
```

- `aria-label="ページネーション"`でも役割は伝わる
- 同じページに複数ある場合は「検索結果のページ」「コメントのページ」のように区別する
- `nav`には暗黙のnavigationランドマークがあるため、`role="navigation"`の追加は不要

## 数字だけのリンクに目的を付ける

リンクテキストが`1`、`2`、`3`だけでも、ページネーションの`nav`内にあれば文脈を推測できる。しかし、リンク一覧から直接移動する利用者には「2ページ目」のような名前の方が明確。

```html
<a href="?page=2" aria-label="2ページ目">2</a>
```

`aria-label`には画面に見える数字を含める。見えるラベルとアクセシブルネームが大きく異なると、音声操作で指定しにくくなる。

別の方法として、視覚的に隠したテキストを使える。

```html
<a href="?page=2">
  <span class="visually-hidden">ページ</span>
  2
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

どちらか一つでよい。`aria-label`と隠しテキストを重ねて、同じ内容を二重に読ませない。

## 現在ページには`aria-current="page"`

```html
<a
  href="?page=3"
  aria-label="3ページ目"
  aria-current="page"
>
  3
</a>
```

`aria-current="page"`は、関連するページ集合の中で現在表示している項目を示す。

- 一つのページネーション内で一項目だけに付ける
- `aria-selected`や`aria-pressed`で代用しない
- 見た目でも現在位置を区別する
- 色だけでなく、太字、下線、形、文字など別の手がかりを使う

アクセシブルネームへ「現在のページ」を含めなくても、対応する支援技術は`aria-current`を読み上げる。利用環境で伝わり方を確認し、補足する場合も冗長にならないようにする。

### 現在ページをリンクにするか

現在ページをリンクとして残す方法と、テキストにする方法がある。

```html
<!-- リンクとして残す -->
<li><a href="?page=3" aria-current="page">3</a></li>

<!-- 再読み込みを不要にする -->
<li><span aria-current="page">3</span></li>
```

どちらでも現在位置は伝えられる。リポジトリ全体で一貫させる。

- リンクなら、ほかの番号と同じ形でリンク一覧から見つけやすい
- テキストなら、現在ページを誤って再読み込みしない
- テキストにしても、リンクと同じ大きさの表示領域を保つ
- `href`のない`a`要素を作らない

## 前後リンクを明確にする

```html
<a href="?page=2" rel="prev">前のページ</a>
<a href="?page=4" rel="next">次のページ</a>
```

`rel="prev"`と`rel="next"`は、そのリンクが論理的な一連の文書における前後のページであることを示す。リンクのアクセシブルネームを置き換えるものではないため、見えるテキストも「前のページ」「次のページ」のようにする。

> 参照: [WHATWG HTML Standard — Sequential link types](https://html.spec.whatwg.org/multipage/links.html#sequential-link-types)

矢印アイコンは装飾として隠す。

```html
<a href="?page=4" rel="next">
  次のページ
  <svg aria-hidden="true" focusable="false" viewBox="0 0 24 24">
    <path d="m9 18 6-6-6-6" />
  </svg>
</a>
```

「前へ」「次へ」だけでもページネーション内なら理解できるが、「前のページ」「次のページ」の方がリンクを単独で読んだときにも明確。

## 先頭・末尾では存在しないリンクを作らない

最初のページには前のページがなく、最後のページには次のページがない。

```html
<!-- 最初のページ：前のページはテキストで表示 -->
<span class="pagination__unavailable" aria-disabled="true">
  前のページ
</span>
```

または、項目自体を表示しない。重要なのは、移動できない`a`要素を作らないこと。

```html
<!-- 避ける -->
<a href="#" aria-disabled="true">前のページ</a>
```

`aria-disabled="true"`はリンクのクリックやキーボード操作を自動的には無効にしない。`href="#"`はページ先頭へ移動してしまう。利用できない状態を表示するなら`span`などの非インタラクティブ要素を使い、見た目でも区別する。

## 省略記号はリンクにしない

ページ数が多い場合は、すべての番号を並べず、現在ページの周辺、先頭、末尾を残す。

```html
<li><a href="?page=1" aria-label="1ページ目">1</a></li>
<li class="pagination__ellipsis" aria-hidden="true">…</li>
<li><a href="?page=24" aria-label="24ページ目">24</a></li>
<li><a href="?page=25" aria-current="page">25</a></li>
<li><a href="?page=26" aria-label="26ページ目">26</a></li>
<li class="pagination__ellipsis" aria-hidden="true">…</li>
<li><a href="?page=100" aria-label="100ページ目">100</a></li>
```

- `…`をリンクやボタンにしない
- 省略記号へ`aria-label="さらにページがあります"`を付ける必要は通常ない
- 特定ページへ直接移動する必要があるなら、ページ番号入力や検索など別の操作を用意する
- 小さい画面でも前後リンクと現在位置を優先して残す

省略記号を押すと番号が増えるUIにするなら、それは単なる記号ではなくボタン。「ページ一覧をさらに表示」のような明確な名前と状態管理が必要になる。

## ページ番号の範囲を作る

次の関数は、先頭、末尾、現在ページの前後一ページを残し、間を省略する。

```js
function createPageItems(currentPage, totalPages) {
  const pages = new Set([
    1,
    totalPages,
    currentPage - 1,
    currentPage,
    currentPage + 1,
  ]);

  const visiblePages = [...pages]
    .filter((page) => page >= 1 && page <= totalPages)
    .sort((a, b) => a - b);

  const items = [];

  visiblePages.forEach((page, index) => {
    const previousPage = visiblePages[index - 1];

    if (previousPage && page - previousPage > 1) {
      items.push({ type: 'ellipsis', key: `${previousPage}-${page}` });
    }

    items.push({ type: 'page', page });
  });

  return items;
}

createPageItems(5, 10);
// 1, …, 4, 5, 6, …, 10 に対応するデータ
```

ページ数が少ない場合や現在位置が端に近い場合も、重複や範囲外の番号を作らない。

## Reactでの実装例

```jsx
function Pagination({ currentPage, totalPages, createHref }) {
  const items = createPageItems(currentPage, totalPages);

  if (totalPages <= 1) return null;

  return (
    <nav className="pagination" aria-label="検索結果のページ">
      <ul className="pagination__list">
        <li>
          {currentPage > 1 ? (
            <a
              href={createHref(currentPage - 1)}
              rel="prev"
            >
              前のページ
            </a>
          ) : (
            <span className="pagination__unavailable" aria-disabled="true">
              前のページ
            </span>
          )}
        </li>

        {items.map((item) => (
          item.type === 'ellipsis' ? (
            <li key={item.key} aria-hidden="true">…</li>
          ) : (
            <li key={item.page}>
              <a
                href={createHref(item.page)}
                aria-label={`${item.page}ページ目`}
                aria-current={
                  item.page === currentPage ? 'page' : undefined
                }
              >
                {item.page}
              </a>
            </li>
          )
        ))}

        <li>
          {currentPage < totalPages ? (
            <a
              href={createHref(currentPage + 1)}
              rel="next"
            >
              次のページ
            </a>
          ) : (
            <span className="pagination__unavailable" aria-disabled="true">
              次のページ
            </span>
          )}
        </li>
      </ul>
    </nav>
  );
}

// 使用例
<Pagination
  currentPage={3}
  totalPages={20}
  createHref={(page) => `/search?query=accessibility&page=${page}`}
/>
```

クライアントサイドルーターの`Link`コンポーネントを使う場合も、最終的なHTMLが有効な`href`を持つ`a`要素になることを確認する。

## CSSで操作対象と現在位置を区別する

```css
.pagination__list {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.5rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.pagination a,
.pagination__unavailable {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-inline-size: 2.75rem;
  min-block-size: 2.75rem;
  padding-inline: 0.75rem;
  border: 1px solid currentColor;
  border-radius: 0.25rem;
}

.pagination a[aria-current="page"] {
  border-width: 0.1875rem;
  font-weight: 700;
  text-decoration: none;
  color: Canvas;
  background: CanvasText;
}

.pagination a:focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

.pagination__unavailable {
  opacity: 0.55;
}

.pagination__ellipsis {
  padding-inline: 0.25rem;
}
```

- 数字が一文字でも十分な操作領域を確保する
- リンク同士を密着させず、誤操作しにくい間隔を取る
- ホバーだけでなく`:focus-visible`を用意する
- 現在ページを色だけで区別しない
- 強制カラーモードでも境界、現在位置、フォーカスを確認する
- 200%・400%拡大では折り返しを許可する

CSSの`order`で番号の見た目だけを並べ替えない。DOM順序、読み上げ順序、Tab順序を、前のページから次のページへ進む論理的な順序に合わせる。

## 通常のページ遷移を基本にする

最も堅牢なのは、各ページを異なるURLで返す通常のリンク遷移。

- ブラウザがフォーカスとスクロールの初期状態を管理する
- ページタイトルと見出しをサーバー側で更新できる
- 戻る・進む、再読み込み、ブックマークがそのまま機能する
- JavaScriptが失敗しても移動できる
- 検索条件、並び順、ページ番号をURLから復元できる

ページ番号を変えたとき、ページ上部へ必ず戻す必要はない。利用者が一覧を続けて読む流れを考え、少なくとも新しい一覧の開始位置が分かるようにする。

## SPAで内容だけを更新する場合

クライアント側で一覧だけを差し替えると、画面は変わってもスクリーンリーダーの読み上げ位置やキーボードフォーカスは古いページネーションに残ることがある。

更新時は次を一つの処理として扱う。

1. 読み込み中であることを示す
2. 新しい一覧とページネーションを描画する
3. URLとブラウザ履歴を更新する
4. `document.title`を現在ページに合わせる
5. 新しい一覧の見出しへフォーカスを移す、または更新完了を通知する
6. 戻る・進む操作でも同じ状態を復元する

```html
<section id="search-results" aria-labelledby="results-heading">
  <h2 id="results-heading" tabindex="-1">
    検索結果 — 3ページ目
  </h2>
  <div id="results-list"><!-- 一覧 --></div>
</section>

<p id="pagination-status" class="visually-hidden" role="status"></p>
```

見出しへフォーカスを移す方法は、新しい内容の開始位置を明確にする。一方、フォーカスを移さない設計なら、「3ページ目、20件を表示しました」のような短いメッセージを既存の`role="status"`要素へ設定する。

フォーカス移動とライブ通知を同時に行うと、同じ情報を二重に読み上げることがある。対象環境で検証し、どちらを主な通知方法にするか決める。

> 参照:
> - [WCAG 2.2 Understanding SC 2.4.3 — Focus Order](https://www.w3.org/WAI/WCAG22/Understanding/focus-order)
> - [WCAG 2.2 Understanding SC 4.1.3 — Status Messages](https://www.w3.org/WAI/WCAG22/Understanding/status-messages)

### 読み込み中の状態

```js
async function updateResults(url) {
  const results = document.getElementById('search-results');
  const heading = document.getElementById('results-heading');
  const status = document.getElementById('pagination-status');

  results.setAttribute('aria-busy', 'true');
  status.textContent = '検索結果を読み込んでいます';

  try {
    const response = await fetch(url, {
      headers: { Accept: 'application/json' },
    });
    if (!response.ok) throw new Error('結果を取得できませんでした');

    const data = await response.json();

    renderResults(data);
    history.pushState({ page: data.page }, '', url);
    document.title = `検索結果 — ${data.page}ページ目`;

    heading.textContent = `検索結果 — ${data.page}ページ目`;
    status.textContent = '';
    heading.focus();
  } catch (error) {
    status.textContent = error.message;
  } finally {
    results.removeAttribute('aria-busy');
  }
}
```

これは状態管理の要点を示す簡略例。実装では次も必要になる。

- 修飾キー付きクリックや新しいタブを妨げない
- 連続操作時に古いリクエストを中止する
- 通信失敗時も現在の内容と再試行手段を残す
- `popstate`でURLに対応する内容を復元する
- 読み込み中に同じ操作を連打した場合を処理する
- サーバー描画されたリンクを残し、JavaScriptなしでも移動可能にする

## フィルターや並び順を変えたとき

検索条件や並び順を変更すると、結果件数と総ページ数が変わる。

- 原則として1ページ目へ戻す
- 現在ページが新しい総ページ数を超えないようにする
- URLへ条件、並び順、ページ番号を反映する
- 「42件、1ページ目を表示」のように結果を伝える
- 現在ページの`aria-current`を更新する
- ページ番号の範囲と前後リンクを再計算する

存在しないページを空の結果として見せるより、有効なページへ戻したことを明確に伝える。

## 「もっと見る」と無限スクロールとの違い

### 「もっと見る」ボタン

同じページの末尾へ項目を追加する操作ならリンクではなくボタンが適切。

```html
<button type="button" aria-controls="product-list">
  商品をさらに20件表示
</button>
```

追加後は、件数を`role="status"`で通知し、フォーカスを予期せず移動しない。ボタン自体がDOMから消える場合は、次の論理的な場所へフォーカスを移す。

### 無限スクロール

スクロールだけで自動追加すると、ページ末尾へ到達できない、現在位置を共有できない、戻ったときに位置を復元できない問題が起きやすい。

- URLまたはページ単位の復元手段を用意する
- 自動読み込みを止められるようにする
- フッターなど後続コンテンツへ到達できるようにする
- 「もっと見る」ボタンや通常のページネーションを代替として提供する
- 追加件数と読み込み状態を支援技術へ伝える

見た目の好みだけでページネーションを無限スクロールへ置き換えない。

## よくある失敗

### 数字を`button`で実装する

別URLの結果へ移動するならリンクを使う。ボタンにすると、新しいタブ、URLのコピー、ブックマークなどリンクの機能を失う。

### 現在ページを色だけで示す

見た目の`active`クラスだけでは現在位置が支援技術へ伝わらない。`aria-current="page"`と色以外の視覚表現を組み合わせる。

### すべてのリンク名が数字だけ

リンク一覧では文脈を失いやすい。「3ページ目」のようなアクセシブルネームを付ける。

### 無効なリンクへ`aria-disabled`だけを付ける

ARIAは動作を止めない。存在しない前後ページはリンクを省くか、非インタラクティブ要素で表示する。

### 省略記号が操作できそうに見える

何もしない`…`へリンクの見た目やフォーカスを与えない。操作させるなら目的が分かるボタンにする。

### SPA更新後も古い位置へフォーカスが残る

新しい一覧の開始位置が分からなくなる。見出しへのフォーカス移動または簡潔なステータス通知を設計する。

### URLを更新しない

再読み込み、共有、戻る・進むで現在ページを失う。ページ番号と検索条件をURLへ残す。

### 見た目の順序だけをCSSで変える

数字の読み上げ順とTab順序が不自然になる。DOMを論理的な前後関係の順に並べる。

## テストのチェックリスト

### 構造と名前

- [ ] ページネーションが名前付きの`nav`に含まれている
- [ ] ページ項目が`ul`または`ol`で表されている
- [ ] 各番号リンクの目的を「3ページ目」のように理解できる
- [ ] 現在ページだけに`aria-current="page"`がある
- [ ] 前後リンクの名前が単独でも理解できる
- [ ] 省略記号がフォーカス可能になっていない
- [ ] 存在しないページへのリンクがない

### キーボードと表示

- [ ] Tab / Shift + Tabで論理的な順序に移動できる
- [ ] Enterキーで各リンクを開ける
- [ ] フォーカス表示が常に見える
- [ ] 現在ページを色だけで区別していない
- [ ] 操作対象に十分な大きさと間隔がある
- [ ] 200%・400%拡大で折り返しても順序を理解できる
- [ ] 強制カラーモードでもリンク、現在位置、フォーカスが分かる

### URLとページ遷移

- [ ] 各リンクに実際の移動先URLがある
- [ ] 検索条件と並び順がページ移動後も維持される
- [ ] 再読み込みしても同じページを表示する
- [ ] 戻る・進むで以前のページと状態を復元できる
- [ ] リンクを新しいタブで開ける
- [ ] JavaScriptが失敗してもページ移動できる

### 動的更新

- [ ] 読み込み中、完了、失敗の状態が伝わる
- [ ] 更新後に新しい一覧の開始位置を把握できる
- [ ] URL、履歴、`document.title`が表示内容と一致する
- [ ] フォーカス移動とライブ通知が重複しすぎない
- [ ] フィルター変更後に有効なページ番号へ戻る
- [ ] 連続操作や通信失敗後も操作を続けられる

## 関連して読むとよいテーマ

- [aria-current — ナビゲーションの現在位置を示す](./aria-current.md)
- [アクセシブルなパンくずリストの実装](./accessible-breadcrumb.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)
- [ランドマークロールとページ構造の設計](./landmark-regions.md)
- [フォーカスインジケーターのカスタマイズ — :focus-visibleの活用](./focus-visible.md)

## まとめ

- ページネーションは名前付きの`nav`と通常のリンク一覧で表す
- 各ページを実URLへリンクし、標準のリンク操作、履歴、共有、再読み込みを保つ
- 現在ページは一つだけ`aria-current="page"`で示し、見た目でも色以外の方法で区別する
- 数字だけのリンクには「3ページ目」のような分かりやすいアクセシブルネームを付ける
- 前後リンクには明確なテキストと`rel="prev"`／`rel="next"`を使う
- 存在しないページや省略記号を操作可能なリンクにしない
- SPAでは一覧だけでなく、URL、履歴、タイトル、フォーカスまたはステータス通知も更新する
- フィルター変更、通信失敗、拡大表示、キーボード、スクリーンリーダーまで含めてテストする
