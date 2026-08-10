# 2.5.4 動きによる起動（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.5.4「動きによる起動」（Motion Actuation）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.5「入力モダリティ」に属する
- 端末やユーザーの動きで操作できる機能は、**通常の UI 部品でも操作できる**こと
- さらに、誤作動を防ぐために**動きへの反応をオフにできる**こと
- 例外：アクセシビリティサポートされたインタフェース、または動きが不可欠な場合

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.4 Motion Actuation](https://www.w3.org/TR/WCAG22/#motion-actuation)
> - [Understanding SC 2.5.4 Motion Actuation](https://www.w3.org/WAI/WCAG22/Understanding/motion-actuation.html)

## 達成基準の原文（要約）

- 端末の動きまたはユーザーの動きで操作できる機能は、ユーザーインタフェースコンポーネントでも操作できること
- かつ、誤作動防止のため、動きへの反応を無効にできること
- ただし次の場合は除く
  - **Supported Interface**：アクセシビリティサポートされたインタフェース経由で動きを使う
  - **Essential**：動きが機能に不可欠で、使わないと活動が成立しない

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.4](https://www.w3.org/TR/WCAG22/#motion-actuation)

## なぜ必要か

- 端末を車椅子などに固定していると、振ったり傾けたりできない
- 運動障害のある人は、必要な動きを正確に行えないことがある
- 振戦などにより、意図せずセンサーが反応することがある
- 手や端末を動かせない状況でも、同じ機能を使える必要がある

> **参照**
> - [Understanding SC 2.5.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/motion-actuation.html#benefits)

## 対象になる動き・対象外

### 対象

- 端末のセンサー入力（加速度センサー、ジャイロなど）
- 端末を振る・傾ける・パンする操作
- カメラ等で認識する、意図的な手・顔・視線のジェスチャ

### 対象外になりやすいもの

- ジオロケーションやビーコンによる「空間の移動」そのもの
- QR コードや書類をカメラで捉えるための向き合わせ
- キーボード入力、マウス操作、タッチ操作、支援技術操作に付随する動き
  （Supported Interface 例外に直結）

```text
対象の例：
  端末を振ると Undo
  端末を傾けると次のページへ進む
  カメラに向かって手を振ると進む／戻る
  端末を動かして写真の視点を変える

対象外の例：
  歩数計の歩行検知（不可欠例外にもなりうる）
  QR コードを枠に入れるためのカメラ移動
  タッチスクロールのための指の動き
```

> **参照**
> - [Understanding SC 2.5.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/motion-actuation.html#intent)

## 要件は2つ

| 要件 | 内容 |
|---|---|
| UI 代替 | 動きと同じ機能を、ボタン等の通常 UI でも実行できる |
| 無効化 | 動きによる起動をオフにでき、誤作動を防げる |

- どちらか一方だけでは不十分なことが多い
- 無効化は、アプリ設定でも、OS の動き検知オフを尊重するのでもよい
- OS 設定を尊重する場合、アプリ側でそれを妨げる実装は避ける

```text
適合の形：
1. 振る / 傾ける / ジェスチャでも動く
2. 同じことをボタンやリンクでもできる
3. 「動き操作をオフ」にできる（または OS 設定に従う）
```

## 手法：通常コントロール + 設定でオフ（G213）

- 動きに依存する機能には、必ず従来型の UI を併設する
- アプリケーション設定で、動き起動を無効化できるようにする
- OS のアクセシビリティ設定（動きを減らす／モーション無効など）も尊重する

```html
<!-- 良い例：傾き操作の代替ボタン -->
<nav aria-label="ページ移動">
  <button type="button" id="prev-page">前のページ</button>
  <button type="button" id="next-page">次のページ</button>
</nav>

<form>
  <label>
    <input type="checkbox" id="enable-tilt" checked>
    端末を傾けてページ移動する
  </label>
</form>
```

```javascript
const enableTilt = document.getElementById("enable-tilt");

window.addEventListener("deviceorientation", (event) => {
  if (!enableTilt.checked) return; // 動き起動をオフにできる
  navigateByTilt(event);
});

document.getElementById("next-page").addEventListener("click", goNext);
document.getElementById("prev-page").addEventListener("click", goPrev);
```

```html
<!-- 良い例：Shake to Undo の代替 -->
<label for="memo">メモ</label>
<textarea id="memo"></textarea>
<button type="button" id="undo">入力を元に戻す</button>

<label>
  <input type="checkbox" id="shake-undo">
  端末を振って元に戻す
</label>
```

```text
良い例：
  傾きで写真をパンできる + 画面上のパンコントロールもある
  手振りで進む + 「次へ」ボタンもある
  設定で Shake to Undo をオフにできる
```

> **参照**
> - [Technique G213: Provide conventional controls and an application setting for motion activated input](https://www.w3.org/WAI/WCAG22/Techniques/general/G213)
> - [Understanding SC 2.5.4 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/motion-actuation.html#examples)
> - [MDN — Device orientation events](https://developer.mozilla.org/en-US/docs/Web/API/Device_orientation_events)

## 例外1：Supported Interface

- 支援技術やアクセシビリティサポートされたインタフェース経由の動きは例外
- キーボード・マウス・タッチ・AT 操作に必要な付随的な動きを対象から外す
- 「普通の UI 操作に伴う動き」まで無効化対象にする必要はない

## 例外2：Essential（不可欠）

- 動きを使わないと活動そのものが成立しない場合
- 例：端末の動きで歩数を数える歩数計
- 「便利だから動きだけ」は不可欠ではない

```text
例外になりうる例：
  歩数計測アプリの歩行検知

例外にならない例：
  振るだけでしか Undo できないメモアプリ
  傾けるだけでしか進めないギャラリー
  手振りだけでしか操作できないプレゼン
```

## よくある失敗（F106）

- 動きで起動できるが、無効化できない
- 動きの代替 UI がない
- OS の動き無効化設定を無視する、または妨げる

```javascript
// 悪い例：振るだけで Undo、オフにもボタン代替にもできない
window.addEventListener("devicemotion", (event) => {
  if (isShake(event)) {
    undoLastInput(); // 無効化手段なし
  }
});
```

```javascript
// 良い例：設定オフ + ボタン代替
window.addEventListener("devicemotion", (event) => {
  if (!shakeUndoEnabled) return;
  if (isShake(event)) undoLastInput();
});

undoButton.addEventListener("click", undoLastInput);
```

> **参照**
> - [Failure F106: inability to deactivate motion actuation](https://www.w3.org/WAI/WCAG22/Techniques/failures/F106)

## 2.5.1・2.3.3 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.5.1 ポインタのジェスチャ | 画面上の複雑なポインタジェスチャ | タッチ面での軌跡・多指 |
| 2.5.4 動きによる起動 | 端末／身体の動きセンサー | 振る・傾ける・カメラジェスチャ |
| 2.3.3 インタラクションによるアニメーション | 動きのアニメをオフにできる | 視覚的な動きの抑制 |

- 2.5.1 は画面上のポインタ操作、2.5.4 は端末や身体のモーション入力
- `prefers-reduced-motion` はアニメ抑制に有用だが、2.5.4 の「機能オフ」そのものとは別
- モーション機能の無効化は、明示的な設定または OS 設定の尊重で満たす

> **参照**
> - [wcag/2-5-1-pointer-gestures.md — 2.5.1 ポインタのジェスチャ](./2-5-1-pointer-gestures.md)
> - [wcag/2-3-3-animation-from-interactions.md — 2.3.3 インタラクションによるアニメーション](./2-3-3-animation-from-interactions.md)

## 実装時の注意点

- `DeviceOrientationEvent` / `DeviceMotionEvent` / カメラジェスチャを使ったら、必ず UI 代替を付ける
- 動き起動のオン／オフをユーザーが変えられるようにする
- デフォルトはオフ、または初回に確認を取る設計も検討する
- 固定端末・車載・キオスクでもボタン操作だけで完結するか確認する
- 権限プロンプト（センサー／カメラ）と、機能そのものの代替は別問題

```text
実務チェック：
  振る・傾ける・手振りで動く機能があるか
  同じことがボタン等でできるか
  動き起動をオフにできるか
  OS の無効化設定を妨げていないか
```

## テスト・確認方法

### 手動テスト

- モーションやカメラジェスチャに依存する機能を洗い出す
- 端末を動かさず、UI だけで同じ結果に到達できるか確認する
- 設定や OS 経由で動き起動をオフにできるか確認する
- オフにしたあと、誤って振っても機能が動かないか確認する
- 不可欠例外を主張する場合、活動の本質か確認する

```text
チェックリスト：
1. 端末／ユーザーの動きで起動する機能があるか
2. 通常の UI 部品でも同じ操作ができるか
3. 動きへの反応を無効化できるか
4. Supported Interface / Essential 例外に当たるか
5. OS の動き無効化を尊重しているか
```

### 開発者ツールでの確認

```bash
# モーション・向き・ジェスチャ実装を探す
rg "devicemotion|deviceorientation|DeviceMotion|DeviceOrientation|shake|gyroscope|accelerometer" --glob "*.js" --glob "*.tsx" --glob "*.jsx" -i
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.5.4 は、動きで起動する機能に**UI 代替**と**無効化手段**を求める達成基準（レベル A）
- 振る・傾ける・カメラジェスチャなどが対象。QR 読み取りの向き合わせや通常のタッチ操作は対象外
- 主な手法は、従来型コントロールの併設と設定でのオフ（G213）
- 無効化できない動き起動は F106 の失敗
- 歩数計など、動きが活動の本質である場合のみ不可欠例外を検討する
