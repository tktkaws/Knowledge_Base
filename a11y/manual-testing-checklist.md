# 手動テストのチェックリスト — 最低限確認すべき項目

## 手動テストが必要な理由

- 自動テスト（axe-core、Lighthouseなど）で検出できるのはアクセシビリティ問題全体の約30〜40%とされている
- 自動テストでは「技術的に正しいが、ユーザー体験として成立しない」ケースを検出できない
- 例：`alt` 属性が存在しても、内容が適切かどうかは人間が判断する必要がある
- 例：フォーカス順序がDOM順に正しくても、視覚的な流れと一致しているかは目視で確認する必要がある
- 例：スクリーンリーダーでの読み上げが自然な文脈になっているかは実際に聞いて確認する必要がある
- 自動テストと手動テストの組み合わせにより、WCAGの達成基準を幅広くカバーできる

> 参照: [Deque — What Can Automated Testing Really Cover?](https://www.deque.com/blog/automated-testing-study-identifies-57-percent-of-digital-accessibility-issues/)

## キーボード操作の確認項目

### Tab移動

- 全てのインタラクティブ要素（リンク、ボタン、フォーム入力）にTabキーで到達できるか
- フォーカス順序が視覚的なレイアウト順と一致しているか
- フォーカスが画面外やvisually hiddenな要素にトラップされていないか
- ページの末尾までTabした後、ブラウザのアドレスバーに戻るか（フォーカスが循環しないか）

### Enter / Space 操作

- ボタンがEnterキーとSpaceキーの両方で動作するか
- リンクがEnterキーで遷移するか
- チェックボックスがSpaceキーで切り替わるか
- カスタムコンポーネント（`div` + `role="button"` など）もキーボードで操作できるか

### Escape キー

- モーダルダイアログがEscapeで閉じるか
- ドロップダウンメニューがEscapeで閉じるか
- 閉じた後、開いたトリガー要素にフォーカスが戻るか

### フォーカス表示

- 現在フォーカスされている要素が視覚的に判別できるか
- `outline: none` でフォーカスリングが消されていないか
- カスタムフォーカススタイルが十分なコントラスト比を持っているか

```html
<!-- 悪い例：フォーカスリングを消している -->
<style>
  *:focus { outline: none; }
</style>
<button>送信</button>

<!-- 良い例：:focus-visible でカスタムフォーカスを提供 -->
<style>
  :focus-visible {
    outline: 2px solid #005fcc;
    outline-offset: 2px;
  }
</style>
<button>送信</button>
```

> 参照: [WCAG 2.1.1 Keyboard](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html), [WCAG 2.4.7 Focus Visible](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html)

## スクリーンリーダーでの確認項目

### 見出し構造

- 見出しレベル（h1〜h6）がページ内容の論理構造を正しく反映しているか
- h1がページに1つだけ存在するか
- 見出しレベルの飛ばし（h1 → h3 など）がないか
- スクリーンリーダーの見出しリスト機能で、ページ構造が把握できるか

```html
<!-- 悪い例：見出しレベルが飛んでいる -->
<h1>サイトタイトル</h1>
<h3>セクション1</h3>  <!-- h2 を飛ばしている -->
<h4>サブセクション</h4>

<!-- 良い例：見出しレベルが連続している -->
<h1>サイトタイトル</h1>
<h2>セクション1</h2>
<h3>サブセクション</h3>
```

### ランドマーク

- `<header>`, `<nav>`, `<main>`, `<footer>` などのランドマーク要素が適切に配置されているか
- `<main>` がページに1つだけ存在するか
- ナビゲーションが複数ある場合、`aria-label` で区別されているか
- スクリーンリーダーのランドマーク一覧で、ページの大まかな構造が把握できるか

```html
<!-- 悪い例：ランドマークなし（全て div） -->
<div class="header">...</div>
<div class="nav">...</div>
<div class="content">...</div>
<div class="footer">...</div>

<!-- 良い例：セマンティックなランドマーク要素 -->
<header>...</header>
<nav aria-label="メインナビゲーション">...</nav>
<main>...</main>
<nav aria-label="フッターナビゲーション">...</nav>
<footer>...</footer>
```

### 代替テキスト

- 意味のある画像に適切な `alt` テキストが設定されているか
- 装飾目的の画像に `alt=""` または `role="presentation"` が設定されているか
- `alt` の内容が画像の目的を適切に伝えているか（「画像」「写真」などの冗長な記述がないか）
- アイコンボタンに `aria-label` が設定されているか

```html
<!-- 悪い例：altが不適切 -->
<img src="chart.png" alt="画像" />
<button><svg>...</svg></button>  <!-- ラベルなし -->

<!-- 良い例：適切な代替テキスト -->
<img src="chart.png" alt="2024年の月別売上推移グラフ。6月が最高で500万円" />
<button aria-label="メニューを開く"><svg>...</svg></button>
```

### ライブリージョン

- 動的に追加されるコンテンツ（トースト通知、エラーメッセージなど）がスクリーンリーダーに通知されるか
- `aria-live="polite"` または `aria-live="assertive"` が適切に使い分けられているか
- ライブリージョンが過剰に発火して、ユーザーの操作を妨げていないか

### フォームラベル

- 全ての入力フィールドにラベルが紐付いているか
- スクリーンリーダーで入力欄にフォーカスしたとき、何を入力するフィールドか分かるか
- 必須フィールドが「必須」と読み上げられるか
- エラーメッセージが該当フィールドに紐付いて読み上げられるか

> 参照: [WCAG 1.3.1 Info and Relationships](https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html), [WCAG 1.1.1 Non-text Content](https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html)

## 視覚的な確認項目

### コントラスト比

- 通常テキスト（14px以下）のコントラスト比が 4.5:1 以上か（WCAG AA）
- 大きなテキスト（18px以上、または14px太字以上）のコントラスト比が 3:1 以上か
- UIコンポーネント（ボタンのボーダー、フォーム入力のアウトラインなど）のコントラスト比が 3:1 以上か
- フォーカスインジケーターのコントラスト比が 3:1 以上か
- 確認ツール：ブラウザDevToolsのコントラスト比表示、[WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)

### 文字サイズ拡大 200%

- ブラウザのズーム機能で200%に拡大した際に、コンテンツが欠けないか
- テキストが重なったり、はみ出して読めなくなっていないか
- 横スクロールが発生していないか（縦スクロールのみで全コンテンツにアクセスできるか）
- ボタンやリンクのタップ領域が十分に確保されているか

```css
/* 悪い例：固定のピクセル値でコンテナを制限 */
.container {
  width: 960px;
  height: 600px;
  overflow: hidden; /* 拡大時にコンテンツが切れる */
}

/* 良い例：相対単位とflexibleなレイアウト */
.container {
  max-width: 60rem;
  min-height: 100vh;
  overflow: visible;
}
```

### 色だけに依存しない情報伝達

- エラー表示が赤色だけでなくテキストやアイコンでも伝えられているか
- グラフやチャートで色以外の手段（パターン、ラベル）で情報が区別できるか
- リンクが色だけでなく下線などで周囲のテキストと区別できるか
- ステータス表示（成功/警告/エラー）が色以外でも判別できるか

```html
<!-- 悪い例：色だけでステータスを表現 -->
<span style="color: green;">完了</span>
<span style="color: red;">エラー</span>

<!-- 良い例：アイコン + テキスト + 色 -->
<span class="status-success">&#10003; 完了</span>
<span class="status-error">&#9888; エラー: メールアドレスが無効です</span>
```

> 参照: [WCAG 1.4.3 Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html), [WCAG 1.4.4 Resize Text](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html), [WCAG 1.4.1 Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html)

## インタラクション確認

### モーダルダイアログ

- 開いたときにフォーカスがモーダル内に移動するか
- Tab / Shift+Tab でフォーカスがモーダル内にトラップされるか（背景の要素にフォーカスが移動しないか）
- Escapeキーで閉じるか
- 閉じた後、トリガー要素にフォーカスが戻るか
- モーダルの背景が `aria-hidden="true"` で支援技術から隠されているか
- `role="dialog"` と `aria-modal="true"` が設定されているか

### ドロップダウンメニュー

- トリガーボタンに `aria-expanded` が正しく切り替わるか
- 矢印キーでメニュー項目間を移動できるか
- Escapeでメニューが閉じ、トリガーにフォーカスが戻るか
- メニュー項目をEnterで選択できるか

### タブUI

- タブリストに `role="tablist"` が設定されているか
- 各タブに `role="tab"`、パネルに `role="tabpanel"` が設定されているか
- 矢印キーでタブ間を移動できるか
- 選択中のタブに `aria-selected="true"` が設定されているか
- タブとパネルが `aria-controls` / `aria-labelledby` で紐付いているか

### フォームエラー

- エラーメッセージが `aria-describedby` で入力欄に紐付いているか
- エラー発生時に `aria-invalid="true"` が設定されるか
- 送信時にエラーがある場合、最初のエラーフィールドにフォーカスが移動するか
- エラーサマリーが `role="alert"` で通知されるか

> 参照: [WAI-ARIA Authoring Practices Guide — Patterns](https://www.w3.org/WAI/ARIA/apg/patterns/)

## テスト手順の具体例

### 手順1: キーボード操作テスト

1. マウスを使わずにページを開く
2. Tabキーだけで全てのインタラクティブ要素を巡回する
3. 各要素でフォーカスリングが見えることを確認する
4. ボタンやリンクをEnter / Spaceで操作する
5. モーダルやドロップダウンを開き、Escapeで閉じる
6. フォームを入力し、Enterで送信する

### 手順2: スクリーンリーダーテスト

1. スクリーンリーダーを起動する（macOSならVoiceOver: `Cmd + F5`）
2. 見出しジャンプ機能でページの構造を確認する（VoiceOverは `VO + Cmd + H`）
3. ランドマーク一覧でページの領域を確認する
4. 画像にフォーカスして代替テキストが読み上げられるか確認する
5. フォームの各入力欄にフォーカスしてラベルが読み上げられるか確認する
6. 動的コンテンツ（通知、エラーなど）が自動的に読み上げられるか確認する

### 手順3: 視覚テスト

1. ブラウザのズームを200%に設定する（`Cmd/Ctrl + +`）
2. 全てのコンテンツが読める状態か、レイアウトが崩れていないかを確認する
3. DevToolsのコントラスト比チェッカーでテキストのコントラストを確認する
4. ページをグレースケールで表示し、色だけに依存している情報がないか確認する
   - macOS: システム環境設定 → アクセシビリティ → ディスプレイ → カラーフィルタ
   - Chrome DevTools: Rendering → Emulate vision deficiencies → Achromatopsia

## 主要なスクリーンリーダーの紹介

### VoiceOver（macOS / iOS）

- Apple製品に標準搭載されている無料のスクリーンリーダー
- macOSでの起動：`Cmd + F5`（またはシステム設定 → アクセシビリティ → VoiceOver）
- 操作の基本キー：`VO`キー（`Control + Option`）
- 主な操作:
  - `VO + →` / `VO + ←` : 次/前の要素に移動
  - `VO + Cmd + H` : 次の見出しに移動
  - `VO + Cmd + L` : 次のリンクに移動
  - `VO + U` : ローター（見出し・リンク・ランドマーク一覧）を表示
- SafariとChromeで動作するが、macOSではSafariとの組み合わせが最も安定

### NVDA（Windows）

- 無料かつオープンソースのスクリーンリーダー
- [NVAccess](https://www.nvaccess.org/) からダウンロード可能
- 主な操作:
  - `H` : 次の見出しに移動
  - `D` : 次のランドマークに移動
  - `Tab` : 次のフォーム要素に移動
  - `Insert + F7` : 要素リスト（見出し・リンク・ランドマーク一覧）を表示
- Chrome、Firefox、Edgeで動作
- 開発者がWindowsでテストする際の第一候補

### JAWS（Windows）

- Freedom Scientific社の商用スクリーンリーダー
- 企業環境や政府機関で広く使用されている
- 有料だが40分間の試用モードあり（PC再起動ごとにリセット）
- 操作体系はNVDAと類似
- WebAIMの調査によると、世界で最も使用されているスクリーンリーダーの1つ

### 開発時の推奨環境

| 環境 | スクリーンリーダー | ブラウザ |
|---|---|---|
| macOS | VoiceOver | Safari |
| Windows | NVDA | Chrome または Firefox |
| iOS | VoiceOver | Safari |
| Android | TalkBack | Chrome |

> 参照: [WebAIM — Screen Reader User Survey](https://webaim.org/projects/screenreadersurvey10/), [WAI — Testing with Assistive Technologies](https://www.w3.org/WAI/test-evaluate/)

## チェックリスト形式のまとめ

### キーボード操作

- [ ] 全てのインタラクティブ要素にTabキーで到達できる
- [ ] フォーカス順序が視覚的な順序と一致している
- [ ] フォーカスインジケーターが全ての要素で視認できる
- [ ] ボタンがEnter / Spaceで動作する
- [ ] リンクがEnterで遷移する
- [ ] モーダル・ドロップダウンがEscapeで閉じる
- [ ] Escape後にフォーカスがトリガー要素に戻る
- [ ] フォーカストラップがモーダル内で正しく動作する

### スクリーンリーダー

- [ ] 見出しレベルが論理的な階層構造になっている
- [ ] ランドマーク要素（header, nav, main, footer）が配置されている
- [ ] 全ての意味のある画像に適切なaltテキストがある
- [ ] 装飾画像に `alt=""` が設定されている
- [ ] アイコンボタンに `aria-label` がある
- [ ] フォームの全入力欄にラベルが紐付いている
- [ ] エラーメッセージが入力欄に紐付いて読み上げられる
- [ ] 動的コンテンツの変更がライブリージョンで通知される

### 視覚

- [ ] 通常テキストのコントラスト比が 4.5:1 以上
- [ ] 大きなテキストのコントラスト比が 3:1 以上
- [ ] UIコンポーネントのコントラスト比が 3:1 以上
- [ ] 200%ズームでコンテンツが欠けない
- [ ] 200%ズームで横スクロールが不要
- [ ] 情報が色だけに依存していない

### インタラクション

- [ ] モーダル: フォーカス移動、トラップ、Escape、フォーカス復帰
- [ ] ドロップダウン: aria-expanded、矢印キー操作、Escape
- [ ] タブUI: role属性、矢印キー操作、aria-selected
- [ ] フォーム: aria-invalid、エラー紐付け、フォーカス管理
