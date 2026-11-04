# Scroll Image Distortionを制作 — スクロールに応じてカード画像を歪ませる

## Scroll Image Distortionとは

- Scroll Image Distortionは、スクロールの速さに応じてカード一覧の画像が上下に歪むUIデモである<sup><a href="#ref-1">[1]</a></sup>
- スクロールすると画像が曲がり、操作を止めると徐々に元の形へ戻る — 上下のスクロール方向によって歪む向きも変わる
- カードは通常のHTMLで並べ、画像の見た目をWebGLで重ねて描画する構成にした
- 使用技術はAstro、three.js、GLSLシェーダー、Tweakpaneである — ページ生成、画像の描画、頂点の変形、パラメータ調整を分担させている<sup><a href="#ref-2">[2]</a></sup>
- 歪みの式はマウス追従版の`Mouse_Image_Distortion_02`をベースにし、入力をマウスの移動差分からスクロールの差分へ変更した
- 同じシェーダーでも、何の値で動かすかを変えると、ホバー中の表現からページ全体のスクロールに連動する表現へ展開できる

<ol start="1">
  <li id="ref-1"><a href="https://scroll-image-distortion.vercel.app/">Scroll Image Distortion — 公開サイト</a></li>
  <li id="ref-2"><a href="https://github.com/tktkaws/scroll_image_distortion">Scroll Image Distortion — GitHubとREADME</a></li>
</ol>

## HTMLのカードにWebGLの画像を重ねる

- Astroでは、`src/scripts/data.js`の配列からカード一覧を生成する — カード全体を`<a>`にし、画像とタイトルを配置している
- 一覧のレイアウトはCSS Gridで作り、通常は2列、狭い画面では1列に切り替える
- WebGL用の`#webgl-stage`は画面全体へ固定配置し、その中に1枚の`<canvas>`を用意する — カードごとにCanvasを増やす構成にはしていない
- 各カードの`.card-media`に対応する平面のメッシュを作り、`getBoundingClientRect()`で取得した位置とサイズへ合わせる<sup><a href="#ref-3">[3]</a></sup>
- HTMLの座標は画面左上を基準にする一方、three.jsではカメラから見える範囲の座標へ変換する必要がある — 画像領域の中心を`mesh.position`、幅と高さを`mesh.scale`へ反映する
- スクロールとリサイズでHTMLの位置が変わっても、毎フレームの同期によってWebGLの画像を追従させる
- 画面外の画像に対応するメッシュは`visible = false`にし、画面内にある画像だけを表示する
- Canvasを含むオーバーレイには`pointer-events: none`を指定するため、ポインターによる操作は下のHTMLへ届く — 画像をWebGLで描いても、リンクはHTML側に残せる
- HTMLの`<img>`も配置し、WebGL用テクスチャの読み込みが完了したら`opacity: 0`にする — 要素を取り除かず、レイアウトを維持したまま表示を切り替える
- テクスチャの読み込みに失敗した場合は、WebGL側に用意したプレースホルダーを使う

<ol start="3">
  <li id="ref-3"><a href="https://github.com/tktkaws/scroll_image_distortion/blob/master/src/scripts/main.js">Scroll Image Distortion — メッシュ生成とDOM位置への同期</a></li>
</ol>

## スクロール差分を歪みの入力にする

- 歪みの入力は、スクロール位置そのものではなく、前回の`scroll`イベントからどれだけ移動したかという差分である
- 同じ位置にいても直前の移動量が異なれば歪み方が変わる — ページの進行位置で演出を決める方式とは異なる
- サイト上では「スクロール速度」と表現しているが、実装では経過時間で割った速度を求めていない — `window.scrollY`の差分を使い、動きの勢いを表している<sup><a href="#ref-4">[4]</a></sup>
- 取得した差分をそのまま渡すと急に変形するため、直前の値との線形補間で変化を滑らかにする
- 次のコードは、差分の取得、補間、上限の処理を抜き出した例である — 描画と、停止後に歪みを戻す処理は含めていない

```js
let previousScrollY = window.scrollY;
let smoothScrollDelta = 0;
const maxDelta = 20;

window.addEventListener('scroll', () => {
  const currentScrollY = window.scrollY;
  const rawDelta = currentScrollY - previousScrollY;
  previousScrollY = currentScrollY;

  // 新しい差分を35%反映し、入力を-20〜20へ収める
  const nextDelta = smoothScrollDelta * 0.65 + rawDelta * 0.35;
  smoothScrollDelta = Math.max(-maxDelta, Math.min(maxDelta, nextDelta));
}, { passive: true });
```

- `maxDelta`は、シェーダーへ渡す差分が大きくなりすぎないようにする上限である — スクロールそのものの移動量を制限する値ではない
- 描画側では差分の符号を反転し、`uDelta`のY成分へ渡すことで歪みの向きを決めている
- アニメーション処理では差分を減衰させ、十分に小さくなった後は`uDelta`自体を減衰させる — スクロールを止めた瞬間に変形を切らず、元の形へ戻している
- カードをクリックした場合はページ先頭へスムーズスクロールし、その際に差分の状態をリセットする — 現在のデモでは、外部ページへの移動を示すリンクとしては使っていない

<ol start="4">
  <li id="ref-4"><a href="https://github.com/tktkaws/scroll_image_distortion/blob/master/src/scripts/main.js">Scroll Image Distortion — スクロールイベントとアニメーション処理</a></li>
</ol>

## 頂点シェーダーで平面を曲げる

- 画像は、テクスチャを貼った平面としてthree.jsのシーンへ配置する
- 平面には`PlaneGeometry(1, 1, 15, 15)`を使い、幅と高さの方向をそれぞれ15分割している — 途中の頂点も動かせるようにし、曲がりを表現する
- 変形はGLSL（GPU上の描画処理を記述する言語）の頂点シェーダーで行う — 画像の画素を直接書き換える処理ではなく、画像を貼った平面の頂点を動かす<sup><a href="#ref-5">[5]</a></sup>
- 頂点位置の変形部分を抜き出すと、次の2行になる — `newPosition`は変形する頂点位置、`uv`は画像上の0〜1の座標である

```glsl
newPosition.x += sin(uv.y * PI) * uDelta.x * uAmplitude;
newPosition.y += sin(uv.x * PI) * uDelta.y * uAmplitude;
```

- このデモでは主に`uDelta.y`を更新するため、Y方向の変形が生まれる
- `sin(uv.x * PI)`は左右の端で0、中央で1になる — 左右の端を大きく動かさず、中央ほど上下へ曲げる形になる
- `uDelta.y`の符号が変われば、中央が曲がる方向も反転する — 上下のスクロールで異なる反応を作れる
- `uAmplitude`は変形量に掛ける倍率であり、同じスクロール差分でも値を変えると歪みの強さが変わる
- フラグメントシェーダーでは、UV座標に対応するテクスチャの色を取得して描画する — 頂点側で形を変え、画像自体はテクスチャとして表示している

<ol start="5">
  <li id="ref-5"><a href="https://github.com/tktkaws/scroll_image_distortion/blob/master/src/scripts/shaders.js">Scroll Image Distortion — 頂点・フラグメントシェーダー</a></li>
</ol>

## Tweakpaneで動きを調整する

- 画面右上には、Tweakpaneによるパラメータ調整UIを配置している — コードを書き換えず、動かしながら歪み方を確認できる<sup><a href="#ref-6">[6]</a></sup>
- `maxDelta`の初期値は`20`で、スクロール差分の上限を調整する
- `amplitude`の初期値は`0.002`で、シェーダー側の変形倍率を調整する
- `maxDelta`を小さくすると、強くスクロールした場合も入力が早く上限へ達する — 大きな入力に対する歪みを抑える調整になる
- `amplitude`を小さくすると、上限に達していない小さなスクロールも含め、変形全体が弱くなる
- 歪みの向きを変えたい場合は、`uDelta`へ渡す符号を変更する — 強さの調整と、向きの調整は分けて扱う
- カードを追加する場合は`src/scripts/data.js`へタイトル、画像パス、リンクを追加し、画像を`public/images/`へ置く
- 画像やカードの数を変えた後は、スクロール中の位置のずれ、狭い画面でのレイアウト、停止後の歪みの戻り方を確認する
- HTMLでカードを並べる部分と、WebGLで変形させる部分を分けたことで、一覧の構成と歪みの設定をそれぞれ調整できる

<ol start="6">
  <li id="ref-6"><a href="https://github.com/tktkaws/scroll_image_distortion#5-tweakpane">Scroll Image Distortion README — Tweakpaneの調整項目</a></li>
</ol>

## まとめ

- Scroll Image Distortionは、スクロールの移動差分によってカード画像を上下に歪ませるUIデモである
- AstroでカードのHTMLを生成し、固定した1枚のCanvasでWebGLの画像を重ねて描画している
- DOMの位置とサイズを計測し、対応するメッシュを同期させることで、スクロール中も画像を追従させる
- スクロール差分には補間と上限を設け、描画側の減衰で元の形へ戻す
- GLSLの頂点シェーダーで平面を曲げ、スクロール方向に応じて歪む向きを変える
- Tweakpaneでは入力上限の`maxDelta`と、変形倍率の`amplitude`を調整できる
- マウス追従版のシェーダーを流用し、入力を変えることでスクロールに連動する表現へ展開した
