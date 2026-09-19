# キーボードショートカットの設計原則とWCAG 2.1要件

## ショートカットとキーボード操作の違い

| 概念 | 意味 | 例 |
|---|---|---|
| キーボード操作 | すべての機能をキーボードで使えること | Tab で移動、Enter で実行 |
| キーボードショートカット | 特定操作への短縮キー | `Ctrl + K` で検索を開く |

- ショートカットは**追加の便利機能**であり、通常のキーボード操作の代替ではない
- ショートカットがなくても、同じ操作をボタンやメニューから実行できる必要がある

> 参照: [WCAG 2.1.1 Keyboard](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html)

## 関係するWCAG達成基準

| 達成基準 | レベル | 要点 |
|---|---|---|
| 2.1.1 キーボード | A | すべての機能をキーボードで操作できる |
| 2.1.2 キーボードトラップなし | A | フォーカスを閉じ込めない（モーダル等は例外的に意図的トラップ） |
| 2.1.4 文字キーのショートカット | A | 文字キーだけのショートカットは無効化・再割当・フォーカス限定のいずれか |

- 2.1.4 は WCAG 2.1 で追加された基準
- 「文字キー」= 文字・数字・句読点・記号だけで、修飾キーを含まないショートカット

> 参照: [Understanding SC 2.1.4: Character Key Shortcuts](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts.html)

## WCAG 2.1.4 の要件

文字キーだけのショートカットを実装する場合、次の**いずれか1つ以上**が必要。

| 対応 | 内容 |
|---|---|
| Turn off（無効化） | 設定でショートカットをオフにできる |
| Remap（再割当） | `Ctrl` / `Alt` などの修飾キー付きに変更できる |
| Active only on focus（フォーカス時のみ） | 対象コンポーネントにフォーカスがあるときだけ有効 |

### なぜ必要か

- 音声入力ユーザーは、発話が文字キー入力として解釈され、意図せずショートカットが発火しやすい
- キー入力が不安定なユーザーも、単一キーで誤操作しやすい
- 修飾キー付き（`Ctrl + S` など）や、フォーカス中だけ有効なショートカットはこの問題が小さい

```js
// 失敗例：ページ全体で "s" だけで検索が開く（無効化手段なし）
document.addEventListener('keydown', (event) => {
  if (event.key === 's') openSearch(); // 2.1.4 違反になりやすい
});
```

```js
// 良い例1：修飾キーを必須にする
document.addEventListener('keydown', (event) => {
  if ((event.ctrlKey || event.metaKey) && event.key.toLowerCase() === 'k') {
    event.preventDefault();
    openSearch();
  }
});
```

```js
// 良い例2：対象にフォーカスがあるときだけ単一キーを受け付ける
listbox.addEventListener('keydown', (event) => {
  // listbox にフォーカス中のタイプアヘッドは 2.1.4 の対象外になりやすい
  if (event.key.length === 1) focusOptionStartingWith(event.key);
});
```

> 参照: [G217: Providing a mechanism to allow users to remap or turn off character key shortcuts](https://www.w3.org/WAI/WCAG22/Techniques/general/G217)

## 設計原則

### 1. 通常操作でも同じことができる

- ショートカットが唯一の手段にならない
- 例: `Ctrl + K` で検索を開けるなら、検索ボタンも置く

### 2. 修飾キー付きを基本にする

- `Ctrl` / `Cmd` / `Alt` / `Shift` との組み合わせを優先
- 単一の文字キーは、フォーカス限定か、無効化設定付きにする

### 3. 入力中は発火させない

```js
function isTypingTarget(element) {
  if (!element) return false;
  const tag = element.tagName;
  return (
    tag === 'INPUT' ||
    tag === 'TEXTAREA' ||
    tag === 'SELECT' ||
    element.isContentEditable
  );
}

document.addEventListener('keydown', (event) => {
  if (isTypingTarget(event.target)) return;

  if ((event.ctrlKey || event.metaKey) && event.key.toLowerCase() === 'k') {
    event.preventDefault();
    openSearch();
  }
});
```

- テキスト入力中にグローバルショートカットが暴発すると編集不能になる

### 4. ブラウザ / OS / 支援技術と衝突させない

| 避けたい例 | 理由 |
|---|---|
| `Ctrl + T` / `Ctrl + W` | ブラウザのタブ操作 |
| `Ctrl + L` | アドレスバーへ移動 |
| `Alt` 単独や `Alt + 文字` | OS・ブラウザのメニュー操作と衝突しやすい |
| スクリーンリーダーのブラウズキー | NVDA / JAWS などの単一キー操作と競合 |

- Webアプリ専用のショートカットでも、一般的なブラウザショートカットの上書きは避ける
- 上書きする場合は、ユーザーがオフにできること

### 5. 発見できるようにする

- メニュー項目やボタンの横にショートカット表記を出す
- ヘルプダイアログ（`?` や「キーボードショートカット」ページ）を用意する
- `aria-keyshortcuts` で支援技術にも伝える

### 6. 一貫性と予測可能性

- 同じ操作には同じキーを使う
- プラットフォーム慣習に寄せる（保存は `Ctrl/Cmd + S` など）
- ドキュメントと実装を一致させる

## aria-keyshortcuts

- ショートカットの**存在を支援技術に伝える**属性
- 属性だけでは動作しない。実際のキー処理は JavaScript で実装する
- 値は修飾キー + 非修飾キーを `+` でつなぎ、複数ある場合はスペース区切り

```html
<button
  type="button"
  id="search-button"
  aria-keyshortcuts="Control+K Meta+K"
>
  検索
</button>
```

```js
document.addEventListener('keydown', (event) => {
  const isSearchShortcut =
    (event.ctrlKey || event.metaKey) && event.key.toLowerCase() === 'k';

  if (!isSearchShortcut || isTypingTarget(event.target)) return;

  event.preventDefault();
  document.getElementById('search-button').click();
});
```

> 参照: [MDN — aria-keyshortcuts](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-keyshortcuts)

### accesskey との違い

| 項目 | `accesskey` | `aria-keyshortcuts` + JS |
|---|---|---|
| 動作の実装 | ブラウザが担う | 自前で実装 |
| 修飾キー | OS / ブラウザ依存で予測しにくい | 値に明示できる |
| 衝突 | ユーザーエージェント依存で起きやすい | 設計次第で制御できる |
| 推奨 | 基本的に避ける | カスタムショートカット向き |

## 文字キーショートカットを無効化するUI例

```html
<label>
  <input type="checkbox" id="enable-char-shortcuts" checked>
  文字キーのショートカットを有効にする
</label>
```

```js
let charShortcutsEnabled = true;

document
  .getElementById('enable-char-shortcuts')
  .addEventListener('change', (event) => {
    charShortcutsEnabled = event.target.checked;
  });

document.addEventListener('keydown', (event) => {
  if (!charShortcutsEnabled) return;
  if (event.ctrlKey || event.metaKey || event.altKey) return;
  if (isTypingTarget(event.target)) return;

  if (event.key === 'n') {
    event.preventDefault();
    createNewItem();
  }
});
```

- チェックボックスや設定画面でオン / オフできれば 2.1.4 の Turn off を満たせる
- より良いのは、最初から修飾キー付きにして単一文字キーを避けること

## React での実装例

```jsx
import { useEffect } from 'react';

function useSearchShortcut(onOpen) {
  useEffect(() => {
    const onKeyDown = (event) => {
      const target = event.target;
      const typing =
        target instanceof HTMLElement &&
        (target.tagName === 'INPUT' ||
          target.tagName === 'TEXTAREA' ||
          target.tagName === 'SELECT' ||
          target.isContentEditable);

      if (typing) return;

      if ((event.ctrlKey || event.metaKey) && event.key.toLowerCase() === 'k') {
        event.preventDefault();
        onOpen();
      }
    };

    window.addEventListener('keydown', onKeyDown);
    return () => window.removeEventListener('keydown', onKeyDown);
  }, [onOpen]);
}

function SearchButton({ onOpen }) {
  useSearchShortcut(onOpen);

  return (
    <button
      type="button"
      onClick={onOpen}
      aria-keyshortcuts="Control+K Meta+K"
    >
      検索
      <kbd>Ctrl</kbd>+<kbd>K</kbd>
    </button>
  );
}
```

## コンポーネント内ショートカット（フォーカス限定）

- リストボックスのタイプアヘッド、メニュー内の文字ジャンプは、フォーカス中だけ有効
- これらは「そのコンポーネントのキーボードインタラクション」であり、ページ全体の文字キーショートカットとは区別する
- それでも、入力欄にフォーカスがあるときは競合しないよう注意する

## よくある間違い

### 1. ショートカットしか手段がない

```js
// 間違い：UI上に同等の操作がない
window.addEventListener('keydown', (e) => {
  if (e.key === 'Delete') deleteSelected();
});
```

- 削除ボタンなど、ポインタ / 通常キーボードでも使える手段を用意する

### 2. 単一文字キーをグローバルに有効化したまま

```js
// 間違い：無効化・再割当なし
if (event.key === 'g') goToDashboard();
```

### 3. 入力中でもショートカットが発火する

- フォーム入力やチャット入力中にページ遷移や削除が走る

### 4. aria-keyshortcuts だけ付けて実装しない / 実装だけして伝えない

- 属性とイベントハンドラはセット
- どちらか片方では不十分

### 5. accesskey に頼る

- 修飾キーが環境ごとに違い、発見も難しい
- カスタム実装 + 明示的な表記の方が制御しやすい

### 6. ブラウザ標準ショートカットを無断で上書きする

- `Ctrl + S` を独自保存に使う場合でも、ユーザー期待と衝突しうる
- 使うならヘルプで明示し、可能なら無効化できるようにする

## 実装チェックリスト

- [ ] ショートカットでできる操作は、UI上の通常操作でもできる（2.1.1）
- [ ] 文字キーのみのショートカットは、無効化・再割当・フォーカス限定のいずれか（2.1.4）
- [ ] 入力中（`input` / `textarea` / `contenteditable`）はグローバルショートカットを止める
- [ ] ブラウザ / OS / スクリーンリーダーと明らかに衝突するキーを避ける
- [ ] 画面上かヘルプでショートカットを発見できる
- [ ] 実装と `aria-keyshortcuts` の表記が一致している
- [ ] `preventDefault()` は必要なキーだけに限定している
- [ ] 音声入力やキーボードのみで誤発火しないか確認した

## まとめ

- ショートカットはキーボード操作の代替ではなく、効率のための追加機能
- WCAG 2.1.4 では、文字キーだけのショートカットに無効化・再割当・フォーカス限定が必要
- 設計の基本は「修飾キー付き」「入力中は無効」「発見可能」「衝突を避ける」
- `aria-keyshortcuts` は伝達用。動作は JavaScript で実装する
- 迷ったら単一文字のグローバルショートカットをやめて、修飾キー付きにする

> 参照: [F99: Failure of Success Criterion 2.1.4](https://www.w3.org/WAI/WCAG22/Techniques/failures/F99)
