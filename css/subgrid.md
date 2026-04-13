# subgridの使い方

## subgridとは

- 親グリッドのトラック（行・列）定義を**子グリッドが継承**できる機能
- 通常のネストされたグリッドでは、親と子のトラックは完全に独立している
- subgridを使うと、孫要素が**親グリッドのラインに揃う**ようになる
- カードレイアウトで「すべてのカードのタイトル・本文・ボタンの高さを揃えたい」といった場面で特に有効

> 参照: [MDN - Subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid)

## subgridが解決する問題

### ネストされたグリッドの課題

```
通常のネストされたグリッド（親と子のトラックが独立）:
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ 短いタイトル   │ │ とても長い     │ │ 中くらいの     │
│              │ │ タイトルが     │ │ タイトル       │
│              │ │ ここに入る     │ │              │
├──────────────┤ ├──────────────┤ ├──────────────┤
│ 本文テキスト   │ │ 本文          │ │ 本文テキスト   │
│ が入る        │ │              │ │ がここに入る   │
├──────────────┤ ├──────────────┤ ├──────────────┤
│  [ボタン]     │ │  [ボタン]     │ │  [ボタン]     │
└──────────────┘ └──────────────┘ └──────────────┘
 ↑ 各カード内で   ↑ タイトルの高さ  ↑ バラバラ
   独自にレイアウト   がカードごとに異なる
```

```
subgridを使った場合（親のトラックを継承）:
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ 短いタイトル   │ │ とても長い     │ │ 中くらいの     │
│              │ │ タイトルが     │ │ タイトル       │
│              │ │ ここに入る     │ │              │
├──────────────┤ ├──────────────┤ ├──────────────┤
│ 本文テキスト   │ │ 本文          │ │ 本文テキスト   │
│ が入る        │ │              │ │ がここに入る   │
├──────────────┤ ├──────────────┤ ├──────────────┤
│  [ボタン]     │ │  [ボタン]     │ │  [ボタン]     │
└──────────────┘ └──────────────┘ └──────────────┘
 ← 全カードで行の高さが揃う（最も高いコンテンツに合わせる） →
```

- subgridなしでは各カード内部のレイアウトが独立し、タイトルや本文の高さが揃わない
- subgridを使うと、各カードの内部要素が**親グリッドの行ライン**に揃い、統一感のあるレイアウトになる

> 参照: [web.dev - CSS subgrid](https://web.dev/articles/css-subgrid)

## 基本構文

```css
.parent {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto auto auto;
  gap: 1rem;
}

.child {
  display: grid;
  /* 列を親から継承 */
  grid-template-columns: subgrid;

  /* または行を親から継承 */
  grid-template-rows: subgrid;

  /* または両方を親から継承 */
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
}
```

- `subgrid` は `grid-template-columns` または `grid-template-rows` の値として指定する
- 子要素は `display: grid` を指定した上で、継承したい軸に `subgrid` を設定する
- 片方の軸だけ `subgrid` にし、もう片方は独自のトラック定義にすることも可能

> 参照: [MDN - Subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid)

## 行方向のsubgrid（カードレイアウト）

### カード内の要素の高さを揃える

```html
<div class="card-grid">
  <article class="card">
    <h2>タイトル1</h2>
    <p>短い本文</p>
    <a href="#">もっと見る</a>
  </article>
  <article class="card">
    <h2>長いタイトルが2行にわたる場合</h2>
    <p>こちらも本文テキストが入ります。少し長めの本文です。</p>
    <a href="#">もっと見る</a>
  </article>
  <article class="card">
    <h2>タイトル3</h2>
    <p>本文テキスト</p>
    <a href="#">もっと見る</a>
  </article>
</div>
```

```css
/* 悪い例：subgridなし → 各カード内の行の高さがバラバラ */
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;
}
.card {
  display: grid;
  grid-template-rows: auto 1fr auto;
  /* 各カードが独自にトラックを定義 → カード間で揃わない */
}

/* 良い例：subgridで親の行トラックを継承 → 全カードの行が揃う */
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: subgrid; /* ← ここではなく子側で指定 */
  gap: 1.5rem;
}
```

```css
/* 正しい実装 */
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* 行トラックを3行分定義（タイトル・本文・リンク） */
  grid-template-rows: auto 1fr auto;
  gap: 1.5rem;
}

.card {
  display: grid;
  /* 親の行トラックを継承する */
  grid-template-rows: subgrid;
  /* カードが3行分を占めるように指定 */
  row-gap: 0.75rem;
}
```

- `.card` は親の行トラック定義を継承するため、全カードでタイトル行・本文行・リンク行の高さが揃う
- カード内の最も高いコンテンツに合わせて、すべてのカードの対応する行が同じ高さになる

> 参照: [web.dev - CSS subgrid](https://web.dev/articles/css-subgrid)

## 列方向のsubgrid（フォームレイアウト）

### ラベルと入力欄を揃える

```html
<form class="form">
  <div class="form-group">
    <label>名前</label>
    <input type="text">
  </div>
  <div class="form-group">
    <label>メールアドレス</label>
    <input type="email">
  </div>
  <div class="form-group">
    <label>お問い合わせ内容</label>
    <textarea></textarea>
  </div>
</form>
```

```css
.form {
  display: grid;
  /* ラベル列とインプット列 */
  grid-template-columns: auto 1fr;
  gap: 1rem 1.5rem;
}

.form-group {
  display: grid;
  /* 親の列トラックを継承 */
  grid-template-columns: subgrid;
  /* このグループが2列分を占める */
  grid-column: span 2;
}
```

- 各 `.form-group` が親の列定義を継承するため、すべての行でラベル列の幅が統一される
- ラベルの最大幅に合わせて `auto` 列が決まり、残りを `1fr` の入力欄が占める

> 参照: [MDN - Subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid)

## 子要素が占める範囲の指定

- subgridを使う子要素は、親グリッドの**どのトラックを継承するか**を `grid-column` / `grid-row` で指定する必要がある

```css
.parent {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(4, auto);
}

/* 2列×3行分の範囲を subgrid として使う */
.child {
  grid-column: 1 / 3;  /* 列ライン1〜3（2列分） */
  grid-row: 1 / 4;     /* 行ライン1〜4（3行分） */
  display: grid;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
}
```

```
親グリッド (4列 × 4行):
  1     2     3     4     5
1 ┌─────┬─────┬─────┬─────┐
  │ .child      │     │     │
2 │(subgrid     │     │     │
  │ 2列×3行)    │     │     │
3 │             │     │     │
  │             │     │     │
4 ├─────┴─────┼─────┼─────┤
  │           │     │     │
5 └─────┴─────┴─────┴─────┘
```

- `grid-column: span 2` のように `span` でも指定可能
- 指定した範囲内の親トラックが、子のsubgridとして使われる

## gapの継承と上書き

```css
.parent {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2rem;
}

.child {
  display: grid;
  grid-template-columns: subgrid;
  /* デフォルトでは親のgapを継承する */

  /* 子のgapだけ変更したい場合は上書き可能 */
  column-gap: 0.5rem;
}
```

- subgridはデフォルトで**親のgapを継承する**
- 子要素で `gap`、`row-gap`、`column-gap` を指定すると上書きできる
- gapだけ変えたい場合に便利

> 参照: [MDN - Subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid)

## 名前付きラインの継承

```css
.parent {
  display: grid;
  grid-template-columns:
    [content-start] 1fr
    [content-end sidebar-start] 300px
    [sidebar-end];
}

.child {
  grid-column: content-start / sidebar-end;
  display: grid;
  grid-template-columns: subgrid;
  /* 親の名前付きラインをそのまま使える */
}

.child-item {
  grid-column: content-start / content-end;
}
```

- 親グリッドで定義した名前付きラインは、subgridの子要素でもそのまま参照できる
- 複雑なレイアウトでもセマンティックなライン名で配置を指定できる

### subgrid側で追加のライン名を付ける

```css
.child {
  display: grid;
  grid-template-columns: subgrid [child-start] [child-middle] [child-end];
}
```

- subgridの各ラインに追加の名前を付与できる
- 親のライン名と子のライン名の両方が使える

> 参照: [MDN - Subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid)

## 実践的な活用例

### ニュース記事カードグリッド

```css
.news-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  grid-template-rows: auto auto 1fr auto;
  gap: 1.5rem;
}

.news-card {
  display: grid;
  grid-template-rows: subgrid;
  row-gap: 0.5rem;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
}

.news-card img {
  width: 100%;
  aspect-ratio: 16 / 9;
  object-fit: cover;
}

.news-card .title {
  font-size: 1.25rem;
  padding-inline: 1rem;
}

.news-card .excerpt {
  padding-inline: 1rem;
  color: #666;
}

.news-card .meta {
  padding: 0.75rem 1rem;
  border-top: 1px solid #e0e0e0;
  font-size: 0.875rem;
  align-self: end;
}
```

### ダッシュボードのウィジェット

```css
.dashboard {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
}

.widget {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.widget-header {
  padding: 1rem;
  font-weight: bold;
  border-bottom: 1px solid #e0e0e0;
}

.widget-body {
  padding: 1rem;
}

.widget-footer {
  padding: 0.75rem 1rem;
  border-top: 1px solid #e0e0e0;
  font-size: 0.875rem;
}

/* 幅2列分のワイドウィジェット */
.widget--wide {
  grid-column: span 2;
}
```

> 参照: [web.dev - CSS subgrid](https://web.dev/articles/css-subgrid)

## subgridと通常のネストされたグリッドの使い分け

| 特徴 | 通常のネストされたグリッド | subgrid |
|---|---|---|
| トラック定義 | 親と子で独立 | 親のトラックを継承 |
| 要素間の揃え | 親子間では揃わない | 親のラインに揃う |
| gap | 親と子で独立 | 親を継承（上書き可） |
| 適した場面 | 独立したレイアウトが必要な場合 | 兄弟要素間でコンテンツを揃えたい場合 |

```css
/* 通常のネストされたグリッドが適する場面 */
/* → 子の内部レイアウトが親と無関係 */
.sidebar {
  display: grid;
  grid-template-rows: auto 1fr auto;
  /* サイドバー内部だけの独自レイアウト */
}

/* subgridが適する場面 */
/* → 兄弟要素と揃えたい */
.card {
  display: grid;
  grid-template-rows: subgrid;
  /* 隣のカードと各行の高さを合わせたい */
}
```

## ブラウザサポート

- Chrome 117+、Safari 16+、Firefox 71+ で全主要ブラウザがサポート
- Firefoxが最初にサポートし、2023年にChrome・Safariが追随して全ブラウザ対応が完了
- 2024年以降、プロダクションで安心して使えるレベルのサポート状況

### フォールバック

```css
/* subgridをサポートしないブラウザ向けのフォールバック */
.card {
  display: grid;
  grid-template-rows: auto 1fr auto;
  /* 独自のトラック定義で最低限のレイアウトを確保 */
}

/* subgridをサポートするブラウザのみ */
@supports (grid-template-rows: subgrid) {
  .card {
    grid-template-rows: subgrid;
  }
}
```

- `@supports (grid-template-rows: subgrid)` で機能検出が可能
- フォールバックとして独自のトラック定義を用意し、subgrid対応ブラウザでは上書きするのが定番パターン

> 参照: [Can I Use - CSS Subgrid](https://caniuse.com/css-subgrid)

## まとめ

- subgridは親グリッドのトラック定義を子グリッドが継承する機能
- `grid-template-columns: subgrid` または `grid-template-rows: subgrid` で指定する
- 片方の軸だけ継承し、もう片方は独自のトラック定義にすることも可能
- **カードレイアウトで各カードの内部要素の高さを揃える**のが最も典型的な用途
- gapはデフォルトで親を継承し、子側で上書きもできる
- 名前付きラインも継承され、子側で追加の名前も付けられる
- 全主要ブラウザでサポート済み、プロダクションで利用可能

## 参照元

- [MDN - Subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid)
- [web.dev - CSS subgrid](https://web.dev/articles/css-subgrid)
- [Can I Use - CSS Subgrid](https://caniuse.com/css-subgrid)
