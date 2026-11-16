# ARIA Authoring Practices Guide（APG）記事一覧

[W3CのARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)を日本語で要約し、既存の「アクセシブルな◯◯の実装」記事をパターン別に編纂した入口。

公式の [Patterns一覧](https://www.w3.org/WAI/ARIA/apg/patterns/)で確認した30パターンを、同じ順序で掲載する。確認日: 2026-11-16。

## はじめに読む

- [APGの全体像 — パターン・実装例・共通原則を読む](./overview.md)
- [APG Practices — パターンに共通する実装原則](./practices.md)
- [APGパターンを組み合わせるUIと周辺の記事](./composed-patterns.md)

## 編纂の読み方

APG側にパターンごとの要約を置き、コードを含む既存記事はa11y側へリンクする。複数パターンを扱う記事は、それぞれから参照できる。

- **直接対応**: 記事の主題がパターンに対応する。
- **部分対応**: 記事内の特定の節や型が対応する。
- **応用**: 複数の部品や原則を組み合わせて使っている。
- **比較・関連基礎**: 選び分けや前提の理解に役立つ。パターンの実装記事とは区別する。

対応は記事本文と参照先に基づく整理であり、執筆時の着想元を断定するものではない。各記事には公式出典、用途、構造とARIA、キー操作、既存記事との関係を記載した。

## 30パターン

- [Accordion — アコーディオン](./accordion.md) — [アクセシブルなアコーディオンの実装](../a11y/accessible-accordion.md)（直接対応）
- [Alert — アラート](./alert.md) — [アクセシブルなトーストとアラート通知](../a11y/accessible-toast-alert.md)（直接対応）
- [Alert and Message Dialogs — アラートダイアログ](./alertdialog.md) — [アクセシブルなモーダルダイアログの実装](../a11y/accessible-modal-dialog.md)（部分対応）
- [Breadcrumb — パンくずリスト](./breadcrumb.md) — [アクセシブルなパンくずリストの実装](../a11y/accessible-breadcrumb.md)（直接対応）
- [Button — ボタン](./button.md) — [アクセシブルなメディアプレイヤーの実装 — 再生操作とキャプション](../a11y/accessible-media-player.md)（応用）
- [Carousel (Slide Show or Image Rotator) — カルーセル](./carousel.md) — [アクセシブルなカルーセル / スライダーの実装](../a11y/accessible-carousel-slider.md)（直接対応）
- [Checkbox — チェックボックス](./checkbox.md) — [アクセシブルな同意バナーの実装 — フォーカス順と拒否できること](../a11y/accessible-consent-banner.md)（応用）
- [Combobox — コンボボックス](./combobox.md) — [アクセシブルなオートコンプリート（コンボボックス）の実装](../a11y/accessible-combobox.md)（直接対応）
- [Dialog (Modal) — モーダルダイアログ](./dialog-modal.md) — [アクセシブルなモーダルダイアログの実装](../a11y/accessible-modal-dialog.md)（直接対応）
- [Disclosure (Show/Hide) — ディスクロージャー](./disclosure.md) — [アクセシブルなディスクロージャーの実装 — 一部だけを開閉する](../a11y/accessible-disclosure.md)（直接対応）
- [Feed — フィード](./feed.md) — [アクセシブルなページネーションの実装 — 現在ページと前後への移動](../a11y/accessible-pagination.md)（比較対象）
- [Grid (Interactive Tabular Data and Layout Containers) — グリッド](./grid.md) — [アクセシブルなデートピッカーの実装 — カレンダーを矢印キーで操作する](../a11y/accessible-date-picker.md)（応用）
- [Landmarks — ランドマーク](./landmarks.md) — [ランドマークロールとページ構造の設計](../a11y/landmark-regions.md)（直接対応）
- [Link — リンク](./link.md) — [アクセシブルなページネーションの実装 — 現在ページと前後への移動](../a11y/accessible-pagination.md)（応用）
- [Listbox — リストボックス](./listbox.md) — [アクセシブルなリストボックスの実装 — 単一選択と複数選択](../a11y/accessible-listbox.md)（直接対応）
- [Menu and Menubar — メニューとメニューバー](./menubar.md) — [アクセシブルなドロップダウンメニューの実装](../a11y/accessible-dropdown-menu.md)（部分対応）
- [Menu Button — メニューボタン](./menu-button.md) — [アクセシブルなドロップダウンメニューの実装](../a11y/accessible-dropdown-menu.md)（直接対応）
- [Meter — メーター](./meter.md) — [アクセシブルなプログレス表示の実装 — progress と meter の違い](../a11y/accessible-progress-and-meter.md)（部分対応）
- [Radio Group — ラジオグループ](./radio.md) — [アクセシブルなラジオグループの実装 — 矢印キーで一つを選ぶ](../a11y/accessible-radio-group.md)（直接対応）
- [Slider — スライダー](./slider.md) — [アクセシブルなレンジスライダーの実装 — 現在値を読み上げる範囲入力](../a11y/accessible-range-slider.md)（直接対応）
- [Slider (Multi-Thumb) — 複数つまみのスライダー](./slider-multithumb.md) — [アクセシブルなレンジスライダーの実装 — 現在値を読み上げる範囲入力](../a11y/accessible-range-slider.md)（部分対応）
- [Spinbutton — スピンボタン](./spinbutton.md) — [アクセシブルなスピンボタンの実装 — 数値の増減](../a11y/accessible-spinbutton.md)（直接対応）
- [Switch — スイッチ](./switch.md) — [アクセシブルなスイッチの実装 — オン／オフとチェックボックスの使い分け](../a11y/accessible-switch.md)（直接対応）
- [Table — テーブル](./table.md) — [アクセシブルなデータテーブルの実装](../a11y/accessible-data-table.md)（直接対応）
- [Tabs — タブ](./tabs.md) — [アクセシブルなタブUIの実装](../a11y/accessible-tabs.md)（直接対応）
- [Toolbar — ツールバー](./toolbar.md) — [アクセシブルなツールバーの実装 — 関連する操作をまとめる](../a11y/accessible-toolbar.md)（直接対応）
- [Tooltip — ツールチップ](./tooltip.md) — [アクセシブルなツールチップの実装](../a11y/accessible-tooltip.md)（直接対応）
- [Tree View — ツリービュー](./treeview.md) — [アクセシブルなツリービューの実装 — 階層の展開と矢印キー](../a11y/accessible-treeview.md)（直接対応）
- [Treegrid — ツリーグリッド](./treegrid.md) — [アクセシブルなツリービューの実装 — 階層の展開と矢印キー](../a11y/accessible-treeview.md)（関連基礎）
- [Window Splitter — ウィンドウスプリッター](./windowsplitter.md) — 専用の実装記事は未作成。

## APGのパターン名と一致しない記事

ハンバーガーメニュー、ドロワー、メガメニュー、ページネーション、デートピッカー、ウィザード、ファイル選択、メディアプレイヤー、同意バナーなどは、[組み合わせ・周辺UIの案内](./composed-patterns.md)へ。

たとえばデートピッカーはDialogとGrid、非モーダルのハンバーガーメニューはDisclosure、ページネーションはLandmarksとLinkに分けて読む。progressとmeter、カルーセルと数値入力のSliderも区別する。

## 参照時の前提

APGは実装の参考資料であり、規範的な適合基準ではない。要約は選択肢を省略しているため、採用時は各ページの原文で条件と任意操作を確認する。既存記事へのリンクは、コードの網羅性や動作を保証するものではない。

TooltipとWindow Splitterには、確認時点で公式ページに検討・レビュー未完了の注記がある。それぞれの記事にも記載した。

## 関連する記事一覧

- [a11y記事一覧](../a11y/README.md)
- [ARIA APGの読み方 — ウィジェット実装パターン集の公式ガイド](../a11y/aria-apg.md)
- [WCAG記事一覧](../wcag/README.md)

