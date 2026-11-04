# Scroll Warsを制作 — Astro・three.js・Lenisでスクロールに連動する宇宙の演出を作る

## Scroll Warsとは

- Scroll Warsは、『スター・ウォーズ』のオープニングからエンディングまでを、スクロールに連動するWeb表現として制作した実験サイトである<sup><a href="#ref-1">[1]</a></sup>
- 青い導入文、大きなタイトル、星空の奥へ流れる本文、惑星が現れる場面転換、エンドクレジットへと進む
- 映像のように時間の経過で自動再生するのではなく、ページのスクロール位置で演出の進行を決める — 下へ進めば先の場面へ、上へ戻せば同じ演出を巻き戻せる
- 画面右側には場面ごとのナビゲーションを用意し、見たい場面へ直接移動できるようにした
- 文章を落ち着いて読みたい場合は、画面下の切り替えボタンから文章モードを選べる
- 使用した主な技術はAstro、three.js、Lenisである — 各場面のHTML、宇宙の描画、スクロールの制御を分担させている<sup><a href="#ref-2">[2]</a></sup>

<ol start="1">
  <li id="ref-1"><a href="https://scroll-wars.vercel.app/">Scroll Wars — 公開サイト</a></li>
  <li id="ref-2"><a href="https://github.com/tktkaws/scroll_wars">Scroll Wars — GitHubとREADME</a></li>
</ol>

## AstroでHTMLを生成し、CSSで文字に遠近感を付ける

- Astroは、導入文、タイトル、本文、クレジットなどのHTMLを生成する役割で使っている
- 場面ごとにAstroコンポーネントを分け、スクロールに応じた更新はブラウザー側のスクリプトで行う — Astroの`<script>`から初期化処理を呼び出す構成である<sup><a href="#ref-3">[3]</a></sup>
- タイトルと本文の文字は、three.jsの中へ描くのではなくHTMLとして配置した — 画面幅に応じた折り返しや文字の高さを、ブラウザーのレイアウトを使って扱える
- タイトルはCSSの`perspective`と`translateZ()`を組み合わせ、手前から奥へ遠ざかるように動かす
- 輪郭だけの文字は`-webkit-text-stroke`で描き、文字サイズは実際の高さと画面の高さから調整している
- 本文は`rotateX()`で傾けた面の上に置き、`translateY()`で移動させる — 画面下から現れた文字が奥へ進み、小さく見えるクロールになる
- 画面上部では`mask-image`で文字を徐々に消し、遠くへ流れていく見え方を作っている
- 本文の高さは実測し、文章が増減した場合も必要なスクロール距離を確保する — 固定の移動距離だけで最後の段落が残ってしまうことを避ける<sup><a href="#ref-4">[4]</a></sup>

<ol start="3">
  <li id="ref-3"><a href="https://docs.astro.build/en/guides/client-side-scripts/">Astro — Scripts and event handling</a></li>
  <li id="ref-4"><a href="https://github.com/tktkaws/scroll_wars#03--crawl">Scroll Wars README — クロールの描画と計測</a></li>
</ol>

## スクロール位置を進行度に変換し、Lenisで動きを整える

- 演出の共通の基準は、ページ全体を0〜1で表した進行度である — 実際の`window.scrollY`を演出用のスクロール距離で割り、範囲内へ収める<sup><a href="#ref-5">[5]</a></sup>
- 各場面には開始位置と終了位置を設定し、全体の進行度をその場面専用の0〜1へ変換する
- 次の例は、その計算をJavaScriptで抜き出したものである — 開始値と終了値は異なる値にし、開始から終了へ増える順序で指定する

```js
const clamp = (value) => Math.max(0, Math.min(1, value));

const portion = (value, start, end) =>
  clamp((value - start) / (end - start));

const smoothstep = (value) => value * value * (3 - 2 * value);

// 全体の20〜40%を、ひとつの場面の0〜1へ変換する例
const sceneProgress = portion(0.3, 0.2, 0.4);
const opacity = smoothstep(sceneProgress);
```

- `portion()`は場面の開始前なら0、終了後なら1を返す — 各場面の処理を、ページ全体の長さから分けて考えられる
- `smoothstep()`は変化の始まりと終わりを緩やかにする — 導入文やクレジットのフェード、カメラ移動などに使っている
- Lenisはスクロールを滑らかにするライブラリである — このサイトでは慣性を伴うスクロールや、場面ボタンからの移動に使っている<sup><a href="#ref-6">[6]</a></sup>
- 文字と星空は、スクロール位置から求めた同じ進行度に追従する — 慣性が残っている間も、それぞれの演出を同期させる
- スクロールイベントのたびに直接描画するのではなく、`requestAnimationFrame()`で更新を1フレームにまとめる
- 進行度は`opening:progress`イベントで背景側へ渡す — 文字を制御する処理から、three.jsのカメラを直接操作しないように分けている
- シーンの順序や長さは`src/data/story.ts`の`sequence`で管理する — 秒数ではなくスクロール距離に対する割合で調整する

<ol start="5">
  <li id="ref-5"><a href="https://github.com/tktkaws/scroll_wars#シーン進行の共通実装">Scroll Wars README — シーン進行の共通実装</a></li>
  <li id="ref-6"><a href="https://github.com/darkroomengineering/lenis">Lenis — 公式リポジトリ</a></li>
</ol>

## three.jsで星空とカメラ移動を描く

- three.jsは、星空、月、惑星、カメラの移動を担当する — 文字の演出とは別に、背景の宇宙を描いている<sup><a href="#ref-7">[7]</a></sup>
- 星は`THREE.Points`による点群としてまとめ、座標、大きさ、明るさを設定する — 星ごとに別のメッシュを用意する構成にはしていない
- カスタムシェーダーで点の輪郭と透明度を調整し、柔らかい光のような星に見せている
- 星の配置にはシード付きの乱数を使い、同じ設定から同じ配置を作れるようにしている
- 月と惑星は画像を使った`THREE.Sprite`として配置し、カメラの向きや位置によって見え方を変える
- 本文のクロール後はカメラを下へ向け、宇宙に配置した月や惑星が視野へ入る — 単に背景画像を切り替えるのではなく、カメラの移動で場面をつないでいる
- 次のクレジットへの移行では、宇宙の表示領域を円形に縮める — 円の内側と外側で背景を分け、ぼかしも組み合わせて場面を切り替える
- 星空は進行度や画面サイズなどが変わったときに描画する — 時間の経過だけで星を流し続ける処理は設けていない
- 初期化時の画面幅に応じて星の数を変え、描画のピクセル比にも上限を設けている — 見た目だけでなく描画負荷も調整している

<ol start="7">
  <li id="ref-7"><a href="https://github.com/tktkaws/scroll_wars#星空の描画">Scroll Wars README — 星空の描画</a></li>
</ol>

## 文章モードとフォールバックを用意する

- 本文を通常のレイアウトでも読めるよう、文章モードを用意した — 各場面の内容を通常の文書の流れで表示する<sup><a href="#ref-8">[8]</a></sup>
- 演出と文章モードで同じAstroコンポーネントを使い、内容を別々に更新する必要がない構成にしている
- 文章モードへ切り替えるとLenisを破棄し、演出モードへ戻す際は保存した進行位置を復元する — 読み終えた後に先頭から演出をやり直さずに済む
- OSで`prefers-reduced-motion: reduce`が有効な場合は、初期表示から文章モードを選択する
- WebGLを初期化できない場合は、SVGの星空とHTML画像の月・惑星を表示する — three.jsで描画できなくても背景を用意できる
- JavaScriptが無効な場合も、HTMLとして生成した本文を読めるようにしている
- 演出表示では変形した本文とは別に読み上げ用の本文を用意し、見えなくなった場面や、まだ操作できない領域には`inert`を使う
- 文章モードへの切り替え、動きを抑える設定、WebGLやJavaScriptが使えない場合を、それぞれ別の条件として扱っている
- スクロールで進める表現を見せつつ、内容を読むための方法も残す構成である

<ol start="8">
  <li id="ref-8"><a href="https://github.com/tktkaws/scroll_wars#文章モードとフォールバック">Scroll Wars README — 文章モードとフォールバック</a></li>
</ol>

## まとめ

- Scroll Warsは、映画のオープニングからエンディングへ進む流れを、スクロールに連動するWeb表現として制作した
- Astroで各場面のHTMLを生成し、文字の遠近感と移動はCSSの3D変形で表現している
- three.jsは星空や天体を描き、スクロール進行度に応じたカメラ移動を担当する
- Lenisでスクロールを滑らかにし、文字と背景は同じ進行度に追従させている
- 本文の高さを計測することで、文章量に合わせて移動距離とスクロール距離を調整している
- 文章モードでは通常のレイアウトで内容を読み、演出に戻ると保存した位置を復元できる
- 動きを抑える設定や、WebGL・JavaScriptが使えない場合も考慮し、演出とは別に本文を読める方法を用意した
