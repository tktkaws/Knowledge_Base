# 2.5.6 並行入力メカニズム（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.5.6「並行入力メカニズム」（Concurrent Input Mechanisms）
- レベル **AAA**（WCAG 2.1 で追加）
- 原則2「操作可能（Operable）」> ガイドライン 2.5「入力モダリティ」に属する
- Web コンテンツは、プラットフォームで使える入力方式の利用を制限しないこと
- 例外：制限が**本質的（essential）**、**セキュリティ**のため、または**ユーザー設定の尊重**のため必要な場合
- タッチ検出を理由に、マウスやキーボードを無効化しないことが実務の中心

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.6 Concurrent Input Mechanisms](https://www.w3.org/TR/WCAG22/#concurrent-input-mechanisms)
> - [Understanding SC 2.5.6 Concurrent Input Mechanisms](https://www.w3.org/WAI/WCAG22/Understanding/concurrent-input-mechanisms.html)

## 達成基準の原文（要約）

- Web コンテンツは、プラットフォームで利用可能な入力モダリティの使用を制限しない
- ただし、次の場合は除く
  - 制限が本質的である
  - コンテンツのセキュリティ確保のために必要である
  - ユーザー設定を尊重するために必要である

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.6](https://www.w3.org/TR/WCAG22/#concurrent-input-mechanisms)

## なぜ必要か

- 利用者は、同じ端末でも複数の入力手段を使い分ける
- スマホ・タブレットでも、外付けキーボードやマウス、スタイラスを使う人がいる
- 手の震えや巧緻運動の困難がある人は、タッチよりキーボード／トラックパッドの方が操作しやすいことがある
- 音声入力を一時オフにしてマウスに切り替える、など状況に応じた切替が必要
- 「タッチ端末だからタッチだけ」と決めつけると、他の入力手段を持つ人を排除する

> **参照**
> - [Understanding SC 2.5.6 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/concurrent-input-mechanisms.html#intent)
> - [Understanding SC 2.5.6 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/concurrent-input-mechanisms.html#benefits)

## 並行入力とは

- キーボード、マウス、タッチ、スタイラス、音声など、複数の入力手段を同時または切替で使えること
- 端末の**主入力**がタッチでも、他の入力を禁止してはならない
- 途中から入力手段を追加・削除しても、コンテンツ側が特定手段に固定しない

```text
想定される利用例：
  タッチスマホ + Bluetooth キーボード / マウス
  タッチ対応ノート PC でキーボードとタッチを併用
  最初はキーボード、途中からタッチモニターを接続
  音声入力とマウスを状況で切替
  マウスでメニューを開き、矢印キーで項目移動
```

> **参照**
> - [Understanding SC 2.5.6 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/concurrent-input-mechanisms.html#examples)

## 例外

| 例外 | 内容 | 例 |
|---|---|---|
| **本質的** | その入力に限定しないと機能の意味が崩れる | タッチタイピング練習アプリがキーボード入力を測る |
| **セキュリティ** | 特定入力の制限が安全確保に必要 | （限定的。安易に使わない） |
| **ユーザー設定** | ユーザー自身が選んだ設定を尊重する | ユーザーが意図的に入力モードを制限している場合 |

- 「モバイルだからタッチだけ」は本質的例外ではない
- タイピング速度測定など、測定対象そのものが特定入力の場合は例外になりうる

> **参照**
> - [Understanding SC 2.5.6 — Intent（essential note）](https://www.w3.org/WAI/WCAG22/Understanding/concurrent-input-mechanisms.html#intent)
> - [WCAG 2.2 — Glossary: essential](https://www.w3.org/TR/WCAG22/#dfn-essential)

## 十分な手法（方針）

- Understanding では、次が十分な手法（将来手法候補を含む）として示されている
  - `focus` / `blur` / `click` など、入力に依存しにくい高レベルイベントを使う
  - キーボード系とポインタ系のイベントハンドラを**同時に**登録する（Pointer Events の例など）
- 実務の要点は「どちらか一方だけ有効」にしないこと

```text
良い方針：
  click / pointer 系と keyboard 系の両方を常に使える
  タッチ用 UI を出しても、マウス・キーボード用コントロールを消さない
  入力手段の検出結果で、操作経路を片方に閉じない

悪い方針：
  タッチ検出時だけ touchend を登録し、click / keyup を登録しない
  pointer: coarse のとき操作ボタンを display: none
  maxTouchPoints > 0 ならキーボード対応を省略
```

> **参照**
> - [Understanding SC 2.5.6 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/concurrent-input-mechanisms.html#techniques)
> - [Pointer Events Level 2](https://www.w3.org/TR/pointerevents2/)

## 手法1：入力非依存のイベントを使う

- 可能なら `click`、`focus`、`blur`、`keydown` / `keyup` など、手段を問わない操作経路を用意する
- タッチ専用イベントだけに依存しない
- Pointer Events を使う場合も、キーボード操作を別途確保する

```js
// 良い例：手段を問わず動く基本操作
button.addEventListener('click', onActivate);

widget.addEventListener('keydown', (event) => {
  if (event.key === 'Enter' || event.key === ' ') {
    event.preventDefault();
    onActivate(event);
  }
});
```

```js
// 悪い例：タッチ検出で分岐し、片方だけ有効（F98）
if ('ontouchstart' in window) {
  target.addEventListener('touchend', onActivate);
} else {
  target.addEventListener('click', onActivate);
}
// タッチ端末 + 外付けマウス / キーボードで破綻しうる
```

> **参照**
> - [Failure F98: Interactions limited to touch-only on touchscreen devices](https://www.w3.org/WAI/WCAG22/Techniques/failures/F98)

## 手法2：タッチ検出後も他入力を残す

- `ontouchstart`、`maxTouchPoints`、`(pointer: coarse)` などでタッチを検出しても、マウス・キーボード経路を消さない
- カルーセルなどでスワイプを用意しても、前へ／次へボタンやキーボード操作を残す

```js
// 悪い例：タッチがあるからキーボード不要、と省略
if (navigator.maxTouchPoints > 0) {
  // キーボード監視なし
} else {
  target.addEventListener('keyup', onKeyNavigate);
}
```

```css
/* 悪い例：粗いポインタ環境で操作 UI を隠す */
@media (pointer: coarse) {
  #carousel .controls {
    display: none;
  }
}
/* 外付けキーボード利用者の操作手段まで消える */
```

```html
<!-- 良い例：ジェスチャがあっても見える操作を残す -->
<div class="carousel" tabindex="0" aria-roledescription="カルーセル">
  <button type="button" aria-label="前のスライド">前へ</button>
  <button type="button" aria-label="次のスライド">次へ</button>
  <!-- スワイプも可。ただしボタンとキーボードも可 -->
</div>
```

> **参照**
> - [Failure F98 — Examples](https://www.w3.org/WAI/WCAG22/Techniques/failures/F98)
> - [wcag/2-5-1-pointer-gestures.md — 2.5.1 ポインタのジェスチャ](./2-5-1-pointer-gestures.md)

## 手法3：途中での入力切替を妨げない

- マウスで開いたメニューをキーボードで移動できる、など手段の混在を許容する
- 「最初に検出した入力」だけを以後の唯一の手段にしない
- OS が入力デバイスの追加・削除をサポートするなら、コンテンツ側も追従できる設計にする

```text
良い体験：
  マウスでメニューを開く → 矢印キーで項目移動 → Enter で決定
  タッチでスクロール → 外付けキーボードでフォーム入力

悪い体験：
  初回がタッチだったので、以降クリックを無視する
  タッチモードに入ると、キーボードショートカットが全滅する
```

> **参照**
> - [Understanding SC 2.5.6 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/concurrent-input-mechanisms.html#examples)

## 2.1.1・2.5.1 との関係

| 達成基準 | 関係 |
|---|---|
| **2.1.1 キーボード** | キーボードで操作できること。タッチ専用化は多くの場合 2.1.1 も不合格 |
| **2.5.1 ポインタのジェスチャ** | 複雑なジェスチャに単一ポインタ代替が必要。代替 UI を隠すと両方不合格になりうる |
| **2.5.6 並行入力メカニズム** | 使える入力手段を制限しないこと（手段の共存・切替） |

- F98 の失敗は、状況によって 2.1.1 や 2.5.1 も同時に落とす
- 2.5.6 は「複数入力の同時利用・切替」に焦点がある AAA 基準

> **参照**
> - [wcag/2-1-1-keyboard.md — 2.1.1 キーボード](./2-1-1-keyboard.md)
> - [wcag/2-5-1-pointer-gestures.md — 2.5.1](./2-5-1-pointer-gestures.md)

## よくある失敗例（F98）

- タッチ検出時に `touchend` だけ登録し、`click` を登録しない
- `(pointer: coarse)` でマウス／キーボード用コントロールを非表示にする
- `maxTouchPoints > 0` のときキーボードイベントを省略する
- 「モバイル版」でスワイプのみのカルーセルにし、ボタンもキー操作もない
- タッチ端末判定で、デスクトップ向けの操作 UI を丸ごと消す

```js
// 失敗例：粗いポインタならタッチのみ
if (window.matchMedia('(pointer: coarse)').matches) {
  target.addEventListener('touchend', onActivate);
} else {
  target.addEventListener('click', onActivate);
}
```

```js
// 改善例：常に両方（または入力非依存イベント）を有効にする
target.addEventListener('click', onActivate);
target.addEventListener('keydown', (event) => {
  if (event.key === 'Enter' || event.key === ' ') {
    event.preventDefault();
    onActivate(event);
  }
});
// 必要なら touch / pointer も追加するが、他を無効化しない
```

> **参照**
> - [Failure F98](https://www.w3.org/WAI/WCAG22/Techniques/failures/F98)

## テスト・確認方法

### 手動テスト

- タッチスクリーンがあり、かつキーボードやマウスも使える端末で確認する
  - 例：タッチノート PC、スマホ／タブレット + 外付けキーボード／マウス
- すべてのインタラクティブ操作が、タッチ以外でもできるか確認する
- タッチ検出でコントロールが消える場合、代替操作があるか確認する
- 入力手段を途中で切り替えても操作できるか確認する

```text
チェックリスト：
1. タッチ + キーボード／マウス環境でテストしたか
2. タッチ専用イベントだけに依存していないか
3. タッチ検出後も click / keyboard が動くか
4. 操作ボタンを coarse pointer だけで隠していないか
5. ジェスチャ UI に、見える代替コントロールがあるか
6. 入力手段の途中切替ができるか
7. 本質的・セキュリティ・ユーザー設定の例外を誤用していないか
8. 2.1.1 / 2.5.1 も同時に満たしているか
```

> **参照**
> - [Failure F98 — Tests](https://www.w3.org/WAI/WCAG22/Techniques/failures/F98)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.5.6 は、プラットフォームで使える入力手段の利用を制限しない達成基準（レベル AAA）
- タッチ端末でも、マウス・キーボードなど他の入力を無効化しない
- 例外は、本質的・セキュリティ・ユーザー設定の尊重に限る
- `click` など入力非依存イベントの利用、キーボードとポインタの同時対応が基本
- タッチ検出で操作 UI やイベントを片方だけにするのは代表的な失敗（F98）
- 多くの失敗は 2.1.1 や 2.5.1 の不合格も伴う
