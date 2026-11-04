# inert属性の使い方 — モーダル背後を一括で無効にする

## inertとは

- `inert` は HTML のグローバル属性（Boolean）で、付けた要素とその子孫をまとめて「操作不能」にする
- 対象サブツリーでは、次が同時に無効になる
  - クリック・タップなどのポインター操作
  - キーボードフォーカス（Tab 順からも外れる）
  - テキスト選択・編集（`contenteditable` 含む）
  - ブラウザのページ内検索（Cmd/Ctrl + F）でのヒット
  - 支援技術への露出（アクセシビリティツリーから除外）
- モーダル・ドロワー・オーバーレイの「背後コンテンツ」を封じる用途が典型
- `aria-hidden="true"` だけだとフォーカスやクリックは残る — `inert` はその抜け穴を属性1つで塞ぐ
- 2023年4月以降、主要ブラウザで利用可能（Baseline Widely available）

> 参照: [MDN — inert](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inert) / [HTML Living Standard — The inert attribute](https://html.spec.whatwg.org/multipage/interaction.html#the-inert-attribute)

## 何が起きるか

`inert` を付けた要素とその flat tree 子孫は、ユーザー操作の対象から外れる。

```html
<div>
  <button>操作できる</button>
</div>

<div inert>
  <button>操作できない</button>
  <a href="/about">フォーカスもクリックもできない</a>
</div>
```

- 見た目はデフォルトでは変わらない — 視覚的な無効化は別途スタイルが必要
- 個々のフォーム部品を無効にしたいだけなら `disabled` のほうが適切
- `inert` は「セクション単位」で使う前提の属性

| 手段 | フォーカス阻止 | クリック阻止 | アクセシビリティツリーから除外 | 想定スコープ |
|---|---|---|---|---|
| `inert` | ○ | ○ | ○ | セクション・背後コンテンツ |
| `aria-hidden="true"` | × | × | ○ | 装飾・重複の読み上げ抑制 |
| `disabled` | ○（その部品） | ○ | ×（発見はされる） | 単一のフォームコントロール |
| `hidden` / `display: none` | ○ | ○ | ○ | そもそも出さないコンテンツ |
| `pointer-events: none` | × | ○（マウスのみ） | × | 見た目だけのクリック抑制 |

## 主な使い方 — モーダルの背後

カスタムモーダルでは、開いている間だけ背景に `inert` を付ける。

```html
<!-- 悪い例：aria-hidden だけでは Tab で背景のリンクに届く -->
<main aria-hidden="true">
  <a href="/">ホーム</a>
  <button>検索</button>
</main>
<div role="dialog" aria-modal="true" aria-labelledby="dlg-title">
  <h2 id="dlg-title">確認</h2>
  <button>閉じる</button>
</div>
```

```html
<!-- 良い例：背景を inert にし、操作も読み上げも封じる -->
<main inert>
  <a href="/">ホーム</a>
  <button>検索</button>
</main>
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="dlg-title"
>
  <h2 id="dlg-title">確認</h2>
  <button type="button">閉じる</button>
</div>
```

```js
const main = document.querySelector('main');
const dialog = document.querySelector('[role="dialog"]');
const openBtn = document.querySelector('#open-dialog');
const closeBtn = dialog.querySelector('button');

function openDialog() {
  main.inert = true;
  dialog.hidden = false;
  closeBtn.focus(); // 開いたらダイアログ内へフォーカスを移す
}

function closeDialog() {
  main.inert = false;
  dialog.hidden = true;
  openBtn.focus(); // 閉じたらトリガーへ戻す
}
```

- 開く前にフォーカスをダイアログ内へ移す
- 閉じたらトリガーへ戻す
- フォーカスがある領域に後から `inert` を付けない — 先にフォーカスを逃がす

## `<dialog>` との関係

- `<dialog>` を `showModal()` で開くと、ブラウザが背後を自動的に inert にする
- `show()` や `open` 属性だけの非モーダルでは背後は inert にならない

```html
<dialog id="confirm">
  <h2>削除しますか？</h2>
  <form method="dialog">
    <button value="cancel">キャンセル</button>
    <button value="ok">削除</button>
  </form>
</dialog>
```

```js
const dialog = document.getElementById('confirm');

// 良い例：モーダルとして開き、背後はブラウザが inert にする
dialog.showModal();

// 悪い例：非モーダル — 背後の操作が残る
// dialog.show();
```

- 自前オーバーレイを作るより、まず `<dialog>` + `showModal()` を検討する
- カスタム実装が必要なときだけ、明示的に `inert` を付ける

> 参照: [MDN — `<dialog>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog)

## 見た目のフィードバック

- `inert` 自体にデフォルトの見た目はない
- 仕様も「視覚的にobscureされている内容に使う」ことを推奨している
- モーダルの backdrop・オーバーレイ・オフスクリーン配置など、文脈に合った視覚的な遮断を併用する

```css
/* 悪い例：inert だけ付けて見た目が変わらない */
main[inert] {
  /* 何もない */
}

/* 良い例：背後が無効だと分かるようにする（モーダル時など） */
main[inert] {
  opacity: 0.5;
  pointer-events: none; /* 補助。本質的な無効化は inert 側 */
}

/* ダイアログ本体は inert の外に置く */
```

- オフスクリーンのナビパネルなど、もともと見えない領域なら追加スタイルは不要なことが多い
- 「操作できないのに普通に見える」状態は避ける

## JavaScript での切り替え

- 属性として書くほか、`HTMLElement.inert` プロパティでも切り替えられる

```js
element.inert = true;  // <div inert> と同じ
element.inert = false; // 解除
```

- 開閉アニメーション中も、操作不能にしたい区間は `inert` を付けたままにする
- 閉じ完了後に解除する — 途中で背後へフォーカスが漏れるのを防ぐ

## よくある間違い

### 1. `aria-hidden` だけで背後を封じようとする

```html
<!-- 間違い：読み上げは隠れるが、Tab で背景ボタンに届く -->
<main aria-hidden="true">...</main>
<div role="dialog">...</div>
```

```html
<!-- 正解：inert で操作も読み上げも封じる -->
<main inert>...</main>
<div role="dialog" aria-modal="true">...</div>
```

- `aria-hidden="true"` 配下にフォーカス可能要素を残すのはアンチパターン
- 背後の無効化には `inert`（または `<dialog showModal()>`）を使う

### 2. 単一コントロールに `inert` を付ける

```html
<!-- 間違い：ボタン1つを inert にする -->
<button inert>送信</button>
```

```html
<!-- 正解：単体なら disabled -->
<button type="submit" disabled>送信</button>
```

- 個別コントロールは `disabled` + `:disabled` スタイル
- `inert` はセクション単位の無効化用

### 3. フォーカスがあるまま領域を inert にする

```js
/* 間違い：フォーカス中の要素を含む領域をいきなり inert にする */
main.inert = true;
```

```js
/* 正解：先にダイアログ内へフォーカスを移してから背後を inert にする */
dialog.querySelector('button').focus();
main.inert = true;
```

- フォーカスが inert な領域に残ると、キーボード操作が迷子になる
- 開く・閉じるの両方でフォーカス移動を設計する

### 4. ダイアログ本体まで inert にする

```html
<!-- 間違い：ページ全体を inert にしてダイアログも操作不能 -->
<body inert>
  <main>...</main>
  <div role="dialog">...</div>
</body>
```

```html
<!-- 正解：背後だけ inert。ダイアログは inert の外 -->
<main inert>...</main>
<div role="dialog">...</div>
```

- inert は子孫にも波及する — 操作させたい UI を内側に置かない
- モーダル本体は inert 付き要素の兄弟として置く

### 5. 視覚的な遮断を忘れる

```css
/* 間違い：操作不能なのに見た目が通常どおり */
```

```css
/* 正解：オーバーレイや opacity などで背後が無効だと分かるようにする */
.modal-backdrop {
  background: rgb(0 0 0 / 0.5);
}
```

- スクリーンリーダー以外のユーザーにも「今は背後を触れない」と伝える
- `<dialog>` のモーダル backdrop はこの役割を標準で担う

## まとめ

- `inert` はサブツリーを操作不能にし、アクセシビリティツリーからも外す
- モーダル・ドロワーなどの背後コンテンツ封じが主な用途
- `aria-hidden` だけではフォーカスとクリックが残る — 背後無効化には不十分
- 単一コントロールには `disabled`、セクションには `inert`
- 可能なら `<dialog>` の `showModal()` で背後 inert をブラウザに任せる
- 見た目のフィードバック（オーバーレイなど）を併用する
- 付ける前後でフォーカスを適切に移し、ダイアログ本体を inert の内側に置かない

> 参照: [MDN — inert](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inert) / [HTML Living Standard — Inert subtrees](https://html.spec.whatwg.org/multipage/interaction.html#inert-subtrees)
