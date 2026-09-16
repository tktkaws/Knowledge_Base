# アクセシブルなオートコンプリート（コンボボックス）の実装

## コンボボックスとは

- 入力欄と、候補を提示するポップアップを組み合わせたUI
- 検索サジェスト、住所・駅名の入力補助、許可された値からの選択などに使う
- ポップアップは listbox / grid / tree / dialog のいずれか
- フォーカスは入力側に残し、候補のフォーカスは `aria-activedescendant` で伝えるのが一般的

> 参照: [WAI-ARIA Authoring Practices — Combobox Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/)

## コンボボックスの種類

| 種類 | テキスト入力 | 用途 |
|---|---|---|
| Editable（編集可） | あり | オートコンプリート、自由入力＋候補提示 |
| Select-only（選択のみ） | なし | `<select>` に近い単一選択 |

### オートコンプリートの4パターン

| `aria-autocomplete` | 動き |
|---|---|
| `none` | 候補リストは出るが、入力文字で絞り込まない |
| `list` | 入力に応じた候補リストを表示。選択は手動 |
| `inline` | 入力欄に補完文字列をインライン表示 |
| `both` | 候補リスト + インライン補完 |

- 実務で最も多いのは `list`（リストオートコンプリート・手動選択）
- 自由入力を許すか、候補からのみ選ばせるかは要件で決める

## ネイティブで足りる場合

```html
<!-- 単純な候補提示なら datalist -->
<label for="fruit">果物</label>
<input id="fruit" list="fruit-list" autocomplete="off">
<datalist id="fruit-list">
  <option value="りんご"></option>
  <option value="みかん"></option>
  <option value="ぶどう"></option>
</datalist>
```

```html
<!-- 許可値からの選択のみなら select -->
<label for="prefecture">都道府県</label>
<select id="prefecture">
  <option value="">選択してください</option>
  <option value="tokyo">東京都</option>
  <option value="osaka">大阪府</option>
</select>
```

- カスタム見た目や非同期検索が不要なら、ネイティブを優先する
- `datalist` はブラウザごとのUI差が大きく、複雑な制御には向かない

## 必要なARIA属性

| 要素 | ロール / 属性 | 説明 |
|---|---|---|
| 入力欄 | `role="combobox"`（`<input>` なら暗黙でも可） | コンボボックス本体 |
| 入力欄 | `aria-expanded` | ポップアップの開閉状態 |
| 入力欄 | `aria-controls="listboxのid"` | 制御するポップアップを参照 |
| 入力欄 | `aria-autocomplete` | オートコンプリートの種類 |
| 入力欄 | `aria-activedescendant` | 現在アクティブな候補の id |
| 入力欄 | `aria-haspopup="listbox"` | ポップアップの種類（任意だが有用） |
| 候補リスト | `role="listbox"` | 候補のコンテナ |
| 各候補 | `role="option"` | 個々の候補 |
| ラベル | `<label>` または `aria-labelledby` | 入力の名前 |

- DOMフォーカスは常に入力欄に置く
- 候補は Tab 順序に含めない（矢印キーで移動）
- 開閉ボタンがある場合も、ポップアップ自体は Tab 順序外

> 参照: [APG — Editable Combobox With List Autocomplete Example](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/examples/combobox-autocomplete-list/)

## キーボード操作の要件

### 入力欄にフォーカスがあるとき

| キー | 動作 |
|---|---|
| `↓` | ポップアップを開き、最初の候補をアクティブにする |
| `↑`（任意） | ポップアップを開き、最後の候補をアクティブにする |
| `Enter` | アクティブな候補があれば確定して閉じる |
| `Escape` | 開いていれば閉じる。既に閉じていれば入力クリア（任意） |
| 文字キー | 入力し、候補を絞り込む / 表示する |
| `Alt` + `↓`（任意） | フォーカスを移さずポップアップだけ開く |

### 候補リストが開いているとき（`aria-activedescendant` 方式）

| キー | 動作 |
|---|---|
| `↓` / `↑` | 次 / 前の候補をアクティブにする |
| `Enter` | アクティブ候補を確定し、ポップアップを閉じる |
| `Escape` | 確定せずに閉じ、フォーカスは入力欄のまま |
| `Home` / `End`（任意） | 先頭 / 末尾の候補へ |

- 左右矢印は、編集可能な場合は入力欄のキャレット移動に使う
- ブラウザ標準の文字編集キーを JavaScript で潰さない

## HTML構造

```html
<label for="station-input">駅名</label>
<div class="combobox">
  <input
    id="station-input"
    type="text"
    role="combobox"
    aria-expanded="false"
    aria-controls="station-listbox"
    aria-autocomplete="list"
    aria-haspopup="listbox"
    autocomplete="off"
  >
  <ul id="station-listbox" role="listbox" hidden>
    <!-- 候補は JS で挿入 -->
  </ul>
</div>
```

### 構造のポイント

- `autocomplete="off"` でブラウザのオートフィルと競合しにくくする
- 候補がないときは listbox を `hidden` にし、`aria-expanded="false"` にする
- 結果件数の通知が必要なら、別途 `aria-live="polite"` のステータス領域を置く

## Vanilla JS での実装

```js
class AccessibleCombobox {
  constructor(input, options) {
    this.input = input;
    this.options = options;
    this.listbox = document.getElementById(input.getAttribute('aria-controls'));
    this.activeIndex = -1;
    this.filtered = [];

    this.input.addEventListener('input', () => this.onInput());
    this.input.addEventListener('keydown', (event) => this.onKeyDown(event));
    this.input.addEventListener('blur', () => {
      // クリック確定の猶予
      setTimeout(() => this.close(), 150);
    });
  }

  onInput() {
    const query = this.input.value.trim().toLowerCase();
    this.filtered = query
      ? this.options.filter((item) => item.toLowerCase().includes(query))
      : [];

    this.render();
    if (this.filtered.length > 0) {
      this.open();
    } else {
      this.close();
    }
  }

  render() {
    this.listbox.innerHTML = '';
    this.filtered.forEach((label, index) => {
      const option = document.createElement('li');
      option.id = `${this.listbox.id}-option-${index}`;
      option.setAttribute('role', 'option');
      option.textContent = label;
      option.addEventListener('mousedown', (event) => {
        event.preventDefault(); // blur より先に確定
        this.select(index);
      });
      this.listbox.appendChild(option);
    });
    this.activeIndex = -1;
    this.input.removeAttribute('aria-activedescendant');
  }

  open() {
    this.listbox.hidden = false;
    this.input.setAttribute('aria-expanded', 'true');
  }

  close() {
    this.listbox.hidden = true;
    this.input.setAttribute('aria-expanded', 'false');
    this.input.removeAttribute('aria-activedescendant');
    this.activeIndex = -1;
  }

  setActive(index) {
    const options = [...this.listbox.querySelectorAll('[role="option"]')];
    if (options.length === 0) return;

    this.activeIndex = (index + options.length) % options.length;
    options.forEach((option, i) => {
      option.classList.toggle('is-active', i === this.activeIndex);
    });
    this.input.setAttribute(
      'aria-activedescendant',
      options[this.activeIndex].id
    );
    options[this.activeIndex].scrollIntoView({ block: 'nearest' });
  }

  select(index) {
    const value = this.filtered[index];
    if (!value) return;
    this.input.value = value;
    this.close();
  }

  onKeyDown(event) {
    const isOpen = this.input.getAttribute('aria-expanded') === 'true';

    switch (event.key) {
      case 'ArrowDown':
        event.preventDefault();
        if (!isOpen && this.filtered.length > 0) this.open();
        this.setActive(this.activeIndex + 1);
        break;
      case 'ArrowUp':
        event.preventDefault();
        if (!isOpen && this.filtered.length > 0) this.open();
        this.setActive(this.activeIndex <= 0 ? this.filtered.length - 1 : this.activeIndex - 1);
        break;
      case 'Enter':
        if (isOpen && this.activeIndex >= 0) {
          event.preventDefault();
          this.select(this.activeIndex);
        }
        break;
      case 'Escape':
        if (isOpen) {
          event.preventDefault();
          this.close();
        } else if (this.input.value) {
          this.input.value = '';
        }
        break;
      default:
        break;
    }
  }
}

new AccessibleCombobox(
  document.getElementById('station-input'),
  ['東京', '品川', '新宿', '渋谷', '池袋', '横浜']
);
```

## React での実装例

```jsx
import { useId, useMemo, useState } from 'react';

function Combobox({ label, options }) {
  const inputId = useId();
  const listboxId = useId();
  const [value, setValue] = useState('');
  const [open, setOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);

  const filtered = useMemo(() => {
    const query = value.trim().toLowerCase();
    if (!query) return [];
    return options.filter((item) => item.toLowerCase().includes(query));
  }, [options, value]);

  const activeId =
    open && activeIndex >= 0
      ? `${listboxId}-option-${activeIndex}`
      : undefined;

  const select = (index) => {
    setValue(filtered[index]);
    setOpen(false);
    setActiveIndex(-1);
  };

  return (
    <div className="combobox">
      <label htmlFor={inputId}>{label}</label>
      <input
        id={inputId}
        type="text"
        role="combobox"
        aria-expanded={open}
        aria-controls={listboxId}
        aria-autocomplete="list"
        aria-activedescendant={activeId}
        aria-haspopup="listbox"
        autoComplete="off"
        value={value}
        onChange={(event) => {
          const next = event.target.value;
          setValue(next);
          setOpen(next.trim().length > 0);
          setActiveIndex(-1);
        }}
        onKeyDown={(event) => {
          if (event.key === 'ArrowDown') {
            event.preventDefault();
            setOpen(true);
            setActiveIndex((current) =>
              Math.min(current + 1, Math.max(filtered.length - 1, 0))
            );
          }
          if (event.key === 'ArrowUp') {
            event.preventDefault();
            setActiveIndex((current) => Math.max(current - 1, 0));
          }
          if (event.key === 'Enter' && activeIndex >= 0) {
            event.preventDefault();
            select(activeIndex);
          }
          if (event.key === 'Escape') {
            setOpen(false);
            setActiveIndex(-1);
          }
        }}
        onBlur={() => {
          setTimeout(() => setOpen(false), 150);
        }}
      />
      <ul id={listboxId} role="listbox" hidden={!open || filtered.length === 0}>
        {filtered.map((item, index) => (
          <li
            key={item}
            id={`${listboxId}-option-${index}`}
            role="option"
            className={index === activeIndex ? 'is-active' : undefined}
            onMouseDown={(event) => {
              event.preventDefault();
              select(index);
            }}
          >
            {item}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## 結果件数の通知（任意だが推奨）

```html
<div id="combobox-status" role="status" aria-live="polite" class="visually-hidden"></div>
```

```js
status.textContent =
  filtered.length === 0
    ? '候補はありません'
    : `${filtered.length}件の候補があります`;
```

- 候補の増減はフォーカス移動なしでは伝わらないことがある
- ライブリージョンで件数を伝えると、スクリーンリーダー利用者が状況を把握しやすい

## スタイルの注意点

```css
[role="option"].is-active,
[role="option"]:hover {
  background: #e8f0fe;
}

/* フォーカスは入力欄に残るため、アクティブ候補は視覚的に明示する */
[role="option"].is-active {
  outline: 2px solid #005fcc;
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
}
```

- アクティブ候補と選択済み候補の見た目を区別する
- ハイコントラストやキーボード操作でも、どの候補がアクティブか分かること

## よくある間違い

### 1. 候補に DOM フォーカスを移す

```js
// 間違い：option に focus() すると入力編集や読み上げが崩れる
option.focus();

// 正解：入力にフォーカスを残し、aria-activedescendant で示す
input.setAttribute('aria-activedescendant', option.id);
```

### 2. aria-expanded を更新しない

```html
<!-- 間違い：常に false のまま -->
<input role="combobox" aria-expanded="false">

<!-- 正解：開閉に合わせて切り替える -->
```

### 3. listbox / option のロール漏れ

```html
<!-- 間違い -->
<ul class="suggestions">
  <li>東京</li>
</ul>

<!-- 正解 -->
<ul role="listbox">
  <li role="option">東京</li>
</ul>
```

### 4. 候補を Tab で辿らせる

- コンボボックスの候補は矢印キーで移動する
- Tab は次のフォーム項目へ進むのが期待動作

### 5. ラベルがない

```html
<!-- 間違い -->
<input role="combobox" placeholder="駅名を検索">

<!-- 正解 -->
<label for="station">駅名</label>
<input id="station" role="combobox" ...>
```

- `placeholder` はラベルの代替にしない

### 6. Escape で閉じられない / 外側クリックで閉じない

- キーボード利用者は Escape でポップアップを閉じられる必要がある
- 確定せずに閉じたとき、既存の入力値を勝手に書き換えない

## 実装チェックリスト

- [ ] 入力に見えるラベルがある
- [ ] `role="combobox"` と `aria-expanded` / `aria-controls` / `aria-autocomplete` がある
- [ ] 候補リストが `role="listbox"`、各候補が `role="option"`
- [ ] DOMフォーカスは入力に残り、`aria-activedescendant` で候補を示している
- [ ] `↓` / `↑` で候補を移動できる
- [ ] `Enter` で確定、`Escape` で閉じられる
- [ ] 候補は Tab 順序に含まれていない
- [ ] 開閉状態と `hidden` / `aria-expanded` が一致している
- [ ] （任意）候補件数を `aria-live` で通知している
- [ ] 単純な要件なら `datalist` / `select` で代替できないか検討した

## まとめ

- オートコンプリートは Combobox パターンで実装する
- 入力にフォーカスを残し、候補は `aria-activedescendant` と矢印キーで操作する
- `aria-expanded`・`aria-controls`・`aria-autocomplete` で状態と振る舞いを伝える
- 単純な候補提示や単一選択なら、まず `datalist` / `select` を検討する
- Escape で閉じられ、ラベルがあり、候補ロールが正しいことが最低ライン
