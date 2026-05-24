# 1.2.4 キャプション（ライブ）（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.2.4「キャプション（ライブ）」（Captions (Live)）
- レベル **AA**（標準的な適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.2「時間依存メディア」に属する
- **ライブ**の**同期メディア**（映像+音声）の**すべての音声コンテンツ**にキャプションを提供
- 1.2.2（収録済・レベル A）の**ライブ版**。収録済みは 1.2.2、ライブは 1.2.4 が対象

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.4 Captions (Live)](https://www.w3.org/TR/WCAG22/#captions-live)
> - [Understanding SC 1.2.4 Captions (Live)](https://www.w3.org/WAI/WCAG22/Understanding/captions-live.html)

## 達成基準の原文（要約）

- **ライブ**の同期メディアに含まれる**すべての音声コンテンツ**にキャプションを提供すること
- 1.2.2 と異なり、**テキストのメディア代替**に関する例外条項はない

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.4](https://www.w3.org/TR/WCAG22/#captions-live)

## ライブとは

- 現実世界の出来事を捕捉し、**放送遅延（broadcast delay）** 以内で受信者に送信される情報
- 放送遅延：短い（通常は自動的な）遅延。放送者が音声・映像をキューイング・検閲するためのもの。**大幅な編集は含まない**
- **完全にコンピュータ生成**された情報はライブではない
- 具体例
  - ライブ配信（ウェビナー、記者会見、コンサート）
  - リアルタイムのニュース中継
  - ライブストリーミングイベント

### ライブ vs 収録済み

| 項目 | ライブ（1.2.4） | 収録済み（1.2.2） |
|---|---|---|
| 適合レベル | AA | A |
| キャプション作成 | リアルタイム（CART 等） | 事前作成・編集可能 |
| 精度 | 若干の誤差が許容される場合も | 高精度が期待される |
| 例外（メディア代替） | なし | あり |

> **参照**
> - [Understanding SC 1.2.4 — Key Terms: live](https://www.w3.org/WAI/WCAG22/Understanding/captions-live.html#dfn-live)

## なぜ必要か

- 聴覚障害者・難聴者が**リアルタイム**のプレゼンテーション・イベントにアクセスするため
- 1.2.2 が収録済みをカバーするのに対し、1.2.4 は**ライブ配信**をカバー
- ニュース・教育・エンターテインメントのライブ配信において、聴覚障害者の情報アクセスを確保

> **参照**
> - [Understanding SC 1.2.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/captions-live.html#benefits)

## 1.2.2 との関係

- キャプションの**内容要件**（台詞 + 話者 + 非発話音）は 1.2.2 と同じ
- 違いは**対象**（ライブ vs 収録済み）と**適合レベル**（AA vs A）
- ライブ配信を**録画して公開**する場合、録画版には **1.2.2** も適用される

| 達成基準 | 対象 | レベル |
|---|---|---|
| 1.2.2 キャプション（収録済） | 収録済み同期メディア | A |
| 1.2.4 キャプション（ライブ） | ライブ同期メディア | AA |

```html
<!-- ライブ配信中：1.2.4 が適用 -->
<div id="live-stream" aria-live="polite">
  <video id="player" autoplay muted></video>
  <div id="live-captions" class="captions-overlay" aria-live="assertive"></div>
</div>

<!-- 同じ配信の録画公開後：1.2.2 も適用 -->
<video controls>
  <source src="webinar-recording.mp4" type="video/mp4">
  <track kind="captions" src="webinar-captions.vtt" srclang="ja" label="日本語">
</video>
```

> **参照**
> - [wcag/1-2-2-captions-prerecorded.md — キャプションの基本](./1-2-2-captions-prerecorded.md)

## 適用範囲の注意：双方向通話

- **Web アプリ上の個人間双方向マルチメディア通話**（1対1や少人数のビデオチャット）は、**本基準の意図する対象外**
- 同期メディアの**放送**（一方向の配信）が主な対象
- 双方向通話でキャプションが必要な場合、責任は**コンテンツ提供者（通話参加者）** または **ホスト** にあり、アプリケーション自体に常時キャプション義務はない

```
対象内：ライブウェビナー、ニュース中継、コンサート配信、株主総会ライブ
対象外（意図）：Zoom/Teams の1対1通話、友人とのビデオチャット
```

> **参照**
> - [Understanding SC 1.2.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/captions-live.html#intent)

## ライブキャプションの実現方法

### CART（Communication Access Realtime Translation）

- 専門の**リアルタイム字幕起こし**サービス
- 速記者または音声認識 + 人手校正が、音声をリアルタイムでテキスト化
- ライブイベント・ウェビナー・裁判所などで広く利用
- 対面・リモート（電話・インターネット経由）のいずれも対応可能

### 自動ライブキャプション（AI 字幕）

- 音声認識エンジンによるリアルタイム字幕生成
- YouTube Live、Zoom、Microsoft Teams 等が提供
- **Technique G9**：ライブ同期メディア用キャプションの作成
- リアルタイムテキスト翻訳サービス（real-time text translation service）の利用も認められている
- 精度は CART より劣る場合がある。可能なら人手校正や CART を優先

### オープンキャプション（焼き込み）

- ライブ配信に字幕を**映像に直接焼き込み**
- クローズドキャプション（オン/オフ切替）が使えない環境向け
- 1.2.2 と同様、オープン・クローズドの区別なく提供されていれば適合

> **参照**
> - [Technique G9: Creating captions for live synchronized media](https://www.w3.org/WAI/WCAG22/Techniques/general/G9)
> - [Technique G87: Providing closed captions](https://www.w3.org/WAI/WCAG22/Techniques/general/G87)
> - [W3C WAI — Live Captions](https://www.w3.org/WAI/media/av/captions/)

## 実装例

### ライブ配信プラットフォームの利用

```html
<!-- 悪い例：ライブ配信にキャプション機能なし -->
<iframe
  src="https://live.example.com/embed/event-123"
  title="株主総会ライブ配信"
  allowfullscreen
></iframe>

<!-- 良い例：キャプション対応プラットフォーム + 有効化を案内 -->
<iframe
  src="https://live.example.com/embed/event-123?captions=1"
  title="株主総会ライブ配信（日本語キャプション対応）"
  allowfullscreen
></iframe>
<p><small>配信画面の CC ボタンでキャプションを表示できます。</small></p>
```

### WebVTT によるライブキャプション（HLS / DASH）

- ライブストリームに **WebVTT 字幕トラック**をリアルタイムで配信
- HLS（HTTP Live Streaming）の `#EXT-X-MEDIA` で字幕トラックを宣言
- プレイヤー（hls.js、Video.js 等）がライブ VTT を読み込んで表示

```html
<!-- ライブ HLS 配信 + 字幕トラック -->
<video id="live-player" controls></video>
<script>
  // hls.js 等でライブストリームと字幕トラックを読み込む（概念例）
  // 字幕はサーバー側でリアルタイム生成され、VTT セグメントとして配信
</script>
```

### JavaScript によるキャプション表示

```html
<!-- リアルタイム字幕 API からテキストを受信して表示 -->
<div id="live-event">
  <video id="stream" autoplay controls aria-describedby="live-captions"></video>
  <div
    id="live-captions"
    class="captions-overlay"
    aria-live="assertive"
    aria-atomic="false"
  ></div>
</div>
```

```javascript
// WebSocket 等で CART サービスから字幕テキストを受信（概念例）
const captionEl = document.getElementById('live-captions');
const ws = new WebSocket('wss://captions.example.com/live/event-123');

ws.onmessage = (event) => {
  const { text, speaker } = JSON.parse(event.data);
  captionEl.textContent = speaker ? `[${speaker}] ${text}` : text;
};
```

- `aria-live="assertive"` で字幕更新を支援技術に即時通知
- キャプション領域が映像の重要部分を**隠さない**よう配置

> **参照**
> - [MDN — aria-live](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-live)
> - [WebVTT — The Web Video Text Tracks Format](https://www.w3.org/TR/webvtt/)

## キャプションに含めるべき内容

- 1.2.2 と同じ要件。ライブでも省略不可
- **台詞**：発言内容をリアルタイムで記述
- **話者の特定**：複数話者の場合は `[司会]` `[質問者]` など
- **非発話音**：笑い、拍手、効果音など、理解に必要な音

```text
<!-- ライブキャプションの表示例（コンサート配信） -->
[管弦楽組曲第3番 ニ長調 BWV 1068「アリア」]
[ヨハン・セバスティアン・バッハ]
♪ 穏やかなメロディ、ゆったりとしたテンポ ♪

<!-- ライブキャプションの表示例（ウェビナー） -->
[講師] 本日はアクセシビリティについてお話しします。
（スライドが切り替わる）
[参加者] 質問があります。
[講師] はい、どうぞ。
```

> **参照**
> - [wcag/1-2-2-captions-prerecorded.md — キャプションの記述例](./1-2-2-captions-prerecorded.md)

## ライブ配信後の録画公開

- ライブ配信終了後、録画を公開する場合は **1.2.2（収録済キャプション）** も満たす必要がある
- ライブキャプションの録画データを**人手で校正**して WebVTT 化する運用が一般的
- 自動ライブキャプションの録画は誤変換が多いため、公開前の編集を推奨

```html
<!-- ライブ終了後：校正済み WebVTT を付与 -->
<video controls aria-label="ウェビナー録画（2024年3月1日）">
  <source src="webinar-2024-03-01.mp4" type="video/mp4">
  <track kind="captions" src="webinar-2024-03-01.vtt" srclang="ja" label="日本語">
</video>
<p><small>ライブ配信時のキャプションを校正・編集したものです。</small></p>
```

> **参照**
> - [W3C WAI — If you have live captions and you post a recording](https://www.w3.org/WAI/media/av/captions/)

## 代表的な失敗例

- ライブ配信にキャプション機能が**一切ない**
- 自動字幕を有効にしているが、**誤変換が多く内容が伝わらない**状態のまま提供
- 台詞のみで**非発話音・話者特定**を省略（1.2.2 の Failure F8 と同様の問題）
- ライブは対応しているが、**録画公開時に 1.2.2 を満たしていない**

```html
<!-- 悪い例：ライブ配信、キャプションなし -->
<video id="live" autoplay controls src="https://live.example.com/stream.m3u8"></video>

<!-- 悪い例：自動字幕のみ、明らかに不正確 -->
<!-- 「アクセシビリティ」→「アクセシビリティー」ではなく「悪路シビリティ」等 -->

<!-- 良い例：CART または校正済み自動字幕 + ユーザーへの案内 -->
<div id="live-event">
  <video autoplay controls aria-describedby="caption-notice"></video>
  <p id="caption-notice">本配信は日本語のライブキャプションに対応しています。</p>
  <div id="captions" aria-live="assertive"></div>
</div>
```

## 主要プラットフォームでの対応

| プラットフォーム | ライブキャプション機能 |
|---|---|
| YouTube Live | 自動字幕 + 手動字幕のアップロード |
| Zoom | 自動字幕（有料プラン）、CART 連携 |
| Microsoft Teams | ライブ字幕（自動） |
| Vimeo Live | ライブ字幕（自動 + 手動） |

- プラットフォーム機能を利用する場合、**配信設定でキャプションを有効化**する
- 埋め込み配信でも、視聴者がキャプションを表示できることを確認

> **参照**
> - [Technique G87: Readily available media format with closed captioning support](https://www.w3.org/WAI/WCAG22/Techniques/general/G87)

## テスト・確認方法

### 手動テスト

- ライブ配信中にキャプションを**オン**にし、音声と**同期**しているか確認
- 複数話者の場面で**話者特定**があるか
- 笑い・拍手など**非発話音**が適含されるか（可能な範囲で）
- 音量ゼロ・ミュート状態で内容が理解できるか
- キャプションが映像の重要部分を**隠していない**か
- 配信終了後の録画に **1.2.2 準拠のキャプション**があるか

### 運用チェックリスト

- [ ] ライブ配信前にキャプション方式（CART / 自動 / 焼き込み）を決定
- [ ] 配信プラットフォームでキャプション機能を有効化
- [ ] 視聴者向けにキャプションの有無・有効化方法を案内
- [ ] 録画公開時は 1.2.2 用にキャプションを校正・提供

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.2.4 は**ライブ同期メディア**の**すべての音声**にリアルタイムキャプションを要求（レベル AA）
- 1.2.2（収録済・A）のライブ版。キャプションの内容要件は同じ
- CART、自動ライブキャプション、焼き込み字幕などで実現
- 双方向ビデオ通話は意図した対象外。一方向のライブ配信・中継が主対象
- ライブ配信を録画公開する場合は **1.2.2** も別途満たす
- 詳細なキャプション記述ルールは [1-2-2-captions-prerecorded.md](./1-2-2-captions-prerecorded.md) を参照
