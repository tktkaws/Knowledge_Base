# アクセシブルなトーストとアラート通知

## トースト / アラート通知とは

- ユーザーの操作や非同期処理の結果をフィードバックするUI
- 画面の端に一時的に表示され、自動または手動で消える
- 例：「保存しました」「エラーが発生しました」「新しいメッセージがあります」

## トーストとアラートの違い

| 種類 | 用途 | 緊急度 | 自動消去 |
|---|---|---|---|
| ステータスメッセージ（トースト） | 操作の成功、情報の通知 | 低い | あり（数秒後） |
| アラート | エラー、警告、重要な変更 | 高い | なし（ユーザーが閉じる） |

## ライブリージョンの仕組み

- スクリーンリーダーは通常、ユーザーがフォーカスしている要素のみ読み上げる
- ライブリージョンを使うと、DOM内のコンテンツ変更をフォーカス移動なしに通知できる
- `aria-live` 属性または `role="alert"` / `role="status"` で設定する

### aria-live の値

| 値 | 意味 | 使い所 |
|---|---|---|
| `polite` | 現在の読み上げが終わった後に通知 | ステータス、成功メッセージ |
| `assertive` | 現在の読み上げを中断して即座に通知 | エラー、緊急の警告 |
| `off` | 通知しない（デフォルト） | — |

### role による暗黙の aria-live

| ロール | 暗黙の aria-live 値 | 用途 |
|---|---|---|
| `role="status"` | `polite` | 成功通知、ステータス更新 |
| `role="alert"` | `assertive` | エラー、警告 |
| `role="log"` | `polite` | チャットログ、イベントログ |

> 参照: [WAI-ARIA Authoring Practices — Alert Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/alert/)

## 重要なルール：ライブリージョンは先にDOMに存在させる

- ライブリージョンの要素は**コンテンツが変更される前に**DOMに存在している必要がある
- 要素をDOMに追加した時点で既にテキストが入っていると、スクリーンリーダーが通知しない場合がある
- 空のライブリージョンを事前に配置し、後からテキストを挿入する

```html
<!-- 悪い例：要素ごとDOMに追加 — 通知されないことがある -->
<script>
  const alert = document.createElement('div');
  alert.setAttribute('role', 'alert');
  alert.textContent = 'エラーが発生しました';
  document.body.appendChild(alert); // 通知されない可能性
</script>

<!-- 良い例：空の要素を事前に配置し、テキストを後から挿入 -->
<div id="notification" role="status" aria-live="polite"></div>

<script>
  // テキストの変更がライブリージョンのトリガーになる
  document.getElementById('notification').textContent = '保存しました';
</script>
```

## ステータスメッセージ（トースト）の実装

### 基本構造

```html
<!-- 空のライブリージョンを事前に配置 -->
<div id="toast-container" aria-live="polite" role="status"></div>
```

```js
function showToast(message, duration = 5000) {
  const container = document.getElementById('toast-container');

  // トースト要素を作成
  const toast = document.createElement('div');
  toast.className = 'toast';
  toast.textContent = message;

  // 閉じるボタンを追加
  const closeBtn = document.createElement('button');
  closeBtn.textContent = '閉じる';
  closeBtn.setAttribute('aria-label', '通知を閉じる');
  closeBtn.addEventListener('click', () => {
    removeToast(toast);
  });
  toast.appendChild(closeBtn);

  // ライブリージョン内に追加（テキスト変更がトリガー）
  container.appendChild(toast);

  // 自動消去
  setTimeout(() => {
    removeToast(toast);
  }, duration);
}

function removeToast(toast) {
  toast.classList.add('toast-exit');
  toast.addEventListener('animationend', () => {
    toast.remove();
  });
}
```

```css
.toast {
  position: fixed;
  bottom: 16px;
  right: 16px;
  padding: 12px 16px;
  background: #333;
  color: #fff;
  border-radius: 8px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.toast-exit {
  animation: fadeOut 0.3s ease-out forwards;
}

@keyframes fadeOut {
  to {
    opacity: 0;
    transform: translateY(10px);
  }
}
```

### 自動消去の注意点

- WCAG 2.2.1「タイミング調整可能」に注意
- 自動消去のタイミングが短すぎると読み終わる前に消えてしまう
- 十分な表示時間を確保する（最低5秒、内容の長さに応じて延長）
- ホバー / フォーカス中は消去を一時停止するのが望ましい

```js
let timeoutId;

function showToast(message, duration = 5000) {
  const toast = createToast(message);

  // ホバー / フォーカスで消去を一時停止
  toast.addEventListener('mouseenter', () => clearTimeout(timeoutId));
  toast.addEventListener('focusin', () => clearTimeout(timeoutId));

  toast.addEventListener('mouseleave', () => {
    timeoutId = setTimeout(() => removeToast(toast), duration);
  });
  toast.addEventListener('focusout', () => {
    timeoutId = setTimeout(() => removeToast(toast), duration);
  });

  timeoutId = setTimeout(() => removeToast(toast), duration);
}
```

> 参照: [WCAG 2.2.1 Timing Adjustable](https://www.w3.org/WAI/WCAG22/Understanding/timing-adjustable.html)

## アラート（緊急通知）の実装

### role="alert" を使用

```html
<!-- 空のアラート領域を事前に配置 -->
<div id="alert-container" role="alert"></div>
```

```js
function showAlert(message) {
  const container = document.getElementById('alert-container');

  container.textContent = ''; // 一度クリアして再通知させる
  // 微小な遅延を入れることでスクリーンリーダーが変更を検知しやすくなる
  requestAnimationFrame(() => {
    container.textContent = message;
  });
}
```

### アラートダイアログとの使い分け

| パターン | 用途 | ユーザーの操作 |
|---|---|---|
| `role="alert"` | 情報の通知のみ | 操作不要（読むだけ） |
| `role="alertdialog"` | 確認や選択が必要 | ボタン等で応答が必要 |

```html
<!-- role="alert"：通知のみ -->
<div role="alert">セッションがまもなく切れます</div>

<!-- role="alertdialog"：応答が必要 -->
<dialog role="alertdialog" aria-labelledby="alert-title">
  <h2 id="alert-title">セッション切れ</h2>
  <p>セッションが切れました。再ログインしてください。</p>
  <button>ログインページへ</button>
</dialog>
```

## React での実装例

```jsx
import { useState, useCallback, useEffect, useRef } from 'react';

// トーストのコンテキスト & プロバイダー
function ToastProvider({ children }) {
  const [toasts, setToasts] = useState([]);
  const timers = useRef(new Map());

  const addToast = useCallback((message, type = 'info', duration = 5000) => {
    const id = crypto.randomUUID();
    setToasts((prev) => [...prev, { id, message, type }]);

    const timer = setTimeout(() => {
      removeToast(id);
    }, duration);
    timers.current.set(id, timer);

    return id;
  }, []);

  const removeToast = useCallback((id) => {
    clearTimeout(timers.current.get(id));
    timers.current.delete(id);
    setToasts((prev) => prev.filter((t) => t.id !== id));
  }, []);

  const pauseTimer = useCallback((id) => {
    clearTimeout(timers.current.get(id));
  }, []);

  const resumeTimer = useCallback((id, duration = 5000) => {
    const timer = setTimeout(() => removeToast(id), duration);
    timers.current.set(id, timer);
  }, [removeToast]);

  return (
    <>
      {children}
      {/* ステータスメッセージ用のライブリージョン */}
      <div
        aria-live="polite"
        role="status"
        className="toast-container"
      >
        {toasts.filter((t) => t.type !== 'error').map((toast) => (
          <div
            key={toast.id}
            className={`toast toast-${toast.type}`}
            onMouseEnter={() => pauseTimer(toast.id)}
            onMouseLeave={() => resumeTimer(toast.id)}
            onFocus={() => pauseTimer(toast.id)}
            onBlur={() => resumeTimer(toast.id)}
          >
            <span>{toast.message}</span>
            <button
              aria-label="通知を閉じる"
              onClick={() => removeToast(toast.id)}
            >
              ✕
            </button>
          </div>
        ))}
      </div>

      {/* エラー用のライブリージョン */}
      <div role="alert" className="toast-container">
        {toasts.filter((t) => t.type === 'error').map((toast) => (
          <div
            key={toast.id}
            className="toast toast-error"
          >
            <span>{toast.message}</span>
            <button
              aria-label="通知を閉じる"
              onClick={() => removeToast(toast.id)}
            >
              ✕
            </button>
          </div>
        ))}
      </div>
    </>
  );
}
```

## 通知の積み重ね

- 複数のトーストが同時に表示される場合の対応
- 古いトーストから順に消去する
- 最大表示数を制限して画面を圧迫しない
- スクリーンリーダーが連続して読み上げることを考慮し、通知が重ならないよう間隔を設ける

```js
const MAX_TOASTS = 3;

function addToast(message) {
  if (toasts.length >= MAX_TOASTS) {
    // 最も古いトーストを削除
    removeToast(toasts[0].id);
  }
  // 新しいトーストを追加
  toasts.push({ id: generateId(), message });
}
```

## よくある間違い

### 1. ライブリージョンを動的に追加する

```js
// 間違い：要素をDOMに追加する時点でテキストが入っている
const div = document.createElement('div');
div.setAttribute('role', 'status');
div.textContent = '保存しました';
document.body.appendChild(div);

// 正解：空のライブリージョンを事前にHTMLに配置
// <div id="status" role="status"></div>
document.getElementById('status').textContent = '保存しました';
```

### 2. aria-live="assertive" を多用する

- `assertive` はユーザーの操作を中断するため、本当に緊急な場合のみ使用
- 成功メッセージや情報通知には `polite` を使う

```html
<!-- 間違い：成功メッセージにassertiveを使う -->
<div aria-live="assertive">保存しました</div>

<!-- 正解：politeで十分 -->
<div aria-live="polite" role="status">保存しました</div>
```

### 3. 自動消去が速すぎる

- 1〜2秒で消えるトーストはスクリーンリーダーが読み上げを完了できない
- 最低5秒、文字数が多い場合はさらに長く表示する

### 4. フォーカスを奪うトースト

- トースト表示時にフォーカスを奪うと、ユーザーの操作が中断される
- トーストはフォーカスを移動させず、ライブリージョンで通知する
- フォーカスを奪うべきなのはアラートダイアログ（`role="alertdialog"`）のみ

### 5. 閉じる手段がない

```html
<!-- 間違い：自動消去のみで手動で閉じられない -->
<div class="toast">保存しました</div>

<!-- 正解：閉じるボタンを提供する -->
<div class="toast">
  保存しました
  <button aria-label="通知を閉じる">✕</button>
</div>
```

## prefers-reduced-motion への対応

- トーストのスライドやフェードアニメーションを、モーション低減設定を尊重して制御する

```css
.toast {
  animation: slideIn 0.3s ease-out;
}

@media (prefers-reduced-motion: reduce) {
  .toast {
    animation: none;
  }
}
```

## まとめ

- ステータスメッセージには `role="status"`（`aria-live="polite"`）を使用
- エラーや警告には `role="alert"`（`aria-live="assertive"`）を使用
- ライブリージョンは**事前にDOMに配置**し、テキストを後から挿入する
- 自動消去は最低5秒、ホバー / フォーカスで一時停止を提供する
- トースト表示時にフォーカスを奪わない
- 手動で閉じるボタンを提供する
- `assertive` は本当に緊急な場合のみ使用する
