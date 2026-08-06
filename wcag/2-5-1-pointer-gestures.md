# 2.5.1 ポインタのジェスチャ（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.5.1「ポインタのジェスチャ」（Pointer Gestures）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.5「入力モダリティ」に属する
- マルチポイントや軌跡ベースのジェスチャで動く機能は、**単一ポインタ**かつ**軌跡不要**の操作でも使えること
- 例外：マルチポイント／軌跡ベースのジェスチャが**不可欠**な場合

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.1 Pointer Gestures](https://www.w3.org/TR/WCAG22/#pointer-gestures)
> - [Understanding SC 2.5.1 Pointer Gestures](https://www.w3.org/WAI/WCAG22/Understanding/pointer-gestures.html)

## 達成基準の原文（要約）

- マルチポイントまたは軌跡ベースのジェスチャで操作する機能はすべて、軌跡ベースでない単一ポインタでも操作できること
- ただし、マルチポイントまたは軌跡ベースのジェスチャが不可欠な場合は除く
- **注記**：Webコンテンツがポインタ操作を解釈する場合に適用。ユーザーエージェントや支援技術の操作に必要な動作は対象外

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.1](https://www.w3.org/TR/WCAG22/#pointer-gestures)

## なぜ必要か

- 複雑なジェスチャやマルチタッチを正確に行えない人がいる
- ヘッドポインタ、視線入力、音声マウスエミュレータなど、単一ポイントしか扱えない入力がある
- カスタムジェスチャの意図が分からない人も、タップやクリックなら使いやすい
- 認知・学習の制約がある人にも、単純な操作の方が分かりやすい

> **参照**
> - [Understanding SC 2.5.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/pointer-gestures.html#benefits)

## 対象になるジェスチャ

### 軌跡ベース（path-based）

- ポインタが**特定の経路・方向**に沿って動くことが本質の操作
- 開始点・終了点そのものより、「直線でフリックした」「決まった形を描いた」などが重要
- 例：横方向のスワイプ／フリック、決められた線に沿った移動、特定の形の描画

### マルチポイント（multipoint）

- 同時に**2点以上**を使う操作
- 例：2本指ピンチ／スプレッドズーム、1本指を置いたまま別の指でタップ、2〜3本指タップやスワイプ

```text
対象の例：
  地図のピンチズームだけ
  カルーセルの高速横フリックだけ
  画面上に特定の形を描かないと起動しない機能

対象外の例：
  OS の通知センターを出すスワイプ
  ブラウザの履歴戻る／進むフリック
  スクリーンリーダーのスワイプ操作
```

> **参照**
> - [Understanding SC 2.5.1 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/pointer-gestures.html#intent)

## 単一ポインタの代替とは

- クリック、タップ、ダブルクリック、ダブルタップ、長押し、クリック＆ホールドなど
- **軌跡を正確に辿らなくてよい**単一点操作
- ジェスチャ自体を禁止する必要はない。同じ結果を単一点でも達成できればよい

```text
許容される単一点操作の例：
  ＋／− ボタンでズーム
  前後ボタンでカルーセル移動
  メニュー項目のタップで機能実行

不十分な例（この基準では）：
  キーボードショートカットだけ用意する
  → 2.1.1 には有効でも、2.5.1 の代替にはならない
```

> **参照**
> - [Understanding SC 2.5.1 — Relationship to keyboard accessibility](https://www.w3.org/WAI/WCAG22/Understanding/pointer-gestures.html#intent)

## 軌跡ベースとドラッグの違い

| 種類 | 何が重要か | 主に関係する基準 |
|---|---|---|
| 軌跡ベース | 経路・方向・（場合により）速度 | **2.5.1** |
| ドラッグ | 掴む開始点と離す終了点 | **2.5.7**（WCAG 2.2） |

- ドラッグは、経路を厳密に辿らなくてもよい移動が多い
- 一般的なスライダーは、掴んだあと軌道から外れても値が変わる実装が多く、ドラッグ扱いになりやすい
- 軌道から外れると掴めなくなるスライダーは、軌跡ベースかつドラッグとして、2.5.1 と 2.5.7 の両方で問題になりうる
- 単一点の代替が「ドラッグだけ」だと、2.5.7 に抵触しうる

```text
ドラッグ（2.5.7）の例：
  リスト項目を掴んで別の位置へ落とす
  通常の range スライダー（軌道から外れても掴みが続く）

軌跡ベース（2.5.1）の例：
  ほぼ真横に一定速度でフリックしないと次へ進まない
  決められた曲線をなぞらないと認識されない操作
```

> **参照**
> - [Understanding SC 2.5.1 — path-based vs dragging](https://www.w3.org/WAI/WCAG22/Understanding/pointer-gestures.html#intent)
> - [WCAG 2.2 — Success Criterion 2.5.7 Dragging Movements](https://www.w3.org/TR/WCAG22/#dragging-movements)

## 手法1：同じ結果を出すコントロールを用意する（G215）

- ジェスチャと同じ結果を、ボタンやリンクなどの単一点操作で提供する
- 地図のズーム、カルーセルの前後移動などが典型

```html
<!-- 良い例：ピンチズームに加えて +/- ボタン -->
<div class="map" data-supports-pinch-zoom="true">
  <!-- 地図キャンバス -->
</div>
<div class="map-controls">
  <button type="button" id="zoom-in" aria-label="拡大">＋</button>
  <button type="button" id="zoom-out" aria-label="縮小">−</button>
</div>
```

```html
<!-- 良い例：横フリックに加えて前後ボタン -->
<section class="carousel" aria-roledescription="カルーセル">
  <button type="button" class="prev" aria-label="前のニュース">前へ</button>
  <ul class="slides">...</ul>
  <button type="button" class="next" aria-label="次のニュース">次へ</button>
</section>
```

```html
<!-- 悪い例：高速フリックだけでしか進めない（F105） -->
<div class="carousel" data-swipe-only="true">
  <!-- 前後ボタンなし -->
</div>
```

> **参照**
> - [Technique G215: Providing controls to achieve the same result as path based or multipoint gestures](https://www.w3.org/WAI/WCAG22/Techniques/general/G215)
> - [Understanding SC 2.5.1 — map / carousel examples](https://www.w3.org/WAI/WCAG22/Understanding/pointer-gestures.html#examples)

## 手法2：スライダーを単一点で操作できるようにする（G216）

- スライダーを、クリック／タップでも値変更できるようにする
- トラック上の位置をタップして値を設定する、増減ボタンを付ける、など
- キーボード操作（矢印キー）もあるとよいが、2.5.1 の代替としてはポインタの単一点操作が必要

```html
<!-- 良い例：スライダー + 増減ボタン -->
<label for="volume">音量</label>
<input id="volume" type="range" min="0" max="100" value="50">
<button type="button" id="volume-down" aria-label="音量を下げる">−</button>
<button type="button" id="volume-up" aria-label="音量を上げる">＋</button>
```

```javascript
// 良い例：トラッククリックで値を変更（概念）
track.addEventListener("pointerdown", (event) => {
  const ratio = event.offsetX / track.clientWidth;
  slider.value = String(Math.round(ratio * 100));
});
```

> **参照**
> - [Technique G216: Providing single point activation for a control slider](https://www.w3.org/WAI/WCAG22/Techniques/general/G216)

## 例外：不可欠なジェスチャ

- 取り除くと機能や情報が本質的に変わる場合のみ例外
- 例：手書き署名そのもの（経路が署名の内容）
- 「確認」「同意」「本人確認」は、署名以外の方法でも実現できることが多い
- 例外を広く解釈しない

```text
例外になりうる例：
  手書きで署名を描く入力（経路自体がデータ）

例外にならない例：
  ピンチでないとズームできない地図
  フリックでないと通れないカルーセル
  2本指でないと開けないメニュー
```

## キーボード対応との関係

- 2.1.1 / 2.1.3 のキーボード操作は別要件として必要
- ただし、キーボードだけで 2.5.1 を満たしたことにはならない
- ポインタしか使えない／ポインタの方が楽なユーザーがいるため
- ポインタをサポートするなら、その恩恵を単一点操作でも提供する

```text
必要なことの整理：
  2.1.1：キーボードでも操作できる
  2.5.1：複雑なジェスチャの単一点ポインタ代替がある
  2.5.7：ドラッグの代替がある（WCAG 2.2）
```

> **参照**
> - [wcag/2-1-1-keyboard.md — 2.1.1 キーボード](./2-1-1-keyboard.md)

## よくある失敗（F105）

- 軌跡ベースのジェスチャだけで機能を提供し、単純なポインタ代替がない
- マルチタッチ必須で、単一指／スタイラスでは使えない
- 代替がキーボードだけで、ポインタの単一点操作がない

```javascript
// 悪い例：横フリックだけ
element.addEventListener("pointerup", (event) => {
  if (isFastHorizontalFlick(event)) {
    goToNextSlide();
  }
});

// 良い例：ボタンでも同じ結果
nextButton.addEventListener("click", goToNextSlide);
```

> **参照**
> - [Failure F105: functionality via a path-based gesture without simple pointer alternative](https://www.w3.org/WAI/WCAG22/Techniques/failures/F105)

## 実装時の注意点

- カスタムスワイプ・ピンチ・図形ジェスチャを入れたら、必ずボタン等の代替を付ける
- 代替コントロールは、見える必要まではないが、ポインタで操作できること
- 見える前後ボタンの方が、発見しやすく望ましい
- OS／ブラウザ／支援技術のジェスチャは対象外なので、それらに頼って代替を省略しない
- ドラッグ操作は別途 2.5.7 を確認する

```text
実務チェック：
  ピンチ／スワイプ／多指操作があるか
  同じことがタップやクリックでできるか
  代替がドラッグだけになっていないか
  キーボード対応も別途あるか
```

## テスト・確認方法

### 手動テスト

- タッチやマウスで、複雑なジェスチャが必要な機能を洗い出す
- 単一指／単一クリックだけで同じ結果に到達できるか確認する
- ヘッドポインタ相当（単一点しか使えない）想定で操作できるか確認する
- 不可欠例外を主張する場合、本当に代替不可能か確認する

```text
チェックリスト：
1. マルチポイントや軌跡ベースの操作があるか
2. 単一点・軌跡不要の代替があるか
3. 代替がキーボードだけになっていないか
4. 代替がドラッグだけになっていないか（2.5.7）
5. OS／UA のジェスチャと混同していないか
```

### 開発者ツールでの確認

```bash
# スワイプ・ピンチ・ジェスチャ実装を探す
rg "swipe|pinch|gesture|touchstart|pointerdown|Hammer|interact\\.js" --glob "*.js" --glob "*.tsx" --glob "*.jsx" -i
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.5.1 は、マルチポイントや軌跡ベースのジェスチャに、**単一点・軌跡不要のポインタ代替**を求める達成基準（レベル A）
- ジェスチャ自体は残してよいが、同じ結果をタップやクリック等でも達成できること
- キーボード代替だけでは不十分。ドラッグ代替は 2.5.7 の領域
- 地図の +/-、カルーセルの前後ボタンなどが代表的な手法（G215）
- 手書き署名など、経路が本質の場合のみ不可欠例外を検討する
