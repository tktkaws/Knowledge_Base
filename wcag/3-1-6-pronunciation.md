# 3.1.6 発音（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.1.6「発音」（Pronunciation）
- レベル **AAA**（最高の適合レベル）
- 原則3「理解可能（Understandable）」> ガイドライン 3.1「読みやすさ」に属する
- **文脈だけでは意味が曖昧**で、発音（読み）が分からないと通じない語について、特定の発音を識別できる仕組みがあること
- 日本語では漢字の読みが典型。`<ruby>` や直後のかながよく使われる
- 文脈で読みが分かれば、マークは不要

> **参照**
> - [WCAG 2.2 — Success Criterion 3.1.6 Pronunciation](https://www.w3.org/TR/WCAG22/#pronunciation)
> - [Understanding SC 3.1.6 Pronunciation](https://www.w3.org/WAI/WCAG22/Understanding/pronunciation.html)

## 達成基準の原文（要約）

- 文脈において、発音を知らなければ意味が曖昧になる語について、特定の発音を識別する仕組みがあること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.1.6](https://www.w3.org/TR/WCAG22/#pronunciation)

## なぜ必要か

- 同じ表記で読みと意味が複数ある語がある（英語の heteronym、日本語の漢字）
- 視覚的には文脈で補えても、スクリーンリーダーが違う読みをすると通じない
- 読字障害や、文脈から意味を取りにくい人にも、読みの手がかりが必要
- 日本語では、人名や難読漢字が特に起きやすい

> **参照**
> - [Understanding SC 3.1.6 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/pronunciation.html#benefits)

## 対象になるとき / ならないとき

- 対象：発音（読み）が分からないと、その文での意味が決まらない
- 対象外：文脈から正しい読みと意味が十分分かる
- 言語に詳しくない人向けの発音ガイドは、この基準の目的ではない
- 日本語では「発音」より **読み** を示す、と Understanding でも注記されている

```text
対象になりやすい例：
  人名の漢字（高橋、南海 など、複数の読みがあり文脈だけでは決まらない）
  同じ漢字で読みが違うと意味が変わる語
  日中韓で同じ文字だが読みが違う引用

対象になりにくい例：
  「砂漠の desert」のように、文脈で読みが一意になる英語
  常用的で、その文では読みが明らかな漢字
```

```text
英語の heteronym の例：
  desert（見捨てる）と desert（砂漠）
  文脈で分かれば不要。分からなければ読みの仕組みが必要
```

> **参照**
> - [Understanding SC 3.1.6 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/pronunciation.html#intent)

## 手法1：語の直後に読みを置く（G120）

- 少なくとも初出の直後に、読みを示す
- 同じ綴りで読みが複数あるページでは、出現ごとに示す
- 括弧のかなが、日本語でよく使う形

```html
<!-- 良い例：人名の読みを直後に置く -->
<p>慶應大学（けいおうだいがく）</p>
<p>執筆者は高橋（たかはし）です。</p>
```

```html
<!-- 悪い例：読みが複数ある人名に手がかりがない -->
<p>執筆者は南海です。</p>
<!-- なんかい / みなみうみ など、文脈だけでは決まらないことがある -->
```

> **参照**
> - [Technique G120: Providing the pronunciation immediately following the word](https://www.w3.org/WAI/WCAG22/Techniques/general/G120)

## 手法2：ruby 要素を使う（H62）

- **H62**：`<ruby>` でベースの文字に読みを付ける
- `<rt>` が読み。`<rp>` は ruby 非対応環境向けの括弧
- 日本語では漢字の上（横書き）または右（縦書き）にかなが出る
- スクリーンリーダーは、漢字のあとでかなをもう一度読むことがある
- HTML では `<rb>` は必須ではない。ベーステキストを `<ruby>` の中に直接書いてよい

```html
<!-- 良い例：現代 HTML でよく使う形 -->
<p>
  <ruby>慶應大学<rp>（</rp><rt>けいおうだいがく</rt><rp>）</rp></ruby>
</p>
```

```html
<!-- 良い例：H62 の rb を使った形 -->
<p>
  <ruby>
    <rb>慶應大学</rb>
    <rp>(</rp>
    <rt>けいおうだいがく</rt>
    <rp>)</rp>
  </ruby>
</p>
```

```html
<!-- 悪い例：見た目だけルビで、読みがマークアップにない -->
<p>
  <span class="with-ruby">慶應大学</span>
</p>
<!-- CSS の ::before でかなが出るだけだと、支援技術に伝わらない -->
```

> **参照**
> - [Technique H62: Using the ruby element](https://www.w3.org/WAI/WCAG22/Techniques/html/H62)
> - [HTML — The ruby element](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-ruby-element)
> - [CSS Ruby Annotation Layout Module](https://www.w3.org/TR/css-ruby-1/)

## 手法3：読みへリンクする / 用語集に載せる

- **G121**：発音（読み）の説明や音声へリンクする
- **G62**：用語集に、読みが意味を左右する語の発音情報を載せる
- 音声ファイルへリンクする方法もある

```html
<!-- 良い例：用語集の読みへリンク -->
<p>
  <a href="#term-nankai">南海</a>
  の研究を紹介します。
</p>

<h2>用語集</h2>
<dl>
  <dt id="term-nankai">南海</dt>
  <dd>読み：なんかい。ここでは地域名として使う。</dd>
</dl>
```

```html
<!-- 良い例：音声で読みを示す -->
<p>
  <a href="/audio/takahashi.mp3">高橋の読みを聞く</a>
</p>
```

> **参照**
> - [Technique G121: Linking to pronunciations](https://www.w3.org/WAI/WCAG22/Techniques/general/G121)
> - [Technique G62: Providing a glossary](https://www.w3.org/WAI/WCAG22/Techniques/general/G62)

## 手法4：標準の発音記号（G163）

- ダイアクリティカルマークなど、標準の発音記号を使う
- オフにできること。常時表示だと、その記号が読めない人の妨げになりうる
- ヘブライ語など、フォント側の記号で読みを示す言語で使われることが多い
- 日本語の漢字読みでは、ruby や括弧のかなの方が一般的

> **参照**
> - [Technique G163: Using standard diacritical marks that can be turned off](https://www.w3.org/WAI/WCAG22/Techniques/general/G163)

## 日中韓の文字が混ざるとき

- 同じ漢字でも、日本語・中国語・韓国語で読みが違う
- 日本語サイトに漢文や韓国語の引用があるときは、正しい読みを示す
- 言語そのものは 3.1.2（`lang`）でもマークする

```html
<!-- 良い例：引用の言語と読みの両方 -->
<blockquote lang="zh">
  <p>
    <ruby>人<rt>rén</rt></ruby>
    <!-- 日本語の「ひと」ではなく、中国語の読みを示す -->
  </p>
</blockquote>
```

> **参照**
> - [wcag/3-1-2-language-of-parts.md — 3.1.2 一部分の言語](./3-1-2-language-of-parts.md)

## 3.1.3 / 3.1.4 との関係

| 達成基準 | 対象 |
|---|---|
| 3.1.3 一般的でない用語 | 意味の定義 |
| 3.1.4 略語 | 略語の展開 |
| 3.1.6 発音 | 読みが分からないと意味が曖昧な語 |

- 定義があっても、読みが複数なら 3.1.6 の対象になりうる
- 略語の読み（WCAG をどう読むか）に ruby を使う例もある（H62）

## よくある失敗例

- 人名や難読語に読みがなく、文脈でも決まらない
- CSS だけでルビを見せて、DOM に読みがない
- 同じ漢字で読みが複数あるのに、初出にしか読みがない
- スクリーンリーダー向けに `aria-label` だけで読みを足し、目で見る人には分からない
- すべての漢字にルビを付けて、かえって読みにくくする（必要な箇所に絞る）

```html
<!-- 失敗例：支援技術にだけ読みがある -->
<span aria-label="たかはし">高橋</span>
```

```html
<!-- 改善例：誰でも読みが分かる -->
<ruby>高橋<rp>（</rp><rt>たかはし</rt><rp>）</rp></ruby>
```

## テスト・確認方法

### 手動テスト

- 同じ表記で読みが複数ある語、人名、難読漢字を洗い出す
- 文脈だけで読みと意味が決まるか判断する
- 決まらないなら、ruby・括弧・用語集・音声のいずれかがあるか確認する
- 読みがマークアップに含まれているか（見た目だけになっていないか）確認する

```text
チェックリスト：
1. 読みが分からないと意味が曖昧な語はあるか
2. 文脈だけで十分か（十分ならマーク不要）
3. 不十分なら、ruby / 直後のかな / 用語集 / 音声があるか
4. CSS だけのルビになっていないか
5. 読みが複数なら、出現ごとに辿れるか
6. 別言語の引用なら lang と読みの両方があるか
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.1.6 は、**読みが分からないと意味が曖昧な語**に、発音（読み）を辿る仕組みを求める達成基準（レベル AAA）
- 文脈で分かれば不要。日本語では人名や難読漢字が典型
- `<ruby>` か、直後の「（かな）」が扱いやすい
- 見た目だけのルビや、`aria-label` だけでは、すべての利用者に届かない
- 日本語では「発音」より **読み** を示す、という理解で実装する
