# APGパターンを組み合わせるUIと周辺の記事

[APG記事一覧](./README.md) / [共通の実装原則](./practices.md)

既存記事のUI名と、APGのパターン名は一対一には対応しない。このページは、記事の本文に基づく編集上の対応付けを示す。記事がどの資料から着想されたかを断定するものではない。

2026-11-16に確認した [APG Patterns](https://www.w3.org/WAI/ARIA/apg/patterns/)には、ドロワー、ウィザード、ファイル選択などの独立したパターンはない。以下では使われている部品と共通原則へ分けて案内する。

## ドロップダウンメニュー

実装記事: [アクセシブルなドロップダウンメニューの実装](../a11y/accessible-dropdown-menu.md)

同じ記事が複数の設計を比較している。サイトのリンク集を開閉する部分は [Disclosure](./disclosure.md)、操作一覧を開く部分は [Menu Button](./menu-button.md)と [Menu and Menubar](./menubar.md)へ分類する。値の選択という用途なら [Combobox](./combobox.md) / [Listbox](./listbox.md)と比較するが、記事全体をそれらの実装と見なさない。

## ハンバーガーメニュー・メガメニュー

実装記事: [アクセシブルなハンバーガーメニューの実装 — ナビゲーションを開いたときのフォーカス](../a11y/accessible-hamburger-menu.md)、[アクセシブルなメガメニューの実装 — 大きなナビゲーションをキーボードで辿る](../a11y/accessible-mega-menu.md)

通常のサイトナビゲーションでは、[Disclosure](./disclosure.md)で開閉し、[Landmarks](./landmarks.md)と通常の [Link](./link.md)で構成する。ハンバーガーメニューが背後の操作を止める方式では [Dialog (Modal)](./dialog-modal.md)も関わる。

「メニュー」という名称や大きなパネルの見た目だけで `menu` / `menubar` を採用しない。

## ドロワー

実装記事: [アクセシブルなドロワーの実装 — 端から出るパネルとモーダルの違い](../a11y/accessible-drawer.md)

非モーダルで開閉する型は [Disclosure](./disclosure.md)、開いている間は背後を操作できない型は [Dialog (Modal)](./dialog-modal.md)。常時表示するサイド領域なら、その内容に合う文書構造を使う。画面の端から出るという見た目だけではロールを決められない。

## デートピッカー

実装記事: [アクセシブルなデートピッカーの実装 — カレンダーを矢印キーで操作する](../a11y/accessible-date-picker.md)

主な対応は、カレンダーを開く [Dialog (Modal)](./dialog-modal.md)と、日付を二次元に移動する [Grid](./grid.md)。APGの [Date Picker Dialog Example](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/examples/datepicker-dialog/)もこの組み合わせを示している。

入力欄自体をComboboxとして構成する別の型もあるが、既存記事のカレンダーダイアログを自動的にComboboxへ分類しない。

## ページネーション

実装記事: [アクセシブルなページネーションの実装 — 現在ページと前後への移動](../a11y/accessible-pagination.md)

[Landmarks](./landmarks.md)と [Link](./link.md)、現在位置を伝える `aria-current` が中心。ページの階層を示すBreadcrumbとは目的が違い、同じページ内のパネルを切り替えるTabsでもない。動的な更新では、URL、タイトル、フォーカスの扱いも確認する。

## プログレス表示

実装記事: [アクセシブルなプログレス表示の実装 — progress と meter の違い](../a11y/accessible-progress-and-meter.md)

測定値を示す部分は [Meter](./meter.md)。処理の進捗を示すprogress / progressbarはMeterへ含めず、[共通原則の範囲内の値](./practices.md)と併せて読む。APGのPatterns一覧には独立したProgressbarの項目はないが、Practicesでは範囲を扱うロールの一つとして説明されている。

## ウィザード

実装記事: [アクセシブルなウィザードの実装 — 複数ステップの進行とエラー](../a11y/accessible-wizard.md)

独立したAPGパターンではなく、フォーム、[Button](./button.md)、ステップ切り替え時のフォーカス管理、エラー表示を組み合わせる。順番に入力・検証する手続きを、外観が似ているという理由でTabsへ置き換えない。

時間切れの重要な確認には [Alert Dialog](./alertdialog.md)が関わる場合がある。手順全体を一つのARIAロールで表そうとしない。

## ファイル選択

実装記事: [アクセシブルなファイル選択の実装 — 選んだファイルと失敗を伝える](../a11y/accessible-file-selection.md)

基本はネイティブの `input type="file"` とラベル。削除・アップロードの [Button](./button.md)、状態通知、進捗表示を組み合わせる。重要なエラー通知には [Alert](./alert.md)の考え方が関わるが、すべての成功・進捗を緊急通知にするわけではない。

OSのファイル選択画面を開くことと、Webページ内でDialogパターンを実装することは区別する。

## メディアプレイヤー

実装記事: [アクセシブルなメディアプレイヤーの実装 — 再生操作とキャプション](../a11y/accessible-media-player.md)

再生などの [Button](./button.md)、音量・再生位置の [Slider](./slider.md)などを組み合わせる。字幕や音声解説など、コンテンツ自体に必要な配慮はAPGパターンだけでは完結しない。記事は操作部品とメディアの代替情報を併せて扱っている。

## 同意バナー

実装記事: [アクセシブルな同意バナーの実装 — フォーカス順と拒否できること](../a11y/accessible-consent-banner.md)

通常のバナーと、詳細設定の [Dialog (Modal)](./dialog-modal.md)を分ける。選択肢には [Button](./button.md)や [Checkbox](./checkbox.md)が関わる。バナーを表示するだけでフォーカスを移したり、重要そうな表示だからと一律にAlert Dialogにしたりしない。

同意の選択肢や保存・再変更の設計は、APGの単一パターンを選ぶだけでは決まらない。

## フォーム・トースト

実装記事: [アクセシブルなフォームの設計 — エラー表示とバリデーション](../a11y/accessible-form.md)、[アクセシブルなトーストとアラート通知](../a11y/accessible-toast-alert.md)

フォーム全体の専用パターンを探すのではなく、[Checkbox](./checkbox.md)、[Radio Group](./radio.md)、[Combobox](./combobox.md)などの入力部品と、名前付け・エラー処理を組み合わせる。

トーストは表示形式の呼び名。重要な通知は [Alert](./alert.md)、通常の状況報告はstatusなど、内容と緊急度で使い分ける。返答を求める [Alert Dialog](./alertdialog.md)とは区別する。

## カード・地図

実装記事: [カード全体をクリック可能にする — 入れ子になったリンクを避ける](../a11y/clickable-card.md)、[地図情報のアクセシブルな出し方 — 埋め込み地図だけに頼らない](../a11y/accessible-maps.md)

クリック可能なカードは、移動なら [Link](./link.md)、追加情報の開閉なら [Disclosure](./disclosure.md)が関わる。複数の操作を含むカード全体を、一つのリンクやボタンへ押し込めない。

地図の記事はAPGの特定パターンに直接対応しない。所在地・アクセス方法などをテキストでも提供するという情報設計の記事として、周辺資料に位置付ける。

## 既存記事が薄い領域

編纂時点では、次の範囲は公式解説からさらに実装記事を増やせる。

- [Button](./button.md)、[Checkbox](./checkbox.md)、[Link](./link.md): 部品単体を主題にした専用記事。
- [Feed](./feed.md)、[Treegrid](./treegrid.md)、[Window Splitter](./windowsplitter.md): 専用の実装記事。
- [Grid](./grid.md): カレンダー以外の汎用データグリッド。
- [Menu and Menubar](./menubar.md): 多階層メニューとメニューバー。
- [Slider (Multi-Thumb)](./slider-multithumb.md): 単独記事としての複数つまみの詳しい実装。

これらもパターンの要約は作成済み。「専用記事なし」はAPGの解説がないという意味ではない。

