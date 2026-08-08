# 2.5.2 ポインタのキャンセル（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.5.2「ポインタのキャンセル」（Pointer Cancellation）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.5「入力モダリティ」に属する
- 単一ポインタで操作できる機能について、誤操作を防いだり取り消したりできること
- 次のいずれかが真であること（No Down-Event / Abort or Undo / Up Reversal / Essential）

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.2 Pointer Cancellation](https://www.w3.org/TR/WCAG22/#pointer-cancellation)
> - [Understanding SC 2.5.2 Pointer Cancellation](https://www.w3.org/WAI/WCAG22/Understanding/pointer-cancellation.html)

## 達成基準の原文（要約）

- 単一ポインタで操作できる機能について、次の少なくとも1つが真であること
  - **No Down-Event**：ポインタのダウンイベントで機能のどの部分も実行しない
  - **Abort or Undo**：アップイベントで完了し、完了前に中止できる、または完了後に元に戻せる
  - **Up Reversal**：アップイベントが、直前のダウンイベントの結果を取り消す
  - **Essential**：ダウンイベントでの完了が不可欠
- **注記1**：キーボードやテンキーのキー押下をエミュレートする機能は不可欠とみなす
- **注記2**：Webコンテンツがポインタ操作を解釈する場合に適用。UA／支援技術の操作は対象外

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.2](https://www.w3.org/TR/WCAG22/#pointer-cancellation)

## なぜ必要か

- 運動障害のある人は、意図せずタッチやクリックを始めやすい
- 視覚・認知の制約がある人も、間違った対象を押しやすい
- 押した瞬間に実行されると、途中でやめられない
- 離す前に指やポインタを外へ逃がせると、誤操作から回復しやすい

> **参照**
> - [Understanding SC 2.5.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/pointer-cancellation.html#benefits)

## ダウンイベントとアップイベント

| 用語 | 意味 | 例 |
|---|---|---|
| ダウンイベント | ポインタを押し下げる瞬間 | `mousedown` / `touchstart` / `pointerdown` |
| アップイベント | ポインタを離す瞬間 | `mouseup` / `touchend` / `pointerup` / `click` |

- JavaScript の `click` は、名前に反して通常**離したとき**に発火する
- タッチでも、対象内で指を離したときに相当する
- 最もアクセシブルな基本は、**アップイベントで実行する**こと

```text
推奨の流れ（単純なクリック相当）：
1. 押す（ダウン）→ 見た目の押し込みフィードバック
2. 間違えたら、対象の外へ移動する
3. 離す（アップ）→ 外で離したら実行しない／中止
```

> **参照**
> - [Understanding SC 2.5.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/pointer-cancellation.html#intent)

## 条件1：ダウンイベントでは実行しない（No Down-Event）

- 機能のどの部分も、ダウンイベントでは実行しない
- ネイティブのボタンやリンク、`click` ハンドラが代表例
- 最もシンプルで推奨される適合方法

```html
<!-- 良い例：ネイティブ button + click（アップ相当） -->
<button type="button" id="buy">購入する</button>
```

```javascript
// 良い例：click で実行（G212）
buyButton.addEventListener("click", () => {
  submitOrder();
});
```

```javascript
// 悪い例：押した瞬間に実行（F101）
buyButton.addEventListener("pointerdown", () => {
  submitOrder();
});
```

> **参照**
> - [Technique G212: Using native controls to ensure functionality is triggered on the up-event](https://www.w3.org/WAI/WCAG22/Techniques/general/G212)

## 条件2：中止または元に戻す（Abort or Undo）

- アップイベントで完了する
- かつ、完了前に中止できる、または完了後に取り消せる
- 単純なクリックでは、「対象の外で離す」こと自体が中止手段になる
- ドラッグ＆ドロップなど複雑な操作では、明示的な Undo や元位置への復帰が有効

```text
ドラッグ＆ドロップの良い流れ（G210）：
1. 押してカードを掴む
2. 移動する
3. ドロップ先で離す → 完了
   またはドロップ先の外／元の位置で離す → 中止（元に戻る）
4. 必要なら「元に戻す」ボタンや確認ダイアログを用意する
```

```javascript
// 良い例：対象外で離したら実行しない
button.addEventListener("pointerdown", (event) => {
  button.dataset.pressed = "true";
});

button.addEventListener("pointerup", (event) => {
  if (button.dataset.pressed !== "true") return;
  button.dataset.pressed = "false";

  const rect = button.getBoundingClientRect();
  const inside =
    event.clientX >= rect.left &&
    event.clientX <= rect.right &&
    event.clientY >= rect.top &&
    event.clientY <= rect.bottom;

  if (inside) {
    submitOrder();
  }
});
```

> **参照**
> - [Technique G210: Ensuring that drag-and-drop actions can be cancelled](https://www.w3.org/WAI/WCAG22/Techniques/general/G210)
> - [Understanding SC 2.5.2 — drag-and-drop example](https://www.w3.org/WAI/WCAG22/Understanding/pointer-cancellation.html#examples)

## 条件3：アップで取り消す（Up Reversal）

- ダウンで何かが起きても、アップでその結果が取り消される
- 例：長押し中だけポップアップや動画を表示し、離すと消える
- ユーザーは元の状態に戻るため、実質的にキャンセルできる

```javascript
// 良い例：押している間だけプレビュー表示
thumb.addEventListener("pointerdown", () => {
  preview.hidden = false;
});

thumb.addEventListener("pointerup", () => {
  preview.hidden = true; // ダウンの結果を取り消す
});

thumb.addEventListener("pointercancel", () => {
  preview.hidden = true;
});
```

## 条件4：ダウンでの完了が不可欠（Essential）

- アップイベントを使えないことが、機能の本質である場合のみ許容
- キーボードやテンキーのキー押下エミュレーションは、注記により不可欠とみなす
- 物理的な「押した瞬間」のタイミングが本質の体験も該当しうる

```text
例外になりうる例：
  スクリーン上のピアノ鍵盤（押した瞬間に音が出る）
  タイミングが重要な射撃ゲーム
  ソフトウェアキーボードで、キーダウンで文字入力する

例外にならない例：
  購入ボタンを mousedown で送信する
  メニュー項目を touchstart で即遷移する
  フォーム送信を pointerdown で行う
```

> **参照**
> - [Understanding SC 2.5.2 — Down-Event / Essential](https://www.w3.org/WAI/WCAG22/Understanding/pointer-cancellation.html#intent)

## よくある失敗（F101）

- コントロールをダウンイベントで起動する
- 押した瞬間に遷移・送信・削除などが起きる
- 対象の外へ逃がしても止められない

```javascript
// 失敗例（F101）
link.addEventListener("touchstart", () => {
  location.href = link.href;
});

dangerousButton.addEventListener("mousedown", () => {
  deleteAccount();
});
```

```javascript
// 改善例
link.addEventListener("click", () => {
  location.href = link.href;
});

dangerousButton.addEventListener("click", () => {
  if (confirm("アカウントを削除しますか？")) {
    deleteAccount();
  }
});
```

> **参照**
> - [Failure F101: activating a control on the down-event](https://www.w3.org/WAI/WCAG22/Techniques/failures/F101)

## 2.5.1・2.5.7 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.5.1 ポインタのジェスチャ | 複雑なジェスチャの単一点代替 | どう操作するか |
| 2.5.2 ポインタのキャンセル | 誤操作の中止・取り消し | 押し始めからの回復 |
| 2.5.7 ドラッグ操作 | ドラッグの代替 | ドラッグ移動そのもの |

- 2.5.2 は「押してから離すまでのキャンセルしやすさ」
- ドラッグがある UI では、2.5.2（中止／Undo）と 2.5.7（ドラッグ以外の手段）の両方を検討する
- ネイティブコントロールと `click` を使えば、2.5.2 は満たしやすい

> **参照**
> - [wcag/2-5-1-pointer-gestures.md — 2.5.1 ポインタのジェスチャ](./2-5-1-pointer-gestures.md)
> - [WCAG 2.2 — Success Criterion 2.5.7 Dragging Movements](https://www.w3.org/TR/WCAG22/#dragging-movements)

## 実装時の注意点

- まずネイティブの `<button>` `<a>` と `click` を使う
- カスタム実装でも、実行は `pointerup` / `click` 側に置く
- `pointerdown` / `touchstart` / `mousedown` では、見た目のフィードバックに留める
- ドラッグは、元位置へ戻せることや Undo を用意する
- 不可欠例外は、タイミングやキーエミュレーションなど本当に本質の場合に限る
- Undo は必須でない場面でも、あると望ましい

```text
実務チェック：
  押した瞬間に遷移・送信していないか
  間違えて押しても、外へ逃がしてキャンセルできるか
  ドラッグを途中でやめられるか
  down で動かしている機能に不可欠理由があるか
```

## テスト・確認方法

### 手動テスト

- マウスでボタンを押し、対象の外へ動かしてから離す
- 実行されなければ、単純クリック相当としては良い兆候
- タッチでも同様に、押したまま指を外へずらして離す
- ドラッグ＆ドロップは、元位置や無効領域で離して中止できるか確認する
- `mousedown` / `touchstart` だけで重要な処理が走っていないか確認する

```text
チェックリスト：
1. 単一ポインタで操作する機能があるか
2. ダウンではなくアップで完了しているか
3. 完了前の中止、または完了後の Undo があるか
4. ダウン完了なら、不可欠例外に当たるか
5. ネイティブ click に寄せられないか
```

### 開発者ツールでの確認

```bash
# ダウンイベントで実行していないか探す
rg "mousedown|touchstart|pointerdown" --glob "*.js" --glob "*.tsx" --glob "*.jsx"
```

```javascript
// 悪い兆候：down で遷移や送信をしている
element.addEventListener("pointerdown", dangerousAction);
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.5.2 は、単一ポインタ操作で誤作動しにくく、**中止や取り消しができる**ことを求める達成基準（レベル A）
- 最もよいのは、ダウンでは実行せず、アップ（`click` など）で完了すること
- 複雑な操作では Abort / Undo、一時表示では Up Reversal も有効
- ダウン完了は、キーボードエミュレーションやタイミングが本質の場合などに限る
- `pointerdown` で購入・削除・遷移する実装は F101 の典型的な失敗
