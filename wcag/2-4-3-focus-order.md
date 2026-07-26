# 2.4.3 フォーカス順序（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.3「フォーカス順序」（Focus Order）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- ページを順次ナビゲートでき、その順序が意味や操作に影響する場合、フォーカス可能な部品は**意味と操作性を保つ順序**でフォーカスを受け取ること
- キーボードだけで操作しても、論理的に進めるようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.3 Focus Order](https://www.w3.org/TR/WCAG22/#focus-order)
> - [Understanding SC 2.4.3 Focus Order](https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html)

## 達成基準の原文（要約）

- Webページを順次ナビゲートでき、そのナビゲーション順序が意味または操作に影響する場合
- フォーカス可能なコンポーネントは、意味と操作性を保持する順序でフォーカスを受け取ること

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.3](https://www.w3.org/TR/WCAG22/#focus-order)

## なぜ必要か

- 運動障害のあるキーボード利用者は、論理的なフォーカス順がないと操作しづらい
- 読字が難しい人は、Tab で予期しない場所へ飛ぶと混乱する
- 視覚障害のある人は、フォーカスが飛び回ると周囲の文脈を見失う
- 画面拡大利用者は、画面の一部しか見えないため、順序が崩れると欄の意味を取り違える
- 一貫したメンタルモデルを作りやすくする

> **参照**
> - [Understanding SC 2.4.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html#benefits)

## 順次ナビゲーションとは

- キーボードインタフェースで、フォーカスを次の要素へ進める順序
- HTML では主に Tab によるタブ順
- `tabindex` やスクリプトを使わなければ、**コンテンツの出現順（DOM順）**がタブ順になる
- ツリーの矢印キー操作など、コンポーネント内の独自ナビは「順次ナビ」とは別物として扱われる場合がある

```text
この基準が扱う例：
  Tab / Shift+Tab でリンクやフォームを順に移動する

この基準の主対象ではない例：
  ツリーで ↑↓ キーでノード移動、→ で展開
  （コンポーネントとして期待される操作なら問題になりにくい）
```

> **参照**
> - [Understanding SC 2.4.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html#intent)

## 意味と操作性を保つ順序

- 視覚レイアウトと完全一致は必須ではない
- ただし、意味・操作が損なわれない論理的な順序であること
- 論理的な順序が複数ある場合、そのうち1つを提供すればよい
- 例：表を行優先で辿る／列優先で辿る、どちらも論理的なら適合しうる
- ベストプラクティスとしては、視覚的な読み順を補強するフォーカス順にする

```text
適合しうる例：
  2カラムで左右が独立していれば、右カラムが先でも意味が壊れない場合がある
  （ただし左→右の方が望ましいことが多い）

不適合になりやすい例：
  氏名 → ニュースレターチェック → 住所 → 別のチェック、のようにセクションを飛び回る
  入れ子のフォーカス可能要素で、同じコントロールに何度もフォーカスが当たる
```

```html
<!-- 悪い例：入れ子でフォーカスが重複しやすい -->
<div tabindex="0">
  <button type="button">保存</button>
</div>
```

> **参照**
> - [Understanding SC 2.4.3 — Note on focusable components](https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html#intent)

## 手法1：DOM順と視覚順を一致させる（C27）

- CSS で見た目だけ並べ替えず、**HTML の出現順を視覚順に近づける**
- `order`・絶対配置・負の `tabindex` 乱用で順序をねじ曲げない
- メインを先にフォーカスしたい場合は、HTML上でメインを先に書き、CSSでナビを左に置く方法もある

```html
<!-- 良い例：DOM順＝視覚順＝フォーカス順 -->
<form>
  <label for="name">氏名</label>
  <input id="name" name="name" autocomplete="name">

  <label for="address">住所</label>
  <input id="address" name="address" autocomplete="street-address">

  <label>
    <input type="checkbox" name="newsletter">
    ニュースレターを受け取る
  </label>
</form>
```

```html
<!-- 理解文書の例：HTMLでは main が先、CSSでナビを左に見せる -->
<!-- 本文へ先にフォーカスでき、意味も保てるなら適合しうる -->
<body>
  <a href="#main">メインへスキップ</a>
  <main id="main">本文...</main>
  <nav class="side-nav">ナビ...</nav>
</body>
```

```css
/* 悪い例：視覚順とDOM順が大きく食い違う */
.form {
  display: flex;
  flex-direction: column;
}
.postal {
  order: -1; /* 見た目は上、DOMでは下 → フォーカスが予期せず飛ぶ */
}
```

> **参照**
> - [Technique C27: Making the DOM order match the visual order](https://www.w3.org/WAI/WCAG22/Techniques/css/C27)

## 手法2：動的コンテンツをトリガーの直後に挿入する（SCR26）

- ボタンで開く非モーダルのパネルなどは、**トリガーの直後**に DOM へ入れる
- フォーカス順が「ボタン → パネル内 → ボタンの次」になる
- 離れた場所に挿入すると、F85 の失敗になりやすい

```html
<!-- 良い例：トリガー直後にパネルがある -->
<button type="button" aria-expanded="true" aria-controls="panel">詳細</button>
<div id="panel">
  <a href="/details">詳細ページ</a>
  <button type="button">閉じる</button>
</div>
<a href="/next">次の項目</a>
```

```html
<!-- 悪い例：パネルがページ末尾にあり、フォーカス順が離れる -->
<button type="button">詳細</button>
<a href="/next">次の項目</a>
<!-- ...大量のコンテンツ... -->
<div id="panel">パネル内容</div>
```

> **参照**
> - [Technique SCR26: Inserting dynamic content into the Document Object Model immediately following its trigger element](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR26)

## 手法3：モーダルダイアログ（H102）

- `<dialog>` などでモーダルを開き、開いている間は外側を操作不能（inert）にする
- 開いたらダイアログ内へフォーカスを移す
- 閉じたらトリガー、またはその次の要素へフォーカスを戻す
- 非モーダルは、トリガー直後のフォーカス順に載せる（上記 SCR26）

```html
<!-- 良い例：HTML dialog -->
<button type="button" id="open-dialog">設定を開く</button>

<dialog id="settings">
  <form method="dialog">
    <h2>設定</h2>
    <label>
      通知を受け取る
      <input type="checkbox" name="notify">
    </label>
    <button type="submit">閉じる</button>
  </form>
</dialog>
```

```javascript
const openButton = document.getElementById("open-dialog");
const settings = document.getElementById("settings");

openButton.addEventListener("click", () => {
  settings.showModal();
});

settings.addEventListener("close", () => {
  openButton.focus(); // 閉じたあとトリガーへ戻す
});
```

> **参照**
> - [Technique H102: Creating modal dialogs with the HTML dialog element](https://www.w3.org/WAI/WCAG22/Techniques/html/H102)
> - [Understanding SC 2.4.3 — modal / non-modal examples](https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html#examples)

## よくある失敗

### F44：意味を壊す tabindex

- 正の `tabindex`（1, 2, 3...）でタブ順を無理やり作ると、意味や操作性が壊れやすい
- 可能な限り `tabindex` は使わない、または `0` / `-1` に限定する

```html
<!-- 悪い例：正の tabindex で順序がジャンプする（F44） -->
<input tabindex="3" name="name">
<input tabindex="1" name="newsletter">
<input tabindex="2" name="address">
```

```html
<!-- 良い例：DOM順に任せ、tabindex を使わない -->
<input name="name">
<input name="address">
<input name="newsletter" type="checkbox">
```

```html
<!-- 良い例：カスタム部品をフォーカス可能にするなら 0 / -1 -->
<div role="button" tabindex="0">メニュー</div>
<div id="panel" tabindex="-1">プログラムでフォーカスを移す先</div>
```

> **参照**
> - [Failure F44: using tabindex to create a tab order that does not preserve meaning and operability](https://www.w3.org/WAI/WCAG22/Techniques/failures/F44)

### F85：トリガーから離れたダイアログ／メニュー

- ダイアログやメニューが、順次ナビ上でトリガーの近くにない
- 開いたあとに遠くへ飛ばされる、閉じたあとに迷子になる

```text
失敗の典型：
  ヘッダーのボタンを押す
  → フォーカスがページ末尾のメニューへ飛ぶ
  → Tab で戻るのに何十回もかかる
```

> **参照**
> - [Failure F85: dialogs or menus that are not adjacent to their trigger control in the sequential navigation order](https://www.w3.org/WAI/WCAG22/Techniques/failures/F85)

## 1.3.2・2.1.1・2.4.7 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 1.3.2 意味のある順序 | 読み上げ順・プログラム上の順序 | 読む順序 |
| 2.4.3 フォーカス順序 | フォーカスの移動順 | 操作する順序 |
| 2.1.1 キーボード | キーボードで操作できるか | 到達・操作の可否 |
| 2.4.7 フォーカスの可視化 | どこにフォーカスがあるか見えるか | 現在地の視覚表示 |

- フォーカス順と読み上げ順は、必ずしも同一でなくてよい
- ただし両方とも意味が通ること。視覚キーボード利用者とスクリーンリーダー利用者の両方を意識する
- フォーカスが当たっても見えなければ 2.4.7 の問題

> **参照**
> - [wcag/1-3-2-meaningful-sequence.md — 1.3.2 意味のある順序](./1-3-2-meaningful-sequence.md)
> - [wcag/2-1-1-keyboard.md — 2.1.1 キーボード](./2-1-1-keyboard.md)

## 実装時の注意点

- まず HTML の順序を正し、CSS は見た目の調整に留める
- 正の `tabindex` は避ける
- 操作できない静的テキストへのフォーカスは必須ではない（可能だが、冗長になりやすい）
- モーダルはフォーカストラップと復帰先を設計する（2.1.2 とも関連）
- `display: contents` やポータル（React Portal など）で DOM 位置がトリガーから離れないか確認する

```text
実務チェック：
  Tab だけで一通り操作できるか
  フォームがセクションを飛ばないか
  モーダル開閉でフォーカスが迷子にならないか
  視覚順と大きく食い違っていないか
```

## テスト・確認方法

### 手動テスト

- マウスを使わず Tab / Shift+Tab だけでページを操作する
- フォーカスが意味の通る順に進むか確認する
- フォームで関連フィールドがまとまって辿れるか確認する
- ダイアログ・メニュー・アコーディオンを開き、順序が論理的か確認する
- 閉じたあと、フォーカスが妥当な位置に戻るか確認する

```text
チェックリスト：
1. フォーカス可能な要素の順序は意味を保っているか
2. 正の tabindex で順序を無理に作っていないか
3. 動的 UI はトリガーの近くのフォーカス順にあるか
4. モーダルは開閉時のフォーカス移動が適切か
5. 入れ子で同じ操作が二重にフォーカスされていないか
```

### 開発者ツールでの確認

```bash
# tabindex や dialog・フォーカス移動を探す
rg "tabindex|showModal|\\.focus\\(|createPortal" --glob "*.html" --glob "*.js" --glob "*.tsx" --glob "*.jsx"
```

- DevTools の Accessibility パネルでフォーカス順を確認する
- 正の `tabindex` がある要素を重点的に見直す

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.3 は、フォーカス可能な部品が**意味と操作性を保つ順序**でフォーカスを受け取ることを求める達成基準（レベル A）
- 基本は DOM順＝視覚順＝タブ順（C27）
- 正の `tabindex`（F44）や、トリガーから離れたダイアログ／メニュー（F85）が典型的な失敗
- 動的 UI はトリガー直後へ挿入するか、モーダルとしてフォーカスを適切に管理する
- 1.3.2（読み順）・2.1.1（キーボード）・2.4.7（可視化）と併せて設計する
