# 1.4.10 リフロー（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.10「リフロー」（Reflow）
- レベル **AA**（標準的な適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- WCAG 2.1 で追加された達成基準
- コンテンツを**情報や機能を失わず**、**2方向スクロールなし**で提示できること
- 縦スクロールコンテンツ：**幅 320 CSS px 相当**
- 横スクロールコンテンツ：**高さ 256 CSS px 相当**

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.10 Reflow](https://www.w3.org/TR/WCAG22/#reflow)
> - [Understanding SC 1.4.10 Reflow](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html)

## 達成基準の原文（要約）

- 次の条件で、情報や機能を失わず、2方向スクロールを要求しないこと
  - **縦スクロールコンテンツ**：幅 320 CSS px 相当
  - **横スクロールコンテンツ**：高さ 256 CSS px 相当
- 例外：使用や意味のために**2次元レイアウトが必要**なコンテンツの一部
- 注記1：320 CSS px は、1280 CSS px 幅のビューポートを **400% ズーム**した場合に相当
- 注記2：地図・図表・動画・ゲーム・プレゼンテーション・データ表（個々のセルではない）・ツールバーを常時表示する必要があるインターフェースなどは例外

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.10](https://www.w3.org/TR/WCAG22/#reflow)

## なぜ必要か

- ロービジョンのユーザーがテキストを拡大しても、行ごとに左右にスクロールしなくてよいようにするため
- 行がビューポートを超えると、読み位置を失いやすく、身体的・認知的負担が増える
- テキストが折り返されれば、読む方向への1方向スクロールだけで続きを読める
- モバイル端末だけでなく、デスクトップでズームするユーザーも対象

> **参照**
> - [Understanding SC 1.4.10 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html#intent)
> - [Understanding SC 1.4.10 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html#intent)

## 320 CSS px と 400% ズーム

- 320 CSS px は、1280 CSS px 幅のビューポートを **400% ズーム**したときの幅に相当
- 400% は面積ではなく**寸法**の4倍（幅・高さそれぞれ4倍）
- 256 CSS px は、1024 CSS px 高さのビューポートを 400% ズームしたときの高さに相当（縦書き言語向け）
- テストは必ずしも 1280px × 400% に限定されない
  - 960px 幅を 300% ズーム → 320 CSS px 相当
  - ブラウザUIの存在で実際のビューポートは理論値より小さくなることもある

```text
縦スクロール（日本語・英語など）：
  1280 CSS px 幅 × 400% ズーム ≒ 320 CSS px 幅
  → 横スクロールなしで読めること

横スクロール（縦書きなど）：
  1024 CSS px 高さ × 400% ズーム ≒ 256 CSS px 高さ
  → 縦スクロールなしで読めること
```

> **参照**
> - [Understanding SC 1.4.10 — Why specifically 320px and 256px?](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html#why-specifically-320px-and-256px)

## 基本方針：1カラムにリフロー

- 記事・ニュース・ブログなど、読むことが主目的のページは**1カラム**に積み上げる
- サイドバー、広告、ウィジェットは縦に並べ替えても情報・機能が失われなければよい
- ナビゲーションはハンバーガーメニュー等に切り替えてもよい
- レスポンシブデザインの一般的なベストプラクティスと整合

```css
/* 良い例：狭い幅で1カラムに */
.layout {
  display: grid;
  grid-template-columns: 1fr 300px;
  gap: 2rem;
}

@media (max-width: 48rem) {
  .layout {
    grid-template-columns: 1fr;
  }
}
```

```html
<!-- 良い例：メインとサイドバーが縦に積み上がる -->
<div class="layout">
  <main>
    <h1>記事タイトル</h1>
    <p>本文テキスト...</p>
  </main>
  <aside aria-label="関連記事">
    <h2>関連記事</h2>
    <ul>...</ul>
  </aside>
</div>
```

> **参照**
> - [Technique C32: Using media queries and grid CSS to reflow columns](https://www.w3.org/WAI/WCAG22/Techniques/css/C32)
> - [Technique C31: Using CSS Flexbox to reflow content](https://www.w3.org/WAI/WCAG22/Techniques/css/C31)

## Flexbox・Grid でリフロー（C31, C32）

- `flex-wrap: wrap` で要素を折り返す
- `grid-template-columns: repeat(auto-fit, minmax(...))` でカードを流動配置
- 固定幅の多段カラムを避ける

```css
/* 良い例：カードが折り返す */
.cards {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.card {
  flex: 1 1 18rem;
  max-width: 100%;
}

/* 良い例：Grid で自動折り返し */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
  gap: 1rem;
}
```

> **参照**
> - [Technique C31: Using CSS Flexbox to reflow content](https://www.w3.org/WAI/WCAG22/Techniques/css/C31)
> - [Technique C32: Using media queries and grid CSS to reflow columns](https://www.w3.org/WAI/WCAG22/Techniques/css/C32)

## 長いURL・文字列の折り返し（C33）

- 長いURL、認証キー、ハッシュ値などが横にはみ出さないようにする
- `overflow-wrap: break-word` や `word-break: break-all` を活用
- 省略表示する場合は、全文を読める手段を提供

```css
/* 悪い例：長いURLが横スクロールを発生 */
.url-bad {
  white-space: nowrap;
}

/* 良い例：折り返しを許可 */
.url-good {
  overflow-wrap: anywhere;
  word-break: break-word;
}
```

```html
<!-- 良い例：省略 + 全文表示手段 -->
<p class="truncated" title="https://example.com/very/long/path/to/resource">
  https://example.com/very/long/...
</p>
<a href="/full-url">全文を表示</a>
```

> **参照**
> - [Technique C33: Allowing for Reflow with Long URLs and Strings of Text](https://www.w3.org/WAI/WCAG22/Techniques/css/C33)

## ラベルと入力欄のリフロー（C38）

- フォームのラベルと入力欄が狭い幅で重ならないようにする
- `flex-wrap`、`max-width: 100%` で折り返し
- ラベルが入力欄の上に積み上がってもよい

```css
/* 良い例：ラベルと入力が折り返す */
.form-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem 1rem;
  align-items: center;
}

.form-row label {
  flex: 0 0 auto;
  max-width: 100%;
}

.form-row input {
  flex: 1 1 12rem;
  max-width: 100%;
  min-width: 0;
}
```

> **参照**
> - [Technique C38: Using CSS width, max-width and flexbox to fit labels and inputs](https://www.w3.org/WAI/WCAG22/Techniques/css/C38)

## viewport の設定

- `user-scalable=no` や `maximum-scale=1` はズームを妨げ、1.4.4・1.4.10 の両方に影響
- ズームを許可する `viewport` メタタグを使う

```html
<!-- 悪い例：ズームを禁止 -->
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">

<!-- 良い例：ズームを許可 -->
<meta name="viewport" content="width=device-width, initial-scale=1">
```

> **参照**
> - [ACT Rule: Meta viewport allows for zoom](https://www.w3.org/WAI/standards-guidelines/act/rules/b4f0c3/)
> - [wcag/1-4-4-resize-text.md — 1.4.4 テキストのサイズ変更](./1-4-4-resize-text.md)

## 固定幅・横スクロールの回避

- `width: 960px` など固定幅のコンテナは、ズーム時に横スクロールを発生させやすい
- `min-width` の過大な指定、`white-space: nowrap`、横スクロール前提のレイアウトに注意
- `width: min(100%, 60rem)` のように上限を持たせつつ流動させる

```css
/* 悪い例：固定幅で横スクロール */
.page-bad {
  width: 1200px;
  min-width: 1200px;
}

/* 良い例：流動幅 */
.page-good {
  width: min(100%, 75rem);
  margin-inline: auto;
  padding-inline: 1rem;
}
```

## 例外：2次元レイアウトが必要なコンテンツ

- 使用や意味のために2次元レイアウトが必要な部分は例外
- 例外は**そのセクションに限定**され、ページ全体には及ばない
- 例
  - 理解に必要な画像（地図、図表、グラフ）
  - 動画
  - ゲーム
  - プレゼンテーション
  - データ表（個々のセルではない表全体）
  - ツールバーを常時表示する必要がある編集インターフェース
  - スライドデッキの作成・表示

```html
<!-- 良い例：表は独自のスクロールコンテナ内 -->
<h2>ファイル一覧</h2>
<label for="search">検索</label>
<input id="search" type="search">

<div class="table-scroll" tabindex="0">
  <table>
    <caption>アクセス可能なファイル</caption>
    <!-- 列が多い表 -->
  </table>
</div>

<nav aria-label="ページネーション">...</nav>
```

```css
/* 良い例：表だけ2方向スクロール */
.table-scroll {
  overflow: auto;
  max-width: 100%;
}

/* 見出し・検索・ページネーションはリフローする */
```

- 表の**前後**にある見出し、検索欄、ページネーションは例外にならない
- 表内の**個々のセル**はリフローを満たす必要がある（セル内のコンテンツが2次元レイアウトを必要としない限り）

> **参照**
> - [Understanding SC 1.4.10 — Content that meets exceptions](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html#content-that-meets-exceptions-for-reflow)

## 画像・動画の扱い（C37）

- 画像・動画は本質的に2次元だが、`max-width: 100%` でビューポート内に収められる
- 収められない場合は例外として2次元スクロールが許容される
- ただし、他のコンテンツが横スクロールを引き起こさないよう配慮

```css
/* 良い例：メディアをコンテナ内に収める */
img,
video,
iframe {
  max-width: 100%;
  height: auto;
}
```

```html
<!-- 悪い例：動画が固定幅のままページ全体を横スクロールさせる -->
<video width="1280" height="720" controls src="demo.mp4"></video>

<!-- 良い例：レスポンシブに収める -->
<video class="responsive-video" controls src="demo.mp4"></video>
```

> **参照**
> - [Technique C37: Using CSS max-width and height to fit images](https://www.w3.org/WAI/WCAG22/Techniques/css/C37)

## カルーセル（G225）

- 縦スクロールページ内のカルーセルは、各パネルが 320 CSS px 幅に収まるように設計
- カルーセル全体の横スクロールは許容されるが、**各パネル内**は縦方向に読めること
- パネル内のコンテンツが 320 CSS px を超えると不適合

```html
<!-- 良い例：各パネルが320px幅に収まる -->
<div class="carousel" role="region" aria-label="物件一覧">
  <article class="carousel-panel">
    <h3>物件A</h3>
    <p>東京都...</p>
    <p>賃料：8万円</p>
  </article>
  <!-- 横スクロールで次パネルへ -->
</div>
```

> **参照**
> - [Technique G225: Section panels that scroll horizontally are designed to fit within a width of 320 CSS pixels](https://www.w3.org/WAI/WCAG22/Techniques/general/G225)

## コード・インデント（G224）

- コードブロックやネストしたリストは、インデントが意味を持つ場合がある
- インデントを削除すると意味や機能が損なわれる場合は例外
- ズーム時はインデント幅を縮小し、テキストが大きくなっても階層が分かるようにする

```css
/* 良い例：ズーム時にインデントを調整 */
pre code {
  font-size: 0.875em;
  overflow-x: auto;
  max-width: 100%;
}

@media (max-width: 20rem) {
  pre {
    padding-left: 0.5em;
  }
}
```

> **参照**
> - [Technique G224: Accounting for meaningful text indentation and Reflow](https://www.w3.org/WAI/WCAG22/Techniques/general/G224)

## 固定・スティッキーヘッダー（C34）

- `position: fixed` や `sticky` のヘッダー・フッター・広告は、ズーム時にコンテンツを覆いやすい
- 狭いビューポートでは `static` に戻す、またはユーザーが非表示にできるようにする
- 2.4.11「フォーカスの非遮蔽（最低限）」とも重なる

```css
/* 悪い例：固定広告が読み取り領域を圧迫 */
.ad-banner {
  position: fixed;
  bottom: 0;
  width: 100%;
  height: 120px;
}

/* 良い例：狭い幅では通常フローに */
@media (max-width: 48rem) {
  .ad-banner {
    position: static;
    height: auto;
  }
}
```

> **参照**
> - [Technique C34: Using media queries to un-fixing sticky headers / footers](https://www.w3.org/WAI/WCAG22/Techniques/css/C34)
> - [Understanding SC 1.4.10 — Focus Not Obscured overlap](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html#focus-not-obscured-minimum)

## レイアウト切り替え（G206）

- 2次元レイアウトが必要なコンテンツに、リフロー対応の代替表示を提供
- 例：2カラム比較ビューと、1カラム積み上げビューの切り替え

```html
<button type="button" id="switch-layout" aria-pressed="false">
  1カラム表示に切り替え
</button>

<div class="diff-view" data-layout="side-by-side">
  <section class="diff-original">...</section>
  <section class="diff-modified">...</section>
</div>
```

> **参照**
> - [Technique G206: Providing options within the content to switch to a layout that does not require horizontal scrolling](https://www.w3.org/WAI/WCAG22/Techniques/general/G206)

## 1.4.4 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 1.4.4 テキストのサイズ変更 | テキストを200%まで拡大 | 拡大しても内容・機能が失われない |
| 1.4.10 リフロー | 320 CSS px 幅で2方向スクロールなし | 拡大時に行が折り返される |

- 1.4.4 と 1.4.10 は目的が重なるが、別の達成基準
- 200% ズームでビューポートが 320 CSS px より小さくなれば、2方向スクロールが起きても 1.4.10 の失敗にはならない（ビューポートが要件未満のため）
- 1.4.4 では 200% 拡大が必要だが、1.4.10 自体は特定の文字サイズ拡大を要求しない
- レスポンシブで小画面向けに文字サイズを下げる場合でも、どこかで 200% 拡大が可能である必要がある（1.4.4）

```text
1280px 幅 × 100% ズーム → テキスト 20px
1280px 幅 × 200% ズーム → テキスト視覚的に 2倍
1280px 幅 × 400% ズーム → 320px 幅にリフロー（1.4.10 のテスト条件）
```

> **参照**
> - [wcag/1-4-4-resize-text.md — 1.4.4 テキストのサイズ変更](./1-4-4-resize-text.md)
> - [Understanding SC 1.4.10 — Resize Text overlap](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html#resize-text)

## よくある失敗例

### F102：リフロー後にコンテンツが消える

- リフロー時にコンテンツが非表示になり、アクセスできなくなる
- `display: none` への切り替えで重要な情報が失われる
- ハンバーガーメニュー内に隠れた情報へのアクセス手段がない

```html
<!-- 悪い例：狭い幅でナビゲーションが完全に消え、代替手段がない -->
<nav class="desktop-nav">...</nav>
<style>
  @media (max-width: 20rem) {
    .desktop-nav { display: none; }
    /* モバイルメニューの提供なし → F102 */
  }
</style>
```

### 表の例外をページ全体に適用

- 表は2次元レイアウトの例外だが、表の前後の段落は例外にならない
- ページ全体が2方向スクロールになる実装は不適合

### 固定広告・スティッキーヘッダー

- ズーム時に読み取り領域を大幅に圧迫
- キーボードフォーカスを覆う（2.4.11 とも関連）

> **参照**
> - [Failure F102: Content disappearing and not being available when content has reflowed](https://www.w3.org/WAI/WCAG22/Techniques/failures/F102)

## テスト・確認方法

### 手動テスト

- ブラウザウィンドウを **1280 CSS px** 幅に設定
- **400% ズーム**（または 320 CSS px 相当になるズーム率）で確認
- 縦方向の1方向スクロールだけで本文を読めるか確認
- 横スクロールが必要な場合、その部分が2次元レイアウトの例外か判断
- ナビゲーション、フォーム、モーダル、固定ヘッダー・広告も確認
- コンテンツが消えていないか、代替手段があるか確認

```text
テスト手順（縦スクロールコンテンツ）：
1. ビューポート幅 1280 CSS px に設定
2. 400% ズーム
3. 横スクロールが必要か確認
4. 必要なら例外コンテンツか判断
5. 非例外コンテンツが2方向スクロールしていないか確認
```

### 開発者ツールでの確認

- 固定幅（`width: 1200px` 等）、`min-width`、`white-space: nowrap` を重点確認
- `position: fixed` / `sticky` のヘッダー・フッター・広告
- `viewport` メタタグの `maximum-scale` / `user-scalable`
- 表が独自スクロールコンテナ内にあるか

```bash
# CSSでリフロー時に問題になりやすい指定を確認
rg "min-width:|white-space:\\s*nowrap|position:\\s*fixed|maximum-scale|user-scalable=no" --glob "*.css" --glob "*.scss" --glob "*.html"
```

### ACT Rules

- [Meta viewport allows for zoom](https://www.w3.org/WAI/standards-guidelines/act/rules/b4f0c3/)

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.4.10 は、**320 CSS px 幅**（1280px × 400% ズーム相当）で**2方向スクロールなし**にコンテンツを提示する達成基準（レベル AA）
- 基本方針はレスポンシブデザインによる**1カラムへのリフロー**
- 地図・表・動画・ゲーム・編集ツールなど、2次元レイアウトが必要な部分は例外
- 例外はそのセクションに限定。見出し・検索・ページネーションはリフローが必要
- 固定幅、長いURL、`white-space: nowrap`、固定広告に注意
- 1.4.4（200%拡大）・2.4.11（フォーカスの非遮蔽）と併せて確認
