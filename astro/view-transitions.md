# View Transitions in Astro — ページ遷移アニメーション

## View Transitions とは

- ページ遷移時にブラウザの **View Transition API** を使い、画面切り替えをアニメーションする仕組み
- MPA（マルチページアプリ）のまま、SPA 的な滑らかな遷移を実現
- Astro では `astro:transitions` モジュールが提供
- 旧 `<ViewTransitions />` は **`<ClientRouter />`** に名称変更（現行 API）

> 参照: [Astro — View transitions](https://docs.astro.build/en/guides/view-transitions/)

## 有効化 — ClientRouter

- デフォルトでは通常のフルページリロード遷移
- View Transitions を使うページの `<head>` に `<ClientRouter />` を追加
- サイト全体で使うなら共通 layout の head に置く

```astro
---
import { ClientRouter } from 'astro:transitions';
---
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <title>My Site</title>
    <ClientRouter />
  </head>
  <body>
    <slot />
  </body>
</html>
```

```astro
---
// 悪い例：body 内に置く（head が正しい）
import { ClientRouter } from 'astro:transitions';
---
<body>
  <ClientRouter />
  <slot />
</body>
```

> 参照: [Astro — ClientRouter](https://docs.astro.build/en/reference/modules/astro-transitions/#clientrouter-)

## 基本的な動作

- 同一サイト内の `<a href>` クリック時、クライアント側ルーティング + トランジション
- 旧ページから新ページへ DOM を差し替えながらアニメーション
- 外部リンク、`target="_blank"`、`download` 属性付きリンクは従来どおりフルナビゲーション
- フォーム POST なども通常のリクエストのまま

## transition:animate — アニメーション指定

- 要素やページ全体のアニメーションを制御するディレクティブ
- 組み込み値：`fade`（デフォルト）、`slide`、`none` など

```astro
---
import { ClientRouter } from 'astro:transitions';
---
<html transition:animate="fade">
  <head>
    <ClientRouter />
  </head>
  <body>
    <header>...</header>
    <main transition:animate="slide">
      <slot />
    </main>
  </body>
</html>
```

```astro
<!-- ページ全体のアニメーションを無効化し、main だけ slide -->
<html transition:name="root" transition:animate="none">
  <head><ClientRouter /></head>
  <body>
    <main transition:animate="slide">
      <slot />
    </main>
  </body>
</html>
```

- ルート `<html>` に `transition:animate="none"` → 子要素で個別に上書き、というパターンが多い
- `transition:name` で同一要素の連続性（マッチング）を制御

> 参照: [Astro — Built-in animation directives](https://docs.astro.build/en/guides/view-transitions/#built-in-animation-directives)

## transition:persist — 状態の保持

- ページ遷移後も特定要素の DOM / 状態を維持
- 動画再生位置、フォーム入力、オーディオプレイヤーなどに向く
- `transition:persist` に一意の名前を付ける

```astro
<audio transition:persist="player" controls>
  <source src="/audio/theme.mp3" type="audio/mpeg" />
</audio>
```

```astro
<!-- フォーム送信エラー時に入力値を保持 -->
<form method="POST">
  <input
    transition:persist
    required
    type="email"
    name="email"
  />
  <button type="submit">送信</button>
</form>
```

```astro
<!-- 悪い例：persist なしでエラー時に入力が消える -->
<input type="email" name="email" value="" />
```

- Astro Actions と組み合わせたフォーム UX 改善に使われる
- persist 対象が多すぎると、意図しない状態持ち越しに注意

> 参照: [Astro — transition:persist](https://docs.astro.build/en/guides/view-transitions/#transitionpersist)

## fallback — 非対応ブラウザ

- View Transition API 非対応ブラウザ向けのフォールバック
- `<ClientRouter fallback="..." />` で指定（デフォルト `'animate'`）

```astro
<ClientRouter fallback="swap" />
```

| fallback | 挙動 |
|---|---|
| `animate` | CSS アニメーションで擬似トランジション |
| `swap` | アニメーションなしで即座に差し替え |

> 参照: [Astro — Fallback control](https://docs.astro.build/en/guides/view-transitions/#fallback-control)

## アクセシビリティ — prefers-reduced-motion

- `<ClientRouter />` は `prefers-reduced-motion: reduce` を検知すると**すべての** view transition アニメーションを無効化
- フォールバックアニメーションも含めて無効
- DOM の差し替えのみ行い、動きのない遷移になる

```css
/* ClientRouter が内部で扱うイメージ（手動実装不要） */
@media (prefers-reduced-motion: reduce) {
  ::view-transition-old(*),
  ::view-transition-new(*) {
    animation: none;
  }
}
```

- 追加実装なしで基本対応される
- **カスタム CSS アニメーション**を足した場合は、同様に `@media (prefers-reduced-motion: reduce)` で無効化を検討

> 参照: [Astro — prefers-reduced-motion](https://docs.astro.build/en/guides/view-transitions/#prefers-reduced-motion)

## クライアントサイドナビゲーションの制御

- 特定リンクだけ通常遷移に戻したい場合

```astro
<a href="/logout" data-astro-reload>ログアウト</a>
```

- `data-astro-reload` でフルページリロードを強制
- 認証状態のリセットなど、完全なリロードが安全なケース向け

> 参照: [Astro — Preventing client-side navigation](https://docs.astro.build/en/guides/view-transitions/#preventing-client-side-navigation)

## 島（client コンポーネント）との関係

- View Transitions はページ単位の DOM 差し替え
- `client:load` 等の島は新ページ読み込み後に再ハイドレーション
- persist しない限り、島のクライアント状態はリセットされる

```astro
---
import Counter from '../components/Counter.jsx';
---
<Counter client:visible />
<!-- ページ遷移でカウンターは 0 に戻る（persist しない場合） -->
```

- ページ横断で状態を保ちたいなら、URL / cookie / サーバー側セッション等を検討
- persist は DOM 要素向けであり、任意の React state を自動保持するわけではない

## 向く / 向かない使い方

### 向く

- コーポレートサイト、ポートフォリオのページ間フェード
- 共通ヘッダー・音声プレイヤーの persist
- 軽い slide トランジションで体験を整えたい

### 向かない / 注意

- アクセシビリティ上、過度な motion が問題になるユーザー層への配慮（→ reduced-motion は組み込み対応）
- 複雑なクライアント状態をページ跨ぎで維持したい本格アプリ（→ SPA フレームワーク向き）
- すべてのリンクに派手なアニメーション（疲弊・混乱の原因）

## 悪い例

```astro
<!-- 悪い例：旧 API 名を使っている -->
import { ViewTransitions } from 'astro:transitions';
<ViewTransitions />
```

```astro
<!-- 悪い例：layout に ClientRouter がなく一部ページだけ -->
<!-- index.astro の head にだけ ClientRouter → 遷移先で効かない -->
```

- 現行は `ClientRouter` を layout に統一するのが無難

## 設計チェックリスト

- [ ] 共通 layout の `<head>` に `<ClientRouter />` がある
- [ ] `import { ClientRouter } from 'astro:transitions'` を使っている
- [ ] 過度なアニメーションを避け、`none` / `fade` から始めている
- [ ] persist は本当に必要な要素だけに付けている
- [ ] カスタムアニメーションに reduced-motion 対応を検討した
- [ ] ログアウト等は `data-astro-reload` でフルリロードしている

## まとめ

- View Transitions は `astro:transitions` の `<ClientRouter />` で opt-in する
- 旧 `ViewTransitions` ではなく現行 API 名 `ClientRouter` を使う
- `transition:animate` で fade / slide / none を制御、`transition:persist` で DOM 状態を保持
- `prefers-reduced-motion` 時は Astro がアニメーションを自動無効化
- コンテンツサイトの体験向上向き、複雑なクライアント状態管理の代替ではない
