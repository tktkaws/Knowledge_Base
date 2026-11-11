# アクセシブルなファイル選択の実装 — 選んだファイルと失敗を伝える

## ファイル選択UIとは

ファイル選択UIは、端末にある画像や文書などをWebサイトへ渡すためのフォームコントロール。プロフィール画像、申請書、メールの添付ファイルなどに使われる。

- 何のファイルを選ぶかをラベルで伝える
- 選べる形式、最大容量、最大件数を選択前に示す
- 選んだファイル名、容量、件数を確認できるようにする
- 不正なファイルと修正方法をテキストで伝える
- 選んだファイルを個別に取り除けるようにする
- アップロード中、成功、失敗、キャンセルを区別する
- ドラッグ＆ドロップ以外の操作方法を必ず残す

通常はネイティブの`input type="file"`を使う。OSのファイル選択画面、キーボード操作、支援技術との連携をブラウザへ任せられるため、見た目だけの`div`で置き換えない。

> 参照:
> - [WHATWG HTML Standard — File Upload state](https://html.spec.whatwg.org/multipage/input.html#file-upload-state-(type=file))
> - [W3C WAI — Labeling Controls](https://www.w3.org/WAI/tutorials/forms/labels/)

## 基本のHTML

```html
<form action="/profile/photo" method="post" enctype="multipart/form-data">
  <div class="file-field">
    <label for="profile-photo">プロフィール画像</label>
    <p id="profile-photo-help">
      JPEGまたはPNG、5 MB以下のファイルを1件選択してください。
    </p>
    <input
      id="profile-photo"
      name="profilePhoto"
      type="file"
      accept=".jpg,.jpeg,.png,image/jpeg,image/png"
      aria-describedby="profile-photo-help"
      required
    />
  </div>

  <button type="submit">画像をアップロード</button>
</form>
```

`form`でファイルを送る場合は`enctype="multipart/form-data"`を指定する。

ラベルは「ファイル」だけでなく、「プロフィール画像」「本人確認書類」のように目的を含める。入力条件は選択後のエラーだけにせず、入力の前に見えるテキストで示す。

## `input type="file"`の主な属性

| 属性 | 役割 |
|---|---|
| `accept` | ファイル選択画面へ受け付ける種類のヒントを渡す |
| `multiple` | 複数ファイルの選択を許可する |
| `required` | ファイルが1件以上必要であることを示す |
| `name` | フォーム送信時のフィールド名 |
| `capture` | 対応端末でカメラやマイクからの取得を提案する |

`value`へ端末上のパスを設定して、あらかじめファイルを選択しておくことはできない。利用者自身が選ぶ必要がある。

## `accept`は検証ではない

```html
<input
  type="file"
  accept=".pdf,application/pdf"
/>
```

`accept`はファイル選択画面を絞り込むためのヒント。環境によっては別形式も選べるため、次の確認が別に必要になる。

- ブラウザ上で早めに検証し、具体的なエラーを示す
- サーバー側で形式、容量、件数を必ず再検証する
- 拡張子やブラウザから渡されたMIMEタイプだけを信用しない
- 保存時のファイル名をサーバー側で安全に扱う
- 必要に応じてマルウェア検査や画像の再エンコードを行う

クライアント側の検証は利用者への早いフィードバックであり、セキュリティ境界ではない。

### MIMEタイプと拡張子を併記する

```html
<input
  type="file"
  accept=".doc,.docx,application/msword,application/vnd.openxmlformats-officedocument.wordprocessingml.document"
/>
```

同じ形式でも端末やアプリによってMIMEタイプが異なる場合がある。受け付ける拡張子とMIMEタイプを仕様に合わせて列挙する。

### ワイルドカードを使う

```html
<input type="file" accept="image/*" />
```

`image/*`は広い種類の画像を選択対象にする。サーバーがJPEGとPNGだけを処理できるなら、`image/*`ではなく対応形式を具体的に指定する。

## 複数ファイルを選ぶ

```html
<label for="attachments">添付ファイル</label>
<p id="attachments-help">
  PDFまたは画像を3件まで選択できます。1件につき5 MBまでです。
</p>
<input
  id="attachments"
  name="attachments[]"
  type="file"
  accept=".pdf,.jpg,.jpeg,.png,application/pdf,image/jpeg,image/png"
  aria-describedby="attachments-help"
  multiple
/>
```

`multiple`は複数選択を許可するだけで、最大件数は制限しない。最大件数は説明し、JavaScriptとサーバーの両方で検証する。

ファイル選択画面での複数選択方法はOSによって異なる。「Shiftキーを押す」のような特定環境だけの説明を主な案内にしない。

## 選択結果をテキストで示す

ブラウザのファイル入力には選択結果が表示されるが、複数ファイルの全件、容量、個別エラーを十分に確認できない場合がある。別の一覧でも示す。

```html
<form id="documents-form" action="/documents" method="post" enctype="multipart/form-data">
  <div class="file-field">
    <label for="documents">申請書類</label>
    <p id="documents-help">
      PDF、JPEG、PNGを3件まで選択できます。1件につき5 MBまでです。
    </p>

    <input
      id="documents"
      name="documents[]"
      type="file"
      accept=".pdf,.jpg,.jpeg,.png,application/pdf,image/jpeg,image/png"
      aria-describedby="documents-help"
      multiple
    />

    <div id="documents-errors" class="file-errors" hidden>
      <p>追加できないファイルがあります。</p>
      <ul id="documents-error-list"></ul>
    </div>

    <p id="documents-status" role="status" aria-atomic="true"></p>
    <ul id="documents-list" class="file-list" aria-label="選択したファイル"></ul>
  </div>

  <button type="submit">選択したファイルをアップロード</button>
</form>
```

選択済み一覧は`ul`で構造化する。ファイル名だけでなく、判断に必要な容量とエラーも表示する。

`role="status"`には「3件選択しました」「1件削除しました」のような短い要約だけを送る。ファイル一覧全体をライブリージョンにすると、更新のたびに多くの情報が読み上げられる可能性がある。

## 形式、容量、件数を検証する

```js
const fileInput = document.getElementById('documents');
const fileList = document.getElementById('documents-list');
const errorBox = document.getElementById('documents-errors');
const errorList = document.getElementById('documents-error-list');
const status = document.getElementById('documents-status');

const MAX_FILES = 3;
const MAX_FILE_SIZE = 5 * 1024 * 1024;
const ALLOWED_MIME_TYPES = new Set([
  'application/pdf',
  'image/jpeg',
  'image/png',
]);
const ALLOWED_EXTENSIONS = new Set(['pdf', 'jpg', 'jpeg', 'png']);

let selectedFiles = [];

function formatBytes(bytes) {
  if (bytes < 1024) return `${bytes} B`;
  if (bytes < 1024 ** 2) return `${(bytes / 1024).toFixed(1)} KB`;
  return `${(bytes / 1024 ** 2).toFixed(1)} MB`;
}

function getExtension(filename) {
  const parts = filename.toLowerCase().split('.');
  return parts.length > 1 ? parts.pop() : '';
}

function isAllowedType(file) {
  return (
    ALLOWED_MIME_TYPES.has(file.type) ||
    ALLOWED_EXTENSIONS.has(getExtension(file.name))
  );
}

function validateFiles(files) {
  const errors = [];
  const seen = new Set();

  files.forEach((file, index) => {
    const messages = [];
    const key = `${file.name}:${file.size}:${file.lastModified}`;

    if (index >= MAX_FILES) {
      messages.push(`最大${MAX_FILES}件までです。`);
    }

    if (!isAllowedType(file)) {
      messages.push('PDF、JPEG、PNGのいずれかを選択してください。');
    }

    if (file.size > MAX_FILE_SIZE) {
      messages.push('ファイル容量を5 MB以下にしてください。');
    }

    if (seen.has(key)) {
      messages.push('同じファイルが重複しています。');
    }

    seen.add(key);

    if (messages.length > 0) {
      errors.push({ index, file, messages });
    }
  });

  return errors;
}

function renderErrors(errors) {
  errorList.replaceChildren();

  errors.forEach(({ file, messages }) => {
    messages.forEach((message) => {
      const item = document.createElement('li');
      item.textContent = `${file.name}: ${message}`;
      errorList.append(item);
    });
  });

  const hasErrors = errors.length > 0;
  errorBox.hidden = !hasErrors;

  if (hasErrors) {
    fileInput.setAttribute('aria-invalid', 'true');
    fileInput.setAttribute(
      'aria-describedby',
      'documents-help documents-errors',
    );
  } else {
    fileInput.removeAttribute('aria-invalid');
    fileInput.setAttribute('aria-describedby', 'documents-help');
  }
}

function renderFiles(errors) {
  const errorsByIndex = new Map(
    errors.map((error) => [error.index, error.messages]),
  );

  fileList.replaceChildren();

  selectedFiles.forEach((file, index) => {
    const item = document.createElement('li');
    const information = document.createElement('span');
    const removeButton = document.createElement('button');

    information.textContent = `${file.name}（${formatBytes(file.size)}）`;
    removeButton.type = 'button';
    removeButton.dataset.removeIndex = String(index);
    removeButton.textContent = `${file.name}を削除`;

    item.append(information);

    const messages = errorsByIndex.get(index) ?? [];
    messages.forEach((message) => {
      const error = document.createElement('p');
      error.className = 'field-error';
      error.textContent = message;
      item.append(error);
    });

    item.append(removeButton);
    fileList.append(item);
  });
}

function updateFileView(message) {
  const errors = validateFiles(selectedFiles);
  renderFiles(errors);
  renderErrors(errors);
  status.textContent = message;
  return errors;
}

fileInput.addEventListener('change', () => {
  selectedFiles = [...fileInput.files];
  const errors = updateFileView(`${selectedFiles.length}件選択しました。`);

  if (errors.length > 0) {
    status.textContent += `${errors.length}件に修正が必要です。`;
  }
});
```

ファイル名を`innerHTML`へ入れず、`textContent`で表示する。ファイル名は利用者が制御できる文字列として扱う。

ブラウザから得た`file.type`が空になる場合もあるため、このクライアント側の例では拡張子も確認している。ただし、どちらも偽装できる。最終判断はサーバーでファイル内容を検査して行う。

## 選んだファイルを個別に削除する

`FileList`を配列のように直接変更することはできない。残すファイルから新しい`FileList`を作り、入力へ同期する。

```js
function syncNativeInput() {
  const transfer = new DataTransfer();

  selectedFiles.forEach((file) => {
    transfer.items.add(file);
  });

  fileInput.files = transfer.files;
}

fileList.addEventListener('click', (event) => {
  const button = event.target.closest('[data-remove-index]');
  if (!button) return;

  const index = Number(button.dataset.removeIndex);
  if (!Number.isInteger(index) || !selectedFiles[index]) return;

  const removedName = selectedFiles[index].name;
  selectedFiles.splice(index, 1);
  syncNativeInput();
  updateFileView(`${removedName}を削除しました。`);

  const remainingButtons = fileList.querySelectorAll('[data-remove-index]');
  const nextButton = remainingButtons[Math.min(index, remainingButtons.length - 1)];

  if (nextButton) {
    nextButton.focus();
  } else {
    fileInput.focus();
  }
});
```

削除後に押したボタンがDOMから消えるため、フォーカスを残った削除ボタンまたはファイル入力へ移す。何を削除したかは`role="status"`で通知する。

「すべて削除」ボタンを用意する場合も、ファイル入力の値、表示一覧、エラー、アップロード対象をまとめて空にする。

```js
function clearAllFiles() {
  selectedFiles = [];
  fileInput.value = '';
  updateFileView('選択したファイルをすべて削除しました。');
  fileInput.focus();
}
```

## 送信時のエラー

選択直後の検証に加え、フォーム送信時にも現在の状態を確認する。

```js
const uploadForm = document.getElementById('documents-form');

uploadForm.addEventListener('submit', (event) => {
  const errors = validateFiles(selectedFiles);

  if (selectedFiles.length === 0) {
    event.preventDefault();
    fileInput.setAttribute('aria-invalid', 'true');
    fileInput.setAttribute(
      'aria-describedby',
      'documents-help documents-errors',
    );
    errorBox.hidden = false;
    errorList.replaceChildren();

    const item = document.createElement('li');
    item.textContent = 'ファイルを1件以上選択してください。';
    errorList.append(item);

    status.textContent = 'ファイルが選択されていません。';
    fileInput.focus();
    return;
  }

  if (errors.length > 0) {
    event.preventDefault();
    renderErrors(errors);
    status.textContent = '追加できないファイルがあります。内容を確認してください。';
    fileInput.focus();
  }
});
```

エラーでは次を伝える。

- 対象のファイル名
- 何が条件に合わないか
- 許可される形式、容量、件数
- 選び直す、削除するなどの修正方法

```text
避ける: ファイルが無効です。

適切: report.zipは追加できません。PDF、JPEG、PNGのいずれかを選択してください。
```

色や警告アイコンだけで失敗を示さない。入力へ`aria-invalid="true"`を付け、エラーテキストを`aria-describedby`で関連付ける。

> 参照: [W3C WAI — Validating Input](https://www.w3.org/WAI/tutorials/forms/validation/)

## 同じファイルを選び直す

`change`イベントは、同じファイルをもう一度選んだときに発火しない場合がある。失敗したファイルを一覧から削除して入力の`FileList`へ同期すれば、同じ名前の修正版を改めて選択できる。

ファイル入力の`click`イベントごとに値を空にすると、選択画面をキャンセルしただけでも表示一覧と入力値が食い違う可能性がある。既存の選択を置き換える場合は、明示的な「すべて削除」または「ファイルを選び直す」操作として状態をまとめてリセットする。

## ドラッグ＆ドロップを追加する

ドラッグ＆ドロップは補助的な入力方法として追加する。ドラッグできない利用者のために、同じ領域内へネイティブのファイル入力を残す。

```html
<div id="drop-zone" class="drop-zone">
  <p>ここへファイルをドロップするか、ファイル選択を使用してください。</p>

  <label for="drop-files">申請書類</label>
  <input
    id="drop-files"
    type="file"
    name="documents[]"
    accept=".pdf,.jpg,.jpeg,.png,application/pdf,image/jpeg,image/png"
    multiple
  />
</div>
```

ドロップ領域へ`role="button"`や`tabindex="0"`を付けて、独自のファイル入力にしない。領域内のネイティブ入力がキーボード、音声入力、タッチ操作の代替になる。

```js
const dropZone = document.getElementById('drop-zone');
const dropInput = document.getElementById('drop-files');

function setDroppedFiles(files) {
  const transfer = new DataTransfer();

  files.forEach((file) => {
    transfer.items.add(file);
  });

  dropInput.files = transfer.files;
  dropInput.dispatchEvent(new Event('change', { bubbles: true }));
}

dropZone.addEventListener('dragover', (event) => {
  event.preventDefault();
  dropZone.classList.add('drop-zone--active');
});

dropZone.addEventListener('dragleave', () => {
  dropZone.classList.remove('drop-zone--active');
});

dropZone.addEventListener('drop', (event) => {
  event.preventDefault();
  dropZone.classList.remove('drop-zone--active');
  setDroppedFiles([...event.dataTransfer.files]);
});
```

この例を前の検証処理と組み合わせる場合は、同じ入力要素と`change`ハンドラーを使う。ファイル選択とドロップで別々の検証規則を持たない。

ドラッグ操作しか提供しないUIは避ける。ファイル選択ボタンなら、キーボードだけでなく、タッチ画面でドラッグが難しい利用者にも単純なタップ操作を提供できる。

> 参照:
> - [Understanding WCAG 2.2 SC 2.1.1 — Keyboard](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html)
> - [Understanding WCAG 2.2 SC 2.5.7 — Dragging Movements](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements.html)

## クリップボードからの貼り付け

画像を貼り付けられる機能を追加する場合も、ファイル選択を残す。

```html
<button type="button" id="paste-image">クリップボードから画像を貼り付け</button>
```

貼り付け後はファイル名がない、または自動生成名になることがある。「クリップボード画像、1.2 MB」のように内容を識別できる表示を用意する。

クリップボードへのアクセスが拒否された場合は、「貼り付けに失敗しました」だけで終わらせず、ブラウザの権限を確認するかファイル選択を使えることを伝える。

## アップロードの開始を分ける

ファイルを選んだ瞬間にアップロードを始めると、選び間違いを取り消す時間がない。特別な理由がなければ、選択と送信を分ける。

```html
<button type="submit">選択した3件をアップロード</button>
```

自動アップロードを使う場合は、選択と同時に開始することを事前に示し、キャンセルや削除方法を提供する。

## アップロード中を伝える

```html
<section aria-labelledby="upload-heading" aria-busy="true">
  <h2 id="upload-heading">ファイルをアップロード中</h2>

  <label for="upload-progress">全体の進捗</label>
  <progress id="upload-progress" value="45" max="100">45%</progress>
  <span>45%</span>

  <button type="button">アップロードをキャンセル</button>
  <p id="upload-status" role="status">3件中1件が完了しました。</p>
</section>
```

- 値が分かる場合は`progress`へ`value`と`max`を設定する
- 値が分からない場合は`value`属性を付けない
- `aria-busy="true"`は処理終了時に`false`へ戻す
- パーセントを1%ごとにライブリージョンへ送らない
- 長時間処理にはキャンセル操作を用意する
- アップロード中でも他のページへ移動できるかを説明する

ファイルごとの進捗が操作判断に不要なら、全体のプログレスバーと簡潔な一覧にする。多数のプログレスバーを同時に更新すると、視覚的にも支援技術にも情報量が多くなる。

## 成功と失敗をファイルごとに示す

複数ファイルでは、一部だけ成功する場合がある。「アップロードに失敗しました」だけで全体を扱わず、各結果を残す。

```html
<section aria-labelledby="upload-result-heading">
  <h2 id="upload-result-heading">アップロード結果</h2>
  <p role="status">3件中2件をアップロードしました。1件に失敗しました。</p>

  <ul>
    <li>application.pdf — 完了</li>
    <li>photo.jpg — 完了</li>
    <li>
      large-image.png — 失敗: サーバーの容量上限を超えました。
      <button type="button">large-image.pngを再試行</button>
      <button type="button">large-image.pngを削除</button>
    </li>
  </ul>
</section>
```

失敗した項目だけを再試行できるようにすると、成功済みファイルの再送信を避けられる。

### 失敗理由の例

| 状態 | 伝える内容 |
|---|---|
| 形式が不正 | ファイル名、許可される形式 |
| 容量超過 | ファイル名、現在の容量、上限 |
| 件数超過 | 選択件数、最大件数 |
| 通信失敗 | 未送信であること、再試行方法 |
| サーバー拒否 | 理由、修正または問い合わせ方法 |
| セキュリティ検査中 | 完了まで処理中であること |
| 一部成功 | 成功件数、失敗件数、各結果 |

エラーでフォーカスを自動的に再試行ボタンへ移す必要はない。結果の見出しやエラーサマリーを通知し、利用者が一覧から次の操作を選べるようにする。

> 参照: [Understanding WCAG 2.2 SC 4.1.3 — Status Messages](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html)

## 画像プレビューを表示する

プレビューは選択確認の補助手段。画像だけでなくファイル名も残す。

```html
<figure>
  <img src="blob:..." alt="" />
  <figcaption>profile-photo.jpg（1.4 MB）</figcaption>
</figure>
```

ファイル名が説明として表示されるなら、プレビュー画像の`alt`は空にして重複を避けられる。画像内容の確認が重要な場合は、用途に合う説明を付ける。

`URL.createObjectURL()`で作ったURLは、ファイルを削除したときやコンポーネントを破棄するときに`URL.revokeObjectURL()`で解放する。

プレビュー生成に失敗しても、元ファイルまで自動的に削除しない。「プレビューを表示できません。ファイルは選択されています」のように状態を分けて伝える。

## カスタムデザインではネイティブ入力を残す

ネイティブのファイル選択ボタンは`::file-selector-button`で装飾できる。

```css
.file-field input[type="file"] {
  max-inline-size: 100%;
  padding: 0.5rem;
  border: 0.125rem solid currentColor;
  border-radius: 0.25rem;
}

.file-field input[type="file"]::file-selector-button {
  padding: 0.625rem 1rem;
  border: 0.125rem solid currentColor;
  border-radius: 0.25rem;
  margin-inline-end: 0.75rem;
  background: Canvas;
  color: CanvasText;
  font: inherit;
  cursor: pointer;
}

.file-field input[type="file"]:focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}
```

ファイル入力を`display: none`にして、クリックイベントしかない`div`へ置き換えない。ネイティブ入力を視覚的に隠す場合は、代わりの操作が実際の`button`または関連付いた`label`で、キーボードフォーカスが見えることを複数ブラウザで確認する。

ボタンの見える文言とアクセシブルネームを一致させる。音声入力利用者が「ファイルを選択をクリック」のように指定できる状態を保つ。

## ドロップ領域と一覧のCSS

```css
.drop-zone {
  padding: 1.5rem;
  border: 0.125rem dashed CanvasText;
  border-radius: 0.5rem;
  background: Canvas;
  color: CanvasText;
}

.drop-zone--active,
.drop-zone:focus-within {
  border-style: solid;
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

.file-list {
  display: grid;
  gap: 0.75rem;
  padding-inline-start: 1.5rem;
}

.file-list li {
  overflow-wrap: anywhere;
}

.file-list button {
  margin-inline-start: 0.75rem;
}

.file-errors,
.field-error {
  color: #b00020;
  font-weight: 700;
}

@media (forced-colors: active) {
  .drop-zone,
  .file-field input[type="file"] {
    border-color: CanvasText;
  }
}
```

長いファイル名が横にはみ出さないようにする。200%・400%拡大や狭い画面で、削除ボタンがファイル名と重なったり画面外へ出たりしないことを確認する。

## Reactで実装する

```jsx
import { useId, useRef, useState } from 'react';

const MAX_FILES = 3;
const MAX_FILE_SIZE = 5 * 1024 * 1024;
const ALLOWED_TYPES = new Set([
  'application/pdf',
  'image/jpeg',
  'image/png',
]);
const ALLOWED_EXTENSIONS = new Set(['pdf', 'jpg', 'jpeg', 'png']);

function isAllowedFile(file) {
  const extension = file.name.toLowerCase().split('.').pop();
  return ALLOWED_TYPES.has(file.type) || ALLOWED_EXTENSIONS.has(extension);
}

function validateFiles(files) {
  const errors = [];

  files.forEach((file, index) => {
    if (index >= MAX_FILES) {
      errors.push(`${file.name}: 最大${MAX_FILES}件までです。`);
    }
    if (!isAllowedFile(file)) {
      errors.push(`${file.name}: PDF、JPEG、PNGを選択してください。`);
    }
    if (file.size > MAX_FILE_SIZE) {
      errors.push(`${file.name}: 5 MB以下にしてください。`);
    }
  });

  return errors;
}

function FilePicker({ onFilesChange, onValidityChange }) {
  const inputId = useId();
  const helpId = useId();
  const errorId = useId();
  const inputRef = useRef(null);
  const [files, setFiles] = useState([]);
  const [status, setStatus] = useState('');
  const errors = validateFiles(files);

  function updateFiles(nextFiles) {
    const nextErrors = validateFiles(nextFiles);
    setFiles(nextFiles);
    onFilesChange(nextFiles);
    onValidityChange(nextErrors.length === 0);
    return nextErrors;
  }

  function handleChange(event) {
    const nextFiles = [...event.target.files];
    const nextErrors = updateFiles(nextFiles);
    setStatus(
      nextErrors.length > 0
        ? `${nextFiles.length}件選択しました。修正が必要なファイルがあります。`
        : `${nextFiles.length}件選択しました。`,
    );
  }

  function removeFile(index) {
    const fileToRemove = files[index];
    if (!fileToRemove) return;

    const removedName = fileToRemove.name;
    const nextFiles = files.filter((_, fileIndex) => fileIndex !== index);
    const transfer = new DataTransfer();

    nextFiles.forEach((file) => transfer.items.add(file));
    inputRef.current.files = transfer.files;
    updateFiles(nextFiles);
    setStatus(`${removedName}を削除しました。`);
    inputRef.current.focus();
  }

  return (
    <div className="file-field">
      <label htmlFor={inputId}>申請書類</label>
      <p id={helpId}>
        PDF、JPEG、PNGを3件まで選択できます。1件につき5 MBまでです。
      </p>

      <input
        ref={inputRef}
        id={inputId}
        type="file"
        name="documents[]"
        accept=".pdf,.jpg,.jpeg,.png,application/pdf,image/jpeg,image/png"
        multiple
        aria-describedby={`${helpId}${errors.length > 0 ? ` ${errorId}` : ''}`}
        aria-invalid={errors.length > 0 || undefined}
        onChange={handleChange}
      />

      {errors.length > 0 && (
        <div id={errorId} className="file-errors">
          <p>追加できないファイルがあります。</p>
          <ul>
            {errors.map((error, index) => (
              <li key={`${index}:${error}`}>{error}</li>
            ))}
          </ul>
        </div>
      )}

      <p role="status" aria-atomic="true">{status}</p>

      <ul aria-label="選択したファイル">
        {files.map((file, index) => (
          <li key={`${file.name}:${file.size}:${file.lastModified}:${index}`}>
            <span>{file.name}</span>
            <button type="button" onClick={() => removeFile(index)}>
              {file.name}を削除
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

親コンポーネントでは、`onValidityChange`が`false`を受け取っている間はアップロードを開始しない。ボタンを単に無効にするだけでなく、修正が必要な理由を表示する。

この例では配列の位置も`key`に含めて重複を避けているが、削除すると後続項目の位置が変わる。安定した識別が必要な実装では、選択時に一時IDを割り当てる。

## よくある失敗

### ファイル入力にラベルがない

見える`label`を付け、目的を具体的に示す。周囲の見出しやプレースホルダーだけに依存しない。

### 条件をエラーになってから示す

形式、最大容量、最大件数を選択前に示し、`aria-describedby`で入力と関連付ける。

### `accept`だけで安全だと考える

`accept`はヒント。クライアントとサーバーで検証し、サーバーでは実際のファイル内容も確認する。

### ファイル名を表示しない

複数選択では全件を一覧にし、名前、容量、状態を確認できるようにする。

### 削除ボタンがすべて「削除」

「report.pdfを削除」のように、アクセシブルネームから対象を区別できるようにする。

### 削除後にフォーカスが消える

次の削除ボタン、前の削除ボタン、ファイル入力の順で適切な移動先を決める。

### ドラッグ＆ドロップしか使えない

同じ機能をネイティブのファイル入力からも実行できるようにする。

### 選択と同時に無断でアップロードする

送信のタイミングを事前に示す。可能なら確認と削除の後に明示的なアップロードボタンを置く。

### 進捗を1%ごとに読み上げる

見えるバーは細かく更新しても、ライブリージョンは開始、意味のある節目、完了、失敗に絞る。

### 一部失敗を全体失敗として扱う

各ファイルの成功・失敗と、再試行・削除の対象を明確にする。

### 失敗後に入力をすべて消す

成功済みと再試行が必要なファイルを保持し、利用者に同じ選択を最初からやり直させない。

## テストのチェックリスト

### ラベルと説明

- [ ] ファイル入力に見える`label`がある
- [ ] ラベルから選択するファイルの目的が分かる
- [ ] 許可形式、最大容量、最大件数を選択前に確認できる
- [ ] 説明が`aria-describedby`で入力と関連付いている
- [ ] 必須の場合は`required`と見える必須表示がある
- [ ] ボタンの見える文言とアクセシブルネームが一致する

### 選択と削除

- [ ] キーボードだけでファイル選択画面を開ける
- [ ] 音声入力から見えるラベル名で操作できる
- [ ] 選んだ全ファイルの名前と容量を確認できる
- [ ] 選択件数をテキストで確認できる
- [ ] 各ファイルを対象名のあるボタンで削除できる
- [ ] 削除後にフォーカスが予測可能な場所へ移る
- [ ] 同じファイルを選び直せる
- [ ] すべて削除した状態とファイル入力の値が一致する

### 検証とエラー

- [ ] `accept`だけに依存していない
- [ ] 形式、容量、件数をサーバー側でも検証している
- [ ] ファイル名を安全な文字列として扱っている
- [ ] エラーに対象のファイル名が含まれている
- [ ] エラーに理由と修正方法が含まれている
- [ ] エラーを色やアイコンだけで示していない
- [ ] エラー状態が入力とプログラムで関連付いている
- [ ] エラー後も問題のない選択内容を保持している

### ドラッグ＆ドロップ

- [ ] ドロップ以外にファイル選択を使用できる
- [ ] ドロップとファイル選択が同じ検証処理を使う
- [ ] ドラッグ中の見た目に色以外の変化がある
- [ ] ドロップ領域を不要なTab停止位置にしていない
- [ ] タッチ操作でも単純なファイル選択を使用できる

### アップロード

- [ ] 選択とアップロード開始のタイミングが明確である
- [ ] アップロード中であることをテキストで確認できる
- [ ] 値が分かる進捗と不確定な進捗を区別している
- [ ] 過剰な頻度で進捗を読み上げていない
- [ ] 長時間処理をキャンセルできる
- [ ] 成功、失敗、キャンセルを区別して伝えている
- [ ] 一部失敗した場合にファイルごとの結果がある
- [ ] 失敗したファイルだけを再試行できる

### 表示と支援技術

- [ ] 長いファイル名が領域からはみ出さない
- [ ] 200%・400%拡大でも選択、削除、送信できる
- [ ] 強制カラーモードで入力とエラーを識別できる
- [ ] スクリーンリーダーでラベル、説明、件数、エラーを確認した
- [ ] モバイル端末のファイル選択と撮影候補を確認した
- [ ] ファイル選択画面をキャンセルしても状態が破損しない

## 関連して読むとよいテーマ

- [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)
- [アクセシブルなプログレス表示の実装 — progress と meter の違い](./accessible-progress-and-meter.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [手動テストのチェックリスト — 最低限確認すべき項目](./manual-testing-checklist.md)
- [スクリーンリーダーでの手動テスト入門 — VoiceOver / NVDA](./screen-reader-manual-testing.md)

## まとめ

- ネイティブの`input type="file"`と見える`label`を基本にする
- 形式、容量、件数を選択前に説明する
- `accept`を検証やセキュリティ対策の代わりにしない
- 選択した全ファイルの名前、容量、状態を一覧で示す
- エラーには対象のファイル名、理由、修正方法を含める
- 各ファイルを対象名のあるボタンで削除できるようにする
- 削除後のフォーカスと状態通知を管理する
- ドラッグ＆ドロップを追加してもファイル選択を残す
- アップロードの開始、進捗、成功、失敗、キャンセルを区別する
- 複数ファイルの一部失敗では、各結果と個別の再試行手段を示す
- クライアントとサーバーの両方で検証し、問題のない入力を失わせない
