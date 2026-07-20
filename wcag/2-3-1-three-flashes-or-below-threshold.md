# 2.3.1 3回の閃光またはしきい値以下（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.3.1「3回の閃光またはしきい値以下」（Three Flashes or Below Threshold）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.3「発作と身体的反応」に属する
- Webページに、**任意の1秒間に3回を超える閃光**を含むものを置かないこと
- または、閃光が**一般閃光・赤色閃光のしきい値以下**であること
- 光過敏性発作を誘発しないことが目的

> **参照**
> - [WCAG 2.2 — Success Criterion 2.3.1 Three Flashes or Below Threshold](https://www.w3.org/TR/WCAG22/#three-flashes-or-below-threshold)
> - [Understanding SC 2.3.1 Three Flashes or Below Threshold](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold.html)

## 達成基準の原文（要約）

- Webページは、任意の1秒間に3回を超えて閃光するものを含まない
- または、その閃光が一般閃光および赤色閃光のしきい値以下である
- **注記**：この基準を満たさないコンテンツはページ全体の利用を妨げるため、適合要件5「非干渉」により、ページ上の**すべてのコンテンツ**が対象

> **参照**
> - [WCAG 2.2 — Success Criterion 2.3.1](https://www.w3.org/TR/WCAG22/#three-flashes-or-below-threshold)

## なぜ必要か

- 光過敏性てんかんなどの発作性障害のある人が、閃光で発作を起こすことがある
- 一定の周波数・大きさ・明るさの閃光がトリガーになる
- **赤色の閃光**には特に敏感な人が多い
- 片頭痛・めまい・吐き気などを誘発する場合もある
- 発作は一瞬で起きるため、「後から止める」では間に合わない

> **参照**
> - [Understanding SC 2.3.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold.html#benefits)

## 閃光（flash）とは

- 相対輝度の**対になる変化のペア**（増→減、または減→増）
- 十分な大きさと周波数帯で、一部の人に発作を起こしうるもの
- コンテンツ側（動画・アニメGIF・CSSアニメーションなど）が原因の閃光が対象
- ディスプレイやPCの描画速度によるちらつきは、作者の制御外

```text
閃光になりうる例：
  ストロボのような点滅動画
  連続する爆発・銃口の閃光のクローズアップ
  高速で明暗が切り替わるアニメGIF
  明るい背景と暗い背景を急速に切り替える演出

作者の制御外の例：
  ディスプレイのリフレッシュによるちらつき
  端末の描画性能によるフリッカー
```

> **参照**
> - [Understanding SC 2.3.1 — Key Terms: flash](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold.html#dfn-flash)

## 点滅（blinking）と閃光（flashing）の違い

| 用語 | 焦点 | 扱い |
|---|---|---|
| 点滅（blinking） | 注意を引くための視覚状態の切り替え | 短時間なら許容。止められること（2.2.2） |
| 閃光（flashing） | 発作を誘発しうる明暗・色の急変 | **1秒でも許容されない**。止める選択肢もない |

- 点滅は通常、1秒間に3回未満だが、3回以上なら閃光にも該当しうる
- 2.2.2「一時停止・停止・非表示」は点滅・動きの制御
- 2.3.1 は発作リスクのある閃光そのものを禁止／しきい値以下に制限

```text
2.2.2 の対象例：
  広告バナーのゆっくりした点滅（注意散漫）
  5秒以上続くスクロール文字

2.3.1 の対象例：
  1秒間に4回以上の明るいストロボ
  大きな面積の赤色の急激な点滅
```

> **参照**
> - [Understanding SC 2.3.1 — blinking vs flashing](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold.html#intent)
> - [wcag/2-2-2-pause-stop-hide.md — 2.2.2 一時停止・停止・非表示](./2-2-2-pause-stop-hide.md)

## 一般閃光・赤色閃光のしきい値

次のいずれかが真なら、しきい値以下（適合）とみなす。

1. **任意の1秒間に一般閃光が3回以下、かつ赤色閃光が3回以下**
2. **同時に閃光する領域の合計面積**が、典型的な視距離で10度視野の25%以下（約0.006ステラジアン）

### 一般閃光の定義

- 相対輝度の対になる変化で、最大相対輝度（1.0）の**10%以上**の変化
- 暗い方の相対輝度が**0.80未満**
- 「増→減」または「減→増」のペアが1回の閃光

### 赤色閃光の定義

- 飽和した赤を含む対になる遷移
- WCAG 2.2 での作業定義：一方の状態が `R/(R+G+B) ≥ 0.8` で、CIE 1976 UCS 色度図上の差が **0.2超**

### 面積の目安（CSSピクセル）

- 10度視野の目安：約 **341 × 256 CSSピクセル**（合計約 87,296 CSSピクセル）
- その25%以下なら面積条件を満たしうる
- フルスクリーン再生される動画は、**最大表示サイズ**で評価する
- ループする短いクリップ（GIFなど）は、ループ再生中として分析する

```text
しきい値を超えない例：
  1秒間に3回以下の閃光
  画面のごく一部だけが点滅（面積が十分小さい）
  細かい市松模様など、視覚的にバランスの取れた微細パターン（例外）

危険になりやすい例：
  フルスクリーンで明るい稲妻が1秒に4回以上
  大きな面積の赤色ストロボ
```

> **参照**
> - [Understanding SC 2.3.1 — general flash and red flash thresholds](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold.html#dfn-general-flash-and-red-flash-thresholds)

## 手法1：1秒間に3回を超えない（G19）

- 最もシンプルな適合方法
- コンテンツのどの要素も、任意の1秒間に3回を超えて閃光しない
- ツール分析なしで判定しやすい
- 稲妻・銃火・ストロボ演出は、編集で頻度を落とす

```text
良い例：
  映画の稲妻シーンを、任意の1秒間に3回以下になるよう編集

悪い例：
  ストロボ演出が1秒間に5回以上続く
```

```css
/* 悪い例：非常に速い明暗アニメーション（閃光になりうる） */
@keyframes strobe {
  0%, 49% { background: #000; }
  50%, 100% { background: #fff; }
}

.danger {
  animation: strobe 0.2s infinite; /* 1秒あたり約5回 */
}

/* 良い例：遅い点滅、または点滅しない */
.safe {
  animation: none;
}
```

> **参照**
> - [Technique G19: Ensuring that no component of the content flashes more than three times in any 1-second period](https://www.w3.org/WAI/WCAG22/Techniques/general/G19)

## 手法2：閃光領域を十分小さくする（G176）

- 閃光の頻度が3回超でも、**面積がしきい値以下**なら適合しうる
- 画面のごく一部だけが閃光する設計にする
- 銃口の閃光などを小さな領域に限定する例

```html
<!-- 概念例：閃光する映像を小さな領域に限定 -->
<video
  width="160"
  height="120"
  controls
  poster="muzzle-flash-poster.jpg"
>
  <source src="muzzle-flash-small.mp4" type="video/mp4">
</video>
<!-- フルスクリーンでも面積条件を満たすか別途確認が必要 -->
```

```text
良い例：
  マシンガンの銃口閃光を、しきい値未満の小さな領域に制限

注意：
  ユーザーがフルスクリーン再生できる場合は、フルスクリーン時も評価する
```

> **参照**
> - [Technique G176: Keeping the flashing area small enough](https://www.w3.org/WAI/WCAG22/Techniques/general/G176)
> - [Understanding SC 2.3.1 — muzzle flash example](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold.html#examples)

## 手法3：ツールでしきい値を確認する（G15）

- 一般閃光・赤色閃光のしきい値を、分析ツールで確認する
- 代表的なツール：[PEAT（Photosensitive Epilepsy Analysis Tool）](https://trace.umd.edu/peat/)
- 画面キャプチャした動画を解析する
- 1秒間に3回以下なら、ツールなしでも自動的に適合

```text
確認の流れ：
1. 疑わしい動画・アニメを最大表示サイズで再生する
2. 画面キャプチャする
3. PEAT などで一般閃光・赤色閃光をしきい値と比較する
4. 不合格なら頻度を落とすか、面積を小さくする
```

> **参照**
> - [Technique G15: Using a tool to ensure that content does not violate the general flash threshold or red flash threshold](https://www.w3.org/WAI/WCAG22/Techniques/general/G15)
> - [Trace Center Photosensitive Epilepsy Analysis Tool (PEAT)](https://trace.umd.edu/peat/)

## 非干渉（Conformance Requirement 5）

- 2.3.1 を満たさないコンテンツは、ページ全体の利用を妨げる
- そのため、**適合判定の対象外にしたコンテンツも含め**、ページ上のすべてが対象
- メインコンテンツだけ適合して、広告や装飾の閃光を放置してはならない

```text
対象になるもの：
  本文の動画
  埋め込み広告
  装飾アニメーション
  背景のCSSエフェクト
  自動再生のヒーロー動画
```

> **参照**
> - [Understanding Conformance — Requirement 5: Non-Interference](https://www.w3.org/WAI/WCAG22/Understanding/conformance#cc5)

## 2.3.2・2.2.2 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.3.1 3回の閃光またはしきい値以下 | 3回超の閃光を禁止、またはしきい値以下 | レベル A。面積・輝度の例外あり |
| 2.3.2 3回の閃光 | 任意の1秒間に3回超の閃光を一切禁止 | レベル AAA。しきい値例外なし |
| 2.2.2 一時停止・停止・非表示 | 動き・点滅・スクロールの制御 | 注意散漫への対策。発作とは別 |

- 2.3.2 は 2.3.1 より厳しく、**しきい値以下でも3回超は不可**
- 2.3.1 に適合しても、面積が大きければ 2.3.2 には不適合になりうる
- 点滅の制御は 2.2.2、発作防止は 2.3.1 / 2.3.2

```text
2.3.1 では適合しうるが 2.3.2 では不適合の例：
  1秒間に4回閃光するが、面積がしきい値以下

両方に適合しやすい例：
  そもそも閃光しない
  任意の1秒間に3回以下の閃光のみ
```

> **参照**
> - [WCAG 2.2 — Success Criterion 2.3.2 Three Flashes](https://www.w3.org/TR/WCAG22/#three-flashes)

## 実装時の注意点

- 動画・GIF・CSSアニメ・Canvas・WebGL の明暗変化を確認する
- `prefers-reduced-motion` は有用だが、2.3.1 の代替にはならない（発作は止められない）
- HDR や sRGB 以外の色空間では、輝度の定義が異なる場合がある
- ズームや画面に近い利用者は、しきい値を通過しても影響を受けうる
- 疑わしいコンテンツは、最初から閃光を避ける設計が最も安全

```css
/* 補助：動きを減らす設定への配慮（2.3.1 の代替ではない） */
@media (prefers-reduced-motion: reduce) {
  .hero-flash {
    animation: none;
  }
}
```

```html
<!-- 良い例：危険な閃光を避け、警告も併用 -->
<video controls>
  <source src="scene-without-strobe.mp4" type="video/mp4">
</video>
<p>この動画にストロボ効果はありません。</p>
```

## テスト・確認方法

### 手動テスト

- ページ内の動画・アニメGIF・CSSアニメ・広告を洗い出す
- 任意の1秒間に3回を超える明暗・色の急変がないか目視する
- 3回以下なら適合（G19）
- 3回超なら、面積が十分小さいか（G176）、ツールでしきい値を確認（G15）
- フルスクリーン再生時も確認する
- ループGIFはループ中として確認する

```text
チェックリスト：
1. 閃光・高速点滅するコンテンツがあるか
2. 任意の1秒間に3回を超えるか
3. 超える場合、面積はしきい値以下か
4. 赤色の強い閃光はないか
5. 広告・装飾も含めてページ全体を確認したか
```

### ツールでの確認

- PEAT などで画面キャプチャを解析する
- 1秒間に3回以下ならツールは不要（自動的に適合）

```bash
# アニメーション・点滅まわりの実装を探す
rg "animation|@keyframes|strobe|flash|blink" --glob "*.css" --glob "*.scss" --glob "*.js" --glob "*.tsx"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)
> - [Trace Center PEAT](https://trace.umd.edu/peat/)

## まとめ

- 2.3.1 は、任意の1秒間に**3回を超える閃光**を置かないこと、または一般閃光・赤色閃光の**しきい値以下**にすることを求める達成基準（レベル A）
- 光過敏性発作の防止が目的。発作は一瞬で起きるため、「後から止める」では不十分
- 点滅（2.2.2）とは異なり、閃光は短時間でも許容されない
- 主な手法は、頻度を3回以下にする（G19）、面積を小さくする（G176）、ツールで確認する（G15）
- ページ上のすべてのコンテンツが対象（非干渉）
- より厳しい 2.3.2（AAA）では、しきい値例外なしで3回超を禁止する
