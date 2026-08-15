# 3.1.2 一部分の言語（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.1.2「一部分の言語」（Language of Parts）
- レベル **AA**
- 原則3「理解可能（Understandable）」> ガイドライン 3.1「読みやすさ」に属する
- コンテンツ中の各パッセージ・フレーズの自然言語を、プログラムで判定できること
- HTML では、該当箇所に `lang` を付ける
- 例外：固有名詞 / 専門用語 / 言語が特定できない語 / 周囲の言語に定着した語

> **参照**
> - [WCAG 2.2 — Success Criterion 3.1.2 Language of Parts](https://www.w3.org/TR/WCAG22/#language-of-parts)
> - [Understanding SC 3.1.2 Language of Parts](https://www.w3.org/WAI/WCAG22/Understanding/language-of-parts.html)

## 達成基準の原文（要約）

- コンテンツ中の各パッセージまたはフレーズの自然言語が、プログラムで判定できること
- ただし次は除く
  - **固有名詞**（proper names）
  - **専門用語**（technical terms）
  - **言語が特定できない語**
  - **直後の周囲テキストの口語・慣用に定着した語や句**

> **参照**
> - [WCAG 2.2 — Success Criterion 3.1.2](https://www.w3.org/TR/WCAG22/#language-of-parts)

## なぜ必要か

- スクリーンリーダーが、その箇所の発音ルール・音声に切り替えられる
- 点訳ソフトが、言語切替に合わせて制御コードを変えられる
- 左右の読み方向（LTR / RTL）や別文字体系の表示が正しくなる
- 辞書参照や機械翻訳も、言語切替を手がかりにできる
- 読字困難や TTS 利用でも、別言語の箇所が正しく読まれる

> **参照**
> - [Understanding SC 3.1.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/language-of-parts.html#benefits)

## 対象になるもの

- 1語でも、別言語として意図されているなら対象になりうる
- 指定がなければ、その箇所の言語は **3.1.1 のページ既定言語** になる
- 単一言語のページは、ページの `lang` だけで全体が判定できる

```text
対象になりやすい例：
  日本語ページ内の英語の引用文
  言語切替リンク（Deutsch / Français など、その言語での表記）
  英語ページ内のドイツ語フレーズ

対象になりにくい例：
  固有名詞（Albert Camus、商品名・地名）
  専門用語（hertz、habeas corpus）
  周囲に定着した外来語（日本語の「ウェブ」、フランス語文中の podcast）
```

> **参照**
> - [Understanding SC 3.1.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/language-of-parts.html#intent)

## 手法：言語が変わる箇所に lang を付ける（H58）

- **H58**：`lang` 属性で自然言語の変化を示す
- 値は **BCP 47**（`ja` / `en` / `de` / `zh-Hant` など）
- 引用なら `blockquote`、短い語なら `span` など、該当要素に付ける
- `lang` は子孫に継承される。内側で上書きできる

```html
<!-- 良い例：日本語ページ内の英語引用 -->
<html lang="ja">
  <p>製品のキャッチコピーは次のとおりです。</p>
  <blockquote lang="en">
    <p>Simply works.</p>
  </blockquote>
</html>
```

```html
<!-- 良い例：英語文中のドイツ語フレーズ -->
<html lang="en">
  <p>
    He called it a
    <span lang="de">Treppenwitz der Weltgeschichte</span>.
  </p>
</html>
```

```html
<!-- 悪い例：別言語の一節なのに lang がない -->
<html lang="ja">
  <blockquote>
    <p>Simply works.</p>
  </blockquote>
</html>
```

> **参照**
> - [Technique H58: Using language attributes to identify changes in the human language](https://www.w3.org/WAI/WCAG22/Techniques/html/H58)

## 例外の見分け方

### 固有名詞

- 人名・地名・ブランド名など、指示対象を特定する名前
- 翻訳せずに使うことが多く、言語切替のマークは必須ではない
- 例：英語の哲学記事で `Albert Camus` に `lang="fr"` は不要

### 専門用語

- 職業上よく使い、各国語に訳されないことが多い語
- 例：`Homo sapiens` / `hertz` / `habeas corpus`

### 周囲に定着した語

- 1語だけ言語が変わったように見えても、実は周囲の言語に取り込まれていることが多い
- 疑わしいときは、周囲の言語でも（アクセントを除き）同じように発音されるか考える
- 例：フランス語文中の `podcast`、日本語文中の「Web」「CSS」

```html
<!-- 例外：定着した語なので lang は必須ではない -->
<p>このページの CSS を調整してください。</p>
```

```html
<!-- 例外ではない：明確に別言語として引用している -->
<p>
  仏語では車を
  <span lang="fr">voiture</span>
  と言います。
</p>
```

### ベストプラクティス

- 例外でも、発音を正確にしたいなら `lang` を付けるとよい
- 文字体系が違う語（ピンインなど）は、特に指定すると読み上げに効く

```html
<!-- 推奨：借りてきた語句にも lang を付ける -->
<p>
  <span lang="fr">faux pas</span>
  にならないよう、lang を付けましょう。
</p>

<p>
  Good
  <span lang="zh-Latn-pinyin">feng shui</span>
  begins with good markup.
</p>
```

> **参照**
> - [Understanding SC 3.1.2 — Intent（固有名詞・定着語・ベストプラクティス）](https://www.w3.org/WAI/WCAG22/Understanding/language-of-parts.html#intent)

## 言語切替リンク

- リンク文言がその言語自身の名前なら、リンクに `lang` を付ける
- 属性値と要素内容が別言語なら、内容側に `lang` を付ける

```html
<!-- 良い例：各リンクをその言語でマーク -->
<ul>
  <li><a href="/de" lang="de">Deutsch</a></li>
  <li><a href="/fr" lang="fr">Français</a></li>
  <li><a href="/zh-hant" lang="zh-Hant">繁體中文</a></li>
</ul>
```

```html
<!-- 良い例：表示はスペイン語、title は英語 -->
<a title="Spanish" href="/es">
  <span lang="es">Español</span>
</a>
```

```html
<!-- 悪い例：全部がページ既定言語のまま -->
<ul>
  <li><a href="/de">Deutsch</a></li>
  <li><a href="/fr">Français</a></li>
</ul>
```

## アクセシブルネームとの注意

- スクリーンリーダーは、リンク一覧などで **計算されたアクセシブルネーム** を読むことがある
- アクセシブルネームはフラットな文字列で、内側の `lang` が伝わらないことがある
- マークアップ上正しく `lang` があれば、読み上げが切り替わらなくても **この達成基準の不合格にはならない**
- 単純なケースでは、子ではなく要素自身に `lang` を付けると通る場合がある
- テストは読み上げだけに頼らず、マークアップを確認する

```html
<!-- より伝わりやすい例：要素自身に lang -->
<a href="/fr" lang="fr">Français</a>
```

```html
<!-- マークアップとしては正しいが、リンク一覧では fr が落ちることがある -->
<a href="/fr">
  <span lang="fr">Français</span>
</a>
```

> **参照**
> - [Understanding SC 3.1.2 — Intent（accessible names）](https://www.w3.org/WAI/WCAG22/Understanding/language-of-parts.html#intent)

## 3.1.1 との関係

| 達成基準 | レベル | 対象 |
|---|---|---|
| 3.1.1 ページの言語 | A | ページ全体のデフォルト言語（`<html lang>`） |
| 3.1.2 一部分の言語 | AA | ページ内で言語が変わるパッセージ・フレーズ |

- 3.1.2 は 3.1.1 の上に乗る。ページの既定言語がないと、部分の判定も崩れる
- 単一言語なら 3.1.1 だけで足りることが多い
- 多言語サイトでは両方を満たす

> **参照**
> - [wcag/3-1-1-language-of-page.md — 3.1.1 ページの言語](./3-1-1-language-of-page.md)

## よくある失敗例

- 別言語の引用・キャッチコピーに `lang` がない
- 言語切替リンクに `lang` がない
- `lang="jp"` など無効なタグ
- ページ既定と違う言語なのに、`lang` の値が間違っている
- テキストノードではなく、CSS の `::before` だけで別言語を出してマークできない

```html
<!-- 失敗例 -->
<html lang="ja">
  <p>公式スローガン：Think different.</p>
</html>
```

```html
<!-- 改善例 -->
<html lang="ja">
  <p>公式スローガン：<span lang="en">Think different.</span></p>
</html>
```

## テスト・確認方法

### 手動テスト

- ページの既定言語（`<html lang>`）を先に確認する
- 本文から別言語の一節・リンクを洗い出す
- それぞれに有効な `lang` があるか確認する
- 固有名詞・定着語は例外として切り分ける
- 読み上げの切替有無だけで合否を決めない

```text
チェックリスト：
1. ページ既定の lang はあるか（3.1.1）
2. 別言語のパッセージ・フレーズはあるか
3. あるなら、該当要素に有効な BCP 47 の lang があるか
4. 値はその箇所の言語と一致しているか
5. 固有名詞・専門用語・定着語なら例外としてよいか
6. 言語切替リンクはその言語でマークされているか
```

### 開発者ツールでの確認

```javascript
// Console：lang が付いている要素を一覧する
[...document.querySelectorAll("[lang]")].map((el) => ({
  tag: el.tagName.toLowerCase(),
  lang: el.getAttribute("lang"),
  text: el.textContent.trim().slice(0, 40),
}));
```

> **参照**
> - [ACT Rule — Element with lang attribute has valid language tag](https://www.w3.org/WAI/standards-guidelines/act/rules/de46e4/)
> - [ACT Rule — HTML element language subtag matches language](https://www.w3.org/WAI/standards-guidelines/act/rules/ff89c9/)
> - [a11y/a11y-automated-testing.md — アクセシビリティ自動テスト](../a11y/a11y-automated-testing.md)

## まとめ

- 3.1.2 は、ページ内の各パッセージ・フレーズの言語をプログラムで判定できるようにする達成基準（レベル AA）
- 言語が変わる箇所に `lang` を付ける（H58）
- 固有名詞、専門用語、定着した外来語は例外
- 例外でも、発音を正確にしたいなら `lang` を付けるのが望ましい
- ページ全体の既定言語は 3.1.1。両方そろえて初めて多言語が正しく伝わる
