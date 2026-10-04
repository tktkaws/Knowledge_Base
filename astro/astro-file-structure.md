# .astroファイルの構造 — フロントマタースクリプトとテンプレート

## .astro ファイルとは

- Astro コンポーネントを定義する拡張子
- ページ（`src/pages`）にも UI 部品（`src/components`）にも使える
- **1ファイル = コンポーネントスクリプト + HTML テンプレート** の2部構成
- デフォルトではクライアントに JavaScript を送らず、HTML として出力される

> 参照: [Astro — Components](https://docs.astro.build/en/basics/astro-components/)

## 2つの領域

```text
---
// ① コンポーネントスクリプト（フロントマター）
//    サーバー側 / ビルド時に実行
---

<!-- ② テンプレート -->
<!--    HTML として出力される -->
```

| 領域 | 実行タイミング | 主な用途 |
|---|---|---|
| コンポーネントスクリプト | ビルド時またはリクエスト時（SSR） | import、データ取得、変数定義 |
| テンプレート | 上記の結果を HTML に展開 | マークアップ、条件分岐、ループ |

- コンポーネントスクリプト内のコードはブラウザに送られない
- テンプレート内の `{expression}` もビルド / サーバー側で評価され、静的 HTML になる

## コンポーネントスクリプト（フロントマター）

### 書けること

- JavaScript / TypeScript の import
- 変数・定数の定義
- 非同期処理（`await fetch()` など）
- 他コンポーネントの import と利用準備
- `Astro.props` から props を受け取る
- `Astro.url` など Astro グローバルオブジェクトへのアクセス

```astro
---
import Card from '../components/Card.astro';
import { getCollection } from 'astro:content';

const { title } = Astro.props;
const posts = await getCollection('blog');
const currentPath = Astro.url.pathname;
---
```

### 書けない / 向かないこと

- ブラウザ API（`window`、`document`）への直接アクセス
- クリックなど DOM イベントのハンドラ（テンプレート側の `onclick` も非推奨）
- クライアント状態（React の `useState` など）— UI フレームワーク側の責務

```astro
---
// 悪い例：サーバー側スクリプトで window を参照
const width = window.innerWidth; // ビルド / SSR 時にエラー
---
```

- インタラクションが必要なら [client: ディレクティブ](./islands-architecture.md) 付きの UI コンポーネントか `<script>` タグを使う

> 参照: [Astro — Component script](https://docs.astro.build/en/basics/astro-components/#the-component-script)

## テンプレート部分

### HTML ベース

- 通常の HTML タグをそのまま書ける
- コンポーネントタグ（`<Header />`）も HTML 内に混在できる
- ルート要素は1つに限定されない（フラグメント的に複数兄弟要素も可）

### JSX 風の式

- `{variable}` で変数を展開
- `{condition && <p>表示</p>}` で条件レンダリング
- `{items.map(item => <li>{item}</li>)}` でリスト描画
- 属性値にも `{title}` を渡せる

```astro
---
const name = 'Astro';
const items = ['HTML', 'CSS', 'JS'];
const showList = true;
---

<h1>{name}</h1>
{showList && (
  <ul>
    {items.map((item) => <li>{item}</li>)}
  </ul>
)}
```

### HTML との違い（覚えておく点）

| 項目 | Astro テンプレート | 通常 HTML |
|---|---|---|
| 式の埋め込み | `{expression}` | 不可 |
| class 属性 | `class` または `class:list` | `class` |
| コンポーネント | `<MyComponent />` | 不可 |
| 自己閉じタグ | `<img />` など JSX 風も可 | 混在 |

```astro
<!-- 良い例：class:list で条件付きクラス -->
<div class:list={['card', { 'card--featured': featured }]} />

<!-- 悪い例：className（React 用。Astro では class） -->
<div className="card" />
```

> 参照: [Astro — Template syntax](https://docs.astro.build/en/reference/astro-syntax/)

## import の使い方

- コンポーネントスクリプト内でのみ import 可能
- `.astro`、`.jsx` / `.tsx`、`.vue`、`.svelte`、JSON、CSS などを読み込める
- import したコンポーネントはテンプレートでタグとして使う

```astro
---
import Layout from '../layouts/Layout.astro';
import Counter from '../components/Counter.jsx';
import data from '../data/site.json';
---

<Layout title={data.siteName}>
  <Counter client:load />
</Layout>
```

- UI フレームワークコンポーネントに `client:*` を付けない限り静的 HTML になる（[アイランドアーキテクチャ](./islands-architecture.md) 参照）

## Astro.props — 親から渡される値

- 親コンポーネントが `<Greeting name="Taro" />` のように渡した属性
- コンポーネントスクリプトで分割代入して使う

```astro
---
// src/components/Greeting.astro
interface Props {
  name: string;
  greeting?: string;
}

const { greeting = 'Hello', name } = Astro.props;
---

<h2>{greeting}, {name}!</h2>
```

```astro
---
// 悪い例：props の型定義なしで typo に気づきにくい
const { nmae } = Astro.props; // undefined のまま出力される
---
<p>{nmae}</p>
```

- TypeScript 利用時は `interface Props` を定義すると補完と型チェックが効く

> 参照: [Astro — Component props](https://docs.astro.build/en/basics/astro-components/#component-props)

## Astro グローバル — よく使うもの

| オブジェクト | 主な用途 |
|---|---|
| `Astro.props` | 親から渡された props |
| `Astro.params` | 動的ルートの URL パラメータ（`[slug].astro` など） |
| `Astro.url` | 現在リクエストの URL（`pathname`、`searchParams` など） |
| `Astro.request` | 生の Request オブジェクト（SSR 時） |
| `Astro.site` | `astro.config` の `site` 設定値 |
| `Astro.glob()` | ファイルパターンにマッチするモジュールの一括 import（レガシー。Content Collections 推奨） |

```astro
---
// src/pages/search.astro
const query = Astro.url.searchParams.get('q') ?? '';
---

<h1>検索: {query || '（未入力）'}</h1>
```

- これらはすべてサーバー側 / ビルド時に評価される
- クライアントで URL を読む必要がある場合は `<script>` か UI コンポーネント側で行う

> 参照: [Astro — Astro global](https://docs.astro.build/en/reference/api-reference/)

## サーバー側で動くことの意味

### ビルド時（SSG がデフォルト）

- `npm run build` のときコンポーネントスクリプトが実行される
- 外部 API や DB からデータを取得し、HTML を事前生成できる
- 生成された HTML にはスクリプトの中身が含まれない

### リクエスト時（SSR / オンデマンド）

- `output: 'server'` や `prerender = false` のページはリクエストごとに実行
- `Astro.request` で Cookie やヘッダーを参照できる
- それでも最終的に送るのは HTML が中心（Zero JS by default）

```astro
---
// 悪い例：毎リクエスト重い処理をページ全体で同期的に待つ設計
const allData = await fetch('https://slow-api.example.com/huge').then(r => r.json());
// → キャッシュ戦略や部分取得の検討が必要
---
```

## ページとコンポーネントの違い

| 観点 | `src/pages/*.astro` | `src/components/*.astro` |
|---|---|---|
| ルート | ファイルパスが URL になる | 単体では URL にならない |
| 役割 | ページ全体のエントリ | 再利用可能な UI 部品 |
| 構造 | 中身は同じ `.astro` 形式 | 同左 |

- 書き方自体に差はない
- 置き場所と import 関係で役割が決まる（[プロジェクト構成](./project-structure.md) 参照）

## `<script>` と `<style>` も同一ファイルに書ける

### `<script>`

- テンプレート内に `<script>` を書くとクライアント JS になる
- 処理はバンドルされ、ブラウザで実行される
- コンポーネントスクリプト（フロントマター）とは別物

```astro
<button id="btn">クリック</button>
<script>
  document.getElementById('btn')?.addEventListener('click', () => {
    alert('clicked');
  });
</script>
```

### `<style>`

- デフォルトでスコープ付き CSS（そのコンポーネントに限定）
- `is:global` でグローバル化も可能

> 参照: [Astro — Styles and CSS](https://docs.astro.build/en/guides/styling/)

## よくある間違い

### 1. フロントマターと `<script>` を混同する

- `---` 内 → サーバー / ビルド時
- `<script>` → クライアント
- DOM 操作は `<script>` か UI コンポーネント側

### 2. props をテンプレートだけで使おうとする

```astro
---
// props の分割代入はフロントマターで行う
const { title } = Astro.props;
---

<h1>{title}</h1>
```

### 3. すべてを1つの巨大 .astro に詰め込む

- 見出し・カード・フッターなどは [コンポーネント](./project-structure.md) に分割
- ページファイルは組み立て役に徹する

## 最小構成の全体像

```astro
---
// コンポーネントスクリプト
import BaseLayout from '../layouts/BaseLayout.astro';

interface Props {
  title: string;
}

const { title } = Astro.props;
const year = new Date().getFullYear();
---

<!-- テンプレート -->
<BaseLayout title={title}>
  <main>
    <h1>{title}</h1>
    <p>© {year}</p>
  </main>
</BaseLayout>
```

## 学習の進め方（このリポジトリ内）

1. [Astroとは何か](./what-is-astro.md)
2. [アイランドアーキテクチャ](./islands-architecture.md)
3. `.astro` ファイルの構造（本記事）
4. [プロジェクト構成](./project-structure.md)
5. [ファイルベースルーティング](./file-based-routing.md)

## まとめ

- `.astro` ファイルは `---` のコンポーネントスクリプトと HTML テンプレートの2部構成
- スクリプトはサーバー / ビルド時に実行され、ブラウザには送られない
- テンプレートは JSX 風の `{}` 式、条件分岐、map が使える
- `Astro.props` で親からの値、`Astro.url` などでリクエスト情報にアクセスできる
- インタラクションは `<script>` か `client:*` 付き UI コンポーネントで別途実装する
