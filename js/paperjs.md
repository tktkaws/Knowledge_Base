# Paper.jsについて調べてみた

## Paper.jsとは

- Paper.jsは、HTML5 Canvasの上で動くオープンソースのベクターグラフィックス向けスクリプティングフレームワーク
- Canvas APIをそのまま使うよりも、パス、レイヤー、グループ、ラスタ画像などをオブジェクトとして扱いやすい
- ベジェ曲線やベクター演算を扱うための機能が多く、図形の生成や編集に向いている
- Adobe Illustrator向けのスクリプティング環境だったScriptographerを元にしている
- 公式サイトでは「The Swiss Army Knife of Vector Graphics Scripting」と表現されている

> 引用: “Paper.js is an open source vector graphics scripting framework that runs on top of the HTML5 Canvas.” — [Paper.js — About](https://paperjs.org/about/)
> 参照: [paperjs/paper.js](https://github.com/paperjs/paper.js/)

## Canvasを直接使う場合との違い

- Canvas APIは、基本的には「この座標に線を描く」「この矩形を塗る」という命令を実行していく仕組み
- 一度描いた図形をあとから選択したり、移動したり、形を変えたりするには、自分で状態を管理する必要がある
- Paper.jsでは、描画するものが`Path`や`Group`などのアイテムとして残る
- そのため、描いたあとに位置、色、線幅、セグメント、階層構造を変更しやすい
- Canvasを「ピクセルに描く場所」として使うのではなく、「ベクターオブジェクトを置く作業場」として扱える

> 参照: [Paper.js — Features](https://paperjs.org/features/)

## 主要な概念

### 1. Project

- Paper.jsで扱う作品全体に近い単位
- レイヤー、パス、グループなどのアイテムを含む
- CanvasごとにPaper.jsのプロジェクトが作られる

### 2. View

- Canvasに表示される表示領域を表す
- サイズ、中心位置、拡大率、フレーム更新などに関係する
- アニメーションでは`onFrame`を使ってViewの更新タイミングに合わせて処理を書く

### 3. LayerとGroup

- `Layer`はアイテムを重ねるための階層
- `Group`は複数のアイテムをまとめて扱うための入れ物
- 図形を個別に操作するだけでなく、まとまりとして移動、回転、拡大縮小できる

### 4. PathとSegment

- `Path`は線や図形を表す基本的なアイテム
- `Segment`はPathを構成する点で、位置と曲線用のハンドルを持つ
- セグメントを追加、削除、移動することで、図形の形をプログラムから変えられる

> 引用: “In Paper.js, paths are represented by a sequence of segments that are connected by curves.” — [Paper.js — Features](https://paperjs.org/features/)

### 5. Point、Size、Rectangle

- `Point`は座標や移動量を表す
- `Size`は幅と高さを表す
- `Rectangle`は矩形領域を表す
- Paper.jsではこれらの型が用意されているので、座標計算やベクター演算を書きやすい

### 6. Raster

- ベクター図形だけでなく、画像も`Raster`として扱える
- 画像を配置したり、ピクセルの色を調べたり、図形の中の平均色を取得したりできる
- ベクターとラスタを組み合わせた表現にも使える

## 何ができるのか

### 1. パスを作る

- 直線、曲線、円、矩形などの図形を作れる
- SVGのパスデータから`Path`を作ることもできる
- 作成したパスは、色、線幅、透明度、位置などをあとから変更できる

### 2. ベジェ曲線を操作する

- セグメントとハンドルを操作して曲線を作れる
- 線を滑らかにしたり、パスを単純化したりするためのメソッドがある
- 手描き風の線や、マウス操作で変形できる図形を作りやすい

### 3. マウスやキーボード入力を扱う

- マウスを押したとき、ドラッグしたとき、離したときの処理を書ける
- キーボード入力にも反応できる
- ドローイングツールやインタラクティブな作品を作るときに便利

> 参照: [Paper.js — Features](https://paperjs.org/features/)

### 4. アニメーションを作る

- `onFrame`を使って、フレームごとに図形を動かせる
- 座標、回転、拡大縮小、色などを少しずつ変えられる
- Canvasアニメーションを、図形オブジェクト中心に書ける

### 5. SVGを読み書きする

- SVGをPaper.jsのアイテムとして読み込める
- 作った図形をSVGとして書き出せる
- 図形編集ツール、アイコン生成、デザイン素材の加工などと相性がよい

> 引用: “Paper.js offers very convenient methods to import and export vector graphics as SVG.” — [Paper.js — Features](https://paperjs.org/features/)

### 6. 画像を扱う

- Canvas上に画像を配置できる
- ピクセルの色や平均色を取得できる
- 画像と図形を組み合わせたジェネラティブアートにも使える

## 基本的な使い方

### 1. PaperScriptで使う

- PaperScriptは、Paper.jsが用意しているJavaScriptの拡張
- 使うには、Paper.js本体を読み込んでからCanvasとPaperScriptを紐づける
- `script`タグに`type="text/paperscript"`を書き、対象のCanvasを指定して使う
- `Point`や`Size`などに対して、通常の数値のような演算を書ける
- 小さなデモや学習用には始めやすい

```html
<script src="https://unpkg.com/paper/dist/paper-full.min.js"></script>
<canvas id="myCanvas" resize></canvas>
<script type="text/paperscript" canvas="myCanvas">
var path = new Path.Circle({
  center: view.center,
  radius: 80,
  fillColor: 'skyblue'
});
</script>
```

> 参照: [Paper.js — Working with Paper.js](https://paperjs.org/tutorials/getting-started/working-with-paper-js/)

### 2. 通常のJavaScriptで使う

- PaperScriptを使わず、通常のJavaScriptからPaper.jsを操作することもできる
- npmで導入する場合や、Reactなどの環境に組み込む場合はこちらの方が扱いやすい
- `paper.setup(canvas)`でCanvasをPaper.jsに紐づけてから、`paper.Path`などを使う

```js
paper.setup(document.getElementById('myCanvas'));

const path = new paper.Path();
path.strokeColor = 'black';
path.add(new paper.Point(100, 100));
path.lineTo(new paper.Point(200, 200));
paper.view.draw();
```

> 参照: [paperjs/paper.js](https://github.com/paperjs/paper.js/)

## 向いている用途

### 1. インタラクティブアート

- マウスの動きに合わせて線や図形を変化させる作品に向いている
- ベクター演算が書きやすいので、幾何学的な表現を作りやすい
- アニメーションと入力処理を組み合わせやすい

### 2. 図形エディタ

- 図形をオブジェクトとして保持できる
- 選択、移動、変形、グループ化のような操作を実装しやすい
- SVGの読み書きができるので、外部ツールとの連携もしやすい

### 3. 教育用のビジュアライゼーション

- 点、線、角度、曲線、ベクトルなどを視覚的に説明しやすい
- 数学や幾何の概念を、動かしながら見せる教材に使える
- コード量を抑えつつ、見た目の変化を作れる

### 4. SVGやアイコンを扱うツール

- SVGを読み込んで加工できる
- パスの形状をプログラムから変更できる
- 簡単なアイコン生成や図形変換ツールの土台にできる

## 注意点

### 1. DOM UIを作るためのライブラリではない

- Paper.jsはCanvas上の図形を扱うためのライブラリ
- ボタン、フォーム、レイアウトのような通常のUI部品はHTMLやCSSで作る必要がある
- アプリ全体を作る場合は、UI部分とCanvas描画部分を分けて考える

### 2. 状態管理は別に考える必要がある

- Paper.jsは図形の管理に強いが、アプリ全体の状態管理をすべて担うわけではない
- 履歴、保存、Undo/Redo、複数ユーザー同期などは別途設計が必要
- 図形データをどの形式で保存するかも、用途に合わせて決める必要がある

### 3. PaperScriptは便利だが使いどころを選ぶ

- PaperScriptは学習や小さなサンプルでは書きやすい
- 一方で、現代的なビルド環境やTypeScript中心の開発では、通常のJavaScript APIの方が統合しやすい
- 長く保守するアプリでは、PaperScriptに依存しすぎない方が見通しがよい場合がある

### 4. Canvasなのでアクセシビリティには注意が必要

- Canvasに描かれた図形は、そのままではDOM要素として読み上げられない
- 操作対象や説明が必要な場合は、別途HTML側に情報を持たせる必要がある
- 視覚表現だけで完結しない設計を考えることが大事

## 実際に触るときの流れ

### 1. 小さな図形を描く

- まずは円や線を1つ描く
- `fillColor`や`strokeColor`を変えて、どのプロパティが見た目に効くか確認する
- `view.center`を使うと、Canvas中央を基準に試しやすい

### 2. マウス操作を足す

- `onMouseDown`、`onMouseDrag`、`onMouseUp`を試す
- ドラッグした場所に線を引くと、Paper.jsの便利さが分かりやすい
- Pathがオブジェクトとして残ることを確認する

### 3. パスを変形する

- セグメントを追加、移動、削除して形を変える
- `smooth()`や`simplify()`を試す
- ベジェ曲線のハンドルを意識すると、Paper.jsの理解が深まる

### 4. SVGを読み込む

- 既存のSVGを読み込んで、色や位置を変える
- 作った図形を書き出して、別のツールで開いてみる
- Paper.jsがCanvasだけでなく、ベクターデータの加工にも使えることが分かる

## よくある勘違い

### 1. Canvas APIの薄いラッパーだと思う

- Paper.jsは、Canvasの描画命令を少し便利にするだけのものではない
- Scene Graphを持ち、図形を階層構造のあるアイテムとして扱える
- 「描いたら終わり」ではなく「描いたものをあとから操作する」ための仕組みがある

### 2. SVGライブラリと同じだと思う

- Paper.jsはCanvas上に描画する
- SVGのように、各図形がDOM要素として存在するわけではない
- ただしSVGの読み込みや書き出しはできるので、SVGデータを扱う処理とは相性がよい

### 3. アニメーションライブラリだと思う

- Paper.jsでアニメーションは作れる
- ただし主な強みは、ベクター図形の作成、編集、演算、管理にある
- タイムライン制御やUIアニメーションだけが目的なら、別のライブラリの方が合う場合もある

### 4. 何でもPaper.jsで作ろうとする

- 通常のWeb UIはHTMLとCSSで作る方がよい
- Paper.jsは、図形やCanvas表現が中心になる部分に使う
- アプリ全体ではなく、描画エリアのための専門ライブラリとして考えると使いやすい

## 調べて感じたこと

- Paper.jsは、Canvasを直接操作する難しさをかなり減らしてくれるライブラリだと感じた
- 特に、描いた図形をあとからオブジェクトとして操作できる点が大きい
- Canvas APIだけで図形エディタのようなものを作ると、図形の状態管理を自分でたくさん書く必要がある
- Paper.jsなら、パス、セグメント、グループ、レイヤーといった概念が最初から用意されている
- ただし、Webアプリ全体を支えるフレームワークではないので、UIや状態管理との役割分担は必要になる
- 学ぶなら、まずはPaperScriptで小さく試し、その後に通常のJavaScript APIでどう組み込むかを見るのがよさそう

## まとめ

- Paper.jsは、HTML5 Canvas上で動くベクターグラフィックス向けのオープンソースライブラリ
- Canvas APIを直接使うより、図形をオブジェクトとして扱いやすい
- Path、Segment、Point、Layer、Groupなどの概念を使って、複雑な図形を管理できる
- ベジェ曲線やベクター演算に強く、インタラクティブアートや図形エディタに向いている
- SVGの読み込みと書き出しができるので、デザイン素材やアイコンの加工にも使える
- PaperScriptは始めやすいが、実際のアプリに組み込むなら通常のJavaScript APIも確認した方がよい
- DOM UIを作るライブラリではないため、HTML/CSSやフレームワークとの分担を考える必要がある
- Paper.jsは、Canvasで「一度描いたら終わり」ではなく「描いた図形を育てて操作する」ための道具だと感じた

> 参照: [Paper.js — About](https://paperjs.org/about/) / [Paper.js — Features](https://paperjs.org/features/) / [Paper.js — Working with Paper.js](https://paperjs.org/tutorials/getting-started/working-with-paper-js/) / [paperjs/paper.js](https://github.com/paperjs/paper.js/)
