# カスケードレイヤー(@layer)の仕組みと使い方

## カスケードレイヤーとは

- W3CのCSS Cascading and Inheritance Level 5 仕様で定義された機能
- `@layer` アットルールを使い、CSSカスケードに新しい優先順位の「レイヤー」を導入する仕組み
- **詳細度よりも上位のレベル**で優先順位を制御できる
- 2022年3月に全主要ブラウザで実装完了

> 参照: [MDN - @layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)

## なぜ導入されたか

- 従来のCSSでは優先順位が主に**詳細度**と**ソース順**で決まっていた
- 大規模プロジェクトで「詳細度戦争（specificity war）」が深刻化していた
  - サードパーティCSSの上書きに高い詳細度が必要
  - CSSリセットとコンポーネントスタイルの競合
  - `!important` の乱用
- カスケードレイヤーは詳細度の問題を根本から解決する

### カスケードの優先順位（上位ほど強い）

1. Origin & Importance（ユーザーエージェント / ユーザー / 作者スタイル、`!important`の有無）
2. **Cascade Layers（ここに新しく追加された）**
3. Specificity（詳細度）
4. Source Order（ソース順）

> 参照: [W3C CSS Cascading and Inheritance Level 5](https://www.w3.org/TR/css-cascade-5/#layering)

## @layer の構文

### 方法1: レイヤー順序の事前宣言

```css
/* レイヤーの順序だけを先に宣言（中身は空） */
@layer reset, base, components, utilities;
```

- 最も重要なパターン
- ファイルの先頭でレイヤーの優先順位を明示的に定義する

### 方法2: ブロック付きレイヤー宣言

```css
@layer base {
  body {
    margin: 0;
    font-family: system-ui, sans-serif;
    line-height: 1.5;
  }
}

@layer components {
  .card {
    padding: 1rem;
    border: 1px solid #ccc;
    border-radius: 8px;
  }
}
```

### 方法3: @import でのレイヤー指定

```css
/* 外部CSSファイルをレイヤーに割り当てる */
@import url("reset.css") layer(reset);
@import url("bootstrap.css") layer(third-party);
@import url("components.css") layer(components);
```

### 方法4: 匿名レイヤー

```css
/* 名前なしのレイヤー（後から追加できない） */
@layer {
  .anonymous-style {
    color: red;
  }
}
```

### 方法5: ネスト（入れ子）レイヤー

```css
@layer framework {
  @layer base {
    /* framework.base として参照される */
    p { color: black; }
  }
  @layer components {
    /* framework.components として参照される */
    .btn { padding: 0.5rem 1rem; }
  }
}

/* ドット記法で直接宣言も可能 */
@layer framework.theme {
  :root {
    --primary: blue;
  }
}
```

### 同名レイヤーへのスタイル追加

```css
/* 同じレイヤー名を複数回使うと、スタイルが追加される */
@layer base {
  body { margin: 0; }
}

@layer base {
  body { font-family: sans-serif; }
}
/* 結果: base レイヤーに body { margin: 0; font-family: sans-serif; } が含まれる */
```

> 参照: [MDN - @layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)

## レイヤーの順序とカスケードへの影響

### 基本ルール

- **後に宣言されたレイヤーが、先に宣言されたレイヤーより優先される**
- レイヤーの順序は**最初に出現した順**で確定する
- **レイヤーに属さないスタイル（unlayered styles）は、すべてのレイヤーより優先される**

### 良い例 vs 悪い例

```css
/* 良い例: レイヤー順序を明示的に宣言 */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; box-sizing: border-box; }
}

@layer base {
  a { color: navy; }
}

@layer components {
  .nav a { color: white; }  /* 詳細度が低くても base の a より優先 */
}

@layer utilities {
  .text-red { color: red; }  /* 最も優先度が高いレイヤー */
}
```

```css
/* 悪い例: レイヤー順序を宣言せず、暗黙の順序に依存 */
@layer components {
  .nav a { color: white; }
}

@layer base {
  a { color: navy; }  /* components より後に宣言 → base が優先されてしまう */
}

/* 意図: components が base より優先されてほしいが、
   宣言順により base が後 = 高優先になる */
```

### レイヤー外スタイルの優先度

```css
@layer base {
  .title { color: blue; }  /* レイヤー内: 優先度が低い */
}

/* レイヤーに属さないスタイル: すべてのレイヤーより優先される */
.title { color: red; }  /* こちらが勝つ */
```

- 既存のレイヤーを使わないCSSコードとの後方互換性を保つための設計

> 参照: [MDN - Cascade layers](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_layers)

## 詳細度との関係

- カスケードレイヤーは詳細度より**上位**の判定基準
- レイヤー間では詳細度は無関係

```css
@layer base, components;

@layer base {
  #main .content p.intro { /* 詳細度: 1-2-1 — 非常に高い */
    color: blue;
  }
}

@layer components {
  p { /* 詳細度: 0-0-1 — 非常に低い */
    color: red;
  }
}

/* 結果: p.intro は赤になる
   → components レイヤーが base より後 = 優先されるため、
     詳細度に関係なく components が勝つ */
```

> 参照: [MDN - @layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)

## !important とレイヤーの優先度逆転

- `!important` が付いたスタイルでは、**レイヤーの優先順位が反転する**

通常のスタイルの優先順位（後のレイヤーが優先）:

```
reset < base < components < utilities < (unlayered)
```

`!important` が付いたスタイルの優先順位（逆順になる）:

```
(unlayered) !important < utilities !important < components !important < base !important < reset !important
```

```css
@layer reset, base, components;

@layer reset {
  * { box-sizing: border-box !important; }
  /* reset は通常では最低優先度だが、
     !important では最も高い優先度になる
     → リセットスタイルが確実に適用される */
}

@layer components {
  .card { box-sizing: content-box !important; }
  /* components は通常では高優先度だが、
     !important では reset より低い
     → reset の box-sizing を上書きできない */
}
```

- この逆転は意図的な設計
- リセットCSSや基盤レイヤーが `!important` で「絶対に変えない」と指定した場合、上位レイヤーから上書きされないことを保証する

> 参照: [MDN - !important](https://developer.mozilla.org/en-US/docs/Web/CSS/important), [W3C CSS Cascading and Inheritance Level 5](https://www.w3.org/TR/css-cascade-5/)

## 実践的なユースケース

### 1. サードパーティCSS管理

```css
/* 良い例: サードパーティCSSをレイヤーで隔離 */
@layer third-party, custom;

@import url("bootstrap.css") layer(third-party);

@layer custom {
  /* Bootstrap の詳細度がどれだけ高くても、
     custom レイヤーのスタイルが優先される */
  .navbar {
    background-color: navy;
  }
}
```

```css
/* 悪い例: サードパーティCSSと詳細度で戦う */
@import url("bootstrap.css");

body .navbar.navbar {
  background-color: navy !important;  /* 詳細度戦争 */
}
```

### 2. デザインシステムの構造化

```css
@layer reset, tokens, base, layout, components, utilities;

@layer reset {
  *, *::before, *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
}

@layer tokens {
  :root {
    --color-primary: #2563eb;
    --color-secondary: #64748b;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --radius-md: 0.5rem;
  }
}

@layer base {
  body {
    font-family: system-ui, sans-serif;
    line-height: 1.6;
    color: var(--color-secondary);
  }
  a { color: var(--color-primary); }
}

@layer components {
  .btn {
    padding: var(--spacing-sm) var(--spacing-md);
    border-radius: var(--radius-md);
    background: var(--color-primary);
    color: white;
    border: none;
    cursor: pointer;
  }
}

@layer utilities {
  .mt-4 { margin-top: 1rem; }
  .text-center { text-align: center; }
  .hidden { display: none; }
}
```

### 3. CSSリセットの保護

```css
@layer reset, base, theme;

@layer reset {
  *, *::before, *::after {
    box-sizing: border-box !important;
    /* reset は !important で最高優先度。
       上位レイヤーからも上書きされない */
  }
  body { margin: 0; }
}
```

### 4. レガシーCSSとの共存

```css
@layer legacy, modern;

/* 既存のレガシーCSSをレイヤーに閉じ込める */
@import url("legacy-styles.css") layer(legacy);

@layer modern {
  /* レガシーCSSの詳細度を気にせずに新しいスタイルを書ける */
  .new-component {
    display: grid;
    gap: 1rem;
  }
}
```

### 5. Tailwind CSS との統合

```css
@layer tailwind-base, tailwind-components, my-components, tailwind-utilities;

@import "tailwindcss/base" layer(tailwind-base);
@import "tailwindcss/components" layer(tailwind-components);
@import "tailwindcss/utilities" layer(tailwind-utilities);

@layer my-components {
  /* tailwind-components より優先、tailwind-utilities より低い */
  .custom-card {
    padding: 1.5rem;
    border: 1px solid #e5e7eb;
    border-radius: 0.75rem;
  }
}
```

> 参照: [Chrome Developers - Cascade layers](https://developer.chrome.com/blog/cascade-layers/), [CSS-Tricks - CSS Cascade Layers](https://css-tricks.com/css-cascade-layers/)

## よくある間違いと注意点

### 注意1: unlayered スタイルの優先度

```css
/* 間違いやすいパターン */
@layer utilities {
  .text-red { color: red; }
}

/* レイヤーに属さないスタイル → utilities レイヤーより優先 */
p { color: black; }

/* <p class="text-red"> は黒になる（期待通りに赤にならない） */
```

```css
/* 正しい対処: すべてのスタイルをレイヤーに入れる */
@layer base, utilities;

@layer base {
  p { color: black; }
}

@layer utilities {
  .text-red { color: red; }  /* base より優先 → 赤になる */
}
```

### 注意2: @layer の宣言位置

```css
/* 間違い: @import の後に @layer 宣言を置く */
@import url("lib.css") layer(lib);

@layer custom, lib;  /* lib はすでに最初のレイヤーとして確定済み */
/* → この宣言は lib レイヤーの順序を変えられない */
```

```css
/* 正しい: @layer 宣言を最初に置く */
@layer lib, custom;  /* 先に順序を宣言 */
@import url("lib.css") layer(lib);

@layer custom {
  /* lib より優先される */
}
```

### 注意3: @layer内の詳細度は通常通り機能する

```css
@layer components {
  /* レイヤー内でのルール同士は通常の詳細度で比較される */
  .btn { color: white; }       /* 0-1-0 */
  #special-btn { color: red; } /* 1-0-0 → レイヤー内ではこちらが優先 */
}
```

- レイヤーが「詳細度を無効にする」わけではない
- レイヤー**間**では詳細度が無関係になるだけ
- レイヤー**内**では通常通り詳細度で比較される

> 参照: [MDN - @layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)

## ブラウザサポート

| ブラウザ | サポート開始バージョン | リリース時期 |
|---|---|---|
| Chrome | 99 | 2022年3月 |
| Edge | 99 | 2022年3月 |
| Firefox | 97 | 2022年2月 |
| Safari | 15.4 | 2022年3月 |

- 2022年3月時点で全主要ブラウザが対応済み
- IE11は非対応（2022年6月にサポート終了済み）
- プロダクション利用に十分な段階

> 参照: [Can I Use - CSS Cascade Layers](https://caniuse.com/css-cascade-layers)

## まとめ

- カスケードレイヤーは詳細度よりも上位のカスケード判定基準
- `@layer` でスタイルをグループ化し、グループ間の優先順位を明示的に制御できる
- 後に宣言されたレイヤーが優先。レイヤー外のスタイルは全レイヤーより強い
- `!important` が付くとレイヤーの優先順位が**逆転**する
- サードパーティCSS管理、デザインシステム構築、レガシーCSS共存に有用
- ファイルの先頭で `@layer` の順序を宣言するのが最も重要なパターン
- すべてのスタイルをレイヤーに入れないと、unlayered スタイルが予期せず優先される

## 参照元

- [W3C CSS Cascading and Inheritance Level 5](https://www.w3.org/TR/css-cascade-5/#layering)
- [MDN - @layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)
- [MDN - Cascade layers](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_layers)
- [Can I Use - CSS Cascade Layers](https://caniuse.com/css-cascade-layers)
- [Chrome Developers - Cascade layers](https://developer.chrome.com/blog/cascade-layers/)
- [CSS-Tricks - CSS Cascade Layers](https://css-tricks.com/css-cascade-layers/)
