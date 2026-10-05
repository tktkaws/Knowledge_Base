# client:ディレクティブ — load, idle, visible, media の使い分け

## この記事の位置づけ

- [Astroのアイランドアーキテクチャ](./islands-architecture.md) で「島」の概念と一覧を説明済み
- 本記事では **実務での選び方** と **パフォーマンス観点** を厚く扱う
- 「全部 `client:load`」のような現場あるあるを避けるための判断基準

> 参照: [Astro — Client directives](https://docs.astro.build/en/reference/directives-reference/#client-directives)

## ディレクティブが決めること

- **いつ** フレームワークのJSを読み込むか
- **いつ** ハイドレーション（イベント結び付け）を行うか
- `client:only` 以外は、まずサーバーで静的HTMLが出力される
- 本文はすぐ読め、島のJSは後から載る、という段階的な体験を作れる

## 各ディレクティブの詳細

### client:load

- **優先度：高**
- ページ読み込み直後にJSを取得し、すぐハイドレーション
- Above the fold（最初の画面）で即操作が必要なUI向け

```astro
<BuyButton client:load />
<MobileMenuToggle client:load />
```

- メインスレッドの初期処理と競合しやすい
- 付けすぎると TTI（操作可能になるまで）が遅れる

### client:idle

- **優先度：中**
- 初期ロード完了後、`requestIdleCallback`（非対応時は `load` イベント）でハイドレーション
- 「あると便利だが、初速を優先したい」部品向け

```astro
<ShareButton client:idle />
<ThemeToggle client:idle />
```

- ユーザーがすぐ触らないウィジェットに向く
- ヘッダー付近でも、操作頻度が低ければ `load` より `idle` が適切なことが多い

### client:visible

- **優先度：低**
- `IntersectionObserver` でビューポートに入ったタイミングでハイドレーション
- 画面外の重いUI、長文ページの下部ウィジェット向け

```astro
<CommentSection client:visible />
<ImageCarousel client:visible />
<EmbeddedMap client:visible />
```

- ユーザーがスクロールしない限りJSを読まない
- 折りたたみ（アコーディオン）内のコンテンツにも有効

### client:media

- **優先度：低**
- 指定した CSS メディアクエリが **一致したとき** にハイドレーション
- 特定の画面幅でのみ必要なUI向け

```astro
<MobileNav client:media="(max-width: 640px)" />
<DesktopSidebar client:media="(min-width: 1024px)" />
```

- CSS で `display: none` しているだけなら、`client:visible` の方がシンプルなこともある
- メディアクエリと表示制御が一致している場合に `client:media` が効く

### client:only

- サーバー描画をスキップし、クライアントのみで描画
- **フレームワーク名が必須**（`react` / `vue` / `svelte` / `preact` / `solid-js`）

```astro
<BrowserOnlyChart client:only="react" />
<LocalStorageWidget client:only="vue" />
```

- 読み込み前は何も表示されない（またはプレースホルダーが必要）
- レイアウトシフト（CLS）のリスク
- `window` / `localStorage` などブラウザAPI前提の最終手段

> 参照: [Astro — client:only](https://docs.astro.build/en/reference/directives-reference/#clientonly)

## 使い分け表（実務向け）

| シナリオ | 推奨ディレクティブ | 理由 |
|---|---|---|
| 購入ボタン・主要CTA | `client:load` | 即操作が必要 |
| ヘッダーの検索（よく使う） | `client:load` | 初回表示で操作される |
| SNSシェアボタン | `client:idle` | 初速より本文優先 |
| ページ下部のコメント欄 | `client:visible` | スクロールまでJS不要 |
| 重いチャート・地図 | `client:visible` | 画面外なら読み込まない |
| モバイル専用ナビ | `client:media="(max-width: 640px)"` | デスクトップではJS不要 |
| localStorage 依存UI | `client:only="react"` | SSR不可 |
| 静的な見出し・本文 | **付けない** | JS自体不要 |

## パフォーマンス観点

### メインスレッドへの影響

```text
client:load  × 5個  →  初期ロード直後に5つの島が競合
client:idle  × 3個  →  本文表示後、アイドル時に順次処理
client:visible × 2個 →  スクロールまでJSゼロ
```

- `client:load` は「今すぐ」なので、LCP 後もメインスレッドが忙しくなりやすい
- 同一フレームワークの島は実行時が1回にまとめられるが、**ハイドレーション処理自体は島の数だけ発生**
- 初速を上げるには `load` の数を減らし、`idle` / `visible` に逃がす

### ネットワーク帯域

- `client:visible` / `client:media` は条件を満たすまでJSファイルを取得しない
- モバイル回線や長文ページで効果が大きい
- Lighthouse の「未使用JavaScript」指標改善にもつながりやすい

### ユーザー体験とのトレードオフ

| ディレクティブ | 初速 | 初回操作の速さ | 未使用JS削減 |
|---|---|---|---|
| `client:load` | △ | ◎ | △ |
| `client:idle` | ◎ | ○ | ○ |
| `client:visible` | ◎ | △（画面外） | ◎ |
| `client:media` | ○ | 条件次第 | ◎ |
| `client:only` | △（空白期間） | △ | △ |

## 悪い例：全部 client:load

```astro
---
import Header from '../components/Header.jsx';
import Hero from '../components/Hero.jsx';
import FeatureList from '../components/FeatureList.jsx';
import Testimonials from '../components/Testimonials.jsx';
import Newsletter from '../components/Newsletter.jsx';
import Footer from '../components/Footer.jsx';
---
<Header client:load />
<Hero client:load />
<FeatureList client:load />
<Testimonials client:load />
<Newsletter client:load />
<Footer client:load />
```

- 静的HTMLで十分な Hero / FeatureList / Footer まで島化している
- 初期JSが一括ロードされ、Astro の Zero JS by default の恩恵が消える
- Next.js 的な「全部クライアント」に近い状態になる

### 良い例：必要な島だけ、タイミングも分散

```astro
---
import Header from '../components/Header.astro';
import Hero from '../components/Hero.astro';
import FeatureList from '../components/FeatureList.astro';
import Testimonials from '../components/Testimonials.jsx';
import Newsletter from '../components/Newsletter.jsx';
import Footer from '../components/Footer.astro';
---
<Header />
<Hero />
<FeatureList />
<Testimonials client:visible />
<Newsletter client:idle />
<Footer />
```

- 静的部分は `.astro` のまま
- 下部の重い部品は `visible`、そこそこ重要な Newsletter は `idle`

## 実務の判断フロー

```text
1. クリック・入力など操作が必要か？
   └ No  → client:* 不要（.astro で十分）
   └ Yes → 2へ

2. 初回表示で即操作されるか？
   └ Yes → client:load を検討
   └ No  → 3へ

3. 画面内（above the fold）にあるか？
   └ No  → client:visible
   └ Yes → 4へ

4. 特定の画面幅だけ必要か？
   └ Yes → client:media
   └ No  → client:idle

5. SSR できない（ブラウザAPI前提）か？
   └ Yes → client:only（最終手段）
```

## アクセシビリティの注意

- キーボード操作やスクリーンリーダー向けJSも、適切なタイミングで読み込む必要がある
- 重要な操作UIを `client:visible` にした場合、スクロール前は操作不能になる点に注意
- フォーカス可能な要素が `client:only` で空白期間を作ると、体験が悪化しやすい

> 参照: [Astro — Accessibility note](https://docs.astro.build/en/guides/framework-components/#hydrating-interactive-components)

## 計測と改善

- Chrome DevTools の Performance / Network タブで、初期ロード時のJS量を確認
- Lighthouse の TBT（Total Blocking Time）と Unused JavaScript を見る
- `client:load` を1つ `client:idle` に変えただけで改善することもある
- 本番に近い回線（Slow 3G）で体感を確認する

## 設計チェックリスト

- [ ] `client:load` は本当に即操作が必要なUIだけに付けている
- [ ] 画面外の部品は `client:visible` を検討した
- [ ] 緊急度の低いウィジェットは `client:idle` を検討した
- [ ] `client:only` を使う理由（SSR不可）が説明できる
- [ ] 静的で足りる部分にディレクティブを付けていない

## まとめ

- `client:*` は「いつJSを載せるか」を制御する実務上の最重要スイッチ
- `load` は即操作向け、`idle` / `visible` / `media` は初速と帯域の節約向け
- 全部 `client:load` は Astro を選んだ意味が薄れる典型的なアンチパターン
- 操作の必要性 → 表示位置 → 画面幅 → SSR可否、の順で判断すると選びやすい
- 計測しながら `load` を `idle` / `visible` に逃がすのが定石

> 参照: [Astroのアイランドアーキテクチャ](./islands-architecture.md) / [React / Vue / Svelteコンポーネントの統合](./framework-components-integration.md)
