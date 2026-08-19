# 3.2.1 フォーカス時（A）

## 達成基準の概要

- WCAG 2.2 達成基準 3.2.1「フォーカス時」（On Focus）
- レベル **A**（最低限の適合レベル）
- 原則3「理解可能（Understandable）」> ガイドライン 3.2「予測可能」に属する
- UI コンポーネントがフォーカスを受けたとき、**コンテキストの変化を起こさない**こと
- フォーカスではなく、クリックや Enter などの**活性化**でページ遷移や送信を行う

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.1 On Focus](https://www.w3.org/TR/WCAG22/#on-focus)
> - [Understanding SC 3.2.1 On Focus](https://www.w3.org/WAI/WCAG22/Understanding/on-focus.html)

## 達成基準の原文（要約）

- いずれのユーザーインターフェースコンポーネントも、フォーカスを受けたときにコンテキストの変化を開始しないこと

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.1](https://www.w3.org/TR/WCAG22/#on-focus)

## なぜ必要か

- キーボードで Tab 移動しているとき、フォーカスしただけでページが変わると迷子になる
- 視覚障害・認知の制約・運動障害のある人は、予期しない変化に特に弱い
- 画面拡大では、変化が画面外で起きても気づきにくい
- スクリーンリーダーは、今いる場所を手がかりに読む。突然フォーカスが飛ぶと文脈が切れる

> **参照**
> - [Understanding SC 3.2.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/on-focus.html#benefits)

## コンテキストの変化とは

大きな変化で、ページ全体を同時に見られない人が、気づかないと方向感覚を失うもの。

- **ユーザーエージェントの変化**：別アプリや別ブラウザが開く
- **ビューポートの変化**：新しいウィンドウ／タブ、フレームの切り替え
- **フォーカスの変化**：今操作していた部品から、別の部品へフォーカスが移る
- **ページの意味が変わるコンテンツの変化**：別ページへ進む、フォームが自動送信される、など

コンテンツの変化が、必ずしもコンテキストの変化ではない。

```text
コンテキストの変化（フォーカス時に起こしてはいけない）：
  フォーカスしただけでフォーム送信
  フォーカスしただけで新しいウィンドウ
  フォーカスしただけで別の部品へフォーカスが移る
  フォーカスしただけで別ページへ遷移

コンテキストの変化ではないことが多い例：
  アコーディオンが開く（フォーカスはそこに残る）
  動的メニューが展開する
  タブパネルが切り替わる（フォーカスが奪われなければ）
  入力欄の下に補足テキストが出る
```

> **参照**
> - [Understanding SC 3.2.1 — Key Terms: changes of context](https://www.w3.org/WAI/WCAG22/Understanding/on-focus.html#dfn-changes-of-context)

## フォーカスと活性化の違い

- **フォーカス**：Tab やクリックで、その部品が操作対象になること
- **活性化**：Enter / Space / ボタンのクリックなど、その部品の機能を実行すること
- マウスオーバーだけでは、通常フォーカスは動かない（スクリプトで動かした場合を除く）
- ボタンをクリックすると、フォーカスと活性化が同時に起きることがある。活性化による遷移は、この基準の対象外になりうる

```text
許可される流れ：
  Tab でセレクトに入る → まだ遷移しない
  選択肢を見て Escape / Tab で抜ける → 遷移しない
  Enter やボタンで確定する → そのとき遷移してよい
```

> **参照**
> - [Understanding SC 3.2.1 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/on-focus.html#intent)

## 手法：フォーカスではなく活性化で変える（G107）

- **G107**：コンテキストの変化は、フォーカスではなく活性化をきっかけにする
- リンクのクリック、送信ボタン、Enter / Space が典型
- `onFocus` / `onfocus` / `focus` イベントで送信・遷移・新規ウィンドウを起こさない

```html
<!-- 良い例：ボタンの活性化で次へ進む -->
<form action="/next" method="post">
  <label for="email">メールアドレス</label>
  <input id="email" name="email" type="email">
  <button type="submit">次へ</button>
</form>
```

```html
<!-- 悪い例：フォーカスしただけで送信 -->
<input
  type="text"
  name="done"
  onfocus="this.form.submit()"
>
```

```jsx
// 良い例：クリック（活性化）で遷移
function JumpMenu({ options }) {
  const [value, setValue] = useState("");

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        if (value) location.assign(value);
      }}
    >
      <label htmlFor="jump">移動先</label>
      <select
        id="jump"
        value={value}
        onChange={(e) => setValue(e.target.value)}
      >
        <option value="">選択してください</option>
        {options.map((o) => (
          <option key={o.href} value={o.href}>
            {o.label}
          </option>
        ))}
      </select>
      <button type="submit">移動</button>
    </form>
  );
}
```

```jsx
// 悪い例：フォーカスしただけで新しいページへ
function BadLink() {
  return (
    <a href="/help" onFocus={() => location.assign("/help")}>
      ヘルプ
    </a>
  );
}
```

> **参照**
> - [Technique G107: Using "activate" rather than "focus" as a trigger for changes of context](https://www.w3.org/WAI/WCAG22/Techniques/general/G107)

## よくある失敗：フォーカスをすぐ外す（F55）

- フォーカスを受けた瞬間に `blur()` すると、キーボードではその部品を操作できない
- システムフォーカスが見苦しいから消す、という理由で行われがち
- 2.1.1（キーボード）、2.4.7（フォーカスの可視化）、2.4.13（フォーカスの外観）も同時に失敗しうる

```html
<!-- 失敗例 -->
<input type="submit" onfocus="this.blur()">
<a href="/page.html" onfocus="this.blur()">詳細</a>
```

```jsx
// 失敗例
<button type="button" onFocus={(e) => e.currentTarget.blur()}>
  送信
</button>
```

```css
/* フォーカスを消したいなら、代替スタイルを付ける（2.4.7） */
button:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}
```

> **参照**
> - [Failure F55: using script to remove focus when focus is received](https://www.w3.org/WAI/WCAG22/Techniques/failures/F55)
> - [wcag/2-4-7-focus-visible.md — 2.4.7 フォーカスの可視化](./2-4-7-focus-visible.md)

## ヘルプダイアログの失敗例

- 入力欄にフォーカスしただけで説明ダイアログが開き、フォーカスが奪われる
- Tab でその欄を通り過ぎようとするたびに、ダイアログが割り込む
- 補足テキストを同じページに出すのはよい。フォーカスを奪うモーダルはだめ

```html
<!-- 良い例：フォーカスは入力欄に残し、説明は近くに出す -->
<label for="tel">電話番号</label>
<input id="tel" name="tel" type="tel" aria-describedby="tel-help">
<p id="tel-help">ハイフンなしの数字で入力してください。</p>
```

```html
<!-- 悪い例：フォーカス時にダイアログがフォーカスを奪う -->
<input id="tel" onfocus="openHelpDialog()">
```

## 3.2.2 との関係

| 達成基準 | きっかけ | 禁止すること |
|---|---|---|
| 3.2.1 フォーカス時 | フォーカスを受ける | コンテキストの変化 |
| 3.2.2 入力時 | 設定を変える（入力・選択） | ユーザーの予測できないコンテキストの変化 |

- 3.2.1 は「乗っただけ」で起きないこと
- 3.2.2 は「値を変えただけ」で勝手に遷移しないこと（送信ボタンなしのセレクト遷移など）
- 両方とも、明示的な活性化（送信・確定）なら変化してよい

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.2 On Input](https://www.w3.org/TR/WCAG22/#on-input)

## テスト・確認方法

### 手動テスト

- キーボードだけで、すべての操作可能な部品にフォーカスを移す
- フォーカスした瞬間に、送信・新規ウィンドウ・別部品へのフォーカス移動・ページ遷移が起きないか見る
- Escape や Tab で抜けられるドロップダウンが、抜けるだけで遷移しないか見る
- `onFocus` で `blur()` していないか見る

```text
チェックリスト：
1. Tab で一周して、フォーカス時にページが変わらないか
2. 新しいウィンドウやダイアログが、フォーカスだけで開かないか
3. フォーカスが勝手に別部品へ飛ばないか
4. onfocus="this.blur()" がないか
5. 遷移や送信は、クリック / Enter / 送信ボタンか
```

```javascript
// Console：onfocus で blur している要素がないか、ざっと探す手がかり
[...document.querySelectorAll("[onfocus]")].map((el) => ({
  tag: el.tagName.toLowerCase(),
  onfocus: el.getAttribute("onfocus"),
}));
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.2.1 は、UI がフォーカスを受けたときに**コンテキストの変化を起こさない**達成基準（レベル A）
- 送信・新規ウィンドウ・フォーカス移動・ページ遷移は、フォーカスではなく活性化で行う
- メニューの展開や補足テキストの表示は、フォーカスが奪われなければ許されやすい
- `onFocus` での `blur()` は、キーボード操作自体を壊す失敗
- 値の変更による予期しない遷移は、次の 3.2.2 の範囲
