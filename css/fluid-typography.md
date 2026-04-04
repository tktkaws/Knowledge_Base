# clamp()を使ったフルイドタイポグラフィ

## フルイドタイポグラフィとは

- ビューポート幅に応じてフォントサイズが**滑らかに変化する**タイポグラフィ手法
- ブレイクポイントごとに段階的にサイズを切り替えるのではなく、連続的にスケールする
- 最小サイズと最大サイズの間を補間し、あらゆる画面幅で適切なサイズになる
- `clamp()` 関数の登場により、1行で簡潔に記述できるようになった

> 参照: [MDN - clamp()](https://developer.mozilla.org/en-US/docs/Web/CSS/clamp)

## 従来のアプローチとその問題

### メディアクエリによる段階的な変更

```css
/* 悪い例：ブレイクポイントでサイズが急に変わる */
h1 {
  font-size: 24px;
}

@media (min-width: 768px) {
  h1 {
    font-size: 32px; /* 768pxを境に急に変わる */
  }
}

@media (min-width: 1200px) {
  h1 {
    font-size: 48px; /* 1200pxを境に急に変わる */
  }
}
```

- ブレイクポイントの境目でフォントサイズが不連続に変化する
- ブレイクポイント間の中間幅では大きすぎたり小さすぎたりする
- ブレイクポイントが増えるとコードが冗長になる

### vw単位のみの使用

```css
/* 悪い例：vw単位だけで指定する */
h1 {
  font-size: 5vw;
  /* 320px幅 → 16px  → 小さすぎる場合がある */
  /* 1920px幅 → 96px → 大きすぎる */
  /* ユーザーがズームしても文字サイズが変わらない問題もある */
}
```

- 極端に小さい画面や大きい画面で不適切なサイズになる
- 最小・最大の制限がないため、アクセシビリティ上の問題がある
- `vw` のみの指定はブラウザのズーム機能に反応しない

> 参照: [web.dev - Responsive type and zoom](https://web.dev/articles/responsive-type-and-zoom)

## clamp() 関数の基本

- `clamp(最小値, 推奨値, 最大値)` の3つの引数を取る
- **推奨値**がメインの計算式で、**最小値**と**最大値**で上下限をクランプ（制限）する
- フォントサイズに限らず、あらゆるCSS値に使える

```
clamp(MIN, VAL, MAX)

VAL < MIN → MIN が適用される
MIN ≤ VAL ≤ MAX → VAL が適用される
VAL > MAX → MAX が適用される
```

```css
/* 良い例：clamp()でフルイドタイポグラフィ */
h1 {
  font-size: clamp(1.5rem, 1rem + 2vw, 3rem);
  /* 最小: 1.5rem（24px）— これ以下にはならない */
  /* 推奨: 1rem + 2vw — ビューポート幅に応じて変化 */
  /* 最大: 3rem（48px）— これ以上にはならない */
}
```

> 参照: [MDN - clamp()](https://developer.mozilla.org/en-US/docs/Web/CSS/clamp)

## 推奨値の計算方法

- 推奨値は `rem + vw` の組み合わせで指定するのがベストプラクティス
- `rem` を含めることで、ユーザーのフォントサイズ設定（ズーム）に対応できる

### 計算式の導出

特定のビューポート幅で特定のフォントサイズにしたい場合、連立方程式で求められる。

```
目標:
  - ビューポート幅 400px で 24px (1.5rem)
  - ビューポート幅 1200px で 48px (3rem)

式: font-size = a + b × ビューポート幅

24 = a + b × 400
48 = a + b × 1200

差分: 48 - 24 = b × (1200 - 400)
24 = b × 800
b = 0.03 = 3vw

a = 24 - 0.03 × 400 = 24 - 12 = 12px = 0.75rem

結果: font-size: clamp(1.5rem, 0.75rem + 3vw, 3rem)
```

```css
h1 {
  font-size: clamp(1.5rem, 0.75rem + 3vw, 3rem);
  /* 400px幅: 0.75rem + 3vw = 12px + 12px = 24px ✓ */
  /* 1200px幅: 0.75rem + 3vw = 12px + 36px = 48px ✓ */
  /* 800px幅: 0.75rem + 3vw = 12px + 24px = 36px（中間で滑らかに変化） */
}
```

> 参照: [CSS-Tricks - Linearly Scale font-size with CSS clamp()](https://css-tricks.com/linearly-scale-font-size-with-css-clamp-based-on-the-viewport/)

## 実践的なフルイドタイポグラフィの設定

### 見出し・本文のスケール設定

```css
:root {
  /* 本文 — 16px〜18px */
  --font-size-body: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);

  /* 小見出し — 18px〜24px */
  --font-size-h3: clamp(1.125rem, 0.95rem + 0.9vw, 1.5rem);

  /* 中見出し — 24px〜36px */
  --font-size-h2: clamp(1.5rem, 1.1rem + 2vw, 2.25rem);

  /* 大見出し — 32px〜60px */
  --font-size-h1: clamp(2rem, 1rem + 3.5vw, 3.75rem);
}

body {
  font-size: var(--font-size-body);
}

h1 { font-size: var(--font-size-h1); }
h2 { font-size: var(--font-size-h2); }
h3 { font-size: var(--font-size-h3); }
```

- CSS変数にまとめることで一元管理できる
- 見出しレベルが上がるほど変化幅を大きくすると、大画面での視覚的階層が際立つ

### line-heightもフルイドにする

```css
h1 {
  font-size: clamp(2rem, 1rem + 3.5vw, 3.75rem);
  line-height: clamp(1.1, 1.05 + 0.15vw, 1.3);
  /* 大きなフォントサイズほど行間の比率を小さくする */
  /* 小さいサイズ: line-height 1.3 → 読みやすい */
  /* 大きいサイズ: line-height 1.1 → 引き締まった見た目 */
}
```

> 参照: [Utopia - Fluid Type Scale Calculator](https://utopia.fyi/type/calculator/)

## アクセシビリティへの配慮

### rem を推奨値に含める理由

```css
/* 悪い例：vwだけの推奨値 */
h1 {
  font-size: clamp(1.5rem, 4vw, 3rem);
  /* ユーザーがブラウザの文字サイズを200%に拡大しても */
  /* 推奨値の 4vw は変化しない → ズームが効かないケースがある */
}

/* 良い例：rem + vw の推奨値 */
h1 {
  font-size: clamp(1.5rem, 0.75rem + 3vw, 3rem);
  /* ユーザーが文字サイズを拡大すると 0.75rem も拡大される */
  /* → ズーム時にフォントサイズが適切にスケールする */
}
```

- WCAG 2.1 Success Criterion 1.4.4 ではテキストの200%ズームをサポートする必要がある
- `vw` のみの指定はズームに反応しないため、`rem` を組み合わせることが重要

### 最小フォントサイズの確保

```css
/* 悪い例：最小値が小さすぎる */
p {
  font-size: clamp(0.625rem, 0.5rem + 1vw, 1.125rem);
  /* 最小 0.625rem = 10px → 読みにくい */
}

/* 良い例：本文は最小16px（1rem）を確保する */
p {
  font-size: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
  /* 最小 1rem = 16px → 十分な可読性 */
}
```

- 本文テキストは最低 `1rem`（16px）を確保する
- 見出しでも `1.25rem`（20px）以上が望ましい
- 補足テキストでも `0.875rem`（14px）を下回らないようにする

> 参照: [WCAG 2.1 - 1.4.4 Resize Text](https://www.w3.org/WAI/WCAG21/Understanding/resize-text.html)

## clamp() の便利ツール

### 計算が面倒な場合のツール

- 手計算が煩雑な場合は、オンラインツールで推奨値を算出できる

```
Utopia — Fluid Type Scale Calculator
https://utopia.fyi/type/calculator/
→ 最小・最大ビューポート幅とフォントサイズを入力すると clamp() を生成

Modern Fluid Typography Editor
https://modern-fluid-typography.vercel.app/
→ ビジュアルにフルイドタイポグラフィを設計できる
```

### CSS変数とcalc()を組み合わせた管理

```css
:root {
  --min-vw: 20;   /* 最小ビューポート幅: 320px → 320/16 = 20rem */
  --max-vw: 75;   /* 最大ビューポート幅: 1200px → 1200/16 = 75rem */
}

/* ヘルパーとして使う場合（Sass等のプリプロセッサーで生成するのがベター） */
h1 {
  font-size: clamp(2rem, 1rem + 3.5vw, 3.75rem);
}
```

## clamp() のフォントサイズ以外への応用

- `clamp()` はフォントサイズだけでなく、余白やコンテナ幅にも使える

```css
/* フルイドな余白 */
section {
  padding: clamp(1rem, 0.5rem + 2vw, 3rem);
}

/* フルイドなギャップ */
.grid {
  display: grid;
  gap: clamp(1rem, 0.5rem + 1.5vw, 2.5rem);
}

/* コンテナの最大幅 */
.container {
  width: clamp(320px, 90vw, 1200px);
  margin-inline: auto;
}
```

> 参照: [MDN - clamp()](https://developer.mozilla.org/en-US/docs/Web/CSS/clamp)

## まとめ

- フルイドタイポグラフィは `clamp(最小値, 推奨値, 最大値)` で実現する
- 推奨値は **`rem + vw`** の組み合わせにしてズーム対応を確保する
- メディアクエリによる段階的な変更より、`clamp()` による連続的な変化のほうが滑らか
- 本文テキストは最小 `1rem`（16px）以上を確保してアクセシビリティに配慮する
- CSS変数にまとめて一元管理するのがベストプラクティス
- フォントサイズだけでなく、余白やギャップにも応用可能

## 参照元

- [MDN - clamp()](https://developer.mozilla.org/en-US/docs/Web/CSS/clamp)
- [CSS-Tricks - Linearly Scale font-size with CSS clamp()](https://css-tricks.com/linearly-scale-font-size-with-css-clamp-based-on-the-viewport/)
- [web.dev - Responsive type and zoom](https://web.dev/articles/responsive-type-and-zoom)
- [WCAG 2.1 - 1.4.4 Resize Text](https://www.w3.org/WAI/WCAG21/Understanding/resize-text.html)
- [Utopia - Fluid Type Scale Calculator](https://utopia.fyi/type/calculator/)
