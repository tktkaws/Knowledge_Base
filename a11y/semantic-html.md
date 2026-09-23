# セマンティックHTMLの原則 — div/spanに頼らないマークアップ

## セマンティックHTMLとは

- 要素の意味に合ったHTMLタグでコンテンツをマークアップすること
- 「見た目」ではなく「役割・意味」で要素を選ぶ
- ブラウザがロール・キーボード操作・デフォルトスタイルを自動提供する
- スクリーンリーダーや検索エンジンも、要素の意味を手がかりにページを理解する

> 参照: [MDN — HTML: A good basis for accessibility](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/HTML)

## なぜ div / span に頼りすぎてはいけないか

| `div` / `span` | セマンティック要素 |
|---|---|
| 意味を持たない汎用コンテナ | 見出し・リスト・ボタンなど役割が明確 |
| アクセシビリティツリー上の手がかりが少ない | 暗黙のロールが付く |
| キーボード操作は自前実装 | 多くは標準動作が付属 |
| 見た目は自由だが構造が弱い | 見た目はCSS、構造はHTMLで分担 |

- `div` / `span` 自体が悪いわけではない — **意味のある選択肢があるのに使わない**のが問題
- スタイリングやレイアウト用のラッパーとして使うのは妥当
- 「全部 `div`」だと、支援技術ユーザーがページ構造を掴みにくくなる

## 原則：適切な要素を選ぶ

1. そのUIの役割を言葉にする（見出し？リンク？ボタン？一覧？）
2. 同じ役割のネイティブ要素があるか確認する
3. あればそれを使う。なければ ARIA とキーボードを自前で揃える
4. 見た目の調整はCSSに任せる

- ARIAで補強する前に、ネイティブHTMLで済まないかを先に考える
- 詳細は [WAI-ARIAのファーストルール](./aria-first-rule.md) も参照

## よく使うセマンティック要素

### 文書構造

| 要素 | 用途 |
|---|---|
| `h1`〜`h6` | 見出し。ページのアウトラインを作る |
| `p` | 段落 |
| `ul` / `ol` / `li` | 箇条書き・順序付きリスト |
| `strong` / `em` | 重要性・強調（見た目の太字・斜体だけならCSS） |
| `blockquote` / `cite` | 引用 |
| `figure` / `figcaption` | 図表とそのキャプション |
| `hr` | テーマの区切り（装飾線だけならCSS） |

### ページ領域

| 要素 | 用途 |
|---|---|
| `header` | 導入・ブランド・ページやセクションの見出し周辺 |
| `nav` | 主要なナビゲーションリンク群 |
| `main` | そのページ固有の主コンテンツ（原則1つ） |
| `article` | 単体で完結する記事・投稿 |
| `section` | 見出し付きのまとまり |
| `aside` | 補足・関連情報 |
| `footer` | 著者情報・著作権・関連リンクなど |

- 領域（ランドマーク）の詳細は [ランドマークロールとページ構造の設計](./landmark-regions.md) を参照
- `section` は名前（見出しなど）がないとランドマークにならない点に注意

### 操作・入力

| 要素 | 用途 |
|---|---|
| `a[href]` | 別の場所へ移動する |
| `button` | その場で動作を起こす |
| `input` / `select` / `textarea` | フォーム入力 |
| `label` | フォームコントロールの名前 |
| `details` / `summary` | 開閉できる開示UI |

## 良い例・悪い例

### 見出し

```html
<!-- 悪い例：見た目だけ大きくした div -->
<div class="title">お知らせ</div>

<!-- 良い例：見出し要素を使い、見た目はCSS -->
<h2 class="title">お知らせ</h2>
```

- 見出しレベルは文書構造に合わせる（デザインの大きさだけで `h1` にしない）
- 見出しを飛ばす（`h2` の次が `h4`）のは避ける

### リスト

```html
<!-- 悪い例 -->
<div class="menu">
  <div><a href="/">ホーム</a></div>
  <div><a href="/about">会社概要</a></div>
</div>

<!-- 良い例 -->
<nav aria-label="メイン">
  <ul class="menu">
    <li><a href="/">ホーム</a></li>
    <li><a href="/about">会社概要</a></li>
  </ul>
</nav>
```

### ボタンとリンク

```html
<!-- 悪い例：クリック可能な div -->
<div class="btn" onclick="save()">保存</div>

<!-- 悪い例：遷移なのに button -->
<button onclick="location.href='/about'">会社概要へ</button>

<!-- 良い例：動作は button、移動は a -->
<button type="button" onclick="save()">保存</button>
<a href="/about">会社概要へ</a>
```

| やりたいこと | 使う要素 |
|---|---|
| ページ遷移・ダウンロード | `a[href]` |
| 送信・開閉・削除などの動作 | `button` |
| `href` のない「リンク風」 | `button`（または適切なコンポーネント） |

### フォーム

```html
<!-- 悪い例：ラベルが紐付いていない -->
<div>メールアドレス</div>
<input type="email" name="email">

<!-- 良い例 -->
<label for="email">メールアドレス</label>
<input id="email" type="email" name="email" autocomplete="email">
```

### テーブル

```html
<!-- 悪い例：レイアウトやデータの表を div で組む -->
<div class="row">
  <div class="cell">氏名</div>
  <div class="cell">部署</div>
</div>

<!-- 良い例：データ表は table -->
<table>
  <caption>社員一覧</caption>
  <thead>
    <tr>
      <th scope="col">氏名</th>
      <th scope="col">部署</th>
    </tr>
  </thead>
</table>
```

- データに行・列の関係があるなら `<table>`
- 見た目のグリッド配置だけなら CSS Grid / Flexbox

## div / span を使ってよい場面

```html
<!-- レイアウト用ラッパー -->
<div class="card-grid">
  <article>...</article>
  <article>...</article>
</div>

<!-- アイコンなど、意味を持たせないインライン装飾 -->
<button type="button">
  <span class="icon" aria-hidden="true"></span>
  設定
</button>

<!-- スタイリングのためのフック（意味は親が持つ） -->
<h2>
  <span class="eyebrow">News</span>
  お知らせ
</h2>
```

- 「意味のある要素で囲めない純粋な箱」が `div` / `span` の役割
- 装飾アイコンは `aria-hidden="true"` で読み上げから外す

## セマンティクスを見た目でごまかす例

```html
<!-- 悪い例：太字にしたいだけなのに strong -->
<p><strong>住所：</strong>東京都...</p>

<!-- 良い例：ラベルは構造かCSSで -->
<p><span class="label">住所：</span>東京都...</p>
```

```html
<!-- 悪い例：余白や区切りのために空の見出し -->
<h2>&nbsp;</h2>

<!-- 良い例：余白は margin / padding -->
```

- `strong` / `em` / 見出しは「意味」がある。見た目だけ変えたいときはCSS

## React などコンポーネント開発での注意

```jsx
// 悪い例：何でも div
export function Card({ title, children }) {
  return (
    <div className="card">
      <div className="card-title">{title}</div>
      <div className="card-body">{children}</div>
    </div>
  );
}

// 良い例：役割に合う要素をデフォルトにし、必要なら as / poly で差し替え
export function Card({ title, children }) {
  return (
    <article className="card">
      <h2 className="card-title">{title}</h2>
      <div className="card-body">{children}</div>
    </article>
  );
}
```

- デザインシステムでも、コンポーネントのルート要素は意味を持たせる
- `onClick` 付きの `div` をコンポーネント化しても、アクセシビリティ負債は残る

## よくある間違い

### 1. クリックできるものすべてを div にする

```html
<!-- 間違い -->
<div tabindex="0" role="button" onclick="...">メニュー</div>

<!-- 正解 -->
<button type="button">メニュー</button>
```

### 2. 見出しなのにフォントサイズだけの p / div

- 目次・ローター・SEOの手がかりを失う

### 3. リストなのに改行付きのテキストだけ

```html
<!-- 間違い -->
<p>りんご<br>みかん<br>ぶどう</p>

<!-- 正解 -->
<ul>
  <li>りんご</li>
  <li>みかん</li>
  <li>ぶどう</li>
</ul>
```

### 4. main / nav などを全部 div + role で再現する

```html
<!-- 避ける：同等のHTMLがある -->
<div role="navigation">...</div>
<div role="main">...</div>

<!-- 使う -->
<nav>...</nav>
<main>...</main>
```

### 5. セマンティック要素を使いすぎて意味が曖昧

```html
<!-- 迷走例：全部 article / section にする -->
<section><article><section>...</section></article></section>
```

- 区切りに必然性のあるところだけ `section` / `article` を使う
- 迷ったら、まず見出しと段落・リストを正しくする

## 実装チェックリスト

- [ ] 見出しが `h1`〜`h6` で、レベルが論理的
- [ ] 一覧は `ul` / `ol`、ナビは `nav` でマークアップ
- [ ] 動作は `button`、遷移は `a[href]`
- [ ] フォームに `label` が紐付いている
- [ ] データ表は `table`（レイアウト用途ではない）
- [ ] `div` / `span` はラッパー・装飾に限定している
- [ ] `onclick` 付きの非インタラクティブ要素がない
- [ ] 見た目のためだけに `strong` / 空見出しなどを使っていない

## まとめ

- セマンティックHTMLは、アクセシビリティの土台になる「正しい要素選び」
- `div` / `span` は意味がない箱。役割があるなら専用要素を使う
- ボタン・リンク・見出し・リスト・フォーム・ランドマークが特に効果が大きい
- 見た目はCSS、意味はHTML。ARIAはネイティブで足りないときの補完
- コンポーネント化しても、ルート要素の意味を捨てない
