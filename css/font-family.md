# font-familyの指定とWebフォントの読み込み

## font-familyの基本

- `font-family` プロパティはテキストの書体を指定する
- カンマ区切りで複数のフォントを指定でき、左から順にフォールバックされる
- ブラウザは指定リストの先頭から順に、利用可能なフォントを探して適用する
- 最後に**総称フォントファミリー（generic family）**を指定するのが必須のベストプラクティス

> 参照: [MDN - font-family](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family)

## 総称フォントファミリー

- すべてのブラウザが持つフォールバック用のキーワード
- フォント名ではなく「カテゴリ」を指す
- 引用符で囲んではいけない

| 総称ファミリー | 説明 | 用途 |
|---|---|---|
| `serif` | 明朝体系。文字にセリフ（装飾）がある | 長文の本文、フォーマルな印象 |
| `sans-serif` | ゴシック体系。セリフがない | UIテキスト、Web本文の定番 |
| `monospace` | 等幅フォント。全文字が同じ幅 | コードブロック、ターミナル表示 |
| `cursive` | 筆記体・手書き風 | 装飾的な見出し（本文には不向き） |
| `fantasy` | 装飾的なフォント | 使用頻度は低い |
| `system-ui` | OSのシステムUIフォント | ネイティブアプリ風のUI |
| `ui-serif` / `ui-sans-serif` / `ui-monospace` / `ui-rounded` | OSのUI用フォント（バリエーション） | より細かいUI指定 |

> 参照: [MDN - font-family](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family)

## font-familyの書き方

### 基本ルール

- フォント名にスペースや特殊文字が含まれる場合は引用符で囲む
- 総称ファミリーは引用符で囲まない
- フォールバックの順序は「最も使いたいフォント → 代替 → 総称」

```css
/* 良い例：適切なフォールバック */
body {
  font-family: "Noto Sans JP", "Helvetica Neue", Arial, sans-serif;
}

/* 悪い例：総称ファミリーがない */
body {
  font-family: "Noto Sans JP", "Helvetica Neue";
  /* → 両方なければブラウザのデフォルトフォントになり、意図しない表示になる */
}

/* 悪い例：総称ファミリーを引用符で囲む */
body {
  font-family: "Noto Sans JP", "sans-serif";
  /* → "sans-serif" という名前のフォントを探してしまう */
}
```

### 日本語サイトでよく使われるフォント指定

```css
/* モダンな日本語サイト向け */
body {
  font-family:
    "Helvetica Neue",
    Arial,
    "Hiragino Kaku Gothic ProN",
    "Hiragino Sans",
    Meiryo,
    sans-serif;
}

/* system-ui を活用するシンプルな指定 */
body {
  font-family: system-ui, sans-serif;
}
```

- `system-ui` はOSのデフォルトUIフォントを使うため、日本語環境ではOS側の日本語フォントが適用される
- macOS: ヒラギノ角ゴシック、Windows: Yu Gothic UI など

> 参照: [MDN - font-family](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family)

## Webフォントの読み込み — @font-face

- `@font-face` はカスタムフォントをブラウザにダウンロードさせて使えるようにするルール
- フォントファイルをサーバーに配置してセルフホスティングする場合に使う

```css
@font-face {
  font-family: "MyCustomFont";
  src:
    url("/fonts/MyCustomFont.woff2") format("woff2"),
    url("/fonts/MyCustomFont.woff") format("woff");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

body {
  font-family: "MyCustomFont", sans-serif;
}
```

### src の記述順序

- **woff2 を最初に記述する**（圧縮率が最も高く、すべてのモダンブラウザが対応）
- woff はフォールバック用（IE11対応が不要ならwoff2だけで十分）
- ttf / otf は基本的に不要（ファイルサイズが大きい）

```css
/* 良い例：woff2のみ（モダンブラウザ対応で十分な場合） */
@font-face {
  font-family: "MyFont";
  src: url("/fonts/MyFont.woff2") format("woff2");
}

/* 悪い例：不要なフォーマットまで列挙する */
@font-face {
  font-family: "MyFont";
  src:
    url("/fonts/MyFont.eot");
  src:
    url("/fonts/MyFont.eot?#iefix") format("embedded-opentype"),
    url("/fonts/MyFont.woff2") format("woff2"),
    url("/fonts/MyFont.woff") format("woff"),
    url("/fonts/MyFont.ttf") format("truetype"),
    url("/fonts/MyFont.svg#MyFont") format("svg");
  /* → eot, svg は不要。2020年代のブラウザ対応なら woff2 のみで十分 */
}
```

> 参照: [MDN - @font-face](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face)

## font-display プロパティ

- Webフォントの読み込み中にテキストをどう表示するかを制御する
- `@font-face` の中で指定する

| 値 | 動作 | ユースケース |
|---|---|---|
| `auto` | ブラウザのデフォルト動作 | 特に指定がない場合 |
| `block` | フォント読み込みまでテキストを非表示（最大3秒） | アイコンフォント |
| `swap` | フォールバックフォントで即座に表示し、読み込み完了後に差し替え | **本文テキスト（推奨）** |
| `fallback` | 短時間（約100ms）非表示→フォールバック表示→差し替え（猶予約3秒） | バランス重視 |
| `optional` | 短時間非表示→フォールバック表示。ネットワークが遅ければ差し替えしない | パフォーマンス最優先 |

```css
/* 良い例：swapで即座にテキストを表示する */
@font-face {
  font-family: "MyFont";
  src: url("/fonts/MyFont.woff2") format("woff2");
  font-display: swap;
}

/* 悪い例：font-displayを指定しない */
@font-face {
  font-family: "MyFont";
  src: url("/fonts/MyFont.woff2") format("woff2");
  /* → ブラウザによっては数秒間テキストが見えない（FOIT） */
}
```

### FOIT と FOUT

- **FOIT (Flash of Invisible Text)**: フォント読み込み中にテキストが見えなくなる現象
- **FOUT (Flash of Unstyled Text)**: フォールバックフォントからWebフォントに切り替わる際のちらつき
- `font-display: swap` はFOUTが起きるが、FOITよりユーザー体験が良い（テキストが読める）

> 参照: [MDN - font-display](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display)

## Google Fontsの読み込み方法

### link タグで読み込む

```html
<!-- Google Fontsの推奨方法 -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700&display=swap" rel="stylesheet">
```

- `preconnect` で事前にDNS解決・TLS接続を行い、読み込みを高速化
- `display=swap` パラメータで `font-display: swap` が適用される

### @import で読み込む（非推奨）

```css
/* 悪い例：@importはレンダリングをブロックする */
@import url("https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700&display=swap");

/* → CSSの解析中にフォントCSSの取得が必要になり、パフォーマンスが低下する */
```

- `@import` はCSSの解析中に追加のHTTPリクエストを発生させる
- `<link>` タグならHTMLの解析と並行してフォントCSSを取得できる

> 参照: [Google Fonts - Get Started](https://fonts.google.com/knowledge)

## パフォーマンス最適化

### 1. サブセット化

- 日本語フォントはファイルサイズが非常に大きい（数MB〜数十MB）
- 使う文字だけを抽出して軽量化するのがサブセット化

```css
/* unicode-rangeで必要な文字範囲だけを読み込む */
@font-face {
  font-family: "NotoSansJP";
  src: url("/fonts/NotoSansJP-Regular-latin.woff2") format("woff2");
  unicode-range: U+0000-00FF; /* ラテン文字 */
  font-display: swap;
}

@font-face {
  font-family: "NotoSansJP";
  src: url("/fonts/NotoSansJP-Regular-jp.woff2") format("woff2");
  unicode-range: U+3000-9FFF; /* 日本語（ひらがな・カタカナ・漢字） */
  font-display: swap;
}
```

- `unicode-range` を使うと、ページ内で実際に使われている文字を含むファイルだけがダウンロードされる
- Google Fonts は自動的にサブセット分割してくれる

### 2. フォントファイルのプリロード

```html
<!-- 最も重要なフォントだけをプリロードする -->
<link
  rel="preload"
  href="/fonts/MyFont.woff2"
  as="font"
  type="font/woff2"
  crossorigin>
```

- `preload` でフォントファイルを早期に取得開始する
- `crossorigin` 属性は同一オリジンでも必要（フォントの仕様上の要件）
- プリロードは1〜2ファイルに絞る（多すぎると逆効果）

### 3. セルフホスティング vs CDN

```
セルフホスティング:
  ✓ サードパーティへの依存なし
  ✓ プライバシー配慮（外部へのリクエストが発生しない）
  ✓ HTTP/2環境ならCDNとの速度差はほぼない
  ✗ フォントファイルの管理・更新が必要

CDN（Google Fonts等）:
  ✓ 設定が簡単
  ✓ 自動サブセット化・最適化
  ✗ 外部サービスへの依存
  ✗ プライバシーの懸念（GDPRなど）
```

> 参照: [web.dev - Best practices for fonts](https://web.dev/font-best-practices/)

## 可変フォント（Variable Fonts）

- 1つのフォントファイルに複数のウェイトやスタイルを含められる技術
- 従来は各ウェイトごとに別ファイルが必要だったが、可変フォントなら1ファイルで済む

```css
/* 従来：ウェイトごとに別ファイル */
@font-face {
  font-family: "MyFont";
  src: url("/fonts/MyFont-Regular.woff2") format("woff2");
  font-weight: 400;
}
@font-face {
  font-family: "MyFont";
  src: url("/fonts/MyFont-Bold.woff2") format("woff2");
  font-weight: 700;
}

/* 可変フォント：1ファイルでウェイト範囲を指定 */
@font-face {
  font-family: "MyFont";
  src: url("/fonts/MyFont-Variable.woff2") format("woff2-variations");
  font-weight: 100 900; /* 100〜900の範囲を1ファイルでカバー */
}

h1 {
  font-weight: 650; /* 従来は不可能だった中間値も指定可能 */
}
```

- `font-weight: 100 900` のように範囲で指定する
- `font-variation-settings` でより細かい軸（幅、傾斜など）の制御も可能

> 参照: [MDN - Variable fonts](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_fonts/Variable_fonts_guide)

## まとめ

- `font-family` は最後に必ず総称ファミリー（`sans-serif` 等）を指定する
- Webフォントは **woff2** 形式を使い、不要なフォーマットは省く
- `font-display: swap` を指定してテキストの即時表示を優先する
- Google Fonts は `<link>` タグ + `preconnect` で読み込む（`@import` は避ける）
- 日本語フォントは `unicode-range` によるサブセット化でパフォーマンスを改善する
- 可変フォントで複数ウェイトを1ファイルに統合できる

## 参照元

- [MDN - font-family](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family)
- [MDN - @font-face](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face)
- [MDN - font-display](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display)
- [MDN - Variable fonts guide](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_fonts/Variable_fonts_guide)
- [web.dev - Best practices for fonts](https://web.dev/font-best-practices/)
- [Google Fonts](https://fonts.google.com/)
