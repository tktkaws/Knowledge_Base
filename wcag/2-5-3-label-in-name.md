# 2.5.3 名前（name）のラベル（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.5.3「名前（name）のラベル」（Label in Name）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.5「入力モダリティ」に属する
- テキストまたは文字画像を含むラベルがある UI 部品では、**アクセシブルネームがその視覚ラベルの文字列を含む**こと
- 音声入力ユーザーが、画面上の文言どおりに話して操作できるようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.3 Label in Name](https://www.w3.org/TR/WCAG22/#label-in-name)
> - [Understanding SC 2.5.3 Label in Name](https://www.w3.org/WAI/WCAG22/Understanding/label-in-name.html)

## 達成基準の原文（要約）

- ラベルにテキストまたは文字画像を含むユーザーインタフェースコンポーネントについて、名前（name）は視覚的に提示されているテキストを含むこと
- **注記**：ベストプラクティスとして、ラベルのテキストを名前の先頭に置く

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.3](https://www.w3.org/TR/WCAG22/#label-in-name)

## なぜ必要か

- 音声入力ユーザーは、画面に見えるラベルを話してボタンやリンクを操作する
- 見える文言とプログラム上の名前が違うと、「パスワードへ移動」などが失敗する
- 名前だけが違う隠れたコマンドになり、意図せず起動されることもある
- スクリーンリーダー利用者も、聞こえる名前と見えるラベルが一致している方が分かりやすい
- 認知的な負荷も減る（見たものと違う言葉を覚えなくてよい）

> **参照**
> - [Understanding SC 2.5.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/label-in-name.html#benefits)

## ラベルと名前（name）の違い

| 用語 | 意味 |
|---|---|
| ラベル | ユーザーに見える識別テキスト（`<label>` に限らない） |
| 名前（accessible name） | 支援技術や音声入力が使うプログラム上の名前 |

- 多くの場合、ラベルと名前は一致する
- この基準は、**見えるテキストラベルがある部品**に適用される
- 見えるテキストラベルがない部品には適用されない

```text
適用される例：
  「検索」と書かれたボタン
  「メールアドレス」ラベル付きの入力欄
  「次へ」と描かれた文字画像ボタン

適用されない例：
  アイコンだけで、近くにテキストラベルがないボタン
  （別途 4.1.2 で accessible name は必要）
```

> **参照**
> - [Understanding SC 2.5.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/label-in-name.html#intent)

## 何を「ラベル」とみなすか

- 基本は、部品に**隣接する文字列**だけをラベルとみなす
- 広く解釈しすぎると、予測しにくくなり音声操作の価値が下がる

```text
左から右の言語での典型位置：
  テキスト入力・セレクト：すぐ左、または上で左揃え
  チェックボックス・ラジオ：すぐ右
  ボタン・タブ：内部のテキスト、またはアイコン直下

通常ラベルに含めない：
  離れた見出し
  長い説明文
  fieldset / legend のグループ見出し（グループ説明としては有用）
```

- `placeholder` は本来ラベルの代替ではない
- ただし近くに他のテキストラベルがない場合、音声ユーザーが placeholder を使おうとすることがあり、候補になりうる
- 実務では、見える `<label>` を別に用意するのが安全

> **参照**
> - [Understanding SC 2.5.3 — Identifying label text](https://www.w3.org/WAI/WCAG22/Understanding/label-in-name.html#intent)

## 手法1：見えるラベルと名前を一致させる（G211）

- 最もわかりやすい適合方法
- ネイティブの `<label for>` や、ボタン内テキストをそのまま使う

```html
<!-- 良い例：見えるラベル = 名前 -->
<label for="email">メールアドレス</label>
<input id="email" type="email" name="email" autocomplete="email">

<button type="submit">検索</button>
```

```html
<!-- 悪い例：見えるのは「検索」、名前は別物（F96） -->
<button type="submit" aria-label="サイト内をキーワードで探す">
  検索
</button>
<!-- 音声で「検索」と言っても一致しない -->
```

> **参照**
> - [Technique G211: Matching the accessible name to the visible label](https://www.w3.org/WAI/WCAG22/Techniques/general/G211)

## 手法2：見えるラベル文字列を名前に含める（G208）

- 名前を少し詳しくしてもよいが、**見える文字列を含める**
- ベストプラクティスは、見えるラベルを**名前の先頭**に置く

```html
<!-- 良い例：見える「検索」が名前の先頭にある -->
<button type="button" aria-label="検索 商品名または型番">
  検索
</button>
```

```html
<!-- 悪い例：順番が入れ替わり、余計な語が挟まる -->
<button type="button" aria-label="商品を今すぐ検索する">
  検索
</button>
<!-- 「検索」は含まれるが、先頭にない／語順が崩れていると音声認識で不利になりうる -->
```

```html
<!-- 良い例：aria-labelledby で見えるテキストを参照する -->
<span id="save-label">保存</span>
<button type="button" aria-labelledby="save-label">保存</button>
```

> **参照**
> - [Technique G208: Including the text of the visible label as part of the accessible name](https://www.w3.org/WAI/WCAG22/Techniques/general/G208)
> - [Understanding SC 2.5.3 — accessible name starts with visible label](https://www.w3.org/WAI/WCAG22/Understanding/label-in-name.html#examples)

## `aria-label` の注意

- `aria-label` / `aria-labelledby` は、ネイティブの見えるテキストより優先されることがある
- 見えるラベルがあるのに `aria-label` で上書きすると、不一致が起きやすい
- 見えるラベルがあるなら、まずそのテキストを名前の根拠にする
- 補足説明は `aria-describedby` に回す（名前計算には通常含まれない）

```html
<!-- 悪い例：aria-label が見えるラベルを上書き -->
<label for="pwd">パスワード</label>
<input id="pwd" type="password" aria-label="秘密の語句">

<!-- 良い例：label 関連付けを使い、補足は description へ -->
<label for="pwd">パスワード</label>
<input
  id="pwd"
  type="password"
  autocomplete="current-password"
  aria-describedby="pwd-help"
>
<p id="pwd-help">8文字以上で入力してください。</p>
```

```html
<!-- 良い例：形式の補足も description へ -->
<label for="date">日付</label>
<input
  id="date"
  name="date"
  aria-describedby="date-format"
>
<span id="date-format">形式：YYYY-MM-DD</span>
```

## 記号・句読点・数式

### 記号的な文字

- 「B」「I」「ABC」など、機能を象徴するだけの文字は、人間言語のラベルとはみなさない場合がある
- この場合、名前は「太字」「スペルチェック」など**機能名**にする

```html
<!-- 良い例：見えるのは象徴文字、名前は機能 -->
<button type="button" aria-label="太字">
  <span aria-hidden="true">B</span>
</button>
```

### 句読点と大文字小文字

- 音声入力では、大文字小文字や多くの句読点は無視されやすい
- 「First Name:」と名前「first name」は同等とみなせる
- 「Next…」と名前「Next」も同等とみなせる
- ただしスクリーンリーダー向けには、大文字小文字を揃えるのが望ましい

### 数式

- 「A>B」「11×3=33」のような数式ラベルは、そのまま名前に残す
- スペルアウト（eleven times three...）に置き換えない
- 正しい演算子記号（`×` など）を使う

> **参照**
> - [Understanding SC 2.5.3 — symbolic text / maths](https://www.w3.org/WAI/WCAG22/Understanding/label-in-name.html#intent)

## 括弧書きの扱い

- 「氏名（必須）」「日付（YYYY-MM-DD）」の括弧内は、音声では言われにくい
- 名前を「氏名」「日付」だけにしてもよい場合がある
- ただし必須や形式制限は、`required` / `aria-required` / `aria-describedby` などでプログラム上伝える（1.3.1）

```html
<!-- 良い例：括弧の重要情報を別途伝える -->
<label for="name">氏名</label>
<input id="name" name="name" required aria-describedby="name-req">
<span id="name-req">必須</span>
```

## よくある失敗

### F96：名前に見えるラベル文字列がない

```html
<!-- 失敗例 -->
<button aria-label="カートの中身を確認する">カートを見る</button>
```

```html
<!-- 改善例 -->
<button>カートを見る</button>
<!-- または -->
<button aria-label="カートを見る 中身を確認">カートを見る</button>
```

### F111：見えるラベルはあるが名前がない

- 見えるテキストがあるのに accessible name が空
- 1.3.1 / 4.1.2 も同時に失敗しやすい

```html
<!-- 失敗例：ラベルっぽいテキストが関連付いていない -->
<span>メールアドレス</span>
<input type="email">
```

```html
<!-- 改善例 -->
<label for="email">メールアドレス</label>
<input id="email" type="email">
```

> **参照**
> - [Failure F96: accessible name not containing the visible label text](https://www.w3.org/WAI/WCAG22/Techniques/failures/F96)
> - [Failure F111: visible label text but no accessible name](https://www.w3.org/WAI/WCAG22/Techniques/failures/F111)

## 2.4.6・3.3.2・4.1.2 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.6 見出しおよびラベル | ラベルが説明的か | 文言の質 |
| 2.5.3 名前のラベル | 見えるラベルが名前に含まれるか | 音声操作との一致 |
| 3.3.2 ラベルまたは説明 | 入力にラベルがあるか | ラベルの存在 |
| 4.1.2 名前・役割・値 | accessible name があるか | 名前の存在 |

```text
よくある誤解：
  aria-label があるから 2.5.3 は合格
  → 見える文言と一致・包含していなければ不合格

  ラベル文言が分かりやすいから 2.5.3 は合格
  → それは主に 2.4.6。名前との一致は別問題
```

> **参照**
> - [wcag/2-4-6-headings-and-labels.md — 2.4.6 見出しおよびラベル](./2-4-6-headings-and-labels.md)
> - [WCAG 2.2 — Success Criterion 4.1.2 Name, Role, Value](https://www.w3.org/TR/WCAG22/#name-role-value)

## 実装時の注意点

- 見えるラベルがある部品では、その文字列を名前に含める
- 可能なら完全一致、少なくとも先頭一致を狙う
- 安易な `aria-label` 上書きを避ける
- 補足は `aria-describedby` へ
- アイコン＋テキストボタンは、テキスト側を名前の主根拠にする
- 翻訳時も、見える文言と名前がずれないよう同じソースから生成する

```text
実務チェック：
  画面のボタン文言を音声で言って操作できるか
  aria-label が見える文言を消していないか
  「詳細を見る」なのに名前が「もっと読む」になっていないか
```

## テスト・確認方法

### 手動テスト

- テキストラベルがあるコントロールを洗い出す
- DevTools の Accessibility パネルで accessible name を確認する
- 見える文字列が名前に含まれているか確認する
- 音声入力やスクリーンリーダーで、見える文言どおりに操作・読み上げできるか確認する

```text
チェックリスト：
1. 見えるテキストラベルがあるか
2. accessible name があるか（なければ F111 / 4.1.2）
3. 名前に見えるラベル文字列が含まれるか
4. 可能なら先頭一致・語順一致か
5. aria-label で不必要に上書きしていないか
```

### 開発者ツールでの確認

```javascript
// Console 概念：ボタンの表示テキストと aria-label の食い違いを探す
[...document.querySelectorAll("button[aria-label]")].map((btn) => ({
  visible: btn.textContent.replace(/\s+/g, " ").trim(),
  name: btn.getAttribute("aria-label"),
}))
```

```bash
# aria-label 上書きを探す
rg "aria-label=" --glob "*.html" --glob "*.tsx" --glob "*.jsx" --glob "*.php"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.5.3 は、見えるテキストラベルがある部品で、**その文字列がアクセシブルネームに含まれる**ことを求める達成基準（レベル A）
- 音声入力ユーザーが、画面の文言どおりに話して操作できるようにするのが目的
- 最も安全なのは、見えるラベルと名前を一致させること（G211）
- 名前を詳しくする場合も、見える文字列を含め、できれば先頭に置く（G208）
- `aria-label` の上書きミス（F96）と、ラベルはあるが名前がない状態（F111）に注意する
