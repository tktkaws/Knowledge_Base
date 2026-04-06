# CSS Gridの基本 — グリッドラインとエリアの定義

## CSS Gridとは

- 2次元のレイアウトモデル（行と列を同時に制御できる）
- 行（row）と列（column）からなるグリッド上にアイテムを配置する
- Flexboxが1次元に特化しているのに対し、Gridは2次元レイアウトに適する

> 参照: [MDN - CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout)

## 基本の仕組み

- `display: grid` を指定した要素がグリッドコンテナになる
- その直接の子要素がグリッドアイテムになる

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  gap: 16px;
}
```

## グリッドの構成要素

```
列ライン:  1     2     3     4
           |     |     |     |
行ライン1 ─┌─────┬─────┬─────┐
           │cell │cell │cell │  ← 行トラック
行ライン2 ─├─────┼─────┼─────┤
           │cell │cell │cell │
行ライン3 ─└─────┴─────┴─────┘
             ↑ 列トラック
```

- **グリッドライン**: グリッドの区切り線。1から始まる番号が自動付与される
- **グリッドトラック**: 2本のラインの間の空間（列トラック / 行トラック）
- **グリッドセル**: 列トラックと行トラックが交差する最小単位
- **グリッドエリア**: 1つ以上のセルで構成される矩形領域

> 参照: [MDN - Basic concepts of grid layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Basic_concepts_of_grid_layout)

## トラックの定義

### grid-template-columns / grid-template-rows

```css
/* 固定幅の3列 */
.grid { grid-template-columns: 200px 400px 200px; }

/* fr単位で比率指定 */
.grid { grid-template-columns: 1fr 2fr 1fr; /* 1:2:1 */ }

/* 固定 + 可変の組み合わせ */
.grid { grid-template-columns: 250px 1fr; /* サイドバー固定、メイン可変 */ }
```

### fr 単位

- **fraction（分数）**の略
- 利用可能な空間を比率で分配する
- 固定幅やgapを差し引いた残りの空間に対して計算される

```css
.grid {
  grid-template-columns: 200px 1fr 1fr;
  width: 800px;
  gap: 16px;
  /* 1fr = (800 - 200 - 16*2) / 2 = 284px */
}
```

### repeat() — 繰り返し

```css
.grid { grid-template-columns: repeat(3, 1fr); }   /* 1fr 1fr 1fr */
.grid { grid-template-columns: repeat(4, 200px); }  /* 200px × 4 */
.grid { grid-template-columns: repeat(3, 1fr 2fr); } /* パターンの繰り返し */
```

### minmax() — 最小値と最大値

```css
.grid {
  grid-template-columns: minmax(200px, 300px) 1fr;
  grid-template-rows: minmax(100px, auto); /* 最小100px、コンテンツに合わせて伸びる */
}
```

### auto-fill と auto-fit — レスポンシブグリッド

```css
/* auto-fill: 空トラックも維持 */
.grid { grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); }

/* auto-fit: 空トラックを潰してアイテムを伸ばす */
.grid { grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); }
```

```
auto-fill（アイテム3つ、5列分のスペースがある場合）:
┌─────┬─────┬─────┬─────┬─────┐
│  A  │  B  │  C  │(空) │(空) │  ← 空トラックが残る
└─────┴─────┴─────┴─────┴─────┘

auto-fit（同じ条件）:
┌────────┬────────┬────────────┐
│   A    │   B    │     C      │  ← 空トラックが潰れて伸びる
└────────┴────────┴────────────┘
```

- メディアクエリなしでレスポンシブグリッドが作れる

### gap — トラック間のスペース

```css
.grid {
  gap: 16px;         /* 行間・列間ともに16px */
  gap: 16px 24px;    /* 行間16px、列間24px */
}
```

> 参照: [MDN - grid-template-columns](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-columns)

## グリッドラインを使ったアイテム配置

### ライン番号で配置

```css
.item {
  grid-column: 1 / 3;  /* 列ライン1から3まで（2列分） */
  grid-row: 1 / 2;     /* 行ライン1から2まで（1行分） */
}
```

```
列ライン: 1      2      3      4
行ライン1 ┌──────────────┬──────┐
          │   .item      │      │  grid-column: 1 / 3
行ライン2 ├──────┬───────┼──────┤
          │      │       │      │
行ライン3 └──────┴───────┴──────┘
```

### span を使った指定

```css
.item { grid-column: span 2; }       /* 2列分を占有 */
.item { grid-column: 2 / span 2; }   /* 列ライン2から2列分 */
.item { grid-row: 1 / span 3; }      /* 行ライン1から3行分 */
```

### 負の番号

```css
/* 最初のラインから最後のラインまで（全列を占有） */
.full-width { grid-column: 1 / -1; }
```

### 名前付きライン

```css
.grid {
  grid-template-columns:
    [sidebar-start] 250px
    [sidebar-end main-start] 1fr
    [main-end];
}

.sidebar { grid-column: sidebar-start / sidebar-end; }
.main    { grid-column: main-start / main-end; }
```

> 参照: [MDN - Line-based placement](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_layout_using_line-based_placement)

## グリッドエリアを使ったレイアウト

### grid-template-areas

- ASCIIアートのような直感的な記法でレイアウトを定義できる
- `.`（ドット）は空のセルを表す

```css
.page {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: 80px 1fr 60px;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  gap: 16px;
  min-height: 100vh;
}

header  { grid-area: header; }
nav     { grid-area: sidebar; }
main    { grid-area: main; }
footer  { grid-area: footer; }
```

```
┌──────────────────────────────┐
│           header              │
├──────────┬───────────────────┤
│ sidebar  │      main          │
├──────────┴───────────────────┤
│           footer              │
└──────────────────────────────┘
```

### エリアの制約

- エリアは**必ず矩形**でなければならない（L字型は不可）
- すべての行で列数を統一する必要がある

```css
/* 悪い例：L字型のエリア（無効） */
grid-template-areas:
  "a a b"
  "a c c";  /* a がL字型 */

/* 良い例：すべて矩形 */
grid-template-areas:
  "a a b"
  "c c b";
```

### レスポンシブ対応

```css
/* モバイル */
.page {
  grid-template-columns: 1fr;
  grid-template-areas:
    "header"
    "main"
    "sidebar"
    "footer";
}

/* デスクトップ */
@media (min-width: 768px) {
  .page {
    grid-template-columns: 250px 1fr;
    grid-template-areas:
      "header  header"
      "sidebar main"
      "footer  footer";
  }
}
```

> 参照: [MDN - Grid template areas](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_template_areas)

## 暗黙のグリッド

- 明示的なトラックを超えてアイテムが配置される場合、ブラウザが自動的にトラックを追加する
- `grid-auto-rows` / `grid-auto-columns` でサイズを制御

```css
/* 良い例：暗黙の行に最小サイズを保証 */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: minmax(150px, auto);
}

/* 悪い例：grid-auto-rows 未指定 → 暗黙の行がコンテンツ分のみ */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}
```

### grid-auto-flow — 自動配置の方向

| 値 | 動作 |
|---|---|
| `row`（デフォルト） | 行方向に順番に配置 |
| `column` | 列方向に配置 |
| `dense` | 隙間を埋めるように配置 |

> 参照: [MDN - Auto-placement in grid layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Auto-placement_in_grid_layout)

## 配置プロパティ

### セル内の配置

```css
.grid {
  justify-items: center; /* 各セル内で水平中央 */
  align-items: center;   /* 各セル内で垂直中央 */
  place-items: center;   /* 上記2つの一括指定 */
}
```

### グリッド全体の配置

```css
.grid {
  justify-content: center; /* グリッド全体を水平中央 */
  align-content: center;   /* グリッド全体を垂直中央 */
}
```

### アイテム個別の配置

```css
.special { justify-self: end; align-self: stretch; }
```

## よく使うパターン

### レスポンシブカードグリッド（メディアクエリ不要）

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}
```

### Holy Grailレイアウト

```css
.page {
  display: grid;
  grid-template: auto 1fr auto / 200px 1fr 200px;
  grid-template-areas:
    "header header  header"
    "nav    main    aside"
    "footer footer  footer";
  min-height: 100vh;
}
```

### 12列グリッドシステム

```css
.grid-12 {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 16px;
}
.col-4  { grid-column: span 4; }
.col-6  { grid-column: span 6; }
.col-12 { grid-column: span 12; }
```

## よくある間違い

### 1. grid-template-areas の列数が行ごとに異なる

```css
/* 悪い例 */
grid-template-areas:
  "header header"
  "sidebar main aside";  /* 3列 → 無効 */
```

### 2. アイテムの意図しない重なり

```css
.item-1 { grid-column: 1 / 3; grid-row: 1; }
.item-2 { grid-column: 2 / 4; grid-row: 1; }
/* → 列2で重なる */
```

### 3. fr と固定単位の混在時にgapを忘れる

```css
.grid {
  grid-template-columns: 200px 1fr 1fr;
  gap: 16px;
  /* 1fr = (コンテナ幅 - 200px - gap×2) / 2 */
  /* gap の分も差し引かれる */
}
```

## まとめ

- CSS Grid は行と列を同時に制御する2次元レイアウトモデル
- `fr` 単位で空間を比率分配、`repeat(auto-fit, minmax())` でレスポンシブ対応
- グリッドラインの番号・名前、または `grid-template-areas` でアイテムを配置
- `grid-template-areas` はASCIIアート風の直感的な記法（矩形のみ）
- 暗黙のグリッドは `grid-auto-rows` でサイズを制御する

## 参照元

- [MDN - CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout)
- [MDN - Basic concepts of grid layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Basic_concepts_of_grid_layout)
- [MDN - Grid template areas](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_template_areas)
- [CSS-Tricks - A Complete Guide to CSS Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
