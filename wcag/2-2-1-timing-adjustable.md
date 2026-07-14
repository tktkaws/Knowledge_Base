# 2.2.1 タイミング調整可能（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.2.1「タイミング調整可能」（Timing Adjustable）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.2「十分な時間」に属する
- コンテンツが設定する**時間制限**ごとに、次のいずれかが真であること
  - **オフにできる**
  - **調整できる**
  - **延長できる**
  - **リアルタイム例外** / **不可欠例外** / **20時間例外**

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.1 Timing Adjustable](https://www.w3.org/TR/WCAG22/#timing-adjustable)
> - [Understanding SC 2.2.1 Timing Adjustable](https://www.w3.org/WAI/WCAG22/Understanding/timing-adjustable.html)

## 達成基準の原文（要約）

- コンテンツが設定する各時間制限について、次のいずれかが真であること
  - **オフ**：ユーザーが時間制限に遭遇する前に、制限をオフにできる
  - **調整**：ユーザーが遭遇する前に、デフォルトの**10倍以上**の範囲で時間を調整できる
  - **延長**：制限が切れる前に警告され、簡単な操作（例：Space キー）で**20秒以上**かけて延長でき、少なくとも**10回**延長できる
  - **リアルタイム例外**：オークションなどリアルタイムイベントの不可欠な一部で、代替が不可能
  - **不可欠例外**：時間制限が不可欠で、延長すると活動が無効になる
  - **20時間例外**：時間制限が20時間より長い

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.1](https://www.w3.org/TR/WCAG22/#timing-adjustable)

## 時間制限とは

- ユーザー操作なしに一定時間後、または定期的に起きる処理
- 例
  - セッションタイムアウト
  - ページの自動更新・自動リダイレクト
  - 入力を求める時間枠の終了
  - 読むのに時間がかかるペースで進む・スクロールするコンテンツ
- ユーザーが開始していないタイマーが対象
- 作者／コンテンツが設定した制限が対象（ユーザーエージェントやインターネット固有の制限は対象外）

```text
対象の例：
  ログイン後 15 分でセッション切断
  5 秒後に meta refresh で別ページへ移動
  確認画面で 2 分以内に購入を確定

対象になりにくい例：
  ユーザーが制御できるカルーセル（2.2.2 で扱う）
  繰り返すスクロール文字で一時停止できるもの（2.2.2）
  即時の 3xx リダイレクト（時間制限がない）
```

> **参照**
> - [Understanding SC 2.2.1 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/timing-adjustable.html#intent)

## なぜ必要か

- 身体障害のあるユーザーは、反応・入力・操作に時間がかかることがある
- ロービジョンのユーザーは、画面上の位置探しや読解に時間がかかる
- スクリーンリーダー利用者は、レイアウト把握や操作に時間がかかる
- 認知・言語の制約があるユーザーは、読解・理解に時間がかかる
- 手話通訳を介して利用する場合も、時間制御が重要

> **参照**
> - [Understanding SC 2.2.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/timing-adjustable.html#benefits)

## 優先順位：オフ > 調整 > 延長

- ユーザーにとって最も助かるのは**オフ**
- 次に、遭遇前の**調整**（デフォルトの10倍以上）
- 最後に、期限前の警告と**延長**
- 可能なら時間依存にしない設計が望ましい

```text
推奨順：
1. 時間制限を設けない
2. オフにできる
3. 10倍以上に調整できる
4. 警告 + 延長（20秒以上・10回以上）
```

## 条件：オフにできる（G198）

- 時間制限に遭遇する前に、制限をオフにできる仕組みを提供する
- マルチパートフォームの最初のページでチェックボックスを置く手法もある（G133）

```html
<!-- 良い例：セッション延長／制限なしを最初に選べる -->
<form action="/application" method="post">
  <fieldset>
    <legend>セッション時間</legend>
    <label>
      <input type="checkbox" name="no-timeout" value="1">
      セッション時間制限をオフにする
    </label>
  </fieldset>
  <button type="submit">申し込みを開始</button>
</form>
```

> **参照**
> - [Technique G198: Providing a way for the user to turn the time limit off](https://www.w3.org/WAI/WCAG22/Techniques/general/G198)
> - [Technique G133: Providing a checkbox on the first page of a multipart form](https://www.w3.org/WAI/WCAG22/Techniques/general/G133)

## 条件：調整できる（G180）

- デフォルト時間の**少なくとも10倍**まで、遭遇前に調整できること
- 例：デフォルト15秒なら、150秒以上まで設定可能

```html
<!-- 良い例：更新間隔をデフォルトの10倍まで調整 -->
<label for="refresh-interval">ヘッドライン更新間隔</label>
<select id="refresh-interval">
  <option value="5">5秒（デフォルト）</option>
  <option value="15">15秒</option>
  <option value="30">30秒</option>
  <option value="50">50秒（10倍）</option>
</select>
```

> **参照**
> - [Technique G180: Providing the user with a means to set the time limit to 10 times the default](https://www.w3.org/WAI/WCAG22/Techniques/general/G180)

## 条件：延長できる（SCR16 + SCR1）

- 制限が切れる前に警告する
- 簡単な操作で、少なくとも**20秒**かけて延長できる
- 少なくとも**10回**延長できる
- 「Space を押す」「延長ボタンをクリック」など、単純な操作にする

```html
<!-- 良い例：セッション延長の警告 -->
<dialog id="timeout-warning" open>
  <p>あと1分でセッションが切れます。</p>
  <button type="button" id="extend-session">時間を延長する</button>
</dialog>
```

```javascript
const warning = document.getElementById("timeout-warning");
const extendButton = document.getElementById("extend-session");
let extendCount = 0;

function showWarning() {
  warning.showModal();
  // 少なくとも20秒は応答を待つ
}

extendButton.addEventListener("click", () => {
  if (extendCount >= 10) return;
  extendCount += 1;
  resetSessionTimer();
  warning.close();
});

document.addEventListener("keydown", (event) => {
  if (warning.open && event.key === " ") {
    event.preventDefault();
    extendButton.click();
  }
});
```

> **参照**
> - [Technique SCR16: Providing a script that warns the user a time limit is about to expire](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR16)
> - [Technique SCR1: Allowing the user to extend the default time limit](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR1)

## 読む速度に関する時間制限

- スクロール・移動・自動更新で読む時間が限られる場合も対象になりうる
- 一時停止して再開できる仕組みがあれば適合しやすい（G4）
- コントロールがあるカルーセルや繰り返すスクロールは、2.2.2 との兼ね合いで本 SC が適用されないことがある

```html
<!-- 良い例：一時停止できる流れるテキスト -->
<div class="ticker" aria-live="off">
  <p id="ticker-text">本日のヘッドライン：...</p>
</div>
<button type="button" id="pause-ticker" aria-pressed="false">一時停止</button>
```

```javascript
const pauseButton = document.getElementById("pause-ticker");
let paused = false;

pauseButton.addEventListener("click", () => {
  paused = !paused;
  pauseButton.setAttribute("aria-pressed", String(paused));
  pauseButton.textContent = paused ? "再開" : "一時停止";
  ticker.setAttribute("data-paused", String(paused));
});
```

> **参照**
> - [Technique G4: Allowing the content to be paused and restarted](https://www.w3.org/WAI/WCAG22/Techniques/general/G4)
> - [Technique SCR33: Using script to scroll content, and providing a mechanism to pause it](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR33)

## 例外：リアルタイム

- オークションなど、リアルタイムイベントの不可欠な一部である時間制限
- 他の入札者と同じ時間制約が必要なため、特定ユーザーだけ延長できない
- 代替が不可能な場合に適用

```text
例外の例：
  オンラインオークションの入札締切

例外にならない例：
  一般的なフォーム入力のセッションタイムアウト
```

> **参照**
> - [Understanding SC 2.2.1 — auction example](https://www.w3.org/WAI/WCAG22/Understanding/timing-adjustable.html#examples)

## 例外：不可欠

- 時間制限が活動の本質で、延長するとその活動が無効になる場合
- 例：チケット販売で座席を短時間だけ確保する
- 時間的に重要な段階の前に、氏名・支払方法などの入力を済ませられる設計にするとよい

```text
例外の例：
  チケット購入サイトで、選定座席の確保時間が2分
  （長く確保すると他ユーザーが買えない）

望ましい工夫：
  時間制限前に必要な情報入力を済ませる
  可能な範囲で延長手段を提供する
```

> **参照**
> - [Understanding SC 2.2.1 — Essential Exception](https://www.w3.org/WAI/WCAG22/Understanding/timing-adjustable.html#success-criterion)

## 例外：20時間

- 時間制限が**20時間より長い**場合は例外
- 起床して活動する1日より長い時間を目安に選ばれた上限

## 時間制限がない代替手段がある場合

- タイマー付きの表示があっても、同じ情報・機能を別手段で得られるなら、そのタイマー自体は本 SC の対象にならないことがある
- 例：新着メールのトーストが5秒で消えても、受信箱で同じ情報を確認できる

```html
<!-- 適合しうる例：トーストは補助、本体は受信箱 -->
<div role="status">新着メールが1件あります</div>
<!-- ユーザーは受信箱で内容と件数を確認できる -->
```

> **参照**
> - [Understanding SC 2.2.1 — toast message example](https://www.w3.org/WAI/WCAG22/Understanding/timing-adjustable.html#intent)

## セキュリティ関連の時間制限

- セキュリティ目的の時間制限も、コンテンツが設定したものなら本 SC の対象になりうる
- 時間ベースの二要素認証トークンなど、延長すると無効になるものは**不可欠例外**になりうる
- 3.3.7（冗長な入力）、3.3.8 / 3.3.9（認証のアクセシビリティ）も併せて確認

> **参照**
> - [Understanding SC 2.2.1 — Notes regarding time limits](https://www.w3.org/WAI/WCAG22/Understanding/timing-adjustable.html#intent)

## よくある失敗例

### F40：時間制限付き meta redirect

```html
<!-- 悪い例：5秒後に強制移動 -->
<meta http-equiv="refresh" content="5;url=/next-page">
```

### F41：meta refresh によるページ再読み込み

```html
<!-- 悪い例：30秒ごとに自動再読み込み -->
<meta http-equiv="refresh" content="30">
```

### F58：サーバー側の時間切れリダイレクト

- サーバー側でタイムアウト後に自動リダイレクトし、延長手段がない

> **参照**
> - [Failure F40: using meta redirect with a time limit](https://www.w3.org/WAI/WCAG22/Techniques/failures/F40)
> - [Failure F41: using meta refresh to reload the page](https://www.w3.org/WAI/WCAG22/Techniques/failures/F41)
> - [Failure F58: using server-side techniques to automatically redirect pages after a time-out](https://www.w3.org/WAI/WCAG22/Techniques/failures/F58)

## 2.2.2・2.2.3・3.2.1 との関係

| 達成基準 | 関係 |
|---|---|
| 2.2.2 一時停止・停止・非表示 | 動く・点滅・スクロール・自動更新の制御 |
| 2.2.3 タイミング非依存 | AAA、時間制限そのものをなくす |
| 3.2.1 フォーカス時 | ユーザー操作によるコンテキスト変化の制限 |

- カルーセルやキャプションなど、ユーザーが制御できる繰り返しコンテンツは 2.2.2 側で扱うことが多い
- 可能な限り時間制限を設けない設計は 2.2.3（AAA）に近づく

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.2 Pause, Stop, Hide](https://www.w3.org/TR/WCAG22/#pause-stop-hide)
> - [WCAG 2.2 — Success Criterion 2.2.3 No Timing](https://www.w3.org/TR/WCAG22/#no-timing)

## テスト・確認方法

### 手動テスト

- セッションタイムアウト、自動更新、自動リダイレクト、流れ読みコンテンツを洗い出す
- 各時間制限について、オフ・調整・延長のいずれがあるか確認
- 例外（リアルタイム／不可欠／20時間）に該当するか確認
- 延長がある場合、警告タイミング・20秒以上の猶予・10回以上の延長を確認

```text
チェックリスト：
1. 時間制限があるか
2. 遭遇前にオフ／調整できるか
3. 期限前警告と簡単な延長があるか
4. 延長は10回以上可能か
5. 例外に該当する場合、根拠は明確か
6. meta refresh / 自動リダイレクトがないか
```

### 開発者ツールでの確認

```bash
# 時間制限・自動更新の実装箇所を確認
rg "meta.*refresh|setTimeout|setInterval|session.*timeout|expires" --glob "*.html" --glob "*.js" --glob "*.tsx" --glob "*.jsx"
```

### ACT Rules

- [Meta element has no refresh delay](https://www.w3.org/WAI/standards-guidelines/act/rules/bc659a/)
- [Meta element has no refresh delay (no exception)](https://www.w3.org/WAI/standards-guidelines/act/rules/bisz58/)

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.2.1 は、コンテンツが設定する時間制限を**オフ・調整・延長**できるように求める達成基準（レベル A）
- 優先順位は、オフ > 調整（10倍以上）> 延長（警告・20秒・10回以上）
- オークションなどのリアルタイム、活動が無効になる不可欠な制限、20時間超は例外
- `meta refresh` やサーバー側の強制リダイレクトはよくある失敗
- 可能なら時間依存にしない設計が最善で、2.2.2・2.2.3 と併せて確認する
