# Astro × swup Fragment Plugin — タグ一覧だけを差し替える

## Fragment Plugin とは

- 通常の swup では `main`（とサイトでは `header`）全体が差し替わる
- タグを切り替えるたびにヘッダーやタグナビまでフェードする — 実際に変わっているのは記事一覧だけなのに「ページ移動した」ように見える
- [Fragment Plugin](https://swup.js.org/plugins/fragment-plugin/) は、URL の組み合わせごとに「今回はどのコンテナだけ差し替えるか」をルールで指定できる
- 公式デモの [Characters フィルタ](https://swup-fragment-plugin.netlify.app/characters/) と同じ発想
- 導入の全体像は [Astroにswupを導入する — ページ遷移をスムーズにする](./swup-setup.md) を参照

トップ（`/`）とタグページ（`/tags/:slug/`）は、どちらも次の二段構成。

1. 上: タグナビ（`TagsList`）
2. 下: 記事一覧（`ArticlesList`）

fragment にすると、次の体験になる。

- タグナビは DOM ごと残る（ちらつかない）
- 記事一覧だけ差し替わってフェード／スタガーする
- スクロール位置がリセットされない（Fragment の既定）
- キーボードで Enter したタグリンクにフォーカスが残る

```mermaid
flowchart LR
  click[タグリンク] --> match{from/to が / または /tags/:slug}
  match -->|yes| frag["差し替え: #articles のみ"]
  match -->|no| full["通常: main と header"]
  frag --> anim[一覧アイテムのスタガー]
  frag --> sync[JS で current 表示を同期]
```

## `@swup/astro` での有効化

- 追加パッケージは不要 — `@swup/astro` が `@swup/fragment-plugin` を含んでいる
- `fragments` オプションを渡すと有効になる

```js
// astro.config.mjs
swup({
  containers: ['main', 'header'],
  globalInstance: true,
  theme: 'fade',
  fragments: [
    {
      from: ['/', '/tags/:slug', '/tags/:slug/'],
      to: ['/', '/tags/:slug', '/tags/:slug/'],
      containers: ['#articles'],
      name: 'tag-filter',
      focus: false,
    },
  ],
}),
```

| キー | 役割 |
|---|---|
| `from` / `to` | 直前 URL と次 URL — どちらもマッチしたときだけ fragment 扱い |
| `containers` | 差し替える要素 — **ID セレクタのみ**（`#articles` は可、`.articles` は不可） |
| `name` | ルール名 — `to-tag-filter` のようなクラスでスタイルを絞れる |
| `focus` | Accessibility Plugin のフォーカス移動を制御 — `false` でトリガー要素にフォーカスを残す |

- ルールにマッチしない遷移（記事詳細 ↔ トップなど）は、従来どおり `containers: ['main', 'header']` が使われる
- 末尾スラッシュの有無でマッチが外れることがあるため、`/tags/:slug` と `/tags/:slug/` の両方を書く
- トップは `/` を `from` / `to` に含め、ホーム ↔ タグも同じ fragment にする

## マークアップ: 差し替え対象に ID を付ける

Fragment の `containers` は ID 必須。記事一覧側に `id="articles"` を付ける。

```astro
<!-- src/components/ArticlesList.astro -->
<section id="articles" class:list={['articles', className]}>
  <ul>
    {posts.map((post, index) => (
      <li
        class="article-item border-b border-border-muted"
        style={`transition-delay: ${index * 0.1}s`}
      >
        <!-- 記事リンク -->
      </li>
    ))}
  </ul>
</section>
```

ポイントは次のとおり。

- fragment 要素は、通常の swup コンテナ（ここでは `main`）の**内側**にある必要がある
- 新旧ページの両方に同じ ID が存在しないと、そのルールはスキップされる
- タグナビにも `id="tags"` を付けるが、今回のルールでは差し替えない — 選択中表示の同期用
- 同じ `ArticlesList` を使う箇所にも `id="articles"` が付くが、ルールの URL がタグ／ホームに限定されているため副作用はない

## タグナビを残す設計と current の同期

- 公式デモでは、フィルタ UI ごとひとつのコンテナに入れて差し替えている
- このサイトでは「タグ一覧は DOM ごと残したい」ので、差し替えは `#articles` だけにした
- その結果、SSR 時点の `.current` / `aria-current` は、タグをクリックしても自動では更新されない
- URL を見てクライアント側で同期する

```ts
// src/scripts/tags-current.ts
function syncCurrentTag() {
  const nav = document.querySelector('#tags nav[aria-label="Tags"]');
  if (!(nav instanceof HTMLElement)) return;

  const currentSlug =
    window.location.pathname.match(/^\/tags\/([^/]+)\/?$/)?.[1] ?? null;

  for (const link of nav.querySelectorAll<HTMLAnchorElement>('a.tag-link')) {
    const linkSlug = link.pathname.match(/^\/tags\/([^/]+)\/?$/)?.[1] ?? null;
    const isCurrent = currentSlug !== null && linkSlug === currentSlug;

    link.classList.toggle('current', isCurrent);
    if (isCurrent) {
      link.setAttribute('aria-current', 'page');
    } else {
      link.removeAttribute('aria-current');
    }
  }
}

syncCurrentTag();
document.addEventListener('astro:page-load', syncCurrentTag);
```

- `@swup/astro` は遷移後に `astro:page-load` をディスパッチするため、初回表示と fragment 遷移の両方で同じ関数を回せる
- スクリプトは差し替えコンテナの外に置き、永続させる

## アニメーションとフォーカス

### コンテナではなく項目を動かす

Fragment 訪問では、アニメーション用クラスが `#articles` に付く。一覧全体ではなく、各 `.article-item` の opacity をスタガーで変える。

```css
#articles.is-changing {
  --duration-leave: 150ms;
  --duration-enter: 400ms;
  transition-duration: var(--duration-enter);
}
#articles .article-item {
  transition-property: opacity;
  transition-duration: 400ms;
  transition-timing-function: cubic-bezier(0.23, 1, 0.32, 1);
}
#articles.is-animating .article-item {
  opacity: 0;
}
```

- `transition-delay` はマークアップ側で `index * 0.1s` を付ける
- transition を `#articles.is-changing .article-item` にだけ書くと、クラス除去後に delay が無効になる
- transition 自体は `.article-item` に常時持たせ、opacity の切り替えだけを `is-animating` に任せる
- モーション低減時は `html[data-motion='reduce']` や `visit.animation.animate = false` が fragment 訪問にも効く

### フォーカスをタグリンクに残す

- Accessibility Plugin は遷移後にフォーカスを `body` などへ移す
- タグナビを残しているのにフォーカスだけ奪われると、キーボード操作で「いま押したリンク」から外れる
- Fragment ルールの `focus: false` は、その訪問について a11y のフォーカス移動を止める

```js
{
  containers: ['#articles'],
  name: 'tag-filter',
  focus: false,
}
```

- 記事詳細など通常遷移では、これまでどおり a11y プラグインがフォーカスを扱う

## フル遷移でトップに戻ったときのスタガー

- 記事詳細 → トップは fragment ルールにマッチしないため、`main` 全体のフェードになる
- `#articles` には fragment 用クラスが付かないので、項目スタガーも自動では走らない
- `page:view` フックで、ホームへのフル遷移のときだけ同じスタガーを JS から再生する

```ts
// src/scripts/articles-home-stagger.ts（要旨）
swup.hooks.on('page:view', (visit) => {
  if (visit.fragmentVisit) return; // fragment 側は CSS に任せる
  playHomeArticlesStagger();       // 内部で pathname === '/' を確認
});
```

実装上の注意。

- `@swup/astro` の `reloadScripts` によりスクリプトが再実行されることがある — `window` 上のフラグでフック二重登録を防ぐ
- `astro:page-load` と「初回スキップ」フラグだけに頼ると、再実行のたびに初回扱いになりスタガーが毎回スキップされる
- 再生時はいったん即時非表示（`transition: none`）してからフェードインすると、退場アニメが混ざらない
- `main` の左右ボーダーがフェードで消える場合は、差し替え対象外に `.site-frame` を置き、絶対配置の線だけを常時表示する

## まとめ

- Fragment Plugin は「フィルタ UI ＋結果リスト」のような、**ページの一部だけが変わる遷移**向け
- `@swup/astro` では `fragments` 配列を渡すだけで有効化できる
- 差し替えを `#articles` に限定し、タグナビは残す／選択状態は JS で同期する、という分け方が扱いやすい
- アニメはコンテナ全体よりリスト項目単位のほうが、「一覧だけ動く」印象になる
- a11y のフォーカス移動と、フル遷移時のアニメ補完は、ルール外の体験として別途整える
- 末尾スラッシュゆれと ID の有無は、ルールがスキップされる典型原因

> 参照: [Fragment Plugin（公式）](https://swup.js.org/plugins/fragment-plugin/) / [Fragment デモ: Characters](https://swup-fragment-plugin.netlify.app/characters/) / [swup Accessibility Plugin — AJAX遷移でも読み上げとフォーカスを保つ](../a11y/swup-accessibility-plugin.md)
