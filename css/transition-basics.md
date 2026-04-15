# transitionプロパティの基本

## transitionとは

- 要素のCSSプロパティが変化したとき、その変化を**滑らかにアニメーション**させる仕組み
- `:hover` や `class` の追加・削除など、状態変化のタイミングで自動的に補間される
- JavaScriptを使わず、CSSだけで手軽にアニメーションを実装できる
- ユーザー操作に対する視覚的フィードバックに最適

> 参照: [MDN - transition](https://developer.mozilla.org/en-US/docs/Web/CSS/transition)

## transitionの4つのサブプロパティ

- `transition` は4つのサブプロパティのショートハンド

```css
.element {
  transition: property duration timing-function delay;
}
```

| サブプロパティ | 説明 | 初期値 |
|---|---|---|
| `transition-property` | アニメーションさせるプロパティ名 | `all` |
| `transition-duration` | アニメーションにかかる時間 | `0s` |
| `transition-timing-function` | 変化の速度曲線 | `ease` |
| `transition-delay` | アニメーション開始までの待ち時間 | `0s` |

```css
/* 個別指定 */
.button {
  transition-property: background-color;
  transition-duration: 0.3s;
  transition-timing-function: ease;
  transition-delay: 0s;
}

/* ショートハンドで同じ意味 */
.button {
  transition: background-color 0.3s ease 0s;
}

/* duration以外デフォルトでよければ省略可 */
.button {
  transition: background-color 0.3s;
}
```

> 参照: [MDN - transition](https://developer.mozilla.org/en-US/docs/Web/CSS/transition)

## transition-property — 何をアニメーションさせるか

### allの問題点

```css
/* 悪い例：allで全プロパティを対象にする */
.card {
  transition: all 0.3s ease;
  /* 意図しないプロパティ（width, height, marginなど）まで
     アニメーションしてパフォーマンスに悪影響 */
}

/* 良い例：アニメーションさせたいプロパティを明示する */
.card {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  /* 明示的に指定することで意図が明確になり、パフォーマンスも向上 */
}
```

- `all` はすべてのアニメーション可能プロパティを対象にする
- 意図しないプロパティが変化したときに予期せぬアニメーションが発生する
- **パフォーマンス上も、対象プロパティを明示するのがベストプラクティス**

### 複数プロパティの指定

```css
.button {
  transition: background-color 0.3s ease,
              color 0.2s ease,
              transform 0.3s ease;
}

.button:hover {
  background-color: #0066cc;
  color: #ffffff;
  transform: translateY(-2px);
}
```

- カンマ区切りで複数のプロパティに異なる `duration` や `timing-function` を設定できる

> 参照: [MDN - transition-property](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-property)

## transition-duration — アニメーションの長さ

```css
/* 悪い例：遅すぎるアニメーション */
.button {
  transition: background-color 1.5s;
  /* 1.5秒は長すぎてUIが鈍く感じられる */
}

/* 良い例：適切な長さのアニメーション */
.button {
  transition: background-color 0.2s;
  /* 0.15s〜0.3sが自然に感じられる範囲 */
}
```

- **UIのインタラクション**: `0.15s` 〜 `0.3s` が推奨
- **装飾的なアニメーション**: `0.3s` 〜 `0.5s` 程度
- `0.1s` 以下は速すぎて変化が認識しづらい
- `0.5s` 以上はUIが鈍重に感じられる

> 参照: [Material Design - Duration](https://m3.material.io/styles/motion/duration-and-easing)

## transition-timing-function — 変化の速度曲線

### 組み込みのキーワード

```
ease        — ゆっくり始まり、途中で加速し、ゆっくり終わる（デフォルト）
linear      — 一定速度で変化する
ease-in     — ゆっくり始まり、加速して終わる
ease-out    — 速く始まり、ゆっくり終わる
ease-in-out — ゆっくり始まり、ゆっくり終わる
```

```css
/* 悪い例：UIの開始アニメーションにease-inを使う */
.dropdown {
  transition: opacity 0.3s ease-in;
  /* ease-inは最初が遅く反応が悪く感じる */
}

/* 良い例：UI要素の表示にはease-outを使う */
.dropdown {
  transition: opacity 0.3s ease-out;
  /* ease-outは最初が速く反応が良く感じる */
}
```

### 使い分けの目安

| 用途 | 推奨 timing-function |
|---|---|
| ホバーなどUIのフィードバック | `ease` または `ease-out` |
| 要素の出現（フェードイン等） | `ease-out` |
| 要素の退場（フェードアウト等） | `ease-in` |
| 移動アニメーション | `ease-in-out` |
| プログレスバーなど一定速度 | `linear` |

### cubic-bezier() でカスタム曲線を作る

```css
.element {
  /* cubic-bezier(x1, y1, x2, y2) — ベジェ曲線の制御点を指定 */
  transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  /* y値が1を超えるとオーバーシュート（行き過ぎて戻る）効果になる */
}
```

- [cubic-bezier.com](https://cubic-bezier.com/) でビジュアルにカーブを調整できる
- Material Designなど各デザインシステムで推奨カーブが公開されている

> 参照: [MDN - transition-timing-function](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function)

## transition-delay — アニメーション開始の遅延

```css
/* 複数要素を順番にアニメーションさせるスタガー効果 */
.list-item:nth-child(1) { transition-delay: 0s; }
.list-item:nth-child(2) { transition-delay: 0.05s; }
.list-item:nth-child(3) { transition-delay: 0.1s; }
.list-item:nth-child(4) { transition-delay: 0.15s; }

.list-item {
  opacity: 0;
  transform: translateY(10px);
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.list-item.visible {
  opacity: 1;
  transform: translateY(0);
}
```

- リスト項目を順番にフェードインさせるなど、スタガーアニメーションに有効
- `transition-delay` に負の値を指定すると、アニメーションの途中状態から開始できる

> 参照: [MDN - transition-delay](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-delay)

## アニメーション可能なプロパティと不可能なプロパティ

### アニメーション可能（数値的に補間できる）

```css
.card {
  transition: transform 0.3s, opacity 0.3s, background-color 0.3s;
}

.card:hover {
  transform: scale(1.05);    /* 数値の補間 → スムーズ */
  opacity: 0.8;              /* 数値の補間 → スムーズ */
  background-color: #f0f0f0; /* 色の補間 → スムーズ */
}
```

- `transform`, `opacity` — GPU合成で処理されパフォーマンスが最も良い
- `color`, `background-color` — 色の補間
- `width`, `height`, `margin`, `padding` — レイアウト再計算が発生するため重い

### アニメーション不可能（離散的な値）

```css
/* 悪い例：displayはアニメーションできない */
.modal {
  display: none;
  transition: display 0.3s; /* 効果なし */
}
.modal.open {
  display: block;
}

/* 良い例：opacityとvisibilityを使う */
.modal {
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}
.modal.open {
  opacity: 1;
  visibility: visible;
}
```

- `display` は `none` と `block` の間に中間値がないためアニメーションできない
- `visibility` は `hidden` → `visible` の切り替えタイミングを `transition-delay` で制御可能
- **注意**: CSS仕様の更新により `transition-behavior: allow-discrete` で `display` のアニメーションが可能になりつつある（後述）

> 参照: [MDN - CSS animated properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animated_properties)

## パフォーマンスの注意点

### レイアウトを引き起こすプロパティを避ける

```css
/* 悪い例：width/heightのアニメーション（レイアウト再計算が発生） */
.panel {
  width: 200px;
  transition: width 0.3s ease;
}
.panel:hover {
  width: 300px;
}

/* 良い例：transformで同等の効果を得る */
.panel {
  transform: scaleX(1);
  transition: transform 0.3s ease;
}
.panel:hover {
  transform: scaleX(1.5);
}
```

- `width`, `height`, `top`, `left` などはレイアウトの再計算（リフロー）を引き起こす
- `transform` と `opacity` はGPU合成レイヤーで処理され、リフローを引き起こさない
- **移動は `translate`、サイズ変更は `scale`、透明度は `opacity`** を優先する

### will-changeの活用と注意

```css
/* 頻繁にアニメーションする要素にのみ使用 */
.animated-element {
  will-change: transform, opacity;
  transition: transform 0.3s ease, opacity 0.3s ease;
}
```

- `will-change` はブラウザにアニメーションを事前通知してGPUレイヤーを確保させる
- 乱用するとメモリ消費が増えるため、実際にアニメーションする要素にのみ使う
- 動的に付け外しするのが理想的（ホバー時のみ追加するなど）

> 参照: [web.dev - Animations guide](https://web.dev/articles/animations-guide)

## transition-behavior: allow-discrete（新しい仕様）

- `display: none` や `overlay` など、従来アニメーションできなかった離散プロパティをトランジション可能にする新しい仕様

```css
/* 新しいアプローチ：displayをトランジションさせる */
.modal {
  opacity: 0;
  display: none;
  transition: opacity 0.3s ease,
              display 0.3s ease allow-discrete;
  /* allow-discreteにより、display: noneへの切り替えを
     opacityのトランジション終了まで遅延できる */
}

.modal.open {
  opacity: 1;
  display: block;
  /* @starting-styleで初期状態を定義する必要がある */
  @starting-style {
    opacity: 0;
  }
}
```

- Chrome 117+、Safari 17.4+、Firefox 129+ でサポート
- `@starting-style` と組み合わせて使うことが多い
- まだ広くサポートされていないため、フォールバックとして `opacity` + `visibility` パターンを併用するのが安全

> 参照: [MDN - transition-behavior](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-behavior)

## よくある実装パターン

### ホバーエフェクト

```css
.button {
  background-color: #0066cc;
  color: #ffffff;
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: background-color 0.2s ease,
              transform 0.2s ease,
              box-shadow 0.2s ease;
}

.button:hover {
  background-color: #0052a3;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
```

### フォーカスリングのアニメーション

```css
.input {
  outline: 2px solid transparent;
  outline-offset: 2px;
  transition: outline-color 0.2s ease;
}

.input:focus-visible {
  outline-color: #0066cc;
}
```

### アコーディオンの開閉

```css
.accordion-content {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 0.3s ease;
}

.accordion-content.open {
  grid-template-rows: 1fr;
}

.accordion-content > div {
  overflow: hidden;
}
```

- `grid-template-rows: 0fr → 1fr` のトランジションで高さを `0` から自動的にアニメーションできる
- `height: auto` へのトランジションが直接できない問題を解決する手法

> 参照: [MDN - transition](https://developer.mozilla.org/en-US/docs/Web/CSS/transition)

## prefers-reduced-motionへの対応

```css
/* アニメーションを控えめにするメディアクエリ */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    transition-duration: 0.01ms !important;
    animation-duration: 0.01ms !important;
  }
}
```

- `prefers-reduced-motion: reduce` は、前庭機能障害のあるユーザーなど、動きを最小限にしたいユーザー向け
- `0s` ではなく `0.01ms` にすると `transitionend` イベントが発火しJSの動作を壊さない
- WCAG 2.1 Success Criterion 2.3.3 に関連する

> 参照: [MDN - prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)

## まとめ

- `transition` はCSS状態変化を滑らかにアニメーションさせるプロパティ
- `transition-property` は `all` ではなく**対象を明示**する
- `duration` はUIフィードバックなら `0.15s` 〜 `0.3s` が適切
- `timing-function` は用途に応じて `ease-out`（出現）や `ease-in`（退場）を使い分ける
- **`transform` と `opacity`** をアニメーション対象にするとパフォーマンスが良い
- `prefers-reduced-motion` でアクセシビリティに配慮する

## 参照元

- [MDN - transition](https://developer.mozilla.org/en-US/docs/Web/CSS/transition)
- [MDN - transition-timing-function](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function)
- [MDN - CSS animated properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animated_properties)
- [MDN - transition-behavior](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-behavior)
- [MDN - prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)
- [web.dev - Animations guide](https://web.dev/articles/animations-guide)
- [Material Design - Duration and easing](https://m3.material.io/styles/motion/duration-and-easing)
- [cubic-bezier.com](https://cubic-bezier.com/)
