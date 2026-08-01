# 2.4.9 リンクの目的（リンクのみ）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.9「リンクの目的（リンクのみ）」（Link Purpose (Link Only)）
- レベル **AAA**（最高の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- 各リンクの目的を、**リンクテキスト単独**で識別できるメカニズムがあること
- 2.4.4 と異なり、周囲の文脈に頼らず、リンク自体で目的が分かること

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.9 Link Purpose (Link Only)](https://www.w3.org/TR/WCAG22/#link-purpose-link-only)
> - [Understanding SC 2.4.9 Link Purpose (Link Only)](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-link-only.html)

## 達成基準の原文（要約）

- 各リンクの目的を、リンクテキスト単独で識別できるメカニズムが利用できること
- ただし、一般のユーザーにとってもリンク目的が本質的に曖昧な場合は例外

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.9](https://www.w3.org/TR/WCAG22/#link-purpose-link-only)

## 2.4.4 との違い

| 項目 | 2.4.4（A） | 2.4.9（AAA） |
|---|---|---|
| 判断材料 | リンクテキスト **または** 文脈との組み合わせ | **リンクテキスト単独** |
| 段落・リスト項目の文脈 | 文脈込みで適合しうる | 文脈だけでは不十分 |
| 「続きを読む」 | 同じ段落の文脈があれば適合しうる | 単独では不適合になりやすい |
| メカニズム | 必須ではない | 短い／長いリンク文言を切り替えられてもよい |

```text
2.4.4 では適合しうるが 2.4.9 では不適合の例：
  <p>アクセシビリティ方針を公開しました。<a>続きを読む</a></p>
  （文脈はあるが、リンク単独では目的不明）

両方に適合しやすい例：
  <a>2026年のアクセシビリティ方針を読む</a>
```

> **参照**
> - [wcag/2-4-4-link-purpose-in-context.md — 2.4.4 リンクの目的（コンテキスト内）](./2-4-4-link-purpose-in-context.md)

## なぜ必要か

- スクリーンリーダーのリンク一覧は、周囲の文脈なしでリンクだけを並べる
- キーボード利用者は、興味のないリンクを事前に避けられる
- 認知障害のあるユーザーは、余分な往復ナビで混乱しにくい
- 視覚障害のあるユーザーは、元の位置を見失いやすいので、不要な遷移を減らせる

> **参照**
> - [Understanding SC 2.4.9 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-link-only.html#benefits)

## 「メカニズム」とは

- 最初からすべてのリンクを単独で分かる文言にする方法
- または、ユーザーが短い文言／長い文言を切り替えられる方法
- 長いリンク文言が読みにくい人も、短い方が速い人もいるため、選択肢を認める
- メカニズム自体も、主張する適合レベルの達成基準を満たす必要がある

```text
メカニズムの例：
  ページ先頭に「詳しいリンク文言を表示」チェックがある
  設定で「短いリンク／説明的なリンク」を切り替えられる
  最初からすべて説明的なリンク文言にする（最も単純）
```

> **参照**
> - [Understanding SC 2.4.9 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-link-only.html#intent)

## 手法1：リンクテキストだけで目的を示す（G91 / H30）

- 最も直接的な適合方法
- リンク一覧や Tab 移動でも、行き先が分かる文言にする
- 文書名やアプリ名でも、目的が分かれば足りる

```html
<!-- 悪い例：リンク単独では目的不明（F84） -->
<a href="/news/1">こちら</a>
<a href="/news/2">詳細</a>
<a href="/news/3">もっと見る</a>
<a href="/news/4">click here</a>

<!-- 良い例：リンク単独で目的が分かる -->
<a href="/news/1">採用イベントの開催報告を読む</a>
<a href="/pricing">料金プランを見る</a>
<a href="/docs/api-reference.pdf">API リファレンス PDF</a>
```

```html
<!-- 良い例：アイコンとテキストを同じリンクに含める -->
<a href="/go-vote">
  <img src="/icons/vote.svg" alt="">
  アイルランド政府の電子投票委員会
</a>
```

> **参照**
> - [Technique G91: Providing link text that describes the purpose of a link](https://www.w3.org/WAI/WCAG22/Techniques/general/G91)
> - [Technique H30: Providing link text that describes the purpose of a link for anchor elements](https://www.w3.org/WAI/WCAG22/Techniques/html/H30)

## 手法2：視覚的に短い文言でも、名前は具体的にする（C7 / ARIA8）

- 見た目は「PDF」「続きを読む」でも、アクセシブルネームに具体名を含める
- CSS で補足テキストを視覚的に隠す（C7）
- `aria-label` で目的を付ける（ARIA8）
- 見える文言と支援技術向けの名前が大きく食い違わないようにする

```html
<!-- 良い例：見た目は形式名、名前には書名を含める（C7） -->
<ul>
  <li>
    ガリバー旅行記
    <a href="/books/gulliver.html">
      HTML
      <span class="visually-hidden">：ガリバー旅行記</span>
    </a>
    <a href="/books/gulliver.mp3">
      MP3
      <span class="visually-hidden">：ガリバー旅行記</span>
    </a>
  </li>
</ul>
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

```html
<!-- 良い例：aria-label で具体化する -->
<a href="/news/accessibility-policy" aria-label="2026年のアクセシビリティ方針を読む">
  続きを読む
</a>
```

```text
注意：
  aria-label はリンク一覧では有効
  ただし、見える「続きを読む」と読み上げが違うと、認知的な不一致が起きる場合がある
  可能なら見える文言自体を具体化する方が望ましい
```

> **参照**
> - [Technique C7: Using CSS to hide a portion of the link text](https://www.w3.org/WAI/WCAG22/Techniques/css/C7)
> - [Technique ARIA8: Using aria-label for link purpose](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA8)
> - [Understanding SC 2.4.9 — book formats example](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-link-only.html#examples)

## 手法3：短い／長いリンク文言を切り替える（G189 / SCR30）

- ページ先頭付近に、リンク文言を詳細化するコントロールを置く
- 短い表示を好む人も、単独で分かる長い表示を好む人も選べる
- 書籍一覧のように、同じタイトルが何度も繰り返されると冗長になる場合に有効

```html
<!-- 良い例：詳細なリンク文言への切り替え -->
<form>
  <label>
    <input type="checkbox" id="verbose-links">
    リンクに詳しい説明を含める
  </label>
</form>

<ul id="book-list">
  <li>
    ガリバー旅行記
    <a href="/books/gulliver.pdf" data-short="PDF" data-long="ガリバー旅行記 PDF">PDF</a>
    <a href="/books/gulliver.mp3" data-short="MP3" data-long="ガリバー旅行記 MP3">MP3</a>
  </li>
</ul>
```

```javascript
const toggle = document.getElementById("verbose-links");
const links = document.querySelectorAll("#book-list a");

toggle.addEventListener("change", () => {
  links.forEach((link) => {
    link.textContent = toggle.checked
      ? link.dataset.long
      : link.dataset.short;
  });
});
```

> **参照**
> - [Technique G189: Providing a control near the beginning of the web page that changes the link text](https://www.w3.org/WAI/WCAG22/Techniques/general/G189)
> - [Technique SCR30: Using scripts to change the link text](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR30)

## 例外：一般のユーザーにも曖昧な場合

- リンクの目的を意図的に伏せる体験では、曖昧さが例外になりうる
- 例：ゲームで `ドア1` `ドア2` `ドア3` を選ばせる
- その曖昧さが障害の有無にかかわらず共通なら、この基準では許容されうる
- ただし、ページ上に使える文脈があるなら、その分はリンクテキストに含める必要がある

```html
<!-- 例外になりうる例 -->
<ul>
  <li><a href="/game/door-1">ドア1</a></li>
  <li><a href="/game/door-2">ドア2</a></li>
  <li><a href="/game/door-3">ドア3</a></li>
</ul>
```

> **参照**
> - [Understanding SC 2.4.9 — ambiguous to users in general](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-link-only.html#dfn-ambiguous-to-users-in-general)

## よくある失敗

### F84：曖昧なリンク文言で、切り替え手段もない

- `こちら` `詳細` `もっと見る` `click here` `more` など
- 文脈はあるが、リンク単独では目的不明
- かつ、詳細文言へ切り替えるメカニズムもない

```html
<!-- 失敗例（F84） -->
<p>
  新製品を発表しました。
  <a href="/products/new">もっと見る</a>
</p>
```

```html
<!-- 改善例 -->
<a href="/products/new">新製品の発表内容を見る</a>
```

### F89：画像だけリンクに名前がない

- リンクの中身が画像だけなのに、`alt` も名前付けもない
- 2.4.4 / 4.1.2 でも失敗になる

```html
<!-- 失敗例 -->
<a href="/cart">
  <img src="/icons/cart.svg" alt="">
</a>

<!-- 改善例 -->
<a href="/cart">
  <img src="/icons/cart.svg" alt="カートを見る">
</a>
```

> **参照**
> - [Failure F84: using a non-specific link such as "click here" or "more" without a mechanism to change the link text](https://www.w3.org/WAI/WCAG22/Techniques/failures/F84)
> - [Failure F89: image-only link without an accessible name](https://www.w3.org/WAI/WCAG22/Techniques/failures/F89)

## 2.4.2・3.2.4 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.2 ページタイトル | 到着先ページの識別 | リンクとタイトルの連続性 |
| 2.4.4 リンクの目的（コンテキスト内） | 文脈込みで分かる（A） | より緩い要件 |
| 2.4.9 リンクの目的（リンクのみ） | リンク単独で分かる（AAA） | より厳しい要件 |
| 3.2.4 一貫した識別 | 同じ機能は一貫して識別する | 同一宛先は同じ文言が望ましい |

- 同じ宛先なら、できるだけ同じリンク文言にする
- 異なる宛先・異なる目的なら、異なる文言にする
- リンク文言と到着先のページタイトルを近づけると分かりやすい

```html
<!-- 良い習慣：リンクとページタイトルが近い -->
<a href="/support/contact">お問い合わせ</a>
<!-- 遷移先 -->
<title>お問い合わせ | サポート | サンプル社</title>
```

> **参照**
> - [wcag/2-4-2-page-titled.md — 2.4.2 ページタイトル](./2-4-2-page-titled.md)
> - [WCAG 2.2 — Success Criterion 3.2.4 Consistent Identification](https://www.w3.org/TR/WCAG22/#consistent-identification)

## 実装時の注意点

- まず見えるリンク文言自体を具体化する
- ニュースカードの `続きを読む` は、記事名を含めるか `aria-label` / 隠しテキストで補う
- 形式リンク（PDF / MP3）は、対象名を名前に含める
- `title` 属性だけへの依存は避ける（advisory）
- リンク一覧機能で読み上げて、曖昧なリンクが残っていないか確認する

```text
実務チェック：
  リンク一覧だけ見て、各リンクの行き先が分かるか
  「こちら」「詳細」「続きを読む」が残っていないか
  残す場合、切り替え手段や具体的な名前があるか
```

## テスト・確認方法

### 手動テスト

- スクリーンリーダーのリンク一覧、またはページ内リンクを抽出して確認する
- 各リンクが、周囲の文脈なしで目的を示しているか確認する
- 曖昧なリンクがある場合、詳細文言への切り替え手段があるか確認する
- 画像だけリンクにアクセシブルネームがあるか確認する

```text
チェックリスト：
1. リンクテキスト単独で目的が分かるか
2. 分からない場合、詳細化するメカニズムがあるか
3. 「こちら」「詳細」「more」などが残っていないか
4. 画像だけリンクに名前があるか
5. 同じ宛先のリンク文言は一貫しているか
```

### 開発者ツールでの確認

```bash
# 曖昧なリンク文言を探す
rg "\">こちら<|>詳細<|>続きを読む<|>もっと見る<|>click here<|>more<|aria-label=" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.php"
```

```javascript
// Console：リンクのアクセシブルネームっぽいテキストを一覧
[...document.links].map((a) => a.textContent.replace(/\s+/g, " ").trim() || a.getAttribute("aria-label"))
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.9 は、各リンクの目的を**リンクテキスト単独**で識別できることを求める達成基準（レベル AAA）
- 2.4.4 と異なり、段落やリスト項目の文脈だけでは足りない
- 最初から説明的な文言にするか、短い／長い文言を切り替えるメカニズムを用意する
- `続きを読む` や形式名だけのリンクは、隠しテキストや `aria-label` で具体化する
- F84（曖昧なリンク）と F89（名前のない画像リンク）が典型的な失敗
