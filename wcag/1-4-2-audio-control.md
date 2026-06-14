# 1.4.2 音声の制御（A）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.2「音声の制御」（Audio Control）
- レベル A（最低限の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- **3秒を超えて自動再生**される音声がある場合、**一時停止・停止**、または**システム音量と独立した音量制御**の仕組みを提供
- **非干渉要件**（Conformance Requirement 5）：本基準を満たさないコンテンツは、ページ全体の適合を妨げる

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.2 Audio Control](https://www.w3.org/TR/WCAG22/#audio-control)
> - [Understanding SC 1.4.2 Audio Control](https://www.w3.org/WAI/WCAG22/Understanding/audio-control.html)

## 達成基準の原文（要約）

- Web ページ上の音声が**3秒を超えて自動再生**される場合、次のいずれかを満たすこと
  - 音声を**一時停止または停止**できる仕組み
  - **システム全体の音量とは独立**した音量制御の仕組み
- 音量制御には**ゼロ（ミュート）**まで下げることが含まれる

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.2](https://www.w3.org/TR/WCAG22/#audio-control)

## 対象となる音声

### 対象

- **自動再生**（autoplay）で **3秒を超える**音声
- 具体例
  - ページ読み込み時の BGM
  - `autoplay` 付き `<video>` の音声トラック
  - `<audio autoplay>` によるナレーション
  - 自動開始する Web アニメーションの効果音（3秒超）

### 対象外・適合しやすいケース

- **3秒以内**で自動停止する音声（Technique G60）
- **ユーザー操作**で開始した音声（Technique G171）
- **ミュート**状態の自動再生（音声なし）
- **双方向**の音声通信（ビデオ会議等）。参加者が話す可能性は「自動再生」とは異なる

```html
<!-- 対象外：3秒以内の効果音（G60） -->
<audio src="notification.mp3" autoplay></audio>
<!-- ファイル自体が2秒 → 適合 -->

<!-- 対象外：ユーザーが再生ボタンを押した音声（G171） -->
<audio src="podcast.mp3" controls></audio>

<!-- 対象外：ミュート自動再生 -->
<video src="hero.mp4" autoplay muted loop></video>
```

> **参照**
> - [Understanding SC 1.4.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/audio-control.html#intent)

## なぜ必要か

- **スクリーンリーダー**の読み上げと自動再生音声が**同時に鳴る**と、内容が聞き取れない
- スクリーンリーダーが**システム音量**を共有する場合、BGM を下げると読み上げも小さくなる
- **難聴**のあるユーザー、**注意の切り替え**が困難なユーザーにとって、予期しない音声は障壁
- 自動再生音声があると、停止ボタンを**見つける前**にナビゲーションが妨げられる

> **参照**
> - [Understanding SC 1.4.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/audio-control.html#benefits)

## 適合の2つの方法

| 方法 | 内容 | 注意 |
|---|---|---|
| 一時停止・停止 | ページ上のコントロールで音声を止められる | システム音量のミュートは**不可** |
| 独立した音量制御 | ページ内でその音声だけ音量を下げられる | **ゼロまで**下げられること |

- OS の音量ミュートは「一時停止・停止」**ではない**
- `<video controls>` の音量スライダーは、多くの場合**そのメディア要素**の音量を制御 → 適合の手段になりうる

> **参照**
> - [Understanding SC 1.4.2 — Intent (volume control)](https://www.w3.org/WAI/WCAG22/Understanding/audio-control.html#intent)

## 実装方法（Sufficient Techniques）

### 方法1：ユーザー操作で再生開始（G171）— 推奨

- 音声は**ユーザーが明示的に開始**する
- 「再生」ボタン・リンクのクリック後に音声を開始
- 自動再生を避けるのが**最も安全**な実装

```html
<!-- 悪い例：ページ読み込みで自動再生、停止手段なし（F23） -->
<audio src="bgm.mp3" autoplay loop></audio>

<!-- 良い例：ユーザーが再生を選択（G171） -->
<audio src="bgm.mp3" controls aria-label="BGM"></audio>
<p><small>BGMを再生する場合は上のコントロールを使用してください。</small></p>

<!-- 良い例：明示的な再生ボタン -->
<button type="button" id="play-intro" aria-label="イントロ音声を再生">
  音声ガイドを再生
</button>
<audio id="intro" src="intro.mp3" aria-label="イントロ音声"></audio>
<script>
  document.getElementById('play-intro').addEventListener('click', () => {
    document.getElementById('intro').play();
  });
</script>
```

### 方法2：ページ冒頭に停止コントロール（G170）

- 自動再生する場合、**ページの先頭付近**に停止・ミュート用コントロールを配置
- スクリーンリーダー利用者が**すぐに見つけられる**位置が重要

```html
<!-- 良い例：自動再生 + ページ上部に停止リンク（G170） -->
<a href="#main" class="skip-link">メインコンテンツへ</a>
<button type="button" id="stop-audio" aria-label="BGMを停止">
  音声を停止
</button>

<audio id="bgm" src="ambient.mp3" autoplay loop aria-label="背景音楽"></audio>

<main id="main">...</main>

<script>
  document.getElementById('stop-audio').addEventListener('click', () => {
    const audio = document.getElementById('bgm');
    audio.pause();
    audio.currentTime = 0;
  });
</script>
```

### 方法3：3秒以内で自動停止（G60）

- 通知音・効果音など、**3秒以内**で終わる音声は自動再生可
- ファイル長が3秒以下であることを確認

```html
<!-- 良い例：短い通知音（G60） -->
<audio src="short-beep.mp3" autoplay aria-label="通知音"></audio>
<!-- short-beep.mp3 が3秒以内 -->
```

### HTML5 メディア要素

- `autoplay` で音声付き `<video>` / `<audio>` を使う場合、**controls** で停止・音量調整を提供
- controls なし + autoplay + 3秒超 → **Failure F93**

```html
<!-- 悪い例：autoplay + 音声 + controls なし（F93） -->
<video src="promo.mp4" autoplay></video>

<!-- 良い例：autoplay でも controls で停止・音量調整可能 -->
<video src="promo.mp4" autoplay controls aria-label="プロモーション動画"></video>

<!-- 良い例：ヒーロー動画はミュート自動再生 -->
<video src="hero.mp4" autoplay muted loop playsinline aria-label="背景動画（音声なし）"></video>
```

> **参照**
> - [Technique G171: Playing sounds only on user request](https://www.w3.org/WAI/WCAG22/Techniques/general/G171)
> - [Technique G170: Control near the beginning that turns off auto-play sounds](https://www.w3.org/WAI/WCAG22/Techniques/general/G170)
> - [Technique G60: Sound that turns off automatically within three seconds](https://www.w3.org/WAI/WCAG22/Techniques/general/G60)
> - [Failure F93: No way to pause or stop autoplaying HTML5 media](https://www.w3.org/WAI/WCAG22/Techniques/failures/F93)

## 代表的な失敗例（Failures）

### F23：3秒超の自動再生に停止手段がない

```html
<!-- F23 -->
<audio src="background-music.mp3" autoplay loop></audio>
<!-- 停止ボタン・controls なし -->

<!-- F23：システム音量に依存する案内だけ -->
<audio src="long-intro.mp3" autoplay></audio>
<p>音量はOSの設定で調整してください。</p>
<!-- → ページ内の独立した制御ではない -->
```

### F93：HTML5 メディアの autoplay に pause/stop がない

```html
<!-- F93 -->
<video src="tutorial.mp4" autoplay></video>
<audio src="narration.mp3" autoplay></audio>
```

> **参照**
> - [Failure F23: Playing sound longer than 3 seconds with no off mechanism](https://www.w3.org/WAI/WCAG22/Techniques/failures/F23)
> - [Failure F93: Absence of way to pause or stop autoplaying HTML5 media](https://www.w3.org/WAI/WCAG22/Techniques/failures/F93)

## 非干渉要件（Conformance Requirement 5）

- 1.4.2 を満たさないコンテンツが**1つでも**あると、ページ全体が**レベル A 未達**になりうる
- ページの**どこか**に3秒超の自動再生音声がある場合、**全コンテンツ**が本基準の影響を受ける
- 埋め込み iframe 内の自動再生音声も**ページ全体**の適合に影響

> **参照**
> - [WCAG 2.2 — Conformance Requirement 5: Non-Interference](https://www.w3.org/TR/WCAG22/#cc5)

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 2.2.2 一時停止・停止・非表示（A） | **5秒超**の自動更新コンテンツ全般（動画・点滅等） |
| 1.4.7 小さな背景音または背景音なし（AAA） | 前景音声に対する背景音のレベル |
| 1.2 時間依存メディア | キャプション等（音声の**内容**へのアクセス） |

### 2.2.2 との違い

| 達成基準 | 対象 | 閾値 |
|---|---|---|
| 1.4.2 音声の制御 | **音声**の自動再生 | **3秒**超 |
| 2.2.2 一時停止・停止・非表示 | 動画・点滅・自動更新等 | **5秒**超 |

- 自動再生動画（音声あり）は**両方**を満たす必要がある場合がある

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.2 Pause, Stop, Hide](https://www.w3.org/TR/WCAG22/#pause-stop-hide)
> - [WCAG 2.2 — Success Criterion 1.4.7 Low or No Background Audio](https://www.w3.org/TR/WCAG22/#low-or-no-background-audio)

## ベストプラクティス

- **自動再生を避ける**。ユーザー操作での開始（G171）が最善
- 自動再生が必要なら**ミュート**をデフォルトにし、ユーザーが音声をオンにする
- 停止コントロールは**ページ先頭**に配置（G170）
- `prefers-reduced-motion` と併せ、動画 + 音声の自動再生を控える

```html
<!-- ベストプラクティス：ミュート自動再生 + ユーザーが音声をオン -->
<video id="hero" src="hero.mp4" autoplay muted loop playsinline></video>
<button type="button" id="unmute" aria-label="動画の音声をオンにする">
  音声をオン
</button>
<script>
  document.getElementById('unmute').addEventListener('click', () => {
    document.getElementById('hero').muted = false;
  });
</script>
```

> **参照**
> - [a11y/prefers-reduced-motion.md — 自動再生アニメーション](../a11y/prefers-reduced-motion.md)

## テスト・確認方法

### 手動テスト

- ページ読み込み後、**3秒を超える**自動再生音声がないか確認
- 自動再生がある場合、**3秒以内**に停止・ミュートできるか
- 停止手段が**ページ先頭付近**にあるか（G170）
- システム音量を変えずに、**その音声だけ**止められるか
- `<video autoplay>` が **muted** か、**controls** があるか
- 埋め込み（iframe、サードパーティウィジェット）の音声も確認

### ACT Rule

- [Audio or video element avoids automatically playing audio](https://www.w3.org/WAI/standards-guidelines/act/rules/80f0bf/proposed/)

```bash
# autoplay 属性を持つ audio/video を検出（開発者ツール Console）
[...document.querySelectorAll('audio, video')].filter(el => el.autoplay).map(el => ({
  tag: el.tagName,
  muted: el.muted,
  controls: el.controls,
  src: el.currentSrc || el.src
}));
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.2 は**3秒超の自動再生音声**に停止または独立音量制御を要求（レベル A）
- **ユーザー操作での再生開始**（G171）が最も安全。自動再生は非推奨
- 自動再生する場合：**controls**、ページ上部の停止ボタン（G170）、3秒以内（G60）
- システム音量の調整案内だけでは**不適合**
- **非干渉要件**：1箇所の違反がページ全体の適合を妨げうる
- 2.2.2（5秒・動画等）・1.4.7（背景音レベル）と併せて確認
