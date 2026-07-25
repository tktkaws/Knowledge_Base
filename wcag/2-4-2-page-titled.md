# 2.4.2 ページタイトル（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.2「ページタイトル」（Page Titled）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- Webページに、**話題または目的を説明するタイトル**があること
- タブ・ウィンドウ・検索結果・サイトマップなどで、ページを識別しやすくする

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.2 Page Titled](https://www.w3.org/TR/WCAG22/#page-titled)
> - [Understanding SC 2.4.2 Page Titled](https://www.w3.org/WAI/WCAG22/Understanding/page-titled.html)

## 達成基準の原文（要約）

- Webページには、話題または目的を説明するタイトルがあること

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.2](https://www.w3.org/TR/WCAG22/#page-titled)

## なぜ必要か

- タイトルだけで、そのページが自分に必要か判断しやすくなる
- 視覚障害のある人は、複数タブを開いたときにタイトルでページを区別する
- 認知障害・短期記憶の制約・読字困難のある人も、タイトルで内容を識別しやすい
- 音声でナビする重度の運動障害のある人も、ページ間移動時にタイトルを頼りにする
- 検索結果やブックマーク一覧でも、説明的なタイトルが役立つ

> **参照**
> - [Understanding SC 2.4.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/page-titled.html#benefits)

## 良いタイトルの条件

- そのページの**話題または目的**が分かること
- 空でないこと
- サイト内の他ページと区別できること
- 文書名やアプリ名だけでも目的が分かれば足りる場合がある
- リンク文言とタイトルを近づけると、遷移後の連続性が高まる（良い習慣）

```text
良い例：
  お問い合わせ | 株式会社サンプル
  2024年12月の明細 | Bank XYZ
  Alert Dialog Example | APG | WAI | W3C

悪い例：
  （空の title）
  Untitled Document
  ページ1 / page / home（どのページか分からない）
  株式会社サンプル（全ページ同じ）
```

> **参照**
> - [Understanding SC 2.4.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/page-titled.html#intent)

## 手法：説明的なタイトルを付ける（G88 + H25）

- **G88**：ページに説明的なタイトルを提供する
- **H25**：HTML の `<title>` 要素でタイトルを関連付ける
- `<head>` 内に、空でない `<title>` を置く
- ユーザーエージェントはタブ名・ウィンドウタイトルなどに表示する

```html
<!-- 良い例：話題が分かる title -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8">
    <title>採用情報 | 株式会社サンプル</title>
  </head>
  <body>
    <h1>採用情報</h1>
  </body>
</html>
```

```html
<!-- 悪い例：空、または内容を示さない -->
<title></title>
<title>Untitled</title>
<title>株式会社サンプル</title>
<!-- 全ページで同じサイト名だけ -->
```

> **参照**
> - [Technique G88: Providing descriptive titles for web pages](https://www.w3.org/WAI/WCAG22/Techniques/general/G88)
> - [Technique H25: Providing a title using the title element](https://www.w3.org/WAI/WCAG22/Techniques/html/H25)

## ページ集合での一貫した命名（G127）

- 必須ではないが、大きなサイトではページ間の関係が分かるとよい
- よくある形：`個別ページ名 | セクション | サイト名`
- 重要な情報を前に置くと、タブ幅が狭いときも識別しやすい

```html
<!-- 良い例：集合内の位置が分かる -->
<title>Patterns | APG | WAI | W3C</title>
<title>Alert Dialog Example | APG | WAI | W3C</title>
```

```text
命名の例：
  商品詳細ページ：ワイヤレスイヤホン X1 | 商品 | ショップ名
  設定画面：通知設定 | アカウント | アプリ名
```

> **参照**
> - [Technique G127: Identifying a web page's relationship to a larger collection of web pages](https://www.w3.org/WAI/WCAG22/Techniques/general/G127)
> - [Understanding SC 2.4.2 — set of web pages example](https://www.w3.org/WAI/WCAG22/Understanding/page-titled.html#examples)

## SPA・動的な画面更新

- SPA では URI が同じでも、表示内容（ビュー）が変わることがある
- ビューが変わったら、**`document.title` も更新**する
- ルートごとのタイトルをルーティング設定で管理するとよい
- 文書アプリや Web アプリでも、現在の文書・画面をタイトルに反映する

```javascript
// 良い例：ルート変更時に title を更新
function setPageTitle(viewTitle) {
  document.title = `${viewTitle} | サンプルアプリ`;
}

// 例：口座一覧へ遷移
setPageTitle("口座一覧");
// => "口座一覧 | サンプルアプリ"

// 例：明細画面へ遷移
setPageTitle("2024年12月の明細");
// => "2024年12月の明細 | サンプルアプリ"
```

```javascript
// 悪い例：画面が変わっても title が固定
document.title = "サンプルアプリ";
// どの画面にいるかタブからは分からない
```

```html
<!-- React / Next.js などの概念例 -->
<title>{pageTitle} | サンプルアプリ</title>
```

> **参照**
> - [Understanding SC 2.4.2 — SPA note](https://www.w3.org/WAI/WCAG22/Understanding/page-titled.html#intent)
> - [Understanding SC 2.4.2 — web application example](https://www.w3.org/WAI/WCAG22/Understanding/page-titled.html#examples)

## よくある失敗（F25）

- タイトルがページ内容を識別しない
- 全ページで同じタイトル
- プレースホルダーや自動生成の無意味な文字列
- `<title>` 自体がない、または空

```html
<!-- 失敗例：内容を識別できない -->
<title>ホーム</title>
<!-- サイト内に「ホーム」が複数ある、または中身がホームではない -->

<title>document</title>
<title>new page</title>
```

```html
<!-- 改善例 -->
<title>お知らせ一覧 | 株式会社サンプル</title>
<title>料金プラン | 株式会社サンプル</title>
```

> **参照**
> - [Failure F25: title of a web page not identifying the contents](https://www.w3.org/WAI/WCAG22/Techniques/failures/F25)

## 見出し（h1）との関係

- `<title>` と `<h1>` は役割が異なる
- `<title>`：タブ・ブックマーク・検索結果など、ページ外でも使われる識別名
- `<h1>`：ページ本文内の主見出し
- 一致または近い内容にすると分かりやすいが、完全一致は必須ではない
- `<h1>` だけでは 2.4.2 は満たせない（`<title>` が必要）

```html
<!-- 良い例：title と h1 が近く、役割が分かれている -->
<title>プライバシーポリシー | 株式会社サンプル</title>
<body>
  <h1>プライバシーポリシー</h1>
</body>
```

## 2.4.4・2.4.9 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.2 ページタイトル | 到着したページの識別 | ページ側の名前 |
| 2.4.4 リンクの目的（コンテキスト内） | リンク先の目的が文脈から分かる | リンク側 |
| 2.4.9 リンクの目的（リンクのみ） | リンク文言だけで目的が分かる（AAA） | より厳しいリンク要件 |

- リンク先が文書やアプリなら、その名前でリンク目的を示せる場合がある
- リンク文言とページタイトルを揃えると、クリック前後の連続性がよい

```html
<!-- 良い習慣：リンクと title が近い -->
<a href="/contact">お問い合わせ</a>
<!-- 遷移先 -->
<title>お問い合わせ | 株式会社サンプル</title>
```

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.4 Link Purpose (In Context)](https://www.w3.org/TR/WCAG22/#link-purpose-in-context)
> - [WCAG 2.2 — Success Criterion 2.4.9 Link Purpose (Link Only)](https://www.w3.org/TR/WCAG22/#link-purpose-link-only)

## 実装時の注意点

- すべての HTML ページに `<title>` を置く
- CMS やテンプレートのデフォルト「無題」を残さない
- 重要な識別情報をタイトルの前方に置く
- 長すぎるとタブで切れるため、冗長な定型句は後ろへ
- PDF などの場合は文書プロパティの Title を設定する（PDF18）
- iframe 内ページにも、必要に応じて適切な title を付ける

```text
実務チェック：
  トップ・下層・フォーム・エラー・検索結果で title が異なるか
  SPA の各ビューで title が更新されるか
  ログイン後の個人向け画面でも識別できるか
```

## テスト・確認方法

### 手動テスト

- ブラウザのタブ名を確認する
- ページソースまたは DevTools で `<title>` の有無と内容を確認する
- 複数ページを開き、タブだけで区別できるか確認する
- SPA では画面遷移後にタブ名が変わるか確認する
- スクリーンリーダーでページ読み込み時にタイトルが読み上げられるか確認する

```text
チェックリスト：
1. <title> があるか（空でないか）
2. 話題または目的が分かるか
3. 他ページと区別できるか
4. 全ページで同じタイトルになっていないか
5. 動的画面では title が更新されるか
```

### 開発者ツールでの確認

```bash
# title 要素や document.title の更新を探す
rg "<title>|document\\.title|useTitle|Helmet" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.vue" --glob "*.php"
```

```javascript
// Console で現在のタイトルを確認
document.title
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.2 は、各 Web ページに**話題または目的を説明するタイトル**を求める達成基準（レベル A）
- HTML では `<title>`（H25）で、説明的なタイトル（G88）を提供する
- 全ページ同一・空・Untitled などは F25 の失敗になりやすい
- SPA ではビュー変更に合わせて `document.title` を更新する
- リンク目的（2.4.4 / 2.4.9）や見出し（h1）と併せて、一貫した命名を心がける
