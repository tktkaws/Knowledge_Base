# アクセシブルなフォームの設計 — エラー表示とバリデーション

## アクセシブルなフォームの基本原則

- 全ての入力欄にラベルを紐付ける
- エラーメッセージを支援技術に伝える
- キーボードのみで全操作を完了できるようにする
- 入力のヒントや制約を事前に伝える

## ラベルの紐付け

### label 要素による紐付け

- `<label>` の `for` 属性と `<input>` の `id` を一致させる
- ラベルをクリックすると対応する入力欄にフォーカスが当たる
- スクリーンリーダーが入力欄のフォーカス時にラベルを読み上げる

```html
<!-- 良い例：for / id で紐付け -->
<label for="email">メールアドレス</label>
<input id="email" type="email" />

<!-- 良い例：labelで囲む -->
<label>
  メールアドレス
  <input type="email" />
</label>

<!-- 悪い例：ラベルなし -->
<input type="email" placeholder="メールアドレス" />
```

### placeholder はラベルの代替にならない

- `placeholder` は入力を開始すると消えるため、ラベルとしては不十分
- スクリーンリーダーによっては `placeholder` を読み上げない場合がある
- ラベルは常に表示される `<label>` 要素で提供する

```html
<!-- 悪い例：placeholderのみでラベルなし -->
<input type="text" placeholder="氏名" />

<!-- 良い例：ラベル + placeholder -->
<label for="name">氏名</label>
<input id="name" type="text" placeholder="例: 山田太郎" />
```

> 参照: [WCAG 3.3.2 Labels or Instructions](https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html)

### aria-label / aria-labelledby の使用

- 視覚的にラベルが表示できないデザインの場合、`aria-label` で非視覚的ラベルを付ける
- ただし視覚的なラベルがある方が全ユーザーにとって望ましい

```html
<!-- 検索欄：アイコンのみでラベルがない場合 -->
<input type="search" aria-label="サイト内検索" />

<!-- 複数のラベル要素を結合する場合 -->
<span id="billing">請求先</span>
<span id="address-label">住所</span>
<input aria-labelledby="billing address-label" />
```

## 入力の説明とヒント

### aria-describedby で補足説明を紐付け

- パスワード要件やフォーマットの説明を入力欄に紐付ける
- スクリーンリーダーはラベルを読み上げた後に補足説明を読み上げる

```html
<label for="password">パスワード</label>
<input
  id="password"
  type="password"
  aria-describedby="password-hint"
/>
<p id="password-hint">8文字以上、大文字・小文字・数字をそれぞれ1文字以上含む</p>
```

### 必須フィールドの示し方

- `required` 属性を付けるとスクリーンリーダーが「必須」と読み上げる
- `aria-required="true"` でも同様の効果
- 視覚的にも必須であることを示す（アスタリスクなど）

```html
<!-- required属性 + 視覚的な必須表示 -->
<label for="name">
  氏名 <span aria-hidden="true">*</span>
</label>
<input id="name" type="text" required />

<!-- フォーム冒頭に注記を付ける -->
<p>
  <span aria-hidden="true">*</span> は必須項目です
</p>
```

> 参照: [WCAG 3.3.2 Labels or Instructions](https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html)

## グループ化

### fieldset と legend

- 関連するフォーム要素をグループ化する
- ラジオボタン群やチェックボックス群に特に重要
- スクリーンリーダーは各項目のフォーカス時に `legend` のテキストも読み上げる

```html
<!-- 良い例：fieldset/legendでグループ化 -->
<fieldset>
  <legend>配送方法</legend>
  <label>
    <input type="radio" name="shipping" value="standard" /> 通常配送
  </label>
  <label>
    <input type="radio" name="shipping" value="express" /> 速達
  </label>
  <label>
    <input type="radio" name="shipping" value="same-day" /> 当日配送
  </label>
</fieldset>

<!-- 悪い例：グループ化なし（「通常配送」が何の選択肢か分からない） -->
<p>配送方法</p>
<label>
  <input type="radio" name="shipping" value="standard" /> 通常配送
</label>
```

### role="group" による代替

- `<fieldset>` が使いにくいレイアウトの場合、`role="group"` と `aria-labelledby` で代替

```html
<div role="group" aria-labelledby="payment-heading">
  <h3 id="payment-heading">支払い方法</h3>
  <label><input type="radio" name="pay" value="card" /> クレジットカード</label>
  <label><input type="radio" name="pay" value="bank" /> 銀行振込</label>
</div>
```

## エラー表示

### エラーを伝えるための3つの手段

1. **視覚的な表示** — エラーメッセージのテキスト、赤色のボーダーなど
2. **プログラム的な紐付け** — `aria-describedby` や `aria-errormessage` で入力欄とエラーメッセージを紐付け
3. **状態の通知** — `aria-invalid="true"` でエラー状態であることを支援技術に伝える

### aria-invalid

- 入力値が無効であることを示す属性
- スクリーンリーダーが「無効」と読み上げる

```html
<!-- エラーなし -->
<input id="email" type="email" />

<!-- エラーあり -->
<input id="email" type="email" aria-invalid="true" aria-describedby="email-error" />
<p id="email-error" class="error">有効なメールアドレスを入力してください</p>
```

### aria-errormessage（ARIA 1.2）

- `aria-describedby` の代替として、エラーメッセージ専用の紐付け属性
- `aria-invalid="true"` と併用する
- ブラウザ / スクリーンリーダーのサポートが十分でないため、`aria-describedby` を併用するのが安全

```html
<input
  id="email"
  type="email"
  aria-invalid="true"
  aria-errormessage="email-error"
  aria-describedby="email-error"
/>
<p id="email-error" class="error">有効なメールアドレスを入力してください</p>
```

### エラーメッセージの内容

- 何が間違っているかを具体的に伝える
- どう修正すればよいかを示す

```html
<!-- 悪い例：曖昧なエラー -->
<p class="error">入力エラーです</p>

<!-- 良い例：具体的なエラー -->
<p class="error">メールアドレスに「@」が含まれていません</p>

<!-- 良い例：修正方法を示す -->
<p class="error">パスワードは8文字以上で入力してください（現在5文字）</p>
```

> 参照: [WCAG 3.3.1 Error Identification](https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html)

## バリデーションのタイミング

### リアルタイムバリデーション（入力中）

- ユーザーの入力途中にエラーを表示するとストレスになる場合がある
- 使うならポジティブなフィードバック（入力が正しいことの確認）に限定する

### フォーカスアウト時（blur）のバリデーション

- フィールドからフォーカスが外れた時点で検証する
- ユーザーが入力を完了した後にエラーを表示するため、自然な体験
- `aria-live="polite"` でエラーの追加をスクリーンリーダーに通知

```js
const emailInput = document.getElementById('email');
const emailError = document.getElementById('email-error');

emailInput.addEventListener('blur', () => {
  if (!emailInput.validity.valid) {
    emailInput.setAttribute('aria-invalid', 'true');
    emailError.textContent = 'メールアドレスの形式が正しくありません';
    emailError.hidden = false;
  } else {
    emailInput.removeAttribute('aria-invalid');
    emailError.textContent = '';
    emailError.hidden = true;
  }
});
```

### 送信時のバリデーション

- フォーム送信時に全フィールドを一括検証する
- エラーがある場合、エラーサマリーを表示して最初のエラーフィールドにフォーカスを移動

```js
form.addEventListener('submit', (event) => {
  const errors = validateForm();

  if (errors.length > 0) {
    event.preventDefault();

    // エラーサマリーを表示
    showErrorSummary(errors);

    // 最初のエラーフィールドにフォーカス
    errors[0].field.focus();
  }
});
```

## エラーサマリー

- フォーム送信時にエラーが複数ある場合、ページ上部にエラー一覧を表示する
- 各エラーメッセージから該当フィールドへのリンクを提供する
- エラーサマリーにフォーカスを移動させ、スクリーンリーダーに通知する

```html
<div id="error-summary" role="alert" tabindex="-1" hidden>
  <h2>入力エラーがあります</h2>
  <ul>
    <li><a href="#name">氏名を入力してください</a></li>
    <li><a href="#email">メールアドレスの形式が正しくありません</a></li>
  </ul>
</div>

<form>
  <label for="name">氏名</label>
  <input id="name" type="text" required aria-invalid="true" aria-describedby="name-error" />
  <p id="name-error" class="error">氏名を入力してください</p>

  <label for="email">メールアドレス</label>
  <input id="email" type="email" required aria-invalid="true" aria-describedby="email-error" />
  <p id="email-error" class="error">メールアドレスの形式が正しくありません</p>
</form>
```

```js
function showErrorSummary(errors) {
  const summary = document.getElementById('error-summary');
  const list = summary.querySelector('ul');

  list.innerHTML = errors
    .map((e) => `<li><a href="#${e.fieldId}">${e.message}</a></li>`)
    .join('');

  summary.hidden = false;
  summary.focus(); // サマリーにフォーカスを移動
}
```

> 参照: [WCAG 3.3.3 Error Suggestion](https://www.w3.org/WAI/WCAG22/Understanding/error-suggestion.html)

## HTMLネイティブバリデーション

- `required`, `type`, `pattern`, `min`, `max`, `minlength`, `maxlength` 等のHTML属性でブラウザネイティブのバリデーションが可能
- ブラウザが自動でエラーメッセージを表示する
- カスタマイズ性に限界があるため、独自UIと併用する場合は `novalidate` を使用

```html
<!-- ネイティブバリデーションの例 -->
<form>
  <label for="age">年齢</label>
  <input id="age" type="number" min="0" max="150" required />

  <label for="zip">郵便番号</label>
  <input id="zip" type="text" pattern="\d{3}-?\d{4}" required />

  <button type="submit">送信</button>
</form>

<!-- カスタムバリデーションUIを使う場合 -->
<form novalidate>
  <!-- JSで独自のバリデーションとエラー表示を実装 -->
</form>
```

## React での実装例

```jsx
import { useState, useRef } from 'react';

function ContactForm() {
  const [errors, setErrors] = useState({});
  const [submitted, setSubmitted] = useState(false);
  const errorSummaryRef = useRef(null);

  const validate = (values) => {
    const newErrors = {};
    if (!values.name) newErrors.name = '氏名を入力してください';
    if (!values.email) {
      newErrors.email = 'メールアドレスを入力してください';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      newErrors.email = 'メールアドレスの形式が正しくありません';
    }
    return newErrors;
  };

  const handleSubmit = (event) => {
    event.preventDefault();
    const formData = new FormData(event.target);
    const values = Object.fromEntries(formData);
    const newErrors = validate(values);

    setErrors(newErrors);
    setSubmitted(true);

    if (Object.keys(newErrors).length > 0) {
      // エラーサマリーにフォーカス
      errorSummaryRef.current?.focus();
    } else {
      submitForm(values);
    }
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      {/* エラーサマリー */}
      {submitted && Object.keys(errors).length > 0 && (
        <div ref={errorSummaryRef} role="alert" tabIndex={-1}>
          <h2>入力エラーがあります</h2>
          <ul>
            {Object.entries(errors).map(([field, message]) => (
              <li key={field}>
                <a href={`#${field}`}>{message}</a>
              </li>
            ))}
          </ul>
        </div>
      )}

      {/* 氏名フィールド */}
      <div>
        <label htmlFor="name">
          氏名 <span aria-hidden="true">*</span>
        </label>
        <input
          id="name"
          name="name"
          type="text"
          required
          aria-invalid={errors.name ? 'true' : undefined}
          aria-describedby={errors.name ? 'name-error' : undefined}
        />
        {errors.name && (
          <p id="name-error" className="error">{errors.name}</p>
        )}
      </div>

      {/* メールフィールド */}
      <div>
        <label htmlFor="email">
          メールアドレス <span aria-hidden="true">*</span>
        </label>
        <input
          id="email"
          name="email"
          type="email"
          required
          aria-invalid={errors.email ? 'true' : undefined}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <p id="email-error" className="error">{errors.email}</p>
        )}
      </div>

      <button type="submit">送信</button>
    </form>
  );
}
```

## よくある間違い

### 1. label を紐付けない

```html
<!-- 間違い：入力欄の横にテキストがあるだけ -->
<span>メールアドレス</span>
<input type="email" />

<!-- 正解：label で紐付け -->
<label for="email">メールアドレス</label>
<input id="email" type="email" />
```

### 2. 色だけでエラーを伝える

```html
<!-- 間違い：赤いボーダーだけ（色覚異常の方に伝わらない） -->
<input style="border-color: red;" />

<!-- 正解：テキストメッセージ + アイコン + 色 -->
<input aria-invalid="true" aria-describedby="err" />
<p id="err" class="error">⚠ メールアドレスを入力してください</p>
```

> 参照: [WCAG 1.4.1 Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html)

### 3. aria-invalid を常に付けっぱなし

```html
<!-- 間違い：初期状態でエラーと読み上げられる -->
<input aria-invalid="true" />

<!-- 正解：バリデーション後にのみ付与 -->
<input />
<!-- バリデーション失敗後 -->
<input aria-invalid="true" aria-describedby="error" />
```

### 4. エラー後のフォーカス管理がない

- エラーがあるのに画面の一番上にスクロールだけして、フォーカスはどこにも移動しない
- スクリーンリーダーユーザーはエラーに気づけない
- エラーサマリーまたは最初のエラーフィールドにフォーカスを移動する

## まとめ

- 全ての入力欄に `<label>` を紐付ける — `placeholder` はラベルの代替にならない
- `fieldset` / `legend` で関連する入力をグループ化する
- `aria-invalid="true"` + `aria-describedby` でエラーを支援技術に伝える
- エラーメッセージは具体的に — 何が問題で、どう修正するかを示す
- 送信時にエラーサマリーを表示し、フォーカスをエラー箇所に移動する
- 色だけでなくテキストでもエラーを伝える
