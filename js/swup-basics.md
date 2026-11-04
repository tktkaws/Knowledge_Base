# swup というライブラリの使い方 — ページ遷移をスムーズにする

## swup とは

- swup は、サーバレンダリングされたサイト向けの **ページ遷移ライブラリ**
- リンククリック時にページ全体をリロードせず、指定したコンテナの中身だけを差し替える
- SPA フレームワークなしで、「アプリっぽい」遷移感を足せるのが特徴
- キャッシュ、履歴操作、スクロール位置の管理、アニメーションのタイミング検出などを担う
- 最近の日本のギャラリーサイト（制作実績を載せるサイト）でも、よく見かける実装のひとつ
- 本記事では、効果のイメージと、最低限の実装手順を整理する

> 参照: [swup — Getting Started](https://swup.js.org/getting-started/)

## どんな効果があるのか

- ページ切り替え時のフェードやスライドなど、遷移アニメーションを付けやすい
- 遷移中も URL とブラウザ履歴は通常どおり動く — 戻る／進むが破綻しにくい
- 一度訪れたページはキャッシュされ、再訪が速くなりやすい
- ヘッダーやフッターを残したまま、メインだけ差し替える構成に向いている
- ギャラリーサイトで使われやすい理由
  - 作品一覧 ↔ 詳細の行き来が多い
  - フルリロードだと「サイトが切り替わる感」が強く、世界観が途切れやすい
  - 軽い遷移で、ビジュアル重視の演出と相性がよい

| 通常の遷移 | swup あり |
|---|---|
| 毎回フルリロード | コンテナ単位で差し替え |
| アニメはブラウザ標準に近い | CSS / テーマで遷移を設計できる |
| 状態は基本リセット | 外側の UI を残しやすい |

## 実装の前提：マークアップ

- 差し替える領域に共通のコンテナ ID を付ける — デフォルトは `#swup`
- 全ページで、同じセレクタのコンテナが存在する必要がある
- ヘッダーなど残したい要素は、コンテナの外に置く

```html
<body>
  <header><!-- 残したい共通 UI --></header>

  <main id="swup" class="transition-fade">
    <!-- ここがページごとに差し替わる -->
  </main>

  <script type="module" src="/js/swup-init.js"></script>
</body>
```

- 特定リンクだけ通常遷移にしたい場合は `data-no-swup` を付ける

```html
<a href="/external.pdf" data-no-swup>PDFを開く</a>
```

> 参照: [swup — Markup](https://swup.js.org/api/markup/)

## 実装方法

### インストールと初期化

```bash
npm install swup
```

```js
import Swup from "swup";

const swup = new Swup({
  containers: ["#swup"],
});
```

- CDN から読み込む場合もあるが、ビルド環境があるなら npm が扱いやすい
- `containers` に差し替えたい要素のセレクタを列挙する

### アニメーション用 CSS

- swup は `html` に状態クラスを付け、CSS トランジションの完了を待ってから中身を入れ替える
- クラス名に `transition-` を含む要素が、タイミング検出の対象になる（デフォルト）

```css
/* フェードの最小例 */
html.is-changing .transition-fade {
  transition: opacity 0.25s ease;
  opacity: 1;
}

html.is-animating .transition-fade {
  opacity: 0;
}
```

- 公式テーマ（fade / slide / overlay）を使うと、初期の見た目を早く決められる
- 自前で書く場合も、「出ていく」「入ってくる」の2相を CSS で設計するイメージ

> 参照: [swup — Options](https://swup.js.org/options/) / [swup — Themes](https://swup.js.org/themes/)

## よく使う拡張

- swup 本体は小さく、追加機能はプラグインで足す設計

| 用途 | 手段の例 |
|---|---|
| スクロール位置の制御 | Scroll Plugin |
| 先読みで体感速度を上げる | Preload Plugin |
| `<head>` の meta / CSS 更新 | Head Plugin |
| スクリーンリーダー向け改善 | A11y Plugin |
| 進捗表示 | Progress Plugin |

```js
import Swup from "swup";
import SwupHeadPlugin from "@swup/head-plugin";
import SwupPreloadPlugin from "@swup/preload-plugin";

const swup = new Swup({
  containers: ["#swup"],
  plugins: [new SwupHeadPlugin(), new SwupPreloadPlugin()],
});
```

- ギャラリーサイトでは、Preload と Head の組み合わせが特に効きやすい
  - 次ページの画像・詳細を先に取りにいける
  - ページ固有の CSS / title も追従させやすい
- `native: true` で View Transitions API を使う道もある — 対応ブラウザ向けの選択肢

## まとめ

- swup はサーバレンダリングサイトに、ページ遷移のスムーズさを足すライブラリ
- ギャラリーサイトでよく見かけるのは、世界観を保ったままページを行き来できるため
- `#swup` などの共通コンテナを全ページに置き、外側に共通 UI を残す
- 初期化は `new Swup({ containers: ['#swup'] })` が基本
- アニメーションは CSS（または公式テーマ）で設計し、タイミングは swup が待つ
- 必要に応じて Preload / Head / A11y などのプラグインを足す
- SPA 化せずに「アプリっぽい遷移」が欲しいときの現実的な選択肢になる

> 参照: [swup.js.org](https://swup.js.org/) / [swup/swup — GitHub](https://github.com/swup/swup)
