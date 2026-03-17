# フォーカス管理 — tabindex, focus(), フォーカストラップ

## フォーカス管理とは

- ユーザーの現在位置（フォーカス）をプログラムで適切に制御すること
- SPA（シングルページアプリケーション）やモーダルなど、動的なUIで特に重要
- フォーカスが迷子になるとキーボードユーザーやスクリーンリーダーユーザーが操作不能に陥る

## tabindex 属性

- HTML要素のフォーカス可否とTab順序を制御する属性
- 3つの値を使い分ける

### tabindex="0"

- 要素をフォーカス可能にする
- Tabキーの順序はDOMの出現順に従う
- 非インタラクティブ要素をフォーカス可能にしたいときに使用

```html
<!-- divをフォーカス可能にする -->
<div tabindex="0" role="button" onclick="handleClick()">カスタムボタン</div>

<!-- ただしbutton要素を使うべき -->
<button onclick="handleClick()">ボタン</button>
```

### tabindex="-1"

- 要素をTabキーでのフォーカス対象から外す
- ただし `element.focus()` でプログラム的にフォーカス可能
- モーダル表示時やSPA画面遷移時にフォーカスを移動させる先として使う

```html
<!-- Tab順序からは外れるが、JSでフォーカスできる -->
<h2 tabindex="-1" id="section-title">セクション見出し</h2>

<script>
  // SPA画面遷移後に見出しへフォーカスを移動
  document.getElementById('section-title').focus();
</script>
```

### tabindex="1" 以上（正の値）

- Tabキーの順序を強制的に変更する
- **使用を避けるべき** — フォーカス順序が予測不能になる
- DOM順で管理する方が保守性が高い

```html
<!-- 悪い例：tabindexの正の値でTab順序を操作 -->
<input tabindex="3" placeholder="3番目" />
<input tabindex="1" placeholder="1番目" />
<input tabindex="2" placeholder="2番目" />

<!-- 良い例：DOM順序でTab順序を制御 -->
<input placeholder="1番目" />
<input placeholder="2番目" />
<input placeholder="3番目" />
```

> 参照: [MDN — tabindex](https://developer.mozilla.org/ja/docs/Web/HTML/Global_attributes/tabindex)

## focus() メソッド

- 任意の要素にプログラム的にフォーカスを移動させるメソッド
- `tabindex="-1"` と組み合わせて使用することが多い

### 基本的な使い方

```js
// 要素にフォーカスを移動
element.focus();

// スクロールせずにフォーカスを移動（画面が飛ぶのを防ぐ）
element.focus({ preventScroll: true });
```

### フォーカスを移動すべき場面

| 場面 | フォーカス先 |
|---|---|
| モーダルを開いた時 | モーダル内の最初のフォーカス可能な要素、またはモーダル自体 |
| モーダルを閉じた時 | モーダルを開いたトリガー要素 |
| SPA画面遷移時 | 遷移先ページの見出しやメインコンテンツ |
| インラインエラー表示時 | エラーメッセージまたはエラーのある入力欄 |
| 要素の削除後 | 削除された要素の前後にある要素 |
| アコーディオンを開いた時 | 展開されたパネル内のコンテンツ |

### モーダル表示時のフォーカス移動

```js
const modal = document.getElementById('modal');
const triggerButton = document.getElementById('open-button');

function openModal() {
  modal.hidden = false;
  // モーダル内の最初のフォーカス可能な要素にフォーカス
  const firstFocusable = modal.querySelector(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  firstFocusable?.focus();
}

function closeModal() {
  modal.hidden = true;
  // トリガーボタンにフォーカスを戻す
  triggerButton.focus();
}
```

### SPA画面遷移時のフォーカス管理

- ページ遷移時、フォーカスが前のページの要素に残ってしまう問題
- 遷移先のメインコンテンツや見出しにフォーカスを移動させる

```js
// React Router での例
import { useEffect, useRef } from 'react';
import { useLocation } from 'react-router-dom';

function PageLayout({ children }) {
  const mainRef = useRef(null);
  const location = useLocation();

  useEffect(() => {
    // 画面遷移後にmain要素にフォーカスを移動
    mainRef.current?.focus();
  }, [location.pathname]);

  return (
    <main ref={mainRef} tabIndex={-1}>
      {children}
    </main>
  );
}
```

> 参照: [WAI — Managing Focus in Single Page Applications](https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/)

### 要素削除後のフォーカス管理

- リストからアイテムを削除した場合、フォーカスが消失する
- 削除後に適切な要素へフォーカスを移す必要がある

```js
function deleteItem(index, list) {
  list.splice(index, 1);
  renderList();

  // 削除後、次のアイテムまたは前のアイテムにフォーカス
  const items = document.querySelectorAll('.list-item');
  if (items.length === 0) {
    // リストが空になった場合、リストの見出しなどにフォーカス
    document.getElementById('list-heading').focus();
  } else {
    const nextIndex = Math.min(index, items.length - 1);
    items[nextIndex].focus();
  }
}
```

## フォーカストラップ

- 特定の領域内にフォーカスを閉じ込めるテクニック
- モーダルダイアログで必須 — モーダルが開いている間、背景のコンテンツにフォーカスが移動してはいけない
- WCAG 2.4.3 Focus Order に関連

### フォーカストラップの実装

```js
function trapFocus(container) {
  const focusableElements = container.querySelectorAll(
    'a[href], button:not([disabled]), input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])'
  );
  const firstFocusable = focusableElements[0];
  const lastFocusable = focusableElements[focusableElements.length - 1];

  container.addEventListener('keydown', (event) => {
    if (event.key !== 'Tab') return;

    if (event.shiftKey) {
      // Shift + Tab：最初の要素から逆方向に出ようとしたら最後へ
      if (document.activeElement === firstFocusable) {
        event.preventDefault();
        lastFocusable.focus();
      }
    } else {
      // Tab：最後の要素から順方向に出ようとしたら最初へ
      if (document.activeElement === lastFocusable) {
        event.preventDefault();
        firstFocusable.focus();
      }
    }
  });
}
```

### HTML `<dialog>` 要素によるフォーカストラップ

- `<dialog>` 要素の `showModal()` メソッドはフォーカストラップを自動で提供する
- 自前実装よりも `<dialog>` の使用が推奨

```html
<button id="open-btn">ダイアログを開く</button>

<dialog id="my-dialog">
  <h2>確認</h2>
  <p>本当に削除しますか？</p>
  <button id="cancel-btn">キャンセル</button>
  <button id="confirm-btn">削除する</button>
</dialog>

<script>
  const dialog = document.getElementById('my-dialog');
  const openBtn = document.getElementById('open-btn');
  const cancelBtn = document.getElementById('cancel-btn');

  openBtn.addEventListener('click', () => {
    dialog.showModal(); // フォーカストラップが自動適用
  });

  cancelBtn.addEventListener('click', () => {
    dialog.close();
    openBtn.focus(); // トリガーにフォーカスを戻す
  });
</script>
```

> 参照: [MDN — dialog 要素](https://developer.mozilla.org/ja/docs/Web/HTML/Element/dialog)

### inert 属性

- `inert` 属性を使うと、指定した要素とその子孫をフォーカス不可・クリック不可・スクリーンリーダーから隠す
- モーダルの背景を無効化するのに使える

```html
<!-- モーダルが開いている間、背景コンテンツをinertにする -->
<div id="app" inert>
  <header>...</header>
  <main>...</main>
</div>

<dialog id="modal" open>
  <h2>モーダルの内容</h2>
  <button>閉じる</button>
</dialog>
```

```js
function openModal() {
  document.getElementById('app').inert = true;
  document.getElementById('modal').showModal();
}

function closeModal() {
  document.getElementById('app').inert = false;
  document.getElementById('modal').close();
}
```

> 参照: [MDN — inert](https://developer.mozilla.org/ja/docs/Web/HTML/Global_attributes/inert)

## よくある間違い

### 1. tabindex の正の値を使う

```html
<!-- 間違い：Tab順序が予測不能になる -->
<input tabindex="5" />
<input tabindex="1" />
<input tabindex="10" />

<!-- 正解：DOM順序で管理する -->
<input />
<input />
<input />
```

### 2. モーダルの背景にフォーカスが漏れる

```js
// 間違い：フォーカストラップなしでモーダルを表示
function openModal() {
  modal.style.display = 'block';
  // Tabキーで背景の要素にフォーカスが移動してしまう
}

// 正解：dialog.showModal() を使う
function openModal() {
  dialog.showModal(); // 自動でフォーカストラップ
}
```

### 3. フォーカスの戻し忘れ

```js
// 間違い：モーダルを閉じた後フォーカスがbodyに飛ぶ
function closeModal() {
  modal.hidden = true;
}

// 正解：トリガー要素にフォーカスを戻す
function closeModal() {
  modal.hidden = true;
  triggerButton.focus();
}
```

### 4. SPA遷移後にフォーカスを放置する

- ページ遷移後、前のページのボタンにフォーカスが残っている
- スクリーンリーダーユーザーは新しいページに遷移したことに気づけない
- 遷移先の見出しやメインコンテンツへフォーカスを移動させる

## まとめ

- `tabindex="0"` で要素をフォーカス可能に、`tabindex="-1"` でプログラム的フォーカスのみ可能に
- `tabindex` の正の値は使わない — DOM順序で管理する
- `focus()` メソッドでフォーカスを適切な場所に移動させる
- モーダルではフォーカストラップが必須 — `<dialog>` 要素の `showModal()` が最も簡単
- `inert` 属性でモーダル背景を無効化できる
- フォーカスの戻し忘れに注意 — モーダルを閉じたらトリガーに戻す
