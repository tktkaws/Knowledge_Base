# 1.4.5 文字画像（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.5「文字画像」（Images of Text）
- レベル **AA**（標準的な適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- 使用している技術で同じ視覚表現を実現できる場合、情報は**文字画像ではなくテキスト**で提供
- 例外は、ユーザーが視覚的にカスタマイズ可能な場合、または特定の文字表現が不可欠な場合

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.5 Images of Text](https://www.w3.org/TR/WCAG22/#images-of-text)
> - [Understanding SC 1.4.5 Images of Text](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text.html)

## 達成基準の原文（要約）

- 使用している技術で視覚的提示を実現できるなら、情報の伝達には**文字画像ではなくテキスト**を使うこと
- 例外
  - **カスタマイズ可能**：文字画像のフォント・サイズ・色・背景をユーザーが変更できる
  - **不可欠**：その文字の特定の提示が、伝えたい情報に不可欠
- ロゴタイプ（ロゴやブランド名の一部である文字）は不可欠とみなされる

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.5](https://www.w3.org/TR/WCAG22/#images-of-text)

## 文字画像とは

- テキストが画像ピクセルとして描画されているもの
- 人間の言語を表す文字列が、`img`、背景画像、SVG画像、Canvas、PDFスキャン等として表示される状態
- 例
  - 見出し画像
  - ナビゲーション画像ボタン
  - バナー内のキャンペーン文言
  - 画像化された引用文
  - スキャンPDF内の文字

```html
<!-- 文字画像：文字が画像ファイル内に埋め込まれている -->
<img src="campaign-heading.png" alt="夏のキャンペーン">

<!-- 実テキスト：文字がHTMLテキストとして存在する -->
<h1 class="campaign-heading">夏のキャンペーン</h1>
```

> **参照**
> - [Understanding SC 1.4.5 — image of text](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text.html#dfn-image-of-text)

## 対象外：重要な他の視覚コンテンツを含む画像内の文字

- 文字画像の定義には「重要な他の視覚コンテンツを含む画像内の文字」は含まれない
- 例
  - スクリーンショット内のUI文字
  - グラフ・図解内のラベル
  - 写真に写り込んだ看板
  - 地図内の地名
- ただし、その文字を読ませることが主目的なら、代替テキスト・本文で情報を提供

```html
<!-- 対象外になりうる：UI全体のスクリーンショット -->
<img
  src="dashboard-screenshot.png"
  alt="売上ダッシュボードのスクリーンショット。折れ線グラフと売上カードが表示されている"
>

<!-- 対象：文字そのものが主情報のバナー -->
<img src="sale-banner.png" alt="全品30%OFF 7月31日まで">
<!-- 可能なら実テキスト + CSSで実装する -->
```

> **参照**
> - [Understanding SC 1.4.5 — definition note](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text.html#dfn-image-of-text)

## なぜ必要か

- ロービジョンのユーザーが、文字サイズ・フォント・色・背景を変更できるようにするため
- 文字画像は拡大するとピクセル化しやすく、ブラウザの文字サイズ設定に追従しにくい
- ユーザーのスタイルシートや高コントラスト表示で色を変更しにくい
- 翻訳、検索、コピー、読み上げ、行間調整などのテキスト操作が難しい
- 認知障害・読字障害のあるユーザーが、自分に合った書体や間隔で読めるようにするため

> **参照**
> - [Understanding SC 1.4.5 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text.html#intent)
> - [Understanding SC 1.4.5 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text.html#benefits)

## 基本方針：CSSで見た目を作る（C22）

- 見出し、ボタン、ナビゲーション、引用文などは実テキスト + CSSで表現
- Webフォント、`font-weight`、`letter-spacing`、`text-shadow`、`transform` 等を活用
- 画像化する前に、CSSで再現できるか検討

```html
<!-- 悪い例：見出しを画像化 -->
<img src="heading-special-font.png" alt="サービス紹介">

<!-- 良い例：テキスト + CSS -->
<h2 class="section-heading">サービス紹介</h2>
```

```css
.section-heading {
  font-family: "Noto Serif JP", serif;
  font-size: clamp(1.75rem, 4vw, 3rem);
  font-weight: 700;
  letter-spacing: 0.08em;
  color: #1a1a1a;
  text-shadow: 0 2px 0 #f2d16b;
}
```

> **参照**
> - [Technique C22: Using CSS to control visual presentation of text](https://www.w3.org/WAI/WCAG22/Techniques/css/C22)
> - [MDN — Fundamental text and font styling](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Fundamentals)

## ナビゲーション・ボタン

- メニューやボタンのラベルを画像化しない
- アイコンは画像でもよいが、ラベルは実テキストにする
- アイコンだけの操作は accessible name を付ける

```html
<!-- 悪い例：ナビゲーション文字を画像化 -->
<nav aria-label="メインナビゲーション">
  <a href="/about"><img src="nav-about.png" alt="会社概要"></a>
  <a href="/contact"><img src="nav-contact.png" alt="お問い合わせ"></a>
</nav>

<!-- 良い例：テキスト + CSS -->
<nav aria-label="メインナビゲーション">
  <a class="nav-link" href="/about">会社概要</a>
  <a class="nav-link" href="/contact">お問い合わせ</a>
</nav>
```

```html
<!-- 良い例：アイコン + 実テキスト -->
<button type="button" class="cart-button">
  <img src="cart.svg" alt="" aria-hidden="true">
  カートに追加
</button>
```

> **参照**
> - [wcag/1-3-6-identify-purpose.md — UI コンポーネントの目的](./1-3-6-identify-purpose.md)

## バナー・キャンペーン画像

- セール情報や告知文を画像内だけに置かない
- 画像を使う場合も、同じ情報を実テキストとして近くに提供
- デザイン表現は背景画像・装飾画像として分離し、情報はHTMLテキストにする

```html
<!-- 悪い例：キャンペーン情報が画像内だけ -->
<img src="summer-sale.png" alt="夏のセール 最大50%OFF 8月31日まで">

<!-- 良い例：装飾画像 + 実テキスト -->
<section class="campaign">
  <img src="summer-decoration.png" alt="" aria-hidden="true">
  <h2>夏のセール</h2>
  <p>最大50%OFF。8月31日まで。</p>
  <a href="/sale">対象商品を見る</a>
</section>
```

```css
.campaign {
  background-image: linear-gradient(135deg, #ffe9a8, #ff8a65);
  padding: 2rem;
}

.campaign h2 {
  font-size: clamp(2rem, 5vw, 4rem);
}
```

> **参照**
> - [Technique G140: Separating information and structure from presentation](https://www.w3.org/WAI/WCAG22/Techniques/general/G140)

## 例外：カスタマイズ可能な文字画像

- 文字画像であっても、ユーザーが次を変更できるなら例外
  - フォント
  - サイズ
  - 前景色
  - 背景色
- サーバー側でユーザー設定に合わせて画像を生成する場合など
- ただし、同じ情報を実テキストでも提供する方が望ましい

```html
<!-- 例外になりうる：ユーザー設定で生成画像の見た目を変更できる -->
<form action="/render-quote" method="get">
  <label for="font-size">文字サイズ</label>
  <select id="font-size" name="fontSize">
    <option value="16">標準</option>
    <option value="24">大きい</option>
  </select>

  <label for="theme">配色</label>
  <select id="theme" name="theme">
    <option value="light">明るい背景</option>
    <option value="dark">暗い背景</option>
  </select>

  <button type="submit">画像を再生成</button>
</form>

<img src="/render-quote?fontSize=24&theme=dark" alt="引用文の内容">
```

> **参照**
> - [Understanding SC 1.4.5 — Customizable](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text.html#intent)

## 例外：不可欠な文字表現

- その特定の見た目自体が情報であり、置き換えると意味や機能が根本的に変わる場合
- 例
  - ロゴタイプ（ロゴやブランド名）
  - フォント見本
  - 歴史的な手紙・古文書の原本画像
  - 書道・タイポグラフィ作品
  - 特定の文字形を説明する教材

```html
<!-- 例外：ロゴタイプ -->
<a href="/" aria-label="ホーム">
  <img src="brand-logo.svg" alt="Example Corp">
</a>

<!-- 例外：フォント見本 -->
<figure>
  <img src="font-sample-mincho.png" alt="明朝体の見本文字「あいうえお」">
  <figcaption>明朝体の書体見本</figcaption>
</figure>

<!-- 例外：原本の見た目が不可欠な手紙 -->
<figure>
  <img src="historical-letter.jpg" alt="明治時代の手紙の原本画像">
  <figcaption>原本の筆跡・紙質・配置を示す資料</figcaption>
</figure>
```

> **参照**
> - [Understanding SC 1.4.5 — Essential](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text.html#intent)

## 画像とテキストを両方提示する場合

- 文字画像が装飾的に使われ、同じ情報が実テキストでも提示されている場合は適合
- 1.4.5 は、文字画像が**テキストの代わり**に使われることを避ける基準
- 1.4.9「文字画像（例外なし）」では、この扱いがさらに厳しくなる

```html
<!-- 良い例：ポスター画像 + 同じ内容を実テキストで提供 -->
<article class="event">
  <img src="event-poster.jpg" alt="イベントポスター">
  <h2>アクセシビリティ勉強会</h2>
  <p>日時：2026年7月20日 19:00〜21:00</p>
  <p>会場：オンライン</p>
  <a href="/events/accessibility-study">詳細・申し込み</a>
</article>
```

> **参照**
> - [Understanding SC 1.4.5 — images of text used in addition to text](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text.html#intent)
> - [WCAG 2.2 — Success Criterion 1.4.9 Images of Text (No Exception)](https://www.w3.org/TR/WCAG22/#images-of-text-no-exception)

## スキャンPDF・OCR（PDF7）

- スキャンPDFは、文字が画像として保存されている場合が多い
- OCRを実行し、実テキストをPDF内に埋め込む
- 見た目の原本画像を残す場合でも、検索・選択・読み上げ可能なテキスト層を提供

```text
悪い例：
  スキャン画像だけのPDF

良い例：
  OCR済みPDF（画像 + テキスト層）
  または HTML本文として同じ内容を提供
```

> **参照**
> - [Technique PDF7: Performing OCR on a scanned PDF document](https://www.w3.org/WAI/WCAG22/Techniques/pdf/PDF7)

## SVG内の文字

- SVGでも、`<text>` 要素で実テキストとして保持できる場合は画像化しない
- アウトライン化された文字（`<path>`）は文字画像と同様に扱われやすい
- 装飾目的の見た目はCSSやSVG属性で表現し、文字情報はテキストとして残す

```svg
<!-- 悪い例：文字をpath化（内容をテキストとして扱いにくい） -->
<svg viewBox="0 0 300 80" aria-label="SALE">
  <path d="..." />
</svg>

<!-- 良い例：text要素として保持 -->
<svg viewBox="0 0 300 80" role="img" aria-label="SALE">
  <text x="20" y="55" font-size="48" font-weight="700">SALE</text>
</svg>
```

- SVGのアクセシビリティ対応はユーザーエージェント差があるため、重要なテキストはHTMLにも提供すると安全

## CSS背景画像内の文字

- CSS背景画像に重要な文字を含めると、代替テキストを付けられない
- 1.1.1・1.4.5・1.4.3の観点で問題になりやすい
- 背景は装飾、情報はHTMLテキストに分ける

```css
/* 悪い例：背景画像内に「50%OFF」の文字 */
.hero-bad {
  background-image: url("sale-50off-text.png");
}

/* 良い例：背景は装飾のみ */
.hero-good {
  background-image: url("sale-decoration.png");
}
```

```html
<section class="hero-good">
  <h1>50%OFF セール</h1>
  <p>7月31日まで</p>
</section>
```

> **参照**
> - [wcag/1-1-1-non-text-content.md — CSS背景画像と1.1.1](./1-1-1-non-text-content.md)

## 1.4.3・1.4.4との関係

| 達成基準 | 対象 | 関係 |
|---|---|---|
| 1.4.3 コントラスト（最低限） | テキスト・文字画像 | 文字画像にもコントラスト要件 |
| 1.4.4 テキストのサイズ変更 | テキスト | 文字画像は対象外だが、拡大に弱い |
| 1.4.5 文字画像 | 情報を伝える文字画像 | 可能なら実テキストにする |
| 1.4.9 文字画像（例外なし） | 文字画像 | AAAでさらに厳しい |

- 1.4.5 は**文字画像を避ける**基準
- どうしても文字画像を使う場合でも、1.4.3 のコントラストや 1.1.1 の代替テキストは必要

> **参照**
> - [wcag/1-4-3-contrast-minimum.md — 1.4.3 コントラスト（最低限）](./1-4-3-contrast-minimum.md)
> - [wcag/1-4-4-resize-text.md — 1.4.4 テキストのサイズ変更](./1-4-4-resize-text.md)

## よくある失敗例

- 見出しを画像にしているが、CSSで同じ見た目を実現できる
- ボタンやナビゲーションのラベルを画像化している
- キャンペーンバナーの情報が画像内だけ
- 文字をSVG pathにアウトライン化している
- スキャンPDFにOCRテキスト層がない
- 画像内の文字に alt はあるが、ユーザーが文字サイズ・色・行間を変更できない

```html
<!-- 悪い例：altがあっても1.4.5としては不十分になりうる -->
<img src="heading.png" alt="お問い合わせ">
<!-- 見た目がCSSで実現できるなら、実テキストにする -->

<!-- 良い例 -->
<h1 class="contact-heading">お問い合わせ</h1>
```

## テスト・確認方法

### 自動テスト

- ACT Rule：[HTML images contain no text](https://www.w3.org/WAI/standards-guidelines/act/rules/0va7u6/proposed/)
- OCRや画像解析を伴うため、自動検出には限界がある
- Lighthouse / axe だけではすべての文字画像を検出できない

### 手動テスト

- ページ内の画像・背景画像・SVG・Canvas・PDFに文字が含まれていないか確認
- その文字表現を CSS + 実テキストで再現できるか判断
- 文字画像が不可欠か、またはカスタマイズ可能か確認
- 画像内文字と同じ情報が実テキストでも表示されているか確認
- 200%拡大・高コントラスト表示・テキスト選択で問題がないか確認

```bash
# 画像要素・背景画像・SVGをざっくり確認
rg "<img|background-image|<svg|canvas" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.md"
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.5 は、情報を伝える文字を**画像ではなく実テキスト**で提供する達成基準（レベル AA）
- CSSで同じ見た目を実現できるなら、見出し・ボタン・ナビゲーション・バナー文字は実テキストにする
- 例外は、ユーザーが視覚的にカスタマイズできる文字画像、または特定の文字表現が不可欠な場合
- ロゴタイプ、フォント見本、原本資料、書道作品などは不可欠になりうる
- 画像と同じ情報を実テキストでも提示すれば、1.4.5では適合しやすい
- 文字画像を使う場合でも、1.1.1（代替テキスト）と1.4.3（コントラスト）は別途必要
