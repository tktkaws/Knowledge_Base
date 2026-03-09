# アクセシブルなモーダルダイアログの実装

## モーダルダイアログとは

- ユーザーの操作を一時的にブロックし、特定のタスクに集中させるUI
- 確認ダイアログ、フォーム入力、警告表示などに使用
- モーダルが開いている間、背景のコンテンツは操作不可にする必要がある

## アクセシビリティ上の要件

- モーダルが開いたらフォーカスをモーダル内に移動する
- モーダル内にフォーカスを閉じ込める（フォーカストラップ）
- Escapeキーでモーダルを閉じられる
- モーダルを閉じたらフォーカスをトリガー要素に戻す
- 背景コンテンツをスクリーンリーダーから隠す
- `role="dialog"` と `aria-modal="true"` を付与する
- `aria-labelledby` または `aria-label` でモーダルに名前を付ける

> 参照: [WAI-ARIA Authoring Practices — Dialog (Modal) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

## `<dialog>` 要素を使った実装（推奨）

- HTML標準の `<dialog>` 要素はモーダルダイアログの要件を多くカバーしている
- `showModal()` メソッドでフォーカストラップと背景の無効化が自動適用される
- 全モダンブラウザでサポート済み

```html
<button id="open-btn" type="button">削除する</button>

<dialog id="confirm-dialog" aria-labelledby="dialog-title">
  <h2 id="dialog-title">削除の確認</h2>
  <p>この操作は取り消せません。本当に削除しますか？</p>
  <div class="dialog-actions">
    <button id="cancel-btn" type="button">キャンセル</button>
    <button id="delete-btn" type="button">削除する</button>
  </div>
</dialog>
```

```js
const dialog = document.getElementById('confirm-dialog');
const openBtn = document.getElementById('open-btn');
const cancelBtn = document.getElementById('cancel-btn');
const deleteBtn = document.getElementById('delete-btn');

openBtn.addEventListener('click', () => {
  dialog.showModal();
});

cancelBtn.addEventListener('click', () => {
  dialog.close();
  openBtn.focus(); // トリガーにフォーカスを戻す
});

deleteBtn.addEventListener('click', () => {
  performDelete();
  dialog.close();
  openBtn.focus();
});
```

```css
/* ::backdrop 疑似要素で背景のオーバーレイをスタイリング */
dialog::backdrop {
  background: rgba(0, 0, 0, 0.5);
}

dialog {
  border: none;
  border-radius: 8px;
  padding: 24px;
  max-width: 480px;
  width: 90%;
}
```

### `<dialog>` が自動で行うこと

| 機能 | 対応状況 |
|---|---|
| フォーカストラップ | `showModal()` で自動 |
| Escapeキーで閉じる | 自動（`close` イベントが発火） |
| 背景のスクロール防止 | 自動 |
| `role="dialog"` | 暗黙のロールとして付与済み |
| `aria-modal="true"` | `showModal()` 使用時に自動 |
| 背景を `inert` にする | `showModal()` で自動 |

### 自前で対応が必要なこと

- フォーカスの戻し先の管理（`close` 後にトリガーへ `focus()`）
- `aria-labelledby` または `aria-label` の付与
- 初期フォーカス先のカスタマイズ（必要な場合）

### 初期フォーカスの制御

- `showModal()` はデフォルトでダイアログ内の最初のフォーカス可能な要素にフォーカスする
- `autofocus` 属性で初期フォーカス先を指定できる

```html
<dialog id="form-dialog" aria-labelledby="form-title">
  <h2 id="form-title">プロフィール編集</h2>
  <label for="name">名前</label>
  <!-- autofocus で初期フォーカスを指定 -->
  <input id="name" type="text" autofocus />
  <button type="button">保存</button>
  <button type="button">キャンセル</button>
</dialog>
```

### Escapeキーのカスタマイズ

- `<dialog>` はEscapeキーで自動的に閉じるが、`cancel` イベントで制御可能
- フォームの未保存データがある場合に確認を挟む例

```js
dialog.addEventListener('cancel', (event) => {
  if (hasUnsavedChanges()) {
    event.preventDefault(); // Escapeで閉じるのを防止
    showUnsavedWarning();
  }
});

// close イベントでフォーカスを戻す
dialog.addEventListener('close', () => {
  openBtn.focus();
});
```

## `<div>` でモーダルを自作する場合

- `<dialog>` を使えない場合のフォールバック実装
- 全ての要件を自前で満たす必要がある

```html
<div
  id="custom-modal"
  role="dialog"
  aria-modal="true"
  aria-labelledby="modal-title"
  hidden
>
  <h2 id="modal-title">確認</h2>
  <p>内容を保存しますか？</p>
  <button id="save-btn">保存</button>
  <button id="discard-btn">破棄</button>
</div>
<div id="overlay" hidden></div>
```

```js
const modal = document.getElementById('custom-modal');
const overlay = document.getElementById('overlay');
let triggerElement = null;

function openModal(trigger) {
  triggerElement = trigger;
  modal.hidden = false;
  overlay.hidden = false;

  // 背景をinertにする
  document.getElementById('app').inert = true;

  // 背景のスクロールを防止
  document.body.style.overflow = 'hidden';

  // フォーカスをモーダル内の最初のフォーカス可能要素に移動
  const firstFocusable = modal.querySelector(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  firstFocusable?.focus();

  // フォーカストラップを設定
  modal.addEventListener('keydown', trapFocus);
  // Escapeキーで閉じる
  modal.addEventListener('keydown', handleEscape);
}

function closeModal() {
  modal.hidden = true;
  overlay.hidden = true;

  document.getElementById('app').inert = false;
  document.body.style.overflow = '';

  modal.removeEventListener('keydown', trapFocus);
  modal.removeEventListener('keydown', handleEscape);

  // フォーカスをトリガーに戻す
  triggerElement?.focus();
}

function handleEscape(event) {
  if (event.key === 'Escape') {
    closeModal();
  }
}

function trapFocus(event) {
  if (event.key !== 'Tab') return;

  const focusableElements = modal.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const first = focusableElements[0];
  const last = focusableElements[focusableElements.length - 1];

  if (event.shiftKey && document.activeElement === first) {
    event.preventDefault();
    last.focus();
  } else if (!event.shiftKey && document.activeElement === last) {
    event.preventDefault();
    first.focus();
  }
}
```

## React での実装例

```jsx
import { useEffect, useRef, useCallback } from 'react';

function Modal({ isOpen, onClose, title, children }) {
  const dialogRef = useRef(null);
  const triggerRef = useRef(null);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (!dialog) return;

    if (isOpen) {
      // 開く前にトリガーを記憶
      triggerRef.current = document.activeElement;
      dialog.showModal();
    } else if (dialog.open) {
      dialog.close();
      // トリガーにフォーカスを戻す
      triggerRef.current?.focus();
    }
  }, [isOpen]);

  const handleClose = useCallback(() => {
    onClose();
  }, [onClose]);

  return (
    <dialog
      ref={dialogRef}
      aria-labelledby="modal-title"
      onCancel={handleClose}
    >
      <h2 id="modal-title">{title}</h2>
      {children}
      <button type="button" onClick={handleClose}>
        閉じる
      </button>
    </dialog>
  );
}
```

## alertdialog ロール

- ユーザーに重要な警告を伝え、確認を求めるダイアログには `role="alertdialog"` を使用
- `role="dialog"` との違いは、スクリーンリーダーがより緊急度の高い通知として扱う点
- データの削除確認、未保存データの警告などに適用

```html
<dialog role="alertdialog" aria-labelledby="alert-title" aria-describedby="alert-desc">
  <h2 id="alert-title">データの削除</h2>
  <p id="alert-desc">この操作は取り消せません。全てのデータが永久に削除されます。</p>
  <button>キャンセル</button>
  <button>削除する</button>
</dialog>
```

> 参照: [WAI-ARIA — alertdialog role](https://www.w3.org/WAI/ARIA/apg/patterns/alertdialog/)

## 非モーダルダイアログ

- `show()` メソッドで開くダイアログは非モーダル（背景を操作可能）
- フォーカストラップは適用されない
- トーストやツールチップ的な用途に適する

```js
// 非モーダル（背景操作可能）
dialog.show();

// モーダル（背景操作不可）
dialog.showModal();
```

## よくある間違い

### 1. フォーカスを戻し忘れる

```js
// 間違い：閉じた後フォーカスがbodyに飛ぶ
function closeModal() {
  dialog.close();
}

// 正解：トリガーにフォーカスを戻す
function closeModal() {
  dialog.close();
  triggerButton.focus();
}
```

### 2. aria-labelledby を付けない

```html
<!-- 間違い：ダイアログに名前がない -->
<dialog>
  <h2>確認</h2>
</dialog>

<!-- 正解：見出しをaria-labelledbyで参照 -->
<dialog aria-labelledby="dialog-title">
  <h2 id="dialog-title">確認</h2>
</dialog>
```

### 3. オーバーレイのクリックで閉じるがキーボードで閉じられない

- マウスでオーバーレイ（背景）をクリックして閉じる機能を付ける場合、Escapeキーでも閉じられるようにする
- キーボードユーザーはオーバーレイをクリックできない

### 4. モーダル内にフォーカス可能な要素がない

```html
<!-- 間違い：フォーカスが当たる先がない -->
<dialog>
  <p>処理が完了しました。</p>
</dialog>

<!-- 正解：閉じるボタンを設ける -->
<dialog aria-labelledby="done-title">
  <h2 id="done-title">完了</h2>
  <p>処理が完了しました。</p>
  <button autofocus>閉じる</button>
</dialog>
```

### 5. スクロール位置が背景で動く

- モーダルが開いているとき、背景がスクロールするとユーザーが混乱する
- `<dialog>` の `showModal()` はブラウザが対応するが、カスタム実装では `body` の `overflow: hidden` が必要

## まとめ

- モーダルダイアログの実装には `<dialog>` 要素の `showModal()` を使うのが最善
- フォーカストラップ・Escape閉じ・背景の無効化が自動で適用される
- `aria-labelledby` でダイアログに名前を付ける
- 閉じた後は必ずトリガー要素にフォーカスを戻す
- 危険な操作の確認には `role="alertdialog"` を検討する
