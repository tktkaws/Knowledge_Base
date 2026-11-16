# Combobox — コンボボックス

[APG記事一覧](./README.md) / [共通の実装原則](./practices.md)

> 出典: [W3C APG — Combobox](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/)（確認日: 2026-11-16）
> 公式パターンの要約と既存記事の案内。操作の選択肢や詳細な条件は原文を参照する。

## 用途

入力または選択用のコントロールと、値の候補を提示するポップアップを組み合わせる。文字入力できる型と、候補から選ぶだけの型がある。

## 構造とARIA

- 入力側の `combobox` に名前、`aria-expanded`、ポップアップを参照する `aria-controls` を設定する。
- ポップアップは `listbox` / `grid` / `tree` / `dialog`。listbox以外なら `aria-haspopup` に種類を示す。編集可能な型では、補完の動作に合わせて `aria-autocomplete` を選ぶ。

## キーボードとフォーカス

- 下矢印で候補へ進み、`Escape` で閉じる。編集可能な型では、候補の確定と通常の文字編集を両立させる。
- listbox・grid・tree型は入力にDOMフォーカスを残し、`aria-activedescendant` で候補を示す。dialog型ではDOMフォーカスをダイアログ内へ移す。

## 実装時に確認すること

候補の種類によってキー操作が変わる。既存のオートコンプリート記事は主にlistbox型として読み、dialog型へ同じフォーカス処理をコピーしない。

## 既存記事との対応

- [アクセシブルなオートコンプリート（コンボボックス）の実装](../a11y/accessible-combobox.md) — 直接対応。主に編集可能なlistbox型の実装を扱う。

