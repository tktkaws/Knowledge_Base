# 1.1.1 非テキストコンテンツ（A）

## 達成基準の概要

- WCAG 2.2 達成基準 1.1.1「非テキストコンテンツ」（Non-text Content）
- レベル A（最低限の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.1「テキストによる代替」に属する
- 画像・アイコン・音声・動画・CAPTCHA など、文字以外で伝えられる情報すべてが対象
- レベル A 達成の前提条件であり、多くの国・組織の法的要件の根幹

> **参照**
> - [WCAG 2.2 — Success Criterion 1.1.1 Non-text Content](https://www.w3.org/TR/WCAG22/#non-text-content)
> - [Understanding SC 1.1.1 Non-text Content](https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html)

## 達成基準の原文（要約）

- ユーザーに提示されるすべての非テキストコンテンツに、**同等の目的を果たすテキストによる代替**を提供すること
- 以下の例外状況はこの要件の対象外

| 例外 | 要件 |
|---|---|
| コントロール・入力 | 目的を説明する名前（name）を提供（4.1.2 も参照） |
| 時間依存メディア | 少なくとも内容を識別できる説明的ラベル（1.2 も参照） |
| テスト・演習 | テキスト化すると無効になる場合、識別用の説明的ラベル |
| 感覚体験 | 特定の感覚体験が主目的の場合、識別用の説明的ラベル |
| CAPTCHA | 目的の説明 + 異なる感覚モダリティの代替形式 |
| 装飾・書式・非表示 | 支援技術が無視できる実装 |

> **参照**
> - [WCAG 2.2 — Success Criterion 1.1.1](https://www.w3.org/TR/WCAG22/#non-text-content)

## 非テキストコンテンツとは

- プログラム的に判定できる文字列ではないコンテンツ、または人間の言語として意味を持たない文字列
- 具体的な例
  - 画像（`<img>`、CSS背景画像、SVG）
  - 音声・動画ファイル
  - グラフ・図表・インフォグラフィック
  - 絵文字・ASCIIアート・leetspeak
  - 画像ボタン・イメージマップ
  - CAPTCHA
- テキストコンテンツ（段落・見出し・リンクテキストなど）は対象外

> **参照**
> - [Understanding SC 1.1.1 — Key Terms: non-text content](https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html#dfn-non-text-content)

## なぜ必要か

- 視覚・聴覚など特定の感覚に依存せず情報を知覚できるようにするため
- テキスト代替は視覚・聴覚・触覚（点字）など任意のモダリティで提示可能
- スクリーンリーダー利用者・画像非表示環境・低速回線ユーザーへの情報提供
- 検索・再利用・翻訳の基盤となる

> **参照**
> - [Understanding SC 1.1.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html#benefits)

## 6つの状況別対応（Sufficient Techniques）

WCAG 2.2 の達成手法は、非テキストコンテンツの種類ごとに6つの状況（Situation A〜F）に分類される。

### Situation A：短い説明で同等の情報を伝えられる場合

- 情報画像・アイコン・シンプルなイラストなど
- 短いテキスト代替だけで非テキストコンテンツと同等の目的・情報を提供できるケース
- 主な手法：`alt` 属性、`aria-label`、`aria-labelledby`

```html
<!-- 悪い例：alt属性がない -->
<img src="warning.png">

<!-- 悪い例：意味のない代替テキスト -->
<img src="warning.png" alt="画像">

<!-- 良い例：同等の情報を短いテキストで提供 -->
<img src="warning.png" alt="警告：入力内容に不備があります">
```

```html
<!-- 悪い例：インラインSVGに代替テキストがない -->
<svg viewBox="0 0 24 24">
  <path d="M12 2L2 22h20L12 2z" />
</svg>

<!-- 良い例：role="img" + aria-label -->
<svg viewBox="0 0 24 24" role="img" aria-label="警告">
  <path d="M12 2L2 22h20L12 2z" />
</svg>
```

> **参照**
> - [Technique H37: Using alt attributes on img elements](https://www.w3.org/WAI/WCAG22/Techniques/html/H37)
> - [Technique G94: Providing short text alternative](https://www.w3.org/WAI/WCAG22/Techniques/general/G94)

### Situation B：短い説明だけでは不十分な場合

- グラフ・図表・複雑なインフォグラフィックなど
- 短いテキスト代替（概要）+ 長いテキスト代替（詳細）の組み合わせが必要
- 主な手法：`aria-describedby`、近傍の詳細テキスト、別ページへのリンク

```html
<!-- 悪い例：altに全データを詰め込む -->
<img
  src="sales-chart.png"
  alt="1月100万、2月120万、3月150万、4月130万、5月180万、6月200万、7月500万..."
>

<!-- 悪い例：概要も詳細もない -->
<img src="sales-chart.png" alt="グラフ">

<!-- 良い例：短い概要 + 詳細は別途提供 -->
<figure>
  <img
    src="sales-chart.png"
    alt="2024年の月別売上推移グラフ。7月が最高で500万円"
    aria-describedby="chart-detail"
  >
  <figcaption>図1: 2024年の月別売上推移</figcaption>
</figure>
<table id="chart-detail">
  <caption>2024年 月別売上（万円）</caption>
  <thead>
    <tr><th>月</th><th>売上</th></tr>
  </thead>
  <tbody>
    <tr><td>7月</td><td>500</td></tr>
    <!-- ... -->
  </tbody>
</table>
```

> **参照**
> - [Technique G95: Providing short text alternatives (brief description)](https://www.w3.org/WAI/WCAG22/Techniques/general/G95)
> - [Technique ARIA15: Using aria-describedby](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA15)
> - [W3C WAI — Complex Images Tutorial](https://www.w3.org/WAI/tutorials/images/complex/)

### Situation C：コントロール・入力を受け付ける非テキストコンテンツ

- 画像ボタン・イメージマップ・画像付きリンクなど
- 見た目ではなく**操作の目的（名前）**をテキストで提供
- 4.1.2「名前・役割・値」との連携が必要

```html
<!-- 悪い例：画像の見た目を説明 -->
<button type="submit">
  <img src="search.svg" alt="虫眼鏡のアイコン">
</button>

<!-- 良い例：操作の目的を説明 -->
<button type="submit">
  <img src="search.svg" alt="検索">
</button>
```

```html
<!-- 悪い例：イメージマップのareaにaltがない -->
<img src="floor-plan.png" usemap="#map" alt="フロアマップ">
<map name="map">
  <area shape="rect" coords="0,0,100,100" href="/room-a">
</map>

<!-- 良い例：各エリアに目的を説明するalt -->
<img src="floor-plan.png" usemap="#map" alt="ビルフロアマップ。部屋を選択してください">
<map name="map">
  <area shape="rect" coords="0,0,100,100" href="/room-a" alt="会議室A">
  <area shape="rect" coords="100,0,200,100" href="/room-b" alt="会議室B">
</map>
```

```html
<!-- 悪い例：リンク画像とテキストが重複 -->
<a href="/cart">
  <img src="cart.svg" alt="カートを見る">
  カートを見る
</a>

<!-- 良い例：隣接テキストがある場合は画像を装飾扱い -->
<a href="/cart">
  <img src="cart.svg" alt="">
  カートを見る
</a>
```

> **参照**
> - [Technique G82: Text alternative that identifies the purpose of interactive non-text content](https://www.w3.org/WAI/WCAG22/Techniques/general/G82)
> - [Technique H36: alt attributes on images used as submit buttons](https://www.w3.org/WAI/WCAG22/Techniques/html/H36)

### Situation D：時間依存メディア・テスト・感覚体験

- 時間依存メディア（音声・映像）：1.2 ガイドラインの詳細要件に加え、少なくとも識別用ラベル
- テスト・演習：テキスト化すると無効になる場合（聴力検査・視覚スキル演習など）
- 感覚体験：交響曲の演奏・美術作品など、言葉だけでは伝えきれない体験

```html
<!-- 悪い例：音声ファイルに識別情報がない -->
<audio src="speech.mp3" controls></audio>

<!-- 良い例：識別用ラベル + 文字起こしへのリンク -->
<audio src="speech.mp3" controls aria-label="会長の総会スピーチ"></audio>
<a href="/transcript/speech">文字起こしを読む</a>
```

```html
<!-- 悪い例：ライブカメラに何の映像か不明 -->
<img src="/api/live-camera" alt="">

<!-- 良い例：識別用の説明的ラベル -->
<img src="/api/live-camera" alt="渋谷交差点の交通ライブカメラ（2分ごとに更新）">
```

```html
<!-- テスト・演習の例：聴力検査 -->
<!-- テキスト代替で答えを教えてしまうと演習が無効になるため、目的の説明のみ -->
<p id="hearing-test-desc">以下の音声を聞いて、聞こえた音を選択してください。</p>
<audio src="tone-440hz.mp3" controls aria-describedby="hearing-test-desc"></audio>
```

> **参照**
> - [Technique G68: Descriptive label for live audio-only and live video-only content](https://www.w3.org/WAI/WCAG22/Techniques/general/G68)
> - [Technique G100: Short text alternative which is the accepted name or descriptive name](https://www.w3.org/WAI/WCAG22/Techniques/general/G100)

### Situation E：CAPTCHA

- 人間であることの確認が目的の非テキストコンテンツ
- テキスト代替で答えを提供するとボット対策が無効になるため、**目的の説明**にとどめる
- **異なる感覚モダリティ**（視覚・聴覚など）の代替 CAPTCHA を最低2種類提供
- 可能であれば認証済みユーザーへの CAPTCHA 免除、人間オペレーターによるバイパスも推奨

```html
<!-- 悪い例：視覚CAPTCHAのみ、説明も代替もない -->
<img src="/captcha/image" alt="">

<!-- 悪い例：視覚CAPTCHAのみ（代替モダリティなし） -->
<img src="/captcha/image" alt="画像に表示されている文字を入力してください">

<!-- 良い例：目的の説明 + 複数モダリティ -->
<fieldset>
  <legend>人間確認</legend>
  <p id="captcha-desc">スパム防止のため、以下の確認を行ってください。</p>
  <img src="/captcha/image" alt="画像CAPTCHA：表示されている文字を入力欄に入力してください">
  <button type="button">音声CAPTCHAに切り替え</button>
</fieldset>
```

> **参照**
> - [Technique G143: Text alternative that describes the purpose of the CAPTCHA](https://www.w3.org/WAI/WCAG22/Techniques/general/G143)
> - [Technique G144: Another CAPTCHA using a different modality](https://www.w3.org/WAI/WCAG22/Techniques/general/G144)
> - [Inaccessibility of CAPTCHA](https://www.w3.org/TR/turingtest/)

### Situation F：装飾・書式・非表示

- 純粋な装飾、視覚的書式付けのみ、ユーザーに提示されないコンテンツ
- 支援技術が**無視できる**実装が必要
- 装飾画像に意味のある alt を付けると、スクリーンリーダー利用者の読み上げを妨げる（Failure F39）

```html
<!-- 悪い例：装飾画像に意味のあるalt -->
<img src="divider.png" alt="区切り線">

<!-- 悪い例：装飾画像にalt属性自体がない -->
<img src="divider.png">

<!-- 良い例：空のaltで支援技術に無視させる -->
<img src="divider.png" alt="">
```

```html
<!-- 悪い例：装飾SVGをそのまま放置 -->
<svg viewBox="0 0 24 24">
  <path d="M0 0h24v24H0z" />
</svg>

<!-- 良い例：aria-hiddenでアクセシビリティツリーから除外 -->
<svg viewBox="0 0 24 24" aria-hidden="true" focusable="false">
  <path d="M0 0h24v24H0z" />
</svg>
```

```css
/* 良い例：装飾目的の画像はCSS背景で配置（情報を持たない場合） */
.hero-decoration {
  background-image: url("pattern.png");
}
```

> **参照**
> - [Technique H67: Using null alt text on img elements](https://www.w3.org/WAI/WCAG22/Techniques/html/H67)
> - [Technique C9: Using CSS to include decorative images](https://www.w3.org/WAI/WCAG22/Techniques/css/C9)

## 代表的な失敗例（Failures）

- **F3**：CSS背景画像で重要な情報を伝えている（alt を付けられない）
- **F13**：画像内の色情報を代替テキストに含めていない
- **F20**：非テキストコンテンツ更新時にテキスト代替を更新していない
- **F30**：ファイル名やプレースホルダーを代替テキストにしている
- **F38**：装飾画像を支援技術が無視できるようマークアップしていない
- **F39**：装飾画像に `alt="spacer"` や `alt="image"` など非nullの代替を付けている
- **F65**：`img`・`area`・`input type="image"` に alt またはテキスト代替がない
- **F71**：CSSや画像で文字を再現し、テキスト代替を提供していない
- **F72**：ASCIIアートにテキスト代替がない

```html
<!-- F3 の例：CSS背景で情報を伝えている -->
<div class="banner" style="background-image: url('sale-30off.jpg')"></div>
<!-- → スクリーンリーダーは「30%OFF」の情報を取得できない -->

<!-- F30 の例：ファイル名をそのまま使用 -->
<img src="IMG_20240301.jpg" alt="IMG_20240301.jpg">

<!-- F65 の例：alt属性の欠落 -->
<img src="important-notice.png">
```

```css
/* F71 の例：画像で文字を表現 */
.logo-text {
  background-image: url("company-name.png");
  width: 200px;
  height: 40px;
}
/* → テキストコンテンツまたはalt付きimgで提供すべき */
```

> **参照**
> - [WCAG 2.2 Techniques — Failures for 1.1.1](https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html#techniques)

## CSS背景画像と1.1.1

- `background-image` には `alt` 属性を付与できない
- 情報を伝える目的の画像を CSS 背景にすると **Failure F3** となる
- 装飾目的のみであれば CSS 背景は Situation F として適合可能
- 情報を持つ画像は `<img>` 要素、または `role="img"` + `aria-label` で代替

```html
<!-- 悪い例：セール情報をCSS背景のみで表示 -->
<div class="hero" style="background-image: url('sale-banner.jpg')"></div>

<!-- 良い例：情報はimg要素で提供 -->
<img src="sale-banner.jpg" alt="春の大セール 全品30%OFF 3月1日〜3月31日">
```

> **参照**
> - [Failure F3: CSS to include images that convey important information](https://www.w3.org/WAI/WCAG22/Techniques/failures/F3)
> - [a11y/image-alt-text.md — CSSの背景画像と代替テキスト](../a11y/image-alt-text.md)

## テキスト代替の基本原則

- **同等の目的**：見た目の描写ではなく、非テキストコンテンツが果たす役割・情報を伝える
- **文脈依存**：同じ画像でも配置場所・用途によって代替テキストを変える
- **簡潔さ**：短い代替で足りる場合は長文にしない（複雑な画像は Situation B）
- **言語の一致**：代替テキストはページの主要言語と一致させる
- **装飾との区別**：情報がなければ `alt=""` または `aria-hidden="true"` で明示

```html
<!-- 同じ地球儀アイコン、文脈で代替テキストを変える -->

<!-- 旅行サイトのナビゲーション -->
<a href="/international">
  <img src="globe.svg" alt="海外旅行">
</a>

<!-- 大学サイトのキャンパス案内 -->
<a href="/campuses">
  <img src="globe.svg" alt="海外キャンパス">
</a>
```

> **参照**
> - [W3C WAI — An alt Decision Tree](https://www.w3.org/WAI/tutorials/images/decision-tree/)
> - [a11y/image-alt-text.md — 画像のalt属性の詳細](../a11y/image-alt-text.md)

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.2 時間依存メディア | 音声・映像の詳細要件（キャプション・音声解説など） |
| 1.4.5 文字画像（AA） | 可能なら文字画像を実テキスト+CSSで表現 |
| 1.4.11 非テキストのコントラスト（AA） | UIコンポーネントの視認性（代替テキストとは別要件） |
| 4.1.2 名前・役割・値（A） | インタラクティブな非テキストの名前（name） |

- 1.1.1 は「情報をテキストで取得できること」が焦点
- 1.2 は時間依存メディアの**追加**要件（キャプション等）を規定
- 両方を満たす必要がある場合がある（例：動画には識別ラベル + キャプション）

> **参照**
> - [WCAG 2.2 — Guideline 1.2 Time-based Media](https://www.w3.org/TR/WCAG22/#time-based-media)
> - [WCAG 2.2 — Success Criterion 4.1.2 Name, Role, Value](https://www.w3.org/TR/WCAG22/#name-role-value)

## テスト・確認方法

### 自動テスト

- `alt` 属性の有無・空文字の適切性
- 装飾画像への不要な alt
- リンク・ボタン内画像の代替テキスト
- axe DevTools、WAVE、Lighthouse などのツールで検出可能

### 手動テスト

- スクリーンリーダー（VoiceOver / NVDA）で画像・SVG・アイコンの読み上げ確認
- 画像を非表示にした状態で情報が欠落しないか確認
- CSS背景画像に情報が含まれていないか確認
- 複雑なグラフ・図表で詳細情報が別途提供されているか確認

```bash
# Lighthouse CLI でアクセシビリティ監査（参考）
npx lighthouse https://example.com --only-categories=accessibility --output=html
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.1.1 はレベル A の根幹。すべての非テキストコンテンツにテキスト代替が必要
- 例外は6種類（コントロール、時間依存メディア、テスト、感覚体験、CAPTCHA、装飾）
- 状況 A〜F に応じて短い代替・長い代替・名前・識別ラベル・無視マークアップを使い分ける
- 情報画像 → 内容を説明、機能画像 → 目的を説明、装飾 → `alt=""` または `aria-hidden`
- CSS背景画像で重要情報を伝えるのは Failure F3
- 詳細な alt の書き方は [a11y/image-alt-text.md](../a11y/image-alt-text.md) を参照
