# 2.1.3 キーボード（例外なし）（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.1.3「キーボード（例外なし）」（Keyboard (No Exception)）
- レベル **AAA**（最高の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.1「キーボード操作可能」に属する
- コンテンツの**すべての機能**が、キーボードインターフェースで操作できること
- 個々のキー入力に**特定のタイミング**を要求しないこと
- 2.1.1 との違い：**経路依存入力の例外がない**

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.3 Keyboard (No Exception)](https://www.w3.org/TR/WCAG22/#keyboard-no-exception)
> - [Understanding SC 2.1.3 Keyboard (No Exception)](https://www.w3.org/WAI/WCAG22/Understanding/keyboard-no-exception.html)

## 達成基準の原文（要約）

- コンテンツのすべての機能は、個々のキー入力に特定のタイミングを要求せず、キーボードインターフェースで操作できること
- **例外なし**

> **参照**
> - [WCAG 2.2 — Success Criterion 2.1.3](https://www.w3.org/TR/WCAG22/#keyboard-no-exception)

## 2.1.1 との違い

| 達成基準 | レベル | 基本方針 | 経路依存入力の例外 |
|---|---|---|---|
| 2.1.1 キーボード | A | すべての機能をキーボードで操作 | **あり** |
| 2.1.3 キーボード（例外なし） | AAA | すべての機能をキーボードで操作 | **なし** |

- 2.1.3 は 2.1.1 の**上位基準**
- 2.1.1 では経路依存の入力（フリーハンド描画、水彩画の筆致など）が例外だったが、2.1.3 では**例外が認められない**
- 経路依存入力が**不可欠なコンテンツ**は、2.1.3 を満たせず、ガイドライン 2.1 の **AAA 適合も不可能**
- 2.1.3 を満たせば 2.1.1 も満たす

```text
2.1.1（A）で適合しうるが 2.1.3（AAA）では適合不可：
  水彩画アプリ（筆の動き・速度が本質）
  フリーハンド描画のみの機能
  モデルヘリコプター飛行シミュレータ（リアルタイム操作が本質）

2.1.1 でも 2.1.3 でも適合可能：
  キーボードでオブジェクトを作成・移動・サイズ変更できる描画プログラム
  切り取り／貼り付けで代替できるドラッグ＆ドロップ
  キーボード入力で代替できる手書き入力
```

> **参照**
> - [wcag/2-1-1-keyboard.md — 2.1.1 キーボード](./2-1-1-keyboard.md)

## なぜ必要か

- キーボードインターフェース**だけ**を使えるユーザーが、すべての機能を完了できるようにするため
- 2.1.1 の例外があると、一部の機能がキーボードで使えないまま残る
- 運動障害・視覚障害のあるユーザーが、マウスやタッチに頼らずに同等の操作を行えるようにするため

> **参照**
> - [Understanding SC 2.1.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/keyboard-no-exception.html#intent)

## 経路依存入力が不可欠な場合

- 基礎機能がユーザーの移動経路（パス）に依存するコンテンツは、2.1.3 を満たせない
- そのようなコンテンツを提供する場合、ガイドライン 2.1 の **Level AAA 適合は不可能**
- 例
  - 水彩画の筆致そのものを教えるアプリ（速度・時間で見た目が変わる）
  - フリーハンド描画のみで完結する機能
  - リアルタイム操作が本質の飛行シミュレータ
- 2.1.1 では「例外」として許容されていたが、2.1.3 では**適合対象外**となる

```text
AAA 適合不可の例：
  マウスで自由に描くことだけが目的のお絵かき機能

AAA 適合可能な例：
  矢印キーで点を移動し Space で結ぶ描画機能
  数値入力で座標・サイズを指定する配置機能
  キーボードで色・太さを選び、直線・図形を描く機能
```

> **参照**
> - [Understanding SC 2.1.3 — path-dependent input](https://www.w3.org/WAI/WCAG22/Understanding/keyboard-no-exception.html#intent)

## 基本方針：2.1.1 と同じ実装

- 追加の専用テクニックは定義されていない
- 2.1.1 のテクニック（G202、H91、SCR35 等）に従う
- 経路依存入力の例外を使わない設計が必要

```html
<!-- 良い例：セマンティックHTML -->
<button type="button">保存</button>
<a href="/about">会社概要</a>

<!-- 悪い例：ポインター専用 -->
<div onclick="save()">保存</div>
```

> **参照**
> - [Understanding SC 2.1.3 — Techniques](https://www.w3.org/WAI/WCAG22/Understanding/keyboard-no-exception.html#techniques)
> - [Technique G202: Ensuring keyboard control for all functionality](https://www.w3.org/WAI/WCAG22/Techniques/general/G202)
> - [Technique H91: Using HTML form controls and links](https://www.w3.org/WAI/WCAG22/Techniques/html/H91)

## 描画・配置のキーボード代替

- 2.1.1 と同様、キーボードで代替可能な操作を提供する
- オブジェクトの作成・移動・サイズ変更・回転
- 点と点の接続（矢印キー + Space）
- 切り取り／貼り付け、数値入力による配置

```text
良い例：
  矢印キーでオブジェクトを移動
  Shift + 矢印キーでサイズ変更
  Enter で確定
  切り取り／貼り付けでドラッグ＆ドロップを代替

悪い例（2.1.3 不適合）：
  マウスドラッグのみで自由描画
  経路依存の筆致のみで表現する機能
```

> **参照**
> - [Understanding SC 2.1.1 — Examples 1, 3](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html#examples)

## ホバー・ドラッグ専用 UI の回避

- ホバーだけで表示・操作できる機能は、キーボードユーザーが使えない
- ドラッグ＆ドロップのみの操作は、キーボード代替を用意する
- 2.1.1 で「例外」がなかった領域だが、2.1.3 ではより厳密にすべての機能をカバーする必要がある

```html
<!-- 良い例：フォーカスでもメニューを開ける -->
<button type="button" aria-expanded="false" aria-controls="menu">製品</button>
<ul id="menu" hidden>...</ul>
```

> **参照**
> - [wcag/1-4-13-content-on-hover-or-focus.md — 1.4.13 ホバーまたはフォーカスで表示されるコンテンツ](./1-4-13-content-on-hover-or-focus.md)

## すべての可視コントロールがフォーカス可能である必要はない

- 2.1.1 と同様、マウスで操作できるすべての見た目のコントロールが個別にフォーカス可能である必要はない
- 要件は、キーボードユーザーが**同等の操作**を実行できること
- 例：送信ボタンが pointer のみでも、Enter でフォーム送信できれば適合しうる

```html
<!-- 適合しうる例 -->
<form action="/search" method="get">
  <input name="q" type="search" aria-label="検索">
</form>
```

> **参照**
> - [Understanding SC 2.1.3 — visible controls note](https://www.w3.org/WAI/WCAG22/Understanding/keyboard-no-exception.html#intent)

## MouseKeys は対象外

- OS の MouseKeys（マウスエミュレーター）は、キーボードインターフェースとはみなされない
- アプリケーションからはマウス操作として見えるため、2.1.1・2.1.3 ともに「キーボード操作」とは別

> **参照**
> - [Understanding SC 2.1.3 — keyboard interface](https://www.w3.org/WAI/WCAG22/Understanding/keyboard-no-exception.html#dfn-keyboard-interface)

## 2.1.2 との関係

| 達成基準 | 焦点 |
|---|---|
| 2.1.1 / 2.1.3 キーボード | すべての機能をキーボードで操作できること |
| 2.1.2 キーボードトラップなし | フォーカスが特定の領域に閉じ込められないこと |

- 2.1.3 を満たす実装は、通常 2.1.2 とも整合する
- キーボードで入れた領域から脱出できないと、2.1.2 違反

> **参照**
> - [wcag/2-1-2-no-keyboard-trap.md — 2.1.2 キーボードトラップなし](./2-1-2-no-keyboard-trap.md)

## よくある失敗例

- 2.1.1 の例外に頼り、フリーハンド描画や経路依存操作だけを提供している
- ホバー専用のメニュー・ツールチップ
- ドラッグ＆ドロップのみで完結する操作
- ポインター専用イベント（`mousedown` のみ等）
- キーの連打や長押しを要求する操作

```html
<!-- 2.1.1 例外・2.1.3 不適合：フリーハンド描画のみ -->
<canvas id="freehand" aria-label="自由描画"></canvas>
<!-- キーボード代替なし -->

<!-- 2.1.3 適合：キーボードでも描画可能 -->
<canvas id="drawing" aria-label="描画"></canvas>
<p>矢印キーでカーソル移動、Space で描画、Enter で確定</p>
```

> **参照**
> - [Failure F54: using only pointing-device-specific event handlers](https://www.w3.org/WAI/WCAG22/Techniques/failures/F54)

## テスト・確認方法

### 手動テスト

- 2.1.1 と同様、マウスを使わずキーボードだけで**すべての機能**を完了できるか確認
- 経路依存の操作に頼っていないか確認
- 描画・ゲーム・シミュレータなど、例外になりうる機能がないか重点確認

```text
チェックリスト：
1. すべてのリンク・ボタン・フォームに Tab で到達できるか
2. Enter / Space で操作できるか
3. ホバーだけで出る機能がないか
4. ドラッグ＆ドロップに代替手段があるか
5. フリーハンド描画など経路依存操作だけの機能がないか
```

### 自動テスト

- 2.1.1 と同様、axe / Lighthouse で一部検出可能
- 経路依存機能の有無は手動確認が必要

> **参照**
> - [a11y/keyboard-navigation-basics.md — キーボードナビゲーションの基本](../a11y/keyboard-navigation-basics.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.1.3 は、**例外なし**ですべての機能をキーボードで操作できることを求める達成基準（レベル AAA）
- 2.1.1 の経路依存入力の例外は**認められない**
- 経路依存入力が不可欠なコンテンツは、ガイドライン 2.1 の AAA 適合が不可能
- 実装方針は 2.1.1 と同じ（セマンティックHTML、キーボード代替、ホバー専用 UI の回避）
- 2.1.2（トラップなし）と併せて確認する
