# prefers-reduced-motion — モーション設定への対応

## prefers-reduced-motionとは

- ユーザーがOSで設定した「視覚効果を減らす」設定を検知するCSSメディアクエリ
- ユーザーの明示的な意思表示（オプトイン）に基づいてアニメーション表示を制御する仕組み
- 値は `no-preference`（設定なし）と `reduce`（モーション軽減を希望）の2つ
- CSS Level 5 Media Queries 仕様で定義されたユーザー嗜好メディア特性のひとつ

> 参照: [MDN — prefers-reduced-motion](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-reduced-motion) / [W3C — Media Queries Level 5](https://www.w3.org/TR/mediaqueries-5/#prefers-reduced-motion)

## なぜ必要か

### 対象となるユーザーと症状

- **前庭障害（Vestibular Disorders）**: 画面上の動きがめまい・吐き気・平衡感覚の喪失を引き起こす
- **てんかん（Epilepsy）**: 点滅やフラッシュが光感受性発作のトリガーになり得る
- **注意欠陥障害（ADHD）**: 不要なアニメーションが注意力の分散を招く
- **片頭痛（Migraine）**: 視覚的な動きが片頭痛発作のトリガーになり得る
- **自閉スペクトラム症（ASD）**: 過度な視覚刺激が感覚過負荷につながる

### WCAG 2.2 での位置付け

- **2.3.1 3回の閃光、又は閾値以下（レベルA）**: 1秒間に3回以上の閃光を含むコンテンツの禁止
- **2.3.3 インタラクションによるアニメーション（レベルAAA）**: インタラクションで発生するアニメーションを無効化できること
- **2.2.2 一時停止、停止、非表示（レベルA）**: 自動的に動くコンテンツを一時停止・停止できること

> 参照: [WCAG 2.2 — Three Flashes or Below Threshold](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold.html) / [WCAG 2.2 — Animation from Interactions](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html) / [W3C — Understanding Vestibular Disorders](https://www.w3.org/WAI/WCAG21/Understanding/animation-from-interactions.html)

## 各OSでの設定方法

| OS | 設定パス |
|---|---|
| **macOS** | システム設定 → アクセシビリティ → ディスプレイ → 「視差効果を減らす」をオン |
| **Windows** | 設定 → アクセシビリティ → 視覚効果 → 「アニメーション効果」をオフ |
| **iOS** | 設定 → アクセシビリティ → 動作 → 「視差効果を減らす」をオン |
| **Android** | 設定 → ユーザー補助 → 「アニメーションの削除」をオン |

- いずれの設定もOSレベルの視覚効果を抑制し、同時にブラウザが `prefers-reduced-motion: reduce` として検知
- 開発中のテストは、OSの設定を切り替えるか、Chrome DevTools の Rendering タブで「Emulate CSS media feature prefers-reduced-motion」をエミュレート可能

> 参照: [web.dev — prefers-reduced-motion](https://web.dev/articles/prefers-reduced-motion)

## CSSでの使い方

### 基本構文

```css
@media (prefers-reduced-motion: reduce) {
  /* モーション軽減を希望するユーザー向けのスタイル */
}

@media (prefers-reduced-motion: no-preference) {
  /* モーション設定なし（デフォルト）のユーザー向けのスタイル */
}
```

### 対象となるCSSプロパティ

- `animation` / `animation-*`: CSSキーフレームアニメーション
- `transition` / `transition-*`: 状態遷移のトランジション
- `transform`: 要素の移動・回転・拡縮（アニメーションと組み合わせた場合）
- `scroll-behavior`: スムーススクロール
- `offset-path`: モーションパスに沿った移動

> 参照: [MDN — prefers-reduced-motion](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-reduced-motion)

## アニメーション削減のアプローチ

### アプローチ1: すべてのアニメーションを無効化する（ユニバーサルリセット）

```css
/* 簡易的な方法 — すべてのアニメーションを一括で無効化 */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

- 導入が簡単で、既存プロジェクトにもすぐに適用可能
- `0.01ms` にすることで `animationend` / `transitionend` イベントが発火し、JSの動作を壊さない
- ただし、意味のあるアニメーション（ローディング表示など）まで消えてしまう問題がある
- 粗い制御のため、あくまで暫定的な対応として位置付け

### アプローチ2: 個別にアニメーションを調整する（推奨）

```css
/* 良い例：個別に調整 — 必要なアニメーションは残し、過剰なものだけ抑制 */

/* 通常時のスタイル */
.hero-banner {
  animation: slideIn 0.6s ease-out;
}

.fade-in {
  animation: fadeIn 0.4s ease-in;
}

.loading-spinner {
  animation: spin 1s linear infinite;
}

/* モーション軽減時 */
@media (prefers-reduced-motion: reduce) {
  .hero-banner {
    animation: none; /* 大きな移動アニメーションは削除 */
  }

  .fade-in {
    animation-duration: 0.1s; /* フェードインは短縮して残す */
  }

  .loading-spinner {
    /* ローディングスピナーはそのまま維持 — 処理中であることの情報伝達に必要 */
  }
}
```

- コンポーネント単位で最適な対応を判断
- 情報を伝えるアニメーション（進捗表示、状態遷移の通知など）は維持
- 装飾的なアニメーション（パララックス、バウンスなど）は削除または簡素化

### 完全削除 vs 控えめに変更（推奨）

```css
/* 悪い例：アニメーションを完全に削除 — 状態変化がわかりにくくなる */
@media (prefers-reduced-motion: reduce) {
  .accordion-content {
    transition: none;
    /* 開閉が瞬時に切り替わり、何が起きたか把握しにくい */
  }
}

/* 良い例：動きを控えめに変更 — 情報は維持しつつ刺激を抑える */
@media (prefers-reduced-motion: reduce) {
  .accordion-content {
    transition: opacity 0.15s ease-in;
    /* スライドではなくフェードで切り替え — 動きは最小限だが状態変化は伝わる */
  }
}
```

```css
/* 悪い例：ページ遷移アニメーションを完全に削除 */
@media (prefers-reduced-motion: reduce) {
  .page-transition {
    animation: none;
    /* ページが瞬時に切り替わり、遷移したことに気づかないユーザーが出る */
  }
}

/* 良い例：大きな動きをフェードに置き換える */
@media (prefers-reduced-motion: reduce) {
  .page-transition {
    animation: fadeOnly 0.2s ease-in;
    /* 移動・拡縮を伴わない、短時間の透明度変化のみ */
  }
}

@keyframes fadeOnly {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

> 参照: [web.dev — prefers-reduced-motion](https://web.dev/articles/prefers-reduced-motion) / [A List Apart — Designing With Reduced Motion](https://alistapart.com/article/designing-safer-web-animation-for-motion-sensitivity/)

## スムーススクロールの制御

```css
/* 通常時はスムーススクロールを有効化 */
html {
  scroll-behavior: smooth;
}

/* モーション軽減時は即座にジャンプ */
@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
}
```

```css
/* より良い設計：デフォルトを auto にし、設定なしの場合のみスムーススクロール */
@media (prefers-reduced-motion: no-preference) {
  html {
    scroll-behavior: smooth;
  }
}
```

- `no-preference` 側にアニメーションを記述するアプローチは「モーション軽減がデフォルト」の発想
- 新規プロジェクトではこのアプローチが安全（設定の検知に失敗しても安全側に倒れる）

> 参照: [MDN — scroll-behavior](https://developer.mozilla.org/ja/docs/Web/CSS/scroll-behavior)

## JavaScriptでの検知方法

### window.matchMedia を使った検知

```js
// 現在の設定を取得
const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');

if (mediaQuery.matches) {
  // モーション軽減が有効
  console.log('ユーザーはモーション軽減を希望しています');
}
```

### 設定変更のリアルタイム検知

```js
const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');

// 悪い例：非推奨のaddListenerを使用
mediaQuery.addListener((e) => { /* ... */ });

// 良い例：addEventListenerを使用
mediaQuery.addEventListener('change', (event) => {
  if (event.matches) {
    // モーション軽減に切り替わった
    pauseAllAnimations();
  } else {
    // モーション軽減が解除された
    resumeAllAnimations();
  }
});
```

### 実用例: JSで制御するアニメーションの分岐

```js
// 悪い例：設定を無視してアニメーションを実行
function animateElement(element) {
  element.animate(
    [
      { transform: 'translateX(-100px)', opacity: 0 },
      { transform: 'translateX(0)', opacity: 1 }
    ],
    { duration: 600, easing: 'ease-out' }
  );
}

// 良い例：prefers-reduced-motion を考慮して分岐
function animateElement(element) {
  const prefersReducedMotion = window.matchMedia(
    '(prefers-reduced-motion: reduce)'
  ).matches;

  if (prefersReducedMotion) {
    // 移動を省略し、フェードインのみに簡素化
    element.animate(
      [
        { opacity: 0 },
        { opacity: 1 }
      ],
      { duration: 200, easing: 'ease-in' }
    );
  } else {
    element.animate(
      [
        { transform: 'translateX(-100px)', opacity: 0 },
        { transform: 'translateX(0)', opacity: 1 }
      ],
      { duration: 600, easing: 'ease-out' }
    );
  }
}
```

> 参照: [MDN — Window.matchMedia()](https://developer.mozilla.org/ja/docs/Web/API/Window/matchMedia)

## React / フレームワークでのカスタムフック実装

### usePrefersReducedMotion フック

```jsx
import { useState, useEffect } from 'react';

const QUERY = '(prefers-reduced-motion: reduce)';

function usePrefersReducedMotion() {
  // SSR対応: サーバー側ではデフォルトで true（安全側）を返す
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(
    typeof window === 'undefined' ? true : window.matchMedia(QUERY).matches
  );

  useEffect(() => {
    const mediaQuery = window.matchMedia(QUERY);

    const handleChange = (event) => {
      setPrefersReducedMotion(event.matches);
    };

    mediaQuery.addEventListener('change', handleChange);

    // マウント時に最新の値を反映
    setPrefersReducedMotion(mediaQuery.matches);

    return () => {
      mediaQuery.removeEventListener('change', handleChange);
    };
  }, []);

  return prefersReducedMotion;
}

export default usePrefersReducedMotion;
```

### フックの使用例

```jsx
function AnimatedCard({ children }) {
  const prefersReducedMotion = usePrefersReducedMotion();

  const style = {
    transition: prefersReducedMotion
      ? 'opacity 0.15s ease-in'
      : 'transform 0.4s ease-out, opacity 0.4s ease-out',
  };

  return (
    <div className="card" style={style}>
      {children}
    </div>
  );
}
```

```jsx
// Framer Motion との組み合わせ例
import { motion, useReducedMotion } from 'framer-motion';

function FadeSlideIn({ children }) {
  const shouldReduceMotion = useReducedMotion();

  const variants = {
    hidden: shouldReduceMotion
      ? { opacity: 0 }
      : { opacity: 0, y: 50 },
    visible: shouldReduceMotion
      ? { opacity: 1, transition: { duration: 0.2 } }
      : { opacity: 1, y: 0, transition: { duration: 0.5, ease: 'easeOut' } },
  };

  return (
    <motion.div initial="hidden" animate="visible" variants={variants}>
      {children}
    </motion.div>
  );
}
```

- Framer Motion は `useReducedMotion` フックを標準提供しており、自前で実装する必要がない
- SSR環境ではサーバー側で `window` が存在しないため、デフォルト値を安全側（モーション軽減）に設定

> 参照: [Framer Motion — useReducedMotion](https://www.framer.com/motion/use-reduced-motion/) / [Josh W Comeau — Accessible Animations in React](https://www.joshwcomeau.com/react/prefers-reduced-motion/)

## Web Animations API との組み合わせ

```js
// Web Animations API でモーション設定を尊重する汎用関数
function createAnimation(element, keyframes, options) {
  const prefersReducedMotion = window.matchMedia(
    '(prefers-reduced-motion: reduce)'
  ).matches;

  if (prefersReducedMotion) {
    // モーション軽減時: 移動・回転を含むキーフレームをフェードのみに変換
    const reducedKeyframes = keyframes.map((frame) => {
      const { transform, ...rest } = frame;
      return rest; // transformプロパティを除外
    });

    return element.animate(reducedKeyframes, {
      ...options,
      duration: Math.min(options.duration, 200), // 最大200ms
    });
  }

  return element.animate(keyframes, options);
}

// 使用例
const element = document.querySelector('.notification');

createAnimation(
  element,
  [
    { transform: 'translateY(-20px) scale(0.95)', opacity: 0 },
    { transform: 'translateY(0) scale(1)', opacity: 1 }
  ],
  { duration: 500, easing: 'ease-out', fill: 'forwards' }
);
```

### アニメーションの一時停止・再開

```js
// ページ内のすべてのアニメーションを制御
function getDocumentAnimations() {
  return document.getAnimations();
}

function pauseAllAnimations() {
  getDocumentAnimations().forEach((animation) => {
    animation.pause();
  });
}

function resumeAllAnimations() {
  getDocumentAnimations().forEach((animation) => {
    animation.play();
  });
}

// ユーザー設定の変更に連動
const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');

mediaQuery.addEventListener('change', (event) => {
  if (event.matches) {
    pauseAllAnimations();
  } else {
    resumeAllAnimations();
  }
});
```

> 参照: [MDN — Web Animations API](https://developer.mozilla.org/ja/docs/Web/API/Web_Animations_API) / [MDN — document.getAnimations()](https://developer.mozilla.org/ja/docs/Web/API/Document/getAnimations)

## 良い例・悪い例のまとめ

### CSS: パララックス効果

```css
/* 悪い例：モーション設定を無視してパララックスを適用 */
.parallax-bad {
  background-attachment: fixed;
  transform: translateZ(-1px) scale(2);
  /* 前庭障害を持つユーザーにめまいを引き起こす可能性 */
}

/* 良い例：モーション軽減時はパララックスを無効化 */
.parallax-good {
  background-attachment: fixed;
  transform: translateZ(-1px) scale(2);
}

@media (prefers-reduced-motion: reduce) {
  .parallax-good {
    background-attachment: scroll;
    transform: none;
  }
}
```

### CSS: ホバーアニメーション

```css
/* 悪い例：モーション設定を考慮しないホバーエフェクト */
.card-bad {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.card-bad:hover {
  transform: translateY(-8px) rotate(2deg);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.15);
}

/* 良い例：モーション軽減時は控えめなエフェクトに切り替え */
.card-good {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.card-good:hover {
  transform: translateY(-8px) rotate(2deg);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.15);
}

@media (prefers-reduced-motion: reduce) {
  .card-good {
    transition: box-shadow 0.2s ease;
    /* transformのトランジションは削除 */
  }
  .card-good:hover {
    transform: none;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    /* 影のみの変化にとどめ、移動・回転を排除 */
  }
}
```

### CSS: 自動再生アニメーション

```css
/* 悪い例：無限ループのアニメーションを常に表示 */
.marquee-bad {
  animation: scroll-text 10s linear infinite;
}

@keyframes scroll-text {
  from { transform: translateX(100%); }
  to { transform: translateX(-100%); }
}

/* 良い例：モーション軽減時は静止表示に切り替え */
.marquee-good {
  animation: scroll-text 10s linear infinite;
}

@media (prefers-reduced-motion: reduce) {
  .marquee-good {
    animation: none;
    /* テキストが静止した状態で表示される — 情報は失われない */
  }
}
```

### JavaScript: スクロール連動アニメーション

```js
// 悪い例：設定を無視したスクロール連動アニメーション
window.addEventListener('scroll', () => {
  const scrollY = window.scrollY;
  document.querySelector('.bg-layer').style.transform =
    `translateY(${scrollY * 0.5}px)`;
});

// 良い例：prefers-reduced-motion を確認してから適用
const prefersReducedMotion = window.matchMedia(
  '(prefers-reduced-motion: reduce)'
).matches;

if (!prefersReducedMotion) {
  window.addEventListener('scroll', () => {
    const scrollY = window.scrollY;
    document.querySelector('.bg-layer').style.transform =
      `translateY(${scrollY * 0.5}px)`;
  });
}
// モーション軽減時は背景がスクロールに連動せず、通常のスクロールのみ
```

> 参照: [Tatiana Mac — prefers-reduced-motion: Taking a No-Motion-First Approach](https://tatianamac.com/posts/prefers-reduced-motion/)

## 実装時の注意点

### モーションを減らすだけで情報は失わない

- アニメーションの削除は「動きを減らすこと」であり、「情報を減らすこと」ではない
- 状態変化（展開/折りたたみ、画面遷移など）はアニメーションがなくても伝わるようにする
- ローディングインジケーターのように進行状況を伝えるアニメーションは、代替手段を用意するか維持する

### reduce は「ゼロ」ではない

- `reduce` は「モーションを減らす」であり「モーションをゼロにする」ではない
- 短時間のフェードや控えめなトランジションは残しても問題ないケースが多い
- ユーザー体験を損なわない範囲で最小限のアニメーションを維持する判断が重要

### No-Motion-First アプローチ

- デフォルトでアニメーションなしのスタイルを記述し、`no-preference` の場合のみアニメーションを追加する設計
- メディアクエリの検知に失敗した場合も安全側に倒れる
- 新規プロジェクトでは推奨されるアプローチ

```css
/* No-Motion-First アプローチ */
.element {
  /* デフォルト: アニメーションなし */
  opacity: 1;
}

@media (prefers-reduced-motion: no-preference) {
  .element {
    /* モーション設定なしの場合のみアニメーションを適用 */
    animation: fadeSlideIn 0.5s ease-out;
  }
}
```

### テスト方法

- Chrome DevTools: Rendering タブ → 「Emulate CSS media feature prefers-reduced-motion」を `reduce` に設定
- Firefox DevTools: アクセシビリティタブからエミュレート可能
- OSの設定を実際に切り替えて、ブラウザでの挙動を確認
- `matchMedia` のリスナーが設定変更に正しく反応するかも確認

### ブラウザサポート

- すべてのモダンブラウザが対応（Chrome 74+, Firefox 63+, Safari 10.1+, Edge 79+）
- IE11 は非対応 — 未対応ブラウザではメディアクエリが無視されるため、デフォルトスタイルが適用される
- No-Motion-First アプローチなら未対応ブラウザでもアニメーションなしのフォールバックになる

> 参照: [Can I Use — prefers-reduced-motion](https://caniuse.com/prefers-reduced-motion) / [web.dev — prefers-reduced-motion](https://web.dev/articles/prefers-reduced-motion)

## まとめ

- `prefers-reduced-motion` はOSの「モーション軽減」設定を検知するメディアクエリ
- 前庭障害・てんかん・ADHD・片頭痛など、モーションによって身体的影響を受けるユーザーへの配慮
- CSSでは `@media (prefers-reduced-motion: reduce)` で分岐、JSでは `window.matchMedia()` で検知
- アニメーションの完全削除より、控えめな代替に置き換えるアプローチが推奨
- 新規プロジェクトでは No-Motion-First（デフォルトでアニメーションなし）が安全
- ローディング表示など情報伝達に必要なアニメーションは維持する
- モーションを減らしても、情報やコンテンツは失わない設計にする
