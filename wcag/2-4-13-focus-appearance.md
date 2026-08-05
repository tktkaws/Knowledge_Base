# 2.4.13 フォーカスの外観（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.13「フォーカスの外観」（Focus Appearance）
- レベル **AAA**（WCAG 2.2 で新規）
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- キーボードフォーカスインジケーターが見えるとき、その一部が**十分な大きさ**と**状態間のコントラスト変化**を満たすこと
- 2.4.7「見えること」に加え、「どれくらいはっきり見えるか」の下限を定める

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.13 Focus Appearance](https://www.w3.org/TR/WCAG22/#focus-appearance)
> - [Understanding SC 2.4.13 Focus Appearance](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html)

## 達成基準の原文（要約）

- キーボードフォーカスインジケーターが可視であるとき、そのインジケーターの領域が次の**すべて**を満たすこと
  1. 未フォーカス時の部品（またはサブコンポーネント）の外周を、太さ **2 CSS ピクセル**の枠で囲んだ面積以上である
  2. フォーカス時と未フォーカス時の**同じピクセル同士**で、コントラスト比が少なくとも **3:1**
- 例外
  - フォーカスインジケーターがユーザーエージェントにより決定され、作者が調整できない
  - フォーカスインジケーターと、その背景色を作者が変更していない

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.13](https://www.w3.org/TR/WCAG22/#focus-appearance)

## なぜ必要か

- 小さな見た目の変化では、ロービジョンや高齢のユーザーが見逃しやすい
- キーボード・スイッチ・音声入力利用者は、フォーカス位置が操作点になる
- 2.4.7 だけでは「見える」ことの下限が曖昧で、点線1pxなど弱い表示でも通りうる
- はっきりしたサイズとコントラスト変化があると、スキャンしやすい

> **参照**
> - [Understanding SC 2.4.13 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html#benefits)

## 2.4.7・1.4.11 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.7 フォーカスの可視化 | フォーカス表示があるか | 「見えること」（AA） |
| 2.4.13 フォーカスの外観 | サイズと状態間コントラスト | 「はっきり見えること」（AAA） |
| 1.4.11 非テキストのコントラスト | 隣接色とのコントラスト | フォーカス時の隣接コントラスト |

```text
違いの要点：
  2.4.13：同じピクセルの「フォーカス前 → フォーカス後」の変化が 3:1 以上
  1.4.11：フォーカス時のインジケーターと、隣の色のコントラストが十分

両方必要になりやすい：
  状態間の変化（2.4.13）と、隣接コントラスト（1.4.11）
```

```text
片方だけ満たす失敗例：
  薄いグレー枠：状態間は 3:1 だが、隣接コントラストが不足 → 1.4.11 失敗
  黒い部品に暗いグレー枠：隣接は足りるが、状態間変化が不足 → 2.4.13 失敗
```

> **参照**
> - [wcag/2-4-7-focus-visible.md — 2.4.7 フォーカスの可視化](./2-4-7-focus-visible.md)
> - [wcag/1-4-11-non-text-contrast.md — 1.4.11 非テキストのコントラスト](./1-4-11-non-text-contrast.md)

## 要件1：最小面積（2px 外周相当）

- インジケーター全体が「2px の外周枠」である必要はない
- **面積が、2px 厚の外周枠と同じかそれ以上**であればよい
- 最も簡単な方法は、**実線の 2px アウトライン**

```text
90×30px のボタンの例：
  外周 2px 相当の最小面積 ≈ 480 CSS px²
  （計算が複雑なら、もっと太い／大きなインジケーターにする）
```

```css
/* 良い例：2px の実線アウトライン（最も単純） */
button:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

```css
/* 悪い例：内側に沈めた 2px だけでは面積不足になりうる */
button:focus-visible {
  outline: none;
  box-shadow: inset 0 0 0 2px #005fcc; /* 外縁から離れた inset は要増厚 */
}
```

```text
inset の注意：
  部品の外縁に接する border / outline なら 2px で足りやすい
  外縁から内側に離れた inset は、2px では不足し 3px 以上が必要な場合がある
```

> **参照**
> - [Understanding SC 2.4.13 — Minimum area](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html#minimum-area)

## 要件2：状態間コントラスト 3:1

- フォーカス前後で**同じ位置のピクセル**の色変化が 3:1 以上
- 文字コントラスト（1.4.3）や隣接コントラスト（1.4.11）とは測り方が異なる
- インジケーター全体が 3:1 である必要はない
- **3:1 を満たす部分の面積**が、最小面積要件を満たせばよい

```css
/* 良い例：背景白 → 青枠で状態間コントラストが大きい */
a:focus-visible {
  outline: 3px solid #005fcc; /* 白背景との変化が大きい */
  outline-offset: 2px;
}
```

```css
/* 悪い例：黒背景ボタンが濃いグレーになるだけ */
.button {
  background: #000;
}
.button:focus-visible {
  background: #555; /* 黒→#555 は 3:1 未満になりやすい */
  outline: none;
}
```

```text
グラデーションの扱い：
  3:1 未満の変化部分は面積計算から除外する
  残った「十分に変わった部分」だけで最小面積を満たす必要がある
```

> **参照**
> - [Understanding SC 2.4.13 — Change of contrast](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html#change-of-contrast)

## 手法1：作者定義の可視フォーカス（G195）

- ブラウザ既定に頼らず、十分なサイズとコントラストのスタイルを自分で付ける
- `:focus-visible` と組み合わせると、キーボード操作時に出しやすい

```css
/* 良い例：十分太い実線 + オフセット */
:focus-visible {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
}

/* リンク・ボタン・フォームにも明示 */
a:focus-visible,
button:focus-visible,
input:focus-visible,
select:focus-visible,
textarea:focus-visible {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
}
```

> **参照**
> - [Technique G195: Using an author-supplied, visible focus indicator](https://www.w3.org/WAI/WCAG22/Techniques/general/G195)

## 手法2：二色フォーカス（C40）

- 背景色がページ内で様々でも、どちらかの色が必ず識別できるようにする
- 白と黒の二重リングなどが代表例
- 画像・グラデーション背景でも状態間コントラストを確保しやすい

```css
/* 良い例：二色のフォーカスリング */
.button:focus-visible {
  outline: 3px solid #ffffff;
  outline-offset: 2px;
  box-shadow: 0 0 0 6px #000000;
}
```

> **参照**
> - [Technique C40: Creating a two-color focus indicator to ensure sufficient contrast with all components](https://www.w3.org/WAI/WCAG22/Techniques/css/C40)

## 手法3：部品内の強いフォーカス（C41）

- 部品の内側に太い枠や背景変化を付ける方法
- 内側に置く場合は、外縁から離れるほど厚みが必要

```css
/* 良い例：内側でも厚みを確保 */
.button:focus-visible {
  outline: none;
  box-shadow: inset 0 0 0 3px #005fcc;
}
```

> **参照**
> - [Technique C41: Creating a strong focus indicator within the component](https://www.w3.org/WAI/WCAG22/Techniques/css/C41)

## 例外：ユーザーエージェント既定に任せる場合

### 作者が調整できない場合

- HTML の `<select>` など、フォーカス表示を作者が変えられない部品
- この場合、見た目の質に関わらずこの基準の対象外になりうる

### 既定インジケーターと背景を変更していない場合

- ブラウザ既定のフォーカスを残し、その背景色も作者が変えていない
- 負担軽減のための例外だが、既定が弱いブラウザもある
- `body` の背景色や、部品の枠色を変えると例外が使えなくなることがある
- 白（`#FFFFFF`）の指定は、未指定の白と同じ扱いとして例外を壊さない

```text
例外が使えなくなる例：
  ページ背景を青にし、ブラウザ既定の青フォーカスがほとんど見えない
  ボタン枠を既定フォーカスに近い色にして、どれがフォーカスか分からない

実務の推奨：
  例外に頼らず、作者側で十分なフォーカススタイルを付ける
```

> **参照**
> - [Understanding SC 2.4.13 — Exceptions](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html#exceptions)

## 複雑な部品（タブリストなど）

- フォーカスが「全体」「サブ部品」「両方」のどれに付くかで評価対象が変わる
- サブ部品（タブ項目・メニュー項目・グリッドセルなど）にフォーカスがあるなら、そのインジケーターを評価する
- 大きな編集領域（ワードプロセッサ風）は、単一の UI 部品とみなされない場合があり、この基準の対象外になりうる（2.4.7 のキャレットは別途必要）

```text
評価の目安：
  全体にだけフォーカス枠 → 全体のインジケーターを評価
  サブ項目にも枠がある → 通常はサブ項目側を評価
  操作できない見出しへのスキップ先フォーカス → この基準の対象外
```

## よくある失敗

### F78：アウトラインを消して見えなくする

```css
/* 失敗例 */
*:focus {
  outline: none !important;
}
```

### F55：フォーカスを受け取ると同時に外す

```javascript
// 失敗例
button.addEventListener("focus", (event) => {
  event.currentTarget.blur();
});
```

### 面積・コントラスト不足

```css
/* 失敗しやすい例：1px 点線だけ */
a:focus {
  outline: 1px dotted #999;
}
```

> **参照**
> - [Failure F78: styling outlines/borders so the visual focus indicator is not visible](https://www.w3.org/WAI/WCAG22/Techniques/failures/F78)
> - [Failure F55: using script to remove focus when focus is received](https://www.w3.org/WAI/WCAG22/Techniques/failures/F55)

## 実装時の注意点

- まず **2px 以上の実線アウトライン**を付けるのが安全
- `outline-offset` で部品から少し離すと視認しやすい（必須ではない）
- 幅・高さの両方にスケールする手法を選ぶ（レスポンシブで面積不足になりにくい）
- 画像・グラデーション背景では二色フォーカス（C40）が有効
- 1.4.11 の隣接コントラストも同時に確認する
- `outline: none` するなら、必ず十分な代替スタイルを付ける

```css
/* 実務で使いやすい基本形 */
:focus-visible {
  outline: 3px solid CanvasText;
  outline-offset: 2px;
}

/* 強制カラーモードでも残りやすい例 */
@media (forced-colors: active) {
  :focus-visible {
    outline: 3px solid Highlight;
  }
}
```

## テスト・確認方法

### 手動テスト

- Tab で各操作可能要素を辿り、フォーカス表示を確認する
- おおよそ 2px 以上の実線に見えるか確認する
- フォーカス前後で色の変化がはっきり分かるか確認する
- 暗い背景・画像背景・モバイル幅でも同様か確認する
- `outline: none` している部品に代替があるか確認する

```text
チェックリスト：
1. フォーカス表示があるか（2.4.7）
2. 面積は 2px 外周相当以上か
3. 状態間コントラストは 3:1 以上か
4. 隣接コントラストも足りるか（1.4.11）
5. 既定例外に安易に頼っていないか
```

### 開発者ツールでの確認

```bash
# 弱いフォーカスや outline 削除を探す
rg "outline:\\s*(none|0|1px)|:focus\\s*\\{|focus-visible" --glob "*.css" --glob "*.scss" --glob "*.tsx"
```

- コントラストチェッカーで、フォーカス前後の色変化を測る
- 面積が怪しい独自デザインは、2px 外周より明らかに大きくする

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.13 は、フォーカスインジケーターに**最小面積**と**状態間コントラスト 3:1**を求める達成基準（レベル AAA、WCAG 2.2 新規）
- 2.4.7 の「見える」を、サイズとコントラストで具体化する
- 最も簡単な適合は、2px 以上の実線アウトライン（できればそれ以上）
- 1.4.11（隣接コントラスト）とは測り方が違うが、両方を満たす設計が必要
- ブラウザ既定への例外はあるが、実務では作者定義の強いフォーカスが安全
