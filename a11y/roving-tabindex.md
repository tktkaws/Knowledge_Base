# ロービングタブインデックスパターン

## ロービングタブインデックスとは

- 複合ウィジェット内のフォーカス管理パターン
- グループ内で1つの要素だけが `tabindex="0"` を持ち、残りは `tabindex="-1"` にする
- 矢印キーでグループ内を移動するたびに `tabindex` の値を付け替える
- Tabキーではグループ全体を1つのフォーカスストップとして扱える

## なぜ必要か

- タブUI、ツールバー、ラジオボタン群など、複数の関連する要素が並ぶウィジェットがある
- 全ての要素に `tabindex="0"` を付けると、Tabキーで1つずつ通過する必要があり操作が煩雑
- 例：20個のタブがあるタブリストを全てTabで移動するのは現実的でない
- ロービングタブインデックスなら、Tabでグループに入り、矢印キーで項目間を移動し、Tabでグループを出る

> 参照: [WAI-ARIA Authoring Practices — Keyboard Navigation Inside Components](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

## 基本的な仕組み

### 初期状態

- 最初の項目（またはアクティブな項目）に `tabindex="0"`
- 他の全項目に `tabindex="-1"`

```html
<!-- タブリストの例 -->
<div role="tablist">
  <button role="tab" tabindex="0" aria-selected="true">タブ1</button>
  <button role="tab" tabindex="-1" aria-selected="false">タブ2</button>
  <button role="tab" tabindex="-1" aria-selected="false">タブ3</button>
</div>
```

### 矢印キーで移動する際の流れ

1. 右矢印キーが押される
2. 現在の要素を `tabindex="-1"` に変更
3. 次の要素を `tabindex="0"` に変更
4. 次の要素に `focus()` を呼ぶ

```html
<!-- 右矢印キーで「タブ2」に移動した後の状態 -->
<div role="tablist">
  <button role="tab" tabindex="-1" aria-selected="false">タブ1</button>
  <button role="tab" tabindex="0" aria-selected="true">タブ2</button>
  <button role="tab" tabindex="-1" aria-selected="false">タブ3</button>
</div>
```

## 実装例

### 基本実装

```js
class RovingTabIndex {
  constructor(container, selector) {
    this.container = container;
    this.items = [...container.querySelectorAll(selector)];
    this.currentIndex = 0;

    this.init();
  }

  init() {
    // 最初の要素のみ tabindex="0"
    this.items.forEach((item, index) => {
      item.setAttribute('tabindex', index === 0 ? '0' : '-1');
    });

    this.container.addEventListener('keydown', (event) => {
      this.handleKeyDown(event);
    });
  }

  handleKeyDown(event) {
    let newIndex = this.currentIndex;

    switch (event.key) {
      case 'ArrowRight':
      case 'ArrowDown':
        event.preventDefault();
        newIndex = (this.currentIndex + 1) % this.items.length;
        break;
      case 'ArrowLeft':
      case 'ArrowUp':
        event.preventDefault();
        newIndex = (this.currentIndex - 1 + this.items.length) % this.items.length;
        break;
      case 'Home':
        event.preventDefault();
        newIndex = 0;
        break;
      case 'End':
        event.preventDefault();
        newIndex = this.items.length - 1;
        break;
      default:
        return;
    }

    this.moveFocus(newIndex);
  }

  moveFocus(newIndex) {
    // 現在の要素を tabindex="-1" に
    this.items[this.currentIndex].setAttribute('tabindex', '-1');
    // 新しい要素を tabindex="0" に
    this.items[newIndex].setAttribute('tabindex', '0');
    // フォーカスを移動
    this.items[newIndex].focus();

    this.currentIndex = newIndex;
  }
}

// 使用例
const tablist = document.querySelector('[role="tablist"]');
new RovingTabIndex(tablist, '[role="tab"]');
```

### React での実装例

```jsx
import { useState, useRef, useCallback } from 'react';

function Tablist({ tabs, onSelect }) {
  const [activeIndex, setActiveIndex] = useState(0);
  const tabRefs = useRef([]);

  const handleKeyDown = useCallback((event) => {
    let newIndex = activeIndex;

    switch (event.key) {
      case 'ArrowRight':
        event.preventDefault();
        newIndex = (activeIndex + 1) % tabs.length;
        break;
      case 'ArrowLeft':
        event.preventDefault();
        newIndex = (activeIndex - 1 + tabs.length) % tabs.length;
        break;
      case 'Home':
        event.preventDefault();
        newIndex = 0;
        break;
      case 'End':
        event.preventDefault();
        newIndex = tabs.length - 1;
        break;
      default:
        return;
    }

    setActiveIndex(newIndex);
    tabRefs.current[newIndex]?.focus();
    onSelect(newIndex);
  }, [activeIndex, tabs.length, onSelect]);

  return (
    <div role="tablist">
      {tabs.map((tab, index) => (
        <button
          key={tab.id}
          role="tab"
          ref={(el) => { tabRefs.current[index] = el; }}
          tabIndex={index === activeIndex ? 0 : -1}
          aria-selected={index === activeIndex}
          aria-controls={`panel-${tab.id}`}
          onClick={() => {
            setActiveIndex(index);
            onSelect(index);
          }}
          onKeyDown={handleKeyDown}
        >
          {tab.label}
        </button>
      ))}
    </div>
  );
}
```

## 適用するコンポーネントパターン

### タブリスト（横方向）

- 左右矢印キーでタブ間を移動
- Home / End で最初 / 最後のタブに移動
- Enterまたは Spaceでタブを選択（自動選択モードの場合はフォーカス移動時に自動選択）

| キー | 動作 |
|---|---|
| `→` | 次のタブにフォーカス |
| `←` | 前のタブにフォーカス |
| `Home` | 最初のタブにフォーカス |
| `End` | 最後のタブにフォーカス |

> 参照: [WAI-ARIA Authoring Practices — Tabs Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/)

### ツールバー（横方向）

- 左右矢印キーでツールバー内のボタン間を移動
- グループ化されたボタンがある場合、矢印キーでグループ内を移動

| キー | 動作 |
|---|---|
| `→` | 次のボタンにフォーカス |
| `←` | 前のボタンにフォーカス |
| `Home` | 最初のボタンにフォーカス |
| `End` | 最後のボタンにフォーカス |

> 参照: [WAI-ARIA Authoring Practices — Toolbar Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/toolbar/)

### ラジオグループ

- 矢印キーで選択を移動（フォーカスと選択が連動）
- 上下 / 左右の矢印キーどちらでも移動可能

| キー | 動作 |
|---|---|
| `→` / `↓` | 次のラジオボタンを選択 |
| `←` / `↑` | 前のラジオボタンを選択 |

```html
<div role="radiogroup" aria-label="配送方法">
  <div role="radio" tabindex="0" aria-checked="true">通常配送</div>
  <div role="radio" tabindex="-1" aria-checked="false">速達</div>
  <div role="radio" tabindex="-1" aria-checked="false">当日配送</div>
</div>
```

> 参照: [WAI-ARIA Authoring Practices — Radio Group Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/radio/)

### メニュー（縦方向）

- 上下矢印キーでメニュー項目間を移動

| キー | 動作 |
|---|---|
| `↓` | 次のメニュー項目にフォーカス |
| `↑` | 前のメニュー項目にフォーカス |
| `Home` | 最初のメニュー項目にフォーカス |
| `End` | 最後のメニュー項目にフォーカス |
| `Enter` | メニュー項目を実行 |
| `Escape` | メニューを閉じる |

> 参照: [WAI-ARIA Authoring Practices — Menu Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/menubar/)

## ロービングタブインデックス vs aria-activedescendant

- フォーカスをグループ内で管理するもう一つの方法として `aria-activedescendant` がある
- コンテナ要素にフォーカスを保持したまま、`aria-activedescendant` で「どの子要素がアクティブか」を示す

### ロービングタブインデックス

- 実際のDOMフォーカスが移動する
- 各項目のイベントハンドラが自然に動作する
- 実装が直感的

### aria-activedescendant

- フォーカスはコンテナに固定される
- 子要素の `id` を `aria-activedescendant` 属性で指定
- コンボボックスやリストボックスなど、入力欄とリストが連動するUIで有効

```html
<!-- aria-activedescendant の例：コンボボックス -->
<input
  role="combobox"
  aria-expanded="true"
  aria-activedescendant="option-2"
  aria-controls="listbox"
/>
<ul role="listbox" id="listbox">
  <li role="option" id="option-1">東京</li>
  <li role="option" id="option-2" aria-selected="true">大阪</li>
  <li role="option" id="option-3">名古屋</li>
</ul>
```

### 使い分けの目安

| パターン | 推奨手法 |
|---|---|
| タブリスト | ロービングタブインデックス |
| ツールバー | ロービングタブインデックス |
| ラジオグループ | ロービングタブインデックス |
| コンボボックス | aria-activedescendant |
| リストボックス | どちらも可（状況による） |
| ツリービュー | どちらも可（状況による） |

> 参照: [WAI-ARIA Authoring Practices — Managing Focus in Composites](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/#kbd_focus_activedescendant)

## disabled な項目の扱い

- disabled な項目にフォーカスを当てるかどうかは2つの方針がある

### 方針1：disabled 項目をスキップする

- 矢印キーで移動する際に disabled 項目を飛ばす
- ユーザーは有効な項目だけを操作できる

```js
moveFocus(direction) {
  let newIndex = this.currentIndex;
  do {
    newIndex = (newIndex + direction + this.items.length) % this.items.length;
  } while (this.items[newIndex].disabled && newIndex !== this.currentIndex);

  if (!this.items[newIndex].disabled) {
    this.setFocus(newIndex);
  }
}
```

### 方針2：disabled 項目にもフォーカスを当てる

- disabled 項目にフォーカスは当たるが、操作はできない
- ユーザーがその項目の存在と状態を認識できる
- WAI-ARIA Authoring Practices ではこちらを推奨するパターンもある

```html
<button role="tab" tabindex="-1" aria-disabled="true">
  無効なタブ
</button>
```

## よくある間違い

### 1. 全項目を tabindex="0" にする

```html
<!-- 間違い：Tabキーで全項目を通過する必要がある -->
<div role="tablist">
  <button role="tab" tabindex="0">タブ1</button>
  <button role="tab" tabindex="0">タブ2</button>
  <button role="tab" tabindex="0">タブ3</button>
</div>

<!-- 正解：アクティブな項目だけ tabindex="0" -->
<div role="tablist">
  <button role="tab" tabindex="0" aria-selected="true">タブ1</button>
  <button role="tab" tabindex="-1">タブ2</button>
  <button role="tab" tabindex="-1">タブ3</button>
</div>
```

### 2. 矢印キーのループ処理を忘れる

- 最後の項目で右矢印を押したとき、最初の項目に戻る循環処理が必要
- 循環しないとキーボードユーザーが最初の項目に戻れない

```js
// 間違い：端で止まる
newIndex = Math.min(currentIndex + 1, items.length - 1);

// 正解：ループする
newIndex = (currentIndex + 1) % items.length;
```

### 3. Home / End キーの実装漏れ

- Home / End キーで最初 / 最後の項目に移動するのはWAI-ARIAのパターンで求められている
- 矢印キーだけでは不十分

### 4. フォーカス移動時に tabindex の付け替えを忘れる

```js
// 間違い：focus() だけでtabindexを更新しない
items[newIndex].focus();

// 正解：tabindex を付け替えてからフォーカス
items[currentIndex].setAttribute('tabindex', '-1');
items[newIndex].setAttribute('tabindex', '0');
items[newIndex].focus();
```

## まとめ

- ロービングタブインデックスは、複合ウィジェット内でTabキーのフォーカスストップを1つに減らすパターン
- グループ内は矢印キーで移動、`tabindex` の `0` と `-1` を付け替える
- タブリスト、ツールバー、ラジオグループ、メニューなどで使用
- Home / End キーの対応、端での循環処理を忘れない
- コンボボックスなど入力と連動するUIでは `aria-activedescendant` も検討
