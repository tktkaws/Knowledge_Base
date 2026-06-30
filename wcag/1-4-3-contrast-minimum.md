# 1.4.3 コントラスト（最低限）（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.3「コントラスト（最低限）」（Contrast (Minimum)）
- レベル **AA**（標準的な適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- テキストおよび文字画像の視覚的提示に、背景との十分な**コントラスト比**を確保
- 通常テキストは **4.5:1 以上**、大きなテキストは **3:1 以上**

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.3 Contrast (Minimum)](https://www.w3.org/TR/WCAG22/#contrast-minimum)
> - [Understanding SC 1.4.3 Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)

## 達成基準の原文（要約）

- テキストおよび文字画像は、背景とのコントラスト比が **4.5:1 以上**
- 例外
  - **大きなテキスト**：3:1 以上
  - **付随的**：非アクティブ UI、純粋な装飾、誰にも見えないテキスト、重要な他の視覚コンテンツを含む画像の一部
  - **ロゴタイプ**：ロゴ・ブランド名の一部であるテキスト

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.3](https://www.w3.org/TR/WCAG22/#contrast-minimum)

## コントラスト比とは

- 前景色（文字色）と背景色の**相対輝度**の差を表す比率
- 範囲は **1:1**（同じ色）から **21:1**（黒と白）
- 計算式：`(L1 + 0.05) / (L2 + 0.05)`
  - `L1`：明るい方の相対輝度
  - `L2`：暗い方の相対輝度
- 実務では DevTools や Contrast Checker で確認。公式を暗記する必要はない

```text
白 #ffffff と黒 #000000 → 21:1
白 #ffffff と #767676 → 約 4.5:1
白 #ffffff と #999999 → 約 2.8:1（通常テキストでは不足）
```

> **参照**
> - [Understanding SC 1.4.3 — contrast ratio](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html#dfn-contrast-ratio)
> - [a11y/color-contrast.md — カラーコントラスト比とは](../a11y/color-contrast.md)

## なぜ必要か

- ロービジョン・高齢者・色覚特性のあるユーザーがテキストを読めるようにするため
- 色相ではなく**明暗差**を確保することで、幅広い視覚特性に対応
- 屋外・低品質ディスプレイ・画面の反射など、一般ユーザーにも読みやすさを提供
- 支援技術なしで読める最低限の視認性を確保

> **参照**
> - [Understanding SC 1.4.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html#benefits)

## 必要なコントラスト比

| 対象 | レベル AA（1.4.3） | レベル AAA（1.4.6） |
|---|---:|---:|
| 通常テキスト | **4.5:1** 以上 | 7:1 以上 |
| 大きなテキスト | **3:1** 以上 | 4.5:1 以上 |

### 大きなテキストの定義

- 18pt（約 **24px**）以上
- または 14pt 太字（約 **18.66px**）以上
- CJK（日本語・中国語・韓国語）では同等サイズの文字
- 極端に細いフォント・装飾的なフォントは、数値上適合しても読みづらい場合がある

```css
/* 通常テキスト：4.5:1 以上が必要 */
.body-text {
  font-size: 16px;
  color: #767676; /* 白背景で約 4.5:1 */
  background-color: #ffffff;
}

/* 大きなテキスト：3:1 以上が必要 */
.hero-title {
  font-size: 24px;
  color: #949494; /* 白背景で約 3:1 */
  background-color: #ffffff;
}

/* 太字の大きなテキスト */
.large-bold {
  font-size: 19px;
  font-weight: 700;
  color: #949494; /* 白背景で約 3:1 */
  background-color: #ffffff;
}
```

> **参照**
> - [Understanding SC 1.4.3 — Large Text](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html#dfn-large-scale)

## 通常テキストの例

```html
<!-- 悪い例：通常テキストのコントラスト不足（約 2.8:1） -->
<p style="color: #999999; background-color: #ffffff;">
  このテキストは薄すぎて読みづらい
</p>

<!-- 良い例：通常テキスト 4.5:1 以上 -->
<p style="color: #767676; background-color: #ffffff;">
  このテキストはAA基準を満たす
</p>

<!-- より良い例：余裕を持ったコントラスト -->
<p style="color: #333333; background-color: #ffffff;">
  このテキストは十分に読みやすい
</p>
```

> **参照**
> - [Technique G18: Contrast ratio of at least 4.5:1](https://www.w3.org/WAI/WCAG22/Techniques/general/G18)

## 大きなテキストの例

```html
<!-- 悪い例：大きなテキストでも 3:1 未満は不足 -->
<h1 style="font-size: 32px; color: #b3b3b3; background-color: #ffffff;">
  キャンペーン見出し
</h1>

<!-- 良い例：大きなテキストは 3:1 以上 -->
<h1 style="font-size: 32px; color: #949494; background-color: #ffffff;">
  キャンペーン見出し
</h1>
```

> **参照**
> - [Technique G145: Contrast ratio of at least 3:1 for large-scale text](https://www.w3.org/WAI/WCAG22/Techniques/general/G145)

## 文字画像も対象

- 画像として描画されたテキストも 1.4.3 の対象
- 背景とのコントラスト比を満たす必要がある
- ただし、重要な他の視覚コンテンツを含む写真内の偶発的な文字は例外

```html
<!-- 悪い例：文字画像のコントラスト不足 -->
<img src="sale-banner-low-contrast.png" alt="春のセール 30%OFF">

<!-- 良い例：実テキスト + CSS で表現 -->
<div class="sale-banner">
  <p>春のセール 30%OFF</p>
</div>
```

```css
.sale-banner {
  background-color: #ffffff;
}

.sale-banner p {
  color: #333333; /* 十分なコントラスト */
  font-size: 2rem;
}
```

- 文字画像は拡大時に劣化しやすく、色変更もしづらい
- 1.4.5「文字画像」も併せて確認

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.5 Images of Text](https://www.w3.org/TR/WCAG22/#images-of-text)

## 例外：付随的なテキスト

### 非アクティブ UI

- disabled 状態など、現在操作できない UI コンポーネント内のテキスト
- 1.4.3 のコントラスト要件は適用されない
- ただし、ユーザビリティ上は読める方が望ましい

```html
<!-- 例外：非アクティブなボタン -->
<button disabled style="color: #cccccc; background-color: #f5f5f5;">
  送信
</button>
```

### 純粋な装飾

- 語順を入れ替えても意味が変わらない背景文字
- 情報・機能を持たない文字

```html
<!-- 例外：装飾的な背景文字 -->
<div aria-hidden="true" class="decorative-words">
  SALE SALE SALE
</div>
```

### 写真内の偶発的な文字

- 街頭写真に写り込んだ看板など、画像の主目的ではない文字
- ただし、その文字を読ませる目的の画像なら対象

```html
<!-- 例外になりうる：街の写真の一部に看板が写っている -->
<img src="street-photo.jpg" alt="夕方の商店街の写真">

<!-- 対象：看板の文字を読ませる目的の画像 -->
<img src="store-sign.jpg" alt="営業時間：10時から19時">
```

> **参照**
> - [Understanding SC 1.4.3 — Incidental](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html#intent)

## 例外：ロゴタイプ

- ロゴやブランド名の一部であるテキストは対象外
- ただし、ロゴがリンクやボタンなど**UI コンポーネント**として機能する場合は、代替手段を検討
- ブランドガイドラインが許すなら、高コントラスト版ロゴの用意が望ましい

```html
<!-- ロゴタイプ自体は例外 -->
<a href="/" aria-label="ホーム">
  <img src="brand-logo.svg" alt="Example Corp">
</a>
```

> **参照**
> - [Understanding SC 1.4.3 — Logotypes](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html#intent)

## 背景画像・グラデーション上のテキスト

- 背景が画像やグラデーションの場合、**通常表示で文字が重なる範囲**の最も低いコントラストを確認
- 背景が変動する場合は、オーバーレイ・テキストシャドウ・背景色で安定させる

```css
/* 悪い例：背景画像次第で読めない */
.hero-bad {
  background-image: url("bright-photo.jpg");
  color: #ffffff;
}

/* 良い例：半透明オーバーレイでコントラストを安定 */
.hero-good {
  position: relative;
  color: #ffffff;
  background-image: linear-gradient(rgba(0, 0, 0, 0.65), rgba(0, 0, 0, 0.65)),
    url("bright-photo.jpg");
}
```

> **参照**
> - [Failure F83: Background images that do not provide sufficient contrast](https://www.w3.org/WAI/WCAG22/Techniques/failures/F83)
> - [a11y/color-contrast.md — 背景画像上のテキスト](../a11y/color-contrast.md)

## 前景色・背景色の片方だけ指定しない（Failure F24）

- 文字色だけ指定し、背景色をユーザーエージェントや親要素に依存すると、組み合わせが予測できない
- 背景色だけ指定し、文字色を未指定にする場合も同様
- 評価不能な組み合わせは Failure F24 になりうる

```css
/* 悪い例：文字色だけ指定 */
.notice-bad {
  color: #777777;
  /* background-color 未指定 */
}

/* 悪い例：背景色だけ指定 */
.panel-bad {
  background-color: #222222;
  /* color 未指定。ユーザー設定次第で読めない */
}

/* 良い例：前景色と背景色をセットで指定 */
.notice-good {
  color: #333333;
  background-color: #ffffff;
}
```

> **参照**
> - [Failure F24: Specifying foreground colors without specifying background colors or vice versa](https://www.w3.org/WAI/WCAG22/Techniques/failures/F24)

## プレースホルダー・ホバー・フォーカス時のテキスト

- `placeholder` テキストも 1.4.3 の対象
- ツールチップ、ホバー表示、フォーカス時に表示されるテキストも対象
- 薄いグレーの placeholder は不足しやすい

```html
<!-- 悪い例：placeholder のコントラスト不足 -->
<input
  type="email"
  placeholder="example@example.com"
  style="color: #333333; background-color: #ffffff;"
>

<!-- CSSで placeholder 色も確認 -->
<style>
  input::placeholder {
    color: #777777; /* 白背景で 4.5:1 以上を目安 */
  }
</style>
```

> **参照**
> - [Understanding SC 1.4.3 — placeholder and hover/focus text](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html#intent)

## 閾値は丸めない

- 4.5:1 や 3:1 は**しきい値**
- 計算値を丸めて適合扱いにしない
- 例：`4.499:1` は **4.5:1 を満たさない**

```text
4.50:1  → 適合
4.499:1 → 不適合（丸めて 4.5 としない）
```

> **参照**
> - [Understanding SC 1.4.3 — threshold values](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html#intent)

## アンチエイリアス・細いフォントの注意

- 評価では CSS で指定された前景色・背景色を基準にする
- ただし、細いフォントや特殊な書体は、アンチエイリアスで実際には薄く見える場合がある
- 数値上ぎりぎりではなく、余裕のある色・太さを選ぶのが望ましい

```css
/* 避けたい例：薄く細い文字でギリギリのコントラスト */
.thin-text {
  font-weight: 100;
  color: #767676; /* 白背景で約 4.5:1 だが実際には薄く見える */
}

/* 望ましい例：余裕を持ったコントラストと通常以上の太さ */
.readable-text {
  font-weight: 400;
  color: #333333;
}
```

> **参照**
> - [Understanding SC 1.4.3 — anti-aliasing note](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html#intent)

## ユーザーが色を切り替えられる場合（G174）

- 初期表示が不足していても、十分なコントラストへ切り替える**コントロール**を提供する手法
- コントロール自体も十分なコントラストが必要
- 低コントラストが必要な認知障害のあるユーザーにも配慮できる

```html
<!-- 高コントラスト表示への切り替え -->
<button type="button" id="contrast-toggle">
  高コントラスト表示に切り替え
</button>
```

```css
.high-contrast {
  color: #000000;
  background-color: #ffffff;
}
```

> **参照**
> - [Technique G174: Providing a control that allows users to switch to sufficient contrast](https://www.w3.org/WAI/WCAG22/Techniques/general/G174)

## 1.4.1・1.4.6・1.4.11 との関係

| 達成基準 | 対象 | レベル | 主な基準 |
|---|---|---|---|
| 1.4.1 色の使用 | 色だけに依存しない | A | 色以外の手がかり |
| 1.4.3 コントラスト（最低限） | テキスト・文字画像 | AA | 4.5:1 / 3:1 |
| 1.4.6 コントラスト（高度） | テキスト・文字画像 | AAA | 7:1 / 4.5:1 |
| 1.4.11 非テキストのコントラスト | UI・アイコン・図形 | AA | 3:1 |

- 1.4.3 は**テキスト**が対象
- ボタンの枠線、アイコン、グラフの線などは 1.4.11 が対象
- 色だけでリンクや状態を示す問題は 1.4.1 も確認

> **参照**
> - [wcag/1-4-1-use-of-color.md — 1.4.1 色の使用](./1-4-1-use-of-color.md)
> - [WCAG 2.2 — Success Criterion 1.4.11 Non-text Contrast](https://www.w3.org/TR/WCAG22/#non-text-contrast)

## テスト・確認方法

### 自動テスト

- axe DevTools、Lighthouse、WAVE で多くのテキストコントラストを検出可能
- 背景画像上のテキスト、状態変化、疑似要素、canvas 内文字は手動確認が必要

### 手動テスト

- Chrome DevTools のカラーピッカーでコントラスト比を確認
- Firefox DevTools のアクセシビリティタブで確認
- WebAIM Contrast Checker / Colour Contrast Analyser で前景色・背景色を入力
- 通常テキスト 4.5:1、大きなテキスト 3:1 を満たすか確認
- `placeholder`、ホバー、フォーカス、エラー表示、モーダル内テキストも確認

```bash
# CSS内の色指定をざっくり確認
rg "color:|background|border-color" --glob "*.css" --glob "*.scss"
```

> **参照**
> - [a11y/color-contrast.md — コントラスト比の確認ツール](../a11y/color-contrast.md)
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.3 はテキスト・文字画像と背景の**最低限のコントラスト**を求める達成基準（レベル AA）
- 通常テキストは **4.5:1**、大きなテキストは **3:1**
- 大きなテキストは 18pt（約24px）以上、または 14pt 太字（約18.66px）以上
- 非アクティブ UI、純粋な装飾、偶発的な写真内文字、ロゴタイプは例外
- 背景画像、グラデーション、placeholder、ホバー/フォーカス時テキストも確認
- 1.4.1（色だけに依存しない）・1.4.11（非テキスト）とは別要件
- しきい値は丸めず、余裕を持ったカラーパレットを設計する
