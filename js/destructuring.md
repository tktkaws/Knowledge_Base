# 分割代入(Destructuring)の基本と活用

## 分割代入とは

- 配列やオブジェクトから値を取り出して、個別の変数に代入する構文
- ES2015で追加された
- プロパティアクセスの繰り返しを減らし、必要な値だけを明示できる
- 関数の引数、Reactのprops、APIレスポンスの受け取りなどで頻出

> 参照: [MDN — Destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring)

## 配列の分割代入

### 基本

```js
const rgb = [255, 128, 0];

// 悪い例：インデックスを繰り返す
const r = rgb[0];
const g = rgb[1];
const b = rgb[2];

// 良い例
const [r, g, b] = rgb;
```

### 飛ばす・残りをまとめる

```js
const scores = [100, 80, 60, 40];

const [first, , third] = scores; // 2番目をスキップ
const [top, ...others] = scores; // top=100, others=[80, 60, 40]
```

### デフォルト値

```js
const [a = 0, b = 0] = [10];
console.log(a, b); // 10, 0
```

- デフォルトが使われるのは値が `undefined` のとき
- `null` にはデフォルトは適用されない

```js
const [x = 1] = [null];
console.log(x); // null
```

### 変数の入れ替え

```js
let left = 'A';
let right = 'B';

[left, right] = [right, left];
console.log(left, right); // B, A
```

## オブジェクトの分割代入

### 基本

```js
const user = { id: 1, name: 'Taro', role: 'admin' };

// 悪い例
const id = user.id;
const name = user.name;

// 良い例：プロパティ名と同じ変数名で取り出す
const { id, name } = user;
```

### 別名（リネーム）

```js
const { name: userName, role: userRole } = user;
console.log(userName, userRole); // Taro, admin
```

### デフォルト値と併用

```js
const { name, age = 0, role: userRole = 'guest' } = {
  name: 'Hanako',
};

console.log(name, age, userRole); // Hanako, 0, guest
```

### 残りのプロパティ

```js
const { id, ...profile } = user;
// id = 1
// profile = { name: 'Taro', role: 'admin' }
```

### 宣言なしで代入するとき

```js
let name;
let role;

// 悪い例：{ がブロックと解釈される
// { name, role } = user;

// 良い例：全体を () で囲む
({ name, role } = user);
```

## ネストした構造

```js
const response = {
  data: {
    user: {
      name: 'Taro',
      tags: ['js', 'css'],
    },
  },
};

const {
  data: {
    user: {
      name,
      tags: [firstTag],
    },
  },
} = response;

console.log(name, firstTag); // Taro, js
```

```js
// 途中のオブジェクトも変数に残したい場合
const {
  data: {
    user,
    user: { name },
  },
} = response;
```

- 深く掘りすぎると読みにくくなる
- 2段階以上ネストするなら、段階を分けるか、必要な部分だけ先に取り出す

## 関数引数での活用

```js
// 悪い例：順番と意味が分かりにくい
function createCard(title, body, isDraft, maxLength) {}

// 良い例：オブジェクト引数 + 分割代入
function createCard({ title, body, isDraft = false, maxLength = 100 }) {
  return { title, body, isDraft, maxLength };
}

createCard({ title: 'Hello', body: '...' });
```

### 引数自体が省略されたとき

```js
// 引数未指定でも落ちないように、オブジェクト全体のデフォルトも置く
function preFilledObject({ z = 3 } = {}) {
  return z;
}

preFilledObject(); // 3
preFilledObject({}); // 3
preFilledObject({ z: 2 }); // 2
```

```js
function preFilledArray([x = 1, y = 2] = []) {
  return x + y;
}

preFilledArray(); // 3
preFilledArray([2]); // 4
```

> 参照: [MDN — Default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)

## 実務でよく使うパターン

### APIレスポンス

```js
const { data, error, status } = await getUser(id);

if (error) {
  throw error;
}
```

### React の props（イメージ）

```jsx
function UserBadge({ name, avatarUrl, isActive = false }) {
  return (
    <div data-active={isActive}>
      <img src={avatarUrl} alt="" />
      <span>{name}</span>
    </div>
  );
}
```

### 配列を返すフックや関数

```js
const [value, setValue] = useState(0);
const [first, second] = getPair();
```

### ループ

```js
for (const [key, value] of Object.entries(settings)) {
  console.log(key, value);
}

users.forEach(({ id, name }) => {
  console.log(id, name);
});
```

## 注意点と落とし穴

### null / undefined は分割できない

```js
const user = null;
const { name } = user; // TypeError
```

```js
// 良い例：先にガードする / デフォルトを置く
const { name } = user ?? {};
```

### 存在しないプロパティは undefined

```js
const { email } = { name: 'Taro' };
console.log(email); // undefined
```

### 計算プロパティ名

```js
const key = 'title';
const { [key]: heading } = { title: 'Docs' };
console.log(heading); // Docs
```

## 良い例・悪い例

```js
// 悪い例：使わない値まで全部取り出す
const { a, b, c, d, e, f } = hugeObject;
doSomething(a, c);
```

```js
// 良い例：使うものだけ取り出す
const { a, c } = hugeObject;
doSomething(a, c);
```

```js
// 悪い例：ネスト分割が長すぎて追いづらい
const {
  page: {
    content: {
      hero: { title },
    },
  },
} = cms;

// 良い例：段階的に取り出す
const hero = cms.page?.content?.hero;
const title = hero?.title;
```

## まとめ

- 分割代入は、配列・オブジェクトから必要な値だけを取り出す構文
- 配列は順番、オブジェクトはプロパティ名が対応の基準
- デフォルト値・別名・レスト・ネスト・関数引数で表現力が高い
- `null` / `undefined` の分割や、深すぎるネストには注意する
- 「何を使うか」がコード上で明示されるのが最大のメリット
