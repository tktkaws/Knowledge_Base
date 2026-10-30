# アクセシブルなスイッチの実装 — オン／オフとチェックボックスの使い分け

## スイッチとは

スイッチ（Switch）は、一つの設定をオンまたはオフへ切り替える入力UI。通知、ダークモード、自動再生、機器の電源など、現在の状態を即時に変更する場面で使われる。

見た目が横長のトグルでも、目的によって適切なセマンティクスは異なる。

- 設定をオン／オフへ切り替える
- フォームで項目を選択する
- 操作モードを押下状態にする
- 詳細領域を開閉する

スイッチを使う前に、利用者が何を変更し、その変更がいつ反映されるかを決める。

> 参照: [W3C WAI-ARIA APG — Switch Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/switch/)

## スイッチ、チェックボックス、トグルボタンの違い

| UI | 主な意味 | 状態 | 代表的な例 |
|---|---|---|---|
| スイッチ | 機能や設定のオン／オフ | `aria-checked="true/false"` | 通知をオンにする |
| チェックボックス | 選択肢へ含める／含めない | checked / unchecked / mixed | 利用規約に同意する |
| トグルボタン | 操作やモードの押下状態 | `aria-pressed="true/false/mixed"` | 太字モードを有効にする |
| Disclosure | 関連する領域の表示／非表示 | `aria-expanded="true/false"` | 詳細設定を開く |

見た目ではなく、利用者に伝えたい意味で選ぶ。

### スイッチが向く場面

- 変更後すぐに設定が反映される
- 状態を「オン」「オフ」と表現すると理解しやすい
- 二つの状態だけがある
- 元に戻せる変更である

```text
メール通知  オン
自動再生    オフ
機内モード  オン
```

### チェックボックスが向く場面

- フォーム送信時に複数の選択をまとめて確定する
- 「選択済み」「未選択」という表現が自然
- 利用規約への同意や、複数項目からの選択である
- 親項目が一部だけ選択された`mixed`状態を使う

```html
<label>
  <input type="checkbox" name="agreement" required>
  利用規約に同意する
</label>
```

同意は設定のオン／オフではない。トグル型の見た目にしても、意味はチェックボックスのままにする。

### トグルボタンが向く場面

```html
<button type="button" aria-pressed="false">
  太字
</button>
```

エディターの「太字」、地図の「交通情報を表示」など、ボタン操作の押下状態を示す場合に使う。フォームのチェック状態を表すものではない。

### Disclosureが向く場面

```html
<button
  type="button"
  aria-expanded="false"
  aria-controls="advanced-settings"
>
  詳細設定
</button>
<div id="advanced-settings" hidden>...</div>
```

押す目的が、単にコンテンツを表示・非表示にすることならスイッチではない。Disclosureとして`aria-expanded`を使う。

## スイッチは二値だけ

`switch`の状態はオンまたはオフの二つ。

```html
<button role="switch" aria-checked="true">通知</button>
<button role="switch" aria-checked="false">通知</button>
```

`aria-checked="mixed"`はスイッチでは無効で、WAI-ARIAでは`false`相当として扱われる。三状態が必要ならチェックボックスまたはトグルボタンを検討する。

> 参照: [WAI-ARIA 1.2 — `switch` role](https://www.w3.org/TR/wai-aria-1.2/#switch)

## 推奨するHTML

ネイティブのチェックボックスへ`role="switch"`を付けると、HTMLが持つフォーム機能とキーボード操作を維持しながら、支援技術へオン／オフのスイッチとして伝えられる。

```html
<label class="switch">
  <input
    class="switch__control"
    type="checkbox"
    role="switch"
    name="email-notifications"
    value="enabled"
    aria-describedby="email-notifications-description"
  >

  <span class="switch__track" aria-hidden="true">
    <span class="switch__thumb"></span>
  </span>

  <span class="switch__label">メール通知</span>

  <span class="switch__state" aria-hidden="true">
    <span class="switch__state-on">オン</span>
    <span class="switch__state-off">オフ</span>
  </span>
</label>

<p id="email-notifications-description">
  新しいメッセージをメールでお知らせします。
</p>
```

### ネイティブ入力を土台にする利点

- Tabキーでフォーカスできる
- Spaceキーで状態を切り替えられる
- `label`全体をクリック・タップできる
- `checked`プロパティで状態を取得できる
- フォーム送信へ参加する
- `disabled`、`required`などHTMLの機能を使える

`role="switch"`を付けた`input type="checkbox"`では、状態はHTMLの`checked`属性・プロパティで管理する。`aria-checked`を別に付けて二重管理しない。

```html
<!-- 避ける：checkedとaria-checkedが食い違う可能性がある -->
<input
  type="checkbox"
  role="switch"
  checked
  aria-checked="false"
>
```

APGのSwitch Patternでも、`input type="checkbox"`を使う場合は`aria-checked`ではなくHTMLのチェック状態を使うとしている。

## ラベルは状態が変わっても同じにする

スイッチの名前は、オンとオフを切り替えても変更しない。

```html
<!-- 避ける：状態によって名前そのものが変わる -->
<button role="switch" aria-checked="false">
  通知をオンにする
</button>

<!-- 名前は固定し、状態をaria-checkedで伝える -->
<button role="switch" aria-checked="false">
  通知
</button>
```

名前まで「通知をオンにする」「通知をオフにする」と変えると、同じコントロールを操作していることが分かりにくくなる。支援技術は、固定された名前とスイッチの状態を組み合わせて伝える。

```text
メール通知、スイッチ、オフ
メール通知、スイッチ、オン
```

画面上に「オン」「オフ」という状態テキストを表示することは有用。ただし、`aria-checked`による読み上げと重複しないよう、補助的な状態表示には`aria-hidden="true"`を付ける方法がある。

ラベルと状態を同じ文字列へ混ぜず、固定名と現在状態を別々に設計する。

## キーボード操作

| キー | 動作 |
|---|---|
| Tab / Shift + Tab | スイッチへ、またはスイッチからフォーカスを移動する |
| Space | オンとオフを切り替える |
| Enter | 切り替える実装もあるが任意 |

ネイティブのチェックボックスはSpaceキーで切り替わる。Enterキーの動作はブラウザやフォームの文脈に任せ、独自実装で必須にしない。

スイッチが複数あっても、矢印キーで移動する複合ウィジェットにはしない。通常は各スイッチがTab順序へ一つずつ入る。

## CSSでスイッチの見た目を作る

ネイティブ入力をアクセシビリティツリーとTab順序に残し、視覚的にはトラックとつまみを表示する。

```css
.switch {
  display: inline-grid;
  grid-template-columns: auto 1fr auto;
  align-items: center;
  gap: 0.75rem;
  min-block-size: 2.75rem;
  cursor: pointer;
}

.switch__control {
  position: absolute;
  inline-size: 1px;
  block-size: 1px;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
}

.switch__track {
  display: inline-flex;
  align-items: center;
  inline-size: 3rem;
  block-size: 1.75rem;
  padding: 0.125rem;
  border: 0.125rem solid currentColor;
  border-radius: 999px;
  background: Canvas;
}

.switch__thumb {
  inline-size: 1.25rem;
  block-size: 1.25rem;
  border-radius: 50%;
  background: currentColor;
  transform: translateX(0);
  transition: transform 160ms ease;
}

.switch__control:checked + .switch__track .switch__thumb {
  transform: translateX(1.25rem);
}

.switch__control:focus-visible + .switch__track {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

.switch__control:checked ~ .switch__state .switch__state-off,
.switch__control:not(:checked) ~ .switch__state .switch__state-on {
  display: none;
}

.switch:has(.switch__control:disabled) {
  cursor: not-allowed;
  opacity: 0.6;
}

@media (prefers-reduced-motion: reduce) {
  .switch__thumb {
    transition: none;
  }
}
```

この見た目は、つまみの位置と「オン／オフ」という文字の両方で状態を示す。背景色だけに依存しない。

### 入力を`display: none`にしない

```css
/* 避ける */
.switch__control {
  display: none;
}
```

`display: none`にすると、入力がTab順序とアクセシビリティツリーから外れる。画面外へ大きく移動する方法も、拡大表示やスクロールへ影響することがある。1pxの領域へ視覚的に隠し、隣接するトラックへフォーカス表示を出す。

### 強制カラーモード

背景色の違いだけでオン／オフを表すと、OSの強制カラーモードで状態を見分けられないことがある。

- `border`でトラックの輪郭を残す
- `currentColor`でつまみを描く
- つまみの位置を変える
- 見える「オン」「オフ」を表示する
- `outline`でフォーカスを示す

実際の強制カラーモードで、トラック、つまみ、状態、フォーカスが同時に識別できるか確認する。

## JavaScriptで状態を利用する

ネイティブ入力なら、状態変更は`change`イベントで取得できる。

```js
const switchInput = document.querySelector(
  'input[name="email-notifications"]'
);

switchInput.addEventListener('change', () => {
  if (switchInput.checked) {
    enableEmailNotifications();
  } else {
    disableEmailNotifications();
  }
});
```

CSSは`:checked`、JavaScriptは`checked`プロパティ、フォーム送信も同じネイティブ状態を参照する。視覚状態のためだけに別の`isOn`変数や`aria-checked`を追加しない。

### 初期状態

サーバーや保存済み設定からオンの状態を描画する場合は、`checked`属性を使う。

```html
<input
  type="checkbox"
  role="switch"
  name="email-notifications"
  value="enabled"
  checked
>
```

HTML属性の`checked`は初期状態を表す。実行中の現在状態はDOMの`checked`プロパティから取得する。

## 即時保存とステータス通知

スイッチは、操作直後に設定が反映されると予想されやすい。サーバー保存が必要なら、成功・失敗を伝える。

```html
<p id="notification-status" role="status" class="visually-hidden"></p>
```

```js
const notificationSwitch = document.querySelector(
  'input[name="email-notifications"]'
);
const notificationStatus = document.getElementById('notification-status');
let savedState = notificationSwitch.checked;
let currentRequest = null;

notificationSwitch.addEventListener('change', async () => {
  const requestedState = notificationSwitch.checked;

  currentRequest?.abort();
  const request = new AbortController();
  currentRequest = request;
  notificationStatus.textContent = '通知設定を保存しています';

  try {
    const response = await fetch('/api/settings/notifications', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ enabled: requestedState }),
      signal: request.signal,
    });

    if (!response.ok) throw new Error('設定を保存できませんでした');
    if (currentRequest !== request) return;

    savedState = requestedState;
    notificationStatus.textContent = requestedState
      ? 'メール通知をオンにしました'
      : 'メール通知をオフにしました';
  } catch (error) {
    if (error.name === 'AbortError') return;
    if (currentRequest !== request) return;

    notificationSwitch.checked = savedState;
    notificationStatus.textContent =
      '通知設定を保存できませんでした。元の状態に戻しました。';
  } finally {
    if (currentRequest === request) currentRequest = null;
  }
});
```

この例は、新しい操作が行われたら古い通信を中止する。最後に成功した状態を保持し、保存に失敗した場合は表示と入力状態を戻して通知する。

- 保存中にスイッチを操作不能にするなら、その状態を見た目でも伝える
- `aria-disabled="true"`だけでは操作を防げない
- `disabled`を使うとTab順序やフォーム送信から外れる
- 通信が遅い場合も、フォーカスを予期せず移動しない
- 失敗時に状態を戻した理由と再試行方法を伝える

操作のたびに画面全体を覆うローディング表示や確認ダイアログを出すと、スイッチの即時性が失われる。取り消せない重大な操作には、そもそもスイッチを使わない。

## フォーム送信時の注意

チェックボックスを土台にしたスイッチは、オンのときだけ`name=value`が送信される。オフの値は送信されない。

```html
<input
  type="checkbox"
  role="switch"
  name="dark-mode"
  value="enabled"
>
```

サーバー側では、値がない場合をオフとして扱う。APIへJSONを送るなら、`checked`から明示的な真偽値を作る。

```js
const payload = {
  darkMode: darkModeSwitch.checked,
};
```

同じ`name`のhidden入力を追加してオフ値を送る方法もあるが、オン時には同名の値が二つ送られる。サーバーフレームワークの解釈を確認してから使う。

## 複数のスイッチをグループ化する

関連する設定は`fieldset`と`legend`でまとめる。

```html
<fieldset>
  <legend>通知設定</legend>

  <label>
    <input type="checkbox" role="switch" name="email">
    メール通知
  </label>

  <label>
    <input type="checkbox" role="switch" name="push">
    プッシュ通知
  </label>

  <label>
    <input type="checkbox" role="switch" name="sound">
    通知音
  </label>
</fieldset>
```

`legend`によって、各スイッチが何の設定に属するかを伝えられる。一つのラベルで複数のスイッチをまとめて操作する場合は、親子の状態同期が必要になるため、チェックボックスの`mixed`状態の方が適切か検討する。

## `button`で実装する場合

フォーム送信へ参加させない設定UIでは、ネイティブ`button`をスイッチとして使える。

```html
<button
  id="location-switch"
  type="button"
  role="switch"
  aria-checked="false"
  aria-describedby="location-description"
>
  位置情報
</button>
<p id="location-description">
  現在地を使って近くの店舗を表示します。
</p>
```

```js
const locationSwitch = document.getElementById('location-switch');

locationSwitch.addEventListener('click', () => {
  const isOn = locationSwitch.getAttribute('aria-checked') === 'true';
  locationSwitch.setAttribute('aria-checked', String(!isOn));
});
```

`button`はTab、Enter、Space、クリックの動作を持つため、`div role="switch"`より実装漏れを減らせる。ただし、状態は`aria-checked`で明示的に管理し、見た目と機能も同じ値へ同期する。

### `div role="switch"`を避ける

```html
<!-- roleを付けただけでは操作できない -->
<div role="switch" aria-checked="false">位置情報</div>
```

`div`には、フォーカス、Space、Enter、クリック、無効状態、フォーム機能がない。すべてを自作する理由がなければ、`input`または`button`を使う。

`role="switch"`では子要素が支援技術上、表示用として扱われる。内部へ見出しや別の操作要素を入れず、説明は外に置いて`aria-describedby`で関連付ける。

## Reactでの実装例

```jsx
function Switch({
  id,
  label,
  description,
  checked,
  onChange,
  disabled = false,
}) {
  const descriptionId = description ? `${id}-description` : undefined;

  return (
    <div>
      <label className="switch">
        <input
          id={id}
          className="switch__control"
          type="checkbox"
          role="switch"
          checked={checked}
          disabled={disabled}
          aria-describedby={descriptionId}
          onChange={(event) => onChange(event.target.checked)}
        />

        <span className="switch__track" aria-hidden="true">
          <span className="switch__thumb" />
        </span>

        <span className="switch__label">{label}</span>

        <span className="switch__state" aria-hidden="true">
          {checked ? 'オン' : 'オフ'}
        </span>
      </label>

      {description && (
        <p id={descriptionId}>{description}</p>
      )}
    </div>
  );
}
```

```jsx
import { useState } from 'react';

function NotificationSettings() {
  const [emailEnabled, setEmailEnabled] = useState(false);

  return (
    <Switch
      id="email-notifications"
      label="メール通知"
      description="新しいメッセージをメールでお知らせします。"
      checked={emailEnabled}
      onChange={setEmailEnabled}
    />
  );
}
```

制御コンポーネントでは、`checked`を唯一の状態源にする。CSS用のクラス、表示テキスト、ARIA状態を別々のstateで管理しない。

サーバー保存を伴う場合は、親コンポーネントで読み込み・成功・失敗を管理し、失敗時に以前の`checked`へ戻す。

## 無効状態と読み取り専用

### 操作できない状態

ネイティブ入力を一時的に操作できなくする場合は`disabled`属性を使える。

```html
<input type="checkbox" role="switch" disabled>
```

ただし、無効な入力はTab順序とフォーム送信から外れる。利用者が無効な理由を理解する必要がある場合は、近くに説明を置く。

```html
<p id="notification-disabled-reason">
  メールアドレスを確認すると変更できます。
</p>
```

### `aria-disabled`を使う場合

`aria-disabled="true"`は状態を支援技術へ伝えるだけで、クリックやSpaceを止めない。フォーカス可能なまま理由を読ませたい場合に選択肢になるが、イベント処理をすべて止め、見た目も同期する必要がある。

### 読み取り専用

変更できない現在状態を伝えるだけなら、操作可能なスイッチに見せない。

```html
<dl>
  <dt>メール通知</dt>
  <dd>オン</dd>
</dl>
```

読み取り専用の情報を、フォーカスできるが操作できないスイッチとして表示すると混乱する。

## よくある失敗

### 見た目だけがスイッチ

`div`とCSSだけで作り、ラベル、ロール、状態、キーボード操作がない。ネイティブ入力を土台にする。

### ラベルが「オンにする」「オフにする」と変わる

コントロールの名前を固定し、現在状態はcheckedまたは`aria-checked`で伝える。

### `aria-checked`だけを変更する

見た目や実際の機能が更新されなければ、支援技術へ伝える状態と動作が食い違う。一つの状態源からすべてを更新する。

### チェックボックスへcheckedと`aria-checked`を両方付ける

二つの状態が矛盾する可能性がある。`input type="checkbox" role="switch"`ではネイティブのcheckedを使う。

### 状態を色だけで示す

つまみの位置、オン／オフの文字、境界など複数の手がかりを用意する。

### フォーカス表示がトラックにない

視覚的に隠したinputへフォーカスしても、画面上で位置が分からない。隣接セレクターなどで見えるトラックへフォーカス枠を出す。

### 保存に失敗してもオン表示のまま

サーバー上の状態と表示が食い違う。以前の状態へ戻し、失敗と再試行方法を通知する。

### 取り消せない操作に使う

アカウント削除や購入確定など、重大で即時に元へ戻せない操作には、目的が明確なボタンと確認手順を使う。

## テストのチェックリスト

### 意味と名前

- [ ] オン／オフという意味が目的に合っている
- [ ] 二値以外の状態を必要としていない
- [ ] ラベルが状態変更後も同じである
- [ ] 説明が必要なら`aria-describedby`で関連付いている
- [ ] スイッチ、チェックボックス、トグルボタン、Disclosureを使い分けている
- [ ] 複数のスイッチが`fieldset`と`legend`などでグループ化されている

### キーボードとフォーカス

- [ ] Tab / Shift + Tabでスイッチへ移動できる
- [ ] Spaceキーで状態を切り替えられる
- [ ] フォーカス表示が常に見える
- [ ] ラベルをクリック・タップして切り替えられる
- [ ] 無効状態では操作できず、理由を理解できる
- [ ] 状態変更後もフォーカスが予期せず移動しない

### 状態と表示

- [ ] 支援技術へオン／オフ状態が伝わる
- [ ] 見た目、機能、checkedまたは`aria-checked`が一致する
- [ ] オン／オフを色だけで区別していない
- [ ] 200%・400%拡大でもラベルとスイッチを利用できる
- [ ] 強制カラーモードでもトラック、つまみ、状態、フォーカスが見える
- [ ] 動きを減らす設定で不要なアニメーションが抑制される

### 保存とフォーム

- [ ] 変更が即時反映か、送信後に反映か分かる
- [ ] 保存中、成功、失敗の状態が必要に応じて通知される
- [ ] 保存失敗後に画面とサーバーの状態が一致する
- [ ] オフ時にフォーム値が送信されない仕様を処理している
- [ ] JavaScriptが失敗してもフォームのチェック状態を送信できる
- [ ] マウス、タッチ、キーボード、音声操作、スクリーンリーダーで確認した

## 関連して読むとよいテーマ

- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)
- [ARIAロールの基本 — role属性の種類と使い方](./aria-roles.md)
- [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカスインジケーターのカスタマイズ — :focus-visibleの活用](./focus-visible.md)
- [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)
- [アクセシブルなディスクロージャーの実装 — 一部だけを開閉する](./accessible-disclosure.md)

## まとめ

- スイッチは、即時に反映される二値のオン／オフ設定に使う
- フォーム上の選択や同意はチェックボックス、押下状態を持つ操作はトグルボタン、領域の開閉はDisclosureを使う
- `input type="checkbox" role="switch"`を土台にすると、ラベル、Space操作、フォーム機能を活用できる
- ネイティブチェックボックスの状態はcheckedで管理し、`aria-checked`を重ねない
- ラベルは状態が変わっても固定し、オン／オフは状態として伝える
- 見た目、動作、フォーム値、支援技術へ伝える状態を一つの状態源から同期する
- 色だけで状態を示さず、つまみの位置、文字、境界、フォーカス表示を用意する
- 即時保存に失敗した場合は以前の状態へ戻し、結果を通知する
