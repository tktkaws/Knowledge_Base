# 1.4.13 ホバーまたはフォーカスで表示されるコンテンツ（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.13「ホバーまたはフォーカスで表示されるコンテンツ」（Content on Hover or Focus）
- レベル **AA**（標準的な適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- WCAG 2.1 で追加された達成基準
- ポインターホバーやキーボードフォーカスで**追加コンテンツ**が表示・非表示される場合、次の3条件を満たすこと
  - **閉じられる**（Dismissible）
  - **ホバーできる**（Hoverable）
  - **持続する**（Persistent）

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.13 Content on Hover or Focus](https://www.w3.org/TR/WCAG22/#content-on-hover-or-focus)
> - [Understanding SC 1.4.13 Content on Hover or Focus](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html)

## 達成基準の原文（要約）

- ポインターホバーまたはキーボードフォーカスの受け取りと解除により、追加コンテンツが表示・非表示される場合、次がすべて真であること
  - **閉じられる**：ポインターやフォーカスを動かさずに追加コンテンツを閉じる仕組みがある（入力エラーを伝える場合、または他のコンテンツを覆わない・置き換えない場合を除く）
  - **ホバーできる**：ポインターホバーで追加コンテンツが表示される場合、その追加コンテンツ上にポインターを移動しても消えない
  - **持続する**：ホバー／フォーカスが外れる、ユーザーが閉じる、または情報が無効になるまで、追加コンテンツが表示され続ける
- 例外：追加コンテンツの視覚的提示がユーザーエージェントによって制御され、作者が変更していない場合

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.13](https://www.w3.org/TR/WCAG22/#content-on-hover-or-focus)

## 対象となるコンテンツ

- ホバーやフォーカスで表示される**カスタムツールチップ**
- ホバーやフォーカスで開く**サブメニュー**
- ホバーやフォーカスで表示される**非モーダルポップアップ**
- トリガー要素に加えて表示される追加コンテンツが対象

```html
<!-- 対象：カスタムツールチップ -->
<button type="button" aria-describedby="tip-1">保存</button>
<div id="tip-1" role="tooltip" hidden>下書きを保存します</div>

<!-- 対象：ホバーで開くサブメニュー -->
<nav>
  <button type="button" aria-expanded="false" aria-controls="submenu">製品</button>
  <ul id="submenu" hidden>...</ul>
</nav>
```

> **参照**
> - [Understanding SC 1.4.13 — Note 2](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html#success-criterion)

## 対象外

### ユーザーエージェントが制御する表示

- HTML の `title` 属性によるブラウザ標準ツールチップ
- 作者が見た目を変更していない UA 標準の表示

```html
<!-- 対象外：ブラウザ標準の title ツールチップ -->
<button type="button" title="下書きを保存します">保存</button>
```

### トリガー自体の表示切替

- フォーカスで表示されるスキップリンクなど、**隠れていたコンポーネント自体**が表示される場合は対象外
- 追加コンテンツではなく、トリガー自身の表示切替だから

```html
<!-- 対象外：スキップリンク（隠れていたリンク自体が表示される） -->
<a class="skip-link" href="#main">メインコンテンツへスキップ</a>
```

### モーダルダイアログ

- モーダルはキーボードフォーカスを取る必要があるため、ホバー／フォーカスだけで出すべきではない
- 3.2.1「フォーカス時」など別の達成基準で扱う

> **参照**
> - [Understanding SC 1.4.13 — Additional Notes](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html#additional-notes)

## なぜ必要か

- 拡大表示では、ビューポート内の大部分がホバートリガーになり、意図せず追加コンテンツが出やすい
- 大きなマウスポインターが追加コンテンツを覆い、読めなくなることがある
- ロービジョン・認知障害のあるユーザーは、追加コンテンツを認識・読むのに時間がかかる
- ポインター精度が低いユーザーは、意図せず出たコンテンツを閉じにくい

> **参照**
> - [Understanding SC 1.4.13 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html#intent)
> - [Understanding SC 1.4.13 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html#benefits)

## 条件1：閉じられる（Dismissible）

- ポインターやフォーカスを動かさずに、追加コンテンツを閉じられること
- 代表的な方法は **Escape キー**で閉じる
- 追加コンテンツが他のコンテンツを覆わない・置き換えない場合は、閉じる仕組みは不要
- 入力エラーメッセージは、注意や修正が必要なため持続してよい

```html
<!-- 良い例：Escape で閉じられるツールチップ -->
<button
  type="button"
  id="save-btn"
  aria-describedby="save-tip"
>
  保存
</button>
<div id="save-tip" role="tooltip" hidden>
  下書きを保存します
</div>
```

```javascript
const button = document.getElementById("save-btn");
const tip = document.getElementById("save-tip");

function showTip() {
  tip.hidden = false;
}

function hideTip() {
  tip.hidden = true;
}

button.addEventListener("mouseenter", showTip);
button.addEventListener("focus", showTip);
button.addEventListener("mouseleave", hideTip);
button.addEventListener("blur", hideTip);

document.addEventListener("keydown", (event) => {
  if (event.key === "Escape" && !tip.hidden) {
    hideTip();
    // フォーカスはトリガーに残す
  }
});
```

```css
/* 良い例：他のコンテンツを覆わない配置 */
.tooltip {
  position: absolute;
  inset-block-start: 100%;
  inset-inline-start: 0;
  margin-block-start: 0.25rem;
}
```

- 小さなトリガーでは、**覆わない配置**と **Escape で閉じる**の両方を実装すると望ましい
- 大きなトリガーでは、離れた位置に出すと気づきにくいため、閉じる仕組みが特に重要

> **参照**
> - [Understanding SC 1.4.13 — Dismissible](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html#dismissible)

## 条件2：ホバーできる（Hoverable）

- ポインターホバーで追加コンテンツが出る場合、その追加コンテンツ上にポインターを移動しても消えないこと
- 拡大表示でスクロール／パンが必要なとき、トリガーから離れて追加コンテンツを読める必要がある
- 大きなポインターが追加コンテンツを覆う場合、ポインターを追加コンテンツ上に移して読める必要がある
- 追加コンテンツはトリガーと重なるか、隣接して配置するのが一般的

```css
/* 悪い例：トリガーから離れるとすぐ消える（ホバー不可） */
.trigger:hover + .tooltip {
  display: block;
}
/* トリガーとツールチップの間に隙間があると、移動中に消える */

/* 良い例：トリガーとツールチップを隣接させ、両方にホバー可能 */
.tooltip-wrapper:hover .tooltip,
.tooltip-wrapper:focus-within .tooltip,
.tooltip:hover {
  display: block;
}
```

```html
<!-- 良い例：ラッパーでホバー領域をまとめる -->
<div class="tooltip-wrapper">
  <button type="button" aria-describedby="tip-2">詳細</button>
  <div id="tip-2" role="tooltip" class="tooltip" hidden>
    この操作は元に戻せません。実行前に内容を確認してください。
  </div>
</div>
```

> **参照**
> - [Understanding SC 1.4.13 — Hoverable](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html#hoverable)
> - [Failure F95: content shown on hover not being hoverable](https://www.w3.org/WAI/WCAG22/Techniques/failures/F95)

## 条件3：持続する（Persistent）

- 追加コンテンツは、次のいずれかまで表示され続けること
  - ホバー／フォーカスがトリガーと追加コンテンツから外れる
  - ユーザーが閉じる仕組みで閉じる
  - 情報が無効になる（例：「処理中」メッセージが不要になった）
- 数秒で自動的に消えるタイムアウトは不適合になりやすい
- ユーザーが読む時間を十分に確保する

```javascript
// 悪い例：3秒で自動的に消える
button.addEventListener("mouseenter", () => {
  tip.hidden = false;
  setTimeout(() => {
    tip.hidden = true;
  }, 3000);
});

// 良い例：ホバー／フォーカスが外れるまで表示
button.addEventListener("mouseenter", () => {
  tip.hidden = false;
});
button.addEventListener("mouseleave", () => {
  tip.hidden = true;
});
```

> **参照**
> - [Understanding SC 1.4.13 — Persistent](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html#persistent)

## 実装パターン：SCR39

- ホバー／フォーカスで表示されるコンテンツを、ホバー可能・閉じ可能・持続可能にする
- キーボードフォーカスでも同じコンテンツを表示する（2.1.1「キーボード」とも関連）
- Escape で閉じ、フォーカスはトリガーに残す

```html
<div class="menu">
  <button
    type="button"
    id="products-btn"
    aria-expanded="false"
    aria-controls="products-menu"
  >
    製品
  </button>
  <ul id="products-menu" role="menu" hidden>
    <li role="none"><a role="menuitem" href="/a">製品A</a></li>
    <li role="none"><a role="menuitem" href="/b">製品B</a></li>
  </ul>
</div>
```

```javascript
const btn = document.getElementById("products-btn");
const menu = document.getElementById("products-menu");

function openMenu() {
  menu.hidden = false;
  btn.setAttribute("aria-expanded", "true");
}

function closeMenu() {
  menu.hidden = true;
  btn.setAttribute("aria-expanded", "false");
}

btn.addEventListener("mouseenter", openMenu);
btn.addEventListener("focus", openMenu);
menu.addEventListener("mouseenter", openMenu);

btn.addEventListener("mouseleave", (event) => {
  if (!menu.contains(event.relatedTarget)) closeMenu();
});
menu.addEventListener("mouseleave", (event) => {
  if (!btn.contains(event.relatedTarget)) closeMenu();
});
btn.addEventListener("blur", () => {
  // メニュー内へフォーカス移動する場合は閉じない
  requestAnimationFrame(() => {
    if (!menu.contains(document.activeElement) && document.activeElement !== btn) {
      closeMenu();
    }
  });
});

document.addEventListener("keydown", (event) => {
  if (event.key === "Escape" && !menu.hidden) {
    closeMenu();
    btn.focus();
  }
});
```

> **参照**
> - [Technique SCR39: Making content on focus or hover hoverable, dismissible, and persistent](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR39)

## ツールチップの設計指針

- トリガーの下や横に配置し、他の重要なコンテンツをできるだけ覆わない
- 覆う場合は Escape で閉じられるようにする
- ホバーとフォーカスの両方で表示する
- `role="tooltip"` と `aria-describedby` で関連付ける
- ツールチップ内に操作可能な要素を置く場合は、ツールチップではなくポップオーバー／メニューとして設計する

```html
<!-- 良い例：フォーカスでも表示、Escape で閉じられる -->
<button
  type="button"
  aria-describedby="help-tip"
  class="help-trigger"
>
  ?
</button>
<div id="help-tip" role="tooltip" class="help-tip" hidden>
  必須項目には * が付いています
</div>
```

> **参照**
> - [WAI-ARIA Authoring Practices — Tooltip Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/tooltip/)

## サブメニュー・メガメニュー

- ホバーで開くメニューは、メニュー上にポインターを移動しても閉じないこと
- Escape で閉じられること
- キーボードフォーカスでも開けること
- ホバーだけでしか開けないメニューは、2.1.1「キーボード」にも抵触しやすい

```html
<!-- 悪い例：ホバーのみで開き、メニュー上に移動すると消える -->
<style>
  .nav-item:hover > .submenu { display: block; }
  /* トリガーとサブメニューの間に隙間があると F95 */
</style>
```

## 入力エラーメッセージとの関係

- 入力エラーを伝える追加コンテンツは、閉じる仕組みの例外になりうる
- エラーは注意・確認・修正が必要なため、持続してよい
- ただし、エラーメッセージ自体のアクセシビリティ（3.3.1、3.3.3 など）は別途必要

```html
<!-- 例外になりうる：入力エラーメッセージ -->
<label for="email">メールアドレス</label>
<input id="email" type="email" aria-invalid="true" aria-describedby="email-error">
<p id="email-error" role="alert">正しいメールアドレスの形式で入力してください</p>
```

## 2.1.1・3.2.1 との関係

| 達成基準 | 関係 |
|---|---|
| 2.1.1 キーボード | ホバーで出る内容は、キーボードフォーカスでも操作・表示できること |
| 3.2.1 フォーカス時 | フォーカスだけでコンテキストが大きく変わらないこと |
| 1.4.13 | ホバー／フォーカスで出る追加コンテンツの閉じ方・ホバー・持続 |

- ホバー専用の情報は、キーボードユーザーが受け取れない
- フォーカスでモーダルやページ遷移が起きると、3.2.1 の問題になりやすい
- 1.4.13 は「出たあとの振る舞い」を規定する

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.1 Keyboard](https://www.w3.org/TR/WCAG22/#keyboard)
> - [WCAG 2.2 — Success Criterion 3.2.1 On Focus](https://www.w3.org/TR/WCAG22/#on-focus)

## よくある失敗例

- ホバーで出たツールチップにポインターを移すと消える（F95）
- Escape などで閉じられず、拡大表示時に画面を覆い続ける
- 数秒で自動的に消える
- ホバーだけで表示され、キーボードフォーカスでは出ない
- トリガーと追加コンテンツの間に隙間があり、移動中に消える
- `title` 属性だけに頼らずカスタムツールチップを作ったが、3条件を満たしていない

```html
<!-- 悪い例：ホバー不可・自動消滅 -->
<button type="button" class="bad-trigger">情報</button>
<div class="bad-tooltip">重要な説明文がここに入ります</div>
```

```css
.bad-trigger:hover + .bad-tooltip {
  display: block;
  animation: fade-out 3s forwards;
}
/* ツールチップ上にホバーできない、かつ自動で消える */
```

> **参照**
> - [Failure F95: content shown on hover not being hoverable](https://www.w3.org/WAI/WCAG22/Techniques/failures/F95)

## テスト・確認方法

### 手動テスト

- ホバーで追加コンテンツが出る要素をすべて洗い出す
- 追加コンテンツ上にポインターを移動しても消えないか確認
- Escape（または同等の仕組み）で、ポインター／フォーカスを動かさずに閉じられるか確認
- 自動で消えないか確認
- キーボードフォーカスでも同じコンテンツが表示されるか確認
- 拡大表示（200%〜400%）で、追加コンテンツが画面を覆ったときに閉じられるか確認

```text
チェックリスト：
1. ホバーで出るか
2. フォーカスでも出るか
3. 追加コンテンツ上にホバーできるか
4. Escape 等で閉じられるか（覆う場合）
5. 自動で消えないか
6. 閉じたあと、トリガー操作を続けられるか
```

### 開発者ツールでの確認

- `:hover` / `:focus` / `:focus-within` で表示切替している箇所を確認
- `setTimeout` で自動非表示していないか確認
- `mouseleave` でトリガーから離れた瞬間に消していないか確認

```bash
# ホバー／フォーカス表示の実装箇所をざっくり確認
rg ":hover|:focus|mouseenter|mouseleave|role=\"tooltip\"|aria-describedby" --glob "*.css" --glob "*.scss" --glob "*.js" --glob "*.tsx" --glob "*.jsx"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.13 は、ホバー／フォーカスで出る追加コンテンツに **閉じられる・ホバーできる・持続する** を求める達成基準（レベル AA）
- カスタムツールチップ、サブメニュー、非モーダルポップアップが主な対象
- `title` 属性のブラウザ標準ツールチップ、スキップリンク、モーダルは対象外または別基準
- Escape で閉じる、追加コンテンツ上にホバーできる、自動消滅しない、が基本実装
- ホバーで出る内容はキーボードフォーカスでも表示する（2.1.1 とも関連）
- 入力エラーメッセージは閉じる仕組みの例外になりうる
