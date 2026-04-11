# scroll-driven animations — スクロール連動アニメーション

## scroll-driven animationsとは

- スクロール位置に連動してアニメーションを進行させるCSS機能
- 従来はJavaScript（Intersection Observer + scroll イベント）で実装していた処理をCSSだけで実現できる
- **時間ベース**ではなく**スクロール位置ベース**でアニメーションが進行する
- スクロールプログレスバー、パララックス効果、要素の登場演出などに活用

> 参照: [MDN - CSS scroll-driven animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations)

## 2つのタイムライン

- scroll-driven animationsには**2種類のタイムライン**がある

| タイムライン | 説明 | 主な用途 |
|---|---|---|
| `scroll()` | スクロールコンテナ全体の進行度（0%〜100%） | プログレスバー、パララックス |
| `view()` | 要素がビューポートに出入りする進行度 | 要素の登場・退場演出 |

```
scroll() タイムライン:
┌─────────────────────────┐
│ ページ先頭 (0%)         │
│                         │
│ ページ中間 (50%)        │
│                         │
│ ページ末尾 (100%)       │
└─────────────────────────┘

view() タイムライン:
                ┌─ viewport ─┐
  要素が         │            │
  入り始める(0%) │  要素      │ 出終わる(100%)
                │            │
                └────────────┘
```

> 参照: [web.dev - Scroll-driven animations](https://web.dev/articles/scroll-driven-animations)

## animation-timeline: scroll() — スクロール進行タイムライン

### 基本的な使い方

```css
@keyframes progress {
  from { width: 0%; }
  to { width: 100%; }
}

/* スクロールプログレスバー */
.progress-bar {
  position: fixed;
  top: 0;
  left: 0;
  height: 4px;
  background-color: #0066cc;
  animation: progress linear;
  animation-timeline: scroll();
  /* ページのスクロール量に応じてwidthが0%→100%に変化 */
}
```

- `animation-timeline: scroll()` を指定すると、`animation-duration` は無視される（スクロール量で進行が決まるため）
- `timing-function` に `linear` を指定するのが一般的（スクロール量と進行度を一致させるため）

### scroll() の引数

```css
.element {
  /* scroll(スクロール軸, スクロールコンテナ) */
  animation-timeline: scroll(y nearest);
}
```

| 引数 | 値 | 説明 |
|---|---|---|
| 軸 | `y`（デフォルト）, `x`, `block`, `inline` | スクロール方向 |
| コンテナ | `nearest`（デフォルト）, `root`, `self` | スクロールコンテナの指定 |

```css
/* ルート（ページ全体）のスクロールに連動 */
.element {
  animation-timeline: scroll(y root);
}

/* 横スクロールに連動 */
.element {
  animation-timeline: scroll(x nearest);
}
```

> 参照: [MDN - scroll()](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline/scroll)

## animation-timeline: view() — ビュー進行タイムライン

### 基本的な使い方

```css
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 要素がビューポートに入るとフェードイン */
.card {
  animation: fadeInUp linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}
```

- `view()` は要素がスクロールポート（ビューポート等）に出入りする進行度に基づく
- Intersection Observerで行っていた「スクロールして見えたらアニメーション」をCSSだけで実現

### view() の引数

```css
.element {
  /* view(軸) */
  animation-timeline: view(y);
  /* デフォルトはblock方向 */
}
```

> 参照: [MDN - view()](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline/view)

## animation-range — アニメーション範囲の制御

- `animation-range` はタイムラインのどの区間でアニメーションを実行するかを指定する

### view() で使用するレンジ名

```
             ┌─── viewport ───┐
             │                │
  ──entry──▶ │                │ ──exit──▶
  (要素が     │   ──contain──▶ │  (要素が
   入る区間)  │   (要素が完全に│   出る区間)
             │    見えている)  │
             └────────────────┘

  cover: entry開始 〜 exit終了（全区間）
  entry: 要素が入り始め〜完全に入るまで
  exit:  要素が出始め〜完全に出るまで
  contain: 要素が完全に見えている区間
```

```css
/* 要素がビューポートに入る途中でアニメーション完了 */
.card {
  animation: fadeInUp linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
  /* entry区間（入り始め〜完全に入るまで）でアニメーションが0%→100% */
}

/* 要素がビューポートの下25%〜上25%に入る間 */
.card {
  animation: fadeInUp linear both;
  animation-timeline: view();
  animation-range: entry 25% cover 50%;
}
```

### animation-range のショートハンド

```css
/* 個別指定 */
.element {
  animation-range-start: entry 0%;
  animation-range-end: entry 100%;
}

/* ショートハンド */
.element {
  animation-range: entry 0% entry 100%;
  /* animation-range: <start> <end>; */
}
```

> 参照: [MDN - animation-range](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-range)

## scroll-timeline / view-timeline — 名前付きタイムライン

- 特定のスクロールコンテナやビュー要素を名前で参照する場合に使用

### scroll-timeline

```css
.scroll-container {
  overflow-y: scroll;
  scroll-timeline-name: --my-scroll;
  scroll-timeline-axis: y;
}

.animated-child {
  animation: progress linear;
  animation-timeline: --my-scroll;
}
```

- 親要素のスクロールに子要素のアニメーションを連動させる場合に便利
- `--` プレフィックスでカスタム名を定義する（CSS変数と同様の命名規則）

### view-timeline

```css
.tracked-element {
  view-timeline-name: --my-view;
  view-timeline-axis: block;
}

.animated-element {
  animation: fadeIn linear both;
  animation-timeline: --my-view;
  animation-range: entry 0% entry 100%;
}
```

- 特定の要素の可視状態に別の要素のアニメーションを連動させる

> 参照: [MDN - scroll-timeline](https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-timeline)

## 実践的なパターン

### スクロールプログレスバー

```css
@keyframes scaleX {
  from { transform: scaleX(0); }
  to { transform: scaleX(1); }
}

.progress-bar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 4px;
  background-color: #0066cc;
  transform-origin: left;
  animation: scaleX linear;
  animation-timeline: scroll(y root);
}
```

- `scaleX` を使うことで `width` のアニメーションよりパフォーマンスが良い

### 要素のフェードイン登場演出

```css
@keyframes reveal {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.section {
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 10% entry 60%;
  /* ビューポートに10%入った時点から60%入った時点で完了 */
}
```

### パララックス効果

```css
@keyframes parallax {
  from { transform: translateY(-50px); }
  to { transform: translateY(50px); }
}

.background-image {
  animation: parallax linear;
  animation-timeline: view();
  /* 要素がビューポートを通過する間、背景がゆっくり移動 */
}
```

### 画像の拡大縮小効果

```css
@keyframes scaleOnScroll {
  from { transform: scale(0.8); }
  to { transform: scale(1); }
}

.hero-image {
  animation: scaleOnScroll linear both;
  animation-timeline: view();
  animation-range: entry 0% cover 50%;
}
```

### 横スクロールギャラリー

```css
.gallery-wrapper {
  overflow-x: scroll;
  scroll-timeline-name: --gallery;
  scroll-timeline-axis: x;
}

@keyframes fadeInItems {
  from { opacity: 0.3; transform: scale(0.9); }
  to { opacity: 1; transform: scale(1); }
}

.gallery-item {
  animation: fadeInItems linear both;
  animation-timeline: view(x);
  animation-range: contain 0% contain 50%;
}
```

> 参照: [web.dev - Scroll-driven animations](https://web.dev/articles/scroll-driven-animations)

## JavaScriptによる従来の実装との比較

```javascript
/* 従来のJavaScript実装（参考） */
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('visible');
    }
  });
}, { threshold: 0.1 });

document.querySelectorAll('.card').forEach(card => {
  observer.observe(card);
});
```

```css
/* CSS scroll-driven animationsでの同等実装 */
.card {
  animation: fadeInUp linear both;
  animation-timeline: view();
  animation-range: entry 10% entry 60%;
}
```

- JavaScriptが不要になり、コード量が大幅に削減される
- ブラウザのメインスレッドを使わないためパフォーマンスが向上する
- スクロール量に応じた連続的なアニメーション（スクラブ）がCSSだけで可能

## ブラウザサポート

- Chrome 115+、Edge 115+ でサポート
- Firefox 110+（`scroll()` と `view()` はフラグで有効化、Firefox 131+で一部正式サポート）
- **Safari は未サポート**（2025年5月時点）

### プログレッシブエンハンスメント

```css
/* フォールバック：animation-timelineをサポートしないブラウザでは通常のアニメーション */
.card {
  opacity: 1;
  transform: translateY(0);
}

/* scroll-driven animationsをサポートするブラウザのみ適用 */
@supports (animation-timeline: view()) {
  .card {
    animation: fadeInUp linear both;
    animation-timeline: view();
    animation-range: entry 10% entry 60%;
  }
}
```

- `@supports` で機能検出し、未対応ブラウザにはフォールバックを提供する
- 未対応ブラウザではアニメーションなしで静的に表示するのが安全

> 参照: [Can I Use - scroll-driven animations](https://caniuse.com/css-scroll-driven-animations)

## prefers-reduced-motionへの対応

```css
@supports (animation-timeline: view()) {
  .card {
    animation: fadeInUp linear both;
    animation-timeline: view();
    animation-range: entry 10% entry 60%;
  }

  @media (prefers-reduced-motion: reduce) {
    .card {
      animation: none;
      opacity: 1;
      transform: none;
    }
  }
}
```

- スクロール連動アニメーションは特に前庭機能障害のあるユーザーへの影響が大きい
- `prefers-reduced-motion: reduce` では無効化するのが望ましい

> 参照: [MDN - prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)

## まとめ

- scroll-driven animationsはスクロール位置に連動するCSSアニメーション機能
- `scroll()` はスクロールコンテナ全体の進行度、`view()` は要素の可視状態に連動
- `animation-range` で `entry`, `exit`, `contain`, `cover` の区間を制御する
- JavaScriptが不要になり、パフォーマンスも向上する
- **Safari未対応**のため `@supports` でプログレッシブエンハンスメントを行う
- `prefers-reduced-motion` でアクセシビリティに配慮する

## 参照元

- [MDN - CSS scroll-driven animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations)
- [MDN - animation-timeline](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline)
- [MDN - scroll()](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline/scroll)
- [MDN - view()](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline/view)
- [MDN - animation-range](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-range)
- [web.dev - Scroll-driven animations](https://web.dev/articles/scroll-driven-animations)
- [Can I Use - scroll-driven animations](https://caniuse.com/css-scroll-driven-animations)
