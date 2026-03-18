# 画像のalt属性 — 適切な代替テキストの書き方

## alt属性の役割

- スクリーンリーダーが画像の代わりに読み上げるテキスト
- 画像が読み込めない場合にブラウザが表示する代替テキスト
- 検索エンジンが画像の内容を理解するための情報源（SEO）
- 画像を非表示にしている環境（低速回線・テキストブラウザなど）での情報補完

> **参照**
> - [HTML Standard — img element](https://html.spec.whatwg.org/multipage/embedded-content.html#the-img-element)
> - [MDN — HTMLImageElement: alt property](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/alt)

## alt属性が必須な理由

- WCAG 2.2 達成基準 1.1.1「非テキストコンテンツ」（レベルA）で要求
- すべての非テキストコンテンツにテキストによる代替を提供する義務
- レベルAは最低限の適合レベルであり、省略は許されない
- 法的要件としてもアクセシビリティ基準の根幹となる項目

> **参照**
> - [WCAG 2.2 — Success Criterion 1.1.1 Non-text Content](https://www.w3.org/TR/WCAG22/#non-text-content)

## 代替テキストの基本原則

- 簡潔に書く（目安は125文字以内、一部スクリーンリーダーの読み上げ制限を考慮）
- 画像の「目的」を伝える（見た目の描写ではなく、何を伝えたいか）
- 「〜の画像」「〜の写真」「〜のイラスト」という表現は不要（スクリーンリーダーが「画像」と読み上げるため二重表現になる）
- 周囲のテキストと重複しない内容にする
- 文脈に応じて同じ画像でも代替テキストを変える

```html
<!-- 悪い例：「の画像」が冗長 -->
<img src="team.jpg" alt="チームメンバーの写真">

<!-- 良い例：内容を簡潔に -->
<img src="team.jpg" alt="開発チーム5名がオフィスで集合している様子">
```

```html
<!-- 悪い例：見た目の描写に終始 -->
<img src="graph.png" alt="青い棒グラフ">

<!-- 良い例：情報としての目的を伝える -->
<img src="graph.png" alt="2024年の月別売上推移。7月が最高で500万円">
```

> **参照**
> - [W3C WAI — An alt Decision Tree](https://www.w3.org/WAI/tutorials/images/decision-tree/)
> - [W3C WAI — Images Tutorial](https://www.w3.org/WAI/tutorials/images/)

## 画像の種類別の書き方

### 1. 情報を伝える画像

- コンテンツの内容理解に必要な画像
- 画像が伝えている情報をテキストで説明する

```html
<!-- 悪い例：意味のない代替テキスト -->
<img src="warning-icon.png" alt="アイコン">

<!-- 良い例：アイコンが伝える意味を記述 -->
<img src="warning-icon.png" alt="警告">
```

```html
<!-- 悪い例：ファイル名のまま -->
<img src="IMG_20240301.jpg" alt="IMG_20240301.jpg">

<!-- 良い例：写真の内容を説明 -->
<img src="IMG_20240301.jpg" alt="渋谷スクランブル交差点の夜景">
```

### 2. 機能を持つ画像（リンク画像・ボタン画像）

- リンクやボタン内に含まれる画像
- 画像の見た目ではなく「操作の結果」を説明する
- リンク先やボタンの機能を代替テキストで伝える

```html
<!-- 悪い例：画像の見た目を説明 -->
<a href="/">
  <img src="logo.png" alt="青いロゴマーク">
</a>

<!-- 良い例：リンク先（機能）を説明 -->
<a href="/">
  <img src="logo.png" alt="ホームに戻る">
</a>
```

```html
<!-- 悪い例：「ボタン」と書くのは冗長（button要素が伝える） -->
<button>
  <img src="search.svg" alt="検索ボタン">
</button>

<!-- 良い例：機能を簡潔に -->
<button>
  <img src="search.svg" alt="検索">
</button>
```

```html
<!-- 悪い例：リンクテキストと画像altが重複 -->
<a href="/cart">
  <img src="cart.svg" alt="カートを見る">
  カートを見る
</a>

<!-- 良い例：テキストが既にあるので画像は装飾扱い -->
<a href="/cart">
  <img src="cart.svg" alt="">
  カートを見る
</a>
```

### 3. 装飾的な画像

- コンテンツの理解に影響しない純粋な装飾
- 背景パターン、区切り線の装飾、雰囲気のためのイメージ写真など
- `alt=""` （空文字）を指定してスクリーンリーダーに無視させる

```html
<!-- 悪い例：装飾画像に説明を付ける -->
<img src="divider.png" alt="装飾的な区切り線">

<!-- 良い例：空のaltで装飾であることを明示 -->
<img src="divider.png" alt="">
```

```html
<!-- 悪い例：装飾に意味のないaltを付ける -->
<img src="bg-pattern.png" alt="背景画像">

<!-- 良い例 -->
<img src="bg-pattern.png" alt="">
```

### 4. テキストを含む画像

- バナーやロゴなど、画像内にテキストが描かれているもの
- 画像内のテキスト内容をそのままalt属性に記述する
- 可能であれば画像ではなく実テキスト+CSSでの表現を推奨（WCAG 1.4.5）

```html
<!-- 悪い例：テキスト内容を書いていない -->
<img src="sale-banner.png" alt="セールバナー">

<!-- 良い例：画像内のテキストをそのまま記述 -->
<img src="sale-banner.png" alt="春の大セール 全品30%OFF 3月1日〜3月31日">
```

```html
<!-- 悪い例：ロゴの見た目を説明 -->
<img src="company-logo.png" alt="ロゴ画像">

<!-- 良い例：ロゴに含まれるテキストを記述 -->
<img src="company-logo.png" alt="株式会社サンプル">
```

### 5. 複雑な画像（グラフ・図表・インフォグラフィック）

- alt属性だけでは情報を十分に伝えられない画像
- alt属性には概要を記述し、本文中またはリンク先で詳細を説明する
- `aria-describedby` で詳細テキストと紐づける方法もある

```html
<!-- 悪い例：altに全情報を詰め込む -->
<img
  src="sales-chart.png"
  alt="2024年の月別売上推移のグラフ。1月100万、2月120万、3月150万、4月130万、5月180万、6月200万、7月500万、8月300万、9月250万、10月220万、11月400万、12月350万"
>

<!-- 良い例：altに概要、詳細は別途提供 -->
<figure>
  <img
    src="sales-chart.png"
    alt="2024年の月別売上推移グラフ。7月が最高で500万円"
    aria-describedby="chart-detail"
  >
  <figcaption>図1: 2024年の月別売上推移</figcaption>
</figure>
<details id="chart-detail">
  <summary>グラフの詳細データ</summary>
  <table>
    <caption>2024年 月別売上</caption>
    <thead>
      <tr><th>月</th><th>売上（万円）</th></tr>
    </thead>
    <tbody>
      <tr><td>1月</td><td>100</td></tr>
      <tr><td>2月</td><td>120</td></tr>
      <!-- ... -->
      <tr><td>7月</td><td>500</td></tr>
      <!-- ... -->
    </tbody>
  </table>
</details>
```

> **参照**
> - [W3C WAI — Complex Images](https://www.w3.org/WAI/tutorials/images/complex/)
> - [W3C WAI — Functional Images](https://www.w3.org/WAI/tutorials/images/functional/)
> - [W3C WAI — Decorative Images](https://www.w3.org/WAI/tutorials/images/decorative/)

## alt="" と alt属性なしの違い

- `alt=""`（空文字）：**意図的に装飾画像として扱う宣言**。スクリーンリーダーはこの画像を完全に無視する
- `alt`属性なし：**代替テキストの指定漏れ**。スクリーンリーダーがファイル名を読み上げてしまう場合がある（例：「画像 IMG underscore 20240301 dot jpg」）
- HTMLバリデーターは`alt`属性なしを警告する
- 装飾画像であっても `alt` 属性自体は必ず書く

```html
<!-- 装飾画像の正しい書き方 -->
<img src="decoration.png" alt="">

<!-- alt属性が欠落している（NG） -->
<img src="decoration.png">
<!-- → スクリーンリーダーが「decoration.png」と読み上げる可能性 -->
```

> **参照**
> - [HTML Standard — Requirements for providing text to act as an alternative for images](https://html.spec.whatwg.org/multipage/images.html#alt)

## figure / figcaption との組み合わせ

- `<figure>` は自己完結した参照コンテンツ（画像・図表・コード例など）を囲む要素
- `<figcaption>` は `<figure>` のキャプション（説明文）
- figcaptionはaltの代わりにはならない（役割が異なる）
- altは画像そのものの代替テキスト、figcaptionは図全体の説明

```html
<!-- 悪い例：altを省略してfigcaptionだけに頼る -->
<figure>
  <img src="architecture.png">
  <figcaption>図2: システム構成図</figcaption>
</figure>

<!-- 良い例：altとfigcaptionの両方を適切に設定 -->
<figure>
  <img src="architecture.png" alt="フロントエンド、API、データベースの3層構成を示す図">
  <figcaption>図2: システム構成図</figcaption>
</figure>
```

```html
<!-- altとfigcaptionで重複を避ける例 -->
<figure>
  <img src="sunset.jpg" alt="海岸線に沈む夕日。空がオレンジと紫のグラデーション">
  <figcaption>2024年8月、湘南海岸にて撮影</figcaption>
</figure>
```

> **参照**
> - [MDN — figure element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/figure)
> - [W3C WAI — Images Tutorial: Groups of Images](https://www.w3.org/WAI/tutorials/images/groups/)

## SVGのアクセシビリティ

- インラインSVGは `<img>` と異なりaltが使えない
- `role="img"` でアクセシビリティツリー上「画像」として認識させる
- `aria-label` または `aria-labelledby` で代替テキストを提供
- SVG内の `<title>` 要素と `aria-labelledby` を組み合わせる方法が最も堅牢

### img要素でSVGを読み込む場合

```html
<!-- 通常のimg要素と同じ扱い -->
<img src="icon.svg" alt="設定">
```

### インラインSVGの場合

```html
<!-- 悪い例：代替テキストが何もない -->
<svg viewBox="0 0 24 24">
  <path d="M12 2L2 22h20L12 2z" />
</svg>

<!-- 良い例：role="img" + aria-label -->
<svg viewBox="0 0 24 24" role="img" aria-label="警告">
  <path d="M12 2L2 22h20L12 2z" />
</svg>

<!-- 良い例：title要素 + aria-labelledby（最も堅牢） -->
<svg viewBox="0 0 24 24" role="img" aria-labelledby="svg-title">
  <title id="svg-title">警告</title>
  <path d="M12 2L2 22h20L12 2z" />
</svg>
```

### 装飾的なSVGの場合

```html
<!-- 装飾SVGはaria-hiddenで隠す -->
<svg viewBox="0 0 24 24" aria-hidden="true" focusable="false">
  <path d="M12 2L2 22h20L12 2z" />
</svg>
```

- `aria-hidden="true"` でアクセシビリティツリーから除外
- `focusable="false"` はIE/旧Edgeでの不要なフォーカスを防ぐ指定（レガシー対応）

> **参照**
> - [Accessible SVGs — CSS-Tricks](https://css-tricks.com/accessible-svgs/)
> - [MDN — SVG: title element](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/title)

## CSSの背景画像と代替テキスト

- `background-image` で設定した画像にはalt属性を付与できない
- スクリーンリーダーはCSS背景画像を認識しない
- 情報を伝える画像をCSS背景にするのは避けるべき
- どうしてもCSS背景にする場合は代替手段を講じる

```html
<!-- 悪い例：情報を持つ画像をCSS背景にして代替テキストなし -->
<div class="hero" style="background-image: url('sale-banner.jpg')"></div>

<!-- 良い例：情報を持つ画像はimg要素で配置 -->
<img src="sale-banner.jpg" alt="春の大セール 全品30%OFF">
```

```html
<!-- CSS背景でどうしても情報を伝える必要がある場合の代替手段 -->

<!-- 方法1: visually-hiddenクラスで隠しテキストを配置 -->
<div class="hero" style="background-image: url('sale-banner.jpg')">
  <span class="visually-hidden">春の大セール 全品30%OFF</span>
</div>

<!-- 方法2: role="img" + aria-label -->
<div
  class="hero"
  style="background-image: url('sale-banner.jpg')"
  role="img"
  aria-label="春の大セール 全品30%OFF"
></div>
```

```css
/* visually-hiddenクラスの定義 */
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

> **参照**
> - [W3C WAI — CSS Techniques for WCAG](https://www.w3.org/WAI/WCAG22/Techniques/#css)
> - [A11Y Project — How to hide content](https://www.a11yproject.com/posts/how-to-hide-content/)

## よくある間違い

### 1. ファイル名をそのまま使う

```html
<!-- NG -->
<img src="DSC_0042.jpg" alt="DSC_0042.jpg">
<img src="hero-banner-v2-final.png" alt="hero-banner-v2-final.png">

<!-- OK -->
<img src="DSC_0042.jpg" alt="東京タワーの夜景">
<img src="hero-banner-v2-final.png" alt="新サービス開始のお知らせ">
```

### 2. 過度に長い説明を書く

```html
<!-- NG：長すぎて要点が伝わらない -->
<img
  src="office.jpg"
  alt="弊社の東京本社オフィスの3階にある広々としたオープンスペースで、白い壁に囲まれた明るい空間の中央に大きな木製テーブルが置かれ、その周りに10脚の椅子が配置されており、窓からは東京タワーが見える眺望が広がっています"
>

<!-- OK：簡潔に要点を伝える -->
<img src="office.jpg" alt="東京本社のオープンスペース。中央に大きな会議テーブル">
```

### 3. すべての画像に同じaltを使う

```html
<!-- NG：商品一覧で全て同じalt -->
<img src="product1.jpg" alt="商品画像">
<img src="product2.jpg" alt="商品画像">
<img src="product3.jpg" alt="商品画像">

<!-- OK：個別の内容を記述 -->
<img src="product1.jpg" alt="ワイヤレスイヤホン XZ-100 ブラック">
<img src="product2.jpg" alt="ワイヤレスイヤホン XZ-100 ホワイト">
<img src="product3.jpg" alt="ワイヤレスイヤホン XZ-200 ノイズキャンセリング対応">
```

### 4. 装飾画像にaltを書く

```html
<!-- NG：装飾に不要な説明 -->
<img src="arrow-right.svg" alt="右向き矢印アイコン">次のページ

<!-- OK：隣接テキストで機能が伝わるなら装飾扱い -->
<img src="arrow-right.svg" alt="">次のページ
```

### 5. 「画像」「写真」「イメージ」を含める

```html
<!-- NG：二重表現 -->
<img src="cat.jpg" alt="猫の写真">
<!-- → スクリーンリーダーの読み上げ：「画像 猫の写真」 -->

<!-- OK -->
<img src="cat.jpg" alt="窓辺で日向ぼっこをしている三毛猫">
<!-- → スクリーンリーダーの読み上げ：「画像 窓辺で日向ぼっこをしている三毛猫」 -->
```

### 6. alt属性を省略する

```html
<!-- NG：alt属性自体がない -->
<img src="important-info.png">

<!-- OK：装飾なら空文字、情報なら内容を記述 -->
<img src="important-info.png" alt="サービス停止のお知らせ: 3月15日 0:00〜6:00">
```

> **参照**
> - [W3C WAI — Images Tutorial: Tips and Tricks](https://www.w3.org/WAI/tutorials/images/tips/)

## まとめ

- alt属性はWCAG 1.1.1で求められる必須の代替テキスト
- 画像の「目的」に応じてaltの書き方を変える
- 情報画像 → 内容を説明、機能画像 → 機能を説明、装飾画像 → `alt=""`
- テキスト画像 → テキストをそのまま記述、複雑な画像 → 概要 + 詳細
- `alt=""` と `alt`属性なしは全く異なる（空文字は意図的、属性なしは指定漏れ）
- figcaptionはaltの代わりにはならない
- インラインSVGには `role="img"` + `aria-labelledby` / `aria-label`
- CSS背景画像には代替テキストを付けられないため、情報画像には使わない
- ファイル名の流用、過度に長い説明、同一altの使い回しは避ける
