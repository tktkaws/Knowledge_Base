# Astroのアイランドアーキテクチャ — 部分的ハイドレーションの仕組み

## アイランドアーキテクチャとは

- ページの大部分を高速な静的HTMLとして出力し、必要な箇所だけを JavaScript の「島」として動かす設計
- 画像カルーセル、いいねボタン、検索UIなど、インタラクティブな部品だけを島にする
- SPAのようにページ全体を1つの巨大なJSアプリとしてハイドレーションしない
- 部分的ハイドレーション（partial / selective hydration）とも呼ばれる

> 参照: [Astro — Islands architecture](https://docs.astro.build/en/concepts/islands/)

## なぜ必要か

| SPA的な全体ハイドレーション | アイランド |
|---|---|
| ページ全体がJSアプリになる | 静的HTMLの海に島だけが動く |
| 初期JSが大きくなりやすい | 必要な部品のJSだけ送る |
| TTI（操作可能になるまで）が遅れやすい | 本文はすぐ読め、島は後から活性化 |

- コンテンツサイトでは「まず読めること」が重要
- 全コンポーネントをクライアントで動かす必要はない、という前提に立つ

## 島（Island）とは何か

- 静的なHTMLページの上に載る、強化されたUIコンポーネント
- **Client island** — ブラウザでハイドレーションされ、インタラクティブになる部品
- **Server island** — 動的なサーバー描画を、ページ本体と分けて行う部品（ログインユーザーの表示など）
- 島は基本的に独立して動き、同じページに複数置ける
- 島どうしで状態共有や通信も可能だが、まずは独立して設計する

```text
┌──────────────────────────────────────────┐
│  静的HTML（ヘッダー・本文・フッタ）         │
│                                          │
│   ┌─────────────┐     ┌──────────────┐   │
│   │ 島: 検索UI   │     │ 島: カルーセル │   │
│   │ (client JS) │     │ (client JS)  │   │
│   └─────────────┘     └──────────────┘   │
│                                          │
│  残りはHTMLのまま（JSなし）                │
└──────────────────────────────────────────┘
```

## デフォルトは島にならない

- Astroコンポーネント（`.astro`）は静的HTMLになり、クライアントランタイムを持たない
- React / Vue / Svelte などのUIコンポーネントも、**何も付けないと静的HTMLとして出力**される
- インタラクティブにしたいときだけ `client:*` を付ける

```astro
---
import LikeButton from '../components/LikeButton.jsx';
---

<!-- 静的HTMLのみ。クリックしても動かない（JSが送られない） -->
<LikeButton />

<!-- 島になる。ページ読み込み時にJSを入れて動かす -->
<LikeButton client:load />
```

- この「明示的にオンにする」設計が、Zero JS by default を支えている

> 参照: [Astro — Framework components](https://docs.astro.build/en/guides/framework-components/)

## 部分的ハイドレーションの流れ

`client:only` 以外のディレクティブでは、おおむね次の順になる。

1. サーバー（またはビルド）でコンポーネントをHTMLとして描画する
2. ブラウザに静的HTMLが届き、すぐ内容が見える
3. 指定したタイミングでコンポーネント用JS（と必要なフレームワーク実行時）を読み込む
4. 既存のHTMLにイベントなどを結び付け、操作可能にする（ハイドレーション）

- 同じフレームワークの島が複数あっても、フレームワーク本体はまとめて1回送られる側に最適化される

## client: ディレクティブ一覧

| ディレクティブ | いつハイドレーションするか | 向く例 |
|---|---|---|
| `client:load` | ページ読み込み直後 | ヘッダーの重要なUI、すぐ使うボタン |
| `client:idle` | 初期ロード後のアイドル時 | 緊急性の低いウィジェット |
| `client:visible` | ビューポートに入ったとき | 下部のカルーセル、埋め込み地図 |
| `client:media` | メディアクエリ一致時 | モバイルだけ必要なUI |
| `client:only` | サーバー描画なしでクライアントのみ | ブラウザAPI前提のコンポーネント |

```astro
---
import InteractiveButton from '../components/InteractiveButton.jsx';
import InteractiveCounter from '../components/InteractiveCounter.jsx';
import InteractiveModal from '../components/InteractiveModal.svelte';
import ShowHideButton from '../components/ShowHideButton.jsx';
import HeavyImageCarousel from '../components/HeavyImageCarousel.jsx';
import MobileNav from '../components/MobileNav.jsx';
---

<InteractiveButton client:load />
<ShowHideButton client:idle />
<InteractiveCounter client:visible />
<HeavyImageCarousel client:visible />
<MobileNav client:media="(max-width: 640px)" />
<InteractiveModal client:only="svelte" />
```

### 使い分けの目安

- すぐ操作される → `client:load`
- あると便利だが初速を優先 → `client:idle`
- 画面外にある重い部品 → `client:visible`
- 特定幅だけで要る → `client:media="..."`
- SSRできない / したくない → `client:only="react"` など（フレームワーク名が必要）

> 参照: [Astro — Client directives](https://docs.astro.build/en/reference/directives-reference/#client-directives)

## 島にすべきもの / すべきでないもの

### 島向き

- クリック・入力・開閉などの操作がある
- ブラウザAPI（`localStorage`、位置情報など）が必要
- フレームワークの状態管理が必要

### 島にしない（静的のまま）

- 見出し、本文、静的なカード一覧
- 見た目だけの装飾
- リンクだけのナビゲーション（CSSで十分なもの）

```astro
---
import ArticleBody from '../components/ArticleBody.astro';
import CommentForm from '../components/CommentForm.jsx';
import ShareButton from '../components/ShareButton.jsx';
---

<article>
  <ArticleBody /> <!-- 静的 -->
  <ShareButton client:idle /> <!-- 島 -->
  <CommentForm client:visible /> <!-- 島 -->
</article>
```

## SPAとの違い

| 観点 | SPA | Astroの島 |
|---|---|---|
| 描画単位 | アプリ全体 | ページ内の部品 |
| JSの量 | 大きくなりやすい | 島の分だけ |
| ルーティング | クライアント側が中心になりやすい | ページ単位（MPA寄り）が基本 |
| 向く用途 | 複雑なアプリUI | コンテンツ + 部分的な操作 |

- SPAをAstroページの中に埋め込むこともできるが、それは例外寄りの選択
- コンテンツサイトでは、島を最小限にする方がAstroの恩恵が大きい

## よくある間違い

### 1. すべてのコンポーネントに client:load を付ける

```astro
<!-- 悪い例：静的で十分なものまで島にしている -->
<Header client:load />
<Hero client:load />
<Footer client:load />
```

- JSが増え、Astroを使う意味が薄れる

### 2. 島にしないのに「動かない」と驚く

- `client:*` が無いフレームワークコンポーネントは、見た目だけのHTML
- イベントが必要ならディレクティブを付ける

### 3. client:only を安易に使う

- サーバーHTMLが無いので、読み込み前は何も見えない / レイアウトシフトしやすい
- 使えるならサーバー描画ありのディレクティブを優先する

### 4. 島を増やしすぎてページが細切れになる

- 近い操作は1つの島にまとめた方が、フレームワーク実行時の重複感や設計が単純になることもある
- 逆に、無関係な機能は分けて遅延読み込みする

## 設計チェックリスト

- [ ] インタラクティブな部品だけに `client:*` を付けている
- [ ] 初速が重要な島は `idle` / `visible` を検討した
- [ ] 画面外の重いUIは `client:visible` になっている
- [ ] `client:only` の理由が説明できる
- [ ] 静的で足りる見出し・本文・カードを島にしていない
- [ ] アクセシビリティ上必要なJSが、適切なタイミングで読み込まれる

## まとめ

- アイランドアーキテクチャは、静的HTMLの海に必要なJSの島だけを浮かべる設計
- Astroではデフォルト静的で、`client:*` を付けた部品だけがハイドレーションされる
- タイミングは `load` / `idle` / `visible` / `media` / `only` で制御する
- 島を最小限にすると、コンテンツサイトの速さを最大化しやすい
- 「全部を島にする」のではなく、「どこを島にするか」を選ぶのが要点

> 参照: [Astroとは何か](./what-is-astro.md)
