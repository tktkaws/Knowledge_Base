# Accordion — アコーディオン

[APG記事一覧](./README.md) / [共通の実装原則](./practices.md)

> 出典: [W3C APG — Accordion](https://www.w3.org/WAI/ARIA/apg/patterns/accordion/)（確認日: 2026-11-16）
> 公式パターンの要約と既存記事の案内。操作の選択肢や詳細な条件は原文を参照する。

## 用途

見出しごとに本文を開閉する。FAQや複数セクションの整理に使う。単一領域の開閉はDisclosure、重なったパネルの切り替えはTabsと比較する。

## 構造とARIA

- 見出し要素の中に開閉用の `button` を置く。ボタンの `aria-expanded` と実際の表示を同期し、`aria-controls` でパネルを参照する。
- パネルの `region` と名前付けは任意。すべてのパネルをランドマークにしてページの構造を細かくしすぎない。閉じられない開いたパネルには、ボタンの `aria-disabled="true"` を使う。

## キーボードとフォーカス

- `Enter` / `Space` で開閉し、`Tab` / `Shift + Tab` は通常の順序で移動する。見出しボタンをすべてTab順序に含める。
- 同時に複数開くか、一つだけ開くかを決める。見た目がタブに似ていても、`tablist` の操作へ置き換えない。

## 実装時に確認すること

閉じた本文のリンクにフォーカスが入らないことと、見出しの階層を確認する。単純な開閉ではネイティブの `details` / `summary` も検討する。

## 既存記事との対応

- [アクセシブルなアコーディオンの実装](../a11y/accessible-accordion.md) — 直接対応。構造、JavaScript、React、detailsとの比較を扱う。

