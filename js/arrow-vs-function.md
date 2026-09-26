# アロー関数と従来の関数宣言の違い

## 関数の書き方は3系統ある

| 種類 | 例 | 特徴 |
|---|---|---|
| 関数宣言 | `function fn() {}` | 巻き上げされる |
| 関数式 | `const fn = function () {}` | 式として扱う。名前は省略可 |
| アロー関数 | `const fn = () => {}` | 短く書ける。`this` がレキシカル |

- ES2015でアロー関数が追加された
- 「短く書ける」だけでなく、**`this` の扱いが根本的に違う**点が重要

> 参照: [MDN — Arrow function expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

## 構文の違い

### 基本形

```js
// 関数宣言
function add(a, b) {
  return a + b;
}

// 関数式
const add = function (a, b) {
  return a + b;
};

// アロー関数
const add = (a, b) => {
  return a + b;
};
```

### アロー関数の省略記法

```js
// 引数が1つなら () を省略できる
const double = (n) => n * 2;
const double2 = n => n * 2;

 // 本体が式1つなら {} と return を省略できる
const sum = (a, b) => a + b;

// オブジェクトを返すときは () で囲む
const makeUser = (name) => ({ name, active: true });
```

```js
// 悪い例：{} だけだとブロック扱いになり、オブジェクトを返せない
const makeUserBad = (name) => { name }; // undefined を返す
```

## this の違い（最重要）

### 従来の関数 — 呼び出し方で this が決まる

```js
const button = {
  label: '保存',
  handleClick: function () {
    console.log(this.label);
  },
};

button.handleClick(); // '保存'

const detached = button.handleClick;
detached(); // undefined（またはグローバル）。呼び出し元が button ではない
```

```js
// コールバックで this が失われやすい
class Counter {
  constructor() {
    this.count = 0;
  }

  start() {
    setInterval(function () {
      this.count += 1; // this は Counter ではない
    }, 1000);
  }
}
```

### アロー関数 — 定義時の外側スコープの this を引き継ぐ

```js
class Counter {
  constructor() {
    this.count = 0;
  }

  start() {
    setInterval(() => {
      this.count += 1; // Counter の this を維持
    }, 1000);
  }
}
```

- アロー関数は独自の `this` を持たない（レキシカル `this`）
- `call` / `apply` / `bind` で `this` を変えられない

```js
const obj = { num: 100 };
globalThis.num = 42;

function classicAdd(a, b, c) {
  return this.num + a + b + c;
}

const arrowAdd = (a, b, c) => this.num + a + b + c;

classicAdd.call(obj, 1, 2, 3); // 106
arrowAdd.call(obj, 1, 2, 3);   // 48（obj には切り替わらない）
```

## arguments / new / メソッドとしての向き不向き

| 項目 | 従来の関数 | アロー関数 |
|---|---|---|
| 独自の `arguments` | ある | ない（外側を参照） |
| `new` で生成 | できる | できない（`TypeError`） |
| メソッド定義 | 向く | 基本向かない |
| `prototype` | ある | ない |
| ジェネレータ（`yield`） | 可能 | 不可 |

```js
function showArgs() {
  console.log(arguments); // Arguments オブジェクト
}

const showArgsArrow = () => {
  console.log(arguments); // 外側になければ ReferenceError
};

// 可変長引数はレスト構文を使う
const sumAll = (...nums) => nums.reduce((a, b) => a + b, 0);
```

```js
const Person = (name) => {
  this.name = name;
};
new Person('Taro'); // TypeError: Person is not a constructor
```

## オブジェクトメソッドでの落とし穴

```js
// 悪い例：メソッドをアローにすると this がオブジェクトにならないことが多い
const user = {
  name: 'Taro',
  greet: () => {
    console.log(`Hello, ${this.name}`); // this.name は期待どおりにならない
  },
};

user.greet();
```

```js
// 良い例：メソッドは従来の関数（またはメソッド短縮記法）
const user = {
  name: 'Taro',
  greet() {
    console.log(`Hello, ${this.name}`);
  },
};

user.greet(); // Hello, Taro
```

```js
// クラスフィールドのアローは「インスタンスに束縛したい」ときに使うパターン
class Button {
  label = '保存';

  // コールバックに渡しても this がずれにくい
  handleClick = () => {
    console.log(this.label);
  };
}
```

## 巻き上げの違い

```js
hoisted(); // OK
function hoisted() {
  console.log('関数宣言は巻き上げられる');
}

notHoisted(); // ReferenceError（TDZ）
const notHoisted = () => {
  console.log('関数式・アローは const/let のルールに従う');
};
```

- 関数宣言はスコープ全体で呼び出せる
- `const fn = () => {}` は変数と同じく、宣言より前では使えない

## いつアロー関数を使うか

### 向く場面

- `map` / `filter` / `reduce` などの短いコールバック
- `setTimeout` / `addEventListener` で外側の `this` を保ちたいとき
- 値を返す小さな純関数

```js
const ids = users.map((user) => user.id);

element.addEventListener('click', () => {
  this.close(); // クラスメソッド内など、外側の this を使いたいとき
});
```

### 向かない場面

- オブジェクトやクラスのメソッド（短縮記法 / 通常のメソッドを使う）
- コンストラクタが必要なとき
- `arguments` や `new.target` が必要なとき
- 意図的に動的な `this` を使いたいとき

## 良い例・悪い例

```js
// 悪い例：迷いなく全部アローにする
const api = {
  baseUrl: '/api',
  get: () => fetch(this.baseUrl), // this.baseUrl が壊れる
};
```

```js
// 良い例：役割で使い分ける
const api = {
  baseUrl: '/api',
  get() {
    return fetch(this.baseUrl);
  },
};

const doubles = [1, 2, 3].map((n) => n * 2);
```

## まとめ

- アロー関数は短い構文に加え、**レキシカル `this`** が本質
- 従来の関数は呼び出し方で `this` が変わる。メソッドやコンストラクタ向き
- アローは `new` 不可、独自 `arguments` なし、メソッドには基本使わない
- コールバックや短い変換処理はアロー、オブジェクトの振る舞い定義は従来の関数 / メソッド記法
- 「短く書けるから」だけで選ばず、`this` の必要性で選ぶ
