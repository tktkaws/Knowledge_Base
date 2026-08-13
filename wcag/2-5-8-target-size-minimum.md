# 2.5.8 ターゲットのサイズ（最低限）（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.5.8「ターゲットのサイズ（最低限）」（Target Size (Minimum)）
- レベル **AA**（WCAG 2.2 で新規）
- 原則2「操作可能（Operable）」> ガイドライン 2.5「入力モダリティ」に属する
- ポインタ入力のターゲットサイズが、少なくとも **24×24 CSS ピクセル**であること
- 小さければ、**十分な間隔**があれば適合しうる（Spacing 例外）
- その他の例外：同等コントロール / インライン / ユーザーエージェント制御 / 不可欠

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.8 Target Size (Minimum)](https://www.w3.org/TR/WCAG22/#target-size-minimum)
> - [Understanding SC 2.5.8 Target Size (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)

## 達成基準の原文（要約）

- ポインタ入力のターゲットのサイズは、少なくとも 24×24 CSS ピクセルであること
- ただし次の場合は除く
  - **Spacing**：24×24 未満のターゲットについて、各バウンディングボックスの中心に直径 24 CSS ピクセルの円を置いたとき、その円が他のターゲットや、他の小型ターゲットの円と交差しない
  - **Equivalent**：同じページに、この基準を満たす別コントロールで同じ機能を達成できる
  - **Inline**：文の中にある、または非ターゲットテキストの行送りにサイズが制約される
  - **User Agent Control**：サイズがユーザーエージェントにより決定され、作者が変更していない
  - **Essential**：その提示が情報にとって不可欠、または法令で要求される
- **注記1**：スライダーやカラーピッカーなど、位置で値を選ぶものは **1つのターゲット** とみなす
- **注記2**：インラインの行送りは、テキストの流れに垂直な方向で解釈する（縦書きなら横方向）

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.8](https://www.w3.org/TR/WCAG22/#target-size-minimum)

## なぜ必要か

- 手先の巧緻運動が難しい人は、小さなコントロールが隣り合うと誤操作しやすい
- 手の震え、痙縮、四肢麻痺などでも、隣接ターゲットを押しやすい
- 専用入力装置は、マウスほど精度が出ないことが多い
- タッチ操作でも、隣のボタンを誤タップしにくくなる
- 振動環境や片手操作でも、間隔があれば押し分けやすい

> **参照**
> - [Understanding SC 2.5.8 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html#benefits)

## サイズ要件：24×24 CSS ピクセル

- ターゲット内に、水平・垂直に揃えた **24×24 の正方形が完全に収まる**こと
- 角丸で内側の実効領域が足りないと、見た目が大きくても undersized になりうる
- CSS ピクセルはズームしても変わらない。拡大すれば足りる、とは言えない
- ベストプラクティスは、間隔に頼らずサイズ自体を 24×24 以上にする

```css
/* 良い例：ヒット領域が 24×24 以上 */
.icon-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 24px;
  min-height: 24px;
  padding: 4px;
}

/* より望ましい：AAA の 44×44 を目指す */
.icon-button-enhanced {
  min-width: 44px;
  min-height: 44px;
}
```

```css
/* 悪い例：16×16 で隣と隙間なし */
.icon-button-bad {
  width: 16px;
  height: 16px;
  padding: 0;
  margin: 0;
}
```

> **参照**
> - [Understanding SC 2.5.8 — Size requirement](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html#intent)

## 例外1：間隔（Spacing）

- 2.5.5（AAA）にはない、2.5.8 特有の例外
- 24×24 未満でも、周囲に十分な空きがあれば適合しうる
- 判定：各小型ターゲットのバウンディングボックス中心に、**直径 24 CSS px の円**を置く
- その円が、他のターゲットや、他の小型ターゲットの円と**交差しない**こと

```text
間隔のイメージ：
  20×20 のボタンが並ぶ
  → 中心に直径 24 の円を置く
  → 円同士・隣のターゲットと重ならなければ適合

失敗しやすい例：
  20×20 が隙間ゼロで密着
  16×16 が大きな画像リンクに接している
  高さ 18px のメニュー項目が縦に密着
```

```css
/* 良い例：小さくても gap で間隔を確保（C42） */
.toolbar {
  display: flex;
  gap: 8px; /* 20×20 なら、24px 円が交差しないよう空きを取る */
}

.toolbar button {
  min-width: 20px;
  min-height: 20px;
}

/* より単純：サイズ自体を 24 以上にする */
.toolbar button {
  min-width: 24px;
  min-height: 24px;
}
```

```text
実務の目安：
  サイズ 24×24 以上 → 間隔チェック不要
  サイズ 20×20 → おおむね 4px 以上の空きが必要になりやすい
  サイズ 16×16 → 隣の大きなターゲットとも 4px 以上空ける必要が出やすい
  孤立した小型ボタン（上下左右に隣接ターゲットがない）→ 間隔例外で適合しうる
```

> **参照**
> - [Understanding SC 2.5.8 — Spacing](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html#intent)
> - [Technique C42: Using min-height and min-width on target container to ensure sufficient target spacing](https://www.w3.org/WAI/WCAG22/Techniques/css/C42)

## 例外2：同等コントロール（Equivalent）

- 同じページに、この基準を満たす別コントロールで同じ機能を達成できるなら、小型でもよい
- 例：大きな「メニュー」ボタンがあり、同等の小さなアイコンもある

```html
<!-- 良い例：同等の十分なサイズのコントロールがある -->
<a class="menu-large" href="/menu">メニュー</a>
<a class="menu-icon" href="/menu" aria-label="メニュー">
  <img src="/icons/menu.svg" alt="">
</a>
```

## 例外3：インライン（Inline）

- 文中のリンク、または非ターゲットテキストの行送りにサイズが制約されるターゲットは対象外
- リフローでリンク位置が変わるため、行間を超えるサイズにすると重なりやすい
- 読みやすい行送りを優先するための例外
- 縦書きでは、行送りは横方向として解釈する

```html
<!-- 例外：段落内のテキストリンク -->
<p>
  詳細は
  <a href="/guide">利用ガイド</a>
  を参照してください。
</p>
```

```html
<!-- 例外になりにくい例：ツールバーのアイコン列 -->
<nav class="toolbar">
  <button type="button" aria-label="太字">B</button>
  <button type="button" aria-label="斜体">I</button>
</nav>
```

> **参照**
> - [Understanding SC 2.5.8 — Inline](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html#intent)

## 例外4：ユーザーエージェント制御（User Agent Control）

- ブラウザ既定のサイズで、作者が変更していない場合は対象外
- 例：未スタイルの日付ピッカー、ブラウザのスクロールバー
- 作者がサイズを変えたら例外は使えない
- スクロールバーをカスタムした場合、コンテンツとの間隔も確認する

## 例外5：不可欠（Essential）

- サイズや間隔が、情報の本質または法令上の要求である場合
- 例：地図上のピン位置、密集したデータ可視化、紙の帳票再現が法令で求められるフォーム
- 例外を使う場合でも、可能なら同等の別手段を用意するのが望ましい

```text
例外になりうる例：
  地図のピンが実際の位置に密集している
  法令で紙フォームと同じ小さなチェック欄が必要

例外にならない例：
  デザイン上、アイコンを小さくしたいだけ
  ツールバーを詰め込んだ結果、隣と密着している
```

## 1つのターゲットとみなすもの

- スライダー、グラデーションのカラーピッカー、カーソル位置で編集する領域など
- 内部の位置で値を選ぶものは、全体で **1ターゲット**
- つまみとトラックを別々の 24×24 として測る必要はない

## オーバーレイ中の扱い

- ユーザー操作やスクリプトで一時的に隠れた背面のターゲットは、隠されている間は対象外
- 例：コンボボックスの候補、モーダル、Cookie バナー表示中の背面
- ただし、前面に新しく出たターゲット自体は対象

## 2.5.5 との関係

| 達成基準 | レベル | サイズ | 間隔例外 |
|---|---|---|---|
| 2.5.8 ターゲットのサイズ（最低限） | AA | **24×24** | **あり** |
| 2.5.5 ターゲットのサイズ（高度） | AAA | **44×44** | なし |

- AA では、小さくても間隔があれば通りうる
- それでも押しにくさは残るため、サイズ自体を確保するのが望ましい
- 重要操作は 2.5.5（44×44）を目標にする

```text
2.5.8 では適合しうるが 2.5.5 では不適合の例：
  20×20 のアイコンが 8px 間隔で並ぶ
  孤立した 16×16 の閉じるボタン

両方に適合しやすい例：
  すべての操作が 44×44 以上
```

> **参照**
> - [wcag/2-5-5-target-size-enhanced.md — 2.5.5 ターゲットのサイズ（高度）](./2-5-5-target-size-enhanced.md)

## 実装時の注意点

- まず `min-width` / `min-height: 24px` を付ける
- 小さくせざるを得ない場合は `gap` や `margin` で円が交差しない空きを取る
- 見た目とヒット領域を分けてよい（padding で広げる）
- 角丸が強いと、内側に 24×24 の正方形が収まらないことがある
- ズームで満たしたことにしない
- メニュー項目の高さ、ツールバー、閉じるボタン、画像上の小さな操作が重点対象

```css
/* 良い例：メニュー項目の高さを確保 */
.menu a {
  display: flex;
  align-items: center;
  min-height: 24px;
  padding-block: 0.5rem;
}
```

```css
/* 悪い例：行間だけ小さく、項目が密着 */
.menu a {
  line-height: 1;
  padding: 0;
  height: 18px;
}
```

## よくある失敗例

- 16×16 のアイコンが隙間なく並ぶ
- 高さ 18px のドロップダウン項目が縦に密着
- 大きな画像リンクの上に、16×16 の拡大ボタンが重なる
- カスタムスクロールバーとリンクが隙間なく接する

```css
/* 失敗例：小さい閉じるボタンが画像に密着 */
.thumb {
  position: relative;
}
.zoom {
  position: absolute;
  width: 16px;
  height: 16px;
  inset: 0 auto auto 0;
}
```

```css
/* 改善例：24×24 にするか、大きなターゲットから離す */
.zoom {
  min-width: 24px;
  min-height: 24px;
}
```

## テスト・確認方法

### 手動テスト

- ポインタで操作するコントロールを洗い出す
- 24×24 の正方形がヒット領域に収まるか確認する
- 収まらない場合、中心の直径 24 円が隣と交差しないか確認する
- 段落内リンクはインライン例外か切り分ける
- メニュー、ツールバー、画像上の小さな操作を重点的に見る

```text
チェックリスト：
1. 各ターゲットは 24×24 CSS px 以上か
2. 未満なら、直径 24 の円が他ターゲット／他円と交差しないか
3. それでもだめなら Equivalent / Inline / UA / Essential か
4. 見た目ではなくヒット領域で測っているか
5. 重要操作は 44×44（2.5.5）に近づけられないか
```

### 開発者ツールでの確認

```javascript
// Console：24×24 未満の操作要素を洗い出す
[...document.querySelectorAll("a, button, [role='button'], input, select, textarea")]
  .map((el) => {
    const r = el.getBoundingClientRect();
    return {
      name: (el.innerText || el.getAttribute("aria-label") || el.tagName).trim().slice(0, 40),
      w: Math.round(r.width),
      h: Math.round(r.height),
      ok: r.width >= 24 && r.height >= 24,
    };
  })
  .filter((x) => !x.ok)
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.5.8 は、ポインタターゲットを少なくとも **24×24 CSS ピクセル**にする達成基準（レベル AA、WCAG 2.2 新規）
- 小さければ、直径 24 CSS px の円が隣と交差しない**間隔**でも適合しうる
- 隣接する誤操作を減らすのが目的。サイズ確保の方が間隔より望ましい
- インラインリンク、同等コントロール、UA 既定、地図ピンなどの不可欠提示は例外
- より押しやすくするには、AAA の 2.5.5（44×44）を目指す
