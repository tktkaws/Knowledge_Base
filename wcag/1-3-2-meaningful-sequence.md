# 1.3.2 意味のある順序（A）

## 達成基準の概要

- WCAG 2.2 達成基準 1.3.2「意味のある順序」（Meaningful Sequence）
- レベル A（最低限の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.3「適応可能」に属する
- コンテンツの**提示順序**が意味に影響する場合、**正しい読み上げ順序**をプログラム的に判定可能にする
- 1.3.1（構造・関係性）とセットで、セマンティック HTML の実践を支える達成基準

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.2 Meaningful Sequence](https://www.w3.org/TR/WCAG22/#meaningful-sequence)
> - [Understanding SC 1.3.2 Meaningful Sequence](https://www.w3.org/WAI/WCAG22/Understanding/meaningful-sequence.html)

## 達成基準の原文（要約）

- コンテンツの**提示順序**がその意味に影響する場合、**正しい読み上げ順序**をプログラム的に判定できること
- 支援技術・代替スタイルシートで表示形式が変わっても、内容の意味が失われないようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.2](https://www.w3.org/TR/WCAG22/#meaningful-sequence)

## 「意味のある順序」とは

- 順序を変えると**意味が変わる**コンテンツの並び
- 具体例
  - 段落内の文の順序
  - 手順説明（1 → 2 → 3）
  - データテーブルの行・列
  - 順序付きリスト（`<ol>`）
  - フォームの入力順（姓 → 名 → 住所）
- HTML では `<table>`・`<ol>`・テキストは**意味のある順序**を持つ要素
- `<ul>`（順序なしリスト）は順序自体に意味を持たない

### 順序が意味に影響しない場合

- すべての並び替えが「正しい」わけではない。**少なくとも1つ**の正しい順序があればよい
- 独立したコンテンツ同士の相対順序が意味に影響しない場合もある
  - `<main>` と `<nav>` のどちらが先でも意味は変わらない
  - 記事本文とサイドバーの独立したコールアウト
  - 複数の独立した記事（インターリーブされていなければ）

```html
<!-- どちらの DOM 順序も 1.3.2 的には問題ない -->
<body>
  <nav>...</nav>
  <main>...</main>
</body>

<body>
  <main>...</main>
  <nav>...</nav>
</body>
```

> **参照**
> - [Understanding SC 1.3.2 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/meaningful-sequence.html#intent)

## なぜ必要か

- スクリーンリーダーは**DOM 順序**でコンテンツを読み上げる
- CSS で視覚的な順序だけを変えると、読み上げ順と見た目が**乖離**する
- 支援技術利用者が**混乱・誤解**する原因になる
- ユーザースタイルシート・1カラム表示・リフロー時も意味が保たれる

> **参照**
> - [Understanding SC 1.3.2 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/meaningful-sequence.html#benefits)

## 1.3.1 との関係

| 達成基準 | 焦点 | 例 |
|---|---|---|
| 1.3.1 情報および関係性 | **構造・関係性**のプログラム的判定 | 見出し、ラベルと入力欄の対応 |
| 1.3.2 意味のある順序 | **順序**のプログラム的判定 | DOM 順と読み上げ順の一致 |

- 1.3.1 で `<table>` の構造を正しくしても、CSS でセルの表示順を入れ替えると 1.3.2 違反になりうる
- 両方を満たすには、**セマンティック HTML + DOM 順序の整合**が必要

> **参照**
> - [wcag/1-3-1-info-and-relationships.md — 1.3.1 情報および関係性](./1-3-1-info-and-relationships.md)

## DOM 順序と視覚順序

- スクリーンリーダー・支援技術は**DOM 順序**（ソースコード上の出現順）で読み上げる
- CSS の `order`・`position: absolute`・`float`・Grid の `grid-area` 等で**見た目だけ**順序を変えても、読み上げ順は変わらない
- **視覚順序と DOM 順序を一致**させるのが基本（Technique C6）

```html
<!-- 悪い例：flexbox の order で視覚順と DOM 順が逆（Failure F1） -->
<div style="display: flex;">
  <p style="order: 2;">第二段落：以上が結論です。</p>
  <p style="order: 1;">第一段落：本製品の特徴を説明します。</p>
</div>
<!-- → スクリーンリーダーは「第二段落 → 第一段落」の順で読む -->

<!-- 良い例：DOM 順序 = 意味の順序 -->
<div style="display: flex; flex-direction: column;">
  <p>第一段落：本製品の特徴を説明します。</p>
  <p>第二段落：以上が結論です。</p>
</div>
```

```html
<!-- 悪い例：position: absolute で意味の順序を崩す -->
<div style="position: relative;">
  <p style="position: absolute; top: 100px;">結論：本製品は業界最高水準です。</p>
  <p style="position: absolute; top: 0;">序論：市場動向について述べます。</p>
</div>
<!-- → DOM 順は「結論 → 序論」だが、見た目は「序論 → 結論」 -->

<!-- 良い例：DOM 順序を論理順に保ち、CSS は見た目の調整のみ -->
<article>
  <p>序論：市場動向について述べます。</p>
  <p>結論：本製品は業界最高水準です。</p>
</article>
```

> **参照**
> - [Technique C6: Positioning content based on structural markup](https://www.w3.org/WAI/WCAG22/Techniques/css/C6)
> - [Failure F1: Changing the meaning of content by positioning information with CSS](https://www.w3.org/WAI/WCAG22/Techniques/failures/F1)
> - [a11y/keyboard-navigation-basics.md — フォーカス順序は DOM 順に従う](../a11y/keyboard-navigation-basics.md)

## マルチカラムレイアウト

- 複数カラムの場合、**上から下 → 次のカラム**の順で線形化される
- カラムをまたいで文が続く場合、DOM 順序がカラム順と一致している必要がある

```html
<!-- 悪い例：2カラムを div で分割し、左カラムの続きが右カラムの DOM より後 -->
<div class="columns">
  <div class="col-left">
    <p>左カラム1段落目。</p>
    <p>左カラム2段落目。</p>
  </div>
  <div class="col-right">
    <p>右カラム1段落目。</p>
    <p>右カラム2段落目。</p>
  </div>
</div>
<!-- → 線形化：左1 → 左2 → 右1 → 右2（カラム順は OK） -->

<!-- 悪い例：意味的に左→右の順が必要なのに DOM が右→左 -->
<div style="display: flex;">
  <div style="order: 2;">右側の結論</div>
  <div style="order: 1;">左側の前提</div>
</div>
<!-- → 読み上げは「結論 → 前提」になり意味が逆転 -->
```

- CSS Grid / Flexbox でレイアウトする場合、**DOM に論理順序で記述**し、CSS で配置を調整

> **参照**
> - [Understanding SC 1.3.2 — Example 1: Multi-column document](https://www.w3.org/WAI/WCAG22/Understanding/meaningful-sequence.html#examples)

## レイアウトテーブルの線形化（Failure F49）

- レイアウト目的の `<table>` を線形化すると、セルの DOM 順で読み上げられる
- 意味のない順序で読み上げられると Failure F49

```html
<!-- 悪い例：レイアウト table で意味のない読み上げ順（F49） -->
<table>
  <tr>
    <td><h1>サイトタイトル</h1></td>
    <td><nav>メニュー</nav></td>
  </tr>
  <tr>
    <td colspan="2"><main>本文...</main></td>
  </tr>
</table>
<!-- → 線形化で nav が h1 の直後に来る等、不自然な順序 -->

<!-- 良い例：CSS Grid / Flexbox でレイアウト -->
<header><h1>サイトタイトル</h1></header>
<nav>...</nav>
<main>本文...</main>
```

> **参照**
> - [Failure F49: Using an HTML layout table that does not make sense when linearized](https://www.w3.org/WAI/WCAG22/Techniques/failures/F49)

## フォームの入力順序

- ラベル → 入力欄の順序が DOM 上で自然であること
- CSS でラベルを入力欄の**下**や**右**に配置しても、DOM ではラベルが先

```html
<!-- 悪い例：input が label より DOM 上で先 -->
<input type="text" id="name">
<label for="name">お名前</label>
<!-- → スクリーンリーダーが入力欄を先に読み、ラベルが後 -->

<!-- 良い例：label → input の DOM 順 -->
<label for="name">お名前</label>
<input type="text" id="name" name="name">
```

```html
<!-- 視覚的にラベルを右に配置しても DOM 順は label → input -->
<label for="email" class="label-right">
  メールアドレス
  <input type="email" id="email" name="email">
</label>
```

> **参照**
> - [Technique G57: Ordering the content in a meaningful sequence](https://www.w3.org/WAI/WCAG22/Techniques/general/G57)

## 双方向テキスト（RTL / LTR）

- アラビア語・ヘブライ語（RTL）と英語（LTR）が混在する場合、**テキスト方向**の順序が意味に影響
- `dir` 属性、`unicode-bidi`、Unicode 制御文字（LRM / RLM）で正しい順序を保持

```html
<!-- 悪い例：RTL テキスト内の LTR 英語の順序が崩れる -->
<p>العربية English mixed text</p>

<!-- 良い例：dir 属性で方向を明示 -->
<p dir="rtl">العربية <span dir="ltr">English</span> mixed text</p>
```

```html
<!-- インライン要素の方向問題：H56 -->
<p dir="rtl">
  <span dir="ltr">123-456</span>
  <!-- ネストした方向の実行順を dir で解決 -->
</p>
```

> **参照**
> - [Technique H34: Using Unicode RLM or LRM to mix text direction inline](https://www.w3.org/WAI/WCAG22/Techniques/html/H34)
> - [Technique H56: Using the dir attribute on an inline element](https://www.w3.org/WAI/WCAG22/Techniques/html/H56)

## tabindex と読み上げ順序

- `tabindex` の**正の値**（1, 2, 3...）は Tab 順序を DOM 順から変更する
- 1.3.2 だけでなく **2.4.3 フォーカス順序** でも問題視される
- 読み上げ順序自体は DOM 順が基本。`tabindex` は主にキーボード操作に影響

```html
<!-- 悪い例：tabindex で意図しない Tab 順 -->
<input tabindex="3" placeholder="3番目">
<input tabindex="1" placeholder="1番目">
<input tabindex="2" placeholder="2番目">

<!-- 良い例：DOM 順 = Tab 順。必要なら tabindex="0" のみ -->
<input placeholder="1番目">
<input placeholder="2番目">
<input placeholder="3番目">
```

- `tabindex="-1"` は Tab 順序から除外。プログラム的フォーカス（スキップリンク等）用

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.3 Focus Order](https://www.w3.org/TR/WCAG22/#focus-order)
> - [a11y/focus-management.md — tabindex, focus()](../a11y/focus-management.md)

## 2.4.3 フォーカス順序との関係

| 達成基準 | 焦点 | 主な対象 |
|---|---|---|
| 1.3.2 意味のある順序 | **コンテンツの読み上げ順序** | すべてのユーザー（支援技術） |
| 2.4.3 フォーカス順序 | **キーボード Tab 順序** | キーボード操作ユーザー |

- 両方とも DOM 順序と視覚順序の**整合**が求められる
- CSS `order` や `tabindex` の乱用は、1.3.2 と 2.4.3 の**両方**に違反しうる

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.3 Focus Order](https://www.w3.org/TR/WCAG22/#focus-order)

## プレーンテキストでの順序（Failure F33, F34, F32）

- 空白文字だけでカラムや表を再現すると、線形化時に意味が崩れる
- 1.3.1 とも重複する Failure

```text
<!-- F33：空白で複数カラム -->
名前          年齢
田中          25
<!-- → スクリーンリーダーが不自然な順序で読む -->

<!-- F32：空白で単語内スペーシング -->
W C A G
<!-- → 「WCAG」ではなく「W C A G」と読まれる -->
```

- プレーンテキストでは標準的な書式規則（T1, T2, T3）を使用

> **参照**
> - [Failure F33: Using white space to create multiple columns](https://www.w3.org/WAI/WCAG22/Techniques/failures/F33)
> - [Failure F34: Using white space to format tables](https://www.w3.org/WAI/WCAG22/Techniques/failures/F34)
> - [Failure F32: Using white space to control spacing within a word](https://www.w3.org/WAI/WCAG22/Techniques/failures/F32)

## 代表的な失敗例（Failures）

| Failure | 内容 |
|---|---|
| F1 | CSS の配置でコンテンツの意味を変える |
| F32 | 空白文字で単語内スペーシングを制御 |
| F33 | 空白文字で複数カラムを作成 |
| F34 | 空白文字で表を作成 |
| F49 | 線形化すると意味をなさないレイアウト table |

```html
<!-- F1：CSS で段落の意味順序を逆転 -->
<style>
  .intro { order: 2; }
  .conclusion { order: 1; }
</style>
<div style="display: flex; flex-direction: column;">
  <p class="intro">はじめに：背景を説明します。</p>
  <p class="conclusion">まとめ：以上が結論です。</p>
</div>
```

> **参照**
> - [WCAG 2.2 Techniques — Failures for 1.3.2](https://www.w3.org/WAI/WCAG22/Understanding/meaningful-sequence.html#failures)

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.3.1 情報および関係性（A） | 構造・関係性（順序とセットで確認） |
| 1.4.10 リフロー（AA） | 400% 拡大時の1カラム表示でも順序が保たれるか |
| 2.4.3 フォーカス順序（A） | Tab 順序と DOM 順序の整合 |
| 2.4.1 ブロックスキップ（A） | スキップリンク先への `tabindex="-1"` |

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.10 Reflow](https://www.w3.org/TR/WCAG22/#reflow)

## テスト・確認方法

### 自動テスト

- レイアウト table の検出
- 正の `tabindex` の有無
- 一部の CSS order による順序逆転（限定的）
- axe DevTools、WAVE 等で部分的に検出可能

### 手動テスト

- **CSS を無効化**し、DOM 順序で読んで意味が通るか
- スクリーンリーダー（VoiceOver / NVDA）で**上から順に**読み上げ、意味が保たれるか
- **1カラム表示**（ブラウザ幅を狭める、リーダーモード）で順序を確認
- Tab キーでフォーカス移動し、視覚順序と一致するか（2.4.3 も兼ねる）
- マルチカラム・Grid レイアウトでカラムをまたぐ文脈が正しいか

```bash
# 正の tabindex を検出（開発者ツール Console）
[...document.querySelectorAll('[tabindex]')].filter(el => parseInt(el.tabIndex, 10) > 0);

# DOM 順序でテキストコンテンツを確認
[...document.body.querySelectorAll('p,h1,h2,h3,h4,h5,h6,li')].map(el => el.textContent.trim().slice(0, 40));
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.3.2 は**意味に影響する順序**をプログラム的に正しく保つ達成基準（レベル A）
- スクリーンリーダーは **DOM 順序**で読み上げる。CSS で見た目だけ順序を変えない
- `flexbox order`・`position: absolute`・レイアウト table は要注意（F1, F49）
- 独立したコンテンツ（nav / main 等）の相対順序は柔軟。意味のある順序が必要な部分を守る
- DOM 順序 = 論理順序 = 視覚順序（C6）を基本とする
- 1.3.1（構造）・2.4.3（フォーカス順）と併せて確認
