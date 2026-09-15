# アクセシブルなカルーセル / スライダーの実装

## カルーセルとスライダーは別物

| UI | 用途 | 主なロール |
|---|---|---|
| カルーセル | 複数スライドの切り替え表示（画像・お知らせなど） | `region` / `group` + `aria-roledescription` |
| スライダー | 範囲内から数値を選ぶ入力（音量、価格帯など） | `role="slider"` または `<input type="range">` |

- 「スライダー」と呼んでカルーセルを実装していると、誤ったARIAを付けやすい
- 記事タイトルの「カルーセル / スライダー」は両方を扱うが、パターンは別々に実装する

## カルーセル

### カルーセルとは

- 複数のコンテンツを1領域で切り替え表示するUI
- 自動回転がある場合、一時停止・再開の手段が必須
- 前へ / 次へボタン、またはスライド選択コントロールで操作する

> 参照: [WAI-ARIA Authoring Practices — Carousel Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/carousel/)

### 必要なARIA属性

| 要素 | ロール / 属性 | 説明 |
|---|---|---|
| カルーセル全体 | `role="region"` または `<section>` + `aria-roledescription="carousel"` | カルーセルとして識別 |
| カルーセル全体 | `aria-label` / `aria-labelledby` | アクセシブルネーム（必須） |
| 各スライド | `role="group"` + `aria-roledescription="slide"` | スライドとして識別 |
| 各スライド | `aria-label`（例: 「3 / 5」） | 位置が分かる名前 |
| 操作ボタン | `<button>` | 前へ / 次へ / 自動回転の停止・開始 |

- `aria-roledescription` の値はページ言語に合わせる（日本語なら `"カルーセル"` / `"スライド"` でも可）
- スライド選択をタブUIにする場合は、各スライドを `role="tabpanel"` にし、選択UIはタブパターンに従う

### キーボード操作

| キー | 動作 |
|---|---|
| `Tab` / `Shift` + `Tab` | カルーセル内のフォーカス可能要素を順に移動 |
| `Enter` / `Space` | フォーカス中のボタンを実行 |

- 前へ / 次へ / 回転制御はネイティブ `<button>` なら追加のキー処理は不要
- ボタン操作後もフォーカスをボタンに残す（連続操作しやすくするため）

### 自動回転がある場合の必須要件

- フォーカスがカルーセル内に入ったら自動回転を**停止**する
- ポインタがカルーセル上にある間も停止する（ホバーで再開してよい場合もあるが、フォーカス停止は必須）
- 停止 / 開始を明示的に切り替えるボタンを置く
- ユーザーが停止したあと、勝手に再開しない
- WCAG 2.2.2（一時停止、停止、非表示）に該当する

> 参照: [Understanding SC 2.2.2: Pause, Stop, Hide](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html)

### HTML構造（前へ / 次へボタン型）

```html
<section
  class="carousel"
  aria-roledescription="carousel"
  aria-label="特集バナー"
>
  <div class="carousel-controls">
    <button type="button" class="carousel-prev" aria-label="前のスライド">
      前へ
    </button>
    <button
      type="button"
      class="carousel-toggle"
      aria-label="スライドの自動回転を停止"
    >
      停止
    </button>
    <button type="button" class="carousel-next" aria-label="次のスライド">
      次へ
    </button>
  </div>

  <div class="carousel-slides">
    <div
      role="group"
      aria-roledescription="slide"
      aria-label="1 / 3"
    >
      <h2>春の新作</h2>
      <a href="/spring">詳細を見る</a>
    </div>
    <div
      role="group"
      aria-roledescription="slide"
      aria-label="2 / 3"
      hidden
    >
      <h2>セール情報</h2>
      <a href="/sale">詳細を見る</a>
    </div>
    <div
      role="group"
      aria-roledescription="slide"
      aria-label="3 / 3"
      hidden
    >
      <h2>会員特典</h2>
      <a href="/members">詳細を見る</a>
    </div>
  </div>
</section>
```

### Vanilla JS での実装（抜粋）

```js
class AccessibleCarousel {
  constructor(root, { interval = 5000, autoStart = true } = {}) {
    this.root = root;
    this.slides = [...root.querySelectorAll('[aria-roledescription="slide"]')];
    this.prevBtn = root.querySelector('.carousel-prev');
    this.nextBtn = root.querySelector('.carousel-next');
    this.toggleBtn = root.querySelector('.carousel-toggle');
    this.index = 0;
    this.interval = interval;
    this.timer = null;
    this.userPaused = false;
    this.autoPlayEnabled = autoStart;

    this.init();
  }

  init() {
    this.show(0);

    this.prevBtn.addEventListener('click', () => this.show(this.index - 1));
    this.nextBtn.addEventListener('click', () => this.show(this.index + 1));
    this.toggleBtn.addEventListener('click', () => this.toggleAutoPlay());

    // フォーカス / ホバー中は自動回転を止める
    this.root.addEventListener('focusin', () => this.pause());
    this.root.addEventListener('focusout', (event) => {
      if (!this.root.contains(event.relatedTarget)) this.maybeResume();
    });
    this.root.addEventListener('pointerenter', () => this.pause());
    this.root.addEventListener('pointerleave', () => this.maybeResume());

    if (this.autoPlayEnabled) this.start();
  }

  show(nextIndex) {
    const total = this.slides.length;
    this.index = ((nextIndex % total) + total) % total;

    this.slides.forEach((slide, i) => {
      const selected = i === this.index;
      slide.hidden = !selected;
      slide.setAttribute('aria-label', `${i + 1} / ${total}`);
    });
  }

  start() {
    this.stopTimer();
    this.timer = setInterval(() => this.show(this.index + 1), this.interval);
    this.toggleBtn.setAttribute('aria-label', 'スライドの自動回転を停止');
    this.toggleBtn.textContent = '停止';
  }

  pause() {
    this.stopTimer();
  }

  maybeResume() {
    if (this.autoPlayEnabled && !this.userPaused) this.start();
  }

  toggleAutoPlay() {
    this.userPaused = !this.userPaused;
    this.autoPlayEnabled = !this.userPaused;

    if (this.autoPlayEnabled) {
      this.start();
    } else {
      this.pause();
      this.toggleBtn.setAttribute('aria-label', 'スライドの自動回転を開始');
      this.toggleBtn.textContent = '再生';
    }
  }

  stopTimer() {
    clearInterval(this.timer);
    this.timer = null;
  }
}
```

### prefers-reduced-motion への対応

```js
const prefersReducedMotion = window.matchMedia(
  '(prefers-reduced-motion: reduce)'
).matches;

new AccessibleCarousel(document.querySelector('.carousel'), {
  autoStart: !prefersReducedMotion,
});
```

- モーション軽減設定がある場合は、自動回転を最初からオフにする

## スライダー

### スライダーとは

- 連続または離散的な範囲から値を選ぶ入力コントロール
- 音量、明るさ、価格フィルタ、進捗の調整などに使う
- 可能ならネイティブの `<input type="range">` を優先する（ARIAファーストルール）

> 参照: [WAI-ARIA Authoring Practices — Slider Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/slider/)

### ネイティブ実装（推奨）

```html
<label for="volume">音量</label>
<input
  id="volume"
  type="range"
  min="0"
  max="100"
  step="1"
  value="50"
>
<output for="volume" id="volume-value">50</output>
```

```js
const input = document.getElementById('volume');
const output = document.getElementById('volume-value');

input.addEventListener('input', () => {
  output.textContent = input.value;
});
```

- キーボード操作（矢印・Home・Endなど）はブラウザが提供
- ラベルは `<label>` で紐付ける
- 現在値の視覚表示は `<output>` や隣接テキストで補う

### カスタムスライダーが必要な場合のARIA

| 属性 | 説明 |
|---|---|
| `role="slider"` | スライダーであることを示す |
| `aria-valuemin` | 最小値 |
| `aria-valuemax` | 最大値 |
| `aria-valuenow` | 現在値 |
| `aria-valuetext` | 数値だけでは分かりにくいときの文言（任意） |
| `aria-label` / `aria-labelledby` | 名前 |
| `aria-orientation="vertical"` | 縦向きの場合（省略時は横向き） |
| `tabindex="0"` | フォーカス可能にする |

### カスタムスライダーのキーボード操作

| キー | 動作 |
|---|---|
| `→` / `↑` | 値を1ステップ増やす |
| `←` / `↓` | 値を1ステップ減らす |
| `Home` | 最小値へ |
| `End` | 最大値へ |
| `PageUp`（任意） | 大きめに増やす |
| `PageDown`（任意） | 大きめに減らす |

### カスタム実装例

```html
<span id="price-label">最大価格</span>
<div
  id="price-slider"
  role="slider"
  tabindex="0"
  aria-valuemin="0"
  aria-valuemax="10000"
  aria-valuenow="5000"
  aria-valuetext="5,000円"
  aria-labelledby="price-label"
></div>
```

```js
const slider = document.getElementById('price-slider');
const min = 0;
const max = 10000;
const step = 100;
const pageStep = 1000;

function setValue(value) {
  const next = Math.min(max, Math.max(min, value));
  slider.setAttribute('aria-valuenow', String(next));
  slider.setAttribute('aria-valuetext', `${next.toLocaleString('ja-JP')}円`);
  // 見た目のつまみ位置も更新する
}

slider.addEventListener('keydown', (event) => {
  const now = Number(slider.getAttribute('aria-valuenow'));

  switch (event.key) {
    case 'ArrowRight':
    case 'ArrowUp':
      event.preventDefault();
      setValue(now + step);
      break;
    case 'ArrowLeft':
    case 'ArrowDown':
      event.preventDefault();
      setValue(now - step);
      break;
    case 'Home':
      event.preventDefault();
      setValue(min);
      break;
    case 'End':
      event.preventDefault();
      setValue(max);
      break;
    case 'PageUp':
      event.preventDefault();
      setValue(now + pageStep);
      break;
    case 'PageDown':
      event.preventDefault();
      setValue(now - pageStep);
      break;
    default:
      break;
  }
});
```

## React でのカルーセル実装例

```jsx
import { useEffect, useState } from 'react';

function Carousel({ slides, interval = 5000 }) {
  const [index, setIndex] = useState(0);
  const [autoPlay, setAutoPlay] = useState(() => {
    return !window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  });
  const [pausedByInteraction, setPausedByInteraction] = useState(false);

  useEffect(() => {
    if (!autoPlay || pausedByInteraction) return undefined;

    const id = setInterval(() => {
      setIndex((current) => (current + 1) % slides.length);
    }, interval);

    return () => clearInterval(id);
  }, [autoPlay, pausedByInteraction, interval, slides.length]);

  return (
    <section
      aria-roledescription="carousel"
      aria-label="特集バナー"
      onFocusCapture={() => setPausedByInteraction(true)}
      onBlurCapture={(event) => {
        if (!event.currentTarget.contains(event.relatedTarget)) {
          setPausedByInteraction(false);
        }
      }}
      onPointerEnter={() => setPausedByInteraction(true)}
      onPointerLeave={() => setPausedByInteraction(false)}
    >
      <button
        type="button"
        aria-label="前のスライド"
        onClick={() =>
          setIndex((current) => (current - 1 + slides.length) % slides.length)
        }
      >
        前へ
      </button>
      <button
        type="button"
        aria-label={
          autoPlay
            ? 'スライドの自動回転を停止'
            : 'スライドの自動回転を開始'
        }
        onClick={() => setAutoPlay((value) => !value)}
      >
        {autoPlay ? '停止' : '再生'}
      </button>
      <button
        type="button"
        aria-label="次のスライド"
        onClick={() => setIndex((current) => (current + 1) % slides.length)}
      >
        次へ
      </button>

      {slides.map((slide, i) => (
        <div
          key={slide.id}
          role="group"
          aria-roledescription="slide"
          aria-label={`${i + 1} / ${slides.length}`}
          hidden={i !== index}
        >
          {slide.content}
        </div>
      ))}
    </section>
  );
}
```

## よくある間違い

### 1. カルーセルに `role="slider"` を付ける

```html
<!-- 間違い：スライダーは数値選択用 -->
<div role="slider" aria-valuenow="2" aria-valuemin="1" aria-valuemax="5">
  ...
</div>

<!-- 正解：カルーセルには roledescription を使う -->
<section aria-roledescription="carousel" aria-label="ニュース">
  ...
</section>
```

### 2. 自動回転を止められない

- 停止ボタンがない
- フォーカスしても回り続ける
- WCAG 2.2.2 違反になりやすい

### 3. 非表示スライドを視覚的に隠すだけにする

```css
/* 間違い：スクリーンリーダーが全スライドを読む */
.slide:not(.is-active) {
  opacity: 0;
  pointer-events: none;
}
```

```html
<!-- 正解 -->
<div role="group" aria-roledescription="slide" hidden>...</div>
```

### 4. 操作ボタンに名前がない

```html
<!-- 間違い -->
<button>‹</button>
<button>›</button>

<!-- 正解 -->
<button type="button" aria-label="前のスライド">‹</button>
<button type="button" aria-label="次のスライド">›</button>
```

### 5. カスタムスライダーで aria-valuenow を更新しない

- 見た目だけ動いても、支援技術には値が伝わらない
- `aria-valuenow` / 必要なら `aria-valuetext` を同期する

### 6. ネイティブ range で十分なのにカスタム実装する

- キーボード・タッチ・アクセシビリティツリー対応を自前で再実装するコストが大きい
- デザイン制約がない限り `<input type="range">` を使う

## 実装チェックリスト

### カルーセル

- [ ] コンテナに `aria-roledescription="carousel"` とアクセシブルネームがある
- [ ] 各スライドに `role="group"` + `aria-roledescription="slide"` と位置ラベルがある
- [ ] 前へ / 次へが `<button>` で、名前がある
- [ ] 自動回転がある場合、停止 / 開始ボタンがある
- [ ] フォーカス・ホバー中は自動回転が止まる
- [ ] ユーザー停止後に勝手に再開しない
- [ ] 非表示スライドに `hidden` がある
- [ ] `prefers-reduced-motion` で自動回転を抑制できる

### スライダー

- [ ] 可能なら `<input type="range">` を使っている
- [ ] 可視ラベルまたは `aria-label` / `aria-labelledby` がある
- [ ] カスタム時は `role="slider"` と min / max / now がある
- [ ] 矢印・Home・End で値が変わる
- [ ] 値変更時に `aria-valuenow`（と必要なら `aria-valuetext`）が更新される

## まとめ

- カルーセルはスライド切替UI、スライダーは範囲から値を選ぶ入力。ロールを混同しない
- カルーセルは `aria-roledescription`、操作ボタン、自動回転の停止手段が中心
- 自動回転はフォーカス / ホバー / 明示停止に対応し、WCAG 2.2.2 を満たす
- スライダーはまず `<input type="range">` を使い、カスタム時だけ `role="slider"` とキーボードを実装する
