# Puppeteerの基本 — ヘッドレスブラウザ操作の入門

## Puppeteerとは

- Googleが開発・メンテナンスするNode.jsライブラリ
- Chrome / Chromiumをプログラムから操作するためのAPI
- ヘッドレス（画面なし）またはヘッドフル（画面あり）でブラウザを制御
- スクレイピング、E2Eテスト、スクリーンショット撮影、PDF生成などに使用

```bash
npm install puppeteer
```

- `puppeteer`をインストールすると、対応するChromiumが自動でダウンロードされる
- ブラウザ不要で軽量な`puppeteer-core`も提供（別途ブラウザのパスを指定する必要あり）

- 参照: [Puppeteer 公式ドキュメント](https://pptr.dev/)

## 基本的な使い方

### 最小構成

```js
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch();
const page = await browser.newPage();

await page.goto('https://example.com');

const title = await page.title();
console.log(title); // "Example Domain"

await browser.close();
```

### 処理の流れ

```
puppeteer.launch()  → ブラウザを起動
browser.newPage()   → 新しいタブ（ページ）を開く
page.goto(url)      → 指定URLに遷移
page.xxx()          → ページ操作（クリック、入力、取得など）
browser.close()     → ブラウザを終了
```

- `browser.close()`を呼ばないとプロセスが残り続けるため、必ず終了処理を行う

- 参照: [Puppeteer — Getting Started](https://pptr.dev/guides/getting-started)

## launch()のオプション

```js
const browser = await puppeteer.launch({
  headless: true,        // false にするとブラウザが画面に表示される（デバッグ用）
  slowMo: 100,           // 各操作に100msの遅延を追加（デバッグ用）
  defaultViewport: {
    width: 1280,
    height: 720,
  },
  args: [
    '--no-sandbox',            // CI環境で必要になる場合がある
    '--disable-setuid-sandbox',
  ],
});
```

| オプション | 型 | 説明 |
|---|---|---|
| `headless` | `boolean \| "shell"` | `true`でヘッドレス起動。デフォルト`true` |
| `slowMo` | `number` | 操作間の遅延(ms)。デバッグ時に便利 |
| `defaultViewport` | `object \| null` | ビューポートサイズ。`null`でブラウザデフォルト |
| `args` | `string[]` | Chromiumに渡す起動引数 |
| `executablePath` | `string` | 使用するブラウザの実行ファイルパス |
| `timeout` | `number` | ブラウザ起動のタイムアウト(ms) |

- 参照: [Puppeteer — Launch Options](https://pptr.dev/api/puppeteer.launchoptions)

## ページ操作

### ナビゲーション

```js
// ページに遷移
await page.goto('https://example.com');

// 遷移完了の待機条件を指定
await page.goto('https://example.com', {
  waitUntil: 'networkidle2', // ネットワーク接続が2つ以下になるまで待機
});
```

| waitUntil | 説明 |
|---|---|
| `load` | loadイベント発火時（デフォルト） |
| `domcontentloaded` | DOMContentLoadedイベント発火時 |
| `networkidle0` | ネットワーク接続が0になってから500ms経過 |
| `networkidle2` | ネットワーク接続が2つ以下になってから500ms経過 |

### 要素の待機

```js
// セレクタに一致する要素が現れるまで待機
await page.waitForSelector('.result-list');

// 要素が非表示になるまで待機
await page.waitForSelector('.loading', { hidden: true });

// タイムアウト付き
await page.waitForSelector('.content', { timeout: 5000 });
```

### クリック・入力

```js
// 要素をクリック
await page.click('#submit-button');

// テキスト入力
await page.type('#search-input', 'Puppeteer 使い方');

// 入力欄をクリアしてから入力
await page.click('#search-input', { clickCount: 3 }); // 全選択
await page.type('#search-input', '新しいテキスト');

// セレクトボックスの選択
await page.select('#country', 'jp');

// キーボードのキーを押す
await page.keyboard.press('Enter');
```

### テキスト・属性の取得

```js
// 要素のテキストを取得
const text = await page.$eval('.title', el => el.textContent);

// 要素の属性を取得
const href = await page.$eval('a.link', el => el.getAttribute('href'));

// 複数要素のテキストを配列で取得
const items = await page.$$eval('.list-item', elements =>
  elements.map(el => el.textContent)
);
```

| メソッド | 説明 |
|---|---|
| `page.$(selector)` | 最初に一致する要素を取得（`querySelector`相当） |
| `page.$$(selector)` | 一致する全要素を取得（`querySelectorAll`相当） |
| `page.$eval(selector, fn)` | 最初の一致要素に対してブラウザ上で関数を実行 |
| `page.$$eval(selector, fn)` | 全一致要素に対してブラウザ上で関数を実行 |
| `page.evaluate(fn)` | ブラウザ上で任意のJavaScriptを実行 |

- 参照: [Puppeteer — Page API](https://pptr.dev/api/puppeteer.page)

## evaluate — ブラウザ上でJavaScriptを実行

```js
// ブラウザのコンテキストでJSを実行し、結果をNode.js側で受け取る
const dimensions = await page.evaluate(() => {
  return {
    width: document.documentElement.clientWidth,
    height: document.documentElement.clientHeight,
    title: document.title,
  };
});
console.log(dimensions);

// Node.js側の変数をブラウザに渡す
const searchQuery = 'Puppeteer';
const results = await page.evaluate((query) => {
  // ここはブラウザ上で実行される
  // Node.jsの変数やモジュールは使えない
  return document.querySelectorAll(`[data-query="${query}"]`).length;
}, searchQuery);
```

### 注意点

- `evaluate`内のコードはブラウザ上で実行される
- Node.jsのモジュール（`fs`、`path`など）は使えない
- 引数はシリアライズされて渡されるため、関数やクラスインスタンスは渡せない

## スクリーンショット

```js
// ページ全体のスクリーンショット
await page.screenshot({ path: 'screenshot.png' });

// ビューポートに表示されている部分のみ
await page.screenshot({ path: 'viewport.png', fullPage: false });

// ページ全体（スクロール分含む）
await page.screenshot({ path: 'fullpage.png', fullPage: true });

// 特定の要素のみ
const element = await page.$('.hero-section');
await element.screenshot({ path: 'hero.png' });

// JPEG形式・品質指定
await page.screenshot({
  path: 'photo.jpg',
  type: 'jpeg',
  quality: 80,
});
```

- 参照: [Puppeteer — Screenshots](https://pptr.dev/guides/screenshots)

## PDF生成

```js
// ページをPDFとして保存
await page.pdf({ path: 'page.pdf' });

// オプション指定
await page.pdf({
  path: 'report.pdf',
  format: 'A4',
  printBackground: true,       // 背景色・背景画像を含める
  margin: {
    top: '20mm',
    right: '15mm',
    bottom: '20mm',
    left: '15mm',
  },
});
```

- PDF生成はヘッドレスモードでのみ動作
- `printBackground: true`を指定しないとCSSの背景が出力されない

- 参照: [Puppeteer — PDF Generation](https://pptr.dev/guides/pdf-generation)

## 実用パターン

### パターン1: Webスクレイピング

```js
import puppeteer from 'puppeteer';

async function scrapeNews() {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  await page.goto('https://example.com/news', {
    waitUntil: 'networkidle2',
  });

  // 記事一覧を取得
  const articles = await page.$$eval('.article-card', cards =>
    cards.map(card => ({
      title: card.querySelector('h2')?.textContent?.trim(),
      url: card.querySelector('a')?.href,
      date: card.querySelector('.date')?.textContent?.trim(),
    }))
  );

  console.log(articles);
  await browser.close();
}

scrapeNews();
```

### パターン2: フォーム自動入力と送信

```js
async function submitForm() {
  const browser = await puppeteer.launch({ headless: false });
  const page = await browser.newPage();

  await page.goto('https://example.com/contact');

  // フォーム入力
  await page.type('#name', '山田太郎');
  await page.type('#email', 'yamada@example.com');
  await page.type('#message', 'お問い合わせ内容');

  // カテゴリ選択
  await page.select('#category', 'support');

  // 送信ボタンをクリックして遷移を待機
  await Promise.all([
    page.waitForNavigation(),
    page.click('#submit'),
  ]);

  const result = await page.$eval('.result-message', el => el.textContent);
  console.log(result);

  await browser.close();
}
```

### パターン3: OGP画像の自動生成

```js
async function generateOgImage(title, outputPath) {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  await page.setViewport({ width: 1200, height: 630 });

  // HTMLを直接読み込む
  await page.setContent(`
    <html>
    <body style="
      display: flex;
      align-items: center;
      justify-content: center;
      width: 1200px;
      height: 630px;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      font-family: sans-serif;
    ">
      <h1 style="font-size: 48px; text-align: center; padding: 40px;">
        ${title}
      </h1>
    </body>
    </html>
  `);

  await page.screenshot({ path: outputPath });
  await browser.close();
}

generateOgImage('JSON-LDの基礎', 'og-image.png');
```

### パターン4: SPAの動的コンテンツ取得

```js
async function scrapeSPA() {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  await page.goto('https://example.com/app');

  // SPAのコンテンツが描画されるまで待機
  await page.waitForSelector('.dynamic-content', { timeout: 10000 });

  // 無限スクロールのシミュレーション
  let previousHeight = 0;
  while (true) {
    const currentHeight = await page.evaluate(() => document.body.scrollHeight);
    if (currentHeight === previousHeight) break;

    previousHeight = currentHeight;
    await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  const allItems = await page.$$eval('.item', items =>
    items.map(item => item.textContent.trim())
  );

  console.log(`取得件数: ${allItems.length}`);
  await browser.close();
}
```

## エラーハンドリング

```js
import puppeteer from 'puppeteer';

async function safeScrape(url) {
  let browser;
  try {
    browser = await puppeteer.launch();
    const page = await browser.newPage();

    // ナビゲーションのタイムアウト設定
    page.setDefaultNavigationTimeout(30000);

    // リクエスト失敗のハンドリング
    page.on('requestfailed', request => {
      console.error(`リクエスト失敗: ${request.url()}`);
    });

    const response = await page.goto(url);

    // HTTPステータスコードの確認
    if (!response.ok()) {
      throw new Error(`HTTP ${response.status()}: ${response.statusText()}`);
    }

    // 要素の待機（タイムアウト付き）
    try {
      await page.waitForSelector('.content', { timeout: 5000 });
    } catch {
      console.error('コンテンツが見つかりませんでした');
      return null;
    }

    return await page.$eval('.content', el => el.textContent);

  } catch (error) {
    console.error(`エラー: ${error.message}`);
    return null;
  } finally {
    // ブラウザは必ず閉じる
    if (browser) await browser.close();
  }
}
```

- 参照: [Puppeteer — Troubleshooting](https://pptr.dev/troubleshooting)

## パフォーマンス改善のヒント

### 不要なリソースの読み込みをブロック

```js
// 画像やフォントの読み込みを無効化して高速化
await page.setRequestInterception(true);
page.on('request', request => {
  const resourceType = request.resourceType();
  if (['image', 'font', 'stylesheet'].includes(resourceType)) {
    request.abort();
  } else {
    request.continue();
  }
});
```

### ブラウザインスタンスの再利用

```js
// 悪い例：毎回ブラウザを起動・終了
for (const url of urls) {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto(url);
  await browser.close();
}

// 良い例：ブラウザを使い回し、ページを都度作成・破棄
const browser = await puppeteer.launch();
for (const url of urls) {
  const page = await browser.newPage();
  await page.goto(url);
  await page.close(); // ページだけ閉じる
}
await browser.close();
```

## Puppeteer vs Playwright

| 比較項目 | Puppeteer | Playwright |
|---|---|---|
| 開発元 | Google | Microsoft |
| 対応ブラウザ | Chrome / Chromium / Firefox | Chromium / Firefox / WebKit |
| 言語 | Node.js | Node.js / Python / Java / .NET |
| 自動待機 | 手動（`waitForSelector`など） | 組み込み（auto-waiting） |
| 用途 | スクレイピング、PDF生成、自動化全般 | E2Eテストに強い |

- PuppeteerはChrome操作に特化したシンプルなAPI
- E2Eテスト目的ならPlaywrightの方が機能が豊富
- スクレイピングやPDF生成など単発の自動化にはPuppeteerが手軽

- 参照: [Playwright 公式](https://playwright.dev/)

## よくある間違い

### 1. browser.close()の呼び忘れ

```js
// 悪い例：エラー時にブラウザが閉じられない
const browser = await puppeteer.launch();
const page = await browser.newPage();
await page.goto(url);
// ここでエラーが発生すると browser.close() が呼ばれない

// 良い例：try-finallyで確実に閉じる
let browser;
try {
  browser = await puppeteer.launch();
  // ...
} finally {
  if (browser) await browser.close();
}
```

### 2. 要素の待機をせずに操作する

```js
// 悪い例：要素がまだ存在しない可能性がある
await page.goto('https://example.com/spa');
await page.click('.dynamic-button'); // エラーになる可能性

// 良い例：要素が表示されるまで待機
await page.goto('https://example.com/spa');
await page.waitForSelector('.dynamic-button');
await page.click('.dynamic-button');
```

### 3. evaluateの中でNode.jsの変数を直接参照する

```js
const fs = require('fs');

// 悪い例：evaluateの中はブラウザ環境なのでNode.jsモジュールは使えない
await page.evaluate(() => {
  fs.writeFileSync('output.txt', document.title); // ReferenceError
});

// 良い例：evaluateの戻り値をNode.js側で受け取って処理する
const title = await page.evaluate(() => document.title);
fs.writeFileSync('output.txt', title);
```

## まとめ

- PuppeteerはChrome/Chromiumをプログラムから操作するNode.jsライブラリ
- `launch()` → `newPage()` → `goto()` → 操作 → `close()` の流れが基本
- `page.$eval` / `page.$$eval` / `page.evaluate`でブラウザ上のDOMやJSにアクセス
- スクリーンショット・PDF生成・スクレイピング・フォーム自動操作など用途が幅広い
- `waitForSelector`で要素の出現を待機してから操作する
- `browser.close()`は`finally`で確実に呼び出す
- E2Eテスト主目的ならPlaywrightも検討
