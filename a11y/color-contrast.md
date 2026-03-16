# カラーコントラスト比の基準と確認方法

## カラーコントラスト比とは

- 2つの色の相対的な明るさの差を数値で表したもの
- 前景色（テキストやアイコン）と背景色の間の視認性を客観的に測定する指標
- 比率は **1:1**（同一色、差なし）から **21:1**（黒と白、最大差）の範囲
- 視覚障害・色覚特性・高齢者・屋外利用など、さまざまな状況での読みやすさに直結

### 計算式の概要

- WCAG 2.2 では **相対輝度（Relative Luminance）** をもとに算出
- 相対輝度 `L` は各色チャンネル（R, G, B）を線形化して加重平均した値
- コントラスト比の公式: `(L1 + 0.05) / (L2 + 0.05)`（L1 は明るい方、L2 は暗い方）
- 実際の開発ではツールで自動計算するため、公式を暗記する必要はない

> 参照: [WCAG 2.2 — Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html) / [W3C — Relative Luminance](https://www.w3.org/WAI/GL/wiki/Relative_luminance)

## WCAG 2.2 のコントラスト比基準

### テキストのコントラスト比（Success Criterion 1.4.3 / 1.4.6）

| 適合レベル | 通常テキスト | 大きなテキスト |
|---|---|---|
| AA（最低限） | **4.5:1** 以上 | **3:1** 以上 |
| AAA（推奨） | **7:1** 以上 | **4.5:1** 以上 |

- **大きなテキスト** の定義: 18pt（24px）以上、または太字の14pt（18.66px）以上
- ロゴやブランド名のテキストはコントラスト比の要件対象外
- 装飾目的のみのテキスト、非アクティブなUIコンポーネント内のテキストも対象外

### 非テキスト要素のコントラスト比（Success Criterion 1.4.11）

| 対象 | 必要なコントラスト比 |
|---|---|
| UIコンポーネント（ボタン境界線、入力欄の枠線など） | **3:1** 以上 |
| グラフィカルオブジェクト（アイコン、チャートなど） | **3:1** 以上 |
| フォーカスインジケーター | **3:1** 以上 |

- UIコンポーネントの「状態」を示す視覚的手がかりも3:1が必要（例: チェックボックスのチェック状態）
- disabled 状態の要素は要件対象外だが、ユーザビリティの観点から配慮が望ましい

> 参照: [WCAG 2.2 — Non-text Contrast](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html)

## 良い例・悪い例

### テキストのコントラスト

```html
<!-- 悪い例：コントラスト比 2.4:1 — AA基準を満たさない -->
<p style="color: #999999; background-color: #ffffff;">
  このテキストは読みにくい
</p>

<!-- 良い例：コントラスト比 7.0:1 — AAA基準を満たす -->
<p style="color: #595959; background-color: #ffffff;">
  このテキストは十分に読みやすい
</p>
```

```css
/* 悪い例：薄いグレーのテキスト */
.caption-bad {
  color: #aaaaaa; /* 白背景に対して 2.3:1 */
  background-color: #ffffff;
}

/* 良い例：十分なコントラストを確保 */
.caption-good {
  color: #767676; /* 白背景に対して 4.5:1（AA基準ちょうど） */
  background-color: #ffffff;
}
```

### ボタンのコントラスト

```html
<!-- 悪い例：ボタンの境界線と背景のコントラスト比が不足 -->
<button style="
  background-color: #ffffff;
  border: 1px solid #cccccc;
  color: #cccccc;
">
  送信
</button>

<!-- 良い例：境界線もテキストも十分なコントラスト -->
<button style="
  background-color: #ffffff;
  border: 2px solid #767676;
  color: #333333;
">
  送信
</button>
```

### 入力フィールドのコントラスト

```css
/* 悪い例：境界線のコントラスト不足（背景との比 1.6:1） */
.input-bad {
  border: 1px solid #cccccc;
  background-color: #ffffff;
  color: #333333;
}

/* 良い例：境界線のコントラスト確保（背景との比 3.1:1） */
.input-good {
  border: 1px solid #767676;
  background-color: #ffffff;
  color: #333333;
}
```

### アイコンのコントラスト

```css
/* 悪い例：アイコンの色が薄い（背景との比 2.1:1） */
.icon-bad {
  color: #bbbbbb;
  background-color: #ffffff;
}

/* 良い例：アイコンのコントラスト確保（背景との比 3.9:1） */
.icon-good {
  color: #808080;
  background-color: #ffffff;
}
```

> 参照: [WebAIM — Contrast Checker](https://webaim.org/resources/contrastchecker/)

## コントラスト比の確認ツール

### ブラウザ DevTools

- Chrome DevTools: 要素を選択 → Styles パネルで色をクリック → カラーピッカーにコントラスト比が表示される
- Firefox DevTools: アクセシビリティタブでコントラスト比を確認可能
- コントラスト比がWCAG基準を満たしているかアイコンで表示（チェックマーク / 警告）

### WebAIM Contrast Checker

- URL: https://webaim.org/resources/contrastchecker/
- 前景色と背景色を入力するとコントラスト比を計算
- AA / AAA の適合結果を通常テキスト・大きなテキスト別に表示
- 最も広く使われているオンラインツール

### axe（axe DevTools / axe-core）

- ブラウザ拡張機能として利用可能
- ページ全体を自動スキャンしてコントラスト違反を検出
- 違反箇所の具体的な修正案を提示
- CI/CDパイプラインに組み込める axe-core ライブラリも提供

### Stark

- Figma / Sketch / Adobe XD のプラグインとして利用可能
- デザイン段階でコントラスト比を確認できる
- 色覚シミュレーション機能も搭載

### その他のツール

| ツール | 特徴 |
|---|---|
| Colour Contrast Analyser（CCA） | デスクトップアプリ、スポイトツールで画面上の色を取得可能 |
| Lighthouse | Chrome DevTools内蔵、アクセシビリティスコアの一部としてコントラストを検証 |
| eslint-plugin-jsx-a11y | React JSXのコード中でコントラスト問題を静的解析 |

> 参照: [axe DevTools](https://www.deque.com/axe/devtools/) / [Stark](https://www.getstark.co/) / [Colour Contrast Analyser](https://www.tpgi.com/color-contrast-checker/)

## CSSでの実践的なテクニック

### カスタムプロパティでカラーパレットを管理

- コントラスト比を満たす色の組み合わせをカスタムプロパティで一元管理
- 変更時に一箇所を修正すれば全体に反映される

```css
:root {
  /* コントラスト比を検証済みのカラーパレット */
  --color-text-primary: #1a1a1a;     /* 白背景に対して 17.4:1 */
  --color-text-secondary: #595959;   /* 白背景に対して 7.0:1（AAA） */
  --color-text-tertiary: #767676;    /* 白背景に対して 4.5:1（AA） */
  --color-bg-primary: #ffffff;
  --color-border: #767676;           /* 白背景に対して 4.5:1 */

  --color-link: #0056b3;             /* 白背景に対して 7.1:1 */
  --color-link-visited: #6b2fa0;     /* 白背景に対して 6.0:1 */

  --color-error: #d32f2f;            /* 白背景に対して 5.6:1 */
  --color-success: #2e7d32;          /* 白背景に対して 5.0:1 */
}

body {
  color: var(--color-text-primary);
  background-color: var(--color-bg-primary);
}

a {
  color: var(--color-link);
}

.error-message {
  color: var(--color-error);
}
```

### transparent の問題

- `transparent` は `rgba(0, 0, 0, 0)` と等価
- 半透明の色はコントラスト比の計算を複雑にする
- 実際に描画される色を確認しないと正確なコントラスト比がわからない

```css
/* 悪い例：背景が透明 — 親要素の背景次第でコントラスト比が変わる */
.overlay-text-bad {
  color: #ffffff;
  background-color: transparent;
  /* 親の背景が白ならコントラスト比 1:1 で読めない */
}

/* 良い例：半透明でも十分な暗さを確保 */
.overlay-text-good {
  color: #ffffff;
  background-color: rgba(0, 0, 0, 0.75);
  /* 黒75%の半透明 — 白テキストとのコントラスト比を十分確保 */
}
```

### hover / focus 状態のコントラスト

- インタラクション状態でもコントラスト比の基準を維持する必要がある
- hover で色が薄くなるデザインは要注意

```css
/* 悪い例：hover時にコントラスト不足 */
.link-bad {
  color: #0056b3; /* 通常時は十分なコントラスト */
}
.link-bad:hover {
  color: #66aaff; /* hover時 2.5:1 — 白背景に対して不足 */
}

/* 良い例：hover時もコントラスト確保 */
.link-good {
  color: #0056b3; /* 7.1:1 */
}
.link-good:hover {
  color: #003d80; /* 10.1:1 — hover時もAAA基準を維持 */
}

/* フォーカスインジケーターのコントラスト */
/* 悪い例：フォーカスリングが薄い */
.button-bad:focus-visible {
  outline: 2px solid #cccccc; /* 白背景に対して 1.6:1 */
}

/* 良い例：フォーカスリングが明瞭 */
.button-good:focus-visible {
  outline: 2px solid #0056b3; /* 白背景に対して 7.1:1 */
  outline-offset: 2px;
}
```

> 参照: [WCAG 2.2 — Focus Appearance](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html)

## ダークモード対応時のコントラスト注意点

### よくある問題

- ライトモードでコントラスト基準を満たしていても、ダークモードで不足するケース
- 色をそのまま反転するだけでは適切なコントラストにならない
- 純黒（`#000000`）の背景に純白（`#ffffff`）は眩しく、長時間の閲覧で疲労を引き起こす

### ダークモードのカラーパレット設計

```css
/* ライトモードとダークモードで個別にコントラスト検証済みの色を定義 */
:root {
  --color-text-primary: #1a1a1a;
  --color-text-secondary: #595959;
  --color-bg-primary: #ffffff;
  --color-bg-surface: #f5f5f5;
  --color-border: #767676;
  --color-link: #0056b3;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-text-primary: #e0e0e0;      /* 暗い背景に対して 13.3:1 */
    --color-text-secondary: #a0a0a0;    /* 暗い背景に対して 6.6:1 */
    --color-bg-primary: #121212;
    --color-bg-surface: #1e1e1e;
    --color-border: #8a8a8a;            /* 暗い背景に対して 5.2:1 */
    --color-link: #6db3f8;              /* 暗い背景に対して 8.0:1 */
  }
}
```

```css
/* 悪い例：ライトモードの色をそのままダークモードに使う */
@media (prefers-color-scheme: dark) {
  .error-bad {
    color: #d32f2f; /* 暗い背景（#121212）に対して 3.2:1 — 不足 */
    background-color: #121212;
  }
}

/* 良い例：ダークモード用に明るさを調整 */
@media (prefers-color-scheme: dark) {
  .error-good {
    color: #f28b82; /* 暗い背景（#121212）に対して 5.9:1 — AA基準達成 */
    background-color: #121212;
  }
}
```

### ダークモード設計の指針

- 純黒（`#000000`）ではなく濃いグレー（`#121212` 〜 `#1e1e1e`）を背景に使用
- 純白（`#ffffff`）ではなく淡いグレー（`#e0e0e0` 〜 `#eeeeee`）をテキストに使用
- ブランドカラーやステータスカラーは、ダークモード用に明度・彩度を調整した別バリエーションを用意
- 各色の組み合わせでコントラスト比を個別に検証

> 参照: [Material Design — Dark theme](https://m3.material.io/styles/color/dynamic/choosing-a-source) / [WCAG 2.2 — Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)

## よくある失敗パターンと修正方法

### 1. プレースホルダーのコントラスト不足

- プレースホルダーテキストはWCAG 1.4.3の対象ではないが、ユーザビリティ上の問題になる
- プレースホルダーに重要な情報（入力形式など）を含めている場合は特に問題

```css
/* 悪い例：ブラウザデフォルトのプレースホルダーは非常に薄い */
.input-bad::placeholder {
  color: #c0c0c0; /* 白背景に対して 1.6:1 */
}

/* 良い例：プレースホルダーにも十分なコントラストを確保 */
.input-good::placeholder {
  color: #767676; /* 白背景に対して 4.5:1 */
}
```

- 根本的な対策: プレースホルダーに頼らず、`<label>` 要素で入力項目を説明する

### 2. disabled 状態のコントラスト

- disabled 要素はWCAGの適合基準の対象外
- ただし「操作できない」ことがユーザーに伝わらなければ混乱を招く
- コントラストを一定以上確保しつつ、操作不可であることを視覚的に示す工夫が必要

```css
/* 悪い例：disabledが背景と区別できない */
.button-disabled-bad {
  color: #e0e0e0;
  background-color: #f0f0f0;
  /* テキストと背景のコントラスト比 1.2:1 — 存在自体が認識できない */
}

/* 良い例：操作不可を伝えつつ読める程度のコントラストを維持 */
.button-disabled-good {
  color: #767676;
  background-color: #e0e0e0;
  cursor: not-allowed;
  /* テキストと背景のコントラスト比 2.8:1 — 読める範囲で薄くする */
}
```

### 3. グラデーション背景上のテキスト

- グラデーションは位置によってコントラスト比が変動する
- 最もコントラストが低くなる箇所で基準を満たす必要がある

```css
/* 悪い例：グラデーションの明るい部分で白テキストが読めない */
.hero-bad {
  background: linear-gradient(to right, #1a237e, #42a5f5);
  color: #ffffff;
  /* 右端（#42a5f5）と白テキストのコントラスト比 2.3:1 — 不足 */
}

/* 良い例：テキスト部分に半透明のオーバーレイを敷く */
.hero-good {
  background: linear-gradient(to right, #1a237e, #42a5f5);
  color: #ffffff;
  position: relative;
}
.hero-good::before {
  content: '';
  position: absolute;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.5);
  /* オーバーレイで全体を暗くし、白テキストのコントラストを確保 */
}
.hero-good .content {
  position: relative; /* オーバーレイの上にテキストを配置 */
}
```

### 4. 色だけで情報を伝える

- コントラスト比を満たしていても、色だけが唯一の識別手段では不十分（WCAG 1.4.1）
- 色覚特性を持つユーザーには色の違いが認識できない場合がある

```html
<!-- 悪い例：エラーを赤色のみで伝える -->
<input style="border: 2px solid red;" />

<!-- 良い例：色 + テキスト + アイコンで伝える -->
<div>
  <input style="border: 2px solid #d32f2f;" aria-describedby="error-msg" aria-invalid="true" />
  <p id="error-msg" style="color: #d32f2f;">
    ⚠ メールアドレスの形式が正しくありません
  </p>
</div>
```

### 5. リンクと通常テキストのコントラスト

- リンクテキストは背景とのコントラスト比だけでなく、周囲のテキストとの区別も必要
- 色の違いだけでリンクを示す場合、リンク色と通常テキスト色のコントラスト比が **3:1** 以上必要
- 下線を付ければこの要件は不要

```css
/* 悪い例：下線なし & 通常テキストとの色差が小さい */
.text-with-link-bad a {
  color: #555555; /* 通常テキスト #333333 との比 1.4:1 */
  text-decoration: none;
}

/* 良い例1：下線を付ける */
.text-with-link-good a {
  color: #0056b3;
  text-decoration: underline;
}

/* 良い例2：下線なしなら通常テキストとの色差3:1以上を確保 */
.text-with-link-alt a {
  color: #0056b3; /* 通常テキスト #333333 との比 3.1:1 */
  text-decoration: none;
}
.text-with-link-alt a:hover,
.text-with-link-alt a:focus {
  text-decoration: underline; /* hover/focusで下線を表示 */
}
```

> 参照: [WCAG 2.2 — Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html) / [WebAIM — Links and Hypertext](https://webaim.org/techniques/hypertext/link_text)

## まとめ

- コントラスト比は前景色と背景色の明るさの比率（1:1 〜 21:1）
- WCAG 2.2 AA 基準: 通常テキスト **4.5:1**、大きなテキスト **3:1**、非テキスト要素 **3:1**
- WCAG 2.2 AAA 基準: 通常テキスト **7:1**、大きなテキスト **4.5:1**
- DevTools・WebAIM Contrast Checker・axe などのツールで確認
- CSSカスタムプロパティで検証済みカラーパレットを一元管理
- hover / focus 状態でもコントラスト基準を維持
- ダークモードではライトモードとは別にコントラスト比を検証
- 色だけで情報を伝えない — テキスト・アイコン・パターンを併用
