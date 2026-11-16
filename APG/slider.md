# Slider — スライダー

[APG記事一覧](./README.md) / [共通の実装原則](./practices.md)

> 出典: [W3C APG — Slider](https://www.w3.org/WAI/ARIA/apg/patterns/slider/)（確認日: 2026-11-16）
> 公式パターンの要約と既存記事の案内。操作の選択肢や詳細な条件は原文を参照する。

## 用途

範囲内の値をつまみで調整する。音量や再生位置などに使う。画像の切り替えを指す「スライダー」はCarouselへ分類する。

## 構造とARIA

- 通常は `input type="range"` を先に検討する。カスタムではフォーカス可能なつまみに `slider` と名前を設定する。
- `aria-valuenow`、`aria-valuemin`、`aria-valuemax` を同期し、必要なら `aria-valuetext` で単位や意味を伝える。縦型は `aria-orientation="vertical"`。

## キーボードとフォーカス

- 右・上矢印で増加、左・下矢印で減少する。`Home` / `End` で範囲の端へ移動する。
- `Page Up` / `Page Down` による大きな刻みでの増減は任意。

## 実装時に確認すること

表示値と支援技術が取得する値を一致させる。タッチ型の支援技術で値を変えられることも実機で確認する。複数つまみは別のパターンで追加条件を読む。

## 既存記事との対応

- [アクセシブルなレンジスライダーの実装 — 現在値を読み上げる範囲入力](../a11y/accessible-range-slider.md) — 直接対応。範囲入力の詳しい実装。
- [アクセシブルなカルーセル / スライダーの実装](../a11y/accessible-carousel-slider.md) — 部分対応。「スライダー」の節が対象。
- [アクセシブルなメディアプレイヤーの実装 — 再生操作とキャプション](../a11y/accessible-media-player.md) — 応用。再生位置や音量の調整。

