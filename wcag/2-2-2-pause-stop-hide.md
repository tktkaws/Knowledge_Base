# 2.2.2 一時停止・停止・非表示（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.2.2「一時停止・停止・非表示」（Pause, Stop, Hide）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.2「十分な時間」に属する
- **動き・点滅・スクロール・自動更新**する情報について、ユーザーが制御できる仕組みを提供
- **非干渉要件**（適合要件5）：本基準を満たさないコンテンツは、ページ全体の適合を妨げる

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.2 Pause, Stop, Hide](https://www.w3.org/TR/WCAG22/#pause-stop-hide)
> - [Understanding SC 2.2.2 Pause, Stop, Hide](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html)

## 達成基準の原文（要約）

- 動き・点滅・スクロール・自動更新する情報について、次がすべて真であること

### 動き・点滅・スクロール

- 次のすべてに当てはまる場合、一時停止・停止・非表示の仕組みがあること
  1. **自動的に開始**する
  2. **5秒より長く**続く
  3. **他のコンテンツと並行**して提示される
- 例外：動き・点滅・スクロールが活動の**不可欠**な一部である場合

### 自動更新

- 次のすべてに当てはまる場合、一時停止・停止・非表示、または更新頻度の制御の仕組みがあること
  1. **自動的に開始**する
  2. **他のコンテンツと並行**して提示される
- 例外：自動更新が活動の**不可欠**な一部である場合
- 自動更新には5秒の例外はない

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.2](https://www.w3.org/TR/WCAG22/#pause-stop-hide)

## 対象となるコンテンツ

### 動き・点滅・スクロール

- 視覚的に動きを感じさせるコンテンツ
- 例
  - アニメーション
  - 動画・同期メディア
  - スクロールする株価ティッカー
  - リアルタイムゲームの動き
  - CSS / JS による点滅

### 自動更新

- 事前に決めた間隔で更新・消えるコンテンツ
- 例
  - 天気予報の自動更新
  - ニュース・株価の自動更新
  - 自動で進むカルーセルやメッセージ

```html
<!-- 対象になりうる：自動開始するカルーセル -->
<div class="carousel" data-autoplay="true">...</div>

<!-- 対象になりうる：流れ続けるティッカー -->
<div class="stock-ticker">日経平均 ...</div>
```

> **参照**
> - [Understanding SC 2.2.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html#intent)

## 「自動的に開始」とは

- ユーザーの**意図的な操作**（ボタンやリンクの起動）の直接結果ではない動き・更新
- 次も含む
  - ページ読み込み時に始まるアニメーション
  - ホバーやフォーカス、スクロールで間接的に始まる動き
- 間接的な開始は、2.3.3「インタラクションによるアニメーション」とも重なりうる

```text
自動開始の例：
  ページ表示と同時にカルーセルが回り始める
  見出しがホバーで点滅し始める

自動開始ではない例：
  「再生」ボタンを押してから動画が始まる
```

## なぜ必要か

- 動き続けるコンテンツは、注意・読解・操作の邪魔になる
- ADHD など注意に困難があるユーザーは、点滅や動きに気を取られやすい
- 静止テキストを速く読めないユーザーや、動くものを追いづらいユーザーの障害になる
- スクリーンリーダー利用時にも、更新や動きが妨げになることがある

> **参照**
> - [Understanding SC 2.2.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html#benefits)

## 5秒ルール（動き・点滅・スクロール）

- 自動開始し、5秒より長く続き、他コンテンツと並行する場合は制御が必要
- **5秒以内に自然に止まる**なら、一時停止機能は不要（G11）
- 5秒は注意を引くには十分で、待てば邪魔にならない長さとして選ばれた

```html
<!-- 良い例：5秒以内に点滅が止まる -->
<p class="blink-notice">送信ボタンを押してください</p>
```

```css
.blink-notice {
  animation: blink 0.5s steps(2, start) 10; /* 約5秒で終了 */
}

@keyframes blink {
  to { visibility: hidden; }
}
```

> **参照**
> - [Technique G11: Creating content that blinks for less than 5 seconds](https://www.w3.org/WAI/WCAG22/Techniques/general/G11)
> - [Technique SCR22: Using scripts to control blinking and stop it in five seconds or less](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR22)

## 一時停止・停止・非表示の仕組み（G4, G186）

- ユーザーがページを使い続けられる仕組みであること
- フォーカスがある間だけ止まり、フォーカスを外すと再開する仕組みは**不十分**
  - ページの他の部分を使えなくなるため
- ボタンによる一時停止が典型例

```html
<!-- 良い例：一時停止／再開ボタン -->
<div class="ticker-region">
  <div id="ticker" class="ticker" aria-live="off">
    本日のヘッドライン：...
  </div>
  <button type="button" id="toggle-ticker" aria-pressed="false">
    一時停止
  </button>
</div>
```

```javascript
const ticker = document.getElementById("ticker");
const toggle = document.getElementById("toggle-ticker");
let paused = false;

toggle.addEventListener("click", () => {
  paused = !paused;
  ticker.dataset.paused = String(paused);
  toggle.setAttribute("aria-pressed", String(paused));
  toggle.textContent = paused ? "再開" : "一時停止";
});
```

> **参照**
> - [Technique G4: Allowing the content to be paused and restarted](https://www.w3.org/WAI/WCAG22/Techniques/general/G4)
> - [Technique G186: Using a control that stops moving, blinking, or auto-updating content](https://www.w3.org/WAI/WCAG22/Techniques/general/G186)

## 再開の2パターン

| パターン | 向いている内容 | 例 |
|---|---|---|
| 一時停止した位置から再開 | 読み物・説明アニメーション | 解説動画、プロセス説明 |
| 最新状態へジャンプ | リアルタイム／ステータス情報 | 株価、天気、交通カメラ |

- 古い情報を再開後も出すと誤解を招くリアルタイム情報では、最新へジャンプ／非表示が適切
- 一時停止中に受け取ったストリーム情報をすべて保持する必要はない（注記3）

```html
<!-- 良い例：株価ティッカーに一時停止と最新へ進む -->
<div class="stock-ticker" id="stocks">...</div>
<button type="button" id="pause-stocks">一時停止</button>
<button type="button" id="latest-stocks">最新へ進む</button>
```

> **参照**
> - [Understanding SC 2.2.2 — Pausing and resuming](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html#intent)

## カルーセル・自動スライド

- 自動で進むカルーセルは、一時停止・停止・非表示、または更新頻度の制御が必要
- 再生／一時停止ボタンを用意する
- ユーザーが次へ／前へ操作したときは、自動再生を止めるとよい

```html
<section class="carousel" aria-roledescription="カルーセル" aria-label="おすすめ">
  <div class="slides">...</div>
  <button type="button" id="carousel-pause" aria-pressed="false">
    自動再生を一時停止
  </button>
  <button type="button" aria-label="前のスライド">前へ</button>
  <button type="button" aria-label="次のスライド">次へ</button>
</section>
```

```javascript
const pauseBtn = document.getElementById("carousel-pause");
let autoplay = true;

pauseBtn.addEventListener("click", () => {
  autoplay = !autoplay;
  pauseBtn.setAttribute("aria-pressed", String(!autoplay));
  pauseBtn.textContent = autoplay
    ? "自動再生を一時停止"
    : "自動再生を再開";
});
```

## スクロールコンテンツ（SCR33）

- 横に流れるテキストやティッカーは、一時停止できること
- 一時停止なしの無限スクロールは不適合になりやすい（F16）

```css
/* 悪い例：止められない無限スクロール */
.marquee-bad {
  animation: scroll-left 20s linear infinite;
}

/* 良い例：一時停止できる */
.marquee[data-paused="true"] {
  animation-play-state: paused;
}
```

> **参照**
> - [Technique SCR33: Using script to scroll content, and providing a mechanism to pause it](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR33)
> - [Failure F16: scrolling content without a mechanism to pause and restart](https://www.w3.org/WAI/WCAG22/Techniques/failures/F16)

## アニメーション GIF

- 5秒を超えて点滅・動き続ける GIF は制御が必要
- ループを少数回で止め、5秒以内に静止させる（G152）
- 制御できない場合は、静止画像や一時停止 UI に置き換える

```html
<!-- 良い例：数回で止まる GIF、または静止画像 + 説明 -->
<img src="loading-once.gif" alt="読み込み中の様子（数秒で停止）">
```

> **参照**
> - [Technique G152: Setting animated gif images to stop blinking after n cycles](https://www.w3.org/WAI/WCAG22/Techniques/general/G152)

## 並行して提示されない場合は対象外になりうる

- 動き・点滅・自動更新が**ページ上の他コンテンツと並行して提示されない**場合は、制御が不要なことがある
- 例
  - ローディング中だけ全画面で表示されるプリロードアニメーション
  - 本編の前に全員が見る全画面広告（他コンテンツと並行しない）
- プリロードアニメは、進捗がないと壊れていると誤解される場合、不可欠とみなせることもある

```html
<!-- 対象外になりうる：読み込み中のみの全画面表示 -->
<div class="preloader" role="status">
  動画を読み込んでいます。しばらくお待ちください。
</div>
```

> **参照**
> - [Understanding SC 2.2.2 — loading animation / full-page ad examples](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html#examples)

## 不可欠な例外

- 動き・自動更新が活動の本質で、止めるとその活動が成立しない場合は例外
- 例：リアルタイム対戦ゲームで、動きを止めると競技として成立しない
- ただし、ターン制にして一時停止できる設計なら、例外に頼らず適合しやすい

## 点滅と閃光の違い

| 用語 | 焦点 | 関連基準 |
|---|---|---|
| **点滅（blinking）** | 注意散漫 | 2.2.2（5秒以内、または止められる） |
| **閃光（flashing）** | 発作のリスク | 2.3.1 / 2.3.2 |

- 1秒間に3回を超える点滅は、閃光としても評価する
- 閃光は止める暇もなく発作を起こしうるため、そもそも出さない

> **参照**
> - [WCAG 2.2 — Guideline 2.3 Seizures and Physical Reactions](https://www.w3.org/TR/WCAG22/#seizures-and-physical-reactions)

## 1.4.2・2.2.1 との関係

| 達成基準 | 対象 | 関係 |
|---|---|---|
| 1.4.2 音声の制御 | 自動再生の音声 | 音声はこちら |
| 2.2.1 タイミング調整可能 | 時間制限全般 | 読む／操作する時間 |
| 2.2.2 一時停止・停止・非表示 | 動き・点滅・スクロール・自動更新 | 視覚的な並行コンテンツ |

- 2.2.2 は**視覚**の動き・更新が対象
- 自動再生の音声は 1.4.2
- セッションタイムアウトなどは 2.2.1

> **参照**
> - [wcag/1-4-2-audio-control.md — 1.4.2 音声の制御](./1-4-2-audio-control.md)
> - [wcag/2-2-1-timing-adjustable.md — 2.2.1 タイミング調整可能](./2-2-1-timing-adjustable.md)

## 複数の動く要素がある場合

- 規範上は要素ごとに制御できれば適合
- ベストプラクティスとして、**一括で一時停止・停止できる仕組み**を用意すると望ましい
- ユーザーが一つずつ止めなくてよい

```html
<!-- 望ましい：ページ内の動きをまとめて止める -->
<button type="button" id="pause-all-motion">
  動きのあるコンテンツをすべて一時停止
</button>
```

> **参照**
> - [Understanding SC 2.2.2 — best practice for multiple elements](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html#intent)

## 適合要件5：非干渉

- 2.2.2 を満たさないコンテンツは、ページ全体の利用を妨げる
- そのため、ページ上の**すべてのコンテンツ**が対象
- 他の達成基準の対象外コンテンツでも、2.2.2 は適用される

> **参照**
> - [WCAG 2.2 — Conformance Requirement 5: Non-Interference](https://www.w3.org/TR/WCAG22/#cc5)

## よくある失敗例

### F16：一時停止できないスクロール

- スクロールコンテンツに一時停止・再開の仕組みがない

### F112 / F50：5秒を超える点滅

- 5秒を超えて点滅し続け、止める手段がない

### F7：5秒を超えて動き続ける object / applet

```html
<!-- 悪い例：止められない点滅広告 -->
<div class="ad blink-forever">今すぐ購入！</div>
```

```css
/* 悪い例 */
.blink-forever {
  animation: blink 0.5s infinite;
}
```

> **参照**
> - [Failure F16: scrolling content without pause/restart](https://www.w3.org/WAI/WCAG22/Techniques/failures/F16)
> - [Failure F112: blinking content lasting more than five seconds without a mechanism to stop it](https://www.w3.org/WAI/WCAG22/Techniques/failures/F112)
> - [Failure F50: script that causes a blink effect without stopping within 5 seconds](https://www.w3.org/WAI/WCAG22/Techniques/failures/F50)

## テスト・確認方法

### 手動テスト

- 自動開始するカルーセル、ティッカー、GIF、点滅、自動更新を洗い出す
- 5秒を超えて続くか確認
- 他コンテンツと並行して表示されているか確認
- 一時停止・停止・非表示、または更新頻度制御があるか確認
- 「フォーカス中だけ止まる」仕組みになっていないか確認

```text
チェックリスト：
1. 自動開始する動き・点滅・スクロールがあるか
2. 5秒を超えて続くか
3. 他コンテンツと並行か
4. 一時停止／停止／非表示できるか
5. 自動更新の頻度を制御できるか
6. フォーカス依存の一時停止になっていないか
```

### 開発者ツールでの確認

```bash
# アニメーション・自動再生・点滅の実装を確認
rg "animation:|autoplay|setInterval|marquee|blink|carousel" --glob "*.css" --glob "*.scss" --glob "*.js" --glob "*.tsx" --glob "*.html"
```

### ACT Rules

- [Text content that changes automatically can be paused, stopped or hidden](https://www.w3.org/WAI/standards-guidelines/act/rules/0thtul/)

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.2.2 は、自動開始する**動き・点滅・スクロール・自動更新**を、ユーザーが一時停止・停止・非表示（または更新頻度制御）できるように求める達成基準（レベル A）
- 動き・点滅・スクロールは、5秒超・他コンテンツと並行なら制御が必要
- 自動更新には5秒例外がない
- フォーカス中だけ止まる仕組みは不十分
- 点滅は 2.2.2、閃光は 2.3.1 / 2.3.2 で別扱い
- ページ全体の適合を左右する非干渉要件がある
