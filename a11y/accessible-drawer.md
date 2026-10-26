# アクセシブルなドロワーの実装 — 端から出るパネルとモーダルの違い

## ドロワーとは

ドロワー（Drawer）は、画面の端から現れるパネルの総称。ナビゲーション、絞り込み条件、設定、補助ツールなどを表示するために使われる。

「右からスライドする」「背景に半透明の幕がある」といった見た目だけでは、アクセシビリティ上の役割は決まらない。

- 常に表示され、本文と並んで操作できる
- 開閉できるが、開いている間も本文を操作できる
- 開くと本文を操作できなくなり、パネル内の作業に限定される

この違いによって、HTML構造とフォーカス管理が変わる。

## `drawer`というARIAロールはない

ドロワー専用の`role="drawer"`は存在しない。見た目ではなく、利用者がどのように操作するかに対応するセマンティクスを選ぶ。

| 種類 | 背景の操作 | 主なHTML・パターン | 開いたときのフォーカス |
|---|---|---|---|
| 常設型 | できる | `aside`、`nav`など | 移動しない |
| 非モーダル型 | できる | Disclosure + `aside`など | 開閉ボタンに残す |
| モーダル型 | できない | `dialog` + `showModal()` | ドロワー内へ移す |

次の質問で判断する。

1. ドロワーを閉じずに本文を操作できる設計か
2. ドロワーが本文の操作対象を覆い隠すか
3. Tabキーをドロワーの外へ移動させるか
4. 背景をクリック・タップ・支援技術で操作できるか

背景を操作できない設計なら、見た目が細いサイドパネルでもモーダルダイアログとして扱う。

> 参照:
> - [W3C WAI-ARIA APG — Disclosure Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/)
> - [W3C WAI-ARIA APG — Dialog (Modal) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

## 常設型のドロワー

デスクトップで本文の横に常に表示する目次やツールパネルは、開閉ウィジェットではない。

```html
<main class="workspace">
  <article>
    <h1>注文一覧</h1>
    <!-- メインコンテンツ -->
  </article>

  <aside aria-labelledby="filter-heading">
    <h2 id="filter-heading">絞り込み</h2>
    <!-- 絞り込みフォーム -->
  </aside>
</main>
```

- 表示時にフォーカスを移動しない
- `role="dialog"`や`aria-modal`を付けない
- 開閉しないなら`aria-expanded`も不要
- 複数の補足領域がある場合、見出しやラベルで区別する
- `aside`が内容に合わない場合は、目的に応じて`nav`、`section`、通常の`div`を選ぶ

CSSで画面端に配置しても、HTML上の意味は変わらない。

## 非モーダル型のドロワー

絞り込みパネルのように、開いたまま検索結果や本文を操作できるドロワーはDisclosureとして実装できる。

### HTML

```html
<button
  id="open-filter-drawer"
  type="button"
  aria-expanded="false"
  aria-controls="filter-drawer"
>
  絞り込み条件
</button>

<aside
  id="filter-drawer"
  class="drawer drawer--non-modal"
  aria-labelledby="filter-drawer-title"
  hidden
>
  <div class="drawer__header">
    <h2 id="filter-drawer-title">絞り込み条件</h2>
    <button id="close-filter-drawer" type="button">
      閉じる
    </button>
  </div>

  <form>
    <fieldset>
      <legend>在庫状況</legend>
      <label>
        <input type="checkbox" name="stock" value="available">
        在庫あり
      </label>
    </fieldset>
  </form>
</aside>
```

### JavaScript

```js
const openButton = document.getElementById('open-filter-drawer');
const closeButton = document.getElementById('close-filter-drawer');
const drawer = document.getElementById('filter-drawer');

function setDrawerOpen(open, { restoreFocus = false } = {}) {
  openButton.setAttribute('aria-expanded', String(open));
  drawer.hidden = !open;

  if (!open && restoreFocus) {
    openButton.focus();
  }
}

openButton.addEventListener('click', () => {
  const isOpen = openButton.getAttribute('aria-expanded') === 'true';
  setDrawerOpen(!isOpen);
});

closeButton.addEventListener('click', () => {
  setDrawerOpen(false, { restoreFocus: true });
});

drawer.addEventListener('keydown', (event) => {
  if (event.key === 'Escape') {
    setDrawerOpen(false, { restoreFocus: true });
  }
});
```

### フォーカスの動き

```text
開くボタン → ドロワー内の閉じるボタン → フォーム → 本文の次の操作要素
```

- 開いた直後は開閉ボタンにフォーカスを残す
- 次のTabキーで、DOM順序に従ってドロワー内へ進める
- ドロワー内へフォーカスを閉じ込めない
- 閉じるボタンやEscapeキーで閉じた場合は、開いたボタンへ戻す
- 外部の状態変更で閉じる場合、フォーカスが内側にあれば先に戻す

開いたパネルが本文の上へ重なり、背景側のフォーカスを隠す場合は、非モーダルとして成立しにくい。本文を横へ押し出す、十分な表示領域を確保する、またはモーダル型へ変更する。

> 参照:
> - [アクセシブルなディスクロージャーの実装 — 一部だけを開閉する](./accessible-disclosure.md)
> - [W3C — Understanding SC 2.4.11 Focus Not Obscured (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum)

## モーダル型のドロワー

ドロワーを開いている間、背景を操作させない場合はモーダルダイアログ。HTMLの`dialog`要素を`showModal()`で開くと、背景のinert化とフォーカス範囲の制限をブラウザに任せられる。

### HTML

```html
<button
  id="open-settings-drawer"
  type="button"
  aria-expanded="false"
  aria-controls="settings-drawer"
>
  表示設定
</button>

<dialog
  id="settings-drawer"
  class="drawer drawer--modal"
  aria-labelledby="settings-drawer-title"
>
  <div class="drawer__header">
    <h2 id="settings-drawer-title">表示設定</h2>
    <button id="close-settings-drawer" type="button" autofocus>
      閉じる
    </button>
  </div>

  <form>
    <label>
      文字サイズ
      <select name="font-size">
        <option value="medium">標準</option>
        <option value="large">大きい</option>
      </select>
    </label>
  </form>
</dialog>
```

`dialog`には見える見出しを置き、`aria-labelledby`で名前にする。内容が複数のフォームや区画を含む場合、全体を`aria-describedby`で一度に読み上げさせる必要はない。

### JavaScript

```js
const openButton = document.getElementById('open-settings-drawer');
const closeButton = document.getElementById('close-settings-drawer');
const drawer = document.getElementById('settings-drawer');

openButton.addEventListener('click', () => {
  openButton.setAttribute('aria-expanded', 'true');
  drawer.showModal();
});

closeButton.addEventListener('click', () => {
  drawer.close();
});

drawer.addEventListener('close', () => {
  openButton.setAttribute('aria-expanded', 'false');

  if (document.contains(openButton)) {
    openButton.focus();
  }
});
```

`dialog`を`show()`で開くと非モーダルになる。背景を操作不能にする設計では`showModal()`を使う。

### モーダル型で必要な動作

| 操作 | 動作 |
|---|---|
| 開く | ドロワー内の論理的な要素へフォーカスを移す |
| `Tab` | ドロワー内の次の要素へ移動し、外へ出ない |
| `Shift + Tab` | ドロワー内の前の要素へ移動し、外へ出ない |
| `Escape` | ドロワーを閉じる |
| 閉じるボタン | ドロワーを閉じる |
| 閉じた後 | 原則として開いたボタンへフォーカスを戻す |

APGは、ダイアログ内に見える閉じるボタンを含めることを強く推奨している。背景のクリックやスワイプだけを終了方法にしない。

> 参照:
> - [W3C WAI-ARIA APG — Dialog (Modal) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)
> - [W3C — H102: Creating modal dialogs with the HTML dialog element](https://www.w3.org/WAI/WCAG22/Techniques/html/H102)

## `showModal()`と`inert`

モーダルな`dialog`がトップレイヤーにある間、HTML標準ではそれ以外の文書内容がinertになる。

- クリックやタップの対象にならない
- Tabキーのフォーカス対象にならない
- 支援技術から通常の操作対象として扱われない
- ダイアログだけが祖先のinert状態から外れて操作できる

そのため、`showModal()`を使う実装では、背景の各要素へ`aria-hidden="true"`を付けて回る必要はない。

```html
<!-- 避ける：dialogの祖先までaria-hiddenにするとdialog自身も隠れ得る -->
<div id="app" aria-hidden="true">
  <dialog open>...</dialog>
</div>
```

カスタム実装で`inert`を使う場合も、ドロワー自身をinertな祖先の中へ置かない。解除漏れが起きるとページ全体を操作できなくなるため、状態管理と例外処理を一か所へまとめる。

> 参照: [WHATWG HTML Standard — Inert subtrees and modal dialogs](https://html.spec.whatwg.org/dev/interaction.html#inert-subtrees)

## 初期フォーカスを決める

モーダル型では、開いたときにフォーカスを内部へ移す。常に最初の入力欄へ移せばよいわけではない。

| 内容 | 初期フォーカスの候補 |
|---|---|
| 短い設定パネル | 閉じるボタン、最初の入力欄 |
| 長い説明や複数の区画 | `tabindex="-1"`を付けた見出し |
| 破壊的操作を含む | キャンセルなど安全な操作 |
| 主要操作が明確 | 最もよく使う操作 |

見出しへ初期フォーカスを置く例。

```html
<h2 id="cart-title" tabindex="-1">カート</h2>
```

```js
drawer.showModal();
document.getElementById('cart-title').focus();
```

大きなドロワーで最初の入力欄へフォーカスすると、見出しが画面外へスクロールし、何が開いたか分かりにくくなることがある。

## CSSで画面端へ配置する

`dialog`の意味とモーダル動作はHTMLとJavaScriptで作り、CSSは見た目を画面端のパネルにする。

```css
.drawer--modal {
  block-size: 100dvh;
  border: 0;
  inline-size: min(90vw, 28rem);
  inset-block: 0;
  inset-inline-end: 0;
  margin: 0;
  max-block-size: none;
  max-inline-size: none;
  overflow-y: auto;
  padding: 1rem;
}

.drawer--modal::backdrop {
  background: rgb(0 0 0 / 55%);
}

.drawer__header {
  align-items: center;
  display: flex;
  gap: 1rem;
  justify-content: space-between;
}

.drawer button:focus-visible,
.drawer a:focus-visible,
.drawer input:focus-visible,
.drawer select:focus-visible {
  outline: 3px solid CanvasText;
  outline-offset: 3px;
}
```

- `inset-inline-end`を使うと、文章方向に応じた終端側へ配置できる
- `100dvh`でモバイルブラウザの動的な表示領域へ追従する
- 内容が画面高を超える場合は、ドロワー内部をスクロール可能にする
- 閉じるボタンを見失わないよう、必要ならヘッダーをstickyにする
- 強制カラーモードでもパネル境界、ボタン、フォーカスが見えるようにする

## 開閉アニメーション

アニメーションは意味やフォーカス管理の代わりにならない。

- 開く前に表示・操作可能な状態へする
- 閉じる開始時に、内側のフォーカスを論理的な場所へ移す
- 閉じ終わったら`hidden`または`dialog.close()`で操作対象から外す
- 連打やアニメーション中断でも、表示とARIAの状態を一致させる
- `prefers-reduced-motion: reduce`では移動量を減らすか、即時に切り替える

```css
@media (prefers-reduced-motion: no-preference) {
  .drawer--modal[open] {
    animation: drawer-enter 180ms ease-out;
  }

  @keyframes drawer-enter {
    from {
      transform: translateX(100%);
    }

    to {
      transform: translateX(0);
    }
  }
}
```

この例は終端が右側の横書きレイアウトを想定する。左側や右から左へ書くレイアウトでは、移動方向も配置に合わせる。

終了アニメーションのために`close()`を遅らせる場合、見えなくなったドロワーが操作可能なまま残らないようにする。要件がなければ、即時に閉じる方が単純で堅牢。

## スワイプだけにしない

「端からスワイプして開く」「横へドラッグして閉じる」は補助操作として提供できるが、唯一の方法にしない。

- クリック・タップできる「開く」ボタンを置く
- ドロワー内に「閉じる」ボタンを置く
- キーボードでも同じ機能を利用できる
- スワイプの方向や距離を正確に再現できなくても操作できる
- ドラッグ中に通常のページスクロールを不必要に妨げない

WCAG 2.2では、パスに依存するジェスチャには単純なポインター操作を、ドラッグ操作にはドラッグ不要の単一ポインター操作を用意するよう求めている。キーボード対応だけでは、タッチやポインティングデバイスの代替にならない。

> 参照:
> - [W3C — Understanding SC 2.5.1 Pointer Gestures](https://www.w3.org/WAI/WCAG22/Understanding/pointer-gestures)
> - [W3C — Understanding SC 2.5.7 Dragging Movements](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements)

## 背景クリックで閉じる場合

背景のオーバーレイをクリックして閉じる機能は任意。実装する場合も、次の方法を残す。

- 画面内の閉じるボタン
- Escapeキー
- 完了・キャンセルなど、タスクに応じた明示的な操作

`event.target === dialog`だけで背景クリックを判定すると、ダイアログ要素内の余白をクリックした場合まで閉じることがある。パネルの表面と背景のクリック領域を明確に分け、フォーム操作中の意図しない終了を防ぐ。

入力途中の値が失われる場合は、背景クリックで即座に閉じる設計自体を再検討する。

## レスポンシブで種類を変える場合

モバイルではモーダル、デスクトップでは常設パネルに変える設計では、CSSで位置だけを変えず、操作モデルも同期する。

- デスクトップへ切り替える前にモーダル状態を終了する
- 非表示になる開閉ボタンへフォーカスを残さない
- モバイルへ戻ったとき、背景のinert状態を残さない
- `aria-expanded`、`open`、`hidden`を現在の表示と一致させる
- 同じフォームを二重に配置して、IDや入力値を重複させない
- 画面回転とブラウザの拡大でも切り替えを確認する

`dialog`をCSSだけで常設の`aside`のように見せても、dialogとしての意味は残る。常設時にdialogが適切でないなら、コンポーネント構造またはレンダリングする要素を切り替える。

## Reactでのモーダル型実装

```jsx
import { useEffect, useRef } from 'react';

function Drawer({ id, title, open, onClose, children }) {
  const dialogRef = useRef(null);
  const triggerRef = useRef(null);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (!dialog) return;

    if (open && !dialog.open) {
      triggerRef.current = document.activeElement;
      dialog.showModal();
    } else if (!open && dialog.open) {
      dialog.close();
    }
  }, [open]);

  const requestClose = () => {
    onClose();
  };

  const handleNativeClose = () => {
    triggerRef.current?.focus();
  };

  return (
    <dialog
      ref={dialogRef}
      id={id}
      className="drawer drawer--modal"
      aria-labelledby={`${id}-title`}
      onCancel={(event) => {
        event.preventDefault();
        requestClose();
      }}
      onClose={handleNativeClose}
    >
      <div className="drawer__header">
        <h2 id={`${id}-title`}>{title}</h2>
        <button type="button" onClick={requestClose} autoFocus>
          閉じる
        </button>
      </div>
      {children}
    </dialog>
  );
}
```

この例では、親コンポーネントが開閉状態を管理する。実装時は次も確認する。

- `onClose`が同じ状態を再設定して無限更新を起こさない
- Reactの`onCancel`とネイティブdialogの終了状態を同期する
- トリガーが削除された場合は、次の論理的なフォーカス先を選ぶ
- 複数のDrawerで`id`が重複しない
- ルート変更時に閉じる場合は、新しい主要コンテンツへのフォーカス移動を検討する

## よくある失敗

### 見た目だけモーダルにする

背景を暗くしても、Tabキーやスクリーンリーダーが背景へ移動できるならモーダルではない。`showModal()`または同等のフォーカス制御とinert化を行う。

### 非モーダルなのにフォーカストラップする

背景を操作できると説明しながら、Tabキーだけ外へ出られない状態は一貫しない。背景を操作できるなら通常のTab順序を保つ。

### `aria-modal="true"`だけを付ける

ARIAは背景を操作不能にしない。実際に背景をinertにし、フォーカスを内部に制限し、見た目でもモーダルだと分かる場合だけ使う。

### 閉じたパネルへフォーカスできる

画面外へ移動しただけのリンクやフォームはTab順序に残る。最終的な閉じた状態では`hidden`、`dialog.close()`、または適切なinert管理で操作対象から外す。

### フォーカスを戻す前に隠す

内側の要素へフォーカスがある状態でパネルを非表示にすると、フォーカスが`body`へ移ることがある。閉じる処理では復帰先を決め、状態変更と一緒に管理する。

### 閉じるボタンがアイコンだけ

アイコンだけを使う場合はアクセシブルネームが必要。可能なら「閉じる」という見えるテキストを用意し、音声操作でも指定しやすくする。

## テストのチェックリスト

### パターンの選択

- [ ] 背景を操作できるかどうかが仕様として決まっている
- [ ] 常設・非モーダル・モーダルのセマンティクスが動作と一致する
- [ ] 存在しない`role="drawer"`を使っていない
- [ ] `aria-modal="true"`だけでモーダル化していない

### キーボードとフォーカス

- [ ] 開くボタンをEnterキーとSpaceキーで操作できる
- [ ] 非モーダル型では開いた直後もフォーカスがボタンに残る
- [ ] 非モーダル型ではTabキーで背景へ移動できる
- [ ] モーダル型では開いたときにフォーカスが内部へ移る
- [ ] モーダル型ではTabキーが内部に留まる
- [ ] Escapeキーと見える閉じるボタンの両方で閉じられる
- [ ] 閉じた後、開いたボタンまたは論理的な次の場所へ戻る
- [ ] 閉じたドロワーの要素へフォーカスできない
- [ ] フォーカス中の要素がパネルや固定ヘッダーに隠れない

### 表示と入力方法

- [ ] 200%・400%拡大でも内容と閉じる操作を利用できる
- [ ] 内容が画面高を超えてもスクロールできる
- [ ] 強制カラーモードでも境界とフォーカスが見える
- [ ] 動きを減らす設定でスライドを抑制できる
- [ ] スワイプやドラッグを使わなくてもボタンで開閉できる
- [ ] タッチ、マウス、キーボード、音声操作で開閉できる
- [ ] 背景クリックが唯一の閉じる方法になっていない

### 支援技術

- [ ] 開くボタンの名前、役割、状態が伝わる
- [ ] ドロワーの名前と役割が伝わる
- [ ] モーダル型では背景へ読み上げ位置を移動できない
- [ ] 見出し、フォーム、ナビゲーションなど内部構造を辿れる
- [ ] 開閉を繰り返しても読み上げ位置を見失わない

## 関連して読むとよいテーマ

- [アクセシブルなディスクロージャーの実装 — 一部だけを開閉する](./accessible-disclosure.md)
- [アクセシブルなハンバーガーメニューの実装 — ナビゲーションを開いたときのフォーカス](./accessible-hamburger-menu.md)
- [アクセシブルなモーダルダイアログの実装](./accessible-modal-dialog.md)
- [aria-expanded / aria-hidden / aria-controls の活用](./aria-expanded-hidden-controls.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [prefers-reduced-motion — モーション設定への対応](./prefers-reduced-motion.md)

## まとめ

- Drawerは見た目の呼び名であり、専用のARIAロールはない
- 常設型は`aside`や`nav`、非モーダル型はDisclosure、モーダル型はDialogとして実装する
- 非モーダル型ではフォーカスを開閉ボタンに残し、背景への通常のTab移動を維持する
- モーダル型ではフォーカスを内部へ移し、背景をinertにして、Tab移動を内部へ制限する
- モーダルな`dialog`は`showModal()`で開き、Escape、見える閉じるボタン、フォーカス復帰を用意する
- 閉じたパネルを画面外へ移動するだけにせず、操作と読み上げの対象から外す
- スワイプやドラッグは補助操作とし、クリック・タップできるボタンでも開閉できるようにする
- レスポンシブで種類を変える場合は、見た目だけでなくセマンティクス、フォーカス、inert状態も同期する
