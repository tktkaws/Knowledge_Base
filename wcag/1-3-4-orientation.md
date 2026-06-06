# 1.3.4 表示の向き（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.3.4「表示の向き」（Orientation）
- レベル **AA**（標準的な適合レベル）
- WCAG **2.1** で追加された達成基準
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.3「適応可能」に属する
- コンテンツの表示・操作を**縦向きまたは横向きのいずれか一方**に限定しない
- **例外**：特定の表示向きが**不可欠（essential）** な場合

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.4 Orientation](https://www.w3.org/TR/WCAG22/#orientation)
> - [Understanding SC 1.3.4 Orientation](https://www.w3.org/WAI/WCAG22/Understanding/orientation.html)

## 達成基準の原文（要約）

- コンテンツは、縦向き（portrait）または横向き（landscape）など**単一の表示向き**に表示・操作を制限してはならない
- **例外**：特定の表示向きが不可欠な場合

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.4](https://www.w3.org/TR/WCAG22/#orientation)

## 表示の向き（orientation）とは

- デバイスの**縦向き（portrait）** と**横向き（landscape）**
- CSS では `orientation` メディアクエリで判定
  - `portrait`：`height` ≥ `width`
  - `landscape`：`width` > `height`
- センサー（加速度計等）に基づき、デバイスが向きを検知して表示を切り替える

### 本基準の対象

- **作者が向きを制限**している場合が問題
- 表示サイズに応じた**レスポンシブレイアウトの変更**は本基準の対象外
- ユーザーがデバイスを回転したとき、**コンテンツが利用可能**であること

> **参照**
> - [Understanding SC 1.3.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/orientation.html#intent)

## なぜ必要か

- 車椅子等に**固定マウント**された端末は、物理的に向きを変えられない
- ロービジョンのユーザーは、横向き表示で**文字を大きく**読むことがある
- 「端末を回転してください」と要求すると、**操作不能**になるユーザーがいる
- 作者側が**両方の向き**をサポートすべき

> **参照**
> - [Understanding SC 1.3.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/orientation.html#benefits)

## 不可欠（essential）な例外

- 特定の向きが**不可欠**な場合のみ、向きの制限が許容される
- 「不可欠」：除去すると情報・機能が**根本的に変わり**、別の方法では同等に達成できない

| 例 | 理由 |
|---|---|
| 小切手の撮影（銀行アプリ） | 用紙が横長。正確な撮影に横向きが必要 |
| ピアノアプリ | 鍵盤の物理的比率を保つため横向きが必要 |
| プロジェクター・テレビ向けスライド | 特定環境で一方向のみ |
| VR コンテンツ | ゴーグル内ディスプレイはユーザー視点に対して固定 |

```html
<!-- 許容されうる例：小切手撮影が主要機能で横向きが不可欠 -->
<!-- 銀行アプリの小切手入金画面（概念例） -->
<div class="check-deposit" aria-label="小切手撮影">
  <p>小切手を枠内に合わせて撮影してください。</p>
  <!-- 横向きでのカメラプレビューが機能上必要 -->
</div>
```

- デザイン上の好み・レイアウト都合だけでは**不可欠**とはみなされない

> **参照**
> - [Understanding SC 1.3.4 — Examples 4, 5](https://www.w3.org/WAI/WCAG22/Understanding/orientation.html#examples)

## デバイス側の向きロックとの関係

- 端末の**システム設定**や**物理スイッチ**による向きロックは、ユーザーの選択
- 作者が向きを制限していなければ、システム設定に従って表示される → **適合**
- 本基準は**作者の責任**で向きを制限しないことを求める
- デバイス側ロックと作者側ロックは別問題

> **参照**
> - [Understanding SC 1.3.4 — Locking a device to an orientation](https://www.w3.org/WAI/WCAG22/Understanding/orientation.html#locking-a-device-to-an-orientation)

## 代表的な失敗例

### F97：向きをロックしている

- 縦向きまたは横向き**のみ**でコンテンツを表示
- `screen.orientation.lock()` で向きを固定
- 一方の向きでコンテンツが横向き・逆さまになる

```javascript
// 悪い例：横向きにロック（F97）
screen.orientation.lock('landscape').catch(() => {});
```

```html
<!-- 悪い例：縦向きのみ対応のニュースアプリ -->
<!-- 横向きにするとコンテンツが90度回転したまま表示される -->
```

### F100：「端末を回転してください」メッセージ（door slam）

- 不要な向きでは**コンテンツを隠し**、回転を促すメッセージだけ表示
- 作者が向きを検知して**再配置せず**、ユーザーに回転を要求している

```html
<!-- 悪い例：横向きではメッセージのみ、縦向きでのみコンテンツ表示（F100） -->
<style>
  @media (orientation: landscape) {
    .rotate-message { display: block; }
    .content { display: none; }
  }
  @media (orientation: portrait) {
    .rotate-message { display: none; }
    .content { display: block; }
  }
</style>

<div class="rotate-message">端末を縦向きに回転してください</div>
<div class="content">
  <!-- 実際のコンテンツ -->
</div>
```

```html
<!-- 良い例：両方の向きでコンテンツを表示 -->
<style>
  .content {
    /* 向きに応じてレイアウト調整は OK */
    padding: 1rem;
  }
  @media (orientation: landscape) {
    .content { max-width: 1200px; margin: 0 auto; }
  }
</style>
<div class="content">
  <!-- 縦・横どちらでも利用可能 -->
</div>
```

> **参照**
> - [Failure F97: Locking the orientation to landscape or portrait view](https://www.w3.org/WAI/WCAG22/Techniques/failures/F97)
> - [Failure F100: Showing a message asking to reorient device](https://www.w3.org/WAI/WCAG22/Techniques/failures/F100)

## 向き制限がある場合の対処（Technique G214）

- 向きの制限が**不可欠**な場合でも、**別の向きで同等のコンテンツにアクセス**できるコントロールを提供
- 制限された向き以外から、代替手段で機能を利用可能にする

```html
<!-- 良い例：横向きが推奨だが、縦向きでも操作できる代替 UI -->
<div class="piano-landscape" hidden aria-hidden="true">
  <!-- 横向き：フル鍵盤 -->
</div>
<div class="piano-portrait">
  <p>横向き表示でフル鍵盤が利用できます。</p>
  <button type="button" id="show-compact-keys">コンパクト鍵盤を表示</button>
  <!-- 縦向き：縮小版またはスクロール可能な鍵盤 -->
</div>
```

> **参照**
> - [Technique G214: Using a control to allow access to content in different orientations](https://www.w3.org/WAI/WCAG22/Techniques/general/G214)

## 実装のポイント

### CSS `orientation` メディアクエリ

- **レイアウト調整**に使うのは問題ない
- **コンテンツの表示/非表示**で向きを強制するのは F100

```css
/* 良い例：向きに応じたレイアウト調整 */
@media (orientation: landscape) {
  .sidebar { display: flex; flex-direction: row; }
}

@media (orientation: portrait) {
  .sidebar { display: flex; flex-direction: column; }
}

/* 悪い例：一方の向きでコンテンツを完全に隠す */
@media (orientation: landscape) {
  main { display: none; }
}
```

### Screen Orientation API

- `screen.orientation.lock()` は**向きを固定**する → 不可欠でない限り F97
- `screen.orientation.unlock()` で解除
- PWA・フルスクリーンアプリで使われがち。使用前に**両方向対応**を検討

```javascript
// 悪い例：ゲーム開始時に横向き固定
async function startGame() {
  await document.documentElement.requestFullscreen();
  await screen.orientation.lock('landscape'); // F97（不可欠でなければ）
}

// 良い例：向き固定せず、両方向でプレイ可能な UI
function startGame() {
  // 縦・横どちらでも操作可能なレイアウト
}
```

### viewport メタタグ

- `<meta name="viewport">` 自体は向き制限ではない
- `user-scalable=no` 等は別基準（1.4.4 等）の問題

```html
<!-- 向き制限ではない（レスポンシブ用） -->
<meta name="viewport" content="width=device-width, initial-scale=1">
```

> **参照**
> - [MDN — Screen Orientation API](https://developer.mozilla.org/en-US/docs/Web/API/Screen_Orientation_API)
> - [MDN — @media orientation](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/orientation)

## レスポンシブデザインとの違い

| 項目 | 1.3.4 表示の向き | レスポンシブデザイン |
|---|---|---|
| 焦点 | **向きの制限**の禁止 | 画面サイズに応じたレイアウト |
| 向き変更時 | **両方**でコンテンツが使える | ブレークポイントで再配置 |
| サイズ変更 | 本基準の対象外 | 1.4.10 リフロー等が関連 |

- ブレークポイントでレイアウトが変わるのは**問題ない**
- 「横向きに回転してください」で縦向きを**拒否**するのが問題

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.10 Reflow](https://www.w3.org/TR/WCAG22/#reflow)

## 1.3.3 感覚的な特徴との関係

- 1.3.3：指示が「横向きに回転して」等の**向きだけ**に依存しない
- 1.3.4：作者が**向きそのもの**を制限しない
- 「端末を横向きにしてください」という指示 + 縦向きで使えない UI → **1.3.3 と 1.3.4 の両方**に問題

> **参照**
> - [wcag/1-3-3-sensory-characteristics.md — 1.3.3 感覚的な特徴](./1-3-3-sensory-characteristics.md)

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.4.4 テキストのサイズ変更（AA） | 200% 拡大。横向きで読みやすくする動機 |
| 1.4.10 リフロー（AA） | 400% 拡大時の1カラム表示 |
| 2.5.4 動きによる起動（A） | デバイスの傾きセンサーでの操作（別要件） |

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.4 Motion Actuation](https://www.w3.org/TR/WCAG22/#motion-actuation)

## テスト・確認方法

### 手動テスト

- **縦向き**と**横向き**の両方でコンテンツが**読める・操作できる**か
- 「端末を回転してください」等の**ブロックメッセージ**が出ないか
- 固定マウントを想定し、**一方の向きだけ**で利用できるか確認
- 向き制限が**不可欠**と主張する場合、本当に代替手段がないか

### 開発者ツール

- Chrome DevTools の**デバイスモード**で縦・横を切り替え
- レスポンシブデザインモードで orientation を変更

```bash
# Screen Orientation API の lock 使用を検索（コードベース）
rg "orientation\.lock|screen\.orientation" --type js --type ts
```

### ACT Rule

- [Orientation of the page is not restricted using CSS transforms](https://www.w3.org/WAI/standards-guidelines/act/rules/b33eff/proposed/)

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.3.4 はコンテンツを**単一の表示向きに制限しない**達成基準（レベル AA、WCAG 2.1 追加）
- 車椅子マウント等、**向きを変えられない**ユーザーを想定
- 「端末を回転してください」でコンテンツを隠す **door slam（F100）** は不適合
- `screen.orientation.lock()` や CSS での表示拒否（**F97**）も不可
- 小切手撮影・ピアノ等、**不可欠**な場合のみ例外
- 向きに応じた**レイアウト調整**は OK。向きによる**アクセス拒否**は NG
- 制限が不可欠な場合は **G214** で別向きからのアクセス手段を提供
