# var / let / const の違いとスコープ

## なぜ3つあるのか

- もともとの変数宣言は `var` のみだった
- ES2015（ES6）で `let` / `const` が追加され、スコープと再代入の制御が明確になった
- 現代のフロントエンドでは **`const` を基本に、再代入が必要なときだけ `let`** が一般的
- `var` はレガシーコード理解のために知る必要はあるが、新規コードでは使わない

> 参照: [MDN — Grammar and types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types)

## 比較一覧

| 項目 | `var` | `let` | `const` |
|---|---|---|---|
| スコープ | 関数スコープ（またはグローバル） | ブロックスコープ | ブロックスコープ |
| 再宣言 | 同じスコープで可能 | 同じスコープで不可 | 同じスコープで不可 |
| 再代入 | 可能 | 可能 | 不可 |
| 初期化必須 | 不要（`undefined`） | 不要（未初期化は TDZ） | **必須** |
| 宣言前アクセス | `undefined`（巻き上げ） | `ReferenceError`（TDZ） | `ReferenceError`（TDZ） |

## スコープの違い

### 関数スコープ（var）

- `var` は関数全体（またはグローバル）がスコープ
- `if` / `for` の `{}` ではスコープが閉じない

```js
function example() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10 — if の外でも見える
}
```

### ブロックスコープ（let / const）

- `let` / `const` は `{}` で囲まれたブロックがスコープ
- `if` / `for` / 単独ブロックの外からは参照できない

```js
function example() {
  if (true) {
    let x = 10;
    const y = 20;
  }
  console.log(x); // ReferenceError
  console.log(y); // ReferenceError
}
```

### よくある対比

```js
var a = 1;
let b = 1;

if (true) {
  var a = 2; // 外側の a を上書き
  let b = 2; // ブロック内だけの b
}

console.log(a); // 2
console.log(b); // 1
```

> 参照: [MDN — block](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/block)

## 巻き上げ（Hoisting）と TDZ

### var の巻き上げ

- 宣言がスコープ先頭に「持ち上げ」られる
- 代入は元の位置まで行われないため、宣言前は `undefined`

```js
console.log(count); // undefined
var count = 3;
console.log(count); // 3
```

実質的には次のように解釈される。

```js
var count;
console.log(count); // undefined
count = 3;
```

### Temporal Dead Zone（TDZ）

- `let` / `const` は、スコープ開始から初期化まで「一時的に死んでいる領域」（TDZ）に入る
- TDZ中にアクセスすると `ReferenceError`
- バグを早期に見つけやすくするための仕様

```js
{
  // ここから foo の TDZ 開始
  console.log(bar); // undefined（var）
  console.log(foo); // ReferenceError
  var bar = 1;
  let foo = 2; // ここで foo の TDZ 終了
}
```

> 参照: [MDN — let（Temporal dead zone）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz)

## 再代入と再宣言

### 再代入

```js
let score = 0;
score = 10; // OK

const max = 100;
max = 200; // TypeError: Assignment to constant variable
```

### 再宣言

```js
var name = 'A';
var name = 'B'; // OK（意図しない上書きが起きやすい）

let age = 20;
let age = 21; // SyntaxError

const id = 1;
const id = 2; // SyntaxError
```

## const でも中身は変えられる

- `const` が固定するのは**束縛（変数と値の対応）**であり、オブジェクトの中身ではない
- 配列やオブジェクトのプロパティ変更は可能

```js
const user = { name: 'Taro' };
user.name = 'Hanako'; // OK
user = {}; // TypeError（再代入は不可）

const tags = ['js'];
tags.push('css'); // OK
tags = []; // TypeError
```

```js
// オブジェクト自体も変えられたくない場合
const config = Object.freeze({ apiUrl: '/api' });
config.apiUrl = '/v2'; // 厳密モードではエラー、通常は無視されることが多い
```

- 深いネストまで凍結したい場合は `structuredClone` + 再帰 freeze や専用ライブラリが必要
- 「再代入しない」意図を示すために、まずは `const` を使う

## for ループでの違い

```js
// 悪い例：var だとクロージャで同じ i を共有しやすい
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 3, 3, 3
```

```js
// 良い例：let は反復ごとに別の束縛
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 0, 1, 2
```

```js
// const も for...of ではよく使う
for (const item of ['a', 'b', 'c']) {
  console.log(item);
}
```

## 使い分けの指針

1. まず `const` で宣言する
2. 後から値を入れ替える必要が分かったら `let` に変える
3. 新規コードで `var` は使わない
4. ループカウンタや累積値など、再代入が自然なときだけ `let`

```js
// 良い例
const users = fetchUsers();
let total = 0;

for (const user of users) {
  total += user.score;
}
```

```js
// 避けたい例：全部 let / 全部 var
let title = 'Home'; // 再代入しないなら const
var count = 0; // 新規なら let / const
```

## グローバルスコープでの注意

```js
var globalA = 1; // ブラウザでは window.globalA になることがある
let globalB = 2; // グローバルオブジェクトのプロパティにはならない
const globalC = 3; // 同上
```

- モジュール（`import` / `export`）内のトップレベル `let` / `const` はモジュールスコープ
- 意図しないグローバル汚染を避けるためにも、`var` とスクリプト直書きの組み合わせは避ける

## よくある間違い

### 1. const は「完全イミュータブル」だと思い込む

```js
const list = [1];
list.push(2); // 動く — 再代入禁止であって破壊的変更禁止ではない
```

### 2. 宣言前に let / const を読む

```js
console.log(value);
let value = 1; // ReferenceError（TDZ）
```

### 3. ブロックスコープのつもりで var を使う

```js
if (user) {
  var message = 'ok';
}
console.log(message); // 外に漏れる
```

### 4. 再代入しない値を let にする

- 読み手が「あとで変わる変数」だと誤解する
- 変わらないなら `const`

## まとめ

- `var` は関数スコープ + 巻き上げ。新規コードでは使わない
- `let` / `const` はブロックスコープ + TDZ。意図しない参照を防げる
- 基本は `const`、再代入が必要なら `let`
- `const` は再代入不可。オブジェクトや配列の中身変更は別問題
- `for` とクロージャでは `let` / `const` の挙動差を特に意識する
