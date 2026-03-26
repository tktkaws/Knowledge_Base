# スクリーンリーダーの基本的な動作原理

## スクリーンリーダーとは

- 画面上のテキストやUI要素の情報を音声合成（TTS）や点字ディスプレイで出力する支援技術
- 視覚障害のあるユーザーがコンピュータやスマートフォンを操作するために使用
- Webブラウザと連携し、Webページの内容を読み上げたりナビゲーションを支援したりするソフトウェア
- 全盲のユーザーだけでなく、ロービジョン（弱視）や学習障害のあるユーザーも利用

> **参照**
> - [WAI - Introduction to Web Accessibility](https://www.w3.org/WAI/fundamentals/accessibility-intro/)
> - [WebAIM - Screen Reader Survey](https://webaim.org/projects/screenreadersurvey10/)

## 主要なスクリーンリーダー

### VoiceOver

- Apple製品に標準搭載のスクリーンリーダー
- 対応OS：macOS / iOS / iPadOS
- 追加インストール不要で、すぐに利用可能
- macOSでは `Cmd + F5` で起動
- iOS / iPadOSでは「設定 > アクセシビリティ > VoiceOver」で有効化
- Safariとの組み合わせが最も安定した動作

### NVDA (NonVisual Desktop Access)

- オープンソースの無料スクリーンリーダー
- 対応OS：Windows
- NV Accessが開発・提供
- Firefoxとの組み合わせで高い互換性
- 世界的にWindowsユーザーの間で広く使用
- ポータブル版もあり、USBメモリから起動可能

### JAWS (Job Access With Speech)

- Freedom Scientific社が開発する商用スクリーンリーダー
- 対応OS：Windows
- 企業・教育機関での導入実績が多い
- 機能が豊富で、スクリプト言語による高度なカスタマイズが可能
- 有料ライセンス（年間契約または買い切り）

### TalkBack

- Android端末に標準搭載のスクリーンリーダー
- 対応OS：Android
- Googleが開発・提供
- Chromeとの組み合わせが推奨
- ジェスチャー操作でナビゲーション

### Narrator

- Microsoftが開発するスクリーンリーダー
- 対応OS：Windows
- Windows 10以降で大幅に改善
- Microsoft Edgeとの組み合わせが推奨
- `Win + Ctrl + Enter` で起動

> **参照**
> - [WebAIM - Screen Reader User Survey #10 Results](https://webaim.org/projects/screenreadersurvey10/)
> - [NV Access - NVDA](https://www.nvaccess.org/)
> - [Freedom Scientific - JAWS](https://www.freedomscientific.com/products/software/jaws/)
> - [Apple - VoiceOver](https://www.apple.com/accessibility/vision/)

## スクリーンリーダーの動作原理

### アクセシビリティツリーとアクセシビリティAPI

- ブラウザがHTMLを解析し、DOMツリーを構築する
- DOMツリーをもとに、ブラウザがアクセシビリティツリーを生成する
- アクセシビリティツリーは、各要素の**ロール・名前・状態・値**を保持するツリー構造
- OSのアクセシビリティAPIを通じて、スクリーンリーダーがアクセシビリティツリーの情報を取得する

```
HTML → DOM ツリー → アクセシビリティツリー → アクセシビリティ API → スクリーンリーダー
```

### 各OSのアクセシビリティAPI

| OS | アクセシビリティAPI |
|---|---|
| Windows | UI Automation (UIA) / MSAA (IAccessible) |
| macOS / iOS | NSAccessibility / UIAccessibility |
| Linux | ATK / AT-SPI |
| Android | AccessibilityNodeInfo |

### アクセシビリティツリーの確認方法

- Chrome DevTools：「Elements」パネル内の「Accessibility」タブ
- Firefox DevTools：「アクセシビリティ」パネル
- Safari：「開発」メニュー >「Web Inspector」>「Elements」>「Node」タブのAccessibilityセクション

> **参照**
> - [MDN - Accessibility tree](https://developer.mozilla.org/en-US/docs/Glossary/Accessibility_tree)
> - [Chrome DevTools - Accessibility features reference](https://developer.chrome.com/docs/devtools/accessibility/reference/)
> - [WAI - Accessibility API](https://www.w3.org/TR/core-aam-1.2/)

## スクリーンリーダーが読み上げる情報

- スクリーンリーダーはアクセシビリティツリーから以下の4つの情報を取得して読み上げる

### 1. ロール (Role)

- 要素の種類・役割（ボタン、リンク、見出し、テキスト入力欄など）
- HTML要素の暗黙のロール、またはrole属性で指定されたロール

### 2. 名前 (Accessible Name)

- 要素のラベル・名前（ボタンのテキスト、画像のalt属性、aria-labelの値など）
- アクセシブルネーム計算（Accessible Name Computation）のアルゴリズムで決定

### 3. 状態 (State)

- 要素の現在の状態（チェック済み、展開中、無効、選択中など）
- aria-checked, aria-expanded, aria-disabled, aria-selected などで表現

### 4. 値 (Value)

- 要素が持つ値（スライダーの数値、テキスト入力欄のテキストなど）

### 読み上げの例

```html
<button disabled>送信</button>
```

- VoiceOverの読み上げ例：「送信、淡色表示のボタン」
- NVDAの読み上げ例：「送信、ボタン、使用不可」

```html
<input type="checkbox" id="agree" checked>
<label for="agree">利用規約に同意する</label>
```

- VoiceOverの読み上げ例：「チェック済み、利用規約に同意する、チェックボックス」
- NVDAの読み上げ例：「利用規約に同意する、チェックボックス、チェック」

> **参照**
> - [W3C - Accessible Name and Description Computation](https://www.w3.org/TR/accname-1.2/)
> - [MDN - ARIA states and properties](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes)

## ナビゲーションモード

- スクリーンリーダーには複数の操作モードがあり、状況に応じて自動的に、または手動で切り替わる

### ブラウズモード（仮想バッファモード / 仮想カーソルモード）

- Webページのコンテンツを順番に読み進めるためのモード
- 仮想的なカーソル（仮想カーソル）でページ上を移動する
- キーボードの矢印キーで1行ずつ、または1文字ずつ読み進めることが可能
- 見出しジャンプ（Hキー）やランドマークジャンプ（Dキー）などのショートカットが使用可能
- NVDA、JAWSではWebページを開くとデフォルトでこのモード

### フォーカスモード（フォームモード / アプリケーションモード）

- フォーム要素やインタラクティブなウィジェットに入力・操作するためのモード
- キーボード入力がそのまま要素に渡される（スクリーンリーダーのショートカットは無効化）
- テキスト入力欄やセレクトボックスにフォーカスすると自動的に切り替わることが多い
- NVDA：`NVDA + Space` で手動切り替え
- JAWS：`Enter` でフォームモードに入り、`Esc` で抜ける

### VoiceOverの場合

- VoiceOverはブラウズモード / フォーカスモードの明確な区別がない
- 「VoiceOverカーソル」で全ての要素をナビゲーション
- フォーム要素にはVoiceOverカーソルで移動後、操作に入る仕組み

> **参照**
> - [MDN - WAI-ARIA basics](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/WAI-ARIA_basics)
> - [NV Access - NVDA User Guide](https://www.nvaccess.org/files/nvda/documentation/userGuide.html)
> - [WebAIM - Using NVDA to Evaluate Web Accessibility](https://webaim.org/articles/nvda/)

## 読み上げ順序

### DOMの順序に依存する

- スクリーンリーダーはDOMツリーの順序（HTMLソースの記述順）に従ってコンテンツを読み上げる
- CSSによる視覚的な配置変更は、読み上げ順序に影響しない
- **見た目の順序とDOMの順序が異なると、スクリーンリーダーユーザーに混乱を招く**

### CSSの視覚的順序変更が問題になるケース

```html
<!-- 悪い例：CSSのorderで視覚的な順序を変更 -->
<style>
  .container { display: flex; }
  .first  { order: 2; }  /* 視覚的には2番目 */
  .second { order: 1; }  /* 視覚的には1番目 */
  .third  { order: 3; }  /* 視覚的には3番目 */
</style>

<div class="container">
  <div class="first">ステップ1：入力</div>    <!-- DOMでは1番目 → 読み上げも1番目 -->
  <div class="second">ステップ2：確認</div>   <!-- DOMでは2番目 → 読み上げも2番目 -->
  <div class="third">ステップ3：完了</div>     <!-- DOMでは3番目 → 読み上げも3番目 -->
</div>
<!-- 視覚的には「確認→入力→完了」だが、読み上げは「入力→確認→完了」 -->
```

```html
<!-- 良い例：DOMの順序を視覚的な順序と一致させる -->
<div class="container">
  <div>ステップ1：入力</div>
  <div>ステップ2：確認</div>
  <div>ステップ3：完了</div>
</div>
<!-- DOMの順序と視覚的な順序が一致 -->
```

### その他の注意点

- `position: absolute` で要素を視覚的に移動しても、読み上げ順序はDOMの順序のまま
- CSS Gridの `grid-row` / `grid-column` による配置変更もDOMの順序に影響しない
- `tabindex` による正の値の指定はフォーカス順序を狂わせるため非推奨

> **参照**
> - [WCAG 2.2 - 1.3.2 Meaningful Sequence](https://www.w3.org/WAI/WCAG22/Understanding/meaningful-sequence.html)
> - [WCAG 2.2 - 2.4.3 Focus Order](https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html)
> - [CSS Flexbox and the keyboard navigation disconnect](https://tink.uk/flexbox-the-keyboard-navigation-disconnect/)

## スクリーンリーダーユーザーのナビゲーション方法

- スクリーンリーダーユーザーは上から順に全てを読むのではなく、ショートカットでページ構造を把握して目的の情報に素早くたどり着く

### 見出しジャンプ

- 見出し要素（`<h1>` ~ `<h6>`）を順番にジャンプしてページの構造を把握
- NVDA / JAWS：`H` キーで次の見出し、`1` ~ `6` キーで特定レベルの見出しにジャンプ
- ページの構造を適切な見出しレベルで構成することが極めて重要

### ランドマークジャンプ

- ランドマーク要素（`<nav>`, `<main>`, `<header>`, `<footer>`, `<aside>`）間をジャンプ
- NVDA：`D` キーで次のランドマーク
- ページの大まかなセクションに素早く移動可能

### リンク一覧

- ページ内の全リンクを一覧表示し、目的のリンクを探す
- NVDA：`NVDA + F7` で要素リストを表示（リンク・見出し・フォーム要素・ボタン・ランドマーク）
- JAWS：`Insert + F7` でリンク一覧
- 「ここをクリック」のようなリンクテキストだと一覧で内容を判別できない

### フォーム要素一覧

- ページ内の全フォーム要素を一覧表示
- NVDA：`F` キーで次のフォーム要素にジャンプ
- 各フォーム要素にラベルが適切に関連付けられていることが重要

### その他のナビゲーション

- テーブル：`T` キーでテーブルにジャンプ、`Ctrl + Alt + 矢印キー` でセル間を移動
- リスト：`L` キーでリストにジャンプ
- 画像：`G` キーで画像にジャンプ
- ボタン：`B` キーでボタンにジャンプ

> **参照**
> - [WebAIM - Keyboard Shortcuts for NVDA](https://webaim.org/resources/shortcuts/nvda)
> - [WebAIM - Keyboard Shortcuts for JAWS](https://webaim.org/resources/shortcuts/jaws)
> - [Deque University - Screen Reader Keyboard Shortcuts](https://dequeuniversity.com/screenreaders/)

## VoiceOverの基本操作（macOS）

### 起動と終了

| 操作 | ショートカット |
|---|---|
| VoiceOverの起動 / 終了 | `Cmd + F5` |
| VoiceOverの一時停止 / 再開 | `Ctrl` |

### VoiceOverキー（VOキー）

- VoiceOverのショートカットには「VOキー」を使用
- デフォルトでは `Ctrl + Option` がVOキー
- 例：「VO + →」 = `Ctrl + Option + →`

### 基本ナビゲーション

| 操作 | ショートカット |
|---|---|
| 次の要素に移動 | `VO + →` |
| 前の要素に移動 | `VO + ←` |
| 要素の操作に入る（グループに入る） | `VO + Shift + ↓` |
| 要素の操作から出る（グループから出る） | `VO + Shift + ↑` |
| 要素を操作する（クリック相当） | `VO + Space` |
| ページ全体を最初から読み上げ | `VO + A` |
| 読み上げの停止 | `Ctrl` |

### Webナビゲーション

| 操作 | ショートカット |
|---|---|
| ローター（要素種別の切り替え） | `VO + U` |
| 次の見出し | `VO + Cmd + H` |
| 前の見出し | `VO + Cmd + Shift + H` |
| 次のリンク | `VO + Cmd + L` |
| 次のフォームコントロール | `VO + Cmd + J` |
| 次のテーブル | `VO + Cmd + T` |
| 次のリスト | `VO + Cmd + X` |

### ローター機能

- `VO + U` でローターを起動すると、見出し・リンク・フォーム要素・ランドマーク・テーブルなどの一覧を表示
- 左右矢印キーで一覧の種類を切り替え
- 上下矢印キーで一覧内の項目を選択しEnterでジャンプ
- テスト時に非常に便利な機能

> **参照**
> - [Apple - VoiceOver User Guide for Mac](https://support.apple.com/guide/voiceover/welcome/mac)
> - [WebAIM - Using VoiceOver to Evaluate Web Accessibility](https://webaim.org/articles/voiceover/)
> - [Deque University - VoiceOver Keyboard Shortcuts on a Mac](https://dequeuniversity.com/screenreaders/voiceover-keyboard-shortcuts)

## フロントエンドエンジニアが意識すべきポイント

### セマンティックなHTMLを書く

- 適切なHTML要素を使用すれば、スクリーンリーダーが自動的にロール・名前・状態を認識
- `<div>` や `<span>` の多用はアクセシビリティ情報を失う原因

### 見出し構造を適切に設計する

- 見出しレベルを飛ばさない（`<h1>` の次に `<h3>` を使わない）
- 見出しジャンプはスクリーンリーダーユーザーの最も重要なナビゲーション手段

### 全てのインタラクティブ要素にアクセシブルな名前を付ける

- ボタンやリンクには明確なテキストを含める
- アイコンのみのボタンには `aria-label` を指定
- フォーム要素には `<label>` を関連付ける

### DOMの順序と視覚的な順序を一致させる

- CSSの `order`, `position`, Grid/Flexboxによる視覚的な並べ替えに注意
- 読み上げ順序はDOMの順序に従う

### 動的な変更をスクリーンリーダーに通知する

- コンテンツの動的な更新には `aria-live` リージョンを使用
- SPAのページ遷移時にはフォーカス管理とページタイトルの更新が必要

### 非表示の扱いを正しく理解する

- `display: none` / `visibility: hidden`：視覚的にもアクセシビリティツリーからも非表示
- `aria-hidden="true"`：視覚的には見えるが、アクセシビリティツリーから除外
- `.visually-hidden`（スクリーンリーダー専用テキスト）：視覚的には非表示だが、スクリーンリーダーには読み上げられる

> **参照**
> - [W3C - ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
> - [WebAIM - Designing for Screen Reader Compatibility](https://webaim.org/techniques/screenreader/)

## 良い例・悪い例のコード対比

### ボタンの実装

```html
<!-- 悪い例：divでボタンを模倣 →ロールが伝わらず、キーボード操作も不可 -->
<div class="btn" onclick="submit()">送信</div>
<!-- スクリーンリーダーの読み上げ：「送信」（ロールなし、操作不可） -->

<!-- 良い例：button要素を使用 -->
<button type="submit">送信</button>
<!-- スクリーンリーダーの読み上げ：「送信、ボタン」 -->
```

### アイコンボタン

```html
<!-- 悪い例：テキストもaria-labelもないボタン →名前なし -->
<button>
  <svg viewBox="0 0 24 24">
    <path d="M3 18h18v-2H3v2zm0-5h18v-2H3v2zm0-7v2h18V6H3z"/>
  </svg>
</button>
<!-- スクリーンリーダーの読み上げ：「ボタン」（名前がない） -->

<!-- 良い例：aria-labelでアクセシブルな名前を付与 -->
<button aria-label="メニューを開く">
  <svg aria-hidden="true" viewBox="0 0 24 24">
    <path d="M3 18h18v-2H3v2zm0-5h18v-2H3v2zm0-7v2h18V6H3z"/>
  </svg>
</button>
<!-- スクリーンリーダーの読み上げ：「メニューを開く、ボタン」 -->
```

### 画像の代替テキスト

```html
<!-- 悪い例：altが空でも装飾画像でもないのに代替テキストがない -->
<img src="chart-sales-2024.png">
<!-- スクリーンリーダーの読み上げ：「chart-sales-2024.png、画像」（ファイル名が読まれる） -->

<!-- 良い例：内容を説明するaltを付与 -->
<img src="chart-sales-2024.png" alt="2024年の売上推移グラフ。1月から12月にかけて右肩上がりの傾向">
<!-- スクリーンリーダーの読み上げ：「2024年の売上推移グラフ。1月から12月にかけて右肩上がりの傾向、画像」 -->

<!-- 装飾画像の場合：alt=""を明示して読み飛ばす -->
<img src="decorative-border.png" alt="">
```

### リンクのテキスト

```html
<!-- 悪い例：リンクの目的が不明 -->
<p>新機能の詳細は<a href="/features">こちら</a>をご覧ください。</p>
<!-- リンク一覧に「こちら」としか表示されず、どこへのリンクか不明 -->

<!-- 良い例：リンクテキストだけで目的が分かる -->
<p><a href="/features">新機能の詳細</a>をご覧ください。</p>
<!-- リンク一覧に「新機能の詳細」と表示され、目的が明確 -->
```

### フォームのラベル

```html
<!-- 悪い例：ラベルが関連付けられていないフォーム -->
<div>
  メールアドレス
  <input type="email">
</div>
<!-- スクリーンリーダーの読み上げ：「テキスト入力欄」（ラベルなし） -->

<!-- 良い例：labelのfor属性で関連付け -->
<div>
  <label for="email">メールアドレス</label>
  <input type="email" id="email">
</div>
<!-- スクリーンリーダーの読み上げ：「メールアドレス、テキスト入力欄」 -->
```

### 見出し構造

```html
<!-- 悪い例：見出しレベルの飛ばし -->
<h1>サイトタイトル</h1>
<h3>最新ニュース</h3>     <!-- h2を飛ばしている -->
<h5>ニュース記事1</h5>   <!-- h4を飛ばしている -->

<!-- 良い例：見出しレベルを順番に使う -->
<h1>サイトタイトル</h1>
<h2>最新ニュース</h2>
<h3>ニュース記事1</h3>
```

### 非表示テキスト（スクリーンリーダー専用）

```html
<!-- 悪い例：aria-hiddenで装飾テキストを隠し忘れ -->
<button>
  <span class="icon">★</span>
  お気に入りに追加
</button>
<!-- スクリーンリーダーの読み上げ：「★ お気に入りに追加、ボタン」 -->

<!-- 良い例：装飾的な要素はaria-hiddenで隠す -->
<button>
  <span class="icon" aria-hidden="true">★</span>
  お気に入りに追加
</button>
<!-- スクリーンリーダーの読み上げ：「お気に入りに追加、ボタン」 -->
```

> **参照**
> - [WCAG 2.2 - 1.1.1 Non-text Content](https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html)
> - [WCAG 2.2 - 2.4.4 Link Purpose (In Context)](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-in-context.html)
> - [WCAG 2.2 - 1.3.1 Info and Relationships](https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships.html)
> - [W3C - ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)

## まとめ

- スクリーンリーダーはアクセシビリティAPI経由でアクセシビリティツリーの情報（ロール・名前・状態・値）を取得して読み上げる
- ブラウズモードとフォーカスモードの切り替えでページ閲覧とフォーム操作を使い分ける
- 読み上げ順序はDOMの順序に依存し、CSSの視覚的な配置変更は影響しない
- スクリーンリーダーユーザーは見出しジャンプやランドマークジャンプで効率的にナビゲーションする
- セマンティックなHTMLを書くことが、スクリーンリーダー対応の最も基本的かつ重要な対策
