# Alert and Message Dialogs — アラートダイアログ

[APG記事一覧](./README.md) / [共通の実装原則](./practices.md)

> 出典: [W3C APG — Alert and Message Dialogs](https://www.w3.org/WAI/ARIA/apg/patterns/alertdialog/)（確認日: 2026-11-16）
> 公式パターンの要約と既存記事の案内。操作の選択肢や詳細な条件は原文を参照する。

## 用途

重要な短いメッセージで作業を中断し、確認や選択を求める。単に目立つ通知を表示するAlertとは、フォーカスの扱いが異なる。

## 構造とARIA

- コンテナーに `alertdialog`、`aria-modal="true"`、アクセシブルな名前を設定する。
- `aria-describedby` で警告本文を関連付ける。モーダルと伝える場合は、実際に背後の操作もできなくする。

## キーボードとフォーカス

- Dialog (Modal)と同じ操作を実装する。開いたら内部にフォーカスを移し、`Tab` / `Shift + Tab` を内部で循環させる。
- `Escape` で閉じ、原則として呼び出し元へ戻す。取り消しにくい操作の確認では、初期フォーカスを安全な選択肢に置くことを検討する。

## 実装時に確認すること

通常の設定画面や長い説明まで一律にalertdialogにしない。短い重要メッセージへの応答を求める場面に絞る。

## 既存記事との対応

- [アクセシブルなモーダルダイアログの実装](../a11y/accessible-modal-dialog.md) — 部分対応。通常のモーダルに加え、alertdialogの使い分けを扱う。

