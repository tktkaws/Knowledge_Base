# テンプレートリテラルの使い方

## テンプレートリテラルとは

- バッククォート（`` ` ``）で囲む文字列構文
- ES2015で追加された
- 文字列への変数埋め込み、複数行文字列、タグ付きテンプレートができる
- 従来の `'...'` / `"..."` + `+` 結合より読みやすく、ミスも減らせる

> 参照: [MDN — Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

## 基本構文

```js
const name = 'Taro';

// 悪い例：結合が続く
const message = 'Hello, ' + name + '!';

// 良い例：埋め込み
const message2 = `Hello, ${name}!`;
```

```js
`単純な文字列`

`1行目
2行目`

`値は ${expression} です`

tag`タグ付き ${expression}`
```

## 式の埋め込み（Interpolation）

- `${}` の中は任意の式を書ける
- 変数だけでなく、計算・三項演算子・関数呼び出しも可能

```js
const a = 5;
const b = 10;

console.log(`合計は ${a + b}、二倍は ${a * 2}`);
// 合計は 15、二倍は 10
```

```js
const user = { name: 'Hanako', isAdmin: true };

const label = `${user.name} (${user.isAdmin ? '管理者' : '一般'})`;
```

```js
const price = 1200;
const text = `税込 ${price.toLocaleString('ja-JP')}円`;
```

- `${}` の結果は文字列に変換される（`toString` 相当の扱い）
- オブジェクトをそのまま埋めると `[object Object]` になりやすいので注意

```js
const user = { name: 'Taro' };
console.log(`user=${user}`); // user=[object Object]
console.log(`user=${user.name}`); // user=Taro
console.log(`user=${JSON.stringify(user)}`);
```

## 複数行文字列

```js
// 悪い例：\n や結合が必要
const html =
  '<div>\n' +
  '  <p>Hello</p>\n' +
  '</div>';

// 良い例：改行をそのまま書ける
const html2 = `<div>
  <p>Hello</p>
</div>`;
```

### インデントに注意

```js
function build() {
  return `
    <section>
      <h1>Title</h1>
    </section>
  `;
}
```

- ソース上のインデント用スペースも文字列に含まれる
- 不要なら `.trim()` する、または各行の書き方を調整する

```js
const block = `
<section>
  <h1>Title</h1>
</section>
`.trim();
```

## ネストしたテンプレート

```js
const item = { isCollapsed: true };

const classes = `header ${
  isLargeScreen() ? '' : `icon-${item.isCollapsed ? 'expander' : 'collapser'}`
}`;
```

- `${}` の中にさらにテンプレートリテラルを書ける
- 複雑になりすぎる場合は、先に変数へ切り出す方が読みやすい

```js
const icon = item.isCollapsed ? 'expander' : 'collapser';
const classes2 = isLargeScreen() ? 'header' : `header icon-${icon}`;
```

## エスケープ

```js
const path = `C:\\Users\\Taro`; // バックスラッシュは \\
const withBacktick = `値は \`code\` です`; // バッククォート自体は \`
const withDollar = `金額は \${price} ではない`; // ${} を文字として出したいとき
```

## タグ付きテンプレート（Tagged templates）

- テンプレートの直前に関数を置くと、文字列と埋め込み値を分けて受け取れる
- i18n、HTMLサニタイズ、スタイルライブラリ（例: CSS-in-JS）などで使われる

```js
const person = 'Mike';
const age = 28;

function myTag(strings, personExp, ageExp) {
  const ageStr = ageExp < 100 ? 'youngster' : 'centenarian';
  return `${strings[0]}${personExp}${strings[1]}${ageStr}${strings[2]}`;
}

const output = myTag`That ${person} is a ${age}.`;
console.log(output);
// That Mike is a youngster.
```

- 第1引数 `strings` は文字列部分の配列
- 続く引数が `${}` の評価結果
- `strings.raw` でエスケープ前の生文字列にもアクセスできる

```js
function logRaw(strings) {
  console.log(strings.raw[0]);
}

logRaw`Line1\nLine2`;
// 出力は文字どおり Line1\nLine2（改行に解釈される前の形）
```

> 参照: [MDN — Tagged templates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates)

## 実務での使いどころ

### UIテキストやメッセージ

```js
const count = 3;
const message = `${count}件の通知があります`;
```

### HTML文字列（注意あり）

```js
const title = 'Hello';
const html = `<h1>${title}</h1>`;
```

- ユーザー入力をそのまま埋め込むと XSS の原因になる
- HTMLに出す値はエスケープするか、DOM API / フレームワークの安全な手段を使う

### URLやクエリ（乱用注意）

```js
const id = 42;
const url = `/api/users/${id}`;
```

- ユーザー入力を載せるなら `encodeURIComponent` も検討する

## 良い例・悪い例

```js
// 悪い例：+ が多くて追いづらい
const msg =
  'User ' + user.name + ' (' + user.id + ') logged in at ' + time;

// 良い例
const msg2 = `User ${user.name} (${user.id}) logged in at ${time}`;
```

```js
// 悪い例：テンプレートの中で重い処理や副作用を書く
const text = `結果 ${expensiveCalc()} ${saveToServer()}`;

// 良い例：先に計算してから埋め込む
const result = expensiveCalc();
saveToServer();
const text2 = `結果 ${result}`;
```

```js
// 悪い例：条件分岐がテンプレート内で巨大
const view = `
  ${items
    .map((item) => `<li>${item.name}</li>`)
    .join('')}
`;

// 可読性のため、配列生成を分けるのもあり
const listItems = items.map((item) => `<li>${item.name}</li>`).join('');
const view2 = `<ul>${listItems}</ul>`;
```

## まとめ

- テンプレートリテラルは `` ` `` で囲み、`${}` で式を埋め込める文字列構文
- 複数行・可読性の高い結合・タグ付きテンプレートが強み
- インデントや `[object Object]`、XSS には注意する
- 複雑な式はテンプレート内に詰め込みすぎず、変数に切り出す
- 通常の文字列結合より、メッセージやHTML断片の組み立てに向く
