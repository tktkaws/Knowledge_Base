# APGの全体像 — パターン・実装例・共通原則を読む

[APG記事一覧](./README.md)

APG（ARIA Authoring Practices Guide）は、ARIAの意味付け、キーボード操作、フォーカス管理を組み合わせてUIを実装するためのガイド。個別のパターンと動作例に加え、複数のUIに共通する実践方法を扱う。[W3C APG](https://www.w3.org/WAI/ARIA/apg/)を基に、このフォルダーでは要点を日本語で整理する。

確認日: 2026-11-16。

## APGと仕様・達成基準の関係

WAI-ARIAは役割や状態などの要件、WCAGはアクセシビリティの達成基準を定める。APGはそれらを実装へ結び付ける参考資料であり、APG自体の適合判定はない。掲載されたコードを使うだけでWCAGへの適合が保証されるわけでもない。

また、APGの例は学習用であり、そのまま製品へ組み込めるUIライブラリではない。[Introduction](https://www.w3.org/WAI/ARIA/apg/about/introduction/)では、ガイドと規範的な標準の違い、サンプルの目的を説明している。

## まず確認する前提

ARIAのロールを付けても、キーボード操作や見た目は追加されない。たとえば `div` を `button` ロールにしても、フォーカスやEnter・Spaceによる起動は自動で実現しない。使えるネイティブHTMLを先に検討する。

ARIAは元の意味を上書きすることもある。見える名前・役割・状態と、支援技術へ伝わる内容を一致させる。実装後は、対象のブラウザと支援技術の組み合わせで操作を確かめる。この前提は [Read Me First](https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/)にまとまっている。

関連する既存記事: [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](../a11y/aria-first-rule.md)、[セマンティックHTMLの原則 — div/spanに頼らないマークアップ](../a11y/semantic-html.md)。

## 公式サイトの四つの入口

- [Patterns](https://www.w3.org/WAI/ARIA/apg/patterns/): UIの目的、キー操作、ロール・状態・プロパティを確認する。各ページから実装例へ進める。
- [Practices](https://www.w3.org/WAI/ARIA/apg/practices/): 名前付けやキーボード操作などの共通原則を確認する。
- [Index](https://www.w3.org/WAI/ARIA/apg/example-index/): ロールや属性などから実装例を探す。
- [About](https://www.w3.org/WAI/ARIA/apg/about/): APGの位置付け、変更履歴、支援技術のサポート情報などを確認する。

## パターンから実装記事へ進む

1. 見た目の呼び名ではなく、利用者が何をするUIかを決める。
2. [パターン一覧](./README.md)から候補を選び、用途を読む。
3. キー操作とフォーカス移動、必要な意味・状態を確認する。
4. 「既存記事との対応」からコードを含む記事へ進む。
5. 任意の操作、採用した型、公式例との違いを把握してテストする。

たとえば「ドロップダウン」は一つのパターン名ではない。リンク集の開閉なら [Disclosure](./disclosure.md)、操作一覧なら [Menu Button](./menu-button.md)、入力値の候補なら [Combobox](./combobox.md)が候補になる。

## この編纂で扱う範囲

このフォルダーは、公式サイトの逐語訳や全サンプルの複製ではない。30パターンの要約、[共通原則](./practices.md)、[組み合わせ・周辺UI](./composed-patterns.md)への案内をまとめる。

既存記事の執筆経緯は推定せず、記事本文の主題と参照先から対応を判断した。「直接対応」でも、すべての型や任意操作を網羅しているという意味ではない。既存の実装コードを今回すべて動作検証したわけではないため、採用時には原文と対象環境で確認する。

詳しい読み方は [ARIA APGの読み方 — ウィジェット実装パターン集の公式ガイド](../a11y/aria-apg.md)、テストは [手動テストのチェックリスト — 最低限確認すべき項目](../a11y/manual-testing-checklist.md) と [スクリーンリーダーでの手動テスト入門 — VoiceOver / NVDA](../a11y/screen-reader-manual-testing.md) を参照する。

