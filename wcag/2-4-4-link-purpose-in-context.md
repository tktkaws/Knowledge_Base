# 2.4.4 リンクの目的（コンテキスト内）（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.4「リンクの目的（コンテキスト内）」（Link Purpose (In Context)）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- 各リンクの目的が、**リンクテキスト単独**または**プログラム上で関連付けられた文脈**と合わせて分かること
- ユーザーがリンクをたどる前に、「どこへ行くか」「何が起きるか」を判断できるようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.4 Link Purpose (In Context)](https://www.w3.org/TR/WCAG22/#link-purpose-in-context)
> - [Understanding SC 2.4.4 Link Purpose (In Context)](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-in-context.html)

## 達成基準の原文（要約）

- 各リンクの目的は、リンクテキスト単独、またはリンクテキストと**プログラム上で決定可能なリンク文脈**を組み合わせて判断できること
- ただし、一般のユーザーにとってもリンク目的が本質的に曖昧な場合は例外

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.4](https://www.w3.org/TR/WCAG22/#link-purpose-in-context)

## なぜ必要か

- キーボード利用者は、興味のないリンクを避けて効率よく移動できる
- 認知障害のあるユーザーは、リンク先の目的が分からないと混乱しやすい
- 視覚障害のあるユーザーは、リンク一覧や Tab 移動でリンクを把握するときに意味のある名前が必要
- 「こちら」「詳細」だけでは、複数リンクの中で選べないことがある
- リンク先へ移動して戻る無駄な操作を減らせる

> **参照**
> - [Understanding SC 2.4.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-in-context.html#benefits)

## リンク文脈とは

- リンクの周囲にあり、**プログラム上で関連付けを判断できる**文脈
- 同じ文・段落・リスト項目・表セル・対応する見出しなどが代表例
- 支援技術が、フォーカスを移したままリンクの意味を把握できる範囲が望ましい
- 文脈は、リンクの**前にある方が分かりやすい**ことが多い

```text
文脈として使いやすい例：
  同じ段落内の説明文
  同じ li 要素内の書籍タイトル
  表の見出しセルとデータセル
  直前の見出し

文脈として弱い例：
  離れた位置にある説明文
  視覚的には近いが DOM 上で関連付いていないテキスト
```

> **参照**
> - [Understanding SC 2.4.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-in-context.html#intent)

## 最善策：リンクテキストだけで目的を示す（G91）

- 可能なら、リンク単独で目的が分かる文言にする
- リンク一覧機能や音声読み上げでも理解しやすい
- 2.4.9（リンクのみ）の AAA 適合にもつながる

```html
<!-- 悪い例：目的が分からない -->
<a href="/pricing">こちら</a>

<!-- 良い例：リンク単独で目的が分かる -->
<a href="/pricing">料金プランを見る</a>
```

```html
<!-- 良い例：文書名やアプリ名で目的を示す -->
<a href="/docs/api-reference.pdf">API リファレンス PDF</a>
<a href="/app/banking">インターネットバンキング</a>
```

> **参照**
> - [Technique G91: Providing link text that describes the purpose of a link](https://www.w3.org/WAI/WCAG22/Techniques/general/G91)

## 文脈と組み合わせて目的を示す

- A レベルでは、リンク単独で足りなくても、文脈と合わせて分かれば適合しうる
- ただし文脈は、支援技術から**プログラム上で取得しやすい構造**に置く
- 単に見た目の近さだけに頼らない

### 段落内の文脈（H78）

```html
<!-- 良い例：同じ段落の文脈で目的が分かる -->
<p>
  電子投票委員会について詳しくは
  <a href="/go-vote">Go Vote!</a>
  を参照。
</p>
```

### リスト項目内の文脈（H77）

```html
<!-- 良い例：書籍名 + 形式 -->
<ul>
  <li>
    アクセシビリティ入門
    <a href="/books/a11y-intro">HTML</a>
    <a href="/books/a11y-intro.pdf">PDF</a>
    <a href="/books/a11y-intro.mp3">mp3</a>
  </li>
</ul>
```

### 表の文脈（H79）

```html
<!-- 良い例：表見出しとセル文脈で目的が分かる -->
<table>
  <caption>会議資料</caption>
  <thead>
    <tr>
      <th scope="col">資料名</th>
      <th scope="col">形式</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>四半期レポート</td>
      <td><a href="/reports/q1.pdf">PDF</a></td>
    </tr>
  </tbody>
</table>
```

### 入れ子リストの文脈（H81）

```html
<!-- 良い例：親項目との組み合わせで意味が分かる -->
<ul>
  <li>
    サポート
    <ul>
      <li><a href="/support/faq">FAQ</a></li>
      <li><a href="/support/contact">お問い合わせ</a></li>
    </ul>
  </li>
</ul>
```

> **参照**
> - [Technique H77: link text combined with enclosing list item](https://www.w3.org/WAI/WCAG22/Techniques/html/H77)
> - [Technique H78: link text combined with enclosing paragraph](https://www.w3.org/WAI/WCAG22/Techniques/html/H78)
> - [Technique H79: link purpose in a data table](https://www.w3.org/WAI/WCAG22/Techniques/html/H79)
> - [Technique H81: link purpose in a nested list](https://www.w3.org/WAI/WCAG22/Techniques/html/H81)

## `aria-label` / `aria-labelledby` で補う（ARIA7 / ARIA8）

- 見た目を大きく変えられない場合に、アクセシブルネームで目的を補う
- ただし、まずは**見えるリンクテキストの改善**を優先する
- `aria-label` は周囲の見える文脈と食い違わないようにする

```html
<!-- 悪い例：アイコンだけで目的不明 -->
<a href="/search">
  <svg aria-hidden="true" viewBox="0 0 24 24">...</svg>
</a>

<!-- 良い例：aria-label で目的を付ける -->
<a href="/search" aria-label="サイト内検索">
  <svg aria-hidden="true" viewBox="0 0 24 24">...</svg>
</a>
```

```html
<!-- 良い例：見出しと関連付ける -->
<h2 id="news-title">新着記事</h2>
<a href="/news/2026-launch" aria-labelledby="news-title news-link">
  <span id="news-link">詳細</span>
</a>
```

> **参照**
> - [Technique ARIA7: Using aria-labelledby for link purpose](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA7)
> - [Technique ARIA8: Using aria-label for link purpose](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA8)

## 補足説明の付け方

- `title` 属性で補足する方法（H33）はあるが、単独依存は避けたい
- スクリーンリーダーやタッチ環境で一貫して使いにくい場合がある
- CSS で補足テキストを視覚的に隠しつつ、アクセシブルネームに含める方法（C7）もある

```html
<!-- title 属性で補足する例 -->
<a href="/minutes" title="2026年7月 定例会議議事録を開く">議事録</a>
```

```html
<!-- 視覚的に隠した補足テキストを含める例 -->
<a href="/pricing">
  詳細<span class="visually-hidden">：料金プラン</span>
</a>
```

```css
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

> **参照**
> - [Technique H33: Supplementing link text with the title attribute](https://www.w3.org/WAI/WCAG22/Techniques/html/H33)
> - [Technique C7: Using CSS to hide a portion of the link text](https://www.w3.org/WAI/WCAG22/Techniques/css/C7)

## よくある例：「続きを読む」

- `続きを読む` 単独は、一覧に複数あると目的が分かりにくい
- A レベルでは、同じ段落やカード内の文脈から分かれば適合しうる
- ただし、可能ならリンク自体を具体化した方がよい

```html
<!-- Aレベルでは適合しうる例：同じ段落内に文脈がある -->
<p>
  2026年のアクセシビリティ方針を公開しました。
  <a href="/news/accessibility-policy">続きを読む</a>
</p>
```

```html
<!-- より良い例：リンク単独でも分かる -->
<a href="/news/accessibility-policy">
  2026年のアクセシビリティ方針を読む
</a>
```

## 例外：一般のユーザーにも曖昧な場合

- リンクの目的を意図的に伏せる体験では、曖昧さが例外になりうる
- 例：ゲームで `ドア1` `ドア2` `ドア3` を選ばせる
- その曖昧さが**障害の有無にかかわらず共通**なら、この基準では許容されうる

```html
<!-- 例外になりうる例 -->
<ul>
  <li><a href="/game/door-1">ドア1</a></li>
  <li><a href="/game/door-2">ドア2</a></li>
  <li><a href="/game/door-3">ドア3</a></li>
</ul>
```

> **参照**
> - [Understanding SC 2.4.4 — ambiguous to users in general](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-in-context.html#dfn-ambiguous-to-users-in-general)

## よくある失敗

### F63：関連しない場所にしか文脈がない

- リンクの説明が離れた位置にあり、プログラム上で関連付いていない
- 見た目では近くても、支援技術では目的が分からない

```html
<!-- 悪い例：離れた見出しだけに頼る -->
<h2>会社情報</h2>
<!-- ...大量の別コンテンツ... -->
<a href="/about">詳細</a>
```

### F89：画像リンクにアクセシブルネームがない

- リンクの中身が画像だけなのに、`alt` も名前付けもない

```html
<!-- 悪い例：画像だけのリンクに名前がない -->
<a href="/cart">
  <img src="/icons/cart.svg" alt="">
</a>

<!-- 良い例：目的を示す -->
<a href="/cart">
  <img src="/icons/cart.svg" alt="カートを見る">
</a>
```

> **参照**
> - [Failure F63: link context only in content not related to the link](https://www.w3.org/WAI/WCAG22/Techniques/failures/F63)
> - [Failure F89: image-only link without an accessible name](https://www.w3.org/WAI/WCAG22/Techniques/failures/F89)

## 2.4.2・2.4.9・3.2.4 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.2 ページタイトル | 到着先ページの識別 | リンク先とタイトルの連続性 |
| 2.4.4 リンクの目的（コンテキスト内） | 文脈込みでリンク先が分かる | A レベル |
| 2.4.9 リンクの目的（リンクのみ） | リンク単独で目的が分かる | AAA、より厳しい |
| 3.2.4 一貫した識別 | 同じ機能は一貫して識別する | 同一リンク文言の一貫性 |

- 同じ宛先なら、できるだけ同じリンク文言にするのが望ましい
- 異なる宛先・異なる目的なら、異なる文言にするのが望ましい
- リンク文言と到着先のページタイトルを近づけると分かりやすい

```html
<!-- 良い習慣：リンクとページタイトルが近い -->
<a href="/support/contact">お問い合わせ</a>
<!-- 遷移先 -->
<title>お問い合わせ | サポート | サンプル社</title>
```

> **参照**
> - [wcag/2-4-2-page-titled.md — 2.4.2 ページタイトル](./2-4-2-page-titled.md)
> - [WCAG 2.2 — Success Criterion 2.4.9 Link Purpose (Link Only)](https://www.w3.org/TR/WCAG22/#link-purpose-link-only)
> - [WCAG 2.2 — Success Criterion 3.2.4 Consistent Identification](https://www.w3.org/TR/WCAG22/#consistent-identification)

## 実装時の注意点

- まずリンク文言そのものを改善する
- 一覧カードやニュース一覧では、`続きを読む` の乱立を避ける
- アイコンリンクは必ずアクセシブルネームを持たせる
- 視覚上の近さではなく、DOM 上の関係で文脈を設計する
- `aria-label` を使う場合、見える文言と意図が食い違わないようにする
- 新しいウィンドウや別形式（PDF, mp3）を開く場合は、その情報も分かるとよい

```text
実務チェック：
  「こちら」「詳細」「続きを読む」だけのリンクが並んでいないか
  画像だけリンクに alt / aria-label があるか
  リンク一覧機能で意味が通るか
```

## テスト・確認方法

### 手動テスト

- ページ内のリンクを一覧で見て、目的が分かるか確認する
- Tab だけでリンクを移動し、周囲の文脈込みで意味が通るか確認する
- スクリーンリーダーのリンク一覧で、曖昧なリンクが多すぎないか確認する
- 画像だけリンクに適切な名前があるか確認する
- `続きを読む` や `詳細` が複数ある場合、各文脈が十分か確認する

```text
チェックリスト：
1. 各リンクの目的が分かるか
2. 分からない場合、同じ段落・リスト項目・表セルなどの文脈で補えているか
3. 離れた場所の説明に依存していないか
4. 画像だけリンクにアクセシブルネームがあるか
5. リンク先の種類や形式が必要に応じて示されているか
```

### 開発者ツールでの確認

```bash
# 曖昧なリンク文言やARIAによる補足を探す
rg "\">こちら<|>詳細<|>続きを読む<|aria-label=|aria-labelledby=|<a [^>]*><img" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.php"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.4 は、各リンクの目的が**リンク単独**または**関連する文脈**と合わせて分かることを求める達成基準（レベル A）
- 最もよいのは、リンク単独で意味が分かる文言にすること
- A レベルでは、段落・リスト項目・表セルなどのプログラム上の文脈で補ってもよい
- `aria-label` / `aria-labelledby` は補助手段として有効だが、まず見える文言の改善を優先する
- 2.4.9（リンクのみ）はより厳しく、AAA ではリンク単独での明確さが求められる
