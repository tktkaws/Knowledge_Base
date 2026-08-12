# 2.5.7 ドラッグ操作（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.5.7「ドラッグ操作」（Dragging Movements）
- レベル **AA**（WCAG 2.2 で新規）
- 原則2「操作可能（Operable）」> ガイドライン 2.5「入力モダリティ」に属する
- ドラッグで操作する機能は、**ドラッグなしの単一ポインタ**でも達成できること
- 例外：ドラッグが**不可欠**、またはユーザーエージェントが決定し作者が変更していない場合

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.7 Dragging Movements](https://www.w3.org/TR/WCAG22/#dragging-movements)
> - [Understanding SC 2.5.7 Dragging Movements](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements.html)

## 達成基準の原文（要約）

- ドラッグ操作を使う機能はすべて、ドラッグなしの単一ポインタでも達成できること
- ただし、ドラッグが不可欠な場合、または機能がユーザーエージェントにより決定され作者が変更していない場合は除く
- **注記**：Webコンテンツがポインタ操作を解釈する場合に適用。UA／支援技術の操作に必要な動作は対象外

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.7](https://www.w3.org/TR/WCAG22/#dragging-movements)

## なぜ必要か

- ドラッグは、押したままポインタを正確に移動する巧緻運動が必要
- 手の震えや運動障害があると、掴んで移動して離す一連の操作が難しい
- トラックボール、ヘッドポインタ、視線入力、音声マウスなどでは、ドラッグが特に負担になる
- タッチ端末では物理キーボードがないことも多く、キーボード代替だけでは足りない

> **参照**
> - [Understanding SC 2.5.7 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements.html#benefits)

## ドラッグ操作とは

- ダウンイベントで要素を掴み、ポインタに追従させ、アップイベントで離す操作
- 経路・方向・速度は本質ではない（開始点と終了点が重要）
- 例：リスト項目、テキスト、画像などを掴んで別位置へ落とす

```text
ドラッグの4ステップ：
1. タップ／クリックで開始点を決める
2. 接触を押したまま保持する
3. ポインタを移動する
4. 終了点で離す
```

```text
対象の例：
  スライダーのつまみをドラッグして値を変える
  カードを列から列へドラッグ＆ドロップする
  地図をドラッグして表示位置を変える
  並べ替え可能なリストをドラッグで並び替える

対象外の例：
  ブラウザのスクロールバーをドラッグする
  タッチ画面でページをドラッグしてスクロールする
  ブラウザの「ドラッグして更新」
  CSS overflow による通常のスクロール
```

> **参照**
> - [Understanding SC 2.5.7 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements.html#intent)
> - [Understanding SC 2.5.7 — Key Terms: dragging movement](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements.html#dfn-dragging-movement)

## 単一ポインタの代替とは

- クリック、タップ、ダブルタップ、長押しなど、**押したまま移動しなくてよい**操作
- 同じページ上で、同等の結果を達成できればよい（同じ部品である必要はない）
- 数値入力欄も、タッチ時のオンスクリーンキーボード経由で単一ポインタ入力になるため代替になりうる
- 代替がスワイプ／フリックだけだと、2.5.1 に抵触しうる

```text
良い代替の例：
  スライダーのトラックをクリック／タップして値を変える
  並べ替え項目の横に「上へ／下へ」ボタンを置く
  カンバンカードをタップして、移動先をメニューで選ぶ
  地図のパンを上下左右ボタンでも行う
  カラーホイール上の別の位置をタップして色を選ぶ
```

> **参照**
> - [Understanding SC 2.5.7 — Alternatives for dragging movements on the same page](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements.html#intent)

## 2.5.1・2.1.1 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.5.1 ポインタのジェスチャ | 軌跡・マルチポイントの代替 | 経路が重要 |
| 2.5.7 ドラッグ操作 | ドラッグの代替 | 開始点と終了点が重要 |
| 2.1.1 キーボード | キーボードでも操作できる | 別要件。これだけでは 2.5.7 を満たさない |

- キーボード操作があっても、クリック／タップで同等操作ができなければ 2.5.7 は不適合
- タッチ端末では Tab や矢印キーが使えないことがある
- 軌道から外れると掴めなくなるスライダーは、2.5.1 と 2.5.7 の両方で問題になりうる

```text
よくある誤解：
  矢印キーでスライダーを動かせるから 2.5.7 は合格
  → キーボード代替であり、単一ポインタ代替ではない

  フリックでカルーセルを送れるからドラッグ代替になる
  → 軌跡ベースなので 2.5.1 の問題。前後ボタンが必要
```

> **参照**
> - [wcag/2-5-1-pointer-gestures.md — 2.5.1 ポインタのジェスチャ](./2-5-1-pointer-gestures.md)
> - [wcag/2-1-1-keyboard.md — 2.1.1 キーボード](./2-1-1-keyboard.md)

## 手法：ドラッグの代替を用意する（G219）

- ドラッグ自体は残してよい。同じ結果を単一ポインタでも達成できればよい
- ボタン、メニュー、トラック上のタップ、2ステップのクリックなどが代表例

### スライダー

```html
<!-- 良い例：トラックタップ + 数値入力でも値を変えられる -->
<label for="volume">音量 <span id="volume-value">50</span></label>
<input id="volume" type="range" min="0" max="100" value="50">
<input id="volume-number" type="number" min="0" max="100" value="50"
  aria-label="音量の数値">
```

```javascript
const slider = document.getElementById("volume");
const number = document.getElementById("volume-number");
const output = document.getElementById("volume-value");

function setVolume(value) {
  slider.value = String(value);
  number.value = String(value);
  output.textContent = String(value);
}

slider.addEventListener("input", () => setVolume(slider.value));
number.addEventListener("input", () => setVolume(number.value));
```

### 並べ替えリスト

```html
<!-- 良い例：ドラッグに加えて上下ボタン -->
<ul>
  <li>
    タスク A
    <button type="button" aria-label="タスク A を上へ">上へ</button>
    <button type="button" aria-label="タスク A を下へ">下へ</button>
  </li>
</ul>
```

### カンバン／ドロップ先の選択

```html
<!-- 良い例：タップ後に移動先を選ぶ -->
<article class="card">
  <h3>ログイン画面の改修</h3>
  <button type="button" aria-haspopup="menu" aria-controls="move-menu">
    列を移動
  </button>
  <ul id="move-menu" role="menu" hidden>
    <li role="menuitem"><button type="button">未着手へ</button></li>
    <li role="menuitem"><button type="button">進行中へ</button></li>
    <li role="menuitem"><button type="button">完了へ</button></li>
  </ul>
</article>
```

### 地図のパン

```html
<!-- 良い例：ドラッグパンに加えて方向ボタン -->
<div class="map" data-draggable="true"></div>
<div class="map-pan" role="group" aria-label="地図の移動">
  <button type="button" aria-label="上へ">↑</button>
  <button type="button" aria-label="左へ">←</button>
  <button type="button" aria-label="右へ">→</button>
  <button type="button" aria-label="下へ">↓</button>
</div>
```

### 2ステップの選択（ドラッグの代わり）

```javascript
// 良い例：押しっぱなしではなく、2回タップで矩形選択
let firstCorner = null;

canvas.addEventListener("click", (event) => {
  const point = getPoint(event);
  if (!firstCorner) {
    firstCorner = point;
    return;
  }
  selectRectangle(firstCorner, point);
  firstCorner = null;
});
```

> **参照**
> - [Technique G219: Ensuring that an alternative is available for dragging movements that operate on content](https://www.w3.org/WAI/WCAG22/Techniques/general/G219)
> - [Understanding SC 2.5.7 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements.html#examples)

## 例外

### 不可欠（Essential）

- ドラッグを除くと、情報や機能が本質的に変わる場合のみ
- 「ドラッグの方が直感的」だけでは不可欠ではない
- 手書きで線を引くこと自体が課題内容である、などごく限られた場合を慎重に検討する

### ユーザーエージェント制御

- ブラウザ標準のスクロールや、作者が変更していない UA 機能は対象外
- 作者が独自スクロールを実装し、UA のスクロールを抑止している場合は対象

```text
例外になりうる例：
  ブラウザ標準のページスクロール
  未改変のネイティブな overflow スクロール

例外にならない例：
  独自実装のドラッグ専用カルーセル
  タッチスクロールを抑止して自前ドラッグだけにする地図
```

## よくある失敗（F108）

- ドラッグだけでしか機能を達成できない
- 代替がキーボード操作だけ
- 代替がスワイプ／フリックだけ

```javascript
// 悪い例：ドラッグだけで並べ替え（F108）
item.addEventListener("pointerdown", startDrag);
item.addEventListener("pointermove", moveItem);
item.addEventListener("pointerup", dropItem);
// クリック／タップの代替なし
```

```html
<!-- 改善例：同じ機能をボタンでも提供 -->
<button type="button" aria-label="この項目を上へ">上へ</button>
```

> **参照**
> - [Failure F108: not providing a single pointer method that does not require a dragging movement](https://www.w3.org/WAI/WCAG22/Techniques/failures/F108)

## 実装時の注意点

- `draggable` や自前の pointer ドラッグを入れたら、必ずタップ代替を付ける
- 代替は同じページ上にあり、発見できること（視覚的に隠してもよいが、ポインタで操作できること）
- スライダーはトラッククリックを有効にするか、数値入力を併設する
- 並べ替え・カンバン・地図パン・カラーピッカーが重点対象
- 2.5.2（ポインタのキャンセル）も併せて、ドラッグを途中でやめられるようにする

```text
実務チェック：
  ドラッグしないとできない操作があるか
  同じことがクリック／タップでできるか
  代替がキーボードだけ／フリックだけになっていないか
  独自スクロールを実装して UA スクロールを消していないか
```

## テスト・確認方法

### 手動テスト

- マウスやタッチで、ドラッグが必要な UI を洗い出す
- 押したまま移動せず、クリック／タップだけで同じ結果に到達できるか確認する
- キーボードが使えないタッチ端末を想定して確認する
- スライダー、並べ替え、ドロップ、地図、カルーセルを重点的に見る

```text
チェックリスト：
1. ドラッグで操作する機能があるか
2. ドラッグなしの単一ポインタ代替があるか
3. 代替がキーボードだけになっていないか
4. 代替が軌跡ベースのジェスチャだけになっていないか
5. UA 標準スクロールと、作者実装のドラッグを切り分けているか
```

### 開発者ツールでの確認

```bash
# ドラッグ実装を探す
rg "draggable|dragstart|pointerdown|sortable|dnd|drag-and-drop" --glob "*.js" --glob "*.tsx" --glob "*.jsx" --glob "*.html" -i
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.5.7 は、ドラッグで操作する機能に、**ドラッグなしの単一ポインタ代替**を求める達成基準（レベル AA、WCAG 2.2 新規）
- ドラッグ自体は残してよい。ボタン、メニュー、トラックタップ、数値入力などで同等操作を提供する
- キーボード代替だけでは不十分。フリック代替は 2.5.1 の問題
- ブラウザ標準のスクロールは対象外。独自ドラッグ実装は対象
- 代替がないドラッグ専用 UI は F108 の失敗
