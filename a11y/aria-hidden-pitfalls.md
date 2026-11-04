# aria-hiddenは禁止したほうがよい？ — 誤用が多い属性と代替手段

## aria-hiddenとは

- `aria-hidden`は、WAI-ARIA（支援技術へ要素の意味や状態を伝える仕様）の属性である — `aria-hidden="true"`を指定すると、要素とその子孫をアクセシビリティツリーから除外する<sup><a href="#ref-1">[1]</a></sup>
- アクセシビリティツリーは、ブラウザが要素の役割や名前などを支援技術へ伝えるための構造である
- `aria-hidden="true"`を指定しても画面上の見た目は変わらない — キーボードフォーカスやクリックも無効にならない
- 親要素に指定すると、その中のテキスト、リンク、ボタンなどもまとめて除外される
- 子要素へ`aria-hidden="false"`を指定しても、祖先の`aria-hidden="true"`を打ち消すことはできない<sup><a href="#ref-1">[1]</a></sup>
- 全面禁止にするより、まずHTMLやCSSで目的を満たせるかを検討し、読み上げから除く必要のある範囲だけに使う方針が現実的である

```html
<!-- 星印は装飾であり、意味は隣のテキストで伝える -->
<span aria-hidden="true">★</span>
<span>おすすめ</span>
```

<ol start="1">
  <li id="ref-1"><a href="https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-hidden">MDN — aria-hidden</a></li>
</ol>

## 誤用で問題になること

- 「見せない」「操作させない」「装飾を読み上げから除く」は異なる目的である — すべてを`aria-hidden`だけで実現しようとすると、目的と効果が一致しなくなる
- フォーカス可能な要素やその祖先には、`aria-hidden="true"`を指定しない — 支援技術から除外された操作対象へキーボードフォーカスが移動する問題につながる<sup><a href="#ref-2">[2]</a></sup>
- 次の例では、背景のリンクを支援技術から隠しても、`Tab`キーによる移動先からは除かれない

```html
<!-- 避けたい例：背景の操作を止める効果はない -->
<main aria-hidden="true">
  <a href="/">ホーム</a>
  <button type="button">検索</button>
</main>
```

- 料金、注意書き、エラーメッセージなど、利用者の判断に必要な画面上の情報を支援技術だけから隠さない
- 「税込」のような短い補足も情報の一部である — 読み上げを短くするためだけに除外しない

```html
<!-- 料金と税の情報を、そのまま伝える -->
<p>12,000円（税込）</p>
```

- 重複を理由に除外する場合は、同じ意味が別の箇所から伝わることと、操作に支障が出ないことを確認する
- 子孫の`aria-hidden="false"`で修正しようとせず、隠す範囲から操作対象を外すなど、マークアップの構造を見直す

<ol start="2">
  <li id="ref-2"><a href="https://www.w3.org/WAI/standards-guidelines/act/rules/6cfa84/">W3C ACT — Element with aria-hidden has no content in sequential focus navigation</a></li>
</ol>

## 非表示と操作の無効化に使う手段

### 画面にも表示しない内容

- 閉じているパネルなど、画面にも支援技術にも提示しない内容は、通常の`hidden`や`display: none`で非表示にする
- すでに`hidden`、`display: none`、`visibility: hidden`で非表示になっている要素へ、`aria-hidden="true"`を重ねる必要はない<sup><a href="#ref-3">[3]</a></sup>
- 非表示と`aria-hidden`を別々に切り替えると、再表示したのに支援技術からは隠れたまま、という状態を作りやすくなる

```html
<!-- hiddenだけで、通常の表示と支援技術への提示を止める -->
<div id="details-panel" hidden>
  <p>詳細な説明</p>
</div>
```

- 画面上だけで隠し、スクリーンリーダーには伝えたい場合は目的が逆になる — 視覚的に隠すCSSを使い、`aria-hidden`は付けない

### モーダルの背後にある内容

- 背景を操作できない状態にする場合は、`inert`や、`<dialog>`を`showModal()`で開く方法を検討する
- `inert`は対象とその子孫をフォーカスやクリックの対象から外し、アクセシビリティツリーからも除外する — 見た目を暗くするなどの表示は別に用意する<sup><a href="#ref-4">[4]</a></sup>
- `showModal()`で開いた`<dialog>`では、同じ文書内のダイアログ外の内容が操作できない状態になる<sup><a href="#ref-5">[5]</a></sup>
- 次の例は、ネイティブのモーダルダイアログを開き、フォームの送信で閉じる最小構成である

```html
<button id="open-dialog" type="button">説明を開く</button>

<dialog id="help-dialog" aria-labelledby="help-title">
  <h2 id="help-title">説明</h2>
  <p>内容を確認したら閉じるボタンで戻る</p>
  <form method="dialog">
    <button autofocus>閉じる</button>
  </form>
</dialog>
```

```js
// 上のHTMLより後で実行する
const openButton = document.querySelector('#open-dialog');
const helpDialog = document.querySelector('#help-dialog');

openButton.addEventListener('click', () => {
  helpDialog.showModal();
});
```

- `<dialog showModal>`というHTML属性はない — `showModal()`はJavaScriptで呼び出すメソッドである
- 独自のモーダルを実装する場合は、背景の`inert`、開くときのフォーカス移動、ダイアログ内でのフォーカス管理、閉じたときの復帰を組み合わせる
- `aria-modal="true"`はモーダルであることを伝える指定であり、背景の操作やフォーカス移動を止める処理を自動で実装するものではない<sup><a href="#ref-6">[6]</a></sup>
- 独自モーダルの本体は、`inert`を指定した背景要素の子孫へ入れない — 本体まで操作できなくなることを避ける
- 個別のフォーム部品を無効にする場合は`disabled`も候補になるが、`aria-hidden`や`inert`とは効果が異なる — 要素の種類と目的に合わせて選ぶ

<ol start="3">
  <li id="ref-3"><a href="https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-hidden">MDN — aria-hiddenが不要な非表示状態</a></li>
  <li id="ref-4"><a href="https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inert">MDN — inert</a></li>
  <li id="ref-5"><a href="https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal">MDN — HTMLDialogElement: showModal()</a></li>
  <li id="ref-6"><a href="https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/">W3C APG — Dialog (Modal) Pattern</a></li>
</ol>

## 装飾アイコンでの使い方

- 隣のテキストで同じ意味を伝えているアイコンや、純粋な装飾を読み上げから除く場合には、`aria-hidden="true"`を使える<sup><a href="#ref-7">[7]</a></sup>
- たとえば検索ボタンに「検索」という可視テキストがあれば、アイコンだけを除外し、ボタンの名前はテキストで伝える

```html
<button type="button">
  <svg aria-hidden="true" focusable="false" viewBox="0 0 24 24"
       width="24" height="24" fill="none" stroke="currentColor">
    <circle cx="10" cy="10" r="6"></circle>
    <path d="m15 15 5 5"></path>
  </svg>
  検索
</button>
```

- ボタン全体へ`aria-hidden="true"`を指定しない — 操作部品は支援技術へ提示したまま、装飾のSVGだけを除外する
- アイコンだけのボタンには、可視テキストや`aria-label`などで操作の名前を付ける — 中身を隠すだけでは名前を補えない

```html
<button type="button" aria-label="閉じる">
  <svg aria-hidden="true" focusable="false" viewBox="0 0 24 24"
       width="24" height="24" fill="none" stroke="currentColor">
    <path d="m6 6 12 12 M18 6 6 18"></path>
  </svg>
</button>
```

- `<img>`で表示する装飾画像には空の`alt=""`を指定する — 意味を持つ画像には、その情報や機能に合う代替テキストを用意する<sup><a href="#ref-8">[8]</a></sup>
- 装飾かどうかは見た目だけで決めない — 状態や注意をアイコンだけで表している場合は、その意味をテキストなどでも伝える

<ol start="7">
  <li id="ref-7"><a href="https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-hidden">MDN — 装飾アイコンへのaria-hiddenの使用例</a></li>
  <li id="ref-8"><a href="https://www.w3.org/WAI/tutorials/images/decorative/">W3C WAI — Decorative Images</a></li>
</ol>

## 実装後に確認すること

- `Tab`と`Shift` + `Tab`で移動し、支援技術から隠した範囲へフォーカスが入らないかを確認する
- マウス操作だけでなく、キーボードとスクリーンリーダーでも、ボタンやリンクの名前と操作結果を確認する
- モーダルを開いたときは内部へフォーカスが移り、閉じたときは通常、開く操作をした要素へ戻ることを確認する — 元の要素を削除した場合などは、次に操作する場所へ戻す設計が必要になる
- モーダルの表示中に背景へフォーカスが移動しないことと、背景の操作が無効になっていることを確認する
- パネルを再表示したときに`aria-hidden="true"`が残っていないかを確認する
- 料金やエラーなどの重要な情報を、見えているのに支援技術からは読めない状態にしていないかを確認する
- ブラウザのアクセシビリティツリーを確認し、装飾だけが除外され、操作部品の名前や状態が残っているかを見る
- 自動検査の`aria-hidden-focus`なども利用できるが、自動検査だけで開閉後の操作や情報の伝わり方まで確認できたとは考えない<sup><a href="#ref-9">[9]</a></sup>

<ol start="9">
  <li id="ref-9"><a href="https://dequeuniversity.com/rules/axe/4.10/aria-hidden-focus">Deque — axe-coreのaria-hidden-focusルール</a></li>
</ol>

## まとめ

- `aria-hidden="true"`は要素とその子孫をアクセシビリティツリーから除外するが、表示を消したり操作を無効にしたりはしない
- フォーカス可能な要素とその祖先へ指定しない — 支援技術から隠した操作対象へフォーカスが移動する問題につながる
- 画面にも表示しない内容には、`hidden`やCSSによる非表示を使う
- モーダルの背景を操作できなくする場合は、`inert`や`<dialog>`の`showModal()`を検討する
- 隣に同等のテキストがあるアイコンなど、装飾や重複した情報の除外には使える
- 料金や注意書きなどの重要な情報を、読み上げを短くするためだけに隠さない
- 子の`aria-hidden="false"`で祖先の指定を打ち消すことはできないため、隠す範囲を構造から見直す
- 全面禁止より、目的に合うHTML・CSSを先に検討し、必要な範囲だけに使う方針にしたい
