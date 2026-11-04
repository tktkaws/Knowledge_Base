# p5-learningを制作 — 動くスケッチと解説でp5.jsを学ぶサイト

## p5-learningとは

- p5.jsをテーマごとに学べるサイト「p5-learning」を制作した — サイト上では「p5.js Learning」として公開している<sup><a href="#ref-1">[1]</a></sup>
- p5.jsは、コードで絵やアニメーションなどを作るためのJavaScriptライブラリである<sup><a href="#ref-2">[2]</a></sup>
- p5.jsで作る描画プログラムをスケッチと呼び、このサイトでは動くスケッチと解説を同じページにまとめている
- トップページから学びたいテーマを選び、描画結果を見ながら関数の使い方や処理の仕組みを確認する構成
- サイトはAstro、学習ページはMDX、描画部分はp5.jsで実装している<sup><a href="#ref-3">[3]</a></sup>

<ol start="1">
  <li id="ref-1"><a href="https://p5-learning-tau.vercel.app/">p5.js Learning — 公開サイト</a></li>
  <li id="ref-2"><a href="https://p5js.org/">p5.js — 公式サイト</a></li>
  <li id="ref-3"><a href="https://github.com/tktkaws/p5_learning">p5_learning — GitHubリポジトリ</a></li>
</ol>

## 基本図形から3D表現までテーマ別に用意する

- 基本図形やアニメーションに加え、幾何学模様、ノイズ、物理シミュレーション、3D表現などを個別のページに分けた
- 「基本図形と色」では、円や四角形の描き方、塗りと線の色の指定を扱う
- 「アニメーション入門」では、`draw()`の繰り返しとフレームの仕組みを使って動きを作る
- 「マウスインタラクション」や「キーボードお絵かき」では、入力に応じて描画を変える処理を扱う
- 「Perlinノイズフロー」や「地形ジェネレーション」では、`noise()`を使った流れや地形の表現を扱う
- 「群れシミュレーション（Boids）」や「布シミュレーション」では、複数の要素の関係から動きを作る
- 「WEBGL入門」や「3D地形」では、立体の描画やカメラ操作を扱う
- トップページには各テーマの説明を添えたリンクを並べ、内容を確認してから学習ページへ進めるようにした<sup><a href="#ref-4">[4]</a></sup>

<ol start="4">
  <li id="ref-4"><a href="https://p5-learning-tau.vercel.app/">p5.js Learning — 学習テーマ一覧</a></li>
</ol>

## スケッチを操作しながら変化を確かめる

- 各学習ページでは、キャンバスの描画結果と、関数の説明・コード例を続けて確認できる
- 「スライダーで制御」では、図形の数、サイズ、色相、回転速度を画面上のスライダーで変更できる
- `createSlider()`で作った入力欄の値を描画処理で読み取り、操作に合わせて見た目を変える構成である<sup><a href="#ref-5">[5]</a></sup>
- 「HSB 色相グリッド」では、HSB（色相・彩度・明度）のうち、どの組み合わせを表示するかをクリックで切り替えられる
- 同じ色相でも彩度や明度によって見た目が変わる様子を、グリッドで確認できる<sup><a href="#ref-6">[6]</a></sup>
- 解説の末尾には「試してみよう」を設け、色や図形の数値の変更、機能の追加といった課題を載せている
- 画面上で操作できる項目と、コードを書き換えて試す課題を組み合わせた学習ページである

<ol start="5">
  <li id="ref-5"><a href="https://p5-learning-tau.vercel.app/sketches/24-slider-control">p5.js Learning — スライダーで制御</a></li>
  <li id="ref-6"><a href="https://p5-learning-tau.vercel.app/sketches/08b-hsb-grid">p5.js Learning — HSB 色相グリッド</a></li>
</ol>

## インスタンスモードでスケッチを実装する

- スケッチはインスタンスモードで実装した — スケッチごとのp5.jsオブジェクトを使って描画処理を定義する書き方である
- `p.circle()`や`p.background()`のように、受け取った`p`を通して描画関数を呼び出す
- サイトの「基本図形と色」でも、この書き方を最初に説明している<sup><a href="#ref-7">[7]</a></sup>
- `p.setup`には、キャンバスの作成など、開始時に1回実行する処理を定義する<sup><a href="#ref-8">[8]</a></sup>
- `p.draw`には繰り返し実行する描画処理を定義する — 座標や色を変えて描き直すことで、動きや入力への反応を作る<sup><a href="#ref-9">[9]</a></sup>
- 以下は学習ページに載せたインスタンスモードの例 — この関数を共通コンポーネント側からp5.jsへ渡して実行する

```js
export default function (p) {
  p.setup = () => {
    p.createCanvas(400, 400);
  };

  p.draw = () => {
    p.background(240);
    p.circle(200, 200, 100);
  };
}
```

<ol start="7">
  <li id="ref-7"><a href="https://p5-learning-tau.vercel.app/sketches/01-basic-shapes">p5.js Learning — 基本図形と色</a></li>
  <li id="ref-8"><a href="https://p5js.org/reference/p5/setup/">p5.js — setup()</a></li>
  <li id="ref-9"><a href="https://p5js.org/reference/p5/draw/">p5.js — draw()</a></li>
</ol>

## 解説と描画を分け、PNG保存を共通化する

- 解説ページは`src/pages/sketches`のMDXファイル、描画処理は`src/sketches`のJavaScriptファイルで管理している
- MDXのページでは文章やコード例を記述し、共通コンポーネントの`P5Canvas.astro`を読み込む構成にした<sup><a href="#ref-10">[10]</a></sup>
- `P5Canvas.astro`はスケッチ名を受け取り、対応するJavaScriptファイルを読み込んで、指定した要素内にp5.jsのインスタンスを作る
- スケッチの読み込みには`import.meta.glob()`を使用し、対象のモジュールを必要なときに読み込む
- p5.jsのインスタンスはスケッチ名をキーにした`Map`で保持し、PNG保存時に取り出す
- 「PNG をダウンロード」ボタンを押すと、対応するインスタンスの`saveCanvas()`を呼び出す — 描画結果を画像として保存できる
- 描画内容は個別のJavaScriptファイルに置き、読み込みと保存の処理は共通コンポーネントにまとめた<sup><a href="#ref-11">[11]</a></sup>

<ol start="10">
  <li id="ref-10"><a href="https://github.com/tktkaws/p5_learning/blob/main/src/pages/sketches/01-basic-shapes.mdx">p5_learning — 基本図形と色のMDXページ</a></li>
  <li id="ref-11"><a href="https://github.com/tktkaws/p5_learning/blob/main/src/components/P5Canvas.astro">p5_learning — P5Canvas.astro</a></li>
</ol>

## まとめ

- p5.jsをテーマ別に学べるサイト「p5-learning」を制作した
- 基本図形、アニメーション、ノイズ、物理シミュレーション、3D表現などを個別のページにまとめた
- 動くスケッチと解説・コード例を同じページで確認できる構成にした
- スライダーやクリック操作に対応した例で、数値や表示の変化を確かめられるようにした
- スケッチはインスタンスモードで実装し、`setup()`と`draw()`に処理を定義した
- MDXの解説とJavaScriptの描画処理を分け、読み込みとPNG保存を共通コンポーネントにまとめた
