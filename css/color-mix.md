# color-mix()とCSS Color Level 4

## color-mix()とは

- 2つの色を指定した割合で**混合**して新しい色を生成するCSS関数
- CSS Color Level 5で定義された機能
- Sassの `mix()` やJavaScriptでの色計算に頼っていた処理が、ネイティブCSSだけで可能になった
- カスタムプロパティとの組み合わせで、テーマカラーのバリエーション生成に特に有効

> 参照: [MDN - color-mix()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix)

## 基本構文

```css
color-mix(in <色空間>, <色1> <割合?>, <色2> <割合?>)
```

```css
/* 青と白を50%ずつ混ぜる（薄い青） */
background-color: color-mix(in srgb, #0066cc, white);

/* 青を70%、白を30%で混ぜる */
background-color: color-mix(in srgb, #0066cc 70%, white);

/* 赤と青を等分に混ぜる */
background-color: color-mix(in srgb, red, blue);
```

- 第1引数: 混合に使う**色空間**（`in srgb`、`in oklch` など）
- 第2引数: 1つ目の色とオプションの割合
- 第3引数: 2つ目の色とオプションの割合
- 割合を省略すると**50%ずつ**（等分）になる
- 片方だけ割合を指定すると、もう片方は `100% - 指定値` になる

> 参照: [MDN - color-mix()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix)

## 色空間の選択

### 主な色空間の特徴

| 色空間 | 特徴 | 適した用途 |
|---|---|---|
| `srgb` | 従来のRGBベースの色空間 | 既存のデザインとの互換性を保ちたい場合 |
| `oklch` | 人間の知覚に基づく均一な色空間 | 自然で美しいグラデーションや混色 |
| `oklab` | oklchの直交座標版 | 知覚的に均一な混色 |
| `hsl` | 色相・彩度・明度 | 色相の中間色を直感的に得たい場合 |

### 色空間による混色結果の違い

```css
/* srgb: RGB値を線形に混ぜる */
.srgb {
  background: color-mix(in srgb, blue, yellow);
  /* → グレーっぽくなりがち */
}

/* oklch: 知覚的に均一な混色 */
.oklch {
  background: color-mix(in oklch, blue, yellow);
  /* → 鮮やかな中間色（緑寄り）になる */
}

/* hsl: 色相環上で混ぜる */
.hsl {
  background: color-mix(in hsl, blue, yellow);
  /* → 色相環の経路で中間の色になる */
}
```

- **迷ったら `oklch` を推奨** — 知覚的に自然な結果が得られる
- `srgb` は直感に反してくすんだ中間色になることがある
- 既存の `rgb()` / `hex` との互換性を重視するなら `srgb`

> 参照: [MDN - color-mix()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix)

## 割合の指定パターン

```css
/* 両方省略 → 50%:50% */
color-mix(in oklch, red, blue)

/* 片方だけ指定 → もう片方は残り */
color-mix(in oklch, red 70%, blue)  /* = red 70%, blue 30% */

/* 両方指定 */
color-mix(in oklch, red 60%, blue 40%)  /* = 60%:40% */

/* 合計が100%未満 → 透明度が加わる */
color-mix(in oklch, red 30%, blue 20%)
/* 合計50% → 結果の色は 60%:40% の比率で混ぜた上で、50%の不透明度になる */
```

- 合計が100%を超える場合は正規化される（各値を合計で割る）
- 合計が100%未満の場合は**アルファ値（透明度）** に反映される

> 参照: [MDN - color-mix()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix)

## 実践的な活用例

### テーマカラーのバリエーション生成

```css
/* 悪い例：バリエーションごとに色を手動で定義 */
:root {
  --primary: #0066cc;
  --primary-light: #3385d6;  /* 手動で計算 */
  --primary-lighter: #66a3e0; /* 手動で計算 */
  --primary-dark: #004d99;    /* 手動で計算 */
  --primary-darker: #003366;  /* 手動で計算 */
}

/* 良い例：color-mix()で動的に生成 */
:root {
  --primary: #0066cc;
  --primary-light: color-mix(in oklch, var(--primary), white 30%);
  --primary-lighter: color-mix(in oklch, var(--primary), white 60%);
  --primary-dark: color-mix(in oklch, var(--primary), black 30%);
  --primary-darker: color-mix(in oklch, var(--primary), black 60%);
}
```

- ベースカラーを変更するだけですべてのバリエーションが自動更新される
- Sassの `lighten()` / `darken()` に相当する処理がネイティブCSSで実現

### ホバー・アクティブ状態の色

```css
.button {
  --button-color: #0066cc;
  background-color: var(--button-color);
  color: #fff;
}

.button:hover {
  background-color: color-mix(in oklch, var(--button-color), black 15%);
}

.button:active {
  background-color: color-mix(in oklch, var(--button-color), black 30%);
}

.button:disabled {
  background-color: color-mix(in oklch, var(--button-color), white 50%);
  cursor: not-allowed;
}
```

- ボタンの色を1箇所変えるだけで、hover / active / disabled の色もすべて連動する

### 半透明のオーバーレイ

```css
/* 悪い例：rgba()で透明度を手動調整 */
.overlay {
  background-color: rgba(0, 0, 0, 0.5);
}

/* 良い例：color-mix()で透明色と混ぜる */
.overlay {
  background-color: color-mix(in srgb, var(--overlay-color) 50%, transparent);
}
```

- `transparent` と混ぜることで任意の色に透明度を加えられる
- カスタムプロパティで定義した色にも動的に透明度を適用できる

### ダークモード対応

```css
:root {
  --surface: #ffffff;
  --text: #1a1a1a;
  --border: color-mix(in oklch, var(--text), var(--surface) 80%);
  --text-muted: color-mix(in oklch, var(--text), var(--surface) 40%);
}

[data-theme="dark"] {
  --surface: #1a1a1a;
  --text: #e0e0e0;
  /* --border と --text-muted は自動的にダークモードに適応 */
}
```

- `--surface` と `--text` を切り替えるだけで、混合色も自動的にテーマに適応する

> 参照: [web.dev - CSS color-mix()](https://developer.chrome.com/docs/css-ui/css-color-mix)

## CSS Color Level 4の新しい色空間

CSS Color Level 4では、従来の `rgb()` / `hsl()` に加えて新しい色指定方法が追加された。

### oklch() — 知覚的に均一な色空間

```css
/* oklch(明度 彩度 色相) */
.text {
  color: oklch(0.5 0.2 240);
  /* 明度: 0〜1, 彩度: 0〜0.4程度, 色相: 0〜360 */
}
```

- **L（Lightness）**: 明度。0（黒）〜 1（白）
- **C（Chroma）**: 彩度。0（無彩色）〜 約0.4（最高彩度）
- **H（Hue）**: 色相。0〜360の角度
- 明度を変えても**色相と彩度が安定する**のが最大の特徴
- hsl()では明度を変えると色味が変わることがあるが、oklch()ではそれが起きにくい

```css
/* hsl: 明度を変えると色味がずれることがある */
.hsl-light { color: hsl(240, 100%, 70%); }
.hsl-dark  { color: hsl(240, 100%, 30%); }

/* oklch: 明度を変えても色相・彩度が安定 */
.oklch-light { color: oklch(0.7 0.15 240); }
.oklch-dark  { color: oklch(0.3 0.15 240); }
```

> 参照: [MDN - oklch()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch)

### oklab() — oklchの直交座標版

```css
/* oklab(明度 a軸 b軸) */
.text {
  color: oklab(0.5 -0.1 0.1);
  /* L: 0〜1, a: -0.4〜0.4（緑〜赤）, b: -0.4〜0.4（青〜黄） */
}
```

- oklchの色相と彩度を直交座標で表現したもの
- 色の補間（グラデーション、アニメーション）でより自然な中間色が得られる

> 参照: [MDN - oklab()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklab)

### lab() / lch()

```css
/* lab(明度 a軸 b軸) */
.text { color: lab(50 -20 40); }

/* lch(明度 彩度 色相) */
.text { color: lch(50 60 240); }
```

- CIE Lab / LCH 色空間に基づく指定
- oklch / oklab の前身で、知覚均一性がやや劣る
- **新規プロジェクトでは oklch / oklab を推奨**

### display-p3 — 広色域

```css
.vivid {
  /* sRGBより広い色域で鮮やかな色を表現 */
  color: color(display-p3 1 0 0);
  /* sRGBのred(#ff0000)よりさらに鮮やかな赤 */
}
```

- sRGBより約50%広い色域を持つ
- 対応ディスプレイ（iPhone、MacのRetinaなど）でより鮮やかな色が表示される
- 非対応ディスプレイでは最も近いsRGB色にフォールバックされる

> 参照: [MDN - color()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color)

## 新しい色構文

### スペース区切り構文

```css
/* CSS Color Level 4: スペース区切り + スラッシュで透明度 */
.new-syntax {
  color: rgb(0 102 204);
  color: rgb(0 102 204 / 0.5);
  color: hsl(210 100% 40%);
  color: hsl(210 100% 40% / 0.8);
}

/* 従来の構文（引き続き有効） */
.old-syntax {
  color: rgb(0, 102, 204);
  color: rgba(0, 102, 204, 0.5);
  color: hsl(210, 100%, 40%);
  color: hsla(210, 100%, 40%, 0.8);
}
```

- カンマ区切りからスペース区切りに変更され、透明度は `/` の後に記述する
- `rgba()` と `hsla()` は `rgb()` と `hsl()` に統合された
- 従来のカンマ区切り構文もそのまま使える

> 参照: [MDN - rgb()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/rgb)

## relative color syntax（相対カラー構文）

- 既存の色を基準に、各チャネルを個別に調整して新しい色を生成する構文
- CSS Color Level 5で定義

```css
/* 基本構文: color-function(from <基準色> チャネル1 チャネル2 チャネル3 / 透明度) */

/* 基準色の明度だけを変更 */
.lighter {
  color: oklch(from var(--primary) calc(l + 0.2) c h);
  /* 明度を0.2上げる */
}

/* 基準色の彩度だけを落とす */
.muted {
  color: oklch(from var(--primary) l calc(c * 0.5) h);
  /* 彩度を半分にする */
}

/* 基準色の透明度を変更 */
.translucent {
  color: oklch(from var(--primary) l c h / 0.5);
}

/* 基準色の色相を回転 */
.complement {
  color: oklch(from var(--primary) l c calc(h + 180));
  /* 補色を生成 */
}
```

- `from <色>` で基準色を指定し、`l`、`c`、`h` などのチャネル変数を使って個別に調整
- `color-mix()` よりも細かい制御が可能
- カラーパレットの自動生成やアクセシブルな色調整に有用

> 参照: [MDN - Relative color](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_colors/Relative_colors)

## ブラウザサポート

### color-mix()

- Chrome 111+、Safari 16.2+、Firefox 113+ で全主要ブラウザがサポート
- 2024年以降、プロダクションで安心して使える

### oklch() / oklab()

- Chrome 111+、Safari 15.4+、Firefox 113+ でサポート

### relative color syntax

- Chrome 119+、Safari 16.4+、Firefox 128+ でサポート
- 比較的新しい機能のためフォールバックを検討する

### フォールバック

```css
.button {
  /* フォールバック: 固定の色値 */
  background-color: #0066cc;

  /* color-mix()をサポートするブラウザのみ */
  background-color: color-mix(in oklch, var(--primary), black 15%);
}
```

```css
/* @supportsによる機能検出 */
.card {
  background-color: #f0f7ff;
}

@supports (background: color-mix(in oklch, red, blue)) {
  .card {
    background-color: color-mix(in oklch, var(--primary), white 90%);
  }
}
```

- CSSのカスケードを利用し、先にフォールバック値を書き、後に `color-mix()` を書くのが最もシンプル
- ブラウザが `color-mix()` を理解できなければ、フォールバック値が使われる

> 参照: [Can I Use - color-mix()](https://caniuse.com/mdn-css_types_color_color-mix)

## まとめ

- `color-mix()` は2つの色を指定した色空間と割合で混合するCSS関数
- 色空間は **`oklch` を推奨** — 知覚的に自然な混色結果が得られる
- カスタムプロパティと組み合わせ、ベースカラーからバリエーションを動的に生成できる
- CSS Color Level 4で `oklch()`、`oklab()`、`display-p3` などの新しい色空間が追加された
- relative color syntaxを使えば、既存の色の特定チャネルだけを調整できる
- `rgb()` / `hsl()` はスペース区切り構文に対応し、`rgba()` / `hsla()` は不要になった
- 全主要ブラウザでサポート済み、カスケードによるフォールバックも容易

## 参照元

- [MDN - color-mix()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix)
- [MDN - oklch()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch)
- [MDN - oklab()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklab)
- [MDN - color()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color)
- [MDN - Relative color](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_colors/Relative_colors)
- [web.dev - CSS color-mix()](https://developer.chrome.com/docs/css-ui/css-color-mix)
- [Can I Use - color-mix()](https://caniuse.com/mdn-css_types_color_color-mix)
