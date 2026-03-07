# aria-expanded / aria-hidden / aria-controls の活用

## 3つの属性の概要

| 属性 | 役割 | 値の形式 |
|---|---|---|
| `aria-expanded` | 要素が制御する領域の開閉状態を示す | `true` / `false` |
| `aria-hidden` | 要素をアクセシビリティツリーから除外する | `true` / `false` |
| `aria-controls` | 要素が制御している別の要素を示す | 要素のID(スペース区切りで複数可) |

- これらはUIの「表示/非表示」「開閉」パターンに頻出する属性
- アコーディオン、ドロップダウン、モーダルなど、インタラクティブなコンポーネントで組み合わせて使用

## aria-expanded

### 基本

- トグルボタンやアコーディオンのヘッダーなど、開閉を制御する要素に付与する属性
- 制御対象の領域が展開されているかどうかをスクリーンリーダーに伝える
- `true`：展開されている状態
- `false`：折りたたまれている状態
- 属性がない場合：要素に開閉の概念がないことを意味する

```html
<!-- アコーディオンの開閉ボタン -->
<button aria-expanded="false">よくある質問</button>
<div hidden>
  <p>回答の内容がここに入ります。</p>
</div>
```

- スクリーンリーダーの読み上げ例：「よくある質問 折りたたみ ボタン」
- `aria-expanded="true"` の場合：「よくある質問 展開済み ボタン」

- 参照: [WAI-ARIA 1.2 — aria-expanded](https://www.w3.org/TR/wai-aria-1.2/#aria-expanded)

### aria-expandedを付ける要素

- **制御する側の要素**（ボタン、リンクなど）に付与する
- 制御される側（展開されるパネル）には付けない

```html
<!-- 良い例：ボタンに付与 -->
<button aria-expanded="false">メニュー</button>
<ul hidden>
  <li><a href="/">ホーム</a></li>
  <li><a href="/about">会社概要</a></li>
</ul>

<!-- 悪い例：展開される側に付与 -->
<button>メニュー</button>
<ul aria-expanded="false" hidden>
  <li><a href="/">ホーム</a></li>
</ul>
```

### JavaScriptによる状態の切り替え

```html
<button id="toggle-btn" aria-expanded="false" aria-controls="panel">
  詳細を表示
</button>
<div id="panel" hidden>
  <p>詳細な内容がここに入ります。</p>
</div>

<script>
  const btn = document.getElementById('toggle-btn');
  const panel = document.getElementById('panel');

  btn.addEventListener('click', () => {
    const isExpanded = btn.getAttribute('aria-expanded') === 'true';
    btn.setAttribute('aria-expanded', String(!isExpanded));
    panel.hidden = isExpanded;
  });
</script>
```

## aria-hidden

### 基本

- 要素とその子孫すべてをアクセシビリティツリーから除外する属性
- `true`：支援技術から完全に隠される（スクリーンリーダーが読み上げない）
- `false`：明示的にアクセシビリティツリーに含める（通常は指定不要）
- 視覚的な表示には影響しない（CSSの`display: none`とは別物）

```html
<!-- 装飾用アイコンを支援技術から隠す -->
<button>
  <span aria-hidden="true">🔍</span>
  検索
</button>
<!-- スクリーンリーダー：「検索 ボタン」（🔍は読み上げない） -->
```

- 参照: [WAI-ARIA 1.2 — aria-hidden](https://www.w3.org/TR/wai-aria-1.2/#aria-hidden)

### aria-hidden="true" と display: none の違い

| 方法 | 視覚的に見える | スクリーンリーダーが読み上げる |
|---|---|---|
| `aria-hidden="true"` | 見える | 読み上げない |
| `display: none` | 見えない | 読み上げない |
| `visibility: hidden` | 見えない（スペースは残る） | 読み上げない |
| `hidden`属性 | 見えない | 読み上げない |

- `aria-hidden="true"`は**見えているが支援技術には不要な要素**に使う
- `display: none` / `hidden`属性は視覚的にもアクセシビリティ的にも隠す

### 使いどころ

- 装飾的なアイコンやSVG（テキストラベルが別にある場合）
- 視覚的な重複コンテンツ（同じ情報がテキストでも提供されている場合）
- モーダル表示時の背景コンテンツ

```html
<!-- 装飾アイコン：テキストラベルがあるので読み上げ不要 -->
<a href="/home">
  <svg aria-hidden="true" focusable="false"><!-- ホームアイコン --></svg>
  ホーム
</a>

<!-- 視覚的な区切り線 -->
<span aria-hidden="true">|</span>

<!-- モーダル表示時：背景コンテンツを支援技術から隠す -->
<div aria-hidden="true" id="main-content">
  <!-- メインコンテンツ -->
</div>
<dialog open aria-labelledby="dialog-title">
  <h2 id="dialog-title">確認</h2>
  <p>本当に削除しますか？</p>
</dialog>
```

### 絶対にやってはいけないこと

- **フォーカス可能な要素に`aria-hidden="true"`を付けない**
- 支援技術からは見えないがフォーカスは移動する、という矛盾した状態になる
- WCAG違反（4.1.2 名前・役割・値）

```html
<!-- 悪い例：フォーカス可能な要素を隠す -->
<button aria-hidden="true">送信</button>
<!-- Tabキーでフォーカスは移動するが、スクリーンリーダーは何も読み上げない -->

<!-- 悪い例：aria-hiddenの中にフォーカス可能な要素がある -->
<div aria-hidden="true">
  <a href="/help">ヘルプ</a>
  <button>閉じる</button>
</div>
```

## aria-controls

### 基本

- ある要素が別の要素を「制御している」という関係性を示す属性
- 値は制御対象の要素のID
- スクリーンリーダーによっては、制御先への移動をサポート（JAWSなど）

```html
<button aria-controls="nav-menu" aria-expanded="false">
  メニュー
</button>
<nav id="nav-menu" hidden>
  <ul>
    <li><a href="/">ホーム</a></li>
    <li><a href="/about">会社概要</a></li>
  </ul>
</nav>
```

- 参照: [WAI-ARIA 1.2 — aria-controls](https://www.w3.org/TR/wai-aria-1.2/#aria-controls)

### aria-controlsの現状

- WAI-ARIAの仕様上は定義されているが、**支援技術のサポートが限定的**
- JAWSは`aria-controls`を認識するが、NVDAやVoiceOverは現状ほぼ無視する
- そのため、aria-controlsは「あれば良い」程度の位置付け
- **aria-controlsに頼らず、aria-expandedやフォーカス管理で操作性を確保する**

```html
<!-- aria-controlsはあくまで補助的に使用 -->
<button
  aria-expanded="false"
  aria-controls="details-panel"
>
  詳細を表示
</button>
<div id="details-panel" hidden>
  <p>詳細情報...</p>
</div>
<!-- aria-expandedだけでも十分に機能する -->
```

- 参照: [a11ysupport.io — aria-controls](https://a11ysupport.io/tech/aria/aria-controls_attribute)

## 3つの属性の組み合わせパターン

### パターン1: アコーディオン

```html
<div class="accordion">
  <h3>
    <button
      aria-expanded="false"
      aria-controls="accordion-panel-1"
    >
      配送について
    </button>
  </h3>
  <div id="accordion-panel-1" role="region" aria-labelledby="accordion-header-1" hidden>
    <p>配送は通常3〜5営業日以内にお届けします。</p>
  </div>
</div>

<script>
  document.querySelectorAll('.accordion button').forEach(btn => {
    btn.addEventListener('click', () => {
      const isExpanded = btn.getAttribute('aria-expanded') === 'true';
      const panelId = btn.getAttribute('aria-controls');
      const panel = document.getElementById(panelId);

      btn.setAttribute('aria-expanded', String(!isExpanded));
      panel.hidden = isExpanded;
    });
  });
</script>
```

- 参照: [ARIA APG — Accordion Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/accordion/)

### パターン2: ドロップダウンメニュー

```html
<div class="dropdown">
  <button
    aria-expanded="false"
    aria-controls="dropdown-menu"
    aria-haspopup="true"
  >
    設定
  </button>
  <ul id="dropdown-menu" role="menu" hidden>
    <li role="menuitem"><a href="/profile">プロフィール</a></li>
    <li role="menuitem"><a href="/settings">アカウント設定</a></li>
    <li role="menuitem"><button>ログアウト</button></li>
  </ul>
</div>

<script>
  const trigger = document.querySelector('.dropdown button');
  const menu = document.getElementById('dropdown-menu');

  trigger.addEventListener('click', () => {
    const isExpanded = trigger.getAttribute('aria-expanded') === 'true';
    trigger.setAttribute('aria-expanded', String(!isExpanded));
    menu.hidden = isExpanded;

    // メニューを開いたら最初の項目にフォーカス
    if (!isExpanded) {
      menu.querySelector('[role="menuitem"]').focus();
    }
  });

  // Escapeキーで閉じる
  menu.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') {
      trigger.setAttribute('aria-expanded', 'false');
      menu.hidden = true;
      trigger.focus();
    }
  });
</script>
```

- 参照: [ARIA APG — Menu Button Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/)

### パターン3: モーダルダイアログ（aria-hiddenで背景を隠す）

```html
<div id="app-root">
  <main id="main-content">
    <h1>ページ本文</h1>
    <button id="open-modal">削除する</button>
  </main>
</div>

<dialog id="confirm-dialog" aria-labelledby="dialog-title" aria-describedby="dialog-desc">
  <h2 id="dialog-title">削除の確認</h2>
  <p id="dialog-desc">この操作は取り消せません。本当に削除しますか？</p>
  <button id="confirm-delete">削除する</button>
  <button id="cancel-delete">キャンセル</button>
</dialog>

<script>
  const openBtn = document.getElementById('open-modal');
  const dialog = document.getElementById('confirm-dialog');
  const mainContent = document.getElementById('main-content');

  openBtn.addEventListener('click', () => {
    // 背景コンテンツをアクセシビリティツリーから隠す
    mainContent.setAttribute('aria-hidden', 'true');
    dialog.showModal();
  });

  function closeDialog() {
    mainContent.removeAttribute('aria-hidden');
    dialog.close();
    openBtn.focus();
  }

  document.getElementById('cancel-delete').addEventListener('click', closeDialog);
  document.getElementById('confirm-delete').addEventListener('click', () => {
    // 削除処理
    closeDialog();
  });
</script>
```

- `<dialog>`要素の`showModal()`を使用する場合、ブラウザが自動的にinert処理を行うため`aria-hidden`は不要な場合もある
- ただし古いブラウザや`show()`を使う場合は明示的に`aria-hidden`を付ける必要がある

### パターン4: タブUI

```html
<div class="tabs">
  <div role="tablist" aria-label="商品情報">
    <button
      role="tab"
      aria-selected="true"
      aria-controls="tab-panel-1"
      id="tab-1"
    >
      概要
    </button>
    <button
      role="tab"
      aria-selected="false"
      aria-controls="tab-panel-2"
      id="tab-2"
      tabindex="-1"
    >
      スペック
    </button>
    <button
      role="tab"
      aria-selected="false"
      aria-controls="tab-panel-3"
      id="tab-3"
      tabindex="-1"
    >
      レビュー
    </button>
  </div>

  <div role="tabpanel" id="tab-panel-1" aria-labelledby="tab-1">
    <p>商品の概要です。</p>
  </div>
  <div role="tabpanel" id="tab-panel-2" aria-labelledby="tab-2" hidden>
    <p>スペック情報です。</p>
  </div>
  <div role="tabpanel" id="tab-panel-3" aria-labelledby="tab-3" hidden>
    <p>レビュー一覧です。</p>
  </div>
</div>
```

- タブUIでは`aria-expanded`ではなく`aria-selected`を使用する
- `aria-controls`でタブとパネルの関係を明示

- 参照: [ARIA APG — Tabs Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/)

## Reactでの使用例

### アコーディオンコンポーネント

```jsx
import { useState } from 'react';

function Accordion({ title, children, id }) {
  const [isExpanded, setIsExpanded] = useState(false);
  const panelId = `${id}-panel`;

  return (
    <div>
      <h3>
        <button
          aria-expanded={isExpanded}
          aria-controls={panelId}
          onClick={() => setIsExpanded(!isExpanded)}
        >
          {title}
        </button>
      </h3>
      <div id={panelId} role="region" hidden={!isExpanded}>
        {children}
      </div>
    </div>
  );
}
```

### アイコンボタン

```jsx
function IconButton({ icon, label, onClick }) {
  return (
    <button onClick={onClick} aria-label={label}>
      <span aria-hidden="true">{icon}</span>
    </button>
  );
}

// 使用例
<IconButton icon="🗑️" label="削除" onClick={handleDelete} />
// スクリーンリーダー：「削除 ボタン」
```

### ドロップダウン

```jsx
import { useState, useRef, useEffect } from 'react';

function Dropdown({ label, items }) {
  const [isOpen, setIsOpen] = useState(false);
  const menuRef = useRef(null);
  const triggerRef = useRef(null);

  useEffect(() => {
    if (isOpen && menuRef.current) {
      menuRef.current.querySelector('[role="menuitem"]')?.focus();
    }
  }, [isOpen]);

  const handleKeyDown = (e) => {
    if (e.key === 'Escape') {
      setIsOpen(false);
      triggerRef.current?.focus();
    }
  };

  return (
    <div>
      <button
        ref={triggerRef}
        aria-expanded={isOpen}
        aria-controls="dropdown-menu"
        aria-haspopup="true"
        onClick={() => setIsOpen(!isOpen)}
      >
        {label}
      </button>
      {isOpen && (
        <ul
          ref={menuRef}
          id="dropdown-menu"
          role="menu"
          onKeyDown={handleKeyDown}
        >
          {items.map((item, i) => (
            <li key={i} role="menuitem" tabIndex={-1}>
              {item}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## よくある間違い

### 1. aria-expandedの値を属性の有無で切り替える

```html
<!-- 悪い例：属性を削除してしまう -->
<script>
  // 閉じる時にaria-expandedを削除
  btn.removeAttribute('aria-expanded');
  // → 「開閉する要素ではない」という意味になってしまう
</script>

<!-- 良い例：true/falseを切り替える -->
<script>
  btn.setAttribute('aria-expanded', 'false');
  // → 「閉じている」という状態が明確に伝わる
</script>
```

### 2. aria-hidden="true"をフォーカス可能な要素に使う

```html
<!-- 悪い例：ボタンをaria-hiddenで隠す -->
<div aria-hidden="true">
  <button>操作ボタン</button>
  <input type="text" placeholder="入力欄">
</div>
<!-- フォーカスは移動するが、スクリーンリーダーは何も読み上げない -->

<!-- 良い例：本当に非表示にするならhidden属性やdisplay:noneを使う -->
<div hidden>
  <button>操作ボタン</button>
  <input type="text" placeholder="入力欄">
</div>
<!-- フォーカスも移動しないし、読み上げもされない -->
```

### 3. aria-hidden="false"を明示的に使って非表示要素を表示しようとする

```html
<!-- 悪い例：display:noneの要素をaria-hiddenで見せようとする -->
<div style="display: none" aria-hidden="false">
  この内容は見えないが読み上げてほしい
</div>
<!-- display:noneの要素はaria-hidden="false"でも読み上げられない -->

<!-- 良い例：視覚的に隠すがスクリーンリーダーには見せたい場合 -->
<div class="sr-only">
  この内容はスクリーンリーダーのみ読み上げる
</div>

<style>
  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
  }
</style>
```

### 4. 装飾アイコンにaria-hiddenを付け忘れる

```html
<!-- 悪い例：アイコンフォントがそのまま読み上げられる -->
<button>
  <i class="icon-search"></i>
  検索
</button>
<!-- スクリーンリーダーによっては意味不明な文字が読み上げられる -->

<!-- 良い例：装飾アイコンを隠す -->
<button>
  <i class="icon-search" aria-hidden="true"></i>
  検索
</button>
```

### 5. aria-controlsだけに頼る

```html
<!-- 悪い例：aria-controlsだけで制御関係を示す -->
<button aria-controls="panel">パネルを開く</button>
<div id="panel" hidden>パネルの内容</div>
<!-- 多くのスクリーンリーダーはaria-controlsを読み上げない -->

<!-- 良い例：aria-expandedも併用する -->
<button aria-expanded="false" aria-controls="panel">パネルを開く</button>
<div id="panel" hidden>パネルの内容</div>
```

## 属性の使い分け判断フローチャート

```
UIに開閉・表示切替の仕組みがある？
  │
  ├── YES：トグルボタン・アコーディオン・ドロップダウンなど
  │     └── 制御するボタンに aria-expanded を付与
  │         └── 制御先の要素と紐付けるなら aria-controls も付与（補助的）
  │
  └── 要素を支援技術から隠したい？
        │
        ├── 視覚的にも隠す → hidden属性 / display: none を使う
        │
        └── 視覚的には見せるが読み上げ不要 → aria-hidden="true"
              例：装飾アイコン、視覚的な区切り、重複するテキスト
```

## まとめ

- `aria-expanded`は**制御する側の要素**に付与し、開閉状態を`true`/`false`で切り替える（属性の削除はしない）
- `aria-hidden="true"`は視覚的に表示されているが支援技術に不要な要素に使用する（装飾アイコンなど）
- `aria-hidden="true"`の配下にフォーカス可能な要素を置かない
- `aria-controls`は補助的な属性であり、支援技術のサポートが限定的なため、単独で頼らない
- 要素を完全に非表示にしたい場合は`hidden`属性や`display: none`を使い、`aria-hidden`は使わない
- アコーディオン、ドロップダウン、モーダルなどでは、これらの属性を組み合わせてアクセシブルなUIを構築する
