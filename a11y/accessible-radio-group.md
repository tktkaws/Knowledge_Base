# アクセシブルなラジオグループの実装 — 矢印キーで一つを選ぶ

## ラジオグループとは

ラジオグループ（Radio Group）は、複数の候補から一つだけを選ぶフォームUI。配送方法、支払い方法、連絡手段、評価などに使われる。

ラジオボタンは、一つずつ独立した入力ではなく、**共通の質問に対する相互排他的な選択肢の集合**として設計する。

- グループ全体の質問を伝える
- 各選択肢へ固有のラベルを付ける
- 同じグループでは一つだけ選択できるようにする
- Tabキーでグループへ入り、矢印キーで選択肢を移動する
- 選択状態を見た目と支援技術の両方へ伝える
- 必須なのに未選択の場合は、グループ単位でエラーを示す

通常は、ネイティブの`input type="radio"`、`fieldset`、`legend`で実装できる。

> 参照:
> - [W3C WAI-ARIA APG — Radio Group Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/radio/)
> - [W3C WAI — Grouping Controls](https://www.w3.org/WAI/tutorials/forms/grouping/)

## ラジオボタンが向く場面

### ラジオボタン

- 一つだけ選ぶ
- 選択肢を画面上へ同時に見せたい
- 選択肢が少なく、比較しやすい
- 選択を変更できる

```text
配送方法
○ 通常配送
● 速達
○ 店舗受け取り
```

### チェックボックス

複数を同時に選べる場合はチェックボックスを使う。

```text
受け取りたい通知
☑ メール
☐ SMS
☑ プッシュ通知
```

### `select`

都道府県のように候補が多い場合、すべてをラジオボタンで並べると読解と移動の負担が増える。候補を常に比較する必要がなければ、ネイティブの`select`も検討する。

### ボタン群

押すとすぐに処理を実行する操作はボタン。値を選んでフォーム送信するラジオボタンとは目的が異なる。

## 基本のHTML

```html
<fieldset
  class="radio-group"
  aria-describedby="shipping-hint"
>
  <legend>配送方法</legend>
  <p id="shipping-hint">
    配送方法を一つ選んでください。
  </p>

  <div class="radio-option">
    <input
      id="shipping-standard"
      type="radio"
      name="shipping"
      value="standard"
      checked
    >
    <label for="shipping-standard">
      <span class="radio-option__title">通常配送</span>
      <span class="radio-option__description">
        3〜5営業日、送料無料
      </span>
    </label>
  </div>

  <div class="radio-option">
    <input
      id="shipping-express"
      type="radio"
      name="shipping"
      value="express"
    >
    <label for="shipping-express">
      <span class="radio-option__title">速達</span>
      <span class="radio-option__description">
        翌営業日、800円
      </span>
    </label>
  </div>

  <div class="radio-option">
    <input
      id="shipping-store"
      type="radio"
      name="shipping"
      value="store"
    >
    <label for="shipping-store">
      <span class="radio-option__title">店舗受け取り</span>
      <span class="radio-option__description">
        準備完了後、指定店舗で受け取れます
      </span>
    </label>
  </div>
</fieldset>
```

### `fieldset`と`legend`

`fieldset`は関連する入力をグループ化し、`legend`はグループ全体の名前になる。

```text
配送方法、グループ
通常配送、ラジオボタン、選択済み、1/3
```

支援技術による読み上げ方は異なるが、利用者は「通常配送」が何についての選択肢かを把握しやすくなる。

- `legend`は「配送方法」のような短い質問にする
- 各`label`は、単独で聞いても選択肢を区別できる名前にする
- 補足説明は`aria-describedby`で`fieldset`へ関連付ける
- 見出しだけを置いて`fieldset`を省略しない

### `name`で一つのグループにする

同じ`name`を持つラジオボタンは、フォーム上で一つのグループになる。

```html
<input type="radio" name="shipping" value="standard">
<input type="radio" name="shipping" value="express">
```

一つを選ぶと、同じグループの以前の選択が解除される。`name`が異なると複数を同時に選べてしまう。

```html
<!-- 避ける：別々のグループとして扱われる -->
<input type="radio" name="standard" value="standard">
<input type="radio" name="express" value="express">
```

別の質問へ同じ`name`を再利用すると、離れた場所の選択肢まで解除される可能性がある。質問ごとに一意で安定した`name`を付ける。

### `id`と`label`

```html
<input id="payment-card" type="radio" name="payment" value="card">
<label for="payment-card">クレジットカード</label>
```

`for`と`id`を一致させると、ラベルをクリック・タップしても選択できる。小さい円だけを狙う必要がなくなり、音声入力でもラベル名を指定しやすい。

`id`はページ内で一意にする。Reactなどで繰り返し描画する場合も重複させない。

## ネイティブのキーボード操作

| キー | 動作 |
|---|---|
| Tab / Shift + Tab | グループへ入る、またはグループから出る |
| Space | フォーカス中の項目を選択する |
| → / ↓ | 次の項目へ移動して選択する |
| ← / ↑ | 前の項目へ移動して選択する |

ラジオグループはTabキーで一項目ずつ巡回するのではなく、グループ全体が一つのTab停止位置として動作する。

- 選択済みの項目があれば、その項目へTabフォーカスが入る
- 未選択なら、通常は先頭項目へ入る
- 矢印キーでフォーカスと選択が一緒に移動する
- 端から次へ移動したときは、反対側へ循環する実装が一般的

未選択のグループへShift + Tabで入る場合、一部のブラウザでは末尾へフォーカスすることがある。ネイティブHTMLの挙動を独自JavaScriptで無理に上書きせず、対象ブラウザと支援技術で確認する。

ネイティブのラジオボタンへ、矢印キー処理やロービング`tabindex`を追加する必要はない。ブラウザの動作と二重になり、選択が二項目進むなどの不具合につながる。

## 初期選択を設けるか

### 安全な既定値がある場合

配送方法のように、多くの利用者にとって安全で費用の低い選択肢があるなら、初期選択を設けられる。

```html
<input
  type="radio"
  name="shipping"
  value="standard"
  checked
>
```

既定値は、利用者が見落としても不利益が少なく、意図を推測しても問題がないものにする。

### 利用者の明示的な選択が必要な場合

同意、重要な契約条件、取り消しにくい選択では、勝手に初期選択しない。

```html
<fieldset aria-describedby="plan-hint">
  <legend>契約期間</legend>
  <p id="plan-hint">契約期間を選択してください。</p>

  <label>
    <input type="radio" name="term" value="monthly" required>
    月契約
  </label>
  <label>
    <input type="radio" name="term" value="yearly" required>
    年契約
  </label>
</fieldset>
```

必須で未選択の状態はあり得るが、一度選択したラジオボタンをもう一度押して全項目未選択へ戻す操作は通常提供しない。「選択なし」が有効な値なら、明示的な選択肢として追加する。

```html
<label>
  <input type="radio" name="contact-time" value="none">
  希望なし
</label>
```

## 必須グループとエラー表示

同じグループ内のラジオボタンのうち一つに`required`があれば、グループ全体が必須になる。コード生成や保守時に意図が分かるよう、すべてへ付ける方法もある。

```html
<fieldset
  id="contact-method-group"
  aria-describedby="contact-method-hint contact-method-error"
>
  <legend>
    希望する連絡方法
    <span aria-hidden="true">*</span>
  </legend>
  <p id="contact-method-hint">一つ選んでください。</p>

  <label>
    <input type="radio" name="contact-method" value="email" required>
    メール
  </label>
  <label>
    <input type="radio" name="contact-method" value="phone" required>
    電話
  </label>

  <p id="contact-method-error" class="error" hidden>
    希望する連絡方法を選んでください。
  </p>
</fieldset>
```

ブラウザの制約検証を利用しながら、製品のフォーム全体で統一したエラー表示が必要なら、グループ単位の検証を追加する。

```js
const form = document.querySelector('form');
const contactRadios = [
  ...form.querySelectorAll('input[name="contact-method"]'),
];
const contactError = document.getElementById('contact-method-error');

function clearContactError() {
  contactError.hidden = true;
  contactRadios.forEach((radio) => {
    radio.removeAttribute('aria-invalid');
  });
}

contactRadios.forEach((radio) => {
  radio.addEventListener('change', clearContactError);
});

form.addEventListener('invalid', (event) => {
  if (!contactRadios.includes(event.target)) return;

  contactError.hidden = false;
  contactRadios.forEach((radio) => {
    radio.setAttribute('aria-invalid', 'true');
  });
}, true);
```

`invalid`イベントはバブリングしないため、フォームでキャプチャしている。ブラウザが最初の無効な入力へ移動する動作と、グループのエラー表示を組み合わせる。

- エラーをグループの質問と関連付ける
- 「入力エラー」ではなく、何を選ぶ必要があるかを書く
- エラーを色だけで示さない
- 送信時は最初のラジオボタンへフォーカスし、修正場所を明確にする
- 選択後はエラー表示と`aria-invalid`を解除する
- サーバー側でも値が許可された候補かを検証する

`required`を付けても、送信リクエストの改変は防げない。`value`を業務処理へ使う前にサーバーで検証する。

> 参照: [WHATWG HTML Standard — Radio Button state](https://html.spec.whatwg.org/multipage/input.html#radio-button-state-(type=radio))

## カード型の見た目にする

ネイティブ入力を残したまま、ラベルをカードとして装飾できる。

```html
<div class="radio-card">
  <input
    id="plan-basic"
    type="radio"
    name="plan"
    value="basic"
  >
  <label for="plan-basic">
    <strong>ベーシック</strong>
    <span>月額1,000円</span>
  </label>
</div>
```

```css
.radio-card {
  position: relative;
}

.radio-card input {
  position: absolute;
  inline-size: 1px;
  block-size: 1px;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
}

.radio-card label {
  display: grid;
  gap: 0.25rem;
  min-block-size: 2.75rem;
  padding: 1rem;
  border: 0.125rem solid currentColor;
  border-radius: 0.5rem;
  cursor: pointer;
}

.radio-card input:checked + label {
  border-width: 0.25rem;
  padding: 0.875rem;
  background: Canvas;
  font-weight: 700;
}

.radio-card input:focus-visible + label {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

.radio-card input:disabled + label {
  cursor: not-allowed;
  text-decoration: line-through;
  opacity: 0.6;
}
```

### 入力を非表示にしすぎない

```css
/* 避ける */
.radio-card input {
  display: none;
}
```

`display: none`や`visibility: hidden`にすると、入力がTab順序とアクセシビリティツリーから外れる。視覚的に隠す場合も、ラベルへフォーカス表示を出し、キーボード操作を確認する。

### 選択状態を色だけにしない

太い境界、ラジオマーク、チェックアイコン、「選択中」という文字など、色以外の手がかりを使う。

CSSの`accent-color`だけで十分な要件なら、ネイティブのラジオボタンを表示したまま色を調整する方が堅牢。

```css
input[type="radio"] {
  accent-color: #005fcc;
}
```

強制カラーモードでは、選択状態とフォーカスが残るかを実機で確認する。背景画像だけでラジオマークを作らない。

## 選択肢の補足説明

選択肢ごとの説明は、ラベル内へ含める方法が分かりやすい。

```html
<label for="delivery-express">
  <span>速達</span>
  <span>翌営業日、800円</span>
</label>
```

ラベルが長くなりすぎる場合は、説明へIDを付けて`aria-describedby`で関連付ける。

```html
<input
  id="delivery-express"
  type="radio"
  name="delivery"
  value="express"
  aria-describedby="delivery-express-description"
>
<label for="delivery-express">速達</label>
<p id="delivery-express-description">翌営業日、800円</p>
```

- 選択肢名は短く、区別できるようにする
- 価格や到着予定など判断に必要な情報は操作前に見せる
- ツールチップを開かないと比較できない設計を避ける
- ラベル内へ別のリンクやボタンを入れない

ラベル内に「詳細を見る」リンクを入れると、ラジオ選択とリンク起動が競合する。補足リンクはラベルの外へ置き、どの選択肢の説明か分かる構造にする。

## 無効な選択肢

```html
<input
  id="delivery-same-day"
  type="radio"
  name="delivery"
  value="same-day"
  disabled
  aria-describedby="same-day-reason"
>
<label for="delivery-same-day">当日配送</label>
<p id="same-day-reason">
  受付時間を過ぎたため選択できません。
</p>
```

`disabled`のラジオボタンは選択できず、通常のフォーカス順序から外れる。

- 無効な理由を画面上へ表示する
- 色だけで無効状態を示さない
- すべての選択肢が無効なら、グループ自体を表示する必要があるか再検討する
- 現在選択中の項目を無効にした場合は、選択解除と次の操作を明確に伝える

無効な項目の説明をキーボードでも確認する必要がある場合、説明テキスト自体は通常の読み上げ順で到達できるようにする。安易に`tabindex="0"`を追加しない。

## Reactでの実装例

```jsx
import { useId } from 'react';

function RadioGroup({
  name,
  legend,
  options,
  value,
  onChange,
  description,
  error,
  required = false,
}) {
  const groupId = useId();
  const descriptionId = description ? `${groupId}-description` : undefined;
  const errorId = error ? `${groupId}-error` : undefined;
  const describedBy = [descriptionId, errorId].filter(Boolean).join(' ');

  return (
    <fieldset aria-describedby={describedBy || undefined}>
      <legend>
        {legend}
        {required && <span aria-hidden="true"> *</span>}
      </legend>

      {description && <p id={descriptionId}>{description}</p>}

      {options.map((option, index) => {
        const id = `${groupId}-option-${index}`;

        return (
          <div className="radio-option" key={option.value}>
            <input
              id={id}
              type="radio"
              name={name}
              value={option.value}
              checked={value === option.value}
              required={required}
              disabled={option.disabled}
              aria-invalid={error ? 'true' : undefined}
              onChange={(event) => onChange(event.target.value)}
            />
            <label htmlFor={id}>
              <span>{option.label}</span>
              {option.description && (
                <span>{option.description}</span>
              )}
            </label>
          </div>
        );
      })}

      {error && (
        <p id={errorId} className="error">
          {error}
        </p>
      )}
    </fieldset>
  );
}
```

```jsx
import { useState } from 'react';

const shippingOptions = [
  {
    value: 'standard',
    label: '通常配送',
    description: '3〜5営業日、送料無料',
  },
  {
    value: 'express',
    label: '速達',
    description: '翌営業日、800円',
  },
  {
    value: 'store',
    label: '店舗受け取り',
    description: '準備完了後、指定店舗で受け取れます',
  },
];

function ShippingForm() {
  const [shipping, setShipping] = useState('standard');

  return (
    <RadioGroup
      name="shipping"
      legend="配送方法"
      options={shippingOptions}
      value={shipping}
      onChange={setShipping}
      description="配送方法を一つ選んでください。"
      required
    />
  );
}
```

制御コンポーネントでは、選択値を一つのstateで管理する。各項目に別々の真偽値を持たせると、複数項目が選択済みになる矛盾を作りやすい。

サーバー描画とハイドレーションで初期値が変わると、表示とchecked状態が切り替わる。保存済みの値や既定値をサーバーとクライアントで一致させる。

## カスタムARIAラジオグループ

ネイティブのラジオボタンで実現できない要件がある場合、`radiogroup`、`radio`、`aria-checked`、ロービング`tabindex`を組み合わせる。

```html
<p id="custom-shipping-label">配送方法</p>

<div
  id="custom-shipping"
  role="radiogroup"
  aria-labelledby="custom-shipping-label"
>
  <div
    role="radio"
    aria-checked="true"
    tabindex="0"
    data-value="standard"
  >
    通常配送
  </div>
  <div
    role="radio"
    aria-checked="false"
    tabindex="-1"
    data-value="express"
  >
    速達
  </div>
  <div
    role="radio"
    aria-checked="false"
    tabindex="-1"
    data-value="store"
  >
    店舗受け取り
  </div>
</div>

<input
  id="custom-shipping-value"
  type="hidden"
  name="shipping"
  value="standard"
>
```

```js
class CustomRadioGroup {
  constructor(group, hiddenInput) {
    this.group = group;
    this.hiddenInput = hiddenInput;
    this.radios = [
      ...group.querySelectorAll('[role="radio"]'),
    ];

    this.group.addEventListener('click', (event) => {
      const radio = event.target.closest('[role="radio"]');
      if (!radio || !this.group.contains(radio)) return;
      if (radio.getAttribute('aria-disabled') === 'true') return;

      this.check(radio, { moveFocus: true });
    });

    this.group.addEventListener('keydown', (event) => {
      this.handleKeyDown(event);
    });
  }

  get enabledRadios() {
    return this.radios.filter(
      (radio) => radio.getAttribute('aria-disabled') !== 'true'
    );
  }

  check(radio, { moveFocus = false } = {}) {
    this.radios.forEach((item) => {
      const isChecked = item === radio;
      item.setAttribute('aria-checked', String(isChecked));
      item.tabIndex = isChecked ? 0 : -1;
    });

    this.hiddenInput.value = radio.dataset.value;
    this.hiddenInput.dispatchEvent(
      new Event('change', { bubbles: true })
    );

    if (moveFocus) radio.focus();
  }

  handleKeyDown(event) {
    const current = event.target.closest('[role="radio"]');
    if (!current) return;

    if (event.key === ' ') {
      event.preventDefault();
      if (current.getAttribute('aria-disabled') !== 'true') {
        this.check(current);
      }
      return;
    }

    const forward = event.key === 'ArrowRight' || event.key === 'ArrowDown';
    const backward = event.key === 'ArrowLeft' || event.key === 'ArrowUp';
    if (!forward && !backward) return;

    const radios = this.enabledRadios;
    const currentIndex = radios.indexOf(current);
    if (currentIndex === -1 || radios.length === 0) return;

    event.preventDefault();
    const offset = forward ? 1 : -1;
    const nextIndex = (currentIndex + offset + radios.length) % radios.length;
    this.check(radios[nextIndex], { moveFocus: true });
  }
}

const customGroup = document.getElementById('custom-shipping');
const customValue = document.getElementById('custom-shipping-value');

new CustomRadioGroup(customGroup, customValue);
```

カスタム実装では、次をすべて自分で維持する必要がある。

- グループに一つだけ`aria-checked="true"`がある
- 選択項目だけが`tabindex="0"`になる
- 矢印キーでフォーカスと選択が移る
- Spaceキーでフォーカス中の項目を選べる
- クリック・タップ時も`tabindex`が同期する
- 無効項目を操作できない
- フォーム値が選択状態と一致する

`role="radio"`を付けても、キーボード操作や相互排他の動作は追加されない。特別な要件がなければネイティブ入力を使う。

### ツールバー内の例外

APGでは、ラジオグループがツールバー内にある場合、矢印キーでツールバーを移動しただけでは選択を変更せず、SpaceまたはEnterで確定する操作モデルを示している。

通常のフォーム用ラジオグループとは異なる高度なパターン。エディターなどで本当にツールバーが必要な場合だけ適用し、一般的なフォームへ持ち込まない。

## 動的に選択肢を変更する場合

在庫、地域、以前の回答によって選択肢を追加・削除・無効化することがある。

- 現在選択中の項目を削除する前に、次の状態を決める
- 勝手に別の有料項目を選択しない
- 選択が解除されたことを必要に応じて通知する
- フォーカス中の項目を削除する場合、論理的な次の項目へ移す
- すべての選択肢がなくなった場合、理由と次の操作を伝える
- DOM順序と視覚順序を一致させる

別の回答によって新しいグループを表示する場合は、DOM上でトリガーの後に置き、Tab順序が自然に続くようにする。自動的なフォーカス移動が本当に必要かは、操作の文脈から判断する。

## よくある失敗

### `fieldset`と`legend`がない

各項目名は読めても、「何について選ぶのか」が分からない。グループ全体の質問をプログラム上も関連付ける。

### 選択肢ごとに異なる`name`を付ける

複数項目を同時に選べてしまう。同じ質問には同じ`name`を使う。

### ラベルが円と関連付いていない

文字を押しても選択できず、アクセシブルネームも不足する。`for`と`id`を一致させる。

### ネイティブ入力へ独自の矢印キー処理を追加する

ブラウザの動作と競合する。ネイティブ入力では標準操作をそのまま使う。

### カスタム実装の全項目がTab順序に入る

Tabキーで一項目ずつ通過させず、選択項目だけを`tabindex="0"`にして矢印キーで移動する。

### カスタム実装でフォーカスと選択が別々に動く

通常のフォーム用ラジオグループでは、矢印キーで移動した項目を選択状態にする。ツールバー内の例外と混同しない。

### 選択済みの項目をクリックすると未選択になる

ラジオボタンは一つを選ぶUI。一項目を再度押して解除する独自動作を追加しない。未選択を許すなら「希望なし」などの項目を用意する。

### `display: none`で入力を隠す

入力がTab順序とアクセシビリティツリーから外れる。視覚的に隠す場合も操作可能な状態を維持する。

### エラーを各項目へ繰り返す

同じエラーを選択肢ごとに表示すると冗長になる。グループに一つのエラーを置き、質問と関連付ける。

### 選択状態を色だけで示す

円の塗り、境界、アイコン、文字など、色以外の手がかりを使う。

## テストのチェックリスト

### 構造と名前

- [ ] 関連する選択肢が`fieldset`でグループ化されている
- [ ] `legend`がグループ全体の短い質問になっている
- [ ] 各入力に固有の`id`と関連付いた`label`がある
- [ ] 同じグループの`name`が一致している
- [ ] 別の質問へ同じ`name`を再利用していない
- [ ] 各ラベルを単独で聞いても選択肢を区別できる

### キーボードとフォーカス

- [ ] Tab / Shift + Tabでグループへ出入りできる
- [ ] 選択済み項目があれば、そこへTabフォーカスが入る
- [ ] Spaceキーでフォーカス中の項目を選べる
- [ ] 左右・上下矢印で前後の項目へ移動して選択できる
- [ ] 端から矢印キーで移動した場合の動作が一貫している
- [ ] フォーカス表示が常に見える
- [ ] ラベル全体をクリック・タップして選択できる

### 状態と表示

- [ ] 一つのグループで複数項目が同時に選択されない
- [ ] 見た目とcheckedまたは`aria-checked`が一致する
- [ ] 選択状態を色だけで区別していない
- [ ] 無効な項目を選択できず、理由を理解できる
- [ ] 200%・400%拡大でも質問、選択肢、説明を利用できる
- [ ] 強制カラーモードでも選択状態とフォーカスが分かる

### 必須、エラー、送信

- [ ] 初期選択が利用者に不利益な既定値になっていない
- [ ] 必須で未選択の場合、グループ単位でエラーが表示される
- [ ] エラーが質問とプログラム上も関連付いている
- [ ] エラー後、最初の項目へフォーカスして修正できる
- [ ] 選択後にエラー状態が解除される
- [ ] 送信される`name`と`value`が業務上の値と一致する
- [ ] サーバー側でも未選択と不正な値を検証する
- [ ] マウス、タッチ、キーボード、音声操作、スクリーンリーダーで確認した

## 関連して読むとよいテーマ

- [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [ロービングタブインデックスパターン](./roving-tabindex.md)
- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)
- [ARIAロールの基本 — role属性の種類と使い方](./aria-roles.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [フォーカスインジケーターのカスタマイズ — :focus-visibleの活用](./focus-visible.md)

## まとめ

- ラジオグループは、複数の候補から一つだけを選ぶ場合に使う
- ネイティブの`input type="radio"`、`fieldset`、`legend`、`label`を基本にする
- 同じ質問のラジオボタンへ同じ`name`を付け、別の質問とは分ける
- Tabキーでグループへ入り、矢印キーでフォーカスと選択を移動する
- ネイティブ入力には独自の矢印キー処理や`tabindex`管理を追加しない
- 初期選択は安全な既定値がある場合だけ設定し、明示的な回答が必要なら未選択から始める
- 必須エラーはグループ単位で示し、質問と関連付ける
- カスタム実装では`radiogroup`、`radio`、`aria-checked`、ロービング`tabindex`、フォーム値を同期する
- 選択状態を色だけに頼らず、拡大、強制カラー、キーボード、支援技術でテストする
