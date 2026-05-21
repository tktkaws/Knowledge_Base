# 1.2.1 音声のみ・映像のみ（収録済）（A）

## 達成基準の概要

- WCAG 2.2 達成基準 1.2.1「音声のみ及び映像のみ（収録済）」（Audio-only and Video-only (Prerecorded)）
- レベル A（最低限の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.2「時間依存メディア」に属する
- **収録済み**の音声のみ・映像のみメディアが対象（ライブは 1.2.9）
- 音声付き動画（映像+音声）は対象外（1.2.2 キャプションなどが対象）

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.1 Audio-only and Video-only (Prerecorded)](https://www.w3.org/TR/WCAG22/#audio-only-and-video-only-prerecorded)
> - [Understanding SC 1.2.1 Audio-only and Video-only (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-and-video-only-prerecorded.html)

## 達成基準の原文（要約）

- 収録済みの音声のみ・映像のみメディアについて、以下を満たすこと
- **例外**：音声・映像がテキストのメディア代替であり、それが明確にラベル付けされている場合

| 種類 | 要件 |
|---|---|
| 収録済み 音声のみ | 同等の情報を提供する**時間依存メディアの代替**（transcript 等）を提供 |
| 収録済み 映像のみ | **時間依存メディアの代替**、または同等の情報を提供する**音声トラック**のいずれかを提供 |

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.1](https://www.w3.org/TR/WCAG22/#audio-only-and-video-only-prerecorded)

## 対象となるメディア

### 音声のみ（audio-only）

- 音声だけを含む時間依存プレゼンテーション
- 映像もユーザー操作も含まない
- 具体例
  - ポッドキャスト
  - スピーチ・講演の録音
  - 音声ガイド
  - `<audio>` 要素で提供する録音ファイル

### 映像のみ（video-only）

- 映像だけを含む時間依存プレゼンテーション
- 音声もユーザー操作も含まない
- 具体例
  - サイレント映画
  - 音声なしのアニメーション解説
  - 監視カメラ映像（音声なし）
  - `<video>` 要素で提供する無音動画

### 対象外

- **音声付き動画**（映像+音声）→ 1.2.2 キャプション、1.2.3 音声解説など
- **ライブ**の音声のみ・映像のみ → 1.2.9
- **テキストのメディア代替**として提供されている音声・映像（後述の例外）

> **参照**
> - [Understanding SC 1.2.1 — Key Terms: audio-only, video-only](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-and-video-only-prerecorded.html#dfn-audio-only)

## なぜ必要か

- 聴覚障害者は音声のみコンテンツを直接知覚できない
- 視覚障害者は映像のみコンテンツを直接知覚できない
- テキストベースの代替は、視覚・聴覚・触覚（点字）など任意のモダリティで提示可能
- 認知・言語・学習障害のあるユーザーにとって、テキストと映像の並行提示は理解を助ける
- 検索・再利用・翻訳の基盤となる

> **参照**
> - [Understanding SC 1.2.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-and-video-only-prerecorded.html#benefits)

## 時間依存メディアの代替とは

- 時間順に整列された視覚・聴覚情報のテキスト記述を含むドキュメント
- 時間依存のインタラクションがあれば、その結果を達成する手段も提供
- 音声のみ → **文字起こし（transcript）** が典型的
- 映像のみ → **テキスト記述** または **音声解説付き音声トラック** が典型的
- 1.1.1 の短い識別ラベル（「会長のスピーチ」など）だけでは不十分。**同等の情報**が必要

> **参照**
> - [Understanding SC 1.2.1 — Key Terms: alternative for time-based media](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-and-video-only-prerecorded.html#dfn-alternative-for-time-based-media)

## 例外：テキストのメディア代替

- 音声・映像が**テキストで既に提示されている情報以上の内容を持たない**場合
- かつ、それがメディア代替であることが**明確にラベル付け**されている場合
- 1.2.1 の詳細な代替要件は適用されない（1.1.1 の識別ラベルで足りる場合がある）

```html
<!-- 良い例：チュートリアル本文が完全な説明、動画はその代替 -->
<article>
  <h2>エンジンの仕組み</h2>
  <p>ピストンが上下運動し、クランクシャフトを回転させます。...</p>
  <video
    src="engine-animation.mp4"
    aria-label="エンジン内部の動作を示すアニメーション（本文と同等の内容）"
  ></video>
  <p><small>上記アニメーションは本文の内容を視覚的に示したものです。</small></p>
</article>
```

```html
<!-- 悪い例：本文が概要のみで、動画に追加情報があるのにメディア代替と称していない -->
<article>
  <h2>エンジンの仕組み</h2>
  <p>エンジンについて学びましょう。</p>
  <video src="engine-animation.mp4" aria-label="エンジンのアニメーション"></video>
  <!-- 動画にしかない詳細説明がある → 1.2.1 の代替が必要 -->
</article>
```

> **参照**
> - [Understanding SC 1.2.1 — Key Terms: media alternative for text](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-and-video-only-prerecorded.html#dfn-media-alternative-for-text)

## 状況別対応（Sufficient Techniques）

### Situation A：収録済み 音声のみ

- **Technique G158**：音声のみコンテンツに時間依存メディアの代替を提供
- 文字起こし（transcript）が最も一般的
- 発言者の特定、拍手・笑い・質疑など**有意な音**も記述

```html
<!-- 悪い例：音声ファイルのみ、代替なし -->
<audio src="speech.mp3" controls></audio>

<!-- 悪い例：識別ラベルのみ（1.1.1 レベル、1.2.1 としては不十分） -->
<audio src="speech.mp3" controls aria-label="会長のスピーチ"></audio>

<!-- 良い例：文字起こしへのリンクを音声の直後に配置 -->
<audio src="speech.mp3" controls aria-label="会長の総会スピーチ"></audio>
<p><a href="#transcript">文字起こしを読む</a></p>

<section id="transcript">
  <h3>文字起こし：会長の総会スピーチ</h3>
  <p><strong>会長：</strong>本日はご参加いただきありがとうございます。</p>
  <p><strong>会長：</strong>第3四半期の業績について報告します。</p>
  <p><em>（拍手）</em></p>
  <p><strong>会員：</strong>質問があります。</p>
  <!-- 発言内容・有意な音を時系列で記述 -->
</section>
```

```html
<!-- 悪い例：ファイル名やプレースホルダーを代替にしている（Failure F30） -->
<audio src="recording_001.mp3" controls></audio>
<p>文字起こし：recording_001.mp3</p>

<!-- 良い例：音声の内容を正確に書き起こした transcript -->
<audio src="press-conference.mp3" controls></audio>
<details>
  <summary>記者会見の文字起こし</summary>
  <p><strong>司会：</strong>本日の記者会見を始めます。</p>
  <p><strong>CEO：</strong>新製品の発表についてお話しします。</p>
  <p><em>（カメラのシャッター音）</em></p>
</details>
```

> **参照**
> - [Technique G158: Alternative for time-based media for audio-only content](https://www.w3.org/WAI/WCAG22/Techniques/general/G158)
> - [W3C WAI — Transcripts](https://www.w3.org/WAI/media/av/transcripts/)

### Situation B：収録済み 映像のみ

- **Technique G159**：映像のみコンテンツに時間依存メディアの代替（テキスト）を提供
- **Technique G166**：重要な映像内容を説明する音声トラックを提供し、それが音声解説であることを明示
- テキスト代替 **または** 音声トラックの**いずれか一方**で適合可能（両方推奨）

#### 方法1：テキストによる代替（transcript / 詳細記述）

```html
<!-- 悪い例：無音動画のみ -->
<video src="silent-demo.mp4" controls></video>

<!-- 悪い例：概要だけでは同等の情報にならない -->
<video src="silent-demo.mp4" controls aria-label="操作デモ"></video>

<!-- 良い例：映像の内容を時系列で記述した transcript -->
<video src="silent-demo.mp4" controls aria-label="ソフトウェア操作デモ"></video>
<p><a href="#video-transcript">映像の文字起こし</a></p>

<section id="video-transcript">
  <h3>操作デモの内容</h3>
  <ol>
    <li>ログイン画面が表示される</li>
    <li>ユーザーがメールアドレスとパスワードを入力する</li>
    <li>「ログイン」ボタンをクリックする</li>
    <li>ダッシュボード画面に遷移する</li>
  </ol>
</section>
```

#### 方法2：音声トラックによる代替

```html
<!-- 良い例：映像の内容を説明する音声トラックを追加 -->
<video controls aria-label="サイレント映画：街の風景">
  <source src="silent-movie.mp4" type="video/mp4">
  <track
    kind="descriptions"
    src="silent-movie-audio-desc.vtt"
    srclang="ja"
    label="映像内容の音声解説"
  >
</video>
<p><small>この映像には、画面の動きを説明する音声トラックが含まれます。</small></p>
```

```html
<!-- サイレント映画の例：音声トラックで視覚的内容を説明 -->
<!-- silent-movie-audio-desc.vtt の内容例 -->
<!--
WEBVTT

00:00:00.000 --> 00:00:05.000
黒白映像。賑やかな街角。人々が歩き交う。

00:00:05.000 --> 00:00:10.000
男性がカメラに向かって走ってくる。帽子を取る。
-->
```

- 映像に音声情報がない場合、その音声解説トラック自体にキャプションは不要（Understanding SC 1.2.1 の Note）
- `<track kind="descriptions">` の利用は Advisory Technique H96

> **参照**
> - [Technique G159: Alternative for time-based media for video-only content](https://www.w3.org/WAI/WCAG22/Techniques/general/G159)
> - [Technique G166: Providing audio that describes the important video content](https://www.w3.org/WAI/WCAG22/Techniques/general/G166)
> - [Technique H96: Using the track element to provide audio descriptions](https://www.w3.org/WAI/WCAG22/Techniques/html/H96)

## 文字起こし（transcript）の書き方

### 音声のみの transcript

- 発言内容を**逐語的（verbatim）**に記述
- **誰が話しているか**を明示（話者名・役割）
- 拍手・笑い・沈黙・BGM・効果音など**有意な音**も記述
- 音声の直後、または近傍にリンクを配置

### 映像のみの transcript

- 画面上の**視覚的情報**を時系列で記述
- テキスト・UI操作・人物の動き・場面転換などを含める
- サイレント映画の場合、登場人物の行動・表情・場面設定を説明

```html
<!-- 悪い例：映像 transcript が視覚情報と一致しない（Failure F67） -->
<video src="tutorial.mp4" controls></video>
<section id="transcript">
  <p>この動画では操作方法を説明します。</p>
  <!-- 実際の映像内容と異なる・不足している -->
</section>

<!-- 良い例：映像と同等の情報を時系列で記述 -->
<video src="tutorial.mp4" controls></video>
<section id="transcript">
  <h3>映像の内容</h3>
  <p>00:00 — ホーム画面。ナビゲーションに「設定」リンクが表示される。</p>
  <p>00:05 — 「設定」をクリック。アカウント設定ページに遷移。</p>
  <p>00:12 — 「通知」タブを選択。通知のオン/オフ切り替えが表示される。</p>
</section>
```

> **参照**
> - [W3C WAI — Transcripts on the Web](http://www.uiaccess.com/transcripts/transcripts_on_the_web.html)
> - [W3C WAI — Making Audio and Video Media Accessible](https://www.w3.org/WAI/media/av/)

## 代表的な失敗例（Failures）

- **F30**：ファイル名やプレースホルダーを代替テキストにしている
- **F67**：非テキストコンテンツと同等の目的・情報を提供しない長い説明

```html
<!-- F30：ファイル名を transcript として提示 -->
<audio src="meeting-2024-03-01.mp3" controls></audio>
<p>文字起こし：<a href="meeting-2024-03-01.mp3">meeting-2024-03-01.mp3</a></p>

<!-- F30：「文字起こしあり」と書くだけで内容がない -->
<audio src="speech.mp3" controls></audio>
<p>文字起こしは準備中です</p>

<!-- F67：概要だけで音声の内容が伝わらない -->
<audio src="press-conference.mp3" controls></audio>
<p>記者会見の録音です。CEOが新製品について話しています。</p>
<!-- → 実際の発言内容が transcript に含まれていない -->
```

> **参照**
> - [Failure F30: Text alternatives that are not alternatives](https://www.w3.org/WAI/WCAG22/Techniques/failures/F30)
> - [Failure F67: Long descriptions that do not serve the same purpose](https://www.w3.org/WAI/WCAG22/Techniques/failures/F67)

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.1.1 非テキストコンテンツ（A） | 識別用ラベルは必要だが、1.2.1 では**同等情報**の代替が追加で必要 |
| 1.2.2 キャプション（収録済）（A） | 音声付き動画の字幕要件（1.2.1 とは別） |
| 1.2.3 音声解説またはメディアの代替（A） | 音声付き動画の音声解説要件（1.2.1 とは別） |
| 1.2.9 音声のみ（ライブ）（AAA） | ライブ音声のみの識別ラベル要件 |

### 1.1.1 と 1.2.1 の違い

- **1.1.1**：非テキストコンテンツに識別用ラベル（「会長のスピーチ」など）で最低限対応可能
- **1.2.1**：時間依存メディアには**同等の情報**を持つ transcript または音声トラックが必要
- 音声のみ・映像のみメディアは**両方**を満たす必要がある

```html
<!-- 1.1.1 は満たすが 1.2.1 は満たさない例 -->
<audio src="speech.mp3" controls aria-label="会長の総会スピーチ"></audio>
<!-- → 識別ラベルはあるが、発言内容の transcript がない -->

<!-- 1.1.1 と 1.2.1 の両方を満たす例 -->
<audio src="speech.mp3" controls aria-label="会長の総会スピーチ"></audio>
<a href="#transcript">文字起こし</a>
<section id="transcript"><!-- 全文の transcript --></section>
```

> **参照**
> - [wcag/1-1-1-non-text-content.md — 1.1.1 非テキストコンテンツ](./1-1-1-non-text-content.md)
> - [WCAG 2.2 — Guideline 1.2 Time-based Media](https://www.w3.org/TR/WCAG22/#time-based-media)

## HTML実装のポイント

### `<audio>` 要素

- `controls` 属性で再生コントロールを提供
- `aria-label` または近傍の見出しで内容を識別（1.1.1）
- transcript は `<details>` / 別セクション / 別ページのいずれかで提供

```html
<figure>
  <figcaption id="podcast-title">第42回：Webアクセシビリティ入門</figcaption>
  <audio src="episode-42.mp3" controls aria-labelledby="podcast-title"></audio>
  <p><a href="#ep42-transcript">文字起こし</a></p>
</figure>
```

### `<video>` 要素（映像のみ）

- 音声トラックがない場合、`muted` 属性の有無に関わらず映像のみとして扱う
- transcript または `<track kind="descriptions">` で代替
- 自動再生は 1.4.2 音声の制御も考慮

```html
<!-- 映像のみ：transcript リンク -->
<video src="demo.mp4" controls aria-label="操作手順のデモ映像"></video>
<p><a href="#demo-transcript">映像の内容（テキスト）</a></p>

<!-- 映像のみ：音声解説トラック -->
<video controls>
  <source src="animation.mp4" type="video/mp4">
  <track kind="descriptions" src="animation-desc.vtt" srclang="ja" label="音声解説">
</video>
```

> **参照**
> - [MDN — audio element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio)
> - [MDN — video element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)
> - [MDN — track element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track)

## テスト・確認方法

### 自動テスト

- `<audio>` 要素に関連する transcript リンクの有無
- `<video>` 要素（音声トラックなし）への代替の有無
- ACT Rules による機械的判定（提案段階）
  - [Audio element content has text alternative](https://www.w3.org/WAI/standards-guidelines/act/rules/e7aa44/proposed/)
  - [Video element visual-only content has transcript](https://www.w3.org/WAI/standards-guidelines/act/rules/ee13b5/proposed/)

### 手動テスト

- transcript を読み、音声・映像と**同等の情報**が得られるか確認
- 話者の特定・有意な音（音声）・視覚的変化（映像）が transcript に含まれるか
- transcript へのリンクがメディアの近傍にあるか
- 「テキストのメディア代替」として例外適用する場合、本文と映像の情報量が本当に同等か

```bash
# ページ内の audio / video 要素を確認（開発者ツール Console）
document.querySelectorAll('audio, video').forEach(el => {
  console.log(el.tagName, el.src || el.currentSrc, el.textTracks?.length);
});
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.2.1 は**収録済み**の音声のみ・映像のみメディアに、**同等の情報**を持つ代替を要求
- 音声のみ → 文字起こし（transcript）が必須
- 映像のみ → テキスト記述 **または** 音声解説トラックのいずれか
- 1.1.1 の識別ラベルだけでは不十分。発言内容・視覚情報を transcript に含める
- テキストのメディア代替として明確にラベル付けされている場合は例外
- 音声付き動画は 1.2.2 以降が対象。1.2.1 とは混同しない
