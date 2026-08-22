# 3.2.2 入力時（A）

## 達成基準の概要

- WCAG 2.2 達成基準 3.2.2「入力時」（On Input）
- レベル **A**（最低限の適合レベル）
- 原則3「理解可能（Understandable）」> ガイドライン 3.2「予測可能」に属する
- UI コンポーネントの**設定を変えただけ**で、自動的にコンテキストの変化を起こさないこと
- 変化が起きるなら、**使う前に**その挙動をユーザーに知らせていること
- 送信ボタンや明示的な活性化で変えるのが典型

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.2 On Input](https://www.w3.org/TR/WCAG22/#on-input)
> - [Understanding SC 3.2.2 On Input](https://www.w3.org/WAI/WCAG22/Understanding/on-input.html)

## 達成基準の原文（要約）

- いずれのユーザーインターフェースコンポーネントの設定を変えても、ユーザーがそのコンポーネントを使う前にその挙動について知らされている場合を除き、自動的にコンテキストの変化を起こさないこと

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.2](https://www.w3.org/TR/WCAG22/#on-input)

## なぜ必要か

- チェックや選択、入力のたびにページが変わると、方向感覚を失いやすい
- 視覚障害のある人は、新しいウィンドウや画面遷移に気づきにくい
- 認知の制約がある人は、予期しない変化で操作をやり直しにくい
- キーボードで `<select>` を移動すると、選択値が変わるたびにイベントが走る
- 戻るボタンの挙動が変わると、さらに混乱する

> **参照**
> - [Understanding SC 3.2.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/on-input.html#benefits)

## 「設定を変える」とは

- ユーザーが部品とやり取りしたあとも残る、部品の状態・値の変化
- チェックボックスのオン／オフ、テキスト入力、`<select>` の選択変更など
- リンクのクリックや送信ボタンの押下は、**活性化**であり、設定変更とは限らない

```text
設定を変える例：
  チェックボックスをオンにする
  テキストフィールドに文字を入力する
  セレクトの選択肢を変える
  aria-pressed でトグル状態を変える

設定を変えない例（活性化）：
  リンクをクリックする
  フォームを送信するボタンを押す
  モーダルを開くボタンを押す
```

> **参照**
> - [Understanding SC 3.2.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/on-input.html#intent)

## コンテキストの変化とは

- 3.2.1 と同じく、ユーザーエージェント・ビューポート・フォーカス・ページの意味が変わる大きな変化
- コンテンツが増えるだけでは、必ずしもコンテキストの変化ではない
- 同じページ内で追加フィールドが出る、タブが切り替わる、などは許されやすい

```text
コンテキストの変化（入力だけで起こしてはいけない）：
  セレクト変更で即ページ遷移
  最後の入力欄を離れただけで自動送信
  ラジオ選択で新しいウィンドウが開く

コンテキストの変化ではないことが多い例：
  予定種別を選ぶと、同じフォーム内に参加者欄が出る
  大陸を選ぶと、国の選択肢が更新される
  電話番号の1欄を埋めると、次の欄へフォーカスが移る（事前説明あり）
```

> **参照**
> - [Understanding SC 3.2.2 — Key Terms: changes of context](https://www.w3.org/WAI/WCAG22/Understanding/on-input.html#dfn-changes-of-context)

## 手法1：送信ボタンで変化させる（G80 + H84）

- **G80**：コンテキストの変化は送信ボタンで起こす
- **H84**：`<select>` と `<button>` を組み合わせ、選択だけでは実行しない
- キーボード利用者が選択肢を見ても、意図しない遷移が起きない

```html
<!-- 良い例：選択後、ボタンで確定 -->
<form action="/calendar" method="get">
  <label for="quarter">四半期</label>
  <select id="quarter" name="quarter">
    <option value="1">第1四半期（1〜3月）</option>
    <option value="2">第2四半期（4〜6月）</option>
    <option value="3">第3四半期（7〜9月）</option>
    <option value="4">第4四半期（10〜12月）</option>
  </select>

  <label for="year">年</label>
  <input id="year" name="year" type="text">

  <button type="submit">表示</button>
</form>
```

```html
<!-- 悪い例：選択した瞬間に送信 -->
<form action="/jump" method="get">
  <label for="page">移動先</label>
  <select id="page" name="page" onchange="this.form.submit()">
    <option value="/home">ホーム</option>
    <option value="/about">会社概要</option>
    <option value="/contact">お問い合わせ</option>
  </select>
</form>
```

```jsx
// 良い例：React でも onChange では遷移しない
function LanguagePicker() {
  const [lang, setLang] = useState("ja");

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        location.assign(`/${lang}`);
      }}
    >
      <label htmlFor="lang">言語</label>
      <select id="lang" value={lang} onChange={(e) => setLang(e.target.value)}>
        <option value="ja">日本語</option>
        <option value="en">English</option>
      </select>
      <button type="submit">切り替え</button>
    </form>
  );
}
```

> **参照**
> - [Technique G80: Providing a submit button to initiate a change of context](https://www.w3.org/WAI/WCAG22/Techniques/general/G80)
> - [Technique H84: Using a button with a select element to perform an action](https://www.w3.org/WAI/WCAG22/Techniques/html/H84)

## 手法2：事前に挙動を説明する（G13）

- 設定変更でコンテキストが変わるなら、**使う前に**説明する
- 説明は、その部品より前の読み順に置く
- 可能なら `aria-describedby` などで部品と関連付ける

```html
<!-- 良い例：言語切替の前に説明 -->
<p id="lang-note">言語を選ぶと、ページ全体がその言語に切り替わります。</p>
<label for="lang">言語</label>
<select id="lang" name="lang" aria-describedby="lang-note" onchange="this.form.submit()">
  <option value="de">Deutsch</option>
  <option value="fr">Français</option>
  <option value="ja">日本語</option>
</select>
```

```html
<!-- 良い例：1問ずつ進むアンケート -->
<p id="survey-note">回答を選ぶと、次の質問へ進みます。</p>
<fieldset aria-describedby="survey-note">
  <legend>Q1. 満足度は？</legend>
  <!-- トグルボタンで回答 -->
</fieldset>
```

```html
<!-- 悪い例：説明なしで即遷移 -->
<select onchange="location.assign(this.value)">
  <option value="/">ホーム</option>
  <option value="/help">ヘルプ</option>
</select>
```

> **参照**
> - [Technique G13: Describing what will happen before a change to a form control that causes a change of context to occur is made](https://www.w3.org/WAI/WCAG22/Techniques/general/G13)

## 手法3：onchange でもコンテキストは変えない（SCR19）

- **SCR19**：`<select>` の `onchange` で、別の `<select>` の選択肢を更新する
- 同じページ内の連動更新なら、コンテキストの変化にならないことが多い
- 更新される部品は、トリガーより**後**の読み順に置く

```html
<!-- 良い例：大陸を選ぶと国リストが更新される -->
<label for="continent">大陸</label>
<select id="continent" onchange="updateCountries(this)">
  <option value="">選択してください</option>
  <option value="asia">アジア</option>
  <option value="europe">ヨーロッパ</option>
</select>

<label for="country">国</label>
<select id="country">
  <option value="">国を選択</option>
</select>
```

```html
<!-- 良い例：同じフォーム内で追加フィールドを出す -->
<label for="type">予定の種類</label>
<select id="type" name="type">
  <option value="meeting">会議</option>
  <option value="reminder">リマインダー</option>
</select>

<div id="participants" hidden>
  <label for="attendees">参加者</label>
  <input id="attendees" name="attendees" type="text">
</div>
```

> **参照**
> - [Technique SCR19: Using an onchange event on a select element without causing a change of context](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR19)

## 3.2.1 との関係

| 達成基準 | きっかけ | 禁止・注意 |
|---|---|---|
| 3.2.1 フォーカス時 | フォーカスを受ける | コンテキストの変化を起こさない |
| 3.2.2 入力時 | 設定を変える | 自動的なコンテキストの変化を起こさない（事前説明があれば可） |

- 3.2.1 は「乗っただけ」
- 3.2.2 は「値を変えただけ」
- どちらも、明示的な送信・確定ボタンなら変化してよい

> **参照**
> - [wcag/3-2-1-on-focus.md — 3.2.1 フォーカス時](./3-2-1-on-focus.md)

## よくある失敗例

### F36：最後の入力欄で自動送信

- 最後の欄を離れただけで `form.submit()` する
- 説明を読むために Tab で戻ったときにも送信される

```html
<!-- 失敗例 -->
<form method="get" id="phone-form">
  <input name="area" maxlength="3"> -
  <input name="prefix" maxlength="3"> -
  <input name="line" maxlength="4" onchange="phone-form.submit()">
</form>
```

### F37：選択変更で新しいウィンドウ

- ラジオ・チェック・セレクトの変更で `window.open()` する
- 事前警告がなければ失敗
- ラジオなら警告があっても、キーボードで選択肢を見るだけで発火し、2.1.1 も問題になりうる

```html
<!-- 失敗例 -->
<input type="radio" onclick="window.open('https://example.com')">
```

```html
<!-- 改善例：ボタンで開く -->
<input type="radio" id="mirror-a" name="mirror" value="a">
<label for="mirror-a">ミラー A</label>
<button type="button" onclick="window.open('https://a.example.com')">
  ミラー A を開く
</button>
```

> **参照**
> - [Failure F36: automatically submitting a form when the last field is given a value](https://www.w3.org/WAI/WCAG22/Techniques/failures/F36)
> - [Failure F37: launching a new window when the selection of a radio button, check box or select list is changed](https://www.w3.org/WAI/WCAG22/Techniques/failures/F37)

## テスト・確認方法

### 手動テスト

- フォームの各コントロールで、値を変えてみる
- 送信ボタンを押さずに、ページ遷移・新規ウィンドウ・自動送信が起きないか確認する
- 変化するなら、事前説明があるか確認する
- キーボードだけで `<select>` の選択肢を上下に移しても、遷移しないか確認する

```text
チェックリスト：
1. セレクト変更だけでページが変わらないか
2. 最後の入力欄を離れただけで送信されないか
3. ラジオ・チェック変更で新しいウィンドウが開かないか
4. 変化するなら、部品の前に説明があるか
5. 同じページ内の連動更新なら、文脈は保たれているか
6. 遷移は送信ボタンや明示的な確定操作か
```

```javascript
// Console：onchange で submit / location / window.open していないか
[...document.querySelectorAll("[onchange]")].map((el) => ({
  tag: el.tagName.toLowerCase(),
  onchange: el.getAttribute("onchange"),
}));
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.2.2 は、UI の**設定を変えただけ**でコンテキストが勝手に変わらないようにする達成基準（レベル A）
- ページ遷移や送信は、送信ボタンや明示的な確定操作で行う
- 変化が起きるなら、使う前に説明する（G13）
- 同じページ内の連動更新や追加フィールド表示は、多くの場合許される
- 自動送信・選択即遷移・選択で新規ウィンドウは典型的な失敗
