# スキップリンクの実装と意義

## スキップリンクとは

- ページ上部に配置し、メインコンテンツへ直接ジャンプできるナビゲーションリンク
- 通常はページの最初のフォーカス可能な要素として配置
- キーボードユーザーがTabキーを押したときに最初に表示される
- 視覚的には非表示（フォーカス時のみ表示）にすることが一般的
- 英語では "Skip to main content" や "Skip navigation" と呼ばれる

> 参照: [W3C — WCAG 2.4.1 Bypass Blocks](https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks.html)

## なぜスキップリンクが必要か

- WCAG 2.4.1「ブロックスキップ」の達成基準を満たすための主要な手段
- キーボードのみで操作するユーザーが、毎回すべてのナビゲーションリンクをTabキーで通過する必要がなくなる
- スクリーンリーダーユーザーにとっても、繰り返されるコンテンツを飛ばす手段
- 大規模なヘッダーやナビゲーションを持つサイトほど効果が大きい

### スキップリンクが解決する問題

- 一般的なWebサイトはヘッダーに10〜30個以上のリンクが存在
- キーボードユーザーはページごとにそれらすべてをTabで通過しなければメインコンテンツに到達できない
- スキップリンクがあれば1回のTabとEnterでメインコンテンツへジャンプ可能
- マウスユーザーにとっては問題にならないが、キーボードユーザーにとっては深刻な操作負担

> 参照: [WebAIM — "Skip Navigation" Links](https://webaim.org/techniques/skipnav/)

## 基本的な実装方法

### HTML構造

```html
<!-- ページの最初の要素としてスキップリンクを配置 -->
<body>
  <a href="#main-content" class="skip-link">メインコンテンツへスキップ</a>

  <header>
    <nav>
      <ul>
        <li><a href="/">ホーム</a></li>
        <li><a href="/about">概要</a></li>
        <li><a href="/contact">お問い合わせ</a></li>
        <!-- 多数のナビゲーションリンク... -->
      </ul>
    </nav>
  </header>

  <main id="main-content">
    <h1>ページタイトル</h1>
    <p>メインコンテンツがここに入る</p>
  </main>
</body>
```

- `<body>` 直後にスキップリンクを配置
- `href="#main-content"` でページ内リンクとしてメインコンテンツへジャンプ
- ジャンプ先の `<main>` 要素に `id="main-content"` を付与

> 参照: [MDN — ページ内リンク（フラグメント識別子）](https://developer.mozilla.org/ja/docs/Web/HTML/Element/a#%E3%83%9A%E3%83%BC%E3%82%B8%E5%86%85%E3%83%AA%E3%83%B3%E3%82%AF)

## スキップリンクの表示パターン

### パターン1: フォーカス時のみ表示（推奨）

- 通常時は画面外に隠し、Tabキーでフォーカスされたときだけ表示
- 視覚デザインを損なわず、必要なユーザーだけが利用可能
- 多くの主要サイトが採用しているパターン

### パターン2: 常に表示

- ページ上部に常にスキップリンクを表示
- アクセシビリティへの配慮が明示的に伝わる
- デザインへの影響があるため採用は少ない

## CSSの実装

### visually-hidden クラスとフォーカス時の表示

```css
/* 通常時：画面外に配置して視覚的に非表示 */
.skip-link {
  position: absolute;
  top: -100%;
  left: 0;
  z-index: 100;
  padding: 0.75rem 1.5rem;
  background-color: #1a1a2e;
  color: #ffffff;
  font-size: 1rem;
  font-weight: bold;
  text-decoration: none;
  border-radius: 0 0 4px 0;
  /* フォーカス時に滑らかに表示 */
  transition: top 0.2s ease;
}

/* フォーカス時：画面内に表示 */
.skip-link:focus {
  top: 0;
  outline: 3px solid #f0a500;
  outline-offset: 2px;
}
```

### 悪い例: display: none や visibility: hidden で非表示にする

```css
/* 悪い例：display: none はフォーカスできなくなる */
.skip-link {
  display: none;
}
.skip-link:focus {
  display: block; /* Tabキーでフォーカスが当たらないため表示されない */
}

/* 悪い例：visibility: hidden もフォーカスできない */
.skip-link {
  visibility: hidden;
}
.skip-link:focus {
  visibility: visible; /* フォーカスが当たらないため動作しない */
}
```

- `display: none` や `visibility: hidden` はフォーカス対象から除外されるため、Tabキーで到達できない
- 視覚的に隠すには `position: absolute` で画面外に配置するか、`clip-path` を使用する

### 別の方法: clip-path を使った visually-hidden

```css
.skip-link {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}

.skip-link:focus {
  position: fixed;
  top: 0;
  left: 0;
  width: auto;
  height: auto;
  padding: 0.75rem 1.5rem;
  margin: 0;
  overflow: visible;
  clip-path: none;
  white-space: normal;
  background-color: #1a1a2e;
  color: #ffffff;
  font-size: 1rem;
  font-weight: bold;
  text-decoration: none;
  z-index: 9999;
  outline: 3px solid #f0a500;
  outline-offset: 2px;
}
```

> 参照: [CSS-Tricks — Inclusively Hidden](https://css-tricks.com/inclusively-hidden/)

## 複数のスキップリンク

- 大規模なサイトではメインコンテンツ以外へのスキップリンクも有用
- ただし多すぎるとかえってキーボード操作の負担が増えるため、2〜4個程度が目安

```html
<body>
  <div class="skip-links">
    <a href="#main-content" class="skip-link">メインコンテンツへスキップ</a>
    <a href="#primary-nav" class="skip-link">ナビゲーションへスキップ</a>
    <a href="#search" class="skip-link">検索へスキップ</a>
    <a href="#footer" class="skip-link">フッターへスキップ</a>
  </div>

  <header>
    <nav id="primary-nav">...</nav>
    <form id="search" role="search">
      <input type="search" aria-label="サイト内検索" />
      <button type="submit">検索</button>
    </form>
  </header>

  <main id="main-content">
    <h1>ページタイトル</h1>
  </main>

  <footer id="footer">...</footer>
</body>
```

```css
.skip-links {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 100;
}

.skip-link {
  position: absolute;
  top: -100%;
  left: 0;
  padding: 0.75rem 1.5rem;
  background-color: #1a1a2e;
  color: #ffffff;
  text-decoration: none;
  font-weight: bold;
  transition: top 0.2s ease;
}

.skip-link:focus {
  position: static;
  top: auto;
  display: block;
  outline: 3px solid #f0a500;
  outline-offset: 2px;
}
```

> 参照: [WCAG 2.4.1 — Bypass Blocks (Level A)](https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks.html)

## JavaScriptでのフォーカス制御

### 問題: ブラウザ間の挙動の違い

- 一部のブラウザではページ内リンク（`#main-content`）をクリックしてもフォーカスがターゲット要素に移動しない
- 特にChromium系ブラウザでは `<main>` 要素がフォーカス可能でない場合、スクロールはするがフォーカスは移動しない
- スクリーンリーダーの読み上げ位置もずれる可能性がある

### 対策: tabindex="-1" と JavaScript によるフォーカス移動

```html
<!-- ターゲット要素に tabindex="-1" を付与 -->
<main id="main-content" tabindex="-1">
  <h1>ページタイトル</h1>
  <p>メインコンテンツ</p>
</main>
```

```js
// スキップリンクのクリック時にフォーカスを確実に移動
document.querySelector('.skip-link').addEventListener('click', (event) => {
  const targetId = event.currentTarget.getAttribute('href').slice(1);
  const targetElement = document.getElementById(targetId);

  if (targetElement) {
    // tabindex="-1" がない場合は動的に追加
    if (!targetElement.hasAttribute('tabindex')) {
      targetElement.setAttribute('tabindex', '-1');
    }
    targetElement.focus();
  }
});
```

### フォーカス時のアウトラインを消す配慮

```css
/* tabindex="-1" の要素がフォーカスされたときアウトラインを非表示 */
[tabindex="-1"]:focus {
  outline: none;
}
```

- `tabindex="-1"` でプログラム的にフォーカスされた要素にはフォーカスリングを表示しない方が自然
- ユーザーが意図的にフォーカスした要素ではないため

> 参照: [WebAIM — Skip Navigation — Browser Support](https://webaim.org/techniques/skipnav/#browser)

## SPAでのスキップリンク

### 問題

- SPA（Single Page Application）ではページ遷移がブラウザのフルリロードを伴わない
- ルート変更後もフォーカスが前のページの要素に残ったままになる
- スキップリンクが初回ロード時しか機能しない可能性

### 対策: ルート変更時のフォーカス管理

```jsx
// React での実装例
import { useEffect, useRef } from 'react';
import { useLocation } from 'react-router-dom';

function App() {
  const skipLinkRef = useRef(null);
  const mainRef = useRef(null);
  const location = useLocation();

  useEffect(() => {
    // ルート変更時にメインコンテンツへフォーカスを移動
    if (mainRef.current) {
      mainRef.current.focus();
    }
  }, [location.pathname]);

  return (
    <>
      <a
        ref={skipLinkRef}
        href="#main-content"
        className="skip-link"
        onClick={(e) => {
          e.preventDefault();
          mainRef.current?.focus();
        }}
      >
        メインコンテンツへスキップ
      </a>

      <header>
        <nav>...</nav>
      </header>

      <main id="main-content" ref={mainRef} tabIndex={-1}>
        {/* ルーティングによるページコンテンツ */}
      </main>
    </>
  );
}
```

### Next.js での対応

- Next.js はフレームワークレベルでスキップリンクに対する配慮がない（v14時点）
- 自前でスキップリンクコンポーネントを実装する必要がある

```tsx
// components/SkipLink.tsx
'use client';

import { usePathname } from 'next/navigation';
import { useEffect, useRef } from 'react';

export function SkipLink() {
  const pathname = usePathname();
  const isFirstRender = useRef(true);

  useEffect(() => {
    // 初回レンダリング時はスキップ
    if (isFirstRender.current) {
      isFirstRender.current = false;
      return;
    }
    // ルート変更時にメインコンテンツにフォーカス
    const main = document.getElementById('main-content');
    if (main) {
      main.setAttribute('tabindex', '-1');
      main.focus();
    }
  }, [pathname]);

  return (
    <a href="#main-content" className="skip-link">
      メインコンテンツへスキップ
    </a>
  );
}
```

> 参照: [Gatsby — Skip Navigation](https://www.gatsbyjs.com/blog/2020-02-10-accessible-client-side-routing-improvements/)

## ランドマークとの併用

- スキップリンクとARIAランドマーク（またはHTML5セクショニング要素）は補完関係
- ランドマークはスクリーンリーダーユーザーがページの構造を把握し素早く移動するための仕組み
- スキップリンクはキーボードユーザー全般に有効
- 両方を併用することで、より多くのユーザーをカバーできる

```html
<body>
  <a href="#main-content" class="skip-link">メインコンテンツへスキップ</a>

  <!-- banner ランドマーク -->
  <header>
    <!-- navigation ランドマーク -->
    <nav aria-label="メインナビゲーション">
      <ul>
        <li><a href="/">ホーム</a></li>
        <li><a href="/products">商品</a></li>
      </ul>
    </nav>
  </header>

  <!-- main ランドマーク -->
  <main id="main-content" tabindex="-1">
    <h1>商品一覧</h1>
  </main>

  <!-- complementary ランドマーク -->
  <aside aria-label="関連情報">
    <h2>おすすめ商品</h2>
  </aside>

  <!-- contentinfo ランドマーク -->
  <footer>
    <p>&copy; 2026 Example Inc.</p>
  </footer>
</body>
```

| 手段 | 対象ユーザー | 機能 |
|---|---|---|
| スキップリンク | キーボードユーザー全般 | メインコンテンツへ直接ジャンプ |
| ランドマーク | スクリーンリーダーユーザー | ページ構造の把握と各セクションへの移動 |

> 参照: [W3C WAI — Landmark Regions](https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/)

## 良い例・悪い例

### 悪い例1: スキップリンクが存在しない

```html
<!-- 悪い例：スキップリンクなし -->
<body>
  <header>
    <nav>
      <a href="/">ホーム</a>
      <a href="/about">概要</a>
      <a href="/services">サービス</a>
      <a href="/blog">ブログ</a>
      <a href="/contact">お問い合わせ</a>
      <!-- さらに多くのリンク... -->
    </nav>
  </header>
  <main>
    <h1>ページタイトル</h1>
  </main>
</body>
```

- キーボードユーザーは毎回すべてのナビゲーションリンクをTabで通過する必要がある

### 悪い例2: display: none で隠している

```html
<!-- 悪い例：display: none はフォーカスできない -->
<a href="#main" style="display: none;">スキップ</a>
```

- `display: none` の要素はフォーカス順序から除外されるため、Tabキーで到達不可能

### 悪い例3: ジャンプ先にIDがない

```html
<!-- 悪い例：ジャンプ先のIDが存在しない -->
<a href="#content" class="skip-link">メインコンテンツへ</a>

<header>...</header>
<!-- id="content" がどこにもない -->
<main>
  <h1>ページタイトル</h1>
</main>
```

- リンク先のIDが存在しないため、クリックしても何も起こらない

### 良い例: 完成形

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>スキップリンクの実装例</title>
  <style>
    .skip-link {
      position: absolute;
      top: -100%;
      left: 0;
      z-index: 9999;
      padding: 0.75rem 1.5rem;
      background-color: #1a1a2e;
      color: #ffffff;
      font-size: 1rem;
      font-weight: bold;
      text-decoration: none;
      transition: top 0.2s ease;
    }
    .skip-link:focus {
      top: 0;
      outline: 3px solid #f0a500;
      outline-offset: 2px;
    }
    [tabindex="-1"]:focus {
      outline: none;
    }
  </style>
</head>
<body>
  <a href="#main-content" class="skip-link">メインコンテンツへスキップ</a>

  <header>
    <nav aria-label="メインナビゲーション">
      <ul>
        <li><a href="/">ホーム</a></li>
        <li><a href="/about">概要</a></li>
        <li><a href="/services">サービス</a></li>
        <li><a href="/blog">ブログ</a></li>
        <li><a href="/contact">お問い合わせ</a></li>
      </ul>
    </nav>
  </header>

  <main id="main-content" tabindex="-1">
    <h1>ようこそ</h1>
    <p>メインコンテンツがここに表示される</p>
  </main>

  <footer>
    <p>&copy; 2026 Example Inc.</p>
  </footer>

  <script>
    document.querySelector('.skip-link').addEventListener('click', (event) => {
      const targetId = event.currentTarget.getAttribute('href').slice(1);
      const target = document.getElementById(targetId);
      if (target) {
        target.focus();
      }
    });
  </script>
</body>
</html>
```

## 実際のサイトでの実装例

### GitHub

- Tabキーを1回押すと「Skip to content」というリンクが画面上部に表示
- フォーカスが外れると非表示に戻る
- ジャンプ先はリポジトリのメインコンテンツエリア

### GOV.UK（英国政府公式サイト）

- 「Skip to main content」が最初のフォーカス要素
- 黄色の背景にはっきりとした黒文字で表示
- 英国のアクセシビリティ基準に基づく模範的な実装

### YouTube

- 「Skip navigation」リンクがページ最上部に配置
- フォーカス時に表示されるパターン
- メインコンテンツ領域（動画一覧やプレーヤー）へジャンプ

### 共通する特徴

- ページ内の最初のフォーカス可能な要素として配置
- フォーカス時のみ視覚的に表示（visually-hidden パターン）
- 高コントラストでフォーカスインジケーターが明確
- `tabindex="-1"` をジャンプ先に設定してフォーカス移動を確実にしている

> 参照: [GitHub](https://github.com), [GOV.UK](https://www.gov.uk), [YouTube](https://www.youtube.com)

## まとめ

- スキップリンクはWCAG 2.4.1（Level A）を満たすための基本的な手段
- `<body>` 直後に配置し、メインコンテンツの `id` へリンクする
- CSSでは `position: absolute` + `:focus` でフォーカス時のみ表示するのが推奨パターン
- `display: none` や `visibility: hidden` で隠してはいけない — フォーカスが当たらなくなる
- ジャンプ先には `tabindex="-1"` を設定し、JavaScriptで `focus()` を呼ぶとブラウザ間の挙動差を吸収できる
- SPAではルート変更時のフォーカス管理と組み合わせる
- ランドマーク（`<header>`, `<nav>`, `<main>`, `<footer>`）との併用でより堅牢なアクセシビリティを実現
