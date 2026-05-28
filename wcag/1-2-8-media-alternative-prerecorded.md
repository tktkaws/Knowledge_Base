# 1.2.8 メディアの代替（収録済）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.2.8「メディアの代替（収録済）」（Media Alternative (Prerecorded)）
- レベル **AAA**（最高の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.2「時間依存メディア」に属する
- **収録済み**の**同期メディア**および**収録済み映像のみ**メディアに、**時間依存メディアの代替**（テキスト）を提供
- 1.2.3（A）のテキスト代替と同形式だが、AAA では**すべての対象メディア**に対して**必須**（条件付き追加要件あり）

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.8 Media Alternative (Prerecorded)](https://www.w3.org/TR/WCAG22/#media-alternative-prerecorded)
> - [Understanding SC 1.2.8 Media Alternative (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/media-alternative-prerecorded.html)

## 達成基準の原文（要約）

- 収録済みの**同期メディア**および収録済みの**映像のみ**メディアに対し、**時間依存メディアの代替**を提供すること
- 同期メディアの**視覚・聴覚の両方**を、動画と同じ順序でテキスト化したドキュメント
- インタラクション（「今すぐ回答を押してください」等）があれば、同等の機能（リンク等）も提供

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.8](https://www.w3.org/TR/WCAG22/#media-alternative-prerecorded)

## 時間依存メディアの代替とは

- 同期メディアの**視覚・聴覚のすべて**を時系列で記述したテキストドキュメント
- 脚本や小説のように読める形式。音声解説より**完全な情報**を提供可能
- 音声解説との決定的な違い：**台詞の休止に制約がない**。映像描写を必要な長さで記述できる

| 項目 | 音声解説（1.2.5） | 時間依存メディアの代替（1.2.8） |
|---|---|---|
| 形式 | 音声トラックにナレーション追加 | テキスト（脚本・transcript 形式） |
| 記述範囲 | 台詞の**休止部分**に挿入 | 休止に**制約なし**。完全記述 |
| 情報量 | 重要な視覚情報に限定 | 視覚・聴覚の**すべて**を時系列で記述 |
| 主な受益者 | 視覚障害者 | 視覚・聴覚の**両方**に障害のあるユーザー（点字ディスプレイ利用者等） |
| 適合レベル | AA | AAA |

```text
<!-- 時間依存メディアの代替の記述例 -->
（タイトル画面：「新技術トレーニング — 分析ツールの使い方」）

ナレーター：「本日は新しい分析ツールの使い方を学びます。」

（画面：ダッシュボードが表示される。左サイドバーに「分析」「設定」「レポート」のメニュー。
 中央に折れ線グラフ。右上にユーザー名「田中太郎」）

ナレーター：「まず、左メニューから分析を選択します。」

（画面：カーソルが「分析」をクリック。グラフ画面に遷移。期間選択の日付ピッカーが表示される）

（効果音：クリック音）

ナレーター：「次に、期間を2024年1月から3月に設定します。」

（画面：日付ピッカーで2024/01/01〜2024/03/31が選択される。グラフが更新される）
```

> **参照**
> - [Understanding SC 1.2.8 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/media-alternative-prerecorded.html#intent)
> - [W3C WAI — Transcripts](https://www.w3.org/WAI/media/av/transcripts/)

## なぜ必要か

- **視覚・聴覚の両方**に障害のあるユーザー（ろうあ者等）は、キャプションも音声解説も利用できない
- 点字ディスプレイで読める**テキスト代替**が唯一のアクセス手段になる
- 音声解説は休止部分に制約があり、**完全な映像描写**を提供できない場合がある
- 自分のペースで読み返せるテキストは、認知障害のあるユーザーにも有益

> **参照**
> - [Understanding SC 1.2.8 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/media-alternative-prerecorded.html#benefits)

## 対象となるメディア

### 同期メディア（synchronized media）

- 映像と音声が同期した時間依存プレゼンテーション
- 具体例：トレーニング動画、講義録画、製品デモ、ニュース映像

### 映像のみ（video-only）

- 音声トラックのない収録済み映像
- 1.2.1（A）でも代替が必要だが、1.2.8（AAA）でも**引き続き要件**

### 対象外・例外

- **ライブ**の同期メディア → 1.2.4 等
- **テキストのメディア代替**として明確にラベル付けされたメディア（1.2.3 と同様の例外）
- 音声トラックですでにすべての重要な映像情報が伝わっている場合 → 追加の記述は最小限で足りる

> **参照**
> - [Understanding SC 1.2.8 — Key Terms: synchronized media](https://www.w3.org/WAI/WCAG22/Understanding/media-alternative-prerecorded.html#dfn-synchronized-media)

## 1.2.3 / 1.2.5 / 1.2.8 の関係

- 3つの達成基準は内容が重複するが、適合レベルごとに選択肢と追加要件が異なる
- **1.2.8 は条件付き追加要件**。既存の対応状況によって追加作業が不要な場合がある

| レベル | 達成基準 | 要件 |
|---|---|---|
| A | 1.2.3 | 音声解説 **または** メディアの代替（テキスト） |
| AA | 1.2.5 | **音声解説**が必須 |
| AAA | 1.2.8 | **拡張テキスト記述**が必須（条件付き） |

### 適合パターン

| 1.2.3 の対応 | 1.2.5 の対応 | 1.2.8 の追加要件 |
|---|---|---|
| 音声解説のみ | 自動的に満たす | **拡張テキスト記述を追加**する必要がある |
| テキスト代替のみ | 音声解説を追加 | 1.2.5 を満たしていれば**追加不要** |
| テキスト代替のみ | 未対応（A のみ） | 音声解説 + テキスト代替の**両方**が必要 |

```html
<!-- パターン1：音声解説のみ → 1.2.8 未達（AAA にはテキスト代替が追加必要） -->
<video controls aria-label="操作デモ動画">
  <source src="demo.mp4" type="video/mp4">
  <track kind="captions" src="demo.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="demo-ad.vtt" srclang="ja" label="音声解説">
</video>
<!-- → AA は満たすが、AAA には完全な transcript の追加が必要 -->

<!-- パターン2：テキスト代替 + 音声解説 → 1.2.8 も満たす -->
<video controls aria-label="操作デモ動画">
  <source src="demo.mp4" type="video/mp4">
  <track kind="captions" src="demo.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="demo-ad.vtt" srclang="ja" label="音声解説">
</video>
<p><a href="#full-transcript">動画の完全な文字起こし（映像描写含む）</a></p>
<section id="full-transcript"><!-- 完全な時間依存メディアの代替 --></section>
```

> **参照**
> - [Understanding SC 1.2.8 — Overlap with 1.2.3 and 1.2.5](https://www.w3.org/WAI/WCAG22/Understanding/media-alternative-prerecorded.html#intent)
> - [wcag/1-2-3-audio-description-or-media-alternative.md — 1.2.3 との関係](./1-2-3-audio-description-or-media-alternative.md)
> - [wcag/1-2-5-audio-description-prerecorded.md — 1.2.5 との関係](./1-2-5-audio-description-prerecorded.md)

## 1.2.7 拡張音声解説との関係

- **1.2.7**（拡張音声解説）と **1.2.8**（メディアの代替）は**別の AAA 要件**
- 1.2.7：台詞の休止が不十分な場合の**音声**ベースの解決策
- 1.2.8：**テキスト**ベースの完全な代替
- 休止不足の動画でも、1.2.8 の完全 transcript を提供すれば AAA の 1.2.8 は満たせる
- 1.2.7 と 1.2.8 は**相互に代替しない**。AAA 適合には**両方**を検討する必要がある場合がある

| 達成基準 | アプローチ | 主な受益者 |
|---|---|---|
| 1.2.7 拡張音声解説 | 動画を一時停止して音声解説を挿入 | 視覚障害者 |
| 1.2.8 メディアの代替 | 完全なテキスト記述 | 視覚・聴覚の両方に障害のあるユーザー |

> **参照**
> - [wcag/1-2-7-extended-audio-description.md — 1.2.7 拡張音声解説](./1-2-7-extended-audio-description.md)

## 実装方法（Sufficient Techniques）

### 状況A：同期メディア（G69 + G58）

- 動画の**視覚・聴覚のすべて**を時系列で記述したテキストを提供
- 動画の**直後**にリンクを配置（G58）

```html
<!-- 悪い例：概要だけの transcript -->
<video controls src="training.mp4" aria-label="トレーニング動画">
  <track kind="captions" src="training.vtt" srclang="ja" label="日本語">
</video>
<p>この動画では新技術の使い方を説明しています。</p>

<!-- 良い例：完全な時間依存メディアの代替 -->
<video controls src="training.mp4" aria-label="新技術トレーニング動画">
  <track kind="captions" src="training.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="training-ad.vtt" srclang="ja" label="音声解説">
</video>
<p><a href="#full-transcript">動画の完全な文字起こし（映像描写含む）</a></p>

<section id="full-transcript">
  <h3>新技術トレーニング動画 — 完全な記述</h3>
  <p><strong>ナレーター：</strong>「本日は新しい分析ツールの使い方を学びます。」</p>
  <p><em>（画面：ダッシュボードが表示される。左サイドバーに「分析」メニュー）</em></p>
  <p><strong>ナレーター：</strong>「まず、左メニューから分析を選択します。」</p>
  <p><em>（画面：カーソルが「分析」をクリック。グラフ画面に遷移）</em></p>
  <p><strong>ナレーター：</strong>「次に、期間を設定します。」</p>
  <p><em>（画面：日付ピッカーが表示され、2024年1月〜3月が選択される）</em></p>
  <p><em>（効果音：クリック音）</em></p>
</section>
```

### 状況B：映像のみ（G159）

- 音声のない収録済み映像に、映像内容を時系列で記述したテキストを提供

```html
<!-- 悪い例：映像のみに alt 的な短い説明だけ -->
<video controls src="assembly-demo.mp4" aria-label="組み立て手順"></video>
<p>組み立て手順を映像で示しています。</p>

<!-- 良い例：映像の完全なテキスト記述 -->
<video controls src="assembly-demo.mp4" aria-label="組み立て手順デモ"></video>
<p><a href="#assembly-transcript">映像の完全な記述</a></p>

<section id="assembly-transcript">
  <h3>組み立て手順 — 映像の記述</h3>
  <p><em>（画面：作業台の上に部品A・部品B・ネジ3本が並んでいる）</em></p>
  <p><em>（画面：作業者の手が部品Aを部品Bに差し込む）</em></p>
  <p><em>（画面：ネジを時計回りに3回締める。トルクレンチで「カチッ」と音）</em></p>
  <p><em>（画面：完成品が回転して全体像を見せる）</em></p>
</section>
```

### リンク配置のポイント（G58）

- メディアの代替テキストへのリンクは、動画の**直後**に配置
- `<details>` / 別セクション / 別ページのいずれか

```html
<!-- 良い例：動画の直後にリンク -->
<video controls src="lecture.mp4" aria-label="第3回講義"></video>
<p><a href="#lecture-transcript">講義の完全な文字起こし</a></p>

<!-- 悪い例：ページ末尾にのみリンク -->
<video controls src="lecture.mp4"></video>
<!-- ... 長い本文 ... -->
<p><a href="#lecture-transcript">文字起こし</a></p>
```

### インタラクティブな動画

- 動画内にインタラクション（クイズ、分岐等）がある場合、テキスト代替でも**同等の機能**を提供
- リンク・ボタンで同じ操作が可能にする

```html
<!-- インタラクティブ動画 + テキスト代替 -->
<video controls src="quiz-video.mp4" aria-label="理解度確認クイズ"></video>
<p><a href="#quiz-transcript">クイズの完全な記述（回答リンク付き）</a></p>

<section id="quiz-transcript">
  <h3>理解度確認クイズ — 完全な記述</h3>
  <p><strong>ナレーター：</strong>「次の図で示されているのは何でしょうか？」</p>
  <p><em>（画面：自由体図に3本の力のベクトルが描かれている）</em></p>
  <p><strong>選択肢：</strong></p>
  <ul>
    <li><a href="answer-a.html">A. 静力学の問題</a></li>
    <li><a href="answer-b.html">B. 動力学の問題</a></li>
    <li><a href="answer-c.html">C. 熱力学の問題</a></li>
  </ul>
</section>
```

> **参照**
> - [Technique G69: Providing an alternative for time based media](https://www.w3.org/WAI/WCAG22/Techniques/general/G69)
> - [Technique G58: Placing a link to the alternative immediately next to the non-text content](https://www.w3.org/WAI/WCAG22/Techniques/general/G58)
> - [Technique G159: Alternative for time-based media for video-only content](https://www.w3.org/WAI/WCAG22/Techniques/general/G159)

## メディアの代替に含めるべき内容

- すべての**台詞**（発言者を明示）
- **非発話音**（笑い、拍手、効果音、BGMの説明）
- **視覚的情報**（動作、登場人物、表情、場面転換、画面上のテキスト、UI操作）
- **時系列**は動画と同じ順序
- インタラクションがあれば、同等の操作手段（リンク等）
- キャプションや音声解説**だけでは不十分**。音声解説で省略された視覚描写も含める

### 音声解説 transcript との違い

```text
<!-- 音声解説（1.2.5）：休止部分に限定、簡潔 -->
00:00:12 — 講師がスライドを指差し、グラフが表示される。

<!-- メディアの代替（1.2.8）：完全記述、休止に制約なし -->
（画面：講師が右手指でスライド左上の折れ線グラフを指差す。
  グラフのタイトルは「2024年 Q1 売上推移」。縦軸は百万円、横軸は1月〜3月。
  1月120、2月95、3月150のデータポイント。3月に急上昇のトレンド）
```

> **参照**
> - [W3C WAI — Transcripts, Captions, and Audio Descriptions](https://www.w3.org/WAI/media/av/transcripts/)

## 1.2.2 キャプションとの関係

- 1.2.2（キャプション）と 1.2.8（メディアの代替）は**独立した要件**
- キャプションは**音声**をテキスト化。1.2.8 は**視覚・聴覚の両方**を完全にテキスト化
- AAA 適合時は**両方**を満たす必要がある
- 1.2.8 の transcript に台詞が含まれるため、キャプションと**内容が重複**するが、それぞれ独立した要件

| 達成基準 | 対象 | 提供するもの | レベル |
|---|---|---|---|
| 1.2.2 キャプション（収録済） | 音声トラック | 同期字幕 | A |
| 1.2.5 音声解説（収録済） | 映像トラック | 音声解説 | AA |
| 1.2.8 メディアの代替（収録済） | 視覚+聴覚 | 完全なテキスト代替 | AAA |

> **参照**
> - [wcag/1-2-2-captions-prerecorded.md — 1.2.2 キャプション](./1-2-2-captions-prerecorded.md)

## 代表的な失敗例（Failures）

### F74：テキストのメディア代替であることをラベル付けしていない

- 動画が**ページ上のテキスト以上の情報を持たない**場合、メディア代替として**明確にラベル付け**が必要
- ラベル付けなしでは 1.2.2 / 1.2.8 の例外として認められない

```html
<!-- F74：本文と同等の動画だが、メディア代替であることが不明 -->
<article>
  <p>ピストンが上下運動し、クランクシャフトを回転させます。</p>
  <video src="engine-demo.mp4" controls></video>
</article>

<!-- 良い例：メディア代替であることを明示 -->
<article>
  <p>ピストンが上下運動し、クランクシャフトを回転させます。</p>
  <video
    src="engine-demo.mp4"
    controls
    aria-label="エンジン内部の動作アニメーション（本文と同等の内容）"
  ></video>
  <p><small>上記動画は本文の内容を視覚的に示したものです。</small></p>
</article>
```

### その他のよくある問題

- **概要だけ**の transcript（「この動画では〇〇を説明しています」）
- 台詞のみの transcript で**映像描写が欠落**
- 音声解説の transcript をそのまま流用（休止部分の記述のみで**不完全**）
- リンクが動画から**離れた位置**に配置されている
- インタラクティブ動画で**回答手段**がテキスト代替にない

> **参照**
> - [Failure F74: Not labeling a synchronized media alternative to text as an alternative](https://www.w3.org/WAI/WCAG22/Techniques/failures/F74)

## テスト・確認方法

### 手動テスト

- transcript を**音声・映像なし**で読み、動画と**同等の情報**が得られるか
- 台詞・映像描写・非発話音が**すべて**含まれ、**時系列**が動画と一致するか
- インタラクティブ要素に**同等の操作手段**があるか
- リンクが動画の**直後**に配置されているか
- 1.2.3 で音声解説のみの場合、**拡張テキスト記述が追加**されているか（AAA 確認）

### ACT Rules

- [Audio and visuals of video element have transcript](https://www.w3.org/WAI/standards-guidelines/act/rules/1a02b0/proposed/)
- [Video element visual content has accessible alternative](https://www.w3.org/WAI/standards-guidelines/act/rules/c5a4ea/proposed/)

```bash
# 動画直後の transcript リンクを確認（開発者ツール Console）
document.querySelectorAll('video').forEach(v => {
  const next = v.nextElementSibling;
  const link = next?.querySelector('a[href*="transcript"]');
  console.log(v.currentSrc, link ? link.href : 'transcript link not found');
});
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.2.8 は**収録済み同期メディア・映像のみ**に**完全なテキスト代替**を要求（レベル AAA）
- 視覚・聴覚の両方に障害のあるユーザー（点字ディスプレイ利用者等）が主な受益者
- 1.2.3 でテキスト代替 + 1.2.5 で音声解説を満たしていれば、**追加不要**
- 1.2.3 / 1.2.5 を音声解説のみで満たした場合、**拡張テキスト記述の追加**が必要
- 台詞・映像描写・非発話音を**時系列**で完全に記述。音声解説 transcript より詳細
- 動画の直後にリンクを配置し、インタラクションがあれば同等の機能も提供
- 1.2.7（拡張音声解説）とは別要件。AAA 適合には両方を検討
