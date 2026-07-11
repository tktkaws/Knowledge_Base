# 2.1.2 キーボードトラップなし（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.1.2「キーボードトラップなし」（No Keyboard Trap）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.1「キーボード操作可能」に属する
- キーボードでフォーカスを移動できるコンポーネントから、**キーボードだけでフォーカスを外せる**こと
- 標準的な脱出方法（Tab、矢印キー、Esc 等）以外が必要な場合は、**その方法をユーザーに知らせる**こと

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.2 No Keyboard Trap](https://www.w3.org/TR/WCAG22/#no-keyboard-trap)
> - [Understanding SC 2.1.2 No Keyboard Trap](https://www.w3.org/WAI/WCAG22/Understanding/no-keyboard-trap.html)

## 達成基準の原文（要約）

- キーボードでページ内のコンポーネントにフォーカスを移動できる場合、そのコンポーネントから**キーボードだけで**フォーカスを外せること
- 未変更の矢印キー、Tab キー、その他の標準的な脱出方法以上の操作が必要な場合は、脱出方法をユーザーに知らせること
- 注記：この達成基準を満たさないコンテンツはページ全体の利用を妨げるため、**ページ上のすべてのコンテンツ**が対象（適合要件5：非干渉）

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.2](https://www.w3.org/TR/WCAG22/#no-keyboard-trap)

## キーボードトラップとは

- キーボードでフォーカスを移動したあと、**そのコンポーネントから抜け出せなくなる**状態
- Tab や Shift+Tab を押しても、同じ領域内を循環し続ける
- マウスなら別の場所をクリックして脱出できるが、キーボードユーザーは操作不能になる
- 例
  - プラグインや埋め込みアプリ内にフォーカスが入り、戻れない
  - カスタムウィジェットが Tab キーを横取りする
  - `iframe` 内のコンテンツから親ページに戻れない
  - モーダルを閉じる手段がキーボードで使えない

```text
トラップの例：
  Tab → ウィジェット内の要素A
  Tab → ウィジェット内の要素B
  Tab → 要素A（ページの他の部分に戻れない）
```

> **参照**
> - [Understanding SC 2.1.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/no-keyboard-trap.html#intent)

## なぜ必要か

- キーボードやキーボードインターフェースに依存するユーザーが、ページ全体を操作し続けられるようにするため
- 視覚障害のあるユーザーは、マウス以外の手段でページを移動する
- 運動障害のあるユーザーは、キーボードが主な操作手段になることが多い
- 1箇所のトラップだけで、ページ全体が使えなくなる

> **参照**
> - [Understanding SC 2.1.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/no-keyboard-trap.html#benefits)

## 2.1.1 との関係

| 達成基準 | 焦点 |
|---|---|
| 2.1.1 キーボード | すべての機能をキーボードで操作できること |
| 2.1.2 キーボードトラップなし | フォーカスが特定の領域に閉じ込められないこと |

- 2.1.1 は「操作できるか」
- 2.1.2 は「抜け出せるか」
- キーボードで入れた領域から、キーボードで出られなければ 2.1.2 違反

> **参照**
> - [wcag/2-1-1-keyboard.md — 2.1.1 キーボード](./2-1-1-keyboard.md)

## 標準的な脱出方法

- **Tab** / **Shift+Tab** で次／前のフォーカス可能要素へ移動できること
- **Esc** キーでモーダルやメニューを閉じられること（多くの環境で一般的）
- 未変更の**矢印キー**で領域を出られる場合もある
- 「標準的な脱出方法」はハードウェア・ユーザーエージェント・OS に依存するため、作者と監査者の解釈が必要
- 一般的には Esc、Tab、矢印キーが標準的な脱出方法とみなされる

```html
<!-- 良い例：Esc と Tab の両方でモーダルから出られる -->
<dialog id="confirm-dialog">
  <p>本当に削除しますか？</p>
  <button type="button" value="cancel">キャンセル</button>
  <button type="submit" value="ok">削除</button>
</dialog>
```

> **参照**
> - [Understanding SC 2.1.2 — standard exit methods](https://www.w3.org/WAI/WCAG22/Understanding/no-keyboard-trap.html#intent)

## 意図的なフォーカス制限は適合しうる

- モーダルダイアログやポップオーバー内では、フォーカスをその領域に留めるのが一般的
- これ自体は 2.1.2 違反ではない
- 条件は、ユーザーが**脱出方法を知っている**こと、かつ**キーボードで脱出できる**こと
- モーダル内で Tab が循環するのは意図的なフォーカストラップであり、Cancel／OK ボタンや Esc で閉じられれば適合

```javascript
// モーダル内で Tab を循環させる（意図的なトラップ）
dialog.addEventListener("keydown", (event) => {
  if (event.key !== "Tab") return;

  const focusable = dialog.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const first = focusable[0];
  const last = focusable[focusable.length - 1];

  if (event.shiftKey && document.activeElement === first) {
    event.preventDefault();
    last.focus();
  } else if (!event.shiftKey && document.activeElement === last) {
    event.preventDefault();
    first.focus();
  }
});

// Esc で閉じる → 2.1.2 を満たす
dialog.addEventListener("cancel", () => {
  dialog.close();
  triggerButton.focus();
});
```

```html
<!-- 良い例：dialog 要素は showModal() でフォーカストラップを提供 -->
<button type="button" id="open-dialog">削除</button>
<dialog id="delete-dialog">
  <p>本当に削除しますか？</p>
  <button type="button" value="cancel">キャンセル</button>
  <button type="submit" value="ok">削除</button>
</dialog>
```

> **参照**
> - [a11y/focus-management.md — フォーカストラップ](../a11y/focus-management.md)
> - [Understanding SC 2.1.2 — modal dialog example](https://www.w3.org/WAI/WCAG22/Understanding/no-keyboard-trap.html#examples)

## 非標準の脱出方法がある場合は案内する（G21）

- Tab や Esc 以外のキーで脱出する場合は、**その方法をユーザーに知らせる**必要がある
- 例：リッチテキストエディタで Tab がインデントに使われる場合、`Alt+F10` でツールバーに戻る旨を表示

```html
<!-- 良い例：脱出方法を案内 -->
<div
  role="textbox"
  contenteditable="true"
  aria-label="本文"
  aria-describedby="editor-help"
>
  本文を入力...
</div>
<p id="editor-help">
  Tab キーはインデントに使われます。ツールバーへ戻るには Alt+F10 を押してください。
</p>
```

```html
<!-- 良い例：アプレットの前後に脱出方法を記載 -->
<p>パズル内では Tab 以外のキーで操作します。Esc キーでパズルから出られます。</p>
<!-- アプレット -->
<p>パズル内で Esc キーを押すと外に戻れます。</p>
```

> **参照**
> - [Technique G21: Ensuring that users are not trapped in content](https://www.w3.org/WAI/WCAG22/Techniques/general/G21)

## iframe・埋め込みコンテンツ

- `iframe` 内にフォーカスが入ると、親ページに戻れないことがある
- キーボードで `iframe` に入れるなら、キーボードで出られる必要がある
- ブラウザの標準操作（Tab で次の要素へ）で出られるか確認
- カスタム `iframe` 実装では、Esc や明示的な「閉じる」操作を用意する

```html
<!-- 確認ポイント：iframe 内から Tab で親ページに戻れるか -->
<iframe
  src="/calendar-widget"
  title="カレンダー"
  width="400"
  height="300"
></iframe>
<a href="/next-section">次のセクションへ</a>
```

> **参照**
> - [Failure F10: combining multiple content formats in a way that traps users](https://www.w3.org/WAI/WCAG22/Techniques/failures/F10)

## カスタムコンポーネントでの Tab 横取り

- Tab キーのデフォルト動作を `preventDefault()` で止めると、トラップの原因になりやすい
- 独自の Tab 処理をする場合は、脱出手段を必ず用意する
- `tabindex` の正の値で Tab 順を乱すと、予期しない循環を起こすことがある

```javascript
// 悪い例：Tab を常に横取りし、領域から出られない
container.addEventListener("keydown", (event) => {
  if (event.key === "Tab") {
    event.preventDefault();
    // 内部だけを循環 → ページから出られない
    moveFocusInsideOnly();
  }
});

// 良い例：モーダル内のみ Tab を循環し、Esc で閉じられる
modal.addEventListener("keydown", (event) => {
  if (event.key === "Escape") {
    closeModal();
    return;
  }
  // Tab 循環はモーダル内に限定
});
```

## 適合要件5：非干渉

- 2.1.2 を満たさないコンテンツは、ページ全体の利用を妨げる
- そのため、**ページ上のすべてのコンテンツ**が 2.1.2 を満たす必要がある
- 他の達成基準の対象外コンテンツであっても、2.1.2 は適用される
- 1つのウィジェットだけがトラップでも、ページ全体が不適合になりうる

> **参照**
> - [WCAG 2.2 — Conformance Requirement 5: Non-Interference](https://www.w3.org/TR/WCAG22/#cc5)

## よくある失敗例

### F10：複数フォーマットの組み合わせでトラップ

- プラグイン、アプレット、埋め込みアプリと HTML を組み合わせ、キーボードで抜け出せなくなる

```html
<!-- 悪い例：アプレット内に入ると Esc も Tab も効かず脱出不能 -->
<applet code="Puzzle.class" width="400" height="300"></applet>
```

### モーダルを閉じる手段がない

- モーダル内で Tab が循環するが、Cancel ボタン・Esc・閉じるボタンがキーボードで使えない

```html
<!-- 悪い例：閉じるボタンが pointer のみ -->
<div class="modal" role="dialog">
  <p>内容</p>
  <button type="button" class="close-btn" tabindex="-1">×</button>
</div>
```

### フォーカスを受け取ったら除去する（2.1.1 とも関連）

- `focus` イベントで `blur()` する実装は、キーボード操作を妨げる（F55）

> **参照**
> - [Failure F10: traps users inside one format type](https://www.w3.org/WAI/WCAG22/Techniques/failures/F10)
> - [Failure F55: using script to remove focus when focus is received](https://www.w3.org/WAI/WCAG22/Techniques/failures/F55)

## 他の達成基準との関係

| 達成基準 | 関係 |
|---|---|
| 2.1.1 キーボード | キーボードで操作できること |
| 2.4.3 フォーカス順序 | Tab 順が意味のある順序であること |
| 2.4.7 フォーカスの可視化 | フォーカス位置が見えること |
| 3.2.1 フォーカス時 | フォーカスだけで大きなコンテキスト変化を起こさないこと |

- モーダルのフォーカストラップは 2.4.3 の文脈でも設計される
- トラップと「フォーカス順序の制御」は別概念
  - トラップ：脱出できない（不適合）
  - 意図的な閉じ込め：Esc 等で脱出できる（適合しうる）

> **参照**
> - [a11y/focus-management.md — フォーカス管理](../a11y/focus-management.md)

## テスト・確認方法

### 手動テスト

- キーボードだけでページ内のすべてのインタラクティブ領域を巡回
- 各領域に入ったあと、Tab / Shift+Tab で次の領域に進めるか確認
- モーダル・アプレット・カスタムウィジェット内で Esc が使えるか確認
- 非標準の脱出キーがある場合、案内文が表示されているか確認
- `iframe` 内に入ってから、親ページに戻れるか確認

```text
チェックリスト：
1. Tab でページ全体を巡回できるか
2. 特定のウィジェット内で Tab が無限循環しないか
3. 循環する場合、Esc や閉じるボタンで脱出できるか
4. 非標準の脱出方法が案内されているか
5. iframe や埋め込みコンテンツから戻れるか
```

### 自動テスト

- ACT Rule：[Focusable element has no keyboard trap](https://www.w3.org/WAI/standards-guidelines/act/rules//gherbi/)
- ただし、モーダルの意図的なフォーカストラップや Esc 脱出は手動確認が必要

```bash
# Tab 横取りや focus/blur 操作の実装を確認
rg "preventDefault.*Tab|event\.key === ['\"]Tab['\"]|\.blur\(\)" --glob "*.js" --glob "*.tsx" --glob "*.jsx"
```

> **参照**
> - [a11y/keyboard-navigation-basics.md — キーボードナビゲーションの基本](../a11y/keyboard-navigation-basics.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.1.2 は、キーボードでフォーカスを移動したコンポーネントから、**キーボードだけで脱出できる**ことを求める達成基準（レベル A）
- モーダル内での意図的なフォーカストラップ自体は問題ではない。Esc や閉じるボタンで脱出できれば適合しうる
- Tab や Esc 以外の脱出方法が必要な場合は、**その方法をユーザーに知らせる**
- `iframe`、プラグイン、カスタムウィジェットでのトラップに注意
- 1箇所のトラップがページ全体を不適合にする（適合要件5：非干渉）
- 2.1.1（キーボード操作）・2.4.3（フォーカス順序）と併せて確認する
