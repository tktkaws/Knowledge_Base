# orientation: portrait について調べてみた

## orientation: portrait とは

- 表示領域が **縦長（または正方形）** かどうかを判定するCSSの向き特性
- 値は2つだけ

| 値 | 条件 |
|---|---|
| `portrait` | 高さ **≧** 幅 |
| `landscape` | 幅 **>** 高さ |

- 使える場所は主に2つ
  - `@media (orientation: portrait)` — **ビューポート**（画面）の向き
  - `@container (orientation: portrait)` — **コンテナ要素**の向き
- 端末の傾きセンサーではなく、**幅と高さの比率**で決まる
- Media Queries Level 4 で定義されている古典的なメディア特性で、対応は広い

```css
@media (orientation: portrait) {
  /* ビューポートが縦長のときのスタイル */
}
```

> 参照: [MDN — orientation](https://developer.mozilla.org/ja/docs/Web/CSS/Reference/At-rules/@media/orientation) / [W3C — Media Queries Level 4](https://www.w3.org/TR/mediaqueries-4/#orientation)

## 想定シチュエーション

- メインビジュアルのヒーローがおおよそビューポート全体（例: `100% × 100dvh`）
- PCでは「テキスト左・画像右」などの横並び
- **表示領域が縦長**になったら、SP向けの縦積みレイアウトに切り替えたい
- 「幅が○○px未満」ではなく、「縦長かどうか」で切り替えたいときに `orientation: portrait` が候補になる

## `@media` で使う場合

- 判定対象は **ブラウザのビューポート**
- ヒーローが画面いっぱいなら、画面の向き ≒ ヒーローの向きになりやすい
- 実装が単純で、追加の `container-type` も不要

```css
/* デフォルト：横長向け（PC） */
.hero__inner {
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  gap: 2rem;
  height: 100%;
}

.hero {
  width: 100%;
  height: 100dvh;
}

/* ビューポートが縦長：SP向け */
@media (orientation: portrait) {
  .hero__inner {
    grid-template-columns: 1fr;
    grid-template-rows: auto 1fr;
    gap: 1.5rem;
  }

  .hero__media {
    aspect-ratio: 4 / 5;
  }
}
```

```html
<section class="hero" aria-label="メインビジュアル">
  <div class="hero__inner">
    <div class="hero__copy">
      <h1 class="hero__title">見出し</h1>
      <p class="hero__lead">リード文</p>
    </div>
    <img class="hero__media" src="/images/hero.jpg" alt="" width="1600" height="900" />
  </div>
</section>
```

- ヒーローが常にフルスクリーン前提なら、これで十分なケースが多い
- 「画面向き」と「ヒーロー箱の向き」が一致しない配置ではズレる — その場合は後述の `@container` 側を検討する

## `@container` で使う場合

- 判定対象は **指定した要素の content box**
- ヒーロー自身が縦長かを見たいときはこちら
- 前提として `container-type: size` と、**明示的な高さ**が必要

```css
.hero {
  container-type: size;
  container-name: hero;
  width: 100%;
  height: 100dvh;
}

.hero__inner {
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  gap: 2rem;
  height: 100%;
}

@container hero (orientation: portrait) {
  .hero__inner {
    grid-template-columns: 1fr;
  }
}
```

- ヒーローがヘッダー下だけ・左右分割の片側だけ、といった配置でも、箱の比率で正しく切り替わる
- `container-type: size` の詳細は別記事側に譲る

> 参照: [コンテナタイプ：サイズを使ってアスペクト比を検知してレイアウトを変更する](./container-size-aspect-ratio.md) / [MDN — @container](https://developer.mozilla.org/ja/docs/Web/CSS/Reference/At-rules/@container)

## `@media` と `@container` の違い

| | `@media (orientation: portrait)` | `@container (orientation: portrait)` |
|---|---|---|
| 測る対象 | ビューポート | コンテナ要素 |
| 追加指定 | 不要 | `container-type: size` と高さ |
| 向いているとき | ヒーロー ≒ 画面全体 | ヒーロー箱単体の比率で決めたい |
| ズレやすいとき | ヒーローが画面の一部だけ | 高さが決まっていない箱 |

```css
/* 悪い例：ヒーローは画面の下半分なのに、画面向きだけで切り替える */
@media (orientation: portrait) {
  .hero__inner {
    grid-template-columns: 1fr;
  }
}

/* 良い例：ヒーロー箱の向きで切り替える */
.hero {
  container-type: size;
  container-name: hero;
  height: 50dvh; /* 画面の一部でも、箱が縦長なら portrait */
}

@container hero (orientation: portrait) {
  .hero__inner {
    grid-template-columns: 1fr;
  }
}
```

## `max-width` との関係

- `orientation: portrait` は **幅の数値**ではなく **比率**で見る
- だから「狭い＝縦長」「広い＝横長」とは限らない

| 状態の例 | `max-width: 768px` | `orientation: portrait` |
|---|---|---|
| スマホ縦持ち | 当たりやすい | 当たる |
| スマホ横持ち | 当たりやすいことが多い | **landscape（当たらない）** |
| PCで縦長にリサイズ | 当たらないことも | 当たる |
| 正方形に近いウィンドウ | 条件による | 当たる（高さ ≧ 幅） |

```css
/* 悪い例：「スマホ＝portrait」と決めつける — 横持ちでレイアウトが戻ってしまう */
@media (orientation: portrait) {
  .hero__inner {
    grid-template-columns: 1fr;
  }
}

/* 良い例：比率と幅の役割を分ける */
@media (orientation: portrait) {
  .hero__inner {
    grid-template-columns: 1fr; /* 縦長なら縦積み */
  }
}

@media (max-width: 768px) {
  .hero__title {
    font-size: 1.75rem; /* 狭い幅でのタイポ調整 */
  }
}
```

- レイアウトの縦／横切り替え — `orientation`
- 余白・フォント・タッチターゲットなど幅依存の調整 — `max-width` / `min-width`
- 「SP用」を1つの条件にまとめすぎないのがポイント

## 端末の向き・キーボード・正方形

### 端末の物理向きではない

- MDNでも強調されている通り、`orientation` はデバイスオリエンテーションAPIの向きではない
- ビューポート（またはコンテナ）の **幅と高さの比較結果**だけを見る

### ソフトキーボードで landscape になることがある

- スマホ縦持ちでも、キーボード表示でビューポートが高く潰れ、幅 ＞ 高さになると **`landscape` 扱い**になる
- `@media (orientation: portrait)` だけで重要UIを切り替えていると、入力中にレイアウトが跳ねることがある
- ヒーロー（キーボードと無関係なファーストビュー）なら影響は小さいが、フォーム周りでは別手段を検討する

### 正方形は portrait

- 高さ ＝ 幅 のときは **`portrait`**
- 「厳密に縦長だけ」にしたいなら `orientation` ではなくアスペクト比クエリを使う

```css
/* 正方形を横長側に寄せたい例（コンテナクエリ） */
@container hero (max-aspect-ratio: 0.99 / 1) {
  .hero__inner {
    grid-template-columns: 1fr;
  }
}
```

> 参照: [MDN — orientation（キーボードの注意）](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/orientation)

## JavaScriptでの検知

- CSSだけで足りないとき（計測・ログ・条件付き初期化）は `matchMedia` を使う

```js
const portraitQuery = window.matchMedia('(orientation: portrait)');

function applyHeroMode(isPortrait) {
  document.documentElement.dataset.heroOrientation = isPortrait
    ? 'portrait'
    : 'landscape';
}

applyHeroMode(portraitQuery.matches);
portraitQuery.addEventListener('change', (event) => {
  applyHeroMode(event.matches);
});
```

- レイアウト自体は可能ならCSSの `@media` / `@container` に任せ、JSは補助に留める
- コンテナの向きをJSで厳密に追うなら `ResizeObserver` で要素の幅・高さを比較する必要がある — ビューポート向きとは別物

## よくある間違い

### 1. 「portrait＝スマホ」と同一視する

- 縦長のタブレット、分割画面、縦に伸ばしたブラウザ窓も portrait
- 横持ちスマホは landscape — SPレイアウトが必要なのに戻る

### 2. 画面向きとヒーロー箱の向きを混同する

```css
/* 間違い：ヒーローは半画面なのに @media だけ使う */
@media (orientation: portrait) {
  .hero__inner { grid-template-columns: 1fr; }
}

/* 正解：箱を測るなら size コンテナ */
.hero { container-type: size; container-name: hero; height: 50dvh; }
@container hero (orientation: portrait) {
  .hero__inner { grid-template-columns: 1fr; }
}
```

### 3. `@container` なのに `inline-size` のまま使う

- `orientation` は高さ情報が必要 — **`container-type: size`** が前提
- 高さ `auto` の箱では向きクエリの土台にならない

### 4. `max-width` の代わりに何でも `orientation` にする

- フォントサイズや余白は幅条件の方が意図に合うことが多い
- `orientation` は「縦並びか横並びか」など比率に依存する判断に使う

### 5. キーボード表示時の向き変化を見落とす

- 入力UIまで `@media (orientation: portrait)` に依存させない
- ヒーロー単体なら影響は限定的だが、挙動は一度実機で確認する

## まとめ

- `orientation: portrait` は **高さ ≧ 幅** のときに一致する — 端末の傾きセンサーではない
- `@media` はビューポート、`@container` は要素箱 — ヒーローが画面全体なら前者、箱単位なら後者
- 正方形も portrait — 厳密な比率制御には `aspect-ratio` / `max-aspect-ratio` を使う
- 「SPレイアウト」を `orientation` だけに押し込むと、横持ちスマホやキーボードで破綻しやすい
- 縦／横の並びは `orientation`、幅依存の調整は `max-width` などと役割分担する
- コンテナで使うなら `container-type: size` と明示高さが必須

> 参照: [MDN — orientation](https://developer.mozilla.org/ja/docs/Web/CSS/Reference/At-rules/@media/orientation) / [MDN — @container](https://developer.mozilla.org/ja/docs/Web/CSS/Reference/At-rules/@container) / [W3C — Media Queries Level 4](https://www.w3.org/TR/mediaqueries-4/#orientation)
