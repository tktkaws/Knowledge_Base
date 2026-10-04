# Astroコンポーネントの書き方 — props, slots

## Astroコンポーネントとは

- `.astro` ファイル1つが1コンポーネント
- フロントマター（`---` 内）でロジック、テンプレート部分でHTMLを記述
- React の props / children に相当する仕組みが **props** と **slots**
- サーバー側（ビルド時）で実行され、静的HTMLとして出力される

> 参照: [Astro — Components](https://docs.astro.build/en/basics/astro-components/)

## props の基本

- 親コンポーネントから子へデータを渡す仕組み
- 子側では `Astro.props` で受け取る
- 分割代入で取り出すのが一般的

```astro
---
// src/components/Greeting.astro
const { name, greeting = 'Hello' } = Astro.props;
---
<p>{greeting}, {name}!</p>
```

```astro
---
// src/pages/index.astro
import Greeting from '../components/Greeting.astro';
---
<Greeting name="Astro" />
<Greeting name="World" greeting="Hi" />
```

### 悪い例：型なしで props を受け取る

```astro
---
// 型が無いと typo や undefined に気づきにくい
const { titel } = Astro.props;
---
<h1>{titel}</h1>
```

### 良い例：Props インターフェースで型付け

```astro
---
interface Props {
  name: string;
  greeting?: string;
}

const { name, greeting = 'Hello' } = Astro.props;
---
<h2>{greeting}, {name}!</h2>
```

- `Props` インターフェースを定義すると、Astro が自動で型チェックする
- オプショナル（`?`）とデフォルト値を組み合わせて使う
> 参照: [Astro — Component Props](https://docs.astro.build/en/basics/astro-components/#component-props)

## デフォルトスロット（default slot）

- 子要素を親コンポーネント内の `<slot />` に挿入する仕組み
- React の `children` に相当
- 名前を付けない `<slot />` がデフォルトスロット

```astro
---
// src/components/Card.astro
interface Props {
  title: string;
}
const { title } = Astro.props;
---
<article class="card">
  <h2>{title}</h2>
  <div class="card-body">
    <slot />
  </div>
</article>
```

```astro
---
import Card from '../components/Card.astro';
---
<Card title="記事タイトル">
  <p>ここがスロットに渡される本文</p>
  <a href="/more">続きを読む</a>
</Card>
```

- 親タグの間に書いた要素が、子の `<slot />` の位置に展開される
- 複数要素をまとめて渡せる

## 名前付きスロット（named slots）

- 複数の挿入位置を持つレイアウト向け
- 子側：`<slot name="スロット名" />`
- 親側：子要素に `slot="スロット名"` 属性を付ける

```astro
---
// src/components/PageLayout.astro
interface Props {
  title: string;
}
const { title } = Astro.props;
---
<div class="layout">
  <header>
    <slot name="header" />
  </header>
  <main>
    <h1>{title}</h1>
    <slot />
  </main>
  <footer>
    <slot name="footer" />
  </footer>
</div>
```

```astro
---
import PageLayout from '../components/PageLayout.astro';
---
<PageLayout title="About">
  <img src="/logo.svg" alt="" slot="header" />
  <p>メインコンテンツはデフォルトスロットへ</p>
  <p slot="footer">© 2026 Example Inc.</p>
</PageLayout>
```

### スロットの割り当てルール

| 属性 | 挿入先 |
|---|---|
| 属性なし | デフォルトスロット（`<slot />`） |
| `slot="default"` | デフォルトスロット |
| `slot="header"` など | 対応する名前付きスロット |

- `slot` 属性は **直接の子要素** に付ける
- 名前付きスロットに渡されなかった要素は、デフォルトスロットへ行く

> 参照: [Astro — Named Slots](https://docs.astro.build/en/basics/astro-components/#named-slots)

## スロットのフォールバック

- `<slot>` タグの **中身** がフォールバック（代替）コンテンツ
- 親から何も渡されなかったときだけ表示される

```astro
---
// src/components/Alert.astro
interface Props {
  type?: 'info' | 'warning';
}
const { type = 'info' } = Astro.props;
---
<div class={`alert alert-${type}`}>
  <slot>
    <p>デフォルトのメッセージです</p>
  </slot>
</div>
```

```astro
---
import Alert from '../components/Alert.astro';
---
<!-- フォールバックが表示される -->
<Alert />

<!-- 渡した内容が優先される -->
<Alert>
  <p>カスタムメッセージ</p>
</Alert>
```

### 悪い例：空のスロット要素を渡す

```astro
<Alert>
  <!-- 空要素を渡すとフォールバックは表示されない -->
</Alert>
```

- 空の `<Alert></Alert>` は「中身あり」とみなされ、フォールバックは出ない
- フォールバックを使いたいなら、子要素を書かない

## props と slots の使い分け

| 用途 | 使うもの |
|---|---|
| 文字列・数値・オブジェクトなどのデータ | props |
| HTML構造・マークアップの差し込み | slots |
| タイトルや日付などのメタ情報 | props |
| 本文やウィジェットの配置 | slots |

- props はシリアライズ可能な値に限定される
- slots はHTML断片を渡すため、レイアウトの柔軟性が高い

## よくある間違い

### 1. props をテンプレート外で変更しようとする

```astro
---
interface Props {
  count: number;
}
let { count } = Astro.props;
count++; // ビルド時に1回だけ実行される。リアクティブではない
---
<p>{count}</p>
```

- Astro コンポーネントはリアクティブではない
- インタラクティブな状態はフレームワークコンポーネント + `client:*` を使う

### 2. slot 属性を間違った要素に付ける

```astro
<!-- 悪い例：slot は直接の子に付ける必要がある -->
<PageLayout title="Home">
  <div>
    <p slot="footer">フッター</p> <!-- 効かない -->
  </div>
</PageLayout>
```

## 設計チェックリスト

- [ ] `Props` インターフェースで型を定義している
- [ ] オプショナル props にデフォルト値を設定している
- [ ] レイアウトは named slots、本文は default slot に分けている
- [ ] フォールバックが必要なスロットに代替コンテンツを書いている
- [ ] インタラクティブな状態を Astro props だけで管理しようとしていない

## まとめ

- Astro コンポーネントは `Astro.props` でデータを受け取り、`<slot />` で子コンテンツを挿入する
- `Props` インターフェースで型安全に書ける
- デフォルトスロットは React の children、named slots は複数挿入位置の指定
- フォールバックは `<slot>` タグ内に書き、空要素を渡すと表示されない
- 静的な構造組み立てに Astro、動的な状態にフレームワークコンポーネントを使い分ける

> 参照: [Astroのアイランドアーキテクチャ](./islands-architecture.md)
