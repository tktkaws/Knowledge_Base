# 2.4.6 見出しおよびラベル（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.6「見出しおよびラベル」（Headings and Labels）
- レベル **AA**
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- 見出しとラベルが、**話題または目的を説明**していること
- ページの内容と構成を把握しやすくする

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.6 Headings and Labels](https://www.w3.org/TR/WCAG22/#headings-and-labels)
> - [Understanding SC 2.4.6 Headings and Labels](https://www.w3.org/WAI/WCAG22/Understanding/headings-and-labels.html)

## 達成基準の原文（要約）

- 見出しおよびラベルは、話題または目的を説明すること

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.6](https://www.w3.org/TR/WCAG22/#headings-and-labels)

## この基準が求めること・求めないこと

| 求めること | 求めないこと |
|---|---|
| 見出しやラベルがある場合、内容が説明的であること | 見出しやラベルの**存在そのもの** |
| 話題・目的が分かる文言であること | 正しいマークアップ（→ 1.3.1） |
| フォームラベルが入力内容を正確に示すこと | 入力へのラベル必須（→ 3.3.2） |
| | アクセシブルネームの有無（→ 4.1.2） |

- 見出しやラベルが**あれば**、説明的でなければならない
- マークアップの正しさは 1.3.1、ラベルの有無は 3.3.2、名前付けは 4.1.2 が担当
- 1.3.1 に適合しても、見出し文言が曖昧なら 2.4.6 は不適合になりうる
- 逆に、説明的な見出しでも、マークアップが誤っていれば 1.3.1 は不適合になりうる

```text
2.4.6 のみ不適合の例：
  <h2>詳細</h2>（何の詳細か分からない）
  <label>入力</label>（何を入力するか分からない）

1.3.1 のみ不適合の例：
  <div class="heading">料金プラン</div>
  （文言は説明的だが、見出しとしてマークアップされていない）
```

> **参照**
> - [Understanding SC 2.4.6 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/headings-and-labels.html#intent)

## なぜ必要か

- 読字が遅いユーザーや短期記憶に制約のあるユーザーは、見出しで内容を予測できると助かる
- フォームでは、何を入力すべきかがラベルから分かると完了しやすい
- スクリーンリーダーの見出し一覧・目次でも、説明的な見出しが役立つ
- Web では全文を読むよりスキャンすることが多いため、見出しの質が探索効率に直結する

> **参照**
> - [Understanding SC 2.4.6 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/headings-and-labels.html#benefits)

## ラベルとは

- コンテンツ内の部品を識別するために、ユーザーへ提示されるテキスト（またはテキスト代替のある部品）
- HTML の `<label>` 要素に限定されない
- **ラベル**はすべてのユーザーに提示される
- **名前（name）**は支援技術向けで、隠されている場合がある
- 多くの場合、ラベルと名前は一致する

```text
ラベルの例：
  「姓」「名」のテキスト
  検索ボタン上の虫眼鏡アイコン（代替テキスト「検索」付き）
  セクションを示す見出し文言

名前だけでラベルがない例：
  aria-label のみで、画面上に対応する文言がない
  → 4.1.2 は満たしうるが、2.4.6 の「ラベル」としては不十分な場合がある
```

> **参照**
> - [Understanding SC 2.4.6 — Key Terms: label / name](https://www.w3.org/WAI/WCAG22/Understanding/headings-and-labels.html#dfn-label)

## 手法1：説明的な見出し（G130）

- 各見出しが、そのセクションの話題や目的を示す
- 長くする必要はない。1語でも適切な手がかりになればよい
- 曖昧な「詳細」「その他」「情報」は避け、具体化する
- 記事間で共通の見出し構造があると、一貫して探しやすい

```html
<!-- 悪い例：曖昧な見出し -->
<h2>詳細</h2>
<h2>情報</h2>
<h2>その他</h2>

<!-- 良い例：話題が分かる見出し -->
<h2>配送料とお届け日数</h2>
<h2>返品・交換について</h2>
<h2>よくある質問</h2>
```

```html
<!-- 良い例：ニュースの見出しが記事内容を示す -->
<article>
  <h2>市役所が新庁舎の完成予想図を公開</h2>
  <p>市は本日、新庁舎の完成予想図を公開した。...</p>
  <a href="/news/city-hall">全文を読む</a>
</article>
```

```html
<!-- 良い例：文書の構成が見出しに反映されている -->
<h1>うまく書く方法</h1>
<h2>無駄な言葉を削る</h2>
<h2>不要な言葉を見つける</h2>
```

> **参照**
> - [Technique G130: Providing descriptive headings](https://www.w3.org/WAI/WCAG22/Techniques/general/G130)
> - [Understanding SC 2.4.6 — news / writing guide examples](https://www.w3.org/WAI/WCAG22/Understanding/headings-and-labels.html#examples)

## 手法2：説明的なラベル（G131）

- フォームコントロールなどのラベルが、期待される入力や操作を正確に示す
- 「名前」だけより、「姓」「名」のように区別すると分かりやすい
- 画像ラベルも、文脈で広く理解されるなら説明的とみなせる

```html
<!-- 悪い例：曖昧なラベル -->
<label for="field1">入力</label>
<input id="field1" name="name">

<!-- 良い例：何を入れるか分かる -->
<label for="given-name">名</label>
<input id="given-name" name="given-name" autocomplete="given-name">

<label for="family-name">姓</label>
<input id="family-name" name="family-name" autocomplete="family-name">
```

```html
<!-- 良い例：虫眼鏡アイコンが検索ラベルとして通用する -->
<label for="q" class="visually-hidden">サイト内検索</label>
<input id="q" type="search" name="q">
<button type="submit">
  <img src="/icons/search.svg" alt="検索">
</button>
```

```text
同じ虫眼鏡でも文脈で意味が変わる：
  検索欄の横 → 検索を実行する
  画像の上 → 拡大表示する
いずれも、その文脈で広く理解されるなら説明的になりうる
```

> **参照**
> - [Technique G131: Providing descriptive labels](https://www.w3.org/WAI/WCAG22/Techniques/general/G131)
> - [Understanding SC 2.4.6 — form / search icon examples](https://www.w3.org/WAI/WCAG22/Understanding/headings-and-labels.html#examples)

## 一貫した見出し構造の例

- 論文やレポートのように、共通セクションがある場合は見出しを揃えるとよい
- ページタイトルで記事の一意性を示しつつ、見出しで構成の一貫性を保つ
- 2.4.2（ページタイトル）や 3.2.4（一貫した識別）とも相性がよい

```text
良い例（会議論文サイト）：
  各記事ページの title：記事名 | 会議名
  共通見出し：要約 / はじめに / 結論 / 著者紹介 / 用語集 / 参考文献
  記事固有の見出し：その論文だけの中間セクション
```

> **参照**
> - [Understanding SC 2.4.6 — consistent headings example](https://www.w3.org/WAI/WCAG22/Understanding/headings-and-labels.html#examples)
> - [wcag/2-4-2-page-titled.md — 2.4.2 ページタイトル](./2-4-2-page-titled.md)

## 1.3.1・3.3.2・4.1.2・2.4.10 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 1.3.1 情報および関係性 | 見出し・ラベルのマークアップ | 「正しく識別されているか」 |
| 2.4.6 見出しおよびラベル | 見出し・ラベルの説明性 | 「文言が分かりやすいか」 |
| 2.4.10 セクション見出し | セクション見出しの提供（AAA） | 「見出しがあるか」 |
| 3.3.2 ラベルまたは説明 | 入力へのラベル／説明の提供 | 「ラベルがあるか」 |
| 4.1.2 名前・役割・値 | アクセシブルネームなど | 「プログラム上の名前があるか」 |

```text
よくある誤解：
  aria-label があるから 2.4.6 は合格
  → 名前はあるが、ユーザーに見える説明的ラベルとは限らない

  h2 があるから 2.4.6 は合格
  → マークアップはあるが、文言が曖昧なら不合格
```

> **参照**
> - [wcag/1-3-1-info-and-relationships.md — 1.3.1 情報および関係性](./1-3-1-info-and-relationships.md)
> - [WCAG 2.2 — Success Criterion 3.3.2 Labels or Instructions](https://www.w3.org/TR/WCAG22/#labels-or-instructions)
> - [WCAG 2.2 — Success Criterion 2.4.10 Section Headings](https://www.w3.org/TR/WCAG22/#section-headings)

## よくある失敗例

- 「詳細」「その他」「クリック」など、内容を予測できない見出し
- 「入力欄1」「項目」など、何を入れるか分からないラベル
- 複数セクションで同じ曖昧な見出しが並ぶ
- 見えるラベルは曖昧なのに、`aria-label` だけ具体的（見た目と支援技術で差が出る）
- プレースホルダだけをラベル代わりにして、消えると目的が分からなくなる

```html
<!-- 悪い例：プレースホルダだけ -->
<input type="email" name="email" placeholder="メールアドレス">

<!-- 良い例：残るラベルがある -->
<label for="email">メールアドレス</label>
<input id="email" type="email" name="email" autocomplete="email"
  placeholder="例：taro@example.com">
```

```html
<!-- 悪い例：見えるラベルが曖昧 -->
<label for="tel">電話</label>
<input id="tel" name="tel">

<!-- 良い例：目的が具体的 -->
<label for="tel">日中連絡が取れる電話番号</label>
<input id="tel" name="tel" type="tel" autocomplete="tel">
```

## 実装時の注意点

- 見出しは短くてよいが、スキャンしたときに内容が予測できること
- フォームラベルは、入力形式や対象者の違いが分かるようにする
- 画像ラベルは、その文脈で広く理解されるか確認する
- 1.3.1 とセットで、説明的かつ正しくマークアップする
- 見出し一覧を読み上げて、ページ構成が伝わるか確認する

```text
実務チェック：
  見出し一覧だけ見て、各セクションの内容が想像できるか
  ラベルだけ見て、何を入力／操作すべきか分かるか
  「詳細」「その他」「入力」が残っていないか
```

## テスト・確認方法

### 手動テスト

- ページの見出しを一覧する（ブラウザ拡張やスクリーンリーダー）
- 各見出しから、セクション内容が予測できるか確認する
- フォームの各コントロールのラベルが、入力内容を示しているか確認する
- アイコンだけのラベルは、文脈で意味が通るか確認する
- 1.3.1 / 3.3.2 / 4.1.2 も併せて確認する

```text
チェックリスト：
1. 見出しがある場合、話題や目的が分かるか
2. ラベルがある場合、対象の部品の目的が分かるか
3. 曖昧な「詳細」「その他」「入力」がないか
4. 見えるラベルと支援技術向けの名前が大きく食い違っていないか
5. マークアップ（1.3.1）やラベル有無（3.3.2）も同時に問題ないか
```

### 開発者ツールでの確認

```bash
# 見出し・ラベルまわりを確認
rg "<h[1-6]|<label|aria-label|placeholder=" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.php"
```

```javascript
// Console：見出し一覧を確認
[...document.querySelectorAll("h1,h2,h3,h4,h5,h6,[role='heading']")]
  .map((el) => `${el.tagName.toLowerCase()}: ${el.textContent.trim()}`)
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.6 は、見出しとラベルが**話題または目的を説明**していることを求める達成基準（レベル AA）
- 見出しやラベルの「存在」や「正しいマークアップ」は、この基準だけでは求めない
- 主な手法は、説明的な見出し（G130）と説明的なラベル（G131）
- 1.3.1（構造）・3.3.2（ラベルの提供）・4.1.2（名前）・2.4.10（セクション見出しの提供）と役割を分けて理解する
- 短くてよいが、スキャンしたときに内容が予測できることが重要
