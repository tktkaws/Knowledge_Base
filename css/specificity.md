# CSSの詳細度(Specificity)の計算方法

## 詳細度とは

- ブラウザが「どのCSS宣言を要素に適用するか」を決定するためのアルゴリズム
- 同じ要素に複数のルールが競合した場合、詳細度が高いセレクタのスタイルが優先される
- カスケードの中で「同じオリジン・同じレイヤー」のルール同士を比較する段階で使われる
- 詳細度はセレクタ単位で計算され、プロパティ単位ではない

> 参照: [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)

## 3カラムモデルによる計算

- 詳細度は **A-B-C** の3つの数値(カラム)で表現される
- 左のカラムほど重みが大きく、左から順に比較される
- 各カラムは独立しており、「クラスをいくつ積んでもIDには勝てない」

| カラム | 記法 | 対象のセレクタ |
|---|---|---|
| A (ID) | `1-0-0` | IDセレクタ (`#header`, `#main`) |
| B (Class) | `0-1-0` | クラスセレクタ (`.btn`)、属性セレクタ (`[type="text"]`)、擬似クラス (`:hover`, `:focus`, `:nth-child()`) |
| C (Type) | `0-0-1` | 型セレクタ (`div`, `p`, `h1`)、擬似要素 (`::before`, `::after`, `::placeholder`) |

### 詳細度に影響しないもの (0-0-0)

- 全称セレクタ (`*`)
- 結合子 (`>`, `+`, `~`, ` `(子孫), `||`)
- `:where()` 擬似クラス
- CSSネスティングの `&`

> 参照: [W3C Selectors Level 4 - Specificity](https://www.w3.org/TR/selectors-4/#specificity-rules)

## セレクタ別の詳細度一覧

```css
/* --- A-B-C の表記で示す --- */

*                          /* 0-0-0 */
p                          /* 0-0-1 */
p::before                  /* 0-0-2 */
.card                      /* 0-1-0 */
p.card                     /* 0-1-1 */
[type="text"]              /* 0-1-0 */
input:focus                /* 0-1-1 */
input[type="text"]:focus   /* 0-2-1 */
.card .title               /* 0-2-0 */
#header                    /* 1-0-0 */
#header .nav               /* 1-1-0 */
#header .nav li.active a   /* 1-2-3 */
```

### 計算の手順

1. セレクタ内のIDセレクタ(`#xxx`)の個数を数える → カラムA
2. クラスセレクタ(`.xxx`)・属性セレクタ(`[xxx]`)・擬似クラス(`:xxx`)の個数を数える → カラムB
3. 型セレクタ(`div`など)・擬似要素(`::xxx`)の個数を数える → カラムC

```css
/* 例: #header .nav li.active a の計算 */
/* #header     → ID × 1       → A = 1 */
/* .nav        → クラス × 1    → B に +1 */
/* .active     → クラス × 1    → B に +1 → B = 2 */
/* li          → 型 × 1        → C に +1 */
/* a           → 型 × 1        → C に +1 → C = 2 */
/* (liも型)    →                → C に +1 → C = 3 */
/* 結果: 1-2-3 */
```

> 参照: [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)

## 3カラム比較のルール

- 左のカラムから順に比較し、大きい方が勝つ
- あるカラムで差がつけば、それより右のカラムは無関係

```css
/* 悪い例：クラスを大量に積んでIDに対抗しようとする */
.a .b .c .d .e .f .g .h .i .j .k {
  color: red; /* 0-11-0 */
}

/* IDセレクタ1つで上記に勝つ */
#title {
  color: blue; /* 1-0-0 → こちらが適用される */
}
```

- カラムAで `1 > 0` なので、カラムBが11であっても `#title` が勝つ
- 「クラス100個積んでもID1つに勝てない」

> 参照: [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)

## :is() / :not() / :has() の詳細度

- これらの擬似クラス自体は詳細度に寄与しない (0-0-0)
- ただし**引数リストの中で最も高い詳細度**がそのセレクタの詳細度として加算される

```css
/* :is() の例 */
:is(.card, #special) {
  color: red;
  /* :is自体 → 0-0-0 */
  /* 引数の最大: #special → 1-0-0 */
  /* 合計: 1-0-0 */
}

/* :not() の例 */
p:not(#fakeId) {
  color: blue;
  /* p → 0-0-1 */
  /* :not自体 → 0-0-0, 引数 #fakeId → 1-0-0 */
  /* 合計: 1-0-1 */
}

/* :has() の例 */
h1:has(+ h2, > #fakeId) {
  color: green;
  /* h1 → 0-0-1 */
  /* :has自体 → 0-0-0, 引数の最大: #fakeId → 1-0-0 */
  /* 合計: 1-0-1 */
}
```

### :where() は例外

- `:where()` は引数の詳細度を**完全に無視**する
- `:where()` 全体の詳細度は常に `0-0-0`
- ライブラリやリセットCSSなど、上書きされやすくしたいスタイルに便利

```css
/* :where() は詳細度ゼロ */
:where(#theme) a {
  color: red; /* 0-0-1（#themeの詳細度は無視される） */
}

/* 通常のセレクタで簡単に上書きできる */
footer a {
  color: blue; /* 0-0-2 → こちらが勝つ */
}
```

```css
/* 悪い例：ライブラリCSSが高い詳細度を持つと上書きが困難 */
#defaultTheme .container a {
  color: red; /* 1-1-1 */
}

/* 良い例：:where()で包んで詳細度をゼロにする */
:where(#defaultTheme .container) a {
  color: red; /* 0-0-1 → 利用者が簡単に上書き可能 */
}
```

> 参照: [MDN - :where()](https://developer.mozilla.org/en-US/docs/Web/CSS/:where)

## CSSネスティングと詳細度

- CSSネスティングで書いたルールの詳細度は `:is()` と同じルールに従う
- 親セレクタの中に詳細度の高いものが含まれると、ネストされた子にもその詳細度が影響する

```css
/* ネスティングの例 */
p, #fakeId {
  span {
    color: red;
    /* :is(p, #fakeId) span と同等 */
    /* :is()の引数の最大: #fakeId → 1-0-0 */
    /* span → 0-0-1 */
    /* 合計: 1-0-1 */
  }
}
```

- 注意: `p` に対するスタイルのつもりでも、`#fakeId` の詳細度が適用される

> 参照: [MDN - CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)

## インラインスタイルの詳細度

- `style` 属性で直接書かれたスタイルはすべてのセレクタより高い詳細度を持つ
- `!important` でのみ上書き可能

```html
<!-- インラインスタイルが最優先 -->
<p id="title" class="highlight" style="color: red;">
  このテキストは赤色になる
</p>
```

```css
/* これらはすべてインラインスタイルに負ける */
#title { color: blue; }            /* 1-0-0 */
#title.highlight { color: green; } /* 1-1-0 */
```

```html
<!-- 悪い例：インラインスタイルの多用 -->
<div style="margin: 10px; padding: 5px; color: red; font-size: 14px;">
  コンテンツ
</div>

<!-- 良い例：クラスで管理する -->
<div class="card-body">
  コンテンツ
</div>
```

- インラインスタイルを多用すると、CSSからの上書きが `!important` なしでは不可能になる
- メンテナンス性が著しく低下するため、原則として避けるべき

> 参照: [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)

## !important とカスケード

### !important の基本

- `!important` は詳細度の計算とは別の仕組み
- 通常のカスケード順序を逆転させる特殊な宣言
- 同じオリジン内で `!important` がついた宣言は、`!important` なしの宣言に常に勝つ

```css
p {
  color: red !important;
}

#content p.highlight {
  color: blue; /* 1-1-1 だが !important に負ける */
}
```

### !important 同士の競合

- `!important` 同士が競合する場合は、通常の詳細度ルールで比較される

```css
p {
  color: red !important; /* 0-0-1 */
}

.text {
  color: blue !important; /* 0-1-0 → こちらが勝つ */
}
```

### !important を使うべきでない理由

```css
/* 悪い例：!important の連鎖 */
.button {
  background: blue !important;
}

/* 上書きしたい → さらに高い詳細度 + !important が必要 */
#sidebar .button {
  background: green !important;
}

/* さらに上書き → 詳細度の軍拡競争が始まる */
body #sidebar .button.special {
  background: red !important;
}
```

```css
/* 良い例：詳細度を適切に管理する */
.button {
  background: blue;
}

.button--primary {
  background: green;
}

.button--danger {
  background: red;
}
```

### !important の正当な使用場面

- ユーティリティクラス（例: `.hidden { display: none !important; }`）
- ユーザー補助スタイル（例: フォントサイズ強制拡大）
- サードパーティCSSの緊急上書き（恒久対策までの一時的な措置）

> 参照: [MDN - !important](https://developer.mozilla.org/en-US/docs/Web/CSS/important)

## よくある間違いと落とし穴

### 1. 詳細度を10進数で計算してしまう

```css
/* 間違った理解: 「IDは100点、クラスは10点、要素は1点」 */
/* → 11個のクラス(110点)がID(100点)に勝つと誤解する */

/* 正しい理解: 各カラムは独立している */
/* 0-11-0 は 1-0-0 に絶対に勝てない */
```

### 2. 継承されたスタイルと直接適用されたスタイルを混同

```css
#parent {
  color: red; /* 1-0-0 で指定 */
}

p {
  color: blue; /* 0-0-1 だが直接適用 → こちらが勝つ */
}
```

```html
<div id="parent">
  <p>このテキストは青色（blueが適用される）</p>
</div>
```

- 継承されたスタイルは、直接適用されたスタイルに常に負ける
- 親の詳細度がどれだけ高くても、子要素に直接当てた低い詳細度のルールが勝つ

### 3. DOM上の距離が詳細度に影響すると思い込む

```css
body .text { color: red; }   /* 0-1-1 */
main .text { color: blue; }  /* 0-1-1 */
```

- 詳細度は同じ(`0-1-1`)
- DOMツリー上でどちらが要素に「近い」かは無関係
- 同じ詳細度の場合、**ソースコード上で後に書かれた方**が勝つ

### 4. 属性セレクタ [id="x"] とIDセレクタ #x の詳細度の違い

```css
[id="header"] { color: red; }   /* 0-1-0（属性セレクタ） */
#header { color: blue; }        /* 1-0-0（IDセレクタ） */
/* → #header が勝つ */
```

- 同じ要素を指していても、書き方によって詳細度が変わる

> 参照: [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)

## 詳細度を管理するベストプラクティス

### 1. 詳細度を低く保つ

```css
/* 悪い例：不必要に詳細度を上げている */
body div#content ul.nav li a.link {
  color: blue; /* 1-2-4 */
}

/* 良い例：必要最小限のセレクタ */
.nav-link {
  color: blue; /* 0-1-0 */
}
```

### 2. IDセレクタをスタイリングに使わない

```css
/* 悪い例 */
#header { background: navy; }  /* 1-0-0 → 上書きが困難 */

/* 良い例 */
.header { background: navy; }  /* 0-1-0 → 上書きが容易 */

/* IDを使いたい場合は属性セレクタで詳細度を抑える */
[id="header"] { background: navy; } /* 0-1-0 */
```

### 3. BEM命名規則で詳細度を均一に保つ

```css
/* BEM: すべてのセレクタが 0-1-0 で揃う */
.card { }
.card__title { }
.card__body { }
.card--featured { }
.card__title--large { }
```

### 4. :where() でリセット・デフォルトスタイルの詳細度をゼロにする

```css
:where(h1, h2, h3, h4, h5, h6) {
  margin: 0;
  font-weight: normal;
  /* 詳細度: 0-0-0 → どんなスタイルでも上書き可能 */
}
```

### 5. @layer でサードパーティCSSの詳細度問題を解消する

```css
@import "bootstrap.css" layer(framework);

/* レイヤー外のスタイルはレイヤー内に常に勝つ */
.my-button {
  background: coral; /* frameworkレイヤーの高詳細度ルールにも勝つ */
}
```

> 参照: [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity), [MDN - @layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)

## まとめ

- 詳細度は **A(ID)-B(Class)-C(Type)** の3カラムで計算される
- カラムは独立しており、左のカラムが常に優先される（クラス100個 < ID1個）
- `:is()` / `:not()` / `:has()` は引数の最大詳細度を引き継ぐ
- `:where()` は詳細度を完全にゼロにする特別な擬似クラス
- インラインスタイルはすべてのセレクタより高い詳細度を持つ
- `!important` は詳細度とは別の仕組みで、カスケードの順序を逆転させる
- **詳細度は低く保ち、BEM命名・`@layer`・`:where()` で管理するのがベストプラクティス**

## 参照元

- [MDN - Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)
- [MDN - Cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade)
- [MDN - !important](https://developer.mozilla.org/en-US/docs/Web/CSS/important)
- [MDN - :where()](https://developer.mozilla.org/en-US/docs/Web/CSS/:where)
- [W3C Selectors Level 4](https://www.w3.org/TR/selectors-4/#specificity-rules)
- [W3C CSS Cascading and Inheritance Level 5](https://www.w3.org/TR/css-cascade-5/)
