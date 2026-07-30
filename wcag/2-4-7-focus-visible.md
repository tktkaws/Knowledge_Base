# 2.4.7 フォーカスの可視化（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.4.7「フォーカスの可視化」（Focus Visible）
- レベル **AA**
- 原則2「操作可能（Operable）」> ガイドライン 2.4「ナビゲーション可能」に属する
- キーボードで操作できる UI には、キーボードフォーカスインジケーターが**見える操作モード**があること
- 今どの部品にフォーカスがあるかを、視覚的に把握できるようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.7 Focus Visible](https://www.w3.org/TR/WCAG22/#focus-visible)
> - [Understanding SC 2.4.7 Focus Visible](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html)

## 達成基準の原文（要約）

- キーボードで操作可能なユーザーインタフェースには、キーボードフォーカスインジケーターが可視である操作モードがあること

> **参照**
> - [WCAG 2.2 — Success Criterion 2.4.7](https://www.w3.org/TR/WCAG22/#focus-visible)

## なぜ必要か

- 視覚のあるキーボード利用者は、フォーカス位置が見えないとページを操作できない
- 注意・短期記憶・実行機能に制約のある人も、今どこを操作しているか分かりやすくなる
- Tab で移動しても、次に Enter / Space が効く対象が分からないと誤操作が起きる
- フォーカス順（2.4.3）が正しくても、見えなければ使えない

> **参照**
> - [Understanding SC 2.4.7 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html#benefits)

## 「操作モード」とは

- ユーザーエージェントによっては、常にフォーカスを表示するとは限らない
- キーボード使用時だけフォーカスを出す最適化もある
- 作者は、少なくとも**1つの操作モード**でフォーカスが見えるようにする
- 多くのサイトでは操作モードは1つなので、実質「フォーカスが見えること」が要件になる
- 表示されたフォーカスは、時間制限で消えてはならない

```text
適合の考え方：
  キーボードで操作しているとき、フォーカス位置が常に見える
  マウスだけの操作では、ブラウザがフォーカス輪郭を出さない場合もある（:focus-visible）

ベストプラクティス：
  ポインタ利用者にも明示的なフォーカス表示があると助かることがある
```

> **参照**
> - [Understanding SC 2.4.7 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html#intent)

## フォーカスインジケーターの例

- テキストフィールド：キャレット（縦棒）や選択ハイライト
- ボタン・リンク：周囲の枠線・アウトライン・背景色の変化
- チェックボックス・ラジオ：枠やリングの強調

```text
見える例：
  入力欄にキャレットが出る
  ボタンの周りに太い枠が出る
  リンクにアウトラインと背景色が付く

見えない例：
  outline: none だけで代替スタイルがない
  フォーカスと同時にスクリプトで blur する
  枠線色が背景と同色で識別できない
```

> **参照**
> - [Understanding SC 2.4.7 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html#examples)

## 手法1：ユーザーエージェントの既定フォーカスを活かす（G149 / G165）

- ネイティブの `<a>` `<button>` `<input>` などは、ブラウザ既定のフォーカス表示を持つ
- 既定を消さなければ、多くの場合これだけで見える
- 高可視の OS / ブラウザ設定がある場合、既定を残すとその恩恵も受けやすい

```css
/* 悪い例：既定フォーカスを消して代替なし */
a:focus,
button:focus,
input:focus {
  outline: none;
}

/* 良い例：既定を残す、または消さない */
/* outline を上書きしない */
```

> **参照**
> - [Technique G149: Using user interface components that are highlighted by the user agent when they receive focus](https://www.w3.org/WAI/WCAG22/Techniques/general/G149)
> - [Technique G165: Using the default focus indicator for the platform](https://www.w3.org/WAI/WCAG22/Techniques/general/G165)

## 手法2：作者が可視フォーカスを提供する（G195 / C15）

- デザイン上、既定アウトラインを変える場合は、**必ず代替の可視スタイル**を付ける
- `:focus` で枠・背景・下線などを変える
- コントラストが低い装飾だけでは不十分になりやすい

```css
/* 良い例：作者定義の可視フォーカス */
a:focus,
button:focus {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
}

.button:focus {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
  background-color: #e8f1ff;
}
```

```css
/* 悪い例：見た目を整えるためにフォーカスを消す */
.button:focus {
  outline: 0;
  box-shadow: none;
}
```

> **参照**
> - [Technique G195: Using an author-supplied, visible focus indicator](https://www.w3.org/WAI/WCAG22/Techniques/general/G195)
> - [Technique C15: Using CSS to change the presentation of a UI component when it receives focus](https://www.w3.org/WAI/WCAG22/Techniques/css/C15)

## 手法3：`:focus-visible` でキーボード時に表示する（C45）

- マウスクリックでは輪郭を抑え、キーボード操作では明確に出す、という実装に使える
- 「操作モード」の考え方と相性がよい
- ただし、キーボードフォーカス時は必ず見えること

```css
/* 良い例：キーボードフォーカス時に可視 */
button:focus {
  outline: none; /* :focus-visible で代替する前提 */
}

button:focus-visible {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
}
```

```css
/* 注意：:focus-visible 非対応環境向けのフォールバック */
button:focus {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
}

@supports selector(:focus-visible) {
  button:focus {
    outline: none;
  }
  button:focus-visible {
    outline: 3px solid #005fcc;
    outline-offset: 2px;
  }
}
```

> **参照**
> - [Technique C45: Using CSS :focus-visible to provide keyboard focus indication](https://www.w3.org/WAI/WCAG22/Techniques/css/C45)

## 手法4：コントラストを意識した二色フォーカス（C40）

- 背景や部品の色が様々でも識別できるよう、二色のフォーカスリングを使う手法
- 1.4.11（非テキストのコントラスト）とも関連が深い
- 暗い部品・明るい部品のどちらでも輪郭が残るようにする

```css
/* 良い例：内側と外側で二色のフォーカス */
.button:focus-visible {
  outline: 3px solid #ffffff;
  outline-offset: 2px;
  box-shadow: 0 0 0 6px #000000;
}
```

> **参照**
> - [Technique C40: Creating a two-color focus indicator to ensure sufficient contrast with all components](https://www.w3.org/WAI/WCAG22/Techniques/css/C40)
> - [wcag/1-4-11-non-text-contrast.md — 1.4.11 非テキストのコントラスト](./1-4-11-non-text-contrast.md)

## よくある失敗

### F78：アウトラインや枠を消して見えなくする

- `outline: none` / `outline: 0` だけで代替がない
- 枠線色を背景と同色にして、実質非表示にする

```css
/* 失敗例（F78） */
*:focus {
  outline: none !important;
}
```

### F55：フォーカスを受け取ると同時に外す

- `onfocus` で即座に `blur()` する
- キーボード利用者がその部品を操作できなくなる
- 2.1.1 や 3.2.1 にも影響しうる

```javascript
// 失敗例（F55）
button.addEventListener("focus", (event) => {
  event.currentTarget.blur();
});
```

> **参照**
> - [Failure F78: styling outlines/borders so the visual focus indicator is not visible](https://www.w3.org/WAI/WCAG22/Techniques/failures/F78)
> - [Failure F55: using script to remove focus when focus is received](https://www.w3.org/WAI/WCAG22/Techniques/failures/F55)

## 2.4.3・2.4.11・2.4.13 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.4.3 フォーカス順序 | フォーカスの進む順 | 順序 |
| 2.4.7 フォーカスの可視化 | フォーカスが見えるか | 可視性（AA） |
| 2.4.11 フォーカスが隠れない（最低限） | フォーカスが他UIに隠れない | 隠れないこと（AA、WCAG 2.2） |
| 2.4.13 フォーカスの外観 | サイズ・コントラストなど | 見た目の質（AAA、WCAG 2.2） |

- 2.4.7 は「見えること」を求めるが、形・太さ・コントラストの詳細は規定しない
- より具体的な見た目の指針は 2.4.13（AAA）
- 見えるが Sticky ヘッダーに隠れる場合は 2.4.11 の問題

```text
2.4.7 では適合しうるが不十分な例：
  ごく細い点線で、背景との差がほとんどない
  （2.4.13 や 1.4.11 では問題になりうる）
```

> **参照**
> - [wcag/2-4-3-focus-order.md — 2.4.3 フォーカス順序](./2-4-3-focus-order.md)
> - [WCAG 2.2 — Success Criterion 2.4.13 Focus Appearance](https://www.w3.org/TR/WCAG22/#focus-appearance)
> - [a11y/keyboard-navigation-basics.md — キーボード操作の基本](../a11y/keyboard-navigation-basics.md)

## 実装時の注意点

- リセット CSS で `outline: none` を全体に掛けない
- 消すなら、必ず `:focus` または `:focus-visible` で代替する
- カスタムコンポーネント（`div[role="button"]` など）も同様に可視フォーカスが必要
- フォーカス表示は時間で消さない
- 高コントラストモードや OS の設定も確認する
- スキップリンクはフォーカス時に必ず見えるようにする

```css
/* 良い例：カスタム部品にもフォーカススタイル */
[role="button"]:focus-visible,
[tabindex="0"]:focus-visible {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
}
```

```html
<!-- 良い例：スキップリンクはフォーカス時に表示 -->
<a class="skip-link" href="#main">メインコンテンツへスキップ</a>
```

```css
.skip-link {
  position: absolute;
  left: -9999px;
}
.skip-link:focus {
  left: 0;
  outline: 3px solid #f0a500;
  outline-offset: 2px;
}
```

## テスト・確認方法

### 手動テスト

- マウスを使わず Tab / Shift+Tab だけでページを操作する
- 各フォーカス可能要素で、フォーカス位置が一目で分かるか確認する
- `outline: none` している部品に代替スタイルがあるか確認する
- モーダル・メニュー・カスタムウィジェットでも同様に確認する
- フォーカスが数秒で消えないか確認する

```text
チェックリスト：
1. キーボード操作時、フォーカス位置が見えるか
2. リンク・ボタン・フォーム・カスタム部品すべてで見えるか
3. outline を消した場合、代替スタイルがあるか
4. フォーカスを受け取ると同時に blur していないか
5. Sticky ヘッダー等で隠れていないか（2.4.11 も確認）
```

### 開発者ツールでの確認

```bash
# フォーカス非表示や blur の疑いを探す
rg "outline:\\s*(none|0)|\\.blur\\(|:focus\\s*\\{|focus-visible" --glob "*.css" --glob "*.scss" --glob "*.js" --glob "*.tsx"
```

- DevTools で要素を選択し、`:focus-visible` 状態を強制表示して確認できる場合がある

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.4.7 は、キーボード操作時に**フォーカス位置が見える**ことを求める達成基準（レベル AA）
- 既定フォーカスを残すか、作者が可視の代替スタイルを提供する
- `:focus-visible` はキーボード時の表示に便利だが、見えない状態を作ってはならない
- `outline: none` のみ、またはフォーカス時の `blur()` は典型的な失敗
- 見え方の詳細は 2.4.13、隠れないことは 2.4.11、順序は 2.4.3 と併せて設計する
