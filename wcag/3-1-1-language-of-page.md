# 3.1.1 ページの言語（A）

## 達成基準の概要

- WCAG 2.2 達成基準 3.1.1「ページの言語」（Language of Page）
- レベル **A**（最低限の適合レベル）
- 原則3「理解可能（Understandable）」> ガイドライン 3.1「読みやすさ」に属する
- 各 Web ページの**デフォルトの自然言語**を、プログラムで判定できること
- HTML では、主に `<html lang="...">` で示す

> **参照**
> - [WCAG 2.2 — Success Criterion 3.1.1 Language of Page](https://www.w3.org/TR/WCAG22/#language-of-page)
> - [Understanding SC 3.1.1 Language of Page](https://www.w3.org/WAI/WCAG22/Understanding/language-of-page.html)

## 達成基準の原文（要約）

- 各 Web ページのデフォルトの自然言語が、プログラムで判定できること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.1.1](https://www.w3.org/TR/WCAG22/#language-of-page)

## なぜ必要か

- スクリーンリーダーが、正しい発音ルール・音声合成エンジンを選べる
- 点訳ソフトが、アクセント記号や Grade 2 点字の制御を適切に行える
- ブラウザが、文字・スクリプトの表示や辞書参照を正しく扱える
- キャプション再生でも、言語情報が役立つ
- 読字困難や認知・学習障害で TTS を使う人も、正しい言語で読み上げられる

> **参照**
> - [Understanding SC 3.1.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/language-of-page.html#benefits)

## デフォルトの自然言語とは

- ページ全体の**テキスト処理の既定言語**
- 複数言語がある場合は、**最も多く使われている言語**を指定する
- 同程度なら、最初に使われている言語を選ぶ
- ページ内の一部だけ別言語にするのは、3.1.2（一部分の言語）の範囲

```text
例：
  日本語が大半で、英語の引用が少しある → lang="ja"
  ドイツ語が大半で、英語セクションがある → lang="de"
  日英がほぼ同量で、先頭が日本語 → lang="ja"
```

> **参照**
> - [Understanding SC 3.1.1 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/language-of-page.html#intent)

## 手法：html 要素に lang を付ける（H57）

- **H57**：`<html>` 要素に `lang` 属性を付ける
- 値が、ページの主要言語を表す **BCP 47** の言語タグであること
- HTML ではこれだけで十分なことが多い

```html
<!-- 良い例：日本語ページ -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8">
    <title>会社概要 | 株式会社サンプル</title>
  </head>
  <body>
    <h1>会社概要</h1>
    <p>当社は東京都に本社を置くウェブ制作会社です。</p>
  </body>
</html>
```

```html
<!-- 良い例：英語ページ -->
<html lang="en">
```

```html
<!-- 悪い例：lang がない -->
<html>
```

```html
<!-- 悪い例：主要言語と一致しない -->
<html lang="en">
  <!-- 本文は日本語なのに en -->
</html>
```

> **参照**
> - [Technique H57: Using the language attribute on the HTML element](https://www.w3.org/WAI/WCAG22/Techniques/html/H57)
> - [BCP 47: Tags for Identifying Languages](https://www.rfc-editor.org/rfc/bcp/bcp47.txt)

## 言語タグの書き方

- 基本は言語サブタグ（例：`ja` / `en` / `zh`）
- 必要なら地域を足す（例：`en-US` / `pt-BR` / `zh-Hans`）
- 大文字小文字は慣習で `ja` / `en-US` のように書くが、照合は大文字小文字を区別しない
- 無効な値（空、`jp`、`japanese` など）は適合しない

```html
<!-- 良い例 -->
<html lang="ja">
<html lang="en">
<html lang="en-GB">
<html lang="zh-Hans">
```

```html
<!-- 悪い例 -->
<html lang="">
<html lang="jp">
<!-- 国コード JP であって、言語タグではない -->
<html lang="japanese">
<html lang="日本語">
```

```text
よく使うタグ：
  ja      日本語
  en      英語
  en-US   アメリカ英語
  zh-Hans 簡体字中国語
  zh-Hant 繁体字中国語
  ko      韓国語
```

> **参照**
> - [Declaring language in HTML (W3C Internationalization)](https://www.w3.org/International/questions/qa-html-language-declarations)
> - [HTML — The lang and xml:lang attributes](https://html.spec.whatwg.org/multipage/dom.html#the-lang-and-xml:lang-attributes)

## 多言語ページでの注意

- レベル A では、ページ全体のデフォルト言語が分かれば足りる
- ページ内の言語切り替えは 3.1.2（AA）
- それでも多言語サイトでは、3.1.2 も合わせて満たすことが強く推奨される
- SPA / フレームワークでも、`<html lang>` をルートで正しく出す

```html
<!-- 良い例：デフォルトは日本語。英語部分は 3.1.2 でマーク -->
<html lang="ja">
  <body>
    <p>製品のキャッチコピーは次のとおりです。</p>
    <blockquote lang="en">
      <p>Simply works.</p>
    </blockquote>
  </body>
</html>
```

```jsx
// 良い例：Next.js のルートレイアウトで lang を指定
export default function RootLayout({ children }) {
  return (
    <html lang="ja">
      <body>{children}</body>
    </html>
  );
}
```

```jsx
// 悪い例：lang を付け忘れる
export default function RootLayout({ children }) {
  return (
    <html>
      <body>{children}</body>
    </html>
  );
}
```

> **参照**
> - [WCAG 2.2 — Success Criterion 3.1.2 Language of Parts](https://www.w3.org/TR/WCAG22/#language-of-parts)

## HTTP ヘッダーとの関係（参考）

- **SVR5**：HTTP の `Content-Language` で既定言語を示す手法もある（助言的）
- HTML の `lang` の方が、ユーザーエージェント・支援技術で広く使われる
- 実務では `html lang` を必須と考えてよい

## よくある失敗例

- `<html>` に `lang` がない
- `lang="jp"` など、無効な言語タグ
- 日本語サイトなのに `lang="en"` のままテンプレートを流用
- 言語切り替え後も `<html lang>` が更新されない
- `xml:lang` だけ付けて `lang` がない（HTML では `lang` を使う）

```html
<!-- 失敗例：テンプレートの lang="en" を直し忘れ -->
<html lang="en">
  <title>お問い合わせ</title>
  <body>
    <h1>お問い合わせ</h1>
  </body>
</html>
```

```html
<!-- 改善例 -->
<html lang="ja">
  <title>お問い合わせ</title>
  ...
</html>
```

## テスト・確認方法

### 手動テスト

- ページソースまたは DevTools で `<html>` の `lang` を確認する
- 値が空・無効でないか確認する
- ページの主要言語と一致しているか確認する
- 言語切替があるサイトでは、切替後の `lang` も確認する

```text
チェックリスト：
1. <html> に lang があるか
2. 値は BCP 47 として有効か（ja / en など）
3. 主要コンテンツの言語と一致しているか
4. フレームワークのルートテンプレートで漏れていないか
5. 多言語部分は 3.1.2 も検討しているか
```

### 開発者ツールでの確認

```javascript
// Console：html の lang を確認
const html = document.documentElement;
console.log({
  lang: html.getAttribute("lang"),
  hasLang: html.hasAttribute("lang"),
  empty: html.getAttribute("lang") === "",
});
```

```bash
# 自動テストでもよく検出される項目
# 例：axe / Lighthouse の「html に lang 属性がない」
```

> **参照**
> - [ACT Rule — HTML page has lang attribute](https://www.w3.org/WAI/standards-guidelines/act/rules/b5c3f8/)
> - [ACT Rule — HTML page lang attribute has valid language tag](https://www.w3.org/WAI/standards-guidelines/act/rules/bf051a/)
> - [a11y/a11y-automated-testing.md — アクセシビリティ自動テスト](../a11y/a11y-automated-testing.md)

## まとめ

- 3.1.1 は、各ページの**デフォルト言語をプログラムで判定できる**ようにする達成基準（レベル A）
- HTML では `<html lang="ja">` のように、主要言語の BCP 47 タグを付ける
- スクリーンリーダーの発音や TTS・点訳・辞書機能が正しく動くための基礎
- ページ内の一部だけ別言語にするのは 3.1.2。多言語サイトでは両方を意識する
- テンプレート流用や SPA のルートで、`lang` の欠落・不一致が起きやすい
