# 1.2.2 キャプション（収録済）（A）

## 達成基準の概要

- WCAG 2.2 達成基準 1.2.2「キャプション（収録済）」（Captions (Prerecorded)）
- レベル A（最低限の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.2「時間依存メディア」に属する
- **収録済み**の**同期メディア**（映像+音声）の**すべての音声コンテンツ**にキャプションを提供
- 聴覚障害者・難聴者が動画の音声情報を理解するための要件

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.2 Captions (Prerecorded)](https://www.w3.org/TR/WCAG22/#captions-prerecorded)
> - [Understanding SC 1.2.2 Captions (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/captions-prerecorded.html)

## 達成基準の原文（要約）

- 収録済みの同期メディアに含まれる**すべての音声コンテンツ**にキャプションを提供すること
- **例外**：メディアがテキストのメディア代替であり、それが明確にラベル付けされている場合

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.2](https://www.w3.org/TR/WCAG22/#captions-prerecorded)

## キャプションとは

- 音声トラックの内容を**映像と同期**して表示するテキスト
- 台詞（speech）と、理解に必要な**非発話音**（non-speech audio）の両方を含む
- メディアプレイヤー内に表示され、音声のタイミングと同期
- 字幕（subtitles）と混同されやすいが、WCAG 文脈では**同一言語**の音声をテキスト化したものをキャプションと呼ぶ

### キャプションに含めるべき内容

- **台詞**：発言内容を正確に（逐語的に）記述
- **話者の特定**：誰が話しているかを明示
- **非発話音**：笑い、拍手、効果音、BGM、音楽タイトルなど、内容理解に必要な音
- **話者の位置・状況**：必要に応じて `[off-screen]` などで明示

### キャプションに含めなくてよい場合

- 内容理解に不要な環境音（微かな風の音など）
- 音声解説（audio description）自体のキャプション化（映像で既に提示されている情報のため）

> **参照**
> - [Understanding SC 1.2.2 — Key Terms: captions](https://www.w3.org/WAI/WCAG22/Understanding/captions-prerecorded.html#dfn-captions)
> - [W3C WAI — Captions/Subtitles](https://www.w3.org/WAI/media/av/captions/)

## キャプション・字幕・transcript の違い

| 用語 | 内容 | 同期 | 主な用途 |
|---|---|---|---|
| キャプション（captions） | 同一言語の台詞 + 非発話音 | 映像と同期 | 聴覚障害者向け（アクセシビリティ） |
| 字幕（subtitles） | 翻訳された台詞（非発話音なしの場合も） | 映像と同期 | 多言語対応（アクセシビリティ要件ではない） |
| transcript | 台詞 + 非発話音（+ 映像描写） | 同期不要 | テキスト閲覧・検索・1.2.3 の代替 |

- キャプションと transcript は**同じテキスト**から作成できることが多い
- 1.2.2 はキャプション（同期テキスト）を要求。transcript だけでは 1.2.2 を満たさない

> **参照**
> - [W3C WAI — Captions/Subtitles: Captions and Subtitles](https://www.w3.org/WAI/media/av/captions/)

## 対象となるメディア

### 同期メディア（synchronized media）

- 映像と音声が同期した時間依存プレゼンテーション
- 具体例
  - 解説動画・チュートリアル
  - ニュース映像・インタビュー
  - 講義・セミナーの録画
  - 製品デモ動画

### 対象外

- **音声のみ**・**映像のみ** → 1.2.1
- **ライブ**の同期メディア → 1.2.4（レベル AA）
- **テキストのメディア代替**として明確にラベル付けされたメディア（例外）

> **参照**
> - [Understanding SC 1.2.2 — Key Terms: synchronized media](https://www.w3.org/WAI/WCAG22/Understanding/captions-prerecorded.html#dfn-synchronized-media)

## なぜ必要か

- 聴覚障害者・難聴者が動画の音声情報にアクセスするため
- 騒がしい環境で動画を視聴するユーザーへの配慮
- 音声よりテキスト処理が得意なユーザーの理解支援
- 1.2.3（映像情報の代替）とは**独立した要件**。キャプションがあっても 1.2.3 は別途必要

> **参照**
> - [Understanding SC 1.2.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/captions-prerecorded.html#benefits)

## クローズドキャプションとオープンキャプション

| 種類 | 特徴 | 実装 |
|---|---|---|
| クローズドキャプション（CC） | ユーザーが表示/非表示を切り替え可能 | `<track kind="captions">` が一般的 |
| オープンキャプション | 常に表示。オフにできない | 動画に焼き込み（burn-in） |

- WCAG 2.2 ではクローズド・オープンの区別なく、**キャプションが提供されていれば適合**
- Web では `<track>` によるクローズドキャプションが標準的

> **参照**
> - [Understanding SC 1.2.2 — Key Terms: captions Note 2, 3](https://www.w3.org/WAI/WCAG22/Understanding/captions-prerecorded.html#dfn-captions)

## 例外：テキストのメディア代替

- 動画が**ページ上のテキスト以上の情報を持たない**場合
- かつ、メディア代替であることが**明確にラベル付け**されている場合
- キャプションは不要（情報はページのテキストで既に提供されている）

```html
<!-- 良い例：各段落のテキストと動画が1対1対応、動画に追加情報なし -->
<article>
  <section>
    <p id="para-1">第1条：本契約は...</p>
    <video
      src="clause-1.mp4"
      controls
      aria-label="第1条の読み上げ（本文と同等）"
    ></video>
  </section>
  <!-- 動画は段落テキストの読み上げのみ → キャプション不要 -->
</article>
```

```html
<!-- 悪い例：本文より動画に追加情報があるのにキャプションなし（Failure F75） -->
<article>
  <p>操作方法の概要</p>
  <video src="detailed-tutorial.mp4" controls></video>
  <!-- 動画内のナレーションが本文より詳しい → キャプション必須 -->
</article>
```

> **参照**
> - [Failure F75: Synchronized media without captions when media presents more information than on the page](https://www.w3.org/WAI/WCAG22/Techniques/failures/F75)
> - [Failure F74: Not labeling a synchronized media alternative to text as an alternative](https://www.w3.org/WAI/WCAG22/Techniques/failures/F74)

## HTML実装（Technique H95）

### `<track kind="captions">` + WebVTT

- **Technique G87**：クローズドキャプションの提供
- **Technique H95**：`<track>` 要素でキャプションを提供
- ファイル形式は **WebVTT**（`.vtt`）が Web 標準

```html
<!-- 悪い例：キャプションなし -->
<video controls src="tutorial.mp4" aria-label="結び方チュートリアル"></video>

<!-- 悪い例：transcript のみ（同期していない） -->
<video controls src="tutorial.mp4"></video>
<p>ロープで結び目を作ることは、船員や兵士にとって重要な技能でした。</p>

<!-- 良い例：同期キャプションを提供 -->
<video controls aria-label="結び方チュートリアル">
  <source src="tutorial.mp4" type="video/mp4">
  <track
    kind="captions"
    src="tutorial-captions.vtt"
    srclang="ja"
    label="日本語"
    default
  >
</video>
```

```text
<!-- tutorial-captions.vtt の内容例 -->
WEBVTT

00:00:01.000 --> 00:00:04.000
（音楽）

00:00:04.000 --> 00:00:08.000
ロープで結び目を作ることは、
船員や兵士、木こりにとって重要な技能でした。

00:00:10.000 --> 00:00:13.000
（効果音：ロープがきしむ音）

00:00:13.000 --> 00:00:16.000
まず、ロープの端を二重に折り返します。
```

### 複数言語のキャプション

```html
<video controls>
  <source src="lecture.mp4" type="video/mp4">
  <track kind="captions" src="lecture-ja.vtt" srclang="ja" label="日本語" default>
  <track kind="captions" src="lecture-en.vtt" srclang="en" label="English">
</video>
```

- `kind="subtitles"` は翻訳字幕向け。同一言語のアクセシビリティ用途は `kind="captions"`
- `default` 属性でデフォルト表示するトラックを指定

> **参照**
> - [Technique G87: Providing closed captions](https://www.w3.org/WAI/WCAG22/Techniques/general/G87)
> - [Technique H95: Using the track element to provide captions](https://www.w3.org/WAI/WCAG22/Techniques/html/H95)
> - [WebVTT — The Web Video Text Tracks Format](https://www.w3.org/TR/webvtt/)

## キャプションの記述例

### チュートリアル動画

```text
00:00:01.000 --> 00:00:04.000
（音楽）

00:00:04.000 --> 00:00:08.000
ロープで結び目を作ることは、
船員や兵士、木こりにとって重要な技能でした。
```

### オーケストラ演奏

```text
00:00:00.000 --> 00:00:05.000
[管弦楽組曲第3番 ニ長調 BWV 1068「アリア」]
[ヨハン・セバスティアン・バッハ]

00:00:05.000 --> 00:00:15.000
♪ 穏やかなメロディ、ゆったりとしたテンポ ♪
```

### 複数話者のインタビュー

```text
00:00:05.000 --> 00:00:08.000
[司会] 本日のゲストを紹介します。

00:00:08.000 --> 00:00:12.000
[ゲスト] よろしくお願いします。

00:00:12.000 --> 00:00:14.000
（拍手）
```

- 言語・文化によってキャプションのスタイルガイドは異なる
- 非発話音は `(笑い)`、`[効果音]`、`♪` など、慣例に従って記述

> **参照**
> - [Captioning Key: Guidelines and Preferred Techniques](https://dcmp.org/learn/captioningkey)
> - [BBC: Subtitle Guidelines](https://www.bbc.co.uk/accessibility/forproducts/guides/subtitles/)

## 自動キャプションの注意点

- 自動生成キャプションは**そのままでは適合しない**（精度が不十分な場合が多い）
- 音声認識の誤変換で意味が変わるリスク（「not」の欠落など）
- 自動キャプションを**起点**として、人手で校正・編集する運用が推奨
- YouTube 等の自動字幕機能も、公開前に内容を確認・修正する

```html
<!-- 悪い例：自動生成キャプションを未編集のまま公開 -->
<!-- YouTube 自動字幕：「今日は非テキストコンテンツについて」→「今日は非テキストこんなについて」 -->

<!-- 良い例：自動生成 → 人手で校正した WebVTT を提供 -->
<video controls>
  <source src="lecture.mp4" type="video/mp4">
  <track kind="captions" src="lecture-reviewed.vtt" srclang="ja" label="日本語">
</video>
```

> **参照**
> - [W3C WAI — Automatic Captions are Not Sufficient](https://www.w3.org/WAI/media/av/captions/)

## 代表的な失敗例（Failures）

- **F8**：台詞や重要な効果音を省略したキャプション
- **F75**：ページのテキストより動画に多くの情報があるのにキャプションなし
- **F74**：テキストのメディア代替であることをラベル付けしていない

```html
<!-- F8：台詞の省略 -->
<!-- 音声：「本日は3つのポイントを説明します。第1に...第2に...第3に...」 -->
<!-- キャプション：「本日は3つのポイントを説明します。」のみ → NG -->

<!-- F8：効果音の省略 -->
<!-- 音声：（ドアが閉まる音）「誰かいるの？」 -->
<!-- キャプション：「誰かいるの？」のみ → 効果音が文脈理解に必要なら NG -->

<!-- F75：本文より動画が詳しいのにキャプションなし -->
<article>
  <p>製品の使い方</p>
  <video src="full-tutorial.mp4" controls></video>
  <!-- 動画内の詳細な手順説明にキャプションがない → NG -->
</article>
```

> **参照**
> - [Failure F8: Captions omitting dialogue or important sound effects](https://www.w3.org/WAI/WCAG22/Techniques/failures/F8)
> - [Failure F75: Synchronized media without captions](https://www.w3.org/WAI/WCAG22/Techniques/failures/F75)
> - [Failure F74: Not labeling media alternative to text](https://www.w3.org/WAI/WCAG22/Techniques/failures/F74)

## 1.2.3 との関係

- 1.2.2 と 1.2.3 は**独立した要件**。音声付き動画は**両方**を満たす必要がある

| 達成基準 | 対象 | 提供するもの | 主な受益者 |
|---|---|---|---|
| 1.2.2 キャプション（A） | 音声トラック | 同期字幕（台詞 + 非発話音） | 聴覚障害者 |
| 1.2.3 音声解説またはメディアの代替（A） | 映像トラック | 音声解説 **または** テキスト代替 | 視覚障害者 |

```html
<!-- 1.2.2 + 1.2.3 を両方満たす例 -->
<video controls aria-label="製品デモ動画">
  <source src="demo.mp4" type="video/mp4">
  <track kind="captions" src="demo-captions.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="demo-ad.vtt" srclang="ja" label="音声解説">
</video>
```

> **参照**
> - [wcag/1-2-3-audio-description-or-media-alternative.md — 1.2.3 との関係](./1-2-3-audio-description-or-media-alternative.md)

## 外部プラットフォームでの提供

- YouTube、Vimeo 等の埋め込み動画でも、プラットフォーム側でキャプションを提供すれば適合可能
- 埋め込み時にキャプションが有効になるか、プラットフォームの設定を確認
- 自前ホスティングの場合は WebVTT ファイルを `<track>` で関連付け

```html
<!-- YouTube 埋め込み：プラットフォームの字幕機能に依存 -->
<iframe
  src="https://www.youtube.com/embed/VIDEO_ID?cc_load_policy=1"
  title="チュートリアル動画"
  allowfullscreen
></iframe>
<!-- cc_load_policy=1 で字幕のデフォルト表示を促す -->
<!-- 字幕の正確性は YouTube 側の設定・編集に依存 -->
```

> **参照**
> - [Technique G87: Readily available media format with closed captioning support](https://www.w3.org/WAI/WCAG22/Techniques/general/G87)

## テスト・確認方法

### 自動テスト

- `<video>` に `captions` トラックの有無
- ACT Rule：[Video element auditory content has accessible alternative](https://www.w3.org/WAI/standards-guidelines/act/rules/eac66b/proposed/)

### 手動テスト

- キャプションをオンにし、**音声と同期**しているか確認
- 台詞の**省略・誤変換**がないか全文チェック
- 笑い・効果音・BGMなど**非発話音**が適切に含まれるか
- 話者の特定が必要な場面で明示されているか
- キャプションが映像の重要部分を**隠していない**か
- 音量ゼロ・ミュート状態で動画の内容が理解できるか

```bash
# video 要素の captions トラックを確認（開発者ツール Console）
document.querySelectorAll('video').forEach(v => {
  const captions = [...v.textTracks].filter(t => t.kind === 'captions');
  console.log(v.src || v.currentSrc, captions.map(t => t.label));
});
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.2.2 は**収録済み同期メディア**の**すべての音声**に同期キャプションを要求
- キャプションには台詞 + 話者特定 + 理解に必要な非発話音を含める
- `<track kind="captions">` + WebVTT が Web での標準実装
- transcript だけでは不十分。**映像と同期**したテキストが必要
- 自動キャプションは校正必須。そのままでは適合しない場合が多い
- 1.2.3（映像情報の代替）とは別要件。音声付き動画は両方必要
- テキストのメディア代替として明確にラベル付けされた場合は例外
