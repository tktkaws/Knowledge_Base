# Flexbox vs Grid — どちらを使うべきか判断基準

## 根本的な違い

| | Flexbox | Grid |
|---|---|---|
| 次元 | **1次元**（行 or 列のどちらか） | **2次元**（行と列を同時に） |
| 設計思想 | コンテンツ主導（中身に合わせて伸縮） | レイアウト主導（枠を決めてから配置） |
| 主な用途 | コンポーネント内の要素配置 | ページ全体やセクションのレイアウト |

> 参照: [MDN - Relationship of grid layout with other layout methods](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Relationship_of_grid_layout_with_other_layout_methods)

## 1次元 vs 2次元

### Flexbox（1次元）

- 要素を**一方向**に並べる
- 折り返し（`flex-wrap`）はできるが、行と列を同時に制御はできない
- 折り返し後の各行は独立しており、列が揃わないことがある

```css
/* Flexboxで折り返すカードレイアウト */
.flex-cards {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

.flex-cards .card {
  flex: 1 1 300px;
}
/* 最後の行のカードが伸びて列が揃わない場合がある */
```

### Grid（2次元）

- **行と列を同時に**定義・制御する
- セルが整然と並び、列の幅が全行で統一される

```css
/* Gridでカードレイアウト */
.grid-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 16px;
}
/* 常に列が揃う */
```

```
Flexbox（flex-wrap: wrap）:
┌──────┬──────┬──────┐
│ 300px│ 300px│ 300px│
├──────┴──┬───┴──────┤
│  450px  │   450px  │  ← 最後の行が伸びる（列が揃わない）
└─────────┴──────────┘

Grid:
┌──────┬──────┬──────┐
│ 300px│ 300px│ 300px│
├──────┼──────┼──────┘
│ 300px│ 300px│          ← 列幅が統一される
└──────┴──────┘
```

## コンテンツ主導 vs レイアウト主導

### Flexbox — コンテンツがレイアウトを決める

- アイテムのサイズや数に応じてレイアウトが変わる
- 「コンテンツの中身に合わせて並べたい」場合に最適

```css
/* ナビゲーション：項目の幅はテキストの長さに依存 */
.nav {
  display: flex;
  gap: 16px;
}
/* 各リンクはテキスト幅に応じた自然なサイズになる */
```

### Grid — レイアウトがコンテンツの配置を決める

- 先にグリッドの構造を定義し、そこにコンテンツを配置する
- 「決まった枠組みに要素をはめ込みたい」場合に最適

```css
/* ダッシュボード：決められたグリッドに配置 */
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr 300px;
  grid-template-rows: 60px 1fr 40px;
  grid-template-areas:
    "header header header"
    "nav    main   aside"
    "footer footer footer";
}
```

> 参照: [MDN - Relationship of grid layout with other layout methods](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Relationship_of_grid_layout_with_other_layout_methods)

## 判断基準チェックリスト

### Flexbox を使うべき場面

- 要素を**一方向**に並べるだけ（横並び or 縦並び）
- アイテムの**数が動的**で、折り返しの制御が不要
- アイテムのサイズが**コンテンツに依存**する
- 要素間の**間隔やアラインメントの調整**が主目的

具体例:
- ナビゲーションメニュー
- ボタングループ
- カードの中身（アイコン + テキストの横並び）
- フォームの入力フィールド + ラベル
- タグやバッジのリスト
- ヘッダー内の要素配置（ロゴ・メニュー・ユーザー情報）

### Grid を使うべき場面

- **行と列の両方**を制御する必要がある
- **整然としたグリッド状**のレイアウトが必要
- レイアウトの**構造が先に決まっている**
- アイテムを**特定の位置に配置**したい
- **重なり**（オーバーラップ）が必要

具体例:
- ページ全体のレイアウト（ヘッダー・サイドバー・メイン・フッター）
- カードグリッド
- 画像ギャラリー
- ダッシュボード
- フォームのラベルと入力欄の2列レイアウト
- 雑誌風の複雑なレイアウト

## 具体的な比較例

### 例1: ナビゲーション → Flexbox

```css
/* 良い例：Flexbox */
.nav {
  display: flex;
  align-items: center;
  gap: 24px;
}

.nav .logo {
  margin-right: auto; /* ロゴを左寄せ、残りを右寄せ */
}
```

```css
/* 悪い例：Grid（過剰） */
.nav {
  display: grid;
  grid-template-columns: auto 1fr auto auto auto;
  align-items: center;
}
/* 項目の数が変わるたびに列定義を変える必要がある */
```

### 例2: カードグリッド → Grid

```css
/* 良い例：Grid */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}
```

```css
/* 悪い例：Flexbox（最後の行の制御が困難） */
.card-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
}
.card {
  flex: 1 1 280px;
  /* 最後の行にカードが1つの場合、幅100%に伸びてしまう */
}
```

### 例3: ページレイアウト → Grid

```css
/* 良い例：Grid */
.page {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  min-height: 100vh;
}
```

```css
/* 悪い例：Flexboxだけで無理に実現 */
.page {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
.page .body {
  display: flex;
  flex: 1;
}
.page .sidebar { flex: 0 0 250px; }
.page .main { flex: 1; }
/* ネストが深くなり、grid-template-areas の直感性が失われる */
```

### 例4: カード内部の要素配置 → Flexbox

```css
/* 良い例：Flexbox */
.card {
  display: flex;
  flex-direction: column;
}
.card .body {
  flex: 1; /* カードの高さが揃うとき、本文が伸びてフッターが下に固定される */
}
.card .footer {
  margin-top: auto;
}
```

### 例5: フォームレイアウト → Grid

```css
/* 良い例：Grid（ラベルと入力欄が綺麗に揃う） */
.form {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 12px 16px;
  align-items: center;
}
/* ラベルの列幅が最も長いラベルに合わせて自動調整される */
```

```css
/* Flexboxの場合：ラベル幅を手動で指定する必要がある */
.form-row {
  display: flex;
  gap: 16px;
  align-items: center;
}
.form-row label {
  width: 120px; /* 固定幅 → 長いラベルが入ると崩れる */
}
```

> 参照: [CSS-Tricks - Quick! What's the Difference Between Flexbox and Grid?](https://css-tricks.com/quick-whats-the-difference-between-flexbox-and-grid/)

## 両方を組み合わせる

- Flexbox と Grid は排他的ではなく、**組み合わせて使う**のが一般的
- Grid でページ全体の構造を定義し、各コンポーネント内部は Flexbox で配置するパターン

```css
/* ページ全体は Grid */
.page {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
}

/* カードグリッドも Grid */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}

/* カード内部は Flexbox */
.card {
  display: flex;
  flex-direction: column;
}

.card .body { flex: 1; }

/* ナビゲーションは Flexbox */
.nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

## 判断フローチャート

```
レイアウトが必要
  │
  ├─ 要素を1方向に並べるだけ？
  │   ├─ はい → Flexbox
  │   └─ いいえ ↓
  │
  ├─ 行と列を同時に制御する必要がある？
  │   ├─ はい → Grid
  │   └─ いいえ ↓
  │
  ├─ アイテムを特定の位置に配置したい？
  │   ├─ はい → Grid
  │   └─ いいえ ↓
  │
  ├─ 等間隔のグリッド状レイアウト？
  │   ├─ はい → Grid
  │   └─ いいえ ↓
  │
  └─ コンテンツサイズに応じて伸縮させたい？
      ├─ はい → Flexbox
      └─ どちらでもよい → 好みで選択
```

## まとめ

- **Flexbox**: 1次元・コンテンツ主導。ナビバー、ボタングループ、カード内部の配置に最適
- **Grid**: 2次元・レイアウト主導。ページレイアウト、カードグリッド、ダッシュボードに最適
- 「一方向に並べるだけ」→ Flexbox、「行と列を同時に制御」→ Grid
- 両者は排他的ではなく、組み合わせて使うのが一般的
- 迷ったら Grid を試す。Grid は Flexbox の上位互換ではないが、より多くの場面に対応できる

## 参照元

- [MDN - Relationship of grid layout with other layout methods](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Relationship_of_grid_layout_with_other_layout_methods)
- [CSS-Tricks - Quick! What's the Difference Between Flexbox and Grid?](https://css-tricks.com/quick-whats-the-difference-between-flexbox-and-grid/)
- [web.dev - Flexbox vs Grid](https://web.dev/learn/css/flexbox)
