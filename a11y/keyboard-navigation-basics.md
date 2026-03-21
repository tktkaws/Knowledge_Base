# キーボードナビゲーションの基本 — Tab, Enter, Escape

## なぜキーボード操作が重要か

- マウスを使えないユーザー（視覚障害、運動障害など）にとって唯一の操作手段
- スクリーンリーダーユーザーはキーボードでページを操作する
- WCAG 2.2 達成基準 2.1.1「キーボード」（レベルA）で全機能のキーボード操作が求められている
- パワーユーザーもキーボードショートカットを多用する

> 参照: [WCAG 2.1.1 Keyboard](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html)

## 基本のキー操作

| キー | 役割 |
|---|---|
| `Tab` | 次のフォーカス可能な要素に移動 |
| `Shift + Tab` | 前のフォーカス可能な要素に移動 |
| `Enter` | リンクの遷移、ボタンの実行、フォームの送信 |
| `Space` | ボタンの実行、チェックボックスの切替、セレクトボックスの展開 |
| `Escape` | モーダルを閉じる、ドロップダウンを閉じる、操作のキャンセル |
| `Arrow Keys` | ラジオボタン群の切替、タブの切替、メニュー内の移動 |

## Tab キーとフォーカス順序

### フォーカス可能な要素

- デフォルトでフォーカスを受け取れる要素（インタラクティブ要素）:
  - `<a href="...">`（href属性があるもの）
  - `<button>`
  - `<input>`, `<select>`, `<textarea>`
  - `<details>` / `<summary>`
- `<div>`, `<span>`, `<p>` などの非インタラクティブ要素はデフォルトではフォーカスされない

### フォーカス順序はDOM順に従う

- Tabキーによるフォーカス移動はDOMの出現順序で決まる
- CSSで見た目の順序を変更しても、フォーカス順序は変わらない
- 視覚的な順序とフォーカス順序が一致しないとユーザーが混乱する

```html
<!-- 悪い例：CSSでorderを変えて視覚順序とフォーカス順序がズレる -->
<div style="display: flex;">
  <button style="order: 2;">ボタンA</button> <!-- Tab 1番目だが右に表示 -->
  <button style="order: 1;">ボタンB</button> <!-- Tab 2番目だが左に表示 -->
</div>

<!-- 良い例：DOM順序と視覚順序を一致させる -->
<div style="display: flex;">
  <button>ボタンB</button>
  <button>ボタンA</button>
</div>
```

> 参照: [WCAG 2.4.3 Focus Order](https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html)

## Enter キーと Space キーの違い

- **Enter キー**: リンクの遷移、ボタンの押下、フォーム送信
- **Space キー**: ボタンの押下、チェックボックスの切替、スクロール
- `<button>` はEnterでもSpaceでも動作する
- `<a>` はEnterでのみ動作する（Spaceではページがスクロールする）

```html
<!-- button要素：Enter / Space 両方で動作 -->
<button onclick="handleClick()">送信</button>

<!-- a要素：Enterのみで動作 -->
<a href="/next">次のページ</a>

<!-- 悪い例：div + onclickはキーボードで操作できない -->
<div onclick="handleClick()">送信</div>

<!-- 悪い例の修正：divをキーボード対応させるなら全て自前実装が必要 -->
<div
  role="button"
  tabindex="0"
  onclick="handleClick()"
  onkeydown="if(event.key === 'Enter' || event.key === ' ') { event.preventDefault(); handleClick(); }"
>
  送信
</div>

<!-- 良い例：素直にbutton要素を使う -->
<button onclick="handleClick()">送信</button>
```

### Enter と Space でボタンの挙動が異なるケース

- `<button>`: Enterキーを押すとkeydown時点で即発火する。Spaceキーはkeyup時に発火する
- `<input type="submit">`: 同様の挙動
- カスタムボタンを実装する場合、この挙動の違いを意識する必要がある

```js
// カスタムボタンにキーボード操作を実装する例
element.addEventListener('keydown', (event) => {
  if (event.key === 'Enter') {
    event.preventDefault();
    element.click();
  }
  if (event.key === ' ') {
    // Spaceはスクロール防止のためdefaultPreventだけ行う
    event.preventDefault();
  }
});

element.addEventListener('keyup', (event) => {
  if (event.key === ' ') {
    element.click();
  }
});
```

## Escape キーの役割

- モーダルダイアログを閉じる
- ドロップダウンメニューやポップアップを閉じる
- 開いているコンボボックスのリストを閉じる
- 操作を取り消して元の状態に戻す
- Escapeで閉じた後、フォーカスを開く前の要素に戻すのが定石

```js
// モーダルのEscapeキー処理
dialog.addEventListener('keydown', (event) => {
  if (event.key === 'Escape') {
    closeModal();
    // モーダルを開いたトリガー要素にフォーカスを戻す
    triggerButton.focus();
  }
});
```

> 参照: [WAI-ARIA Authoring Practices — Dialog Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

## フォーカスインジケーター（フォーカスリング）

- 現在フォーカスされている要素を視覚的に示す枠線やアウトライン
- WCAG 2.4.7「フォーカスの可視化」（レベルAA）で必須とされている
- WCAG 2.4.11（レベルAAA）/ 2.4.13（レベルAA、WCAG 2.2で追加）ではフォーカスインジケーターの最小サイズも規定

```css
/* 悪い例：アウトラインを消してしまう */
*:focus {
  outline: none;
}

/* 良い例：カスタムのフォーカススタイルを提供する */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* :focus-visible を使うことでマウスクリック時にはリングを表示しない */
button:focus:not(:focus-visible) {
  outline: none;
}
```

### :focus と :focus-visible の違い

- `:focus` — 要素がフォーカスされた全ての場合に適用（クリック含む）
- `:focus-visible` — キーボード操作でフォーカスされた場合にのみ適用（ブラウザが判定）
- モダンブラウザでは `:focus-visible` を使い、キーボード操作時のみフォーカスリングを表示するのが推奨

> 参照: [WCAG 2.4.7 Focus Visible](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html)

## インタラクティブ要素を使うことの重要性

- ネイティブHTML要素（`<button>`, `<a>`, `<input>` など）はキーボード操作が組み込み済み
- `<div>` や `<span>` で同等の機能を実現するには多くの実装が必要

| 機能 | `<button>` | `<div role="button">` |
|---|---|---|
| フォーカス可能 | 自動 | `tabindex="0"` が必要 |
| Enter / Space で動作 | 自動 | `keydown` / `keyup` のハンドリングが必要 |
| `disabled` 属性 | 使用可能 | `aria-disabled="true"` + クリック/キーイベント無効化が必要 |
| フォーム送信 | `type="submit"` で可能 | 不可（JSで実装） |
| スクリーンリーダーへの通知 | 自動（「ボタン」と読み上げ） | `role="button"` が必要 |

## よくある間違い

### 1. outline: none でフォーカスリングを消す

```css
/* 間違い：全てのフォーカスリングを消す */
* { outline: none; }

/* 正解：:focus-visible でキーボード時のみ表示 */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

### 2. クリックだけ対応してキーボードに未対応

```html
<!-- 間違い：キーボードで操作できない -->
<div class="card" onclick="navigateTo('/detail')">
  <h3>商品名</h3>
  <p>説明文</p>
</div>

<!-- 正解：リンクを使う -->
<a href="/detail" class="card">
  <h3>商品名</h3>
  <p>説明文</p>
</a>
```

### 3. CSSの order や position でフォーカス順序を壊す

- `display: flex` の `order` プロパティや `position: absolute` で要素の表示位置を変えても、Tabキーのフォーカス順序はDOM順のまま
- 解決策：DOM順序と視覚順序を一致させる

## まとめ

- キーボードナビゲーションの基本は `Tab`, `Enter`, `Space`, `Escape`, `矢印キー`
- フォーカス順序はDOM順に従う — 視覚的な順序と一致させること
- フォーカスインジケーターを消さない — `:focus-visible` でカスタマイズ
- ネイティブHTML要素（`<button>`, `<a>` など）を使えばキーボード操作は自動で対応される
- `<div>` + `onclick` をキーボード対応させるより、適切なHTML要素を使う方がシンプルかつ確実
