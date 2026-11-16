# Menu and Menubar — メニューとメニューバー

[APG記事一覧](./README.md) / [共通の実装原則](./practices.md)

> 出典: [W3C APG — Menu and Menubar](https://www.w3.org/WAI/ARIA/apg/patterns/menubar/)（確認日: 2026-11-16）
> 公式パターンの要約と既存記事の案内。操作の選択肢や詳細な条件は原文を参照する。

## 用途

OSのメニューのように、項目間を矢印キーで移動して操作を選ぶ。常時見えるメニューバーと、開いたときだけ見えるメニューを扱う。

## 構造とARIA

- `menu` または `menubar` に `menuitem` を置く。状態を持つ項目には `menuitemcheckbox` / `menuitemradio` を使う。
- サブメニューを開く項目には `aria-haspopup` と `aria-expanded` を設定する。

## キーボードとフォーカス

- メニュー内部の移動は矢印キーで行う。横型メニューバーと縦型メニューで使う方向が変わる。
- `Enter` で実行またはサブメニューを開き、`Escape` で閉じて親や起点へ戻る。`Tab` は項目を一つずつ巡回するためのキーではない。

## 実装時に確認すること

一般的なサイトのリンク集ならDisclosureを先に検討する。既存記事は一段のアクションメニューを扱うが、多階層のMenubarを網羅する専用記事は未作成。

## 既存記事との対応

- [アクセシブルなドロップダウンメニューの実装](../a11y/accessible-dropdown-menu.md) — 部分対応。アクションメニュー内部の操作が対象。
- [アクセシブルなメガメニューの実装 — 大きなナビゲーションをキーボードで辿る](../a11y/accessible-mega-menu.md) — 比較対象。主な実装はDisclosureであり、見た目の名前だけでMenubarへ分類しない。

