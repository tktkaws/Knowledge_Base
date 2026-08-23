# 3.2.4 一貫した識別性（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.2.4「一貫した識別性」（Consistent Identification）
- レベル **AA**
- 原則3「理解可能（Understandable）」> ガイドライン 3.2「予測可能」に属する
- **Webページの集合内**で、同じ機能を持つコンポーネントは、**一貫して識別**されること
- ラベル・名前・テキストによる代替が、同じ機能では同じパターンで使われること
- 「一貫」は「同一」とは限らない（「4ページへ」→「5ページへ」は一貫）

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.4 Consistent Identification](https://www.w3.org/TR/WCAG22/#consistent-identification)
> - [Understanding SC 3.2.4 Consistent Identification](https://www.w3.org/WAI/WCAG22/Understanding/consistent-identification.html)

## 達成基準の原文（要約）

- Webページの集合内で、同じ機能を持つコンポーネントは、一貫して識別されること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.4](https://www.w3.org/TR/WCAG22/#consistent-identification)

## なぜ必要か

- スクリーンリーダー利用者は、ページをまたいで同じ機能名を頼りに操作する
- 同じ機能なのにラベルが違うと、別の操作だと誤解しやすい
- 認知の制約がある人は、覚えるべき名前が増え、負荷が上がる
- テキストによる代替が一定だと、検索や予測がしやすい
- 非テキストコンテンツを手がかりにする人も、同じ見た目・同じ意味だと理解しやすい

> **参照**
> - [Understanding SC 3.2.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/consistent-identification.html#benefits)

## 同じ機能とは

- 使ったときの**結果が同じ**であること
- 見た目が似ていても、結果が違えば別機能
- 同じページに2つ、別ページに1つ、合わせて3つあるなら、**3つとも**一貫させる

```text
同じ機能の例：
  各ページの「検索」ボタン（用語検索をして一覧を出す）
  全ページ共通の「お問い合わせ」リンク
  保存アイコン（ページを保存する）

別機能の例：
  チェックマークが「承認」と「含まれる」で使い分けられる
  「領収書を印刷」と「請求書を印刷」（対象が違う）
```

> **参照**
> - [Understanding SC 3.2.4 — Key Terms: same functionality](https://www.w3.org/WAI/WCAG22/Understanding/consistent-identification.html#dfn-same-functionality)

## 一貫と同一の違い

- **一貫**：同じパターン・同じ言い回しで識別できる
- **同一**：文字列が完全に同じ
- この基準が求めるのは一貫。同一である必要はない

```text
一貫している例：
  「2ページへ」→「3ページへ」→「4ページへ」
  「領収書を印刷」と「請求書を印刷」（「○を印刷」のパターン）
  文書アイコンの代替テキストが常に「ダウンロード：文書名」

一貫していない例：
  あるページは「検索」、別ページは「探す」（同じ検索機能）
  あるページは「保存」、別ページは「ページを保存」
  見た目は同じボタンなのに、aria-label だけがページごとに違う
```

> **参照**
> - [Understanding SC 3.2.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/consistent-identification.html#intent)

## 対象範囲

- **Webページの集合内**で、複数ページに繰り返し現れる同じ機能が対象
- 単一ページ内だけの一貫性は、この基準の必須ではない（ベストプラクティスではある）
- 言語が異なる版は、別の集合として扱う
- ラベル・アクセシブルネーム・テキストによる代替が対象

```text
対象になりやすい例：
  ヘッダーの検索ボタン
  共通の「閉じる」「ヘルプ」「保存」アイコン
  記事の「次へ／前へ」リンクの言い方
  フッターの「お問い合わせ」リンク文言

対象外になりやすい例：
  そのページにしかない操作
  別言語サイトの同じ機能（別集合）
  機能が違うのに見た目が似ているアイコン（別ラベルでよい）
```

## 手法：同じ機能には同じラベル・名前・代替テキスト（G197）

- **G197**：同じ機能のコンテンツに、ラベル・名前・テキストによる代替を一貫して使う
- 1.1.1（非テキストコンテンツ）と 4.1.2（名前・役割・値）の十分な手法も併せて満たす
- デザインシステムや共通コンポーネントで文言を1か所にまとめると守りやすい

```html
<!-- 良い例：検索は常に「検索」 -->
<form role="search" action="/search">
  <label for="q">検索</label>
  <input id="q" name="q" type="search">
  <button type="submit">検索</button>
</form>
```

```html
<!-- 悪い例：ページによって検索ボタンの文言が違う -->
<!-- トップ -->
<button type="submit">検索</button>
<!-- ブログ一覧 -->
<button type="submit">探す</button>
```

```html
<!-- 良い例：文書ダウンロードの代替テキストが一貫 -->
<a href="/docs/guide.pdf">
  <img src="/icons/document.svg" alt="ダウンロード：利用ガイド">
</a>
<a href="/docs/terms.pdf">
  <img src="/icons/document.svg" alt="ダウンロード：利用規約">
</a>
```

```html
<!-- 良い例：機能が違うチェックマークは、代替テキストも分ける -->
<img src="/icons/check.svg" alt="承認済み">
<img src="/icons/check.svg" alt="含まれる">
```

```jsx
// 良い例：共通コンポーネントでラベルを固定
export function SearchButton() {
  return <button type="submit">検索</button>;
}

export function ContactLink() {
  return <a href="/contact">お問い合わせ</a>;
}
```

```jsx
// 悪い例：見た目は同じだが、アクセシブルネームだけ違う
function CloseButtonA() {
  return <button type="button" aria-label="閉じる">×</button>;
}
function CloseButtonB() {
  return <button type="button" aria-label="ウィンドウを閉じる">×</button>;
}
```

> **参照**
> - [Technique G197: Using labels, names, and text alternatives consistently for content that has the same functionality](https://www.w3.org/WAI/WCAG22/Techniques/general/G197)
> - [wcag/1-1-1-non-text-content.md — 1.1.1 非テキストコンテンツ](./1-1-1-non-text-content.md)

## アイコンと隣接リンク

- 同じ行き先のアイコンリンクとテキストリンクが並ぶ場合、両方のリンク文言は一貫させる
- ベストプラクティスは、1つのリンクにまとめる（H2）
- ソース上分かれていても、識別が一致していればこの基準は満たしうる

```html
<!-- より望ましい：1つのリンクにまとめる -->
<a href="/contact">
  <img src="/icons/mail.svg" alt="">
  お問い合わせ
</a>
```

```html
<!-- 許容されうる：文言が一貫している -->
<a href="/contact"><img src="/icons/mail.svg" alt="お問い合わせ"></a>
<a href="/contact">お問い合わせ</a>
```

## 3.2.3 との関係

| 達成基準 | レベル | 対象 |
|---|---|---|
| 3.2.3 一貫したナビゲーション | AA | 繰り返されるナビの**相対順序** |
| 3.2.4 一貫した識別性 | AA | 同じ機能の部品の**ラベル・名前・代替テキスト** |

- 3.2.3 は「どこにあるか」
- 3.2.4 は「何と呼ぶか」
- 両方そろうと、サイト全体が予測しやすくなる

> **参照**
> - [wcag/3-2-3-consistent-navigation.md — 3.2.3 一貫したナビゲーション](./3-2-3-consistent-navigation.md)

## よくある失敗例（F31）

- 同じ検索機能なのに、「検索」と「探す」が混在する
- 同じ保存機能なのに、「保存」と「ページを保存」がページごとに違う
- 視覚的なラベルは同じなのに、`aria-label` だけがページごとに違う
- ヘルプアイコンの `alt` が、「ヘルプ」「詳細」「もっと見る」でばらつく

```html
<!-- 失敗例：同じ検索機能なのにラベルが違う -->
<!-- ページA -->
<button type="submit">検索</button>
<!-- ページB -->
<button type="submit">探す</button>
```

```html
<!-- 失敗例：見た目は同じ、読み上げだけ違う -->
<button type="button" aria-label="検索を実行">🔍</button>
<button type="button" aria-label="サイト内を探す">🔍</button>
```

```html
<!-- 改善例：文言とアクセシブルネームを統一 -->
<button type="submit">検索</button>
<button type="button" aria-label="検索">🔍</button>
```

> **参照**
> - [Failure F31: using two different labels for the same function on different web pages](https://www.w3.org/WAI/WCAG22/Techniques/failures/F31)

## 実装時の注意点

- デザインシステムの Button / IconButton に、デフォルト文言を持たせる
- `aria-label` は、見えるテキストと矛盾させない
- 翻訳時も、同じ機能は同じ訳語にする
- A/B テストでボタン文言だけをページごとに変えない
- アイコン単体のときは、代替テキストのパターンを決める（例：常に「ダウンロード：○○」）

## テスト・確認方法

### 手動テスト

- 集合内の複数ページで、同じ結果になる操作を洗い出す
- ボタン文言、リンク文言、`aria-label`、`alt` を比較する
- 「一貫だが同一でない」パターン（ページ番号など）は許容する
- 機能が違うアイコンは、代替テキストが分かれているか確認する

```text
チェックリスト：
1. 複数ページに同じ機能があるか
2. 見えるラベルは一貫しているか
3. aria-label / alt も一貫しているか
4. 「検索」と「探す」のような言い換えが混在していないか
5. 機能が違うのに同じ代替テキストになっていないか
6. 共通コンポーネントで文言が1か所にまとまっているか
```

```javascript
// Console：ボタンとリンクの名前をざっと一覧する
[...document.querySelectorAll("button, a, [role='button']")].map((el) => ({
  text: (el.innerText || "").trim().slice(0, 40),
  ariaLabel: el.getAttribute("aria-label"),
  href: el.getAttribute("href"),
}));
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.2.4 は、Webページの集合内で同じ機能を持つ部品を、**一貫して識別**する達成基準（レベル AA）
- ラベル・アクセシブルネーム・テキストによる代替が対象
- 「一貫」は「同一」とは限らない。パターンが揃っていればよい
- 同じ機能なのに「検索」と「探す」が混在するのは典型的な失敗
- 順序（3.2.3）と識別性（3.2.4）を合わせると、サイト全体が予測しやすくなる
