# コンテナタイプ：サイズ — アスペクト比でヒーローのレイアウトを切り替える

## コンテナクエリと `container-type: size` とは

- コンテナクエリは、**ビューポートではなく要素自身のサイズ**に応じてスタイルを切り替える仕組み
- コンテナに `container-type` を指定し、子要素側で `@container` を書く
- `container-type` の主な値

| 値 | 測定できる軸 | 高さの指定 |
|---|---|---|
| `inline-size` | インライン方向（通常は幅）のみ | 高さはコンテンツ任せでよい |
| `size` | **幅と高さの両方** | **両方に明示的なサイズが必要** |
| `normal` | なし（デフォルト） | — |

- アスペクト比や縦向き／横向きを見るには高さ情報がいるため、**`container-type: size`** を使う
- 想定シチュエーション
  - ヒーローがおおよそビューポート全体（例: `100% × 100dvh`）
  - 横長ならテキスト左・画像右、**箱が縦長**なら縦積みに切り替えたい
- `@media (orientation: portrait)` だと判定対象はブラウザ画面全体になる
- コンテナクエリなら「ヒーローという箱の比率」で決められる — 半画面配置やコンポーネント再利用にも強い

> 参照: [MDN — CSS container queries](https://developer.mozilla.org/ja/docs/Web/CSS/CSS_containment/Container_queries) / [MDN — container-type](https://developer.mozilla.org/ja/docs/Web/CSS/Reference/Properties/container-type)

## 前提：`size` は明示サイズが必須

- `container-type: size` はサイズ包含を有効にする
- コンテナ自身の幅・高さが、中身の高さに依存してはいけない
- ヒーローなら「箱のサイズが先に決まる」書き方が前提になる

```css
.hero {
  container-type: size;
  container-name: hero;
  width: 100%;
  height: 100dvh; /* 高さが決まっていることが必須 */
}
```

```css
/* 悪い例：高さがコンテンツ任せ — size コンテナとして機能しない */
.hero-bad {
  container-type: size;
  width: 100%;
  /* height なし → 高さ方向のクエリが使えない */
}
```

- 高さの候補: `100dvh` / `100vh`、Grid / Flex で割り当て済みの高さ
- `height: auto` のままでは、アスペクト比クエリの土台にならない
- モバイルでは `100vh` がアドレスバー分ズレることがある — 可能なら `100dvh` を検討する

## アスペクト比でレイアウトを切り替える

- `@container` で使える主な特性

| 特性 | 意味 |
|---|---|
| `orientation: portrait` | 高さ **≧** 幅（縦長・正方形含む） |
| `orientation: landscape` | 幅 **>** 高さ（横長） |
| `aspect-ratio` / `min-aspect-ratio` / `max-aspect-ratio` | 幅÷高さの比率で細かく指定 |

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

```css
.hero {
  container-type: size;
  container-name: hero;
  width: 100%;
  height: 100dvh;
}

/* デフォルト：横長向け */
.hero__inner {
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  gap: 2rem;
  height: 100%;
  padding: 2rem;
}

/* コンテナが縦長：SP向け */
@container hero (orientation: portrait) {
  .hero__inner {
    grid-template-columns: 1fr;
    grid-template-rows: auto 1fr;
    gap: 1.5rem;
    padding: 1.25rem;
  }

  .hero__title {
    font-size: clamp(1.5rem, 8cqmin, 2.75rem);
  }

  .hero__media {
    aspect-ratio: 4 / 5;
    width: 100%;
  }
}
```

- クエリ対象は `.hero`、切り替えるのは `.hero__inner` 以降 — コンテナは自分自身をクエリできない
- `container-name` を付けると、ネスト時に意図しないコンテナへ紐づきにくい
- 閾値を変えたいときは `max-aspect-ratio` が分かりやすい

```css
/* 正方形〜縦長で切り替える */
@container hero (max-aspect-ratio: 1 / 1) {
  .hero__inner {
    grid-template-columns: 1fr;
  }
}
```

> 参照: [MDN — Using container size and style queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_size_and_style_queries)

## メディアクエリとの使い分け

| 切り口 | 向いているケース |
|---|---|
| `@media (orientation: portrait)` | 画面全体の向きだけで足りるとき |
| `@media (max-width: …)` | 端末幅のブレークポイント運用が既にあるとき |
| `@container … (orientation / aspect-ratio)` | **ヒーロー箱そのもの**の比率で決めたいとき |

```css
/* 悪い例：ヒーローが半画面でも画面向きで切り替わる */
@media (orientation: portrait) {
  .hero__inner {
    grid-template-columns: 1fr;
  }
}

/* 良い例：ヒーローの箱の比率で切り替える */
@container hero (orientation: portrait) {
  .hero__inner {
    grid-template-columns: 1fr;
  }
}
```

- 両方が必要なこともある
  - サイト全体の余白・タイポは `@media`
  - ヒーロー内部の並び・画像比率だけ `@container`
- 「幅が狭い」と「縦長」は別問題 — 必要なら条件を重ねる

```css
@container hero (orientation: portrait) and (max-width: 480px) {
  .hero__lead {
    font-size: 0.875rem;
  }
}
```

## よくある間違い

### 1. `inline-size` のままアスペクト比を見ようとする

- `inline-size` では高さ方向の情報が取れない
- `orientation` / `aspect-ratio` には **`size` + 明示的な高さ** が必要

### 2. コンテナ自身にクエリ結果のスタイルを当てる

```css
/* 間違い */
@container hero (orientation: portrait) {
  .hero {
    padding: 1rem;
  }
}

/* 正解：子要素側を切り替える */
@container hero (orientation: portrait) {
  .hero__inner {
    padding: 1rem;
  }
}
```

### 3. `orientation: portrait` と「スマホ」を同一視する

- 縦長のタブレット・リサイズ窓・分割画面も portrait になり得る
- 横向きスマホは landscape — SP向けUIが必要なのに切り替わらないことがある
- 「端末種別」ではなく **表示領域の比率** で設計する

## まとめ

- アスペクト比・縦横判定には **`container-type: size`** が必要 — `inline-size` では足りない
- `size` は **幅と高さの両方が明示されている箱** にだけ使える
- ヒーローが縦長なら SP レイアウト、は `@container (orientation: portrait)` または `max-aspect-ratio` で表現できる
- 判定対象はビューポートではなく **ヒーロー要素自身**
- スタイルはコンテナの **子孫** に当てる — コンテナ自身には当てない
- 「スマホだから」ではなく「箱が縦長だから」で設計し、必要なら幅条件や `@media` と組み合わせる
- `cqmin` などでタイポグラフィも箱基準に揃えると、切り替え後のバランスが取りやすい

> 参照: [MDN — CSS container queries](https://developer.mozilla.org/ja/docs/Web/CSS/CSS_containment/Container_queries) / [web.dev — CSS container queries](https://web.dev/articles/cq-stable)
