# アクセシブルなウィザードの実装 — 複数ステップの進行とエラー

## ウィザードとは

ウィザード（Wizard）は、一つの長い手続きを複数のステップへ分け、順番に入力や確認を進めるUI。会員登録、購入手続き、各種申請、初期設定などに使われる。

- 手続きを意味のある単位へ分ける
- 現在のステップと全体の長さを伝える
- 前のステップへ戻っても入力を失わない
- エラーがあるステップから先へ進ませない
- 最終確定の前に入力内容を確認できるようにする
- ページ遷移後のタイトル、見出し、フォーカスを同期する

専用の`wizard`要素や`role="wizard"`は存在しない。通常の見出し、フォーム、リスト、リンク、ボタンを組み合わせて構成する。

> 参照:
> - [W3C WAI — Multi-page Forms](https://www.w3.org/WAI/tutorials/forms/multi-page/)
> - [WAI-ARIA 1.2 — `aria-current`](https://www.w3.org/TR/wai-aria-1.2/#aria-current)

## ウィザードが向く場面

### 長いフォームを論理的に分けられる

```text
1. アカウント情報
2. 配送先
3. 支払い方法
4. 内容確認
```

入力項目を目的ごとに分けることで、一度に把握する量を減らせる。最初の画面で必要な情報や所要時間の目安を伝えると、利用者が手続きを始めるか判断しやすい。

### 後の質問が前の回答で変わる

個人と法人、配送の有無、申請の種類などによって必要なステップが変わる場合にも使える。ただし、途中で総ステップ数が変わるなら「全5ステップ」と断定せず、現在位置や残りの内容を正確に更新する。

## ウィザードを使わない方がよい場面

- 項目が少なく、一画面の方が全体を把握しやすい
- 任意の順序で編集する必要がある
- 前後の項目を見比べながら入力する必要がある
- ステップを分けることで、確認や修正の移動が増えるだけになる

単に画面を短く見せるためだけに分割しない。各ステップに明確な目的があるかを確認する。

## 基本構造

複数ページで構成する場合、各ページへ次の情報を置く。

```html
<!doctype html>
<html lang="ja">
  <head>
    <title>ステップ2 / 4: 配送先 — 注文手続き</title>
  </head>
  <body>
    <main>
      <nav aria-label="注文手続きの進捗">
        <ol class="stepper">
          <li>
            <a href="/checkout/account">
              <span class="visually-hidden">完了: </span>
              アカウント
            </a>
          </li>
          <li>
            <span aria-current="step">配送先</span>
          </li>
          <li><span>支払い</span></li>
          <li><span>内容確認</span></li>
        </ol>
      </nav>

      <h1>配送先（ステップ2 / 4）</h1>

      <form action="/checkout/shipping" method="post">
        <!-- 配送先の入力項目 -->
      </form>
    </main>
  </body>
</html>
```

ページタイトルでは進捗を先に書く。スクリーンリーダー利用者やブラウザのタブ一覧からも、現在位置を早く把握できる。

メイン見出しにもステップ名と位置を含める。視覚的にページを走査する利用者へも同じ情報を示せる。

## ステップ表示を作る

### 順序付きリストを使う

ステップには順序があるため`ol`を使う。見た目を横並びにするためだけにリストの意味を失わせない。

```html
<nav aria-label="申込みの進捗">
  <ol>
    <li><a href="/apply/profile">基本情報</a></li>
    <li><span aria-current="step">連絡先</span></li>
    <li><span>内容確認</span></li>
    <li><span>完了</span></li>
  </ol>
</nav>
```

`nav`の名前は「進捗」だけでなく、「申込みの進捗」「注文手続きの進捗」のように対象を含める。

### 現在位置は`aria-current="step"`

```html
<span aria-current="step">連絡先</span>
```

一つのステップ一覧で`aria-current="step"`を付ける項目は一つだけにする。`aria-selected`や`aria-pressed`で代用しない。

現在のステップがリンクである必要がなければ、通常のテキストでよい。再読み込みできるURLを持たせたい場合はリンクにも`aria-current="step"`を付けられる。

### 完了済みのステップだけをリンクにする

```html
<li>
  <a href="/apply/profile">
    <span class="visually-hidden">完了: </span>
    基本情報
  </a>
</li>
```

過去のステップへ戻れる場合は実際のリンクを使う。戻ったときに入力済みデータを復元できることが前提。

まだ到達していないステップへ移動できないなら、リンクや無効なボタンにしない。テキストとして示せばよい。

```html
<!-- 避ける：操作できない要素をリンクのように見せる -->
<a aria-disabled="true">支払い</a>

<!-- 適切：まだ移動できないステップはテキスト -->
<span>支払い</span>
```

### ステップ表示をタブにしない

ウィザードは通常、入力検証や保存を伴う順序付きの手続き。コンテンツ表示を切り替えるタブとは目的が異なる。

- `role="tablist"`、`role="tab"`、`role="tabpanel"`を付けない
- 左右矢印キーによるタブ操作を追加しない
- 到達していないステップを自由に開けるようにしない
- 「次へ」「戻る」を通常のリンクまたはボタンで提供する

## `progress`を併用する場合

ステップ一覧に加えて、完了割合を簡潔に示したい場合は`progress`を使える。

```html
<label for="application-progress">申込みの進捗</label>
<progress id="application-progress" value="2" max="4">
  2 / 4
</progress>
<span>ステップ2 / 4</span>
```

`progress`だけでは各ステップの名前や、戻って編集できる場所は分からない。既知の段階を案内する主な手段としては、名前のある順序付きリストと見出しを使う。

分岐によって総数がまだ決まらない場合、誤った最大値を表示しない。「現在: 配送先」のような位置情報を優先する。

## 各ステップを通常のフォームとして作る

```html
<h1>連絡先（ステップ2 / 4）</h1>

<p><span aria-hidden="true">*</span> は必須項目です。</p>

<form action="/apply/contact" method="post">
  <div class="field">
    <label for="email">
      メールアドレス <span aria-hidden="true">*</span>
    </label>
    <input
      id="email"
      name="email"
      type="email"
      autocomplete="email"
      required
    />
  </div>

  <div class="wizard-actions">
    <button
      type="submit"
      name="intent"
      value="back"
      formnovalidate
    >
      戻る
    </button>
    <button type="submit" name="intent" value="next">
      次へ
    </button>
  </div>
</form>
```

- 入力欄へ永続的な`label`を付ける
- 必須、形式、文字数などの条件を入力前に示す
- 関連する選択肢は`fieldset`と`legend`でまとめる
- `autocomplete`を適切に指定する
- ステップごとに送信して、サーバー側でも検証する

一つの画面に複数の`form`を入れ子にしない。HTMLではフォームの入れ子はできない。

## 「次へ」と「戻る」の意味

### 「次へ」

現在のステップを検証して保存する操作なら、`button type="submit"`を使う。

```html
<button type="submit" name="intent" value="next">次へ</button>
```

ボタン名は処理に合わせて具体的にできる。

```html
<button type="submit">配送先を保存して支払いへ</button>
```

### 「戻る」

戻るために検証を要求しない。未完成の入力があっても前の画面へ戻れるようにする。

```html
<button
  type="submit"
  name="intent"
  value="back"
  formnovalidate
>
  戻る
</button>
```

この例では、サーバーが`intent=back`を先に判定し、現在の入力を一時保存して前のステップへ移動する。

入力を保存する必要がなく、前のページへ安全に戻れるならリンクも使える。

```html
<a href="/apply/profile">基本情報へ戻る</a>
```

JavaScriptの`history.back()`だけに依存すると、外部ページへ戻る、履歴がない、再読み込み後に意図しない場所へ戻る可能性がある。手続き内の移動先をURLで示す方が予測しやすい。

## 現在のステップだけを検証する

ステップ2にいる利用者へ、まだ見ていないステップ3のエラーを表示しない。現在のステップを通過するときに、そのステップの項目を検証する。

```text
ステップ1を入力
  ↓ 「次へ」
ステップ1を検証
  ├─ エラーあり → ステップ1に残して修正箇所を示す
  └─ エラーなし → 保存してステップ2へ
```

クライアント側の検証だけに依存せず、サーバー側でも同じ制約を確認する。サーバーからエラーを返す場合も、入力済みの値を消さない。

## エラーサマリーと個別エラー

複数のエラーがある場合は、ステップの先頭にサマリーを置き、各入力の近くにも具体的なエラーを示す。

```html
<div
  id="error-summary"
  class="error-summary"
  tabindex="-1"
  aria-labelledby="error-summary-title"
  hidden
>
  <h2 id="error-summary-title">入力内容を確認してください</h2>
  <ul id="error-summary-list"></ul>
</div>

<form id="contact-form" action="/apply/contact" method="post" novalidate>
  <div class="field">
    <label for="postal-code">郵便番号</label>
    <p id="postal-code-hint">例: 123-4567</p>
    <input
      id="postal-code"
      name="postalCode"
      inputmode="numeric"
      autocomplete="postal-code"
      aria-describedby="postal-code-hint postal-code-error"
      required
    />
    <p id="postal-code-error" class="field-error" hidden></p>
  </div>

  <div class="field">
    <label for="contact-email">メールアドレス</label>
    <input
      id="contact-email"
      name="email"
      type="email"
      autocomplete="email"
      aria-describedby="contact-email-error"
      required
    />
    <p id="contact-email-error" class="field-error" hidden></p>
  </div>

  <button type="submit">次へ</button>
</form>
```

```js
const form = document.getElementById('contact-form');
const errorSummary = document.getElementById('error-summary');
const errorList = document.getElementById('error-summary-list');

const rules = [
  {
    field: document.getElementById('postal-code'),
    error: document.getElementById('postal-code-error'),
    message: '郵便番号を7桁で入力してください。',
    isValid(field) {
      return /^\d{3}-?\d{4}$/.test(field.value.trim());
    },
  },
  {
    field: document.getElementById('contact-email'),
    error: document.getElementById('contact-email-error'),
    message: '有効なメールアドレスを入力してください。',
    isValid(field) {
      return field.value.trim() !== '' && !field.validity.typeMismatch;
    },
  },
];

function clearError({ field, error }) {
  field.removeAttribute('aria-invalid');
  error.textContent = '';
  error.hidden = true;
}

function showError({ field, error, message }) {
  field.setAttribute('aria-invalid', 'true');
  error.textContent = message;
  error.hidden = false;
}

function addSummaryLink({ field, message }) {
  const item = document.createElement('li');
  const link = document.createElement('a');

  link.href = `#${field.id}`;
  link.textContent = message;
  item.append(link);
  errorList.append(item);
}

form.addEventListener('submit', (event) => {
  const invalidRules = rules.filter((rule) => !rule.isValid(rule.field));

  rules.forEach(clearError);
  errorList.replaceChildren();

  if (invalidRules.length === 0) {
    errorSummary.hidden = true;
    return;
  }

  event.preventDefault();

  invalidRules.forEach((rule) => {
    showError(rule);
    addSummaryLink(rule);
  });

  errorSummary.hidden = false;
  errorSummary.focus();
});

errorList.addEventListener('click', (event) => {
  const link = event.target.closest('a');
  if (!link) return;

  const field = document.querySelector(link.getAttribute('href'));
  if (!field) return;

  event.preventDefault();
  field.focus();
});
```

エラーサマリーへ`tabindex="-1"`を付けると、通常のTab順序には追加せず、送信失敗時だけフォーカスを移せる。フォーカスによって見出しとエラー一覧が読み上げられるため、この例では`role="alert"`を重ねていない。

個別エラーでは次をそろえる。

- 入力へ`aria-invalid="true"`を付ける
- エラーテキストを`aria-describedby`で入力と関連付ける
- 何が誤りで、どう直すかを文章で示す
- 色やアイコンだけでエラーを示さない
- 修正後は`aria-invalid`と古いエラー文を取り除く

> 参照: [Understanding WCAG 2.2 SC 3.3.1 — Error Identification](https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html)

## ステップ遷移後のフォーカス

通常のページ遷移では、ブラウザが新しい文書の先頭から読み始める。ページタイトルと`h1`に現在位置を含めることで、新しいステップを把握しやすくなる。

SPAで同じページの一部だけを切り替える場合は、押した「次へ」ボタンを非表示にする前に、表示したステップの見出しへフォーカスを移す。

```html
<section
  id="step-shipping"
  data-wizard-step
  data-title="配送先"
  aria-labelledby="step-shipping-title"
  hidden
>
  <h1 id="step-shipping-title" tabindex="-1">
    配送先（ステップ2 / 4）
  </h1>
  <!-- 入力項目 -->
</section>
```

```js
function focusStepHeading(step) {
  const heading = step.querySelector('h1');

  requestAnimationFrame(() => {
    heading?.focus();
  });
}
```

見出しの`tabindex="-1"`はプログラムからのフォーカスだけを可能にする。見出しを通常のTab停止位置にはしない。

- 正常に次へ進んだら、新しいステップの見出しへ移す
- 戻った場合も、戻り先の見出しへ移す
- 検証に失敗したら、エラーサマリーまたは最初のエラーへ移す
- フォーカスを`body`や非表示要素に残さない
- 正の`tabindex`でフォーカス順序を作らない

> 参照: [Understanding WCAG 2.2 SC 2.4.3 — Focus Order](https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html)

## SPAでステップを切り替える

```html
<nav aria-label="申込みの進捗">
  <ol id="wizard-steps">
    <li><span data-step-indicator aria-current="step">基本情報</span></li>
    <li><span data-step-indicator>配送先</span></li>
    <li><span data-step-indicator>内容確認</span></li>
  </ol>
</nav>

<form id="application-wizard">
  <section data-wizard-step data-title="基本情報">
    <h1 tabindex="-1">基本情報（ステップ1 / 3）</h1>
    <label for="full-name">氏名</label>
    <input id="full-name" name="fullName" required />
    <button type="button" data-next>次へ</button>
  </section>

  <section data-wizard-step data-title="配送先" hidden>
    <h1 tabindex="-1">配送先（ステップ2 / 3）</h1>
    <label for="address">住所</label>
    <input id="address" name="address" required />
    <button type="button" data-back>戻る</button>
    <button type="button" data-next>次へ</button>
  </section>

  <section data-wizard-step data-title="内容確認" hidden>
    <h1 tabindex="-1">内容確認（ステップ3 / 3）</h1>
    <!-- 入力内容の確認表示 -->
    <button type="button" data-back>戻る</button>
    <button type="submit">申込みを確定する</button>
  </section>
</form>
```

```js
const wizard = document.getElementById('application-wizard');
const panels = [...wizard.querySelectorAll('[data-wizard-step]')];
const indicators = [...document.querySelectorAll('[data-step-indicator]')];
let currentIndex = 0;

function validateCurrentStep() {
  const fields = [
    ...panels[currentIndex].querySelectorAll('input, select, textarea'),
  ];
  const firstInvalid = fields.find((field) => !field.checkValidity());

  if (!firstInvalid) return true;

  firstInvalid.setAttribute('aria-invalid', 'true');
  firstInvalid.focus();
  firstInvalid.reportValidity();
  return false;
}

wizard.addEventListener('input', (event) => {
  const field = event.target;
  if (!field.matches('input, select, textarea')) return;
  if (field.checkValidity()) field.removeAttribute('aria-invalid');
});

function updateUrl(index, method = 'pushState') {
  const url = new URL(window.location.href);
  url.searchParams.set('step', String(index + 1));
  history[method]({ step: index }, '', url);
}

function showStep(index, { updateHistory = true, moveFocus = true } = {}) {
  if (!Number.isInteger(index) || index < 0 || index >= panels.length) return;

  panels.forEach((panel, panelIndex) => {
    panel.hidden = panelIndex !== index;
  });

  indicators.forEach((indicator, indicatorIndex) => {
    if (indicatorIndex === index) {
      indicator.setAttribute('aria-current', 'step');
    } else {
      indicator.removeAttribute('aria-current');
    }
  });

  currentIndex = index;
  const currentPanel = panels[currentIndex];
  document.title = `ステップ${currentIndex + 1} / ${panels.length}: ${currentPanel.dataset.title} — 申込み`;

  if (updateHistory) updateUrl(currentIndex);
  if (moveFocus) focusStepHeading(currentPanel);
}

wizard.addEventListener('click', (event) => {
  const nextButton = event.target.closest('[data-next]');
  const backButton = event.target.closest('[data-back]');

  if (nextButton) {
    if (!validateCurrentStep()) return;
    showStep(currentIndex + 1);
  }

  if (backButton) {
    showStep(currentIndex - 1);
  }
});

window.addEventListener('popstate', (event) => {
  const index = Number.isInteger(event.state?.step) ? event.state.step : 0;
  showStep(index, { updateHistory: false });
});

updateUrl(0, 'replaceState');
showStep(0, { updateHistory: false, moveFocus: false });
```

この簡略例は遷移と履歴管理を示している。実運用では、前節のような見える個別エラーとエラーサマリーを組み合わせる。

SPAでもURLとブラウザ履歴を更新すると、戻る・進む操作と表示中のステップを同期できる。URLから直接開いた場合に、そのステップへ移動してよいか、必要なデータが保存済みかもサーバー側で確認する。

## 入力内容を失わせない

前のステップへ戻ったとき、検証に失敗したとき、セッションを延長したときに、入力済みデータを復元する。

- サーバーセッションや下書きAPIへ保存する
- 戻る操作でも現在の入力を一時保存する
- サーバーエラー時に入力値をフォームへ戻す
- 同じ情報を後のステップで再入力させない
- 配送先と請求先が同じなら、既存情報を選択できるようにする

```html
<label>
  <input type="checkbox" name="sameAsShipping" />
  請求先に配送先と同じ住所を使う
</label>
```

同じ手続き内で以前入力した情報を再び求める場合は、自動入力するか選択できるようにする。これは認知負荷だけでなく、音声入力やスイッチ操作による再入力も減らす。

> 参照: [Understanding WCAG 2.2 SC 3.3.7 — Redundant Entry](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry.html)

`localStorage`へ氏名、住所、決済情報などを無条件に保存しない。共有端末、保存期限、暗号化、ログアウト時の削除を含めてデータ保護を設計する。カード番号やパスワードは保存しない。

## 最終確認ステップ

購入、契約、申請、データ削除など重大な結果がある操作では、確定前に内容を確認して修正できるようにする。

```html
<h1>申込み内容の確認（ステップ4 / 4）</h1>

<section aria-labelledby="review-contact">
  <h2 id="review-contact">連絡先</h2>
  <dl>
    <div>
      <dt>メールアドレス</dt>
      <dd>taro@example.com</dd>
    </div>
  </dl>
  <a href="/apply/contact">連絡先を修正する</a>
</section>

<section aria-labelledby="review-shipping">
  <h2 id="review-shipping">配送先</h2>
  <!-- 確認内容 -->
  <a href="/apply/shipping">配送先を修正する</a>
</section>

<form action="/apply/complete" method="post">
  <button type="submit">この内容で申込みを確定する</button>
</form>
```

汎用的な「編集」リンクを並べず、「連絡先を修正する」のように移動先を区別する。修正後は確認ステップへ戻れるようにする。

重大な送信では、少なくとも次のいずれかを提供する。

- 送信後に取り消せる
- 入力エラーを検出し、修正できる
- 確定前に内容を見直し、修正し、確認できる

> 参照: [Understanding WCAG 2.2 SC 3.3.4 — Error Prevention](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data.html)

## 完了画面

送信後は、手続きが成功したことと次に必要な行動を明示する。

```html
<main>
  <h1>申込みが完了しました</h1>
  <p>受付番号は<strong>AB-12345</strong>です。</p>
  <p>確認メールをtaro@example.comへ送信しました。</p>
  <a href="/account/applications/AB-12345">申込み状況を確認する</a>
</main>
```

- ステップ一覧を単に「完了」へ進めるだけにしない
- 成功か失敗かをテキストで示す
- 受付番号、保存状態、次の操作を示す
- 二重送信を防ぎ、再読み込み時の挙動を設計する
- エラー時は入力内容と再試行手段を残す

## 任意ステップと分岐

任意のステップは名前だけで判別できるようにする。

```html
<h1>プロフィール画像（任意、ステップ3 / 5）</h1>
<button type="submit" name="intent" value="skip">このステップをスキップ</button>
```

「次へ」を押せば空のまま進める仕様でも、明示的な「スキップ」操作があると選択肢を理解しやすい。

前の回答でステップが増減する場合は、次を更新する。

- ページタイトル
- メイン見出し
- ステップ一覧
- `aria-current="step"`
- `progress`の`value`と`max`を使っている場合はその値
- URLとブラウザ履歴

存在しなくなったステップの入力値を送信するか破棄するかも明確に決める。

## 時間制限と途中保存

可能なら入力時間を制限しない。セキュリティ上の理由でセッション期限が必要な場合は、期限前に警告し、延長できるようにする。

```html
<div role="alertdialog" aria-labelledby="timeout-title" aria-describedby="timeout-description">
  <h2 id="timeout-title">セッションの有効期限が近づいています</h2>
  <p id="timeout-description">入力内容は2分後に破棄されます。</p>
  <button type="button">入力時間を延長する</button>
  <button type="button">下書きを保存して終了する</button>
</div>
```

実際に`role="alertdialog"`を使う場合は、モーダルダイアログとして名前、説明、初期フォーカス、フォーカストラップ、閉じた後のフォーカス復元まで実装する。

- 残り時間を毎秒読み上げない
- 警告を十分早く出す
- 延長ボタンを最初に操作しやすくする
- 自動保存の状態を`role="status"`で簡潔に伝える
- 保存できなかった場合は理由と再試行手段を示す

## Reactで実装する

```jsx
import { useEffect, useId, useRef, useState } from 'react';

function Wizard({ steps, onReview }) {
  const [currentIndex, setCurrentIndex] = useState(0);
  const headingRef = useRef(null);
  const hasMountedRef = useRef(false);
  const headingId = useId();
  const currentStep = steps[currentIndex];

  useEffect(() => {
    document.title = `ステップ${currentIndex + 1} / ${steps.length}: ${currentStep.label}`;

    if (hasMountedRef.current) {
      headingRef.current?.focus();
    } else {
      hasMountedRef.current = true;
    }
  }, [currentIndex, currentStep.label, steps.length]);

  function goNext() {
    const isValid = currentStep.validate?.() ?? true;
    if (!isValid) return;

    if (currentIndex === steps.length - 1) {
      onReview();
      return;
    }

    setCurrentIndex((index) => index + 1);
  }

  function goBack() {
    setCurrentIndex((index) => Math.max(index - 1, 0));
  }

  return (
    <div>
      <nav aria-label="申込みの進捗">
        <ol>
          {steps.map((step, index) => (
            <li key={step.id}>
              <span aria-current={index === currentIndex ? 'step' : undefined}>
                {step.label}
                {index < currentIndex ? '（完了）' : ''}
              </span>
            </li>
          ))}
        </ol>
      </nav>

      <section aria-labelledby={headingId}>
        <h1 id={headingId} ref={headingRef} tabIndex={-1}>
          {currentStep.label}
          {`（ステップ${currentIndex + 1} / ${steps.length}）`}
        </h1>

        {currentStep.content}

        <div className="wizard-actions">
          {currentIndex > 0 && (
            <button type="button" onClick={goBack}>
              戻る
            </button>
          )}
          <button type="button" onClick={goNext}>
            {currentIndex === steps.length - 1 ? '内容を確認する' : '次へ'}
          </button>
        </div>
      </section>
    </div>
  );
}
```

このコンポーネントでは、各ステップの入力値を親コンポーネントなど、ステップを切り替えても破棄されない場所で管理する。エラーがある場合、`validate`側で個別エラーとサマリーを表示し、エラーサマリーへフォーカスを移してから`false`を返す。

実際のルーティングを使う場合は、コンポーネント内の数値だけでなくURLを状態の基準にする。ブラウザの戻る・進む、再読み込み、直接アクセスでも同じステップを復元できるようにする。

## CSSでステップを表現する

```css
.stepper {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  padding: 0;
  list-style: none;
}

.stepper li {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.stepper li:not(:last-child)::after {
  content: "";
  inline-size: 2rem;
  border-block-start: 0.125rem solid currentColor;
}

.stepper [aria-current="step"] {
  padding: 0.25rem 0.5rem;
  border: 0.125rem solid currentColor;
  border-radius: 0.25rem;
  font-weight: 700;
}

.error-summary {
  padding: 1rem;
  border: 0.25rem solid #b00020;
}

.field-error {
  color: #b00020;
  font-weight: 700;
}

.wizard-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-block-start: 2rem;
}

.visually-hidden {
  position: absolute;
  inline-size: 1px;
  block-size: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}

@media (max-width: 40rem) {
  .stepper {
    display: grid;
  }

  .stepper li::after {
    display: none;
  }
}

@media (forced-colors: active) {
  .stepper [aria-current="step"],
  .error-summary {
    border-color: CanvasText;
  }
}
```

現在位置、完了、エラーを色だけで区別しない。テキスト、枠線、太さ、形などを組み合わせる。

横並びが狭い画面で押しつぶされる場合は、縦並びにするか、「ステップ2 / 5」と現在名だけを表示する簡潔な表現へ切り替える。DOMから現在位置の情報を消さない。

## よくある失敗

### 現在位置が色でしか分からない

`aria-current="step"`と見えるステップ番号・名前を使う。色以外の見た目でも区別する。

### ステップ表示をクリックできるように見せる

移動できる項目だけをリンクまたはボタンにする。未来のステップはテキストにする。

### 戻ると入力が消える

各ステップの値を保存し、サーバーエラーや再表示でも復元する。完了済みステップへのリンクを提供するなら特に必要。

### 「戻る」でも検証エラーになる

戻る操作では現在のステップの必須検証を回避する。移動と保存の方針をサーバー側でも分ける。

### 新しいステップを表示してもフォーカスが動かない

SPAでは、非表示になった「次へ」ボタンへフォーカスを残さず、新しい見出しへ移す。

### エラーを出して次のステップへ進む

現在ステップに残し、エラーサマリーと各入力のエラーを示す。入力値は保持する。

### ステップが変わってもページタイトルが同じ

タイトルの先頭に「ステップ2 / 4: 配送先」のような現在位置を含める。

### 最後の「次へ」で即座に確定する

重大な処理では確認画面を用意し、項目ごとに修正先を提供する。確定ボタンには結果が分かる文言を使う。

### 同じ情報を何度も入力させる

以前の入力を再利用するか、選択できるようにする。記憶や再入力を利用者へ要求しない。

### URLと表示が一致しない

SPAでも履歴とステップ状態を同期する。存在しないステップや未保存のステップへ直接アクセスした場合の処理を決める。

## テストのチェックリスト

### 構造と現在位置

- [ ] 各ステップが論理的な入力グループになっている
- [ ] ページタイトルに現在位置とステップ名がある
- [ ] メイン見出しに現在位置とステップ名がある
- [ ] ステップ一覧が`ol`で構造化されている
- [ ] ステップ一覧の`nav`に用途が分かる名前がある
- [ ] 現在の一項目だけに`aria-current="step"`がある
- [ ] 未来のステップを操作可能に見せていない
- [ ] ステップ表示をタブやメニューとして実装していない

### 操作とフォーカス

- [ ] キーボードだけで全ステップを完了できる
- [ ] 「次へ」と「戻る」の順序が一貫している
- [ ] 「戻る」で不要な検証を要求しない
- [ ] SPAの遷移後に新しい見出しへフォーカスが移る
- [ ] エラー時にサマリーまたは最初のエラーを認識できる
- [ ] フォーカスが非表示要素へ残らない
- [ ] 正の`tabindex`を使っていない
- [ ] ブラウザの戻る・進むと表示中のステップが一致する

### 入力とエラー

- [ ] 各入力にラベル、説明、必要な制約がある
- [ ] 現在のステップだけを検証している
- [ ] サーバー側でも入力を検証している
- [ ] エラーをテキストで具体的に説明している
- [ ] エラー入力へ`aria-invalid="true"`を付けている
- [ ] 個別エラーが入力とプログラムで関連付いている
- [ ] エラーサマリーから該当入力へ移動できる
- [ ] 修正後に古いエラー状態を取り除いている

### データと完了

- [ ] 戻る、検証失敗、再表示で入力値が保持される
- [ ] 同じ手続きで同じ情報を再入力させていない
- [ ] 途中保存するデータと保存期限を決めている
- [ ] 機密情報をブラウザへ不用意に保存していない
- [ ] 重大な送信の前に確認・修正できる
- [ ] 確定ボタンの文言から結果を予測できる
- [ ] 完了画面で成功、受付情報、次の操作が分かる
- [ ] 失敗時に入力を保持し、再試行できる

### 表示と条件分岐

- [ ] 現在位置やエラーを色だけで伝えていない
- [ ] 200%・400%拡大でも手順とボタンを操作できる
- [ ] 狭い画面でステップ名が重ならない
- [ ] 強制カラーモードで現在位置とエラーが分かる
- [ ] 任意ステップとスキップ方法が明確である
- [ ] 分岐時に総数、一覧、タイトル、URLを同期している
- [ ] 時間制限がある場合、警告と延長手段がある

## 関連して読むとよいテーマ

- [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [aria-current — 現在位置を支援技術に伝える](./aria-current.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)
- [アクセシブルなプログレス表示の実装 — progress と meter の違い](./accessible-progress-and-meter.md)
- [アクセシブルなタブの実装](./accessible-tabs.md)
- [手動テストのチェックリスト — 最低限確認すべき項目](./manual-testing-checklist.md)

## まとめ

- 長い手続きを意味のある小さなステップへ分ける
- ページタイトル、メイン見出し、ステップ一覧で現在位置を示す
- 現在のステップへ`aria-current="step"`を使う
- 完了済みで入力を復元できるステップだけをリンクにする
- ウィザードをタブ、メニュー、独自ARIAロールにしない
- 正常な遷移では新しい見出し、エラー時はサマリーへフォーカスを移す
- 「戻る」で検証を要求せず、入力済みデータを保持する
- 現在のステップだけを検証し、個別エラーとサマリーを示す
- 同じ手続きで入力済みの情報を再入力させない
- 重大な送信では確定前に見直しと修正の機会を提供する
- SPAでもURL、履歴、タイトル、見出し、フォーカスを同期する
