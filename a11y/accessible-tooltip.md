# アクセシブルなツールチップの実装

## ツールチップとは

- 要素にフォーカスまたはホバーしたときに、関連情報をポップアップ表示するUI
- アイコンボタンの補足説明、省略されたテキストの全文表示などに使う
- フォーカスはトリガー側に残り、ツールチップ自体はフォーカスを受け取らない
- 中にリンクやボタンなどのインタラクティブ要素を置かない

> 参照: [WAI-ARIA Authoring Practices — Tooltip Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/tooltip/)

## ツールチップ vs 類似UI

| UI | 開き方 | 中身 | 用途 |
|---|---|---|---|
| ツールチップ | ホバー / フォーカスで自動表示 | 短いテキストのみ | 補足説明 |
| トグルチップ | ボタン操作で明示的に開閉 | テキスト中心（対話要素は避ける） | クリックで確認したい補足 |
| ダイアログ | 明示的な操作で開く | フォーカス可能な要素可 | 操作や詳細な説明が必要 |
| `title` 属性 | ホバーのみ（キーボード非対応が多い） | 短いテキスト | **アクセシブルな代替にはしない** |

- 重要な情報はツールチップだけに閉じ込めない — 常時見えるラベルや本文に置く
- クリックやEnterで開くUIが必要なら、ツールチップではなくトグルチップやダイアログを検討

## 必要なARIA属性

| 要素 | ロール / 属性 | 説明 |
|---|---|---|
| ツールチップ本体 | `role="tooltip"` | ツールチップであることを示す |
| トリガー要素 | `aria-describedby="ツールチップのid"` | 補足説明として紐付ける |

- `aria-describedby` は**トリガー側**に付ける（ツールチップ側ではない）
- 表示中はスクリーンリーダーがトリガー名に続けて説明を読み上げる
- 名前（ラベル）が不足している要素の代替にツールチップを使わない — まずアクセシブルネームを確保する

> 参照: [MDN — ARIA: tooltip role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles/tooltip_role)

## キーボード・ポインタ操作の要件

| 操作 | 動作 |
|---|---|
| トリガーへフォーカス | ツールチップを表示（短い遅延を付けてもよい） |
| トリガーからフォーカスが外れる | ツールチップを非表示 |
| トリガーへホバー | ツールチップを表示 |
| ポインタがトリガーにもツールチップにもない | ツールチップを非表示 |
| `Escape` | フォーカス / ホバーを動かさずに閉じる |

- フォーカスは常にトリガーに残す
- ポインタをツールチップ上に移動でき、その間は消えないこと（WCAG 1.4.13 Hoverable）

## WCAG 1.4.13（ホバーまたはフォーカスで表示されるコンテンツ）

カスタムツールチップは次の3条件を満たす必要がある。

| 条件 | 意味 | 実装のポイント |
|---|---|---|
| Dismissible（閉じられる） | ポインタやフォーカスを動かさずに消せる | `Escape` で閉じる |
| Hoverable（ホバー可能） | 追加コンテンツ上にポインタを移せる | ツールチップ上でも表示を維持 |
| Persistent（持続する） | ホバー / フォーカス中は勝手に消えない | タイマーだけで強制消去しない |

- ネイティブの `title` 属性によるブラウザツールチップは、上記を満たさないことが多い
- エラーメッセージなど、入力エラーを伝える内容で他コンテンツを隠さないものは例外になりうるが、通常の補足ツールチップは3条件を満たす

> 参照: [Understanding SC 1.4.13: Content on Hover or Focus](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html)

## HTML構造

```html
<button
  type="button"
  class="icon-button"
  aria-describedby="save-tooltip"
  aria-label="保存"
>
  <!-- アイコンのみのボタン。名前は aria-label で確保 -->
  <svg aria-hidden="true" focusable="false">...</svg>
</button>

<div id="save-tooltip" role="tooltip" hidden>
  変更内容を保存します
</div>
```

### 構造のポイント

- トリガーはフォーカス可能な要素（`<button>` やリンクなど）にする
- ツールチップは短い説明テキストのみ。リンクやフォームを入れない
- 非表示時は `hidden` を使う
- アイコンだけのボタンは、ツールチップ以前に `aria-label` などで名前を付ける

## Vanilla JS での実装

```js
class AccessibleTooltip {
  constructor(trigger) {
    this.trigger = trigger;
    this.tooltipId = trigger.getAttribute('aria-describedby');
    this.tooltip = document.getElementById(this.tooltipId);
    this.hideTimer = null;

    if (!this.tooltip) return;

    this.show = this.show.bind(this);
    this.hide = this.hide.bind(this);
    this.hideSoon = this.hideSoon.bind(this);
    this.cancelHide = this.cancelHide.bind(this);
    this.onKeyDown = this.onKeyDown.bind(this);

    this.init();
  }

  init() {
    this.trigger.addEventListener('focus', this.show);
    this.trigger.addEventListener('blur', this.hide);
    this.trigger.addEventListener('pointerenter', this.show);
    this.trigger.addEventListener('pointerleave', this.hideSoon);
    this.trigger.addEventListener('keydown', this.onKeyDown);

    // ツールチップ上へポインタ移動できるようにする（WCAG 1.4.13）
    this.tooltip.addEventListener('pointerenter', this.cancelHide);
    this.tooltip.addEventListener('pointerleave', this.hide);
  }

  show() {
    this.cancelHide();
    this.tooltip.hidden = false;
  }

  hide() {
    this.tooltip.hidden = true;
  }

  hideSoon() {
    // トリガー → ツールチップへの移動猶予
    this.hideTimer = setTimeout(() => {
      this.hide();
    }, 100);
  }

  cancelHide() {
    clearTimeout(this.hideTimer);
    this.hideTimer = null;
  }

  onKeyDown(event) {
    if (event.key === 'Escape' && !this.tooltip.hidden) {
      this.hide();
    }
  }
}

document
  .querySelectorAll('[aria-describedby]')
  .forEach((trigger) => {
    const described = document.getElementById(
      trigger.getAttribute('aria-describedby')
    );
    if (described?.getAttribute('role') === 'tooltip') {
      new AccessibleTooltip(trigger);
    }
  });
```

## React での実装例

```jsx
import { useId, useRef, useState } from 'react';

function IconButtonWithTooltip({ label, tooltipText, children }) {
  const tooltipId = useId();
  const [open, setOpen] = useState(false);
  const hideTimerRef = useRef(null);

  const cancelHide = () => {
    clearTimeout(hideTimerRef.current);
    hideTimerRef.current = null;
  };

  const show = () => {
    cancelHide();
    setOpen(true);
  };

  const hide = () => {
    setOpen(false);
  };

  const hideSoon = () => {
    hideTimerRef.current = setTimeout(() => {
      setOpen(false);
    }, 100);
  };

  return (
    <span className="tooltip-wrapper">
      <button
        type="button"
        aria-label={label}
        aria-describedby={tooltipId}
        onFocus={show}
        onBlur={hide}
        onPointerEnter={show}
        onPointerLeave={hideSoon}
        onKeyDown={(event) => {
          if (event.key === 'Escape') hide();
        }}
      >
        {children}
      </button>
      <span
        id={tooltipId}
        role="tooltip"
        hidden={!open}
        onPointerEnter={cancelHide}
        onPointerLeave={hide}
      >
        {tooltipText}
      </span>
    </span>
  );
}
```

## トグルチップ（クリックで開く補足）

- ホバー操作が難しいタッチデバイスや、明示的な操作で開きたい場合に使う
- トリガーは `<button>`。`aria-expanded` で開閉状態を伝える
- 補足領域は `aria-describedby` または近くに配置した説明要素で紐付ける
- Escape と外側クリックで閉じ、フォーカスをトリガーに戻す

```html
<button
  type="button"
  aria-expanded="false"
  aria-controls="help-tip"
  aria-label="ユーザー名の説明"
>
  ?
</button>
<div id="help-tip" hidden>
  半角英数字4〜20文字で入力してください。
</div>
```

```js
const btn = document.querySelector('[aria-controls="help-tip"]');
const tip = document.getElementById('help-tip');

btn.addEventListener('click', () => {
  const next = btn.getAttribute('aria-expanded') !== 'true';
  btn.setAttribute('aria-expanded', String(next));
  tip.hidden = !next;
});

document.addEventListener('keydown', (event) => {
  if (event.key === 'Escape' && btn.getAttribute('aria-expanded') === 'true') {
    btn.setAttribute('aria-expanded', 'false');
    tip.hidden = true;
    btn.focus();
  }
});
```

## `title` 属性を使わない理由

```html
<!-- 悪い例：キーボードで表示できないブラウザが多い / Escapeで消せない / 表示遅延が長い -->
<button title="保存">💾</button>

<!-- 良い例：フォーカスでも表示でき、Escapeで閉じられるカスタムツールチップ -->
<button type="button" aria-label="保存" aria-describedby="tip-save">💾</button>
<div id="tip-save" role="tooltip" hidden>変更内容を保存します</div>
```

- `title` は補完的なヒントとしては残せるが、唯一の説明手段にしない
- アクセシブルネームの代替としても不適切

## スタイルの注意点

```css
[role="tooltip"] {
  position: absolute;
  z-index: 10;
  max-width: 16rem;
  padding: 0.5rem 0.75rem;
  background: #111;
  color: #fff;
  /* コントラスト比 4.5:1 以上を確保 */
}

/* トリガーとツールチップの隙間をなくし、ポインタ移動で消えないようにする */
.tooltip-wrapper {
  position: relative;
  display: inline-block;
}

.tooltip-wrapper [role="tooltip"] {
  top: calc(100% + 4px);
  left: 0;
}

@media (prefers-reduced-motion: reduce) {
  [role="tooltip"] {
    transition: none;
  }
}
```

- 他のコンテンツを大きく隠す配置は避ける。隠す場合は Escape で必ず消せるようにする
- 小さいタッチターゲットの問題はツールチップでは解決しない — ボタン自体のサイズを確保する

## よくある間違い

### 1. ホバーだけでしか開かない

```js
// 間違い：キーボード利用者が情報にアクセスできない
trigger.addEventListener('mouseenter', show);
trigger.addEventListener('mouseleave', hide);

// 正解：フォーカスでも表示する
trigger.addEventListener('focus', show);
trigger.addEventListener('blur', hide);
trigger.addEventListener('pointerenter', show);
trigger.addEventListener('pointerleave', hideSoon);
```

### 2. Escape で閉じられない

- WCAG 1.4.13 の Dismissible を満たせない
- 拡大表示でツールチップがポインタ下に残り続けると操作不能になる

### 3. ツールチップにリンクやボタンを入れる

```html
<!-- 間違い：フォーカスを受け取れない / 操作できない -->
<div role="tooltip">
  詳細は <a href="/help">ヘルプ</a> を参照
</div>

<!-- 正解：対話が必要ならダイアログや常時表示のテキストにする -->
```

### 4. aria-label の代わりにツールチップだけを使う

```html
<!-- 間違い：名前がなく、説明だけ紐付いている -->
<button aria-describedby="tip">?</button>
<div id="tip" role="tooltip">削除</div>

<!-- 正解：まず名前を付け、必要なら説明を追加 -->
<button aria-label="削除" aria-describedby="tip-delete">?</button>
<div id="tip-delete" role="tooltip">この項目を完全に削除します</div>
```

### 5. ポインタがツールチップ上に来ると消える

```js
// 間違い：trigger の mouseleave だけで即 hide
// → テキスト選択や拡大表示時に読めない

// 正解：tooltip 上でも表示を維持し、両方から離れたら閉じる
```

### 6. 必須情報をツールチップに隠す

- フォームの入力条件、エラー内容、操作手順など、タスク完了に必要な情報は常時表示する
- ツールチップはあくまで補足

## 実装チェックリスト

- [ ] トリガーにアクセシブルネームがある（`aria-label` / 可視テキストなど）
- [ ] ツールチップに `role="tooltip"` がある
- [ ] トリガーが `aria-describedby` でツールチップを参照している
- [ ] フォーカスとホバーの両方で表示される
- [ ] `Escape` で閉じられる
- [ ] ポインタをツールチップ上に移しても消えない
- [ ] ホバー / フォーカス中に勝手に消えない
- [ ] 中にインタラクティブ要素がない
- [ ] 色のコントラストが十分である
- [ ] 必須情報がツールチップのみに依存していない

## まとめ

- ツールチップは `role="tooltip"` + トリガーの `aria-describedby` で紐付ける
- フォーカス / ホバーで開き、Escape で閉じ、ツールチップ上へのホバーを維持する（WCAG 1.4.13）
- フォーカスはトリガーに残し、中に操作可能な要素を置かない
- `title` 属性やホバー専用実装はアクセシブルな代替にならない
- 重要な情報は常時見えるUIに置き、ツールチップは補足に限定する
