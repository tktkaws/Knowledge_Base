# @keyframesとanimationプロパティ

## CSS Animationとは

- `@keyframes` で**アニメーションの各段階**を定義し、`animation` プロパティで要素に適用する仕組み
- `transition` が状態A→状態Bの変化を扱うのに対し、`animation` はより複雑な多段階のアニメーションを表現できる
- ユーザー操作に依存せず、ページ読み込み時や常時ループなど**自動的に再生**させることが可能

> 参照: [MDN - Using CSS animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animations/Using_CSS_animations)

## @keyframes — アニメーションの定義

### from / to の構文

```css
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
```

- `from` は `0%`、`to` は `100%` のエイリアス
- 2段階（開始→終了）のシンプルなアニメーションに適する

### パーセンテージの構文

```css
@keyframes bounce {
  0% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-30px);
  }
  60% {
    transform: translateY(-15px);
  }
  80% {
    transform: translateY(-5px);
  }
  100% {
    transform: translateY(0);
  }
}
```

- `0%` 〜 `100%` の任意の地点でスタイルを定義できる
- 中間点を設定することで、バウンスやパルスなど**非線形な動き**を表現可能

### 複数プロパティのアニメーション

```css
@keyframes slideInFade {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}
```

- 1つの `@keyframes` 内で複数のCSSプロパティを同時にアニメーションできる
- 関連するプロパティは1つの `@keyframes` にまとめるほうが管理しやすい

> 参照: [MDN - @keyframes](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes)

## animationプロパティの構成

- `animation` は8つのサブプロパティのショートハンド

| サブプロパティ | 説明 | 初期値 |
|---|---|---|
| `animation-name` | `@keyframes` の名前 | `none` |
| `animation-duration` | 1サイクルの長さ | `0s` |
| `animation-timing-function` | 速度曲線 | `ease` |
| `animation-delay` | 開始までの待ち時間 | `0s` |
| `animation-iteration-count` | 繰り返し回数 | `1` |
| `animation-direction` | 再生方向 | `normal` |
| `animation-fill-mode` | 開始前・終了後のスタイル | `none` |
| `animation-play-state` | 再生・一時停止 | `running` |

```css
/* ショートハンド */
.element {
  animation: slideInFade 0.5s ease-out 0.2s 1 normal forwards running;
  /*         name       dur  timing   delay count dir    fill    state */
}

/* 最低限の指定 */
.element {
  animation: slideInFade 0.5s ease-out forwards;
}
```

> 参照: [MDN - animation](https://developer.mozilla.org/en-US/docs/Web/CSS/animation)

## animation-iteration-count — 繰り返し回数

```css
/* 1回だけ再生（デフォルト） */
.element {
  animation: fadeIn 0.5s ease-out;
}

/* 3回繰り返す */
.element {
  animation: pulse 1s ease-in-out 3;
}

/* 無限ループ */
.loader {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
```

- `infinite` を指定するとアニメーションが永続的にループする
- ローディングスピナーやパルスアニメーションに使用

> 参照: [MDN - animation-iteration-count](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-iteration-count)

## animation-direction — 再生方向

```css
/* normal — 毎回 0% → 100% で再生（デフォルト） */
.element { animation-direction: normal; }

/* reverse — 毎回 100% → 0% で再生 */
.element { animation-direction: reverse; }

/* alternate — 奇数回は 0%→100%、偶数回は 100%→0% */
.element { animation-direction: alternate; }

/* alternate-reverse — alternateの逆 */
.element { animation-direction: alternate-reverse; }
```

```css
/* alternateの活用例：行って戻るアニメーション */
.float {
  animation: floatUpDown 2s ease-in-out infinite alternate;
}

@keyframes floatUpDown {
  from { transform: translateY(0); }
  to { transform: translateY(-10px); }
}
```

- `alternate` を使うと往復アニメーションを1つの `@keyframes` で表現できる
- `100%` → `0%` の戻りも滑らかに補間される

> 参照: [MDN - animation-direction](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-direction)

## animation-fill-mode — アニメーション前後のスタイル

```css
/* 悪い例：fill-modeなし → アニメーション終了後に元の状態に戻る */
.element {
  opacity: 1;
  animation: fadeIn 0.5s ease-out;
  /* fadeInの最終状態(opacity: 1)はアニメーション後に破棄される */
  /* 元のスタイルに戻るため、一瞬ちらつくことがある */
}

/* 良い例：forwardsで最終状態を維持する */
.element {
  opacity: 0;
  animation: fadeIn 0.5s ease-out forwards;
  /* forwards: アニメーション最終フレームのスタイルを維持 */
}
```

| 値 | アニメーション前 | アニメーション後 |
|---|---|---|
| `none` | 元のスタイル | 元のスタイルに戻る |
| `forwards` | 元のスタイル | **最終フレームを維持** |
| `backwards` | **最初のフレームを適用** | 元のスタイルに戻る |
| `both` | **最初のフレームを適用** | **最終フレームを維持** |

- `forwards` が最もよく使われる — アニメーション後の状態をそのまま保持する
- `backwards` は `animation-delay` がある場合に便利 — 待機中も最初のフレームが適用される
- `both` は `forwards` と `backwards` の両方を組み合わせた動作

> 参照: [MDN - animation-fill-mode](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode)

## animation-play-state — 再生の制御

```css
.animated-element {
  animation: rotate 3s linear infinite;
  animation-play-state: running;
}

/* ホバーで一時停止 */
.animated-element:hover {
  animation-play-state: paused;
}
```

- `running`（デフォルト）と `paused` の2値
- JavaScriptから動的に切り替えることで、再生・停止を制御できる
- 一時停止から再開すると、停止した位置からアニメーションが続行される

> 参照: [MDN - animation-play-state](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-play-state)

## transitionとanimationの使い分け

| 観点 | transition | animation |
|---|---|---|
| トリガー | 状態変化（:hover, class追加等） | 自動再生（ページ読み込み時等） |
| 制御 | 開始→終了の2状態間 | 複数の中間状態（keyframes） |
| ループ | 不可 | `infinite` で可能 |
| 再生方向 | 状態変化に依存 | `direction` で制御可能 |
| 適用場面 | UIフィードバック、ホバー効果 | ローディング、登場演出、装飾 |

```css
/* transition向き：ホバー時の色変化 */
.button {
  background-color: #0066cc;
  transition: background-color 0.2s ease;
}
.button:hover {
  background-color: #0052a3;
}

/* animation向き：ローディングスピナー */
.spinner {
  animation: spin 1s linear infinite;
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
```

> 参照: [MDN - Using CSS animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animations/Using_CSS_animations)

## 実践的なアニメーションパターン

### フェードイン + スライドアップ（登場演出）

```css
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card {
  animation: fadeInUp 0.5s ease-out forwards;
}

/* スタガー（順次表示）効果 */
.card:nth-child(1) { animation-delay: 0s; }
.card:nth-child(2) { animation-delay: 0.1s; }
.card:nth-child(3) { animation-delay: 0.2s; }
```

### パルスアニメーション（注意喚起）

```css
@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

.notification-badge {
  animation: pulse 2s ease-in-out infinite;
}
```

### スケルトンスクリーンのシマー

```css
@keyframes shimmer {
  from {
    background-position: -200% 0;
  }
  to {
    background-position: 200% 0;
  }
}

.skeleton {
  background: linear-gradient(
    90deg,
    #e0e0e0 25%,
    #f0f0f0 50%,
    #e0e0e0 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
}
```

### タイピングアニメーション

```css
@keyframes typing {
  from { width: 0; }
  to { width: 100%; }
}

@keyframes blink {
  0%, 100% { border-color: transparent; }
  50% { border-color: #333; }
}

.typing-text {
  width: 0;
  overflow: hidden;
  white-space: nowrap;
  border-right: 2px solid #333;
  animation: typing 3s steps(20) forwards,
             blink 0.8s step-end infinite;
}
```

- `steps()` は `timing-function` の一種で、滑らかではなく段階的に変化させる
- タイプライター効果やフレームアニメーションに使用

> 参照: [MDN - animation](https://developer.mozilla.org/en-US/docs/Web/CSS/animation)

## 複数アニメーションの適用

```css
.element {
  animation: fadeIn 0.5s ease-out forwards,
             slideUp 0.5s ease-out forwards,
             pulse 2s ease-in-out 1s infinite;
}
```

- カンマ区切りで複数のアニメーションを1つの要素に同時適用できる
- 各アニメーションに独立した `duration`、`delay`、`iteration-count` を設定可能
- 同じプロパティをアニメーションする場合は後に記述したものが優先される

## パフォーマンスの注意点

```css
/* 悪い例：レイアウトを変更するプロパティ */
@keyframes bad {
  from { width: 100px; height: 100px; }
  to { width: 200px; height: 200px; }
}

/* 良い例：GPU合成で処理されるプロパティ */
@keyframes good {
  from { transform: scale(1); }
  to { transform: scale(2); }
}
```

- `transform` と `opacity` のみをアニメーションするのがパフォーマンスのベストプラクティス
- `width`, `height`, `top`, `left` などはレイアウト再計算（リフロー）を引き起こす
- 多数の要素を同時にアニメーションする場合は特に注意が必要

> 参照: [web.dev - Animations guide](https://web.dev/articles/animations-guide)

## prefers-reduced-motionへの対応

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
  }
}
```

- ユーザーがOSの設定で「動きを減らす」を有効にしている場合に適用される
- 装飾的なアニメーションは無効化し、意味のあるアニメーション（プログレスバー等）は維持する選択も可能

```css
/* より細かい制御：装飾的なアニメーションだけ無効化 */
@media (prefers-reduced-motion: reduce) {
  .decorative-animation {
    animation: none;
  }

  /* プログレスバーなど機能的なアニメーションは維持 */
  .progress-bar {
    /* animation はそのまま */
  }
}
```

> 参照: [MDN - prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)

## まとめ

- `@keyframes` で**アニメーションの段階**を定義し、`animation` で要素に適用する
- `transition` は状態間の変化、`animation` は多段階の自律的なアニメーション向き
- `animation-fill-mode: forwards` で終了後の状態を維持するのが一般的
- `animation-direction: alternate` で往復アニメーションを簡潔に記述できる
- **`transform` と `opacity`** のみをアニメーション対象にするとパフォーマンスが良い
- `prefers-reduced-motion` で動きを減らす設定に対応する

## 参照元

- [MDN - @keyframes](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes)
- [MDN - animation](https://developer.mozilla.org/en-US/docs/Web/CSS/animation)
- [MDN - animation-fill-mode](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode)
- [MDN - Using CSS animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animations/Using_CSS_animations)
- [web.dev - Animations guide](https://web.dev/articles/animations-guide)
- [MDN - prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)
