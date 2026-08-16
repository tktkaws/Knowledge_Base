# 3.1.4 略語（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.1.4「略語」（Abbreviations）
- レベル **AAA**（最高の適合レベル）
- 原則3「理解可能（Understandable）」> ガイドライン 3.1「読みやすさ」に属する
- 略語の**展開形または意味**を特定できる仕組みがあること
- 頭字語（acronym）・イニシャリズム（initialism）も含む
- すでにその言語の語として定着したものは対象外（例：laser）

> **参照**
> - [WCAG 2.2 — Success Criterion 3.1.4 Abbreviations](https://www.w3.org/TR/WCAG22/#abbreviations)
> - [Understanding SC 3.1.4 Abbreviations](https://www.w3.org/WAI/WCAG22/Understanding/abbreviations.html)

## 達成基準の原文（要約）

- 略語の展開形または意味を識別する仕組みがあること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.1.4](https://www.w3.org/TR/WCAG22/#abbreviations)

## なぜ必要か

- 認知・言語・学習障害のある人は、短縮形の復号が難しいことがある
- 画面拡大では前後の文脈が見えにくく、略語の意味が取りづらい
- 記憶の制約があると、初出の展開を覚えておけない
- 同じ綴りが別の意味を持つと、文脈判断がさらに難しい
- 発音規則に乗らない略語は、読み上げでも誤解されやすい

> **参照**
> - [Understanding SC 3.1.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/abbreviations.html#benefits)

## 対象になる略語

- 語・句・名称の短縮形で、**まだその言語の語として定着していないもの**
- **イニシャリズム**：各語の頭文字などから作る短縮形（例：W3C、BBC）
- **頭字語（acronym）**：頭文字などを単語として読む短縮形（例：JAWS、NATO）
- 一般的・馴染みがある略語でも、定着語でなければ免除されない
- 会社名・商標が語として使われているものは、略語とみなさないことがある（例：IBM、NASA、Ecma）

```text
対象になりやすい例：
  W3C / WCAG / UNHCR
  英語の rm.（room）
  同じページの Dr.（Doctor）と Dr.（Drive）
  製品名の頭字語で、普通の語と同じ綴り（JAWS）

対象外になりやすい例：
  laser（元は LASER だが、今は普通の語）
  辞書に単語として載っている短縮形
  語として定着した組織名・商標（IBM、NASA など）
```

> **参照**
> - [Understanding SC 3.1.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/abbreviations.html#intent)

## 略語が分かりにくい理由

- 普通の語に見えない／発音規則に乗らない（`rm.` を room と読めない）
- 同じ略語が文脈で別の意味になる（`Dr. Johnson lives on Boswell Dr.`）
- 頭字語が普通の語と同じ綴りだが、別の使い方（スクリーンリーダーの JAWS）
- 発音は普通の語に似て、綴りが違う（SMIL を smile のように読む）

## 初出だけか、毎回か

- **ページ内で意味が1つ**なら、初出で展開すれば足りる（毎回でもよい）
- **同じページで意味が複数**なら、出現のたびに展開または説明が必要
- 複数リソースを1ページにまとめるなら、リソース先頭ごとに展開した方が分かりやすいこともある

```text
初出で足りる例：
  そのページでは常に「W3C = World Wide Web Consortium」

毎回必要な例：
  同じページに
    Dr. = Doctor
    Dr. = Drive（通り）
  が混在する
```

> **参照**
> - [Understanding SC 3.1.4 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/abbreviations.html)

## 手法1：初出の直前または直後に展開する（G97 + G102）

- **G102**：略語の展開または説明を提供する
- **G97**：初出の直前または直後に展開形を置く
- 英語では「正式名称のあと括弧で略語」が多い。言語によって慣習は異なる
- 展開ではなく、説明が必要な略語もある（G97 だけでは足りない）

```html
<!-- 良い例：正式名称のあとに略語 -->
<p>
  Web Content Accessibility Guidelines（WCAG）は、
  Web コンテンツのアクセシビリティに関する国際的なガイドラインです。
</p>
```

```html
<!-- 良い例：略語のあとに展開（G97 の例） -->
<p>
  The WAI（Web Accessibility Initiative）demonstrates
  the W3C commitment to accessibility.
</p>
```

```html
<!-- 悪い例：初出から略語だけ -->
<p>WCAG 2.2 の 3.1.4 を確認してください。</p>
```

> **参照**
> - [Technique G97: Providing the first use of an abbreviation immediately before or after the expanded form](https://www.w3.org/WAI/WCAG22/Techniques/general/G97)
> - [Technique G102: Providing the expansion or explanation of an abbreviation](https://www.w3.org/WAI/WCAG22/Techniques/general/G102)

## 手法2：定義へリンクする / 用語集を置く

- **G55**：定義へリンクする
- **G62**：用語集を提供する
- **G70**：オンライン辞書を検索できる
- 略語一覧をサイト独自の辞書として置く方法もある

```html
<!-- 良い例：本文から用語集へ -->
<p>
  <a href="#abbr-wcag"><abbr>WCAG</abbr></a>
  は、Web コンテンツのアクセシビリティガイドラインです。
</p>

<h2>略語一覧</h2>
<dl>
  <dt id="abbr-wcag">WCAG</dt>
  <dd>Web Content Accessibility Guidelines</dd>
  <dt id="abbr-wai">WAI</dt>
  <dd>Web Accessibility Initiative</dd>
</dl>
```

```html
<!-- 悪い例：用語集はあるが、本文から辿れない -->
<p>WCAG と WAI の関係を説明します。</p>
<!-- ページ末尾に一覧があるが、リンクも案内もない -->
```

> **参照**
> - [Technique G55: Linking to definitions](https://www.w3.org/WAI/WCAG22/Techniques/general/G55)
> - [Technique G62: Providing a glossary](https://www.w3.org/WAI/WCAG22/Techniques/general/G62)

## `<abbr>` について（H28・助言的）

- **H28**：`<abbr>` で略語の定義を示す手法。かつては十分な手法だったが、**いまは助言的**
- よくある書き方は `<abbr title="展開">略語</abbr>`
- `title` はマウスのツールチップ依存になりやすく、キーボード・タッチでは辿れないことが多い
- マークとして `<abbr>` を使うのはよい。展開は**見えるテキスト**でも示す

```html
<!-- 助言：意味のマークにはなるが、title だけでは不十分なことが多い -->
<p>
  砂糖は普通 5
  <abbr title="pound">lb.</abbr>
  入りで売られる。
</p>
```

```html
<!-- より確実：見える展開 + 任意で abbr -->
<p>
  Cascading Style Sheets（<abbr>CSS</abbr>）の色モジュールが更新されました。
</p>
```

> **参照**
> - [Technique H28: Providing definitions for abbreviations by using the abbr element](https://www.w3.org/WAI/WCAG22/Techniques/html/H28)
> - [HTML — The abbr element](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-abbr-element)

## 3.1.3 との関係

| 達成基準 | レベル | 対象 |
|---|---|---|
| 3.1.3 一般的でない用語 | AAA | 慣用句・専門用語・限定された意味の語 |
| 3.1.4 略語 | AAA | 略語の展開形または意味 |

- 略語の「何の短縮か」は 3.1.4
- その語の専門的な意味の説明は 3.1.3
- 両方に当てはまることもある（専門分野の頭字語に、展開と定義の両方を付ける）

> **参照**
> - [wcag/3-1-3-unusual-words.md — 3.1.3 一般的でない用語](./3-1-3-unusual-words.md)

## よくある失敗例

- 初出から略語だけで、展開がどこにもない
- `<abbr title>` だけに頼る
- 同じページで意味が複数あるのに、初出の展開だけ
- 用語集があるが、本文からリンクしていない
- 定着語でない「よく使う略語」を省略してよいと誤解する

```html
<!-- 失敗例 -->
<p>
  SPA では SSR のあとにハイドレーションが走ります。
</p>
```

```html
<!-- 改善例：初出で展開 -->
<p>
  Single Page Application（SPA）では、
  Server-Side Rendering（SSR）のあとにハイドレーションが走ります。
</p>
```

## テスト・確認方法

### 手動テスト

- ページ内の略語・頭字語・イニシャリズムを洗い出す
- 定着語（laser、IBM など）は対象外として切り分ける
- 意味が1つか複数かを確認する
- 初出（または毎回）に、見える展開・用語集・辞書のいずれかがあるか確認する
- `title` だけになっていないか確認する

```text
チェックリスト：
1. 略語・頭字語はあるか
2. 言語に定着した語としてよいか（対象外か）
3. ページ内の意味は1つか、複数か
4. 初出（複数なら毎回）に展開または説明があるか
5. 仕組みはキーボード・タッチでも辿れるか
6. 専門的な意味の説明が必要なら 3.1.3 も見ているか
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.1.4 は、略語の**展開形または意味**を特定できる仕組みを求める達成基準（レベル AAA）
- 初出の直前・直後に正式名称を置く方法が分かりやすい（G97）
- ページ内で意味が複数なら、出現ごとに展開が必要
- `<abbr title>` だけでは不十分なことが多い。見えるテキストで展開する
- すでに普通の語になったもの（laser など）は対象外。よく使う略語でも、定着していなければ対象
