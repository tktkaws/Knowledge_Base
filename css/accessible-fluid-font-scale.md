# フォントサイズをremで指定したまま画面幅に応じて拡縮させる

フォントサイズは `rem` で指定しつつ、デザイン上の都合から画面幅に合わせて滑らかに拡大・縮小したいことがあります。

ただし、画面幅と文字サイズの連動はブラウザズームによる文字拡大を妨げる可能性があります。この記事では、最初にアクセシビリティ上の注意点を整理したうえで、JavaScriptとCSSを使う方法、CSSだけで実装する方法を紹介します。

## 目次

- アクセシビリティの観点では、基本的に `rem` で指定して画面幅とは連動させない
- フォントサイズに関係するWCAG 2.2の達成基準を確認する
- 既存の `vw` 実装を改善する場合や、比率維持が不可欠なレイアウトに限定して使用する
- JavaScriptで画面幅を検知し、CSS変数をリアルタイムに変更する
- `@property` と `tan(atan2())` を使い、CSSだけで実装する
- `@property` と `tan(atan2())` は `cqw` でも使用できる

## 注意

### 基本は `rem` で固定する

通常の本文や見出しは、画面幅と連動させず `rem` で指定するのが基本です。

```css
body {
  font-size: 1rem;
}

h1 {
  font-size: 2rem;
}
```

`rem` はルート要素の文字サイズを基準にするため、ブラウザやOSの文字サイズ設定を反映しやすくなります。画面が狭い場合も文字自体を小さくするのではなく、折り返しやレイアウトのリフローで対応するほうが読みやすさを保てます。

画面幅ごとの調整が必要であっても、まずはメディアクエリで段階的に `rem` の値を切り替える方法を検討します。

```css
.heading {
  font-size: 1.75rem;
}

@media (width >= 48rem) {
  .heading {
    font-size: 2rem;
  }
}
```

### `vw` だけで指定すると文字を拡大しにくい

次のように `vw` だけでフォントサイズを指定すると、文字サイズがビューポート幅だけで決まります。

```css
.heading {
  font-size: 4vw;
}
```

ブラウザでズームすると、CSSピクセルで見たビューポート幅は狭くなります。`vw` で指定した文字も小さく再計算されるため、ズームによる拡大が相殺される場合があります。

W3Cは、ビューポート単位の不適切な使用によって文字を200%まで拡大できない状態を、WCAG 2.2の達成基準1.4.4「テキストのサイズ変更」に対する失敗例として示しています。

`clamp(1rem, 4vw, 3rem)` のように `rem` を含めても、上限によって200%まで拡大できない可能性があります。単位の組み合わせだけで判断せず、実際に文字サイズ変更とブラウザズームを試す必要があります。

### フォントサイズに関係するWCAGの達成基準

WCAG 2.2は、本文を必ず `16px` 以上にするといった最小フォントサイズを定めていません。特定の単位を使うこと自体も達成条件ではなく、ユーザーが文字を拡大したり間隔を変更したりしても、コンテンツと機能を利用できることが重要です。

フォントサイズを設計・実装するときは、主に次の達成基準を確認します。

#### 1.4.4 テキストのサイズ変更（レベルAA）

キャプションと画像化された文字を除き、支援技術を使わずに文字を200%まで拡大しても、コンテンツや機能が失われないことを求めています。

確認するのは、文字が大きく見えるかだけではありません。

- 文字が切れたり、ほかの要素と重なったりしない
- ボタンや入力欄のラベルを読める
- 拡大後もすべての操作を実行できる
- ブレークポイントが切り替わっても、元の文字サイズの200%まで拡大できる

`vw` や `cqw` による拡縮がズームを相殺し、200%まで拡大できなければ、この達成基準を満たせない可能性があります。

#### 1.4.10 リフロー（レベルAA）

コンテンツを拡大したときに、上下と左右の両方へスクロールしなければ読めない状態を避けるための基準です。横書きのコンテンツでは、幅320 CSS px相当まで狭くしても、原則として横スクロールなしで読めることが求められます。一般的な1280 CSS px幅の画面を400%にズームした状態が目安です。

長い文字列、固定幅・固定高、複数カラム、表やコードブロックなどを確認します。表や地図のように二次元配置が意味の理解に不可欠なコンテンツには例外がありますが、ページ全体を横スクロールさせる理由にはできません。

#### 1.4.12 テキストの間隔（レベルAA）

ユーザーが次の値へ文字間隔を変更しても、コンテンツや機能が失われないことを求めています。

- 行の高さ：フォントサイズの1.5倍以上
- 段落後の間隔：フォントサイズの2倍以上
- 文字間隔：フォントサイズの0.12倍以上
- 単語間隔：フォントサイズの0.16倍以上

これは、制作者が常にこれらの値を指定するという意味ではありません。ユーザーが独自のスタイルで上書きしたときに、文字の欠けや重なり、操作不能が起きないようにする基準です。

日本語では単語間の空白を通常使用しないなど、言語や文字体系によって適用できない項目があります。適用される文字間隔や行間を変更して確認します。

#### 1.4.3 コントラスト（最低限）（レベルAA）

通常の文字には4.5:1以上、大きな文字には3:1以上のコントラスト比が必要です。ここでいう大きな文字の目安は18pt以上、または14pt以上の太字です。

画面幅に応じて文字サイズを変える場合、一部の幅で「大きな文字」の条件を下回ることがあります。最小サイズの状態でも必要なコントラスト比を満たすように設計すると安全です。

これらをまとめると、フォントサイズの実装では「何pxにするか」だけでなく、200%の文字拡大、400%ズーム時のリフロー、文字間隔の上書き、最小状態でのコントラストを一緒に検証する必要があります。

### 使用する場面を限定する

画面幅とフォントサイズの連動は、次のような場面に限定します。

- すでに `vw` だけで実装されているサイトを段階的に改善する
- キービジュアルなど、画面全体との比率を保たなければデザインが成立しない
- 埋め込みウィジェットなど、コンポーネント幅と内部要素の比率を保つ必要がある

本文、フォーム、ナビゲーション、主要な操作部品など、読みやすさや操作性に直接関わる要素には、幅と連動しない `rem` を優先します。

また、`rem` を使うだけでアクセシビリティが保証されるわけではありません。文字を拡大したときに、固定高の要素や `overflow: hidden` によって文字が欠けないか、要素同士が重ならないかも確認します。

## 実装方法

ここからは、どうしても画面幅との連動が必要な場合の実装例を紹介します。

どちらの方法でも、フォントサイズそのものを `vw` にするのではなく、画面幅から単位なしのスケール係数を作り、`rem` に掛けます。

### JS+CSS

JavaScriptで `window.innerWidth` を取得し、計算したスケール係数をCSSカスタムプロパティへ設定します。

次の例では、幅320pxから767pxまでを `0.9375` から `1`、768pxから1200pxまでを `1` から `1.125` の範囲で補間します。ブレークポイントをまたいだときに、フォントサイズが急変しない値にしています。

```js
(() => {
  const root = document.documentElement;

  const clamp = (min, value, max) => {
    return Math.min(Math.max(value, min), max);
  };

  const interpolate = (width, minWidth, maxWidth, minScale, maxScale) => {
    const progress = clamp(
      0,
      (width - minWidth) / (maxWidth - minWidth),
      1
    );

    return minScale + (maxScale - minScale) * progress;
  };

  const getFontScale = (width) => {
    if (width <= 767) {
      return interpolate(width, 320, 767, 0.9375, 1);
    }

    return interpolate(width, 768, 1200, 1, 1.125);
  };

  let frameId = 0;

  const updateFontScale = () => {
    frameId = 0;
    const fontScale = getFontScale(window.innerWidth);
    root.style.setProperty("--font-scale", String(fontScale));
  };

  const requestUpdate = () => {
    if (frameId !== 0) return;
    frameId = requestAnimationFrame(updateFontScale);
  };

  updateFontScale();
  window.addEventListener("resize", requestUpdate, { passive: true });
})();
```

`resize` イベントは連続して発生するため、`requestAnimationFrame()` を使って1フレームにつき1回だけ更新します。

CSS側では、初期値として `--font-scale: 1` を用意し、基準となる `rem` にスケール係数を掛けます。

```css
:root {
  --font-scale: 1;
}

.visual-heading {
  font-size: calc(2rem * var(--font-scale));
}
```

この例では、基準のフォントサイズはあくまで `2rem` です。JavaScriptが実行できない場合はスケール係数が `1` のままになるため、幅との連動だけが無効になり、文字は通常の `rem` サイズで表示されます。

画面幅、ブレークポイント、最小値、最大値は案件に合わせて変更します。変化量は必要最小限に抑え、ズーム時の文字拡大をできるだけ妨げないようにします。

なお、ブラウザズームによって `window.innerWidth` が変わると、スケール係数も再計算されます。`rem` を使っていてもズームの効果を一部相殺する可能性は残るため、実機での検証は必要です。

### CSSのみ

CSSだけでも、`@property` と `tan(atan2())` を使ってビューポート幅から単位なしの数値を作れます。

#### なぜ `@property` と `tan(atan2())` が必要なのか

`calc()` の中で `rem` と `vw` を足し引きすること自体は可能です。

```css
font-size: calc(1rem + 1vw);
```

一方、画面幅を基準幅で割った「単位なしのスケール係数」を作るには、長さの単位を取り除く必要があります。新しいCSS Typed Arithmeticでは `calc(100vw / 1px)` と書けますが、2026年時点ではブラウザ間の対応に差があります。

そこで、`@property` で `100vw` を計算済みの長さとして保持し、`tan(atan2())` で比率を単位なしの数値へ変換します。

```css
@property --viewport-width {
  syntax: "<length>";
  initial-value: 0px;
  inherits: false;
}

:root {
  --viewport-width: 100vw;

  /*
   * 100vwを計算値のpxへ変換し、単位なしの数値にする。
   * 1pxとの極端な比率を避けるため、10000pxを基準にする。
   */
  --viewport-width-number:
    calc(10000 * tan(atan2(var(--viewport-width), 10000px)));
}
```

`@property` によって、`--viewport-width` は `<length>` 型として解決されます。`atan2()` が2つの長さの比から角度を求め、`tan()` がその比を単位なしの数値として取り出します。

理論上は次のようにも書けます。

```css
--viewport-width-number:
  tan(atan2(var(--viewport-width), 1px));
```

ただし、大きな画面幅を `1px` と比較すると角度が90度に近づき、ブラウザ実装の精度や相互運用性の影響を受けやすくなります。`10000px` を基準にして、最後に10000倍する形のほうが安定します。

JavaScript版と同じ範囲でスケールさせる場合は、次のように記述します。

```css
:root {
  /* PC: 768pxで1、1200pxで1.125 */
  --font-scale: clamp(
    1,
    calc(
      1
      + (var(--viewport-width-number) - 768)
      * (0.125 / 432)
    ),
    1.125
  );
}

@media (width <= 767px) {
  :root {
    /* SP: 320pxで0.9375、767pxで1 */
    --font-scale: clamp(
      0.9375,
      calc(
        0.9375
        + (var(--viewport-width-number) - 320)
        * (0.0625 / 447)
      ),
      1
    );
  }
}

.visual-heading {
  font-size: calc(2rem * var(--font-scale));
}
```

CSS版とJavaScript版をフォールバックとして併用する場合は、ブレークポイント、最小値、最大値、計算式を揃えます。対象ブラウザに応じて、未対応環境では `--font-scale: 1` の固定表示へフォールバックする方法もあります。

#### `cqw` でも使用できる？

使用できます。`cqw` も長さなので、`@property` で `<length>` として登録し、同じ方法で単位なしの数値へ変換できます。

まず、祖先要素をクエリコンテナにします。

```css
.card-list {
  container-type: inline-size;
}
```

次に、`100cqw` からコンテナ幅を取得します。

```css
@property --container-width {
  syntax: "<length>";
  initial-value: 0px;
  inherits: false;
}

.card {
  --container-width: 100cqw;
  --container-width-number:
    calc(10000 * tan(atan2(var(--container-width), 10000px)));

  /* 320pxで1、640pxで1.125 */
  --card-font-scale: clamp(
    1,
    calc(
      1
      + (var(--container-width-number) - 320)
      * (0.125 / 320)
    ),
    1.125
  );
}

.card__title {
  font-size: calc(1.5rem * var(--card-font-scale));
}
```

`cqw` は、最も近い適格なクエリコンテナの幅の1%です。`100cqw` でコンテナ幅全体を表します。

使用時には、次の点に注意します。

- 対象要素自身ではなく、祖先要素に `container-type` を指定する
- 適格なコンテナがない場合、その軸のsmall viewport単位にフォールバックする
- 縦書きも考慮する場合は、物理的な幅を表す `cqw` ではなく、インライン方向を表す `cqi` を検討する
- コンテナ幅との連動でも、文字拡大やズームを相殺する可能性は残る

コンポーネントの配置場所によって幅が変わる場合は、ビューポート基準の `vw` よりも `cqw` や `cqi` のほうが意図を表しやすく、再利用しやすくなります。

## まとめ

- フォントサイズは、画面幅と連動しない `rem` 指定を基本にする
- 幅ごとの調整が必要な場合は、まずメディアクエリによる段階的な切り替えを検討する
- 滑らかな拡縮は、既存の `vw` 実装の改善や、比率維持が不可欠なレイアウトに限定する
- JavaScriptでは、画面幅から計算した係数をCSS変数へリアルタイムに設定できる
- `@property` と `tan(atan2())` を使えば、CSSだけでも単位なしのスケール係数を作れる
- 同じ方法は `cqw` や `cqi` にも使えるが、祖先のクエリコンテナが必要になる
- `rem` と組み合わせてもズームを相殺する可能性は残るため、文字の200%拡大とブラウザズームで確認する

可変フォントスケールは、通常のタイポグラフィに代わるものではありません。制約の強いデザインを改善するための、限定的な手段として使うのが安全です。

## 参考資料

- [Understanding Success Criterion 1.4.4: Resize Text | W3C WAI](https://www.w3.org/WAI/WCAG22/Understanding/resize-text)
- [Understanding Success Criterion 1.4.10: Reflow | W3C WAI](https://www.w3.org/WAI/WCAG22/Understanding/reflow)
- [Understanding Success Criterion 1.4.12: Text Spacing | W3C WAI](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing)
- [Understanding Success Criterion 1.4.3: Contrast (Minimum) | W3C WAI](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum)
- [F94: Failure due to incorrect use of viewport units to resize text | W3C WAI](https://www.w3.org/WAI/WCAG22/Techniques/failures/F94)
- [@property | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/%40property)
- [atan2() | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/atan2)
- [tan() | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/tan)
- [CSS container queries | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Containment/Container_queries)
- [Using CSS typed arithmetic | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Values_and_units/Using_typed_arithmetic)