# アクセシビリティ(a11y) 記事リスト

## 基礎
- [x] [Webアクセシビリティとは何か — なぜ重要なのか](./what-is-web-accessibility.md)
- [x] [WCAG 2.2の4原則 — 知覚可能・操作可能・理解可能・堅牢](./wcag-four-principles.md)
- [x] [アクセシビリティツリーの仕組み](./accessibility-tree.md)
- [x] [スクリーンリーダーの基本的な動作原理](./screen-reader-basics.md)
- [x] [支援技術の種類と概要 — スクリーンリーダー以外を知る](./assistive-technologies.md)

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
- [ ] キーボードショートカットの設計原則とWCAG 2.1要件

## コンポーネントパターン
- [x] [アクセシブルなモーダルダイアログの実装](./accessible-modal-dialog.md)
- [x] [アクセシブルなタブUIの実装](./accessible-tabs.md)
- [x] [アクセシブルなドロップダウンメニューの実装](./accessible-dropdown-menu.md)
- [x] [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [x] [アクセシブルなトーストとアラート通知](./accessible-toast-alert.md)
- [ ] アクセシブルなアコーディオンの実装
- [ ] アクセシブルなツールチップの実装
- [ ] アクセシブルなカルーセル / スライダーの実装
- [ ] アクセシブルなオートコンプリート（コンボボックス）の実装
- [ ] アクセシブルなデータテーブルの実装
- [ ] アクセシブルなパンくずリストの実装

## テスト・検証
- [x] [axe-core / Lighthouseを使ったアクセシビリティ自動テスト](./a11y-automated-testing.md)
- [x] [手動テストのチェックリスト — 最低限確認すべき項目](./manual-testing-checklist.md)
- [x] [カラーコントラスト比の基準と確認方法](./color-contrast.md)
- [ ] スクリーンリーダーでの手動テスト入門 — VoiceOver / NVDA
- [ ] CI/CDにアクセシビリティテストを組み込む方法

## 実践
- [x] [画像のalt属性 — 適切な代替テキストの書き方](./image-alt-text.md)
- [x] [prefers-reduced-motion — モーション設定への対応](./prefers-reduced-motion.md)
- [x] [prefers-color-scheme — ダークモード対応の基礎](./prefers-color-scheme.md)
- [x] [スキップリンクの実装と意義](./skip-link.md)
- [ ] セマンティックHTMLの原則 — div/spanに頼らないマークアップ
- [ ] ランドマークロールとページ構造の設計
- [ ] フォーカスインジケーターのカスタマイズ — :focus-visibleの活用
