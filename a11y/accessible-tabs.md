# アクセシブルなタブUIの実装

## タブUIとは

- 複数のコンテンツパネルを切り替えて表示するUI
- 画面スペースを節約しつつ、関連コンテンツをグループ化できる
- ネイティブHTML要素が存在しないため、WAI-ARIAのロールとキーボード操作の実装が必須

## 必要なARIA属性

| 要素 | ロール / 属性 | 説明 |
|---|---|---|
| タブリストのコンテナ | `role="tablist"` | タブのグループであることを示す |
| 各タブ | `role="tab"` | タブであることを示す |
| 各パネル | `role="tabpanel"` | タブに対応するコンテンツ領域 |
| 選択中のタブ | `aria-selected="true"` | 現在アクティブなタブ |
| 非選択のタブ | `aria-selected="false"` | アクティブでないタブ |
| タブ → パネル | `aria-controls="パネルのid"` | タブが制御するパネルを紐付け |
| パネル → タブ | `aria-labelledby="タブのid"` | パネルのラベルをタブから取得 |

> 参照: [WAI-ARIA Authoring Practices — Tabs Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/)

## キーボード操作の要件

| キー | 動作 |
|---|---|
| `Tab` | タブリスト内のアクティブなタブにフォーカス → 次のTabでタブパネルへ |
| `→` | 次のタブにフォーカス（末尾から先頭へループ） |
| `←` | 前のタブにフォーカス（先頭から末尾へループ） |
| `Home` | 最初のタブにフォーカス |
| `End` | 最後のタブにフォーカス |
| `Enter` / `Space` | 手動選択モード時にタブを選択 |

- ロービングタブインデックスを使用し、アクティブなタブのみ `tabindex="0"`
- 縦方向のタブリストでは `↑` / `↓` を使用

## HTML構造

```html
<div class="tabs">
  <div role="tablist" aria-label="プロジェクト情報">
    <button
      role="tab"
      id="tab-overview"
      aria-selected="true"
      aria-controls="panel-overview"
      tabindex="0"
    >
      概要
    </button>
    <button
      role="tab"
      id="tab-members"
      aria-selected="false"
      aria-controls="panel-members"
      tabindex="-1"
    >
      メンバー
    </button>
    <button
      role="tab"
      id="tab-settings"
      aria-selected="false"
      aria-controls="panel-settings"
      tabindex="-1"
    >
      設定
    </button>
  </div>

  <div
    role="tabpanel"
    id="panel-overview"
    aria-labelledby="tab-overview"
    tabindex="0"
  >
    <p>プロジェクトの概要情報</p>
  </div>
  <div
    role="tabpanel"
    id="panel-members"
    aria-labelledby="tab-members"
    tabindex="0"
    hidden
  >
    <p>メンバー一覧</p>
  </div>
  <div
    role="tabpanel"
    id="panel-settings"
    aria-labelledby="tab-settings"
    tabindex="0"
    hidden
  >
    <p>プロジェクト設定</p>
  </div>
</div>
```

### tabpanel に tabindex="0" を付ける理由

- パネル内にフォーカス可能な要素がない場合、Tabキーでパネルを飛ばしてしまう
- `tabindex="0"` を付けることでパネル自体がフォーカスを受け取れる
- パネル内にフォーカス可能な要素がある場合は不要

## 自動選択 vs 手動選択

### 自動選択（推奨）

- 矢印キーでフォーカスを移動すると同時にパネルが切り替わる
- 操作ステップが少なく直感的
- パネルの読み込みが軽い場合に適する

### 手動選択

- 矢印キーでフォーカスを移動しただけではパネルは切り替わらない
- Enter / Space を押して明示的に選択する
- パネルの読み込みにコストがかかる場合（API呼び出しなど）に適する

## Vanilla JS での実装

```js
class AccessibleTabs {
  constructor(container) {
    this.tablist = container.querySelector('[role="tablist"]');
    this.tabs = [...this.tablist.querySelectorAll('[role="tab"]')];
    this.panels = this.tabs.map((tab) =>
      container.querySelector(`#${tab.getAttribute('aria-controls')}`)
    );

    this.init();
  }

  init() {
    this.tablist.addEventListener('keydown', (event) => {
      this.handleKeyDown(event);
    });

    this.tabs.forEach((tab) => {
      tab.addEventListener('click', () => {
        this.selectTab(tab);
      });
    });
  }

  handleKeyDown(event) {
    const currentIndex = this.tabs.indexOf(document.activeElement);
    if (currentIndex === -1) return;

    let newIndex;

    switch (event.key) {
      case 'ArrowRight':
        event.preventDefault();
        newIndex = (currentIndex + 1) % this.tabs.length;
        break;
      case 'ArrowLeft':
        event.preventDefault();
        newIndex = (currentIndex - 1 + this.tabs.length) % this.tabs.length;
        break;
      case 'Home':
        event.preventDefault();
        newIndex = 0;
        break;
      case 'End':
        event.preventDefault();
        newIndex = this.tabs.length - 1;
        break;
      default:
        return;
    }

    this.tabs[newIndex].focus();
    // 自動選択モード：フォーカス移動時にパネルも切り替える
    this.selectTab(this.tabs[newIndex]);
  }

  selectTab(selectedTab) {
    // 全タブを非選択状態にする
    this.tabs.forEach((tab, index) => {
      const isSelected = tab === selectedTab;
      tab.setAttribute('aria-selected', String(isSelected));
      tab.setAttribute('tabindex', isSelected ? '0' : '-1');
      this.panels[index].hidden = !isSelected;
    });
  }
}

// 使用
new AccessibleTabs(document.querySelector('.tabs'));
```

## React での実装例

```jsx
import { useState, useRef, useCallback } from 'react';

function Tabs({ items }) {
  const [activeIndex, setActiveIndex] = useState(0);
  const tabRefs = useRef([]);

  const selectTab = useCallback((index) => {
    setActiveIndex(index);
    tabRefs.current[index]?.focus();
  }, []);

  const handleKeyDown = useCallback(
    (event) => {
      const currentIndex = tabRefs.current.indexOf(document.activeElement);
      if (currentIndex === -1) return;

      let newIndex;

      switch (event.key) {
        case 'ArrowRight':
          event.preventDefault();
          newIndex = (currentIndex + 1) % items.length;
          break;
        case 'ArrowLeft':
          event.preventDefault();
          newIndex = (currentIndex - 1 + items.length) % items.length;
          break;
        case 'Home':
          event.preventDefault();
          newIndex = 0;
          break;
        case 'End':
          event.preventDefault();
          newIndex = items.length - 1;
          break;
        default:
          return;
      }

      selectTab(newIndex);
    },
    [items.length, selectTab]
  );

  return (
    <div className="tabs">
      <div role="tablist" aria-label="コンテンツ" onKeyDown={handleKeyDown}>
        {items.map((item, index) => (
          <button
            key={item.id}
            role="tab"
            ref={(el) => { tabRefs.current[index] = el; }}
            id={`tab-${item.id}`}
            aria-selected={index === activeIndex}
            aria-controls={`panel-${item.id}`}
            tabIndex={index === activeIndex ? 0 : -1}
            onClick={() => selectTab(index)}
          >
            {item.label}
          </button>
        ))}
      </div>

      {items.map((item, index) => (
        <div
          key={item.id}
          role="tabpanel"
          id={`panel-${item.id}`}
          aria-labelledby={`tab-${item.id}`}
          tabIndex={0}
          hidden={index !== activeIndex}
        >
          {item.content}
        </div>
      ))}
    </div>
  );
}
```

## 縦方向のタブ

- タブが縦に並ぶ場合は `aria-orientation="vertical"` を付与
- 矢印キーの方向を上下に変更

```html
<div role="tablist" aria-label="設定カテゴリ" aria-orientation="vertical">
  <button role="tab" aria-selected="true" tabindex="0">一般</button>
  <button role="tab" aria-selected="false" tabindex="-1">通知</button>
  <button role="tab" aria-selected="false" tabindex="-1">セキュリティ</button>
</div>
```

```js
// 縦方向タブのキーハンドリング
handleKeyDown(event) {
  switch (event.key) {
    case 'ArrowDown': // ArrowRight の代わり
      event.preventDefault();
      newIndex = (currentIndex + 1) % this.tabs.length;
      break;
    case 'ArrowUp': // ArrowLeft の代わり
      event.preventDefault();
      newIndex = (currentIndex - 1 + this.tabs.length) % this.tabs.length;
      break;
  }
}
```

## よくある間違い

### 1. ロールや属性の付け忘れ

```html
<!-- 間違い：ロールなし、スクリーンリーダーがタブUIと認識できない -->
<div class="tab-list">
  <button class="tab active">タブ1</button>
  <button class="tab">タブ2</button>
</div>
<div class="panel">パネル1</div>

<!-- 正解：全ての必要なロールと属性を付与 -->
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="p1" tabindex="0">タブ1</button>
  <button role="tab" aria-selected="false" aria-controls="p2" tabindex="-1">タブ2</button>
</div>
<div role="tabpanel" id="p1" aria-labelledby="...">パネル1</div>
```

### 2. 全タブを tabindex="0" にする

```html
<!-- 間違い：Tabキーで全タブを通過する -->
<div role="tablist">
  <button role="tab" tabindex="0">タブ1</button>
  <button role="tab" tabindex="0">タブ2</button>
  <button role="tab" tabindex="0">タブ3</button>
</div>

<!-- 正解：選択中のタブのみ tabindex="0" -->
<div role="tablist">
  <button role="tab" tabindex="0" aria-selected="true">タブ1</button>
  <button role="tab" tabindex="-1" aria-selected="false">タブ2</button>
  <button role="tab" tabindex="-1" aria-selected="false">タブ3</button>
</div>
```

### 3. 非表示パネルを display: none ではなく視覚的に隠すだけ

```css
/* 間違い：スクリーンリーダーが非表示パネルも読み上げる */
.panel:not(.active) {
  opacity: 0;
  height: 0;
}

/* 正解：hidden属性またはdisplay: noneで完全に非表示 */
```

```html
<!-- hidden属性で非表示にする -->
<div role="tabpanel" hidden>...</div>
```

### 4. タブとパネルの紐付けがない

- `aria-controls` と `aria-labelledby` を忘れると、スクリーンリーダーがタブとパネルの関係を伝えられない

### 5. タブリストに aria-label を付けない

```html
<!-- 間違い：タブリストの用途が不明 -->
<div role="tablist">...</div>

<!-- 正解：何のタブリストかラベルを付ける -->
<div role="tablist" aria-label="商品情報">...</div>
```

## まとめ

- タブUIは `role="tablist"`, `role="tab"`, `role="tabpanel"` の3つのロールで構成
- `aria-selected`, `aria-controls`, `aria-labelledby` で状態と関係性を示す
- ロービングタブインデックスで矢印キーによるタブ間移動を実装
- 自動選択モード（フォーカスで切替）が一般的で直感的
- 縦方向タブでは `aria-orientation="vertical"` を付与
