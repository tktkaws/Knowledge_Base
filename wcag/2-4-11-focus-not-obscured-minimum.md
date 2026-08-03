# 2.4.11 フォーカスが隠れない（最低限）（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.11「フォーカスが隠れない（最低限）」（Focus Not Obscured (Minimum)）
- レベル **AA**（WCAG 2.2 で新規）
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- キーボードフォーカスを受けた UI 部品が、作者が作ったコンテンツによって**完全に隠れない**こと
- Sticky ヘッダー・フッターやバナーで、フォーカス位置がビューポートから消えないようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.11 Focus Not Obscured (Minimum)](https://www.w3.org/TR/WCAG22/#focus-not-obscured-minimum)
> - [Understanding SC 2.4.11 Focus Not Obscured (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html)

## 達成基準の原文（要約）

- ユーザーインタフェースコンポーネントがキーボードフォーカスを受けたとき、作者が作成したコンテンツによって**完全には隠れない**こと
- **注記1**：ユーザーが再配置できる UI では、**初期位置**だけを適合判定の対象とする
- **注記2**：ユーザーが開いたコンテンツがフォーカスを隠しても、キーボードフォーカスを進めずに再表示できるなら、作者作成コンテンツによる隠れとはみなさない

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.11](https://www.w3.org/TR/WCAG22/#focus-not-obscured-minimum)

## なぜ必要か

- マウスを使えない人は、キーボードフォーカス位置が次の操作点になる
- フォーカスした部品が完全に見えないと、進め方が分からず、固まったように感じることがある
- 音声入力・スイッチ・オンスクリーンキーボードなども、キーボードインタフェース経由で操作する
- 拡大表示で画面の一部しか見えない人にも、フォーカス位置の把握が重要
- 注意・短期記憶に制約のある人も、今どこを操作しているか分かりやすくなる

> **参照**
> - [Understanding SC 2.4.11 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html#benefits)

## AA では「完全に隠れなければ」よい

- 2.4.11（AA）は、フォーカスを受けた部品が**一部分でも見えていれば**適合しうる
- 複雑なレスポンシブデザインを踏まえ、部分的な重なりは許容している
- それでも、隠れる面積が大きいほど分かりにくくなる
- **まったく隠れない**設計が望ましく、それは 2.4.12（AAA）の対象

```text
2.4.11（AA）で適合しうる例：
  Sticky フッターがボタンの下端を少し覆うが、大部分は見える

2.4.11（AA）で不適合の例：
  Sticky ヘッダーが、フォーカス中のリンクを完全に覆い隠す

2.4.12（AAA）を目指す例：
  フォーカス部品が一切重ならないよう、余白や scroll-padding を確保
```

> **参照**
> - [Understanding SC 2.4.11 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html#intent)
> - [WCAG 2.2 — Success Criterion 2.4.12 Focus Not Obscured (Enhanced)](https://www.w3.org/TR/WCAG22/#focus-not-obscured-enhanced)

## 隠れやすい典型パターン

- Sticky / fixed のヘッダー・フッター
- Cookie 同意バナー、通知バー
- 非モーダルダイアログ、チャットウィンドウ
- 半透明オーバーレイやブラー付きレイヤー

```text
問題が起きやすい流れ：
1. ページ下部のリンクへ Tab で移動する
2. Sticky フッターがリンク全体を覆う
3. フォーカスは当たっているが、画面上では見えない
4. ユーザーは次の操作が分からなくなる
```

## 手法：CSS の scroll-padding（C43）

- `scroll-padding-top` / `scroll-padding-bottom` などで、Sticky 領域の高さを確保する
- フォーカス移動時にスクロールしても、部品がヘッダー／フッターの下に潜り込まない
- Sticky の高さに合わせた余白をビューポート側に持たせる

```css
/* 良い例：Sticky ヘッダー／フッター分の余白を確保 */
:root {
  --header-height: 64px;
  --footer-height: 72px;
}

html {
  scroll-padding-top: var(--header-height);
  scroll-padding-bottom: var(--footer-height);
}

.site-header {
  position: sticky;
  top: 0;
  height: var(--header-height);
}

.site-footer {
  position: sticky;
  bottom: 0;
  height: var(--footer-height);
}
```

```css
/* 悪い例：Sticky だけで、フォーカス時の隠れを考慮していない */
.site-header {
  position: fixed;
  top: 0;
  height: 80px;
  z-index: 1000;
}
/* ページ先頭付近のリンクがヘッダーに完全に隠れることがある */
```

> **参照**
> - [Technique C43: Using CSS scroll-padding to un-obscure content](https://www.w3.org/WAI/WCAG22/Techniques/css/C43)

## Cookie バナー・通知の扱い

- フォーカス中の部品を**完全に隠す** Sticky 通知は不適合になりやすい（F110）
- 通過しやすい設計の例
  - **モーダル**にして、閉じるまで他コントロールへ進めない
  - `scroll-padding` などで本文と重ならないようにする
  - フォーカスを失ったら閉じる通知にする
  - 通知自体へフォーカスを移し、少なくとも一部が見えるようにする

```html
<!-- 良い例：同意バナーをモーダルにする -->
<dialog id="cookie-banner" open>
  <h2>Cookie の利用について</h2>
  <p>本サイトでは Cookie を使用します。</p>
  <button type="button" id="accept-cookies">同意する</button>
</dialog>
```

```javascript
const banner = document.getElementById("cookie-banner");
const accept = document.getElementById("accept-cookies");

banner.showModal(); // 他コントロールへフォーカスが抜けない
accept.addEventListener("click", () => banner.close());
```

```html
<!-- 悪い例：ページ上に被さり、下のリンクを完全に隠す Sticky バナー -->
<div class="cookie-banner sticky-bottom">
  Cookie を使用します。<button>OK</button>
</div>
<!-- Tab で移動したリンクがバナーの下に完全に隠れる -->
```

> **参照**
> - [Understanding SC 2.4.11 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html#examples)
> - [Failure F110: sticky footer or header completely hiding focused elements](https://www.w3.org/WAI/WCAG22/Techniques/failures/F110)

## ユーザーが開いたコンテンツ（注記2）

- メニューやツールチップなど、操作直後だけ開く一時的な開示は、通常この基準の問題になりにくい
- チャットや固定のフィードバックパネルなど、**ユーザーが閉じるまで残る**開示は隠れの原因になりうる
- 残る開示でも、次のいずれかなら適合しうる
  - 開いた内容が既存コンテンツを押し下げる／リフローする
  - 開いた内容にフォーカスを閉じ込め、閉じるまで外へ出ない
  - 余白側へ展開し、他の操作可能要素と重ならない
  - Esc やスクロールなど、**フォーカスを進めずに**隠れた部品を再表示できる

```text
適合しやすい例：
  Esc でチャットを閉じると、隠れていたリンクが見える
  Space / ↓ でスクロールすると、Sticky パネルの下のリンクが見える
  アコーディオンが下へ押し下げ、重ならない

不適合になりやすい例：
  非モーダルのオーバーレイが残り、下のフォーカス部品を完全に隠す
  閉じる手段がなく、フォーカスを進めても見え続けない
```

> **参照**
> - [Understanding SC 2.4.11 — User-opened content](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html#intent)

## ユーザーが動かせるパネル（注記1）

- オーサリングツールのツールバーなど、ユーザーが再配置できる UI がある
- 適合判定は、**初期位置**でフォーカスを隠さないかに限定される
- ユーザーが後から重ねて隠しても、その配置までは作者責任に含めない
- それでも、初期状態で隠れない設計が望ましい

## モーダルダイアログ

- 適切に実装したモーダルは、通常この基準に適合する
- 開いた時点でダイアログ側へフォーカスが移り、見える範囲で操作する
- 外側を操作できないため、背後の部品が隠れても問題にならない
- フォーカスを奪わず、外側へ抜けられる「ダイアログ風」オーバーレイは不適合リスクが高い

```javascript
// 良い例：dialog でフォーカスを内側に保つ
dialog.showModal();

// 悪い例：オーバーレイを出すがフォーカスは背面のまま
overlay.hidden = false;
// Tab で背面のリンクへ移り、オーバーレイに完全に隠れる
```

## 評価対象は「部品」自体

- この基準が見るのは、フォーカスを受けた**コンポーネント**が完全に隠れていないか
- 外側に付くフォーカスリングは、原則として部品の一部としては数えない
- フォーカスインジケーター自体が完全に見えない場合は、別途 2.4.7 の問題になりうる
- 半透明オーバーレイで部品が部分的に見えても、コントラスト不足なら 1.4.11 の問題になりうる

```text
関連チェック：
  2.4.11：部品が完全に隠れていないか
  2.4.7：フォーカス表示が見えるか
  1.4.11：フォーカス表示や UI のコントラストは足りるか
  2.4.12：部品が一切隠れていないか（AAA）
```

> **参照**
> - [wcag/2-4-7-focus-visible.md — 2.4.7 フォーカスの可視化](./2-4-7-focus-visible.md)
> - [wcag/1-4-11-non-text-contrast.md — 1.4.11 非テキストのコントラスト](./1-4-11-non-text-contrast.md)

## 2.4.7・2.4.12 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.7 フォーカスの可視化 | フォーカス表示があるか | 見えるスタイル |
| 2.4.11 フォーカスが隠れない（最低限） | 部品が完全に隠れない | 部分的な隠れは許容（AA） |
| 2.4.12 フォーカスが隠れない（高度） | 部品が一切隠れない | より厳しい（AAA） |

- 2.4.7 は「インジケーターがあるか」、2.4.11 は「部品がレイヤーに隠れないか」
- Sticky デザインでは、両方を同時に満たす必要がある

## 実装時の注意点

- `position: sticky` / `fixed` を使う領域の高さを把握する
- `scroll-padding` や本文側の `padding` で、フォーカス時の隠れを防ぐ
- Cookie バナーはモーダル化、または本文と重ならない配置にする
- チャットやスライドアウトナビは、リフロー・フォーカストラップ・Esc 閉じを検討する
- レスポンシブの各ブレークポイントで、隠れが起きないか確認する

```css
/* 良い例：本文側にも Sticky 分の余白を持たせる */
main {
  padding-top: var(--header-height);
  padding-bottom: var(--footer-height);
}
```

## テスト・確認方法

### 手動テスト

- Sticky ヘッダー／フッター／バナーがあるページで、Tab だけで全操作可能要素を辿る
- フォーカスを受けた部品が、完全に隠れていないか確認する
- Cookie バナー表示中も同様に確認する
- チャットや固定パネルを開いた状態で、背面へフォーカスが移ったとき見え方を確認する
- Esc やスクロールで、フォーカスを進めずに再表示できるかも確認する

```text
チェックリスト：
1. Sticky / fixed / バナー等の重なりコンテンツがあるか
2. フォーカス部品が完全に隠れる箇所がないか
3. scroll-padding や余白で回避しているか
4. モーダル化や Esc 閉じなど、代替手段があるか
5. モバイル幅でも同様に隠れないか
```

### 開発者ツールでの確認

```bash
# Sticky / fixed / scroll-padding まわりを探す
rg "position:\\s*(sticky|fixed)|scroll-padding|z-index" --glob "*.css" --glob "*.scss" --glob "*.tsx"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.11 は、キーボードフォーカスを受けた部品が、作者作成コンテンツで**完全に隠れない**ことを求める達成基準（レベル AA、WCAG 2.2 新規）
- 部分的な隠れは AA では許容されうるが、隠れない設計（2.4.12）が望ましい
- Sticky ヘッダー／フッターや Cookie バナーが典型的な失敗要因（F110）
- `scroll-padding`（C43）、モーダル化、リフロー、Esc 閉じなどが主な対策
- 2.4.7（可視化）や 1.4.11（コントラスト）と併せて確認する
