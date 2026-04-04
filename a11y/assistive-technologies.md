# 支援技術の種類と概要 — スクリーンリーダー以外を知る

## 支援技術（Assistive Technology）とは

- 障害のあるユーザーがコンピュータやWebを利用するために使用するソフトウェア・ハードウェアの総称
- スクリーンリーダーが最も有名だが、それ以外にも多様な支援技術が存在
- フロントエンドエンジニアが対応すべき支援技術はスクリーンリーダーだけではない
- WCAG（Web Content Accessibility Guidelines）は特定の支援技術に限定せず、幅広いユーザーを対象としている

> **参照**
> - [WAI - Assistive Technologies](https://www.w3.org/WAI/people-use-web/tools-techniques/)
> - [WAI - How People with Disabilities Use the Web](https://www.w3.org/WAI/people-use-web/)

## 支援技術の分類

### 1. スクリーンリーダー

- 画面の情報を音声合成や点字ディスプレイで出力するソフトウェア
- 主な対象：全盲・ロービジョン（弱視）・学習障害のあるユーザー
- 代表的な製品：VoiceOver、NVDA、JAWS、TalkBack、Narrator
- アクセシビリティツリーの情報（ロール・名前・状態・値）をもとに読み上げ

### 2. 画面拡大ソフト（スクリーンマグニファイア）

- 画面の一部または全体を拡大表示するソフトウェア
- 主な対象：ロービジョン（弱視）のユーザー
- 代表的な製品：
  - **ZoomText**（Windows、商用）
  - **macOSズーム機能**（macOS標準、`Cmd + Option + =` で起動）
  - **Windows拡大鏡**（Windows標準、`Win + +` で起動）
- 画面全体の拡大だけでなく、マウスカーソル周辺のみの拡大も可能
- 一部の製品はスクリーンリーダー機能も内蔵（ZoomText Fusionなど）

#### フロントエンドでの注意点

- テキストを200%に拡大しても情報が欠落しないこと（WCAG 1.4.4）
- リフロー：320pxの幅でも水平スクロールなしで閲覧可能であること（WCAG 1.4.10）
- 固定サイズ（px）ではなく相対単位（rem, em）でフォントサイズを指定
- `overflow: hidden` で拡大時にコンテンツが切れないよう注意

```css
/* 悪い例：固定サイズで拡大に対応できない */
.container {
  width: 960px;
  overflow: hidden;
  font-size: 14px;
}

/* 良い例：相対単位とリフロー対応 */
.container {
  max-width: 60rem;
  overflow: visible;
  font-size: 0.875rem;
}
```

> **参照**
> - [WCAG 2.2 - 1.4.4 Resize Text](https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html)
> - [WCAG 2.2 - 1.4.10 Reflow](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html)

### 3. 音声認識ソフトウェア

- 音声でコンピュータを操作するソフトウェア
- 主な対象：上肢障害・運動障害のあるユーザー（キーボードやマウスの操作が困難）
- 代表的な製品：
  - **Dragon NaturallySpeaking**（Windows、商用）
  - **音声コントロール**（macOS / iOS標準）
  - **Voice Access**（Android / Windows標準）
- 「〇〇をクリック」のように、画面上の要素名を声で指定して操作
- ボタンやリンクに表示されているテキストとアクセシブルネームが一致していることが重要

#### フロントエンドでの注意点

- 表示テキストとアクセシブルネーム（aria-label）の一致（WCAG 2.5.3 Label in Name）
- すべてのインタラクティブ要素がキーボードで操作可能であること

```html
<!-- 悪い例：表示テキストとaria-labelが不一致 -->
<button aria-label="フォームを送信する">送信</button>
<!-- 音声認識ユーザーが「送信をクリック」と言っても動作しない可能性がある -->

<!-- 良い例：表示テキストがアクセシブルネームに含まれる -->
<button>送信</button>
<!-- 「送信をクリック」で正しく操作できる -->

<!-- aria-labelを使う場合は表示テキストを含める -->
<button aria-label="送信 フォームの内容を送信します">送信</button>
```

> **参照**
> - [WCAG 2.2 - 2.5.3 Label in Name](https://www.w3.org/WAI/WCAG22/Understanding/label-in-name.html)
> - [Apple - Voice Control](https://support.apple.com/guide/mac-help/voice-control-mh40719/mac)

### 4. スイッチデバイス / スイッチアクセス

- 1つまたは少数の物理スイッチでコンピュータを操作する入力デバイス
- 主な対象：重度の運動障害のあるユーザー
- 操作方法：画面上の項目を順番にハイライト（スキャン）し、目的の項目でスイッチを押して選択
- 代表的な製品・機能：
  - **iOS / macOSスイッチコントロール**（標準機能）
  - **Androidスイッチアクセス**（標準機能）
  - 外部スイッチデバイス（ボタン型、吸引/呼気型など）

#### フロントエンドでの注意点

- フォーカス順序が論理的であること（WCAG 2.4.3）
- フォーカスインジケーターが明確に見えること（WCAG 2.4.7）
- すべてのインタラクティブ要素がフォーカス可能でキーボード操作可能であること

> **参照**
> - [Apple - Switch Control](https://support.apple.com/guide/mac-help/switch-control-mh43607/mac)
> - [Android - Switch Access](https://support.google.com/accessibility/android/answer/6122836)

### 5. 視線追跡（アイトラッキング）デバイス

- 目の動きでマウスカーソルを制御する入力デバイス
- 主な対象：重度の運動障害（四肢麻痺など）のあるユーザー
- まばたきやドウェル（一定時間注視）でクリック操作
- 代表的な製品：Tobii Dynavox、EyeTech Digital Systems
- ターゲット（クリック可能な領域）のサイズが十分に大きいことが重要

#### フロントエンドでの注意点

- ターゲットサイズを最低24×24px以上に確保（WCAG 2.5.8、AAレベル）
- ターゲット間に十分なスペースを確保して誤操作を防止
- ホバーのみで表示される重要なコンテンツを避ける

```css
/* 悪い例：ターゲットが小さすぎる */
.icon-button {
  width: 16px;
  height: 16px;
  padding: 0;
}

/* 良い例：十分なターゲットサイズ */
.icon-button {
  min-width: 44px;
  min-height: 44px;
  padding: 10px;
}
```

> **参照**
> - [WCAG 2.2 - 2.5.8 Target Size (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)

### 6. 点字ディスプレイ

- テキスト情報を点字のピンで物理的に表示する出力デバイス
- 主な対象：盲ろう（視覚と聴覚の両方に障害がある）ユーザー、全盲ユーザー
- スクリーンリーダーと連携して動作
- 1行分のテキストを点字セル（通常20〜80セル）で表示
- 代表的な製品：HumanWare Brailliant、Freedom Scientific Focus Blue

#### フロントエンドでの注意点

- 画像のみで情報を伝えないこと（テキストの代替手段が必要）
- テーブルの構造が適切にマークアップされていること（点字ディスプレイは1行ずつしか表示できない）

> **参照**
> - [AFB - Refreshable Braille Displays](https://www.afb.org/blindness-and-low-vision/using-technology/assistive-technology-products/refreshable-braille-displays)

### 7. ハイコントラストモード / カスタムスタイルシート

- 色やコントラストをユーザーの好みに合わせて変更する機能
- 主な対象：ロービジョン・色覚多様性のあるユーザー
- 代表的な機能：
  - **Windowsハイコントラストモード**（forced-colors）
  - **ブラウザの強制カラー設定**
  - **ユーザースタイルシート**
- Windowsハイコントラストモードでは、背景画像・ボックスシャドウ・カスタムカラーが上書きされる

#### フロントエンドでの注意点

- `forced-colors` メディアクエリで必要に応じたスタイル調整
- 色だけで情報を伝えないこと（WCAG 1.4.1）
- ボーダーやアウトラインでUI要素の境界を明示

```css
/* ハイコントラストモードでボタンの境界を保証 */
@media (forced-colors: active) {
  .button {
    border: 2px solid ButtonText;
  }
}
```

> **参照**
> - [MDN - forced-colors](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/forced-colors)
> - [WCAG 2.2 - 1.4.1 Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html)

## 支援技術とアクセシビリティAPIの関係

```
Webコンテンツ (HTML/CSS/JS)
    ↓
ブラウザ (DOM → アクセシビリティツリー)
    ↓
OS アクセシビリティAPI (UIA, NSAccessibility, ATK等)
    ↓
支援技術 (スクリーンリーダー, 拡大ソフト, 音声認識等)
```

- すべての支援技術はOSのアクセシビリティAPIを通じてWebコンテンツの情報を取得
- セマンティックなHTMLを書くことが、あらゆる支援技術への対応の基盤

> **参照**
> - [W3C - Core Accessibility API Mappings](https://www.w3.org/TR/core-aam-1.2/)

## まとめ

- 支援技術はスクリーンリーダーだけではない — 拡大ソフト、音声認識、スイッチデバイス、視線追跡、点字ディスプレイ、ハイコントラストモードなど多様な技術が存在
- 各支援技術に対応するWCAG達成基準が異なるため、幅広い基準を満たすことが重要
- セマンティックなHTML・十分なターゲットサイズ・相対単位の使用・表示テキストとアクセシブルネームの一致が、多くの支援技術に共通して有効な対策
- 特定の支援技術だけをテストするのではなく、WCAG基準に準拠することであらゆる支援技術への対応を実現できる
