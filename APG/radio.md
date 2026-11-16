# Radio Group — ラジオグループ

[APG記事一覧](./README.md) / [共通の実装原則](./practices.md)

> 出典: [W3C APG — Radio Group](https://www.w3.org/WAI/ARIA/apg/patterns/radio/)（確認日: 2026-11-16）
> 公式パターンの要約と既存記事の案内。操作の選択肢や詳細な条件は原文を参照する。

## 用途

複数の選択肢から一つを選ぶ。各項目を独立して選べるCheckboxとは選択モデルが違う。

## 構造とARIA

- 通常は同じ `name` の `input type="radio"` と、`fieldset` / `legend` を使う。
- カスタムでは名前付きの `radiogroup` に `radio` を置き、`aria-checked` を同期する。

## キーボードとフォーカス

- 通常のグループでは、右・下矢印で次、左・上矢印で前へ移り、選択も切り替える。端では循環する。`Space` はフォーカス中の項目を選ぶ。
- Toolbar内では操作が異なる。ツール間の移動だけで選択を変えず、`Space` などで選択する。

## 実装時に確認すること

ネイティブ入力の動作を不必要に上書きしない。カスタム実装では、グループへの出入りと選択済み項目への復帰も確認する。

## 既存記事との対応

- [アクセシブルなラジオグループの実装 — 矢印キーで一つを選ぶ](../a11y/accessible-radio-group.md) — 直接対応。通常のグループとToolbar内の違いを扱う。
- [アクセシブルなツールバーの実装 — 関連する操作をまとめる](../a11y/accessible-toolbar.md) — 応用。ラジオグループを含むToolbar。

