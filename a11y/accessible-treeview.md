# アクセシブルなツリービューの実装 — 階層の展開と矢印キー

## ツリービューとは

ツリービュー（Tree View）は、親子関係を持つ項目を階層的に表示する複合ウィジェット。ファイルとフォルダー、組織階層、アウトライン、レイヤーパネルなどに使われる。

- Tabキーではツリー全体へ一度だけ入る
- 上下矢印キーで現在見えている項目を移動する
- 右矢印キーで親項目を開く、または最初の子へ移動する
- 左矢印キーで親項目を閉じる、または親へ戻る
- HomeとEndで見えている先頭・末尾へ移動する
- 開閉状態、階層、選択状態を支援技術へ伝える

ツリービューはキーボード規則と状態管理が複雑。階層が見た目だけに必要な場合や、一般的なサイトナビゲーションでは、通常のリストとディスクロージャーを優先する。

> 参照:
> - [W3C WAI-ARIA APG — Tree View Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/treeview/)
> - [W3C WAI-ARIA APG — File Directory Treeview Example](https://www.w3.org/WAI/ARIA/apg/patterns/treeview/examples/treeview-1a/)

## ツリービューが向く場面

### 階層を頻繁に探索する

```text
プロジェクト
├─ デザイン
│  ├─ ロゴ.svg
│  └─ バナー.png
└─ ドキュメント
   ├─ 要件定義.md
   └─ README.md
```

親子関係を保ったまま、多数の項目をキーボードで素早く探索する必要がある場合に向いている。

### 一般的なサイトナビゲーション

Webサイトのカテゴリを開閉するだけなら、ディスクロージャーと通常のリンクを使う方が単純で理解しやすい。

```html
<nav aria-label="カテゴリ">
  <button
    type="button"
    aria-expanded="false"
    aria-controls="products-links"
  >
    製品
  </button>
  <ul id="products-links" hidden>
    <li><a href="/products/a">製品A</a></li>
    <li><a href="/products/b">製品B</a></li>
  </ul>
</nav>
```

ツリービューにすると、利用者は矢印キーによるデスクトップUI型の操作を期待する。見た目が字下げされているだけのナビゲーションへ`role="tree"`を付けない。

## 似たUIとの違い

| UI | 主な目的 | 特徴 |
|---|---|---|
| Tree View | 階層を探索し、項目を選択・実行する | 親子移動と開閉がある |
| Disclosure | 一部のコンテンツを開閉する | ボタンごとに独立して操作する |
| Listbox | 一階層の候補から選択する | 親子関係を持たない |
| Treegrid | 階層と複数列の表データを扱う | 行・列の移動が必要 |
| Navigation | リンクでページや位置を移動する | 通常はTabキーで移動する |

項目ごとに複数列の値や編集ボタンが必要なら、Tree ViewではなくTreegrid、通常の表、階層リストと詳細パネルなどを検討する。

## 用語を整理する

| 用語 | 意味 |
|---|---|
| Root node | 親を持たない最上位の項目 |
| Parent node | 一つ以上の子を持つ項目 |
| Child node | 親を持つ項目 |
| End node | 子を持たない末端の項目 |
| Open node | 展開され、子が見えている親項目 |
| Closed node | 折りたたまれ、子が隠れている親項目 |

この記事では、各`treeitem`を「項目」、子を持つ項目を「親項目」と呼ぶ。

## 基本のHTML

```html
<h2 id="project-tree-label">プロジェクトファイル</h2>

<ul
  id="project-tree"
  class="tree"
  role="tree"
  aria-labelledby="project-tree-label"
>
  <li
    id="tree-design"
    role="treeitem"
    aria-labelledby="tree-design-label"
    aria-expanded="true"
    aria-selected="false"
    tabindex="0"
    data-value="design"
  >
    <span id="tree-design-label" data-tree-label>デザイン</span>
    <ul role="group">
      <li
        id="tree-logo"
        role="treeitem"
        aria-labelledby="tree-logo-label"
        aria-selected="false"
        tabindex="-1"
        data-value="logo.svg"
      >
        <span id="tree-logo-label" data-tree-label>ロゴ.svg</span>
      </li>
      <li
        id="tree-banner"
        role="treeitem"
        aria-labelledby="tree-banner-label"
        aria-selected="false"
        tabindex="-1"
        data-value="banner.png"
      >
        <span id="tree-banner-label" data-tree-label>バナー.png</span>
      </li>
    </ul>
  </li>

  <li
    id="tree-documents"
    role="treeitem"
    aria-labelledby="tree-documents-label"
    aria-expanded="false"
    aria-selected="false"
    tabindex="-1"
    data-value="documents"
  >
    <span id="tree-documents-label" data-tree-label>ドキュメント</span>
    <ul role="group" hidden>
      <li
        id="tree-requirements"
        role="treeitem"
        aria-labelledby="tree-requirements-label"
        aria-selected="false"
        tabindex="-1"
        data-value="requirements.md"
      >
        <span id="tree-requirements-label" data-tree-label>
          要件定義.md
        </span>
      </li>
      <li
        id="tree-readme"
        role="treeitem"
        aria-labelledby="tree-readme-label"
        aria-selected="false"
        tabindex="-1"
        data-value="README.md"
      >
        <span id="tree-readme-label" data-tree-label>README.md</span>
      </li>
    </ul>
  </li>
</ul>

<p id="selected-file" role="status">項目は選択されていません。</p>
```

### 必要なロールと属性

| 要素 | 属性 | 役割 |
|---|---|---|
| ルート | `role="tree"` | ツリービュー全体として伝える |
| ルート | `aria-label` / `aria-labelledby` | ツリーの目的を伝える |
| 各項目 | `role="treeitem"` | ツリー内の項目として伝える |
| 子項目のコンテナ | `role="group"` | 親項目の子グループとして伝える |
| 親項目 | `aria-expanded` | 開閉状態を伝える |
| 選択可能な項目 | `aria-selected` | 選択状態を伝える |
| 現在の項目 | `tabindex="0"` | Tabキーでツリーへ入る位置にする |
| その他の項目 | `tabindex="-1"` | 矢印キーでフォーカスできるようにする |

子を持たない末端項目へ`aria-expanded="false"`を付けない。`aria-expanded`があると、閉じた親項目として伝わる。

`aria-labelledby`を各項目の見えるラベルへ向けると、子孫項目の文字列まで親項目の名前へ含まれることを避けられる。

## キーボード操作

| キー | 動作 |
|---|---|
| Tab | ツリーへ入る、または次のコンポーネントへ出る |
| Shift + Tab | 前のコンポーネントへ出る |
| ↓ | 開閉状態を変えず、次に見えている項目へ移動する |
| ↑ | 開閉状態を変えず、前に見えている項目へ移動する |
| → | 閉じた親を開く。開いていれば最初の子へ移動する |
| ← | 開いた親を閉じる。閉じているか末端なら親へ移動する |
| Home | 開閉せず、最初に見えている項目へ移動する |
| End | 開閉せず、最後に見えている項目へ移動する |
| Enter | 項目の既定動作を実行する |
| Space | フォーカス中の項目を選択する |
| 文字入力 | 入力文字で始まる次の見えている項目へ移動する |
| `*` | 同じ階層にある親項目をすべて開く（任意） |

上下矢印キーは、閉じた項目の子孫へ入らない。現在DOMに存在する全項目ではなく、現在見えている項目だけを移動対象にする。

## フォーカス、展開、選択を分ける

ツリービューでは、次の状態を別々に管理する。

| 状態 | 意味 | 主な表現 |
|---|---|---|
| DOMフォーカス | キーボード操作の現在位置 | `tabindex`と`focus()` |
| 展開状態 | 子項目が見えているか | `aria-expanded`と`hidden` |
| 選択状態 | 操作対象として選ばれているか | `aria-selected` |
| 現在ページ | ナビゲーション先の現在位置 | `aria-current="page"` |

この記事の例は、フォーカスを移動しても選択を変更しない。利用者はツリーを探索し、Spaceまたは末端項目のEnterで選択を確定する。

単一選択で、フォーカス移動による副作用が小さい場合は「selection follows focus」も選べる。ただし、項目を移動するたびに詳細表示や通信が発生するなら、フォーカスと選択を分ける。

複数選択では、フォーカスと選択を必ず分離する。

## JavaScript実装

```js
class AccessibleTree {
  constructor(tree) {
    this.tree = tree;
    this.typeahead = '';
    this.typeaheadTimer = null;

    this.syncGroups();
    this.refresh();
    this.bindEvents();
  }

  get items() {
    return [...this.tree.querySelectorAll('[role="treeitem"]')];
  }

  get visibleItems() {
    return this.items.filter(
      (item) => !item.closest('[role="group"][hidden]')
    );
  }

  directGroup(item) {
    return [...item.children].find(
      (child) => child.getAttribute('role') === 'group'
    ) ?? null;
  }

  directChildren(item) {
    const group = this.directGroup(item);
    if (!group) return [];

    return [...group.children].filter(
      (child) => child.getAttribute('role') === 'treeitem'
    );
  }

  parentItem(item) {
    const group = item.parentElement;
    if (group?.getAttribute('role') !== 'group') return null;

    const parent = group.parentElement;
    return parent?.getAttribute('role') === 'treeitem' ? parent : null;
  }

  labelElement(item) {
    return [...item.children].find(
      (child) => child.hasAttribute('data-tree-label')
    ) ?? null;
  }

  itemName(item) {
    return this.labelElement(item)?.textContent.trim() ?? '';
  }

  isParent(item) {
    return Boolean(this.directGroup(item));
  }

  isExpanded(item) {
    return item.getAttribute('aria-expanded') === 'true';
  }

  syncGroups() {
    this.items.forEach((item) => {
      const group = this.directGroup(item);
      if (!group) {
        item.removeAttribute('aria-expanded');
        return;
      }

      const expanded = item.getAttribute('aria-expanded') === 'true';
      item.setAttribute('aria-expanded', String(expanded));
      group.hidden = !expanded;
    });
  }

  refresh(preferredItem = null) {
    const visible = this.visibleItems;
    const selected = visible.find(
      (item) => item.getAttribute('aria-selected') === 'true'
    );
    const current = preferredItem && visible.includes(preferredItem)
      ? preferredItem
      : visible.find((item) => item.tabIndex === 0) ?? selected ?? visible[0];

    this.items.forEach((item) => {
      item.tabIndex = item === current ? 0 : -1;
    });
    this.currentItem = current ?? null;
  }

  setCurrent(item, { focus = true } = {}) {
    if (!item || !this.visibleItems.includes(item)) return;

    this.items.forEach((candidate) => {
      candidate.tabIndex = candidate === item ? 0 : -1;
    });
    this.currentItem = item;

    if (focus) {
      item.focus();
      item.scrollIntoView({ block: 'nearest' });
    }
  }

  setExpanded(item, expanded) {
    const group = this.directGroup(item);
    if (!group) return;

    item.setAttribute('aria-expanded', String(expanded));
    group.hidden = !expanded;
  }

  toggle(item) {
    if (!this.isParent(item)) return;
    this.setExpanded(item, !this.isExpanded(item));
  }

  moveVertical(offset) {
    const visible = this.visibleItems;
    const currentIndex = visible.indexOf(this.currentItem);
    if (currentIndex === -1) return;

    const nextIndex = Math.min(
      Math.max(currentIndex + offset, 0),
      visible.length - 1
    );
    this.setCurrent(visible[nextIndex]);
  }

  moveRight() {
    const item = this.currentItem;
    if (!item || !this.isParent(item)) return;

    if (!this.isExpanded(item)) {
      this.setExpanded(item, true);
      return;
    }

    this.setCurrent(this.directChildren(item)[0]);
  }

  moveLeft() {
    const item = this.currentItem;
    if (!item) return;

    if (this.isParent(item) && this.isExpanded(item)) {
      this.setExpanded(item, false);
      return;
    }

    this.setCurrent(this.parentItem(item));
  }

  moveToEdge(edge) {
    const visible = this.visibleItems;
    const item = edge === 'start' ? visible[0] : visible.at(-1);
    this.setCurrent(item);
  }

  expandSiblings(item) {
    const parent = this.parentItem(item);
    const siblings = parent
      ? this.directChildren(parent)
      : [...this.tree.children].filter(
          (child) => child.getAttribute('role') === 'treeitem'
        );

    siblings.forEach((sibling) => {
      if (this.isParent(sibling)) this.setExpanded(sibling, true);
    });
  }

  select(item) {
    this.items.forEach((candidate) => {
      candidate.setAttribute(
        'aria-selected',
        String(candidate === item)
      );
    });

    this.tree.dispatchEvent(
      new CustomEvent('treeselectionchange', {
        bubbles: true,
        detail: {
          id: item.id,
          value: item.dataset.value ?? '',
          label: this.itemName(item),
        },
      })
    );
  }

  activate(item) {
    if (this.isParent(item)) {
      this.toggle(item);
    } else {
      this.select(item);
    }
  }

  handleTypeahead(key) {
    window.clearTimeout(this.typeaheadTimer);

    const nextBuffer = this.typeahead + key.toLocaleLowerCase();
    const repeatedCharacter = [...nextBuffer].every(
      (character) => character === nextBuffer[0]
    );
    this.typeahead = repeatedCharacter
      ? key.toLocaleLowerCase()
      : nextBuffer;

    const visible = this.visibleItems;
    const currentIndex = visible.indexOf(this.currentItem);
    const ordered = [
      ...visible.slice(currentIndex + 1),
      ...visible.slice(0, currentIndex + 1),
    ];
    const match = ordered.find((item) =>
      this.itemName(item).toLocaleLowerCase().startsWith(this.typeahead)
    );

    if (match) this.setCurrent(match);

    this.typeaheadTimer = window.setTimeout(() => {
      this.typeahead = '';
    }, 700);
  }

  handleKeyDown(event) {
    if (event.ctrlKey || event.metaKey || event.altKey) return;

    switch (event.key) {
      case 'ArrowDown':
        event.preventDefault();
        this.moveVertical(1);
        return;
      case 'ArrowUp':
        event.preventDefault();
        this.moveVertical(-1);
        return;
      case 'ArrowRight':
        event.preventDefault();
        this.moveRight();
        return;
      case 'ArrowLeft':
        event.preventDefault();
        this.moveLeft();
        return;
      case 'Home':
        event.preventDefault();
        this.moveToEdge('start');
        return;
      case 'End':
        event.preventDefault();
        this.moveToEdge('end');
        return;
      case 'Enter':
        event.preventDefault();
        this.activate(this.currentItem);
        return;
      case ' ':
        event.preventDefault();
        this.select(this.currentItem);
        return;
      case '*':
        event.preventDefault();
        this.expandSiblings(this.currentItem);
        return;
      default:
        break;
    }

    const isPrintable =
      event.key.length === 1 &&
      !event.ctrlKey &&
      !event.metaKey &&
      !event.altKey &&
      !event.isComposing;

    if (isPrintable) this.handleTypeahead(event.key);
  }

  bindEvents() {
    this.tree.addEventListener('keydown', (event) => {
      this.handleKeyDown(event);
    });

    this.tree.addEventListener('focusin', (event) => {
      const item = event.target.closest('[role="treeitem"]');
      if (item && this.tree.contains(item)) {
        this.setCurrent(item, { focus: false });
      }
    });

    this.tree.addEventListener('click', (event) => {
      const item = event.target.closest('[role="treeitem"]');
      if (!item || !this.tree.contains(item)) return;

      this.setCurrent(item);
      this.activate(item);
    });
  }
}

const projectTree = new AccessibleTree(
  document.getElementById('project-tree')
);
const selectedFile = document.getElementById('selected-file');

projectTree.tree.addEventListener('treeselectionchange', (event) => {
  selectedFile.textContent = `${event.detail.label}を選択しました。`;
});
```

### 実装の責務

- `aria-expanded`と子グループの`hidden`を同時に更新する
- 見えている項目の一つだけを`tabindex="0"`にする
- 上下移動では閉じた子孫を除外する
- 右・左矢印で開閉と親子移動を切り替える
- 文字検索の対象も見えている項目だけにする
- フォーカス移動と選択変更を別の処理にする

ARIA属性だけを更新しても、キーボード操作や表示は変わらない。DOMの表示、フォーカス、選択、アプリケーションのデータを同じ状態から同期する。

## 開閉アイコンを表示する

```css
.tree [role="treeitem"] > [data-tree-label]::before {
  display: inline-block;
  inline-size: 1.25em;
  content: "";
}

.tree [role="treeitem"][aria-expanded="false"]
  > [data-tree-label]::before {
  content: "▸";
}

.tree [role="treeitem"][aria-expanded="true"]
  > [data-tree-label]::before {
  content: "▾";
}
```

アイコンは補助的な表示。開閉状態は`aria-expanded`で伝える。CSSのクラスとARIAを別々に更新して食い違わせない。

アイコンだけを独立したフォーカス可能ボタンにすると、一つの項目内に複数のTab停止位置ができ、Tree Viewの操作モデルが複雑になる。項目全体を一つのフォーカス対象として扱う。

## CSSで階層と状態を区別する

```css
.tree,
.tree [role="group"] {
  margin: 0;
  padding: 0;
  list-style: none;
}

.tree {
  max-block-size: 24rem;
  overflow: auto;
  border: 0.125rem solid CanvasText;
  border-radius: 0.375rem;
}

.tree [role="group"] {
  margin-inline-start: 1.5rem;
}

.tree [role="treeitem"] > [data-tree-label] {
  display: block;
  min-block-size: 2.25rem;
  padding: 0.375rem 0.625rem;
  border: 0.125rem solid transparent;
}

.tree [role="treeitem"]:focus {
  outline: none;
}

.tree [role="treeitem"]:focus > [data-tree-label] {
  border-color: Highlight;
  outline: 0.125rem solid Highlight;
  outline-offset: -0.125rem;
}

.tree [role="treeitem"][aria-selected="true"]
  > [data-tree-label] {
  font-weight: 700;
  text-decoration: underline;
  background: SelectedItem;
  color: SelectedItemText;
}

@media (forced-colors: active) {
  .tree [role="treeitem"][aria-selected="true"]
    > [data-tree-label] {
    border-color: CanvasText;
  }
}
```

フォーカスと選択は同時に別の項目へ存在できる。フォーカスはアウトライン、選択は背景と文字装飾のように、両方を区別できる見た目にする。

`hidden`の子グループをCSSで無理に表示しない。見た目だけ閉じてアクセシビリティツリーには残る状態も避ける。

## 文字検索

APGはすべてのツリーで文字検索を推奨し、特にルート項目が七つを超える場合に重要としている。

- 入力した文字列で始まる次の見える項目へ移動する
- 末尾まで探したら先頭へ戻って検索する
- 大文字と小文字を区別しない
- 同じ文字を繰り返したら、同じ文字で始まる次の項目へ移動する
- IME変換中のキーを処理しない
- 日本語など対象言語で実際にテストする

文字検索は項目を絞り込む機能ではない。大量の項目を検索して表示を減らす必要があるなら、ツリー外に検索入力を設け、検索結果と元の階層の関係を説明する。

## 単一選択と複数選択

### 単一選択

```html
<ul role="tree" aria-label="ファイル">
  <li role="treeitem" aria-selected="true" tabindex="0">README.md</li>
  <li role="treeitem" aria-selected="false" tabindex="-1">LICENSE</li>
</ul>
```

選択可能なすべての項目へ`aria-selected="true"`または`false`を明示する。選択を持たない単なるナビゲーションツリーでは、不要な`aria-selected`を付けない。

### 複数選択

```html
<ul
  role="tree"
  aria-label="ファイル"
  aria-multiselectable="true"
>
  <!-- aria-selectedを持つtreeitem -->
</ul>
```

複数選択では、修飾キーを押し続けなくても選択を維持できるAPG推奨モデルを優先する。

| キー | 動作 |
|---|---|
| ↑ / ↓ | 選択を変えず、フォーカスだけを移動する |
| Space | フォーカス中の項目の選択を切り替える |
| Shift + ↑ / ↓ | 移動先の選択を切り替える（任意） |
| Shift + Space | 選択起点から現在項目までを選ぶ（任意） |
| Ctrl / Command + A | すべて選択または解除する（任意） |

この記事のJavaScript例は単一選択。複数選択へ拡張する場合、`select()`で他項目を解除せず、Spaceで現在項目だけを切り替える。選択件数が多い場合は、見える「すべて選択」「すべて解除」も用意する。

## ナビゲーションツリー

ツリー項目がページへのリンクなら、`aria-selected`ではなく`aria-current="page"`で現在ページを示す。

```html
<nav aria-label="ドキュメント">
  <ul role="tree" aria-label="ドキュメント">
    <li role="none">
      <a href="/guide/" role="treeitem" tabindex="-1">
        ガイド
      </a>
    </li>
    <li role="none">
      <a
        href="/api/"
        role="treeitem"
        tabindex="0"
        aria-current="page"
      >
        APIリファレンス
      </a>
    </li>
  </ul>
</nav>
```

リンク自体を`treeitem`兼フォーカス対象にし、外側の`li`は`role="none"`にする。`treeitem`の中へ別のフォーカス可能なリンクを置くと、ロービング`tabindex`とリンクのフォーカスが二重になる。階層を含む実装では、APG例を参照し、一つの項目に一つのフォーカス位置を保つ。

SPAで項目を実行して本文を切り替える場合は、次のどちらかを一貫して選ぶ。

- 新しい本文の見出しへフォーカスを移し、遷移完了を伝える
- ツリー項目へフォーカスを残し、`aria-current`の移動などで現在ページを伝える

サイトナビゲーションにTree Viewが本当に必要か、先にディスクロージャーパターンと比較する。

> 参照: [W3C WAI-ARIA APG — Navigation Treeview Example](https://www.w3.org/WAI/ARIA/apg/patterns/treeview/examples/treeview-navigation/)

## 階層情報を明示する

DOMの入れ子と`role="group"`から、ブラウザが階層レベル、同階層内の位置、項目数を計算できる場合がある。

仮想スクロールや一部だけをDOMへ描画する場合、または対象環境のテストで自動計算が正しく伝わらない場合は、明示的に指定する。

```html
<li
  role="treeitem"
  aria-level="3"
  aria-posinset="4"
  aria-setsize="12"
  aria-selected="false"
  tabindex="-1"
>
  4番目の項目
</li>
```

| 属性 | 意味 |
|---|---|
| `aria-level` | ツリー内の階層レベル（1始まり） |
| `aria-posinset` | 同じ階層の兄弟内での位置（1始まり） |
| `aria-setsize` | 同じ階層にある項目の総数 |

DOM構造と明示値が食い違うと、かえって誤った情報を伝える。通常の入れ子構造で十分なら重複して付けず、必要な環境で検証してから使う。

## 動的読み込み

閉じた親項目を初めて開いたときに、子項目を取得する場合がある。

```html
<li
  id="remote-folder"
  role="treeitem"
  aria-expanded="false"
  aria-busy="false"
  tabindex="-1"
>
  <span data-tree-label>共有フォルダー</span>
  <ul role="group" hidden></ul>
</li>
<p id="tree-loading-status" role="status" class="visually-hidden"></p>
```

読み込み時は次を同期する。

1. 親項目を`aria-busy="true"`にする
2. 「共有フォルダーを読み込んでいます」と必要な場合だけ通知する
3. 子項目を追加し、ロール、名前、`tabindex`を設定する
4. `aria-busy="false"`へ戻す
5. 親を開き、最初の子へ移動できるようにする
6. 失敗した場合は理由と再試行手段を示す

読み込み開始だけでフォーカスを別の場所へ移さない。利用者が次の操作をした時点で子項目がまだなければ、読み込み完了まで現在の親項目へフォーカスを残す。

子項目の追加後は、JavaScriptクラスの`syncGroups()`と`refresh()`を呼び、新しい項目をロービング`tabindex`へ含める。

## 動的な削除とフォーカス

フォーカス中または選択中の項目を削除する場合は、削除前に移動先を決める。

1. 次に見えている兄弟
2. 次がなければ前の兄弟
3. 兄弟がなければ親項目
4. ツリーが空になるなら、ツリー外の論理的な操作

```js
const visible = treeController.visibleItems;
const currentIndex = visible.indexOf(itemToRemove);
const fallback = visible[currentIndex + 1] ??
  visible[currentIndex - 1] ??
  treeController.parentItem(itemToRemove);

itemToRemove.remove();
treeController.syncGroups();
treeController.refresh(fallback);
treeController.currentItem?.focus();
```

フォーカス中のDOMを先に削除すると、フォーカスがページ本文へ失われる場合がある。削除、フォーカス管理、選択状態、詳細表示を一つの処理として設計する。

## 大量の項目と仮想化

ツリー全体を仮想化すると、閉じた項目だけでなく画面外の見えている項目もDOMから消える。支援技術へ伝える位置情報とキーボード移動を自分で管理する必要がある。

- `aria-level`、`aria-posinset`、`aria-setsize`を正しく更新する
- フォーカス中の項目をDOMから削除しない
- Endキーが論理的な最後の見える項目へ到達できる
- 文字検索が未描画の項目も対象にするか仕様化する
- 読み込み失敗後も現在位置を失わない

項目数が非常に多い場合は、検索、絞り込み、段階的な選択、パンくずリストと一覧の組み合わせなど、階層全体を一度に操作させない設計も検討する。

## よくある失敗

### `role="tree"`を付けただけ

ARIAは矢印キー操作を追加しない。開閉、親子移動、ロービング`tabindex`、文字検索を実装する。

### すべての項目がTab順序へ入る

ツリー内の有効な項目は一つだけを`tabindex="0"`にする。内部移動は矢印キーで行う。

### 末端項目へ`aria-expanded="false"`を付ける

子がない項目を閉じた親として伝えない。開閉できる項目だけに`aria-expanded`を付ける。

### 見た目だけを開閉する

子グループの表示と`aria-expanded`を同時に更新する。閉じた子孫がキーボード移動やアクセシビリティツリーへ残らないようにする。

### 上下矢印で閉じた子孫へ移動する

DOM上の全項目ではなく、現在見えている項目のリストから次の移動先を求める。

### フォーカス移動で意図せず選択する

詳細表示、通信、ファイル操作などの副作用がある場合は、フォーカスと選択を分ける。

### フォーカスと選択を同じ見た目にする

探索中の現在位置と、操作対象として選ばれた項目を視覚的に区別する。

### 親項目の名前へ子孫の文字が含まれる

見えるラベル要素を`aria-labelledby`で参照し、親項目の名前を限定する。

### 項目内に複数の操作要素を入れる

チェックボックス、リンク、メニューボタンなどを一つの`treeitem`へ詰め込むと、フォーカス規則が複雑になる。Treegridや詳細パネルへ操作を分ける。

### サイトナビゲーションへ過剰に使う

開閉式リンク一覧だけなら、ディスクロージャーと通常のリストを使う。

## テストのチェックリスト

### パターンの選択

- [ ] 階層を矢印キーで頻繁に探索する必要がある
- [ ] 通常のリストとディスクロージャーでは不足する理由がある
- [ ] 複数列が必要ならTreegridを検討した
- [ ] 項目内に複数の操作要素を入れていない
- [ ] 大量項目なら検索や段階的な選択も検討した

### 構造と名前

- [ ] ルートに`role="tree"`と名前がある
- [ ] すべての項目に`role="treeitem"`がある
- [ ] 子項目が`role="group"`で親項目の中にある
- [ ] 開閉できる親項目だけに`aria-expanded`がある
- [ ] 選択可能な項目の`aria-selected`が正しい
- [ ] 親項目の名前へ子孫のラベルが混ざらない
- [ ] 明示した階層・位置・総数がDOMと一致する

### フォーカスとキーボード

- [ ] 見えている項目の一つだけが`tabindex="0"`である
- [ ] Tab / Shift + Tabでツリーへ出入りできる
- [ ] 上下矢印で前後の見えている項目へ移動できる
- [ ] 右矢印で親を開き、開いていれば最初の子へ移動できる
- [ ] 左矢印で親を閉じ、閉じていれば親へ移動できる
- [ ] HomeとEndで見えている先頭・末尾へ移動できる
- [ ] 文字入力で一致する見えている項目へ移動できる
- [ ] 閉じた子孫へフォーカスが入らない
- [ ] 項目の削除後もフォーカスが見える位置に残る

### 選択と実行

- [ ] フォーカスと選択の関係を仕様化している
- [ ] SpaceまたはEnterの動作が画面内で一貫している
- [ ] 複数選択では矢印移動だけで選択を失わない
- [ ] 選択とフォーカスを異なる見た目で確認できる
- [ ] ナビゲーションでは現在ページを`aria-current`で示している
- [ ] 実行後のフォーカス移動が予測できる

### 表示と動的更新

- [ ] 開閉状態をアイコンだけに依存せず伝えている
- [ ] フォーカス表示が明確に見える
- [ ] 200%・400%拡大でも階層とラベルを確認できる
- [ ] 強制カラーモードでもフォーカスと選択を区別できる
- [ ] 読み込み中、失敗、空の状態を伝えている
- [ ] 動的追加後の階層とロービング`tabindex`が正しい
- [ ] 仮想化時の位置情報とフォーカスが正しい
- [ ] キーボード、マウス、タッチ、スクリーンリーダーで確認した

## 関連して読むとよいテーマ

- [ロービングタブインデックスパターン](./roving-tabindex.md)
- [アクセシブルなリストボックスの実装 — 単一選択と複数選択](./accessible-listbox.md)
- [アクセシブルなディスクロージャーの実装 — 一部だけを開閉する](./accessible-disclosure.md)
- [アクセシブルなツールバーの実装 — 関連する操作をまとめる](./accessible-toolbar.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)

## まとめ

- 階層を頻繁に探索する必要がある場合にTree Viewを使う
- 一般的なサイトナビゲーションでは、通常のリストとディスクロージャーを優先する
- `tree`、`treeitem`、`group`で親子構造を伝える
- 開閉できる親項目だけに`aria-expanded`を付け、子グループの表示と同期する
- 見えている項目の一つだけを`tabindex="0"`にする
- 上下矢印は見えている前後、右・左矢印は開閉と親子移動に使う
- フォーカス、展開、選択、現在ページを別の状態として管理する
- 親項目の名前へ子孫の文字列を含めない
- 動的な追加・削除・読み込み後も、階層情報とフォーカスを維持する
- 対象ブラウザと支援技術で階層、位置、開閉、選択を確認する
