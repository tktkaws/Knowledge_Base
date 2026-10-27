# アクセシブルなメガメニューの実装 — 大きなナビゲーションをキーボードで辿る

## メガメニューとは

メガメニュー（Mega Menu）は、サイトの主要カテゴリから、複数の見出しやリンクを含む大きなパネルを展開するナビゲーション。商品数が多いECサイト、大学、行政、企業サイトなどで使われる。

通常のドロップダウンより多くの情報を一度に見せられる一方、次の問題が起きやすい。

- Tabキーで何十個ものリンクを通過しなければならない
- ポインターが少し外れただけでパネルが閉じる
- フォーカスが見えない、または開いたパネルに隠れる
- 見た目の列だけがあり、見出しやリストの構造がない
- `role="menu"`を付けたのに、期待される矢印キー操作がない
- 拡大表示や狭い画面で、リンクや閉じる手段が画面外へ出る

メガメニューは専用のARIAパターンではない。一般的なサイト内リンクを並べるなら、**Disclosureパターンと通常のナビゲーション構造**を組み合わせる。

> 参照:
> - [W3C WAI-ARIA APG — Example Disclosure Navigation Menu](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/examples/disclosure-navigation/)
> - [W3C WAI — Menus Tutorial](https://www.w3.org/WAI/tutorials/menus/)

## `menu`ロールを使わない

日常語の「メニュー」と、ARIAの`menu`は意味が異なる。

| 種類 | 主な内容 | 基本の操作 |
|---|---|---|
| サイトナビゲーション | 別ページへ移動するリンク | Tab / Shift + Tabで順番に移動 |
| ARIA Menu | アプリケーションのコマンド | 矢印キーで項目間を移動 |

ARIAの`menu`と`menubar`は、デスクトップアプリのメニューバーに近い複合ウィジェット。ロービング`tabindex`、矢印キー、Home、End、文字入力など、一式の操作が期待される。

```html
<!-- 避ける：通常のサイトリンクをARIA Menuとして扱う -->
<ul role="menubar">
  <li role="menuitem">製品</li>
  <li role="menuitem">サポート</li>
</ul>

<!-- サイトナビゲーションはネイティブHTMLで表す -->
<nav aria-label="メインナビゲーション">
  <ul>
    <li><a href="/products/">製品</a></li>
    <li><a href="/support/">サポート</a></li>
  </ul>
</nav>
```

APGのDisclosure Navigationも「menu」という日常語を使っているが、`menu`ロールは使っていない。通常のリンクとしてスクリーンリーダーの読み上げ機能を利用でき、Tabキーの予測可能な操作を保てる。

> 参照:
> - [W3C WAI-ARIA APG — Menu and Menubar Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/menubar/)
> - [W3C WAI — Application Menus](https://www.w3.org/WAI/tutorials/menus/application-menus/)

## 基本の操作モデル

この記事では、次の操作モデルを採用する。

| 操作 | 動作 |
|---|---|
| Tab / Shift + Tab | ボタンとリンクをDOM順に移動する |
| Enter / Space | フォーカス中の開閉ボタンでパネルを開閉する |
| Escape | 開いているパネルを閉じ、開閉ボタンへ戻る |
| パネル外のクリック | パネルを閉じる |
| ナビゲーション外へのフォーカス移動 | パネルを閉じる |

矢印キー、Home、Endは追加しない。APGのDisclosure Navigationでは任意の操作として紹介されているが、通常のWebページでは矢印キーをスクロールやスクリーンリーダーの読み上げに使うことがある。

実装する場合も、Tab移動を置き換えず、追加のショートカットとして提供する。サイト固有の操作を増やす前に、実際の利用者に必要かを確認する。

## HTMLの構造

メガメニュー全体を一つの`nav`で囲み、カテゴリとパネルを入れ子のリストで表す。

```html
<nav class="mega-navigation" aria-label="メインナビゲーション">
  <ul class="mega-navigation__list">
    <li class="mega-navigation__item">
      <button
        id="products-button"
        class="mega-navigation__button"
        type="button"
        aria-expanded="false"
        aria-controls="products-panel"
        hidden
      >
        製品
        <span class="mega-navigation__icon" aria-hidden="true"></span>
      </button>

      <div
        id="products-panel"
        class="mega-panel"
        aria-labelledby="products-button"
      >
        <div class="mega-panel__intro">
          <p class="mega-panel__title">製品を探す</p>
          <a href="/products/">製品一覧</a>
        </div>

        <section aria-labelledby="products-by-use">
          <h2 id="products-by-use">用途から探す</h2>
          <ul>
            <li><a href="/products/team/">チームで使う</a></li>
            <li><a href="/products/personal/">個人で使う</a></li>
            <li><a href="/products/education/">教育で使う</a></li>
          </ul>
        </section>

        <section aria-labelledby="products-by-type">
          <h2 id="products-by-type">種類から探す</h2>
          <ul>
            <li><a href="/products/software/">ソフトウェア</a></li>
            <li><a href="/products/devices/">デバイス</a></li>
            <li><a href="/products/services/">サービス</a></li>
          </ul>
        </section>
      </div>
    </li>

    <li class="mega-navigation__item">
      <button
        id="support-button"
        class="mega-navigation__button"
        type="button"
        aria-expanded="false"
        aria-controls="support-panel"
        hidden
      >
        サポート
        <span class="mega-navigation__icon" aria-hidden="true"></span>
      </button>

      <div
        id="support-panel"
        class="mega-panel"
        aria-labelledby="support-button"
      >
        <section aria-labelledby="support-heading">
          <h2 id="support-heading">サポートを探す</h2>
          <ul>
            <li><a href="/support/start/">使い始める</a></li>
            <li><a href="/support/manual/">マニュアル</a></li>
            <li><a href="/support/contact/">お問い合わせ</a></li>
          </ul>
        </section>
      </div>
    </li>

    <li class="mega-navigation__item">
      <a href="/company/">会社情報</a>
    </li>
  </ul>
</nav>
```

### 開閉ボタン

カテゴリがパネルの開閉だけを行うなら、`button`を使う。

| 属性 | 役割 |
|---|---|
| `type="button"` | フォーム内でも送信を起こさない |
| `aria-expanded` | パネルの開閉状態を伝える |
| `aria-controls` | 制御するパネルのIDを示す |
| `aria-labelledby` | パネルを開閉ボタンの名前と関連付ける |

初期HTMLでは開閉ボタンだけを`hidden`にし、リンク群は表示しておく。JavaScriptが利用できない場合も主要な移動先へ到達できる。初期化に成功したらボタンを表示し、パネルを閉じて開閉UIへ切り替える。

カテゴリ名からもページへ移動させたい場合、同じ要素を「移動」と「開閉」の両方に使うと動作が予測しにくい。次のどちらかに整理する。

- カテゴリ名は開閉ボタンにし、パネル内へ「製品一覧」のリンクを置く
- カテゴリのリンクと、隣接する開閉ボタンを別の操作として置く

リンクの中にボタンを入れたり、ボタンの中にリンクを入れたりしてはいけない。

### パネル内の構造

視覚的な列だけでなく、見出しとリストでリンクの関係を表す。

- 大分類ごとに見出しを付ける
- 各グループのリンクを`ul`と`li`でまとめる
- 宣伝文や画像より、主要な移動先を先に置く
- 同じ名前で移動先が異なるリンクを並べない
- 現在ページには`aria-current="page"`を付ける

```html
<a href="/products/software/" aria-current="page">
  ソフトウェア
</a>
```

## JavaScriptで状態を同期する

一度に一つだけパネルを開き、`hidden`と`aria-expanded`を常に同期する。

```js
class MegaNavigation {
  constructor(root) {
    this.root = root;
    this.buttons = [
      ...root.querySelectorAll('button[aria-controls]'),
    ];

    this.root.dataset.enhanced = 'true';

    this.buttons.forEach((button) => {
      button.hidden = false;

      button.addEventListener('click', () => {
        const shouldOpen = button.getAttribute('aria-expanded') !== 'true';
        this.closeAll();

        if (shouldOpen) {
          this.open(button);
        }
      });
    });

    this.root.addEventListener('keydown', (event) => {
      if (event.key !== 'Escape') return;

      const button = this.getOpenButton();
      if (!button) return;

      event.preventDefault();
      this.close(button);
      button.focus();
    });

    this.root.addEventListener('focusout', () => {
      requestAnimationFrame(() => {
        if (!this.root.contains(document.activeElement)) {
          this.closeAll();
        }
      });
    });

    document.addEventListener('pointerdown', (event) => {
      if (!this.root.contains(event.target)) {
        this.closeAll();
      }
    });

    this.closeAll();
  }

  getPanel(button) {
    return document.getElementById(button.getAttribute('aria-controls'));
  }

  getOpenButton() {
    return this.buttons.find(
      (button) => button.getAttribute('aria-expanded') === 'true'
    );
  }

  open(button) {
    const panel = this.getPanel(button);
    if (!panel) return;

    button.setAttribute('aria-expanded', 'true');
    panel.hidden = false;
  }

  close(button) {
    const panel = this.getPanel(button);

    button.setAttribute('aria-expanded', 'false');
    if (panel) panel.hidden = true;
  }

  closeAll() {
    this.buttons.forEach((button) => this.close(button));
  }
}

const navigation = document.querySelector('.mega-navigation');

if (navigation) {
  new MegaNavigation(navigation);
}
```

### 開いた直後にフォーカスを移さない

Disclosure型では、ボタンを押した直後もフォーカスをボタンに残す。次にTabキーを押すと、DOM上で直後にあるパネルの最初のリンクへ移動する。

```text
製品ボタン → 製品一覧 → チームで使う → 個人で使う → …
```

自動的に最初のリンクへ移すと、開閉ボタンへ戻る操作が増え、ポインター操作とキーボード操作で状態遷移も変わる。モーダルではないため、パネル内にフォーカスを閉じ込めない。

### Escapeキーで閉じる

Escapeキーでは、開いているパネルを閉じ、対応するボタンへフォーカスを戻す。フォーカス中のリンクを`hidden`にした後で復帰先を探すのではなく、制御元のボタンを保持してから閉じる。

パネル外のクリックだけを閉じる方法にしてはいけない。キーボードや音声操作でも閉じられる必要がある。

## CSSで大きなパネルを配置する

```css
.mega-navigation {
  position: relative;
}

.mega-navigation__list {
  display: flex;
  gap: 0.5rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.mega-navigation__button,
.mega-navigation a {
  min-block-size: 2.75rem;
  padding: 0.625rem 0.875rem;
  color: inherit;
}

.mega-navigation__button {
  border: 0;
  background: transparent;
  font: inherit;
  cursor: pointer;
}

.mega-navigation__icon {
  display: inline-block;
  inline-size: 0.5rem;
  block-size: 0.5rem;
  margin-inline-start: 0.4rem;
  border-inline-end: 0.125rem solid currentColor;
  border-block-end: 0.125rem solid currentColor;
  transform: rotate(45deg) translateY(-0.15rem);
}

[aria-expanded="true"] .mega-navigation__icon {
  transform: rotate(225deg) translate(-0.15rem, -0.15rem);
}

.mega-panel {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 2rem;
  padding: 1.5rem;
  border: 1px solid CanvasText;
  background: Canvas;
  color: CanvasText;
}

.mega-navigation[data-enhanced="true"] .mega-panel {
  position: absolute;
  z-index: 10;
  inset-block-start: 100%;
  inset-inline: 0;
  max-block-size: min(70vh, 40rem);
  overflow: auto;
  box-shadow: 0 0.5rem 1.5rem rgb(0 0 0 / 20%);
}

.mega-panel[hidden] {
  display: none;
}

.mega-panel ul {
  margin: 0;
  padding: 0;
  list-style: none;
}

.mega-panel a {
  display: inline-block;
}

.mega-navigation :focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}
```

`hidden`の要素へ`.mega-panel { display: grid; }`が適用されると、CSSの詳細度や読み込み順によって再表示されることがある。`.mega-panel[hidden] { display: none; }`を明示し、閉じたリンクがTab順序に残らないことを確認する。

### 拡大表示とスクロール

大きなパネルは、200%・400%拡大や文字間隔の変更で画面に収まらなくなる。

- 高さを固定せず、`max-block-size`と`overflow: auto`を使う
- パネルの端でフォーカス表示を切らない
- 固定ヘッダーとパネルで、フォーカス中のリンクを完全に隠さない
- 横スクロールを前提にせず、一列へ折り返せるようにする
- ブラウザのズーム、文字サイズ、狭いウィンドウで確認する

> 参照: [WCAG 2.2 Understanding SC 2.4.11 — Focus Not Obscured (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum)

## ホバーだけで開かない

メガメニューを`:hover`だけで開くと、キーボード、タッチ、音声操作では利用できない。ポインターが意図せず通過しただけで大きなパネルが表示される問題もある。

基本はボタンのクリック、Enter、Spaceで明示的に開く。ホバーを追加する場合も、ボタン操作を残し、次の要件を満たす。

### Dismissible：移動せず閉じられる

パネルがほかの内容を覆う場合、ポインターやフォーカスを動かさずにEscapeキーなどで閉じられるようにする。

### Hoverable：パネルへ移動できる

ボタンからパネルへポインターを移しても閉じないようにする。ボタンとパネルの間に反応しない隙間を作らない。

### Persistent：読む時間を奪わない

ポインターが境界から一瞬外れただけで閉じないようにする。表示は、ホバーまたはフォーカスが外れる、利用者が閉じる、内容が無効になるまで維持する。

CSSの`:hover`と`:focus-within`だけで複雑な状態を混在させると、`aria-expanded`や`hidden`と表示が食い違いやすい。ホバー対応が本当に必要なら、同じ状態管理へ統合して入力方法ごとにテストする。

> 参照: [WCAG 2.2 Understanding SC 1.4.13 — Content on Hover or Focus](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus)

## モバイルでは同じ情報構造を保つ

狭い画面では、横長のパネルをそのまま縮小せず、縦に展開するDisclosureへ変える。

```css
@media (max-width: 48rem) {
  .mega-navigation__list {
    display: block;
  }

  .mega-panel {
    position: static;
    grid-template-columns: 1fr;
    max-block-size: none;
    padding: 1rem;
    overflow: visible;
    border-inline: 0;
    box-shadow: none;
  }
}
```

- デスクトップとモバイルでリンク名と移動先を変えない
- DOMを複製して、同じIDや現在位置を二重に作らない
- 画面回転やウィンドウ幅の変更後も、開閉状態を確認する
- モバイルで画面全体を覆う場合は、DisclosureではなくモーダルDialogとして要件を満たす
- ハンバーガーメニューの内側に置く場合、外側と内側のEscape操作を整理する

モバイル専用ナビゲーションを別に複製するより、同じHTMLをCSSで一列に変える方が、情報構造と状態を一致させやすい。

## リンク数を増やしすぎない

技術的にTab移動できても、数十個のリンクを毎回通過する設計は負担になる。メガメニューをサイトマップの代わりにしない。

- 利用頻度の高い移動先を優先する
- 見出しでグループを予測できるようにする
- 「すべて見る」リンクから一覧ページへ移動できるようにする
- サイト内検索、パンくず、フッターなど別の移動方法も用意する
- 宣伝カードや自動再生コンテンツをナビゲーションへ詰め込まない
- 実際の利用データとユーザーテストで項目を見直す

Tabキーの回数だけを減らすために、通常のリンクをTab順序から外してはいけない。まず情報設計を整理する。

## よくある失敗

### `role="menu"`だけを付ける

ロールは操作を自動実装しない。一般的なサイトリンクなら、`nav`、リスト、Disclosureを使う。

### フォーカスだけで自動的に開く

Tabキーで移動しただけで大きなパネルが開くと、表示変化が予測しにくい。EnterまたはSpaceによる明示的な操作を基本にする。

### ポインターが斜めに移動すると閉じる

ボタンとパネルの間に隙間がある、または`mouseleave`ですぐ閉じる実装は、細かなポインター操作が難しい利用者を妨げる。クリックで状態を固定できるようにする。

### 複数のパネルが同時に開く

大きなパネルが重なり、現在のカテゴリやフォーカス位置が分かりにくくなる。新しいカテゴリを開くときは、以前のパネルを閉じる。

### DOM順序と見た目の順序が違う

CSS Gridの配置だけで順序を入れ替えると、Tab移動と読み上げが視覚順序から外れる。意味のあるDOM順序を先に作る。

### 閉じたリンクへフォーカスできる

透明化や画面外への移動だけでは、リンクがTab順序に残る。閉じたパネルは`hidden`などで操作対象から外す。

### フォーカス表示がパネルの背面に隠れる

重なり順、固定ヘッダー、スクロール領域を確認する。フォーカス中の項目を少なくとも一部は見える状態に保つ。

## テストのチェックリスト

### 構造と状態

- [ ] メガメニュー全体が名前付きの`nav`に含まれている
- [ ] カテゴリとリンクの階層がリストで表されている
- [ ] リンクグループに分かりやすい見出しがある
- [ ] 通常のサイトナビゲーションへ`menu`ロールを付けていない
- [ ] `aria-expanded`とパネルの表示状態が一致する
- [ ] `aria-controls`が存在する一意なIDを参照している
- [ ] 現在ページが`aria-current="page"`で伝わる

### キーボードとフォーカス

- [ ] Tab / Shift + Tabでボタンと全リンクへDOM順に移動できる
- [ ] EnterキーとSpaceキーでパネルを開閉できる
- [ ] 開いた直後、フォーカスが開閉ボタンに残る
- [ ] Escapeキーで閉じ、対応するボタンへ戻る
- [ ] 閉じたパネルのリンクへフォーカスできない
- [ ] フォーカス表示が常に見える
- [ ] パネル外へ移動した後に不要なパネルが残らない

### ポインター、タッチ、表示

- [ ] ホバーを使わなくてもクリックまたはタップで開閉できる
- [ ] ホバーで表示する場合、パネルへポインターを移動できる
- [ ] パネルがほかの内容を覆う場合、移動せずに閉じられる
- [ ] 200%・400%拡大でもリンクと閉じる操作を利用できる
- [ ] 内容が画面高を超えてもスクロールできる
- [ ] タッチターゲット同士が過度に密集していない
- [ ] 強制カラーモードでも開閉状態とフォーカスが分かる

### レスポンシブと支援技術

- [ ] 狭い画面でリンクが一列に並び、横へはみ出さない
- [ ] 画面回転後も表示と`aria-expanded`が一致する
- [ ] 読み上げでナビゲーション名、ボタン名、開閉状態が分かる
- [ ] 見出しやリスト単位でパネル内を辿れる
- [ ] VoiceOver、NVDAなど対象環境で開閉を繰り返して確認した
- [ ] JavaScriptが失敗した場合も主要ページへ移動する代替手段がある

## 関連して読むとよいテーマ

- [アクセシブルなドロップダウンメニューの実装](./accessible-dropdown-menu.md)
- [アクセシブルなディスクロージャーの実装 — 一部だけを開閉する](./accessible-disclosure.md)
- [アクセシブルなハンバーガーメニューの実装 — ナビゲーションを開いたときのフォーカス](./accessible-hamburger-menu.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [aria-expanded / aria-hidden / aria-controls の活用](./aria-expanded-hidden-controls.md)
- [ランドマークロールとページ構造の設計](./landmark-regions.md)

## まとめ

- メガメニューは専用のARIAパターンではなく、サイトナビゲーションならDisclosureと通常のリンク一覧で構成する
- 一般的なサイトリンクへ`menu`や`menubar`ロールを付けず、Tab / Shift + Tabの操作を維持する
- 開閉には`button`、`aria-expanded`、`aria-controls`、`hidden`を使い、状態を同期する
- 開いた直後はフォーカスをボタンに残し、次のTabキーでパネル内へ進める
- Escapeキーで閉じ、対応する開閉ボタンへフォーカスを戻す
- ホバーは唯一の操作にせず、追加する場合は閉じられる・ポインターを移せる・表示が維持される要件を満たす
- 見出しとリストで情報を整理し、リンク数そのものも増やしすぎない
- 拡大、狭い画面、強制カラー、キーボード、スクリーンリーダーで実際にテストする
