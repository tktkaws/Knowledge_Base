# 1.4.9 文字画像（例外なし）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.9「文字画像（例外なし）」（Images of Text (No Exception)）
- レベル **AAA**（最高の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- 文字画像は**純粋な装飾**、または**情報の伝達に不可欠な特定の文字表現**の場合にのみ使用
- 1.4.5 より厳しく、**カスタマイズ可能な文字画像**の例外は認めない

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.9 Images of Text (No Exception)](https://www.w3.org/TR/WCAG22/#images-of-text-no-exception)
> - [Understanding SC 1.4.9 Images of Text (No Exception)](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text-no-exception.html)

## 達成基準の原文（要約）

- 文字画像は、**純粋な装飾**として使う場合、または**情報の伝達に不可欠な特定の文字表現**として使う場合にのみ使用すること
- 注記：ロゴタイプ（ロゴやブランド名の一部である文字）は不可欠とみなされる

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.9](https://www.w3.org/TR/WCAG22/#images-of-text-no-exception)

## 1.4.5 との違い

| 達成基準 | レベル | 基本方針 | 主な例外 |
|---|---|---|---|
| 1.4.5 文字画像 | AA | 技術で再現できるなら実テキストを使う | カスタマイズ可能、不可欠 |
| 1.4.9 文字画像（例外なし） | AAA | 文字画像は装飾か不可欠な場合のみ | **不可欠のみ**（カスタマイズ可能は例外にならない） |

- 1.4.9 は 1.4.5 の**上位基準**
- 1.4.5 では「ユーザーがフォント・サイズ・色・背景を変更できる文字画像」は例外だったが、1.4.9 では**例外にならない**
- 1.4.5 では同じ情報を実テキストでも提示すれば適合しやすいが、1.4.9 では文字画像自体が**装飾か不可欠**でない限り不適合
- 1.4.9 を満たせば 1.4.5 も満たす

```html
<!-- 1.4.5 では適合しうるが、1.4.9 では不適合 -->
<!-- ユーザー設定で画像を再生成できるが、文字画像自体は残る -->
<form action="/render-heading" method="get">
  <label for="font-size">文字サイズ</label>
  <select id="font-size" name="fontSize">
    <option value="16">標準</option>
    <option value="24">大きい</option>
  </select>
  <button type="submit">見出しを再生成</button>
</form>
<img src="/render-heading?fontSize=24" alt="サービス紹介">

<!-- 1.4.9 に適合：実テキスト + CSS -->
<h2 class="section-heading">サービス紹介</h2>
```

> **参照**
> - [wcag/1-4-5-images-of-text.md — 1.4.5 文字画像](./1-4-5-images-of-text.md)

## 文字画像とは

- テキストが画像ピクセルとして描画されているもの
- 人間の言語を表す文字列が、`img`、背景画像、SVG path、Canvas、PDFスキャン等として表示される状態
- 「重要な他の視覚コンテンツを含む画像内の文字」は文字画像の定義に含まれない
  - スクリーンショット内のUI文字
  - グラフ・図解内のラベル
  - 写真に写り込んだ看板

```html
<!-- 文字画像：情報を伝える文字が画像内に埋め込まれている -->
<img src="campaign-heading.png" alt="夏のキャンペーン">

<!-- 実テキスト：文字がHTMLテキストとして存在する -->
<h1 class="campaign-heading">夏のキャンペーン</h1>
```

> **参照**
> - [Understanding SC 1.4.9 — image of text](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text-no-exception.html#dfn-image-of-text)

## なぜ必要か

- 特定の視覚提示が必要なユーザーが、文字の見た目を自分に合わせて調整できるようにするため
- 文字画像内の文字は、フォント・サイズ・色・行間・配置をユーザー側で変更できない
- ロービジョンのユーザーが、読みやすい書体・サイズ・配色で文字を表示できるようにするため
- 視線追跡に困難があるユーザーが、行間・配置を調整して読めるようにするため
- 認知障害・読字障害のあるユーザーが、自分に合った文字提示で読めるようにするため

> **参照**
> - [Understanding SC 1.4.9 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text-no-exception.html#intent)
> - [Understanding SC 1.4.9 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text-no-exception.html#benefits)

## 基本方針：実テキスト + CSS（C22）

- 見出し、ボタン、ナビゲーション、引用文、バナー文言は実テキスト + CSSで表現
- Webフォント、`font-weight`、`letter-spacing`、`text-shadow`、`transform` 等を活用
- 1.4.5 と同様の実装方針だが、**文字画像へのフォールバック自体を避ける**

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

## 純粋な装飾としての文字画像

- **純粋な装飾**（pure decoration）：美的目的のみで、情報も機能も持たない
- 文字が純粋な装飾かどうかは、語句を入れ替えても目的が変わらないかで判断
- 装飾的な文字画像は、同じ情報を別の実テキストで提供する必要はない
- ただし、装飾画像でも alt は空にするか `aria-hidden="true"` で支援技術から隠す

```html
<!-- 良い例：装飾的な背景文字（情報を伝えない） -->
<section class="hero" aria-hidden="true">
  <img src="decorative-watermark.png" alt="">
</section>

<!-- 悪い例：キャンペーン情報を含む「装飾」と称するバナー -->
<img src="sale-50off.png" alt="50%OFF 7月31日まで">
<!-- 情報を伝えているため、純粋な装飾ではない -->
```

> **参照**
> - [Understanding SC 1.4.9 — pure decoration](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text-no-exception.html#dfn-pure-decoration)

## 不可欠な文字表現

- その特定の視覚提示自体が情報であり、置き換えると意味や機能が根本的に変わる場合
- ロゴタイプ（ロゴやブランド名の一部である文字）は**不可欠とみなされる**
- 例
  - ロゴタイプ
  - フォント見本・書体サンプル
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
> - [Understanding SC 1.4.9 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text-no-exception.html#examples)

## 1.4.5 で許されていたが 1.4.9 では認められない例

### カスタマイズ可能な文字画像

- 1.4.5 では、ユーザーがフォント・サイズ・色・背景を変更できる文字画像は例外
- 1.4.9 では**この例外は存在しない**
- サーバー側で設定に応じて画像を生成しても、文字画像の使用自体が問題

```html
<!-- 1.4.5 適合・1.4.9 不適合：カスタマイズ可能な引用文画像 -->
<form action="/render-quote" method="get">
  <label for="theme">配色</label>
  <select id="theme" name="theme">
    <option value="light">明るい背景</option>
    <option value="dark">暗い背景</option>
  </select>
  <button type="submit">画像を再生成</button>
</form>
<img src="/render-quote?theme=dark" alt="引用文の内容">

<!-- 1.4.9 適合：CSS + ユーザー設定で実テキストの見た目を変更 -->
<blockquote class="quote" data-theme="dark">
  <p>アクセシビリティは選択肢ではなく、基本です。</p>
  <footer>— W3C WAI</footer>
</blockquote>
```

### 実テキストと併用する文字画像

- 1.4.5 では、同じ情報を実テキストでも提示すれば適合しやすい
- 1.4.9 では、文字画像が**純粋な装飾**でない限り、実テキストが併存しても不適合
- ポスター画像にイベント情報が含まれる場合、画像から文字を除き、HTMLテキストで情報を提供

```html
<!-- 1.4.5 適合・1.4.9 不適合：ポスター画像内にイベント情報 -->
<article class="event">
  <img src="event-poster.jpg" alt="アクセシビリティ勉強会 7月20日 19:00">
  <h2>アクセシビリティ勉強会</h2>
  <p>日時：2026年7月20日 19:00〜21:00</p>
</article>

<!-- 1.4.9 適合：装飾画像 + 実テキスト -->
<article class="event">
  <img src="event-decoration.jpg" alt="">
  <h2>アクセシビリティ勉強会</h2>
  <p>日時：2026年7月20日 19:00〜21:00</p>
  <p>会場：オンライン</p>
  <a href="/events/accessibility-study">詳細・申し込み</a>
</article>
```

> **参照**
> - [Understanding SC 1.4.5 — images of text used in addition to text](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text.html#intent)

## テキストと画像の切り替え（C30）

- CSSで文字画像に置き換えつつ、ユーザーが実テキスト表示に切り替えられるUIを提供する手法
- 切り替え後は文字画像ではなく実テキストが表示されるため、1.4.9 を満たしうる
- デフォルトが文字画像のままでは不適合

```html
<!-- 良い例：切り替えコントロール付き -->
<div class="quote-block" data-mode="text">
  <button type="button" id="toggle-presentation" aria-pressed="true">
    装飾表示を切り替え
  </button>
  <blockquote class="quote-text">
    <p>アクセシビリティは選択肢ではなく、基本です。</p>
  </blockquote>
</div>
```

```css
/* テキストモード（デフォルト）：実テキストを表示 */
.quote-block[data-mode="text"] .quote-text {
  font-family: "Noto Serif JP", serif;
  font-style: italic;
  padding-left: 2em;
}

/* 画像モード：CSSで文字画像に置き換え（ユーザーが選択した場合のみ） */
.quote-block[data-mode="image"] .quote-text {
  background-image: url("quote-decorative.png");
  color: transparent;
  /* 切り替えUIでユーザーが明示的に選択 */
}
```

> **参照**
> - [Technique C30: Using CSS to replace text with images of text and providing user interface controls to switch](https://www.w3.org/WAI/WCAG22/Techniques/css/C30)

## 情報と表現の分離（G140）

- 情報・構造と視覚的表現を分離し、異なる提示方法を可能にする
- バナー・ヒーローでは、背景や装飾は画像、情報はHTMLテキスト

```html
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

## ナビゲーション・ボタン・アイコン

- メニューやボタンのラベルを画像化しない
- シンボル文字（「B」「I」「ABC」等）を画像化したツールバーボタンも、実テキストまたはSVGテキストで代替
- アイコンは画像でもよいが、ラベルは実テキストにする

```html
<!-- 悪い例：ナビゲーション文字を画像化 -->
<nav aria-label="メインナビゲーション">
  <a href="/about"><img src="nav-about.png" alt="会社概要"></a>
</nav>

<!-- 良い例：テキスト + CSS -->
<nav aria-label="メインナビゲーション">
  <a class="nav-link" href="/about">会社概要</a>
  <a class="nav-link" href="/contact">お問い合わせ</a>
</nav>
```

```html
<!-- 悪い例：書式ボタンの文字を画像化 -->
<button type="button"><img src="bold-icon.png" alt="太字"></button>

<!-- 良い例：実テキストまたは aria-label -->
<button type="button" aria-label="太字"><strong>B</strong></button>
```

> **参照**
> - [Understanding SC 1.4.9 — Symbolic text characters example](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text-no-exception.html#examples)

## スキャンPDF・OCR（PDF7）

- スキャンPDFは文字が画像として保存されている場合が多い
- OCRを実行し、実テキストをPDF内に埋め込む
- 原本の見た目が不可欠な資料（歴史的文書等）を除き、検索・選択・読み上げ可能なテキスト層を提供

```text
悪い例：
  スキャン画像だけのPDF（情報伝達に文字画像を使用）

良い例：
  OCR済みPDF（画像 + テキスト層）
  または HTML本文として同じ内容を提供

例外：
  原本の筆跡・レイアウト自体が不可欠な歴史的資料
```

> **参照**
> - [Technique PDF7: Performing OCR on a scanned PDF document](https://www.w3.org/WAI/WCAG22/Techniques/pdf/PDF7)

## SVG内の文字

- SVGでも、`<text>` 要素で実テキストとして保持
- アウトライン化された文字（`<path>`）は文字画像と同様に扱われ、1.4.9 では不可欠でない限り不適合
- 装飾目的の見た目はCSSやSVG属性で表現し、文字情報はテキストとして残す

```svg
<!-- 悪い例：文字をpath化 -->
<svg viewBox="0 0 300 80" aria-label="SALE">
  <path d="..." />
</svg>

<!-- 良い例：text要素として保持 -->
<svg viewBox="0 0 300 80" role="img" aria-label="SALE">
  <text x="20" y="55" font-size="48" font-weight="700">SALE</text>
</svg>
```

## CSS背景画像内の文字

- CSS背景画像に重要な文字を含めない
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

## 1.4.3・1.4.4・1.4.8との関係

| 達成基準 | 対象 | 関係 |
|---|---|---|
| 1.4.3 コントラスト（最低限） | テキスト・文字画像 | 文字画像にもコントラスト要件 |
| 1.4.4 テキストのサイズ変更 | テキスト | 文字画像は対象外、拡大に弱い |
| 1.4.5 文字画像 | 情報を伝える文字画像 | AA、カスタマイズ可能な例外あり |
| 1.4.8 視覚的提示 | テキストブロック | 行幅・行間・配色の調整 |
| 1.4.9 文字画像（例外なし） | 文字画像 | AAA、装飾か不可欠のみ |

- 1.4.9 を満たす実テキスト実装は、1.4.4（拡大）や 1.4.8（視覚的提示）とも相性がよい
- 文字画像を使う場合でも、1.1.1（代替テキスト）と 1.4.3（コントラスト）は別途必要

> **参照**
> - [wcag/1-4-3-contrast-minimum.md — 1.4.3 コントラスト（最低限）](./1-4-3-contrast-minimum.md)
> - [wcag/1-4-4-resize-text.md — 1.4.4 テキストのサイズ変更](./1-4-4-resize-text.md)
> - [wcag/1-4-8-visual-presentation.md — 1.4.8 視覚的提示](./1-4-8-visual-presentation.md)

## よくある失敗例

- 見出しを画像にしているが、CSSで同じ見た目を実現できる（不可欠でも装飾でもない）
- カスタマイズ可能な文字画像を使っている（1.4.5 では例外だが 1.4.9 では不適合）
- 実テキストと併用しているが、画像内の文字が情報を伝えている（純粋な装飾ではない）
- ボタンやナビゲーションのラベルを画像化している
- キャンペーンバナーの情報が画像内に含まれている
- 文字をSVG pathにアウトライン化している
- スキャンPDFにOCRテキスト層がない（原本資料の例外を除く）

```html
<!-- 悪い例：altがあっても1.4.9としては不適合 -->
<img src="heading.png" alt="お問い合わせ">

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
- 文字画像が**純粋な装飾**か**不可欠な文字表現**かを判断
- 1.4.5 で「カスタマイズ可能」や「実テキスト併用」で適合していた箇所を再確認
- 200%拡大・高コントラスト表示・テキスト選択・ユーザースタイルシートで問題がないか確認

```bash
# 画像要素・背景画像・SVGをざっくり確認
rg "<img|background-image|<svg|canvas" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.md"
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.9 は、文字画像を**純粋な装飾**または**不可欠な文字表現**に限定する達成基準（レベル AAA）
- 1.4.5 の「カスタマイズ可能」例外は**認められない**
- 1.4.5 で「実テキストと併用」で適合していたケースも、1.4.9 では文字画像自体が問題になりうる
- ロゴタイプ、フォント見本、原本資料、書道作品などは不可欠として例外
- 基本方針は実テキスト + CSS（C22）、情報と表現の分離（G140）
- 1.4.9 を満たす実装は、1.4.4（拡大）や 1.4.8（視覚的提示）とも整合する
