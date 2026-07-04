# 1.4.7 小さな背景音または背景音なし（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.7「小さな背景音または背景音なし」（Low or No Background Audio）
- レベル **AAA**（最高の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- **収録済みの音声のみコンテンツ**で、主に前景の発話を含む場合、背景音をなくす・消せる・十分に小さくする
- 難聴のあるユーザーが、発話と背景音を聞き分けられるようにするための基準

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.7 Low or No Background Audio](https://www.w3.org/TR/WCAG22/#low-or-no-background-audio)
> - [Understanding SC 1.4.7 Low or No Background Audio](https://www.w3.org/WAI/WCAG22/Understanding/low-or-no-background-audio.html)

## 達成基準の原文（要約）

- 次の条件をすべて満たす**収録済み音声のみコンテンツ**が対象
  - 主に**前景の発話**を含む
  - 音声 CAPTCHA または音声ロゴではない
  - 歌唱・ラップなど、主に音楽表現として意図された発声ではない
- 対象コンテンツでは、次のいずれかを満たすこと
  - 背景音がない
  - 背景音をオフにできる
  - 背景音が前景の発話より **20dB 以上**小さい（一時的な1〜2秒の音を除く）

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.7](https://www.w3.org/TR/WCAG22/#low-or-no-background-audio)

## 対象となるコンテンツ

### 対象

- **収録済み**の**音声のみ**コンテンツ
- 主に**発話**で情報を伝えるもの
- 具体例
  - ポッドキャスト
  - 音声教材
  - ナレーション音声
  - インタビュー録音
  - 音声ガイド

```html
<!-- 対象：収録済み音声のみ、発話が主 -->
<audio controls src="lesson.mp3">
  <a href="lesson.mp3">音声教材をダウンロード</a>
</audio>
```

### 対象外

- 映像付き動画（音声のみではない）
- ライブ音声（収録済みではない）
- 音声 CAPTCHA
- 音声ロゴ
- 歌唱・ラップなど、主に音楽表現として意図された発声

```html
<!-- 対象外：映像付き動画 -->
<video controls src="interview.mp4"></video>

<!-- 対象外：ライブ音声 -->
<audio controls src="https://radio.example.com/live"></audio>

<!-- 対象外：歌唱が主目的 -->
<audio controls src="song.mp3"></audio>
```

> **参照**
> - [Understanding SC 1.4.7 — Success Criterion](https://www.w3.org/WAI/WCAG22/Understanding/low-or-no-background-audio.html#success-criterion)

## なぜ必要か

- 難聴のあるユーザーは、発話と背景音を分離して聞き取ることが難しい場合がある
- BGM・環境音・効果音が大きいと、発話の内容が理解できない
- 前景の発話が情報の中心である場合、背景音は十分に小さい必要がある
- 発話を聞き取れることは、キャプションや transcript とは別の音声体験のアクセシビリティ

> **参照**
> - [Understanding SC 1.4.7 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/low-or-no-background-audio.html#intent)
> - [Understanding SC 1.4.7 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/low-or-no-background-audio.html#benefits)

## 適合する3つの方法

| 方法 | 内容 | 典型例 |
|---|---|---|
| 背景音なし | 発話以外の背景音を入れない | 音声教材、朗読、ニュース読み上げ |
| 背景音をオフにできる | BGM・効果音なし版を選べる | BGMあり/なし音声の切替 |
| 20dB以上小さい | 背景音を発話より十分小さくミックス | ポッドキャストの薄いBGM |

## 方法1：背景音なし

- 最も確実な方法
- 発話の理解が主目的なら、BGMや環境音を入れない
- 効果音が必要な場合も、発話に重ならない短い音にする

```html
<!-- 良い例：発話のみの音声教材 -->
<figure>
  <figcaption>第1回：CSS Grid 入門</figcaption>
  <audio controls src="css-grid-lesson-voice-only.mp3"></audio>
</figure>
```

```text
音声構成：
00:00 講師の発話
00:10 講師の発話
00:30 講師の発話
背景BGMなし
```

## 方法2：背景音をオフにできる

- BGMあり版とBGMなし版を提供
- プレイヤー上で音声トラックを切り替えられる
- 同じ情報を、背景音なしで聞けることが重要

```html
<!-- 良い例：BGMなし版を別ファイルで提供 -->
<figure>
  <figcaption>製品紹介ナレーション</figcaption>
  <audio controls src="product-intro-with-bgm.mp3"></audio>
  <p>
    <a href="product-intro-voice-only.mp3">
      BGMなし版を再生
    </a>
  </p>
</figure>
```

```html
<!-- 悪い例：BGMあり版しかない -->
<audio controls src="product-intro-with-loud-bgm.mp3"></audio>
```

## 方法3：背景音を20dB以上小さくする（G56）

- 背景音が発話より **20dB 以上**低い音量になるようにミックス
- 20dB差は、背景音が前景発話よりおおよそ**4分の1程度に聞こえる**差として扱われる
- 1〜2秒程度の一時的な音は例外として許容される
- 発話に重なるBGM・環境音・効果音は特に注意

```text
良い例：
前景発話: -16 LUFS
背景BGM: -36 LUFS 相当
→ 差が約20dB

悪い例：
前景発話: -16 LUFS
背景BGM: -24 LUFS 相当
→ 差が約8dBで、発話を妨げる可能性
```

> **参照**
> - [Technique G56: Mixing audio files so that non-speech sounds are at least 20 decibels lower than speech](https://www.w3.org/WAI/WCAG22/Techniques/general/G56)

## 一時的な音の扱い

- 1〜2秒程度の短い効果音は、20dB差の例外として扱われる
- ただし、発話の重要部分に重なる場合は避ける
- 頻繁に鳴る短い音は、実質的に背景音として聞こえるため注意

```text
許容されやすい例：
章の切り替わりに1秒だけチャイムが鳴る

避けたい例：
講師の説明中にクリック音や通知音が何度も鳴る
```

## 音声CAPTCHA・音声ロゴ・音楽表現の例外

### 音声CAPTCHA

- 背景ノイズが、人間と機械を区別する目的に関わる場合がある
- ただし、音声CAPTCHA自体は多くのユーザーに困難を生むため、別モダリティの代替が必要

```html
<!-- 音声 CAPTCHA は本基準の対象外だが、代替手段が必要 -->
<button type="button">別の確認方法を選択</button>
```

### 音声ロゴ

- ブランドを表す短いジングル等
- 発話の理解を主目的とする音声ではない

### 歌唱・ラップなど

- 発声自体が音楽表現の中心である場合は対象外
- 歌詞の理解支援には、歌詞テキストやキャプション等を別途提供するとよい

> **参照**
> - [WCAG 2.2 — CAPTCHA definition](https://www.w3.org/TR/WCAG22/#dfn-captcha)
> - [wcag/1-1-1-non-text-content.md — CAPTCHA](./1-1-1-non-text-content.md)

## 1.4.2 音声の制御との関係

| 達成基準 | レベル | 対象 | 主な要件 |
|---|---|---|---|
| 1.4.2 音声の制御 | A | 3秒超の自動再生音声 | 停止・一時停止・独立音量制御 |
| 1.4.7 小さな背景音または背景音なし | AAA | 収録済み音声のみ・発話中心 | 背景音なし/オフ/20dB低い |

- 1.4.2 は、音声が**自動再生**されるかどうかが焦点
- 1.4.7 は、音声内容の中で**発話と背景音のバランス**が焦点
- 自動再生しないポッドキャストでも、発話にBGMが重なるなら 1.4.7 の対象

> **参照**
> - [wcag/1-4-2-audio-control.md — 1.4.2 音声の制御](./1-4-2-audio-control.md)

## 1.2 系メディア要件との関係

- 1.2.1：収録済み音声のみには transcript が必要
- 1.4.7：同じ音声で、背景音が発話を妨げないこと
- transcript があっても、音声そのものの聞き取りやすさは別途必要

```html
<!-- 1.2.1 + 1.4.7 を両方考慮 -->
<figure>
  <figcaption>講義音声：アクセシビリティ入門</figcaption>
  <audio controls src="lecture-voice-only.mp3"></audio>
  <p><a href="#transcript">文字起こしを読む</a></p>
</figure>

<section id="transcript">
  <h2>文字起こし</h2>
  <p><strong>講師：</strong>今日はアクセシビリティの基本について説明します。</p>
</section>
```

> **参照**
> - [wcag/1-2-1-audio-only-and-video-only.md — 収録済み音声のみ](./1-2-1-audio-only-and-video-only.md)

## 実装・制作時のポイント

- 録音段階でノイズを抑える
- BGMを入れる場合は、発話中は十分に音量を下げる（ダッキング）
- 音楽や効果音は発話の前後に置く
- 音声編集時にメーターで発話と背景音の差を確認
- BGMなし版を書き出して提供する

```text
制作フロー例：
1. 発話トラックを録音
2. ノイズ除去
3. BGMを追加
4. 発話中のBGMを20dB以上下げる
5. BGMなし版も書き出す
6. transcript を作成
```

## よくある失敗例

- ポッドキャスト全編に大きなBGMが流れている
- インタビュー録音でカフェの環境音が発話と同程度に大きい
- 音声教材に効果音が頻繁に重なり、説明が聞き取りづらい
- BGMあり版しかなく、背景音をオフにできない
- transcript はあるが、音声ファイル自体の背景音が大きい

```html
<!-- 悪い例：背景音が大きい音声しか提供していない -->
<audio controls src="interview-cafe-noise.mp3"></audio>

<!-- 良い例：ノイズ低減版・背景音なし版を提供 -->
<audio controls src="interview-clean.mp3"></audio>
<p><a href="interview-transcript.html">文字起こし</a></p>
```

## テスト・確認方法

### 手動テスト

- 発話を聞き、BGM・環境音・効果音が内容理解を妨げないか確認
- 難聴のあるユーザーや、ノイズ環境を想定して確認
- 発話中に背景音が継続している箇所を重点確認
- BGMなし版・背景音オフ機能がある場合、同等の内容を聞けるか確認

### 音声編集ツールでの確認

- 発話トラックと背景音トラックを分けて、音量差を確認
- 背景音が発話より **20dB以上**低いか確認
- 波形だけでなく、ラウドネスメーター・ピークメーターも併用

```bash
# ffmpegで音声のラウドネスを確認する例（参考）
ffmpeg -i audio.mp3 -filter:a loudnorm=print_format=summary -f null -
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.7 は、発話中心の**収録済み音声のみコンテンツ**で背景音を小さくする達成基準（レベル AAA）
- 適合方法は、背景音なし・背景音をオフにできる・背景音を発話より20dB以上小さくする、のいずれか
- 音声CAPTCHA、音声ロゴ、歌唱・ラップ等の音楽表現は対象外
- transcript があっても、音声自体の聞き取りやすさは別要件
- 1.4.2 は自動再生の制御、1.4.7 は発話と背景音のバランス
- 制作段階でBGMなし版やダッキングを用意すると対応しやすい
