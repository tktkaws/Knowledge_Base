# アクセシブルなハンバーガーメニューの実装 — ナビゲーションを開いたときのフォーカス

## ハンバーガーメニューとは

ハンバーガーメニューは、主に狭い画面でサイトナビゲーションを開閉するUI。「三本線のアイコン」ではなく、**ナビゲーションの表示を切り替えるボタン** として設計する。

- ボタンの目的を「メニュー」とテキストで伝える
- 開閉状態を`aria-expanded`で伝える
- 制御対象を`aria-controls`で関連付ける
- ナビゲーションは`nav`と通常のリンク一覧で構成する
- 閉じたリンクをTab順序とアクセシビリティツリーから外す
- 開閉方式に応じてフォーカスの移動方法を決める

ハンバーガーアイコンが見えていても、名前・状態・キーボード操作がなければ、支援技術やキーボードの利用者にはメニューとして伝わらない。

> 参照: [W3C WAI-ARIA APG — Example Disclosure Navigation Menu](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/examples/disclosure-navigation/)

## 二つの開き方を区別する

ナビゲーションを開いたときの見た目ではなく、背景を操作できるかでパターンを選ぶ。

| 方式 | 開いた後の画面 | フォーカス |
|---|---|---|
| Disclosure型 | ページ内へ展開し、背景の本文も操作できる | 開閉ボタンに残す。Tabでリンクへ進む |
| モーダル型 | 画面を覆い、閉じるまで背景を操作できない | メニュー内へ移し、内部に閉じ込める |

見た目が画面端から出てくるだけでは、モーダルとは限らない。

- 背景を操作できるなら、Disclosure型として通常のTab順序を維持する
- 背景を操作できないなら、Dialogパターンとしてモーダルの要件をすべて満たす
- `aria-modal="true"`を付けながら背景へTab移動できるような、中間状態を作らない

この記事ではDisclosure型を基本実装とし、後半で`dialog`を使うモーダル型を扱う。

> 参照:
> - [W3C WAI-ARIA APG — Disclosure Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/)
> - [W3C WAI-ARIA APG — Dialog (Modal) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

## `menu`ロールを付けない

一般的なサイトナビゲーションは、別ページへ移動するリンクの一覧。デスクトップアプリのコマンド一覧を表すARIAの`menu`とは操作モデルが異なる。

```html
<!-- 避ける：通常のサイトナビゲーションをARIA menuにしている -->
<ul role="menu">
  <li role="menuitem"><a href="/products">製品</a></li>
  <li role="menuitem"><a href="/support">サポート</a></li>
</ul>

<!-- 通常のリンク一覧を使う -->
<nav aria-label="メインナビゲーション">
  <ul>
    <li><a href="/products">製品</a></li>
    <li><a href="/support">サポート</a></li>
  </ul>
</nav>
```

- TabキーとShift + Tabキーでリンク間を移動する
- 矢印キー操作やロービングタブインデックスを追加しない
- リスト構造でリンクのまとまりと項目数を伝える
- 複数の`nav`がある場合は、それぞれを区別できる名前を付ける

APGのDisclosure Navigationも、日常語としてmenuと呼ぶ一方、ARIAの`menu`ロールは使っていない。

## 基本のHTML

JavaScriptが読み込めない場合にもナビゲーションへ到達できるよう、初期HTMLではリンクを表示し、開閉ボタンを隠しておく。JavaScriptの初期化時に、狭い画面だけ開閉UIへ切り替える。

```html
<header class="site-header">
  <a class="site-logo" href="/">Example</a>

  <button
    class="menu-toggle"
    type="button"
    aria-expanded="false"
    aria-controls="primary-navigation"
    hidden
  >
    <svg
      class="menu-toggle__icon"
      aria-hidden="true"
      focusable="false"
      viewBox="0 0 24 24"
      width="24"
      height="24"
    >
      <path d="M4 6h16M4 12h16M4 18h16" />
    </svg>
    <span>メニュー</span>
  </button>

  <nav id="primary-navigation" aria-label="メインナビゲーション">
    <ul>
      <li><a href="/products">製品</a></li>
      <li><a href="/pricing">料金</a></li>
      <li><a href="/support">サポート</a></li>
      <li><a href="/company">会社情報</a></li>
    </ul>
  </nav>
</header>
```

### 開閉ボタン

| 属性 | 役割 |
|---|---|
| `type="button"` | フォーム内でも送信ボタンにならないようにする |
| `aria-expanded` | ナビゲーションが開いているかを伝える |
| `aria-controls` | 制御するナビゲーションの`id`を参照する |
| `hidden` | JavaScriptの初期化前は不要なボタンを隠す |

三本線のSVGは装飾なので`aria-hidden="true"`にする。ボタンには画面上でも読める「メニュー」というテキストを置くと、アイコンの意味を知らない利用者にも目的が分かる。

### ナビゲーション

- `nav`要素の暗黙のnavigationランドマークを使う
- `aria-label`は「メニュー」ではなく「メインナビゲーション」のように、ナビゲーション自体の種類を表す
- 現在のページに対応するリンクには`aria-current="page"`を付ける
- DOM上で開閉ボタンの直後に置くと、開いた後のTab順序が自然になる

```html
<li><a href="/products" aria-current="page">製品</a></li>
```

## Disclosure型のJavaScript

```js
class ResponsiveNavigation {
  constructor(button, navigation, breakpoint = '(min-width: 64rem)') {
    this.button = button;
    this.navigation = navigation;
    this.desktopQuery = window.matchMedia(breakpoint);

    this.button.addEventListener('click', () => {
      this.setOpen(!this.isOpen);
    });

    const closeOnEscape = (event) => {
      if (
        event.key === 'Escape' &&
        !this.desktopQuery.matches &&
        this.isOpen
      ) {
        this.setOpen(false, { restoreFocus: true });
      }
    };

    this.button.addEventListener('keydown', closeOnEscape);
    this.navigation.addEventListener('keydown', closeOnEscape);

    this.desktopQuery.addEventListener('change', () => {
      this.syncViewport();
    });

    this.syncViewport();
  }

  get isOpen() {
    return this.button.getAttribute('aria-expanded') === 'true';
  }

  setOpen(open, { restoreFocus = false } = {}) {
    this.button.setAttribute('aria-expanded', String(open));
    this.navigation.hidden = !open;

    if (!open && restoreFocus) {
      this.button.focus();
    }
  }

  syncViewport() {
    if (this.desktopQuery.matches) {
      // デスクトップでは常にナビゲーションを表示する
      const buttonHadFocus = document.activeElement === this.button;

      this.button.hidden = true;
      this.button.setAttribute('aria-expanded', 'true');
      this.navigation.hidden = false;

      if (buttonHadFocus) {
        this.navigation.querySelector('a')?.focus();
      }

      return;
    }

    // モバイル表示へ切り替わったときは閉じた状態から始める
    const navigationHadFocus = this.navigation.contains(document.activeElement);

    this.button.hidden = false;
    this.setOpen(false, { restoreFocus: navigationHadFocus });
  }
}

const menuButton = document.querySelector('.menu-toggle');
const primaryNavigation = document.getElementById('primary-navigation');

if (menuButton && primaryNavigation) {
  new ResponsiveNavigation(menuButton, primaryNavigation);
}
```

この実装では、次の状態を一つの処理で同期する。

| 状態 | ボタン | ナビゲーション |
|---|---|---|
| モバイル・閉じている | 表示、`aria-expanded="false"` | `hidden` |
| モバイル・開いている | 表示、`aria-expanded="true"` | 表示 |
| デスクトップ | 非表示 | 常に表示 |

ブレークポイントをまたいだときも、CSSの見た目だけでなくHTMLの状態を更新する。デスクトップからモバイルへ戻ったときは、以前の状態を引き継がず閉じる設計にしている。

## Disclosure型のフォーカス

### 開いた直後はボタンに残す

```text
メニューボタン → 製品 → 料金 → サポート → 会社情報 → 次の操作要素
```

ナビゲーションを開いても、最初のリンクへ自動的にフォーカスを移さない。ボタンの直後に`nav`があるため、次にTabキーを押すと最初のリンクへ進む。

- 状態変化を`aria-expanded`から確認できる
- 利用者が意図せず現在位置を失わない
- Shift + Tabキーで自然にボタンへ戻れる
- スクリーンリーダーの読み上げ順とDOM順序が一致する

これはページ内へ展開するDisclosureの動作。モーダル型では、開いた内部へフォーカスを移す。

### Escapeキーで閉じる

APGのDisclosureパターン自体が求める基本キーはEnterとSpaceだが、展開したナビゲーションがほかの内容を覆う場合、Escapeキーで閉じられると操作しやすい。

- ナビゲーション内でEscapeキーを押す
- ナビゲーションを閉じる
- 開閉ボタンへフォーカスを戻す

閉じる前にフォーカスを戻さないと、フォーカス中のリンクが`hidden`になり、現在位置を失う可能性がある。

> 参照: [W3C WAI-ARIA APG — Disclosure Navigation Keyboard Support](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/examples/disclosure-navigation/#kbd_label)

### Tabキーを閉じ込めない

Disclosure型では背景の本文も操作できる。最後のナビゲーションリンクからTabキーを押したら、ページ内の次の操作要素へ進める。

- 最後のリンクから最初のリンクへループさせない
- 背景を`inert`や`aria-hidden`にしない
- フォーカストラップを実装しない
- 矢印キーを必須操作にしない

APGの例には任意の矢印キー操作もあるが、Tab順序を置き換えるものではない。一般的なサイトナビゲーションなら、標準のTab操作を保つ方が単純で予測しやすい。

## CSSの基本

```css
.site-header {
  align-items: center;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}

.menu-toggle {
  align-items: center;
  background: transparent;
  border: 1px solid currentColor;
  color: inherit;
  display: inline-flex;
  gap: 0.5rem;
  min-block-size: 44px;
  padding-inline: 0.75rem;
}

.menu-toggle:focus-visible {
  outline: 3px solid CanvasText;
  outline-offset: 3px;
}

.menu-toggle__icon {
  fill: none;
  stroke: currentColor;
  stroke-linecap: round;
  stroke-width: 2;
}

#primary-navigation {
  flex-basis: 100%;
}

#primary-navigation ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

#primary-navigation a {
  display: block;
  min-block-size: 44px;
  padding: 0.75rem;
}

@media (min-width: 64rem) {
  #primary-navigation {
    flex-basis: auto;
  }

  #primary-navigation ul {
    display: flex;
  }
}
```

- フォーカスインジケーターを消さない
- アイコンに`currentColor`を使い、強制カラーモードでも見えるようにする
- リンクとボタンに十分な操作領域を持たせる
- 開いたナビゲーションがズーム時や狭い画面で画面外へはみ出さないようにする
- 固定ヘッダーを使う場合、フォーカス中のリンクをヘッダーや画面端で隠さない

## 見た目だけで隠さない

```css
/* 不十分：画面外へ動かしただけで、リンクはTab順序に残る */
.site-nav--closed {
  transform: translateX(-100%);
}
```

閉じたナビゲーションを`transform`、`opacity`、高さだけで見えなくすると、キーボードフォーカスやスクリーンリーダーからは利用できる場合がある。

- 閉じた最終状態では`hidden`を使う
- アニメーションする場合、閉じ終わった後に`hidden`を付ける
- 閉じる途中も見えないリンクへフォーカスが移らないようにする
- `aria-hidden="true"`だけでリンクを隠さない
- `display: none`と`aria-expanded`を別々の状態として管理しない

状態管理を単純にできるなら、アニメーションなしで`hidden`を切り替える実装が堅牢。

## モーダル型にする場合

画面全体を覆い、閉じるまで背景を操作させないナビゲーションには、HTMLの`dialog`要素を`showModal()`で開く方法を使える。

```html
<button
  id="open-navigation"
  type="button"
  aria-expanded="false"
  aria-controls="navigation-dialog"
>
  メニュー
</button>

<dialog id="navigation-dialog" aria-labelledby="navigation-title">
  <div class="navigation-dialog__header">
    <h2 id="navigation-title">メニュー</h2>
    <button id="close-navigation" type="button" autofocus>
      閉じる
    </button>
  </div>

  <nav aria-label="メインナビゲーション">
    <ul>
      <li><a href="/products">製品</a></li>
      <li><a href="/pricing">料金</a></li>
      <li><a href="/support">サポート</a></li>
      <li><a href="/company">会社情報</a></li>
    </ul>
  </nav>
</dialog>
```

```js
const openButton = document.getElementById('open-navigation');
const closeButton = document.getElementById('close-navigation');
const navigationDialog = document.getElementById('navigation-dialog');

openButton.addEventListener('click', () => {
  openButton.setAttribute('aria-expanded', 'true');
  navigationDialog.showModal();
});

closeButton.addEventListener('click', () => {
  navigationDialog.close();
});

navigationDialog.addEventListener('close', () => {
  openButton.setAttribute('aria-expanded', 'false');

  if (document.contains(openButton)) {
    openButton.focus();
  }
});
```

`showModal()`を使うことで、ブラウザが背景をinertにし、Tabキーの移動をダイアログ内に制限する。Escapeキーによる終了にも対応する。

### モーダル型で必要な動作

- 開いたらメニュー内へフォーカスを移す
- TabキーとShift + Tabキーをメニュー内で循環させる
- Escapeキーで閉じられるようにする
- 画面上に明確な「閉じる」ボタンを置く
- 背景を見た目でも覆い、操作不能にする
- 閉じたら原則として開いたボタンへフォーカスを戻す
- ダイアログに見える名前を付ける

`dialog`を`show()`で開くと非モーダルになる。背景を操作不能にするナビゲーションには`showModal()`を使う。

> 参照:
> - [W3C — H102: Creating modal dialogs with the HTML dialog element](https://www.w3.org/WAI/WCAG22/Techniques/html/H102)
> - [WHATWG HTML Standard — The `dialog` element](https://html.spec.whatwg.org/multipage/interactive-elements.html#the-dialog-element)

## レスポンシブ切り替えの注意点

CSSだけでボタンとナビゲーションの見た目を切り替えると、ARIAと実際の表示がずれる場合がある。

### 状態をブレークポイントに同期する

- デスクトップでは開閉ボタンを隠し、ナビゲーションを常に表示する
- モバイルへ切り替わったら、閉じた状態へ初期化する
- モバイルで開いたままデスクトップへ切り替わっても、リンクを隠さない
- 画面回転やウィンドウ幅変更でも状態を更新する
- メディアクエリとJavaScriptで同じブレークポイントを使う

### 同じナビゲーションを二重に置かない

```html
<!-- 避ける：デスクトップ用とモバイル用で同じリンクを重複させる -->
<nav class="desktop-nav">...</nav>
<nav class="mobile-nav">...</nav>
```

重複したナビゲーションは、IDの重複、現在位置の更新漏れ、非表示側へのフォーカス、スクリーンリーダーで同じランドマークが二つ現れる問題を作りやすい。可能なら一つのDOMを表示方法だけ切り替える。

## SPAでリンクを選んだ後

クライアントサイドルーティングでは、リンクを選んでもページ全体が再読み込みされない。

- モバイルメニューを閉じる
- `aria-expanded`を`false`へ戻す
- 新しいページタイトルを更新する
- 主要コンテンツの見出しなど、論理的な場所へフォーカスを移す
- 現在位置のリンクへ`aria-current="page"`を付け替える

リンク選択後に毎回メニューボタンへ戻すと、新しいコンテンツを探すために同じナビゲーションをもう一度通ることになる。ルート変更時のフォーカス先は、単なる「閉じる操作」と分けて設計する。

## よくある失敗

### アイコンに名前がない

```html
<!-- 悪い例：目的が伝わらない -->
<button type="button">
  <svg><!-- 三本線 --></svg>
</button>

<!-- 良い例：画面にも名前を表示する -->
<button type="button" aria-expanded="false" aria-controls="site-nav">
  <svg aria-hidden="true"><!-- 三本線 --></svg>
  <span>メニュー</span>
</button>
```

### CSSだけで開閉する

チェックボックスや`:hover`だけで開閉すると、buttonとしての意味、開閉状態、Escape操作、フォーカス管理が不足しやすい。実際のbuttonと明示的な状態管理を使う。

### 開いたら必ず最初のリンクへ飛ばす

Disclosure型では、ボタンの次にナビゲーションを置き、フォーカスをボタンへ残す。モーダル型の「内部へフォーカスを移す」という要件を混ぜない。

### 閉じてもリンクがTab順序に残る

見た目だけを画面外へ移動していないか確認する。閉じた状態では`hidden`を使い、リンクをTab順序から外す。

### 画面を覆うのに背景を操作できる

ナビゲーションが本文を覆い隠す一方で、Tabキーが見えない背景要素へ移ると、フォーカス位置を見失う。背景を操作させない設計ならモーダルとして実装する。

### 閉じる方法がメニューボタンだけ

大きく展開するナビゲーションでは、Escapeキーや内部の「閉じる」ボタンも用意する。拡大表示中でも、閉じる操作が画面内に見えるか確認する。

## テストのチェックリスト

### 名前・状態・構造

- [ ] 開閉ボタンがbuttonとして認識される
- [ ] ボタンの名前からメニューを開く操作だと分かる
- [ ] 開閉状態と`aria-expanded`が一致する
- [ ] `aria-controls`が実在するナビゲーションの`id`を参照する
- [ ] ナビゲーションがnavigationランドマークとして認識される
- [ ] リンクがリスト構造で伝わる
- [ ] 現在ページのリンクに`aria-current="page"`がある

### キーボードとフォーカス

- [ ] EnterキーとSpaceキーで開閉できる
- [ ] Disclosure型では、開いた直後もフォーカスがボタンに残る
- [ ] Tabキーで開いたナビゲーションのリンクへ順番に移動できる
- [ ] 閉じたリンクへTabキーで移動しない
- [ ] Escapeキーで閉じるとボタンへフォーカスが戻る
- [ ] Disclosure型では最後のリンクからページ本文へ移動できる
- [ ] モーダル型ではフォーカスが内部に留まり、閉じると起点へ戻る
- [ ] すべてのフォーカス位置を視覚的に確認できる

### レスポンシブ表示

- [ ] JavaScriptが使えなくてもナビゲーションリンクへ到達できる
- [ ] デスクトップではナビゲーションが常に表示される
- [ ] モバイルへ切り替えたとき、ボタンとナビゲーションの状態が一致する
- [ ] 画面回転後もリンクが消えたり二重表示されたりしない
- [ ] 200%・400%拡大でも開閉ボタンとリンクを操作できる
- [ ] 固定ヘッダーやオーバーレイがフォーカスを隠さない
- [ ] 強制カラーモードでもアイコンとフォーカスが見える
- [ ] タッチで操作できる十分な領域がある

### 支援技術

- [ ] ボタンの名前、役割、開閉状態が読み上げられる
- [ ] ナビゲーションの名前とリンク数を把握できる
- [ ] 開閉後に読み上げ位置を見失わない
- [ ] モーダル型では背景コンテンツへ移動できない
- [ ] 音声操作で「メニュー」ボタンを指定できる

## 関連して読むとよいテーマ

- [アクセシブルなディスクロージャーの実装 — 一部だけを開閉する](./accessible-disclosure.md)
- [アクセシブルなドロップダウンメニューの実装](./accessible-dropdown-menu.md)
- [アクセシブルなモーダルダイアログの実装](./accessible-modal-dialog.md)
- [aria-expanded / aria-hidden / aria-controls の活用](./aria-expanded-hidden-controls.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [aria-current — ナビゲーションの現在位置を示す](./aria-current.md)
- [フォーカスインジケーターのカスタマイズ — :focus-visibleの活用](./focus-visible.md)

## まとめ

- 一般的なハンバーガーメニューは、ARIAの`menu`ではなく、Disclosureボタンと通常のナビゲーションリンクで実装する
- Disclosure型では開いた後もフォーカスをボタンに残し、Tabキーでリンクへ進める
- 閉じたナビゲーションは`hidden`でTab順序とアクセシビリティツリーから外す
- Escapeキーで閉じる場合は、開閉ボタンへフォーカスを戻す
- 背景を操作不能にする場合だけモーダル型とし、フォーカス移動、フォーカストラップ、Escape、復帰を実装する
- JavaScriptが無効でもリンクを利用できる初期HTMLにし、レスポンシブ切り替えでは表示とARIAの状態を同期する
- デスクトップ用とモバイル用でナビゲーションを複製せず、可能なら一つのDOMを使う
