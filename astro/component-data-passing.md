# コンポーネント間のデータ受け渡し

## Astroにおけるデータの流れ

- 基本は **親 → 子** への props 渡し
- Astro コンポーネントはビルド時 / サーバー側で1回評価される（リアクティブではない）
- 島（`client:*` 付きフレームワークコンポーネント）同士の状態共有は **限定的**
- 共有が必要なら nanostores、URL、サーバーAPI などを組み合わせる

> 参照: [Astro — Passing props to framework components](https://docs.astro.build/en/guides/framework-components/#passing-props-to-framework-components)

## Astro コンポーネント間の props

- `.astro` 同士は `Props` インターフェース + 属性で渡す
- 詳細は [Astroコンポーネントの書き方 — props, slots](./astro-components-props-slots.md) を参照

```astro
---
// src/components/UserBadge.astro
interface Props {
  name: string;
  role: string;
}
const { name, role } = Astro.props;
---
<span class="badge">{name} ({role})</span>
```

```astro
---
import UserBadge from '../components/UserBadge.astro';

const user = { name: 'Hanako', role: 'Admin' };
---
<UserBadge name={user.name} role={user.role} />
```

- ページのフロントマターで fetch したデータを、子コンポーネントへ props で流すのが基本パターン

```astro
---
import ArticleList from '../components/ArticleList.astro';

const res = await fetch('https://api.example.com/posts');
const posts = await res.json();
---
<ArticleList posts={posts} />
```

## フレームワークコンポーネントへの props

```astro
---
import Counter from '../components/Counter.jsx';

const initialCount = 10;
const label = 'いいね';
---
<Counter initialCount={initialCount} label={label} client:load />
```

### サーバー → クライアント：シリアライズ可能な props のみ

- `client:*` 付きコンポーネントへ渡す props は **ネットワーク越しに転送** される
- Astro がシリアライズできる型だけがクライアントで使える

| 渡せる | 渡せない（クライアントで使えない） |
|---|---|
| 文字列、数値、真偽値 | 関数 |
| プレーンオブジェクト | クラスインスタンス |
| 配列、`Map`、`Set` | DOM 要素 |
| `Date`、`URL`、`RegExp` | Symbol（一部除く） |
| `BigInt`、`Uint8Array` など | |

### 悪い例：関数を props で渡す

```astro
---
import SearchBox from '../components/SearchBox.jsx';

const handleSearch = (query) => {
  console.log(query);
};
---
<!-- handleSearch はクライアントで実行できない -->
<SearchBox onSearch={handleSearch} client:load />
```

### 良い例：クライアント側でロジックを完結

```jsx
// src/components/SearchBox.jsx
import { useState } from 'react';

export default function SearchBox() {
  const [query, setQuery] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    // クライアント側で処理
    window.location.href = `/search?q=${encodeURIComponent(query)}`;
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
    </form>
  );
}
```

- コールバックが必要なら、フレームワークコンポーネント内で定義する
- サーバー側の関数をクライアントに「渡して実行」はできない

## 島同士の状態共有は限定的

- React Context や Vue provide/inject は **同一フレームワークの島同士** なら動く場合がある
- ただし Astro ページ上の島は独立してハイドレーションされる
- `.astro` コンポーネントは React Context の Provider になれない

### 悪い例：Astro で Context Provider を期待する

```astro
---
import { CartProvider } from '../components/CartContext.jsx';
import CartButton from '../components/CartButton.jsx';
import CartPanel from '../components/CartPanel.jsx';
---
<!-- Astro は Provider として機能しない -->
<CartProvider client:load>
  <CartButton client:load />
  <CartPanel client:load />
</CartProvider>
```

- 子が別々の島になると Context が途切れることがある
- 設計段階で「どこまで1つの島にまとめるか」を決める

## nanostores による共有（推奨パターン）

- Astro 公式レシピでも推奨される軽量ストア
- フレームワーク非依存で、複数の島から同じ状態を読み書きできる

```js
// src/cartStore.js
import { atom } from 'nanostores';

export const isCartOpen = atom(false);
export const cartItems = atom([]);
```

```jsx
// src/components/CartButton.jsx
import { useStore } from '@nanostores/react';
import { isCartOpen } from '../cartStore';

export default function CartButton() {
  const $isCartOpen = useStore(isCartOpen);
  return (
    <button onClick={() => isCartOpen.set(!$isCartOpen)}>
      Cart
    </button>
  );
}
```

- 他の島も同じ store を import すれば状態を共有できる
- 各フレームワーク向けアダプタ（`@nanostores/react` / `@nanostores/vue` など）がある
- `.astro` の `<script>` タグからも import して subscribe できる

> 参照: [Astro — Share state between Astro components](https://docs.astro.build/en/recipes/sharing-state/) / [Share state between Islands](https://docs.astro.build/en/recipes/sharing-state-islands/)

## URL / searchParams による共有

- ページをまたがない状態なら、URL が最もシンプルな共有手段
- 検索クエリ、タブ、フィルタなど「共有可能な状態」に向く

```astro
---
// src/pages/search.astro
const query = Astro.url.searchParams.get('q') ?? '';
---
<h1>「{query}」の検索結果</h1>
```

```jsx
// src/components/SearchForm.jsx
export default function SearchForm() {
  const handleSubmit = (e) => {
    e.preventDefault();
    const q = new FormData(e.target).get('q');
    window.location.search = `?q=${encodeURIComponent(q)}`;
  };
  return (/* form */);
}
```

- サーバー側（Astro）とクライアント側（島）の両方から同じ URL を参照できる
- ブックマーク・共有・SSR との相性が良い

## 同一フレームワーク内での共有

- 1つの React コンポーネントにまとめれば、通常の React 状態管理が使える
- 近い操作は1島にまとめる方が、Context や props drilling が素直

```astro
---
import HeaderWithSearch from '../components/HeaderWithSearch.jsx';
---
<!-- 検索とナビを1島にまとめる -->
<HeaderWithSearch client:load />
```

## サーバーから取得したデータの流れ

```text
fetch / getCollection（サーバー）
        ↓ props（シリアライズ可能な値）
フレームワークコンポーネント（client:*）
        ↓ nanostores / URL / API
他の島
```

- 初期データは Astro フロントマターで取得 → props で渡す
- クライアント側の更新は nanostores や fetch API で行う
- 更新結果をサーバーと同期したい場合は API Route や Server Actions 相当の仕組みを別途用意

## よくある間違い

| 間違い | 対処 |
|---|---|
| 関数を props で渡して「動かない」 | クライアント側でロジックを定義 |
| Context が島をまたいで効かない | nanostores か1島にまとめる |
| Astro props でリアクティブ更新を期待 | フレームワーク or nanostores を使う |
| 全状態を nanostores に入れる | URL や props で足りる部分はシンプルに |

## 設計チェックリスト

- [ ] 初期データは Astro フロントマター → props で渡している
- [ ] `client:*` へ渡す props がシリアライズ可能な型だけ
- [ ] 島間の共有手段（nanostores / URL）を選んでいる
- [ ] 近い操作は1島にまとめる選択肢を検討した
- [ ] 関数やクラスインスタンスを props にしていない

## まとめ

- Astro コンポーネント間は props、フレームワークコンポーネントへも props で渡せる
- `client:*` 付きコンポーネントへの props はシリアライズ可能な値に限定される
- 島同士のリアクティブ共有は限定的。nanostores が公式推奨の定番
- URL / searchParams は検索条件など共有可能な状態に向く
- サーバーで取得 → props で初期化 → クライアントで nanostores 等で更新、が基本フロー

> 参照: [Astroコンポーネントの書き方 — props, slots](./astro-components-props-slots.md) / [client:ディレクティブ](./client-directives.md)
