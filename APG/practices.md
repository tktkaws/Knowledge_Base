# APG Practices — パターンに共通する実装原則

[APG記事一覧](./README.md) / [APGの全体像](./overview.md)

パターンを選んでも、名前、フォーカス、値、文書構造の設計は残る。[APG Practices](https://www.w3.org/WAI/ARIA/apg/practices/)の7テーマを、既存記事へつなぐために要約する。確認日: 2026-11-16。

## ランドマークで主要領域を伝える

本文、ナビゲーション、検索などを意味のある領域として示す。対応するHTML要素を先に使い、ARIAを重ねて付けることを目的にしない。

`region` と `form` はアクセシブルな名前があるときにランドマークとして扱われる。複数の同種の領域には目的を区別できる名前を付ける。小さな区切りまで領域化すると、移動先の一覧が分かりにくくなる。出典: [Landmark Regions](https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/)。

- パターン: [Landmarks](./landmarks.md)
- 既存記事: [ランドマークロールとページ構造の設計](../a11y/landmark-regions.md)、[セマンティックHTMLの原則 — div/spanに頼らないマークアップ](../a11y/semantic-html.md)、[スキップリンクの実装と意義](../a11y/skip-link.md)

## 名前と説明を分ける

アクセシブルな名前は「何のコントロールか」、説明は補足情報を伝える。見えるラベルやHTMLの標準の名前付けを優先し、必要に応じて `aria-labelledby`、`aria-label`、`aria-describedby` を使う。

`aria-label` を追加すると、元のテキストによる名前を上書きする場合がある。見えるラベルと読み上げられる名前を照合する。名前付けが許される方法はロールごとに異なるため、すべての要素へ同じ属性を付けない。出典: [Providing Accessible Names and Descriptions](https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/)。

- 既存記事: [aria-label / aria-labelledby / aria-describedby の使い分け](../a11y/aria-label-labelledby-describedby.md)、[アクセシブルなフォームの設計 — エラー表示とバリデーション](../a11y/accessible-form.md)
- 応用: アイコンボタンの名前、ダイアログのタイトル、入力欄の補足説明

## キーボード操作とフォーカスを設計する

ページ内の部品間を移る操作と、複合ウィジェット内の操作を分ける。多くの複合ウィジェットはTabで出入りし、内部を矢印キーで移動する。フォーカス位置を見えるようにし、選択状態と区別する。

内部の移動には、各項目の `tabindex` を切り替えてDOMフォーカスを動かす方法と、対応するロールで `aria-activedescendant` を使う方法がある。後者でもフォーカス表示とスクロールは実装側で管理する。出典: [Developing a Keyboard Interface](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)。

- 既存記事: [キーボードナビゲーションの基本 — Tab, Enter, Escape](../a11y/keyboard-navigation-basics.md)、[フォーカス管理 — tabindex, focus(), フォーカストラップ](../a11y/focus-management.md)、[ロービングタブインデックスパターン](../a11y/roving-tabindex.md)、[フォーカスインジケーターのカスタマイズ — :focus-visibleの活用](../a11y/focus-visible.md)
- 関連: [キーボードショートカットの設計原則とWCAG 2.1要件](../a11y/keyboard-shortcut-design.md)

## 表やグリッドの全体構造を伝える

DOMに全行・全列がある場合は、ブラウザが構造から情報を取得できる。一部しか描画しない場合は、`aria-rowcount` / `aria-colcount` で全体数、`aria-rowindex` / `aria-colindex` で全体の中の位置を伝える。位置は0ではなく1から数える。

並べ替えには見出しセルの `aria-sort`、セル結合には適切なspan属性を使う。HTMLのtableでは `rowspan` / `colspan` を使い、ARIAの同名用途の属性へ置き換えない。出典: [Grid and Table Properties](https://www.w3.org/WAI/ARIA/apg/practices/grid-and-table-properties/)。

- パターン: [Table](./table.md)、[Grid](./grid.md)、[Treegrid](./treegrid.md)
- 既存記事: [アクセシブルなデータテーブルの実装](../a11y/accessible-data-table.md)

## 範囲内の値と限界を伝える

範囲を持つロールでは現在値、最小値、最大値を扱う。ただし、既定値や属性が必要となる条件はロールごとに異なる。すべてに同じ属性セットを機械的に付けない。

数値だけでは意味が分からない場合は `aria-valuetext` を使う。不確定な進捗では `aria-valuenow` を省略する。値の表示や入力制約と支援技術への情報を一致させる。出典: [Communicating Value and Limits for Range Widgets](https://www.w3.org/WAI/ARIA/apg/practices/range-related-properties/)。

- パターン: [Slider](./slider.md)、[複数つまみ](./slider-multithumb.md)、[Spinbutton](./spinbutton.md)、[Meter](./meter.md)、[Window Splitter](./windowsplitter.md)
- 既存記事: [アクセシブルなレンジスライダーの実装 — 現在値を読み上げる範囲入力](../a11y/accessible-range-slider.md)、[アクセシブルなスピンボタンの実装 — 数値の増減](../a11y/accessible-spinbutton.md)、[アクセシブルなプログレス表示の実装 — progress と meter の違い](../a11y/accessible-progress-and-meter.md)

## 文書構造はネイティブHTMLから考える

見出し、段落、リスト、表などの意味は、通常は対応するHTML要素で表せる。`div` とロールで置き換える前に、HTMLで実現できるか確認する。

ARIAによる構造の補足は、既存DOMを変更しにくい場合やカスタム要素など、必要な場面で検討する。見た目だけを根拠に文書の役割を変えない。出典: [Structural Roles](https://www.w3.org/WAI/ARIA/apg/practices/structural-roles/)。

- 既存記事: [セマンティックHTMLの原則 — div/spanに頼らないマークアップ](../a11y/semantic-html.md)、[ARIAロールの基本 — role属性の種類と使い方](../a11y/aria-roles.md)、[WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](../a11y/aria-first-rule.md)

## 意味を消すことと内容を隠すことを区別する

`role="presentation"` と `role="none"` は、要素の意味を抑えるための同義のロール。中のテキストを一律に読み上げ対象から消すものではなく、`aria-hidden="true"` とは違う。

フォーカス可能な要素やグローバルARIA属性を持つ要素では、presentationが無視される。表やリストに適用すると、構造上依存する子の意味にも影響する。出典: [Hiding Semantics with the presentation Role](https://www.w3.org/WAI/ARIA/apg/practices/hiding-semantics/)。

- 既存記事: [aria-hiddenは禁止したほうがよい？ — 誤用が多い属性と代替手段](../a11y/aria-hidden-pitfalls.md)、[アクセシビリティツリーの仕組み](../a11y/accessibility-tree.md)
- 関連: [inert属性の使い方 — モーダル背後を一括で無効にする](../a11y/inert.md)は操作不能にする仕組みを扱う。presentationの代用品ではない。

## 実装後に確認すること

名前、状態、キー操作、フォーカス、見た目を一組として確かめる。自動検査だけで操作の自然さや読み上げの分かりやすさは判断できない。

- [axe-core / Lighthouseを使ったアクセシビリティ自動テスト](../a11y/a11y-automated-testing.md)
- [手動テストのチェックリスト — 最低限確認すべき項目](../a11y/manual-testing-checklist.md)
- [スクリーンリーダーでの手動テスト入門 — VoiceOver / NVDA](../a11y/screen-reader-manual-testing.md)

