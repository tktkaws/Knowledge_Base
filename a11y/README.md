# アクセシビリティ(a11y) 記事リスト

## 基礎
- [x] [Webアクセシビリティとは何か — なぜ重要なのか](./what-is-web-accessibility.md)
- [x] [WCAG 2.2の4原則 — 知覚可能・操作可能・理解可能・堅牢](./wcag-four-principles.md)
- [x] [アクセシビリティツリーの仕組み](./accessibility-tree.md)
- [x] [スクリーンリーダーの基本的な動作原理](./screen-reader-basics.md)
- [x] [支援技術の種類と概要 — スクリーンリーダー以外を知る](./assistive-technologies.md)
- [x] [ニューロダイバーシティとWebアクセシビリティ — 認知特性の違いを設計の前提にする](./neurodiversity-and-web-accessibility.md)
- [x] [障害の社会モデルとWebアクセシビリティ — 個人の能力不足ではなく、環境側の障壁として捉える](./social-model-of-disability.md)
- [ ] 一時的・状況的な制約 — 片手操作、騒音、屋外のまぶしさでも同じ設計が効く理由
- [ ] 認知アクセシビリティ（COGA） — 記憶・注意・読解の負荷を下げるW3Cの指針
- [ ] アクセシビリティ・オーバーレイの問題 — 後付けの修正ウィジェットでは代替にならない理由

## ガイドライン
- [ ] ATAG 2.0について — オーサリングツール向けのアクセシビリティガイドライン
- [ ] WCAG 3.0について — 次期ガイドラインの方針とWCAG 2との違い
- [ ] Beyond WCAGとは — 達成基準のチェックを超えた配慮
- [ ] UAAGについて — ブラウザなどユーザーエージェント側のガイドライン
- [ ] JIS X 8341-3について — 日本のウェブアクセシビリティ規格とWCAGの対応
- [ ] EN 301 549について — 欧州の調達で参照されるアクセシビリティ規格
- [ ] Section 508について — 米国でVPATが求められる法的な背景
- [ ] ARIA APGの読み方 — ウィジェット実装パターン集の公式ガイド
- [ ] WCAG2ICTについて — Web以外の文書やソフトウェアへWCAGを当てはめる考え方

## WAI-ARIA
- [x] [ARIAロールの基本 — role属性の種類と使い方](./aria-roles.md)
- [x] [aria-label / aria-labelledby / aria-describedby の使い分け](./aria-label-labelledby-describedby.md)
- [x] [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)
- [x] [aria-expanded / aria-hidden / aria-controls の活用](./aria-expanded-hidden-controls.md)
- [x] [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)
- [x] [aria-current — ナビゲーションの現在位置を示す](./aria-current.md)

## キーボード操作
- [x] [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [x] [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [x] [ロービングタブインデックスパターン](./roving-tabindex.md)
- [x] [キーボードショートカットの設計原則とWCAG 2.1要件](./keyboard-shortcut-design.md)

## コンポーネントパターン
- [x] [アクセシブルなモーダルダイアログの実装](./accessible-modal-dialog.md)
- [x] [アクセシブルなタブUIの実装](./accessible-tabs.md)
- [x] [アクセシブルなドロップダウンメニューの実装](./accessible-dropdown-menu.md)
- [x] [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [x] [アクセシブルなトーストとアラート通知](./accessible-toast-alert.md)
- [x] [アクセシブルなアコーディオンの実装](./accessible-accordion.md)
- [x] [アクセシブルなツールチップの実装](./accessible-tooltip.md)
- [x] [アクセシブルなカルーセル / スライダーの実装](./accessible-carousel-slider.md)
- [x] [アクセシブルなオートコンプリート（コンボボックス）の実装](./accessible-combobox.md)
- [x] [アクセシブルなデータテーブルの実装](./accessible-data-table.md)
- [x] [アクセシブルなパンくずリストの実装](./accessible-breadcrumb.md)
- [ ] アクセシブルなディスクロージャーの実装 — 一部だけを開閉する
- [ ] アクセシブルなハンバーガーメニューの実装 — ナビゲーションを開いたときのフォーカス
- [ ] アクセシブルなドロワーの実装 — 端から出るパネルとモーダルの違い
- [ ] アクセシブルなメガメニューの実装 — 大きなナビゲーションをキーボードで辿る
- [ ] アクセシブルなページネーションの実装 — 現在ページと前後への移動
- [ ] アクセシブルなデートピッカーの実装 — カレンダーを矢印キーで操作する
- [ ] アクセシブルなスイッチの実装 — オン／オフとチェックボックスの使い分け
- [ ] アクセシブルなラジオグループの実装 — 矢印キーで一つを選ぶ
- [ ] アクセシブルなリストボックスの実装 — 単一選択と複数選択
- [ ] アクセシブルなレンジスライダーの実装 — 現在値を読み上げる範囲入力
- [ ] アクセシブルなスピンボタンの実装 — 数値の増減
- [ ] アクセシブルなツールバーの実装 — 関連する操作をまとめる
- [ ] アクセシブルなツリービューの実装 — 階層の展開と矢印キー
- [ ] アクセシブルなプログレス表示の実装 — progress と meter の違い
- [ ] アクセシブルなウィザードの実装 — 複数ステップの進行とエラー
- [ ] アクセシブルなファイル選択の実装 — 選んだファイルと失敗を伝える
- [ ] アクセシブルなメディアプレイヤーの実装 — 再生操作とキャプション
- [ ] カード全体をクリック可能にする — 入れ子になったリンクを避ける
- [ ] アクセシブルな同意バナーの実装 — フォーカス順と拒否できること

## テスト・検証
- [x] [axe-core / Lighthouseを使ったアクセシビリティ自動テスト](./a11y-automated-testing.md)
- [x] [手動テストのチェックリスト — 最低限確認すべき項目](./manual-testing-checklist.md)
- [x] [カラーコントラスト比の基準と確認方法](./color-contrast.md)
- [x] [スクリーンリーダーでの手動テスト入門 — VoiceOver / NVDA](./screen-reader-manual-testing.md)
- [x] [CI/CDにアクセシビリティテストを組み込む方法](./a11y-ci-cd.md)
- [ ] WCAG-EMについて — サイト単位で適合性を評価する手順
- [ ] VPATについて — 製品のアクセシビリティ適合を報告する書式
- [ ] アクセシビリティ方針の書き方 — 対象範囲、目標レベル、例外の示し方
- [ ] 適合レベル A / AA / AAA の選び方 — どこを目標ラインにするか
- [ ] 適合・不適合・適用なしの読み方 — 試験結果の4区分を報告書でどう書くか
- [ ] 支援技術ユーザーによる利用テスト — 自動チェックと手動チェックの先にある確認

## 実践
- [x] [画像のalt属性 — 適切な代替テキストの書き方](./image-alt-text.md)
- [x] [prefers-reduced-motion — モーション設定への対応](./prefers-reduced-motion.md)
- [x] [prefers-color-scheme — ダークモード対応の基礎](./prefers-color-scheme.md)
- [x] [スキップリンクの実装と意義](./skip-link.md)
- [x] [セマンティックHTMLの原則 — div/spanに頼らないマークアップ](./semantic-html.md)
- [x] [ランドマークロールとページ構造の設計](./landmark-regions.md)
- [x] [フォーカスインジケーターのカスタマイズ — :focus-visibleの活用](./focus-visible.md)
- [ ] inert属性の使い方 — 操作不要な部分をフォーカスと支援技術から外す
- [ ] hidden / aria-hidden / inert の使い分け — 非表示にする手段ごとの、読み上げとフォーカスへの影響
- [ ] アクセシブルネームの計算 — 画面のラベルと、支援技術が読む名前の決まり方
- [ ] visually-hidden（sr-only） — 画面では隠し、支援技術には渡す手法
- [ ] prefers-contrast と forced-colors — OSのハイコントラスト設定への対応
- [ ] Popover API とアクセシビリティ — ポップオーバーを開いたときのフォーカスと読み上げ
- [ ] lang属性 — ページ全体と一部分の言語を読み上げエンジンに渡す
