# 2.3.2 3回の閃光（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.3.2「3回の閃光」（Three Flashes）
- レベル **AAA**（最高の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.3「発作と身体的反応」に属する
- Webページに、**任意の1秒間に3回を超える閃光**を含むものを置かないこと
- 2.3.1 と異なり、**明るさ・面積のしきい値例外がない**
- 発作リスクをさらに下げることが目的

> **参照**
> - [WCAG 2.2 — Success Criterion 2.3.2 Three Flashes](https://www.w3.org/TR/WCAG22/#three-flashes)
> - [Understanding SC 2.3.2 Three Flashes](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes.html)

## 達成基準の原文（要約）

- Webページは、任意の1秒間に3回を超えて閃光するものを含まない

> **参照**
> - [WCAG 2.2 — Success Criterion 2.3.2](https://www.w3.org/TR/WCAG22/#three-flashes)

## なぜ必要か

- 2.3.1 でも国際的な放送基準などに基づくしきい値でリスクを下げられる
- それでも、極めて敏感な人の発作を完全には防げない
- 1秒間に3回超の閃光を**画面上のどこにも**置かないことで、リスクをさらに減らす
- ズームや高コントラスト設定で、小さな閃光が大きく見える場合がある
- そのため、1ピクセルの閃光でも、この基準では不適合になりうる

> **参照**
> - [Understanding SC 2.3.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes.html#intent)
> - [Understanding SC 2.3.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes.html#benefits)

## 2.3.1 との違い

| 項目 | 2.3.1（A） | 2.3.2（AAA） |
|---|---|---|
| 頻度制限 | 1秒間に3回超を原則禁止 | 1秒間に3回超を**全面禁止** |
| しきい値例外 | 一般閃光・赤色閃光のしきい値以下なら許容 | **例外なし** |
| 小さな面積 | 面積が十分小さければ適合しうる | 面積に関係なく不適合 |
| 暗い／弱い閃光 | しきい値以下なら適合しうる | 頻度が3回超なら不適合 |
| 主な手法 | G19 / G176 / G15 | **G19 のみ** |

```text
2.3.1 では適合しうるが 2.3.2 では不適合の例：
  1秒間に4回閃光するが、面積がしきい値以下
  小さな銃口の閃光が高速で繰り返される
  1ピクセルでも、1秒間に3回超点滅する

両方に適合しやすい例：
  そもそも閃光しない
  任意の1秒間に3回以下の閃光のみ
```

> **参照**
> - [wcag/2-3-1-three-flashes-or-below-threshold.md — 2.3.1 3回の閃光またはしきい値以下](./2-3-1-three-flashes-or-below-threshold.md)

## 点滅と閃光の関係

- **点滅（blinking）**：注意を引くための視覚状態の切り替え。短時間なら許容され、止められること（2.2.2）
- **閃光（flashing）**：発作を誘発しうる明暗・色の急変。止められないほど速いため、**許容されない**
- 点滅は通常1秒間に3回未満だが、3回以上なら閃光にも該当しうる
- 2.3.2 では、しきい値を問わず頻度だけで判定する

```text
2.2.2 の対象例：
  広告バナーのゆっくりした点滅

2.3.2 の対象例：
  1秒間に4回以上の明暗変化（面積・明るさは問わない）
```

> **参照**
> - [Understanding SC 2.3.2 — blinking vs flashing](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes.html#intent)
> - [wcag/2-2-2-pause-stop-hide.md — 2.2.2 一時停止・停止・非表示](./2-2-2-pause-stop-hide.md)

## 手法：1秒間に3回を超えない（G19）

- 2.3.2 の十分な手法は **G19 のみ**
- コンテンツのどの要素も、任意の1秒間に3回を超えて閃光しない
- 明るさ・面積・色を問わず、頻度を3回以下にする
- 稲妻・ストロボ・銃火などの演出は、編集で頻度を落とす

```text
良い例：
  映画の稲妻シーンを、任意の1秒間に3回以下になるよう編集

悪い例：
  小さな領域でも、1秒間に5回以上のストロボが続く
  「面積が小さいから大丈夫」と判断して高速閃光を残す
```

```css
/* 悪い例：1秒あたり約5回の明暗変化（2.3.2 では面積を問わず不適合） */
@keyframes strobe {
  0%, 49% { opacity: 0; }
  50%, 100% { opacity: 1; }
}

.danger {
  width: 4px;
  height: 4px;
  background: #fff;
  animation: strobe 0.2s infinite;
}

/* 良い例：閃光しない、または十分遅い変化 */
.safe {
  animation: none;
}
```

```html
<!-- 良い例：稲妻を編集して頻度を落とす -->
<video controls>
  <source src="lightning-max-3-per-second.mp4" type="video/mp4">
</video>
<p>このシーンの稲妻は、任意の1秒間に3回以下です。</p>
```

> **参照**
> - [Technique G19: Ensuring that no component of the content flashes more than three times in any 1-second period](https://www.w3.org/WAI/WCAG22/Techniques/general/G19)
> - [Understanding SC 2.3.2 — lightning example](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes.html#examples)

## 2.3.1 のしきい値手法は使えない

- G176（閃光領域を小さくする）は **2.3.2 では十分ではない**
- G15（しきい値ツール確認）も、頻度が3回超なら 2.3.2 では不合格
- 「暗いから」「赤い以外だから」「1ピクセルだから」は理由にならない
- ズームや拡大鏡で大きく見える可能性を考慮している

```text
2.3.1 向けの判断（2.3.2 では不可）：
  面積が 341×256 CSSピクセルの 25% 以下 → 2.3.1 では適合しうる
  一般閃光・赤色閃光のしきい値以下 → 2.3.1 では適合しうる

2.3.2 の判断：
  任意の1秒間に3回超があるか？ → あるなら不適合
```

> **参照**
> - [Understanding SC 2.3.1 — Techniques G176 / G15](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold.html#techniques)

## 実装時の注意点

- 動画・GIF・CSSアニメ・Canvas・WebGL・広告の明暗変化を確認する
- フルスクリーン再生時も、頻度が3回超になっていないか確認する
- ループGIFはループ中として確認する
- `prefers-reduced-motion` は有用だが、2.3.2 の代替にはならない
- 疑わしい演出は、最初から閃光を避けるか、編集で3回以下にする

```css
/* 補助：動きを減らす設定への配慮（2.3.2 の代替ではない） */
@media (prefers-reduced-motion: reduce) {
  .hero-effect {
    animation: none;
  }
}
```

```javascript
// 悪い例：小さなパーティクルでも高速点滅
function blinkParticle(el) {
  setInterval(() => {
    el.hidden = !el.hidden;
  }, 100); // 1秒あたり約5回
}

// 良い例：閃光にならないゆっくりした変化、または点滅しない
function fadeInOnce(el) {
  el.style.transition = "opacity 1s";
  el.style.opacity = "1";
}
```

## テスト・確認方法

### 手動テスト

- ページ内の動画・アニメGIF・CSSアニメ・広告・Canvas を洗い出す
- 任意の1秒間に3回を超える明暗・色の急変がないか目視する
- 面積や明るさは問わない（小さな点でも対象）
- 3回以下なら適合（G19）
- 3回超なら、編集・削除・速度低下が必要

```text
チェックリスト：
1. 閃光・高速点滅するコンテンツがあるか
2. 任意の1秒間に3回を超えるか
3. 「面積が小さい」「暗い」で見逃していないか
4. フルスクリーン・ループ再生時も確認したか
5. 広告・装飾も含めてページ全体を確認したか
```

### ツールでの確認

- 1秒間に3回以下なら、ツールなしでも適合
- 疑わしい場合は PEAT などで画面キャプチャを解析する
- ただし 2.3.2 は頻度判定が本質で、しきい値合格だけでは不十分

```bash
# アニメーション・点滅まわりの実装を探す
rg "animation|@keyframes|strobe|flash|blink|setInterval" --glob "*.css" --glob "*.scss" --glob "*.js" --glob "*.tsx"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)
> - [Trace Center PEAT](https://trace.umd.edu/peat/)

## まとめ

- 2.3.2 は、任意の1秒間に**3回を超える閃光を一切置かない**ことを求める達成基準（レベル AAA）
- 2.3.1 と異なり、明るさ・面積のしきい値例外はない
- 1ピクセルの閃光でも、頻度が3回超なら不適合になりうる
- 十分な手法は G19（頻度を3回以下にする）のみ
- 発作は一瞬で起きるため、「後から止める」や「小さいから大丈夫」では不十分
- 2.3.1（A）と 2.2.2（点滅の制御）と併せて理解する
