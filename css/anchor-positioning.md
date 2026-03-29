# anchor positioning — ツールチップやドロップダウンの配置

## CSS Anchor Positioningとは

- ある要素（アンカー）を基準にして、別の要素（ポップオーバー、ツールチップなど）の**位置を宣言的に指定**できるCSSの新機能
- 従来JavaScriptで計算していた「この要素の下に表示」「はみ出したら上に表示」といった配置ロジックをCSSだけで実現
- ポップオーバー、ツールチップ、ドロップダウンメニュー、コンテキストメニューなどの配置に最適
- `anchor-name`、`position-anchor`、`anchor()` 関数、`@position-try` などで構成される

> 参照: [MDN - CSS anchor positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning)

## 基本的な仕組み

### 1. アンカーの定義

```css
/* アンカーとなる要素に名前を付ける */
.trigger-button {
  anchor-name: --my-anchor;
}
```

- `anchor-name` プロパティでアンカー名を定義する
- アンカー名は `--` で始まるダッシュ付き識別子（CSS変数と同じ命名規則）

### 2. ポジショニングされる要素の紐付け

```css
/* アンカーを基準に配置される要素 */
.tooltip {
  position: absolute; /* または fixed */
  position-anchor: --my-anchor;
}
```

- `position-anchor` でどのアンカーを基準にするか指定
- 配置される要素は `position: absolute` または `position: fixed` である必要がある

### 3. anchor()関数で位置を指定

```css
.tooltip {
  position: absolute;
  position-anchor: --my-anchor;

  /* アンカーの下端に配置し、左端を揃える */
  top: anchor(bottom);
  left: anchor(left);
}
```

- `anchor()` 関数はアンカー要素の**辺の位置**を返す
- 使用可能な値：`top`, `right`, `bottom`, `left`, `center`, パーセンテージ

> 参照: [MDN - anchor()](https://developer.mozilla.org/en-US/docs/Web/CSS/anchor)

## ツールチップの実装

### HTML構造

```html
<button class="trigger" popovertarget="tip1">ヘルプ</button>
<div id="tip1" popover class="tooltip">
  ここに補足説明が入ります
</div>
```

### CSS

```css
.trigger {
  anchor-name: --help-trigger;
}

.tooltip {
  position: absolute;
  position-anchor: --help-trigger;

  /* アンカーの下、中央揃え */
  top: anchor(bottom);
  left: anchor(center);
  translate: -50% 0;

  /* 基本スタイル */
  margin: 0;
  padding: 0.5rem 1rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  background: #333;
  color: white;
  font-size: 0.875rem;
  width: max-content;
  max-width: 250px;
}
```

- Popover APIと組み合わせることで、表示/非表示のロジックもHTML/CSSだけで完結
- `popovertarget` 属性でボタンとポップオーバーを紐付ける

> 参照: [MDN - Popover API](https://developer.mozilla.org/en-US/docs/Web/API/Popover_API)

## anchor()関数の詳細

### 使用可能な位置キーワード

```css
.positioned {
  position: absolute;
  position-anchor: --my-anchor;

  /* 垂直方向の位置 */
  top: anchor(top);       /* アンカーの上端 */
  top: anchor(bottom);    /* アンカーの下端 */
  top: anchor(center);    /* アンカーの垂直中央 */
  top: anchor(50%);       /* 同上（パーセンテージ指定） */

  /* 水平方向の位置 */
  left: anchor(left);     /* アンカーの左端 */
  left: anchor(right);    /* アンカーの右端 */
  left: anchor(center);   /* アンカーの水平中央 */
}
```

### 論理プロパティでの指定

```css
.positioned {
  position: absolute;
  position-anchor: --my-anchor;

  /* 書字方向に依存しない指定 */
  inset-block-start: anchor(end);    /* ブロック軸の末端 */
  inset-inline-start: anchor(start); /* インライン軸の始端 */
}
```

- 論理プロパティでは `start`, `end`, `self-start`, `self-end` が使える
- 多言語対応が必要な場合に有用

> 参照: [MDN - anchor()](https://developer.mozilla.org/en-US/docs/Web/CSS/anchor)

## 配置パターン

### アンカーの上に表示

```css
.tooltip-top {
  position: absolute;
  position-anchor: --trigger;

  bottom: anchor(top);
  left: anchor(center);
  translate: -50% 0;
  margin-bottom: 8px; /* アンカーとの間隔 */
}
```

### アンカーの右に表示

```css
.tooltip-right {
  position: absolute;
  position-anchor: --trigger;

  left: anchor(right);
  top: anchor(center);
  translate: 0 -50%;
  margin-left: 8px;
}
```

### アンカーの下・左揃え

```css
.dropdown-menu {
  position: absolute;
  position-anchor: --trigger;

  top: anchor(bottom);
  left: anchor(left);
  margin-top: 4px;
}
```

### アンカーの下・右揃え

```css
.dropdown-right {
  position: absolute;
  position-anchor: --trigger;

  top: anchor(bottom);
  right: anchor(right);
  margin-top: 4px;
}
```

## position-area による簡略記法

- `position-area` プロパティを使うと、`anchor()` 関数を使わずに**グリッドベースの直感的な配置**ができる

```css
/* anchor()を使った記述 */
.tooltip {
  position: absolute;
  position-anchor: --trigger;
  bottom: anchor(top);
  left: anchor(center);
  translate: -50% 0;
}

/* position-areaを使った簡略記述 */
.tooltip {
  position: absolute;
  position-anchor: --trigger;
  position-area: top center;
}
```

### position-areaの値

```css
/* アンカーの上 */
.top    { position-area: top center; }

/* アンカーの下 */
.bottom { position-area: bottom center; }

/* アンカーの左 */
.left   { position-area: left center; }

/* アンカーの右 */
.right  { position-area: right center; }

/* アンカーの左上 */
.top-left { position-area: top left; }

/* アンカーの右下 */
.bottom-right { position-area: bottom right; }
```

- 3x3のグリッド（top/center/bottom × left/center/right）で位置を指定するイメージ
- `span` キーワードで幅を広げることも可能

```css
/* アンカーの下、幅をアンカーと同じにする */
.dropdown {
  position: absolute;
  position-anchor: --trigger;
  position-area: bottom span-all;
}
```

> 参照: [MDN - position-area](https://developer.mozilla.org/en-US/docs/Web/CSS/position-area)

## はみ出し対策 — position-try-fallbacks

- 画面端で要素がはみ出す場合に、**自動的に別の位置にフォールバック**する仕組み
- 従来JavaScriptで実装していた「上に表示 → はみ出したら下に表示」というロジックをCSSだけで実現

### 組み込みキーワード

```css
.tooltip {
  position: absolute;
  position-anchor: --trigger;
  position-area: top center;

  /* はみ出したら反転して下に表示 */
  position-try-fallbacks: flip-block;
}
```

| キーワード | 動作 |
|---|---|
| `flip-block` | ブロック軸方向に反転（上↔下） |
| `flip-inline` | インライン軸方向に反転（左↔右） |
| `flip-block flip-inline` | 両方向に反転 |

```css
/* ドロップダウン：下にはみ出したら上に表示 */
.dropdown-menu {
  position: absolute;
  position-anchor: --trigger;
  position-area: bottom span-all;
  position-try-fallbacks: flip-block;
}

/* ツールチップ：右にはみ出したら左に表示 */
.tooltip-right {
  position: absolute;
  position-anchor: --trigger;
  position-area: right center;
  position-try-fallbacks: flip-inline;
}
```

### @position-try でカスタムフォールバック

```css
/* カスタムフォールバック位置を定義 */
@position-try --try-bottom {
  position-area: bottom center;
}

@position-try --try-left {
  position-area: left center;
}

@position-try --try-right {
  position-area: right center;
}

.tooltip {
  position: absolute;
  position-anchor: --trigger;
  position-area: top center;

  /* 優先順に試行する */
  position-try-fallbacks: --try-bottom, --try-left, --try-right;
}
```

- 上に表示 → はみ出したら下 → まだはみ出したら左 → 右、と順番にフォールバックする
- `@position-try` 内では位置関連のプロパティのみ指定可能

> 参照: [MDN - position-try-fallbacks](https://developer.mozilla.org/en-US/docs/Web/CSS/position-try-fallbacks)

## anchor-size()でアンカーのサイズを参照

- `anchor-size()` 関数でアンカー要素の**幅や高さを取得**してサイズ指定に使える

```css
.dropdown-menu {
  position: absolute;
  position-anchor: --trigger;
  position-area: bottom span-all;

  /* アンカー（トリガーボタン）と同じ幅にする */
  width: anchor-size(width);

  /* アンカーの幅を最小幅にする */
  min-width: anchor-size(width);
}
```

| 値 | 意味 |
|---|---|
| `anchor-size(width)` | アンカーの幅 |
| `anchor-size(height)` | アンカーの高さ |
| `anchor-size(block)` | アンカーのブロック軸サイズ |
| `anchor-size(inline)` | アンカーのインライン軸サイズ |

> 参照: [MDN - anchor-size()](https://developer.mozilla.org/en-US/docs/Web/CSS/anchor-size)

## 実践例：ドロップダウンメニュー

### HTML

```html
<nav>
  <button class="menu-trigger" popovertarget="dropdown1">
    メニュー ▾
  </button>
  <ul id="dropdown1" popover class="dropdown-menu">
    <li><a href="#">プロフィール</a></li>
    <li><a href="#">設定</a></li>
    <li><a href="#">ログアウト</a></li>
  </ul>
</nav>
```

### CSS

```css
.menu-trigger {
  anchor-name: --menu-trigger;
}

.dropdown-menu {
  position: absolute;
  position-anchor: --menu-trigger;

  /* ボタンの下、左揃え */
  position-area: bottom span-all;

  /* はみ出したら上に表示 */
  position-try-fallbacks: flip-block;

  /* ボタンと同じ幅を最小幅にする */
  min-width: anchor-size(width);

  /* スタイル */
  margin: 4px 0 0;
  padding: 0.25rem 0;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  background: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  list-style: none;
}

.dropdown-menu li a {
  display: block;
  padding: 0.5rem 1rem;
  color: #333;
  text-decoration: none;
}

.dropdown-menu li a:hover {
  background: #f5f5f5;
}
```

- Popover APIとAnchor Positioningの組み合わせで、JavaScript不要のドロップダウンが実現可能
- `position-try-fallbacks: flip-block` で画面下端でのはみ出しに自動対応

> 参照: [MDN - CSS anchor positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning)

## 実践例：コンテキストメニュー風ツールチップ

```css
/* 複数のアンカーに対して1つのツールチップを使い回す */
.item-1 { anchor-name: --item-1; }
.item-2 { anchor-name: --item-2; }
.item-3 { anchor-name: --item-3; }

/* HTMLのanchor属性で動的にアンカーを切り替え */
```

```html
<button class="item-1" popovertarget="shared-tip"
  popovertargetaction="toggle">項目1</button>
<button class="item-2" popovertarget="shared-tip"
  popovertargetaction="toggle">項目2</button>

<div id="shared-tip" popover class="tooltip">
  ツールチップの内容
</div>
```

```css
.tooltip {
  position: absolute;
  position-area: top center;
  position-try-fallbacks: flip-block, flip-inline;
  margin-bottom: 8px;
}
```

- HTMLの `anchor` 属性を使って暗黙的にアンカーを設定することもできる（CSS側で `position-anchor` を省略可能）

## JavaScriptとの比較

```javascript
/* 悪い例：JavaScriptで位置計算する従来の方法 */
function positionTooltip(trigger, tooltip) {
  const rect = trigger.getBoundingClientRect();
  tooltip.style.top = `${rect.bottom + 8}px`;
  tooltip.style.left = `${rect.left + rect.width / 2}px`;
  tooltip.style.transform = 'translateX(-50%)';

  // はみ出し判定
  const tooltipRect = tooltip.getBoundingClientRect();
  if (tooltipRect.bottom > window.innerHeight) {
    tooltip.style.top = `${rect.top - tooltipRect.height - 8}px`;
  }
  if (tooltipRect.right > window.innerWidth) {
    tooltip.style.left = `${rect.right}px`;
    tooltip.style.transform = 'translateX(-100%)';
  }
}
// スクロールやリサイズのたびに再計算が必要
```

```css
/* 良い例：CSS Anchor Positioningならすべて宣言的 */
.tooltip {
  position: absolute;
  position-anchor: --trigger;
  position-area: bottom center;
  position-try-fallbacks: flip-block, flip-inline;
  margin-top: 8px;
}
/* スクロール・リサイズにも自動追従、はみ出しも自動対応 */
```

- JSの命令的なコードが不要になり、保守性が大幅に向上
- スクロール・リサイズへの追従もブラウザが自動的に処理

## ブラウザサポート

- Chrome 125+、Edge 125+ でサポート
- Firefox・Safariは2026年3月時点で未サポート（開発中）
- プロダクション利用にはフォールバックが必要

### フォールバック戦略

```css
/* フォールバック：従来のposition指定 */
.tooltip {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-top: 8px;
}

/* Anchor Positioningをサポートするブラウザのみ */
@supports (anchor-name: --a) {
  .tooltip {
    position-anchor: --trigger;
    position-area: bottom center;
    position-try-fallbacks: flip-block;
    transform: none;
    top: auto;
    left: auto;
  }
}
```

- `@supports (anchor-name: --a)` で機能検出が可能
- 未サポートブラウザでは従来のCSSポジショニングをフォールバックとして使用

> 参照: [Can I Use - CSS Anchor Positioning](https://caniuse.com/css-anchor-positioning)

## まとめ

- CSS Anchor Positioningは要素間の位置関係を**宣言的に**定義できる新機能
- `anchor-name` でアンカーを定義し、`position-anchor` と `anchor()` / `position-area` で配置
- `position-try-fallbacks` で画面端のはみ出しに自動対応（従来のJSロジックが不要に）
- `anchor-size()` でアンカーのサイズを参照してサイズ指定に使える
- Popover APIと組み合わせることで、JS不要のツールチップやドロップダウンが実現可能
- 2026年3月時点ではChromium系ブラウザのみサポートのため、フォールバックの実装が必要

## 参照元

- [MDN - CSS anchor positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning)
- [MDN - anchor()](https://developer.mozilla.org/en-US/docs/Web/CSS/anchor)
- [MDN - anchor-size()](https://developer.mozilla.org/en-US/docs/Web/CSS/anchor-size)
- [MDN - position-area](https://developer.mozilla.org/en-US/docs/Web/CSS/position-area)
- [MDN - position-try-fallbacks](https://developer.mozilla.org/en-US/docs/Web/CSS/position-try-fallbacks)
- [MDN - Popover API](https://developer.mozilla.org/en-US/docs/Web/API/Popover_API)
- [Can I Use - CSS Anchor Positioning](https://caniuse.com/css-anchor-positioning)
