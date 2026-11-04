# ACF PROで標準スタイルとWordの貼り付け書式を両立する

## ACF PROのリッチエディタとは

- ACF PROのリッチエディタは、文章や画像を編集するWYSIWYGフィールド — WordPressのコンテンツエディタを利用する入力欄
- 今回の要望は、サイト側で標準スタイルを用意しつつ、Wordから貼り付けた箇所ではWordの装飾を優先すること
- たとえば、通常の本文はサイトの文字色と文字サイズで表示し、Wordで赤くした文字は貼り付け後も赤く表示する
- この方針を実現するには、貼り付けた装飾をHTMLに残すことと、サイトのCSSがその装飾を妨げないことの両方が必要になる
- ACFのツールバーをFullにするだけでは、Wordの書式を保持する設定にはならない — ツールバーは編集ボタンの構成を指定するもの
- 以下は設計方針であり、個別サイトでの検証結果ではない — 情報の確認日は2026年10月21日

> 参照: [ACF — Wysiwyg Editor](https://www.advancedcustomfields.com/resources/wysiwyg-editor/) / [ACF — Customize the WYSIWYG toolbars](https://www.advancedcustomfields.com/resources/customize-the-wysiwyg-toolbars/)

## 顧客には標準表示と上書きの範囲を提示する

- 最初に、本文・見出し・リスト・リンク・表の標準表示をサンプルページで提示する
- 標準スタイルは、対応する装飾の指定がない場合に使う見た目として説明する
- 上書き対象の候補は、文字色・背景色・文字サイズ・太字・斜体・下線・段落の配置 — 実際に保持できる項目は顧客のWord文書で確認する
- たとえば「標準は黒い文字、Wordで赤くした箇所は赤、サイズ指定がなければ標準サイズ」と、項目ごとの動作を示す
- Wordの通常の本文にも、フォントやサイズなどの書式が含まれる場合がある — 見た目を変更していない文章でも、貼り付け後に標準スタイルを上書きする可能性がある
- 「Wordの指定を広く残す」と「指定した項目だけ残す」のどちらにするかを決める — 後者では、たとえば色と強調は保持し、フォントと文字サイズはサイト側に統一する
- 顧客が意図して変更した書式だけを自動判別できるとは限らないため、原稿テンプレートや貼り付け時の保持ルールも合わせて決める
- 用紙サイズ・改ページ・ヘッダー・フッター・図形の配置などは、文字の装飾とは別の要件として扱う
- 顧客への説明例は「標準の見た目を用意し、貼り付け時に保持できた文字色や強調を優先する運用とする。再現する項目は実際の原稿で確認し、合意した範囲を対応対象とする」

> 参照: [WordPress — Classic Content Editor Overview](https://learn.wordpress.org/lesson-plan/content-editor-overview/) / [TinyMCE — Introduction to PowerPaste](https://www.tiny.cloud/docs/tinymce/latest/introduction-to-powerpaste/)

## 標準CSSは貼り付けた装飾で上書きできるようにする

- 標準の文字色・フォント・文字サイズ・行間を本文領域に設定し、見出し・段落・リスト・表には必要な標準スタイルを用意する
- Word由来の装飾が要素の`style`属性に残っていれば、同じ要素の同じプロパティに対する通常のサイトCSSより、そのインラインスタイルが優先される
- たとえば、段落の標準色を黒にしていても、段落内の文字に赤い文字色が直接指定されていれば、その文字は赤く表示できる
- 上書きを認めるプロパティには、サイト側で`!important`を使わない — 通常のインラインスタイルより優先され、Wordの装飾が反映されない原因になる
- 親要素の指定は継承されるが、子要素に直接指定された値があればそちらが使われる — 親の文字サイズを変えても、内側の文字にサイズ指定が残っていると変わらない場合がある
- Wordの装飾がクラス名だけで残っていても、そのクラスに対応するCSSがなければ見た目は再現されない — `style`属性が残るケースと区別する
- 標準スタイルへ戻す操作も用意する — 対象の装飾を解除し、必要に応じてエディタ上で整え直す運用を確認する
- 管理画面の編集領域にも公開画面に近い標準スタイルを適用し、編集時と公開時の差を小さくする

> 参照: [W3C — CSS Cascading and Inheritance Level 5](https://www.w3.org/TR/css-cascade-5/) / [WordPress — Classic Content Editor Overview](https://learn.wordpress.org/lesson-plan/content-editor-overview/)

## 装飾が消える場所を切り分けて調整する

- 顧客が使うWord・OS・ブラウザーと、サイトのWordPress・ACF・TinyMCEのバージョンを記録し、実際の原稿で確認する
- 最初に、書式なしのテキストとして貼り付けるモードになっていないか確認する — このモードでは貼り付け時に装飾が取り除かれる
- 貼り付け直後のHTMLを確認し、文字色やサイズの指定が残っていなければ、貼り付け処理やエディタのHTML整理処理を調べる
- 保存して編集画面を開き直したときに指定が消えるなら、保存処理や独自のフィルターなどを調べる
- 保存後も指定が残っているのに公開ページのHTMLから消えるなら、テンプレートや出力時のHTMLフィルタリングを調べる
- 公開ページのHTMLに指定が残っているのに見た目へ反映されないなら、ブラウザーの開発者ツールで適用されたCSSと打ち消されたCSSを確認する
- ACFの`wysiwyg_tinymce_settings`で対象フィールドのTinyMCE設定を調整する方法がある — 実際に組み込まれたバージョンに対応する設定を確認する
- 文字色やサイズなど、合意した装飾を残す範囲で調整する — HTMLの許可範囲や出力時の処理も合わせて確認し、フィルタリング全体の無効化で済ませない
- 標準の貼り付け処理では要件を満たせない場合、Wordからの貼り付けに対応する有料のPowerPasteなどを検討する — ACFへそのまま追加できるとは限らず、対応バージョン・組み込み方法・費用の確認が必要になる

> 参照: [ACF — JavaScript API](https://www.advancedcustomfields.com/resources/javascript-api/) / [ACF — HTML Escaping](https://www.advancedcustomfields.com/resources/html-escaping/) / [TinyMCE — Introduction to PowerPaste](https://www.tiny.cloud/docs/tinymce/latest/introduction-to-powerpaste/)

## 標準表示とWord由来の表示をセットで検証する

- 標準表示の確認 — 装飾のない文章を入力し、本文・見出し・リストなどが提示したサンプルどおりに表示されるか確認する
- 上書きの確認 — Wordで色・サイズ・強調を変えた文章を貼り付け、合意した項目が標準スタイルより優先されるか確認する
- 混在の確認 — 装飾のない文章とWordから貼り付けた文章を同じ入力欄に置き、それぞれが意図した見た目になるか確認する
- 保存後の確認 — 保存・再編集・再保存を行っても、保持した装飾が消えたり増えたりしないか確認する
- 戻す操作の確認 — 装飾を解除した箇所が標準表示に戻り、見出しやリンクなど必要な構造を保てるか確認する
- スマートフォンの確認 — Word由来の大きな文字や表の幅によって、はみ出しや読みにくさが発生しないか確認する
- 実装例として、本文は標準の文字サイズ、Word由来の文字色は保持、表は必要に応じて横スクロールさせるなど、上書きの自由度と表示領域の扱いを分けて設計する
- 納品時には、標準表示のサンプル・保持する装飾の一覧・貼り付け手順・標準表示へ戻す手順を共有する

> 参照: [ACF — Wysiwyg Editor](https://www.advancedcustomfields.com/resources/wysiwyg-editor/) / [WordPress — Classic Content Editor Overview](https://learn.wordpress.org/lesson-plan/content-editor-overview/)

## まとめ

- サイトの標準スタイルを用意し、Wordから保持できた装飾で項目ごとに上書きする方針を取る
- Wordの通常の本文にも書式が含まれるため、何を残すかを顧客と決める
- 貼り付け処理で装飾を残すことと、CSSで上書きを妨げないことの両方が必要になる
- 上書きを認めるプロパティでは、サイト側の`!important`を避ける
- 装飾が消える場合は、貼り付け直後・保存後・公開時の順に切り分ける
- 標準表示へ戻す操作も含め、実際の原稿と利用環境で検証する
- 対応範囲は、標準表示とWord由来の表示を並べて確認したうえで合意する

> 参照: [W3C — CSS Cascading and Inheritance Level 5](https://www.w3.org/TR/css-cascade-5/) / [ACF — JavaScript API](https://www.advancedcustomfields.com/resources/javascript-api/)
