# 2.2.3 タイミング非依存（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.2.3「タイミング非依存」（No Timing）
- レベル **AAA**（最高の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.2「十分な時間」に属する
- コンテンツが提示するイベントや活動で、**タイミングを不可欠な要件にしない**こと
- 例外は、**非インタラクティブな同期メディア**と**リアルタイムイベント**

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.3 No Timing](https://www.w3.org/TR/WCAG22/#no-timing)
> - [Understanding SC 2.2.3 No Timing](https://www.w3.org/WAI/WCAG22/Understanding/no-timing.html)

## 達成基準の原文（要約）

- コンテンツが提示するイベントや活動において、タイミングが不可欠な要件ではないこと
- 例外
  - **非インタラクティブな同期メディア**
  - **リアルタイムイベント**

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.3](https://www.w3.org/TR/WCAG22/#no-timing)

## 2.2.1 との違い

| 達成基準 | レベル | 主な要件 | 例外 |
|---|---|---|---|
| 2.2.1 タイミング調整可能 | A | オフ・調整・延長のいずれか | リアルタイム、不可欠、20時間超 |
| 2.2.3 タイミング非依存 | AAA | タイミングを活動の不可欠な要件にしない | 非インタラクティブ同期メディア、リアルタイム |

- 2.2.1 は、時間制限があってもユーザーが調整できれば適合可能
- 2.2.3 は、ユーザーの成果や活動完了を**時間に依存させない**上位基準
- 2.2.1 の「不可欠な時間制限」と「20時間超」の例外は、2.2.3 では認められない
- 2.2.3 を満たす設計では、可能な限り時間制限そのものを設けない

```text
2.2.1 では適合しうるが、2.2.3 では不適合になりうる例：
  15分の入力制限を、事前設定で150分まで延長可能
  20時間を超える時間制限
  制限時間を延ばすと成立しない一般的な活動

2.2.3 に適合しやすい例：
  入力完了まで時間制限のないフォーム
  回答時間が得点に影響しない試験
  一時停止できるターン制ゲーム
```

> **参照**
> - [wcag/2-2-1-timing-adjustable.md — 2.2.1 タイミング調整可能](./2-2-1-timing-adjustable.md)

## なぜ必要か

- 身体障害のあるユーザーは、反応・入力・操作により長い時間が必要なこと
- ロービジョンのユーザーは、画面上の情報を探して読むのに時間が必要なこと
- スクリーンリーダー利用者は、レイアウトの把握やコントロールの操作に時間が必要なこと
- 認知・言語・学習に関する障害のあるユーザーは、情報の読解や理解に時間が必要なこと
- 手話通訳を介して音声情報を受け取る場合も、追加の時間が必要なこと

> **参照**
> - [Understanding SC 2.2.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/no-timing.html#intent)
> - [Understanding SC 2.2.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/no-timing.html#benefits)

## 基本方針：時間制限を設けない（G5）

- フォーム、申請、試験、学習コンテンツなどを、ユーザーが自分のペースで完了できる設計
- 入力中のデータを自動保存し、途中から再開できる仕組み
- 読む速度や操作速度を評価結果に含めない設計
- タイマーを使う必要がある場合も、時間切れで成果・進捗を失わせない設計

```html
<!-- 良い例：時間制限のないフォーム -->
<form action="/application" method="post">
  <label for="reason">申請理由</label>
  <textarea id="reason" name="reason"></textarea>
  <button type="submit">申請する</button>
</form>
```

```javascript
// 良い例：入力途中の内容を保存
const form = document.querySelector("form");

form.addEventListener("input", () => {
  const data = new FormData(form);
  saveDraft(Object.fromEntries(data));
});
```

> **参照**
> - [Technique G5: Allowing users to complete an activity without any time limit](https://www.w3.org/WAI/WCAG22/Techniques/general/G5)

## 試験・クイズ

- 回答時間を得点や合否に影響させない
- 制限時間なしで受験できる設計
- 速度を測定することが学習目的でない限り、カウントダウンを不可欠にしない
- 第三者が延長時間を設定する方法は2.2.1では有効でも、2.2.3では時間依存自体を避ける必要

```html
<!-- 悪い例：時間切れで自動提出され、回答時間も得点に影響 -->
<p role="timer" aria-live="polite">残り時間：5分</p>

<!-- 良い例：時間制限なし -->
<form>
  <fieldset>
    <legend>HTMLの見出し要素を選んでください</legend>
    <label><input type="radio" name="answer" value="h1"> h1</label>
    <label><input type="radio" name="answer" value="div"> div</label>
  </fieldset>
  <button type="submit">回答を送信</button>
</form>
```

> **参照**
> - [Understanding SC 2.2.3 — Test example](https://www.w3.org/WAI/WCAG22/Understanding/no-timing.html#examples)

## ゲーム・対戦

- リアルタイム競争ではなく、ターン制にすることで時間依存を避ける方法
- 各プレイヤーが操作を完了するまで次へ進まない設計
- 一時停止しても競技結果が無効にならないルール

```text
悪い例：
  10秒以内に操作しないと自動的に敗北するゲーム

良い例：
  プレイヤーが操作を確定してから相手のターンへ進む
  ユーザーが任意に一時停止・再開できる
```

> **参照**
> - [Understanding SC 2.2.3 — Turn-based game example](https://www.w3.org/WAI/WCAG22/Understanding/no-timing.html#examples)

## セッションタイムアウト

- セキュリティ上の理由でセッションを終了する場合でも、活動の完了を時間に依存させない工夫
- 再認証後に、失われていない入力データや進捗から続行できる設計
- 入力途中の内容をサーバーまたは安全なローカル領域へ保存
- タイムアウトによって入力内容や購入手続きの進捗が失われると、時間依存の活動になりやすい

```text
望ましい流れ：
  1. 一定時間の無操作でセッション終了
  2. 再認証を要求
  3. 保存済みの入力内容と同じ手順へ復帰
  4. 時間切れを理由に活動を最初からやり直させない
```

- 再認証後のデータ保持は2.2.5「再認証」とも関係

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.5 Re-authenticating](https://www.w3.org/TR/WCAG22/#re-authenticating)

## 自動リダイレクト・自動更新

- 時間経過後の強制リダイレクトやページ更新を避ける
- リンクやボタンでユーザーが移動・更新する設計
- 即時のHTTPリダイレクトは時間制限ではないが、数秒後の `meta refresh` は時間依存

```html
<!-- 悪い例：5秒後に強制移動 -->
<meta http-equiv="refresh" content="5;url=/next">

<!-- 良い例：ユーザーが移動を選択 -->
<p>ページの移転先を確認してください。</p>
<a href="/next">新しいページへ移動する</a>
```

> **参照**
> - [Failure F40: using meta redirect with a time limit](https://www.w3.org/WAI/WCAG22/Techniques/failures/F40)
> - [Failure F41: using meta refresh to reload the page](https://www.w3.org/WAI/WCAG22/Techniques/failures/F41)

## 例外：非インタラクティブな同期メディア

- 音声・映像が時間軸に沿って同期するメディア
- ユーザーの時間制限付き操作を要求しないもの
- 例
  - 収録済み動画
  - 映像と音声が同期したプレゼンテーション
- 再生時間そのものはメディアの本質であるため例外
- 一時停止・停止・シークなど、他のメディア要件は別途必要

```html
<!-- 例外：非インタラクティブな同期メディア -->
<video controls>
  <source src="lecture.mp4" type="video/mp4">
  <track kind="captions" src="lecture-ja.vtt" srclang="ja" label="日本語">
</video>
```

- 手話などのビデオのみのコンテンツは、ガイドライン1.1でも扱われる

> **参照**
> - [Understanding SC 2.2.3 — synchronized media](https://www.w3.org/WAI/WCAG22/Understanding/no-timing.html#dfn-synchronized-media)
> - [WCAG 2.2 — Guideline 1.1 Text Alternatives](https://www.w3.org/TR/WCAG22/#text-alternatives)

## 例外：リアルタイムイベント

- 視聴と同時に発生し、コンテンツが完全には生成していないイベント
- 例
  - ライブオークション
  - ライブ配信
  - 現実の競技や会議の生中継
- イベント自体の時間進行を個別ユーザーのために止められない場合は例外
- 録画・議事録・結果一覧など、後から利用できる代替を提供すると望ましい

```text
例外になりうる：
  ライブオークションの入札締切
  現実のスポーツ大会の生中継

例外にならない：
  コンテンツ側で生成した一般的なクイズの制限時間
  ターン制へ変更できるゲーム
```

> **参照**
> - [Understanding SC 2.2.3 — real-time event](https://www.w3.org/WAI/WCAG22/Understanding/no-timing.html#dfn-real-time-event)

## 例外を広く解釈しない

- 「開発上便利」「業務上一般的」「緊張感を出したい」は、タイミングを不可欠にする根拠にならない
- 時間制限を削除しても活動の情報・機能を維持できるなら、不可欠ではない
- AAA適合を主張する場合は、各時間制限について例外の根拠を明確化

```text
例外にならない理由の例：
  「10分でフォームを入力してもらう仕様だから」
  「競争感を演出したいから」
  「サーバー負荷を減らしたいから」
```

## 2.2.2 との関係

| 達成基準 | 対象 | 主な要件 |
|---|---|---|
| 2.2.2 一時停止・停止・非表示 | 動き・点滅・スクロール・自動更新 | ユーザーが動きを制御 |
| 2.2.3 タイミング非依存 | イベント・活動の時間依存 | タイミングを不可欠にしない |

- 自動で進むカルーセルは2.2.2で一時停止等を確認
- 制限時間内に操作しないと成果を失う活動は2.2.3で確認
- 同じコンテンツに両方が適用される場合がある

> **参照**
> - [wcag/2-2-2-pause-stop-hide.md — 2.2.2 一時停止・停止・非表示](./2-2-2-pause-stop-hide.md)

## よくある失敗例

- フォームを一定時間内に送信しないと入力内容が失われる
- 試験の回答速度が得点や合否に影響する
- 一般的なゲームで制限時間を不可欠にしている
- 20時間を超える制限なので2.2.3にも適合すると誤認する
- 2.2.1の延長機能だけで、2.2.3にも適合すると誤認する
- 数秒後の `meta refresh` で強制移動する

```javascript
// 悪い例：時間切れで入力内容を破棄
setTimeout(() => {
  form.reset();
  location.href = "/timeout";
}, 10 * 60 * 1000);

// 良い例：自動保存し、時間制限なしで続行
form.addEventListener("input", saveDraft);
```

## テスト・確認方法

### 手動テスト

- ページ内のフォーム、試験、ゲーム、予約、購入、セッションを確認
- 制限時間、カウントダウン、自動提出、自動終了がないか確認
- 時間切れで情報・進捗・機能が失われないか確認
- 例外の場合、非インタラクティブ同期メディアまたはリアルタイムイベントか確認

```text
チェックリスト：
1. ユーザー操作に制限時間があるか
2. 回答速度や完了時間が成果に影響するか
3. 時間切れで入力・進捗が失われるか
4. タイミングを取り除いても活動が成立するか
5. 例外は同期メディアまたはリアルタイムイベントか
```

### 開発者ツールでの確認

```bash
# 時間制限になりうる実装を確認
rg "setTimeout|setInterval|meta.*refresh|timeout|countdown|expires" --glob "*.html" --glob "*.js" --glob "*.tsx" --glob "*.jsx"
```

- 自動検出だけでは、タイミングが活動に不可欠か判断できない
- 仕様書・業務要件を含む手動レビューが必要

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.2.3 は、イベントや活動で**タイミングを不可欠な要件にしない**達成基準（レベル AAA）
- 2.2.1のオフ・調整・延長より厳しく、基本方針は時間制限そのものを設けないこと
- 例外は、非インタラクティブな同期メディアとリアルタイムイベント
- フォーム、試験、ゲームは、時間切れで成果や進捗を失わせない設計
- セッション終了が必要でも、自動保存と再認証後の再開で活動を時間非依存に近づける
- 2.2.1（時間制限の調整）・2.2.2（動きの制御）と併せて確認する
