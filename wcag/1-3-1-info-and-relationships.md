# 1.3.1 情報および関係性（A）

## 達成基準の概要

- WCAG 2.2 達成基準 1.3.1「情報及び関係性」（Info and Relationships）
- レベル A（最低限の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.3「適応可能」に属する
- **見た目や聴覚**で伝わっている情報・構造・関係性を、**プログラム的に判定可能**にする、または**テキストで利用可能**にする
- セマンティック HTML の根幹となる達成基準

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.1 Info and Relationships](https://www.w3.org/TR/WCAG22/#info-and-relationships)
> - [Understanding SC 1.3.1 Info and Relationships](https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html)

## 達成基準の原文（要約）

- **提示**によって伝えられる情報・構造・関係性が、**プログラム的に判定可能**であること、または**テキストで利用可能**であること
- スクリーンリーダーやユーザースタイルシートで表示形式が変わっても、意味が失われないようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.1](https://www.w3.org/TR/WCAG22/#info-and-relationships)

## 何を「プログラム的に判定可能」にするか

- 視覚的・聴覚的な手がかりで伝わっている構造と関係性
- 具体例
  - **見出し**：大きな太字フォント、段落との余白
  - **リスト**：箇条書き記号、インデント
  - **テーブル**：行・列の対応、ヘッダーとデータの関係
  - **フォーム**：ラベルと入力欄の対応、必須項目の表示、グループ化
  - **ナビゲーション**：関連リンクのまとまり
  - **引用**：斜体・インデント
  - **現在位置**：アクティブなメニュー項目の強調
- 聴覚的手がかり（チャイム、声のトーン変化等）も同様。テキストまたはプログラム的な情報で代替

> **参照**
> - [Understanding SC 1.3.1 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html#intent)

## なぜ必要か

- スクリーンリーダー利用者が、見た目のレイアウトなしに**構造を理解**するため
- ユーザースタイルシート・拡大表示・リフロー時も**意味が保持**されるため
- 点字ディスプレイ利用者が、色やレイアウトに依存しない情報を得るため
- 支援技術がコンテンツを**適応的に提示**するための基盤

> **参照**
> - [Understanding SC 1.3.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html#benefits)

## 1.4.1 色の使用との関係

- **色の値そのもの**をプログラム的に判定する必要はない
- 色で伝えている**情報**（必須項目、エラー、リンク等）は、色以外の手がかりでも伝わる必要がある
- 色に関する具体的要件は **1.4.1 色の使用（A）** が担当
- 1.3.1 では、色と**併用**されているテキスト・記号・アイコン等がプログラム的に関連付けられているかが焦点

```html
<!-- 1.3.1 + 1.4.1：色とテキストの両方で必須を示し、プログラム的にも関連付け -->
<label for="email">
  メールアドレス
  <span class="required" aria-hidden="true">*</span>
  <span class="sr-only">（必須）</span>
</label>
<input type="email" id="email" required aria-required="true">
<p>必須項目は <span aria-hidden="true">*</span> で示しています。</p>
```

> **参照**
> - [Understanding SC 1.3.1 — Note on color](https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html#intent)
> - [WCAG 2.2 — Success Criterion 1.4.1 Use of Color](https://www.w3.org/TR/WCAG22/#use-of-color)

## 見出し（Technique H42）

- 見出しは `<h1>`〜`<h6>` でマークアップ
- フォントサイズ・太字・余白だけで見出しを表現しない（Failure F2）
- 見出しレベルは**論理的な階層**を反映（飛ばしすぎない）

```html
<!-- 悪い例：見出しを div + CSS で表現（F2） -->
<div class="section-title">製品情報</div>
<p>当社の主力製品をご紹介します。</p>
<div class="subsection-title">機能一覧</div>

<!-- 良い例：見出し要素で構造を明示 -->
<h2>製品情報</h2>
<p>当社の主力製品をご紹介します。</p>
<h3>機能一覧</h3>
```

```html
<!-- 悪い例：見出しレベルの飛ばし（h2 の次に h4） -->
<h2>サービス</h2>
<h4>料金プラン</h4>

<!-- 良い例：論理的な階層 -->
<h2>サービス</h2>
<h3>料金プラン</h3>
```

> **参照**
> - [Technique H42: Using h1-h6 to identify headings](https://www.w3.org/WAI/WCAG22/Techniques/html/H42)
> - [Technique G141: Organizing a page using headings](https://www.w3.org/WAI/WCAG22/Techniques/general/G141)

## リスト（Technique H48）

- 箇条書き・番号付きリストは `<ul>` / `<ol>` / `<li>` を使用
- 定義リストは `<dl>` / `<dt>` / `<dd>`
- 改行や `-` 記号だけでリストを表現しない

```html
<!-- 悪い例：改行と記号だけでリスト -->
<p>
  対応ブラウザ：<br>
  - Chrome<br>
  - Firefox<br>
  - Safari
</p>

<!-- 良い例：ul 要素で構造を明示 -->
<p>対応ブラウザ：</p>
<ul>
  <li>Chrome</li>
  <li>Firefox</li>
  <li>Safari</li>
</ul>
```

```html
<!-- 悪い例：手順を p タグの羅列で -->
<p>1. アカウントを作成</p>
<p>2. メールを確認</p>
<p>3. ログイン</p>

<!-- 良い例：ol 要素で順序を明示 -->
<ol>
  <li>アカウントを作成</li>
  <li>メールを確認</li>
  <li>ログイン</li>
</ol>
```

> **参照**
> - [Technique H48: Using ol, ul and dl for lists](https://www.w3.org/WAI/WCAG22/Techniques/html/H48)

## データテーブル（Technique H51, H39, H63, H43）

- 表形式の情報は `<table>` でマークアップ
- レイアウト目的の `<table>` 使用は Failure F46
- データテーブルには `<caption>`、`<th>`、適切な `scope` または `headers` / `id` でヘッダーとセルの関係を明示

```html
<!-- 悪い例：div + CSS で表を再現（F2, F43） -->
<div class="table-row">
  <span>路線</span><span>始発</span><span>終電</span>
</div>
<div class="table-row">
  <span>1号線</span><span>5:00</span><span>0:30</span>
</div>

<!-- 良い例：table + th + scope -->
<table>
  <caption>地下鉄 始発・終電時刻</caption>
  <thead>
    <tr>
      <th scope="col">路線</th>
      <th scope="col">始発</th>
      <th scope="col">終電</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">1号線</th>
      <td>5:00</td>
      <td>0:30</td>
    </tr>
  </tbody>
</table>
```

### 複雑なテーブル（headers / id）

- 行・列ヘッダーが複数段ある場合、`headers` 属性でセルとヘッダーを関連付け

```html
<!-- 複雑なテーブル：headers で関連付け -->
<table>
  <caption>2024年 四半期別売上（万円）</caption>
  <thead>
    <tr>
      <th id="dept" scope="col">部門</th>
      <th id="q1" scope="col">Q1</th>
      <th id="q2" scope="col">Q2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="sales" scope="row" headers="dept">営業</th>
      <td headers="sales q1">1200</td>
      <td headers="sales q2">1500</td>
    </tr>
  </tbody>
</table>
```

> **参照**
> - [Technique H51: Using table markup to present tabular information](https://www.w3.org/WAI/WCAG22/Techniques/html/H51)
> - [Technique H39: Using caption elements](https://www.w3.org/WAI/WCAG22/Techniques/html/H39)
> - [Technique H63: Using the scope attribute](https://www.w3.org/WAI/WCAG22/Techniques/html/H63)
> - [Technique H43: Using id and headers attributes](https://www.w3.org/WAI/WCAG22/Techniques/html/H43)

## フォーム（Technique H44, H71, H85）

### ラベルと入力欄の関連付け（H44）

- すべてのフォームコントロールに `<label>` を関連付け
- `for` / `id` の対応、または `<label>` で入力要素を囲む
- プレースホルダーのみをラベル代わりにしない（Failure F111）

```html
<!-- 悪い例：ラベルなし、placeholder のみ -->
<input type="text" placeholder="お名前">

<!-- 悪い例：視覚的ラベルはあるがプログラム的に関連付けなし（F111） -->
<span>お名前</span>
<input type="text" id="name">

<!-- 良い例：label と input を関連付け -->
<label for="name">お名前</label>
<input type="text" id="name" name="name">
```

### グループ化（H71）

- 関連するフォーム要素は `<fieldset>` + `<legend>` でグループ化
- ラジオボタン・チェックボックス群に特に重要

```html
<!-- 悪い例：グループ化なし -->
<p>配送方法</p>
<label><input type="radio" name="ship" value="std"> 通常</label>
<label><input type="radio" name="ship" value="exp"> 速達</label>

<!-- 良い例：fieldset + legend -->
<fieldset>
  <legend>配送方法</legend>
  <label><input type="radio" name="ship" value="std"> 通常</label>
  <label><input type="radio" name="ship" value="exp"> 速達</label>
</fieldset>
```

### select の optgroup（H85）

```html
<!-- 良い例：optgroup で選択肢をグループ化 -->
<label for="city">都市</label>
<select id="city" name="city">
  <optgroup label="関東">
    <option value="tokyo">東京</option>
    <option value="yokohama">横浜</option>
  </optgroup>
  <optgroup label="関西">
    <option value="osaka">大阪</option>
    <option value="kyoto">京都</option>
  </optgroup>
</select>
```

> **参照**
> - [Technique H44: Using label elements to associate text labels with form controls](https://www.w3.org/WAI/WCAG22/Techniques/html/H44)
> - [Technique H71: Using fieldset and legend](https://www.w3.org/WAI/WCAG22/Techniques/html/H71)
> - [Technique H85: Using optgroup](https://www.w3.org/WAI/WCAG22/Techniques/html/H85)
> - [a11y/accessible-form.md — フォームのアクセシビリティ](../a11y/accessible-form.md)

## ナビゲーション（Technique H97, ARIA26）

### nav 要素

- 主要なナビゲーションリンク群は `<nav>` で囲む
- ページ内に `<nav>` が複数ある場合、`aria-label` で区別

```html
<!-- 悪い例：div だけでナビゲーション -->
<div class="menu">
  <a href="/">ホーム</a>
  <a href="/about">会社概要</a>
  <a href="/contact">お問い合わせ</a>
</div>

<!-- 良い例：nav 要素で役割を明示 -->
<nav aria-label="メインナビゲーション">
  <ul>
    <li><a href="/">ホーム</a></li>
    <li><a href="/about">会社概要</a></li>
    <li><a href="/contact">お問い合わせ</a></li>
  </ul>
</nav>
```

### aria-current

- 現在のページ・項目を `aria-current="page"` 等で示す

```html
<!-- 良い例：現在位置をプログラム的に明示 -->
<nav aria-label="メインナビゲーション">
  <ul>
    <li><a href="/">ホーム</a></li>
    <li><a href="/about" aria-current="page">会社概要</a></li>
    <li><a href="/contact">お問い合わせ</a></li>
  </ul>
</nav>
```

> **参照**
> - [Technique H97: Grouping related links using the nav element](https://www.w3.org/WAI/WCAG22/Techniques/html/H97)
> - [Technique ARIA26: Using aria-current to identify the current item](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA26)

## 色の手がかりとセマンティックマークアップ（G138）

- 色で意味を伝える場合、**テキスト・記号・アイコン**と**プログラム的な関連付け**を併用
- 必須項目：`*` + `aria-required`、エラー：`aria-invalid` + テキストメッセージ

```html
<!-- 悪い例：色（赤）だけで必須を示す -->
<style>.required-label { color: red; }</style>
<label class="required-label" for="email">メールアドレス</label>
<input type="email" id="email">

<!-- 良い例：色 + 記号 + プログラム的関連付け + 説明テキスト -->
<p>必須項目は <span aria-hidden="true">*</span> で示しています。</p>
<label for="email">
  メールアドレス <span aria-hidden="true">*</span>
  <span class="sr-only">（必須）</span>
</label>
<input type="email" id="email" required aria-required="true">
```

> **参照**
> - [Technique G138: Using semantic markup whenever color cues are used](https://www.w3.org/WAI/WCAG22/Techniques/general/G138)
> - [Technique ARIA2: Identifying a required field with aria-required](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA2)

## セマンティック HTML が使えない場合（Situation B）

- 技術的にセマンティック構造を提供できない場合、**標準的なテキスト書式規則**で構造を伝える
- プレーンテキスト、一部の PDF、レガシーシステム等
- 見出し・リスト・段落の**テキスト上の規約**（空行、`*`、番号等）で構造を示す
- 説明テキストは**対象情報の近く**に配置（ページが線形化されたとき）

```text
<!-- プレーンテキストでの構造（T1, T2, T3） -->

製品マニュアル

第1章 はじめに

本章では製品の概要を説明します。

第2章 セットアップ

* パッケージの開封
* 電源の接続
* 初期設定

必須項目は * で示しています。
```

> **参照**
> - [Technique T1: Using standard text formatting conventions for paragraphs](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/T1)
> - [Technique T2: Using standard text formatting conventions for lists](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/T2)
> - [Technique T3: Using standard text formatting conventions for headings](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/T3)

## role="presentation" の注意（Failure F92）

- 意味を持つコンテンツに `role="presentation"` / `role="none"` を付与すると、支援技術が構造を無視する
- レイアウト目的の `<table>` には使ってもよいが、**データの関係性**を伝えるテーブルには不可

```html
<!-- F92：データテーブルに role="presentation" -->
<table role="presentation">
  <tr><th>商品</th><th>価格</th></tr>
  <tr><td>りんご</td><td>150円</td></tr>
</table>

<!-- 良い例：レイアウト目的のみに presentation -->
<table role="presentation">
  <tr>
    <td><img src="logo.png" alt="会社ロゴ"></td>
    <td><h1>株式会社サンプル</h1></td>
  </tr>
</table>
```

> **参照**
> - [Failure F92: Using role presentation on content which conveys semantic information](https://www.w3.org/WAI/WCAG22/Techniques/failures/F92)

## 代表的な失敗例（Failures）

| Failure | 内容 |
|---|---|
| F2 | テキストの見た目変更（太字・大きさ等）だけで情報を伝え、適切なマークアップを使わない |
| F33 | プレーンテキストで空白文字だけ使い複数カラムを作成 |
| F34 | プレーンテキストで空白文字だけ使い表を作成 |
| F42 | `<a>` 以外の要素でリンクをエミュレート |
| F43 | 構造マークアップを内容の関係性を表さない形で使用 |
| F46 | レイアウト目的で `<table>` / `<th>` を使用 |
| F48 | `<pre>` で表形式情報をマークアップ |
| F90 | `headers` / `id` でテーブルヘッダーとセルの関連付けが誤り |
| F91 | テーブルヘッダーのマークアップが不正 |
| F92 | 意味を持つコンテンツに `role="presentation"` |
| F111 | 視覚的ラベルはあるが accessible name が空 |

```html
<!-- F2：太字だけで見出し -->
<p><strong>重要なお知らせ</strong></p>

<!-- F42：span + onclick でリンクを模倣 -->
<span onclick="location.href='/about'" style="color: blue; text-decoration: underline">
  会社概要
</span>

<!-- F46：レイアウト用 table -->
<table>
  <tr>
    <td><nav>...</nav></td>
    <td><main>...</main></td>
  </tr>
</table>
<!-- → CSS Grid / Flexbox を使用すべき -->
```

> **参照**
> - [WCAG 2.2 Techniques — Failures for 1.3.1](https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html#failures)

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.3.2 意味のある順序（A） | DOM 順序が読み上げ順序と一致するか（構造の**順序**） |
| 1.3.3 感覚的な特徴（A） | 「右のボタンを押す」等、感覚だけに依存した指示の禁止 |
| 1.4.1 色の使用（A） | 色だけで情報を伝えない |
| 4.1.2 名前・役割・値（A） | インタラクティブ要素の name / role / value |

- 1.3.1 は**構造と関係性**、1.3.2 は**順序**、4.1.2 は**コントロールの識別**
- フォームでは 1.3.1（ラベル関連付け）+ 3.3.2（ラベルまたは説明）+ 4.1.2 が重なる

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.2 Meaningful Sequence](https://www.w3.org/TR/WCAG22/#meaningful-sequence)
> - [WCAG 2.2 — Success Criterion 4.1.2 Name, Role, Value](https://www.w3.org/TR/WCAG22/#name-role-value)

## テスト・確認方法

### 自動テスト

- 見出し階層の妥当性（h1 の有無、レベルの飛ばし）
- フォーム要素への `<label>` 関連付け
- データテーブルの `<th>` / `scope` / `headers`
- リスト要素の適切な使用
- axe DevTools、WAVE、Lighthouse 等で検出可能

### 手動テスト

- **CSS を無効化**または**線形化**し、構造が理解できるか
- スクリーンリーダーの**見出し一覧**・**ランドマーク一覧**で構造を確認
- テーブルを行・列ナビゲーションで読み、ヘッダーとデータの対応が正しいか
- フォーム入力時にラベル・グループ名（legend）が読み上げられるか
- 色だけで伝わっている情報が、色なしでも理解できるか

```bash
# 見出し構造を確認（開発者ツール Console）
[...document.querySelectorAll('h1,h2,h3,h4,h5,h6')].map(h => `${h.tagName}: ${h.textContent.trim()}`);

# label 未関連付けの input を検出
[...document.querySelectorAll('input,select,textarea')].filter(el => {
  const id = el.id;
  return !el.labels?.length && !el.closest('label') && !el.getAttribute('aria-label') && !el.getAttribute('aria-labelledby');
});
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.3.1 は**見た目で伝わる構造・関係性**をプログラム的に利用可能にする達成基準（レベル A）
- 見出し（h1-h6）、リスト（ul/ol/dl）、テーブル（table/th/scope）、フォーム（label/fieldset）が中核
- セマンティック HTML を優先。CSS の見た目だけに依存しない（Failure F2）
- 色の手がかりはテキスト・記号と併用し、プログラム的にも関連付ける
- セマンティック構造が使えない場合は、テキスト規約 + 近傍の説明で代替
- 1.3.2（順序）、1.4.1（色）、4.1.2（名前・役割・値）と併せて確認
