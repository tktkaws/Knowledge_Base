# アクセシブルなリストボックスの実装 — 単一選択と複数選択

## リストボックスとは

リストボックス（Listbox）は、候補の一覧から一つまたは複数の項目を選ぶウィジェット。単一選択の設定、複数のタグ選択、項目の移動対象などに使われる。

通常の縦型リストとは異なり、リストボックスは複合ウィジェット。

- Tabキーではリストボックス全体へ一度だけ入る
- 上下矢印キーでアクティブな項目を移動する
- HomeとEndで先頭・末尾へ移動する
- 文字入力で項目名を検索する
- 単一選択と複数選択で、選択の操作モデルを変える
- アクティブな項目と選択済みの項目を区別する

カスタム実装は複雑なため、まずネイティブの`select`で要件を満たせるか確認する。

> 参照: [W3C WAI-ARIA APG — Listbox Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/listbox/)

## まずネイティブの`select`を使う

### 単一選択

```html
<label for="office">勤務拠点</label>
<select id="office" name="office" required>
  <option value="">選択してください</option>
  <option value="tokyo">東京</option>
  <option value="osaka">大阪</option>
  <option value="fukuoka">福岡</option>
</select>
```

### 複数選択

```html
<label for="skills">得意な分野</label>
<select
  id="skills"
  name="skills[]"
  multiple
  size="6"
  aria-describedby="skills-hint"
>
  <option value="html">HTML</option>
  <option value="css">CSS</option>
  <option value="javascript">JavaScript</option>
  <option value="accessibility">アクセシビリティ</option>
  <option value="performance">パフォーマンス</option>
  <option value="testing">テスト</option>
</select>
<p id="skills-hint">
  複数選択できます。操作方法はOSとブラウザによって異なります。
</p>
```

### ネイティブ要素の利点

- 名前、役割、選択状態をブラウザが支援技術へ伝える
- キーボード、タッチ、音声入力へOSに合った操作を提供する
- フォーム送信と制約検証へ参加する
- JavaScriptが失敗しても選択できる
- `<optgroup>`で候補を分類できる

```html
<label for="city">都市</label>
<select id="city" name="city">
  <optgroup label="関東">
    <option value="tokyo">東京</option>
    <option value="yokohama">横浜</option>
  </optgroup>
  <optgroup label="関西">
    <option value="osaka">大阪</option>
    <option value="kyoto">京都</option>
  </optgroup>
</select>
```

見た目を統一するためだけにカスタムリストボックスへ置き換えない。

### ネイティブ複数選択の注意点

`select multiple`の操作はOSによって異なり、修飾キーが必要になる場合もある。候補が少なく、各項目を独立して選ぶ意味が自然なら、チェックボックス群の方が分かりやすいことがある。

```html
<fieldset>
  <legend>受け取りたい通知</legend>
  <label><input type="checkbox" name="notice" value="email"> メール</label>
  <label><input type="checkbox" name="notice" value="sms"> SMS</label>
  <label><input type="checkbox" name="notice" value="push"> プッシュ通知</label>
</fieldset>
```

候補数、比較の必要性、モバイル操作、利用者の習熟度から`select multiple`、チェックボックス、カスタムリストボックスを選ぶ。

> 参照: [WHATWG HTML Standard — The `select` element](https://html.spec.whatwg.org/multipage/form-elements.html#the-select-element)

## リストボックスと似たUIの違い

| UI | 主な目的 | 特徴 |
|---|---|---|
| `select` | 定型候補から値を選ぶ | ネイティブHTML |
| Listbox | 一覧を表示したまま一つ・複数を選ぶ | 矢印キーで内部移動 |
| Combobox | 入力またはボタンとポップアップ候補を組み合わせる | 開閉状態がある |
| Radio Group | 少数の候補から一つ選ぶ | すべての候補を比較しやすい |
| Checkbox Group | 各項目を独立して複数選ぶ | 各項目へ直接Tab移動できる場合がある |
| Menu | コマンドを実行する | 値の選択フォームではない |

リストボックスが常に表示されているなら、Comboboxのような`aria-expanded`や開閉ボタンは不要。

## Optionの中へ操作要素を入れない

`option`のアクセシブルネームは、通常、その内容から一つの文字列として計算される。見出し、段落、リストなどを入れても、その構造は支援技術へ保たれないことがある。

```html
<!-- 避ける：option内のリンクやボタンを個別操作できない -->
<div role="option" aria-selected="false">
  プロジェクトA
  <button type="button">編集</button>
  <a href="/projects/a">詳細</a>
</div>
```

Optionの中へリンク、ボタン、チェックボックスなどを入れない。項目ごとに複数の操作が必要なら、Grid、通常のリストとボタン、データテーブルなど別のパターンを使う。

項目名も簡潔にする。長い説明を一つの名前として毎回読み上げると、候補を比較しにくい。

## フォーカスと選択は別の状態

リストボックスでは、次の状態を区別する。

| 状態 | 意味 | 主な表現 |
|---|---|---|
| DOMフォーカス | キーボード入力を受け取る要素 | `document.activeElement` |
| アクティブ項目 | 矢印キー移動の現在位置 | `aria-activedescendant`またはロービング`tabindex` |
| 選択済み | フォーム値として選ばれた項目 | `aria-selected="true"` |

複数選択では特に、フォーカスを動かしただけで以前の選択を解除してはいけない。

```text
DOMフォーカス: リストボックス
アクティブ:     JavaScript
選択済み:       HTML、CSS、アクセシビリティ
```

この記事のカスタム例では、DOMフォーカスを`role="listbox"`のコンテナへ置き、`aria-activedescendant`でアクティブ項目を伝える。

## 単一選択のHTML

```html
<p id="language-label">主に使用する言語</p>

<div
  id="language-listbox"
  class="listbox"
  role="listbox"
  tabindex="0"
  aria-labelledby="language-label"
  aria-activedescendant="language-ja"
>
  <div
    id="language-ja"
    role="option"
    aria-selected="true"
    data-value="ja"
  >
    日本語
  </div>
  <div
    id="language-en"
    role="option"
    aria-selected="false"
    data-value="en"
  >
    英語
  </div>
  <div
    id="language-es"
    role="option"
    aria-selected="false"
    data-value="es"
  >
    スペイン語
  </div>
  <div
    id="language-fr"
    role="option"
    aria-selected="false"
    data-value="fr"
  >
    フランス語
  </div>
</div>

<input
  id="language-value"
  type="hidden"
  name="language"
  value="ja"
>
```

### 必要なロールと属性

| 要素 | 属性 | 役割 |
|---|---|---|
| コンテナ | `role="listbox"` | リストボックスとして伝える |
| コンテナ | `tabindex="0"` | DOMフォーカスを受け取る |
| コンテナ | `aria-labelledby` | 見えるラベルと関連付ける |
| コンテナ | `aria-activedescendant` | 現在の項目IDを参照する |
| 項目 | `role="option"` | 選択肢として伝える |
| 項目 | `aria-selected` | 選択状態を伝える |

`aria-activedescendant`が参照するIDは、存在するOptionの一意なIDにする。アクティブ項目を削除した場合は、属性も論理的な次の項目へ更新する。

## 複数選択のHTML

```html
<p id="skills-label">得意な分野</p>
<p id="skills-instructions">
  上下矢印キーで移動し、Spaceキーで選択を切り替えます。
</p>

<div
  id="skills-listbox"
  class="listbox"
  role="listbox"
  tabindex="0"
  aria-labelledby="skills-label"
  aria-describedby="skills-instructions"
  aria-multiselectable="true"
  aria-activedescendant="skill-html"
>
  <div
    id="skill-html"
    role="option"
    aria-selected="true"
    data-value="html"
  >
    HTML
  </div>
  <div
    id="skill-css"
    role="option"
    aria-selected="true"
    data-value="css"
  >
    CSS
  </div>
  <div
    id="skill-js"
    role="option"
    aria-selected="false"
    data-value="javascript"
  >
    JavaScript
  </div>
  <div
    id="skill-a11y"
    role="option"
    aria-selected="true"
    data-value="accessibility"
  >
    アクセシビリティ
  </div>
</div>

<div id="skills-hidden-inputs">
  <input type="hidden" name="skills[]" value="html">
  <input type="hidden" name="skills[]" value="css">
  <input type="hidden" name="skills[]" value="accessibility">
</div>

<p id="skills-status" role="status" class="visually-hidden"></p>
```

複数選択では`aria-multiselectable="true"`を付ける。選択可能なすべてのOptionへ`aria-selected="true"`または`false`を明示する。

`aria-selected`と`aria-checked`を同じ選択状態へ重ねない。この記事では単一選択と複数選択の両方で`aria-selected`へ統一する。

## キーボード操作

### 共通操作

| キー | 動作 |
|---|---|
| Tab / Shift + Tab | リストボックスへ入る、または外へ出る |
| ↓ | 次のOptionへ移動する |
| ↑ | 前のOptionへ移動する |
| Home | 先頭のOptionへ移動する |
| End | 末尾のOptionへ移動する |
| 文字入力 | 入力文字で始まるOptionへ移動する |

HomeとEndは任意だが、五項目を超えるリストでは強く推奨される。文字検索は、七項目を超える場合に特に重要。

### 単一選択

単一選択では、アクティブ項目の移動と同時に選択を変更する「selection follows focus」を採用できる。

```text
↓: 次の項目へ移動し、その項目を選択
↑: 前の項目へ移動し、その項目を選択
Space: アクティブ項目を選択
```

選択によって大きな通信、ページ遷移、破壊的な処理が即時発生する場合、矢印キーだけで選択を確定させると負担になる。フォーカス移動と確定を分けるか、ネイティブ`select`へ戻す。

### 複数選択

APGが推奨する、修飾キーを必要としないモデルを使う。

| キー | 動作 |
|---|---|
| ↓ / ↑ | 選択状態を変えず、アクティブ項目だけを移動する |
| Space | アクティブ項目の選択を切り替える |
| Shift + ↓ / ↑ | 次・前へ移動し、移動先の選択を切り替える（任意） |
| Shift + Space | 最後の選択起点から連続範囲を選ぶ（任意） |
| Ctrl / Command + A | すべて選択する（任意） |

通常の矢印移動で既存の選択を解除しない。修飾キーを押し続けなければ選択が失われる操作モデルは、利用者が意図せず選択を失いやすい。

「すべて選択」「すべて解除」が重要なら、キーボードショートカットだけでなく見えるボタンも用意する。

```html
<button type="button" id="select-all-skills">すべて選択</button>
<button type="button" id="clear-all-skills">すべて解除</button>
```

## JavaScript実装

次のクラスは、単一選択と複数選択の両方を扱う。DOMフォーカスはコンテナに残し、アクティブ項目を`aria-activedescendant`で更新する。

```js
class AccessibleListbox {
  constructor(listbox) {
    this.listbox = listbox;
    this.options = [
      ...listbox.querySelectorAll('[role="option"]'),
    ];
    this.multiple = listbox.getAttribute('aria-multiselectable') === 'true';
    this.typeahead = '';
    this.typeaheadTimer = null;

    const selectedOption = this.options.find(
      (option) => option.getAttribute('aria-selected') === 'true' &&
        option.getAttribute('aria-disabled') !== 'true'
    );
    this.activeOption = selectedOption ?? this.enabledOptions[0] ?? null;
    this.options.forEach((option) => {
      option.classList.toggle('is-active', option === this.activeOption);
    });
    this.syncActiveDescendant();

    this.listbox.addEventListener('focus', () => {
      this.activeOption?.scrollIntoView({ block: 'nearest' });
    });
    this.listbox.addEventListener('keydown', (event) => {
      this.handleKeyDown(event);
    });
    this.listbox.addEventListener('click', (event) => {
      const option = event.target.closest('[role="option"]');
      if (!option || !this.listbox.contains(option)) return;
      if (option.getAttribute('aria-disabled') === 'true') return;

      this.setActive(option, { select: !this.multiple });
      if (this.multiple) this.toggle(option);
      this.listbox.focus();
    });
  }

  get enabledOptions() {
    return this.options.filter(
      (option) => option.getAttribute('aria-disabled') !== 'true'
    );
  }

  get selectedOptions() {
    return this.options.filter(
      (option) => option.getAttribute('aria-selected') === 'true'
    );
  }

  syncActiveDescendant() {
    if (this.activeOption) {
      this.listbox.setAttribute(
        'aria-activedescendant',
        this.activeOption.id
      );
    } else {
      this.listbox.removeAttribute('aria-activedescendant');
    }
  }

  setActive(option, { select = false } = {}) {
    this.options.forEach((item) => {
      item.classList.toggle('is-active', item === option);
    });
    this.activeOption = option;
    this.syncActiveDescendant();
    option.scrollIntoView({ block: 'nearest' });

    if (select) this.selectOnly(option);
  }

  selectOnly(option) {
    this.options.forEach((item) => {
      item.setAttribute('aria-selected', String(item === option));
    });
    this.emitChange();
  }

  toggle(option) {
    const isSelected = option.getAttribute('aria-selected') === 'true';
    option.setAttribute('aria-selected', String(!isSelected));
    this.emitChange();
  }

  selectAll() {
    if (!this.multiple) return;
    this.enabledOptions.forEach((option) => {
      option.setAttribute('aria-selected', 'true');
    });
    this.emitChange();
  }

  clearAll() {
    if (!this.multiple) return;
    this.options.forEach((option) => {
      option.setAttribute('aria-selected', 'false');
    });
    this.emitChange();
  }

  moveActive(offset) {
    const options = this.enabledOptions;
    if (options.length === 0) return;

    const currentIndex = options.indexOf(this.activeOption);
    const startIndex = currentIndex === -1
      ? (offset > 0 ? -1 : options.length)
      : currentIndex;
    const nextIndex = Math.min(
      Math.max(startIndex + offset, 0),
      options.length - 1
    );
    this.setActive(options[nextIndex], { select: !this.multiple });
  }

  moveToEdge(edge) {
    const options = this.enabledOptions;
    if (options.length === 0) return;

    const option = edge === 'start' ? options[0] : options.at(-1);
    this.setActive(option, { select: !this.multiple });
  }

  handleTypeahead(key) {
    window.clearTimeout(this.typeaheadTimer);
    this.typeahead += key.toLocaleLowerCase();

    const options = this.enabledOptions;
    const startIndex = Math.max(options.indexOf(this.activeOption), -1);
    const ordered = [
      ...options.slice(startIndex + 1),
      ...options.slice(0, startIndex + 1),
    ];
    const match = ordered.find((option) =>
      option.textContent.trim().toLocaleLowerCase().startsWith(this.typeahead)
    );

    if (match) this.setActive(match, { select: !this.multiple });

    this.typeaheadTimer = window.setTimeout(() => {
      this.typeahead = '';
    }, 700);
  }

  handleKeyDown(event) {
    switch (event.key) {
      case 'ArrowDown':
        event.preventDefault();
        this.moveActive(1);
        return;
      case 'ArrowUp':
        event.preventDefault();
        this.moveActive(-1);
        return;
      case 'Home':
        event.preventDefault();
        this.moveToEdge('start');
        return;
      case 'End':
        event.preventDefault();
        this.moveToEdge('end');
        return;
      case ' ':
        if (this.activeOption) {
          event.preventDefault();
          if (this.multiple) {
            this.toggle(this.activeOption);
          } else {
            this.selectOnly(this.activeOption);
          }
        }
        return;
      default:
        break;
    }

    const isSelectAll =
      this.multiple &&
      (event.ctrlKey || event.metaKey) &&
      event.key.toLocaleLowerCase() === 'a';

    if (isSelectAll) {
      event.preventDefault();
      this.selectAll();
      return;
    }

    const isPrintable =
      event.key.length === 1 &&
      !event.ctrlKey &&
      !event.metaKey &&
      !event.altKey &&
      !event.isComposing;

    if (isPrintable) this.handleTypeahead(event.key);
  }

  emitChange() {
    const values = this.selectedOptions.map(
      (option) => option.dataset.value
    );

    this.listbox.dispatchEvent(
      new CustomEvent('selectionchange', {
        bubbles: true,
        detail: { values },
      })
    );
  }
}

const languageListbox = new AccessibleListbox(
  document.getElementById('language-listbox')
);
const languageValue = document.getElementById('language-value');

languageListbox.listbox.addEventListener('selectionchange', (event) => {
  languageValue.value = event.detail.values[0] ?? '';
});

const skillsListbox = new AccessibleListbox(
  document.getElementById('skills-listbox')
);
const skillsInputs = document.getElementById('skills-hidden-inputs');
const skillsStatus = document.getElementById('skills-status');

skillsListbox.listbox.addEventListener('selectionchange', (event) => {
  const { values } = event.detail;
  const inputs = values.map((value) => {
    const input = document.createElement('input');
    input.type = 'hidden';
    input.name = 'skills[]';
    input.value = value;
    return input;
  });

  skillsInputs.replaceChildren(...inputs);
  skillsStatus.textContent = `${values.length}件を選択しました`;
});

document.getElementById('select-all-skills').addEventListener('click', () => {
  skillsListbox.selectAll();
  skillsListbox.listbox.focus();
});

document.getElementById('clear-all-skills').addEventListener('click', () => {
  skillsListbox.clearAll();
  skillsListbox.listbox.focus();
});
```

### 端で循環させるか

この例では、先頭で上矢印、末尾で下矢印を押しても移動しない。端から反対側へ循環させる実装もあるが、リストが長い場合は現在位置を見失いやすい。製品内で一貫した方針を決める。

### アクティブ項目を見える範囲へ入れる

`aria-activedescendant`は支援技術へ現在位置を伝えるが、スクロール位置は変えない。矢印キーや文字検索で移動したら、`scrollIntoView({ block: 'nearest' })`などで項目を見える範囲へ入れる。

### 文字検索

入力された文字を短時間だけ蓄積し、その文字列で始まる次の項目へ移動する。

- 同じ文字を繰り返したときの移動を確認する
- 大文字・小文字を区別しない
- IME変換中のキーを処理しない
- 日本語など対象言語で実際にテストする
- 文字検索をフィルタリング機能と混同しない

候補を入力文字で絞り込みたい場合は、Listbox単体ではなくComboboxを検討する。

## 複数選択の一括操作

前に示した一括操作ボタンはリストボックスの外へ置き、通常のTab順序で操作できるようにする。複数の一括操作をまとめる場合は、見出しを付けるか、名前のあるグループとして関連を伝える。

操作後はリストボックスへフォーカスを戻すか、ボタンに残すかを一貫させる。前のJavaScript例ではリストボックスへ戻している。

選択件数を`role="status"`で通知するときは、「3」のような数字だけではなく「3件を選択しました」と文脈を含める。矢印移動だけでは通知せず、選択が変わったときだけ更新する。

## Reactでネイティブ`select`を使う

### 単一選択

```jsx
import { useId } from 'react';

function OfficeSelect({ value, onChange, offices }) {
  const selectId = useId();

  return (
    <div>
      <label htmlFor={selectId}>勤務拠点</label>
      <select
        id={selectId}
        name="office"
        value={value}
        onChange={(event) => onChange(event.target.value)}
      >
        <option value="">選択してください</option>
        {offices.map((office) => (
          <option key={office.value} value={office.value}>
            {office.label}
          </option>
        ))}
      </select>
    </div>
  );
}
```

複数配置する可能性があるコンポーネントでは、固定IDではなく`useId()`や親から渡されたIDを使う。

### 複数選択

```jsx
import { useId } from 'react';

function SkillsSelect({ values, onChange, skills }) {
  const selectId = useId();

  const handleChange = (event) => {
    const nextValues = [...event.target.selectedOptions].map(
      (option) => option.value
    );
    onChange(nextValues);
  };

  return (
    <div>
      <label htmlFor={selectId}>得意な分野</label>
      <select
        id={selectId}
        name="skills[]"
        multiple
        size={Math.min(skills.length, 8)}
        value={values}
        onChange={handleChange}
      >
        {skills.map((skill) => (
          <option key={skill.value} value={skill.value}>
            {skill.label}
          </option>
        ))}
      </select>
    </div>
  );
}
```

複数選択のstateは配列または`Set`として管理する。一項目ごとに別々の真偽値を持つと、フォーム値との同期が複雑になる。

## CSSで状態を区別する

```css
.listbox {
  position: relative;
  max-block-size: 18rem;
  padding: 0.25rem;
  overflow: auto;
  border: 0.125rem solid CanvasText;
  border-radius: 0.25rem;
  color: CanvasText;
  background: Canvas;
}

.listbox [role="option"] {
  padding: 0.625rem 0.75rem;
  border: 0.125rem solid transparent;
  cursor: default;
}

.listbox [role="option"][aria-selected="true"] {
  font-weight: 700;
  text-decoration: underline;
}

.listbox [role="option"][aria-selected="true"]::before {
  content: "✓ ";
}

.listbox:focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

.listbox[aria-activedescendant] [role="option"].is-active {
  border-color: Highlight;
}

.listbox [role="option"][aria-disabled="true"] {
  text-decoration: line-through;
  opacity: 0.6;
}
```

CSSだけでは、どのOptionが`aria-activedescendant`に指定されているかを選択できない。現在のCSSでは属性値とIDを動的に照合できないため、前のJavaScript例は初期化時と`setActive()`で`.is-active`も同期している。

- アクティブ項目と選択済み項目を異なる見た目にする
- 色だけで状態を区別しない
- 強制カラーモードでも境界とチェック印を確認する
- 200%・400%拡大でも項目名が切れず、スクロールできるようにする
- フォーカス中の項目が固定ヘッダーなどに完全に隠れないようにする

ARIA属性だけでなく、DOMフォーカス、アクティブ表示、スクロール位置を同じ状態源から更新する。

## グループ化したOption

カスタムリストボックスで分類が必要なら、`role="group"`へ名前を付け、その中へOptionを置く。

```html
<p id="animal-label">好きな動物</p>

<div role="listbox" tabindex="0" aria-labelledby="animal-label">
  <div role="group" aria-labelledby="land-label">
    <p id="land-label">陸の動物</p>
    <div id="animal-cat" role="option" aria-selected="false">猫</div>
    <div id="animal-dog" role="option" aria-selected="false">犬</div>
  </div>

  <div role="group" aria-labelledby="water-label">
    <p id="water-label">水の動物</p>
    <div id="animal-dolphin" role="option" aria-selected="false">イルカ</div>
    <div id="animal-eel" role="option" aria-selected="false">ウナギ</div>
  </div>
</div>
```

- 各Groupに一つ以上のOptionを入れる
- Groupのラベル自体を選択項目にしない
- 矢印キーではGroupラベルを飛ばし、Option間だけを移動する
- 深い階層を作らない

国と都市のように候補名の共通部分が長くなる場合、グループ化より「国」と「都市」を別々の入力に分ける方が使いやすいこともある。

## 無効なOption

```html
<div
  id="office-sapporo"
  role="option"
  aria-selected="false"
  aria-disabled="true"
>
  札幌 — 現在選択できません
</div>
```

無効な項目を矢印移動でスキップするか、存在と理由を確認できるようアクティブにするかを仕様化する。この記事のJavaScript例はスキップする。

- クリックとSpaceで選択できないようにする
- 色だけで無効状態を示さない
- 項目名または関連する説明で理由を伝える
- 選択済み項目を後から無効にする場合、選択状態とフォーム値を整理する

## 動的読み込みと仮想スクロール

大量の候補を一部だけDOMへ描画する場合、支援技術へ全体件数と位置を伝える。

```html
<div
  role="option"
  aria-selected="false"
  aria-posinset="51"
  aria-setsize="500"
>
  51番目の候補
</div>
```

- `aria-setsize`へ全候補数を設定する
- `aria-posinset`へ全体内での1始まりの位置を設定する
- `aria-activedescendant`がDOMから消えたOptionを参照しない
- 読み込み中や失敗を`role="status"`などで伝える
- キーボード移動で次の範囲を必要としたら、読み込んでからアクティブ状態を更新する
- 絞り込みで件数が変わったら位置情報も更新する

仮想化は支援技術との組み合わせで問題が起きやすい。候補数が非常に多いなら、検索可能なComboboxや段階的な選択へ分けられないか先に検討する。

## よくある失敗

### OptionをすべてTab順序へ入れる

Tabキーで項目を一つずつ通過させない。Tabはリストボックスへ出入りし、内部は矢印キーで移動する。

### フォーカスと選択を同じものとして扱う

複数選択で矢印キーを押すたびに以前の選択を失わない。アクティブ項目と`aria-selected`を分ける。

### `aria-selected="true"`を複数付けても`aria-multiselectable`がない

複数選択ならリストボックスへ`aria-multiselectable="true"`を付ける。

### 選択されていないOptionに状態がない

選択可能なOptionには`aria-selected="false"`も明示し、状態のない項目と区別する。

### `aria-selected`と`aria-checked`を重ねる

同じ選択状態を二種類の属性で表さない。製品内で一つの方式へ統一する。

### Option内にボタンやリンクを入れる

Listboxの操作モデルでは内部の操作要素へ移動できない。Gridや通常のリストなど別のパターンを使う。

### `aria-activedescendant`だけを更新する

見た目とスクロール位置も更新し、現在項目を視覚的に確認できるようにする。

### 文字検索でブラウザ機能を奪う

Ctrl、Command、Altとの組み合わせやIME変換中のキーを処理しない。対象言語で検証する。

### 選択変更のたびに過剰に通知する

支援技術はOptionの選択状態を読み上げる。ライブリージョンは一括操作や件数確認など、追加情報が必要な場面に限定する。

## テストのチェックリスト

### パターンの選択

- [ ] ネイティブ`select`で要件を満たせない理由がある
- [ ] 少数の複数選択ならチェックボックスも検討した
- [ ] 絞り込みが必要ならComboboxも検討した
- [ ] Option内にリンク、ボタン、チェックボックスがない
- [ ] 項目名が短く、違いを早く聞き取れる

### 構造と状態

- [ ] リストボックスに見える名前が関連付いている
- [ ] すべての選択肢に`role="option"`がある
- [ ] 選択可能なOptionに`aria-selected`が明示されている
- [ ] 複数選択に`aria-multiselectable="true"`がある
- [ ] `aria-activedescendant`が存在する一意なIDを参照している
- [ ] 見た目、ARIA状態、フォーム値が一致する

### キーボードとフォーカス

- [ ] Tab / Shift + Tabでリストボックスへ出入りできる
- [ ] 上下矢印で前後の有効なOptionへ移動できる
- [ ] Home / Endで先頭・末尾へ移動できる
- [ ] 文字入力で一致するOptionへ移動できる
- [ ] 複数選択では矢印移動だけで選択が変わらない
- [ ] Spaceでアクティブ項目の選択を切り替えられる
- [ ] アクティブ項目がスクロール領域内に見える
- [ ] フォーカス表示とアクティブ表示を確認できる

### 表示と入力方法

- [ ] アクティブ項目と選択済み項目を区別できる
- [ ] 状態を色だけに依存していない
- [ ] 無効項目を操作できず、理由を理解できる
- [ ] 200%・400%拡大でも項目名と操作を利用できる
- [ ] 強制カラーモードでも境界、選択、現在位置が見える
- [ ] クリック・タップした項目がアクティブ状態にもなる
- [ ] マウス、タッチ、キーボード、音声操作、スクリーンリーダーで確認した

### 複数選択と動的更新

- [ ] 選択方法を画面上の説明で伝えている
- [ ] 一括操作をショートカットだけに依存していない
- [ ] 選択件数の通知が過剰になっていない
- [ ] 項目追加・削除後もアクティブIDと選択状態が有効である
- [ ] 仮想化時の`aria-setsize`と`aria-posinset`が正しい
- [ ] 通信失敗後も選択状態とフォーム値が一致する

## 関連して読むとよいテーマ

- [アクセシブルなオートコンプリート（コンボボックス）の実装](./accessible-combobox.md)
- [ロービングタブインデックスパターン](./roving-tabindex.md)
- [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)
- [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)

## まとめ

- 単純な単一選択・複数選択では、まずネイティブの`select`を使う
- カスタムListboxでは、Tabは全体へ一度だけ入り、内部は矢印キーで移動する
- DOMフォーカス、アクティブ項目、選択済み項目を別の状態として管理する
- 単一選択では選択をフォーカスへ追従させられるが、大きな副作用がある場合は慎重に選ぶ
- 複数選択では、矢印キーは移動だけ、Spaceキーは選択切り替えにする
- `aria-selected`と`aria-checked`を重ねず、選択状態の表現を統一する
- Option内にリンクやボタンを入れず、項目名を簡潔にする
- アクティブ項目を見える範囲へスクロールし、視覚状態とフォーム値も同期する
- 大量候補の仮想化より、検索可能なComboboxや段階的な選択を先に検討する
