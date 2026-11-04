# Figmaプラグインの作り方

## Figmaプラグインとは

- Figmaプラグインは、FigmaやFigJamなどのエディタを拡張するための仕組み
- JavaScriptとHTMLを使って、Figmaファイル内のノードを読み取ったり、作成したり、変更したりできる
- 例えば、選択中のレイヤー名を一括変更する、デザイン内の色を一覧化する、指定した数の図形を自動生成する、といった処理を作れる
- フロントエンド開発に近い感覚で作れるが、通常のWebアプリとは実行環境が少し違う
- 最初は「Figma上で動く小さな自動化スクリプト」と考えると分かりやすい

> 参照: [Figma Developer Docs — Introduction](https://developers.figma.com/docs/plugins/)

## 作る前に知っておくこと

### 1. 開発にはFigmaデスクトップアプリが必要

- 開発中のプラグインは、ローカルにあるファイルをFigmaが読み込んで実行する
- そのため、ブラウザ版ではなくFigmaデスクトップアプリが必要になる
- Figmaデスクトップアプリでデザインファイルを開き、`Plugins > Development` から開発用プラグインを作成・実行する
- プラグインのコードは、VS Codeなどのエディタで編集する

### 2. TypeScriptが推奨されている

- Figma公式のQuickstartでは、TypeScriptを使う開発環境が案内されている
- Figma Plugin APIの型定義を使えるため、`figma.currentPage.selection` などのAPIを補完しながら書ける
- TypeScriptで書いた `code.ts` は、そのままではFigmaから実行されない
- ビルドしてJavaScriptの `code.js` に変換し、`manifest.json` の `main` から読み込む

### 3. プラグイン本体とUIは別の場所で動く

- Figmaプラグインには、主に2つの実行環境がある
- プラグイン本体はFigmaのメインスレッド上のサンドボックスで動く
- UIは `figma.showUI()` で表示されるiframe内で動く
- プラグイン本体はFigmaのノードにアクセスできるが、DOMやブラウザAPIには直接アクセスできない
- UIはHTML、CSS、ブラウザAPIを使えるが、Figmaのノードには直接アクセスできない
- 両者の間では、メッセージ通信で情報を受け渡す

> 参照: [Figma Developer Docs — Plugin Quickstart Guide](https://developers.figma.com/docs/plugins/plugin-quickstart-guide/) / [Figma Developer Docs — How Plugins Run](https://developers.figma.com/docs/plugins/how-plugins-run/)

## 基本ファイル

| ファイル | 役割 |
|---|---|
| `manifest.json` | プラグイン名、対象エディタ、実行ファイル、UIファイルを指定する設定ファイル |
| `code.ts` | Figma Plugin APIを使って、ノードの読み書きや作成処理を書くTypeScriptファイル |
| `code.js` | `code.ts` をJavaScriptに変換した実行用ファイル |
| `ui.html` | 入力フォームやボタンなど、ユーザーが操作する画面を作るHTMLファイル |
| `package.json` | 依存関係やビルド用スクリプトを管理するファイル |
| `tsconfig.json` | TypeScriptのコンパイル設定を管理するファイル |

- すべてを最初から理解する必要はない
- まずは `manifest.json`、`code.ts`、`ui.html` の関係を押さえる
- TypeScriptで書く場合、Figmaが直接実行するのはビルド後の `code.js`

## 最小手順

- Figmaデスクトップアプリでデザインファイルを開く
- `Plugins > Development > New plugin` を選ぶ
- 対象として `Figma design` を選び、必要に応じて `Custom UI` を選ぶ
- 保存されたプラグインフォルダをVS Codeなどで開く
- 依存関係をインストールする

```shell
npm install
```

- TypeScriptをビルドして、`code.ts` から `code.js` を作る

```shell
npm run build
```

- Figmaデスクトップアプリに戻り、`Plugins > Development` から作成したプラグイン名を選ぶ
- 変更が反映されない場合は、ビルド後の `code.js` が更新されているかを確認する

> 参照: [Figma Developer Docs — Plugin Quickstart Guide](https://developers.figma.com/docs/plugins/plugin-quickstart-guide/)

## UIなしの最小コード例

- まずはUIなしで、選択中のノード数を通知するだけのプラグインを考える
- Figma Plugin APIだけを使うため、`ui.html` は不要
- 処理が終わったら `figma.closePlugin()` を呼ぶ

```ts
const selectedNodes = figma.currentPage.selection;

figma.notify(`選択中のノード数: ${selectedNodes.length}`);

figma.closePlugin();
```

- `figma.currentPage.selection` で、現在のページで選択されているノードを取得する
- `figma.notify()` で、Figma上に通知を表示する
- `figma.closePlugin()` を呼ばないと、プラグインが実行中のまま残ることがある

## UIありの最小コード例

### `code.ts`

- UIを表示する場合は、プラグイン本体から `figma.showUI(__html__)` を呼ぶ
- UIから送られたメッセージは、`figma.ui.onmessage` で受け取る
- Figmaノードを操作する処理は、プラグイン本体側に書く

```ts
figma.showUI(__html__, { width: 240, height: 120 });

figma.ui.onmessage = (message) => {
  if (message.type === 'count-selection') {
    const count = figma.currentPage.selection.length;
    figma.notify(`選択中のノード数: ${count}`);
  }

  if (message.type === 'close') {
    figma.closePlugin();
  }
};
```

### `ui.html`

- UI側では、通常のHTMLとJavaScriptを書ける
- Figma本体へ指示を送るときは、`parent.postMessage()` を使う
- UI側から `figma.currentPage.selection` のようなFigma APIには直接アクセスできない

```html
<button id="count">選択数を確認</button>
<button id="close">閉じる</button>

<script>
  document.getElementById('count').onclick = () => {
    parent.postMessage(
      { pluginMessage: { type: 'count-selection' } },
      '*'
    );
  };

  document.getElementById('close').onclick = () => {
    parent.postMessage(
      { pluginMessage: { type: 'close' } },
      '*'
    );
  };
</script>
```

- UIはユーザー入力を受け取る場所
- Figmaのデータを触る処理は `code.ts` 側に寄せる
- この分担を理解すると、複雑なプラグインも整理しやすくなる

> 参照: [Figma Developer Docs — How Plugins Run](https://developers.figma.com/docs/plugins/how-plugins-run/)

## `manifest.json` の見方

### 最小構成の例

```json
{
  "name": "My Plugin",
  "api": "1.0.0",
  "main": "code.js",
  "ui": "ui.html",
  "editorType": ["figma"],
  "documentAccess": "dynamic-page",
  "networkAccess": {
    "allowedDomains": ["none"]
  }
}
```

### 主な項目

| 項目 | 役割 |
|---|---|
| `name` | Figma上に表示されるプラグイン名 |
| `api` | 使用するFigma Plugin APIのバージョン |
| `main` | Figmaが実行するJavaScriptファイル |
| `ui` | `figma.showUI()` で表示するHTMLファイル |
| `editorType` | プラグインを実行できるエディタの種類 |
| `documentAccess` | 新しいプラグインでは `dynamic-page` が必要 |
| `networkAccess` | 外部通信を許可するドメイン |

- `main` は `code.ts` ではなく、ビルド後の `code.js` を指す
- API通信やCDN読み込みを使う場合は、`networkAccess` を確認する
- 外部通信が不要なら、`allowedDomains` を `["none"]` にしておく

> 参照: [Figma Developer Docs — Plugin Manifest](https://developers.figma.com/docs/plugins/manifest/)

## よくある間違い

### 1. `manifest.json` の `main` に `code.ts` を指定する

- `main` はJavaScriptファイルを指す必要がある
- TypeScriptで書く場合は、ビルド後の `code.js` を指定する
- `code.ts` を指定すると、Figmaが実行できない

### 2. `code.ts` を編集したのにビルドしていない

- Figmaは `code.ts` ではなく `code.js` を読み込む
- TypeScriptを編集しただけでは、実行されるJavaScriptが変わらない
- 変更が反映されないときは、まずビルドやwatchが動いているかを確認する

### 3. UI側からFigmaノードに直接触ろうとする

- UIはiframe内で動くため、Figmaのノードには直接アクセスできない
- UIからはメッセージを送り、`code.ts` 側でFigma APIを使う
- `figma.currentPage.selection` などはプラグイン本体側で扱う

### 4. `figma.closePlugin()` を呼ばない

- UIなしで処理だけ実行するプラグインは、最後に `figma.closePlugin()` を呼ぶ
- 呼ばないと、プラグインが実行中のまま残ることがある
- UIありの場合も、ユーザーが閉じる操作をしたら `figma.closePlugin()` につなげる

### 5. 外部通信の許可を忘れる

- UIで外部APIを呼ぶ場合や、外部のスクリプトを読み込む場合は `networkAccess` を確認する
- 許可されていないドメインはブロックされる
- ただし、最初の学習用プラグインでは外部通信を使わない方が分かりやすい
- まずはローカルの `code.ts` と `ui.html` だけで完結するものから作る

## まとめ

- Figmaプラグインは、JavaScriptとHTMLでFigmaの作業を自動化・拡張する仕組み
- 開発中の実行にはFigmaデスクトップアプリが必要
- TypeScriptで書く場合、Figmaが実行するのはビルド後の `code.js`
- `manifest.json` は、プラグイン名、実行ファイル、UI、対象エディタなどを定義する
- プラグイン本体はFigmaノードにアクセスできるが、DOMやブラウザAPIには直接アクセスできない
- UIはブラウザAPIを使えるが、Figmaノードには直接アクセスできない
- UIとプラグイン本体は、`postMessage` と `figma.ui.onmessage` で通信する
