# アクセシブルなドロップダウンメニューの実装

## ドロップダウンメニューの種類

- ドロップダウンには複数のパターンがあり、それぞれ適切なARIAパターンが異なる
- 使い分けを誤るとスクリーンリーダーの操作モードが変わり、ユーザーが混乱する

| パターン | 用途 | ARIAパターン |
|---|---|---|
| ナビゲーションメニュー | サイトのリンク集 | Disclosure パターン |
| アクションメニュー | 編集、削除などの操作一覧 | Menu Button パターン |
| セレクト（選択） | 値の選択 | Listbox / Combobox パターン |

> 参照: [WAI-ARIA Authoring Practices — Menu Button Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/)

## Disclosure パターン（ナビゲーション用）

- サイトナビゲーションの子メニューに使用
- `role="menu"` は**使わない** — ナビゲーションリンクにはmenuロールは不適切
- ボタンで開閉を制御し、中身は通常のリンクリスト

### キーボード操作

| キー | 動作 |
|---|---|
| `Enter` / `Space` | メニューを開閉 |
| `Escape` | メニューを閉じてトリガーにフォーカスを戻す |
| `Tab` | メニュー内のリンクを順に移動 |

### 実装例

```html
<nav aria-label="メインナビゲーション">
  <ul>
    <li>
      <a href="/">ホーム</a>
    </li>
    <li>
      <button
        aria-expanded="false"
        aria-controls="products-menu"
      >
        製品
      </button>
      <ul id="products-menu" hidden>
        <li><a href="/products/a">製品A</a></li>
        <li><a href="/products/b">製品B</a></li>
        <li><a href="/products/c">製品C</a></li>
      </ul>
    </li>
    <li>
      <a href="/about">会社概要</a>
    </li>
  </ul>
</nav>
```

```js
const trigger = document.querySelector('[aria-controls="products-menu"]');
const menu = document.getElementById('products-menu');

trigger.addEventListener('click', () => {
  const isOpen = trigger.getAttribute('aria-expanded') === 'true';
  trigger.setAttribute('aria-expanded', String(!isOpen));
  menu.hidden = isOpen;

  if (!isOpen) {
    // メニューを開いたら最初のリンクにフォーカス
    menu.querySelector('a')?.focus();
  }
});

// Escapeキーでメニューを閉じる
menu.addEventListener('keydown', (event) => {
  if (event.key === 'Escape') {
    trigger.setAttribute('aria-expanded', 'false');
    menu.hidden = true;
    trigger.focus();
  }
});
```

> 参照: [WAI-ARIA Authoring Practices — Disclosure (Show/Hide) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/)

## Menu Button パターン（アクション用）

- 「編集」「削除」「共有」などのアクション一覧を表示するメニュー
- デスクトップアプリのメニューバーに近い操作モデル
- `role="menu"` と `role="menuitem"` を使用
- 矢印キーで項目間を移動する

### キーボード操作

| キー | 動作 |
|---|---|
| `Enter` / `Space` / `↓` | メニューを開き、最初の項目にフォーカス |
| `↑` | メニューを開き、最後の項目にフォーカス |
| `↓` (メニュー内) | 次の項目にフォーカス |
| `↑` (メニュー内) | 前の項目にフォーカス |
| `Home` | 最初の項目にフォーカス |
| `End` | 最後の項目にフォーカス |
| `Escape` | メニューを閉じてトリガーにフォーカスを戻す |
| `Enter` / `Space` (項目) | 項目を実行してメニューを閉じる |
| 文字キー | その文字で始まる項目にフォーカス |

### 必要なARIA属性

| 要素 | 属性 |
|---|---|
| トリガーボタン | `aria-haspopup="true"`, `aria-expanded`, `aria-controls` |
| メニューコンテナ | `role="menu"`, `aria-labelledby`（トリガーのid） |
| メニュー項目 | `role="menuitem"` |
| 区切り線 | `role="separator"` |

### 実装例

```html
<button
  id="action-btn"
  aria-haspopup="true"
  aria-expanded="false"
  aria-controls="action-menu"
>
  操作 ▾
</button>

<ul id="action-menu" role="menu" aria-labelledby="action-btn" hidden>
  <li role="menuitem" tabindex="-1">編集</li>
  <li role="menuitem" tabindex="-1">複製</li>
  <li role="separator"></li>
  <li role="menuitem" tabindex="-1">削除</li>
</ul>
```

```js
class MenuButton {
  constructor(button, menu) {
    this.button = button;
    this.menu = menu;
    this.items = [...menu.querySelectorAll('[role="menuitem"]')];
    this.currentIndex = -1;

    this.button.addEventListener('click', () => this.toggle());
    this.button.addEventListener('keydown', (e) => this.handleButtonKeyDown(e));
    this.menu.addEventListener('keydown', (e) => this.handleMenuKeyDown(e));

    // メニュー外クリックで閉じる
    document.addEventListener('click', (e) => {
      if (!this.button.contains(e.target) && !this.menu.contains(e.target)) {
        this.close();
      }
    });
  }

  open(focusIndex = 0) {
    this.button.setAttribute('aria-expanded', 'true');
    this.menu.hidden = false;
    this.focusItem(focusIndex);
  }

  close() {
    this.button.setAttribute('aria-expanded', 'false');
    this.menu.hidden = true;
    this.currentIndex = -1;
    this.button.focus();
  }

  toggle() {
    const isOpen = this.button.getAttribute('aria-expanded') === 'true';
    isOpen ? this.close() : this.open();
  }

  focusItem(index) {
    this.currentIndex = index;
    this.items[index]?.focus();
  }

  handleButtonKeyDown(event) {
    switch (event.key) {
      case 'ArrowDown':
      case 'Enter':
      case ' ':
        event.preventDefault();
        this.open(0); // 最初の項目にフォーカス
        break;
      case 'ArrowUp':
        event.preventDefault();
        this.open(this.items.length - 1); // 最後の項目にフォーカス
        break;
    }
  }

  handleMenuKeyDown(event) {
    switch (event.key) {
      case 'ArrowDown':
        event.preventDefault();
        this.focusItem((this.currentIndex + 1) % this.items.length);
        break;
      case 'ArrowUp':
        event.preventDefault();
        this.focusItem(
          (this.currentIndex - 1 + this.items.length) % this.items.length
        );
        break;
      case 'Home':
        event.preventDefault();
        this.focusItem(0);
        break;
      case 'End':
        event.preventDefault();
        this.focusItem(this.items.length - 1);
        break;
      case 'Escape':
        this.close();
        break;
      case 'Enter':
      case ' ':
        event.preventDefault();
        this.items[this.currentIndex]?.click();
        this.close();
        break;
      default:
        // 文字キーで始まる項目にフォーカス
        this.handleTypeAhead(event.key);
    }
  }

  handleTypeAhead(char) {
    if (char.length !== 1) return;
    const index = this.items.findIndex(
      (item) => item.textContent.trim().toLowerCase().startsWith(char.toLowerCase())
    );
    if (index !== -1) {
      this.focusItem(index);
    }
  }
}

// 使用
const btn = document.getElementById('action-btn');
const menu = document.getElementById('action-menu');
new MenuButton(btn, menu);
```

## React での実装例

```jsx
import { useState, useRef, useCallback, useEffect } from 'react';

function DropdownMenu({ label, items }) {
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);
  const buttonRef = useRef(null);
  const itemRefs = useRef([]);

  // メニュー外クリックで閉じる
  useEffect(() => {
    if (!isOpen) return;
    const handleClick = (e) => {
      if (!buttonRef.current?.contains(e.target) &&
          !itemRefs.current.some((ref) => ref?.contains(e.target))) {
        setIsOpen(false);
      }
    };
    document.addEventListener('click', handleClick);
    return () => document.removeEventListener('click', handleClick);
  }, [isOpen]);

  // activeIndexが変わったらフォーカス
  useEffect(() => {
    if (activeIndex >= 0) {
      itemRefs.current[activeIndex]?.focus();
    }
  }, [activeIndex]);

  const open = useCallback((focusIndex = 0) => {
    setIsOpen(true);
    setActiveIndex(focusIndex);
  }, []);

  const close = useCallback(() => {
    setIsOpen(false);
    setActiveIndex(-1);
    buttonRef.current?.focus();
  }, []);

  const handleButtonKeyDown = useCallback((event) => {
    switch (event.key) {
      case 'ArrowDown':
      case 'Enter':
      case ' ':
        event.preventDefault();
        open(0);
        break;
      case 'ArrowUp':
        event.preventDefault();
        open(items.length - 1);
        break;
    }
  }, [open, items.length]);

  const handleMenuKeyDown = useCallback((event) => {
    switch (event.key) {
      case 'ArrowDown':
        event.preventDefault();
        setActiveIndex((prev) => (prev + 1) % items.length);
        break;
      case 'ArrowUp':
        event.preventDefault();
        setActiveIndex((prev) => (prev - 1 + items.length) % items.length);
        break;
      case 'Home':
        event.preventDefault();
        setActiveIndex(0);
        break;
      case 'End':
        event.preventDefault();
        setActiveIndex(items.length - 1);
        break;
      case 'Escape':
        close();
        break;
      case 'Enter':
      case ' ':
        event.preventDefault();
        items[activeIndex]?.action();
        close();
        break;
    }
  }, [activeIndex, items, close]);

  return (
    <div className="dropdown">
      <button
        ref={buttonRef}
        aria-haspopup="true"
        aria-expanded={isOpen}
        onClick={() => (isOpen ? close() : open())}
        onKeyDown={handleButtonKeyDown}
      >
        {label} ▾
      </button>

      {isOpen && (
        <ul role="menu" onKeyDown={handleMenuKeyDown}>
          {items.map((item, index) => (
            <li
              key={item.id}
              role="menuitem"
              ref={(el) => { itemRefs.current[index] = el; }}
              tabIndex={-1}
              onClick={() => { item.action(); close(); }}
            >
              {item.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## role="menu" を使うべきでない場面

- サイトのナビゲーション（`<nav>` + Disclosureパターンを使う）
- 値を選択するUI（Listbox / Combobox パターンを使う）
- `role="menu"` はアプリケーションメニュー（ファイル→新規作成 のような操作メニュー）用

```html
<!-- 間違い：ナビゲーションに role="menu" を使う -->
<nav>
  <ul role="menu">
    <li role="menuitem"><a href="/">ホーム</a></li>
    <li role="menuitem"><a href="/about">概要</a></li>
  </ul>
</nav>

<!-- 正解：ナビゲーションは通常のリスト -->
<nav aria-label="メインナビゲーション">
  <ul>
    <li><a href="/">ホーム</a></li>
    <li><a href="/about">概要</a></li>
  </ul>
</nav>
```

> 参照: [Menus & Menu Buttons — Adrian Roselli](https://adrianroselli.com/2019/06/link-disclosure-widget-navigation.html)

## メニュー外クリックでの閉じ方

- メニュー外のクリックやフォーカス移動でメニューを閉じる処理は必須
- 実装方法は複数ある

```js
// 方法1: document全体のclickイベントで検知
document.addEventListener('click', (event) => {
  if (!menuContainer.contains(event.target)) {
    closeMenu();
  }
});

// 方法2: focusoutイベントで検知
menuContainer.addEventListener('focusout', (event) => {
  // フォーカスがメニュー外に移動したか確認
  if (!menuContainer.contains(event.relatedTarget)) {
    closeMenu();
  }
});
```

## Popover API との組み合わせ

- HTML Popover API を使うと、メニュー外クリックでの閉じ処理やトップレイヤー表示が自動化される
- `popover` 属性と `popovertarget` 属性で宣言的に実装可能

```html
<button popovertarget="action-menu" aria-haspopup="true">
  操作 ▾
</button>

<div id="action-menu" popover>
  <ul role="menu">
    <li role="menuitem" tabindex="-1">編集</li>
    <li role="menuitem" tabindex="-1">削除</li>
  </ul>
</div>
```

- ただしARIAロール、キーボード操作、フォーカス管理は引き続き自前で実装が必要

> 参照: [MDN — Popover API](https://developer.mozilla.org/ja/docs/Web/API/Popover_API)

## よくある間違い

### 1. ナビゲーションに role="menu" を使う

- `role="menu"` を付けるとスクリーンリーダーがアプリケーションメニューとして扱い、通常のブラウジング操作が効かなくなる
- ナビゲーションリンクには `<nav>` + 通常のリストを使う

### 2. aria-expanded を更新しない

```html
<!-- 間違い：開閉状態がスクリーンリーダーに伝わらない -->
<button>メニュー</button>

<!-- 正解：aria-expandedを開閉に連動させる -->
<button aria-expanded="false" aria-haspopup="true">メニュー</button>
```

### 3. Escapeキーで閉じない / フォーカスを戻さない

- Escapeキーでメニューを閉じたら、必ずトリガーボタンにフォーカスを戻す
- これがないとキーボードユーザーがフォーカスを見失う

### 4. 矢印キーのナビゲーションがない

- `role="menu"` を使う場合、矢印キーでの項目間移動は必須
- Tabキーだけではmenuロールの期待する操作モデルと一致しない

## まとめ

- ドロップダウンは用途によってパターンが異なる — ナビゲーション / アクション / 選択
- ナビゲーションには Disclosure パターン（`role="menu"` を使わない）
- アクション一覧には Menu Button パターン（`role="menu"` + 矢印キー操作）
- `aria-expanded`, `aria-haspopup` でメニューの状態をスクリーンリーダーに伝える
- Escapeで閉じ、フォーカスをトリガーに戻す
