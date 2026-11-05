# アクセシブルなツールバーの実装 — 関連する操作をまとめる

## ツールバーとは

ツールバー（Toolbar）は、同じ対象に作用する関連コントロールを一つの複合ウィジェットとしてまとめるUI。文章の書式設定、画像編集、地図操作、表の行操作などに使われる。

- Tabキーではツールバー全体へ一度だけ入る
- 横向きでは左右矢印キーで内部を移動する
- 縦向きでは上下矢印キーで内部を移動する
- EnterまたはSpaceキーでボタンを実行する
- トグルボタンの押下状態とキーボードフォーカスを区別する
- ツールバー全体の目的を支援技術へ伝える

すべてのボタン群をツールバーにする必要はない。APGは、三つ以上のコントロールをまとめ、Tab停止位置を減らす効果がある場合に`toolbar`を使うよう案内している。

> 参照:
> - [W3C WAI-ARIA APG — Toolbar Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/toolbar/)
> - [W3C WAI-ARIA APG — Developing a Keyboard Interface](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

## ツールバーが向く場面

### 同じ対象へ複数の操作を行う

```text
テキスト書式
[太字] [斜体] [下線] | [左揃え] [中央揃え] [右揃え]
```

各操作が独立したボタンでも、同じ編集対象へ作用し、繰り返し使うならツールバーとしてまとめられる。

### 単純なボタン群

「保存」「キャンセル」の二つだけなら、通常のボタン群で十分。

```html
<div class="form-actions">
  <button type="submit">保存</button>
  <button type="button">キャンセル</button>
</div>
```

通常のボタン群では、各ボタンがTab順序へ入る。矢印キーによるフォーカス管理は追加しない。

### 名前のあるグループ

関連を伝えたいが、各項目をTabキーで操作させたい場合は`role="group"`も検討する。

```html
<div role="group" aria-label="表示方法">
  <button type="button">一覧表示</button>
  <button type="button">グリッド表示</button>
</div>
```

`group`はツールバーの矢印キー操作を意味しない。見た目が似ていても、実装するキーボード規則とロールを一致させる。

## 似たUIとの違い

| UI | 主な目的 | 内部操作 |
|---|---|---|
| Toolbar | 関連するコマンドや設定をまとめる | 主軸の矢印キーでフォーカス移動 |
| Button Group | 少数の操作を並べる | 各ボタンへTab移動 |
| Menu | 一時的に表示するコマンド一覧 | 上下矢印、Escapeで閉じる |
| Tab List | 表示するパネルを切り替える | 矢印キーでタブ移動 |
| Radio Group | 一つの値を選ぶ | 矢印キーで選択変更 |
| Navigation | 別ページや位置へ移動するリンク群 | 通常はTab移動 |

ページ移動が目的ならリンクとナビゲーションを使う。「見た目が横一列」という理由だけでリンク群へ`role="toolbar"`を付けない。

## 基本のHTML

```html
<div
  class="toolbar"
  role="toolbar"
  aria-label="テキスト書式"
  aria-orientation="horizontal"
>
  <button
    type="button"
    data-toolbar-item
    tabindex="0"
    aria-pressed="false"
  >
    太字
  </button>

  <button
    type="button"
    data-toolbar-item
    tabindex="-1"
    aria-pressed="false"
  >
    斜体
  </button>

  <button
    type="button"
    data-toolbar-item
    tabindex="-1"
    aria-pressed="false"
  >
    下線
  </button>

  <span
    class="toolbar__separator"
    role="separator"
    aria-orientation="vertical"
  ></span>

  <button
    type="button"
    data-toolbar-item
    tabindex="-1"
    aria-haspopup="menu"
    aria-expanded="false"
  >
    文字色
  </button>
</div>
```

### 必要なロールと属性

| 要素 | 属性 | 役割 |
|---|---|---|
| コンテナ | `role="toolbar"` | 関連コントロールのツールバーとして伝える |
| コンテナ | `aria-label` / `aria-labelledby` | ツールバーの目的を伝える |
| コンテナ | `aria-orientation` | 縦向きの場合に方向を伝える |
| 現在の項目 | `tabindex="0"` | Tabキーでツールバーへ入る位置にする |
| その他の項目 | `tabindex="-1"` | 矢印キーと`focus()`で移動できるようにする |
| トグルボタン | `aria-pressed` | オン・オフ状態を伝える |

横向きは`aria-orientation="horizontal"`が既定値なので省略できる。実装例では、DOMとキーボード方針を読み取りやすくするため明示している。

## ツールバーへ名前を付ける

### 見える見出しがある場合

```html
<h2 id="image-tools-heading">画像編集</h2>
<div role="toolbar" aria-labelledby="image-tools-heading">
  <!-- コントロール -->
</div>
```

### 見える見出しがない場合

```html
<div role="toolbar" aria-label="画像編集">
  <!-- コントロール -->
</div>
```

同じページに複数のツールバーがある場合は、「テキスト書式」「画像配置」「表の行操作」のように区別できる名前を付ける。

名前へ「ツールバー」を含めると、支援技術がロール名も読み上げて重複する場合がある。「テキスト書式」のように目的を名前にする。

## キーボード操作

### 横向き

| キー | 動作 |
|---|---|
| Tab | ツールバーへ入る、または次のコンポーネントへ出る |
| Shift + Tab | 前のコンポーネントへ出る |
| → | 次の有効なコントロールへ移動する |
| ← | 前の有効なコントロールへ移動する |
| Home | 最初の有効なコントロールへ移動する（任意） |
| End | 最後の有効なコントロールへ移動する（任意） |
| Enter / Space | フォーカス中のボタンを実行する |

### 縦向き

| キー | 動作 |
|---|---|
| ↓ | 次の有効なコントロールへ移動する |
| ↑ | 前の有効なコントロールへ移動する |
| Home | 最初の有効なコントロールへ移動する（任意） |
| End | 最後の有効なコントロールへ移動する（任意） |

縦向きでは`aria-orientation="vertical"`を付ける。CSSで並びだけを変え、ARIAとキー操作を横向きのままにしない。

端から反対側へ循環するかは任意。循環させる場合も、製品内で一貫させる。

## ロービング`tabindex`を実装する

次のクラスは、`data-toolbar-item`を持つ項目だけを矢印キーの移動対象にする。区切り線や説明テキストは対象に含めない。

```js
class AccessibleToolbar {
  constructor(toolbar, {
    wrap = true,
    includeAriaDisabled = false,
  } = {}) {
    this.toolbar = toolbar;
    this.wrap = wrap;
    this.includeAriaDisabled = includeAriaDisabled;
    this.orientation =
      toolbar.getAttribute('aria-orientation') ?? 'horizontal';
    this.direction = getComputedStyle(toolbar).direction;

    this.refresh();
    this.bindEvents();
  }

  get items() {
    return [
      ...this.toolbar.querySelectorAll('[data-toolbar-item]'),
    ];
  }

  isUnavailable(item) {
    return item.matches(':disabled') ||
      (
        !this.includeAriaDisabled &&
        item.getAttribute('aria-disabled') === 'true'
      ) ||
      item.hidden;
  }

  get availableItems() {
    return this.items.filter((item) => !this.isUnavailable(item));
  }

  refresh(preferredItem = null) {
    const items = this.items;
    const available = this.availableItems;
    const current = preferredItem && available.includes(preferredItem)
      ? preferredItem
      : available.find((item) => item.tabIndex === 0) ?? available[0] ?? null;

    items.forEach((item) => {
      item.tabIndex = item === current ? 0 : -1;
    });

    this.currentItem = current;
  }

  setCurrent(item, { focus = true } = {}) {
    if (!item || this.isUnavailable(item)) return;

    this.items.forEach((candidate) => {
      candidate.tabIndex = candidate === item ? 0 : -1;
    });
    this.currentItem = item;

    if (focus) item.focus();
  }

  move(offset) {
    const items = this.availableItems;
    if (items.length === 0) return;

    const currentIndex = items.indexOf(this.currentItem);
    const startIndex = currentIndex === -1 ? 0 : currentIndex;
    let nextIndex = startIndex + offset;

    if (this.wrap) {
      nextIndex = (nextIndex + items.length) % items.length;
    } else {
      nextIndex = Math.min(Math.max(nextIndex, 0), items.length - 1);
    }

    this.setCurrent(items[nextIndex]);
  }

  moveToEdge(edge) {
    const items = this.availableItems;
    const item = edge === 'start' ? items[0] : items.at(-1);
    this.setCurrent(item);
  }

  handleKeyDown(event) {
    if (event.ctrlKey || event.metaKey || event.altKey) return;

    const item = event.target.closest('[data-toolbar-item]');
    if (!item || !this.toolbar.contains(item)) return;

    const nextKey = this.orientation === 'vertical'
      ? 'ArrowDown'
      : this.direction === 'rtl' ? 'ArrowLeft' : 'ArrowRight';
    const previousKey = this.orientation === 'vertical'
      ? 'ArrowUp'
      : this.direction === 'rtl' ? 'ArrowRight' : 'ArrowLeft';

    if (event.key === nextKey) {
      event.preventDefault();
      this.move(1);
      return;
    }

    if (event.key === previousKey) {
      event.preventDefault();
      this.move(-1);
      return;
    }

    if (event.key === 'Home') {
      event.preventDefault();
      this.moveToEdge('start');
    }

    if (event.key === 'End') {
      event.preventDefault();
      this.moveToEdge('end');
    }
  }

  bindEvents() {
    this.toolbar.addEventListener('keydown', (event) => {
      this.handleKeyDown(event);
    });

    this.toolbar.addEventListener('focusin', (event) => {
      const item = event.target.closest('[data-toolbar-item]');
      if (item && this.toolbar.contains(item)) {
        this.setCurrent(item, { focus: false });
      }
    });

    this.toolbar.addEventListener('click', (event) => {
      const item = event.target.closest('[data-toolbar-item]');
      if (item && this.toolbar.contains(item)) {
        this.setCurrent(item, { focus: false });
      }
    });
  }
}

document.querySelectorAll('[role="toolbar"]').forEach((toolbar) => {
  new AccessibleToolbar(toolbar, {
    includeAriaDisabled:
      toolbar.dataset.toolbarIncludeDisabled === 'true',
  });
});
```

### 初期化時の規則

- 有効な項目のうち一つだけを`tabindex="0"`にする
- 他のすべての項目を`tabindex="-1"`にする
- 初回は通常、最初の有効な項目をTab停止位置にする
- 一度操作した後は、最後にフォーカスした項目から再開できる

HTMLに誤って複数の`tabindex="0"`があっても、初期化時に一つへ整理する。

### 動的な追加・削除

項目を追加、削除、無効化した後は`refresh()`を呼ぶ。現在項目を削除する場合は、削除前に論理的な次の項目を決める。

```js
const nextItem = toolbarController.availableItems.find(
  (item) => item !== itemToRemove
);

itemToRemove.remove();
toolbarController.refresh(nextItem);
```

フォーカス中の要素を削除すると、フォーカスが`body`へ失われる場合がある。必要なら`refresh()`後に新しい現在項目へ`focus()`する。

## ボタンの実行はネイティブへ任せる

`button`はEnterとSpaceで実行できる。ツールバーのJavaScriptでクリック処理を再実装しない。

```html
<button type="button" data-toolbar-item tabindex="0">
  コピー
</button>
```

```js
copyButton.addEventListener('click', () => {
  copySelection();
});
```

矢印キー処理はフォーカス移動だけを担当し、各コントロールの実行処理と分離する。

`div role="button"`を使うと、Enter、Space、無効状態、フォーム動作をすべて自前で実装する必要がある。通常はネイティブの`button`を使う。

## トグルボタンの状態

太字やミュートのようなオン・オフ操作には`aria-pressed`を使う。

```html
<button
  id="bold-button"
  type="button"
  data-toolbar-item
  tabindex="0"
  aria-pressed="false"
>
  太字
</button>
```

```js
const boldButton = document.getElementById('bold-button');

boldButton.addEventListener('click', () => {
  const isPressed = boldButton.getAttribute('aria-pressed') === 'true';
  boldButton.setAttribute('aria-pressed', String(!isPressed));
});
```

`aria-pressed`を使うトグルボタンでは、状態が変わっても名前を「太字」のまま保つ。「太字を解除」のようにラベルも変更すると、ボタン自体が別の操作へ変わったように伝わる場合がある。

```text
太字、切り替えボタン、押されている
```

複数のボタンから一つだけ選ぶ「左揃え・中央揃え・右揃え」は、選択時に他の`aria-pressed`を`false`へ更新する。フォーカス位置と選択状態は別々に管理する。

> 参照: [W3C WAI-ARIA APG — Button Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/button/)

## アイコンだけのボタン

```html
<button
  type="button"
  data-toolbar-item
  tabindex="-1"
  aria-label="元に戻す"
>
  <svg aria-hidden="true" focusable="false" viewBox="0 0 24 24">
    <!-- アイコン -->
  </svg>
</button>
```

- 各ボタンに操作を区別できる名前を付ける
- SVGアイコンは重複して読み上げないようにする
- アイコンの意味を知っていることを前提にせず、見えるツールチップも検討する
- `title`属性だけを唯一の説明にしない
- 音声操作で指定できる名前と画面上の文言を一致させる

同じ矢印アイコンでも、「元に戻す」「左へ移動」「前のページ」のように機能は異なる。形ではなく実行する操作を名前にする。

## 区切り線

```html
<span
  class="toolbar__separator"
  role="separator"
  aria-orientation="vertical"
></span>
```

見た目が縦線なら`aria-orientation="vertical"`を付ける。区切り線は操作項目ではないため、`data-toolbar-item`や`tabindex`を付けない。

単なる余白だけでグループの違いが十分伝わるなら、装飾要素を`aria-hidden="true"`にする方法もある。支援技術へ区切りを伝える必要があるか、グループの意味から判断する。

## 無効なコントロール

### 通常はスキップする

```html
<button type="button" data-toolbar-item tabindex="-1" disabled>
  やり直す
</button>
```

ネイティブの`disabled`ボタンはフォーカスを受け取らず、実行もされない。前のJavaScript例は矢印移動でもスキップする。

### 存在を発見できるようにする

機能の存在を伝えることが重要な場合は、`aria-disabled="true"`でフォーカス可能なままにする設計もある。ただし、クリックとキーボードの実行を自分で防ぐ必要がある。

```html
<div
  role="toolbar"
  aria-label="編集"
  data-toolbar-include-disabled="true"
>
  <!-- その他の操作 -->
  <button
    type="button"
    data-toolbar-item
    tabindex="-1"
    aria-disabled="true"
    aria-describedby="redo-disabled-reason"
  >
    やり直す
  </button>
  <span id="redo-disabled-reason" class="visually-hidden">
    やり直せる操作がありません。
  </span>
</div>
```

```js
redoButton.addEventListener('click', (event) => {
  if (redoButton.getAttribute('aria-disabled') === 'true') {
    event.preventDefault();
    return;
  }

  redoLastAction();
});
```

この記事の`AccessibleToolbar`は、既定では`aria-disabled="true"`もスキップする。発見可能性を優先するツールバーへ`data-toolbar-include-disabled="true"`を付けると、`aria-disabled`の項目を矢印移動に含める。どちらの方針でも、製品内で一貫させる。

## 矢印キーを使う入力を含めない

横向きツールバーでは左右矢印を移動に使うため、テキスト入力、横向きスライダーなどと競合する。縦向きでは上下矢印を使うスピンボタンやリストボックスと競合する。

```html
<!-- 避ける：左右矢印が文字カーソルとツールバー移動で競合する -->
<div role="toolbar" aria-label="検索と編集">
  <button type="button">元に戻す</button>
  <input type="search" aria-label="文書内検索">
</div>
```

まず、矢印キーを使う入力をツールバーの外へ分ける。避けられない場合、APGは競合するコントロールを一つだけ、最後に配置する方法を示している。どのキーをツールバー移動へ使い、どのキーを入力操作へ残すかを明文化してテストする。

入力の標準的な文字編集キーを奪わない。`event.target`が入力要素だからという理由で左右矢印を常にツールバー移動へ使うと、文字カーソルを動かせなくなる。

## 縦向きツールバー

```html
<div
  class="toolbar toolbar--vertical"
  role="toolbar"
  aria-label="地図操作"
  aria-orientation="vertical"
>
  <button type="button" data-toolbar-item tabindex="0">拡大</button>
  <button type="button" data-toolbar-item tabindex="-1">縮小</button>
  <button type="button" data-toolbar-item tabindex="-1">現在地</button>
</div>
```

前のJavaScript例は`aria-orientation`を読み取り、上下矢印へ切り替える。左右矢印も同じ動作にすることは任意だが、内部コントロールの操作へ必要なら予約する。

## CSSで状態を区別する

```css
.toolbar {
  display: flex;
  gap: 0.25rem;
  align-items: center;
  inline-size: fit-content;
  max-inline-size: 100%;
  padding: 0.25rem;
  border: 0.125rem solid CanvasText;
  border-radius: 0.375rem;
  overflow-x: auto;
}

.toolbar--vertical {
  flex-direction: column;
  align-items: stretch;
  overflow-x: visible;
}

.toolbar button {
  min-inline-size: 2.75rem;
  min-block-size: 2.75rem;
  border: 0.125rem solid transparent;
  color: ButtonText;
  background: ButtonFace;
}

.toolbar button[aria-pressed="true"] {
  border-color: CanvasText;
  font-weight: 700;
  text-decoration: underline;
}

.toolbar button:focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.125rem;
}

.toolbar button:disabled,
.toolbar button[aria-disabled="true"] {
  opacity: 0.55;
}

.toolbar__separator {
  align-self: stretch;
  inline-size: 0;
  margin-inline: 0.25rem;
  border-inline-start: 0.125rem solid CanvasText;
}

.toolbar--vertical .toolbar__separator {
  inline-size: auto;
  block-size: 0;
  margin-block: 0.25rem;
  border-inline-start: 0;
  border-block-start: 0.125rem solid CanvasText;
}

@media (forced-colors: active) {
  .toolbar button[aria-pressed="true"] {
    border-color: Highlight;
  }
}
```

フォーカスと押下状態を同じ見た目だけで表さない。フォーカスはアウトライン、押下状態は境界や文字装飾など、同時に存在しても区別できる表現にする。

狭い画面でボタンを複数行へ折り返すと、左右矢印でどこへ移動するか予測しにくくなる。この例は横スクロールで一列を保つ。折り返す場合は、DOM順序と視覚順序を一致させ、行をまたぐ移動も検証する。

## Reactで実装する

```jsx
import { useEffect, useRef, useState } from 'react';

function Toolbar({
  label,
  actions,
  orientation = 'horizontal',
  direction = 'ltr',
}) {
  const firstAvailableId = actions.find((action) => !action.disabled)?.id;
  const [currentId, setCurrentId] = useState(firstAvailableId);
  const itemRefs = useRef(new Map());

  useEffect(() => {
    const current = actions.find((action) => action.id === currentId);
    if (!current || current.disabled) {
      setCurrentId(actions.find((action) => !action.disabled)?.id);
    }
  }, [actions, currentId]);

  const move = (offset) => {
    const available = actions.filter((action) => !action.disabled);
    if (available.length === 0) return;

    const currentIndex = available.findIndex(
      (action) => action.id === currentId
    );
    const startIndex = currentIndex === -1 ? 0 : currentIndex;
    const nextIndex = (
      startIndex + offset + available.length
    ) % available.length;
    const nextId = available[nextIndex].id;

    setCurrentId(nextId);
    itemRefs.current.get(nextId)?.focus();
  };

  const moveToEdge = (edge) => {
    const available = actions.filter((action) => !action.disabled);
    const action = edge === 'start' ? available[0] : available.at(-1);
    if (!action) return;

    setCurrentId(action.id);
    itemRefs.current.get(action.id)?.focus();
  };

  const handleKeyDown = (event) => {
    if (event.ctrlKey || event.metaKey || event.altKey) return;

    const nextKey = orientation === 'vertical'
      ? 'ArrowDown'
      : direction === 'rtl' ? 'ArrowLeft' : 'ArrowRight';
    const previousKey = orientation === 'vertical'
      ? 'ArrowUp'
      : direction === 'rtl' ? 'ArrowRight' : 'ArrowLeft';

    if (event.key === nextKey) {
      event.preventDefault();
      move(1);
    } else if (event.key === previousKey) {
      event.preventDefault();
      move(-1);
    } else if (event.key === 'Home') {
      event.preventDefault();
      moveToEdge('start');
    } else if (event.key === 'End') {
      event.preventDefault();
      moveToEdge('end');
    }
  };

  return (
    <div
      className={`toolbar toolbar--${orientation}`}
      role="toolbar"
      aria-label={label}
      aria-orientation={orientation}
      dir={direction}
      onKeyDown={handleKeyDown}
    >
      {actions.map((action) => (
        <button
          key={action.id}
          ref={(element) => {
            if (element) itemRefs.current.set(action.id, element);
            else itemRefs.current.delete(action.id);
          }}
          type="button"
          tabIndex={action.id === currentId ? 0 : -1}
          disabled={action.disabled}
          aria-pressed={action.pressed}
          onFocus={() => setCurrentId(action.id)}
          onClick={action.onActivate}
        >
          {action.label}
        </button>
      ))}
    </div>
  );
}
```

`actions`の`id`は配列位置ではなく、操作を識別する安定した値にする。項目の並び替えや削除後も、フォーカス対象とReactの要素を正しく対応させられる。

`aria-pressed`が不要な通常ボタンでは`undefined`を渡し、属性自体を出力しない。すべてのボタンを誤ってトグルボタンとして公開しない。

## フォーカスを失わせない

ツールバーの操作で現在ボタンを削除、非表示、無効化する場合は、次にフォーカスする場所を先に決める。

- 右隣の有効な操作
- 右隣がなければ左隣
- ツールバーが空になれば、次の論理的なコンポーネント
- ダイアログを開く操作なら、ダイアログ内の適切な要素

操作後も同じ画面文脈なら、通常は実行したボタンへフォーカスを残す。フォーカスを毎回編集領域へ戻すと、連続してツールバーを操作しにくい。

エディターからツールバーへ素早く戻る必要がある場合は、通常のTab順序に加えてショートカットを提供できる。ただしショートカットを唯一の移動手段にせず、画面上のヘルプで発見できるようにする。

## 右から左へ書く言語と表示順

矢印キーは、利用者が見ている項目の並びと一致するようにする。CSSの`direction`や`order`だけで見た目を反転すると、DOM順序とフォーカス移動が食い違うことがある。

- 対象言語の書字方向で視覚順序を確認する
- 左右矢印を押したとき、隣接する見える項目へ移動する
- DOM順序、読み上げ順序、フォーカス順序を一致させる
- CSSの`order`で意味のある順序を入れ替えない

実装する言語とブラウザ・支援技術の組み合わせでテストし、製品内で一貫した規則にする。

## よくある失敗

### `role="toolbar"`を付けただけ

ARIAは矢印キー操作を追加しない。ロービング`tabindex`とフォーカス移動をJavaScriptで実装する。

### すべてのボタンがTab順序へ入る

ツールバーとして実装するなら、通常は一つだけを`tabindex="0"`にし、内部は矢印キーで移動する。

### ツールバーに名前がない

複数のツールバーを区別できるよう、目的を表す名前を付ける。

### 縦向きなのに左右矢印を使う

見た目、`aria-orientation`、主軸の矢印キーを一致させる。

### フォーカスと押下状態を混同する

矢印キーでフォーカスを動かしただけで、太字や配置の状態を変えない。実行はEnter、Space、クリックで行う。

### トグル時に名前と状態を同時に変える

`aria-pressed`を使うなら「太字」という名前は維持し、状態を`true`と`false`で伝える。

### 入力欄の矢印キーを奪う

テキスト入力、スライダー、スピンボタンなどの標準操作を妨げない。競合する入力をツールバー外へ分ける。

### 無効な項目へフォーカスを残す

現在項目を無効にしたら、別の有効な項目へ`tabindex="0"`を移す。フォーカス中なら、論理的な移動先も決める。

### アイコンだけで名前がない

各操作に固有のアクセシブルネームを付ける。色や形だけで機能を区別しない。

### レスポンシブ表示で順序が変わる

折り返しやCSSによる並び替え後も、矢印キーの移動先が見た目と一致するか確認する。

## テストのチェックリスト

### パターンの選択

- [ ] 三つ以上の関連コントロールをまとめている
- [ ] Tab停止位置を減らすことが操作効率を改善する
- [ ] 少数の操作なら通常のボタン群も検討した
- [ ] ページ移動のリンク群やタブリストへ誤用していない
- [ ] 矢印キーを使う入力をツールバーから分離した

### 構造と名前

- [ ] コンテナに`role="toolbar"`がある
- [ ] ツールバーに目的を表す名前がある
- [ ] 複数のツールバーを名前で区別できる
- [ ] 縦向きに`aria-orientation="vertical"`がある
- [ ] 操作項目にネイティブの`button`などを使っている
- [ ] 区切り線がTab順序や矢印移動へ入らない

### フォーカス管理

- [ ] 有効な項目のうち一つだけが`tabindex="0"`である
- [ ] Tab / Shift + Tabでツールバーへ出入りできる
- [ ] 横向きは左右、縦向きは上下矢印で移動できる
- [ ] HomeとEndの実装が説明と一致する
- [ ] 端で循環するかどうかが製品内で一貫している
- [ ] 再入場時のフォーカス位置が予測できる
- [ ] 項目の追加・削除・無効化後もフォーカスを失わない

### 操作と状態

- [ ] EnterとSpaceでボタンを実行できる
- [ ] 矢印移動だけでは操作を実行しない
- [ ] トグルボタンに`aria-pressed`がある
- [ ] 通常ボタンへ不要な`aria-pressed`を付けていない
- [ ] フォーカス位置と押下・選択状態を区別している
- [ ] メニューボタンの開閉状態を`aria-expanded`で同期している
- [ ] 無効状態と実際に実行できるかが一致する

### 表示と入力方法

- [ ] フォーカス表示が明確に見える
- [ ] フォーカスと押下状態を同時に区別できる
- [ ] 状態を色だけに依存していない
- [ ] アイコンボタンの名前と見える説明を確認できる
- [ ] 200%・400%拡大でも操作が重ならない
- [ ] 狭い画面で視覚順序と矢印移動が一致する
- [ ] 強制カラーモードでも境界、状態、フォーカスが見える
- [ ] キーボード、マウス、タッチ、音声操作で確認した
- [ ] スクリーンリーダーで名前、ロール、状態、位置を確認した

## 関連して読むとよいテーマ

- [ロービングタブインデックスパターン](./roving-tabindex.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [キーボードショートカットの設計原則とWCAG 2.1要件](./keyboard-shortcut-design.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [アクセシブルなラジオグループの実装 — 矢印キーで一つを選ぶ](./accessible-radio-group.md)
- [アクセシブルなスピンボタンの実装 — 数値の増減](./accessible-spinbutton.md)
- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)

## まとめ

- 三つ以上の関連コントロールを効率よく操作する場合に`toolbar`を使う
- 単純なボタン群は無理にツールバーへせず、通常のTab操作を保つ
- ツールバーへ目的を表す名前を付ける
- Tabキーでは全体へ一度だけ入り、主軸の矢印キーで内部を移動する
- ロービング`tabindex`では有効な項目の一つだけを`tabindex="0"`にする
- フォーカス位置とトグルボタンの押下状態を分けて管理する
- ボタンのEnter・Space操作はネイティブHTMLへ任せる
- 矢印キーを使う入力を含めず、標準の編集操作を奪わない
- 動的な追加・削除・無効化後も、フォーカス可能な現在項目を維持する
- 見た目、DOM順序、ARIAの方向、矢印キーの移動を一致させる
