# Flexboxの基本 — 主軸・交差軸の考え方

## Flexboxとは

- 1次元のレイアウトモデル（横方向 or 縦方向のどちらか一方に要素を並べる）
- コンテナ内のアイテムの配置・整列・間隔を柔軟に制御できる
- コンテンツのサイズが不明・動的な場合でも適切にレイアウトできる
- CSS Grid が2次元（行と列）を同時に扱うのに対し、Flexbox は1次元に特化

> 参照: [MDN - Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox)

## 基本の仕組み — コンテナとアイテム

- Flexbox は**コンテナ（親）**と**アイテム（子）**の2つの役割で成り立つ
- `display: flex` を指定した要素がFlexコンテナになる
- その直接の子要素が自動的にFlexアイテムになる
- 孫要素はFlexアイテムにならない

```css
/* 良い例：display: flex でコンテナを作成 */
.container {
  display: flex;
}
```

```html
<div class="container">
  <div>アイテム1</div>  <!-- Flexアイテム -->
  <div>アイテム2</div>  <!-- Flexアイテム -->
  <div>アイテム3</div>  <!-- Flexアイテム -->
</div>
```

### display: flex vs display: inline-flex

| プロパティ | コンテナ自体の振る舞い |
|---|---|
| `display: flex` | ブロックレベル要素として振る舞う（幅が親いっぱいに広がる） |
| `display: inline-flex` | インラインレベル要素として振る舞う（コンテンツ幅に収まる） |

- どちらも内部のFlexレイアウトは同じ
- 違いはコンテナ自体が外部に対してどう振る舞うか

> 参照: [MDN - display](https://developer.mozilla.org/en-US/docs/Web/CSS/display)

## 主軸と交差軸

Flexbox のレイアウトを理解する上で最も重要な概念。

### 主軸（Main Axis）

- アイテムが並ぶ方向の軸
- `flex-direction` で制御する
- デフォルトは横方向（左→右）

### 交差軸（Cross Axis）

- 主軸に対して垂直な方向の軸
- 主軸が横方向なら交差軸は縦方向、主軸が縦方向なら交差軸は横方向

```
flex-direction: row（デフォルト）の場合:

主軸（Main Axis）→→→→→→→→→→→→
↓ ┌──────┐ ┌──────┐ ┌──────┐
交 │Item 1│ │Item 2│ │Item 3│
差 └──────┘ └──────┘ └──────┘
軸

flex-direction: column の場合:

交差軸（Cross Axis）→→→→→→→→→→
  ┌──────────────────────┐
主 │       Item 1         │
軸 ├──────────────────────┤
↓ │       Item 2         │
↓ ├──────────────────────┤
↓ │       Item 3         │
  └──────────────────────┘
```

> 参照: [MDN - Basic concepts of flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox)

## コンテナに指定するプロパティ

### flex-direction — 主軸の方向

| 値 | 方向 |
|---|---|
| `row`（デフォルト） | 左→右（横並び） |
| `row-reverse` | 右→左 |
| `column` | 上→下（縦並び） |
| `column-reverse` | 下→上 |

```css
/* 横並び（デフォルト） */
.row { display: flex; flex-direction: row; }

/* 縦並び */
.column { display: flex; flex-direction: column; }
```

### flex-wrap — 折り返し

| 値 | 動作 |
|---|---|
| `nowrap`（デフォルト） | 折り返さない（アイテムが縮む） |
| `wrap` | 折り返す（次の行へ） |
| `wrap-reverse` | 逆方向に折り返す |

```css
/* 悪い例：折り返さないため、アイテムがコンテナからはみ出す可能性 */
.container {
  display: flex;
  /* flex-wrap: nowrap がデフォルト */
}

/* 良い例：折り返しを許可 */
.container {
  display: flex;
  flex-wrap: wrap;
}
```

### flex-flow — flex-direction と flex-wrap のショートハンド

```css
/* flex-flow: <flex-direction> <flex-wrap> */
.container {
  display: flex;
  flex-flow: row wrap;
}
```

### justify-content — 主軸方向の配置

主軸方向にアイテムをどう並べるかを制御する。

| 値 | 配置 |
|---|---|
| `flex-start`（デフォルト） | 主軸の始点に寄せる |
| `flex-end` | 主軸の終点に寄せる |
| `center` | 中央に寄せる |
| `space-between` | 最初と最後のアイテムを端に置き、残りを均等配置 |
| `space-around` | 各アイテムの左右に均等な余白 |
| `space-evenly` | すべての間隔を完全に均等 |

```
justify-content の各値のイメージ（flex-direction: row の場合）:

flex-start:      |■ ■ ■          |
flex-end:        |          ■ ■ ■|
center:          |     ■ ■ ■     |
space-between:   |■      ■      ■|
space-around:    |  ■    ■    ■  |
space-evenly:    |   ■   ■   ■   |
```

```css
/* ナビゲーションの例：ロゴを左、メニューを右に配置 */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

### align-items — 交差軸方向の配置（1行分）

交差軸方向に各アイテムをどう揃えるかを制御する。

| 値 | 配置 |
|---|---|
| `stretch`（デフォルト） | コンテナの高さいっぱいに伸ばす |
| `flex-start` | 交差軸の始点に寄せる |
| `flex-end` | 交差軸の終点に寄せる |
| `center` | 交差軸の中央に揃える |
| `baseline` | テキストのベースラインに揃える |

```css
/* 上下中央揃え（最頻出パターン） */
.container {
  display: flex;
  align-items: center;
}

/* 上下左右中央揃え */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### align-content — 交差軸方向の配置（複数行）

- `flex-wrap: wrap` で複数行になった場合に、行全体の配置を制御する
- 1行しかない場合は効果なし

| 値 | 配置 |
|---|---|
| `stretch`（デフォルト） | 行を均等に伸ばす |
| `flex-start` | 交差軸の始点に寄せる |
| `flex-end` | 交差軸の終点に寄せる |
| `center` | 中央に寄せる |
| `space-between` | 最初と最後の行を端に置く |
| `space-around` | 各行の上下に均等な余白 |

### gap — アイテム間のスペース

- アイテム間の余白を指定するプロパティ
- `margin` を使うより簡潔で直感的
- `row-gap` と `column-gap` で個別指定も可能

```css
/* 良い例：gap でアイテム間隔を指定 */
.container {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;        /* 行間・列間ともに16px */
}

.container {
  display: flex;
  gap: 16px 24px;   /* 行間16px、列間24px */
}
```

```css
/* 悪い例：margin でアイテム間隔を制御（最後の要素に余分なmarginがつく） */
.item {
  margin-right: 16px;
  margin-bottom: 16px;
}
.item:last-child {
  margin-right: 0;  /* 調整が必要 */
}
```

> 参照: [MDN - gap](https://developer.mozilla.org/en-US/docs/Web/CSS/gap)

## アイテムに指定するプロパティ

### flex-grow — 余白の分配比率

- コンテナに余白がある場合に、アイテムがどれだけ伸びるかを決定する
- デフォルトは `0`（伸びない）
- 値は比率で解釈される

```css
/* 全アイテムが均等に伸びる */
.item { flex-grow: 1; }

/* item-2 だけ他の2倍伸びる */
.item-1 { flex-grow: 1; }
.item-2 { flex-grow: 2; }
.item-3 { flex-grow: 1; }
```

```
コンテナ幅: 600px、アイテム基本幅: 各100px、余白: 300px

flex-grow: 1, 2, 1 の場合:
item-1: 100 + 300*(1/4) = 175px
item-2: 100 + 300*(2/4) = 250px
item-3: 100 + 300*(1/4) = 175px
```

### flex-shrink — 縮小の比率

- コンテナに収まらない場合に、アイテムがどれだけ縮むかを決定する
- デフォルトは `1`（均等に縮む）
- `0` にすると縮まない

```css
/* サイドバーは縮まない、メインコンテンツだけが縮む */
.sidebar { flex-shrink: 0; width: 250px; }
.main { flex-shrink: 1; }
```

### flex-basis — アイテムの基本サイズ

- Flex計算の前にアイテムの初期サイズを決定する
- `width` / `height` と似ているが、主軸方向のサイズを指定する
- デフォルトは `auto`（コンテンツのサイズ or width/height の値）

```css
/* flex-basis は主軸方向のサイズ */
.item {
  flex-basis: 200px; /* 主軸方向の初期サイズが200px */
}
```

| `flex-basis` と `width` の違い | |
|---|---|
| `flex-basis` | 主軸方向のサイズ。`flex-direction` によって幅にも高さにもなる |
| `width` | 常に幅を指定 |

- 両方が指定された場合、`flex-basis` が優先される（`flex-basis: auto` の場合を除く）

### flex ショートハンド（推奨）

```css
/* flex: <flex-grow> <flex-shrink> <flex-basis> */

/* 均等に伸縮、基本サイズ0 */
.item { flex: 1; }
/* → flex: 1 1 0% と同等 */

/* 伸びるが縮まない、基本サイズ200px */
.item { flex: 1 0 200px; }

/* 伸びない、縮まない、固定サイズ */
.item { flex: 0 0 300px; }
/* → flex: none と width: 300px も同等 */
```

#### よく使う flex の値

| 値 | 展開 | 用途 |
|---|---|---|
| `flex: 1` | `1 1 0%` | 均等に伸縮。余白を均等に分配 |
| `flex: auto` | `1 1 auto` | コンテンツサイズを基本に伸縮 |
| `flex: none` | `0 0 auto` | 固定サイズ。伸縮しない |
| `flex: 0` | `0 1 0%` | 伸びないが縮む |

```css
/* 良い例：flex ショートハンドを使用 */
.sidebar { flex: 0 0 250px; }
.main { flex: 1; }

/* 悪い例：個別プロパティをバラバラに指定（冗長） */
.sidebar {
  flex-grow: 0;
  flex-shrink: 0;
  flex-basis: 250px;
}
.main {
  flex-grow: 1;
  flex-shrink: 1;
  flex-basis: 0%;
}
```

### align-self — アイテム個別の交差軸配置

- コンテナの `align-items` を個別のアイテムで上書きする
- 特定のアイテムだけ位置を変えたい場合に使用

```css
.container {
  display: flex;
  align-items: flex-start; /* 全体は上揃え */
}

.special-item {
  align-self: flex-end; /* このアイテムだけ下揃え */
}
```

### order — 表示順の変更

- HTML上の順番を変えずに、視覚的な表示順を変更する
- デフォルトは `0`。値が小さいほど前に表示される
- アクセシビリティ上の注意：スクリーンリーダーはDOM順で読み上げるため、`order` で変えた順番は反映されない

```css
/* 視覚的に3番目のアイテムを最初に表示 */
.item-3 { order: -1; }
```

> 参照: [MDN - Ordering flex items](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Ordering_flex_items)

## よく使うレイアウトパターン

### 1. 上下左右中央揃え

```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
```

### 2. ナビバー（ロゴ左・メニュー右）

```css
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 24px;
}
```

### 3. サイドバー + メインコンテンツ

```css
.layout {
  display: flex;
}

.sidebar {
  flex: 0 0 250px; /* 固定幅 */
}

.main {
  flex: 1; /* 残りの幅を埋める */
  min-width: 0; /* テキストのオーバーフロー防止 */
}
```

### 4. カードの均等配置（折り返しあり）

```css
.card-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

.card {
  flex: 1 1 300px; /* 最小300px、余白があれば伸びる */
}
```

### 5. フッターをページ下部に固定（Sticky Footer）

```css
body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

main {
  flex: 1; /* メインコンテンツが伸びてフッターを押し下げる */
}

footer {
  flex-shrink: 0;
}
```

> 参照: [CSS-Tricks - A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

## よくある間違いと注意点

### 1. min-width: 0 の未指定によるオーバーフロー

- Flexアイテムのデフォルトの `min-width` は `auto`（コンテンツの最小サイズ）
- 長いテキストやURLがコンテナをはみ出す原因になる

```css
/* 悪い例：長いテキストがはみ出す */
.container { display: flex; }
.item { flex: 1; }
/* → 長い単語や URL がコンテナ幅を超えてはみ出す */

/* 良い例：min-width: 0 を指定 */
.item {
  flex: 1;
  min-width: 0; /* コンテンツに基づく最小幅を無効化 */
  overflow: hidden;
  text-overflow: ellipsis;
}
```

### 2. flex-basis: 0 と flex-basis: auto の違い

```css
/* flex: 1 → flex: 1 1 0% */
/* アイテムのコンテンツサイズを無視し、余白を均等に分配 */
.item { flex: 1; }

/* flex: auto → flex: 1 1 auto */
/* アイテムのコンテンツサイズを考慮した上で、余白を分配 */
.item { flex: auto; }
```

```
コンテナ幅: 600px
item-1のコンテンツ: "短い"(50px)
item-2のコンテンツ: "これは長いテキスト"(200px)

flex: 1 (basis: 0%) の場合:
  item-1: 300px, item-2: 300px（均等）

flex: auto (basis: auto) の場合:
  item-1: 225px, item-2: 375px（コンテンツサイズの差が反映される）
```

### 3. margin: auto の特殊な挙動

- Flexアイテムに `margin: auto` を使うと、余白をその方向に吸収する
- `justify-content` や `align-items` の代替として使える

```css
/* ナビバーで最後のアイテムだけ右寄せ */
.navbar { display: flex; }
.navbar .login { margin-left: auto; }
```

```
|ロゴ  ホーム  概要          ログイン|
                    ←margin-left: auto→
```

### 4. flex-wrap: wrap 時のアイテム幅の制御

```css
/* 悪い例：flex: 1 だけだと最後の行のアイテムが伸びすぎる */
.grid { display: flex; flex-wrap: wrap; gap: 16px; }
.item { flex: 1; }
/* 最後の行にアイテムが1つしかない場合、幅100%に伸びる */

/* 良い例：flex-basis で最小幅を指定 */
.item { flex: 1 1 calc(33.333% - 16px); max-width: calc(33.333% - 16px); }

/* より良い例：この場合は CSS Grid を使うべき */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 16px;
}
```

> 参照: [MDN - Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox)

## まとめ

- Flexbox は1次元レイアウトモデル。主軸と交差軸の2つの軸で考える
- `flex-direction` で主軸の方向を決め、`justify-content` で主軸方向、`align-items` で交差軸方向の配置を制御する
- アイテムには `flex` ショートハンドで伸縮と基本サイズを指定するのが推奨
- `gap` でアイテム間のスペースを制御する（`margin` より推奨）
- 上下左右中央揃え、サイドバーレイアウト、Sticky Footer など頻出パターンを覚える
- `min-width: 0` の指定漏れによるオーバーフローに注意
- 等間隔のグリッドレイアウトには CSS Grid のほうが適している場合がある

## 参照元

- [MDN - Basic concepts of flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox)
- [MDN - Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox)
- [CSS-Tricks - A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [MDN - gap](https://developer.mozilla.org/en-US/docs/Web/CSS/gap)
