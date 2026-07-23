# 2.3.3 インタラクションによるアニメーション（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.3.3「インタラクションによるアニメーション」（Animation from Interactions）
- レベル **AAA**（最高の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.3「発作と身体的反応」に属する
- ユーザー操作によって始まる**動きのアニメーション**を、無効にできること
- 例外：機能や伝えられる情報にとって**不可欠**なアニメーション
- 前庭障害（めまい・吐き気など）への配慮が目的

> **参照**
> - [WCAG 2.2 — Success Criterion 2.3.3 Animation from Interactions](https://www.w3.org/TR/WCAG22/#animation-from-interactions)
> - [Understanding SC 2.3.3 Animation from Interactions](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html)

## 達成基準の原文（要約）

- インタラクションによってトリガーされる動きのアニメーションは、無効にできること
- ただし、そのアニメーションが機能または伝えられる情報にとって不可欠な場合は除く

> **参照**
> - [WCAG 2.2 — Success Criterion 2.3.3](https://www.w3.org/TR/WCAG22/#animation-from-interactions)

## なぜ必要か

- 一部のユーザーは、動きのあるコンテンツで注意散漫や吐き気を起こす
- 前庭（内耳）障害のある人は、スクロールや操作に連動する余計な動きで症状が出る
- 症状の例：めまい、吐き気、片頭痛、回復のための安静が必要になること
- パララックススクロールなど、装飾的な動きがトリガーになりやすい
- 「動きを止めたい」という設定やコントロールがないと、ページを使えなくなる

> **参照**
> - [Understanding SC 2.3.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html#benefits)

## 動きのアニメーション（motion animation）とは

- 状態と状態のあいだに段階を足し、**動きの錯覚**や滑らかな遷移を作るもの
- 例：スライドイン、ズーム、パララックス、ページめくり、バウンス
- ユーザーの操作（スクロール・クリック・ホバー・フォーカスなど）をきっかけに始まるものが対象
- 色の変化だけ・透明度の変化だけなど、動きの錯覚を伴わないものは対象外になりやすい

```text
対象の例：
  スクロールに連動して横に動く装飾要素
  クリックでページがめくれる遷移
  ホバーで大きくバウンスするカード
  パララックス背景

対象になりにくい例：
  色だけが変わるホバー状態
  動きを伴わないフェードのみ（実装・解釈による）
  スクロール自体による、新しいコンテンツの表示（不可欠な動き）
```

> **参照**
> - [Understanding SC 2.3.3 — Key Terms: motion animation](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html#dfn-motion-animation)

## 2.2.2 との違い

| 達成基準 | 焦点 | トリガー |
|---|---|---|
| 2.2.2 一時停止・停止・非表示 | 自動で始まる動き・点滅・更新 | ページ側が自動開始 |
| 2.3.3 インタラクションによるアニメーション | 操作で始まる動きのアニメーション | ユーザーの意図的な操作 |

- 2.2.2 は「自動再生・自動更新」への制御
- 2.3.3 は「ユーザー操作がきっかけの非本質的な動き」への無効化
- 同じアニメーションが、両方の基準で不適合になる場合もある

```text
2.2.2 の例：
  ページ読み込み後、自動で動き続けるカルーセル

2.3.3 の例：
  ユーザーがスクロールすると背景がパララックスで動く
  ユーザーがリンクを押すとページめくりアニメが再生される
```

> **参照**
> - [wcag/2-2-2-pause-stop-hide.md — 2.2.2 一時停止・停止・非表示](./2-2-2-pause-stop-hide.md)
> - [Understanding SC 2.3.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html#intent)

## スクロールと不可欠な動き

- スクロールで新しいコンテンツがビューポートに入る動きは**不可欠**
- ユーザーが制御する本質的なスクロールは許容される
- 問題になるのは、スクロールに**付け足す非本質的なアニメーション**
- パララックスや装飾の横移動などは、オフにできる必要がある

```text
許容される例：
  通常の縦スクロールで本文が表示される

オフが必要な例：
  縦スクロールに合わせて装飾が横から出入りする
  前景と背景が異なる速度で動くパララックス
```

## 適合の3つの経路

1. **不要なアニメーションを使わない**
2. **サイト側のコントロール**で、操作由来の非本質的アニメをオフにできる
3. **OS / ユーザーエージェントの「動きを減らす」設定**（`prefers-reduced-motion`）を尊重する

```text
推奨順：
1. そもそも不要な動きを入れない
2. prefers-reduced-motion を尊重する
3. サイト全体でアニメーションOFFの設定を用意する
```

> **参照**
> - [Understanding SC 2.3.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html#intent)

## 手法1：CSS の prefers-reduced-motion（C39）

- OS やブラウザの「動きを減らす」設定をメディアクエリで検出する
- 設定が有効なとき、非本質的な動きのアニメーションを無効化する
- 実装コストが低く、広く使える

```css
/* 悪い例：動きを減らす設定を無視する */
.card:hover {
  transform: translateY(-12px) scale(1.05);
  transition: transform 0.4s ease;
}

.parallax-bg {
  transform: translateY(calc(var(--scroll) * -0.3));
}

/* 良い例：prefers-reduced-motion を尊重する */
.card:hover {
  transform: translateY(-12px) scale(1.05);
  transition: transform 0.4s ease;
}

@media (prefers-reduced-motion: reduce) {
  .card:hover {
    transform: none;
    transition: none;
  }

  .parallax-bg {
    transform: none;
  }

  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

> **参照**
> - [Technique C39: Using the CSS prefers-reduced-motion query to prevent motion](https://www.w3.org/WAI/WCAG22/Techniques/css/C39)
> - [MDN — prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)

## 手法2：JavaScript で prefers-reduced-motion を尊重する（SCR40）

- JS で動かすアニメーションも、同じユーザー設定を確認する
- `matchMedia("(prefers-reduced-motion: reduce)")` を使う
- 設定変更を `change` イベントで監視するとよい

```javascript
const motionQuery = window.matchMedia("(prefers-reduced-motion: reduce)");

function startPageFlip() {
  if (motionQuery.matches) {
    // アニメなしでコンテンツを表示
    showContentImmediately();
    return;
  }
  playPageFlipAnimation();
}

motionQuery.addEventListener("change", () => {
  if (motionQuery.matches) {
    stopNonEssentialAnimations();
  }
});
```

```html
<!-- 良い例：ページめくりは設定次第でオフ -->
<button type="button" id="next-chapter">次の章へ</button>
```

> **参照**
> - [Technique SCR40: Using the CSS prefers-reduced-motion query in JavaScript to prevent motion](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR40)
> - [Understanding SC 2.3.3 — page-flip example](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html#examples)

## 手法3：サイト側でアニメーションOFFを用意する

- ページ先頭などに「不要なアニメーションをオフ」するコントロールを置く
- サイト全体の設定として保存するとよい
- `prefers-reduced-motion` 非対応環境や、OS設定を触りたくないユーザーにも有効

```html
<!-- 良い例：サイト全体で動きをオフにできる -->
<form>
  <label>
    <input type="checkbox" id="reduce-motion" name="reduce-motion">
    装飾的なアニメーションをオフにする
  </label>
</form>
```

```javascript
const reduceMotion = document.getElementById("reduce-motion");

reduceMotion.addEventListener("change", () => {
  document.documentElement.classList.toggle(
    "reduce-motion",
    reduceMotion.checked
  );
  localStorage.setItem("reduce-motion", String(reduceMotion.checked));
});

// 初期化
if (localStorage.getItem("reduce-motion") === "true") {
  reduceMotion.checked = true;
  document.documentElement.classList.add("reduce-motion");
}
```

```css
html.reduce-motion .parallax-bg,
html.reduce-motion .scroll-decoration {
  transform: none !important;
  animation: none !important;
  transition: none !important;
}
```

> **参照**
> - [Understanding SC 2.3.3 — parallax scrolling example](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html#examples)

## 例外：不可欠なアニメーション

- 取り除くと、情報や機能が本質的に変わる場合は例外
- 同じ情報・機能を、適合する別の方法で提供できない場合
- 例：アニメーション作成ツールで、作成中のシーケンスをプレビューする機能

```text
例外になりうる例：
  アニメーション編集アプリのプレビュー再生

例外にならない例：
  装飾的なパララックス
  ページ遷移の演出めくり
  ホバー時の大きなバウンス
```

> **参照**
> - [Understanding SC 2.3.3 — essential animation example](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html#examples)
> - [Understanding SC 2.3.3 — Key Terms: essential](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html#dfn-essential)

## 2.3.1・2.3.2・2.2.2 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.3.1 / 2.3.2 | 閃光による発作防止 | 明暗の急変が対象 |
| 2.2.2 | 自動で始まる動き・点滅の制御 | ページ側の自動開始 |
| 2.3.3 | 操作で始まる動きの無効化 | 前庭障害への配慮 |

- 閃光対策（2.3.1 / 2.3.2）と、動きによるめまい対策（2.3.3）は別問題
- 閃光がなくても、パララックスで不適合になりうる
- 自動再生カルーセルは主に 2.2.2、スクロール連動パララックスは主に 2.3.3

## 実装時の注意点

- スクロール連動・ホバー・フォーカス・クリックのアニメを洗い出す
- CSS / JS / Canvas / WebGL / Lottie など、実装手段を問わず対象になりうる
- `scroll-behavior: smooth` も、動きを減らす設定では `auto` にする
- OS設定を尊重しつつ、サイト独自のOFFも用意するとよりよい
- 不可欠と主張する前に、「同じ情報を静的に伝えられないか」を検討する

```css
/* 良い例：スムーズスクロールも設定に従う */
html {
  scroll-behavior: smooth;
}

@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
}
```

## テスト・確認方法

### 手動テスト

- 操作で始まる動き（スクロール、ホバー、クリック、フォーカス）を洗い出す
- 非本質的な動きがあるか確認する
- OS の「動きを減らす」をオンにして、動きが止まるか確認する
- サイト独自のOFFコントロールがある場合、それで止まるか確認する
- 不可欠と主張するアニメは、本当に代替不可能か確認する

```text
チェックリスト：
1. ユーザー操作で始まる動きのアニメがあるか
2. それは機能・情報に不可欠か
3. 不可欠でない場合、無効化できるか
4. prefers-reduced-motion を尊重しているか
5. サイト独自のOFFがある場合、サイト全体で効くか
```

### OS 設定の確認場所（例）

```text
iOS / iPadOS: 設定 > アクセシビリティ > 動作 > 視差効果を減らす
macOS: システム設定 > アクセシビリティ > ディスプレイ > 動きを減らす
Windows: 設定 > アクセシビリティ > 視覚効果 > アニメーション効果
```

### 開発者ツールでの確認

```bash
# アニメーション・パララックスまわりを探す
rg "parallax|prefers-reduced-motion|scroll-behavior|requestAnimationFrame|lottie" --glob "*.css" --glob "*.js" --glob "*.tsx"
```

- DevTools のレンダリング設定で `prefers-reduced-motion: reduce` をエミュレートできる場合がある

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.3.3 は、ユーザー操作で始まる**動きのアニメーションを無効にできる**ことを求める達成基準（レベル AAA）
- 前庭障害によるめまい・吐き気・頭痛などの防止が目的
- スクロール自体の移動は不可欠だが、パララックスなどの付け足し動きはオフが必要
- 主な手法は、不要な動きを避ける、`prefers-reduced-motion` を尊重する（C39 / SCR40）、サイト側OFFを用意する
- 不可欠なアニメーションのみ例外
- 2.2.2（自動開始）・2.3.1 / 2.3.2（閃光）と併せて設計する
