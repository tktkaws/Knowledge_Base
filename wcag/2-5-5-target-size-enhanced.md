# 2.5.5 ターゲットのサイズ（高度）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.5.5「ターゲットのサイズ（高度）」（Target Size (Enhanced)）
- レベル **AAA**（最高の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.5「入力モダリティ」に属する
- ポインタ入力のターゲットサイズが、少なくとも **44×44 CSS ピクセル**であること
- 例外：同等コントロール / インライン / ユーザーエージェント制御 / 不可欠

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.5 Target Size (Enhanced)](https://www.w3.org/TR/WCAG22/#target-size-enhanced)
> - [Understanding SC 2.5.5 Target Size (Enhanced)](https://www.w3.org/WAI/WCAG22/Understanding/target-size-enhanced.html)

## 達成基準の原文（要約）

- ポインタ入力のターゲットのサイズは、少なくとも 44×44 CSS ピクセルであること
- ただし次の場合は除く
  - **Equivalent**：同じページに、44×44 CSS ピクセル以上の同等リンク／コントロールがある
  - **Inline**：文またはテキストブロック内のターゲット
  - **User Agent Control**：サイズがユーザーエージェントにより決定され、作者が変更していない
  - **Essential**：その提示が、伝えられる情報にとって不可欠

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.5](https://www.w3.org/TR/WCAG22/#target-size-enhanced)

## なぜ必要か

- 手の震えや巧緻運動の制約があると、小さなターゲットを押しにくい
- タッチはマウスより精度が粗く、指が着地点を隠す
- 公共交通などの振動環境でも、小さなボタンは誤タップしやすい
- 片手操作・大きな指・指の一部や関節での操作でも助かる
- ロービジョンのユーザーにも、大きいターゲットは見つけやすい

> **参照**
> - [Understanding SC 2.5.5 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/target-size-enhanced.html#benefits)

## ターゲットとは

- ポインタ操作を受け付ける画面上の領域
- 見た目のアイコンより、実際のクリック／タップ可能範囲が対象
- 重なるターゲットがある場合、重なり領域はサイズ測定に含めない
  （同じ動作／同じページを開く場合を除く）

```text
ターゲットの例：
  ボタンのクリック可能領域
  リンクのヒット領域
  アイコンの周囲に広げた透明なタップ領域

注意：
  見た目は 24px でも、padding で 44×44 のヒット領域があれば適合しうる
```

> **参照**
> - [Understanding SC 2.5.5 — Key Terms: target](https://www.w3.org/WAI/WCAG22/Understanding/target-size-enhanced.html#dfn-target)

## 44×44 CSS ピクセル

- CSS ピクセルは端末ピクセルとは別の、密度に依存しない単位
- 作者は `width` / `height` / `padding` / `min-width` などで確保する
- 最低サイズであり、よく使う・取り消せない・端にある操作は、さらに大きくするとよい

```css
/* 良い例：見た目のアイコンは小さくても、ヒット領域は 44×44 */
.icon-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 44px;
  min-height: 44px;
  padding: 10px;
  border: 0;
  background: transparent;
}

.icon-button img {
  width: 24px;
  height: 24px;
}
```

```css
/* 悪い例：見た目もヒット領域も小さい */
.icon-button-bad {
  width: 24px;
  height: 24px;
  padding: 0;
}
```

> **参照**
> - [Technique C44: Using CSS to ensure targets are at least 44 by 44 CSS pixels](https://www.w3.org/WAI/WCAG22/Techniques/css/C44)

## 例外1：同等コントロール（Equivalent）

- 同じページに、同じ機能を持つ 44×44 以上のコントロールがあれば、他は小さくてもよい
- 例：大きな「メニュー」ボタンがあり、同等の小さなアイコンもある

```html
<!-- 良い例：同等の大きなコントロールがある -->
<a class="menu-large" href="/menu">メニュー</a>
<!-- 44×44 以上 -->

<a class="menu-icon" href="/menu" aria-label="メニュー">
  <img src="/icons/menu.svg" alt="">
</a>
<!-- 小さくても、同等の大きなリンクがある -->
```

## 例外2：インライン（Inline）

- 文やテキストブロック内のリンクは、44×44 を求めない
- リフローするとリンク位置が変わるため、行間を超えるサイズにすると重なりやすい
- 文末の脚注リンクやヘルプアイコンも、文の一部として除外されうる
- **文全体がリンク**で、かつテキストブロック内でない場合は、44×44 が必要

```html
<!-- 例外：段落内のテキストリンク -->
<p>
  詳細は
  <a href="/guide">利用ガイド</a>
  を参照してください。
</p>

<!-- 例外：文末の脚注 -->
<p>
  売上は前年比で増加した<a href="#fn1"><sup>1</sup></a>。
</p>
```

```html
<!-- 例外にならない例：単独の文リンク（ブロック外） -->
<a class="cta" href="/signup">今すぐ登録する</a>
<!-- この場合は 44×44 が必要 -->
```

> **参照**
> - [Understanding SC 2.5.5 — Inline](https://www.w3.org/WAI/WCAG22/Understanding/target-size-enhanced.html#intent)

## 例外3：ユーザーエージェント制御（User Agent Control）

- サイズがブラウザ既定で、作者が CSS 等で変えていない場合は対象外
- 作者がサイズを変更したら、例外は使えなくなる

```text
例外になりうる例：
  未スタイルのネイティブコントロール（作者がサイズを変えていない）

例外にならない例：
  作者が width / height / padding で小さくしたボタン
```

## 例外4：不可欠（Essential）

- 特定のサイズ提示が、情報や機能の本質で、変えられない場合
- 広く解釈しない。単にデザイン上小さいだけでは不可欠ではない

## 2.5.8 との関係

| 達成基準 | レベル | サイズ | 特徴 |
|---|---|---|---|
| 2.5.8 ターゲットのサイズ（最低限） | AA（WCAG 2.2 新規） | **24×24** | 小さければ**間隔**でも適合しうる |
| 2.5.5 ターゲットのサイズ（高度） | AAA | **44×44** | 間隔例外はなく、より厳しい |

- AA 適合を目指すなら、まず 2.5.8（24×24 または十分な間隔）
- 重要操作やタッチ中心の UI では、2.5.5（44×44）を目標にするとよい
- プラットフォーム指針でも、おおよそ 44〜48 CSS px 前後が推奨されることが多い

```text
実務の目安：
  必須に近い：2.5.8（24×24 / 間隔）
  より押しやすく：2.5.5（44×44）
  頻繁・取り消し困難・画面端：44 より大きく
```

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.8 Target Size (Minimum)](https://www.w3.org/TR/WCAG22/#target-size-minimum)
> - [Understanding SC 2.5.8 Target Size (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)

## 実装時の注意点

- 見た目サイズとヒット領域を分けて設計してよい
- `min-width` / `min-height: 44px` が扱いやすい
- アイコンボタンは padding でヒット領域を広げる
- 隣接する小さなターゲット同士が重ならないよう注意する
- ズームで満たしたことにしない（CSS px サイズはズームで変わらない）
- モバイルとデスクトップの両方で確認する

```css
/* 良い例：タッチ向けに十分なサイズ */
.button {
  min-width: 44px;
  min-height: 44px;
  padding: 0.75rem 1rem;
}

.toolbar {
  display: flex;
  gap: 8px; /* 隣接誤タップも減らせる */
}
```

```html
<!-- 良い例：ツールバーの各操作が 44×44 -->
<div class="toolbar" role="toolbar" aria-label="編集">
  <button type="button" class="icon-button" aria-label="太字">B</button>
  <button type="button" class="icon-button" aria-label="斜体">I</button>
  <button type="button" class="icon-button" aria-label="リンク">🔗</button>
</div>
```

## よくある失敗例

- 24×24 のアイコンボタンだけで、ヒット領域も 24×24
- 閉じるボタン（×）が画面端で極端に小さい
- カルーセルの前後矢印が細い
- 同等の大きなコントロールもなく、インラインでもない

```css
/* 失敗例 */
.close {
  width: 16px;
  height: 16px;
  padding: 0;
}
```

```css
/* 改善例 */
.close {
  min-width: 44px;
  min-height: 44px;
  padding: 14px;
}
```

## テスト・確認方法

### 手動テスト

- ポインタで操作するコントロールを洗い出す
- DevTools でヒット領域がおおよそ 44×44 以上か確認する
- 段落内リンクや脚注はインライン例外か切り分ける
- 小さいターゲットに、同等の大きなコントロールがあるか確認する
- タッチ端末でも押しやすいか確認する

```text
チェックリスト：
1. 各ターゲットは 44×44 CSS px 以上か
2. 不足なら Equivalent / Inline / UA / Essential 例外か
3. 見た目だけでなくヒット領域で測っているか
4. 重要操作はさらに大きくできないか
5. 2.5.8（24×24）も併せて満たしているか
```

### 開発者ツールでの確認

```javascript
// Console：フォーカス可能な要素のサイズをざっと確認
[...document.querySelectorAll("a, button, [role='button'], input, select, textarea")]
  .map((el) => {
    const r = el.getBoundingClientRect();
    return {
      name: (el.innerText || el.getAttribute("aria-label") || el.tagName).trim().slice(0, 40),
      w: Math.round(r.width),
      h: Math.round(r.height),
      ok: r.width >= 44 && r.height >= 44,
    };
  })
  .filter((x) => !x.ok)
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.5.5 は、ポインタターゲットを少なくとも **44×44 CSS ピクセル**にする達成基準（レベル AAA）
- タッチや手先の不正確さによる押しづらさ・誤操作を減らす
- インラインリンクや、同等の大きなコントロールがある場合などは例外
- AA の 2.5.8（24×24／間隔）より厳しく、間隔だけでは足りない
- 見た目が小さくても、padding 等でヒット領域を 44×44 に広げれば適合しうる
