# アクセシブルなデートピッカーの実装 — カレンダーを矢印キーで操作する

## デートピッカーとは

デートピッカー（Date Picker）は、日付入力欄とカレンダーUIを組み合わせ、日付を選択するコンポーネント。予約日、配送日、生年月日、期間検索などで使われる。

カレンダーは、見た目以上に複雑なUI。

- 月、曜日、日付の関係を伝える
- キーボードで二次元に移動する
- 今日、フォーカス中の日付、選択済みの日付を区別する
- 前後の月へ移動してもフォーカスを維持する
- 選べない日付と、その理由を伝える
- 入力形式とエラーを明確にする
- ダイアログを開閉し、フォーカスを適切に戻す

見た目だけのカレンダーを一から作る前に、ネイティブHTMLで要件を満たせるか確認する。

> 参照: [W3C WAI-ARIA APG — Date Picker Dialog Example](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/examples/datepicker-dialog/)

## まず`input type="date"`を検討する

単一の日付を入力するだけなら、ネイティブの`input type="date"`を優先する。

```html
<label for="delivery-date">配送希望日</label>
<input
  id="delivery-date"
  name="delivery-date"
  type="date"
  min="2026-10-30"
  max="2027-03-31"
  aria-describedby="delivery-date-hint"
  required
>
<p id="delivery-date-hint">
  2026年10月30日から2027年3月31日まで選べます。
</p>
```

### ネイティブ入力の利点

- ブラウザとOSに合った入力UIを利用できる
- キーボード、タッチ、音声入力との統合をブラウザに任せられる
- `min`、`max`、`step`、`required`などの制約を使える
- フォーム送信値が`YYYY-MM-DD`形式になる
- JavaScriptが失敗しても日付を入力できる

画面上の表示形式は環境によって異なっても、`value`は`YYYY-MM-DD`形式。表示を無理に統一するためだけにカスタムUIへ置き換えない。

> 参照: [WHATWG HTML Standard — Date state (`type=date`)](https://html.spec.whatwg.org/dev/input.html#date-state-(type=date))

### ネイティブ入力でも確認すること

- 対象ブラウザとスクリーンリーダーでラベル、値、制約が伝わるか
- キーボードだけで値を入力・変更できるか
- モバイルのソフトウェアキーボードやピッカーを利用できるか
- エラーが色だけでなくテキストでも伝わるか
- `min`と`max`の範囲を入力前に理解できるか

ブラウザ標準のピッカーを開くアイコンは、製品側で細かく制御できない。ブランド表現より、入力の堅牢性を優先する。

## カスタム実装が必要になる場合

次のような要件では、カスタムデートピッカーを検討することがある。

- 宿泊日のように開始日と終了日を連続して選ぶ
- 営業日、予約枠、料金など日付ごとの情報を表示する
- 特定の曜日や満席日を選択不可にする
- 複数の日付を選ぶ
- 月をまたぐ範囲を視覚的に確認する

一方、次の理由だけならネイティブ入力を維持できないか再検討する。

- 色や角丸をデザインシステムへ合わせたい
- すべてのブラウザで同じカレンダーを見せたい
- JavaScriptライブラリを導入済みだから使いたい

カスタム実装では、ARIA属性だけでなく、日付計算、キーボード操作、フォーカス管理、入力検証をすべて製品側が保守する。

## カスタムデートピッカーの構造

この記事では、次の要素を組み合わせる。

| 部分 | HTML・パターン | 役割 |
|---|---|---|
| 日付入力 | ラベル付きの`input` | 値の入力と送信 |
| 開く操作 | `button` | カレンダーを開く |
| ポップアップ | モーダル`dialog` | カレンダー内の操作へ集中させる |
| カレンダー | `table role="grid"` | 日付を二次元に移動する |
| 日付 | 各セル内の`button` | 日付を選ぶ |
| 月移動 | 前月・次月の`button` | 表示月を変更する |

一般的な`table`は読み取るための表だが、カレンダーでは矢印キーで日付間を移動するため、インタラクティブな`grid`として扱う。

`grid`は複合ウィジェット。Tabキーではグリッド全体を一つの移動先として扱い、内部では矢印キーを使う。ARIAを付けただけでは動かないため、ロービング`tabindex`をJavaScriptで実装する。

> 参照:
> - [W3C WAI-ARIA APG — Grid Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/grid/)
> - [W3C WAI-ARIA APG — Dialog (Modal) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

## HTML構造

```html
<div class="date-field">
  <label for="reservation-date">予約日</label>

  <div class="date-field__input">
    <input
      id="reservation-date"
      name="reservation-date"
      type="text"
      inputmode="numeric"
      autocomplete="off"
      placeholder="YYYY-MM-DD"
      aria-describedby="reservation-date-hint reservation-date-error"
    >

    <button
      id="open-calendar"
      type="button"
      aria-haspopup="dialog"
      aria-controls="calendar-dialog"
    >
      <span aria-hidden="true">📅</span>
      カレンダーを開く
    </button>
  </div>

  <p id="reservation-date-hint">
    西暦4桁、月2桁、日2桁の順で入力してください。例: 2026-10-30
  </p>
  <p id="reservation-date-error" class="error" hidden></p>
</div>

<dialog
  id="calendar-dialog"
  class="date-picker"
  aria-labelledby="calendar-dialog-title"
>
  <h2 id="calendar-dialog-title">予約日を選択</h2>

  <div class="date-picker__header">
    <button id="previous-month" type="button" aria-label="前の月">
      <span aria-hidden="true">←</span>
    </button>

    <p id="calendar-month" aria-live="polite" aria-atomic="true"></p>

    <button id="next-month" type="button" aria-label="次の月">
      <span aria-hidden="true">→</span>
    </button>
  </div>

  <table
    id="calendar-grid"
    role="grid"
    aria-labelledby="calendar-month"
    aria-describedby="calendar-help"
  >
    <thead>
      <tr>
        <th scope="col" abbr="日曜日">日</th>
        <th scope="col" abbr="月曜日">月</th>
        <th scope="col" abbr="火曜日">火</th>
        <th scope="col" abbr="水曜日">水</th>
        <th scope="col" abbr="木曜日">木</th>
        <th scope="col" abbr="金曜日">金</th>
        <th scope="col" abbr="土曜日">土</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <p id="calendar-help">
    矢印キーで日付を移動し、EnterキーまたはSpaceキーで選択します。
    Page UpとPage Downで月を移動します。
  </p>

  <div class="date-picker__actions">
    <button id="choose-today" type="button">今日</button>
    <button id="cancel-calendar" type="button">キャンセル</button>
  </div>
</dialog>
```

### 入力欄

カレンダーを使えない、または使いたくない利用者のために、テキストでも入力できるようにする。

- 見える`label`を付ける
- 期待する形式を入力前に示す
- `placeholder`だけをラベルや説明にしない
- エラー要素を最初から`aria-describedby`で関連付ける
- 書式だけでなく、実在する日付かを検証する

`2026-02-31`はパターンには一致しても存在しない。正規表現だけで日付の妥当性を判定しない。

### カレンダーを開くボタン

カレンダーアイコンだけにせず、「カレンダーを開く」という見えるテキストを置く。日付選択後は、アクセシブルネームへ現在値を加えると、フォーカスが戻ったときに選択結果を確認できる。

```js
openButton.setAttribute(
  'aria-label',
  `カレンダーを開く、選択中: ${formattedDate}`
);
```

見える「カレンダーを開く」という文言をアクセシブルネームにも残すため、音声操作でも指定しやすい。

### 曜日の見出し

狭いセルでは「日」「月」のように省略しても、`abbr`属性で「日曜日」「月曜日」という完全な名前を提供できる。

週の開始曜日は地域とサービスの対象に合わせる。月曜日始まりへ変える場合は、見出しの順序、Home／Endの移動先、日付計算をすべて同期する。

## キーボード操作

### カレンダーを開くボタン

| キー | 動作 |
|---|---|
| Enter / Space | ダイアログを開き、選択日または今日へフォーカスを移す |

### ダイアログ

| キー | 動作 |
|---|---|
| Tab / Shift + Tab | 前後の月、日付グリッド、今日、キャンセルの間を移動する |
| Escape | 選択を変更せず閉じ、開くボタンへ戻る |

グリッド内でTabキーを押すたびに42個の日付を通過させない。グリッド内では一つの日付だけを`tabindex="0"`にし、残りを`-1`にする。

### 日付グリッド

| キー | 動作 |
|---|---|
| ← / → | 前日 / 翌日へ移動 |
| ↑ / ↓ | 7日前 / 7日後へ移動 |
| Home / End | 現在の週の先頭 / 末尾へ移動 |
| Page Up / Page Down | 前月 / 翌月の同じ日へ移動 |
| Shift + Page Up / Page Down | 前年 / 翌年の同じ日へ移動 |
| Enter / Space | フォーカス中の日付を選択する |

移動先の月に同じ日が存在しない場合は、その月の末日に合わせる。たとえば3月31日からPage Upで2月へ移動した場合、2月の末日へ移動する。

ブラウザのスクロールに矢印キーやSpaceキーが使われないよう、処理したキーだけ`preventDefault()`する。

## 日付の三つの状態を区別する

カレンダーには、少なくとも三つの異なる状態がある。

| 状態 | 意味 | 表現例 |
|---|---|---|
| フォーカス | キーボード操作の現在位置 | `:focus-visible` |
| 選択済み | 入力欄へ設定される値 | グリッドセルの`aria-selected="true"` |
| 今日 | 現実の今日の日付 | 日付ボタンの`aria-current="date"` |

フォーカス中の日付が、選択済みとは限らない。ダイアログ内を移動しただけで入力値を変更しない。EnterまたはSpaceで確定したときに選択値を更新する。

```html
<td role="gridcell" aria-selected="true">
  <button type="button" tabindex="0" data-date="2026-10-30">
    30
  </button>
</td>
```

今日と選択日が同じ場合は、`aria-current="date"`と`aria-selected="true"`の両方が存在し得る。見た目でも、今日の印と選択中の塗りを区別する。

## 日付計算のヘルパー

カレンダーの日付は、ローカルタイムの正午として扱うと、夏時間の切り替わりで日付がずれるリスクを減らせる。

```js
const pad = (value) => String(value).padStart(2, '0');

function createLocalDate(year, monthIndex, day) {
  return new Date(year, monthIndex, day, 12);
}

function toISODate(date) {
  return [
    date.getFullYear(),
    pad(date.getMonth() + 1),
    pad(date.getDate()),
  ].join('-');
}

function parseISODate(value) {
  const match = /^(\d{4})-(\d{2})-(\d{2})$/.exec(value);
  if (!match) return null;

  const [, yearText, monthText, dayText] = match;
  const year = Number(yearText);
  const monthIndex = Number(monthText) - 1;
  const day = Number(dayText);
  const date = createLocalDate(year, monthIndex, day);

  if (
    date.getFullYear() !== year ||
    date.getMonth() !== monthIndex ||
    date.getDate() !== day
  ) {
    return null;
  }

  return date;
}

function addDays(date, amount) {
  return createLocalDate(
    date.getFullYear(),
    date.getMonth(),
    date.getDate() + amount
  );
}

function addMonths(date, amount) {
  const day = date.getDate();
  const firstOfTarget = createLocalDate(
    date.getFullYear(),
    date.getMonth() + amount,
    1
  );
  const lastDay = createLocalDate(
    firstOfTarget.getFullYear(),
    firstOfTarget.getMonth() + 1,
    0
  ).getDate();

  return createLocalDate(
    firstOfTarget.getFullYear(),
    firstOfTarget.getMonth(),
    Math.min(day, lastDay)
  );
}
```

`new Date('2026-10-30')`はUTCとして解釈される環境があり、ローカルタイムへ変換したときに前日になる可能性がある。日付だけの値は文字列から年月日を分けて生成する。

同様に、ローカル日付を`toISOString().slice(0, 10)`で保存するとずれる可能性がある。日付だけを扱う場合は、ローカルの年月日から`YYYY-MM-DD`を組み立てる。

## カレンダーの描画

次の例では、常に6週間分の42セルを描画する。前月・翌月の日付も含めることで、矢印キーで月境界を連続して移動できる。

```js
const dateFormatter = new Intl.DateTimeFormat('ja-JP', {
  dateStyle: 'full',
});
const monthFormatter = new Intl.DateTimeFormat('ja-JP', {
  year: 'numeric',
  month: 'long',
});

class DatePicker {
  constructor({ input, openButton, dialog, grid }) {
    this.input = input;
    this.openButton = openButton;
    this.dialog = dialog;
    this.grid = grid;
    this.gridBody = grid.querySelector('tbody');
    this.monthLabel = document.getElementById('calendar-month');
    const now = new Date();
    this.activeDate = createLocalDate(
      now.getFullYear(),
      now.getMonth(),
      now.getDate()
    );
    this.selectedDate = null;

    this.openButton.addEventListener('click', () => this.open());
    this.input.addEventListener('change', () => this.syncButtonLabel());
    document.getElementById('previous-month').addEventListener('click', () => {
      this.changeMonth(-1);
    });
    document.getElementById('next-month').addEventListener('click', () => {
      this.changeMonth(1);
    });
    document.getElementById('choose-today').addEventListener('click', () => {
      const now = new Date();
      this.selectDate(
        createLocalDate(now.getFullYear(), now.getMonth(), now.getDate())
      );
    });
    document.getElementById('cancel-calendar').addEventListener('click', () => {
      this.dialog.close();
    });

    this.grid.addEventListener('keydown', (event) => {
      this.handleGridKeyDown(event);
    });
    this.grid.addEventListener('click', (event) => {
      const button = event.target.closest('button[data-date]');
      if (!button || button.getAttribute('aria-disabled') === 'true') return;

      this.selectDate(parseISODate(button.dataset.date));
    });
    this.dialog.addEventListener('close', () => {
      this.openButton.focus();
    });

    this.syncButtonLabel();
  }

  open() {
    const inputDate = parseISODate(this.input.value);
    const now = new Date();
    const today = createLocalDate(
      now.getFullYear(),
      now.getMonth(),
      now.getDate()
    );

    this.selectedDate = inputDate;
    this.activeDate = inputDate ?? today;
    this.render();
    this.dialog.showModal();
    this.focusActiveDate();
  }

  render() {
    this.monthLabel.textContent = monthFormatter.format(this.activeDate);
    this.gridBody.replaceChildren();

    const firstOfMonth = createLocalDate(
      this.activeDate.getFullYear(),
      this.activeDate.getMonth(),
      1
    );
    const gridStart = addDays(firstOfMonth, -firstOfMonth.getDay());
    const today = new Date();
    const todayValue = toISODate(
      createLocalDate(today.getFullYear(), today.getMonth(), today.getDate())
    );

    for (let week = 0; week < 6; week += 1) {
      const row = document.createElement('tr');

      for (let weekday = 0; weekday < 7; weekday += 1) {
        const date = addDays(gridStart, week * 7 + weekday);
        const value = toISODate(date);
        const cell = document.createElement('td');
        const button = document.createElement('button');
        const isActive = value === toISODate(this.activeDate);
        const isSelected =
          this.selectedDate && value === toISODate(this.selectedDate);
        const isOutsideMonth = date.getMonth() !== this.activeDate.getMonth();

        cell.setAttribute('role', 'gridcell');
        if (isSelected) cell.setAttribute('aria-selected', 'true');

        button.type = 'button';
        button.dataset.date = value;
        button.tabIndex = isActive ? 0 : -1;
        button.textContent = String(date.getDate());
        button.setAttribute('aria-label', dateFormatter.format(date));
        button.classList.toggle('is-outside-month', isOutsideMonth);

        if (value === todayValue) {
          button.setAttribute('aria-current', 'date');
        }

        cell.appendChild(button);
        row.appendChild(cell);
      }

      this.gridBody.appendChild(row);
    }
  }

  focusActiveDate() {
    const value = toISODate(this.activeDate);
    this.grid.querySelector(`[data-date="${value}"]`)?.focus();
  }

  moveActiveDate(date) {
    this.activeDate = date;
    this.render();
    this.focusActiveDate();
  }

  changeMonth(amount) {
    this.activeDate = addMonths(this.activeDate, amount);
    this.render();
  }

  selectDate(date) {
    if (!date) return;

    this.selectedDate = date;
    this.input.value = toISODate(date);
    this.input.dispatchEvent(new Event('input', { bubbles: true }));
    this.input.dispatchEvent(new Event('change', { bubbles: true }));
    this.dialog.close();
  }

  syncButtonLabel() {
    const date = parseISODate(this.input.value);

    if (date) {
      this.openButton.setAttribute(
        'aria-label',
        `カレンダーを開く、選択中: ${dateFormatter.format(date)}`
      );
    } else {
      this.openButton.removeAttribute('aria-label');
    }
  }

  handleGridKeyDown(event) {
    const button = event.target.closest('button[data-date]');
    if (!button) return;

    const date = parseISODate(button.dataset.date);
    if (!date) return;

    let nextDate = null;

    switch (event.key) {
      case 'ArrowLeft':
        nextDate = addDays(date, -1);
        break;
      case 'ArrowRight':
        nextDate = addDays(date, 1);
        break;
      case 'ArrowUp':
        nextDate = addDays(date, -7);
        break;
      case 'ArrowDown':
        nextDate = addDays(date, 7);
        break;
      case 'Home':
        nextDate = addDays(date, -date.getDay());
        break;
      case 'End':
        nextDate = addDays(date, 6 - date.getDay());
        break;
      case 'PageUp':
        nextDate = addMonths(date, event.shiftKey ? -12 : -1);
        break;
      case 'PageDown':
        nextDate = addMonths(date, event.shiftKey ? 12 : 1);
        break;
      case 'Enter':
      case ' ':
        event.preventDefault();
        if (button.getAttribute('aria-disabled') !== 'true') {
          this.selectDate(date);
        }
        return;
      default:
        return;
    }

    event.preventDefault();
    this.moveActiveDate(nextDate);
  }
}

const datePicker = new DatePicker({
  input: document.getElementById('reservation-date'),
  openButton: document.getElementById('open-calendar'),
  dialog: document.getElementById('calendar-dialog'),
  grid: document.getElementById('calendar-grid'),
});
```

この例は単一日付を選ぶための基礎。実際の要件に合わせて、入力検証、選択可能範囲、ロケール、エラー処理を追加する。

### 月移動ボタンのフォーカス

前月・次月ボタンを押したときは、フォーカスをそのボタンに残す。月見出しを`aria-live="polite"`にしているため、新しい月が通知される。

グリッド内でPage Up／Page Downを押したときは、同じ日または移動先月の末日へフォーカスを移す。同じ「月移動」でも、操作の起点によってフォーカスの扱いが異なる。

## 選択できない日付

予約済みや範囲外の日付には、見た目だけでなくプログラム上の状態を付ける。

```html
<td role="gridcell">
  <button
    type="button"
    tabindex="-1"
    data-date="2026-11-03"
    aria-disabled="true"
    aria-label="2026年11月3日 火曜日、予約できません"
  >
    3
  </button>
</td>
```

`disabled`属性を使うとフォーカスできなくなり、矢印キーで日付を確認する流れから抜け落ちる。利用者が「その日は存在するが選べない」と理解する必要がある場合は、`aria-disabled="true"`でフォーカス可能なままにし、選択処理を止める方法がある。

- 選択できない理由をアクセシブルネームまたは関連する説明で伝える
- 色だけで無効状態を表さない
- ポインター操作でも選択処理を止める
- EnterとSpaceでも選択処理を止める
- 矢印キーでは無効な日付へ移動できるようにするか、スキップするかを仕様化する

無効な日付が多い場合、カレンダーだけで空きを探させず、「次に予約できる日」への移動や日付候補のリストも検討する。

## 入力値の検証とエラー

テキスト入力では、フォーカスが外れたときやフォーム送信時に次を検証する。

1. 空欄を許可するか
2. `YYYY-MM-DD`形式か
3. 実在する日付か
4. 選択可能な範囲内か
5. 営業日や予約枠など業務ルールを満たすか

```js
function showDateError(input, errorElement, message) {
  input.setAttribute('aria-invalid', 'true');
  errorElement.textContent = message;
  errorElement.hidden = false;
}

function clearDateError(input, errorElement) {
  input.removeAttribute('aria-invalid');
  errorElement.textContent = '';
  errorElement.hidden = true;
}
```

- エラー時だけ`aria-invalid="true"`を付ける
- エラー文を入力欄と`aria-describedby`で関連付ける
- 「形式が違います」だけでなく、修正方法を示す
- 入力内容を勝手に消さない
- フォーム送信時は最初のエラーへフォーカスを移す
- クライアント側だけでなくサーバー側でも検証する

```text
入力された「2026-02-31」は実在しない日付です。
YYYY-MM-DD形式で、実在する日付を入力してください。
```

カレンダーで選択した値も、送信前に同じルールで検証する。UIを迂回したリクエストや、選択後に予約状況が変わる場合がある。

## CSSの例

```css
.date-field__input {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.date-picker {
  inline-size: min(92vw, 30rem);
  max-block-size: 90vh;
  padding: 1.25rem;
  overflow: auto;
  border: 1px solid CanvasText;
  color: CanvasText;
  background: Canvas;
}

.date-picker::backdrop {
  background: rgb(0 0 0 / 55%);
}

.date-picker__header {
  display: grid;
  grid-template-columns: auto 1fr auto;
  align-items: center;
  gap: 0.5rem;
}

#calendar-month {
  margin: 0;
  text-align: center;
  font-weight: 700;
}

#calendar-grid {
  inline-size: 100%;
  border-collapse: separate;
  border-spacing: 0.25rem;
}

#calendar-grid th {
  font-weight: 400;
}

#calendar-grid button {
  inline-size: 100%;
  min-inline-size: 2.75rem;
  min-block-size: 2.75rem;
  border: 1px solid transparent;
  color: inherit;
  background: transparent;
}

#calendar-grid [aria-selected="true"] button {
  color: Canvas;
  background: CanvasText;
}

#calendar-grid [aria-current="date"] {
  border-color: currentColor;
  font-weight: 700;
}

#calendar-grid button[aria-disabled="true"] {
  text-decoration: line-through;
  opacity: 0.6;
}

#calendar-grid .is-outside-month {
  opacity: 0.65;
}

.date-picker :focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.125rem;
}

.date-picker__actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
  margin-block-start: 1rem;
}

@media (max-width: 30rem) {
  #calendar-grid {
    border-spacing: 0.125rem;
  }

  #calendar-grid button {
    min-inline-size: 2.5rem;
  }
}
```

フォーカス枠を`box-shadow`だけで表現すると、強制カラーモードで消えることがある。`outline`や`border`を使い、選択、今日、フォーカス、無効状態をそれぞれ確認する。

小さい画面でも日付セルを過度に縮めない。収まらない場合は、ダイアログの余白や曜日表記を調整し、横スクロールを最後の手段にする。

## 日付範囲を選ぶ場合

開始日と終了日を選ぶデートレンジピッカーでは、単一日付より状態が増える。

- どちらを選択中かを見出しや説明で伝える
- 開始日、終了日、範囲内の日付を見た目で区別する
- 終了日が開始日より前の場合の扱いを明示する
- キーボード操作だけで開始・終了を確定できるようにする
- 二つのテキスト入力を用意し、カレンダーなしでも入力できるようにする
- 選択後、「2026年11月2日から11月5日」のように範囲全体を確認できるようにする

ドラッグだけで範囲を選ばせない。開始日と終了日を個別にクリック、タップ、Enterで選べるようにする。

カレンダーを二か月横並びにする場合も、DOM順序と矢印キーの順序を一致させる。狭い画面では一か月表示へ切り替える。

## ロケールとタイムゾーン

### 表示形式

`Intl.DateTimeFormat`を使うと、対象ロケールに合った日付名や月名を生成できる。

```js
new Intl.DateTimeFormat('ja-JP', { dateStyle: 'full' }).format(date);
// 例: 2026年10月30日金曜日
```

- 保存形式と表示形式を分ける
- 入力形式を固定するなら、例と説明を明示する
- 曜日の開始位置を対象地域に合わせる
- 月名や曜日名を文字列の配列へ固定しない

### 日付だけか、日時か

誕生日や予約日のような「日付だけの値」と、イベント開始時刻のような「瞬間」を区別する。

- 日付だけ: `YYYY-MM-DD`として扱い、不要なUTC変換をしない
- 日時: タイムゾーンを含む仕様を決める
- サーバーとクライアントで解釈を一致させる

日付だけの値へ安易に`Z`を付けてUTCの瞬間へ変換しない。

## よくある失敗

### テキスト入力を削除する

カレンダー操作が難しい利用者が日付を直接入力できなくなる。カレンダーは入力方法の一つとして追加する。

### すべての日付をTabキーで移動させる

一か月だけで最大42個のTab停止位置ができる。グリッド内は一つだけTab順序へ入れ、矢印キーで移動する。

### 矢印キーでフォーカスだけが見えなくなる

月境界でDOMを再描画した後、対応する日付へ実際のDOMフォーカスを移す。`tabindex`の値だけを変えない。

### 今日と選択日を同じ状態として扱う

今日が選択されているとは限らない。`aria-current="date"`、`aria-selected`、フォーカスを分ける。

### 月を変えても通知しない

画面の見出しだけが変わると、スクリーンリーダー利用者は表示月を把握できない。月・年の見出しをライブリージョンにする。

### アイコンだけの月移動ボタン

左右矢印だけでは目的が伝わらない。「前の月」「次の月」というアクセシブルネームを付ける。

### 選べない日付を色だけで薄くする

状態が伝わらず、選択処理も残る。`aria-disabled`、視覚表現、イベント処理を同期する。

### ダイアログを閉じた後にフォーカスを失う

選択、キャンセル、Escapeのどの経路でも、カレンダーを開いたボタンへフォーカスを戻す。

### `toISOString()`で日付がずれる

日付だけの値をUTCへ変換すると、タイムゾーンによって前日・翌日になる。ローカルの年月日から保存文字列を作る。

## テストのチェックリスト

### 入力欄と値

- [ ] 見えるラベルと入力形式の説明がある
- [ ] カレンダーを使わずテキストだけでも入力できる
- [ ] 実在しない日付、範囲外、業務ルール違反を検証する
- [ ] エラー内容と修正方法がテキストで伝わる
- [ ] エラー時に`aria-invalid`と説明の関連付けが更新される
- [ ] 送信後も日付がタイムゾーンでずれない

### ダイアログとフォーカス

- [ ] Enter / Spaceでカレンダーを開ける
- [ ] 開いたとき、選択日または今日へフォーカスが移る
- [ ] Tab / Shift + Tabがダイアログ内に留まる
- [ ] Escapeで変更せず閉じられる
- [ ] 選択、キャンセル、Escapeの後に開くボタンへ戻る
- [ ] ダイアログの名前と月・年が読み上げられる

### 日付グリッド

- [ ] グリッド内のTab停止位置が一つだけである
- [ ] 左右矢印で前日・翌日へ移動できる
- [ ] 上下矢印で前週・翌週へ移動できる
- [ ] Home / Endで週の先頭・末尾へ移動できる
- [ ] Page Up / Page Downで前月・翌月へ移動できる
- [ ] Shiftとの組み合わせで前年・翌年へ移動できる
- [ ] Enter / Spaceでフォーカス中の日付を選べる
- [ ] 月境界を越えてもフォーカスと表示月が一致する

### 状態と表示

- [ ] 今日、選択日、フォーカス、無効日を区別できる
- [ ] 状態を色だけに依存していない
- [ ] 月変更が支援技術へ通知される
- [ ] 曜日の完全な名前が伝わる
- [ ] 無効日の理由と選択できない状態が伝わる
- [ ] 200%・400%拡大でも日付と操作ボタンを利用できる
- [ ] 強制カラーモードでも境界とフォーカスが見える
- [ ] タッチ、マウス、キーボード、スクリーンリーダーで確認した

## 関連して読むとよいテーマ

- [アクセシブルなモーダルダイアログの実装](./accessible-modal-dialog.md)
- [ロービングタブインデックスパターン](./roving-tabindex.md)
- [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)
- [フォーカスインジケーターのカスタマイズ — :focus-visibleの活用](./focus-visible.md)

## まとめ

- 単一の日付入力なら、まずネイティブの`input type="date"`を検討する
- カスタム実装でもテキスト入力を残し、カレンダーを別の入力方法として提供する
- モーダルDialogとGridを組み合わせ、Tabキーはグリッド全体で一停止位置にする
- グリッド内は矢印、Home、End、Page Up、Page Downで日付を移動する
- フォーカス、選択日、今日、無効日を別の状態として伝える
- 月変更を通知し、月境界を越えても日付へフォーカスを維持する
- 日付だけの値を不要にUTC変換せず、入力形式とタイムゾーンの仕様を明確にする
- ブラウザと支援技術の組み合わせで、キーボード、読み上げ、拡大、強制カラーをテストする
