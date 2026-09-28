# スプレッド構文とレスト構文

## `...` は同じ見た目で役割が逆

| 名前 | 役割 | 典型的な場所 |
|---|---|---|
| スプレッド（spread） | 展開する | 配列リテラル、オブジェクトリテラル、関数呼び出し |
| レスト（rest） | まとめる | 関数の仮引数、分割代入の末尾 |

- どちらも `...` を使うが、**文脈で意味が変わる**
- スプレッドは「ばらす」、レストは「残りを集める」

> 参照: [MDN — Spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)

## スプレッド構文

### 配列を展開する

```js
const base = [1, 2, 3];

// コピー（浅い）
const copy = [...base];

// 結合
const merged = [...base, 4, 5];
const front = [0, ...base];

// 関数引数へ展開
Math.max(...base); // 3
```

```js
// 悪い例：apply を無理に使う
Math.max.apply(null, base);

// 良い例
Math.max(...base);
```

### オブジェクトを展開する

```js
const defaults = { theme: 'light', lang: 'ja' };
const overrides = { theme: 'dark' };

const cloned = { ...defaults };
const settings = { ...defaults, ...overrides };
// { theme: 'dark', lang: 'ja' } — 後ろが優先
```

- 自分自身の列挙可能なプロパティをコピーする（浅いコピー）
- 同じキーは**右側が上書き**する

### 文字列なども展開できる

```js
const chars = [...'abc']; // ['a', 'b', 'c']
```

- 反復可能（iterable）なものなら配列スプレッドの対象になる

## レスト構文

### 関数のレストパラメータ

```js
function sum(...nums) {
  return nums.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3); // 6
```

```js
function greet(greeting, ...names) {
  return `${greeting} ${names.join(', ')}`;
}

greet('Hello', 'Taro', 'Hanako'); // 'Hello Taro, Hanako'
```

- レストパラメータは**引数リストの末尾だけ**
- 受け取った値は本物の配列（`arguments` オブジェクトではない）

```js
// 悪い例：末尾以外にレストを置く
// function bad(...rest, last) {}
```

> 参照: [MDN — Rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)

### 分割代入でのレスト

```js
const [first, ...restItems] = [10, 20, 30, 40];
// first = 10, restItems = [20, 30, 40]

const { id, ...profile } = { id: 1, name: 'Taro', role: 'admin' };
// id = 1, profile = { name: 'Taro', role: 'admin' }
```

- 分割代入のレストも末尾に置く
- こちらも浅いコピー。ネストしたオブジェクトの参照は共有される

> 参照: [MDN — Destructuring（rest）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring#rest_properties_and_rest_elements)

## 見分け方

```js
// スプレッド：すでに存在する配列/オブジェクトを「ここに展開」
const all = [...a, ...b];
fn(...args);
const obj = { ...base, extra: true };

// レスト：まだ束ねられていないものを「変数に集める」
const [head, ...tail] = list;
const { a, ...others } = data;
function f(...args) {}
```

- 「右辺や呼び出し側でばらしている」→ スプレッド
- 「左辺や仮引数でまとめている」→ レスト

## 実務パターン

### イミュータブルな更新（浅い）

```js
const todos = [{ id: 1, done: false }];

// 配列に追加
const added = [...todos, { id: 2, done: false }];

// オブジェクトの一部更新
const user = { name: 'Taro', age: 20 };
const updated = { ...user, age: 21 };
```

### デフォルト設定のマージ

```js
function createConfig(options = {}) {
  return {
    retries: 3,
    timeout: 1000,
    ...options,
  };
}
```

### React の props 転送（イメージ）

```jsx
function Button({ className, ...rest }) {
  return <button className={`btn ${className}`} {...rest} />;
}
```

### 不要プロパティを除いたコピー

```js
const { password, ...safeUser } = user;
sendToClient(safeUser);
```

## 浅いコピーであることの注意

```js
const original = {
  name: 'Taro',
  address: { city: 'Tokyo' },
};

const copied = { ...original };
copied.address.city = 'Osaka';

console.log(original.address.city); // 'Osaka' — ネストは共有される
```

```js
// ネストまでコピーしたい場合の例
const deep = structuredClone(original);
```

- スプレッド / レストはディープクローンではない
- ネストした参照まで切り離すなら `structuredClone()` などを検討する

## よくある間違い

### 1. スプレッドとレストを混同する

```js
// これはレスト（まとめ）
function f(...args) {}

// これはスプレッド（展開）
f(...[1, 2, 3]);
```

### 2. 深いコピーだと思い込む

- ネストしたオブジェクトや配列は参照が残る

### 3. レストを末尾以外に書く

```js
// SyntaxError
// const [ ...rest, last ] = list;
// const { ...rest, id } = user;
```

### 4. null / undefined をオブジェクトスプレッドするつもりで失敗する

```js
const a = { ...null }; // {} になる（例外にはならない）
const b = { ...undefined }; // {}

// 配列スプレッドは iterable が必要
// [...null]; // TypeError
```

### 5. 配列のマージで参照を共有したまま破壊的に変更する

```js
const a = [1, 2];
const b = a;
b.push(3); // a も変わる

const c = [...a]; // 浅いが、トップレベルは別配列
```

## まとめ

- `...` は文脈でスプレッド（展開）とレスト（集約）に分かれる
- スプレッドは配列・オブジェクトのコピーやマージ、引数展開に使う
- レストは可変長引数や「残り」の受け取りに使う
- どちらも浅い操作。深い複製が必要なら別手段を使う
- 分割代入・イミュータブル更新・props 転送と組み合わせると実務で特に有用
