# アクセシブルなアコーディオンの実装

## アコーディオンとは

- 見出しが縦に並び、それぞれが対応するコンテンツの表示 / 非表示を切り替えるUI
- FAQ、設定画面、長いフォームのセクション分割などに使われる
- スクロール量を減らしつつ、複数セクションを1ページにまとめられる
- タブUIと異なり、複数パネルを同時に開ける実装が一般的
- パネルはドキュメント順に並び、ページ内の見出し構造の一部になる

> 参照: [WAI-ARIA Authoring Practices — Accordion Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/accordion/)

## アコーディオン vs タブ vs `<details>`

| UI | 特徴 | 使いどころ |
|---|---|---|
| アコーディオン | 見出し + 開閉パネル。複数同時オープン可 | FAQ、長いコンテンツの折りたたみ |
| タブ | 1つだけ表示。矢印キーで切替 | 関連するビューの切替 |
| `<details>` / `<summary>` | ネイティブの開閉。1セクション向き | 単純な開示UI、追加のキーボード操作が不要な場合 |

- 単純な開閉だけでよいなら、まず `<details>` / `<summary>` を検討する（ARIAファーストルール）
- 複数セクションをまとめて制御する・開閉状態をアプリ側で厳密に管理するならアコーディオンパターン

## 必要なARIA属性と構造

| 要素 | ロール / 属性 | 説明 |
|---|---|---|
| 各ヘッダーのラッパー | `h2`〜`h6`（または `role="heading"` + `aria-level`） | ページの見出し階層に合わせる |
| 開閉コントロール | `<button>` | ヘッダー内に置く。開閉の操作対象 |
| 展開状態 | `aria-expanded="true"` / `"false"` | パネルが開いているかどうか |
| ボタン → パネル | `aria-controls="パネルのid"` | 制御対象パネルとの紐付け |
| 展開中パネル（任意） | `role="region"` + `aria-labelledby` | ランドマークとして識別。名前は必須 |
| 閉じられないパネル | `aria-disabled="true"` | 常に1つ開いている実装で、閉じ操作を禁止する場合 |

- `role="accordion"` は**存在しない** — 付けない
- ヘッダーは「見出しの中にボタン」がAPGの推奨構造
- 見出しレベルはページ全体のアウトラインに合わせて選ぶ（例：`h2`の下なら`h3`）

> 参照: [APG Accordion Example](https://www.w3.org/WAI/ARIA/apg/patterns/accordion/examples/accordion/)

## キーボード操作の要件

| キー | 動作 |
|---|---|
| `Enter` / `Space` | フォーカス中のヘッダーに対応するパネルを開閉 |
| `Tab` | 次のフォーカス可能要素へ（ヘッダー・パネル内のリンク等すべて含む） |
| `Shift` + `Tab` | 前のフォーカス可能要素へ |

- アコーディオン内のフォーカス可能要素は、すべて通常のTab順序に含める
- タブUIのようなロービングタブインデックスは**不要**
- かつては矢印キー / `Home` / `End` が任意推奨だったが、現行APGでは推奨から外れる方向 — 必須ではない

### 開閉のバリエーション

- **複数同時オープン（推奨の基本形）**: 各パネルを独立して開閉できる
- **単一オープン**: 1つ開くと他は閉じる。FAQなどでよく使われる
- **常に1つ以上オープン**: 最後の1つは閉じられない（そのヘッダーに `aria-disabled="true"`）

## HTML構造

```html
<div class="accordion">
  <h3>
    <button
      type="button"
      id="accordion-btn-1"
      aria-expanded="true"
      aria-controls="accordion-panel-1"
    >
      配送について
    </button>
  </h3>
  <div
    id="accordion-panel-1"
    role="region"
    aria-labelledby="accordion-btn-1"
  >
    <p>通常2〜3営業日でお届けします。</p>
  </div>

  <h3>
    <button
      type="button"
      id="accordion-btn-2"
      aria-expanded="false"
      aria-controls="accordion-panel-2"
    >
      返品について
    </button>
  </h3>
  <div
    id="accordion-panel-2"
    role="region"
    aria-labelledby="accordion-btn-2"
    hidden
  >
    <p>到着後14日以内であれば返品できます。</p>
  </div>

  <h3>
    <button
      type="button"
      id="accordion-btn-3"
      aria-expanded="false"
      aria-controls="accordion-panel-3"
    >
      お支払い方法
    </button>
  </h3>
  <div
    id="accordion-panel-3"
    role="region"
    aria-labelledby="accordion-btn-3"
    hidden
  >
    <p>クレジットカード、コンビニ払い、銀行振込に対応しています。</p>
  </div>
</div>
```

### 構造のポイント

- `<button type="button">` を使う — `<div onclick>` や `<a href="#">` は避ける
- 閉じたパネルは `hidden` 属性で非表示にする（視覚的に隠すだけだとスクリーンリーダーに読まれる）
- `role="region"` は展開パネルに付ける任意のランドマーク。付けるなら `aria-labelledby` で名前を必ず付ける
- 開閉アイコンは装飾なら `aria-hidden="true"`。状態は `aria-expanded` で伝える

## Vanilla JS での実装

```js
class AccessibleAccordion {
  /**
   * @param {HTMLElement} container
   * @param {{ allowMultiple?: boolean }} options
   */
  constructor(container, { allowMultiple = true } = {}) {
    this.container = container;
    this.allowMultiple = allowMultiple;
    this.buttons = [...container.querySelectorAll('h3 > button[aria-controls]')];
    this.init();
  }

  init() {
    this.buttons.forEach((button) => {
      button.addEventListener('click', () => {
        this.toggle(button);
      });
    });
  }

  toggle(button) {
    const isExpanded = button.getAttribute('aria-expanded') === 'true';
    const panel = document.getElementById(button.getAttribute('aria-controls'));

    if (!panel) return;

    // 単一オープン: 他を閉じてから対象を開く
    if (!this.allowMultiple && !isExpanded) {
      this.buttons.forEach((other) => {
        if (other === button) return;
        other.setAttribute('aria-expanded', 'false');
        const otherPanel = document.getElementById(
          other.getAttribute('aria-controls')
        );
        if (otherPanel) otherPanel.hidden = true;
      });
    }

    button.setAttribute('aria-expanded', String(!isExpanded));
    panel.hidden = isExpanded;
  }
}

new AccessibleAccordion(document.querySelector('.accordion'), {
  allowMultiple: true,
});
```

- `<button>` は Enter / Space をネイティブに発火するため、クリックハンドラだけでキーボード操作が満たせる
- `keydown` で Enter / Space を自前実装する必要はない（ボタンを使っている場合）

## React での実装例

```jsx
import { useId, useState } from 'react';

function Accordion({ items, allowMultiple = true }) {
  const baseId = useId();
  const [openIds, setOpenIds] = useState(() =>
    items.filter((item) => item.defaultOpen).map((item) => item.id)
  );

  const isOpen = (id) => openIds.includes(id);

  const toggle = (id) => {
    setOpenIds((prev) => {
      const currentlyOpen = prev.includes(id);

      if (allowMultiple) {
        return currentlyOpen
          ? prev.filter((openId) => openId !== id)
          : [...prev, id];
      }

      return currentlyOpen ? [] : [id];
    });
  };

  return (
    <div className="accordion">
      {items.map((item) => {
        const buttonId = `${baseId}-btn-${item.id}`;
        const panelId = `${baseId}-panel-${item.id}`;
        const expanded = isOpen(item.id);

        return (
          <div key={item.id}>
            <h3>
              <button
                type="button"
                id={buttonId}
                aria-expanded={expanded}
                aria-controls={panelId}
                onClick={() => toggle(item.id)}
              >
                {item.title}
              </button>
            </h3>
            <div
              id={panelId}
              role="region"
              aria-labelledby={buttonId}
              hidden={!expanded}
            >
              {item.content}
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

## `<details>` / `<summary>` で足りる場合

- ネイティブ要素だけで開閉できる
- 追加のARIAやキーボード処理がほぼ不要
- 複数をまとめて「1つだけ開く」などの制御が難しい
- 見出し階層や見た目のカスタマイズはCSSで対応可能だが、ブラウザ差に注意

```html
<!-- 良い例：単純なFAQの1項目 -->
<details>
  <summary>配送について</summary>
  <p>通常2〜3営業日でお届けします。</p>
</details>
```

```html
<!-- アコーディオンパターンが向く例：状態をJSで一括管理したい -->
<!-- 単一オープン、初期展開、アニメーション連携など -->
```

> 参照: [MDN — `<details>`: The Details disclosure element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details)

## スタイルの注意点

```css
.accordion button {
  width: 100%;
  text-align: left;
  /* フォーカスが見えるようにする */
}

.accordion button:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* 開閉アイコンは装飾。状態は aria-expanded で伝える */
.accordion button::after {
  content: "+";
}

.accordion button[aria-expanded="true"]::after {
  content: "−";
}

/* prefers-reduced-motion への配慮 */
@media (prefers-reduced-motion: reduce) {
  .accordion [role="region"] {
    transition: none;
  }
}
```

- `outline: none` だけにしてフォーカスインジケーターを消さない
- アニメーションで高さを変える場合も、閉じた状態は最終的に `hidden` または `display: none` にする

## よくある間違い

### 1. ボタンではなくクリック可能な見出しだけにする

```html
<!-- 間違い：キーボードで操作できない / ロールが不明 -->
<h3 class="accordion-header" onclick="toggle()">配送について</h3>

<!-- 正解：見出しの中に button を置く -->
<h3>
  <button type="button" aria-expanded="false" aria-controls="panel-1">
    配送について
  </button>
</h3>
```

### 2. aria-expanded の更新忘れ

```js
// 間違い：見た目だけ切り替えて状態を伝えない
button.classList.toggle('is-open');
panel.hidden = !panel.hidden;

// 正解：aria-expanded と hidden をセットで更新
const next = button.getAttribute('aria-expanded') !== 'true';
button.setAttribute('aria-expanded', String(next));
panel.hidden = !next;
```

### 3. 閉じたパネルを視覚的に隠すだけにする

```css
/* 間違い：スクリーンリーダーが内容を読み上げる */
.accordion-panel.is-closed {
  height: 0;
  overflow: hidden;
  opacity: 0;
}
```

```html
<!-- 正解 -->
<div id="panel-1" hidden>...</div>
```

### 4. role="accordion" や不適切なロールを付ける

```html
<!-- 間違い：存在しない / 不適切なロール -->
<div role="accordion">
  <div role="tab">...</div>
</div>

<!-- 正解：見出し + button + パネル。専用の accordion ロールは不要 -->
```

### 5. 見出しレベルを無視する

```html
<!-- 間違い：ページ構造と無関係に全部 h2 -->
<h2><button>...</button></h2>

<!-- 正解：親見出しの階層に合わせる（例：セクションが h2 なら項目は h3） -->
<section>
  <h2>よくある質問</h2>
  <h3><button type="button" aria-expanded="false" aria-controls="q1">Q1</button></h3>
</section>
```

### 6. アコーディオンをタブUIと同じキーボードにしてしまう

- アコーディオンは Tab で各ヘッダーを順に移動するのが基本
- 矢印キーだけでヘッダー間を移動するロービングは、タブUI向けのパターン
- 混同すると期待する操作と食い違う

## 実装チェックリスト

- [ ] 各ヘッダーが適切な見出しレベルでマークアップされている
- [ ] 開閉コントロールが `<button type="button">` である
- [ ] `aria-expanded` が開閉状態と一致して更新される
- [ ] `aria-controls` でパネルIDを参照している
- [ ] 閉じたパネルに `hidden`（または同等の非表示）が付いている
- [ ] Enter / Space / クリックで開閉できる
- [ ] Tab でヘッダーとパネル内のフォーカス可能要素を移動できる
- [ ] フォーカスインジケーターが見える
- [ ] （任意）展開パネルに `role="region"` + `aria-labelledby` がある
- [ ] （単一オープン時）他パネルが正しく閉じる

## まとめ

- アコーディオンの基本は「見出しの中のボタン + `aria-expanded` + パネルの表示切替」
- Tab 順序にすべてのフォーカス可能要素を含め、Enter / Space で開閉する
- 単純な開示なら `<details>` / `<summary>` を優先し、複雑な制御が必要ならAPGパターンを使う
- タブUIのロールやロービングタブインデックスを流用しない
- `aria-expanded` と `hidden` の同期を忘れない
