# 1.4.8 視覚的提示（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.8「視覚的提示」（Visual Presentation）
- レベル **AAA**（最高の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- **テキストブロック**の視覚的提示について、ユーザーが読みやすく調整できる仕組みを提供
- 色、行幅、両端揃え、行間・段落間、200%拡大時の横スクロール回避を扱う

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.8 Visual Presentation](https://www.w3.org/TR/WCAG22/#visual-presentation)
> - [Understanding SC 1.4.8 Visual Presentation](https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation.html)

## 達成基準の原文（要約）

- テキストブロックの視覚的提示について、次を達成できる**仕組み**があること
  - 前景色と背景色をユーザーが選択できる
  - 幅が 80 文字（CJK は 40 字）を超えない
  - テキストが両端揃えではない
  - 段落内の行間が 1.5 以上、段落間が行間の 1.5 倍以上
  - 支援技術なしで 200% までテキストを拡大しても、全画面ウィンドウで1行を読むための横スクロールが不要
- コンテンツが最初からこの値で表示される必要はなく、**ユーザーが変更できる仕組み**があればよい

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.8](https://www.w3.org/TR/WCAG22/#visual-presentation)

## 対象：テキストブロック

- テキストブロック：**2文以上**のまとまったテキスト
- 具体例
  - 記事本文
  - 説明文
  - 利用規約
  - ヘルプ・ドキュメント
  - 長いカード本文
- 単一のボタンラベル、短い見出し、1文だけの通知文は主対象ではない

```html
<!-- 対象：複数文の本文ブロック -->
<article class="content">
  <p>アクセシビリティは、誰もが情報にアクセスできるようにする考え方です。Webでは、支援技術や多様な表示環境への対応が重要です。</p>
  <p>テキストの幅や行間を調整できると、読みやすさが向上します。</p>
</article>
```

> **参照**
> - [Understanding SC 1.4.8 — blocks of text](https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation.html#dfn-blocks-of-text)

## なぜ必要か

- ロービジョンのユーザーが、自分に合う文字色・背景色・サイズで読めるようにするため
- 認知障害・学習障害・読字障害のあるユーザーが、行を追いやすくするため
- 長すぎる行は、次の行に移るときに読み位置を失いやすい
- 両端揃えは単語間隔が不規則になり、「白い川」のような空白ができて読みにくくなる
- 行間・段落間が狭いと、行や段落の区切りを追いにくい

> **参照**
> - [Understanding SC 1.4.8 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation.html#intent)
> - [Understanding SC 1.4.8 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation.html#benefits)

## 重要：仕組みがあればよい

- ページの初期表示が必ずすべての値を満たす必要はない
- ユーザーエージェント（ブラウザ等）やページ内 UI で変更できればよい
- 作者は、ユーザーの変更を**妨げない**設計にする
- ページ側で独自の表示切替を提供してもよい

```html
<!-- 例：読みやすさ設定パネル -->
<section aria-labelledby="reading-settings-heading">
  <h2 id="reading-settings-heading">読みやすさ設定</h2>
  <button type="button" id="theme-toggle">高コントラスト配色</button>
  <button type="button" id="line-spacing-toggle">行間を広げる</button>
  <button type="button" id="narrow-width-toggle">行幅を狭くする</button>
</section>
```

> **参照**
> - [Understanding SC 1.4.8 — Note 1](https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation.html#success-criterion)

## 要件1：前景色と背景色を選択できる

- ユーザーが文字色と背景色を選べる仕組みが必要
- ブラウザ・ユーザースタイルシート・拡張機能に任せてもよい
- 作者は、色変更を妨げるような実装を避ける
- 本文の色を固定しすぎず、ユーザーの配色変更に対応しやすくする

```css
/* 良い例：本文はユーザーエージェントやテーマ変更に追従しやすい */
body {
  color: CanvasText;
  background-color: Canvas;
}

/* 良い例：CSSカスタムプロパティで配色を切り替えやすくする */
:root {
  --text-color: #1a1a1a;
  --bg-color: #ffffff;
}

body {
  color: var(--text-color);
  background-color: var(--bg-color);
}

body.high-contrast {
  --text-color: #000000;
  --bg-color: #ffffff;
}
```

```html
<!-- 悪い例：背景画像上の本文など、色変更しにくい設計 -->
<div class="text-as-background-image"></div>
```

> **参照**
> - [Technique G156: User agents can change foreground and background of text blocks](https://www.w3.org/WAI/WCAG22/Techniques/general/G156)
> - [Technique G175: Multi color selection tool](https://www.w3.org/WAI/WCAG22/Techniques/general/G175)

## 要件2：行幅は80文字以下（CJKは40字以下）

- 英文などの非CJK：1行 **80文字** 以下
- 日本語・中国語・韓国語など CJK：1行 **40字** 以下
- 長すぎる行は、読み位置を見失いやすい
- `max-width` を `ch` などで制限すると管理しやすい

```css
/* 悪い例：全幅に広がる長い本文 */
.article-bad {
  width: 100%;
}

/* 良い例：CJK本文は40字程度に抑える */
.article-good {
  max-width: 40em;
  margin-inline: auto;
}

/* 英文中心なら80ch程度 */
.article-en {
  max-width: 80ch;
  margin-inline: auto;
}
```

```html
<article class="article-good">
  <p>日本語本文は行幅が広すぎると、次の行に移るときに読み位置を見失いやすくなります。</p>
</article>
```

> **参照**
> - [Technique C20: Relative measurements for column widths](https://www.w3.org/WAI/WCAG22/Techniques/css/C20)
> - [Technique G204: Not interfering with reflow as window narrows](https://www.w3.org/WAI/WCAG22/Techniques/general/G204)

## 要件3：両端揃えにしない

- `text-align: justify` は左右両端に揃えるため、単語間隔が不規則になりやすい
- 読字障害・認知障害のあるユーザーにとって、単語境界や行の追跡が難しくなる
- 左揃え、または言語・書字方向に応じた片側揃えを使う

```css
/* 悪い例：両端揃え（F88） */
.article-bad {
  text-align: justify;
}

/* 良い例：日本語・左横書きでは左揃え */
.article-good {
  text-align: start;
}
```

```html
<!-- 悪い例：本文が両端揃え -->
<article class="article-bad">
  <p>本文が両端揃えになると、行ごとの空白が不規則になり、読みづらくなる場合があります。</p>
</article>
```

> **参照**
> - [Failure F88: Using text that is justified](https://www.w3.org/WAI/WCAG22/Techniques/failures/F88)
> - [Technique G169: Aligning text on only one side](https://www.w3.org/WAI/WCAG22/Techniques/general/G169)
> - [Technique C19: Specifying alignment either to the left or right in CSS](https://www.w3.org/WAI/WCAG22/Techniques/css/C19)

## 要件4：行間と段落間を確保する

- 段落内の行間：**1.5以上**
- 段落間：行間の **1.5倍以上**
- 単位なし `line-height` を使うと、文字サイズ変更に追従しやすい

```css
/* 悪い例：行間・段落間が狭い */
.content-bad {
  line-height: 1.1;
}

.content-bad p {
  margin-block: 0.25em;
}

/* 良い例：行間1.5以上、段落間も広め */
.content-good {
  line-height: 1.6;
}

.content-good p {
  margin-block: 0 1.6em;
}
```

```html
<article class="content-good">
  <p>行間が十分にあると、次の行を追いやすくなります。</p>
  <p>段落間が十分にあると、段落の区切りを把握しやすくなります。</p>
</article>
```

> **参照**
> - [Technique C21: Specifying line spacing in CSS](https://www.w3.org/WAI/WCAG22/Techniques/css/C21)
> - [Technique G188: Button to increase line and paragraph spacing](https://www.w3.org/WAI/WCAG22/Techniques/general/G188)

## 要件5：200%拡大時に1行を読むための横スクロールが不要

- 支援技術なしでテキストを 200% まで拡大できる
- 全画面ウィンドウで、1行を読むために左右へ行ったり来たりする必要がない
- リキッドレイアウト・相対単位・折り返しで対応
- 1.4.4 と似ているが、1.4.8 では**横スクロールなしで読む**点が明確

```css
/* 悪い例：固定幅で200%拡大時に横スクロールが必要 */
.content-bad {
  width: 960px;
}

/* 良い例：画面幅に合わせて流動 */
.content-good {
  width: min(100%, 40em);
  margin-inline: auto;
  padding-inline: 1rem;
}

.content-good p {
  overflow-wrap: anywhere;
}
```

> **参照**
> - [Technique G146: Using liquid layout](https://www.w3.org/WAI/WCAG22/Techniques/general/G146)
> - [Technique C24: Using percentage values in CSS for container sizes](https://www.w3.org/WAI/WCAG22/Techniques/css/C24)
> - [wcag/1-4-4-resize-text.md — 1.4.4 テキストのサイズ変更](./1-4-4-resize-text.md)

## 横スクロールの補足

- 1.4.8 は「横スクロールが一切発生してはならない」という意味ではない
- 1行を読むために、左右へ何度もスクロールする必要がないことが重要
- 複数カラムの場合、各カラム単位で読めればよい場合がある
- 非常に長いURLや長い単語は、通常の文章として読むテキストとは扱いが異なる場合がある

```html
<!-- 許容されうる例：長いURLは折り返し・省略・リンクテキストで扱う -->
<p>
  詳細は
  <a href="https://example.com/very/long/path/that/may/not/wrap">
    詳細ページ
  </a>
  を参照してください。
</p>
```

> **参照**
> - [Understanding SC 1.4.8 — horizontal scrolling](https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation.html#intent)

## ページ内で仕組みを提供する例

- ブラウザやユーザースタイルシートに任せてもよい
- サイト側で読みやすさ設定を提供すると、ユーザーが見つけやすい
- 設定状態は `localStorage` 等で保存してもよい

```html
<section class="reading-controls" aria-labelledby="reading-controls-heading">
  <h2 id="reading-controls-heading">読みやすさ設定</h2>
  <button type="button" data-reading="narrow">行幅を狭くする</button>
  <button type="button" data-reading="spacing">行間を広げる</button>
  <button type="button" data-reading="contrast">高コントラスト</button>
</section>

<article class="article" id="article">
  <p>本文がここに入ります。ユーザーは読みやすさ設定で表示を調整できます。</p>
</article>
```

```css
.article {
  max-width: 40em;
  line-height: 1.6;
  text-align: start;
}

.article.is-wide {
  max-width: none;
}

.article.is-spaced {
  line-height: 2;
}

body.is-high-contrast {
  color: #000000;
  background-color: #ffffff;
}
```

```javascript
document.querySelectorAll('[data-reading]').forEach((button) => {
  button.addEventListener('click', () => {
    const value = button.dataset.reading;
    if (value === 'narrow') {
      document.getElementById('article').classList.toggle('is-wide');
    }
    if (value === 'spacing') {
      document.getElementById('article').classList.toggle('is-spaced');
    }
    if (value === 'contrast') {
      document.body.classList.toggle('is-high-contrast');
    }
  });
});
```

## 書字体系による違い

- 言語・書字体系によって読みやすい提示方法は異なる
- ある書字体系で使われない提示項目は、その設定を使う必要はない
- 日本語のようなCJKでは、行幅上限は 40 字が目安
- 縦書きや特定の組版では、別の読みやすさ配慮が必要になる場合がある

> **参照**
> - [Understanding SC 1.4.8 — Note 2](https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation.html#success-criterion)

## 1.4.4・1.4.10・1.4.12 との関係

| 達成基準 | レベル | 主な焦点 |
|---|---|---|
| 1.4.4 テキストのサイズ変更 | AA | 200%拡大で内容・機能が失われない |
| 1.4.8 視覚的提示 | AAA | 色・行幅・揃え・行間・200%時の横スクロール |
| 1.4.10 リフロー | AA | 320 CSS px相当で2方向スクロールを避ける |
| 1.4.12 テキストの間隔 | AA | ユーザー指定の行間・字間等で内容が失われない |

- 1.4.8 は、読みやすい**表示の選択肢**を広く扱う
- 1.4.12 は、ユーザーが特定のテキスト間隔を適用しても破綻しないことを扱う
- 1.4.10 は、ズーム時のリフローをより具体的に扱う

> **参照**
> - [wcag/1-4-4-resize-text.md — 1.4.4 テキストのサイズ変更](./1-4-4-resize-text.md)
> - [WCAG 2.2 — Success Criterion 1.4.10 Reflow](https://www.w3.org/TR/WCAG22/#reflow)
> - [WCAG 2.2 — Success Criterion 1.4.12 Text Spacing](https://www.w3.org/TR/WCAG22/#text-spacing)

## よくある失敗例

| Failure | 内容 |
|---|---|
| F24 | 前景色だけ、または背景色だけを指定し、ユーザーが色を選べない・評価できない |
| F88 | 本文テキストを両端揃えにしている |

```css
/* F24：文字色だけ指定 */
.content {
  color: #333333;
  /* background-color 未指定 */
}

/* F88：両端揃え */
.article {
  text-align: justify;
}
```

> **参照**
> - [Failure F24: Specifying foreground colors without specifying background colors or vice versa](https://www.w3.org/WAI/WCAG22/Techniques/failures/F24)
> - [Failure F88: Using text that is justified](https://www.w3.org/WAI/WCAG22/Techniques/failures/F88)

## テスト・確認方法

### 手動テスト

- 本文ブロックの行幅が 80 文字（CJK 40 字）程度を超えないか確認
- 本文が両端揃えになっていないか確認
- 行間が 1.5 以上、段落間が十分にあるか確認
- 200%ズームで1行を読むために横スクロールが必要ないか確認
- ユーザーが前景色・背景色を変更できる、または変更を妨げていないか確認

### 開発者ツールでの確認

- `text-align: justify` を検索
- `max-width` が本文ブロックに設定されているか確認
- `line-height` と段落マージンを確認
- 固定幅・固定高さ・`overflow: hidden` が本文の調整を妨げていないか確認

```bash
# 視覚的提示に関係するCSSを確認
rg "text-align|line-height|max-width|width:|overflow" --glob "*.css" --glob "*.scss"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.8 は、テキストブロックの視覚的提示をユーザーが読みやすく調整できるようにする達成基準（レベル AAA）
- 前景色・背景色を選べる仕組み、または選択を妨げない実装が必要
- 行幅は 80 文字以内、CJKでは 40 字以内が目安
- 両端揃えを避け、片側揃えにする
- 行間は 1.5 以上、段落間は行間の 1.5 倍以上を確保
- 200%拡大時、1行を読むための横スクロールを避ける
- 1.4.4、1.4.10、1.4.12 と併せて確認する
