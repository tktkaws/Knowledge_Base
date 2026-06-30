# 1.4.4 テキストのサイズ変更（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.4「テキストのサイズ変更」（Resize Text）
- レベル **AA**（標準的な適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- キャプションと文字画像を除き、支援技術なしでテキストを **200%** まで拡大できること
- 200% 拡大しても、**内容や機能が失われない**こと

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.4 Resize Text](https://www.w3.org/TR/WCAG22/#resize-text)
> - [Understanding SC 1.4.4 Resize Text](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html)

## 達成基準の原文（要約）

- キャプションと文字画像を除き、テキストは支援技術なしで **200%** までサイズ変更できること
- 200% 拡大しても、コンテンツや機能が失われないこと
- ユーザーエージェントのズーム、テキストサイズ変更、または作者が用意したサイズ変更機能で達成可能

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.4](https://www.w3.org/TR/WCAG22/#resize-text)

## なぜ必要か

- ロービジョンのユーザーが、画面拡大ソフトなしでも文字を読みやすくするため
- ブラウザのズームやテキストサイズ変更を使っても、操作や読解を継続できるようにするため
- 文字が大きくなったときに、ボタン・入力欄・ナビゲーションが欠けないようにするため
- テキストは画像より拡大・折り返し・色変更に適しているため

> **参照**
> - [Understanding SC 1.4.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html#intent)
> - [Understanding SC 1.4.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html#benefits)

## 「200%まで」とは

- 文字が初期表示の**2倍**の大きさになること
- ブラウザのページズームで 200% にする方法でもよい
- ブラウザのテキストのみ拡大機能で 200% にする方法でもよい
- 作者がページ内に「文字サイズ 大」などの切り替え機能を用意してもよい
- 100% と 200% の間の各段階でも、内容・機能が失われない必要がある

```text
100% → 125% → 150% → 175% → 200%
各段階で、文字の欠け・重なり・操作不能がないこと
```

> **参照**
> - [Understanding SC 1.4.4 — 200% scaling](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html#intent)

## 対象外：キャプションと文字画像

- **キャプション**は 1.4.4 の対象外
- **文字画像**も 1.4.4 の対象外だが、1.4.5「文字画像」で別途扱う
- ただし、文字画像は拡大時に劣化しやすいため、可能な限り実テキストにする

```html
<!-- 悪い例：見出しを画像化している -->
<img src="heading.png" alt="キャンペーン情報">

<!-- 良い例：実テキスト + CSS -->
<h1 class="campaign-title">キャンペーン情報</h1>
```

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.5 Images of Text](https://www.w3.org/TR/WCAG22/#images-of-text)

## 相対単位を使う（C12, C13, C14）

- `rem` / `em` / `%` / キーワードなど、ユーザー設定に追従しやすい単位を使う
- `px` 指定が常に不適合ではないが、固定サイズに依存した設計は破綻しやすい
- `line-height` は単位なし推奨

```css
/* 悪い例：固定サイズ中心で拡大に弱い */
.card-title {
  font-size: 14px;
  line-height: 18px;
}

/* 良い例：相対単位 + 単位なし line-height */
.card-title {
  font-size: 1rem;
  line-height: 1.5;
}

.card-title--large {
  font-size: 1.25rem;
}
```

```css
/* 良い例：ルートサイズに追従 */
html {
  font-size: 100%;
}

body {
  font-size: 1rem;
}
```

> **参照**
> - [Technique C12: Using percent for font sizes](https://www.w3.org/WAI/WCAG22/Techniques/css/C12)
> - [Technique C13: Using named font sizes](https://www.w3.org/WAI/WCAG22/Techniques/css/C13)
> - [Technique C14: Using em units for font sizes](https://www.w3.org/WAI/WCAG22/Techniques/css/C14)

## コンテナもテキストに合わせて拡大する

- テキストだけ拡大しても、コンテナが固定高さだと欠ける
- `height` 固定より `min-height` を優先
- テキスト量が増えたときに折り返し・余白・ボタンサイズが追従する設計にする

```css
/* 悪い例：固定高さで文字が欠ける */
.button-bad {
  height: 32px;
  font-size: 16px;
  overflow: hidden;
}

/* 良い例：内容に応じて高さが伸びる */
.button-good {
  min-height: 2.5rem;
  padding: 0.75em 1em;
  font-size: 1rem;
  line-height: 1.4;
}
```

```html
<!-- 悪い例：拡大時にラベルが切れる -->
<button class="button-bad">申し込みを確定する</button>

<!-- 良い例：ボタンがテキストに追従 -->
<button class="button-good">申し込みを確定する</button>
```

> **参照**
> - [Technique SCR34: Calculating size and position in a way that scales with text size](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR34)
> - [Failure F69: Text, images or controls clipped, truncated or obscured at 200%](https://www.w3.org/WAI/WCAG22/Techniques/failures/F69)

## リキッドレイアウトを使う（G146）

- 固定幅だけに依存せず、画面幅や文字サイズに応じてレイアウトが流動するようにする
- `max-width`、`minmax()`、`flex-wrap`、`grid-template-columns` などを活用
- 横幅が足りない場合は折り返し・縦積みにする

```css
/* 悪い例：固定幅で200%拡大時に横にはみ出す */
.layout-bad {
  width: 960px;
}

/* 良い例：流動幅 */
.layout-good {
  width: min(100%, 60rem);
  margin-inline: auto;
  padding-inline: 1rem;
}

/* 良い例：カードを折り返す */
.cards {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.card {
  flex: 1 1 18rem;
}
```

> **参照**
> - [Technique G146: Using liquid layout](https://www.w3.org/WAI/WCAG22/Techniques/general/G146)

## フォームコントロールの拡大（F80）

- 入力欄・セレクト・ボタン内のテキストも 200% 拡大に対応
- テキストだけ大きくなり、フォームコントロールが固定サイズのままだと不適合
- `font: inherit` で親の文字サイズに追従させる

```css
/* 悪い例：フォームだけ固定で小さいまま */
input,
select,
button {
  font-size: 12px;
  height: 28px;
}

/* 良い例：フォームもテキストサイズに追従 */
input,
select,
textarea,
button {
  font: inherit;
  line-height: 1.4;
}

input,
select,
button {
  min-height: 2.5rem;
}
```

```html
<label for="email">メールアドレス</label>
<input id="email" type="email" autocomplete="email">
```

> **参照**
> - [Failure F80: Text-based form controls do not resize at 200%](https://www.w3.org/WAI/WCAG22/Techniques/failures/F80)
> - [Technique C17: Scaling form elements which contain text](https://www.w3.org/WAI/WCAG22/Techniques/css/C17)

## viewport 単位の誤用（F94）

- `vw` / `vh` だけで文字サイズを決めると、ブラウザズームやテキスト拡大に追従しない場合がある
- 特に `font-size: 2vw` のような指定は、画面幅にだけ依存しやすい
- `rem` と組み合わせて最小値・最大値を持たせる

```css
/* 悪い例：viewport幅だけで文字サイズを決める */
.headline-bad {
  font-size: 3vw;
}

/* 良い例：rem を含めてユーザー設定に追従 */
.headline-good {
  font-size: clamp(1.5rem, 2vw + 1rem, 3rem);
}
```

> **参照**
> - [Failure F94: Incorrect use of viewport units to resize text](https://www.w3.org/WAI/WCAG22/Techniques/failures/F94)

## クリップ・重なり・隠れに注意（F69）

- 200% 拡大時に文字がコンテナからはみ出して隠れると不適合
- 絶対配置・固定高さ・`overflow: hidden`・1行固定の UI は注意
- 重要なテキストを省略する場合、全文へアクセスできる仕組みが必要

```css
/* 悪い例：1行固定で重要な情報が消える */
.alert-bad {
  height: 2rem;
  overflow: hidden;
  white-space: nowrap;
}

/* 良い例：折り返しを許可 */
.alert-good {
  min-height: 2rem;
  white-space: normal;
  overflow-wrap: anywhere;
}
```

```html
<!-- 条件付きで許容される例：一覧では省略、詳細画面で全文表示 -->
<a href="/messages/123" class="subject">
  【重要】契約更新に関する長いお知らせ...
</a>
<!-- リンク先で件名全文が読める -->
```

- メール一覧や表計算のセルなど、UI上の制約で省略される場合でも、フォーカス・活性化・詳細画面で全文にアクセスできる必要がある

> **参照**
> - [Understanding SC 1.4.4 — truncation in UI components](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html#intent)
> - [Failure F69: Text, images or controls clipped, truncated or obscured at 200%](https://www.w3.org/WAI/WCAG22/Techniques/failures/F69)

## ブラウザズームとテキストのみ拡大

- 現代ブラウザのページズームは、テキスト・画像・レイアウト全体を拡大
- テキストのみ拡大に対応するユーザーエージェントもある
- 作者の責任は、これらの拡大機能を**妨げない**こと

```css
/* 避けたい例：ユーザーの文字サイズ設定を上書きしやすい */
html {
  font-size: 10px;
}

/* 良い例：ユーザー既定サイズを尊重 */
html {
  font-size: 100%;
}
```

> **参照**
> - [Understanding SC 1.4.4 — user agent responsibility](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html#intent)

## レスポンシブデザインとの関係

- 200% ズーム時に別のメディアクエリが適用されてもよい
- 重要なのは、最終的に初期表示の **200% 相当**までテキストが大きくなり、内容・機能が失われないこと
- 1.4.10「リフロー」は、320 CSS px 幅相当で水平スクロールなしに読めることを扱う別要件

```css
/* 悪い例：小さい画面で文字サイズを半分にして拡大効果を打ち消す */
@media (max-width: 400px) {
  body {
    font-size: 0.5rem;
  }
}

/* 良い例：狭い画面でも読みやすいサイズを維持 */
@media (max-width: 400px) {
  body {
    font-size: 1rem;
  }
}
```

> **参照**
> - [Understanding SC 1.4.4 — responsive variations](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html#intent)
> - [WCAG 2.2 — Success Criterion 1.4.10 Reflow](https://www.w3.org/TR/WCAG22/#reflow)

## 文字サイズ切り替え機能を用意する場合

- ブラウザ機能に任せてもよいが、作者側で切り替え UI を提供してもよい
- 切り替えボタン自体もキーボード操作・ラベル・コントラストを満たす必要がある
- 状態が分かるように `aria-pressed` 等を使う

```html
<button type="button" id="large-text" aria-pressed="false">
  文字を大きくする
</button>
```

```css
body.large-text {
  font-size: 125%;
}
```

```javascript
const button = document.getElementById('large-text');

button.addEventListener('click', () => {
  const enabled = document.body.classList.toggle('large-text');
  button.setAttribute('aria-pressed', String(enabled));
});
```

> **参照**
> - [Technique G178: Providing controls on the Web page that allow users to incrementally change the size of all text on the page](https://www.w3.org/WAI/WCAG22/Techniques/general/G178)

## よくある失敗例（Failures）

| Failure | 内容 |
|---|---|
| F69 | 200%拡大時にテキスト・画像・コントロールが切れる、重なる、隠れる |
| F80 | テキストベースのフォームコントロールが拡大しない |
| F94 | viewport単位の誤用でテキスト拡大が効かない |

```html
<!-- F69：固定高さでテキストが切れる -->
<div style="height: 40px; overflow: hidden;">
  重要なお知らせが長く続きます。200%拡大時に途中で切れます。
</div>

<!-- F80：入力欄内の文字が小さいまま -->
<input style="font-size: 10px; height: 20px;" value="小さい入力欄">
```

> **参照**
> - [WCAG 2.2 Techniques — Failures for 1.4.4](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html#failures)

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.4.3 コントラスト（最低限）（AA） | 拡大しても文字色と背景色のコントラストは必要 |
| 1.4.5 文字画像（AA） | 画像化された文字は拡大・変更しづらい |
| 1.4.10 リフロー（AA） | 400%拡大相当の狭い幅で横スクロールを避ける |
| 1.4.12 テキストの間隔（AA） | ユーザー指定の文字間隔・行間で内容が失われない |

> **参照**
> - [wcag/1-4-3-contrast-minimum.md — 1.4.3 コントラスト（最低限）](./1-4-3-contrast-minimum.md)
> - [WCAG 2.2 — Success Criterion 1.4.12 Text Spacing](https://www.w3.org/TR/WCAG22/#text-spacing)

## テスト・確認方法

### 手動テスト

- ブラウザズームを **200%** にする
- 内容が欠けない・重ならない・隠れないか確認
- ナビゲーション、ボタン、入力欄、モーダル、ツールチップも確認
- テキストだけ拡大できるブラウザ設定があれば、それでも確認
- 100%〜200%の途中段階でも破綻しないか確認

### 開発者ツールでの確認

- `height` 固定、`overflow: hidden`、`white-space: nowrap` を重点確認
- `font-size: vw` など viewport 単位のみの指定を確認
- `html { font-size: ... }` がユーザー設定を妨げていないか確認

```bash
# CSSで拡大時に問題になりやすい指定を確認
rg "height:|overflow:\\s*hidden|white-space:\\s*nowrap|font-size:.*vw" --glob "*.css" --glob "*.scss"
```

### ACT Rules

- [Meta viewport allows for zoom](https://www.w3.org/WAI/standards-guidelines/act/rules/b4f0c3/)
- [Zoomed text node is not clipped with CSS overflow](https://www.w3.org/WAI/standards-guidelines/act/rules/59br37/proposed/)

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.4 はテキストを **200%** まで拡大しても内容・機能が失われないことを求める達成基準（レベル AA）
- キャプションと文字画像は対象外。ただし文字画像は 1.4.5 で別途確認
- `rem` / `em` / `%` など相対単位、リキッドレイアウト、可変コンテナを使う
- 固定高さ、`overflow: hidden`、`white-space: nowrap`、viewport単位のみの文字サイズに注意
- フォームコントロール内の文字も拡大対象
- 1.4.10（リフロー）・1.4.12（テキスト間隔）と併せて確認
