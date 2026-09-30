# Optional Chaining(?.)とNullish Coalescing(??)

## なぜこの2つがセットで語られるか

- どちらも「値が無いとき」を安全に扱うための演算子
- `?.` は途中が `null` / `undefined` でもエラーにせずアクセスを止める
- `??` は `null` / `undefined` のときだけデフォルト値に切り替える
- APIレスポンスや設定オブジェクトなど、存在しないプロパティが多い場面で組み合わせると強力

> 参照: [MDN — Optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)、[MDN — Nullish coalescing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)

## Optional Chaining（`?.`）

### 基本

```js
const user = { profile: { name: 'Taro' } };

// 悪い例：途中が無いと TypeError
// user.address.city;

// 良い例
user.address?.city; // undefined
user.profile?.name; // 'Taro'
```

- 左側が `null` または `undefined` なら、そこで評価を止め `undefined` を返す
- それ以外の falsy（`0` / `''` / `false`）では止まらない

```js
const count = 0;
count?.toFixed(2); // '0.00' — 0 でも続行される
```

### 使える場所

```js
obj?.prop; // プロパティ
obj?.[key]; // 計算プロパティ
arr?.[0]; // 配列要素
fn?.(arg); // 関数呼び出し
obj.method?.(); // メソッド呼び出し
```

```js
const key = 'email';
const users = [{ name: 'A' }];

users[0]?.[key]; // undefined
document.querySelector('.missing')?.classList.add('on');

const maybeFn = Math.random() > 0.5 ? () => 'ok' : null;
maybeFn?.(); // 関数でないときは呼ばない
```

### 連鎖できる

```js
const city = response?.data?.user?.address?.city;
```

- 長いチェーンは便利だが、何が欠けているか分かりにくくなることもある
- 重要な欠落は、明示的な存在チェックやエラー処理と併用する

## Nullish Coalescing（`??`）

### 基本

```js
null ?? 'default'; // 'default'
undefined ?? 'default'; // 'default'
0 ?? 'default'; // 0
'' ?? 'default'; // ''
false ?? 'default'; // false
```

- 左側が `null` または `undefined` のときだけ右側を返す
- それ以外（`0` / `''` / `false` など）は左側をそのまま返す

### `||` との違い

```js
const volumeA = 0 || 50; // 50 — 0 が「無し」扱いになってしまう
const volumeB = 0 ?? 50; // 0 — 有効な値として残る

const titleA = '' || '無題'; // '無題'
const titleB = '' ?? '無題'; // '' — 空文字を許可したいときに使う
```

| 演算子 | フォールバックする条件 | 向くケース |
|---|---|---|
| `\|\|` | falsy全般（`0`/`''`/`false`/`NaN`/`null`/`undefined`） | 「空っぽっぽいもの全部」を弾きたい |
| `??` | `null` / `undefined` のみ | `0` や空文字も正当な値として残したい |

```js
// 設定値のデフォルトは ?? が安全なことが多い
function createConfig(options = {}) {
  return {
    retries: options.retries ?? 3,
    timeout: options.timeout ?? 1000,
    debug: options.debug ?? false,
  };
}
```

## 組み合わせパターン

```js
const foo = { someFooProp: 'hi' };

foo.someFooProp?.toUpperCase() ?? 'not available'; // 'HI'
foo.someBarProp?.toUpperCase() ?? 'not available'; // 'not available'
```

```js
const name = user?.profile?.displayName ?? 'ゲスト';
const firstTag = post?.tags?.[0] ?? '未分類';
```

- 「安全に辿る」→ `?.`
- 「無かったときの決め打ち」→ `??`
- この順番（アクセス → デフォルト）が基本形

## 短絡評価

```js
a?.b.c; // a が nullish なら b.c は評価されない
a ?? (b || c); // a が nullish でないなら右側は評価されない
```

```js
// 右側に副作用がある場合、必要になるまで実行されない
const value = cached ?? fetchFromServer(); // cached があれば fetch しない
```

## 構文上の注意

### `??` と `||` / `&&` の混在には括弧が必要

```js
// SyntaxError
// null || undefined ?? 'foo';
// true && undefined ?? 'foo';

// 良い例：意図を括弧で明示
(null || undefined) ?? 'foo';
true && (undefined ?? 'foo');
```

### 代入との組み合わせ

```js
let port;
port ??= 3000; // port が nullish のときだけ代入（論理代入）
```

- `??=` / `||=` / `&&=` は短い更新記法
- 「無記入のときだけ初期化」には `??=` が向く

## 実務での使いどころ

### APIレスポンス

```js
const email = payload?.user?.contacts?.email ?? '';
const items = payload?.data?.items ?? [];
```

### DOM

```js
document.getElementById('toast')?.remove();
```

### オプション引数

```js
function connect({ host, port, secure } = {}) {
  const finalHost = host ?? 'localhost';
  const finalPort = port ?? 8080;
  const finalSecure = secure ?? true;
}
```

## 良い例・悪い例

```js
// 悪い例：|| で 0 を潰す
const limit = options.limit || 10; // limit: 0 が来ると 10 になる

// 良い例
const limit2 = options.limit ?? 10;
```

```js
// 悪い例：毎回冗長なガード
const city =
  user && user.address && user.address.city
    ? user.address.city
    : '不明';

// 良い例
const city2 = user?.address?.city ?? '不明';
```

```js
// 悪い例：エラーを握りつぶして常にデフォルト
const price = product?.price ?? 0;
// 在庫なしと価格0を区別できないなら、設計を見直す

// 良い例：必要な欠落は明示的に扱う
if (product?.price == null) {
  throw new Error('price is required');
}
```

```js
// 悪い例：メソッドが無いのに ?. だけで済ませ、何も起きないバグに気づかない
form?.submit?.();

// 良い例：重要処理は存在確認や型をはっきりさせる
if (typeof form?.submit === 'function') {
  form.submit();
}
```

## まとめ

- `?.` は `null` / `undefined` のときに安全にアクセスを打ち切る
- `??` は `null` / `undefined` のときだけデフォルトへ切り替える
- `||` と違い、`0` や `''` を正当な値として残せるのが `??` の強み
- `user?.profile?.name ?? 'ゲスト'` のように組み合わせて使う
- すべてを黙ってデフォルトにするのではなく、必須データの欠落は別途検知する
