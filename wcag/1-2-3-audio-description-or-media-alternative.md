# 1.2.3 音声解説またはメディアの代替（収録済）（A）

## 達成基準の概要

- WCAG 2.2 達成基準 1.2.3「音声解説（収録済）及びメディアの代替（収録済）」（Audio Description or Media Alternative (Prerecorded)）
- レベル A（最低限の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.2「時間依存メディア」に属する
- **収録済み**の**同期メディア**（映像+音声）が対象
- **音声解説** または **時間依存メディアの代替（テキスト）** の**いずれか一方**で適合可能

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.3 Audio Description or Media Alternative (Prerecorded)](https://www.w3.org/TR/WCAG22/#audio-description-or-media-alternative-prerecorded)
> - [Understanding SC 1.2.3 Audio Description or Media Alternative (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-or-media-alternative-prerecorded.html)

## 達成基準の原文（要約）

- 収録済みの同期メディアに対し、**時間依存メディアの代替** または **音声解説** を提供すること
- **例外**：メディアがテキストのメディア代替であり、それが明確にラベル付けされている場合

| 選択肢 | 内容 |
|---|---|
| 音声解説（audio description） | 映像の重要な視覚情報を、既存の音声トラックの休止部分にナレーションで追加 |
| 時間依存メディアの代替 | 映像・音声の両方の情報を時系列で記述したテキスト（脚本形式） |

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.3](https://www.w3.org/TR/WCAG22/#audio-description-or-media-alternative-prerecorded)

## 対象となるメディア

### 同期メディア（synchronized media）

- 映像と音声が同期した時間依存プレゼンテーション
- 具体例
  - 解説付きトレーニング動画
  - ニュース映像
  - 講義・セミナーの録画
  - `<video>` 要素で提供する音声付き動画

### 対象外

- **音声のみ**・**映像のみ** → 1.2.1
- **ライブ**の同期メディア → 1.2.4 / 1.2.5 など
- **テキストのメディア代替**として明確にラベル付けされたメディア（例外）
- 音声トラックですでにすべての重要な映像情報が伝わっている場合 → 追加の音声解説は不要

> **参照**
> - [Understanding SC 1.2.3 — Key Terms: synchronized media](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-or-media-alternative-prerecorded.html#dfn-synchronized-media)

## なぜ必要か

- 視覚障害者は映像の視覚情報を直接知覚できない
- 1.2.2 キャプションは**音声**をテキスト化するが、**映像**の情報はカバーしない
- 画面上の動作・登場人物・場面転換・テロップなど、音声だけでは伝わらない情報を提供する必要
- テキストベースの代替は、視覚・聴覚・触覚など任意のモダリティで提示可能

> **参照**
> - [Understanding SC 1.2.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-or-media-alternative-prerecorded.html#benefits)

## 音声解説とメディアの代替の違い

| 項目 | 音声解説 | 時間依存メディアの代替 |
|---|---|---|
| 形式 | 音声トラックにナレーション追加 | テキスト（脚本・transcript 形式） |
| 記述範囲 | 既存の台詞の**休止部分**に挿入 | 台詞の休止に**制約なし**。映像・音声を完全に記述 |
| 情報量 | 重要な視覚情報に限定 | 視覚・聴覚の**すべて**を時系列で記述 |
| 典型例 | 映画の音声解説版 | 動画の完全な文字起こし + 映像描写 |
| 1.2.5（AA） | これを選べば AA も満たせる | AA では追加で音声解説が必要 |

### 音声解説（audio description）とは

- メインの音声トラックだけでは理解できない**重要な視覚的詳細**をナレーションで補足
- 台詞・ナレーションの**既存の休止部分**に挿入（標準的な音声解説）
- 説明対象：動作、登場人物、場面転換、画面上のテキストなど
- 別名：video description、descriptive narration

### 時間依存メディアの代替とは

- 同期メディアの**視覚・聴覚の両方**を時系列で記述したドキュメント
- 脚本や小説のように読める形式
- 台詞の transcript + 非発話音（笑い・効果音）+ 映像描写を**同じ順序**で含める
- インタラクション（「今すぐ回答を押してください」など）があれば、同等の機能も提供

> **参照**
> - [Understanding SC 1.2.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-or-media-alternative-prerecorded.html#intent)
> - [W3C WAI — Audio Descriptions](https://www.w3.org/WAI/media/av/descriptions/)

## 追加の音声解説が不要な場合

- 映像トラックの**すべての重要な情報**が、すでに音声トラックで伝わっている場合
- 典型例：**トーキングヘッド**（話者の顔だけが映るインタビュー・講義）
- Advisory Technique G203：トーキングヘッド動画は静的テキスト代替で十分な場合がある

```html
<!-- 音声解説不要：話者の発言がすべての情報 -->
<video src="interview.mp4" controls aria-label="CEOインタビュー">
  <track kind="captions" src="interview.vtt" srclang="ja" label="日本語">
</video>
<!-- 映像は話者の顔のみ。内容はすべて音声（→ 1.2.2 キャプションで足りる） -->

<!-- 音声解説必要：映像に追加情報がある -->
<video src="tutorial.mp4" controls aria-label="ソフトウェア操作チュートリアル">
  <track kind="captions" src="tutorial.vtt" srclang="ja" label="日本語">
</video>
<!-- 画面操作・UI・マウスの動きが映像にのみ存在 → 1.2.3 の代替が必要 -->
```

> **参照**
> - [Understanding SC 1.2.3 — Note on audio description not necessary](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-or-media-alternative-prerecorded.html#intent)
> - [Technique G203: Static text alternative for talking head video](https://www.w3.org/WAI/WCAG22/Techniques/general/G203)

## 例外：テキストのメディア代替

- 動画が**ページ上のテキスト以上の情報を持たない**場合
- かつ、メディア代替であることが**明確にラベル付け**されている場合
- 1.2.3 の音声解説 / メディアの代替要件は適用されない

```html
<!-- 良い例：本文が完全な説明、動画は視覚的な補助 -->
<article>
  <h2>エンジンの仕組み</h2>
  <p>ピストンが上下運動し、クランクシャフトを回転させます。...</p>
  <video
    src="engine-demo.mp4"
    controls
    aria-label="エンジン内部の動作アニメーション（本文と同等の内容）"
  ></video>
  <p><small>上記動画は本文の内容を視覚的に示したものです。</small></p>
</article>
```

> **参照**
> - [Understanding SC 1.2.3 — Key Terms: media alternative for text](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-or-media-alternative-prerecorded.html#dfn-media-alternative-for-text)

## 方法1：音声解説（Technique G173）

- 音声解説付きバージョンの動画を提供
- 実装方法
  - 別ファイルとして音声解説版を用意
  - ナレーションを元の音声に組み込む（G226）
  - `<track kind="descriptions">` で提供（H96）
  - SMIL で音声解説を同期（SM6 / SM7）

```html
<!-- 悪い例：キャプションのみで映像の視覚情報が伝わらない -->
<video controls>
  <source src="training.mp4" type="video/mp4">
  <track kind="captions" src="training.vtt" srclang="ja" label="日本語">
</video>

<!-- 良い例：音声解説トラックを追加 -->
<video controls aria-label="新技術トレーニング動画">
  <source src="training.mp4" type="video/mp4">
  <track kind="captions" src="training.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="training-ad.vtt" srclang="ja" label="音声解説">
</video>
<p><small>音声解説トラックをオンにすると、映像の内容がナレーションされます。</small></p>
```

```html
<!-- 良い例：音声解説版を別リンクで提供 -->
<video controls aria-label="操作デモ動画">
  <source src="demo.mp4" type="video/mp4">
  <track kind="captions" src="demo.vtt" srclang="ja" label="日本語">
</video>
<p><a href="demo-with-ad.mp4">音声解説付きバージョンを視聴</a></p>
```

### 音声解説の記述例

```
（タイトル表示：「進化のケーススタディ — Bonnie Chen」）

解説者：教師が長いくちばしの鳥の写真を見せている。

Bonnie Chen：「これらの写真はすべてエバーグレーズで撮影されました。」

解説者：教師が各生徒に2本の薄い木の棒を渡す。

Bonnie Chen：「今日は、こういうくちばしを持つ水鳥の仲間を演じてもらいます。」

解説者：教師が2本の棒を口元に当て、くちばしの形を作る。
```

- 台詞の**休止部分**に視覚情報のナレーションを挿入
- 休止が短すぎる場合は**拡張音声解説**（1.2.7 / Technique G8）を検討

> **参照**
> - [Technique G173: Providing a version of a movie with audio descriptions](https://www.w3.org/WAI/WCAG22/Techniques/general/G173)
> - [Technique G226: Providing audio descriptions by incorporating narration in the soundtrack](https://www.w3.org/WAI/WCAG22/Techniques/general/G226)
> - [Technique H96: Using the track element to provide audio descriptions](https://www.w3.org/WAI/WCAG22/Techniques/html/H96)

## 方法2：時間依存メディアの代替（Technique G69）

- 動画の**視覚・聴覚のすべて**を時系列で記述したテキストを提供
- 動画の**直後**にリンクを配置（Technique G58）
- 台詞休止に制約がないため、音声解説より**完全な情報**を提供可能

```html
<!-- 悪い例：概要だけの transcript -->
<video controls src="training.mp4" aria-label="トレーニング動画"></video>
<p>この動画では新技術の使い方を説明しています。</p>

<!-- 良い例：完全な時間依存メディアの代替 -->
<video controls src="training.mp4" aria-label="新技術トレーニング動画">
  <track kind="captions" src="training.vtt" srclang="ja" label="日本語">
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
  <!-- 台詞・映像描写・非発話音を時系列で記述 -->
</section>
```

### メディアの代替に含めるべき内容

- すべての**台詞**（発言者を明示）
- **非発話音**（笑い、拍手、効果音、BGMの説明）
- **視覚的情報**（動作、登場人物、場面転換、画面上のテキスト、UI操作）
- **時系列**は動画と同じ順序
- インタラクションがあれば、同等の操作手段（リンク等）

> **参照**
> - [Technique G69: Providing an alternative for time based media](https://www.w3.org/WAI/WCAG22/Techniques/general/G69)
> - [Technique G58: Placing a link to the alternative immediately next to the non-text content](https://www.w3.org/WAI/WCAG22/Techniques/general/G58)

## 1.2.2 キャプションとの関係

- 1.2.2 と 1.2.3 は**独立した要件**。両方を満たす必要がある
- 1.2.2：音声トラックの**キャプション**（聴覚障害者向け）
- 1.2.3：映像トラックの**音声解説またはテキスト代替**（視覚障害者向け）

| 達成基準 | 対象 | 提供するもの |
|---|---|---|
| 1.2.2 キャプション（A） | 音声 | 字幕（台詞 + 非発話音） |
| 1.2.3 音声解説またはメディアの代替（A） | 映像 | 音声解説 **または** 完全なテキスト代替 |

```html
<!-- 1.2.2 + 1.2.3 を両方満たす例 -->
<video controls aria-label="製品デモ動画">
  <source src="product-demo.mp4" type="video/mp4">
  <track kind="captions" src="demo-captions.vtt" srclang="ja" label="日本語字幕">
  <track kind="descriptions" src="demo-ad.vtt" srclang="ja" label="音声解説">
</video>
```

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.2 Captions (Prerecorded)](https://www.w3.org/TR/WCAG22/#captions-prerecorded)
> - [wcag/1-2-1-audio-only-and-video-only.md — 1.2.1 との違い](./1-2-1-audio-only-and-video-only.md)

## 上位レベルとの関係（1.2.5 / 1.2.8）

- 1.2.3 / 1.2.5 / 1.2.8 は内容が重複するが、適合レベルごとに選択肢と追加要件が異なる

| レベル | 達成基準 | 要件 |
|---|---|---|
| A | 1.2.3 | 音声解説 **または** メディアの代替（テキスト） |
| AA | 1.2.5 | **音声解説**が必須 |
| AAA | 1.2.8 | **拡張テキスト記述**が必須（条件付き） |

### 適合パターン

- **1.2.3 で音声解説を選んだ場合** → 1.2.5（AA）も自動的に満たす。1.2.8（AAA）では拡張テキスト記述が追加要件
- **1.2.3 でテキスト代替を選んだ場合** → 1.2.5（AA）で追加の音声解説が必要。1.2.8（AAA）は 1.2.5 も満たしていれば追加不要

> **参照**
> - [Understanding SC 1.2.3 — Overlap with 1.2.5 and 1.2.8](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-or-media-alternative-prerecorded.html#intent)
> - [WCAG 2.2 — Success Criterion 1.2.5 Audio Description (Prerecorded)](https://www.w3.org/TR/WCAG22/#audio-description-prerecorded)
> - [WCAG 2.2 — Success Criterion 1.2.8 Media Alternative (Prerecorded)](https://www.w3.org/TR/WCAG22/#media-alternative-prerecorded)

## HTML実装のポイント

### `<track kind="descriptions">`

- 音声解説トラックを WebVTT 形式で提供
- ブラウザ・プレイヤーによって対応状況が異なる（H96 は Advisory）
- 確実性を求める場合は音声解説版の別ファイル（G173）も併用

```html
<video controls>
  <source src="movie.mp4" type="video/mp4">
  <track kind="captions" src="movie-captions.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="movie-ad.vtt" srclang="ja" label="音声解説" default>
</video>
```

```text
<!-- movie-ad.vtt の内容例 -->
WEBVTT

00:00:05.000 --> 00:00:08.000
タイトル画面。「Teaching Evolution Case Studies」と表示。

00:00:10.000 --> 00:00:13.000
教師が鳥の写真を見せている。

00:00:18.000 --> 00:00:21.000
教師が生徒に2本の木の棒を渡す。
```

### リンク配置

- メディアの代替テキストへのリンクは、動画の**直後**に配置（G58）
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

> **参照**
> - [MDN — track element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track)
> - [MDN — WebVTT](https://developer.mozilla.org/en-US/docs/Web/API/WebVTT_API)

## テスト・確認方法

### 自動テスト

- 音声付き `<video>` に `descriptions` トラックまたは代替テキストリンクの有無
- ACT Rule：[Video element visual content has accessible alternative](https://www.w3.org/WAI/standards-guidelines/act/rules/c5a4ea/proposed/)

### 手動テスト

- **音声のみ**で動画を視聴し、視覚情報が伝わるか確認（音声解説の場合）
- transcript を読み、映像の内容と**同等の情報**が得られるか確認（テキスト代替の場合）
- 1.2.2 キャプションと独立して確認（キャプションがあっても 1.2.3 は別要件）
- トーキングヘッド動画など、音声解説が本当に不要か判断

```bash
# video 要素の track 一覧を確認（開発者ツール Console）
document.querySelectorAll('video').forEach(v => {
  console.log(v.src, [...v.textTracks].map(t => `${t.kind}: ${t.label}`));
});
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.2.3 は**収録済み同期メディア**の**映像情報**を視覚障害者に伝えるための達成基準
- **音声解説** または **時間依存メディアの代替（テキスト）** のいずれかで適合
- 1.2.2 キャプション（音声→テキスト）とは**別要件**。両方必要
- 音声ですべての映像情報が伝わる場合（トーキングヘッド等）は追加の音声解説不要
- AA（1.2.5）では音声解説が必須。A でテキスト代替を選んだ場合は AA で追加対応が必要
- 動画の直後に代替へのリンクを配置し、台詞・映像描写・非発話音を時系列で記述
