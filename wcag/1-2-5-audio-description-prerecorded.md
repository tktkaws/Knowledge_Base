# 1.2.5 音声解説（収録済）（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.2.5「音声解説（収録済）」（Audio Description (Prerecorded)）
- レベル **AA**（標準的な適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.2「時間依存メディア」に属する
- **収録済み**の**同期メディア**（映像+音声）の**すべての映像コンテンツ**に**音声解説**を提供
- 1.2.3（A）ではテキスト代替でも可だったが、AA では**音声解説が必須**

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.5 Audio Description (Prerecorded)](https://www.w3.org/TR/WCAG22/#audio-description-prerecorded)
> - [Understanding SC 1.2.5 Audio Description (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-prerecorded.html)

## 達成基準の原文（要約）

- 収録済みの同期メディアに含まれる**すべての映像コンテンツ**に**音声解説**を提供すること
- 1.2.3 と異なり、テキスト代替だけでは適合しない（AA では音声解説が必要）

> **参照**
> - [WCAG 2.2 — Success Criterion 1.2.5](https://www.w3.org/TR/WCAG22/#audio-description-prerecorded)

## 音声解説（audio description）とは

- メインの音声トラックだけでは理解できない**重要な視覚的詳細**を、ナレーションで補足する手法
- 台詞・ナレーションの**既存の休止部分**に解説を挿入（標準的な音声解説）
- 説明対象
  - 登場人物の動作・表情
  - 場面転換
  - 画面上のテキスト・UI操作
  - 映像のみで伝わる情報
- 別名：video description、descriptive narration

```text
<!-- 音声解説の記述例 -->
解説者：タイトル画面。「Teaching Evolution Case Studies. Bonnie Chen.」
        教師が長いくちばしの鳥の写真を見せている。

Bonnie Chen：「これらの写真はすべてエバーグレーズで撮影されました。」

解説者：教師が各生徒に2本の薄い木の棒を渡す。

Bonnie Chen：「今日は、こういうくちばしを持つ水鳥の仲間を演じてもらいます。」

解説者：教師が2本の棒を口元に当て、くちばしの形を作る。
```

> **参照**
> - [Understanding SC 1.2.5 — Key Terms: audio description](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-prerecorded.html#dfn-audio-description)
> - [W3C WAI — Description of Visual Information](https://www.w3.org/WAI/media/av/description/)

## なぜ必要か

- 視覚障害者・ロービジョンのユーザーが、動画の**映像情報**にアクセスするため
- 1.2.2 キャプションは**音声**をテキスト化するが、**映像**はカバーしない
- 1.2.3（A）のテキスト代替はレベル A では足りるが、**AA 適合には音声解説が必須**
- 視覚情報の理解が難しい認知障害のあるユーザーにも有益

> **参照**
> - [Understanding SC 1.2.5 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-prerecorded.html#benefits)

## 1.2.3 との関係

- 1.2.3 / 1.2.5 / 1.2.8 は内容が重複するが、適合レベルごとに要件が異なる

| レベル | 達成基準 | 要件 |
|---|---|---|
| A | 1.2.3 | 音声解説 **または** メディアの代替（テキスト） |
| AA | 1.2.5 | **音声解説**が必須 |
| AAA | 1.2.8 | 拡張テキスト記述（条件付き追加要件） |

### 適合パターン

- **1.2.3 で音声解説を選んだ場合** → 1.2.5 も**自動的に満たす**（追加作業不要）
- **1.2.3 でテキスト代替のみを選んだ場合** → 1.2.5 達成のために**音声解説を追加**する必要がある

```html
<!-- 1.2.3（A）のみ満たす：テキスト代替のみ → 1.2.5（AA）は未達 -->
<video controls src="tutorial.mp4" aria-label="操作チュートリアル">
  <track kind="captions" src="tutorial.vtt" srclang="ja" label="日本語">
</video>
<a href="#full-transcript">完全な文字起こし（映像描写含む）</a>
<!-- → AA 適合には音声解説トラックの追加が必要 -->

<!-- 1.2.3 + 1.2.5 を両方満たす -->
<video controls aria-label="操作チュートリアル">
  <source src="tutorial.mp4" type="video/mp4">
  <track kind="captions" src="tutorial.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="tutorial-ad.vtt" srclang="ja" label="音声解説">
</video>
```

> **参照**
> - [wcag/1-2-3-audio-description-or-media-alternative.md — 1.2.3 との関係](./1-2-3-audio-description-or-media-alternative.md)
> - [Understanding SC 1.2.5 — Overlap with 1.2.3 and 1.2.8](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-prerecorded.html#intent)

## 追加の音声解説が不要な場合

- 映像トラックの**すべての重要な情報**が、すでに音声トラックで伝わっている場合
- 典型例：**トーキングヘッド**（話者の顔だけが映るインタビュー・講義）
- Advisory Technique G203：静的テキスト代替で十分な場合がある

```html
<!-- 音声解説不要：内容はすべて音声で伝わる -->
<video controls aria-label="CEOインタビュー">
  <source src="interview.mp4" type="video/mp4">
  <track kind="captions" src="interview.vtt" srclang="ja" label="日本語">
</video>
<!-- 映像は話者の顔のみ → 1.2.5 の追加音声解説は不要 -->

<!-- 音声解説必要：映像に追加情報がある -->
<video controls aria-label="ソフトウェア操作デモ">
  <source src="demo.mp4" type="video/mp4">
  <track kind="captions" src="demo.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="demo-ad.vtt" srclang="ja" label="音声解説">
</video>
<!-- 画面操作・UI・マウスの動き → 音声解説必須 -->
```

> **参照**
> - [Understanding SC 1.2.5 — Note on audio description not necessary](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-prerecorded.html#intent)
> - [Technique G203: Static text alternative for talking head video](https://www.w3.org/WAI/WCAG22/Techniques/general/G203)

## 標準音声解説と拡張音声解説

| 種類 | 挿入タイミング | 対応する達成基準 |
|---|---|---|
| 標準音声解説 | 台詞の**既存の休止部分**に挿入 | 1.2.5（AA） |
| 拡張音声解説（extended） | 動画を**一時停止**して解説を挿入 | 1.2.7（AAA） |

- 1.2.5 では**標準音声解説**が要件
- 台詞の休止が短すぎて標準音声解説では情報が伝わらない場合、Technique G8（拡張音声解説）も 1.2.5 の sufficient technique として認められる
- AAA の 1.2.7 では拡張音声解説がより明確に要求される

> **参照**
> - [Technique G8: Providing a movie with extended audio descriptions](https://www.w3.org/WAI/WCAG22/Techniques/general/G8)
> - [WCAG 2.2 — Success Criterion 1.2.7 Extended Audio Description (Prerecorded)](https://www.w3.org/TR/WCAG22/#extended-audio-description-prerecorded)

## 実装方法（Sufficient Techniques）

### 方法1：`<track kind="descriptions">` + WebVTT（H96）

- Advisory Technique だが、Web では最も一般的な実装
- 音声解説を WebVTT 形式で提供

```html
<!-- 悪い例：キャプションのみ -->
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
<p><small>プレイヤーの設定から音声解説トラックを選択できます。</small></p>
```

```text
<!-- training-ad.vtt の内容例 -->
WEBVTT

00:00:05.000 --> 00:00:08.000
タイトル画面。「新技術トレーニング」と表示。

00:00:12.000 --> 00:00:15.000
講師がスライドを指差し、グラフが表示される。

00:00:22.000 --> 00:00:25.000
画面共有が始まり、ソフトウェアのダッシュボードが映る。
```

### 方法2：音声解説版を別ファイルで提供（G173）

- 音声解説を組み込んだ**別バージョン**の動画ファイルを用意
- ナレーションを元の音声に焼き込む（G226）方法も含む

```html
<!-- 良い例：通常版 + 音声解説版の両方を提供 -->
<video controls aria-label="操作デモ動画">
  <source src="demo.mp4" type="video/mp4">
  <track kind="captions" src="demo.vtt" srclang="ja" label="日本語">
</video>
<p><a href="demo-with-ad.mp4">音声解説付きバージョンを視聴</a></p>
```

### 方法3：ナレーションを音声トラックに組み込む（G226）

- 制作段階で解説ナレーションを**元の音声に統合**
- 別トラック不要。すべての視聴者が同じ音声を聴く
- 教育コンテンツ・解説動画でよく使われる手法

```html
<!-- 制作時にナレーションを組み込んだ動画 -->
<video controls aria-label="エンジンの仕組み解説">
  <source src="engine-explained.mp4" type="video/mp4">
  <track kind="captions" src="engine.vtt" srclang="ja" label="日本語">
</video>
<!-- ナレーションが「ピストンが上下に動き...」と映像を説明済み -->
```

> **参照**
> - [Technique G173: Providing a version of a movie with audio descriptions](https://www.w3.org/WAI/WCAG22/Techniques/general/G173)
> - [Technique G226: Providing audio descriptions by incorporating narration in the soundtrack](https://www.w3.org/WAI/WCAG22/Techniques/general/G226)
> - [Technique H96: Using the track element to provide audio descriptions](https://www.w3.org/WAI/WCAG22/Techniques/html/H96)

## 音声解説の書き方

### 記述の原則

- **簡潔に**：休止部分に収まる長さ。冗長な描写は避ける
- **客観的に**：解釈ではなく、見えている事実を述べる
- **重要な視覚情報に限定**：装飾的な背景描写は省略
- **既存の音声と重複しない**：台詞や効果音で伝わっている情報は繰り返さない

### 記述例：操作チュートリアル

```text
00:00:05 — ログイン画面が表示される。
00:00:08 — カーソルがメールアドレス欄をクリックする。
00:00:12 — パスワードを入力し、「ログイン」ボタンを押す。
00:00:18 — ダッシュボード画面に遷移。左サイドバーに「分析」メニュー。
```

### 記述例：ニュース映像

```text
00:00:03 — 国会議事堂の外観。雨が降っている。
00:00:08 — 記者会見場。首相が壇上に立つ。
00:00:15 — 首相の後ろに「経済対策」のバナーが掲げられている。
00:00:22 — 首相が書類を手に取り、読み上げ始める。
```

### 悪い例

```text
<!-- 悪い例：主観的・冗長 -->
00:00:05 — 美しい青空の下、壮大な国会議事堂がそびえ立っている。

<!-- 悪い例：音声と重複 -->
00:00:15 — 首相が「経済対策についてお話しします」と言う。
<!-- → 台詞そのものは音声で伝わる。バナーの存在を述べるべき -->

<!-- 良い例 -->
00:00:15 — 首相の後ろに「経済対策」のバナーが掲げられている。
```

> **参照**
> - [Standard Techniques in Audio Description](http://joeclark.org/access/description/ad-principles.html)
> - [W3C WAI — Description of Visual Information](https://www.w3.org/WAI/media/av/description/)

## 代表的な失敗例（Failures）

### F113：台詞の休止を活用していない

- 台詞の**休止部分**が存在するのに、音声解説を挿入していない
- 重要な視覚情報が音声解説なしでは理解できない状態

```html
<!-- F113：休止部分があるのに音声解説なし -->
<!-- 動画：講師が黙ってスライドを操作 → 休止があるが解説なし -->
<video controls src="lecture.mp4">
  <track kind="captions" src="lecture.vtt" srclang="ja" label="日本語">
</video>
<!-- → 視覚情報（スライド内容・操作）が伝わらない -->

<!-- 良い例：休止部分に音声解説を挿入 -->
<video controls src="lecture.mp4">
  <track kind="captions" src="lecture.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="lecture-ad.vtt" srclang="ja" label="音声解説">
</video>
```

### その他のよくある問題

- 1.2.3 でテキスト代替のみ提供し、**AA 適合を想定していない**
- 音声解説トラックがあるが、**重要な視覚情報が欠落**している
- `<track kind="descriptions">` を設定したが、**プレイヤーが対応しておらず**再生できない

> **参照**
> - [Failure F113: Not using available pauses in dialogue to provide audio descriptions](https://www.w3.org/WAI/WCAG22/Techniques/failures/F113)

## 1.2.2 キャプションとの関係

- 1.2.2（キャプション）と 1.2.5（音声解説）は**独立した要件**
- 音声付き動画は**両方**を満たす必要がある（AA 適合時）

| 達成基準 | 対象 | 提供するもの | レベル |
|---|---|---|---|
| 1.2.2 キャプション（収録済） | 音声トラック | 同期字幕 | A |
| 1.2.5 音声解説（収録済） | 映像トラック | 音声解説 | AA |

```html
<!-- AA 適合に必要な構成 -->
<video controls aria-label="製品デモ動画">
  <source src="demo.mp4" type="video/mp4">
  <track kind="captions" src="demo-captions.vtt" srclang="ja" label="日本語字幕">
  <track kind="descriptions" src="demo-ad.vtt" srclang="ja" label="音声解説">
</video>
```

- 音声解説トラック自体にキャプション化は**不要**（映像で既に提示されている情報のため）

> **参照**
> - [wcag/1-2-2-captions-prerecorded.md — 1.2.2 キャプション](./1-2-2-captions-prerecorded.md)

## プレイヤー対応の注意点

- `<track kind="descriptions">` のブラウザ・プレイヤー対応は**キャプションより限定的**
- 主要ブラウザは対応が進んでいるが、古いプレイヤーでは再生できない場合がある
- **確実性を求める場合**は音声解説版の別ファイル（G173）も併用
- ユーザーに音声解説の**有効化方法**を案内する

```html
<video controls aria-label="トレーニング動画">
  <source src="training.mp4" type="video/mp4">
  <track kind="captions" src="training.vtt" srclang="ja" label="日本語">
  <track kind="descriptions" src="training-ad.vtt" srclang="ja" label="音声解説">
</video>
<p><small>
  音声解説：プレイヤーの設定（⚙）→「音声解説」を選択してください。
  うまく再生できない場合は
  <a href="training-with-ad.mp4">音声解説付き版</a>をご利用ください。
</small></p>
```

> **参照**
> - [MDN — track element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track)

## テスト・確認方法

### 手動テスト

- **音声のみ**（または目を閉じて）動画を視聴し、映像の内容が理解できるか
- 台詞の**休止部分**に音声解説が挿入されているか（Failure F113 の確認）
- 重要な視覚情報（UI操作、テロップ、登場人物の動作）が音声解説に含まれるか
- 音声解説トラックの**オン/オフ**が可能か
- 1.2.3 でテキスト代替のみの場合、音声解説が**追加されている**か（AA 確認）

```bash
# video 要素の descriptions トラックを確認（開発者ツール Console）
document.querySelectorAll('video').forEach(v => {
  const desc = [...v.textTracks].filter(t => t.kind === 'descriptions');
  console.log(v.currentSrc, desc.map(t => t.label));
});
```

### ACT Rules

- [Video element visual content has accessible alternative](https://www.w3.org/WAI/standards-guidelines/act/rules/c5a4ea/proposed/)
- [Video element visual content has strict accessible alternative](https://www.w3.org/WAI/standards-guidelines/act/rules/1ea59c/proposed/)

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.2.5 は**収録済み同期メディア**の**映像情報**に**音声解説**を要求（レベル AA）
- 1.2.3（A）でテキスト代替のみ選んだ場合、AA 達成には**音声解説の追加**が必要
- 1.2.3 で音声解説を選んでいれば、1.2.5 も**自動的に満たす**
- トーキングヘッドなど音声ですべて伝わる場合は追加不要
- `<track kind="descriptions">` + WebVTT、別ファイル、ナレーション組み込みが主な実装
- 1.2.2（キャプション）とは別要件。AA 適合には両方必要
- 休止部分を活用しないと Failure F113
