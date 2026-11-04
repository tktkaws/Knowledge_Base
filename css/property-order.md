# CSSプロパティの順番を整理する — VS CodeとPostCSS

## プロパティの順番を揃えるとは

- 宣言ブロックの中で、`position` や `display` などのプロパティが並ぶ順を揃えること
- 見た目や詳細度は変わらない — 変わるのは読みやすさと、差分の見やすさ
- やり方は2つ
  - VS Code上で、開いているファイルをその場で並べる
  - ビルド時にPostCSSで並べる
- どちらも中身は `postcss-sorting`

> 参照: [postcss-sorting](https://github.com/hudochenkov/postcss-sorting)

## VS Codeプラグインで行う方法

- 拡張機能は PostCSS Sorting（`mrmlnc.vscode-postcss-sorting`）
- 拡張機能ビューで `postcss-sorting` を検索して入れる
- コマンドパレットから `PostCSS Sorting: Run` を実行すると、開いているCSSが並び替わる
- 順の指定は、ワークスペースの `.vscode/settings.json` に `postcssSorting.config` を書く
- `properties-order` に、上から並べたいプロパティ名を書く

```json
{
  "postcssSorting.config": {
    "properties-order": [
      "position",
      "top",
      "right",
      "bottom",
      "left",
      "z-index",
      "display",
      "width",
      "height",
      "margin",
      "padding",
      "color",
      "background"
    ]
  }
}
```

```css
/* 変換前 */
.card {
  color: #333;
  position: relative;
  display: flex;
  z-index: 1;
}

/* 変換後 */
.card {
  position: relative;
  z-index: 1;
  display: flex;
  color: #333;
}
```

- 配列に無いプロパティは、デフォルトではブロックの末尾に回る

> 参照: [PostCSS Sorting](https://marketplace.visualstudio.com/items?itemName=mrmlnc.vscode-postcss-sorting)

## PostCSSで行う方法

- 同じ順を、コマンドを手で打たずにビルドへ載せる場合は `postcss-sorting` を使う
- 前提は Node.js と、npm scripts や Vite などのビルドがあること

```bash
npm install -D postcss postcss-cli postcss-sorting
```

- `properties-order` の指定は2通り
  - `"alphabetical"` — アルファベット順
  - 配列 — position、display、幅と余白、色のように自分でグループを決める
- 配列は、VS Code側と同じ順にする — 結果も前節の変換後と同じになる
- 配列に無いプロパティは末尾に回す（`unspecified-properties-position: 'bottom'`）

```js
module.exports = {
  plugins: {
    'postcss-sorting': {
      'properties-order': [
        'position',
        'top',
        'right',
        'bottom',
        'left',
        'z-index',
        'display',
        'width',
        'height',
        'margin',
        'padding',
        'color',
        'background',
      ],
      'unspecified-properties-position': 'bottom',
    },
  },
};
```

```js
// アルファベット順にする場合は、配列の代わりに文字列を渡す
module.exports = {
  plugins: {
    'postcss-sorting': {
      'properties-order': 'alphabetical',
    },
  },
};
```

```bash
npx postcss src/**/*.css --dir dist
```

- Viteを使っている場合は、同じ `postcss.config.js` を置けばビルド時に読まれる
- 拡張機能とPostCSSで同じ順を使うなら、配列を `.postcss-sorting.json` に置く
  - 拡張機能は `.postcss-sorting.json` を自動で読む — `settings.json` の `postcssSorting.config` は不要になる
  - PostCSS側は `postcss.config.js` からそのファイルを読む

```js
module.exports = {
  plugins: {
    'postcss-sorting': require('./.postcss-sorting.json'),
  },
};
```

> 参照: [postcss-cli](https://github.com/postcss/postcss-cli) / [postcss-sorting — properties-order](https://github.com/hudochenkov/postcss-sorting/blob/master/lib/properties-order/README.md) / [Vite — CSS](https://vitejs.dev/guide/features.html#css)

## 注意点

- 拡張機能の最終リリースは 3.0.1（2017年）
- `postcss-sorting` のREADMEも、エディタ向けプラグインはメンテナンスされていないと書いている
- 手元で一度並べる用途には使える
- ネストなど新しい構文まで含めて揃えるなら、npmの現行プラグイン側で行う
- プロパティ直上のコメントは、そのプロパティと一緒に移動する
- 並べる対象はソース — Autoprefixer適用後のファイルは対象にしない
- 順を揃えても、詳細度やカスケードは変わらない

> 参照: [postcss-sorting — Text editor](https://github.com/hudochenkov/postcss-sorting#text-editor) / [vscode-postcss-sorting 3.0.1](https://github.com/mrmlnc/vscode-postcss-sorting/releases/tag/3.0.1) / [postcss-sorting — Handling comments](https://github.com/hudochenkov/postcss-sorting#handling-comments)

## まとめ

- 宣言ブロック内のプロパティ順を揃える話で、見た目や詳細度は変わらない
- 手元で一度並べるなら、VS Codeの PostCSS Sorting — コマンドは `PostCSS Sorting: Run`
- 順は `properties-order` — アルファベット順か、自分で決めた配列
- ビルドに載せるなら `postcss-sorting`。配列を揃えれば、VS Codeで実行した結果と同じになる
- `.postcss-sorting.json` に置けば、拡張機能とPostCSSで同じ順を共有できる
- 拡張機能の更新は止まっている。新しい構文まで揃えるなら npm 側で行う
- プロパティ直上のコメントは一緒に移動する。並べるのはソース
- 順を揃えても、カスケードは直らない
