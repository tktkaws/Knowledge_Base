# 2.4.12 フォーカスが隠れない（高度）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.12「フォーカスが隠れない（高度）」（Focus Not Obscured (Enhanced)）
- レベル **AAA**（WCAG 2.2 で新規）
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- キーボードフォーカスを受けた UI 部品が、作者が作ったコンテンツによって**一切隠れない**こと
- 2.4.11（AA）より厳しく、部分的な重なりも許容しない

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.12 Focus Not Obscured (Enhanced)](https://www.w3.org/TR/WCAG22/#focus-not-obscured-enhanced)
> - [Understanding SC 2.4.12 Focus Not Obscured (Enhanced)](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-enhanced.html)

## 達成基準の原文（要約）

- ユーザーインタフェースコンポーネントがキーボードフォーカスを受けたとき、作者が作成したコンテンツによって、その部品の**どの部分も隠れない**こと

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.12](https://www.w3.org/TR/WCAG22/#focus-not-obscured-enhanced)

## 2.4.11 との違い

| 項目 | 2.4.11（AA） | 2.4.12（AAA） |
|---|---|---|
| 許容される隠れ | **完全に**隠れなければよい | **一切**隠れてはならない |
| Sticky が端を少し覆う | 適合しうる | 不適合 |
| 半透明オーバーレイの部分的な重なり | 部品が見えていれば適合しうる | 不適合 |
| Cookie バナーの部分的な被覆 | 完全でなければ適合しうる | 不適合 |

```text
2.4.11 では適合しうるが 2.4.12 では不適合の例：
  Sticky フッターがボタンの下端 10px だけ覆う
  半透明のプロモーション帯がリンクの上半分に重なる
  Cookie バナーがフォーカス中の部品の一部を覆う

両方に適合しやすい例：
  scroll-padding で Sticky とフォーカス部品が完全に分離している
  Cookie バナーがモーダルで、閉じるまで他へ進めない
```

> **参照**
> - [wcag/2-4-11-focus-not-obscured-minimum.md — 2.4.11 フォーカスが隠れない（最低限）](./2-4-11-focus-not-obscured-minimum.md)

## なぜ必要か

- マウスを使えない人は、フォーカス位置が次の操作点になる
- 部品の一部でも隠れていると、何にフォーカスしているか判別しにくくなる
- 拡大表示では、見える領域がさらに狭く、部分的な隠れの影響が大きい
- 注意・短期記憶に制約のある人も、フォーカス位置を見つけやすくなる
- 音声入力・スイッチ・オンスクリーンキーボードなどでも同様に重要

> **参照**
> - [Understanding SC 2.4.12 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-enhanced.html#benefits)

## 半透明・ブラーのオーバーレイも不可

- 主観的には「まだ見える」と感じても、AAA では不適合になりうる
- 半透明・ブラー・減光などでフォーカス部品に重なると、識別が難しくなる
- フォーカスインジケーターにも影響する場合、1.4.11 や 2.4.13 も同時に問題になりうる

```css
/* 悪い例：フォーカス部品に半透明レイヤーが重なる */
.promo-overlay {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 120px;
  background: rgb(0 0 0 / 40%);
  backdrop-filter: blur(4px);
  pointer-events: none;
}
/* Tab で移動したリンクの一部が、この帯の下に入る → 2.4.12 不適合 */
```

```text
2.4.11 との対比：
  AA：半透明でも部品の一部が見えていれば適合しうる
  AAA：重なっている時点で不適合
```

> **参照**
> - [Understanding SC 2.4.12 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-enhanced.html#intent)
> - [wcag/1-4-11-non-text-contrast.md — 1.4.11 非テキストのコントラスト](./1-4-11-non-text-contrast.md)

## 手法：CSS の scroll-padding（C43）

- 2.4.11 と同じ手法だが、AAA では**余白を十分に取り、重なりをゼロにする**必要がある
- Sticky の高さちょうどではなく、フォーカス部品の全体が必ず見える値にする
- レスポンシブで Sticky の高さが変わる場合は、各ブレークポイントで再計算する

```css
/* 良い例：Sticky 分を確保し、フォーカス部品が一切隠れない */
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
  z-index: 10;
}

.cookie-banner {
  position: sticky;
  bottom: 0;
  /* バナー表示中は footer 相当の余白も加算する */
}
```

```css
/* 悪い例：scroll-padding が足りず、下端が少し隠れる */
html {
  scroll-padding-bottom: 40px; /* 実際の Sticky フッターは 72px */
}
```

> **参照**
> - [Technique C43: Using CSS scroll-padding to un-obscure content](https://www.w3.org/WAI/WCAG22/Techniques/css/C43)

## Cookie バナー・通知の扱い

- AA では「完全に隠れなければ」よいが、AAA では**一部でも覆ってはならない**
- 通過しやすい設計
  - バナーを**モーダル**にし、閉じるまで他コントロールへ進めない
  - `scroll-padding` で本文と**完全に**重ならないようにする
  - フォーカスを失ったら閉じる通知にする
  - 通知自体へフォーカスを移し、他部品と重ねない

```html
<!-- 良い例：モーダルにして重なり問題を避ける -->
<dialog id="cookie-banner">
  <h2>Cookie の利用について</h2>
  <p>本サイトでは Cookie を使用します。</p>
  <button type="button" id="accept-cookies">同意する</button>
</dialog>
```

```javascript
const banner = document.getElementById("cookie-banner");
const accept = document.getElementById("accept-cookies");

banner.showModal();
accept.addEventListener("click", () => banner.close());
```

```html
<!-- 悪い例（AAA）：非モーダル Sticky が部品の一部を覆う -->
<div class="cookie-banner sticky-bottom">
  Cookie を使用します。<button type="button">OK</button>
</div>
<!-- 背面のリンク下端がバナーに隠れる → 2.4.12 不適合 -->
```

> **参照**
> - [Understanding SC 2.4.12 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-enhanced.html#examples)

## よくある失敗例

- Sticky フッター／ヘッダーが、フォーカス部品の一部を覆う
- カタログ閲覧中に、広告や追加情報がフォーカス部品の一部に重なる
- `scroll-padding` が Sticky の実寸より小さい
- 半透明のプロモーション帯が、フォーカス中のカードに重なる

```text
失敗の典型：
1. 商品一覧を Tab で辿る
2. 「詳しく見る」リンクにフォーカスが当たる
3. 下部の Sticky キャンペーン帯がリンクの下半分を覆う
4. 2.4.11 では見えれば適合しうるが、2.4.12 では不適合
```

## 評価対象は「部品」自体

- この基準が見るのは、フォーカスを受けた**コンポーネント**が一切隠れていないか
- 外側に付くフォーカスリングは、原則として部品の一部としては数えない
- フォーカスインジケーター自体が完全に見えない場合は、別途 2.4.7 の問題になりうる
- 部品は見えても、インジケーターの見た目が弱い場合は 2.4.13 の問題になりうる

```text
関連チェック：
  2.4.12：部品が一切隠れていないか（AAA）
  2.4.11：部品が完全に隠れていないか（AA）
  2.4.7：フォーカス表示が見えるか
  2.4.13：フォーカス表示のサイズ・コントラストは十分か
  1.4.11：非テキストのコントラストは足りるか
```

> **参照**
> - [wcag/2-4-7-focus-visible.md — 2.4.7 フォーカスの可視化](./2-4-7-focus-visible.md)
> - [WCAG 2.2 — Success Criterion 2.4.13 Focus Appearance](https://www.w3.org/TR/WCAG22/#focus-appearance)

## 2.4.7・2.4.11・2.4.13 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.7 フォーカスの可視化 | フォーカス表示があるか | 見えるスタイル |
| 2.4.11 フォーカスが隠れない（最低限） | 完全に隠れない | 部分的な隠れは許容（AA） |
| 2.4.12 フォーカスが隠れない（高度） | 一切隠れない | 部分的な隠れも不可（AAA） |
| 2.4.13 フォーカスの外観 | サイズ・コントラストなど | 見た目の質 |

- 2.4.12 を満たせば、通常 2.4.11 も満たす
- Sticky デザインでは、余白を「足りる」ではなく「重なりゼロ」まで確保する

## 実装時の注意点

- Sticky / fixed の実寸を測り、`scroll-padding` をそれに合わせる
- バナー表示中は、追加の余白やモーダル化を検討する
- 半透明レイヤーを「まだ見えるから大丈夫」と判断しない
- 広告・関連商品・チャットなど、後から出るレイヤーも対象
- モバイル幅・拡大表示・動的な Sticky 高さでも再確認する

```css
/* 良い例：バナー表示中に余白を切り替える */
html.has-cookie-banner {
  scroll-padding-bottom: calc(var(--footer-height) + var(--banner-height));
}
```

```javascript
// バナー表示に合わせてクラスを付与
document.documentElement.classList.toggle(
  "has-cookie-banner",
  bannerIsVisible
);
```

## テスト・確認方法

### 手動テスト

- Sticky ヘッダー／フッター／バナーがある状態で、Tab だけで全操作可能要素を辿る
- フォーカス部品の**どの部分も**重なっていないか確認する
- 「少しだけ隠れている」箇所を不合格として記録する
- 半透明・ブラーのオーバーレイも含めて確認する
- Cookie バナー表示中・非表示中の両方で確認する

```text
チェックリスト：
1. Sticky / fixed / バナー等の重なりコンテンツがあるか
2. フォーカス部品に 1px でも重なりがないか
3. scroll-padding は Sticky 実寸以上か
4. 半透明オーバーレイが部品に重なっていないか
5. モバイル幅でも同様か
```

### 開発者ツールでの確認

```bash
# Sticky / fixed / scroll-padding / overlay まわりを探す
rg "position:\\s*(sticky|fixed)|scroll-padding|backdrop-filter|z-index" --glob "*.css" --glob "*.scss" --glob "*.tsx"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.12 は、キーボードフォーカスを受けた部品が、作者作成コンテンツで**一切隠れない**ことを求める達成基準（レベル AAA、WCAG 2.2 新規）
- 2.4.11（AA）より厳しく、部分的な重なりや半透明オーバーレイも不可
- Sticky ヘッダー／フッターや Cookie バナーが典型的な失敗要因
- `scroll-padding`（C43）やモーダル化で、重なりをゼロにする
- 2.4.7（可視化）・2.4.11（最低限）・2.4.13（外観）と併せて設計する
