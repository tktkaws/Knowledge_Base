# Webアクセシビリティとは何か — なぜ重要なのか

## Webアクセシビリティの定義

- Webアクセシビリティとは、障害のある人々がWebを利用できるようにするための取り組み
- W3C（World Wide Web Consortium）配下のWAI（Web Accessibility Initiative）による定義：「障害のある人々がWebを知覚し、理解し、ナビゲートし、インタラクションできること」
- 対象はWebサイト、ツール、技術の設計・開発すべて
- 「誰もが同等の体験を得られること」がゴール
- 障害の有無にかかわらず、すべての人にとって使いやすいWebを目指す考え方

> 参照元
> - [W3C WAI - Introduction to Web Accessibility](https://www.w3.org/WAI/fundamentals/accessibility-intro/)
> - [W3C WAI - Accessibility Fundamentals](https://www.w3.org/WAI/fundamentals/)

## アクセシビリティが必要な人々

### 永続的な障害

- **視覚障害** — 全盲、ロービジョン（弱視）、色覚特性（色覚多様性）
- **聴覚障害** — 全聾、難聴
- **運動障害** — 上肢の麻痺、振戦、筋力低下、関節の可動域制限
- **認知障害** — 学習障害、注意欠陥、記憶障害、読字障害（ディスレクシア）

### 一時的な障害

- 骨折でマウスが使えない
- 目の手術後で画面が見づらい
- 耳の感染症で音声が聞こえにくい

### 状況的な制約

- 明るい屋外でスマートフォンの画面が見えにくい
- 騒がしい環境で音声が聞こえない
- 片手で子どもを抱えながらスマートフォンを操作する
- 低速なネットワーク環境で重いコンテンツが読み込めない

### 加齢に伴う変化

- 視力の低下
- 聴力の低下
- 運動能力・認知能力の変化
- 高齢化社会においてアクセシビリティ対応の重要性はさらに増大

> 参照元
> - [W3C WAI - Diverse Abilities and Barriers](https://www.w3.org/WAI/people-use-web/abilities-barriers/)
> - [W3C WAI - How People with Disabilities Use the Web](https://www.w3.org/WAI/people-use-web/)

## なぜ重要なのか

### 1. 人権・社会的包摂

- WebはUNの「障害者の権利に関する条約」で認められた基本的権利の一部
- 情報へのアクセスは現代社会における基本的な権利
- デジタルデバイドの解消に直結

### 2. 法的義務

- 多くの国でWebアクセシビリティへの対応が法律で義務化
- 未対応の場合、訴訟リスクや行政指導の対象となる可能性
- 詳細は後述の「法的背景」セクションを参照

### 3. ビジネス上のメリット

- 世界人口の約16%（13億人）が何らかの障害を持つ（WHO調査）
- アクセシブルなサイトはより多くのユーザーにリーチ可能
- ブランドイメージの向上・企業の社会的責任（CSR）の実践
- モバイル対応やSEO改善と施策が重複するため、投資効率が高い

### 4. ユーザー体験（UX）の向上

- アクセシビリティ改善はすべてのユーザーの体験を改善
- キーボードナビゲーション対応はパワーユーザーにも恩恵
- 適切なコントラスト比は明るい屋外でのモバイル利用時にも有益
- 明確な見出し構造や一貫した操作体系は認知負荷を軽減

### 5. SEOとの関連

- セマンティックなHTML構造は検索エンジンのクローラーにとっても理解しやすい
- 適切な見出し階層（h1〜h6）はSEOの基本でもある
- 画像のalt属性は画像検索の最適化にも寄与
- ページの構造化はリッチスニペット表示にもつながる
- サイトのパフォーマンス改善（アクセシビリティの一環）は検索順位に影響

> 参照元
> - [WHO - Disability and Health](https://www.who.int/news-room/fact-sheets/detail/disability-and-health)
> - [W3C WAI - The Business Case for Digital Accessibility](https://www.w3.org/WAI/business-case/)

## 日本における法的背景

### 障害者差別解消法

- 2016年4月に施行
- 2024年4月に改正法が施行（重要な転換点）
- **改正のポイント：民間事業者による「合理的配慮の提供」が努力義務から法的義務に変更**
- Webサイトやアプリも合理的配慮の対象に含まれると解釈される

### JIS X 8341-3

- 日本産業規格として策定されたWebアクセシビリティの規格
- 「高齢者・障害者等配慮設計指針 — 情報通信における機器、ソフトウェア及びサービス — 第3部：ウェブコンテンツ」
- WCAG 2.0と技術的に一致する内容（2016年版）
- 公共機関のWebサイトでは JIS X 8341-3 への準拠が求められる

### デジタル庁の取り組み

- 「ウェブアクセシビリティ導入ガイドブック」を公開
- 行政機関のWebサイトにおけるアクセシビリティ確保を推進

> 参照元
> - [内閣府 - 障害を理由とする差別の解消の推進](https://www8.cao.go.jp/shougai/suishin/sabekai.html)
> - [デジタル庁 - ウェブアクセシビリティ導入ガイドブック](https://www.digital.go.jp/resources/introduction-to-web-accessibility-guidebook)
> - [JIS X 8341-3:2016](https://www.jisc.go.jp/)

## 海外の法的背景

### アメリカ

- **ADA（Americans with Disabilities Act）** — 1990年制定の障害を持つアメリカ人法。Webサイトも対象と解釈される判例が増加
- **Section 508（リハビリテーション法508条）** — 連邦政府機関のICT（情報通信技術）に対するアクセシビリティ要件。WCAG 2.0 Level AAを基準として採用
- アメリカではWebアクセシビリティ関連の訴訟が年間数千件規模

### ヨーロッパ

- **European Accessibility Act（EAA）** — 2025年6月から施行されるEU全体のアクセシビリティ指令
- ECサイト、銀行、交通機関など幅広い民間サービスが対象
- **EN 301 549** — EU圏内のICT製品・サービスに対するアクセシビリティ規格。WCAG 2.1 Level AAを参照

### その他の国

- **カナダ** — Accessible Canada Act（2019年）
- **オーストラリア** — Disability Discrimination Act（DDA）
- **イギリス** — Equality Act 2010 + Public Sector Bodies Accessibility Regulations 2018
- グローバルにアクセシビリティ法制化の流れが加速

> 参照元
> - [ADA.gov](https://www.ada.gov/)
> - [Section508.gov](https://www.section508.gov/)
> - [European Accessibility Act](https://ec.europa.eu/social/main.jsp?catId=1202)

## アクセシビリティの4原則（POUR原則）

- WCAGの土台となる4つの原則
- すべてのガイドラインはこの4原則に基づいて分類される
- 詳細は別記事「WCAG 2.2の4原則 — 知覚可能・操作可能・理解可能・堅牢」を参照

| 原則 | 英語 | 概要 |
|---|---|---|
| **知覚可能** | Perceivable | 情報やUIコンポーネントをユーザーが知覚できる方法で提示すること |
| **操作可能** | Operable | UIコンポーネントやナビゲーションを操作可能にすること |
| **理解可能** | Understandable | 情報やUIの操作が理解可能であること |
| **堅牢** | Robust | 支援技術を含む多様なユーザーエージェントで確実に解釈できること |

> 参照元
> - [W3C WCAG 2.2 - 4 Principles](https://www.w3.org/TR/WCAG22/#perceivable)

## WCAGの概要

### WCAGとは

- Web Content Accessibility Guidelines の略称
- W3C WAIが策定するWebコンテンツのアクセシビリティに関する国際標準ガイドライン
- 世界中の法律・規格がWCAGを参照・採用

### バージョンの変遷

| バージョン | 公開年 | 特徴 |
|---|---|---|
| WCAG 1.0 | 1999年 | 初版。HTMLに特化した技術的なガイドライン |
| WCAG 2.0 | 2008年 | 技術非依存の原則ベースに刷新。ISO/IEC 40500として国際標準化 |
| WCAG 2.1 | 2018年 | モバイル、ロービジョン、認知障害への対応を追加（2.0の上位互換） |
| WCAG 2.2 | 2023年 | 認知障害・運動障害への対応をさらに強化。9つの新しい達成基準を追加 |
| WCAG 3.0 | 策定中 | 構造を根本から見直す次世代ガイドライン（W3C Working Draft） |

### 適合レベル

- **Level A** — 最低限のアクセシビリティ。対応しないと一部のユーザーがまったく利用できない
- **Level AA** — 一般的に目標とされるレベル。多くの法律・規格がこのレベルを要求
- **Level AAA** — 最も高いレベル。サイト全体での完全な適合は現実的に困難な場合がある

> 参照元
> - [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/)
> - [W3C WCAG 2.2 - Understanding Conformance](https://www.w3.org/WAI/WCAG22/Understanding/conformance)

## 支援技術の種類

- 障害のあるユーザーがWebを利用するために使うソフトウェアやハードウェアの総称

### スクリーンリーダー

- 画面の内容を音声で読み上げるソフトウェア
- 主に視覚障害のあるユーザーが使用
- 代表的な製品：NVDA（Windows/無料）、JAWS（Windows/有料）、VoiceOver（macOS・iOS/標準搭載）、TalkBack（Android/標準搭載）

### 画面拡大ソフト（スクリーンマグニファイア）

- 画面の一部または全体を拡大表示
- ロービジョンのユーザーが使用
- OS標準の拡大鏡機能、ZoomTextなど

### 音声入力

- 音声でコンピュータを操作する技術
- 運動障害のあるユーザーや手が塞がっている状況で使用
- Dragon NaturallySpeaking、OSの音声コントロール機能など

### スイッチデバイス

- 1つまたは少数のスイッチ（ボタン）でコンピュータを操作する入力装置
- 重度の運動障害のあるユーザーが使用
- 走査（スキャン）方式で画面上の項目を順番に選択

### 代替キーボード・代替ポインティングデバイス

- 標準のキーボードやマウスの代わりに使用する入力機器
- 大型キーボード、ヘッドポインタ、視線入力装置など

### 点字ディスプレイ

- 画面の内容を点字で出力する触覚デバイス
- 視覚と聴覚の両方に障害がある盲ろう者にとって不可欠

> 参照元
> - [W3C WAI - Tools and Techniques](https://www.w3.org/WAI/people-use-web/tools-techniques/)

## フロントエンドエンジニアが最初に取り組むべきこと

### 1. セマンティックHTMLを正しく使う

- 見た目ではなく意味に基づいて適切なHTML要素を選択
- これだけで多くのアクセシビリティ問題を未然に防止
- 詳細は次のセクション「良い例・悪い例」を参照

### 2. キーボードで操作できるか確認する

- Tabキーですべてのインタラクティブ要素にフォーカスが移動するか
- Enterキー・Spaceキーで操作できるか
- フォーカスの順序が論理的か
- フォーカスインジケーター（アウトライン）が視認できるか

### 3. 画像に適切なalt属性を設定する

- 装飾画像には `alt=""`（空のalt）を設定
- 情報を伝える画像には内容を説明するaltテキストを記述
- altを省略するのはNG（alt属性自体がない状態はスクリーンリーダーがファイル名を読み上げる原因）

### 4. カラーコントラスト比を確認する

- テキストと背景色のコントラスト比が WCAG AA 基準を満たすか確認
- 通常テキスト：4.5:1以上
- 大きなテキスト（18pt以上または14pt太字以上）：3:1以上

### 5. 自動テストツールを導入する

- axe DevTools（ブラウザ拡張機能）で手軽にチェック
- Lighthouse のアクセシビリティ監査を定期的に実行
- 自動テストで検出できるのは全体の約30〜40%。手動テストも必須

> 参照元
> - [W3C WAI - Tips for Getting Started](https://www.w3.org/WAI/tips/)
> - [Deque - axe DevTools](https://www.deque.com/axe/devtools/)

## 良い例・悪い例

### セマンティックHTMLの使用

```html
<!-- 悪い例：divとspanで見た目だけ整えたページ構造 -->
<div class="header">
  <span class="logo">サイト名</span>
  <div class="nav">
    <span class="nav-item" onclick="location.href='/'">ホーム</span>
    <span class="nav-item" onclick="location.href='/about'">概要</span>
  </div>
</div>
<div class="main">
  <div class="title">記事タイトル</div>
  <div class="text">本文テキスト...</div>
</div>
<div class="footer">
  <div class="copyright">© 2026</div>
</div>

<!-- 良い例：セマンティックなHTML要素を使用 -->
<header>
  <a href="/" class="logo">サイト名</a>
  <nav aria-label="メインナビゲーション">
    <ul>
      <li><a href="/">ホーム</a></li>
      <li><a href="/about">概要</a></li>
    </ul>
  </nav>
</header>
<main>
  <h1>記事タイトル</h1>
  <p>本文テキスト...</p>
</main>
<footer>
  <p><small>© 2026</small></p>
</footer>
```

- 悪い例の問題点：
  - スクリーンリーダーがページの構造（ヘッダー、ナビ、メイン、フッター）を認識できない
  - ランドマークナビゲーションが機能しない
  - spanにonclickを付けてもキーボードで操作できない
  - 見出し階層がないためページの概要を把握できない

### ボタンの実装

```html
<!-- 悪い例：divで作ったボタン -->
<div class="btn" onclick="handleSubmit()">送信</div>

<style>
.btn {
  padding: 8px 16px;
  background: #0066cc;
  color: #fff;
  cursor: pointer;
  display: inline-block;
  border-radius: 4px;
}
</style>

<!-- 良い例：button要素を使用 -->
<button type="submit" class="btn">送信</button>

<style>
.btn {
  padding: 8px 16px;
  background: #0066cc;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}
.btn:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}
</style>
```

- 悪い例の問題点：
  - Tabキーでフォーカスできない
  - Enterキー・Spaceキーで押下できない
  - スクリーンリーダーが「ボタン」と認識しない
  - フォーカスインジケーターがない

### フォーカスインジケーター

```css
/* 悪い例：フォーカスインジケーターを削除 */
*:focus {
  outline: none;
}

/* 良い例：カスタムフォーカスインジケーターを提供 */
*:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}
```

- 悪い例の問題点：
  - キーボードユーザーが現在どの要素にフォーカスしているか視認できない
  - WCAG 2.4.7（フォーカスの可視化）に違反

### 画像の代替テキスト

```html
<!-- 悪い例1：alt属性がない -->
<img src="chart.png">

<!-- 悪い例2：altが不適切 -->
<img src="chart.png" alt="画像">
<img src="chart.png" alt="chart.png">

<!-- 良い例：意味のある画像には説明的なaltを設定 -->
<img src="chart.png" alt="2025年の月別売上推移グラフ。6月に最高値120万円を記録">

<!-- 良い例：装飾画像にはalt=""（空文字）を設定 -->
<img src="decorative-line.png" alt="">
```

- 悪い例の問題点：
  - alt属性がないとスクリーンリーダーがファイル名（"chart.png"）を読み上げる
  - 「画像」だけでは何の画像か伝わらない
  - ファイル名をaltに入れても情報として無意味

## まとめ

- Webアクセシビリティは「障害のある人もWebを利用できるようにする」取り組み
- 永続的な障害だけでなく、一時的な障害や状況的な制約もカバーする概念
- 法的義務化の流れは日本を含め世界的に加速（日本は2024年4月に民間事業者の合理的配慮提供が義務化）
- WCAGはアクセシビリティの国際標準ガイドライン。Level AAが一般的な目標水準
- フロントエンドエンジニアはまず「セマンティックHTML」「キーボード操作」「alt属性」「コントラスト比」から着手
- アクセシビリティはすべてのユーザーの体験を向上させるもの
