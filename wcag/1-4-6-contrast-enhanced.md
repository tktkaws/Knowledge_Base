# 1.4.6 コントラスト（高度）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.6「コントラスト（高度）」（Contrast (Enhanced)）
- レベル **AAA**（最高の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- テキストおよび文字画像の視覚的提示に、1.4.3 より高い**コントラスト比**を確保
- 通常テキストは **7:1 以上**、大きなテキストは **4.5:1 以上**

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.6 Contrast (Enhanced)](https://www.w3.org/TR/WCAG22/#contrast-enhanced)
> - [Understanding SC 1.4.6 Contrast (Enhanced)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html)

## 達成基準の原文（要約）

- テキストおよび文字画像は、背景とのコントラスト比が **7:1 以上**
- 例外
  - **大きなテキスト**：4.5:1 以上
  - **付随的**：非アクティブ UI、純粋な装飾、誰にも見えないテキスト、重要な他の視覚コンテンツを含む画像の一部
  - **ロゴタイプ**：ロゴ・ブランド名の一部であるテキスト

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.6](https://www.w3.org/TR/WCAG22/#contrast-enhanced)

## 1.4.3 との違い

| 達成基準 | レベル | 通常テキスト | 大きなテキスト |
|---|---|---:|---:|
| 1.4.3 コントラスト（最低限） | AA | 4.5:1 以上 | 3:1 以上 |
| 1.4.6 コントラスト（高度） | AAA | **7:1 以上** | **4.5:1 以上** |

- 1.4.6 は 1.4.3 の**上位基準**
- 通常テキストで 7:1、大きなテキストで 4.5:1 を満たせば、1.4.3 も満たす
- 例外条件・対象範囲は 1.4.3 とほぼ同じ

> **参照**
> - [wcag/1-4-3-contrast-minimum.md — 1.4.3 コントラスト（最低限）](./1-4-3-contrast-minimum.md)

## なぜ必要か

- 1.4.3 の最低限コントラストでは読みにくいユーザーに対応するため
- より強いコントラストは、ロービジョン・高齢者・色覚特性のあるユーザーの読みやすさを高める
- 7:1 は、視力 20/80 程度のコントラスト感度低下を補う意図で選ばれた比率
- 支援技術を使わない、または使えない状況でも本文を読みやすくする

> **参照**
> - [Understanding SC 1.4.6 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html#intent)
> - [Understanding SC 1.4.6 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html#benefits)

## コントラスト比の基本

- コントラスト比は、前景色（文字色）と背景色の**相対輝度**の比率
- 範囲は **1:1**（同じ色）から **21:1**（黒と白）
- 計算式：`(L1 + 0.05) / (L2 + 0.05)`
  - `L1`：明るい方の相対輝度
  - `L2`：暗い方の相対輝度

```text
白 #ffffff と黒 #000000 → 21:1
白 #ffffff と #595959 → 約 7:1
白 #ffffff と #767676 → 約 4.5:1（AA通常テキストは可、AAA通常テキストは不足）
```

> **参照**
> - [Understanding SC 1.4.6 — contrast ratio](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html#dfn-contrast-ratio)
> - [a11y/color-contrast.md — カラーコントラスト比とは](../a11y/color-contrast.md)

## 必要なコントラスト比

### 通常テキスト：7:1 以上

```html
<!-- 悪い例：AAは満たすがAAAは不足（約4.5:1） -->
<p style="color: #767676; background-color: #ffffff;">
  このテキストは1.4.6では不足
</p>

<!-- 良い例：AAAの通常テキスト基準を満たす（約7:1） -->
<p style="color: #595959; background-color: #ffffff;">
  このテキストは1.4.6を満たす
</p>

<!-- より良い例：さらに余裕のあるコントラスト -->
<p style="color: #333333; background-color: #ffffff;">
  このテキストは十分に読みやすい
</p>
```

> **参照**
> - [Technique G17: Contrast ratio of at least 7:1](https://www.w3.org/WAI/WCAG22/Techniques/general/G17)

### 大きなテキスト：4.5:1 以上

- 18pt（約 **24px**）以上
- または 14pt 太字（約 **18.66px**）以上
- CJK（日本語・中国語・韓国語）では同等サイズの文字

```html
<!-- 悪い例：大きなテキストでもAAAでは不足（約3:1） -->
<h1 style="font-size: 32px; color: #949494; background-color: #ffffff;">
  キャンペーン見出し
</h1>

<!-- 良い例：大きなテキストのAAA基準を満たす（約4.5:1） -->
<h1 style="font-size: 32px; color: #767676; background-color: #ffffff;">
  キャンペーン見出し
</h1>
```

> **参照**
> - [Technique G18: Contrast ratio of at least 4.5:1](https://www.w3.org/WAI/WCAG22/Techniques/general/G18)

## 文字画像も対象

- 文字画像にも 1.4.6 のコントラスト比が適用される
- 通常サイズの文字画像は 7:1 以上
- 大きな文字画像は 4.5:1 以上
- 文字画像は色変更・拡大に弱いため、可能なら実テキストにする

```html
<!-- 悪い例：文字画像のコントラストがAAA基準を満たさない -->
<img src="campaign-low-contrast.png" alt="夏のキャンペーン">

<!-- 良い例：実テキスト + CSSで高コントラスト -->
<section class="campaign">
  <h2>夏のキャンペーン</h2>
  <p>最大50%OFF。8月31日まで。</p>
</section>
```

```css
.campaign {
  color: #1a1a1a;
  background-color: #ffffff;
}
```

> **参照**
> - [wcag/1-4-5-images-of-text.md — 1.4.5 文字画像](./1-4-5-images-of-text.md)

## 例外：付随的なテキスト

### 非アクティブ UI

- disabled 状態など、現在操作できない UI コンポーネント内のテキストは対象外
- ただし、ユーザビリティ上は読める方が望ましい

```html
<!-- 例外：非アクティブなボタン -->
<button disabled style="color: #cccccc; background-color: #f5f5f5;">
  送信
</button>
```

### 純粋な装飾

- 情報・機能を持たない装飾文字は対象外
- 文字を並べ替えたり置き換えたりしても意味が変わらないもの

```html
<!-- 例外：装飾的な背景文字 -->
<div aria-hidden="true" class="decorative-words">
  SALE SALE SALE
</div>
```

### 写真内の偶発的な文字

- 画像の主目的ではない看板・服の文字・背景のポスター等は対象外になりうる
- その文字を読ませる目的の画像なら対象

```html
<!-- 例外になりうる：街の写真の一部に看板が写っている -->
<img src="street-photo.jpg" alt="夕方の商店街の写真">

<!-- 対象：看板の文字を読ませる目的の画像 -->
<img src="store-hours.jpg" alt="営業時間：10時から19時">
```

> **参照**
> - [Understanding SC 1.4.6 — Incidental](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html#intent)

## 例外：ロゴタイプ

- ロゴやブランド名の一部であるテキストは対象外
- ただし、ロゴがリンク・ボタンなど UI コンポーネントとして使われる場合は、同等機能を持つ高コントラストな代替も検討
- ブランドガイドラインが許すなら、高コントラスト版ロゴを用意するとよい

```html
<!-- ロゴタイプ自体は例外 -->
<a href="/" aria-label="ホーム">
  <img src="brand-logo.svg" alt="Example Corp">
</a>
```

> **参照**
> - [Understanding SC 1.4.6 — Logotypes](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html#intent)

## 背景画像・グラデーション上のテキスト

- 背景が画像・動画・グラデーションの場合、文字が重なる範囲で 7:1 / 4.5:1 を満たす必要
- 背景が変動する場合は、オーバーレイ・背景色・テキスト周囲の塗りで安定させる
- AAでは足りていても、AAAでは不足するケースが多い

```css
/* 悪い例：背景画像次第でAAAコントラストが不足 */
.hero-bad {
  color: #ffffff;
  background-image: url("bright-photo.jpg");
}

/* 良い例：濃いオーバーレイで7:1以上を確保しやすくする */
.hero-good {
  color: #ffffff;
  background-image:
    linear-gradient(rgba(0, 0, 0, 0.78), rgba(0, 0, 0, 0.78)),
    url("bright-photo.jpg");
}
```

> **参照**
> - [Failure F83: Background images that do not provide sufficient contrast](https://www.w3.org/WAI/WCAG22/Techniques/failures/F83)

## 前景色・背景色をセットで指定する（F24）

- 文字色だけ、または背景色だけを指定すると、ユーザー設定や親要素次第でコントラストを評価できない
- 1.4.6 でも Failure F24 の対象
- コンポーネント単位で前景色・背景色をセットで設計する

```css
/* 悪い例：文字色だけ指定 */
.message-bad {
  color: #595959;
  /* background-color 未指定 */
}

/* 悪い例：背景色だけ指定 */
.panel-bad {
  background-color: #1a1a1a;
  /* color 未指定 */
}

/* 良い例：前景色と背景色をセットで指定 */
.message-good {
  color: #333333;
  background-color: #ffffff;
}
```

> **参照**
> - [Failure F24: Specifying foreground colors without specifying background colors or vice versa](https://www.w3.org/WAI/WCAG22/Techniques/failures/F24)

## プレースホルダー・ホバー・フォーカス時のテキスト

- `placeholder` もテキストなので対象
- ツールチップ、ホバー表示、フォーカス時に表示されるテキストも対象
- AAAでは薄い placeholder が不足しやすい

```css
/* 悪い例：AAAでは不足しやすい薄いplaceholder */
input::placeholder {
  color: #767676; /* 白背景で約4.5:1。通常テキストAAAでは不足 */
}

/* 良い例：AAA基準を満たす濃さ */
input::placeholder {
  color: #595959; /* 白背景で約7:1 */
}
```

> **参照**
> - [Understanding SC 1.4.6 — placeholder and hover/focus text](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html#intent)

## 閾値は丸めない

- 7:1 と 4.5:1 は**しきい値**
- 計算値を丸めて適合扱いにしない
- 例：`6.999:1` は **7:1 を満たさない**

```text
7.00:1  → 通常テキストAAAに適合
6.999:1 → 不適合（丸めて7:1にしない）
4.50:1  → 大きなテキストAAAに適合
4.499:1 → 不適合
```

> **参照**
> - [Understanding SC 1.4.6 — threshold values](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html#intent)

## 細いフォント・アンチエイリアスへの注意

- 評価ではCSSで指定された前景色・背景色を基準にする
- ただし、細いフォントや特殊な書体は、アンチエイリアスにより実際には薄く見える場合がある
- AAAでは数値上ぎりぎりではなく、太さ・色ともに余裕を持たせる

```css
/* 避けたい例：数値上は近くても細く見える */
.thin-caption {
  font-weight: 100;
  color: #595959;
  background-color: #ffffff;
}

/* 望ましい例：余裕のあるコントラストと読みやすい太さ */
.caption {
  font-weight: 400;
  color: #333333;
  background-color: #ffffff;
}
```

> **参照**
> - [Understanding SC 1.4.6 — anti-aliasing note](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced.html#intent)

## ユーザーが色を切り替えられる場合（G174）

- 高コントラスト表示へ切り替えるコントロールを提供する方法
- 初期表示が7:1未満でも、十分なコントラストに切り替えられる仕組みがあれば達成手法になりうる
- 切り替えコントロール自体も十分なコントラストが必要
- 低コントラストを必要とする認知障害のあるユーザー向けにも、戻せる仕組みを用意

```html
<button type="button" id="contrast-toggle" aria-pressed="false">
  高コントラスト表示
</button>
```

```css
.high-contrast {
  color: #000000;
  background-color: #ffffff;
}

.high-contrast a {
  color: #003366;
}
```

```javascript
const button = document.getElementById('contrast-toggle');

button.addEventListener('click', () => {
  const enabled = document.body.classList.toggle('high-contrast');
  button.setAttribute('aria-pressed', String(enabled));
});
```

> **参照**
> - [Technique G174: Providing a control that allows users to switch to sufficient contrast](https://www.w3.org/WAI/WCAG22/Techniques/general/G174)

## AAAカラーパレットの設計

- 通常テキスト用の色は背景との 7:1 以上を基準にする
- 補助テキスト・注釈・placeholder も通常テキストなら 7:1 以上
- 大見出しだけ 4.5:1 に緩和できるが、本文と同じ基準にそろえると運用しやすい

```css
:root {
  /* 白背景でAAAを満たしやすいテキスト色 */
  --color-text-primary: #1a1a1a;   /* 約17:1 */
  --color-text-secondary: #333333; /* 約12.6:1 */
  --color-text-muted: #595959;     /* 約7:1 */

  --color-bg-primary: #ffffff;
  --color-link: #003d79;           /* 白背景で十分なコントラスト */
  --color-error: #9f1d1d;          /* 白背景でAAAを満たしやすい */
}

body {
  color: var(--color-text-primary);
  background-color: var(--color-bg-primary);
}
```

> **参照**
> - [a11y/color-contrast.md — カスタムプロパティでカラーパレットを管理](../a11y/color-contrast.md)

## 1.4.1・1.4.3・1.4.11 との関係

| 達成基準 | 対象 | レベル | 主な基準 |
|---|---|---|---|
| 1.4.1 色の使用 | 色だけに依存しない | A | 色以外の手がかり |
| 1.4.3 コントラスト（最低限） | テキスト・文字画像 | AA | 4.5:1 / 3:1 |
| 1.4.6 コントラスト（高度） | テキスト・文字画像 | AAA | 7:1 / 4.5:1 |
| 1.4.11 非テキストのコントラスト | UI・アイコン・図形 | AA | 3:1 |

- 1.4.6 は**テキスト**と**文字画像**が対象
- アイコン、グラフ線、入力欄の枠線、フォーカスインジケーターは 1.4.11 が対象
- 色だけでリンクや状態を示す問題は 1.4.1 も確認

> **参照**
> - [wcag/1-4-1-use-of-color.md — 1.4.1 色の使用](./1-4-1-use-of-color.md)
> - [WCAG 2.2 — Success Criterion 1.4.11 Non-text Contrast](https://www.w3.org/TR/WCAG22/#non-text-contrast)

## テスト・確認方法

### 自動テスト

- axe DevTools、Lighthouse、WAVE でテキストコントラストを検出
- 多くのツールは AA / AAA の切り替え確認が可能
- 背景画像上のテキスト、疑似要素、canvas、状態変化は手動確認が必要

### 手動テスト

- Chrome DevTools のカラーピッカーでコントラスト比を確認
- WebAIM Contrast Checker / Colour Contrast Analyser で前景色・背景色を入力
- 通常テキスト 7:1、大きなテキスト 4.5:1 を満たすか確認
- `placeholder`、ホバー、フォーカス、エラー、モーダル、通知など一時表示テキストも確認
- しきい値ぎりぎりの値を丸めて合格にしていないか確認

```bash
# CSS内の色指定をざっくり確認
rg "color:|background|border-color" --glob "*.css" --glob "*.scss"
```

> **参照**
> - [a11y/color-contrast.md — コントラスト比の確認ツール](../a11y/color-contrast.md)
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.6 はテキスト・文字画像と背景の**高度なコントラスト**を求める達成基準（レベル AAA）
- 通常テキストは **7:1**、大きなテキストは **4.5:1**
- 1.4.3 の上位基準で、例外条件はほぼ同じ
- 非アクティブ UI、純粋な装飾、偶発的な写真内文字、ロゴタイプは例外
- 背景画像、グラデーション、placeholder、ホバー/フォーカス時テキストも確認
- 数値上ぎりぎりではなく、余裕のある色・太さ・背景設計にする
- アイコンやUI枠線は 1.4.11、色だけの区別は 1.4.1 も併せて確認
