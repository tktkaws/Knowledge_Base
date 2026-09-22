# CI/CDにアクセシビリティテストを組み込む方法

## なぜCI/CDに入れるか

- PRやデプロイのたびに自動で検証し、アクセシビリティのリグレッションを防ぐ
- 「後で直す」を減らし、マージ前に最低ラインを機械的に担保できる
- 自動テストで見つかる問題は全体の一部だが、繰り返し起きる違反の再発防止に効く
- 手動のスクリーンリーダーテストやキーボード確認と併用する前提

> 参照: [Playwright — Accessibility testing](https://playwright.dev/docs/accessibility-testing)

## レイヤー別の組み込み方

| レイヤー | ツール例 | 検証対象 |
|---|---|---|
| コンポーネント | jest-axe / Vitest + axe | 単体UIのDOM |
| E2E / ページ | `@axe-core/playwright` | 実ブラウザ上の画面状態 |
| URL監査 | Pa11y / Lighthouse CI | ビルド成果物の主要URL |
| コンポーネントカタログ | Storybook addon-a11y | Storyごとの見た目とDOM |

- いきなり全ページ網羅より、**主要テンプレート + 重要なコンポーネント**から始める
- CIでは「違反0で失敗」にするルールを先に決め、例外は明示的に管理する

## 導入の進め方

1. ローカルで axe を1ページに当て、現状の違反数を把握する
2. 重大度の高い違反（画像の `alt` 欠落、フォームラベルなしなど）から修正する
3. コンポーネントテストまたは Playwright に1本入れる
4. GitHub Actions などで PR ゲートにする
5. 対象ページ・コンポーネントを段階的に増やす

- 最初から全違反をブロックすると導入が止まりやすい
- 「新規違反を増やさない」方針でも効果がある

## コンポーネント層: jest-axe

```bash
npm install --save-dev jest-axe
```

```js
// button.a11y.test.jsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { Button } from './Button';

expect.extend(toHaveNoViolations);

test('Button にアクセシビリティ違反がない', async () => {
  const { container } = render(<Button>保存</Button>);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

- レンダリング結果のDOMに対して axe を実行する
- コントラストなど、一部ルールは jsdom 環境では限定的
- 見た目や実ブラウザ依存の検証は Playwright 側に回す

## E2E層: Playwright + axe-core

```bash
npm install --save-dev @playwright/test @axe-core/playwright
```

```js
// tests/a11y/home.spec.js
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('トップページに重大な a11y 違反がない', async ({ page }) => {
  await page.goto('/');

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

### 操作後の状態もスキャンする

```js
test('モーダル表示中にも違反がない', async ({ page }) => {
  await page.goto('/settings');
  await page.getByRole('button', { name: '削除する' }).click();

  const results = await new AxeBuilder({ page })
    .include('[role="dialog"]')
    .analyze();

  expect(results.violations).toEqual([]);
});
```

- `analyze()` は**呼んだ時点のDOM状態**を見る
- 開閉後・入力後・エラー表示後など、状態ごとにスキャンする

### 共通設定を fixture にする

```js
// a11y-fixture.js
import { test as base } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

export const test = base.extend({
  makeAxeBuilder: async ({ page }, use) => {
    const makeAxeBuilder = () =>
      new AxeBuilder({ page }).withTags([
        'wcag2a',
        'wcag2aa',
        'wcag21aa',
        'wcag22aa',
      ]);

    await use(makeAxeBuilder);
  },
});

export { expect } from '@playwright/test';
```

## URL監査層: Pa11y

```bash
npm install --save-dev pa11y-ci
```

```json
// .pa11yci
{
  "defaults": {
    "standard": "WCAG2AA",
    "timeout": 30000,
    "chromeLaunchConfig": {
      "args": ["--no-sandbox"]
    }
  },
  "urls": [
    "http://localhost:3000/",
    "http://localhost:3000/about",
    "http://localhost:3000/contact"
  ]
}
```

```bash
# ビルド成果物を配信してから実行
npx serve -s dist -l 3000 &
npx wait-on http://localhost:3000
npx pa11y-ci
```

- 複数URLをまとめて監査したいときに向く
- ログイン後画面や複雑な操作後状態は Playwright の方が扱いやすい

## GitHub Actions への組み込み例

```yaml
# .github/workflows/a11y.yml
name: Accessibility

on:
  pull_request:
  push:
    branches: [main]

jobs:
  unit-a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm test -- --runTestsByPath src/**/*.a11y.test.jsx

  e2e-a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run build
      - run: npx serve -s dist -l 3000 &
      - run: npx wait-on http://localhost:3000
      - run: npx playwright test tests/a11y
        env:
          CI: true
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-a11y-report
          path: playwright-report/
          retention-days: 7
```

### ゲートの考え方

| 方針 | 内容 | 向く段階 |
|---|---|---|
| Soft fail | レポートだけ出し、マージは止めない | 導入初期 |
| Hard fail | 違反があればジョブ失敗 | ルールが安定してから |
| 差分ゲート | 新規違反のみ失敗 | 既存負債が多いとき |

- main へのマージ条件に a11y ジョブを含めると実効性が高い
- 失敗時はレポート成果物を残し、どのルール・どの要素かを追えるようにする

## 違反の扱い方

```js
// 一時的に特定ルールを除外する（理由をコメントで残す）
const results = await new AxeBuilder({ page })
  .disableRules(['color-contrast']) // TODO: デザイン刷新後に再有効化 (issue #123)
  .analyze();
```

```js
// 影響範囲を限定する
const results = await new AxeBuilder({ page })
  .include('main')
  .exclude('.third-party-widget')
  .analyze();
```

- 除外は「見なかったこと」と同じなので、Issue と期限をセットにする
- サードパーティ埋め込みは `exclude` し、自社 Dom を優先して守る

## Lighthouse CI（任意）

```bash
npm install --save-dev @lhci/cli
```

```yaml
# lighthouserc.js 相当の設定イメージ
ci:
  collect:
    url:
      - http://localhost:3000/
    numberOfRuns: 1
  assert:
    assertions:
      categories:accessibility:
        - error
        - minScore: 0.9
```

- アクセシビリティカテゴリのスコア監視に使える
- axe ほどルール詳細の制御はしづらいので、詳細検証は axe 側が主、Lighthouse は補助が一般的

## CIに入れても検出できないもの

- 代替テキストの文言の適切さ
- フォーカス順序の自然さ
- スクリーンリーダーでの読み上げ体験
- キーボード操作の分かりやすさ
- 一部の動的なフォーカス管理の是非

- CIは「壊れていないか」の安全網
- リリース前の手動チェックリストやスクリーンリーダー確認は残す

> 参照: 同ディレクトリの [axe-core / Lighthouseを使ったアクセシビリティ自動テスト](./a11y-automated-testing.md)、[手動テストのチェックリスト](./manual-testing-checklist.md)

## よくある間違い

### 1. トップページだけスキャンして終わりにする

- フォーム、ログイン後、モーダル表示中など、状態の異なる画面を含める

### 2. 違反を全部 ignore して緑にする

```js
// 間違い：意味がないゲート
expect(true).toBe(true);
```

### 3. jsdom だけの結果を過信する

- 実ブラウザでのコントラストや描画後状態は Playwright 等で補う

### 4. ローカルでは通るが CI でだけ落ちる構成にする

- ベース待ち不足、`serve` 忘れ、認証付きページ未考慮などが原因になりやすい
- `wait-on` と成果物アップロードを入れる

### 5. 自動テスト導入で手動テストをやめる

- 自動で見える範囲は限定的
- キーボードとスクリーンリーダー確認は別途残す

## 実装チェックリスト

- [ ] コンポーネントまたは E2E の少なくとも一方で axe を実行している
- [ ] WCAG の対象レベル（例: A/AA）を `withTags` などで明示している
- [ ] PR で a11y ジョブが動く
- [ ] 違反時にレポートやログが残る
- [ ] 除外ルールに理由と Issue がある
- [ ] 主要ユーザーフローの「操作後状態」もスキャンしている
- [ ] 手動テスト（キーボード / スクリーンリーダー）の位置づけがチームで共有されている

## まとめ

- CI/CDへの組み込みは、アクセシビリティのリグレッション防止が目的
- コンポーネント（jest-axe）・ページ（Playwright + axe）・URL監査（Pa11y）を段階的に重ねる
- GitHub Actions ではビルド → 配信 → スキャン → 失敗時レポート保存が基本形
- 違反0ゲートは有効だが、導入初期は対象範囲と除外を現実的に設計する
- 自動テストは必須の安全網であり、手動確認の代替にはならない
