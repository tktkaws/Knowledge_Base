# Meter — メーター

[APG記事一覧](./README.md) / [共通の実装原則](./practices.md)

> 出典: [W3C APG — Meter](https://www.w3.org/WAI/ARIA/apg/patterns/meter/)（確認日: 2026-11-16）
> 公式パターンの要約と既存記事の案内。操作の選択肢や詳細な条件は原文を参照する。

## 用途

ストレージ使用量など、既知の範囲にある測定値を表示する。処理が完了するまでの進捗にはprogressを使う。

## 構造とARIA

- ネイティブの `meter` を先に検討する。カスタム実装では `meter` ロールと名前を設定する。
- カスタムの `aria-valuenow`、`aria-valuemin`、`aria-valuemax` を値と同期する。数値だけでは分かりにくい場合に `aria-valuetext` を補う。

## キーボードとフォーカス

- 値を利用者が変更する入力ではないため、専用のキーボード操作はない。

## 実装時に確認すること

同じ棒状の見た目でも、測定値はMeter、進捗はprogress、値の調整はSliderと分ける。更新のたびに通知が必要かも別に検討する。

## 既存記事との対応

- [アクセシブルなプログレス表示の実装 — progress と meter の違い](../a11y/accessible-progress-and-meter.md) — 部分対応。meterを扱う節が対象。progressの節までMeterとして分類しない。

