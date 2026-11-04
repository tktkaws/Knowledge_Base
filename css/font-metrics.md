# フォントメトリクスを調整する方法

## フォントメトリクスとは

- フォントファイルに埋め込まれた寸法情報のこと
  - **ascent** — ベースラインから上方向の高さ
  - **descent** — ベースラインから下方向の深さ
  - **cap height** — 大文字の高さ
  - **x-height** — 小文字 x の高さ
  - **line-gap** — 行間として確保される追加スペース
- ブラウザはこれらの値から行ボックス（line box）の高さを計算する
- 同じ `font-size` / `line-height` を指定しても、フォントが違えばメトリクスが違うため、見た目の高さや余白がズレる

### メトリクスが原因で起こる問題

- テキストの上下に「謎の余白」が生まれ、ボタン内のテキストが光学的に中央に見えない
- 見出しの上端をグリッドに揃えたいのに、cap height の上に余白が乗ってズレる
- Webフォントの読み込み前後でフォールバックフォントと行の高さが変わり、レイアウトシフト（CLS）が発生する

> 参照: [MDN — @font-face](https://developer.mozilla.org/ja/docs/Web/CSS/@font-face) / [Capsize 公式サイト](https://seek-oss.github.io/capsize/)

## CSSでできること1: text-box で上下の余白を削る

- `text-box`（ロングハンドは `text-box-trim` / `text-box-edge`）は、フォントメトリクスを基準にテキスト上下の余白をトリミングするプロパティ
- 旧称 `leading-trim` として提案されていた仕様

```css
/* 大文字の上端からベースラインまでにトリミング */
.heading {
  text-box: trim-both cap alphabetic;
}

/* ロングハンドで書いた場合 */
.heading {
  text-box-trim: trim-both;       /* 上下両方をトリム */
  text-box-edge: cap alphabetic;  /* 上端=cap height, 下端=ベースライン */
}
```

- `text-box-trim`: `trim-start`（上のみ）/ `trim-end`（下のみ）/ `trim-both`（両方）
- `text-box-edge`: 上端は `cap`（大文字上端）/ `ex`（x-height）/ `text`、下端は `alphabetic`（ベースライン）/ `text` などを指定
- ボタンやバッジ内テキストの中央揃え、見出しとグリッドの整列に効く

```css
/* 悪い例：マジックナンバーのpaddingで光学的中央を再現 */
.button-bad {
  padding: 13px 16px 11px; /* フォントを変えると破綻する */
}

/* 良い例：text-boxで余白そのものを削る */
.button-good {
  padding: 12px 16px;
  text-box: trim-both cap alphabetic;
}
```

> 参照: [Chrome for Developers — CSS text-box-trim](https://developer.chrome.com/blog/css-text-box-trim) / [CSS Inline Layout Module Level 3](https://drafts.csswg.org/css-inline-3/#leading-trim)

## CSSでできること2: @font-face でフォールバックのメトリクスを上書きする

- `@font-face` のディスクリプタで、フォールバックフォントのメトリクスを Web フォントに合わせて上書きできる
  - `ascent-override` / `descent-override` / `line-gap-override` — 行ボックス計算に使う値を上書き
  - `size-adjust` — グリフ全体の表示サイズを拡縮
- Web フォント読み込み前後の行の高さ・行折り返しのズレを抑え、CLS を削減できる

```css
/* Webフォント本体 */
@font-face {
  font-family: 'Noto Sans JP';
  src: url('/fonts/NotoSansJP.woff2') format('woff2');
  font-display: swap;
}

/* メトリクスをNoto Sans JPに合わせたフォールバック */
@font-face {
  font-family: 'Noto Sans JP Fallback';
  src: local('Hiragino Kaku Gothic ProN');
  ascent-override: 116%;
  descent-override: 28.8%;
  line-gap-override: 0%;
  size-adjust: 100.5%;
}

body {
  font-family: 'Noto Sans JP', 'Noto Sans JP Fallback', sans-serif;
}
```

- パーセント値は「フォントの em サイズに対する比率」— 手計算は大変なので、後述の Capsize などのツールで算出するのが現実的

> 参照: [MDN — ascent-override](https://developer.mozilla.org/ja/docs/Web/CSS/@font-face/ascent-override) / [web.dev — Improved font fallbacks](https://web.dev/articles/css-size-adjust)

## 全ブラウザに対応しているか

- 結論: **どちらもまだ全ブラウザ対応ではない**（2026年6月時点）

| 機能 | Chrome | Edge | Safari | Firefox |
|---|---|---|---|---|
| `text-box` / `text-box-trim` | **133+** | **133+** | **18.2+** | 未対応 |
| `ascent-override` 等 | **87+** | **87+** | 未対応 | **89+** |
| `size-adjust` | **92+** | **92+** | **17+** | **92+** |

- `text-box` は Firefox が未実装（仕様への立場は positive で、実装待ちの状態）
- `ascent-override` / `descent-override` / `line-gap-override` は Safari が未実装
- ただしどちらも **非対応ブラウザで壊れない** のが特徴
  - `text-box` 非対応 → 余白が従来どおり残るだけ（プログレッシブエンハンスメント）
  - メトリクスオーバーライド非対応 → フォールバック表示が少しズレるだけ
- 対応状況で分岐したい場合は `@supports` が使える

```css
@supports (text-box: trim-both cap alphabetic) {
  .heading {
    text-box: trim-both cap alphabetic;
  }
}
```

> 参照: [Can I use — CSS Text Box](https://caniuse.com/css-text-box-trim) / [web-features — Font metric overrides](https://web-platform-dx.github.io/web-features-explorer/features/font-metric-overrides/)

## Capsizeの使い方

- [Capsize](https://seek-oss.github.io/capsize/) は SEEK 製のフォントメトリクス計算ライブラリ
- cap height 基準のトリミングを、`text-box` 非対応ブラウザを含む **全ブラウザで今すぐ実現できる**
- 仕組み: 疑似要素（`::before` / `::after`）の負のマージンで余白を相殺する CSS を、フォントメトリクスから自動計算して生成する

### インストール

```bash
npm install @capsizecss/core @capsizecss/metrics
```

- `@capsizecss/core` — スタイル計算の本体
- `@capsizecss/metrics` — システムフォントと Google Fonts のメトリクスデータ集（自前でフォントを解析する必要がない）

### createStyleObject — トリミング済みスタイルの生成

```ts
import { createStyleObject } from '@capsizecss/core';
import robotoMetrics from '@capsizecss/metrics/roboto';

const capsizeStyles = createStyleObject({
  capHeight: 48,  // 大文字の高さを48pxにする（font-sizeではない点に注意）
  lineGap: 24,    // 行と行の間の実際の空きを24pxにする
  fontMetrics: robotoMetrics,
});

// CSS-in-JS でそのまま展開できる
// <div css={{ fontFamily: 'Roboto', ...capsizeStyles }}>Capsized!</div>
```

- 生成されるのは次のような構造のスタイル — 値はフォントごとに計算される

```css
.capsized {
  font-size: 65.25px;
  line-height: 72px;
}
.capsized::before {
  content: '';
  display: table;
  margin-bottom: -0.1em; /* 上の余白を相殺（値は自動計算） */
}
.capsized::after {
  content: '';
  display: table;
  margin-top: -0.1em;    /* 下の余白を相殺（値は自動計算） */
}
```

- `capHeight` 指定により「フォントサイズ」ではなく「見た目の文字の高さ」でテキストを設計できる
- `fontSize` + `leading` の組み合わせでも指定可能

### createFontStack — フォールバック調整の自動生成

- `ascent-override` / `size-adjust` などの値を手計算せず、フォールバック用の `@font-face` を丸ごと生成できる

```ts
import { createFontStack } from '@capsizecss/core';
import lobster from '@capsizecss/metrics/lobster';
import arial from '@capsizecss/metrics/arial';

const { fontFamily, fontFaces } = createFontStack([lobster, arial]);

// fontFamily: "Lobster, 'Lobster Fallback', Arial"
// fontFaces: メトリクスを揃えた @font-face 宣言の文字列
```

```css
/* fontFaces の出力イメージ */
@font-face {
  font-family: 'Lobster Fallback';
  src: local('Arial');
  ascent-override: 115.17%;
  descent-override: 28.97%;
  size-adjust: 86.59%;
}
```

- 配列の先頭が本命の Web フォント、以降がフォールバックの優先順
- 出力をビルド時に CSS へ埋め込めば、ランタイムコストはゼロ

> 参照: [GitHub — seek-oss/capsize](https://github.com/seek-oss/capsize) / [@capsizecss/metrics](https://www.npmjs.com/package/@capsizecss/metrics)

## よくある間違い

### 1. line-height: 1 にすれば余白が消えると思っている

```css
/* 間違い：line-height: 1 でも余白は残る */
.heading {
  line-height: 1;
  /* ascent/descentがcap heightより大きいフォントでは上下に余白が残る */
}

/* 正解：余白を消したいならメトリクスベースでトリムする */
.heading {
  line-height: 1.5;
  text-box: trim-both cap alphabetic;
}
```

- 上下の余白は `line-height` のハーフレディングだけでなく、フォント自体の ascent / descent からも生まれる
- `line-height: 1` は行間を詰めるだけで、フォント内部のメトリクス由来の余白は制御できない

### 2. text-box 前提のレイアウトを組んでしまう

- Firefox では余白が残るため、トリミングされている前提でピクセル単位の位置合わせをすると崩れる
- 「非対応ブラウザでは余白が少し広い」だけで済むデザインにしておく

### 3. フォールバック調整をせずに font-display: swap を使う

```css
/* 間違い：swapだけ指定 — フォント切り替え時に行送りが変わりCLSが発生 */
@font-face {
  font-family: 'Web Font';
  src: url('/fonts/webfont.woff2') format('woff2');
  font-display: swap;
}

/* 正解：メトリクスを揃えたフォールバックをセットで用意する */
@font-face {
  font-family: 'Web Font Fallback';
  src: local('Arial');
  ascent-override: 105%;
  descent-override: 27%;
  size-adjust: 98%;
}
```

- `font-display: swap` は表示は速くするが、切り替え時のレイアウトシフトは防がない
- メトリクスオーバーライド（または Capsize の `createFontStack`）とセットで使う

## まとめ

- フォントメトリクスは行ボックス計算の元データ — フォントごとに違うため余白やズレの原因になる
- `text-box`（`text-box-trim` / `text-box-edge`）でテキスト上下の余白をメトリクス基準で削れる
- `@font-face` の `ascent-override` / `size-adjust` などでフォールバックフォントを Web フォントに揃えられる
- 全ブラウザ対応はまだ — `text-box` は Firefox、メトリクスオーバーライドは Safari が未対応（2026年6月時点）
- ただしどちらも非対応ブラウザで壊れないため、プログレッシブエンハンスメントとして導入できる
- Capsize を使えば、トリミングは全ブラウザ対応の形（疑似要素 + 負のマージン）で、フォールバック調整は `createFontStack` で自動生成できる
- 値の手計算はしない — メトリクスの算出はツールに任せる
