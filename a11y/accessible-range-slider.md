# アクセシブルなレンジスライダーの実装 — 現在値を読み上げる範囲入力

## レンジスライダーとは

レンジスライダー（Range Slider）は、最小値と最大値の間から値を選ぶフォームUI。音量、明るさ、拡大率、温度など、連続した範囲を感覚的に調整する場面に向いている。

- 現在値を視覚的に確認できる
- キーボードで一段階ずつ変更できる
- 最小値、最大値、現在値を支援技術へ伝える
- ポインターでつまみをドラッグできる
- 値の単位や意味を理解できる

通常はネイティブの`input type="range"`を使う。見た目を独自にするためだけに、`div`とJavaScriptで置き換えない。

> 参照:
> - [W3C WAI-ARIA APG — Slider Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/slider/)
> - [WHATWG HTML Standard — Range state](https://html.spec.whatwg.org/multipage/input.html#range-state-(type=range))

## レンジ入力が向く場面

### おおよその値を素早く選ぶ

`input type="range"`は、範囲内の値を位置で選ぶコントロール。HTML仕様では、正確な値そのものが重要ではない場面を想定している。

```text
音量
小さい ─────●───── 大きい
             60%
```

音量や明るさのように、前後の値を試しながら調整する操作と相性がよい。

### 正確な値を入力する

金額、人数、寸法など、利用者が正確な数値を直接入力する必要があるなら`input type="number"`を使う。

```html
<label for="participants">参加人数</label>
<input
  id="participants"
  name="participants"
  type="number"
  min="1"
  max="20"
  step="1"
  value="2"
>
```

スライダーと数値入力を併設する場合は、二つの値を常に同期し、どちらからでも変更できるようにする。

### 未選択を表現する

レンジ入力は空の値を持てない。`value`を省略しても、ブラウザは通常、最小値と最大値の中間に初期値を設定する。

「まだ回答していない」状態が必要なら、レンジ入力だけで未選択を表現しない。

- 「設定しない」チェックボックスを用意する
- 候補が少なければラジオボタンを使う
- 数値入力を使い、空欄を許可する

同意や評価を、知らないうちに中間値で回答済みにしない。

## 基本のHTML

```html
<div class="range-field">
  <label for="volume">音量</label>

  <div class="range-field__control">
    <input
      id="volume"
      name="volume"
      type="range"
      min="0"
      max="100"
      step="5"
      value="50"
      aria-describedby="volume-hint"
    >
    <output id="volume-output" for="volume">50%</output>
  </div>

  <p id="volume-hint">
    0%から100%まで、5%単位で調整できます。
  </p>
</div>
```

### 必要な属性

| 属性 | 役割 |
|---|---|
| `type="range"` | ネイティブの範囲入力にする |
| `min` | 選択できる最小値 |
| `max` | 選択できる最大値 |
| `step` | 一回の操作で変わる間隔 |
| `value` | 初期値 |
| `name` | フォーム送信時の名前 |
| `id`と`label` | 見えるラベルを関連付ける |
| `aria-describedby` | 範囲や操作の補足を関連付ける |

ネイティブ要素には`role="slider"`、`aria-valuemin`、`aria-valuemax`、`aria-valuenow`を重ねて付けない。ブラウザが`min`、`max`、`value`から必要なアクセシビリティ情報を公開する。

### 現在値を画面へ表示する

レンジ入力は、現在値をテキストとして表示しないブラウザがある。`output`を隣へ置き、`input`イベントで同期する。

```js
const volume = document.getElementById('volume');
const volumeOutput = document.getElementById('volume-output');

function updateVolumeOutput() {
  volumeOutput.value = `${volume.value}%`;
}

volume.addEventListener('input', updateVolumeOutput);
updateVolumeOutput();
```

`input`イベントはドラッグ中にも発生する。`change`イベントだけを使うと、操作を終えるまで表示が更新されないことがある。

スクリーンリーダーはフォーカス中のネイティブ入力から現在値を取得できるため、`output`を`aria-live`にして毎回重ねて通知する必要はない。過剰な読み上げが起きないか、実際の組み合わせで確認する。

## ラベル、単位、補足説明

### ラベルは値ではなく目的を表す

```html
<label for="brightness">画面の明るさ</label>
<input id="brightness" type="range" min="0" max="100" value="75">
```

「75%」は現在値であり、入力の名前ではない。「画面の明るさ」のように何を変更するかをラベルにする。

### 単位を視覚的にも示す

```html
<label for="temperature">設定温度</label>
<div>
  <input
    id="temperature"
    name="temperature"
    type="range"
    min="16"
    max="30"
    step="0.5"
    value="22"
    aria-describedby="temperature-unit"
  >
  <output for="temperature">22℃</output>
</div>
<span id="temperature-unit">単位は摂氏です。</span>
```

単位をCSSの擬似要素だけで追加しない。テキストとして表示し、必要に応じて補足説明へ関連付ける。

### 数字だけでは意味が伝わらない値

満足度やサイズのように、内部値が数値でも利用者には言葉で示したい場合がある。

```html
<label for="text-size">文字サイズ</label>
<input
  id="text-size"
  name="text-size"
  type="range"
  min="1"
  max="3"
  step="1"
  value="2"
  aria-valuetext="標準"
>
<output id="text-size-output" for="text-size">標準</output>
```

値を変更するたびに、見える表示と`aria-valuetext`を同じ対応表から更新する。

```js
const textSize = document.getElementById('text-size');
const textSizeOutput = document.getElementById('text-size-output');
const textSizeLabels = {
  1: '小さい',
  2: '標準',
  3: '大きい',
};

function updateTextSize() {
  const label = textSizeLabels[textSize.value];
  textSize.setAttribute('aria-valuetext', label);
  textSizeOutput.value = label;
}

textSize.addEventListener('input', updateTextSize);
updateTextSize();
```

`aria-valuetext`は、`aria-valuenow`の数字だけでは意味が不十分な場合に使う。支援技術によっては数値の代わりに値テキストを伝えるため、単に数字を繰り返す目的では付けない。

> 参照: [W3C WAI-ARIA APG — Communicating Value and Limits for Range Widgets](https://www.w3.org/WAI/ARIA/apg/practices/range-related-properties/)

## ネイティブのキーボード操作

| キー | 一般的な動作 |
|---|---|
| → / ↑ | 値を一段階増やす |
| ← / ↓ | 値を一段階減らす |
| Home | 最小値にする |
| End | 最大値にする |
| Page Up | 値を大きな間隔で増やす場合がある |
| Page Down | 値を大きな間隔で減らす場合がある |

ブラウザ、OS、書字方向、縦向き表示によって細部が異なる場合がある。ネイティブ入力へ独自の`keydown`処理を追加すると、値が二回変わったり、標準操作を壊したりするため避ける。

`step`を極端に細かくすると、矢印キーだけで目的値へ到達するのが難しくなる。想定する範囲と調整精度から、現実的な段階数にする。

## 目盛りと値の区切り

`datalist`を関連付けると、ブラウザが目盛りの候補を表示できる。

```html
<label for="zoom">拡大率</label>
<input
  id="zoom"
  name="zoom"
  type="range"
  min="50"
  max="200"
  step="10"
  value="100"
  list="zoom-marks"
>
<datalist id="zoom-marks">
  <option value="50" label="50%"></option>
  <option value="100" label="100%"></option>
  <option value="150" label="150%"></option>
  <option value="200" label="200%"></option>
</datalist>
```

目盛りやラベルの表示方法はブラウザに委ねられる。重要な最小値、最大値、単位を`datalist`の見た目だけに依存させず、入力の周囲にもテキストで示す。

```html
<div class="range-limits" aria-hidden="true">
  <span>50%</span>
  <span>200%</span>
</div>
```

`aria-hidden="true"`にするのは、同じ範囲をラベルや補足説明ですでに伝えており、重複読み上げを避けたい場合だけにする。

## CSSでネイティブ操作を保つ

```css
.range-field {
  max-inline-size: 32rem;
}

.range-field__control {
  display: grid;
  grid-template-columns: minmax(12rem, 1fr) 5rem;
  gap: 1rem;
  align-items: center;
}

.range-field input[type="range"] {
  inline-size: 100%;
  min-block-size: 2.75rem;
}

.range-field input[type="range"]:focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

.range-field output {
  min-inline-size: 4ch;
  font-variant-numeric: tabular-nums;
  text-align: end;
}

@media (forced-colors: active) {
  .range-field input[type="range"] {
    forced-color-adjust: auto;
  }
}
```

`appearance: none`でネイティブ表示を消すと、つまみ、トラック、フォーカス表示をすべて自分で再構築する必要がある。装飾する場合も、次を確認する。

- つまみとトラックを十分なコントラストで区別できる
- フォーカス表示が消えない
- つまみの操作領域が小さすぎない
- 強制カラーモードで現在位置を確認できる
- 200%・400%拡大でも値とラベルが重ならない
- 縦スクロール中に意図せず値が変わらない

## Reactでネイティブ入力を使う

```jsx
import { useId } from 'react';

function VolumeSlider({ value, onChange }) {
  const inputId = useId();
  const hintId = useId();

  return (
    <div className="range-field">
      <label htmlFor={inputId}>音量</label>
      <div className="range-field__control">
        <input
          id={inputId}
          name="volume"
          type="range"
          min="0"
          max="100"
          step="5"
          value={value}
          aria-describedby={hintId}
          onChange={(event) => onChange(Number(event.target.value))}
        />
        <output htmlFor={inputId}>{value}%</output>
      </div>
      <p id={hintId}>0%から100%まで、5%単位で調整できます。</p>
    </div>
  );
}
```

レンジ入力の`value`は文字列として取得される。アプリケーションで数値として扱うなら、境界で`Number()`へ変換する。

コンポーネントを複数配置できるよう、固定IDを使わない。ラベル、補足説明、入力の関連付けをインスタンスごとに一意にする。

## カスタムスライダーが必要な場合

非線形の目盛り、製品固有の複数つまみ、ネイティブでは表現できない視覚化が必要な場合に限り、カスタム実装を検討する。

### 必要なARIA

```html
<div class="custom-slider-field">
  <span id="custom-temperature-label">設定温度</span>

  <div
    id="custom-temperature"
    class="custom-slider"
    role="slider"
    tabindex="0"
    aria-labelledby="custom-temperature-label"
    aria-valuemin="16"
    aria-valuemax="30"
    aria-valuenow="22"
    aria-valuetext="22度"
    data-step="0.5"
    data-page-step="2"
  >
    <span class="custom-slider__track" aria-hidden="true">
      <span class="custom-slider__fill"></span>
      <span class="custom-slider__thumb"></span>
    </span>
  </div>

  <output id="custom-temperature-output">22℃</output>
  <input
    id="custom-temperature-value"
    type="hidden"
    name="temperature"
    value="22"
  >
</div>
```

| 属性 | 役割 |
|---|---|
| `role="slider"` | スライダーとして伝える |
| `tabindex="0"` | キーボードフォーカスを受け取る |
| `aria-labelledby` | 見えるラベルと関連付ける |
| `aria-valuemin` | 最小値を伝える |
| `aria-valuemax` | 最大値を伝える |
| `aria-valuenow` | 現在の数値を伝える |
| `aria-valuetext` | 単位や値の意味を言葉で伝える |

横向きは`aria-orientation="horizontal"`が既定値。縦向きの場合は`aria-orientation="vertical"`を明示する。

`role="slider"`は、値を操作するフォーカス可能な要素へ付ける。トラック全体へ別のTab停止位置を追加しない。

### JavaScript実装

```js
class AccessibleSlider {
  constructor(root, {
    output,
    input,
    formatValue = (value) => String(value),
    formatValueText = null,
  } = {}) {
    this.root = root;
    this.track = root.querySelector('.custom-slider__track');
    this.output = output;
    this.input = input;
    this.formatValue = formatValue;
    this.formatValueText = formatValueText;

    this.min = Number(root.getAttribute('aria-valuemin') ?? 0);
    this.max = Number(root.getAttribute('aria-valuemax') ?? 100);
    this.step = Number(root.dataset.step ?? 1);
    this.pageStep = Number(root.dataset.pageStep ?? this.step * 10);
    this.orientation = root.getAttribute('aria-orientation') ?? 'horizontal';
    this.pointerChanged = false;

    this.validateConfiguration();
    this.value = this.min;
    this.setValue(Number(root.getAttribute('aria-valuenow')), false);

    root.addEventListener('keydown', (event) => {
      this.handleKeyDown(event);
    });
    root.addEventListener('pointerdown', (event) => {
      this.handlePointerDown(event);
    });
    root.addEventListener('pointermove', (event) => {
      this.handlePointerMove(event);
    });
    root.addEventListener('pointerup', (event) => {
      this.handlePointerUp(event);
    });
    root.addEventListener('pointercancel', () => {
      this.pointerChanged = false;
    });
  }

  validateConfiguration() {
    const values = [this.min, this.max, this.step, this.pageStep];
    if (values.some((value) => !Number.isFinite(value))) {
      throw new TypeError('Slider values must be finite numbers.');
    }
    if (this.max <= this.min || this.step <= 0 || this.pageStep <= 0) {
      throw new RangeError('Slider range and steps are invalid.');
    }
    const stepCount = (this.max - this.min) / this.step;
    const tolerance = Number.EPSILON * Math.max(1, Math.abs(stepCount)) * 10;
    if (Math.abs(stepCount - Math.round(stepCount)) > tolerance) {
      throw new RangeError('Slider maximum must align with its step.');
    }
    if (!this.track) {
      throw new Error('Slider track was not found.');
    }
  }

  clamp(value) {
    return Math.min(Math.max(value, this.min), this.max);
  }

  alignToStep(value) {
    const steps = Math.round((value - this.min) / this.step);
    const aligned = this.min + steps * this.step;
    const precision = Math.max(
      this.decimalPlaces(this.min),
      this.decimalPlaces(this.max),
      this.decimalPlaces(this.step)
    );

    return Number(this.clamp(aligned).toFixed(precision));
  }

  decimalPlaces(value) {
    const [, decimals = ''] = String(value).split('.');
    return decimals.length;
  }

  setValue(value, emit = true) {
    if (!Number.isFinite(value)) return false;

    const nextValue = this.alignToStep(value);
    const changed = nextValue !== this.value;
    this.value = nextValue;

    const percent = (
      (nextValue - this.min) / (this.max - this.min)
    ) * 100;

    this.root.setAttribute('aria-valuenow', String(nextValue));
    if (this.formatValueText) {
      this.root.setAttribute(
        'aria-valuetext',
        this.formatValueText(nextValue)
      );
    } else {
      this.root.removeAttribute('aria-valuetext');
    }
    this.root.style.setProperty('--slider-percent', `${percent}%`);

    if (this.output) this.output.value = this.formatValue(nextValue);
    if (this.input) this.input.value = String(nextValue);

    if (emit && changed) {
      this.root.dispatchEvent(new Event('input', { bubbles: true }));
    }

    return changed;
  }

  emitChange() {
    this.root.dispatchEvent(new Event('change', { bubbles: true }));
  }

  handleKeyDown(event) {
    const changes = {
      ArrowRight: this.step,
      ArrowUp: this.step,
      ArrowLeft: -this.step,
      ArrowDown: -this.step,
      PageUp: this.pageStep,
      PageDown: -this.pageStep,
    };

    let nextValue;

    if (event.key === 'Home') nextValue = this.min;
    if (event.key === 'End') nextValue = this.max;
    if (Object.hasOwn(changes, event.key)) {
      nextValue = this.value + changes[event.key];
    }
    if (nextValue === undefined) return;

    event.preventDefault();
    if (this.setValue(nextValue)) this.emitChange();
  }

  valueFromPointer(event) {
    const rect = this.track.getBoundingClientRect();
    let ratio;

    if (this.orientation === 'vertical') {
      ratio = (rect.bottom - event.clientY) / rect.height;
    } else {
      ratio = (event.clientX - rect.left) / rect.width;
    }

    return this.min + this.clampRatio(ratio) * (this.max - this.min);
  }

  clampRatio(ratio) {
    return Math.min(Math.max(ratio, 0), 1);
  }

  handlePointerDown(event) {
    if (event.button !== 0) return;

    event.preventDefault();
    this.root.focus();
    this.root.setPointerCapture(event.pointerId);
    this.pointerChanged = this.setValue(this.valueFromPointer(event));
  }

  handlePointerMove(event) {
    if (!this.root.hasPointerCapture(event.pointerId)) return;

    const changed = this.setValue(this.valueFromPointer(event));
    this.pointerChanged = changed || this.pointerChanged;
  }

  handlePointerUp(event) {
    if (!this.root.hasPointerCapture(event.pointerId)) return;

    this.root.releasePointerCapture(event.pointerId);
    if (this.pointerChanged) this.emitChange();
    this.pointerChanged = false;
  }
}

const temperatureSlider = new AccessibleSlider(
  document.getElementById('custom-temperature'),
  {
    output: document.getElementById('custom-temperature-output'),
    input: document.getElementById('custom-temperature-value'),
    formatValue: (value) => `${value}℃`,
    formatValueText: (value) => `${value}度`,
  }
);
```

この例は、APGの基本操作に合わせて右・上矢印で増加、左・下矢印で減少する。縦向きや右から左へ書く言語で方向を変える場合は、見た目、ポインター計算、キーボード操作を同じ方針で設計する。

### カスタムスライダーのCSS

```css
.custom-slider-field {
  display: grid;
  gap: 0.5rem;
  max-inline-size: 30rem;
}

.custom-slider {
  --slider-percent: 0%;
  display: flex;
  align-items: center;
  min-block-size: 2.75rem;
  cursor: pointer;
  direction: ltr;
  touch-action: pan-y;
}

.custom-slider__track {
  position: relative;
  display: block;
  inline-size: calc(100% - 2rem);
  block-size: 0.5rem;
  margin-inline: 1rem;
  border: 0.125rem solid CanvasText;
  border-radius: 999rem;
  background: Canvas;
}

.custom-slider__fill {
  position: absolute;
  inset-block: 0;
  inset-inline-start: 0;
  inline-size: var(--slider-percent);
  background: Highlight;
}

.custom-slider__thumb {
  position: absolute;
  inset-block-start: 50%;
  inset-inline-start: var(--slider-percent);
  inline-size: 2rem;
  block-size: 2rem;
  border: 0.125rem solid CanvasText;
  border-radius: 50%;
  background: Canvas;
  transform: translate(-50%, -50%);
}

.custom-slider:focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

.custom-slider:focus-visible .custom-slider__thumb {
  box-shadow: 0 0 0 0.1875rem Canvas;
}

.custom-slider[aria-orientation="vertical"] {
  justify-content: center;
  inline-size: 2.75rem;
  block-size: 14rem;
  touch-action: pan-x;
}

.custom-slider[aria-orientation="vertical"] .custom-slider__track {
  inline-size: 0.5rem;
  block-size: calc(100% - 2rem);
  margin-block: 1rem;
  margin-inline: 0;
}

.custom-slider[aria-orientation="vertical"] .custom-slider__fill {
  inset-block: auto 0;
  inset-inline: 0;
  inline-size: auto;
  block-size: var(--slider-percent);
}

.custom-slider[aria-orientation="vertical"] .custom-slider__thumb {
  inset-block-start: calc(100% - var(--slider-percent));
  inset-inline-start: 50%;
}

@media (forced-colors: active) {
  .custom-slider__fill {
    background: Highlight;
  }

  .custom-slider__thumb {
    background: Canvas;
  }
}
```

横向きスライダーで`touch-action: pan-y`を指定すると、縦方向のページスクロールを許可しつつ、横方向の操作をスライダーで扱える。縦向きの例では`pan-x`へ切り替えているが、実際の配置でも操作とページスクロールが衝突しないか検証する。

この例は、文書の書字方向にかかわらず左を最小値、右を最大値にするため、操作部分へ`direction: ltr`を指定している。右から左へ値が増える設計に変える場合は、CSSだけを反転せず、ポインター計算とキーボードの方針も合わせて変更する。

### タッチ支援技術の注意点

カスタムスライダーは、タッチ操作のスクリーンリーダーで値の増減ジェスチャーが期待どおり動かない組み合わせがある。APGも、タッチが主な入力方法の端末と支援技術で十分にテストするよう警告している。

- iOSとAndroidの対象スクリーンリーダーで確認する
- スワイプ操作でスライダーへ移動できるか確認する
- 値を増減する支援技術の操作を確認する
- ドラッグだけを唯一の変更方法にしない
- 可能ならネイティブ入力または数値入力を代替として提供する

## 二つの値を選ぶ場合

価格の下限と上限のような複数つまみは、各つまみが独立したスライダーになる。単一スライダーよりフォーカス、重なり、値の制約、タッチ操作が複雑になる。

まず、二つのネイティブ入力を別々に表示できないか検討する。

```html
<fieldset>
  <legend>価格帯</legend>

  <label for="minimum-price">最低価格</label>
  <input
    id="minimum-price"
    name="minimum-price"
    type="range"
    min="0"
    max="80000"
    step="1000"
    value="20000"
  >
  <output id="minimum-price-output" for="minimum-price">20,000円</output>

  <label for="maximum-price">最高価格</label>
  <input
    id="maximum-price"
    name="maximum-price"
    type="range"
    min="20000"
    max="100000"
    step="1000"
    value="80000"
  >
  <output id="maximum-price-output" for="maximum-price">80,000円</output>
</fieldset>
```

下限を変更したら上限入力の`min`を、上限を変更したら下限入力の`max`を更新する。現在値が新しい範囲外にならないよう、値、制約、見える出力を同時に同期する。

```js
const minimumPrice = document.getElementById('minimum-price');
const maximumPrice = document.getElementById('maximum-price');
const minimumPriceOutput = document.getElementById('minimum-price-output');
const maximumPriceOutput = document.getElementById('maximum-price-output');
const priceFormatter = new Intl.NumberFormat('ja-JP');

function formatPrice(value) {
  return `${priceFormatter.format(Number(value))}円`;
}

function updatePriceRange() {
  minimumPrice.max = maximumPrice.value;
  maximumPrice.min = minimumPrice.value;
  minimumPriceOutput.value = formatPrice(minimumPrice.value);
  maximumPriceOutput.value = formatPrice(maximumPrice.value);
}

minimumPrice.addEventListener('input', updatePriceRange);
maximumPrice.addEventListener('input', updatePriceRange);
updatePriceRange();
```

カスタムの複数つまみを作る場合は、次を満たす。

- 各つまみをTab順序へ入れる
- 各つまみに「最低価格」「最高価格」の固有名を付ける
- それぞれの`aria-valuemin`、`aria-valuemax`、`aria-valuenow`を更新する
- 値や見た目の位置が入れ替わってもTab順序を変えない
- つまみが重なっても、キーボードとタッチで両方を操作できる

> 参照: [W3C WAI-ARIA APG — Slider (Multi-Thumb) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/slider-multithumb/)

## 値変更による副作用

ドラッグや矢印キーのたびに、重い通信、ページ遷移、フォーカス移動を実行しない。

### 即時反映が自然なもの

- 音量のプレビュー
- 画面の明るさ
- 画像の拡大率

処理が重い場合は、描画頻度を調整しても、見える値とアクセシビリティ状態は操作へ追従させる。

### 確定操作を分けるもの

- 検索条件を変更してサーバーへ問い合わせる
- 商品を購入する
- 取り消しにくい設定を保存する

スライダーで値を選んだ後に「適用」「保存」ボタンを押す設計にする。値を一段階変えただけで予期しないコンテキスト変更を起こさない。

## よくある失敗

### 現在値が見えない

つまみの位置だけでは正確な値を確認しにくい。入力の近くへテキストで現在値を表示する。

### ラベルがない

最小値と最大値があっても、何を調整する入力かは伝わらない。見える`label`を関連付ける。

### 単位を色やアイコンだけで示す

「℃」「%」「円」などをテキストで示す。必要なら補足説明や`aria-valuetext`にも反映する。

### `aria-valuetext`を更新しない

見える表示だけを更新すると、支援技術には以前の値が残る。現在値と値テキストを同じ状態から更新する。

### フォーカス表示を消す

CSSでアウトラインを消したままにしない。つまみやスライダー全体の現在位置を確認できる表示を用意する。

### ドラッグでしか変更できない

矢印キー、Home、Endで操作できるようにする。タッチ支援技術でも検証する。

### 小さなつまみだけを操作領域にする

見た目のつまみだけでなく、周囲を含む十分なポインター操作領域を用意する。WCAG 2.2のターゲットサイズも確認する。

> 参照: [Understanding WCAG 2.2 SC 2.5.8 — Target Size (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum)

### ネイティブ入力へ不要なARIAを追加する

`input type="range"`へ`role="slider"`や重複した値属性を付けない。HTML属性とARIA属性が食い違う状態を作らない。

### スライダーで空欄を表現する

レンジ入力には常に数値がある。未回答を扱う必要があるなら、別のコントロールまたは状態を設計する。

## テストのチェックリスト

### パターンの選択

- [ ] 値を位置で素早く調整する操作にスライダーが適している
- [ ] 正確な直接入力が重要なら数値入力を検討した
- [ ] 未選択状態が必要か確認した
- [ ] カスタム実装が必要な理由がある
- [ ] 複数つまみを二つの入力へ分けられないか検討した

### 名前、値、説明

- [ ] 見えるラベルが入力へ関連付いている
- [ ] 最小値、最大値、刻み幅が目的に合っている
- [ ] 現在値をテキストでも確認できる
- [ ] 単位を色や位置だけに依存せず伝えている
- [ ] 数字だけでは意味が不明な場合に値テキストを提供している
- [ ] 見える値、フォーム値、アクセシビリティ上の値が一致する

### キーボード

- [ ] Tab / Shift + Tabでスライダーへ出入りできる
- [ ] 矢印キーで一段階ずつ増減できる
- [ ] Homeで最小値、Endで最大値へ移動できる
- [ ] 値が最小値未満、最大値超過にならない
- [ ] 細かすぎる刻み幅で操作回数が過剰にならない
- [ ] ネイティブ入力の標準キー操作をJavaScriptで上書きしていない

### ポインターとタッチ

- [ ] トラックをクリックまたはタップして値を変更できる
- [ ] ドラッグ中に現在値を確認できる
- [ ] 操作領域が小さすぎない
- [ ] ページスクロールとドラッグ操作が衝突しない
- [ ] タッチ端末のスクリーンリーダーで値を変更できる

### 表示

- [ ] つまみとトラックを区別できる
- [ ] フォーカス表示が見える
- [ ] 状態を色だけに依存していない
- [ ] 強制カラーモードで現在位置を確認できる
- [ ] 200%・400%拡大でラベル、値、入力が重ならない
- [ ] 狭い画面でも横スクロールなしで操作できる

### 動的処理

- [ ] 値変更で予期しないページ遷移やフォーカス移動が起きない
- [ ] 重い処理は必要に応じて「適用」操作と分けている
- [ ] 制約を動的に変更した後も現在値が範囲内にある
- [ ] フォーム送信値と表示値が一致する
- [ ] リセット後に出力とアクセシビリティ状態も同期する

## 関連して読むとよいテーマ

- [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [アクセシブルなラジオグループの実装 — 矢印キーで一つを選ぶ](./accessible-radio-group.md)
- [アクセシブルなスイッチの実装 — オン／オフとチェックボックスの使い分け](./accessible-switch.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)

## まとめ

- 範囲内のおおよその値を調整するなら、まずネイティブの`input type="range"`を使う
- 正確な値の直接入力には`input type="number"`も検討する
- ラベル、最小値、最大値、刻み幅、現在値、単位を理解できるようにする
- 現在値はつまみの位置だけでなく、テキストでも表示する
- 数字だけでは意味が不足する場合に限り、`aria-valuetext`を使う
- ネイティブのキーボード操作とフォーカス表示を壊さない
- カスタム実装では、ARIA属性、キーボード、ポインター、フォーム値を同じ状態から更新する
- 複数つまみは各つまみに固有の名前と値を持たせ、Tab順序を固定する
- ドラッグだけに依存せず、タッチ支援技術を含む実機で検証する
