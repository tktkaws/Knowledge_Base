# WCAG 2.2の4原則 — 知覚可能・操作可能・理解可能・堅牢

## WCAG 2.2の概要

- WCAG（Web Content Accessibility Guidelines）はW3Cが策定するWebアクセシビリティの国際標準ガイドライン
- WCAG 2.2は2023年10月5日にW3C勧告（Recommendation）として正式公開
- WCAG 2.1（2018年）の上位互換であり、2.1の全達成基準を含む
- 2.2では9つの新しい達成基準が追加され、1つ（4.1.1 構文解析）が削除
- 主にモバイルデバイス操作、認知障害、ロービジョンのユーザーへの対応を強化した改訂

### WCAG 2.1からの主な変更点

- 新規追加：9つの達成基準（2.4.11、2.4.12、2.4.13、2.5.7、2.5.8、3.2.6、3.3.7、3.3.8、3.3.9）
- 削除：4.1.1 構文解析（Parsing）— 現代のブラウザが自動的に処理するため不要と判断
- 既存の一部達成基準のレベル変更（2.4.7 フォーカスの可視化がレベルAAからレベルAに変更）

> 参照元:
> - [W3C — WCAG 2.2](https://www.w3.org/TR/WCAG22/)
> - [W3C — What's New in WCAG 2.2](https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/)

## 4原則（POUR）の全体像

- WCAGの全達成基準は4つの原則に分類される
- 頭文字を取って「POUR原則」と呼ばれる
  - **P**erceivable（知覚可能）
  - **O**perable（操作可能）
  - **U**nderstandable（理解可能）
  - **R**obust（堅牢）
- この4原則はアクセシビリティの土台であり、どれか1つでも欠けるとコンテンツにアクセスできないユーザーが生まれる
- 4原則の下にガイドラインがあり、各ガイドラインに具体的な達成基準（Success Criteria）が紐づく構造

```
原則 (Principle)
  └─ ガイドライン (Guideline)
       └─ 達成基準 (Success Criterion) ← レベル A / AA / AAA
```

> 参照元:
> - [W3C — WCAG 2.2 原則](https://www.w3.org/TR/WCAG22/#perceivable)

---

## 原則1: 知覚可能（Perceivable）

- 情報やUIコンポーネントを、ユーザーが知覚できる方法で提供しなければならないという原則
- 視覚・聴覚など特定の感覚に依存せず、代替手段を用意することが求められる

### ガイドライン

| ガイドライン | 概要 |
|---|---|
| 1.1 テキストの代替 | 非テキストコンテンツにテキストによる代替を提供 |
| 1.2 時間依存メディア | 音声・映像コンテンツに代替手段を提供 |
| 1.3 適応可能 | 情報や構造を失わずに異なる方法で提示可能にする |
| 1.4 判別可能 | コンテンツの見やすさ・聞き取りやすさの確保 |

### 代表的な達成基準

#### 1.1.1 非テキストコンテンツ（レベルA）

- すべての非テキストコンテンツ（画像・アイコン・グラフなど）にテキストによる代替を提供
- スクリーンリーダーが画像の内容をユーザーに伝えるために必須

```html
<!-- 悪い例：alt属性がないため、スクリーンリーダーがファイル名を読み上げる -->
<img src="chart-q3-sales.png">

<!-- 悪い例：altが空でないが情報として不十分 -->
<img src="chart-q3-sales.png" alt="グラフ">

<!-- 良い例：画像の内容を適切に説明 -->
<img src="chart-q3-sales.png" alt="2024年第3四半期の売上推移グラフ。7月120万円、8月135万円、9月150万円と右肩上がり">

<!-- 良い例：装飾画像は空のaltで読み上げをスキップ -->
<img src="decorative-border.png" alt="">
```

#### 1.4.3 コントラスト（最低限）（レベルAA）

- テキストと背景のコントラスト比が最低4.5:1以上であること
- 大きなテキスト（18pt以上、または太字14pt以上）は3:1以上
- ロゴや装飾的テキストは対象外

```html
<!-- 悪い例：コントラスト比が不十分（グレーテキスト on 白背景 ≒ 2.5:1） -->
<p style="color: #aaaaaa; background-color: #ffffff;">読みにくいテキスト</p>

<!-- 良い例：コントラスト比4.5:1以上を確保（#595959 on #ffffff ≒ 7:1） -->
<p style="color: #595959; background-color: #ffffff;">読みやすいテキスト</p>
```

#### 1.3.1 情報及び関係性（レベルA）

- 見た目で伝わっている情報・構造・関係性をプログラム的にも判定可能にする
- 見出し・リスト・テーブルなどは適切なHTML要素でマークアップ

```html
<!-- 悪い例：見出しをフォントサイズだけで表現 -->
<div style="font-size: 24px; font-weight: bold;">セクションタイトル</div>

<!-- 良い例：見出し要素を使用 -->
<h2>セクションタイトル</h2>

<!-- 悪い例：テーブルの構造が不明瞭 -->
<div>
  <span>名前</span><span>年齢</span>
  <span>田中</span><span>25</span>
</div>

<!-- 良い例：table要素で構造を明確に -->
<table>
  <thead>
    <tr><th>名前</th><th>年齢</th></tr>
  </thead>
  <tbody>
    <tr><td>田中</td><td>25</td></tr>
  </tbody>
</table>
```

> 参照元:
> - [W3C — WCAG 2.2 ガイドライン1.1](https://www.w3.org/TR/WCAG22/#text-alternatives)
> - [W3C — WCAG 2.2 ガイドライン1.4](https://www.w3.org/TR/WCAG22/#distinguishable)

---

## 原則2: 操作可能（Operable）

- UIコンポーネントやナビゲーションをユーザーが操作できなければならないという原則
- マウスだけでなく、キーボード・音声入力・スイッチデバイスなど多様な入力手段への対応が求められる

### ガイドライン

| ガイドライン | 概要 |
|---|---|
| 2.1 キーボード操作 | すべての機能をキーボードで操作可能にする |
| 2.2 十分な時間 | コンテンツの閲覧・操作に十分な時間を提供 |
| 2.3 発作と身体的反応 | 発作を引き起こすコンテンツの回避 |
| 2.4 ナビゲーション可能 | ページ内の移動や現在位置の把握を支援 |
| 2.5 入力モダリティ | キーボード以外の入力手段への対応 |

### 代表的な達成基準

#### 2.1.1 キーボード（レベルA）

- すべての機能をキーボードで操作可能にすること
- マウスホバーやドラッグでしか使えない機能を作らない

```html
<!-- 悪い例：onclickのみでキーボードからは操作不可能なdiv -->
<div onclick="openMenu()" style="cursor: pointer;">メニューを開く</div>

<!-- 良い例：button要素でキーボード操作を自動サポート -->
<button type="button" onclick="openMenu()">メニューを開く</button>
```

#### 2.4.7 フォーカスの可視化（レベルA ※2.2でAに変更）

- キーボード操作時に、現在フォーカスされている要素が視覚的に判別可能であること
- WCAG 2.1ではレベルAAだったが、2.2でレベルAに引き上げ

```css
/* 悪い例：フォーカスインジケータを完全に削除 */
*:focus {
  outline: none;
}

/* 良い例：カスタムフォーカススタイルを提供 */
*:focus-visible {
  outline: 2px solid #1a73e8;
  outline-offset: 2px;
}
```

#### 2.4.1 ブロックスキップ（レベルA）

- 繰り返し表示されるコンテンツブロックをスキップする手段の提供
- スキップリンクの実装が代表的

```html
<!-- 良い例：スキップリンクの実装 -->
<body>
  <a href="#main-content" class="skip-link">メインコンテンツへスキップ</a>
  <header>
    <nav><!-- ナビゲーション --></nav>
  </header>
  <main id="main-content">
    <!-- メインコンテンツ -->
  </main>
</body>
```

```css
/* スキップリンクのスタイル：通常時は画面外、フォーカス時に表示 */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  padding: 8px 16px;
  background: #000;
  color: #fff;
  z-index: 100;
}
.skip-link:focus {
  top: 0;
}
```

> 参照元:
> - [W3C — WCAG 2.2 ガイドライン2.1](https://www.w3.org/TR/WCAG22/#keyboard-accessible)
> - [W3C — WCAG 2.2 ガイドライン2.4](https://www.w3.org/TR/WCAG22/#navigable)

---

## 原則3: 理解可能（Understandable）

- 情報やUIの操作方法をユーザーが理解できなければならないという原則
- コンテンツの読みやすさ、動作の予測可能性、入力時のエラー防止・修正支援が求められる

### ガイドライン

| ガイドライン | 概要 |
|---|---|
| 3.1 読みやすさ | テキストコンテンツを読みやすく理解可能にする |
| 3.2 予測可能 | Webページの動作を予測可能にする |
| 3.3 入力支援 | 入力ミスの防止と修正を支援する |

### 代表的な達成基準

#### 3.1.1 ページの言語（レベルA）

- ページのデフォルト言語をプログラム的に判定可能にすること
- スクリーンリーダーが正しい言語で読み上げるために必要

```html
<!-- 悪い例：lang属性がない -->
<html>
  <head><title>サイトタイトル</title></head>
  <body>...</body>
</html>

<!-- 良い例：html要素にlang属性を指定 -->
<html lang="ja">
  <head><title>サイトタイトル</title></head>
  <body>...</body>
</html>

<!-- 良い例：部分的に異なる言語がある場合もlang属性で明示 -->
<p>この文章は日本語です。<span lang="en">This is English.</span></p>
```

#### 3.3.2 ラベルまたは説明（レベルA）

- ユーザー入力を要求するフォームコントロールには、ラベルまたは説明を提供
- label要素の関連付け、またはaria-label/aria-labelledbyを使用

```html
<!-- 悪い例：ラベルが関連付けられていない -->
<span>メールアドレス</span>
<input type="email">

<!-- 良い例：label要素でfor/idを使って関連付け -->
<label for="email">メールアドレス</label>
<input type="email" id="email" name="email">

<!-- 良い例：labelで要素を囲む -->
<label>
  メールアドレス
  <input type="email" name="email">
</label>
```

#### 3.3.1 エラーの特定（レベルA）

- 入力エラーが自動的に検出された場合、エラー箇所を特定し、テキストでユーザーに伝える
- エラーメッセージは色だけでなくテキストでも伝達

```html
<!-- 悪い例：エラーを色だけで伝える -->
<input type="email" style="border-color: red;">

<!-- 良い例：エラーメッセージをテキストで表示し、入力欄と関連付け -->
<label for="email">メールアドレス</label>
<input
  type="email"
  id="email"
  aria-describedby="email-error"
  aria-invalid="true"
>
<p id="email-error" role="alert">メールアドレスの形式が正しくありません</p>
```

> 参照元:
> - [W3C — WCAG 2.2 ガイドライン3.1](https://www.w3.org/TR/WCAG22/#readable)
> - [W3C — WCAG 2.2 ガイドライン3.3](https://www.w3.org/TR/WCAG22/#input-assistance)

---

## 原則4: 堅牢（Robust）

- コンテンツが現在および将来の多様なユーザーエージェント（ブラウザ・支援技術）で解釈可能であること
- 標準に準拠した実装が求められる

### ガイドライン

| ガイドライン | 概要 |
|---|---|
| 4.1 互換性 | 支援技術を含む多様なユーザーエージェントとの互換性を最大化 |

### 代表的な達成基準

#### 4.1.2 名前・役割・値（レベルA）

- すべてのUIコンポーネントの名前（name）・役割（role）・値（value）をプログラム的に判定可能にすること
- ネイティブHTML要素を正しく使えば自動的に満たされることが多い
- カスタムコンポーネントではARIA属性で明示が必要

```html
<!-- 悪い例：役割・名前・状態が不明なカスタムチェックボックス -->
<div class="checkbox" onclick="toggle()">
  <div class="checkmark"></div>
  利用規約に同意する
</div>

<!-- 良い例：ネイティブ要素を使用 -->
<label>
  <input type="checkbox" name="agree">
  利用規約に同意する
</label>

<!-- 良い例：カスタム実装時はARIAで名前・役割・値を明示 -->
<div
  role="checkbox"
  aria-checked="false"
  aria-label="利用規約に同意する"
  tabindex="0"
  onclick="toggleCheckbox(this)"
  onkeydown="if(event.key===' ')toggleCheckbox(this)"
>
  <span class="checkmark"></span>
  利用規約に同意する
</div>
```

#### 4.1.1 構文解析（削除）

- WCAG 2.1まで存在した達成基準だが、WCAG 2.2で削除
- 現代のブラウザはHTMLの構文エラーを自動修正するため、個別の達成基準として不要と判断
- ただしHTMLの妥当性を保つことは引き続き重要（他の達成基準に影響するため）

> 参照元:
> - [W3C — WCAG 2.2 ガイドライン4.1](https://www.w3.org/TR/WCAG22/#compatible)
> - [W3C — WCAG 2.2 変更点（4.1.1の削除について）](https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/)

---

## WCAG 2.2で新しく追加された達成基準

### 2.4.11 フォーカスの外観（Focus Appearance）（レベルAA）

- キーボードフォーカスインジケータが十分な面積とコントラストを持つことを要求
- フォーカスインジケータの面積が最低限 `2CSSピクセル` の太さの周囲線以上
- フォーカスされた状態とされていない状態の間で3:1以上のコントラスト比

```css
/* 悪い例：薄くて小さいフォーカスインジケータ */
button:focus-visible {
  outline: 1px solid #cccccc;
}

/* 良い例：十分な太さとコントラストのフォーカスインジケータ */
button:focus-visible {
  outline: 2px solid #0d47a1;
  outline-offset: 2px;
}
```

### 2.4.13 フォーカスが隠れない（Focus Not Obscured）（最低限：レベルAA）

- キーボードフォーカスを受けたコンポーネントが、他のコンテンツ（固定ヘッダー・Cookie同意バナー等）に完全に隠されないこと
- レベルAAでは「完全に隠れない」、レベルAAA（2.4.12）では「一部も隠れない」

```css
/* 悪い例：固定ヘッダーがフォーカス要素を隠す可能性 */
.sticky-header {
  position: fixed;
  top: 0;
  z-index: 100;
}

/* 良い例：scroll-paddingでフォーカス移動時の余白を確保 */
html {
  scroll-padding-top: 80px; /* 固定ヘッダーの高さ分の余白 */
}
```

### 3.2.6 一貫したヘルプ（Consistent Help）（レベルA）

- ヘルプ機能（チャット、問い合わせ先、FAQなど）がある場合、複数ページにわたって一貫した位置に配置
- ページごとにヘルプの場所が変わるとユーザーが混乱する

```html
<!-- 悪い例：ページによってヘルプの位置が異なる -->
<!-- ページAではヘッダーに配置 -->
<header>
  <a href="/help">ヘルプ</a>
</header>

<!-- ページBではフッターに配置 -->
<footer>
  <a href="/help">ヘルプ</a>
</footer>

<!-- 良い例：すべてのページで同じ位置（例：フッター）に統一 -->
<footer>
  <nav aria-label="サポート">
    <a href="/faq">よくある質問</a>
    <a href="/contact">お問い合わせ</a>
  </nav>
</footer>
```

### 3.3.7 冗長な入力（Redundant Entry）（レベルA）

- 同じプロセス内でユーザーが以前入力した情報を再度入力させない
- 自動入力、選択肢からの選択、または以前の入力値の再表示で対応
- セキュリティ上の理由がある場合（パスワードの再入力等）は例外

```html
<!-- 悪い例：確認画面で配送先住所を再度手入力させる -->
<h2>請求先住所</h2>
<input type="text" name="billing-address" placeholder="住所を入力">

<h2>配送先住所</h2>
<input type="text" name="shipping-address" placeholder="住所を再入力してください">

<!-- 良い例：チェックボックスで同じ住所を再利用 -->
<h2>請求先住所</h2>
<input type="text" name="billing-address" id="billing-address">

<h2>配送先住所</h2>
<label>
  <input type="checkbox" onchange="copyAddress()">
  請求先住所と同じ
</label>
<input type="text" name="shipping-address" id="shipping-address">
```

### 3.3.8 アクセシブルな認証（最低限）（Accessible Authentication (Minimum)）（レベルAA）

- 認証プロセスで認知機能テスト（パズル、記憶、文字の書き写し等）を要求しない
- CAPTCHA等を使う場合は、オブジェクト認識やパスワードマネージャーのサポートなどの代替手段を提供
- パスワード入力はペースト可能であること（パスワードマネージャーの利用を阻害しない）

```html
<!-- 悪い例：パスワードのペーストを禁止 -->
<input
  type="password"
  onpaste="return false;"
  placeholder="パスワードを手入力してください"
>

<!-- 良い例：パスワードのペーストを許可（パスワードマネージャー対応） -->
<label for="password">パスワード</label>
<input
  type="password"
  id="password"
  name="password"
  autocomplete="current-password"
>
```

### 3.3.9 アクセシブルな認証（拡張）（Accessible Authentication (Enhanced)）（レベルAAA）

- 3.3.8よりさらに厳しい基準
- オブジェクト認識やユーザー提供の画像・動画の識別も認知機能テストとして扱い、禁止

### 2.5.7 ドラッグ操作（Dragging Movements）（レベルAA）

- ドラッグ操作で実現する機能には、ドラッグを必要としない代替手段を提供
- 運動障害のあるユーザーがシングルポインタ操作で同等の操作を行えるようにする

### 2.5.8 ターゲットサイズ（最低限）（Target Size (Minimum)）（レベルAA）

- タッチ・クリックターゲットのサイズが最低24x24 CSSピクセル以上
- インラインリンクやブラウザのデフォルトコントロールは例外

```css
/* 悪い例：ターゲットサイズが小さすぎる */
.icon-button {
  width: 16px;
  height: 16px;
  padding: 0;
}

/* 良い例：最低限のターゲットサイズを確保 */
.icon-button {
  min-width: 24px;
  min-height: 24px;
  padding: 4px;
}
```

> 参照元:
> - [W3C — What's New in WCAG 2.2](https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/)
> - [W3C — WCAG 2.2 達成基準一覧](https://www.w3.org/TR/WCAG22/#new-features-in-wcag-2-2)

---

## 適合レベル A / AA / AAA の違い

### 3つのレベルの定義

| レベル | 位置づけ | 概要 |
|---|---|---|
| A | 最低限の適合レベル | 満たさないとアクセス不可能になるユーザーが生じる最も基本的な基準 |
| AA | 標準的な適合レベル | 大半のユーザーにとってのアクセシビリティを確保する基準 |
| AAA | 最高の適合レベル | 可能な限り多くのユーザーをカバーする最も厳格な基準 |

### 実務上の目安

- **レベルA**: 最低限の必須対応。これを満たさないと重大なアクセシビリティバリアが存在する
- **レベルAA**: 多くの法律・ガイドラインが求める水準。日本のJIS X 8341-3:2016もAA適合を推奨
- **レベルAAA**: サイト全体でのAAA適合はW3C自身が「推奨しない」と明言（すべてのコンテンツで達成困難なため）。特定のページやコンテンツで部分的に目指す
- 実務では**レベルAA適合**を目標にすることが一般的
- 公共機関・行政サイトではレベルAA適合が法的要件となる国も多い

### 各レベルの達成基準数（WCAG 2.2）

| レベル | 達成基準数 |
|---|---|
| A | 32 |
| AA | 24 |
| AAA | 31 |
| **合計** | **87** |

> 参照元:
> - [W3C — WCAG 2.2 適合について](https://www.w3.org/TR/WCAG22/#conformance)
> - [ウェブアクセシビリティ基盤委員会 — JIS X 8341-3:2016](https://waic.jp/docs/jis2016/understanding/)

---

## まとめ

- WCAG 2.2は2023年10月に勧告された最新のWebアクセシビリティ国際標準
- 4原則（POUR）はアクセシビリティの根幹
  - **知覚可能**: すべてのユーザーが情報を知覚できるように代替手段を提供
  - **操作可能**: キーボードを含む多様な入力手段で操作可能にする
  - **理解可能**: コンテンツの読みやすさと動作の予測可能性を確保
  - **堅牢**: 多様なブラウザ・支援技術で正しく解釈可能にする
- 実務ではレベルAA適合を目標にするのが標準的
- ネイティブHTML要素を正しく使うことが、多くの達成基準を満たす最も効果的な方法
