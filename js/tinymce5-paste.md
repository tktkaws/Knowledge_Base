# TinyMCE 5のPasteプラグイン — Wordの書式保持と貼り付け設定

## Pasteプラグインとは

- TinyMCE 5で、Wordなどから貼り付けたHTMLを整理するプラグイン
- コピー元の装飾をすべて再現する機能ではなく、貼り付ける内容の保持・除去を調整する機能
- `plugins`に`paste`を追加して利用する — `pastetext`は書式なしで貼り付けるモードのボタン
- 本記事はTinyMCE 5の公式ドキュメントを対象とする — WordPressやACFで利用する場合は、実際に読み込まれているTinyMCEのバージョンを先に確認する
- TinyMCE 6では貼り付け機能が本体に統合され、Word専用のクリーンアップ機能にも変更があるため、5向けの設定をそのまま移植しない
- 情報の確認日は2026年10月21日

> 参照: [TinyMCE 5 — Paste plugin](https://www.tiny.cloud/docs/tinymce/5/paste/) / [TinyMCE 6 — Migrating from TinyMCE 5：Paste](https://www.tiny.cloud/docs/tinymce/6/migration-from-5x/#paste)

## 最初に押さえる貼り付けモードとフィルター

- `paste_as_text` — 書式なしで貼り付けるモードの初期状態を指定し、既定値は`false`
- `false`は書式保持の保証ではない — 文字色などが残るかは、その後のフィルターやHTML制限にも左右される
- `paste_enable_default_filters` — 標準の貼り付けフィルターを有効にする設定で、既定値は`true`
- この設定を`false`にしても、エディタ全体のHTML制限まで無効になるわけではない
- `paste_merge_formats` — 重複する同じ装飾の入れ子をまとめる設定で、既定値は`true`
- 検証では、まず初期設定で貼り付けたHTMLを確認し、失われた装飾に関係する設定だけを変更する

> 参照: [TinyMCE 5 — Paste plugin：Options](https://www.tiny.cloud/docs/tinymce/5/paste/#options) / [TinyMCE 5 — Content filtering options](https://www.tiny.cloud/docs/tinymce/5/content-filtering/)

## Wordの書式保持に関わる設定

- `paste_retain_style_properties` — Word由来のCSSを保持する項目を指定する旧設定で、`color font-size`のようにスペースで区切る
- `all`も指定できるが、記事本文に不要なフォントや余白まで残す方針になるため、まず必要な項目に絞って検証したい
- `paste_word_valid_elements` — Word由来のHTMLで許可する要素・属性を指定する旧設定
- 公式説明には、利用時に`paste_enable_default_filters`を`false`にする条件がある — 同ページの短いコード例だけではこの条件を読み落としやすい
- `paste_convert_word_fake_lists` — Wordのリスト状の段落をHTMLのリストへ変換する旧設定で、既定値は`true`
- この3つはTinyMCE 5.10で非推奨、6.0で削除と案内されている — 新しい構成へ採用する際は対応するバージョンと代替手段を確認する
- `paste_webkit_styles` — WebKit由来の貼り付けで保持するCSSを指定する設定で、既定値は`none`
- `paste_remove_styles_if_webkit` — WebKit向けの標準スタイル除去を制御する設定で、既定値は`true`
- Word向けの設定とWebKit向けの設定は対象が異なる — どちらか一方を指定すれば、すべての環境で同じ装飾が残るとは限らない

> 参照: [TinyMCE 5 — paste_retain_style_properties](https://www.tiny.cloud/docs/tinymce/5/paste/#paste_retain_style_properties) / [TinyMCE 5 — paste_word_valid_elements](https://www.tiny.cloud/docs/tinymce/5/paste/#paste_word_valid_elements) / [TinyMCE 5 — paste_webkit_styles](https://www.tiny.cloud/docs/tinymce/5/paste/#paste_webkit_styles)

## HTMLの文字列とDOMを加工するコールバック

- `paste_preprocess` — 挿入前のHTML文字列を`args.content`で確認・変更する
- `paste_postprocess` — HTMLがDOMへ解析された後、挿入前の内容を`args.node`で確認・変更する
- DOMは、HTMLを要素とその親子関係として扱うための仕組み — 要素ごとに属性を確認する処理に使える
- たとえば、貼り付けた要素を調べて、保持対象外の装飾を取り除く処理を検討できる
- コールバックで指定を追加しても、HTML制限や保存・出力処理との整合性は別に確認する必要がある
- `valid_elements`はエディタ全体で許可する要素・属性、`valid_styles`は許可するCSSプロパティを制御する
- `valid_styles`は要素名と許可するプロパティの対応を指定するオブジェクトで、全要素を対象にする場合は`*`を使う
- `valid_styles`のプロパティ一覧はカンマ区切り — `paste_retain_style_properties`のスペース区切りと混同しない
- `valid_styles`は、先に削除された装飾を復元する設定ではない — 貼り付け処理で残す条件と、エディタ全体で許可する条件を合わせる

> 参照: [TinyMCE 5 — paste_preprocess](https://www.tiny.cloud/docs/tinymce/5/paste/#paste_preprocess) / [TinyMCE 5 — paste_postprocess](https://www.tiny.cloud/docs/tinymce/5/paste/#paste_postprocess) / [TinyMCE 5 — valid_styles](https://www.tiny.cloud/docs/tinymce/5/content-filtering/#valid_styles)

## 画像・URL・ドラッグ操作は別に確認する

- `paste_data_images` — 貼り付けたデータURL画像を許可する設定で、TinyMCE 5の既定値は`false`
- 許可するだけで画像がサーバーへアップロードされるわけではない — 画像の保存方法は別に設計する
- TinyMCE 6ではこの既定値が`true`へ変わっているため、移行時には画像の貼り付けも確認する
- `smart_paste` — URLをリンクへ変換したり、画像URLを画像へ変換しようとしたりする機能で、既定値は`true`
- URLをそのままの文字列として扱いたい場合は、`false`での動作を確認する
- `paste_block_drop` — ドラッグ＆ドロップによる挿入を禁止する設定で、既定値は`false` — コピー＆ペーストは引き続き利用できる
- 貼り付けで書式を調整できても、ドラッグ操作まで同じ結果になるとは限らないため、操作を分けて確認する
- 検証用の原稿には、通常文・文字色・文字サイズ・リスト・表・画像・URLを含め、貼り付け直後と保存後を比較する

> 参照: [TinyMCE 5 — paste_data_images](https://www.tiny.cloud/docs/tinymce/5/paste/#paste_data_images) / [TinyMCE 5 — smart_paste](https://www.tiny.cloud/docs/tinymce/5/paste/#smart_paste) / [TinyMCE 5 — paste_block_drop](https://www.tiny.cloud/docs/tinymce/5/paste/#paste_block_drop) / [TinyMCE 6 — Migrating from TinyMCE 5：Paste](https://www.tiny.cloud/docs/tinymce/6/migration-from-5x/#paste)

## まとめ

- Pasteプラグインは、TinyMCE 5で貼り付けた内容を整理する機能
- `paste_as_text`が`false`でも、Wordの書式がすべて保持されるとは限らない
- Word向けの書式保持設定には非推奨・削除されたものがあり、対象バージョンの確認が必要になる
- HTML文字列を扱う場合は`paste_preprocess`、DOMを扱う場合は`paste_postprocess`を使う
- 貼り付け専用の処理と、`valid_elements`や`valid_styles`による全体の制限を区別する
- 画像・URL・ドラッグ操作も含め、実際の原稿で保存後まで確認する

> 参照: [TinyMCE 5 — Paste plugin](https://www.tiny.cloud/docs/tinymce/5/paste/) / [TinyMCE 5 — Content filtering options](https://www.tiny.cloud/docs/tinymce/5/content-filtering/) / [TinyMCE 6 — Migrating from TinyMCE 5](https://www.tiny.cloud/docs/tinymce/6/migration-from-5x/)
