# map / filter / reduce の使い分け

## 3つの役割を一言で

| メソッド | 役割 | 戻り値のイメージ |
|---|---|---|
| `map` | 各要素を変換する | 同じ長さの新しい配列 |
| `filter` | 条件に合う要素だけ残す | 同じか短い新しい配列 |
| `reduce` | 配列を1つの値（や構造）に畳み込む | 数値・オブジェクト・配列など何でも可 |

- いずれも**元の配列は変更しない**（新しい結果を返す）
- `for` ループでも同じことはできるが、意図がメソッド名で伝わるのが利点

> 参照: [MDN — Indexed collections](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections)

## map — 変換する

```js
const nums = [1, 2, 3];
const doubled = nums.map((n) => n * 2);
// [2, 4, 6]
```

```js
const users = [
  { id: 1, name: 'Taro' },
  { id: 2, name: 'Hanako' },
];

const names = users.map((user) => user.name);
// ['Taro', 'Hanako']

const options = users.map((user) => ({
  value: user.id,
  label: user.name,
}));
```

### 向くとき

- 要素数は変えず、形や値だけ変えたい
- UI用のデータ形に整えたい
- `id` 一覧など、プロパティだけ抜き出したい

### 向かないとき

- 要素を減らしたい → `filter`
- 合計や集計をしたい → `reduce`（または専用メソッド）

```js
// 悪い例：map の中で副作用だけ行う（戻り値を使わない）
users.map((user) => {
  sendAnalytics(user.id);
});

// 良い例：副作用は forEach / for...of
users.forEach((user) => {
  sendAnalytics(user.id);
});
```

> 参照: [MDN — Array.prototype.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

## filter — 絞り込む

```js
const values = [12, 5, 8, 130, 44];
const big = values.filter((value) => value >= 10);
// [12, 130, 44]
```

```js
const mixed = ['a', 10, 'b', 20];
const onlyNumbers = mixed.filter((item) => typeof item === 'number');
// [10, 20]
```

```js
const activeUsers = users.filter((user) => user.active);
```

### 向くとき

- 条件に合うものだけ残したい
- 検索・タブ切替・権限による一覧の絞り込み

### コールバックの戻り値

- truthy なら残る、falsy なら除外
- 明示的に `true` / `false` を返すと読みやすい

```js
// 分かりにくい例
items.filter((item) => item.score);

// 良い例
items.filter((item) => item.score > 0);
```

> 参照: [MDN — Array.prototype.filter()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

## reduce — 畳み込む

```js
const nums = [5, 6, 13, 0, 1];
const sum = nums.reduce((total, n) => total + n, 0);
// 25
```

- 第1引数: コールバック `(accumulator, currentValue, index, array)`
- 第2引数: 初期値（省略すると先頭要素が初期値になり、混乱しやすい）
- **初期値は基本的に渡す**

```js
const cart = [
  { name: 'Book', price: 1200 },
  { name: 'Pen', price: 150 },
];

const total = cart.reduce((sum, item) => sum + item.price, 0);
```

### オブジェクトへの集約

```js
const posts = [
  { id: 1, tag: 'js' },
  { id: 2, tag: 'css' },
  { id: 3, tag: 'js' },
];

const byTag = posts.reduce((acc, post) => {
  acc[post.tag] ??= [];
  acc[post.tag].push(post);
  return acc;
}, {});
// { js: [...], css: [...] }
```

### 向くとき

- 合計・最大値・文字列結合など「1つの結果」にまとめたい
- 配列からオブジェクトへの変換など、形が大きく変わる処理

### 無理に reduce しなくてよい場合

| やりたいこと | より適切な手段 |
|---|---|
| 条件で減らすだけ | `filter` |
| 変換だけ | `map` |
| 最初の一致を探す | `find` / `findIndex` |
| 存在判定 | `some` / `every` |
| 配列の配列を平坦化 | `flat` / `flatMap` |
| 重複除去 | `[...new Set(array)]` |
| プロパティでグループ化 | `Object.groupBy()`（対応環境） |

> 参照: [MDN — Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

## チェーンでの組み合わせ

```js
const users = [
  { name: 'Taro', age: 28, active: true },
  { name: 'Hanako', age: 17, active: true },
  { name: 'Jiro', age: 32, active: false },
];

const adultNames = users
  .filter((user) => user.active)
  .filter((user) => user.age >= 18)
  .map((user) => user.name);
// ['Taro']
```

```js
const totalActiveAge = users
  .filter((user) => user.active)
  .reduce((sum, user) => sum + user.age, 0);
```

### 読みやすさのコツ

1. 先に `filter` で対象を絞る
2. 次に `map` で形を整える
3. 最後に必要なら `reduce` で集計する
4. 1行が長くなったら中間変数に切り出す

```js
// 悪い例：1つの reduce に絞り込み・変換・集計を全部詰める
const result = users.reduce((acc, user) => {
  if (user.active && user.age >= 18) {
    acc.push(user.name.toUpperCase());
  }
  return acc;
}, []);

// 良い例：意図ごとに分ける
const result2 = users
  .filter((user) => user.active && user.age >= 18)
  .map((user) => user.name.toUpperCase());
```

## for との使い分け

```js
// 単純な集計や早期 return が絡むなら for も十分あり
let max = -Infinity;
for (const n of nums) {
  if (n > max) max = n;
}
```

- 「変換・絞り込み・畳み込み」が明確なら `map` / `filter` / `reduce`
- 複雑な分岐・途中での中断・複数の累積変数なら `for...of` の方が読みやすいこともある
- パフォーマンスが極端に気になる巨大配列では、不要な中間配列を避ける設計も検討する

## よくある間違い

### 1. map で filter 相当のことをする

```js
// 悪い例：欠番や穴ができやすい
const result = nums.map((n) => (n > 0 ? n : null)).filter(Boolean);
```

### 2. reduce の初期値忘れ

```js
[].reduce((a, b) => a + b); // TypeError
[].reduce((a, b) => a + b, 0); // 0
```

### 3. 元配列を破壊する

```js
// 悪い例
items.filter((item) => {
  item.checked = true; // 元オブジェクトを書き換えている
  return item.active;
});
```

- `map` / `filter` / `reduce` は配列自体は新しいが、要素オブジェクトの参照は共有されうる
- 要素もイミュータブルに扱うなら、新しいオブジェクトを返す

### 4. 非同期処理を map で待つつもりになる

```js
// 悪い例：Promise の配列が返るだけ
const data = ids.map(async (id) => fetchUser(id));

// 良い例
const data2 = await Promise.all(ids.map((id) => fetchUser(id)));
```

## 使い分けチートシート

| 欲しい結果 | 使うもの |
|---|---|
| 同じ件数で別の形 | `map` |
| 条件に合う要素の配列 | `filter` |
| 合計・集計・1つの構造 | `reduce`（または専用API） |
| 副作用だけ | `forEach` / `for...of` |
| 1件見つける | `find` |

## まとめ

- `map` は変換、`filter` は絞り込み、`reduce` は畳み込み
- まずは「何を返したいか」で選ぶ
- チェーンするなら filter → map → reduce の順が分かりやすい
- `reduce` は強力だが、専用メソッドで足りるならそちらを優先する
- 元配列を変えず、要素の意図しない破壊的更新にも注意する
