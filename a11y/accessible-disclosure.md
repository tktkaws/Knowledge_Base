# アクセシブルなディスクロージャーの実装 — 一部だけを開閉する

## ディスクロージャーとは

ディスクロージャー（Disclosure）は、ボタンを操作して、対応する一部分を表示・非表示にするUI。

- 「詳細を見る」で補足説明を開く
- 検索フォームの「詳細条件」を開く
- カードの概要から追加情報を開く
- FAQの質問から回答を開く
- ナビゲーションの下位リンクを開く

基本構造は、**開閉ボタン** と **そのボタンが制御するコンテンツ** の二つ。

| 状態 | ボタン | 制御するコンテンツ |
|---|---|---|
| 閉じている | `aria-expanded="false"` | 非表示 |
| 開いている | `aria-expanded="true"` | 表示 |

> 参照: [W3C WAI-ARIA APG — Disclosure (Show/Hide) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/)

## アコーディオンやメニューとの違い

外観が似ていても、目的と操作方法が異なる。

| パターン | 主な目的 | 特徴 |
|---|---|---|
| Disclosure | 一つの領域を開閉する | ボタンと制御対象の最小構成 |
| Accordion | 複数の見出しとパネルをまとめる | 見出し構造や、複数パネル間の制御を持つ |
| Menu Button | コマンドの一覧を開く | `menu` / `menuitem` と矢印キー操作を使う |
| Dialog | 作業や確認を別の層で行う | フォーカス移動、背景の操作制限、閉じる操作が必要 |
| Popover | 他の内容の上へ一時的に表示する | 外側操作で閉じるなど、表示方式に固有の挙動がある |

単に追加情報をページ内へ表示して、後続コンテンツを下へ押し広げるならDisclosureが合う。開いた領域を矢印キーで巡回させたり、フォーカスを閉じ込めたりする必要はない。

> 参照:
> - [アクセシブルなアコーディオンの実装](./accessible-accordion.md)
> - [アクセシブルなドロップダウンメニューの実装](./accessible-dropdown-menu.md)
> - [アクセシブルなモーダルダイアログの実装](./accessible-modal-dialog.md)

## まず`<details>`と`<summary>`を検討する

単純な開閉には、ネイティブHTMLの`details`要素と`summary`要素を使える。

```html
<details>
  <summary>配送にかかる日数</summary>
  <p>通常は注文から2〜3営業日で発送します。</p>
</details>
```

ブラウザが次の機能を提供する。

- `summary`を操作できるコントロールとして公開する
- マウス、タッチ、キーボードで開閉できる
- `open`属性で開閉状態を管理する
- 開閉状態を支援技術へ伝える
- JavaScriptが無効でも基本的な開閉が動作する

`summary`へ`role="button"`や`aria-expanded`を重ねて追加する必要はない。ネイティブの意味と状態をブラウザに任せる。

### 初期状態を開いておく

```html
<details open>
  <summary>現在適用中の条件</summary>
  <p>在庫あり、送料無料の商品を表示しています。</p>
</details>
```

`open="false"`と書いても閉じない。`open`は真偽属性なので、属性が存在すれば開いた状態になる。閉じる場合は`open`属性自体を付けない。

```html
<!-- 閉じた状態 -->
<details>
  <summary>詳細</summary>
  <p>追加情報</p>
</details>

<!-- open属性が存在するため、開いた状態になる -->
<details open="false">
  <summary>詳細</summary>
  <p>追加情報</p>
</details>
```

### 開閉をJavaScriptで監視する

`details`の状態変更は`toggle`イベントで取得できる。

```js
const details = document.querySelector('details');

details.addEventListener('toggle', () => {
  const state = details.open ? 'open' : 'closed';
  console.log(`Disclosure is ${state}`);
});
```

分析イベントや状態保存のために監視する場合も、クリックイベントから状態を推測せず、`details.open`を確認する。キーボード操作やスクリプトによる変更も扱える。

### 複雑な内容を`summary`へ詰め込まない

`summary`は、開く内容を簡潔に表すラベルにする。

```html
<!-- 避けたい例：summary内に別の操作を詰め込む -->
<details>
  <summary>
    商品情報
    <a href="/favorite">お気に入りに追加</a>
  </summary>
  <p>商品の詳細...</p>
</details>
```

別のリンクやボタンを入れると、summaryの開閉操作と内側の操作が競合しやすい。複数の操作があるカードでは、開閉ボタンを独立させるカスタムDisclosureを検討する。

> 参照: [WHATWG HTML Standard — The `details` and `summary` elements](https://html.spec.whatwg.org/multipage/interactive-elements.html#the-details-element)

## カスタムDisclosureが必要な場合

次のような場合は、`button`と制御対象を組み合わせる。

- 開閉ボタンと表示領域を別の場所へ置く必要がある
- アプリの状態管理と開閉状態を同期する
- 一つのボタンから特定の領域を明示的に制御する
- カード内に見出し、リンク、フォームなど複数の要素があり、開閉ボタンを分離したい
- `<details>`では実現しにくい表示設計が必要

見た目を変えたいだけなら、最初に`details`と`summary`をCSSで調整できないか確認する。

## 必要なHTML構造

```html
<section class="disclosure">
  <h2>商品検索</h2>

  <button
    type="button"
    id="advanced-search-trigger"
    data-disclosure
    aria-expanded="false"
    aria-controls="advanced-search-panel"
  >
    詳細条件
    <svg aria-hidden="true" focusable="false" viewBox="0 0 20 20">
      <path d="M5 7.5 10 12.5 15 7.5" />
    </svg>
  </button>

  <div id="advanced-search-panel" hidden>
    <label>
      価格の上限
      <input type="number" name="max-price">
    </label>
  </div>
</section>
```

| 要素・属性 | 役割 |
|---|---|
| `button` | ネイティブの操作、フォーカス、buttonロールを提供する |
| `type="button"` | フォーム内で意図せず送信されるのを防ぐ |
| `aria-expanded` | 制御する領域の現在の開閉状態を伝える |
| `aria-controls` | 制御対象の`id`を参照する。APGでは任意 |
| `hidden` | 閉じた内容を画面、Tab順序、アクセシビリティツリーから除外する |
| `aria-hidden="true"` | 装飾アイコンを支援技術から除外する |

### `aria-expanded`はボタンに付ける

開閉状態を持つのは、制御されるパネルではなく、開閉を行うボタン。

```html
<!-- 良い例 -->
<button aria-expanded="false" aria-controls="shipping-details">
  配送の詳細
</button>
<div id="shipping-details" hidden>...</div>

<!-- 悪い例：パネル側にaria-expandedを付けている -->
<button>配送の詳細</button>
<div id="shipping-details" aria-expanded="false" hidden>...</div>
```

### `aria-controls`だけに頼らない

`aria-controls`は制御関係を表すが、パネルを表示したり、フォーカスを移動したりする機能はない。支援技術による利用方法にも差がある。

- `aria-controls`の値とパネルの`id`を一致させる
- 実際の表示は`hidden`などで制御する
- 状態は`aria-expanded`で伝える
- DOM上でもボタンとパネルを近くに置くと関係を理解しやすい

## JavaScriptで状態を同期する

```js
class Disclosure {
  constructor(button) {
    this.button = button;
    this.panel = document.getElementById(
      button.getAttribute('aria-controls')
    );

    if (!this.panel) return;

    this.setExpanded(button.getAttribute('aria-expanded') === 'true');
    this.button.addEventListener('click', () => this.toggle());
  }

  get isExpanded() {
    return this.button.getAttribute('aria-expanded') === 'true';
  }

  setExpanded(expanded) {
    this.button.setAttribute('aria-expanded', String(expanded));
    this.panel.hidden = !expanded;
  }

  toggle() {
    this.setExpanded(!this.isExpanded);
  }
}

document
  .querySelectorAll('button[data-disclosure]')
  .forEach((button) => new Disclosure(button));
```

一つの状態変更で、`aria-expanded`と`hidden`の両方を更新する。別々のイベントハンドラーで管理すると、画面は閉じているのに「展開済み」と読み上げる不整合が起きやすい。

### ボタンのラベルを安定させる

`aria-expanded`が開閉状態を伝えるため、ボタンのラベルは「詳細条件」のように固定できる。

```html
<button aria-expanded="false" aria-controls="details-panel">
  詳細条件
</button>
```

「詳細を表示」から「詳細を非表示」へラベルを変更する設計も可能だが、音声操作で呼ぶ名前や、自動テストで識別する名前も変わる。変更する場合は、視覚表示とアクセシブルネームを一致させ、状態変化後も操作対象を見失わないことを確認する。

## Reactでの実装例

```jsx
import { useState } from 'react';

function Disclosure({ id, label, children, defaultExpanded = false }) {
  const [expanded, setExpanded] = useState(defaultExpanded);

  return (
    <div className="disclosure">
      <button
        type="button"
        aria-expanded={expanded}
        aria-controls={id}
        onClick={() => setExpanded((current) => !current)}
      >
        {label}
        <span aria-hidden="true">⌄</span>
      </button>

      <div id={id} hidden={!expanded}>
        {children}
      </div>
    </div>
  );
}

function SearchOptions() {
  return (
    <Disclosure id="advanced-search" label="詳細条件">
      <label>
        価格の上限
        <input type="number" name="maxPrice" />
      </label>
    </Disclosure>
  );
}
```

- 同じページで`id`が重複しないようにする
- サーバーとクライアントの初期状態を一致させる
- DOMからパネルを削除する実装でもよいが、再表示時に入力値や内部状態を失うか確認する
- CSSで隠すだけでなく、`hidden`が実際の開閉状態と一致するようにする

## キーボード操作

APGがDisclosureに求める基本操作は二つ。

| キー | 動作 |
|---|---|
| `Enter` | 開閉ボタンを起動し、表示状態を切り替える |
| `Space` | 開閉ボタンを起動し、表示状態を切り替える |

ネイティブの`button`を使えば、ブラウザがこの操作を提供する。`keydown`を追加して独自にクリックを発火すると、Spaceキーで二重に切り替わるなどの不具合を作ることがある。

```html
<!-- 避ける：buttonに標準操作を重ねて実装している -->
<button
  onclick="toggleDetails()"
  onkeydown="if (event.key === ' ') toggleDetails()"
>
  詳細
</button>
```

### 開いたときにフォーカスを移動しない

通常のDisclosureでは、開いた後もフォーカスをボタンに残す。その後のTab操作で、表示された領域内にある最初のフォーカス可能要素へ自然に移動できる。

- 開くだけでパネルや最初の入力欄へフォーカスを飛ばさない
- パネル内に操作要素がなくても、パネルへ`tabindex="0"`を付ける必要はない
- `Escape`で閉じる操作は基本パターンの必須要件ではない
- フォーカストラップを実装しない

ダイアログやメニューの操作を、単純なDisclosureへ持ち込まない。

### フォーカスが内側にある状態で閉じる場合

利用者が同じ開閉ボタンを押す場合、フォーカスはすでにボタンにある。一方、外部操作やアプリの状態変更でパネルを閉じるとき、フォーカスがパネル内に残っている場合がある。

```js
function closeDisclosure(button, panel) {
  if (panel.contains(document.activeElement)) {
    button.focus();
  }

  button.setAttribute('aria-expanded', 'false');
  panel.hidden = true;
}
```

フォーカス中の要素を先に非表示にすると、フォーカスが`body`へ戻るなど、利用者が現在位置を見失うことがある。先に論理的な移動先へフォーカスを戻してから隠す。

## 見た目でも状態を伝える

```css
.disclosure > button {
  align-items: center;
  display: inline-flex;
  gap: 0.5rem;
  min-block-size: 44px;
}

.disclosure > button:focus-visible {
  outline: 3px solid CanvasText;
  outline-offset: 3px;
}

.disclosure > button [aria-hidden="true"] {
  transition: transform 150ms ease;
}

.disclosure > button[aria-expanded="true"] [aria-hidden="true"] {
  transform: rotate(180deg);
}

@media (prefers-reduced-motion: reduce) {
  .disclosure > button [aria-hidden="true"] {
    transition: none;
  }
}
```

- アイコンの向きだけでなく、`aria-expanded`で状態を伝える
- 色だけに頼らず、形やテキストなども組み合わせる
- フォーカスインジケーターを消さない
- アイコンは`currentColor`を使うと、強制カラーモードでも見えやすい
- 動きを減らす設定では、回転や開閉アニメーションを抑える

## 開閉アニメーションの注意点

`hidden`属性が付いた要素は表示されないため、閉じたまま高さをアニメーションできない。アニメーションのために`hidden`を外したまま、`height: 0`や`opacity: 0`だけで隠すと、見えないリンクや入力欄へTabキーで移動できることがある。

- 閉じた最終状態では`hidden`を付ける
- 開くときは表示してからアニメーションを開始する
- 閉じるときはアニメーション完了後に`hidden`を付ける
- 閉じる途中で、内容が操作可能なままにならないようにする
- 連打、アニメーション中断、`transitioncancel`も考慮する
- `prefers-reduced-motion: reduce`では即時に開閉できるようにする

アニメーションが必須でなければ、`hidden`を即時に切り替える実装が最も単純で堅牢。

## `role="region"`は通常不要

開いたパネルへ一律に`role="region"`を付けると、ランドマークが増え、スクリーンリーダーでページ構造を把握しにくくなる。

- 単なる補足説明なら、通常の`div`でよい
- ページ内で独立した重要な区画なら、見出しを付けて`section`を検討する
- `region`を使うなら、`aria-labelledby`または`aria-label`で名前を付ける
- FAQや多数のカードすべてをランドマークにしない

意味のあるHTML構造を先に選び、ARIAロールを一律に追加しない。

## よくある失敗

### `div`をボタンにする

```html
<!-- 悪い例：フォーカスとキーボード操作がない -->
<div onclick="toggleDetails()">詳細を見る</div>

<!-- 良い例 -->
<button type="button" aria-expanded="false" aria-controls="details">
  詳細を見る
</button>
```

### 状態を片方しか更新しない

```js
// 悪い例：読み上げる状態だけが変わり、パネルは表示されない
button.setAttribute('aria-expanded', 'true');
```

`aria-expanded`、表示状態、矢印アイコンなどを同じ状態から更新する。

### `aria-hidden`と`hidden`を重ねる

```html
<!-- hiddenだけで画面と支援技術の両方から隠れる -->
<div id="details" hidden>...</div>
```

単純な開閉では`aria-hidden`を追加する必要はない。複数の非表示手段を別々に更新すると、`hidden`は外れたのに`aria-hidden="true"`が残る不整合を作りやすい。

### カード全体を無条件にクリック可能にする

カード内にリンク、ボタン、入力欄、テキスト選択がある場合、どこをクリックしても開閉すると、本来の操作を妨げる。

- 明確な開閉ボタンを用意する
- カード全体のクリックを補助的に実装する場合、内側の操作要素とテキスト選択を除外する
- キーボードフォーカスは実際の開閉ボタンへ置く
- ボタンのアクセシブルネームを短く具体的にする

> 参照: [W3C WAI-ARIA APG — Example Disclosure Card](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/examples/disclosure-card/)

## テストのチェックリスト

### キーボード

- [ ] Tabキーで開閉ボタンへ移動できる
- [ ] EnterキーとSpaceキーのどちらでも開閉できる
- [ ] 開いた直後もフォーカスがボタンに残る
- [ ] 開いた内容のリンクや入力欄へ通常のTab順序で移動できる
- [ ] 閉じた内容へフォーカスが移動しない
- [ ] 外部操作で閉じてもフォーカスを失わない
- [ ] フォーカス位置を見た目で確認できる

### 状態と構造

- [ ] 閉じた状態で`aria-expanded="false"`になっている
- [ ] 開いた状態で`aria-expanded="true"`になっている
- [ ] `aria-controls`が存在する要素の`id`を参照している
- [ ] `id`がページ内で重複していない
- [ ] `hidden`と表示状態が一致している
- [ ] アイコンを非表示にしても、ボタンの目的が分かる
- [ ] ボタンの名前が開閉する内容を説明している

### 表示と支援技術

- [ ] 200%拡大や狭い画面でもボタンと内容を利用できる
- [ ] 強制カラーモードでも開閉状態とフォーカスを確認できる
- [ ] 動きを減らす設定で不要なアニメーションが抑制される
- [ ] スクリーンリーダーでボタン、名前、開閉状態が伝わる
- [ ] タッチで操作でき、十分な操作領域がある

## 関連して読むとよいテーマ

- [ARIA APGの読み方 — ウィジェット実装パターン集の公式ガイド](./aria-apg.md)
- [aria-expanded / aria-hidden / aria-controls の活用](./aria-expanded-hidden-controls.md)
- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)
- [アクセシブルなアコーディオンの実装](./accessible-accordion.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [prefers-reduced-motion — モーション設定への対応](./prefers-reduced-motion.md)

## まとめ

- Disclosureは、ボタンで対応する一つの領域を表示・非表示にするパターン
- 単純な開閉では、まずネイティブの`details`と`summary`を使う
- カスタム実装では`button`に`aria-expanded`を付け、制御対象の`hidden`と常に同期する
- `aria-controls`は制御関係を補足する任意の属性で、表示やフォーカスを自動制御しない
- 開いたときは通常ボタンにフォーカスを残し、Tabキーで内容へ進めるようにする
- 閉じた内容を高さや透明度だけで隠さず、最終的に`hidden`で操作と読み上げの対象から外す
- アコーディオン、メニュー、ダイアログの操作モデルを単純なDisclosureへ持ち込まない
