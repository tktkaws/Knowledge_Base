# View Transitions API — ページ遷移アニメーション

## View Transitions APIとは

- ページ遷移やDOM更新時に**滑らかなアニメーション**を適用するためのブラウザAPI
- 従来は複雑なJavaScriptで実装していたページ遷移アニメーションをシンプルに実現できる
- **同一ドキュメント内の遷移**（SPA）と**クロスドキュメント遷移**（MPA）の両方に対応
- ブラウザが変更前後のスナップショットを取得し、その間をCSSアニメーションで補間する仕組み

> 参照: [MDN - View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API)

## 基本的な仕組み

### View Transitionsの流れ

```
1. document.startViewTransition() を呼び出す
2. ブラウザが現在の状態のスナップショット（old）を取得
3. コールバック内でDOMを更新する
4. ブラウザが更新後の状態のスナップショット（new）を取得
5. old → new のクロスフェードアニメーションが自動実行される
```

### 擬似要素ツリー

```
::view-transition
├── ::view-transition-group(root)
│   └── ::view-transition-image-pair(root)
│       ├── ::view-transition-old(root)   ← 変更前のスナップショット
│       └── ::view-transition-new(root)   ← 変更後のスナップショット
├── ::view-transition-group(card)
│   └── ::view-transition-image-pair(card)
│       ├── ::view-transition-old(card)
│       └── ::view-transition-new(card)
...
```

- ブラウザが自動的に擬似要素ツリーを生成する
- `::view-transition-old` は変更前のスクリーンショット（フェードアウト）
- `::view-transition-new` は変更後のスクリーンショット（フェードイン）
- CSSで各擬似要素のアニメーションをカスタマイズできる

> 参照: [web.dev - View Transitions](https://developer.chrome.com/docs/web-platform/view-transitions)

## 同一ドキュメント内の遷移（SPA）

### 基本的な実装

```javascript
// DOMの更新をstartViewTransitionで囲む
document.startViewTransition(() => {
  // DOM更新処理
  updateContent();
});
```

```javascript
// async/awaitパターン
document.startViewTransition(async () => {
  const response = await fetch('/api/data');
  const data = await response.json();
  renderContent(data);
});
```

- `document.startViewTransition()` にDOMを更新するコールバックを渡す
- コールバック実行前後のスナップショットが自動的に取得される
- デフォルトではページ全体のクロスフェードアニメーションが適用される

### Reactでの使用例

```javascript
// React 18+ でstartViewTransitionを使用
function handleNavigation(newPage) {
  document.startViewTransition(() => {
    // flushSyncで即座にDOMを更新
    flushSync(() => {
      setCurrentPage(newPage);
    });
  });
}
```

- Reactの `flushSync` を使うことで、`startViewTransition` のコールバック内で同期的にDOMを更新できる
- Next.jsやReact Routerなどのフレームワークでは専用のView Transitions統合が提供されている場合がある

> 参照: [MDN - document.startViewTransition()](https://developer.mozilla.org/en-US/docs/Web/API/Document/startViewTransition)

## view-transition-name — 要素の紐付け

### 特定の要素をアニメーションさせる

```css
/* ヘッダーはページ遷移時に固定したい */
.header {
  view-transition-name: header;
}

/* カード画像を遷移時に滑らかに移動させたい */
.card-image {
  view-transition-name: hero-image;
}
```

- `view-transition-name` を指定すると、その要素は**個別のview-transitionグループ**として扱われる
- 遷移前後で**同じ `view-transition-name`** を持つ要素が自動的に紐付けられる
- 紐付けられた要素間で位置・サイズの補間アニメーション（モーフィング）が行われる

### 注意: 名前の一意性

```css
/* 悪い例：同じページ内で同じ名前を複数の要素に付ける */
.card {
  view-transition-name: card;
  /* 複数のカードに同じ名前 → エラーになる */
}

/* 良い例：一意な名前を付ける */
.card:nth-child(1) { view-transition-name: card-1; }
.card:nth-child(2) { view-transition-name: card-2; }
.card:nth-child(3) { view-transition-name: card-3; }
```

- **同一ページ内で `view-transition-name` は一意**でなければならない
- 同じ名前が複数あるとView Transitionがスキップされる

> 参照: [MDN - view-transition-name](https://developer.mozilla.org/en-US/docs/Web/CSS/view-transition-name)

## CSSでのアニメーションカスタマイズ

### デフォルトのクロスフェードを変更する

```css
/* デフォルト: ページ全体が250msでクロスフェード */
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 0.4s;
}

/* フェードアウトを速く、フェードインをゆっくり */
::view-transition-old(root) {
  animation-duration: 0.2s;
}
::view-transition-new(root) {
  animation-duration: 0.5s;
}
```

### スライドアニメーション

```css
@keyframes slideOutLeft {
  to {
    transform: translateX(-100%);
  }
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
  }
}

::view-transition-old(root) {
  animation: slideOutLeft 0.3s ease-in both;
}

::view-transition-new(root) {
  animation: slideInRight 0.3s ease-out both;
}
```

### 特定の要素だけ別のアニメーションを適用

```css
/* ヘッダーはアニメーションしない（固定） */
::view-transition-old(header),
::view-transition-new(header) {
  animation: none;
}

/* ヒーロー画像はモーフィング（デフォルトの位置・サイズ補間） */
::view-transition-group(hero-image) {
  animation-duration: 0.5s;
  animation-timing-function: ease-in-out;
}

/* コンテンツエリアはフェード */
::view-transition-old(content) {
  animation: fadeOut 0.2s ease-out both;
}
::view-transition-new(content) {
  animation: fadeIn 0.3s ease-in both;
}
```

> 参照: [web.dev - View Transitions](https://developer.chrome.com/docs/web-platform/view-transitions)

## クロスドキュメント遷移（MPA）

### 基本的な設定

```css
/* 遷移元のページと遷移先のページの両方に記述 */
@view-transition {
  navigation: auto;
}
```

- `@view-transition` ルールで `navigation: auto` を指定するだけで、同一オリジン内のナビゲーションにView Transitionsが適用される
- JavaScriptは不要
- 遷移元と遷移先の両方のページにこのCSSを記述する必要がある

### MPAでの要素の紐付け

```css
/* 一覧ページ */
.article-thumbnail {
  view-transition-name: article-image;
}

/* 詳細ページ */
.article-hero {
  view-transition-name: article-image;
}
```

- 同じ `view-transition-name` を遷移前後のページで指定すると、要素間のモーフィングが行われる
- 一覧ページのサムネイルが詳細ページのヒーロー画像にスムーズに変化するような演出が可能

### MPAでの遷移タイプ

```css
@view-transition {
  navigation: auto;
  types: slide, forward;
}
```

```css
/* 進む遷移 */
::view-transition-old(root):active-view-transition-type(forward) {
  animation: slideOutLeft 0.3s ease-in both;
}
::view-transition-new(root):active-view-transition-type(forward) {
  animation: slideInRight 0.3s ease-out both;
}

/* 戻る遷移 */
::view-transition-old(root):active-view-transition-type(back) {
  animation: slideOutRight 0.3s ease-in both;
}
::view-transition-new(root):active-view-transition-type(back) {
  animation: slideInLeft 0.3s ease-out both;
}
```

- `types` で遷移の種類を指定し、CSSで種類ごとに異なるアニメーションを適用できる
- ブラウザの戻る/進む操作に応じてスライド方向を変えるなどの実装が可能

> 参照: [MDN - @view-transition](https://developer.mozilla.org/en-US/docs/Web/CSS/@view-transition)

## view-transition-class — 複数要素の一括スタイリング

```css
/* 個別に名前を付ける（一意性のため） */
.card:nth-child(1) { view-transition-name: card-1; }
.card:nth-child(2) { view-transition-name: card-2; }
.card:nth-child(3) { view-transition-name: card-3; }

/* クラスでアニメーションをまとめて定義 */
.card {
  view-transition-class: card;
}

::view-transition-group(*.card) {
  animation-duration: 0.3s;
  animation-timing-function: ease-in-out;
}
```

- `view-transition-name` は一意でなければならないが、`view-transition-class` で同じアニメーションスタイルをグルーピングできる
- `::view-transition-group(*.className)` のワイルドカード構文で一括スタイリング

> 参照: [MDN - view-transition-class](https://developer.mozilla.org/en-US/docs/Web/CSS/view-transition-class)

## 実践的なパターン

### リスト→詳細のモーフィング遷移

```css
/* 一覧ページ */
.product-card {
  view-transition-name: product-hero;
}
.product-card img {
  view-transition-name: product-image;
}
.product-card h2 {
  view-transition-name: product-title;
}

/* 詳細ページ */
.product-detail {
  view-transition-name: product-hero;
}
.product-detail img {
  view-transition-name: product-image;
}
.product-detail h1 {
  view-transition-name: product-title;
}

/* アニメーションのカスタマイズ */
::view-transition-group(product-image) {
  animation-duration: 0.4s;
  animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}
```

### ダークモード切り替え

```javascript
function toggleDarkMode() {
  document.startViewTransition(() => {
    document.documentElement.classList.toggle('dark');
  });
}
```

```css
/* 円形のクリップアニメーション */
::view-transition-old(root),
::view-transition-new(root) {
  animation: none;
  mix-blend-mode: normal;
}

::view-transition-new(root) {
  animation: circleReveal 0.5s ease-in-out;
}

@keyframes circleReveal {
  from {
    clip-path: circle(0% at top right);
  }
  to {
    clip-path: circle(150% at top right);
  }
}
```

### タブ切り替え

```javascript
function switchTab(tabId) {
  document.startViewTransition(() => {
    tabs.forEach(tab => tab.hidden = tab.id !== tabId);
  });
}
```

```css
.tab-content {
  view-transition-name: tab-content;
}

::view-transition-old(tab-content) {
  animation: fadeOut 0.15s ease-out both;
}
::view-transition-new(tab-content) {
  animation: fadeIn 0.15s ease-in both;
}
```

> 参照: [web.dev - View Transitions](https://developer.chrome.com/docs/web-platform/view-transitions)

## ブラウザサポートとフォールバック

- Chrome 111+（SPA）、Chrome 126+（MPA）
- Safari 18+（SPA）、Safari 18.2+（MPA の一部）
- **Firefox は未サポート**（2025年5月時点、開発中）

### プログレッシブエンハンスメント

```javascript
// 機能検出してからstartViewTransitionを使用
function navigateTo(url) {
  if (!document.startViewTransition) {
    // フォールバック：通常のDOM更新
    updateContent(url);
    return;
  }

  document.startViewTransition(() => {
    updateContent(url);
  });
}
```

```css
/* CSSでの機能検出 */
@supports (view-transition-name: test) {
  .card {
    view-transition-name: card;
  }
}
```

- `document.startViewTransition` の存在チェックでフォールバック処理を分岐する
- View Transitions APIが利用できない場合でもアプリケーションは正常に動作する
- アニメーションなしでも機能するように設計するのが基本

> 参照: [Can I Use - View Transitions](https://caniuse.com/view-transitions)

## prefers-reduced-motionへの対応

```css
@media (prefers-reduced-motion: reduce) {
  ::view-transition-old(root),
  ::view-transition-new(root) {
    animation-duration: 0.01ms;
  }

  /* すべてのview-transition-groupのアニメーションを無効化 */
  ::view-transition-group(*) {
    animation-duration: 0.01ms;
  }
}
```

```javascript
function navigateTo(url) {
  const prefersReducedMotion = window.matchMedia(
    '(prefers-reduced-motion: reduce)'
  ).matches;

  if (prefersReducedMotion || !document.startViewTransition) {
    updateContent(url);
    return;
  }

  document.startViewTransition(() => {
    updateContent(url);
  });
}
```

- `prefers-reduced-motion` が有効な場合はView Transitions自体をスキップする選択肢もある
- アニメーションのdurationを極小にして、遷移の論理的な構造は維持しつつ視覚的な動きだけ抑制する方法もある

> 参照: [MDN - prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)

## まとめ

- View Transitions APIはページ遷移やDOM更新に滑らかなアニメーションを加える仕組み
- `document.startViewTransition()` でSPA遷移、`@view-transition { navigation: auto }` でMPA遷移に対応
- `view-transition-name` で遷移前後の要素を紐付けると、位置・サイズのモーフィングが自動で行われる
- CSSの擬似要素（`::view-transition-old` / `::view-transition-new`）でアニメーションをカスタマイズ可能
- **Firefoxは未サポート**のため、プログレッシブエンハンスメントで実装する
- `prefers-reduced-motion` でアクセシビリティに配慮する

## 参照元

- [MDN - View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API)
- [MDN - document.startViewTransition()](https://developer.mozilla.org/en-US/docs/Web/API/Document/startViewTransition)
- [MDN - view-transition-name](https://developer.mozilla.org/en-US/docs/Web/CSS/view-transition-name)
- [MDN - @view-transition](https://developer.mozilla.org/en-US/docs/Web/CSS/@view-transition)
- [MDN - view-transition-class](https://developer.mozilla.org/en-US/docs/Web/CSS/view-transition-class)
- [web.dev - View Transitions](https://developer.chrome.com/docs/web-platform/view-transitions)
- [Can I Use - View Transitions](https://caniuse.com/view-transitions)
