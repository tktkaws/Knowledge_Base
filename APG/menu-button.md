# Menu Button — メニューボタン

[APG記事一覧](./README.md) / [共通の実装原則](./practices.md)

> 出典: [W3C APG — Menu Button](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/)（確認日: 2026-11-16）
> 公式パターンの要約と既存記事の案内。操作の選択肢や詳細な条件は原文を参照する。

## 用途

ボタンからアクションなどのメニューを開く。開いた後の項目操作はMenu and Menubarパターンと組み合わせる。

## 構造とARIA

- 起点のボタンに `aria-haspopup="menu"` または `true` を指定し、`aria-expanded` を表示状態と同期する。
- 開く先は `menu`。ボタンから `aria-controls` で参照することは任意。

## キーボードとフォーカス

- `Enter` / `Space` で開き、最初のメニュー項目へフォーカスを移す。
- 下矢印で開いて先頭へ、上矢印で開いて末尾へ移動する操作は任意。開いた後はMenu and Menubarのキー操作を実装する。

## 実装時に確認すること

値の候補を開くコントロールならComboboxやListbox、サイトのリンク集を開くならDisclosureも比較する。

## 既存記事との対応

- [アクセシブルなドロップダウンメニューの実装](../a11y/accessible-dropdown-menu.md) — 直接対応。アクションメニューの節が対象。ナビゲーション型はDisclosureへ分類する。

