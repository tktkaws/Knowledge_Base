# 3.2.6 一貫したヘルプ（A）

## 達成基準の概要

- WCAG 2.2 達成基準 3.2.6「一貫したヘルプ」（Consistent Help）
- レベル **A**（最低限の適合レベル）
- **WCAG 2.2 で新規**追加
- 原則3「理解可能（Understandable）」> ガイドライン 3.2「予測可能」に属する
- **Webページの集合内**で、次のヘルプ機構が複数ページに繰り返される場合、他のページ内容に対する**同じ相対順序**で現れること
- ヘルプを**提供すること自体**は必須ではない。提供するなら一貫させる

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.6 Consistent Help](https://www.w3.org/TR/WCAG22/#consistent-help)
> - [Understanding SC 3.2.6 Consistent Help](https://www.w3.org/WAI/WCAG22/Understanding/consistent-help.html)

## 達成基準の原文（要約）

- Web ページに次のいずれかのヘルプ機構があり、それが Webページの集合内の複数ページで繰り返される場合、ユーザーが変更を開始した場合を除き、他のページ内容に対して同じ相対順序で現れること
  - **人的連絡先**（Human contact details）
  - **人的連絡手段**（Human contact mechanism）
  - **セルフヘルプ**（Self-help option）
  - **完全自動の連絡手段**（A fully automated contact mechanism）
- ヘルプはページ上に直接置いても、別ページへの直接リンクでもよい

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.6](https://www.w3.org/TR/WCAG22/#consistent-help)

## なぜ必要か

- フォームや手続きでつまずいたとき、ヘルプを探す余力が残っていない人がいる
- 認知の制約がある人は、サイト内をあちこち探すのが負担になる
- 視覚的な位置の手がかりでヘルプを見つける人も、場所が変わると迷う
- ヘルプがすぐ見つかれば、タスクを完了しやすく、個人情報を不適切な経路で共有しにくくなる
- チャットボットや FAQ も、毎回同じ場所にあると予測しやすい

> **参照**
> - [Understanding SC 3.2.6 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/consistent-help.html#benefits)

## 対象になるヘルプ機構

- **人的連絡先**：電話番号、メールアドレス、受付時間
- **人的連絡手段**：メッセージ、チャット、お問い合わせフォーム、SNS
- **セルフヘルプ**：FAQ、使い方、サポートページ
- **完全自動の連絡手段**：チャットボット
- リストの順序は優先度を意味しない
- ページ上に直接置くか、ヘルプ専用ページへのリンクでもよい

```text
対象になりやすい例：
  ヘッダーの「お問い合わせ」リンク
  フッターの電話番号
  全ページ共通のチャット起動ボタン
  「よくある質問」へのリンク

対象外になりやすい例：
  フォーム内の入力説明（3.3.2 / 3.3.5 の範囲）
  スペルチェックなど UI レベルのヘルプ
  PDF や静的ファイル内の連絡先（集合の一部ではない）
  1ページだけのサイト
```

> **参照**
> - [Understanding SC 3.2.6 — Help Mechanisms](https://www.w3.org/WAI/WCAG22/Understanding/consistent-help.html#help-mechanisms)

## 同じ相対順序とは

- ページを**直列化**したとき、ヘルプ機構が他コンテンツに対して同じ位置関係にあること
- 他ページでヘルプの前にあるコンテンツは、このページでも前にある
- 他ページでヘルプの後にあるコンテンツは、このページでも後にある
- 3.2.3（一貫したナビゲーション）と似るが、対象は**ヘルプ機構**に限定
- 視覚位置が違っても DOM 順が同じなら、この基準は満たしうる（使いにくさは残る）

```text
許容される例：
  全ページで「FAQ → お問い合わせ → チャット」の順
  あるページだけ FAQ がなくても、残りの順序は同じ

失敗例：
  ページA：ヘッダーに「お問い合わせ」
  ページB：フッターだけに「お問い合わせ」（他ページのヘルプより前後の関係が変わる）
  ページA：FAQ → お問い合わせ
  ページB：お問い合わせ → FAQ
```

> **参照**
> - [Understanding SC 3.2.6 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/consistent-help.html#intent)

## 必須ではないこと

- すべてのページにヘルプを置く必要はない
- あるページにヘルプがなくても、他ページにあるなら違反ではない
- 24時間有人対応を求めるものでもない（不在時の案内が望ましい）
- PDF などダウンロード用静的文書内の連絡先は、Webページの集合には含めない
- 複雑なサイトでは、用途の違う集合ごとにヘルプ位置を変えてもよい（ベストは全体で統一）

## ユーザーによる変更の例外

- ズーム、画面向き、ビューポートサイズの変更でレイアウトが変わるのは、ユーザー開始の変更として例外
- **同じページ variation**（同じズーム・向き）同士で相対順序を比較する
- レスポンシブで小画面の位置が変わることは許されるが、見た目も揃えた方が使いやすい
- ページ間の通常ナビゲーションやログインは、この例外には当たらない

> **参照**
> - [Understanding SC 3.2.6 — Limitations and Exceptions](https://www.w3.org/WAI/WCAG22/Understanding/consistent-help.html#limitations-and-exceptions)

## 手法：お問い合わせリンクを一貫した場所に置く（G220）

- **G220**：連絡先へのリンクを、集合内の各ページで同じ場所に置く
- ヘッダー先頭付近、またはフッター内の一定位置が典型
- 共通レイアウトコンポーネントで実装する

```html
<!-- 良い例：全ページでヘッダー内の同じ位置 -->
<header>
  <a href="/">株式会社サンプル</a>
  <nav aria-label="ユーティリティ">
    <ul>
      <li><a href="/faq">よくある質問</a></li>
      <li><a href="/contact">お問い合わせ</a></li>
    </ul>
  </nav>
</header>
```

```html
<!-- 良い例：フッターでも同じ相対順序 -->
<footer>
  <nav aria-label="フッターナビゲーション">
    <ul>
      <li><a href="/faq">よくある質問</a></li>
      <li><a href="/contact">お問い合わせ</a></li>
      <li><a href="tel:0120-000-000">0120-000-000</a></li>
    </ul>
  </nav>
</footer>
```

```html
<!-- 悪い例：ページによってヘルプの位置関係が違う -->
<!-- トップページ：ヘッダーに FAQ、フッターにお問い合わせ -->
<!-- 申込ページ：本文内だけにお問い合わせ、FAQ なし -->
```

```jsx
// 良い例：共通 Footer でヘルプリンクを固定
export function SiteFooter() {
  return (
    <footer>
      <nav aria-label="サポート">
        <ul>
          <li><a href="/faq">よくある質問</a></li>
          <li><a href="/contact">お問い合わせ</a></li>
          <li><a href="/chat">チャットで相談</a></li>
        </ul>
      </nav>
    </footer>
  );
}
```

> **参照**
> - [Technique G220: Provide a contact-us link in a consistent location](https://www.w3.org/WAI/WCAG22/Techniques/general/G220)

## 3.2.3 / 3.3.5 との関係

| 達成基準 | レベル | 対象 |
|---|---|---|
| 3.2.3 一貫したナビゲーション | AA | 繰り返される**ナビゲーション**の相対順序 |
| 3.2.6 一貫したヘルプ | A | 繰り返される**ヘルプ機構**の相対順序 |
| 3.3.5 ヘルプ | AAA | **入力**に関するヘルプの提供 |

- 3.2.6 はサイト全体の「困ったときの連絡先・FAQ・チャット」の位置
- 3.3.5 はフォーム入力の文脈ヘルプ
- グローバルナビの「お問い合わせ」は 3.2.3 も 3.2.6 の両方に関係しうる

> **参照**
> - [wcag/3-2-3-consistent-navigation.md — 3.2.3 一貫したナビゲーション](./3-2-3-consistent-navigation.md)
> - [WCAG 2.2 — Success Criterion 3.3.5 Help](https://www.w3.org/TR/WCAG22/#help)

## チャットボットの実装メモ

- 必須ではないが、認知障害向けの推奨として Understanding に記載がある
-  typo をある程度許容する
- 3回うまく答えられなければ、人的連絡先を示す
- 1操作で閉じられ、同じ場所から再度開ける
- チャットボタン自体の位置は、他ページと同じ相対順序にする

## よくある失敗例

- 申込フローだけヘルプを本文中に置き、他ページはフッターにある
- FAQ とお問い合わせの順序がページごとに入れ替わる
- あるページだけチャットボタンが画面右下に浮き、DOM 順も他ページと違う
- 開閉式メニューの中だけにヘルプがあり、他ページでは常時表示（相対順序が揃わない）

```html
<!-- 失敗例：ページA -->
<header>
  <a href="/faq">FAQ</a>
  <a href="/contact">お問い合わせ</a>
</header>

<!-- 失敗例：ページB — 順序が逆 -->
<header>
  <a href="/contact">お問い合わせ</a>
  <a href="/faq">FAQ</a>
</header>
```

```html
<!-- 改善例：共通コンポーネントで順序を固定 -->
<header>
  <a href="/faq">FAQ</a>
  <a href="/contact">お問い合わせ</a>
</header>
```

> **参照**
> - [Understanding SC 3.2.6 — Failures: Inconsistent Help Location](https://www.w3.org/WAI/WCAG22/Understanding/consistent-help.html)

## テスト・確認方法

### 手動テスト

- 集合内の複数ページで、ヘルプ機構（FAQ、お問い合わせ、チャット、電話など）を洗い出す
- ヘルプがあるページ同士で、DOM 上の相対順序を比較する
- 同じビューポート（ズーム・向き）で、視覚位置も揃っているか確認する
- ヘルプがないページだけがあっても、違反ではないことを確認する

```text
チェックリスト：
1. 複数ページに同じ種類のヘルプがあるか
2. FAQ / お問い合わせ / チャットなどの相対順序は同じか
3. 他の共通コンテンツとの前後関係はページ間で一致しているか
4. レスポンシブで順序が変わっても、同じ variation 内では一貫しているか
5. フォーム内の入力説明と混同していないか
6. 共通 Header / Footer コンポーネントで実装されているか
```

```javascript
// Console：ヘルプっぽいリンクの DOM 順をざっと確認
[...document.querySelectorAll("a, button")].filter((el) =>
  /faq|help|contact|support|問い合わせ|ヘルプ|サポート/i.test(
    el.textContent + (el.getAttribute("href") || "")
  )
).map((el) => ({
  text: el.textContent.trim().slice(0, 40),
  href: el.getAttribute("href"),
}));
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.2.6 は、**ヘルプ機構**を Webページの集合内で同じ相対順序に置く達成基準（レベル A、WCAG 2.2 新規）
- ヘルプを提供すること自体は必須ではない。提供するなら FAQ・お問い合わせ・チャットなどの位置を揃える
- フォーム内の入力説明（3.3.5）とは別
- 共通 Header / Footer が実装の定番
- 3.2.3 はナビ全体、3.2.6 は困ったときの支援に焦点
