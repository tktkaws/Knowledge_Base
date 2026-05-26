# 1.2.6 手話（収録済）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.2.6「手話（収録済）」（Sign Language (Prerecorded)）
- レベル **AAA**（最高の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.2「時間依存メディア」に属する
- **収録済み**の**同期メディア**（映像+音声）の**すべての音声コンテンツ**に**手話通訳**を提供
- 多くの法令・ガイドラインでは AA までが目標。AAA は部分的な適用が現実的

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.6 Sign Language (Prerecorded)](https://www.w3.org/TR/WCAG22/#sign-language-prerecorded)
> - [Understanding SC 1.2.6 Sign Language (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/sign-language-prerecorded.html)

## 達成基準の原文（要約）

- 収録済みの同期メディアに含まれる**すべての音声コンテンツ**に**手話通訳**（sign language interpretation）を提供すること

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.6](https://www.w3.org/TR/WCAG22/#sign-language-prerecorded)

## 手話通訳とは

- 口語（spoken language）を**手話**（sign language）に翻訳した映像
- 手話通訳者（sign language interpreter）が、音声トラックの内容を手話で表現
- 手話は独立した言語。口語の単語をそのまま手の動きに置き換えるものではない
- 日本では**日本手話**（Japanese Sign Language / JSL）が一般的

### 手話とキャプションの違い

| 項目 | キャプション（1.2.2） | 手話通訳（1.2.6） |
|---|---|---|
| 形式 | テキスト（同期字幕） | 手話通訳者の映像 |
| 言語 | 口語の文字化 | 手話（独立言語） |
| 適合レベル | A | AAA |
| 感情・イントネーション | 限定的 | 表情・動作で表現可能 |
| 対象ユーザー | 難聴者全般 | 手話を母語とする聴覚障害者 |

- 手話を母語とするユーザーにとって、キャプション（書面語）は**第二言語**になることが多い
- 読解力が限られている場合、キャプションだけでは内容理解が不十分
- 手話通訳はイントネーション・感情・文脈など、音声情報をより**等価に**伝えられる

> **参照**
> - [Understanding SC 1.2.6 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/sign-language-prerecorded.html#intent)
> - [W3C WAI — Sign Languages](https://www.w3.org/WAI/media/av/sign-languages/)

## なぜ必要か

- 手話を**母語**とする聴覚障害者が、同期メディアの音声情報にアクセスするため
- キャプションは文字言語。手話ネイティブにとっては理解速度・精度で劣る場合がある
- 手話は時間ベースの表現であり、同期メディアとの相性が良い
- キャプションと手話通訳を**同時に**利用したいユーザーもいる

> **参照**
> - [Understanding SC 1.2.6 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/sign-language-prerecorded.html#benefits)

## 対象となるメディア

### 同期メディア（synchronized media）

- 映像と音声が同期した**収録済み**プレゼンテーション
- 具体例
  - 企業の重要発表動画
  - 大学講義の録画
  - 行政の説明動画
  - eラーニング教材

### 対象外

- **ライブ**の同期メディア（ライブ手話通訳は別途検討。WCAG 1.2.6 は収録済みのみ）
- **音声のみ**・**映像のみ** → 1.2.1
- 手話を母語としないユーザー向けの代替としては、1.2.2 キャプション（A）が優先

> **参照**
> - [Understanding SC 1.2.6 — Key Terms: synchronized media, prerecorded](https://www.w3.org/WAI/WCAG22/Understanding/sign-language-prerecorded.html)

## 他の達成基準との関係

| 達成基準 | 内容 | レベル |
|---|---|---|
| 1.2.2 キャプション（収録済） | 音声の同期字幕 | A |
| 1.2.4 キャプション（ライブ） | ライブ音声の同期字幕 | AA |
| 1.2.5 音声解説（収録済） | 映像の音声解説 | AA |
| 1.2.6 手話（収録済） | 音声の手話通訳 | AAA |

- 1.2.6 は 1.2.2 の**代替ではなく追加**の提供手段
- AAA 適合を目指す場合、キャプション・音声解説に加えて手話通訳も必要
- サイト全体の AAA 適合は W3C も推奨していない。重要コンテンツに部分的に適用するのが現実的

> **参照**
> - [wcag/1-2-2-captions-prerecorded.md — キャプション（1.2.2）](./1-2-2-captions-prerecorded.md)
> - [WCAG 2.2 — Conformance](https://www.w3.org/TR/WCAG22/#conformance)

## 実装方法（Technique G81）

- **Technique G81**：手話通訳者の同期映像を、別ビューポートまたはプレイヤーによるオーバーレイで表示
- **SM13 / SM14**：SMIL で手話通訳映像ストリームを同期

### 方法1：ピクチャーインピクチャ（PiP）オーバーレイ

- メイン動画の一角（通常は右下）に手話通訳者を**重ねて表示**
- 最も一般的な実装。ニュース番組・行政動画で広く採用

```html
<!-- 悪い例：キャプションのみ、手話通訳なし -->
<video controls aria-label="重要発表">
  <source src="announcement.mp4" type="video/mp4">
  <track kind="captions" src="announcement.vtt" srclang="ja" label="日本語">
</video>

<!-- 良い例：手話通訳版への切り替えを提供 -->
<div class="video-player" role="group" aria-label="重要発表（手話通訳対応）">
  <video id="main-video" controls aria-label="重要発表">
    <source src="announcement.mp4" type="video/mp4">
    <track kind="captions" src="announcement.vtt" srclang="ja" label="日本語">
  </video>
  <button type="button" aria-pressed="false" aria-controls="sign-video" id="toggle-sign">
    手話通訳を表示
  </button>
  <video
    id="sign-video"
    class="sign-overlay"
    muted
    aria-label="日本手話通訳"
    hidden
  >
    <source src="announcement-jsl.mp4" type="video/mp4">
  </video>
</div>
```

```css
/* 手話通訳のオーバーレイ配置 */
.sign-overlay {
  position: absolute;
  bottom: 1rem;
  right: 1rem;
  width: 25%;
  min-width: 160px;
  border: 2px solid #fff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.5);
}

/* メイン動画の下部情報（ニュースティッカー等）を隠さない */
.video-player {
  position: relative;
}
```

### 方法2：手話通訳版を別動画として提供

- 手話通訳を組み込んだ**別バージョン**の動画を用意
- メイン動画の直下に、目立つトグルボタンまたはリンクで切り替え

```html
<!-- 良い例：通常版と手話通訳版を明示的に提供 -->
<figure>
  <video controls aria-labelledby="video-title">
    <source src="lecture.mp4" type="video/mp4">
    <track kind="captions" src="lecture.vtt" srclang="ja" label="日本語">
  </video>
  <figcaption id="video-title">第3回講義：Webアクセシビリティ</figcaption>
</figure>

<div class="sign-language-option">
  <a href="lecture-jsl.mp4">
    <img src="jsl-icon.svg" alt="">
    日本手話通訳版を視聴
  </a>
  <p><small>手話通訳付きバージョンでは、右下に通訳者が表示されます。</small></p>
</div>
```

### 方法3：2映像ストリームの同期再生

- メイン動画と手話通訳映像を**時間同期**して同時再生
- JavaScript で `currentTime` を同期

```html
<div class="dual-video" role="group" aria-label="講義動画（手話通訳同期）">
  <video id="main" controls aria-label="講義本編">
    <source src="lecture.mp4" type="video/mp4">
    <track kind="captions" src="lecture.vtt" srclang="ja" label="日本語">
  </video>
  <video id="sign" muted aria-label="日本手話通訳" class="sign-overlay">
    <source src="lecture-jsl.mp4" type="video/mp4">
  </video>
</div>
```

```javascript
// メイン動画と手話通訳映像を同期（概念例）
const main = document.getElementById('main');
const sign = document.getElementById('sign');

main.addEventListener('play', () => sign.play());
main.addEventListener('pause', () => sign.pause());
main.addEventListener('seeked', () => { sign.currentTime = main.currentTime; });
main.addEventListener('timeupdate', () => {
  if (Math.abs(sign.currentTime - main.currentTime) > 0.3) {
    sign.currentTime = main.currentTime;
  }
});
```

> **参照**
> - [Technique G81: Synchronized video of sign language interpreter](https://www.w3.org/WAI/WCAG22/Techniques/general/G81)
> - [Technique SM13: Sign language in SMIL 1.0](https://www.w3.org/WAI/WCAG22/Techniques/smil/SM13)
> - [Technique SM14: Sign language in SMIL 2.0](https://www.w3.org/WAI/WCAG22/Techniques/smil/SM14)

## 手話通訳映像の制作ガイドライン

### 撮影・編集

- **背景と服装**：肌の色とコントラストのある無地の背景・服装。手と顔が見やすいこと
- **照明**：通訳者がはっきり見える十分な照明
- **Signing space（手話空間）**：腰下から頭上、左右に肘幅以上をフレームに収める
- **通訳者のサイズ**：手の動き・表情が明確に見える大きさ
- **重要情報を隠さない**：通訳者は通常右下。ニュースティッカー等と重ならない位置
- **制作計画**：撮影前に通訳者の配置を storyboard に含める

### 通訳者の選定

- **資格を持つ手話通訳者**に依頼
- 対象の手話（日本手話、ASL、BSL 等）をコンテンツの言語に合わせて選定
- 手話は地域・国ごとに異なる独立言語（ASL ≠ BSL ≠ 日本手話）
- 自動手話アバターは現時点では**十分な品質に達していない**（W3C WAI）

```html
<!-- 悪い例：手話を知らない人による不適切なジェスチャー -->
<!-- 自動生成アバターのみ、通訳者の確認なし -->

<!-- 良い例：資格を持つ通訳者による収録 + クレジット表示 -->
<video controls aria-label="説明動画（日本手話通訳付き）">
  <source src="explain-jsl.mp4" type="video/mp4">
  <track kind="captions" src="explain.vtt" srclang="ja" label="日本語">
</video>
<p><small>手話通訳：（通訳者名）、日本手話</small></p>
```

> **参照**
> - [W3C WAI — Creating Sign Language Alternatives](https://www.w3.org/WAI/media/av/sign-languages/)
> - [Sign Language Interpretation in HBBTV (PDF)](https://www.ebu.ch/files/live/sites/ebu/files/Shared%20%20pdf/techreview_tva_signing.pdf)

## 日本手話と日本語対応手話

- **日本手話（JSL）**：聴覚障害者コミュニティの自然言語。独立した言語体系
- **日本語対応手話（Signed Japanese）**：日本語の語順・語彙に沿った手話。JSL とは異なる
- WCAG の「手話通訳」は、対象ユーザーの**母語となる手話**（日本では JSL）を指す
- コンテンツの主要言語が日本語の場合、**日本手話**による通訳が適切

> **参照**
> - [Understanding SC 1.2.6 — Key Terms: sign language interpretation](https://www.w3.org/WAI/WCAG22/Understanding/sign-language-prerecorded.html#dfn-sign-language-interpretation)

## 代表的な失敗例

- キャプションのみ提供し、手話通訳が**一切ない**（AAA 未達）
- 手話通訳映像があるが、メイン動画と**同期していない**
- 通訳者が小さすぎて**手の動き・表情が見えない**
- 通訳者がメイン映像の**重要情報を隠している**
- 不適切な手話（自動アバター、資格のない通訳）を使用
- 手話通訳版への**切り替え方法が分かりにくい**

```html
<!-- 悪い例：手話通訳版がページ末尾にしかリンクがない -->
<video controls src="important.mp4"></video>
<!-- ... 長い本文 ... -->
<p><a href="important-jsl.mp4">手話版</a></p>

<!-- 良い例：プレイヤー直下に目立つ切り替え -->
<video controls src="important.mp4" aria-label="重要なお知らせ">
  <track kind="captions" src="important.vtt" srclang="ja" label="日本語">
</video>
<button type="button" aria-pressed="false">
  日本手話通訳を表示
</button>
```

## 実務上の位置づけ

- 1.2.6 はレベル **AAA**。日本の JIS X 8341-3 や多くの法令は **AA** が目標
- 公共機関・教育機関の**重要コンテンツ**（政策説明、災害情報、入学説明等）に部分的適用
- すべての動画に手話通訳を付けるのはコスト・時間が大きい。優先度の高いコンテンツから
- キャプション（1.2.2）を先に整備し、手話通訳は段階的に追加する運用が現実的

| 優先度 | コンテンツ例 | 推奨対応 |
|---|---|---|
| 高 | 行政説明、災害情報、入学・採用説明 | 手話通訳 + キャプション |
| 中 | 講義録画、研修動画 | キャプション（A/AA）+ 手話は要望に応じ |
| 低 | 短いプロモーション、SNS 動画 | キャプション優先 |

> **参照**
> - [WCAG 2.2 — Conformance at AAA](https://www.w3.org/TR/WCAG22/#cc1)

## テスト・確認方法

### 手動テスト

- 手話通訳版が**存在し**、メイン動画から**容易にアクセス**できるか
- 手話通訳映像がメイン動画の音声と**同期**しているか
- 通訳者の**手・顔・上半身**が十分なサイズで見えるか
- 通訳者がメイン映像の**重要部分を隠していない**か
- **キャプション**も引き続き提供されているか（手話のみに頼らない）
- 手話通訳の**言語**（日本手話等）が明示されているか

### ユーザーテスト

- 手話を母語とするユーザーにレビューを依頼
- 通訳の正確性・自然さ・速度を確認

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.2.6 は**収録済み同期メディア**の**すべての音声**に**手話通訳**を要求（レベル AAA）
- キャプションは第二言語になりうる。手話ネイティブには手話通訳がより等価なアクセス
- 実装は PiP オーバーレイ、別動画、2ストリーム同期が一般的（Technique G81）
- 資格を持つ通訳者による収録。自動アバターは現時点では不十分
- 日本では**日本手話**が対象。1.2.2 キャプションと併用が望ましい
- サイト全体 AAA は非現実的。重要コンテンツに部分的適用する
