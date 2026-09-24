# フォーカスインジケーターのカスタマイズ — :focus-visibleの活用

## フォーカスインジケーターとは

- キーボードフォーカスが今どこにあるかを示す視覚的な手がかり
- ブラウザ標準では多くの場合 `outline`（フォーカスリング）として表示される
- キーボード利用者・ロービジョン利用者にとって、操作位置を把握する必須のUI
- デザイン都合で消すだけではWCAG違反になりやすい

> 参照: [Understanding SC 2.4.7: Focus Visible](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html)

## 関係するWCAG達成基準

| 達成基準 | レベル | 要点 |
|---|---|---|
| 2.4.7 Focus Visible | AA | キーボードフォーカスが見えるモードがある |
| 1.4.11 Non-text Contrast | AA | フォーカス表示と隣接色のコントラストが十分（著者スタイル時） |
| 2.4.11 Focus Not Obscured (Minimum) | AA | フォーカス対象が著者作コンテンツで完全に隠れない |
| 2.4.13 Focus Appearance | AAA | インジケーターの面積・コントラストの最低基準 |

- 最低ラインは「消さない・見える」（2.4.7）
- カスタムするならコントラストと大きさも意識する（1.4.11 / 2.4.13）
- スティッキーヘッダーやモーダルでフォーカスが隠れていないか確認する（2.4.11）

> 参照: [Understanding SC 2.4.13: Focus Appearance](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html)

## :focus と :focus-visible の違い

| 疑似クラス | いつマッチするか |
|---|---|
| `:focus` | 要素がフォーカスを持っているとき常に |
| `:focus-visible` | フォーカスがあり、**ブラウザが「見える表示が必要」と判断したとき** |

- キーボード操作では、通常 `:focus-visible` もマッチする
- マウスクリックでは、ボタンなどではリングを出さないことが多い
- テキスト入力など、入力が必要な要素ではポインタ操作でも表示されやすい
- カスタムスタイルは基本的に `:focus-visible` に書く

> 参照: [MDN — :focus-visible](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors/:focus-visible)、[WCAG Technique C45](https://www.w3.org/WAI/WCAG22/Techniques/css/C45)

## やってはいけないこと

```css
/* 悪い例：フォーカス表示を消して代替を用意しない */
*:focus {
  outline: none;
}

button:focus {
  outline: 0;
}
```

- `outline: none` 自体が禁止なのではなく、**代わりの見える表示がない**のが問題
- リセットCSSやデザインシステムの初期スタイルに紛れ込みやすい

## 基本的なカスタム例

```css
/* キーボードフォーカス時だけ強いリングを出す */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* マウス操作時にブラウザ既定のアウトラインが残る場合の抑え込み */
:focus:not(:focus-visible) {
  outline: none;
}
```

```css
/* コンポーネント単位の例 */
.button:focus-visible {
  outline: 3px solid #0b57d0;
  outline-offset: 3px;
}

.link:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 3px;
  border-radius: 2px;
}
```

### デザインに馴染ませるポイント

- ブランドカラーを使ってよいが、背景とのコントラストを確保する
- `outline-offset` で要素本体と少し離すと視認しやすい
- `border-radius` を要素に合わせると見た目が揃う
- `outline` はレイアウトを押し広げないので、フォーカス用に向く

## コントラストと大きさの目安

### 実務で守りやすい形

```css
.interactive:focus-visible {
  /* 2px以上の実線アウトラインが分かりやすい */
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

- フォーカス時と非フォーカス時で、同じピクセルの色変化がおおむね **3:1** 以上あるとよい
- 2.4.13（AAA）では、おおむね **2CSS px厚の外周分以上**の面積が目安
- 細い点線・透明度の高い色・要素と同系色だけだと見えにくい

### 二色リング（背景が不定のとき）

```css
/* 明るい背景でも暗い背景でも見えやすくする */
.button:focus-visible {
  outline: 2px solid #fff;
  box-shadow: 0 0 0 4px #0b57d0;
}
```

- 外側と内側で明暗を分けると、写真の上や色付きボタンでも判別しやすい

> 参照: [WCAG Technique C40](https://www.w3.org/WAI/WCAG22/Techniques/css/C40)

## outline 以外の表現

```css
/* 背景色を変える */
.nav-link:focus-visible {
  outline: none;
  background-color: #e8f0fe;
  box-shadow: inset 0 0 0 2px #0b57d0;
}

/* 下線を強調する */
.text-link:focus-visible {
  outline: none;
  text-decoration: underline;
  text-decoration-thickness: 3px;
  text-underline-offset: 3px;
}
```

- `outline: none` するなら、必ず別の見えるスタイルをセットで書く
- 色の変化だけに頼らず、太さ・位置・形状の変化も併用する

## フォーカスが隠れる問題（2.4.11）

```css
/* 悪い例：固定ヘッダーがフォーカス中のリンクを完全に隠す */
.site-header {
  position: sticky;
  top: 0;
  z-index: 10;
}

html {
  scroll-padding-top: 0; /* アンカー先が見えない */
}
```

```css
/* 良い例：固定ヘッダー分の余白を確保 */
html {
  scroll-padding-top: 4.5rem;
}

:target {
  scroll-margin-top: 4.5rem;
}
```

- スティッキーヘッダー、クッキーバナー、チャットウィジェットがフォーカス対象を覆わないか確認する
- 少なくとも一部が見えていれば 2.4.11（AA）の最低要件側

## 古いブラウザへのフォールバック

```css
/* :focus-visible 非対応環境向け */
@supports not selector(:focus-visible) {
  .button:focus {
    outline: 2px solid #005fcc;
    outline-offset: 2px;
  }
}

@supports selector(:focus-visible) {
  .button:focus-visible {
    outline: 2px solid #005fcc;
    outline-offset: 2px;
  }

  .button:focus:not(:focus-visible) {
    outline: none;
  }
}
```

- 現行の主要ブラウザは `:focus-visible` 対応済み
- レガシー対応が必要なら `@supports` で分岐する
- 何も書かなくても、ブラウザ既定のアウトラインが残る方が、消すよりましな場合がある

## React / デザインシステムでの扱い

```css
/* グローバルな土台 */
:where(a, button, input, select, textarea, [tabindex]:not([tabindex="-1"])):focus-visible {
  outline: 2px solid var(--focus-color, #005fcc);
  outline-offset: 2px;
}
```

```jsx
// className で上書きする場合も、focus-visible 用クラスを用意する
<button className="btn btn--primary">保存</button>
```

```css
.btn:focus-visible {
  outline: 2px solid var(--color-focus);
  outline-offset: 2px;
}

/* 間違い: コンポーネントCSSで outline を消したまま放置 */
.btn:focus {
  outline: none;
}
```

- デザインシステムのリセットで `outline: none` を入れるなら、同時に `:focus-visible` の定義を必須化する
- 高コントラストモードや強制カラーでも見え方を確認する

```css
@media (forced-colors: active) {
  .button:focus-visible {
    outline: 2px solid Highlight;
    outline-offset: 2px;
  }
}
```

## よくある間違い

### 1. outline を消して終わり

```css
/* 間違い */
* {
  outline: none !important;
}
```

### 2. :focus に強いスタイルを付けてマウス操作でも常にリングが出る

- 要件としては満たしやすいが、意図せずUIが騒がしくなる
- キーボード中心に見せたいなら `:focus-visible` を使う

### 3. コントラスト不足の薄い色

```css
/* 弱い例 */
button:focus-visible {
  outline: 1px solid #ccc;
}
```

### 4. フォーカススタイルが border の変化だけで、サイズがほぼ変わらない

- 1pxの色変化だけでは気づきにくい
- 太さや offset を足す

### 5. モーダルやドロワーの下にフォーカスが見える / 隠れる

- フォーカストラップと合わせて、見える位置にフォーカスが移っているか確認する

## 実装チェックリスト

- [ ] `outline: none` だけのスタイルがない
- [ ] キーボード操作で全インタラクティブ要素のフォーカスが見える
- [ ] カスタム表示は主に `:focus-visible` で定義している
- [ ] フォーカス色が背景・コンポーネント色と十分に差がある
- [ ] 2px以上のアウトラインなど、十分な大きさがある
- [ ] スティッキーUIでフォーカス対象が完全に隠れていない
- [ ] デザインシステムのリセットに代替スタイルが付いている
- [ ] 強制カラー / 高コントラストでも判別できる

## まとめ

- フォーカスインジケーターはキーボード操作の現在地表示であり、消すだけではいけない
- カスタムは `:focus-visible` を使い、キーボード利用者には強く、ポインタ操作では過剰にならないようにする
- 見えること（2.4.7）に加え、コントラスト・大きさ・隠れないことも意識する
- `outline: none` するなら、必ず同等以上に分かりやすい代替スタイルをセットで用意する
