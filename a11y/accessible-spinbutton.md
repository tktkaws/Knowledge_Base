# アクセシブルなスピンボタンの実装 — 数値の増減

## スピンボタンとは

スピンボタン（Spinbutton）は、決められた範囲や刻み幅で値を増減できる入力UI。数量、人数、時間、温度などに使われる。

一般的なスピンボタンには、次の操作がある。

- テキストフィールドへ値を直接入力する
- 上下矢印キーで一段階ずつ増減する
- 増加・減少ボタンを押して値を変更する
- 最小値、最大値、現在値を確認する
- 不正な値をエラーとして修正する

通常はネイティブの`input type="number"`を使う。ブラウザが数値入力、キーボード操作、制約検証、フォーム送信を提供するため、見た目を変える目的だけでカスタム実装へ置き換えない。

> 参照:
> - [W3C WAI-ARIA APG — Spinbutton Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/spinbutton/)
> - [WHATWG HTML Standard — Number state](https://html.spec.whatwg.org/multipage/input.html#number-state-(type=number))

## スピンボタンが向く値

### 一段階増減する意味がある数値

```text
数量
[-] [ 3 ] [+]
```

- 商品の数量
- 予約人数
- 会議の時間
- 温度設定
- 表示する行数

値を1増減しても同じ種類の値として意味が通る場合に向いている。

### 数字で構成される識別子には使わない

郵便番号、電話番号、会員番号、クレジットカード番号は数値ではなく識別子。前後の番号へ増減する意味がなく、先頭の`0`も必要になる場合がある。

```html
<label for="postal-code">郵便番号</label>
<input
  id="postal-code"
  name="postal-code"
  type="text"
  inputmode="numeric"
  autocomplete="postal-code"
>
```

`type="number"`を使うか迷ったら、上下の増減操作が役に立つ値かを確認する。役に立たなければ`type="text"`、`inputmode`、必要に応じた`pattern`を使う。

## 基本のHTML

```html
<div class="number-field">
  <label for="quantity">数量</label>
  <input
    id="quantity"
    name="quantity"
    type="number"
    min="1"
    max="10"
    step="1"
    value="1"
    required
    aria-describedby="quantity-hint"
  >
  <p id="quantity-hint">
    1個から10個まで選べます。
  </p>
</div>
```

### 主な属性

| 属性 | 役割 |
|---|---|
| `type="number"` | ネイティブの数値入力にする |
| `min` | 許可する最小値 |
| `max` | 許可する最大値 |
| `step` | 増減する間隔 |
| `value` | 初期値 |
| `required` | 空欄を許可しない |
| `name` | フォーム送信時の名前 |
| `id`と`label` | 見えるラベルを入力へ関連付ける |
| `aria-describedby` | 範囲や単位の補足を関連付ける |

ネイティブの数値入力には、`role="spinbutton"`や`aria-valuenow`を追加しない。ブラウザがHTML属性と現在値から必要なアクセシビリティ情報を公開する。

### 初期値と空欄

数量に安全な既定値があるなら`value="1"`のように指定できる。利用者の明示的な入力が必要なら、初期値を省略する。

```html
<label for="attendees">参加人数</label>
<input
  id="attendees"
  name="attendees"
  type="number"
  min="1"
  max="20"
  required
>
```

空欄と`0`は異なる状態。未入力を勝手に`0`へ変換しない。

## ラベル、範囲、単位を伝える

### ラベルは入力の目的を示す

```html
<label for="room-temperature">設定温度</label>
<input
  id="room-temperature"
  name="room-temperature"
  type="number"
  min="16"
  max="30"
  step="0.5"
  value="22"
  aria-describedby="temperature-hint"
>
<span aria-hidden="true">℃</span>
<p id="temperature-hint">
  単位は摂氏です。16℃から30℃まで、0.5℃単位で設定できます。
</p>
```

CSSの擬似要素だけで単位を追加しない。見えるテキストとして示し、入力の補足説明にも含める。

### 制約を事前に説明する

`min`、`max`、`step`はブラウザの検証に使われるが、すべての利用者が操作前に制約を把握できるとは限らない。「1から10」「0.5単位」のような重要な条件を画面上にも示す。

> 参照: [Understanding WCAG 2.2 SC 3.3.2 — Labels or Instructions](https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html)

## `min`、`max`、`step`の設計

### 整数を一つずつ増減する

```html
<input type="number" min="0" max="100" step="1" value="10">
```

### 小数を増減する

```html
<input type="number" min="0" max="10" step="0.1" value="1.5">
```

小数を許可する場合は、`step`を明示する。`step`の既定値は`1`であるため、`1.5`を入力できる見た目でも制約検証では不正になることがある。

### 任意の小数を許可する

```html
<input type="number" min="0" max="10" step="any">
```

`step="any"`は刻み幅による不一致をなくすが、一定間隔で増減するスピンボタンの利点は小さくなる。必要な精度が決まっているなら、具体的な刻み幅を使う。

### 範囲がない値

`min`と`max`がない数値入力も作れる。ただし、業務上の限界があるならHTMLとサーバー側の両方で制約する。カスタムの`role="spinbutton"`には最小値・最大値の既定値がないため、既知の範囲があれば`aria-valuemin`と`aria-valuemax`を明示する。

## ネイティブのキーボード操作

| キー | 一般的な動作 |
|---|---|
| ↑ | `step`に従って値を増やす |
| ↓ | `step`に従って値を減らす |
| Home | 実装によって最小値へ移動する |
| End | 実装によって最大値へ移動する |
| 数字・小数点・符号 | 値を直接編集する |
| Tab / Shift + Tab | 次・前のフォーカス可能要素へ移動する |

APGのカスタムスピンボタンでは、Homeは最小値、Endは最大値へ変更する操作として定義される。ネイティブ入力の細かな挙動はブラウザとOSに委ね、独自の`keydown`処理を重ねない。

入力欄では、文字選択、コピー、貼り付け、取り消し、カーソル移動など、プラットフォーム標準のテキスト編集操作を使えるようにする。

## 直接入力を妨げない

### `keydown`で文字を禁止しない

次の実装は避ける。

```js
// 避ける：貼り付け、IME、ショートカット、負数、小数などを壊しやすい
input.addEventListener('keydown', (event) => {
  if (!/[0-9]/.test(event.key)) event.preventDefault();
});
```

利用者は、値を貼り付けたり、一度すべて削除してから入力し直したりする。負数を入力するときは、途中で`-`だけの状態にもなる。入力途中の文字列を毎回完成した数値へ変換しない。

### 入力中と確定後を分ける

- `input`イベントでは表示や関連情報を同期する
- `blur`またはフォーム送信時に範囲と刻み幅を検証する
- 不正な値を見つけても、理由を示さず勝手に別の値へ丸めない
- サーバー側でも同じ制約を検証する

ブラウザの制約検証は入力を完全には防がない。利用者が`max`より大きな値を手入力できても、フォーム送信時にエラーとして扱える設計にする。

## エラーを伝える

```html
<form id="ticket-form" class="number-field" novalidate>
  <label for="ticket-count">チケット枚数</label>
  <input
    id="ticket-count"
    name="ticket-count"
    type="number"
    min="1"
    max="6"
    step="1"
    required
    aria-describedby="ticket-hint ticket-error"
  >
  <p id="ticket-hint">一度に申し込めるのは1枚から6枚です。</p>
  <p id="ticket-error" class="error" hidden></p>

  <button type="submit">申し込む</button>
</form>
```

```js
const ticketForm = document.getElementById('ticket-form');
const ticketCount = document.getElementById('ticket-count');
const ticketError = document.getElementById('ticket-error');

function getTicketError() {
  const { validity } = ticketCount;

  if (validity.valueMissing) return 'チケット枚数を入力してください。';
  if (validity.badInput) return 'チケット枚数を数字で入力してください。';
  if (validity.rangeUnderflow) return '1枚以上を入力してください。';
  if (validity.rangeOverflow) return '6枚以下を入力してください。';
  if (validity.stepMismatch) return 'チケット枚数は整数で入力してください。';
  return '';
}

function updateTicketError() {
  const message = getTicketError();
  ticketCount.setAttribute('aria-invalid', String(Boolean(message)));
  ticketError.textContent = message;
  ticketError.hidden = !message;
  return !message;
}

ticketForm.addEventListener('submit', (event) => {
  if (updateTicketError()) return;

  event.preventDefault();
  ticketCount.focus();
});

ticketCount.addEventListener('input', () => {
  if (ticketCount.getAttribute('aria-invalid') === 'true') {
    updateTicketError();
  }
});
```

この例は`novalidate`でブラウザ標準のメッセージを無効にし、独自表示を実装している。独自エラーが不要なら`novalidate`を付けず、ネイティブの制約検証を利用する。

- エラーを色だけで示さない
- 入力とエラーメッセージを関連付ける
- 何が不正で、どう直すかを具体的に伝える
- エラー発生時は問題の入力へフォーカスを移す
- 修正後は`aria-invalid`とエラー表示を解除する

## 見える増加・減少ボタンを追加する

ブラウザ標準の増減ボタンは表示や操作領域が環境によって異なる。製品要件として大きなボタンが必要なら、ネイティブ数値入力の横へ通常の`button`を追加できる。

```html
<div class="quantity-field">
  <label for="order-quantity">数量</label>

  <div class="quantity-field__controls">
    <button
      id="decrease-quantity"
      type="button"
      aria-label="数量を1減らす"
      aria-controls="order-quantity"
    >
      −
    </button>

    <input
      id="order-quantity"
      name="quantity"
      type="number"
      min="1"
      max="10"
      step="1"
      value="1"
    >

    <button
      id="increase-quantity"
      type="button"
      aria-label="数量を1増やす"
      aria-controls="order-quantity"
    >
      ＋
    </button>
  </div>

  <output id="quantity-output" for="order-quantity">1個</output>
  <p id="quantity-status" role="status" class="visually-hidden"></p>
</div>
```

ボタンは`type="button"`にし、フォームを誤送信しないようにする。記号だけに頼らず、「数量を1増やす」「数量を1減らす」という名前を付ける。

```js
const quantity = document.getElementById('order-quantity');
const decreaseQuantity = document.getElementById('decrease-quantity');
const increaseQuantity = document.getElementById('increase-quantity');
const quantityOutput = document.getElementById('quantity-output');
const quantityStatus = document.getElementById('quantity-status');

function numericAttribute(name) {
  const value = quantity.getAttribute(name);
  return value === null ? null : Number(value);
}

function updateQuantity({ announce = false } = {}) {
  const value = quantity.valueAsNumber;
  const hasValue = Number.isFinite(value);
  const min = numericAttribute('min');
  const max = numericAttribute('max');

  decreaseQuantity.disabled = hasValue && min !== null && value <= min;
  increaseQuantity.disabled = hasValue && max !== null && value >= max;
  quantityOutput.value = hasValue ? `${value}個` : '未入力';

  if (announce) {
    quantityStatus.textContent = hasValue
      ? `数量を${value}個に変更しました。`
      : '数量は未入力です。';
  }
}

function changeQuantity(direction) {
  if (!Number.isFinite(quantity.valueAsNumber)) {
    const min = numericAttribute('min');
    quantity.value = String(min ?? 0);
  } else if (direction > 0) {
    quantity.stepUp();
  } else {
    quantity.stepDown();
  }

  quantity.dispatchEvent(new Event('input', { bubbles: true }));
  quantity.dispatchEvent(new Event('change', { bubbles: true }));
  updateQuantity({ announce: true });
}

decreaseQuantity.addEventListener('click', () => changeQuantity(-1));
increaseQuantity.addEventListener('click', () => changeQuantity(1));
quantity.addEventListener('input', () => updateQuantity());
updateQuantity();
```

外付けボタンはTab順序へ追加される。キーボード利用者は数値入力の上下矢印だけでも操作できるため、Tab停止位置が三つになる負担と、ボタンの大きな操作領域の利点を比較する。

ボタン操作中はフォーカスがボタンに残るため、この例は変更結果を`role="status"`で通知する。入力欄を直接操作しているときは現在値が伝わるので、ライブリージョンを更新しない。

## CSSで操作領域と状態を示す

```css
.quantity-field__controls {
  display: grid;
  grid-template-columns: 2.75rem minmax(5rem, 8rem) 2.75rem;
  gap: 0.5rem;
  align-items: stretch;
}

.quantity-field__controls button,
.quantity-field__controls input {
  min-block-size: 2.75rem;
  font: inherit;
}

.quantity-field__controls input {
  inline-size: 100%;
  text-align: center;
}

.quantity-field__controls :focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.125rem;
}

.quantity-field__controls button:disabled {
  opacity: 0.55;
}

.number-field input[aria-invalid="true"],
.quantity-field input[aria-invalid="true"] {
  border-color: LinkText;
  border-width: 0.1875rem;
}

.error {
  font-weight: 700;
  color: CanvasText;
  border-inline-start: 0.25rem solid LinkText;
  padding-inline-start: 0.75rem;
}
```

ブラウザ標準の増減ボタンだけをCSSで非表示にする場合、利用者が値を変える代替操作を確認する。マウス利用者へテキスト入力だけを強制しない。

## Reactでネイティブ入力を使う

```jsx
import { useId } from 'react';

function QuantityInput({ value, onChange }) {
  const inputId = useId();
  const hintId = useId();

  return (
    <div className="number-field">
      <label htmlFor={inputId}>数量</label>
      <input
        id={inputId}
        name="quantity"
        type="number"
        min="1"
        max="10"
        step="1"
        value={value}
        aria-describedby={hintId}
        onChange={(event) => onChange(event.target.value)}
      />
      <p id={hintId}>1個から10個まで選べます。</p>
    </div>
  );
}
```

編集中の値は文字列として保持すると、利用者が一度空欄にしたり、負数の符号を入力したりする途中状態を扱いやすい。`onChange`のたびに`Number()`へ変換すると、空欄が`0`になり、入力を妨げることがある。

送信時や入力確定時に検証し、有効な値だけを数値へ変換する。コンポーネントを複数配置できるよう、固定IDも避ける。

## カスタムスピンボタンが必要な場合

曜日、時間帯、製品固有の単位など、ネイティブ数値入力では表現できない離散値を扱う場合に限り、カスタム実装を検討する。

### 必要なHTMLとARIA

```html
<div class="custom-spinbutton-field">
  <label id="guest-count-label" for="guest-count">宿泊人数</label>

  <div class="custom-spinbutton-field__controls">
    <button
      type="button"
      data-spin-action="decrease"
      aria-label="宿泊人数を1人減らす"
      aria-controls="guest-count"
    >
      −
    </button>

    <input
      id="guest-count"
      name="guest-count"
      type="text"
      inputmode="numeric"
      role="spinbutton"
      aria-labelledby="guest-count-label"
      aria-describedby="guest-count-hint"
      aria-errormessage="guest-count-error"
      aria-valuemin="1"
      aria-valuemax="12"
      aria-valuenow="2"
      aria-valuetext="2人"
      value="2"
      data-step="1"
      data-page-step="5"
    >

    <button
      type="button"
      data-spin-action="increase"
      aria-label="宿泊人数を1人増やす"
      aria-controls="guest-count"
    >
      ＋
    </button>
  </div>

  <p id="guest-count-hint">1人から12人まで選べます。</p>
  <p id="guest-count-error" class="error" hidden></p>
  <p data-spin-status role="status" class="visually-hidden"></p>
</div>
```

| 属性 | 役割 |
|---|---|
| `role="spinbutton"` | スピンボタンとして伝える |
| `aria-valuemin` | 既知の最小値を伝える |
| `aria-valuemax` | 既知の最大値を伝える |
| `aria-valuenow` | 現在の有効な数値を伝える |
| `aria-valuetext` | 単位を含む分かりやすい現在値を伝える |
| `aria-invalid` | 入力値が不正かを伝える |
| `aria-errormessage` | エラーメッセージを関連付ける |

カスタムの`role="spinbutton"`には、最小値と最大値の既定値がない。範囲が既知なら両方を指定する。

### JavaScript実装

```js
class AccessibleSpinbutton {
  constructor(input, {
    formatValueText = null,
    errorMessage = '許可された範囲の数値を入力してください。',
  } = {}) {
    this.input = input;
    this.field = input.closest('.custom-spinbutton-field');
    this.decreaseButton = this.field?.querySelector(
      '[data-spin-action="decrease"]'
    );
    this.increaseButton = this.field?.querySelector(
      '[data-spin-action="increase"]'
    );
    this.error = document.getElementById(
      input.getAttribute('aria-errormessage')
    );
    this.status = this.field?.querySelector('[data-spin-status]');
    this.formatValueText = formatValueText;
    this.errorMessage = errorMessage;

    this.min = this.numberAttribute('aria-valuemin');
    this.max = this.numberAttribute('aria-valuemax');
    this.step = Number(input.dataset.step ?? 1);
    this.pageStep = Number(input.dataset.pageStep ?? this.step * 10);

    this.validateConfiguration();
    this.bindEvents();
    this.syncFromText(false);
  }

  numberAttribute(name) {
    const value = this.input.getAttribute(name);
    return value === null ? null : Number(value);
  }

  validateConfiguration() {
    if (
      !this.field ||
      !this.decreaseButton ||
      !this.increaseButton ||
      !this.error ||
      !this.status
    ) {
      throw new Error('Spinbutton controls were not found.');
    }
    if (!Number.isFinite(this.step) || this.step <= 0) {
      throw new RangeError('Spinbutton step must be a positive number.');
    }
    if (!Number.isFinite(this.pageStep) || this.pageStep <= 0) {
      throw new RangeError('Spinbutton page step must be positive.');
    }
    if (this.min !== null && this.max !== null && this.max < this.min) {
      throw new RangeError('Spinbutton range is invalid.');
    }
    if (this.min !== null && this.max !== null) {
      const stepCount = (this.max - this.min) / this.step;
      const tolerance = Number.EPSILON * Math.max(1, Math.abs(stepCount)) * 10;
      if (Math.abs(stepCount - Math.round(stepCount)) > tolerance) {
        throw new RangeError('Spinbutton maximum must align with its step.');
      }
    }
  }

  parseValue() {
    const text = this.input.value.trim();
    if (text === '') return null;

    const value = Number(text);
    return Number.isFinite(value) ? value : null;
  }

  isStepAligned(value) {
    const base = this.min ?? 0;
    const steps = (value - base) / this.step;
    const tolerance = Number.EPSILON * Math.max(1, Math.abs(steps)) * 10;
    return Math.abs(steps - Math.round(steps)) <= tolerance;
  }

  isValid(value) {
    if (value === null) return false;
    if (this.min !== null && value < this.min) return false;
    if (this.max !== null && value > this.max) return false;
    return this.isStepAligned(value);
  }

  clamp(value) {
    let result = value;
    if (this.min !== null) result = Math.max(result, this.min);
    if (this.max !== null) result = Math.min(result, this.max);
    return result;
  }

  alignToStep(value) {
    const base = this.min ?? 0;
    const steps = Math.round((value - base) / this.step);
    const precision = Math.max(
      this.decimalPlaces(base),
      this.decimalPlaces(this.max ?? 0),
      this.decimalPlaces(this.step)
    );
    return Number((base + steps * this.step).toFixed(precision));
  }

  decimalPlaces(value) {
    const [, decimals = ''] = String(value).split('.');
    return decimals.length;
  }

  syncAria(value) {
    if (this.isValid(value)) {
      this.input.setAttribute('aria-valuenow', String(value));
      if (this.formatValueText) {
        this.input.setAttribute(
          'aria-valuetext',
          this.formatValueText(value)
        );
      } else {
        this.input.removeAttribute('aria-valuetext');
      }
    } else {
      this.input.removeAttribute('aria-valuenow');
      this.input.removeAttribute('aria-valuetext');
    }

    this.decreaseButton.disabled =
      value !== null && this.min !== null && value <= this.min;
    this.increaseButton.disabled =
      value !== null && this.max !== null && value >= this.max;
  }

  syncFromText(showError) {
    const value = this.parseValue();
    const valid = this.isValid(value);
    this.syncAria(value);

    this.input.setAttribute(
      'aria-invalid',
      String(showError && !valid)
    );
    this.error.textContent = showError && !valid ? this.errorMessage : '';
    this.error.hidden = !(showError && !valid);

    return valid;
  }

  setValue(value, announce = false) {
    const previousValue = this.parseValue();
    const nextValue = this.clamp(this.alignToStep(value));
    const changed =
      previousValue !== nextValue ||
      this.input.value.trim() !== String(nextValue);

    this.input.value = String(nextValue);
    this.syncFromText(false);

    if (changed) {
      this.input.dispatchEvent(new Event('input', { bubbles: true }));
      this.input.dispatchEvent(new Event('change', { bubbles: true }));
    }

    if (announce && changed) {
      const text = this.formatValueText
        ? this.formatValueText(nextValue)
        : String(nextValue);
      this.status.textContent = `${text}に変更しました。`;
    }

    return changed;
  }

  changeBy(amount, announce = false) {
    const current = this.parseValue();
    const start = current ?? this.min ?? 0;
    const nextValue = current === null ? start : start + amount;
    this.setValue(nextValue, announce);
  }

  handleKeyDown(event) {
    const changes = {
      ArrowUp: this.step,
      ArrowDown: -this.step,
      PageUp: this.pageStep,
      PageDown: -this.pageStep,
    };

    if (Object.hasOwn(changes, event.key)) {
      event.preventDefault();
      this.changeBy(changes[event.key]);
      return;
    }

    if (event.key === 'Home' && this.min !== null) {
      event.preventDefault();
      this.setValue(this.min);
    }

    if (event.key === 'End' && this.max !== null) {
      event.preventDefault();
      this.setValue(this.max);
    }
  }

  bindEvents() {
    this.input.addEventListener('keydown', (event) => {
      this.handleKeyDown(event);
    });
    this.input.addEventListener('input', () => {
      this.syncFromText(false);
    });
    this.input.addEventListener('blur', () => {
      this.syncFromText(true);
    });
    this.decreaseButton.addEventListener('click', () => {
      this.changeBy(-this.step, true);
    });
    this.increaseButton.addEventListener('click', () => {
      this.changeBy(this.step, true);
    });
  }
}

new AccessibleSpinbutton(document.getElementById('guest-count'), {
  formatValueText: (value) => `${value}人`,
  errorMessage: '宿泊人数は1人から12人までの整数で入力してください。',
});
```

この実装は、上下矢印、Home、End、任意操作のPage Up・Page Downを処理する。それ以外のキーはブラウザへ渡すため、文字入力、カーソル移動、選択、コピー、貼り付け、取り消しを妨げない。

手入力中に値が一時的に空欄や不完全になった場合は、`aria-valuenow`と`aria-valuetext`を外す。フォーカスが外れた時点で不正なら、`aria-invalid="true"`とエラーメッセージを示す。

## 曜日など文字で表す値

内部値が`1`から`7`でも、利用者へ曜日として示すなら`aria-valuetext`を使う。

```js
const weekdays = [
  '月曜日',
  '火曜日',
  '水曜日',
  '木曜日',
  '金曜日',
  '土曜日',
  '日曜日',
];

function formatWeekday(value) {
  return weekdays[value - 1];
}
```

見える入力欄にも数字だけでなく曜日名を表示する場合、数値の直接編集をどのように受け付けるかを明確にする。候補が七つしかないなら、ネイティブの`select`やラジオボタンの方が理解しやすいこともある。

`aria-valuetext`は、数字だけでは意味が不十分な場合に使う。数値と同じ文字列を繰り返すためには使わない。

> 参照: [W3C WAI-ARIA APG — Communicating Value and Limits for Range Widgets](https://www.w3.org/WAI/ARIA/apg/practices/range-related-properties/)

## マウスホイールを扱う場合

フォーカス中の数値入力をスクロールしたとき、ブラウザによって値が変わる場合がある。ページを読むためのスクロールで値が変わると、利用者が変更に気づかない可能性がある。

独自に`wheel`イベントで増減機能を追加しない。製品としてホイール操作を提供する場合は、フォーカス状態、値変更の表示、ページスクロールとの衝突を対象ブラウザで検証する。

ホイールによる値変更を止めるために、スクロールのたびに入力からフォーカスを外す実装も避ける。キーボード利用者の現在位置を失わせる。

## 動的な最小値と最大値

在庫、日付、別の入力値によって範囲が変わる場合は、制約だけでなく現在値と説明も同期する。

```js
function updateAvailableQuantity(input, availableStock, hint) {
  input.max = String(availableStock);
  hint.textContent = `現在は最大${availableStock}個まで注文できます。`;

  if (
    Number.isFinite(input.valueAsNumber) &&
    input.valueAsNumber > availableStock
  ) {
    input.setAttribute('aria-invalid', 'true');
  }
}
```

範囲が狭くなったとき、現在値を黙って最大値へ書き換えるか、エラーとして利用者に修正を求めるかを仕様化する。価格や注文数など利用者の意思に影響する値は、勝手に変更せず理由を伝える方が安全な場合がある。

## `disabled`と`readonly`

```html
<input type="number" value="3" disabled>
<input type="number" value="3" readonly>
```

| 状態 | フォーカス | フォーム送信 | 用途 |
|---|---|---|---|
| `disabled` | 通常は受け取らない | 送信されない | 現在利用できない入力 |
| `readonly` | 受け取れる | 送信される | 値を確認できるが編集できない入力 |

値を単に表示するだけなら、入力欄ではなく通常のテキストを使う。無効にした理由が分かりにくい場合は、周囲の説明で伝える。

外付けの増加・減少ボタンを使う場合、入力の状態と各ボタンの`disabled`も同期する。

## よくある失敗

### 郵便番号へ`type="number"`を使う

数字で構成されていても識別子は数値入力ではない。`type="text"`と適切な`inputmode`を使う。

### ラベルを増減ボタンだけに付ける

入力欄自体にも「数量」「宿泊人数」などの名前が必要。記号ボタンにも固有の操作名を付ける。

### 範囲や単位を説明しない

HTML属性だけに依存せず、許可範囲、刻み幅、単位を画面上へ示す。

### 入力のたびに値を丸める

直接入力の途中状態を壊さない。検証するタイミングと、自動補正する範囲を決める。

### 不正な文字をすべて`keydown`で止める

標準の編集、貼り付け、IME、音声入力を妨げる。入力後の制約検証を使う。

### 最小値や最大値を超えてもボタンを有効にする

値が変わらないボタンを押し続けさせない。境界では該当ボタンを無効にし、入力の制約とも一致させる。

### ボタン操作後の値が伝わらない

フォーカスが外付けボタンに残る場合は、必要に応じて変更結果を簡潔に通知する。入力欄の操作と重複して過剰に読み上げない。

### `role="spinbutton"`だけを付ける

ARIAは動作を追加しない。現在値の同期、キーボード操作、直接入力、エラー処理、フォーム送信をJavaScriptで実装する必要がある。

### クライアント側だけで検証する

HTML属性やJavaScriptは変更できる。サーバー側でも型、範囲、刻み幅、業務上の制約を検証する。

## テストのチェックリスト

### パターンの選択

- [ ] 一段階増減する意味がある値に使っている
- [ ] 郵便番号、電話番号、カード番号などの識別子ではない
- [ ] 候補が少ない場合は`select`やラジオボタンも検討した
- [ ] ネイティブの`input type="number"`で要件を満たせない理由がある

### 名前と制約

- [ ] 見えるラベルが入力へ関連付いている
- [ ] 必要な最小値、最大値、刻み幅が設定されている
- [ ] 範囲、刻み幅、単位を画面上でも理解できる
- [ ] 空欄と`0`を区別している
- [ ] 初期値が利用者へ不利益を与えない
- [ ] 見える値、フォーム値、アクセシビリティ上の値が一致する

### キーボードと直接入力

- [ ] Tab / Shift + Tabで入力へ出入りできる
- [ ] 上下矢印で値を増減できる
- [ ] カスタム実装ではHomeとEndを操作できる
- [ ] 文字選択、コピー、貼り付け、取り消しを利用できる
- [ ] 空欄、負数、小数を入力する途中状態を壊さない
- [ ] 最小値未満、最大値超過、刻み幅不一致を検出できる

### 増加・減少ボタン

- [ ] 記号だけでなく操作を表す名前がある
- [ ] `type="button"`でフォームを誤送信しない
- [ ] 操作領域が十分に大きい
- [ ] 境界で該当するボタンが無効になる
- [ ] ボタン操作後の現在値を確認できる
- [ ] 入力が無効・読み取り専用になったときボタンも同期する

### エラーと動的更新

- [ ] エラーを色だけに依存せずテキストで示す
- [ ] エラーと入力がプログラム的に関連付いている
- [ ] 修正方法を具体的に伝える
- [ ] 修正後にエラー状態を解除する
- [ ] 動的な制約変更を説明とボタンへ反映する
- [ ] サーバー側でも値を検証する

### 表示と入力方法

- [ ] フォーカス表示が見える
- [ ] 200%・400%拡大でもラベル、入力、ボタンが重ならない
- [ ] 強制カラーモードで境界と状態を確認できる
- [ ] キーボード、マウス、タッチ、音声入力で操作できる
- [ ] スクリーンリーダーで名前、役割、現在値、範囲、エラーを確認した
- [ ] マウスホイールで意図しない値変更が起きないか確認した

## 関連して読むとよいテーマ

- [アクセシブルなレンジスライダーの実装 — 現在値を読み上げる範囲入力](./accessible-range-slider.md)
- [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [アクセシブルなラジオグループの実装 — 矢印キーで一つを選ぶ](./accessible-radio-group.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)

## まとめ

- 数量や人数のように増減する意味がある数値には、まず`input type="number"`を使う
- 郵便番号やカード番号などの識別子には数値入力を使わない
- ラベル、範囲、刻み幅、単位を操作前に理解できるようにする
- 空欄と`0`を区別し、直接入力の途中状態を壊さない
- 標準のテキスト編集キー、貼り付け、IMEをJavaScriptで妨げない
- 外付けボタンには固有の名前を付け、境界で無効にする
- カスタム実装では現在値、値テキスト、エラー、フォーム値を同じ状態から更新する
- 入力値を勝手に補正せず、問題と修正方法を伝える
- クライアント側とサーバー側の両方で値を検証する
