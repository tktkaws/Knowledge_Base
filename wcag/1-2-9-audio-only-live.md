# 1.2.9 音声のみ（ライブ）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.2.9「音声のみ（ライブ）」（Audio-only (Live)）
- レベル **AAA**（最高の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.2「時間依存メディア」に属する
- **ライブ**の**音声のみ**メディアに、**同等の情報**を持つ時間依存メディアの代替を提供
- 1.2.1（収録済・レベル A）の**ライブ版**。収録済み音声のみは 1.2.1、ライブ音声のみは 1.2.9 が対象

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.9 Audio-only (Live)](https://www.w3.org/TR/WCAG22/#audio-only-live)
> - [Understanding SC 1.2.9 Audio-only (Live)](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-live.html)

## 達成基準の原文（要約）

- **ライブ**の**音声のみ**コンテンツに対し、**同等の情報**を提示する**時間依存メディアの代替**を提供すること
- リアルタイムのライブキャプション（CART 等）が推奨。原稿に沿う場合は transcript も可

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.9](https://www.w3.org/TR/WCAG22/#audio-only-live)

## 対象となるメディア

### 音声のみ（audio-only）

- 音声だけを含む時間依存プレゼンテーション
- 映像もユーザー操作も含まない
- **ライブ**であること（放送遅延以内で配信）

### 具体例

- インターネットラジオのライブ配信
- 電話記者会見のライブストリーミング（音声のみ）
- ライブスピーチ・講演の音声中継
- 音声のみのライブポッドキャスト
- ウェブベースの音声会議の**一方向放送**（聴衆向け配信）

### 対象外

- **収録済み**の音声のみ → 1.2.1（A）
- **映像を含む**ライブ配信（同期メディア）→ 1.2.4 キャプション（ライブ）（AA）
- **完全にコンピュータ生成**された音声 → ライブではない
- **双方向の個人間通話**（1対1ビデオチャット等）→ 本基準の意図する対象外（後述）

> **参照**
> - [Understanding SC 1.2.9 — Key Terms: audio-only, live](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-live.html#dfn-audio-only)

## ライブとは

- 現実世界の出来事を捕捉し、**放送遅延（broadcast delay）** 以内で受信者に送信される情報
- 放送遅延：短い（通常は自動的な）遅延。キューイング・検閲用。**大幅な編集は含まない**
- **完全にコンピュータ生成**された情報はライブではない

| 項目 | ライブ（1.2.9） | 収録済み（1.2.1） |
|---|---|---|
| 適合レベル | AAA | A |
| 代替の作成 | リアルタイム（CART 等）または原稿 | 事前作成・編集可能 |
| 同期 | 音声と**同じペース**でテキストが進むことが望ましい | 事後閲覧でよい |

> **参照**
> - [Understanding SC 1.2.9 — Key Terms: live](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-live.html#dfn-live)
> - [wcag/1-2-4-captions-live.md — ライブの定義](./1-2-4-captions-live.md)

## なぜ必要か

- 聴覚障害者・難聴者が**リアルタイム**の音声のみコンテンツにアクセスするため
- 1.2.1 が収録済みをカバーするのに対し、1.2.9 は**ライブ音声のみ**をカバー
- WCAG 1.2 では、ライブ音声のみに対する A / AA の達成基準はなく、**AAA のみ**が要件
- ニュース速報・株主総会・ライブラジオなど、映像のないライブ音声配信のアクセシビリティを確保

> **参照**
> - [Understanding SC 1.2.9 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-live.html#benefits)

## 1.2.1 との関係

- **内容要件**（同等の情報を持つテキスト代替）は 1.2.1 と同じ
- 違いは**対象**（ライブ vs 収録済み）と**適合レベル**（AAA vs A）
- ライブ配信を**録音して公開**する場合、録音版には **1.2.1** も適用される

| 達成基準 | 対象 | レベル | 提供するもの |
|---|---|---|---|
| 1.2.1 音声のみ（収録済） | 収録済み音声のみ | A | 文字起こし（transcript） |
| 1.2.9 音声のみ（ライブ） | ライブ音声のみ | AAA | ライブキャプションまたは同等のテキスト代替 |

```html
<!-- ライブ配信中：1.2.9 が適用 -->
<audio id="live-radio" controls aria-label="ニュース速報ライブ"></audio>
<div id="live-transcript" aria-live="assertive" class="caption-viewport"></div>
<p><small>上記エリアにライブ文字起こしが表示されます。</small></p>

<!-- 同じ配信の録音公開後：1.2.1 も適用 -->
<audio src="news-2024-03-01.mp3" controls aria-label="ニュース速報（3月1日）"></audio>
<p><a href="#transcript">文字起こし</a></p>
<section id="transcript"><!-- 校正済み全文 --></section>
```

> **参照**
> - [wcag/1-2-1-audio-only-and-video-only.md — 収録済み音声のみ](./1-2-1-audio-only-and-video-only.md)

## 1.2.4 との関係

- **1.2.4**：ライブの**同期メディア**（映像+音声）→ キャプション（AA）
- **1.2.9**：ライブの**音声のみ** → テキスト代替（AAA）
- 映像のないライブ音声配信では 1.2.4 は適用されない
- ライブウェビナー（映像+音声）では 1.2.4 が主な要件。音声のみに切り替えた配信では 1.2.9

| 達成基準 | メディア種別 | レベル |
|---|---|---|
| 1.2.4 キャプション（ライブ） | 同期メディア（映像+音声） | AA |
| 1.2.9 音声のみ（ライブ） | 音声のみ | AAA |

> **参照**
> - [wcag/1-2-4-captions-live.md — ライブキャプション](./1-2-4-captions-live.md)

## 適用範囲の注意：双方向通話

- **Web アプリ上の個人間双方向マルチメディア通話**（1対1や少人数の音声・ビデオチャット）は、**本基準の意図する対象外**
- 音声の**放送**（一方向の配信・中継）が主な対象
- 双方向通話でキャプションが必要な場合、責任は**コンテンツ提供者（通話参加者）** または **ホスト** にあり、アプリケーション自体に常時キャプション義務はない

```
対象内：ライブラジオ、記者会見の音声ライブストリーム、株主総会の音声中継
対象外（意図）：Zoom/Teams の1対1通話、友人との音声チャット
```

> **参照**
> - [Understanding SC 1.2.9 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-live.html#intent)

## ライブ代替の実現方法

### 方法1：ライブキャプションサービス（G157）— 推奨

- 訓練された**速記者（stenographer）** または CART オペレーターが、音声をリアルタイムでテキスト化
- 音声と**同じペース**で進行。台本からの逸脱にも対応可能
- 非発話音（笑い、効果音等）の注記も挿入可能
- キャプション表示用の**ビューポート**を、ライブ音声と**同じ Web ページ**に配置
- 遅延は**30秒以内**が Technique G157 のテスト基準

```html
<!-- 良い例：ライブ音声 + キャプションビューポート（G157） -->
<div id="live-broadcast">
  <audio id="stream" controls aria-describedby="live-captions"></audio>
  <div
    id="live-captions"
    class="caption-viewport"
    aria-live="assertive"
    aria-atomic="false"
  ></div>
  <p id="caption-notice"><small>本配信はリアルタイム文字起こしに対応しています。</small></p>
</div>
```

```javascript
// CART サービスから WebSocket で字幕を受信（概念例）
const captionEl = document.getElementById('live-captions');
const ws = new WebSocket('wss://captions.example.com/live/radio-news');

ws.onmessage = (event) => {
  const { text, speaker } = JSON.parse(event.data);
  captionEl.textContent = speaker ? `[${speaker}] ${text}` : text;
};
```

### 方法2：テキストベースの代替（G150）

- ライブ音声のみ向けの**テキスト代替**を提供
- ライブキャプションと同様、リアルタイムで更新されるテキスト領域
- 自動音声認識のみで、**人手校正なし**かつ精度が低い場合は不適合の可能性

### 方法3：原稿・台本の transcript（G151）

- 話者が**決められた原稿・声明文**を読み上げる場合
- 原稿に沿うことが保証されるなら、**同時に transcript をリンク**提供して適合可能
- 予算・時間の制約で CART を間に合わせられない**緊急の記者会見**等で用いられる
- 原稿から**大きく逸脱**する場合は不適合

```html
<!-- 良い例：読み上げ原稿と同時に transcript を公開（G151） -->
<audio id="ceo-statement" controls aria-label="CEO声明（ライブ）"></audio>
<p>
  <a href="/press/2024-03-01-statement.txt" download>
    声明文全文（読み上げ原稿）
  </a>
</p>
<p><small>CEOは上記原稿に沿って読み上げます。</small></p>
```

```html
<!-- 悪い例：原稿と実際の発言が異なるのに transcript のみ -->
<audio controls aria-label="CEO声明（ライブ）"></audio>
<p><a href="/press/old-draft.txt">声明文</a></p>
<!-- → ライブで即興の追記・訂正があるのに古い原稿を掲載 -->
```

> **参照**
> - [Technique G157: Incorporating a live audio captioning service into a web page](https://www.w3.org/WAI/WCAG22/Techniques/general/G157)
> - [Technique G150: Providing text based alternatives for live audio-only content](https://www.w3.org/WAI/WCAG22/Techniques/general/G150)
> - [Technique G151: Providing a link to a text transcript of a prepared statement or script if the script is followed](https://www.w3.org/WAI/WCAG22/Techniques/general/G151)

## ライブキャプション vs 事後 transcript

| 項目 | ライブキャプション（推奨） | 原稿 transcript（G151） |
|---|---|---|
| タイミング | 音声と**同時**に表示 | 原稿を**同時公開** |
| 即興・逸脱 | 対応可能 | 原稿通りの場合のみ可 |
| 典型用途 | 討論・Q&A・ニュース中継 | 声明文の読み上げ |
| 1.2.9 適合 | 高い信頼性 | 条件付きで可 |

- Understanding SC 1.2.9：**訓練されていないオペレーター**、または**実際の内容と大きく異なる transcript** は適合とみなされない

> **参照**
> - [Understanding SC 1.2.9 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-live.html#intent)

## テキスト代替に含めるべき内容

- 1.2.1 の transcript と同じ要件。ライブでも省略不可
- **台詞**：発言内容をリアルタイムで記述
- **話者の特定**：複数話者の場合は `[司会]` `[記者]` など
- **非発話音**：笑い、拍手、効果音など、理解に必要な音

```text
<!-- ライブキャプションの表示例（記者会見） -->
[司会] 本日の記者会見を始めます。
[CEO] 新製品の発表についてお話しします。
（カメラのシャッター音）
[記者] 価格について教えてください。
[CEO] 希望小売価格は9,800円です。
```

```text
<!-- ライブキャプションの表示例（ライブラジオ） -->
[アナウンサー] 速報です。首都圏で地震が発生しました。
（緊急音楽）
[アナウンサー] 震度は現在調査中です。
```

> **参照**
> - [wcag/1-2-1-audio-only-and-video-only.md — 文字起こしの書き方](./1-2-1-audio-only-and-video-only.md)

## 実装例

### インターネットラジオ

```html
<!-- 悪い例：ライブ音声のみ、代替なし -->
<audio controls src="https://radio.example.com/live.mp3"></audio>

<!-- 良い例：ライブキャプションビューポート付き -->
<figure>
  <figcaption id="station-news">○○放送 ニュース速報（ライブ）</figcaption>
  <audio controls aria-labelledby="station-news"></audio>
  <div id="news-captions" aria-live="assertive" class="caption-viewport"></div>
</figure>
```

### 音声会議の一方向配信

```html
<!-- 良い例：聴衆向けライブ配信 + CART ビューポート -->
<div id="conference-broadcast">
  <h2>株主総会（音声ライブ）</h2>
  <audio controls aria-label="株主総会ライブ音声"></audio>
  <div
    id="cart-viewport"
    aria-live="assertive"
    role="region"
    aria-label="リアルタイム文字起こし"
  ></div>
</div>
```

### 自動音声認識（AI）字幕

- 自動ライブ字幕は **G150** の手段として認められる場合がある
- **誤変換が多く内容が伝わらない**状態は不適合
- 可能なら CART または人手校正を優先
- 1.2.4 の自動キャプションと同様の注意が適用される

```html
<!-- 悪い例：自動字幕のみ、明らかに不正確 -->
<audio controls aria-label="ライブ講演"></audio>
<div id="auto-captions" aria-live="polite"></div>
<!-- 「アクセシビリティ」→「悪路シビリティ」等、内容が伝わらない -->

<!-- 良い例：CART または校正済み自動字幕 + 案内 -->
<audio controls aria-label="ライブ講演"></audio>
<div id="captions" aria-live="assertive"></div>
<p><small>本配信は専門のリアルタイム文字起こしサービスを利用しています。</small></p>
```

> **参照**
> - [WebAIM — Real time captioning](https://webaim.org/techniques/captions/realtime)
> - [W3C WAI — Captions/Subtitles](https://www.w3.org/WAI/media/av/captions/)

## 代表的な失敗例

- ライブ音声配信に**テキスト代替が一切ない**
- **訓練されていない**担当者による不正確なリアルタイム入力
- **原稿と実際の発言が大きく異なる**のに、原稿 transcript のみを提供
- 自動音声認識の誤変換が多く、**内容が理解できない**まま提供
- ライブキャプションの**遅延が30秒を大幅に超える**（G157 のテスト基準）
- ライブは対応しているが、**録音公開時に 1.2.1 を満たしていない**

```html
<!-- 悪い例：「文字起こし準備中」のみ -->
<audio controls src="https://live.example.com/stream"></audio>
<p>文字起こしは後日公開予定です。</p>

<!-- 悪い例：プレースホルダーのみ（Failure F30 と同様の問題） -->
<audio controls></audio>
<p>文字起こし：live-stream.mp3</p>
```

> **参照**
> - [Failure F30: Text alternatives that are not alternatives](https://www.w3.org/WAI/WCAG22/Techniques/failures/F30)
> - [Failure F67: Long descriptions that do not serve the same purpose](https://www.w3.org/WAI/WCAG22/Techniques/failures/F67)

## ライブ配信後の録音公開

- ライブ配信終了後、録音を公開する場合は **1.2.1（収録済み音声のみ）** も満たす必要がある
- ライブキャプションの記録を**人手で校正**して transcript 化する運用が一般的
- AAA 適合（1.2.9）と A 適合（1.2.1）は**別々に**確認する

```html
<!-- ライブ終了後：校正済み transcript を付与（1.2.1） -->
<audio src="press-conference-2024-03-01.mp3" controls aria-label="記者会見録音"></audio>
<p><a href="#transcript">文字起こし（ライブ配信時のキャプションを校正）</a></p>
<section id="transcript"><!-- 全文 --></section>
```

> **参照**
> - [W3C WAI — If you have live captions and you post a recording](https://www.w3.org/WAI/media/av/captions/)

## HTML実装のポイント

### `<audio>` 要素

- `controls` で再生コントロールを提供
- `aria-label` または `aria-labelledby` で内容を識別（1.1.1）
- ライブキャプション領域は `aria-describedby` で関連付け
- `aria-live="assertive"` で字幕更新を支援技術に即時通知

```html
<audio
  id="live-stream"
  controls
  aria-label="○○ラジオ ライブニュース"
  aria-describedby="live-captions"
></audio>
<div id="live-captions" aria-live="assertive"></div>
```

### キャプションビューポートの配置

- ライブ音声プレイヤーの**近傍**に配置（G58 の精神）
- 別ウィンドウのみだと**ユーザビリティの障壁**になる（Understanding の Example 参照）
- 可能なら**同一 Web ページ内**に音声とキャプションをまとめる

> **参照**
> - [MDN — audio element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio)
> - [MDN — aria-live](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-live)
> - [Technique G58: Placing a link to the alternative immediately next to the non-text content](https://www.w3.org/WAI/WCAG22/Techniques/general/G58)

## テスト・確認方法

### 手動テスト

- ライブ配信中にテキスト代替が**表示・更新**されているか
- 音声とテキストの**遅延が30秒以内**か（G157）
- 音量ゼロ・ミュート状態で**同等の情報**が得られるか
- 複数話者の場面で**話者特定**があるか
- 原稿 transcript を使う場合、**実際の発言と一致**しているか
- 配信終了後の録音に **1.2.1 準拠の transcript** があるか

### 運用チェックリスト

- [ ] ライブ配信前に代替方式（CART / 自動 / 原稿）を決定
- [ ] キャプションビューポートを同一ページに配置
- [ ] 視聴者向けにライブ文字起こしの有無を案内
- [ ] 原稿使用時は「原稿に沿って読み上げ」を明示
- [ ] 録音公開時は 1.2.1 用に transcript を校正・提供

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.2.9 は**ライブ音声のみ**メディアに**同等の情報**を持つテキスト代替を要求（レベル AAA）
- 1.2.1（収録済・A）のライブ版。WCAG 1.2 ではライブ音声のみの A / AA 要件はない
- **ライブキャプション（CART）** が推奨。原稿に沿う場合は同時 transcript（G151）も可
- 訓練されていないオペレーターや、実際と異なる transcript は不適合
- 双方向ビデオ・音声通話は意図した対象外。一方向のライブ配信・中継が主対象
- ライブ配信を録音公開する場合は **1.2.1** も別途満たす
- 映像付きライブ配信は **1.2.4** が対象。音声のみと混同しない
