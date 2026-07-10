# 2.1.1 キーボード（A）

## 達成基準の概要

- WCAG 2.2 達成基準 2.1.1「キーボード」（Keyboard）
- レベル **A**（最低限の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.1「キーボード操作可能」に属する
- コンテンツの**すべての機能**が、キーボードインターフェースで操作できること
- 個々のキー入力に**特定のタイミング**を要求しないこと
- 例外：基礎機能が**ユーザーの移動経路（パス）に依存する**入力

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.1 Keyboard](https://www.w3.org/TR/WCAG22/#keyboard)
> - [Understanding SC 2.1.1 Keyboard](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html)

## 達成基準の原文（要約）

- コンテンツのすべての機能は、個々のキー入力に特定のタイミングを要求せず、キーボードインターフェースで操作できること
- 例外：基礎となる機能が、端点だけでなくユーザーの移動経路に依存する入力を必要とする場合
- 注記1：例外は入力手法ではなく、基礎機能に関するもの（手書き入力は経路依存だが、テキスト入力という基礎機能自体は経路依存ではない）
- 注記2：マウスなど他の入力方法を併用することは禁止されない

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.1](https://www.w3.org/TR/WCAG22/#keyboard)

## キーボードインターフェースとは

- ソフトウェアがキー入力を受け取るためのインターフェース
- 物理キーボードがなくても、代替キーボードやキーボードエミュレーターで操作できること
- 例
  - 物理キーボード
  - 画面上キーボード
  - 音声入力ソフト
  - スイッチ入力（スキャン入力）
  - sip-and-puff デバイス
- **MouseKeys**（OS のマウスエミュレーター）は、キーボードインターフェースではない
  - アプリケーションからはマウス操作として見えるため

> **参照**
> - [Understanding SC 2.1.1 — keyboard interface](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html#dfn-keyboard-interface)

## なぜ必要か

- 視覚障害のあるユーザーは、目と手の協調が必要なマウスを使えないことが多い
- ロービジョンのユーザーは、ポインターを追いにくく、キーボード操作の方が使いやすい
- 手の震えなど運動障害のあるユーザーは、マウスよりキーボードの方が操作しやすい
- スクリーンリーダー利用者は、基本的にキーボードで操作する

> **参照**
> - [Understanding SC 2.1.1 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html#intent)
> - [Understanding SC 2.1.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html#benefits)

## 基本方針：セマンティックHTMLを使う（H91, G202）

- `<button>`、`<a href>`、`<input>`、`<select>`、`<textarea>` など、ネイティブ要素はキーボード操作が組み込まれている
- 可能な限りネイティブ要素を使う
- カスタム UI は、同等のキーボード操作を実装する

```html
<!-- 悪い例：div でボタンを模倣 -->
<div class="fake-button" onclick="save()">保存</div>

<!-- 良い例：button 要素 -->
<button type="button" onclick="save()">保存</button>
```

```html
<!-- 良い例：フォームコントロールとリンク -->
<form action="/search" method="get">
  <label for="q">検索</label>
  <input id="q" name="q" type="search">
  <button type="submit">検索</button>
</form>

<nav>
  <a href="/about">会社概要</a>
  <a href="/contact">お問い合わせ</a>
</nav>
```

> **参照**
> - [Technique H91: Using HTML form controls and links](https://www.w3.org/WAI/WCAG22/Techniques/html/H91)
> - [Technique G202: Ensuring keyboard control for all functionality](https://www.w3.org/WAI/WCAG22/Techniques/general/G202)
> - [a11y/aria-first-rule.md — ネイティブHTMLを優先する](../a11y/aria-first-rule.md)

## キーボードで操作できること

- ポインターでできるほとんどの操作は、キーボードでも可能にする
- 例
  - クリック → `Enter` / `Space`
  - リンク遷移 → `Enter`
  - フォーム送信 → `Enter`
  - チェックボックス切替 → `Space`
  - ドロップダウン展開 → `Space` / `Enter` / 矢印キー
  - ドラッグ＆ドロップ → 切り取り／貼り付け、フォーム操作などの代替手段

```html
<!-- 良い例：Enter でフォーム送信可能（送信ボタンがフォーカス不可でも可） -->
<form action="/search" method="get">
  <label for="keyword">キーワード</label>
  <input id="keyword" name="q" type="search">
  <!-- 送信ボタンが pointer のみでも、Enter で送信できれば適合 -->
</form>
```

> **参照**
> - [Understanding SC 2.1.1 — Example 7](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html#examples)

## キーボードイベントの実装（SCR35, SCR2）

- `<button>` や `<a href>` の `onclick` は、キーボードからも発火する
- `onclick` だけを `<div>` に付けるとキーボードから操作できない（F54）
- マウス専用イベントだけに頼らない

```html
<!-- 悪い例：div に onclick のみ -->
<div onclick="openMenu()">メニュー</div>

<!-- 良い例：button に onclick -->
<button type="button" onclick="openMenu()">メニュー</button>
```

```html
<!-- カスタム要素が必要な場合：キーボード操作を追加 -->
<div
  role="button"
  tabindex="0"
  onclick="togglePanel()"
  onkeydown="if (event.key === 'Enter' || event.key === ' ') { event.preventDefault(); togglePanel(); }"
>
  パネルを開く
</div>
```

> **参照**
> - [Technique SCR35: Making actions keyboard accessible by using the onclick event of anchors and buttons](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR35)
> - [Technique SCR2: Using redundant keyboard and mouse event handlers](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR2)
> - [Technique G90: Providing keyboard-triggered event handlers](https://www.w3.org/WAI/WCAG22/Techniques/general/G90)

## ホバー専用の機能を避ける

- マウスホバーだけで表示・操作できる機能は、キーボードユーザーが使えない
- ホバーで出るメニュー・ツールチップは、フォーカスでも同じ操作ができるようにする
- 1.4.13「ホバーまたはフォーカスで表示されるコンテンツ」とも重なる

```html
<!-- 悪い例：ホバーのみでサブメニュー表示 -->
<style>
  .nav-item:hover .submenu { display: block; }
</style>

<!-- 良い例：フォーカスでも開く -->
<button
  type="button"
  aria-expanded="false"
  aria-controls="submenu"
  onclick="toggleSubmenu()"
>
  製品
</button>
<ul id="submenu" hidden>...</ul>
```

> **参照**
> - [wcag/1-4-13-content-on-hover-or-focus.md — 1.4.13 ホバーまたはフォーカスで表示されるコンテンツ](./1-4-13-content-on-hover-or-focus.md)

## ドラッグ＆ドロップの代替手段

- ドラッグ＆ドロップだけで完結する操作は、キーボードユーザーが使えない
- 切り取り／貼り付け、上下ボタン、番号入力などの代替手段を提供する

```html
<!-- 悪い例：ドラッグ＆ドロップのみ -->
<div class="kanban" ondrop="moveCard(event)">...</div>

<!-- 良い例：キーボードでも移動可能 -->
<div class="kanban-card">
  <h3>タスクA</h3>
  <label for="status-a">ステータス</label>
  <select id="status-a">
    <option>未着手</option>
    <option>進行中</option>
    <option>完了</option>
  </select>
</div>
```

> **参照**
> - [Understanding SC 2.1.1 — Example 2](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html#examples)

## 描画・配置のキーボード操作

- 描画プログラムでは、キーボードでオブジェクトの作成・サイズ変更・配置・回転を可能にする
- 点と点を結ぶ操作は、矢印キーで点間を移動し `Space` で接続するなどの代替が可能
- 直線・図形・ウィンドウサイズ変更・位置移動（経路が重要でない場合）はキーボードで実現可能

```text
良い例：
  矢印キーでオブジェクトを移動
  Shift + 矢印キーでサイズ変更
  Enter で確定

悪い例：
  マウスドラッグのみでサイズ変更・移動
```

> **参照**
> - [Understanding SC 2.1.1 — Examples 1, 3](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html#examples)

## 特定のタイミングを要求しない

- 短時間に複数回のキー入力を要求する操作は避ける
- キーを長押ししないと認識されない操作は避ける
- ダブルタップや長押しだけで動く UI は、キーボード代替を用意する

```javascript
// 悪い例：0.5秒以内に3回 Space を押す必要がある
let pressCount = 0;
document.addEventListener("keydown", (event) => {
  if (event.key === " ") {
    pressCount++;
    setTimeout(() => { pressCount = 0; }, 500);
    if (pressCount >= 3) activate();
  }
});

// 良い例：1回の Enter で実行
button.addEventListener("click", activate);
```

## 例外：経路依存の入力

- 基礎機能がユーザーの移動経路に依存する場合は例外
- 例
  - フリーハンド描画
  - 水彩画の筆致（速度・時間で変化する）
  - モデルヘリコプター飛行シミュレータ（リアルタイム操作が本質）
- 手書き文字入力は経路依存の入力手法だが、テキスト入力という基礎機能は経路依存ではないため、OCR やキーボード入力の代替が必要

```text
例外になりうる：
  水彩画アプリの筆の動きそのものを教える機能

例外にならない：
  手書き入力 → キーボード入力や音声入力で代替可能
  ドラッグで位置移動 → 切り取り／貼り付けや数値入力で代替可能
```

> **参照**
> - [Understanding SC 2.1.1 — path dependent input](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html#intent)

## すべての可視コントロールがフォーカス可能である必要はない

- マウスで操作できるすべての見た目のコントロールが、個別にフォーカス可能である必要はない
- 要件は、キーボードユーザーが**同等の操作**を実行できること
- 別のキーボード専用モードを用意しても適合しうるが、ユーザーがその存在に気づける設計が望ましい

```html
<!-- 適合しうる例：送信ボタンは pointer のみだが、Enter で送信できる -->
<form action="/search" method="get">
  <input name="q" type="search" aria-label="検索">
</form>
```

> **参照**
> - [Understanding SC 2.1.1 — visible controls note](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html#intent)

## プラットフォーム慣習との関係

- OS やユーザーエージェントには、キーボード操作の慣習がある
- 例：`Enter` と `Space` の両方でボタンを実行できる
- 慣習から外れても適合要件そのものには違反しないが、学習コストが上がる
- 修飾キーのロック（Sticky Keys 等）が有効でも動作するよう設計する

```html
<!-- 望ましい：標準的なボタン操作 -->
<button type="button">実行</button>
<!-- Enter でも Space でも動作 -->

<!-- 適合はしうるが非推奨：Enter のみ反応するカスタムボタン -->
<div role="button" tabindex="0" onkeydown="if (event.key === 'Enter') action()">
  実行
</div>
```

## tabindex とフォーカス

- `tabindex="0"` で DOM 順にフォーカス可能にする
- `tabindex="-1"` はプログラム的フォーカス用（Tab 順には入らない）
- `tabindex` の正の値は Tab 順を乱すため避ける
- フォーカスを受け取った要素に `outline: none` だけ指定し、代替のフォーカス表示がないと 2.4.7 とも問題

```html
<!-- 悪い例：正の tabindex で順序をずらす -->
<button tabindex="3">後</button>
<button tabindex="1">先</button>

<!-- 良い例：DOM 順に従う -->
<button>先</button>
<button>後</button>
```

> **参照**
> - [a11y/keyboard-navigation-basics.md — Tab キーとフォーカス順序](../a11y/keyboard-navigation-basics.md)
> - [a11y/focus-management.md — フォーカス管理](../a11y/focus-management.md)

## よくある失敗例

### F54：ポインター専用イベントのみ

- `click` / `mousedown` / `touchstart` だけで機能を実装し、キーボードから操作できない

```html
<!-- 悪い例：mousedown のみ -->
<div onmousedown="startDrag()">ドラッグ</div>
```

### F55：フォーカスを受け取ったら除去する

- フォーカスを受け取ると `blur()` するなど、キーボード操作を妨げる

```javascript
// 悪い例
element.addEventListener("focus", () => {
  element.blur();
});
```

### F42：リンクを模倣した非セマンティック要素

- `div` に `onclick` だけ付けてリンクのように使う

```html
<!-- 悪い例 -->
<div onclick="location.href='/about'">会社概要</div>

<!-- 良い例 -->
<a href="/about">会社概要</a>
```

> **参照**
> - [Failure F54: using only pointing-device-specific event handlers](https://www.w3.org/WAI/WCAG22/Techniques/failures/F54)
> - [Failure F55: using script to remove focus when focus is received](https://www.w3.org/WAI/WCAG22/Techniques/failures/F55)
> - [Failure F42: emulating links](https://www.w3.org/WAI/WCAG22/Techniques/failures/F42)

## 他の達成基準との関係

| 達成基準 | 関係 |
|---|---|
| 2.1.2 キーボードトラップなし | キーボードで入ったら出られない状態を禁止 |
| 2.1.3 キーボード（例外なし） | AAA、経路依存の例外も認めない |
| 2.4.7 フォーカスの可視化 | キーボード操作時にフォーカスが見えること |
| 2.4.3 フォーカス順序 | Tab 順が意味のある順序であること |
| 1.4.13 ホバーまたはフォーカスで表示されるコンテンツ | ホバーで出る内容はフォーカスでも出ること |

- 2.1.1 は「キーボードで操作できるか」
- 2.1.2 は「キーボードから抜け出せるか」
- 2.4.7 は「今どこにいるか分かるか」

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.2 No Keyboard Trap](https://www.w3.org/TR/WCAG22/#no-keyboard-trap)
> - [WCAG 2.2 — Success Criterion 2.4.7 Focus Visible](https://www.w3.org/TR/WCAG22/#focus-visible)

## テスト・確認方法

### 手動テスト

- マウスを使わず、キーボードだけで主要な操作を完了できるか確認
- `Tab` / `Shift+Tab` でフォーカス移動
- `Enter` / `Space` でボタン・リンク・フォーム操作
- 矢印キーでメニュー・タブ・ラジオボタン操作
- `Escape` でモーダル・メニューを閉じられるか確認
- ホバー専用の機能がないか確認

```text
チェックリスト：
1. すべてのリンク・ボタン・フォームに Tab で到達できるか
2. Enter / Space で操作できるか
3. ホバーだけで出るメニューがないか
4. ドラッグ＆ドロップに代替手段があるか
5. キーの連打や長押しを要求していないか
```

### 自動テスト

- axe / Lighthouse で `click` イベントのみの要素を検出できる場合がある
- ただし、キーボード操作の実際の可否は手動確認が必要

```bash
# ポインター専用イベントの使用箇所を確認
rg "onmousedown|onmouseup|ontouchstart|ontouchend|pointerdown|pointerup" --glob "*.html" --glob "*.js" --glob "*.tsx" --glob "*.jsx"
```

### ACT Rules

- [Scrollable content can be reached with sequential focus navigation](https://www.w3.org/WAI/standards-guidelines/act/rules/0ssw9k/)
- [Iframe with interactive elements is not excluded from tab-order](https://www.w3.org/WAI/standards-guidelines/act/rules/akn7c5/)

> **参照**
> - [a11y/keyboard-navigation-basics.md — キーボードナビゲーションの基本](../a11y/keyboard-navigation-basics.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)

## まとめ

- 2.1.1 は、コンテンツの**すべての機能**をキーボードで操作できることを求める達成基準（レベル A）
- 基本方針は `<button>`、`<a href>`、フォームコントロールなど**セマンティックHTML**を使うこと
- マウス専用イベント、ホバー専用 UI、ドラッグ＆ドロップのみの操作は不適合になりやすい
- 経路依存の入力（フリーハンド描画など）のみ例外
- MouseKeys はキーボードインターフェースとはみなされない
- 2.1.2（トラップなし）・2.4.7（フォーカス可視化）と併せて確認する
