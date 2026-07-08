# 1.4.11 非テキストのコントラスト（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.11「非テキストのコントラスト」（Non-text Contrast）
- レベル **AA**（標準的な適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- **UI コンポーネントの識別・状態表示**と、**理解に必要なグラフィカルオブジェクト**に **3:1 以上**のコントラスト比を確保
- **非アクティブ UI**、および特定の提示が不可欠なグラフィックは例外

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.11 Non-text Contrast](https://www.w3.org/TR/WCAG22/#non-text-contrast)
> - [Understanding SC 1.4.11 Non-text Contrast](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html)

## 達成基準の原文（要約）

- 次の視覚的提示が、隣接色とのコントラスト比 **3:1 以上**であること
  - **ユーザーインターフェースコンポーネント**：識別や状態把握に必要な視覚情報
  - **グラフィカルオブジェクト**：理解に必要な図形・線・アイコンなど
- 例外
  - **非アクティブな UI コンポーネント**
  - **ユーザーエージェントが見た目を決定し、作者が変更していない UI**
  - **特定のグラフィック提示が情報伝達に不可欠**な場合

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.11](https://www.w3.org/TR/WCAG22/#non-text-contrast)

## 対象は何か

### UI コンポーネント

- ボタン、リンク、チェックボックス、ラジオボタン、入力欄、セレクト、スイッチなど
- コンポーネントの**存在を識別するために必要な境界・背景・アイコン**
- コンポーネントの**状態**を示す視覚情報
  - フォーカス枠
  - チェックマーク
  - 選択中の塗り
  - ドロップダウン矢印

### グラフィカルオブジェクト

- 単独アイコン
- グラフの線、目盛線、円グラフの区切り
- 図やインフォグラフィックのうち、理解に必要な図形部分

```html
<!-- UI コンポーネントの例 -->
<input type="text" aria-label="キーワード検索">
<button type="button">保存</button>

<!-- グラフィカルオブジェクトの例 -->
<svg viewBox="0 0 24 24" role="img" aria-label="印刷">
  <path d="..." />
</svg>
```

> **参照**
> - [Understanding SC 1.4.11 — User Interface Components](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#user-interface-components)
> - [Understanding SC 1.4.11 — Graphical Objects](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#graphical-objects)

## 1.4.3 との違い

| 達成基準 | 対象 | レベル | 基準 |
|---|---|---|---|
| 1.4.3 コントラスト（最低限） | テキスト・文字画像 | AA | 通常 4.5:1 / 大きな文字 3:1 |
| 1.4.11 非テキストのコントラスト | UI・アイコン・図形・状態表示 | AA | **3:1** |

- 1.4.3 は**文字**が対象
- 1.4.11 は**文字以外の視覚情報**が対象
- ボタンのラベル文字は 1.4.3、ボタンの境界線やフォーカス枠は 1.4.11
- チャートのラベル文字は 1.4.3、チャートの線や円グラフの区切りは 1.4.11

```html
<!-- テキストは 1.4.3、枠線は 1.4.11 -->
<button class="button-outline">送信</button>
```

```css
.button-outline {
  color: #333333;              /* 1.4.3 */
  background-color: #ffffff;
  border: 1px solid #767676;   /* 1.4.11 */
}
```

> **参照**
> - [wcag/1-4-3-contrast-minimum.md — 1.4.3 コントラスト（最低限）](./1-4-3-contrast-minimum.md)

## なぜ必要か

- ロービジョンのユーザーが、操作できる部品や状態変化を見つけやすくするため
- コントラスト不足の入力欄やチェックボックスは、存在自体を見落とされやすいため
- グラフやアイコンの重要な図形が背景に埋もれると、情報理解が難しくなるため
- 文字が読めても、境界や状態表示が見えなければ操作できないことがあるため

> **参照**
> - [Understanding SC 1.4.11 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#intent)
> - [Understanding SC 1.4.11 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#benefits)

## UI コンポーネントの境界

- コントロールの存在を示す視覚情報が **3:1 以上**
- 文字ラベルや十分なコントラストのアイコンで存在が分かる場合、**ヒット領域全体の境界線は必須ではない**
- ただし、境界線しか手がかりがない場合、その境界線は 3:1 以上必要

```html
<!-- 悪い例：入力欄の境界が薄すぎる -->
<input class="input-bad" type="text" aria-label="氏名">

<!-- 良い例：境界で入力欄を識別できる -->
<input class="input-good" type="text" aria-label="氏名">
```

```css
.input-bad {
  border: 1px solid #aaaaaa; /* 白背景で約 2.3:1 */
  background-color: #ffffff;
}

.input-good {
  border: 1px solid #767676; /* 白背景で約 4.5:1 */
  background-color: #ffffff;
}
```

> **参照**
> - [Understanding SC 1.4.11 — Boundaries](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#boundaries)

## 隣接色で評価する

- 1.4.11 は**隣接する色**とのコントラストで評価する
- 例えば入力欄の枠線は、通常は**外側の背景色**とのコントラストを見る
- 状態表示がコンポーネント内部にある場合、その部分の隣接色は**コンポーネント内部色**になることがある

```css
/* 入力欄の外側背景と境界線のコントラストを見る */
.search-field {
  border: 1px solid #767676;
  background-color: #ffffff;
}

.search-wrapper {
  background-color: #ffffff;
}
```

> **参照**
> - [Understanding SC 1.4.11 — Adjacent colors](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#adjacent-colors)

## チェックボックス・ラジオボタン・スイッチ

- 枠線だけでコントロールの存在を示すなら、その枠線が 3:1 以上必要
- チェック済み状態の**チェックマーク**や**塗り**も、隣接色との 3:1 以上が必要
- hover だけの微妙な色変化は、それ自体が 3:1 なくてもよいが、コントロール自体の視認性は維持する

```html
<!-- 良い例：枠線とチェックマークの両方が十分なコントラスト -->
<label class="check-row">
  <input type="checkbox" checked>
  通知を受け取る
</label>
```

```css
.check-row input[type="checkbox"] {
  accent-color: #005fcc;
}

/* カスタムUIにする場合は、枠線とチェックの両方を確認 */
.checkbox-custom {
  border: 2px solid #333333;
  background-color: #ffffff;
}

.checkbox-custom[aria-checked="true"]::after {
  content: "";
  display: block;
  width: 0.5rem;
  height: 0.25rem;
  border-left: 2px solid #ffffff;
  border-bottom: 2px solid #ffffff;
  transform: rotate(-45deg);
}
```

> **参照**
> - [Understanding SC 1.4.11 — Checkbox examples](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#user-interface-components)

## フォーカス表示（2.4.7 とも関係）

- 作者がフォーカス表示をカスタマイズした場合、**フォーカスインジケーター**は 3:1 以上必要
- 外側のアウトラインなら、外側背景とのコントラストを確認
- 内側ボーダーなら、コンポーネント内部色とのコントラストも確認
- デフォルトの UA フォーカス表示を変更していない場合は、この要件の対象外になりうる

```css
/* 悪い例：白背景に黄色アウトラインで見えづらい */
.button-bad:focus-visible {
  outline: 2px solid #ffff00;
  outline-offset: 2px;
}

/* 良い例：白背景に十分見える濃い緑 */
.button-good:focus-visible {
  outline: 3px solid #008000;
  outline-offset: 2px;
}
```

> **参照**
> - [Understanding SC 1.4.11 — Relationship with Focus Visible](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#relationship-with-focus-visible)
> - [Technique G195: Using an author-supplied, visible focus indicator](https://www.w3.org/WAI/WCAG22/Techniques/general/G195)
> - [Technique C40: Creating a two-color focus indicator](https://www.w3.org/WAI/WCAG22/Techniques/css/C40)

## hover 状態

- `hover` の追加装飾は、それ自体が必ずしも 3:1 必要ではない
- ただし hover によって**元のコントロールの視認性**や、focus/selected 状態の視認性を下げてはいけない
- 位置そのものやポインタ形状が hover 状態の主要な手がかりになる場合もある

```css
/* 許容される例：hover 背景を少し変えるが枠線は十分見える */
.checkbox:hover {
  background-color: #dedede;
  border-color: #333333;
}
```

> **参照**
> - [Understanding SC 1.4.11 — Hover states](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#hover-states)

## アイコン

- テキストなしの単独アイコンは、理解に必要なら 3:1 以上必要
- 塗りつぶしアイコンでも、線画アイコンでも同じ
- 薄いグレーのアイコンを白背景に置くと不足しやすい

```html
<!-- 悪い例：薄いアイコンで見えにくい -->
<button type="button" aria-label="印刷" class="icon-button-bad">
  <svg viewBox="0 0 24 24" aria-hidden="true"><path d="..." /></svg>
</button>

<!-- 良い例：濃いアイコン -->
<button type="button" aria-label="印刷" class="icon-button-good">
  <svg viewBox="0 0 24 24" aria-hidden="true"><path d="..." /></svg>
</button>
```

```css
.icon-button-bad svg {
  fill: #b3b3b3; /* 白背景で不足しやすい */
}

.icon-button-good svg {
  fill: #333333;
}
```

> **参照**
> - [Technique G207: Ensuring that a contrast ratio of 3:1 is provided for icons](https://www.w3.org/WAI/WCAG22/Techniques/general/G207)

## グラフ・図・インフォグラフィック

- 理解に必要な線・図形・区切りが 3:1 以上必要
- 目盛線、折れ線、棒、円グラフの境界、重要な記号が対象
- 同じ情報を十分なコントラストの**テキストや表**で提供している場合、図形自体が「理解に必須」でなくなることがある

```html
<!-- 良い例：グラフに加えて数値表も提供 -->
<figure>
  <img src="sales-chart.png" alt="">
  <figcaption>四半期売上の推移</figcaption>
</figure>

<table>
  <caption>四半期売上</caption>
  <tr><th>Q1</th><td>120</td></tr>
  <tr><th>Q2</th><td>140</td></tr>
</table>
```

```css
/* 良い例：線が背景と十分にコントラストを持つ */
.chart-line-primary {
  stroke: #005fcc;
}

.chart-grid-line {
  stroke: #767676;
}
```

> **参照**
> - [Understanding SC 1.4.11 — Graphical Objects](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#graphical-objects)
> - [Technique G209: Provide sufficient contrast at the boundaries between adjoining colors](https://www.w3.org/WAI/WCAG22/Techniques/general/G209)

## 円グラフと隣接色

- 円グラフでは、スライス同士の境界が理解に必要
- 背景とのコントラストだけでなく、**隣接するスライス同士**の区別も確認
- 境界線を追加して 3:1 を確保すると分かりやすい

```css
.pie-slice-warning {
  fill: #f5a623;
  stroke: #333333;
  stroke-width: 2;
}
```

> **参照**
> - [Understanding SC 1.4.11 — Pie Charts](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#pie-charts)

## ロゴ・装飾・不可欠な図

### 対象外になりうるもの

- 純粋な装飾
- 理解に不要な背景図形
- ブランドガイドライン上のロゴ
- スクリーンショット、医療図、ヒートマップなど、特定の見た目自体が不可欠な図

```html
<!-- ロゴは原則例外 -->
<img src="brand-logo.svg" alt="Example Corp">

<!-- 装飾図形 -->
<img src="background-shape.svg" alt="" aria-hidden="true">
```

- ただし、ロゴを**UI コンポーネント**として使う場合は注意
- 低コントラスト版ロゴしか見えないなら、別のホームリンクや高コントラスト版ロゴを検討
- 作者の都合で薄く表示しているだけなら、「不可欠」とは言いにくい

> **参照**
> - [Understanding SC 1.4.11 — Logos](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#logos)
> - [Understanding SC 1.4.11 — Essential Exception](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#essential-exception)

## 非アクティブ UI は例外

- `disabled` など、現在操作できないコントロールは対象外
- ただし、ユーザビリティ上は見分けやすい方が望ましい
- 有効/無効の切り替えが頻繁に起きるUIでは、説明文や状態の明示も検討

```html
<button disabled class="submit-disabled">送信</button>
```

> **参照**
> - [Understanding SC 1.4.11 — Inactive User Interface Components](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#inactive-user-interface-components)

## 記号として使う文字

- `X`、`>`、`B` のような文字を、言語としてではなく**記号**として使う場合は非テキスト扱い
- この場合は 1.4.11 の 3:1 が基準になりうる
- ラベル文字として読ませるなら 1.4.3 側も確認

```html
<!-- 記号としての文字 -->
<button type="button" aria-label="閉じる">X</button>
```

> **参照**
> - [Understanding SC 1.4.11 — Symbolic text characters](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#symbolic-text-characters)

## 1.4.1・2.4.7 との関係

| 達成基準 | 対象 | 主な確認点 |
|---|---|---|
| 1.4.1 色の使用 | 色だけに依存しない | 色以外の手がかりがあるか |
| 1.4.11 非テキストのコントラスト | UI・図形・状態表示 | 3:1 以上あるか |
| 2.4.7 フォーカスの可視化 | キーボードフォーカス | フォーカスが見えるか |

- 色の違いだけで状態を示すと 1.4.1 の問題になりやすい
- フォーカスが見えても、コントラスト不足なら 1.4.11 の問題になりうる
- 1.4.11 は**状態差そのもの**の比較ではなく、各状態で必要な視覚情報が背景と十分コントラストを持つかを見る

> **参照**
> - [wcag/1-4-1-use-of-color.md — 1.4.1 色の使用](./1-4-1-use-of-color.md)
> - [Understanding SC 1.4.11 — Relationship with Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#relationship-with-use-of-color)

## しきい値は丸めない

- 3:1 は**しきい値**
- 計算値を丸めて適合扱いにしない
- 例：`2.999:1` は不適合

```text
3.00:1  → 適合
2.999:1 → 不適合
```

> **参照**
> - [Understanding SC 1.4.11 — threshold values](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#intent)

## 細い線・アンチエイリアスの注意

- 評価では、画面見た目より**CSSや指定色**を基準にする
- ただし、細すぎる線や図形は実際にはかなり薄く見えることがある
- 数値ぎりぎりで設計せず、太さや色に余裕を持たせる

```css
/* 避けたい例：細すぎるグラフ線 */
.chart-line-thin {
  stroke: #767676;
  stroke-width: 1;
}

/* 望ましい例：少し太く、余裕ある色 */
.chart-line-good {
  stroke: #4a4a4a;
  stroke-width: 2;
}
```

> **参照**
> - [Understanding SC 1.4.11 — anti-aliasing note](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html#intent)

## よくある失敗例

- 薄いグレーの入力欄枠線で、入力欄の存在が分かりにくい
- カスタムチェックボックスの枠線やチェックが背景と区別しづらい
- 独自フォーカスリングが白背景で見えない
- グラフ線や円グラフ境界が背景や隣接色と近すぎる
- アイコンボタンが薄いグレーで埋もれている
- hover や selected の視覚効果で、かえって部品の視認性が落ちる

```html
<!-- 悪い例：ラベルもなく、薄い枠線だけで入力欄を示している -->
<input style="border: 1px solid #aaa; background: #fff;" type="text">

<!-- 良い例：ラベル + 十分な枠線 -->
<label for="search">検索</label>
<input id="search" style="border: 1px solid #767676; background: #fff;" type="search">
```

> **参照**
> - [Failure F78: styling outlines and borders in a way that removes or renders non-visible the visual focus indicator](https://www.w3.org/WAI/WCAG22/Techniques/failures/F78)

## テスト・確認方法

### 手動テスト

- ページ内のボタン、入力欄、チェックボックス、ラジオ、セレクト、スイッチを確認
- **存在の識別に必要な境界・背景・アイコン**が 3:1 以上あるか確認
- `:focus-visible`、`checked`、`selected`、`open` など状態ごとに確認
- 単独アイコン、グラフ線、円グラフ境界、図の重要部分を確認
- 同じ情報がテキストや表で提供されているかも確認

### 開発者ツールでの確認

- Chrome DevTools のカラーピッカーで色を取得して contrast checker を利用
- デザインツールや Contrast Checker で隣接色との比率を測る
- 枠線やアイコンは「背景との比較」、内部状態表示は「隣接部分との比較」を意識する

```bash
# 境界線・アウトライン・アイコン色をざっくり確認
rg "border:|border-color:|outline:|outline-color:|stroke:|fill:" --glob "*.css" --glob "*.scss" --glob "*.tsx" --glob "*.jsx"
```

> **参照**
> - [a11y/color-contrast.md — コントラスト比の確認ツール](../a11y/color-contrast.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.11 は、**UI の識別・状態表示**と**理解に必要なグラフィック**に **3:1 以上**のコントラストを求める達成基準（レベル AA）
- テキストは 1.4.3、非テキストの境界・アイコン・線・図形は 1.4.11 が対象
- 入力欄枠線、チェックマーク、ドロップダウン矢印、フォーカス枠は見落としやすい重点項目
- グラフやインフォグラフィックは、理解に必要な図形部分だけを見極めて評価する
- 非アクティブ UI、装飾、不可欠な図、ブランドロゴは例外になりうる
- 数値ぎりぎりではなく、線の太さや色に余裕を持たせると実運用で安全
