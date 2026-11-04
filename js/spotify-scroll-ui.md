# Spotify風のUIを制作 — GSAPでカバーの縮小とヘッダーの切り替えを作る

## Spotify風のUIとは

- Spotifyの画面を参考に、スクロールに合わせてカバー画像が小さくなり、コンパクトなヘッダーへ切り替わるUIを制作した<sup><a href="#ref-1">[1]</a></sup>
- 最初はカバー画像を大きく表示し、スクロールが進むと縮小させる構成
- 縮小後はカバー画像をフェードアウトさせ、ヘッダーをフェードインさせる
- カバー画像から取り出した代表色を背景グラデーションにも使い、画像と背景の色をそろえる
- 使用技術はAstro、GSAPのScrollTrigger、Canvas API — スクロールに連動する動きと画像の色抽出を組み合わせた実装である<sup><a href="#ref-2">[2]</a></sup>

<ol start="1">
  <li id="ref-1"><a href="https://spotify-ui-six.vercel.app/">制作したSpotify風UIのデモ</a></li>
  <li id="ref-2"><a href="https://front-dispatch.vercel.app/blog/2zw22mgth">元記事 — GSAPでSpotify風のUIを作ってみた</a></li>
</ol>

## ScrollTriggerでカバー画像を縮小する

- ScrollTriggerは、スクロール位置を基準にアニメーションを制御するGSAPのプラグインである
- `pin: true`でカバー画像を含むヒーロー領域を固定し、`scrub: true`でスクロール位置とアニメーションの進行を同期させる
- `start: 'top top'`は、ヒーロー領域の上端がスクロール領域の上端に来た位置で開始する指定
- `end`には開始位置からのスクロール距離を指定する — この区間をスクロールすると、カバー画像が元の大きさから縮小後の大きさへ変わる<sup><a href="#ref-3">[3]</a></sup>
- 変更するのはカバー画像のラッパー要素の`width`と`padding-top` — 幅と上側の余白を同じ比率で縮める
- 縮小に使うスクロール距離は、`--content-max-width`に対する現在の幅の比率で調整する
- 以下は元記事の縮小処理からの抜粋 — GSAPとScrollTriggerの読み込み・登録は省略している
- `hero`は固定対象、`coverWrap`はラッパー要素で、`settings`には縮小率とスクロール距離を設定する — `responsiveScale`は幅に応じた比率である

```ts
const coverSize = coverWrap.getBoundingClientRect().width;
const coverPadding = Number.parseFloat(getComputedStyle(coverWrap).paddingTop);
const shrinkDistance = settings.shrinkDistance * responsiveScale;

const shrinkTimeline = gsap.timeline({
  defaults: { ease: 'none' },
  scrollTrigger: {
    trigger: hero,
    start: 'top top',
    end: `+=${shrinkDistance}`,
    scrub: true,
    pin: true,
    pinSpacing: false,
  },
});

shrinkTimeline.to(coverWrap, {
  width: coverSize * settings.minScale,
  paddingTop: coverPadding * settings.minScale,
  duration: shrinkDistance,
});
```

<ol start="3">
  <li id="ref-3"><a href="https://gsap.com/docs/v3/Plugins/ScrollTrigger/">GSAP — ScrollTrigger</a></li>
</ol>

## 縮小後にコンパクトヘッダーへ切り替える

- 縮小用とフェード用でタイムラインを分け、縮小が終わるスクロール位置をフェードの開始位置にする
- カバー画像を消す処理とヘッダーを表示する処理を重ね、スクロールに合わせて表示を切り替える
- フェードにはGSAPの`autoAlpha`を使用 — `opacity`と`visibility`をまとめて制御し、値が`0`になったときに`visibility: hidden`へ切り替える<sup><a href="#ref-4">[4]</a></sup>
- ヘッダーは`y: -8`の位置から`y: 0`へ移動させながら表示する
- フェードの進行率が`settings.headerReveal`に達したかどうかで、ヘッダーの`aria-hidden`も切り替える
- `aria-hidden="true"`は、要素とその子孫をアクセシビリティツリーから除外する指定 — スクリーンリーダーなどの支援技術が参照する情報から外す役割がある
- `aria-hidden`自体に、画面上の表示を消したりキーボード操作を無効にしたりする働きはない — フォーカス可能な要素や、その要素を含む祖先には`aria-hidden="true"`を付けない<sup><a href="#ref-5">[5]</a></sup>
- 縮小モーションを無効にした場合は、アニメーションを挟まずに表示と非表示を切り替える構成である

<ol start="4">
  <li id="ref-4"><a href="https://gsap.com/docs/v3/GSAP/CorePlugins/CSS/#autoalpha">GSAP — autoAlpha</a></li>
  <li id="ref-5"><a href="https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-hidden">MDN — aria-hidden</a></li>
</ol>

## Canvasでカバー画像の代表色を取り出す

- Canvas APIは、JavaScriptから画像や図形を描画するためのAPI — 今回はカバー画像の画素データを読み取るために使う
- 画像を48×48ピクセルのCanvasに縮小して描き、`getImageData()`で各画素のRGBAデータを取得する<sup><a href="#ref-6">[6]</a></sup>
- 画素データは、赤・緑・青・アルファ値の4要素が1組で並ぶ配列である
- 元記事のコードは配列の添字を`16`ずつ進める構成 — 4画素ごとに色を調べる
- アルファ値が`180`未満の画素は除外する
- RGBの最大値と最小値の平均を明るさの目安にし、`28`未満、または`232`を超える画素も除外する
- 残った画素は、RGBの各値を`32`で割って四捨五入した値で分類する — 近い色を同じグループにまとめる処理である
- 画素数が最も多いグループを選び、そのグループ内のRGBの平均を代表色とする
- 以下は分類に使うキーの計算部分の抜粋 — `r`、`g`、`b`は取得済みの画素のRGB値である

```ts
const key = `${Math.round(r / 32)}-${Math.round(g / 32)}-${Math.round(b / 32)}`;
```

<ol start="6">
  <li id="ref-6"><a href="https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/getImageData">MDN — CanvasRenderingContext2D.getImageData()</a></li>
</ol>

## 抽出した色を背景グラデーションへ反映する

- 代表色は、RGB値をカンマで区切った文字列としてCSSカスタムプロパティの`--accent-rgb`に設定する
- 背景側では`var()`で色を読み取り、`rgba()`で不透明度を指定して、黒へ変化するグラデーションに使う<sup><a href="#ref-7">[7]</a></sup>
- 色の抽出をJavaScript、グラデーションの描画をCSSで行う構成である
- 元の実装ではグラデーションの色が切り替わる位置も変数にし、カバー画像の縮小中に更新する — 画像の大きさの変化に合わせて背景も調整する
- 以下は色を背景に反映する部分だけを簡略化した例 — 初期色を設定しておき、画像から色を抽出した後に`--accent-rgb`を上書きする

```css
:root {
  --accent-rgb: 80, 120, 160;
}

.app-shell {
  background: linear-gradient(
    180deg,
    rgba(var(--accent-rgb), 0.8) 0%,
    #000 100%
  );
}
```

<ol start="7">
  <li id="ref-7"><a href="https://developer.mozilla.org/ja/docs/Web/CSS/Using_CSS_custom_properties">MDN — CSSカスタムプロパティの使用</a></li>
</ol>

## まとめ

- Astro、GSAPのScrollTrigger、Canvas APIを使ってSpotify風のUIを制作した
- `pin`でヒーロー領域を固定し、`scrub`でスクロール位置と縮小を同期させた
- ラッパー要素の`width`と`padding-top`を同じ比率で縮めた
- 縮小用とフェード用のタイムラインを分け、カバー画像からコンパクトヘッダーへ表示を切り替えた
- `autoAlpha`による見た目の制御と、`aria-hidden`による支援技術への表示制御は役割が異なる
- カバー画像を48×48ピクセルのCanvasに描き、近い色をまとめて代表色を選んだ
- 抽出した色をCSSカスタムプロパティに渡し、背景グラデーションへ反映した
