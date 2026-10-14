# アクセシビリティ・オーバーレイの問題 — 後付けの修正ウィジェットでは代替にならない理由

## アクセシビリティ・オーバーレイとは

- ページ読み込み後に、第三者のJavaScriptを差し込んで表示やDOMを変えようとする製品群
- 画面端の丸いボタンから、文字サイズ・コントラスト・読み上げなどを切り替える**ウィジェット型**
- ページのHTMLを走査し、ラベルやARIAを自動で足そうとする**自動修復型**
- 製品によっては両方が同じスクリプトに入っている
- 「1行のscriptタグを貼るだけでWCAG適合」という売り方が典型
- 別名で再販（ホワイトラベル）されることもあり、見た目のブランド名だけでは判断しにくい

> 参照: [Overlay Fact Sheet — What is a web accessibility overlay?](https://overlayfactsheet.com/en/)

## なぜ後付けでは適合にならないか

- WCAGの適合は、適用される達成基準を**すべて**満たすこと
- 一部の見た目や属性が良くなっても、残った違反があれば適合ではない
- オーバーレイはブラウザ上の描画後に動くため、元のHTML・CSS・テンプレートは直らない
- スクリプトが落ちた、ブロックされた、読み込み前の瞬間は、元のページのまま
- クロスオリジンのiframe（決済フォームなど）は、ページ上のスクリプトから中を直せない
- React / Vue / Angular のように描画後にDOMが変わるUIは、修復スクリプトの検知から外れることがある
- PDF・動画・Canvas・SVGの中身も、HTMLへの属性追加では直らない

> 参照:
> - [Overlay Fact Sheet — Fitness for achieving compliance](https://overlayfactsheet.com/en/)
> - [W3C WAI — Understanding Conformance（WCAG 2）](https://www.w3.org/WAI/WCAG22/Understanding/conformance.html)

## 自動修復が信頼しにくい箇所

| 直しにくいもの | 理由 |
|---|---|
| 画像の代替テキスト | 見た目の説明と、その画像の役割（装飾・リンク先・機能）は一致しないことが多い |
| フォームのラベル・エラー | どの入力の失敗か、どう直せばよいかは画面の文脈がないと決められない |
| キーボード操作 | フォーカス順・トラップ・カスタムウィジェットのキー操作は自動補完が不安定 |
| 見出し・表・ランドマーク | 属性を足すことはあっても、マークアップの順序や意味そのものは作り直せない |
| 動的UI | カルーセル、モーダル、日付選択など、状態が変わる部品は取りこぼしやすい |

- 開発中に使うaxeやLighthouseは、問題を見つけるための検査
- オーバーレイは、利用者のブラウザ上で問題を隠そうとする別物
- 検査ツールですら自動化できる範囲は達成基準の一部。修復スクリプトがその範囲を超えて適合させることはできない

> 参照: [Overlay Fact Sheet — Strengths and weaknesses of automated repair](https://overlayfactsheet.com/en/)

## ウィジェットが利用者の役に立ちにくい理由

- 文字拡大、コントラスト、読み上げ、拡大は、OS・ブラウザ・支援技術にすでに備わっている
- その機能が必要な人は、そのサイトだけではなく、日常的に使うすべてのサイトで必要
- サイトごとのウィジェットは、良くて重複、悪くすると利用者の設定を上書きする
- コントラスト切替は任意の操作。初期表示が低コントラストなら、切替を使わない状態では達成基準を満たさない
- 欧州障害フォーラム（EDF）とIAAPも、オーバーレイはブラウザやOSの機能の複製であり、サイト自体の修正の代替ではないと表明

> 参照:
> - [Overlay Fact Sheet — Overlay widgets](https://overlayfactsheet.com/en/)
> - [EDF / IAAP — Joint statement on accessibility overlays](https://www.edf-feph.org/publications/joint-statement-on-accessibility-overlays/)

## 支援技術とぶつかり、かえって使いにくくなる

- スクリーンリーダー利用者は、自分の読み上げ・ブラウザ・ショートカットをすでに調整済み
- オーバーレイが「このサイトはスクリーンリーダー向けに調整した」と割り込み、フォーカスを奪うことがある
- 見出しの水増し、Tabの奪取、読み上げの中断など、元のページより操作が増える報告がある
- 利用者がhostsや拡張機能でオーバーレイをブロックして、ようやく普通に読めた、という声もある
- 障害のある人に、そのサイト専用の別ツールの習得を求める設計は、アクセシビリティの目的と逆

> 参照: [Overlay Fact Sheet — In their own words](https://overlayfactsheet.com/en/)

## コードで見る違い

### 1行の導入では、名前も操作も残る

```html
<!-- 悪い例：スクリプトを足しただけで、ボタンの名前も開閉も未実装 -->
<script src="https://cdn.example-overlay.com/widget.js" async></script>

<button>☰</button>
<div class="nav">
  <a href="/about">会社情報</a>
</div>
```

```html
<!-- 良い例：ボタンの名前、開閉状態、移動先をHTML側で持たせる -->
<button
  type="button"
  aria-expanded="false"
  aria-controls="site-nav"
>
  メニュー
</button>
<nav id="site-nav" hidden>
  <a href="/about">会社情報</a>
</nav>
```

- オーバーレイが `aria-label` を推測で足しても、開閉の状態やフォーカス移動までは保証されない
- 直す場所はコンポーネントのテンプレート

### 代替テキストは役割で書く

```html
<!-- 悪い例：自動生成がやりがちな、見た目だけの説明 -->
<a href="/cart">
  <img src="/icon-bag.svg" alt="白いバッグのアイコン">
</a>

<!-- 良い例：リンクの目的が名前になる -->
<a href="/cart">
  <img src="/icon-bag.svg" alt="">
  カートを見る
</a>
```

- 装飾アイコンに文章のaltを足すと、リンク名が二重になったり、目的とずれる
- 機能画像は行き先、装飾画像は空の `alt`。この判断は自動修復が苦手

### コントラストは初期状態で満たす

```css
/* 悪い例：薄いグレーの本文。ウィジェットの「コントラスト切替」待ち */
.body-copy {
  color: #b0b0b0;
  background: #ffffff;
}

/* 良い例：切替がなくても読める色 */
.body-copy {
  color: #1a1a1a;
  background: #ffffff;
}
```

- 達成基準は、利用者がウィジェットを見つけた後の表示ではなく、提供されるコンテンツに対して見る
- 利用者設定の上書き用トグルは、元の色の不足を埋めない

> 参照:
> - [W3C WAI — Understanding Conformance](https://www.w3.org/WAI/WCAG22/Understanding/conformance.html)
> - [W3C WAI — Non-text Content](https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html)
> - [W3C WAI — Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)

## 「入れれば法的に安全」は成り立たない

- Overlay Fact Sheetの結論は、市場のオーバーレイ製品は既存のアクセシビリティ規格への完全適合をもたらせず、法的リスクも消せない、というもの
- 2025年、米FTCは accessiBe に対し、accessWidget が任意のサイトをWCAG適合にできるという表示などが虚偽・誤解を招く・根拠不十分だったとして、100万ドルの支払いなどを含む同意命令を確定
- 命令は、根拠なく「どのサイトでもWCAG適合にできる」「コンテンツ変更後も自動で適合を維持できる」と表示することを禁じている
- 欧州委員会は、サイト自体が EN 301 549 の基準を満たさないツールは適切な解決策ではなく、問題は発生源で直すべきだと説明
- EDFは、欧州のアクセシビリティ法への適合もオーバーレイでは保証されないとしている
- 日本のJIS X 8341-3もWCAG対応が中心であり、ウィジェットの有無は適合の根拠にならない

> 参照:
> - [Overlay Fact Sheet — Conclusion](https://overlayfactsheet.com/en/)
> - [FTC — Final order requiring accessiBe to pay $1 million（2025-04-22）](https://www.ftc.gov/news-events/news/press-releases/2025/04/ftc-approves-final-order-requiring-accessibe-pay-1-million)
> - [European Commission — Web accessibility（Accessibility overlays）](https://digital-strategy.ec.europa.eu/en/policies/web-accessibility)
> - [EDF — Overlays don’t guarantee compliance with European legislation](https://www.edf-feph.org/accessibility-overlays-dont-guarantee-compliance-with-european-legislation/)

## プライバシーの副作用

- スクリーンリーダー利用などを検知して機能を自動で切り替える製品は、その端末の利用者に障害があることをサイト側が知る形になる
- 障害の有無は機微な個人情報であり、同意のない収集は避ける対象
- 同じオーバーレイを使う別サイトへ設定をクッキーで持ち回る実装は、利用者のオプトインがない追跡になり得る
- Overlay Fact Sheetは、この種の実装がGDPRやCCPA上のリスクになると指摘

> 参照: [Overlay Fact Sheet — Privacy of personal data](https://overlayfactsheet.com/en/)

## 実務者の評価

- WebAIMのアクセシビリティ実務者調査（2021年1月、有効回答758）では、オーバーレイ／プラグイン／ウィジェットについて回答者の67%が「まったく効果がない」または「あまり効果がない」
- 障害のある回答者では72%が同様の評価で、「非常に効果的」は2.4%
- 標本は全実務者を代表するよう制御されていない、という調査側の注記がある
- Overlay Fact Sheetには、WCAG・ARIA・HTMLの編集者、支援技術の開発者、障害当事者を含む多数の署名がある
- 署名者の共通方針は、出典で直すこと、適合をうたうオーバーレイを推奨しないこと

> 参照:
> - [WebAIM — Survey of Web Accessibility Practitioners #3 Results](https://webaim.org/projects/practitionersurvey3/)
> - [Overlay Fact Sheet — Signatories](https://overlayfactsheet.com/en/)

## フロントエンドが代わりにやること

| 状況 | やること |
|---|---|
| クライアントが導入を依頼 | 適合や訴訟回避の根拠にならないことを伝え、テンプレート修正の範囲を見積もる |
| すでにscriptタグがある | 外した状態でキーボード・スクリーンリーダー・自動検査をやり直す |
| 自動検査で検出された問題 | コンポーネントとCSSの出典を直す。検出されなかった操作系は手動で確認する |
| 文字サイズやコントラストの要望 | 相対単位、十分なコントラスト、`prefers-reduced-motion` など、ページ自体の対応に落とす |
| リリース後の担保 | CIのaxe、主要画面の手動確認、可能なら支援技術利用者の確認 |

- オーバーレイを残したまま「アクセシビリティ対応済み」とREADMEや提案書に書かない
- 検査ツールは開発者向け。利用者向けの後付けウィジェットの導入とは分けて説明する

> 参照: [Overlay Fact Sheet — Statement from sponsors and signatories](https://overlayfactsheet.com/en/)

## 関連して読むとよいテーマ

- axe-core / Lighthouseを使ったアクセシビリティ自動テスト — 検出と修復は別
- セマンティックHTMLの原則 — 出典で直すときの土台
- 画像のalt属性 — 自動生成がずれる典型
- アクセシブルなフォームの設計 — ラベルとエラーは文脈が必要
- キーボードナビゲーションの基本 — 後付けスクリプトが壊しやすい操作
- WCAG 2.2の4原則 — 適合は達成基準の積み上げ

## まとめ

- アクセシビリティ・オーバーレイは、読み込み後のJavaScriptで表示変更やDOM修復を試みる第三者スクリプト
- WCAG適合は全達成基準が対象であり、一部の自動修復や任意の表示切替では満たせない
- 代替テキスト、フォーム、キーボード、動的UIは自動修復が苦手で、支援技術の操作を妨げることもある
- 文字サイズやコントラストのウィジェットは、OSとブラウザに既にある機能と重複しやすい
- 米FTCのaccessiBe同意命令、欧州委員会、EDFは、いずれも「入れれば適合」という扱いに否定的
- フロントエンドの仕事はscriptタグの追加ではなく、テンプレート・CSS・操作の出典を直すこと
