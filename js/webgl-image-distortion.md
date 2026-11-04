# グニョンとする画像歪み演出はどう実装するのか

## 調べたきっかけ

- マウスに追従して画像が表示される
- マウスを動かすと、画像が進行方向に合わせてグニョンと歪む
- 画像そのものが柔らかい板のように曲がって見える
- この演出がどう実装されているのか気になった

調べた対象は、Olivier Larose氏の「Mouse Image Distortion」というデモである

> 参照: [Mouse Image Distortion](https://blog.olivierlarose.com/tutorials/mouse-image-distortion) / [GitHub — olivierlarose/mouse-image-distortion](https://github.com/olivierlarose/mouse-image-distortion) / [Live Demo](https://blog.olivierlarose.com/demos/mouse-image-distortion)

## 先に結論

- 元実装はWebGLを使っている
- React Three FiberでThree.jsのシーンを作り、画像をDOMの `<img>` ではなく3D空間上の平面メッシュに貼っている
- 歪みはフラグメントシェーダで画像のUVをずらしているのではなく、頂点シェーダで平面メッシュの頂点座標を動かしている
- マウス位置と、少し遅れて追従するマウス位置の差分を使い、「マウスの勢い」を歪み量にしている
- WebGLなしでも近い演出は作れるが、同じ自由度・滑らかさで再現するならWebGLが自然
- WebGLなしで試すなら、SVGの `feDisplacementMap` が最有力

## 元実装で使われているもの

- `next`
- `react`
- `@react-three/fiber`
- `@react-three/drei`
- `framer-motion`
- `framer-motion-3d`
- `lenis`

`@react-three/fiber` と `@react-three/drei` が入っているため、描画の中心はThree.jsである

> 参照: [GitHub — package.json](https://github.com/olivierlarose/mouse-image-distortion/blob/main/package.json)

## 画像はDOMではなくメッシュに貼られている

元実装では、画像を通常の `<img>` として表示しているのではなく、Three.jsの `planeGeometry` にテクスチャとして貼っている

簡略化すると、構造は次のようなものになる

```jsx
<motion.mesh position-x={x} position-y={y} scale={scale}>
  <planeGeometry args={[1, 1, 15, 15]} />
  <shaderMaterial
    vertexShader={vertex}
    fragmentShader={fragment}
    uniforms={uniforms}
    transparent
  />
</motion.mesh>
```

重要なのは `planeGeometry args={[1, 1, 15, 15]}` の部分である

- 幅1、高さ1の平面を作る
- その平面を横15、縦15に分割する
- 分割された頂点をシェーダで動かす
- その結果、画像を貼った板が曲がっているように見える

分割数がなければ、四隅の頂点しか動かせないため、滑らかな「しなり」にはなりにくい

## マウスの勢いを作っている

この演出では、マウスの現在位置をそのまま画像位置に使っているだけではない

元実装では次の2つの座標を持っている

- 実際のマウス位置
- 少し遅れて追従するマウス位置

この2つの差分を取ることで、マウスの移動量、つまり「勢い」を作っている

簡略化すると次のような処理である

```js
const lerp = (from, to, amount) => {
  return from * (1 - amount) + to * amount;
};

smoothX = lerp(smoothX, mouseX, 0.1);
smoothY = lerp(smoothY, mouseY, 0.1);

const delta = {
  x: mouseX - smoothX,
  y: -1 * (mouseY - smoothY),
};
```

`lerp()` は線形補間である

- `mouseX` は現在のマウス位置
- `smoothX` は少し遅れて追いかける位置
- `0.1` は追従の強さ
- 差分が大きいほど、マウスが速く動いているとみなせる

この `delta` がシェーダ側の `uDelta` に渡される

> 参照: [GitHub — Model.jsx](https://github.com/olivierlarose/mouse-image-distortion/blob/main/src/components/Model.jsx)

## 歪みの本体は頂点シェーダ

歪みの核心は `Shader.js` の頂点シェーダである

簡略化すると次のような処理になっている

```glsl
varying vec2 vUv;
uniform vec2 uDelta;
uniform float uAmplitude;

float PI = 3.141592653589793238;

void main() {
  vUv = uv;

  vec3 newPosition = position;
  newPosition.x += sin(uv.y * PI) * uDelta.x * uAmplitude;
  newPosition.y += sin(uv.x * PI) * uDelta.y * uAmplitude;

  gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition, 1.0);
}
```

やっていることは、平面メッシュの頂点座標を少し動かすことである

- `position` は元の頂点座標
- `uv` は画像上の位置を0〜1で表す座標
- `uDelta` はマウスの勢い
- `uAmplitude` は歪みの強さ
- `sin()` は歪ませ方のカーブを作る

`sin(uv.y * PI)` は、上端と下端で0、中央付近で1に近くなる

つまり横方向の歪みは、画像の上下端では小さく、中央で大きくなる

同じように、`sin(uv.x * PI)` は左端と右端で0、中央付近で1に近くなる

つまり縦方向の歪みは、画像の左右端では小さく、中央で大きくなる

> 参照: [GitHub — Shader.js](https://github.com/olivierlarose/mouse-image-distortion/blob/main/src/components/Shader.js)

## 画像のピクセルを直接歪ませているわけではない

最初は「画像のピクセルをシェーダで曲げている」のだと思っていた

しかし、この実装ではフラグメントシェーダ側の処理はかなり素直である

```glsl
varying vec2 vUv;
uniform sampler2D uTexture;
uniform float uAlpha;

void main() {
  vec3 texture = texture2D(uTexture, vUv).rgb;
  gl_FragColor = vec4(texture, uAlpha);
}
```

フラグメントシェーダでは、`vUv` の位置からテクスチャを読んで色を出しているだけである

そのため、この演出の正体は次のように整理できる

- 画像を1枚の平面メッシュに貼る
- 平面メッシュを細かく分割する
- マウスの勢いに応じて頂点を動かす
- 頂点の間はGPUが補間する
- 結果として、画像が柔らかく曲がって見える

「画像処理」というより「画像を貼った板の変形」に近い

## なぜWebGLだと実装しやすいのか

WebGLでは、画像をテクスチャとして扱い、頂点やピクセルに対してGPU上で計算できる

今回のような演出では、次の点がWebGLと相性がよい

- メッシュの頂点を毎フレーム動かせる
- `sin()` などの数式でなめらかな歪みを作れる
- マウス位置、時間、スクロール量などを `uniform` としてシェーダに渡せる
- GPUで処理するため、細かい分割や毎フレーム更新に向いている
- 画像をテクスチャとして扱うため、DOMレイアウトから切り離して表現できる

CodropsのWebGLによるバルジ歪みの記事でも、画像テクスチャの座標やマウス位置を使って歪みを作る考え方が紹介されている

> 参照: [Codrops — Creating a Bulge Distortion Effect with WebGL](https://tympanus.net/codrops/2023/06/28/creating-a-bulge-distortion-effect-with-webgl/)

## WebGLなしで同じことはできるのか

結論として、完全に同じではないが、近い演出は作れる

ただし「何をもって同じとするか」で選択肢が変わる

- 要素全体が傾く程度でよい — CSS transform
- 画像のピクセルが揺らぐ感じがほしい — SVG filter
- 画像をスライスして疑似的に曲げたい — Canvas 2D
- マウス速度に応じてなめらかにメッシュ変形したい — WebGL

## 方法1: CSS transformでそれっぽくする

もっとも軽い方法は、画像要素に `translate`, `skew`, `scale`, `rotate` をかける方法である

```css
.cursor-image {
  transform:
    translate(var(--x), var(--y))
    skewX(var(--skew-x))
    skewY(var(--skew-y))
    scale(var(--scale));
  transition: transform 120ms ease-out;
}
```

マウスの移動量から `--skew-x` や `--skew-y` を更新すれば、画像が引っ張られるような印象は作れる

ただし、この方法には限界がある

- 要素全体の四角形を変形しているだけ
- 画像の中央だけが曲がるような表現は難しい
- 端が固定されて中央がしなるような動きにはならない
- 実装は軽いが、WebGLのメッシュ変形とは別物

「グニョン」というより「ビュッと傾く」演出に近い

## 方法2: SVG filterのfeDisplacementMapを使う

WebGLなしで画像そのものを歪ませたい場合は、SVG filterの `feDisplacementMap` が候補になる

`feDisplacementMap` は、別の画像やノイズのピクセル値をもとに、元の画像のピクセル位置をずらすSVGフィルタである

MDNでは次の式で説明されている

```txt
P'(x,y) ← P(x + scale * (XC(x,y) - 0.5), y + scale * (YC(x,y) - 0.5))
```

ざっくり言うと、次のような仕組みである

- `SourceGraphic` が歪ませたい元画像
- `in2` が変位の地図になる画像
- `scale` がずらす強さ
- `xChannelSelector` がX方向の変位に使う色チャンネル
- `yChannelSelector` がY方向の変位に使う色チャンネル

`feTurbulence` と組み合わせると、ノイズを変位マップとして使える

```html
<svg width="0" height="0" aria-hidden="true">
  <filter id="distortion">
    <feTurbulence
      type="fractalNoise"
      baseFrequency="0.02"
      numOctaves="1"
      result="noise"
    />
    <feDisplacementMap
      in="SourceGraphic"
      in2="noise"
      scale="0"
      xChannelSelector="R"
      yChannelSelector="G"
    />
  </filter>
</svg>

<img class="cursor-image" src="/image.jpg" alt="" />
```

```css
.cursor-image {
  filter: url("#distortion");
}
```

JavaScriptでマウスの移動量に応じて `scale` を変えれば、動いた瞬間だけ歪ませることができる

```js
const displacement = document.querySelector('feDisplacementMap');

let previousX = 0;
let previousY = 0;
let currentScale = 0;
let targetScale = 0;

window.addEventListener('mousemove', (event) => {
  const deltaX = event.clientX - previousX;
  const deltaY = event.clientY - previousY;
  const distance = Math.hypot(deltaX, deltaY);

  targetScale = Math.min(distance * 0.6, 40);

  previousX = event.clientX;
  previousY = event.clientY;
});

function tick() {
  currentScale += (targetScale - currentScale) * 0.2;
  targetScale *= 0.9;

  displacement.setAttribute('scale', currentScale.toFixed(2));
  requestAnimationFrame(tick);
}

tick();
```

この方法は、WebGLなしでDOM要素にピクセル単位の歪みをかけられる点が強い

一方で、元実装のような「マウス方向に沿って板がしなる」動きとは少し性質が違う

- ノイズ由来の歪みになりやすい
- 水面、熱揺らぎ、グリッチのような質感に寄りやすい
- 方向性のあるしなりを細かく制御するには工夫が必要
- フィルタの更新は重くなる場合があるため、強度や適用範囲を抑える必要がある

> 参照: [MDN — feDisplacementMap](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feDisplacementMap) / [MDN — feTurbulence](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feTurbulence) / [Codrops — Distorted Button Effects with SVG Filters](https://tympanus.net/codrops/2016/05/11/distorted-button-effects-with-svg-filters/)

## 方法3: Canvas 2Dでスライスして歪ませる

Canvas 2Dでも疑似的な歪みは作れる

考え方は、画像を細い短冊に分けて、それぞれの描画位置を少しずつずらす方法である

```js
function drawDistortedImage(ctx, image, deltaX) {
  const sliceCount = 30;
  const sliceHeight = image.height / sliceCount;

  ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);

  for (let i = 0; i < sliceCount; i++) {
    const y = i * sliceHeight;
    const progress = i / (sliceCount - 1);
    const wave = Math.sin(progress * Math.PI);
    const offsetX = wave * deltaX * 0.05;

    ctx.drawImage(
      image,
      0,
      y,
      image.width,
      sliceHeight,
      offsetX,
      y,
      image.width,
      sliceHeight
    );
  }
}
```

この方法でも、中央ほど大きくずれる表現は作れる

ただし、WebGLの頂点シェーダとは違い、自分で分割と再描画を管理する必要がある

- DOM画像ではなくCanvasに描く必要がある
- 毎フレーム `drawImage()` を繰り返す
- 縦方向と横方向を同時に自然に歪ませるのは難しい
- 画像の拡大縮小、Retina対応、レスポンシブ対応がやや面倒

軽い実験には向いているが、本格的なマウス追従の歪み演出ならWebGLの方が扱いやすい

## CSS、SVG、Canvas、WebGLの使い分け

CSS transform

- 実装がもっとも簡単
- DOMのまま扱える
- 低コストで動かせる
- ピクセル単位の歪みではなく、要素全体の変形になる

SVG filter

- WebGLなしでピクセル単位の変位ができる
- DOM要素に `filter` として適用できる
- `feTurbulence` と組み合わせると有機的な揺らぎを作りやすい
- 方向性のあるメッシュ変形は制御しにくい

Canvas 2D

- 画像を自分で描画・加工できる
- スライス方式で疑似的な波打ちを作れる
- DOMレイアウトから外れる
- 状態管理と再描画の実装量が増える

WebGL

- 頂点やUVをシェーダで制御できる
- マウス速度、時間、スクロール量を歪みに反映しやすい
- GPUで処理できる
- 実装の学習コストは高い

## アクセシビリティ上の注意

マウス追従や画像の歪みは、装飾としては楽しいが、動きが強い演出でもある

実装する場合は、`prefers-reduced-motion` への配慮を入れたい

```css
@media (prefers-reduced-motion: reduce) {
  .cursor-image {
    transition: opacity 120ms ease-out;
    transform: translate(var(--x), var(--y));
    filter: none;
  }
}
```

- モーション軽減を希望するユーザーには、歪みや大きな追従遅延を切る
- 画像表示が必要な場合は、フェード程度に抑える
- 装飾目的なら、演出自体を無効化してもよい

## まとめ

- Olivier Larose氏のMouse Image Distortionは、React Three Fiberを使ったWebGL実装である
- 画像はDOMの `<img>` ではなく、Three.jsの平面メッシュにテクスチャとして貼られている
- マウスの現在位置と、遅れて追従する位置の差分から「勢い」を作っている
- 歪みの本体は頂点シェーダで、`sin()` を使って平面メッシュの中央付近が大きく動くようにしている
- フラグメントシェーダで画像のピクセルを直接歪ませているのではなく、画像を貼った板を曲げている
- WebGLなしなら、CSS transform、SVG filter、Canvas 2Dで近い表現は作れる
- 元実装に近いピクセル単位・方向性あり・滑らかな歪みを作るならWebGLが自然
- WebGLなしでDOM画像を歪ませるなら、`feDisplacementMap` が実用的な候補になる
