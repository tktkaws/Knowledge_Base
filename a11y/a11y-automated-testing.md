# axe-core / Lighthouseを使ったアクセシビリティ自動テスト

## アクセシビリティ自動テストの概要

- Webページがアクセシビリティの基準（WCAGなど）を満たしているかを機械的に検証する手法
- HTMLの構造やARIA属性の誤用、コントラスト比不足などをプログラムで検出
- 手動テストに比べて高速・再現性が高く、CI/CDに組み込むことで継続的な品質担保が可能

### なぜ自動テストが必要か

- 開発者が全ページ・全状態を手動で確認するのは現実的でない
- リグレッション（修正済みの問題の再発）を自動で検知できる
- プルリクエストやデプロイのたびにチェックを実行し、問題の早期発見が可能
- チーム全体でアクセシビリティの最低ラインを機械的に維持できる

> 参照: [W3C — Accessibility Testing](https://www.w3.org/wiki/Accessibility_testing)

## axe-core とは

### 概要

- Deque Systems社が開発したオープンソースのアクセシビリティテストエンジン
- WCAG 2.0 / 2.1 / 2.2 のA・AAレベルの多くのルールを網羅
- JavaScript製で、ブラウザ上で実行されるDOM解析ベースのツール
- 業界標準のアクセシビリティテストライブラリとして広く採用

### 仕組み

- ブラウザに描画されたDOMツリーを解析し、定義されたルールセットに照合
- 各ルールはCSS セレクタで対象要素を絞り込み、チェック関数で合否を判定
- 結果は violations（違反）、passes（合格）、incomplete（要手動確認）、inapplicable（該当なし）に分類

### axe-core が検出できるもの

- `alt` 属性の欠落した画像
- `label` が紐付けられていないフォーム要素
- 不十分なカラーコントラスト比
- 不正なARIAロール・属性の使用
- ランドマークの重複や欠落
- 見出しレベルの飛び（h1 → h3 など）
- `tabindex` の不適切な値
- `lang` 属性の欠落

### axe-core が検出できないもの

- フォーカス順序が論理的かどうか
- スクリーンリーダーでの読み上げ体験が自然かどうか
- キーボード操作が直感的かどうか
- 代替テキスト（alt）の内容が適切かどうか（存在の有無は検出可能）
- 動的なインタラクション後のアクセシビリティ状態
- 認知的な負荷やわかりやすさ

> 参照: [axe-core GitHub リポジトリ](https://github.com/dequelabs/axe-core)
> 参照: [Deque — axe Rules](https://dequeuniversity.com/rules/axe/)

## axe-core の導入方法

### npm パッケージとして導入

```bash
npm install --save-dev axe-core
```

```js
// ブラウザ環境で実行
import axe from 'axe-core';

axe.run(document, (err, results) => {
  if (err) throw err;
  console.log('違反:', results.violations);
  console.log('合格:', results.passes);
});
```

### ブラウザ拡張機能

- **axe DevTools**（Chrome / Firefox / Edge 対応）
  - ブラウザのDevToolsパネルに「axe」タブが追加される
  - ボタン一つでページ全体をスキャンし、違反箇所をハイライト表示
  - 各違反に対してWCAGの該当基準、修正方法、影響度を表示
  - 開発中のサッとした確認に最適

### React 開発中のリアルタイムチェック（@axe-core/react）

```bash
npm install --save-dev @axe-core/react
```

```jsx
// main.jsx または index.js（開発環境のみ）
import React from 'react';
import ReactDOM from 'react-dom/client';

if (process.env.NODE_ENV === 'development') {
  import('@axe-core/react').then((axe) => {
    axe.default(React, ReactDOM, 1000);
    // 1000ms のデバウンス間隔でDOMの変更を監視し、
    // 違反があればブラウザコンソールに警告を出力
  });
}
```

> 参照: [@axe-core/react GitHub](https://github.com/dequelabs/axe-core-npm/tree/develop/packages/react)

## テストフレームワークとの統合

### Jest + jsdom での統合（jest-axe）

```bash
npm install --save-dev jest-axe
```

```js
// コンポーネントのアクセシビリティテスト
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

// 良い例：アクセシブルなフォームのテスト
test('フォームにアクセシビリティ違反がないこと', async () => {
  const { container } = render(
    <form>
      <label htmlFor="name">氏名</label>
      <input id="name" type="text" />
      <button type="submit">送信</button>
    </form>
  );

  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

// 悪い例：このテストはラベルなしの違反を検出する
test('ラベルなしフォームは違反を検出する', async () => {
  const { container } = render(
    <form>
      {/* label が紐付けられていない入力欄 */}
      <input type="text" />
      <button type="submit">送信</button>
    </form>
  );

  const results = await axe(container);
  // results.violations に "label" 関連の違反が含まれる
  expect(results.violations.length).toBeGreaterThan(0);
});
```

### 特定のルールに絞ったテスト

```js
test('カラーコントラストのみをチェック', async () => {
  const { container } = render(<MyComponent />);

  const results = await axe(container, {
    runOnly: {
      type: 'rule',
      values: ['color-contrast'],
    },
  });

  expect(results).toHaveNoViolations();
});
```

> 参照: [jest-axe GitHub](https://github.com/nickcolley/jest-axe)

### Playwright での統合（@axe-core/playwright）

```bash
npm install --save-dev @axe-core/playwright
```

```ts
// e2e/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('アクセシビリティテスト', () => {
  // 良い例：ページ全体のスキャン
  test('トップページにアクセシビリティ違反がないこと', async ({ page }) => {
    await page.goto('http://localhost:3000');

    const accessibilityScanResults = await new AxeBuilder({ page }).analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  // 良い例：特定の領域に絞ったスキャン
  test('ナビゲーションにアクセシビリティ違反がないこと', async ({ page }) => {
    await page.goto('http://localhost:3000');

    const accessibilityScanResults = await new AxeBuilder({ page })
      .include('nav')          // nav 要素のみを対象にスキャン
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  // 良い例：WCAG 2.2 AA レベルに限定したスキャン
  test('WCAG 2.2 AA 基準を満たすこと', async ({ page }) => {
    await page.goto('http://localhost:3000');

    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });
});
```

### 悪い例：テストとして不十分なパターン

```ts
// 悪い例：違反を無視してテストを通す
test('アクセシビリティチェック（不十分）', async ({ page }) => {
  await page.goto('http://localhost:3000');

  const results = await new AxeBuilder({ page })
    .disableRules(['color-contrast', 'label', 'image-alt']) // 重要なルールを無効化
    .analyze();

  expect(results.violations).toEqual([]);
  // ルールを無効化しているので通るが、実質的にテストの意味がない
});
```

```ts
// 良い例：既知の問題を明示的に除外しつつ、理由をコメントで残す
test('アクセシビリティチェック（既知の問題を除外）', async ({ page }) => {
  await page.goto('http://localhost:3000');

  const results = await new AxeBuilder({ page })
    // TODO: サードパーティ製ウィジェットの色コントラスト問題（チケット #1234 で対応予定）
    .exclude('.third-party-widget')
    .analyze();

  expect(results.violations).toEqual([]);
});
```

> 参照: [@axe-core/playwright GitHub](https://github.com/dequelabs/axe-core-npm/tree/develop/packages/playwright)

## Lighthouse のアクセシビリティ監査

### 概要

- Googleが開発したWebページ品質監査ツール
- パフォーマンス、アクセシビリティ、SEO、ベストプラクティスなど複数カテゴリを一括でスコアリング
- アクセシビリティカテゴリは内部的にaxe-coreのルールを使用（一部独自ルール含む）
- 0〜100点のスコアで評価し、改善すべき項目を優先度付きで提示

### Chrome DevTools での実行

1. Chrome DevTools を開く（F12 または Cmd+Option+I）
2. 「Lighthouse」タブを選択
3. 「Accessibility」カテゴリにチェックを入れる
4. 「Analyze page load」をクリック
5. スコアと個別の監査結果が表示される

### Lighthouse CLI での実行

```bash
npm install -g lighthouse
```

```bash
# 基本的な実行
lighthouse https://example.com --only-categories=accessibility --output=html --output-path=./report.html

# JSON 形式で出力（CI/CD での解析に便利）
lighthouse https://example.com --only-categories=accessibility --output=json --output-path=./report.json

# ヘッドレスモードで実行
lighthouse https://example.com --only-categories=accessibility --chrome-flags="--headless=new"
```

### Lighthouse スコアの読み方

| スコア | 評価 |
|---|---|
| 90〜100 | 良好（緑） |
| 50〜89 | 改善が必要（オレンジ） |
| 0〜49 | 不十分（赤） |

- スコアは各監査項目の重み付きで算出
- 100点 = Lighthouseが検出可能な全項目に合格
- 100点でもアクセシビリティが完璧というわけではない（自動テストの限界）

> 参照: [Lighthouse ドキュメント](https://developer.chrome.com/docs/lighthouse/)

## Lighthouse を CI/CD で使う

### GitHub Actions での実行

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse Accessibility Check

on:
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Start server
        run: npm run preview &

      - name: Wait for server
        run: npx wait-on http://localhost:3000

      - name: Run Lighthouse
        uses: treosh/lighthouse-ci-action@v12
        with:
          urls: |
            http://localhost:3000
            http://localhost:3000/about
          configPath: ./.lighthouserc.json
```

### Lighthouse CI の設定ファイル

```json
// .lighthouserc.json
{
  "ci": {
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 0.9 }]
      }
    },
    "collect": {
      "numberOfRuns": 1,
      "settings": {
        "onlyCategories": ["accessibility"]
      }
    }
  }
}
```

- `minScore: 0.9` — アクセシビリティスコアが90点未満の場合にCIを失敗させる
- チームの方針に応じてしきい値を調整

### Node.js スクリプトでの Lighthouse 実行

```js
// scripts/lighthouse-a11y.mjs
import lighthouse from 'lighthouse';
import * as chromeLauncher from 'chrome-launcher';

async function runAccessibilityAudit(url) {
  const chrome = await chromeLauncher.launch({ chromeFlags: ['--headless=new'] });

  const result = await lighthouse(url, {
    port: chrome.port,
    onlyCategories: ['accessibility'],
    output: 'json',
  });

  const { categories, audits } = result.lhr;
  const score = categories.accessibility.score * 100;

  console.log(`アクセシビリティスコア: ${score}`);

  // 失敗した監査項目を表示
  const failedAudits = Object.values(audits).filter(
    (audit) => audit.score !== null && audit.score < 1
  );

  failedAudits.forEach((audit) => {
    console.log(`  ❌ ${audit.title}: ${audit.description}`);
  });

  await chrome.kill();

  // スコアが基準未満なら終了コード1で終了
  if (score < 90) {
    console.error(`スコアが基準値（90）を下回りました: ${score}`);
    process.exit(1);
  }
}

runAccessibilityAudit('http://localhost:3000');
```

> 参照: [Lighthouse CI GitHub](https://github.com/GoogleChrome/lighthouse-ci)

## axe-core と Lighthouse の使い分け・比較

| 観点 | axe-core | Lighthouse |
|---|---|---|
| 開発元 | Deque Systems | Google |
| 主な用途 | ユニットテスト・E2Eテストへの組み込み | ページ全体の品質監査 |
| 実行タイミング | テストコード内で任意のタイミング | ページロード完了後のスナップショット |
| 出力形式 | violations / passes の詳細なJSON | スコア（0〜100）+ 監査項目一覧 |
| テストフレームワーク連携 | Jest, Playwright, Cypress 等と統合可 | CLI / CI向けの独立ツール |
| 特定要素のスキャン | `include` / `exclude` で要素単位の絞り込み可 | ページ全体が対象 |
| 動的コンテンツ | ページ操作後の状態もテスト可 | ページロード時点のみ |
| アクセシビリティ以外 | アクセシビリティ専用 | パフォーマンス・SEO等も同時に監査 |

### 使い分けの指針

- **axe-core を使うべき場面**
  - コンポーネント単位でアクセシビリティを検証したいとき
  - ボタンクリック後、モーダル表示後など動的な状態をテストしたいとき
  - Jest や Playwright の既存テストスイートにアクセシビリティチェックを追加したいとき
  - 特定の要素やコンポーネントに絞ったスキャンが必要なとき

- **Lighthouse を使うべき場面**
  - ページ全体のアクセシビリティスコアを定点観測したいとき
  - パフォーマンスやSEOと一緒にアクセシビリティも監査したいとき
  - CI/CDでデプロイ前のゲートチェックとして使いたいとき
  - 非エンジニア（デザイナー、PMなど）にレポートを共有したいとき

- **両方を組み合わせるのが理想的**
  - axe-core：開発中のコンポーネントテスト・E2Eテストに組み込み、早期に問題を検出
  - Lighthouse：CI/CDのデプロイゲートとして、ページ全体のスコアを監視

## 自動テストの限界

### カバーできる範囲は全体の30〜50%程度

- アクセシビリティの問題のうち、自動テストで検出できるのは全体の約30〜50%とされている
- 残りの50〜70%は人間の判断が必要な問題

### 自動テストで検出できない問題の例

| カテゴリ | 具体例 |
|---|---|
| コンテンツの適切さ | alt属性の内容が画像を正しく説明しているか |
| 操作の論理性 | タブ順序が直感的か、フォーカス移動が自然か |
| 読み上げ体験 | スクリーンリーダーでの読み上げ順序が意味をなすか |
| 認知的負荷 | 情報量が多すぎないか、操作がわかりやすいか |
| 文脈依存の問題 | リンクテキスト「こちら」が文脈上理解可能か |
| インタラクション | キーボードのみで全操作を完了できるか |
| 動的コンテンツ | AJAX更新後の通知が適切か、ライブリージョンが機能しているか |

### 自動テスト + 手動テストの組み合わせが必須

```
┌─────────────────────────────────────────┐
│          アクセシビリティテスト          │
├───────────────────┬─────────────────────┤
│  自動テスト(30-50%)│  手動テスト(50-70%) │
│                   │                     │
│ ・axe-core        │ ・スクリーンリーダー │
│ ・Lighthouse      │ ・キーボード操作     │
│ ・HTMLバリデーション│ ・拡大表示確認      │
│                   │ ・認知的評価         │
│                   │ ・当事者テスト       │
└───────────────────┴─────────────────────┘
```

- 自動テストは「最低限の品質を機械的に保証する」ためのもの
- 自動テストをパスしても「アクセシブルである」とは言えない
- 手動テストやユーザーテストと組み合わせて初めて十分な検証になる

> 参照: [Deque — What Can Automated Testing Really Cover?](https://www.deque.com/blog/automated-testing-study-identifies-57-percent-of-digital-accessibility-issues/)
> 参照: [GOV.UK — Testing for accessibility](https://www.gov.uk/service-manual/helping-people-to-use-your-service/testing-for-accessibility)

## 良い例・悪い例のコード対比

### テスト対象のコンポーネント

```html
<!-- 悪い例：アクセシビリティ上の問題が多いカード -->
<div class="card" onclick="openDetail()">
  <img src="product.jpg" />
  <div class="title">ワイヤレスヘッドホン</div>
  <div class="price" style="color: #aaa;">¥12,800</div>
  <div class="btn">詳細を見る</div>
</div>
```

問題点:
- `img` に `alt` がない
- `div.btn` はボタンとして認識されない
- 価格のコントラスト比が不十分
- クリック可能な `div` にキーボード操作やroleがない

```html
<!-- 良い例：アクセシブルなカード -->
<article class="card">
  <img src="product.jpg" alt="ワイヤレスヘッドホン XM5 ブラック" />
  <h3 class="title">ワイヤレスヘッドホン</h3>
  <p class="price">¥12,800</p>
  <a href="/products/xm5" class="btn">ワイヤレスヘッドホンの詳細を見る</a>
</article>
```

### テストコードの書き方

```ts
// 悪い例：アクセシビリティテストを書かない、または形だけのテスト
test('カードが表示される', () => {
  const { container } = render(<ProductCard />);
  expect(container.querySelector('.card')).toBeTruthy();
  // アクセシビリティの検証が一切ない
});
```

```ts
// 良い例：アクセシビリティテストを含む
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('商品カードにアクセシビリティ違反がないこと', async () => {
  const { container } = render(<ProductCard />);

  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

test('商品画像に適切なalt属性があること', () => {
  const { getByRole } = render(<ProductCard />);
  const img = getByRole('img');
  expect(img).toHaveAttribute('alt');
  expect(img.getAttribute('alt')).not.toBe('');
});
```

## まとめ

- アクセシビリティ自動テストは、機械的に検出可能な問題を継続的にチェックする仕組み
- **axe-core** はテストフレームワークに組み込んでコンポーネント単位・動的状態のテストに最適
- **Lighthouse** はページ全体のスコアリングとCI/CDでのゲートチェックに最適
- 両者を組み合わせて使うのが効果的
- 自動テストでカバーできるのは全体の30〜50%程度 — 手動テスト・ユーザーテストとの併用が必須
- 自動テストの目的は「完璧なアクセシビリティの証明」ではなく「最低限の品質を機械的に担保する」こと
