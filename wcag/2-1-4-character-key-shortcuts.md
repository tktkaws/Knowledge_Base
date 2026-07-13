# 2.1.4 文字キーのショートカット（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.1.4「文字キーのショートカット」（Character Key Shortcuts）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.1「キーボード操作可能」に属する
- WCAG 2.1 で追加された達成基準
- **文字キーのみ**（文字・記号・数字・句読点）で構成されるキーボードショートカットがある場合、次のいずれかが真であること
  - **オフにできる**
  - **再マッピングできる**
  - **フォーカス時のみ有効**

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.4 Character Key Shortcuts](https://www.w3.org/TR/WCAG22/#character-key-shortcuts)
> - [Understanding SC 2.1.4 Character Key Shortcuts](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts.html)

## 達成基準の原文（要約）

- コンテンツで、文字（大文字・小文字）、句読点、数字、記号のみを使うキーボードショートカットを実装している場合、次のいずれかが真であること
  - **オフ**：ショートカットをオフにする仕組みがある
  - **再マッピング**：Ctrl、Alt など**印字されないキー**を含むように再マッピングできる仕組みがある
  - **フォーカス時のみ**：UI コンポーネントのショートカットは、そのコンポーネントがフォーカスを持っているときだけ有効

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.4](https://www.w3.org/TR/WCAG22/#character-key-shortcuts)

## 対象となるショートカット

- 印刷可能な文字だけからなるショートカット
  - 文字（`a`〜`z`、`A`〜`Z`）
  - 数字（`0`〜`9`）
  - 句読点・記号（`?`、`.`、`/` など）
- 単一キー（例：`k` で次のメッセージ）
- 文字キーの連続（例：`g` のあと `a` でアクション）も対象
- 物理キーが1つか複数かは問わない（`?` が `Shift+/` でも文字ショートカット）

```text
対象の例：
  k → 次の会話へ
  y → アーカイブ
  m → ミュート
  / → 検索
  g → a → アクション

対象外の例：
  Ctrl+P → 印刷
  Alt+F → ファイルメニュー
  Ctrl+Shift+S → 別名で保存
```

> **参照**
> - [Understanding SC 2.1.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts.html#intent)

## 対象外

### 修飾キーを含むショートカット

- `Ctrl`、`Alt`、`Meta`、`Shift` など、印字されないキーを含むショートカットは対象外
- 音声入力ユーザーも「press Control P」のように呼び出せる

### フォーカス時のみ有効なコンポーネント

- リストボックス、ドロップダウン、セレクトなどで、フォーカスがあるときだけ文字キーで項目を選べる場合は対象外
- メニューを `Alt` や `Alt+F` で開いてから文字キーで選択する場合も、経路全体に非印字キーが含まれるため対象外になりやすい

### accesskey

- HTML の `accesskey` は、多くのユーザーエージェントで修飾キーと組み合わせて発動するため、通常は対象外

```html
<!-- 通常は対象外：accesskey は修飾キーと組み合わせて発動 -->
<button type="button" accesskey="s">保存</button>
```

> **参照**
> - [Understanding SC 2.1.4 — Access keys](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts.html#intent)

## なぜ必要か

- 音声入力ユーザーは、口述とコマンドを同じモードで使うことが多い
- 単一文字ショートカットがあると、マイクが拾った単語が連続したコマンドになる
  - 例：「Hey Kim」→ `y`（アーカイブ）、`k`（次へ）、`m`（ミュート）が誤発動
- 手が不自由なキーボードユーザーも、誤ってキーを押しやすい
- ショートカットの再マッピングは、認知障害のあるユーザーがアプリ間で同じキーを使う助けにもなる

> **参照**
> - [Understanding SC 2.1.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts.html#benefits)
> - [Understanding SC 2.1.4 — Background on speech input](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts.html#intent)

## 条件1：オフにできる

- 文字キーショートカットを無効にする仕組みを提供する
- ショートカットが唯一の操作手段でないこと（2.1.1 との整合）
- 設定画面やトグルで切り替えられると分かりやすい

```html
<!-- 良い例：ショートカットをオフにする設定 -->
<label>
  <input type="checkbox" id="enable-shortcuts" checked>
  キーボードショートカットを有効にする
</label>
```

```javascript
const enableShortcuts = document.getElementById("enable-shortcuts");

document.addEventListener("keydown", (event) => {
  if (!enableShortcuts.checked) return;
  if (event.target.matches("input, textarea, [contenteditable]")) return;

  if (event.key === "k") {
    goToNextMessage();
  }
});
```

> **参照**
> - [Technique G217: Providing a mechanism to allow users to remap or turn off character key shortcuts](https://www.w3.org/WAI/WCAG22/Techniques/general/G217)

## 条件2：再マッピングできる

- 文字キーのみのショートカットを、`Ctrl` や `Alt` などを含む組み合わせに変更できる仕組みを提供する
- キーボードのみのユーザーが誤操作を減らすのに有効
- 認知障害のあるユーザーが、アプリ間で同じショートカットを使えるようになる

```html
<!-- 良い例：ショートカットの再マッピング UI -->
<label for="shortcut-next">次のメッセージ</label>
<select id="shortcut-next">
  <option value="k">k</option>
  <option value="Alt+k">Alt + k</option>
  <option value="Ctrl+k">Ctrl + k</option>
  <option value="off">オフ</option>
</select>
```

```javascript
// 再マッピング後は修飾キー付きで発火
document.addEventListener("keydown", (event) => {
  if (event.altKey && event.key === "k") {
    goToNextMessage();
  }
});
```

> **参照**
> - [Understanding SC 2.1.4 — Alternate Control example](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts.html#examples)

## 条件3：フォーカス時のみ有効

- UI コンポーネントのショートカットが、**そのコンポーネントにフォーカスがあるときだけ**有効なら適合
- リストボックスやカスタムセレクトで、フォーカス中に文字キーで選択する場合など
- ページ全体で常時アクティブな単一キーショートカットは、この条件を満たさない

```html
<!-- 良い例：フォーカス時のみ文字キーで選択 -->
<div
  role="listbox"
  tabindex="0"
  aria-label="都市"
  aria-activedescendant="opt-tokyo"
>
  <div role="option" id="opt-osaka">大阪</div>
  <div role="option" id="opt-tokyo" aria-selected="true">東京</div>
  <div role="option" id="opt-nagoya">名古屋</div>
</div>
```

```javascript
listbox.addEventListener("keydown", (event) => {
  // listbox にフォーカスがあるときだけ有効
  if (document.activeElement !== listbox) return;

  const key = event.key.toLowerCase();
  const match = [...options].find((opt) =>
    opt.textContent.toLowerCase().startsWith(key)
  );
  if (match) selectOption(match);
});
```

> **参照**
> - [Understanding SC 2.1.4 — listboxes and drop-down menus](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts.html#intent)

## おすすめの実装方針

- 単一文字ショートカットを使う場合は、**オフ**または**再マッピング**を用意する
- 可能なら最初から修飾キー付きショートカットにする（例：`Alt+K`）
- 入力欄にフォーカスがあるときはショートカットを無効にする
- ショートカット一覧をヘルプページに記載する

```javascript
// 良い例：入力中はショートカットを無視し、修飾キー必須
document.addEventListener("keydown", (event) => {
  const editing = event.target.matches(
    "input, textarea, select, [contenteditable='true']"
  );
  if (editing) return;

  // Alt+K のみ（文字キーのみではないので 2.1.4 対象外）
  if (event.altKey && event.key.toLowerCase() === "k") {
    event.preventDefault();
    goToNextMessage();
  }
});
```

## 実サービスの例

- Gmail：文字キーショートカットがあり、設定で無効化できる
- WordPress：文字キーショートカットの無効化・変更をサポートしているサービスがある
- これらの実装が、オフ／再マッピングの参考になる

> **参照**
> - [Understanding SC 2.1.4 — Related Resources](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts.html#resources)

## 2.1.1 との関係

| 達成基準 | 焦点 |
|---|---|
| 2.1.1 キーボード | すべての機能をキーボードで操作できること |
| 2.1.4 文字キーのショートカット | 文字キーのみのショートカットの誤発動を防ぐこと |

- ショートカットをオフにしても、同等の操作がキーボードや UI でできる必要がある（2.1.1）
- 2.1.4 は「ショートカットがある場合の安全策」であり、ショートカット自体を必須にしない

> **参照**
> - [wcag/2-1-1-keyboard.md — 2.1.1 キーボード](./2-1-1-keyboard.md)

## よくある失敗例（F99）

- 単一文字ショートカットを実装しているが、オフにも再マッピングにもできない
- ページ全体で `k` / `j` / `/` などが常時有効で、設定がない
- 入力欄以外でも文字キーが誤発動する

```javascript
// 悪い例：常時有効な単一キー、オフ不可（F99）
document.addEventListener("keydown", (event) => {
  if (event.key === "k") goToNext();
  if (event.key === "j") goToPrevious();
  if (event.key === "/") focusSearch();
});
```

```javascript
// 良い例：設定でオフ、または修飾キー付き
document.addEventListener("keydown", (event) => {
  if (!settings.shortcutsEnabled) return;
  if (event.altKey && event.key === "k") goToNext();
});
```

> **参照**
> - [Failure F99: character key shortcuts that cannot be turned off or remapped](https://www.w3.org/WAI/WCAG22/Techniques/failures/F99)

## テスト・確認方法

### 手動テスト

- ページで文字キーのみのショートカットがあるか確認
- ある場合、次のいずれかを満たすか確認
  1. オフにできる
  2. 修飾キー付きに再マッピングできる
  3. フォーカス時のみ有効
- 入力欄にフォーカスがあるときに誤発動しないかも確認

```text
チェックリスト：
1. 単一文字ショートカットがあるか
2. 設定でオフにできるか
3. Ctrl / Alt 等を含むキーに変更できるか
4. 特定コンポーネントのフォーカス時のみか
5. オフにしても同等操作ができるか（2.1.1）
```

### 開発者ツールでの確認

- `keydown` / `keypress` で単一キーを処理している箇所を確認
- `Ctrl` / `Alt` / `Meta` / `Shift` のチェックがあるか確認

```bash
# 文字キーショートカットの実装箇所をざっくり確認
rg "event\.key === ['\"][a-zA-Z0-9]['\"]|keydown|keypress" --glob "*.js" --glob "*.tsx" --glob "*.jsx"
```

### ACT Rules

- [No keyboard shortcut uses only printable characters](https://www.w3.org/WAI/standards-guidelines/act/rules/c6cf69/)

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)
> - [a11y/keyboard-navigation-basics.md — キーボードナビゲーションの基本](../a11y/keyboard-navigation-basics.md)

## まとめ

- 2.1.4 は、**文字キーのみ**のショートカットに、オフ・再マッピング・フォーカス時のみのいずれかが必要とする達成基準（レベル A）
- 音声入力ユーザーと、誤ってキーを押しやすいキーボードユーザーを守るための基準
- `Ctrl` / `Alt` など修飾キーを含むショートカットは対象外
- リストボックスなど、フォーカス時のみ文字キーが効く UI も対象外になりやすい
- ショートカットをオフにしても、同等の操作経路は残す（2.1.1）
- 可能なら最初から修飾キー付きショートカットを採用すると安全
