# 1.4.12 テキストの間隔（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.12「テキストの間隔」（Text Spacing）
- レベル **AA**（標準的な適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- WCAG 2.1 で追加された達成基準
- ユーザーがテキスト間隔を上書きしても、**内容や機能が失われない**こと
- マークアップ言語で指定可能な次の4つの間隔を、ユーザーが変更できること

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.12 Text Spacing](https://www.w3.org/TR/WCAG22/#text-spacing)
> - [Understanding SC 1.4.12 Text Spacing](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html)

## 達成基準の原文（要約）

- マークアップ言語で次のテキストスタイルを指定できる場合、**次の4つすべて**を設定しても内容や機能が失われないこと
  - **行間**（line height）：フォントサイズの **1.5 倍以上**
  - **段落後の間隔**：フォントサイズの **2 倍以上**
  - **字間**（letter spacing）：フォントサイズの **0.12 倍以上**
  - **語間**（word spacing）：フォントサイズの **0.16 倍以上**
- 他のスタイルプロパティは変更しない
- 例外：言語や表記体系で使わないプロパティは、その組み合わせで存在するものだけで適合可能
- 注記：作者がこれらの値で表示する必要はない。ユーザーが上書きしたときに問題が起きないことが要件

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.12](https://www.w3.org/TR/WCAG22/#text-spacing)

## テストで使う4つの値

| プロパティ | 最小値 | CSS 例（font-size: 16px の場合） |
|---|---|---|
| `line-height` | 1.5 × font-size | `line-height: 1.5` または `24px` |
| 段落後の間隔 | 2 × font-size | `margin-block-end: 2em` |
| `letter-spacing` | 0.12 × font-size | `letter-spacing: 0.12em` |
| `word-spacing` | 0.16 × font-size | `word-spacing: 0.16em` |

```css
/* テスト用：4つの値をまとめて上書き */
* {
  line-height: 1.5 !important;
  letter-spacing: 0.12em !important;
  word-spacing: 0.16em !important;
}

p {
  margin-block-end: 2em !important;
}
```

- テストでは**4つすべて**を同時に適用する
- 他のプロパティ（色、フォント、幅など）は変更しない
- 値は**下限**であり、作者はこれ以上の間隔にも対応できると望ましい

> **参照**
> - [Understanding SC 1.4.12 — Success Criterion](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html#success-criterion)

## なぜ必要か

- ロービジョンのユーザーが、行・語・字の間隔を広げて読みやすくするため
- 読字障害（ディスレクシア）のあるユーザーが、読書速度を上げるため
- 認知障害のあるユーザーが、段落やブロックの区切りを追いやすくするため
- ユーザーが自分に合った間隔に調整できると、他の表示設定（フォント変更など）も使いやすくなる

> **参照**
> - [Understanding SC 1.4.12 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html#intent)
> - [Understanding SC 1.4.12 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html#benefits)

## 作者の責任

- ページを最初からこの間隔で表示する必要は**ない**
- ユーザーが間隔を上書きしても、内容や機能が失われない設計にする
- ユーザーの上書きを**妨げない**こと
- ページ内に間隔調整 UI を用意する必要もない
- ユーザーはブラウザ拡張、ユーザースタイルシート、ブックマークレット等で変更できる

```css
/* 悪い例：!important でユーザー上書きを妨げる */
.title {
  line-height: 1.2 !important;
  letter-spacing: -0.05em !important;
}

/* 良い例：固定値でも上書き可能な設計 */
.title {
  line-height: 1.2;
  letter-spacing: normal;
}
```

> **参照**
> - [Understanding SC 1.4.12 — Author Responsibility](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html#author-responsibility)

## 対象外

### マークアップベースでないコンテンツ

- **PDF** はマークアップ実装ではないため、この SC の対象外
- HTML/CSS、SVG テキスト、適切なマークアップを持つコンテンツが対象

### スタイルで調整できないテキスト

- 動画フレームに直接埋め込まれたキャプション（関連ファイルではない場合）
- **文字画像**
- Canvas 上に描画されたテキスト（文字画像と同様に扱われる）

```html
<!-- 対象外：文字画像 -->
<img src="heading.png" alt="見出し">

<!-- 対象：HTMLテキスト -->
<h1>見出し</h1>
```

> **参照**
> - [Understanding SC 1.4.12 — Applicability](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html#applicability)

## 言語・表記体系の例外

- 言語や表記体系で使わないプロパティは、その組み合わせで存在するものだけで適合可能
- **日本語**では段落間の空白を通常使わないため、`word-spacing` や段落後の間隔は実質影響しにくい
- 日本語で `word-spacing` を適用しても効果がないのは想定内
- 作者は各言語の読みやすさに関するローカルなガイダンスに従うことが推奨される

```text
日本語の例：
  word-spacing の変更 → 通常は効果なし（想定内）
  line-height の変更 → 影響あり、対応が必要
  letter-spacing の変更 → 影響あり、対応が必要
```

> **参照**
> - [Understanding SC 1.4.12 — Languages and Scripts](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html#languages-and-scripts)

## 固定高さ・overflow で文字が切れる（F104）

- 間隔を広げると、固定高さのコンテナから文字がはみ出して**切れる**と不適合
- `overflow: hidden`、`height` 固定、`line-height` 固定が原因になりやすい
- 縦方向・横方向のどちらの切り捨ても問題

```css
/* 悪い例：固定高さで行間拡大時に切れる */
.card-title-bad {
  height: 2rem;
  line-height: 1.2;
  overflow: hidden;
}

/* 良い例：内容に応じて高さが伸びる */
.card-title-good {
  min-height: 2rem;
  line-height: 1.4;
  overflow: visible;
}
```

```html
<!-- 悪い例：見出しの下半分が切れる -->
<h2 class="card-title-bad">We Provide a Mobile Application Service to Meet Your Needs</h2>

<!-- 良い例：行間を広げても全文表示 -->
<h2 class="card-title-good">We Provide a Mobile Application Service to Meet Your Needs</h2>
```

> **参照**
> - [Failure F104: clipped or overlapped content when text spacing is adjusted](https://www.w3.org/WAI/WCAG22/Techniques/failures/F104)
> - [Understanding SC 1.4.12 — Text Cut Off](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html#effects)

## 文字の重なり（F104）

- 間隔を広げると、隣接要素と**重なる**と不適合
- 見出しと本文、ボタン内テキスト、ナビゲーション項目で起きやすい
- 絶対配置、負のマージン、固定レイアウトに注意

```css
/* 悪い例：行間拡大で下の要素と重なる */
.heading-bad {
  line-height: 1.1;
  margin-block-end: 0;
  position: relative;
  z-index: 1;
}

.body-bad {
  margin-block-start: -0.5rem;
}

/* 良い例：十分な余白を確保 */
.heading-good {
  line-height: 1.4;
  margin-block-end: 0.5em;
}
```

> **参照**
> - [Understanding SC 1.4.12 — Text Overlap](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html#effects)

## テキスト間隔の上書きを許可する（C36）

- 固定高さを避け、コンテナを内容に合わせて伸縮させる
- `line-height` は単位なし推奨
- `em` 単位でコンテナサイズを指定し、文字サイズに追従させる

```css
/* 良い例：間隔変更に耐える設計 */
.article {
  max-width: 40em;
}

.article p {
  line-height: 1.6;
  margin-block: 0 1em;
}

.article h2 {
  line-height: 1.3;
  margin-block: 1.5em 0.5em;
}
```

> **参照**
> - [Technique C36: Allowing for text spacing override](https://www.w3.org/WAI/WCAG22/Techniques/css/C36)
> - [Technique C21: Specifying line spacing in CSS](https://www.w3.org/WAI/WCAG22/Techniques/css/C21)
> - [Technique C28: Specifying the size of text containers using em units](https://www.w3.org/WAI/WCAG22/Techniques/css/C28)

## 折り返しなしでも対応する（C35）

- `white-space: nowrap` を使う場合でも、間隔変更に耐える設計が必要
- 省略表示（ellipsis）を使う場合は、**全文を読める手段**を提供すれば適合しうる
- フォーカスやクリックで全文表示、リンク先で全文表示など

```html
<!-- 条件付きで許容：省略 + 全文表示手段 -->
<a href="/messages/123" class="subject" title="【重要】契約更新に関する長いお知らせについて">
  【重要】契約更新に関する長いお知らせ...
</a>
<!-- リンク先で件名全文が読める -->
```

```css
/* 悪い例：省略のみで全文にアクセス不能 */
.subject-bad {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 10rem;
}

/* 良い例：省略 + title またはリンク先で全文 */
.subject-good {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 10rem;
}
```

> **参照**
> - [Technique C35: Allowing for text spacing without wrapping](https://www.w3.org/WAI/WCAG22/Techniques/css/C35)
> - [Understanding SC 1.4.12 — Use of ellipses](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html#use-of-ellipses)

## ボタン・ナビゲーション・フォーム

- ボタン内テキスト、ナビゲーション項目、フォームラベルも対象
- 固定高さのボタンで `line-height` 拡大時に文字が切れないようにする
- `padding` を `em` で指定し、文字サイズに追従させる

```css
/* 悪い例：固定高さボタンで文字が切れる */
.button-bad {
  height: 32px;
  line-height: 32px;
  overflow: hidden;
}

/* 良い例：padding で高さを確保 */
.button-good {
  padding: 0.75em 1em;
  line-height: 1.4;
  min-height: 2.5rem;
}
```

```html
<button type="button" class="button-good">申し込みを確定する</button>
```

## 字間の指定（C8）

- `letter-spacing` はデザイン上の調整に使われることがある
- 負の `letter-spacing` は、ユーザーが正の値に上書きしたとき問題を起こしやすい
- 必要最小限にとどめ、コンテナに余裕を持たせる

```css
/* 避けたい例：詰め字間 */
.tight-heading {
  letter-spacing: -0.05em;
}

/* 良い例：標準または正の字間 */
.heading-good {
  letter-spacing: 0.02em;
}
```

> **参照**
> - [Technique C8: Using CSS letter-spacing to control spacing within a word](https://www.w3.org/WAI/WCAG22/Techniques/css/C8)

## 1.4.4・1.4.8・1.4.10 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 1.4.4 テキストのサイズ変更 | 200% まで拡大 | サイズ変更でも内容が失われない |
| 1.4.8 視覚的提示 | 行幅・行間・配色の調整 | AAA、ユーザーが調整できる仕組み |
| 1.4.10 リフロー | 320 CSS px で2方向スクロールなし | 狭い幅でも読める |
| 1.4.12 テキストの間隔 | 行間・字間・語間・段落間の上書き | 間隔変更でも内容が失われない |

- 1.4.12 は**間隔の上書き**に特化
- 1.4.4 は**サイズ拡大**、1.4.8 は**読みやすさ設定全般**（AAA）
- 間隔変更でレイアウトが崩れると、1.4.4 や 1.4.10 の問題にもつながりやすい

> **参照**
> - [wcag/1-4-4-resize-text.md — 1.4.4 テキストのサイズ変更](./1-4-4-resize-text.md)
> - [wcag/1-4-8-visual-presentation.md — 1.4.8 視覚的提示](./1-4-8-visual-presentation.md)
> - [wcag/1-4-10-reflow.md — 1.4.10 リフロー](./1-4-10-reflow.md)

## よくある失敗例

- 見出しやカードタイトルが `height` 固定で、行間拡大時に下半分が切れる
- ナビゲーション項目が横に並び、字間拡大で隣の項目と重なる
- ボタン内テキストが `overflow: hidden` で隠れる
- `!important` で `line-height` や `letter-spacing` を固定し、ユーザー上書きを妨げる
- 省略表示のみで、間隔変更後の全文にアクセスできない
- 負の `letter-spacing` と固定レイアウトの組み合わせ

```html
<!-- 悪い例：3つの見出しが横並びで、字間拡大時に切れる -->
<div class="cards-row">
  <h3 class="card-heading-fixed">A cog in the wheel</h3>
  <h3 class="card-heading-fixed">A penny for your thoughts</h3>
  <h3 class="card-heading-fixed">Back to the drawing board</h3>
</div>
```

> **参照**
> - [Failure F104: clipped or overlapped content when text spacing is adjusted](https://www.w3.org/WAI/WCAG22/Techniques/failures/F104)

## テスト・確認方法

### 手動テスト

- ブックマークレットやユーザースタイルシートで4つの値を一括適用
- [Text Adaptation Bookmarklet](https://github.com/alastc/text-spacing) や Stylus 拡張を利用
- 次を確認
  - 文字が切れていないか
  - 要素同士が重なっていないか
  - ボタン・リンク・フォームが操作できるか
  - 省略テキストに全文アクセス手段があるか

```css
/* 手動テスト用スタイル */
* {
  line-height: 1.5 !important;
  letter-spacing: 0.12em !important;
  word-spacing: 0.16em !important;
}

p,
h1, h2, h3, h4, h5, h6,
li,
td,
th {
  margin-block-end: 2em !important;
}
```

### 開発者ツールでの確認

- `height` 固定、`overflow: hidden`、`line-height` と `height` の同値指定を重点確認
- 負の `letter-spacing`、負の `margin` の使用箇所を確認
- ボタン、見出し、ナビゲーション、カードタイトルを優先確認

```bash
# 間隔変更時に問題になりやすい指定を確認
rg "height:|overflow:\\s*hidden|letter-spacing:\\s*-|line-height:" --glob "*.css" --glob "*.scss"
```

### ACT Rules

- [Important line height in style attributes is wide enough](https://www.w3.org/WAI/standards-guidelines/act/rules/24afc2/)
- [Important letter spacing in style attributes is wide enough](https://www.w3.org/WAI/standards-guidelines/act/rules/9e762e/)
- [Important word spacing in style attributes is wide enough](https://www.w3.org/WAI/standards-guidelines/act/rules/9eb3f6/)

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)
> - [Text Adaptation Bookmarklet](https://github.com/alastc/text-spacing)

## まとめ

- 1.4.12 は、ユーザーが**行間・字間・語間・段落間**を上書きしても内容・機能が失われないことを求める達成基準（レベル AA）
- テスト値：行間 1.5 倍、段落後 2 倍、字間 0.12 倍、語間 0.16 倍（いずれも font-size 基準）
- 作者はこの値で表示する必要はなく、**上書きを妨げない**ことが重要
- 固定高さ、`overflow: hidden`、文字の重なり、省略のみで全文不可アクセスが主な失敗パターン
- 日本語では `word-spacing` や段落間隔は通常影響しにくいが、`line-height` と `letter-spacing` は確認が必要
- 1.4.4（拡大）・1.4.10（リフロー）と併せて確認すると効率的
