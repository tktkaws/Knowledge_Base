# 1.2.7 拡張音声解説（収録済）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.2.7「拡張音声解説（収録済）」（Extended Audio Description (Prerecorded)）
- レベル **AAA**（最高の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.2「時間依存メディア」に属する
- **収録済み**の**同期メディア**において、台詞の休止が不十分な場合に**拡張音声解説**を提供
- 1.2.5（AA）の標準音声解説では足りない場面向けの AAA 要件

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.7 Extended Audio Description (Prerecorded)](https://www.w3.org/TR/WCAG22/#extended-audio-description-prerecorded)
> - [Understanding SC 1.2.7 Extended Audio Description (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/extended-audio-description-prerecorded.html)

## 達成基準の原文（要約）

- 前景音声（foreground audio）の**休止が不十分**で、標準の音声解説では映像の意味を伝えられない場合
- 収録済みの同期メディアの**すべての映像コンテンツ**に**拡張音声解説**を提供すること

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.7](https://www.w3.org/TR/WCAG22/#extended-audio-description-prerecorded)

## 拡張音声解説（extended audio description）とは

- 動画を**一時停止**し、追加の音声解説を再生してから**再開**する手法
- 標準音声解説は台詞の**既存の休止部分**にのみ挿入可能
- 休止が短い・ない場合、映像の意味が伝わらない → **拡張音声解説**が必要
- 視聴体験は一時中断されるが、視覚情報を等価に伝えるために不可欠な場合がある

### 標準音声解説との違い

| 項目 | 標準音声解説（1.2.5） | 拡張音声解説（1.2.7） |
|---|---|---|
| 挿入方法 | 台詞の**休止部分**に挿入 | 動画を**一時停止**して挿入 |
| 適合レベル | AA | AAA |
| 視聴への影響 | 連続再生 | 定期的に停止・再開 |
| 必要な条件 | 常に（映像情報がある場合） | 休止が不十分な場合のみ |

```text
<!-- 標準音声解説：休止部分に挿入 -->
Bonnie Chen：「これらの写真はすべてエバーグレーズで撮影されました。」
（休止）
解説者：教師が各生徒に2本の薄い木の棒を渡す。
Bonnie Chen：「今日は、こういうくちばしを持つ水鳥の...」

<!-- 拡張音声解説：休止がないため動画を停止 -->
Bonnie Chen：「問題1の図を見てください。次に問題2に移ります。」
（休止なし — 図の説明が入らない）
→ 動画停止
解説者：ホワイトボードに描かれた自由体図。力のベクトルが3本、
        角度30度で作用している。下向きの重力も示されている。
→ 動画再開
Bonnie Chen：「問題2では...」
```

> **参照**
> - [Understanding SC 1.2.7 — Key Terms: extended audio description](https://www.w3.org/WAI/WCAG22/Understanding/extended-audio-description-prerecorded.html#dfn-extended-audio-description)

## なぜ必要か

- 台詞・ナレーションが**途切れなく続く**動画では、標準音声解説を挿入する余地がない
- 講義・デモ・実験映像など、**映像と音声が同時進行**するコンテンツで問題になりやすい
- 視覚障害者・ロービジョン・視覚情報の理解が難しいユーザーが、動画の意味を把握するため
- 1.2.5 の標準音声解説だけでは**情報欠落**が生じる場面を補完

> **参照**
> - [Understanding SC 1.2.7 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/extended-audio-description-prerecorded.html#benefits)

## 拡張音声解説が必要な典型例

- **講義・授業**：教授が早口で話しながらホワイトボードに描き、消して次の問題へ
- **操作デモ**：ナレーションと同時に画面操作・UI の変化が続く
- **実験映像**：実験者が説明しながら器具を操作
- **速いカット・モンタージュ**：場面転換が早く、休止がほとんどない
- **インタビュー**：話者が途切れなく話し、映像に重要なジェスチャー・資料がある

### 拡張音声解説が不要な場合

- 映像の重要情報が**すでに音声トラックで伝わっている**（トーキングヘッド等）
- 台詞の**休止が十分**あり、標準音声解説（1.2.5）で対応可能
- 1.2.5 / 1.2.3 の Note と同様の例外が適用される

> **参照**
> - [Understanding SC 1.2.7 — Example: Video of a lecture](https://www.w3.org/WAI/WCAG22/Understanding/extended-audio-description-prerecorded.html#examples)

## 他の達成基準との関係

| 達成基準 | 内容 | レベル |
|---|---|---|
| 1.2.3 音声解説またはメディアの代替 | 音声解説 **または** テキスト代替 | A |
| 1.2.5 音声解説（収録済） | 標準音声解説 | AA |
| 1.2.7 拡張音声解説（収録済） | 休止不足時の拡張音声解説 | AAA |
| 1.2.8 メディアの代替（収録済） | 拡張テキスト記述 | AAA |

- **1.2.5 で足りる場合** → 1.2.7 は追加不要（休止が十分なら標準 AD で適合）
- **1.2.5 では足りない場合** → AAA 適合には 1.2.7 の拡張音声解説が必要
- **1.2.8** はテキストベースの拡張代替。1.2.7 は**音声**ベース

```html
<!-- 1.2.5 の標準 AD で足りる例：休止がある -->
<video controls>
  <source src="interview.mp4" type="video/mp4">
  <track kind="captions" src="interview.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="interview-ad.vtt" srclang="ja" label="音声解説">
</video>

<!-- 1.2.7 が必要な例：休止不足 → 拡張 AD 版を別提供 -->
<video controls aria-label="物理学講義">
  <source src="lecture.mp4" type="video/mp4">
  <track kind="captions" src="lecture.vtt" srclang="ja" label="日本語">
</video>
<p><a href="lecture-extended-ad.mp4">拡張音声解説付きバージョンを視聴</a></p>
```

> **参照**
> - [wcag/1-2-5-audio-description-prerecorded.md — 1.2.5 音声解説](./1-2-5-audio-description-prerecorded.md)
> - [WCAG 2.2 — Success Criterion 1.2.8 Media Alternative (Prerecorded)](https://www.w3.org/TR/WCAG22/#media-alternative-prerecorded)

## 実装方法（Technique G8）

- **Technique G8**：拡張音声解説付き動画を提供
- **SM1 / SM2**：SMIL で拡張音声解説を同期
- 拡張 AD は視聴を中断するため、**オン/オフ切り替え**または**別バージョン**の提供が推奨

### 方法1：拡張 AD 版を別ファイルで提供

- 制作段階で動画を停止・解説挿入・再開を編集した**別バージョン**を用意
- 最も確実。ユーザーは必要な場合のみ選択

```html
<!-- 悪い例：休止不足の動画に標準 AD のみ -->
<video controls src="physics-lecture.mp4" aria-label="物理学講義">
  <track kind="captions" src="lecture.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="lecture-ad.vtt" srclang="ja" label="音声解説">
</video>
<!-- 休止がなく、図の説明が音声解説に入らない -->

<!-- 良い例：通常版 + 拡張 AD 版 -->
<video controls aria-label="物理学講義">
  <source src="physics-lecture.mp4" type="video/mp4">
  <track kind="captions" src="lecture.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="lecture-ad.vtt" srclang="ja" label="音声解説">
</video>
<p>
  <a href="physics-lecture-extended-ad.mp4">
    拡張音声解説付きバージョン（図解の詳細説明あり）
  </a>
</p>
```

### 方法2：プレイヤーでオン/オフ切り替え

- 拡張 AD モードを**ユーザーが選択**できる UI
- 不要なユーザーは通常再生、必要なユーザーは拡張 AD を有効化

```html
<div class="video-player" role="group" aria-label="操作チュートリアル">
  <video id="tutorial" controls aria-label="ソフトウェア操作デモ">
    <source src="tutorial.mp4" type="video/mp4">
    <track kind="captions" src="tutorial.vtt" srclang="ja" label="日本語">
  </video>
  <label>
    <input type="checkbox" id="extended-ad-mode">
    拡張音声解説を有効にする（説明のため動画が一時停止します）
  </label>
</div>
```

```javascript
// 拡張 AD のタイムライン定義（概念例）
const extendedADSegments = [
  { pauseAt: 45.0, resumeAt: 45.0, description: 'ダッシュボード画面。左サイドバーに分析メニュー。' },
  { pauseAt: 78.0, resumeAt: 78.0, description: 'グラフが表示され、2024年7月のピークが強調されている。' },
];

const video = document.getElementById('tutorial');
const extendedMode = document.getElementById('extended-ad-mode');
let currentSegment = 0;
let isPlayingDescription = false;

video.addEventListener('timeupdate', () => {
  if (!extendedMode.checked || isPlayingDescription) return;
  const seg = extendedADSegments[currentSegment];
  if (seg && video.currentTime >= seg.pauseAt) {
    isPlayingDescription = true;
    video.pause();
    speakDescription(seg.description, () => {
      isPlayingDescription = false;
      currentSegment++;
      video.play();
    });
  }
});

function speakDescription(text, callback) {
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.lang = 'ja-JP';
  utterance.onend = callback;
  speechSynthesis.speak(utterance);
}
```

- 本番では事前録音したナレーション + 正確なタイムコード管理が望ましい
- `SpeechSynthesis` は品質・ブラウザ差があるため、**収録済み音声**が推奨

### 方法3：SMIL による同期（SM1 / SM2）

- SMIL プレゼンテーションで動画停止・音声解説挿入・再開を宣言
- レガシー環境向け。現代 Web では別ファイル方式が主流

> **参照**
> - [Technique G8: Providing a movie with extended audio descriptions](https://www.w3.org/WAI/WCAG22/Techniques/general/G8)
> - [Technique SM1: Adding extended audio description in SMIL 1.0](https://www.w3.org/WAI/WCAG22/Techniques/smil/SM1)
> - [Technique SM2: Adding extended audio description in SMIL 2.0](https://www.w3.org/WAI/WCAG22/Techniques/smil/SM2)

## 拡張音声解説の制作フロー

1. **動画を分析**：休止が不十分で説明が入れられない箇所を特定
2. **停止ポイントを決定**：映像の意味が伝わらない区間ごとに停止位置を設定
3. **解説ナレーションを収録**：停止区間の視覚情報を詳述
4. **編集**：停止 → ナレーション → 再開のタイムラインを組み立て
5. **別バージョンまたは切り替え UI** として提供

```text
<!-- 拡張 AD タイムライン例：物理学講義 -->
00:00:00 - 00:02:30  通常再生（教授の導入）
00:02:30             【停止】
00:02:30 - 00:02:50  解説：「ホワイトボードの自由体図。物体に3つの力が作用。
                     上向き20N、右向き15N、下向き重力25N。角度は水平から30度。」
00:02:50             【再開】
00:02:50 - 00:05:00  通常再生（教授が次の問題へ）
00:05:00             【停止】
00:05:00 - 00:05:25  解説：「前の図を消し、新しい図を描き始める。
                     斜面上の物体。摩擦係数μ=0.3と記載。」
00:05:25             【再開】
```

> **参照**
> - [Standard Techniques in Audio Description](http://joeclark.org/access/description/ad-principles.html)
> - [W3C WAI — Description of Visual Information](https://www.w3.org/WAI/media/av/description/)

## UX 上の配慮

- 拡張 AD は**視聴を中断**するため、不要なユーザーへの影響を最小化
- **デフォルトは通常版**。拡張 AD はオプトイン（ユーザーが選択）
- 拡張 AD 版であることを**明示**（「説明のため動画が停止します」等）
- キーボード操作で拡張 AD のオン/オフが可能に
- 1.2.2 キャプションは拡張 AD 版にも提供

```html
<!-- 良い例：拡張 AD の説明 + オプトイン -->
<video controls aria-label="講義動画">
  <source src="lecture.mp4" type="video/mp4">
  <track kind="captions" src="lecture.vtt" srclang="ja" label="日本語">
</video>
<p>
  <a href="lecture-extended-ad.mp4">拡張音声解説版</a>
  — 図やジェスチャーの詳細説明のため、動画が一時停止する箇所があります。
</p>
```

## 代表的な失敗例

- 休止不足の動画に**標準 AD のみ**提供し、重要な視覚情報が欠落
- 拡張 AD 版があるが、**切り替え方法が不明**またはページ末尾にしかない
- 拡張 AD が**デフォルトオン**で、すべての視聴者の体験を損なう
- 停止区間の解説が**不十分**（「図を見てください」程度）
- 1.2.5 を満たしていると**1.2.7 も自動的に満たす**と誤解（休止不足なら別途必要）

```html
<!-- 悪い例：休止なしの講義に短い標準 AD のみ -->
<!-- 音声：「この図を見て次に進みます」（図の内容は音声にない） -->
<track kind="descriptions" src="lecture-ad.vtt" srclang="ja" label="音声解説">
<!-- lecture-ad.vtt に図の説明を入れる休止がない → 1.2.7 未達 -->

<!-- 良い例：拡張 AD 版で図を詳述 -->
<a href="lecture-extended-ad.mp4">拡張音声解説版（図解説あり）</a>
```

## 実務上の位置づけ

- 1.2.7 はレベル **AAA**。AA 適合が目標の多くのプロジェクトでは必須ではない
- **休止不足の動画**（講義、デモ、実験）に限定して部分的適用
- 制作コストが高い（停止ポイント分析 + 追加ナレーション + 編集）
- 代替として **1.2.8 の拡張テキスト記述**（完全 transcript）も AAA で検討可能
- 優先度：公共教育コンテンツ、必修 e ラーニング、行政の説明動画

| 動画タイプ | 1.2.5 で足りるか | 1.2.7 の必要性 |
|---|---|---|
| トーキングヘッド | 足りる（追加 AD 不要） | 低 |
| インタビュー（休止あり） | 足りる | 低 |
| 操作デモ（連続ナレーション） | 不足しがち | 高 |
| 講義（板書+早口） | 不足しがち | 高 |

> **参照**
> - [WCAG 2.2 — Conformance at AAA](https://www.w3.org/TR/WCAG22/#cc1)

## テスト・確認方法

### 手動テスト

- **音声のみ**で動画を視聴し、休止不足区間で情報が欠落しないか
- 拡張 AD 版で**停止 → 解説 → 再開**が正しく動作するか
- 停止区間の解説が**映像の視覚情報と一致**しているか
- 拡張 AD 版への**アクセス**が容易か（プレイヤー直下等）
- 通常版と拡張 AD 版の**両方**が提供されているか
- 1.2.2 **キャプション**も拡張 AD 版に含まれるか

```bash
# 動画の descriptions トラックと拡張 AD リンクを確認
document.querySelectorAll('video').forEach(v => {
  const links = v.closest('figure, div')?.querySelectorAll('a[href*="extended"], a[href*="ad"]');
  console.log(v.currentSrc, links?.length ?? 0);
});
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.2.7 は**前景音声の休止が不十分**な場合に**拡張音声解説**を要求（レベル AAA）
- 動画を**一時停止**して追加ナレーションを挿入し、再開する手法
- 1.2.5 の標準 AD で足りる場合は 1.2.7 は不要
- 講義・デモ・実験映像など休止不足のコンテンツで問題になりやすい
- 別バージョン提供またはオン/オフ切り替えで UX を配慮
- 1.2.8（拡張テキスト記述）とは別の AAA 要件。音声 vs テキストの選択
