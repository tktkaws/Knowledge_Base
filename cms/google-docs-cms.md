# Google Docsを使用したCMSを作成したい — GASで本文を取得し、許可した書式だけを公開する

## Google Docsを使ったCMSとは

- Google Docsを記事の入力画面にし、GAS（Google Apps Script）で本文を取得すれば、ドキュメントを編集してサイトを更新するCMSを構築できる
- 前回の「GASを使用したCMSを作成したい」では、スプレッドシートの記事データをGitHubへ送り、Astroでビルドしてレンタルサーバーへ配置する構成を考えた<sup><a href="#ref-1">[1]</a></sup>
- 今回は本文の入力先をGoogle Docsに変更する — 記事のID、公開タイトル、`slug`、公開状態、ドキュメントIDは管理用スプレッドシートへ残す
- 処理の流れは「Google Docsで執筆 → GASで本文を検証・変換 → 公開用JSONをGitHubへ同期 → Astroで静的生成 → レンタルサーバーへ配置」となる
- Google Docsの見た目をそのまま再現せず、許可した意味とテキストだけを取り出す — フォントや色などの見た目はサイトのCSSで決める
- 読者がページを開くたびにGoogle Docsへアクセスする構成にはしない — 文書を一般公開せず、同期用GASを実行するアカウントに読み取り権限を与える
- 公式情報の確認日は2026年10月26日

<ol start="1">
  <li id="ref-1"><a href="./GASを使用したCMSを作成したい.md">前回の記事 — GASを使用したCMSを作成したい</a></li>
</ol>

## 許可する書式と、取り込まない要素を決める

- ホワイトリスト方式は、あらかじめ許可した要素だけを出力する方法である — 不要な指定を後から削る方式に比べ、出力できる内容を限定しやすい
- 最初の実装では、通常の段落、見出し2、見出し3だけを許可する — JSONではそれぞれ`paragraph`、`heading2`、`heading3`として表す
- Google Docsの段落には`NORMAL`や`HEADING2`などの見出し種別があり、GASから取得できる — 大きな文字を見出しと推測せず、段落に指定された種別で判定する<sup><a href="#ref-2">[2]</a></sup>
- 記事タイトルの`<h1>`は管理用スプレッドシートの公開タイトルから生成する — 本文にある見出し1、タイトル、副題は許可しない
- 色、文字サイズ、フォント、背景色、配置、太字、下線などは取り込まず、文字だけを残す — この段階ではリンクもクリック可能なリンクとして出力しない
- 表、箇条書き、画像、脚注など、対応していない本文要素があれば同期を止め、文書内の位置を編集者へ知らせる — 内容が欠落したまま公開しないためである
- 不要な装飾の指定を捨てることと、対応できない内容をエラーにすることを分ける — 前者は本文を保ったまま見た目を統一し、後者は編集者に修正を求める
- 画像やリストが必要になったら、変換処理とAstro側の表示処理を用意してから許可対象へ追加する
- 文字の一部に付いた書式を扱う場合は、GASの`getTextAttributeIndices()`などで書式の区切りを取得できる — 太字やリンクへの対応は、その単位で検証する実装になる<sup><a href="#ref-3">[3]</a></sup>

<ol start="2">
  <li id="ref-2"><a href="https://developers.google.com/apps-script/reference/document/paragraph-heading">Google — Enum ParagraphHeading</a></li>
  <li id="ref-3"><a href="https://developers.google.com/apps-script/reference/document/text">Google — Class Text</a></li>
</ol>

## GASで本文をJSONへ変換する

- Google Docsには文書内のタブと子タブがあるため、取得する範囲を明示する — 従来の`Document.getBody()`だけでは、すべてのタブの本文を取得することにはならない<sup><a href="#ref-4">[4]</a></sup>
- 次の例は「1文書1タブ、子タブなし」に限定する — 複数タブがあればエラーにし、最初のタブだけを意図せず公開することを防ぐ
- 本文の段落を順に読み、許可した種別とテキストだけを出力する — 画像には段落内に入るものと段落に固定されるものがあるため、両方を確認する<sup><a href="#ref-5">[5]</a></sup>
- `exportArticleBody()`は本文の変換だけを行う関数である — 同期用アカウントで読み取れるドキュメントIDを渡し、返り値を記事の公開タイトルなどと組み合わせてJSONにする

```js
function exportArticleBody(documentId) {
  const doc = DocumentApp.openById(documentId);
  const tabs = doc.getTabs();

  if (tabs.length !== 1 || tabs[0].getChildTabs().length !== 0) {
    throw new Error('本文は1タブだけにしてください');
  }

  const body = tabs[0].asDocumentTab().getBody();
  const blocks = [];
  const heading = DocumentApp.ParagraphHeading;
  const elementType = DocumentApp.ElementType;

  for (let i = 0; i < body.getNumChildren(); i++) {
    const element = body.getChild(i);
    const position = `本文の${i + 1}番目の要素`;

    if (element.getType() !== elementType.PARAGRAPH) {
      throw new Error(`${position}に未対応の要素があります`);
    }

    const paragraph = element.asParagraph();

    if (paragraph.getPositionedImages().length > 0) {
      throw new Error(`${position}に画像があります`);
    }

    for (let j = 0; j < paragraph.getNumChildren(); j++) {
      if (paragraph.getChild(j).getType() !== elementType.TEXT) {
        throw new Error(`${position}にテキスト以外の要素があります`);
      }
    }

    let type;
    switch (paragraph.getHeading()) {
      case heading.NORMAL:
        type = 'paragraph';
        break;
      case heading.HEADING2:
        type = 'heading2';
        break;
      case heading.HEADING3:
        type = 'heading3';
        break;
      default:
        throw new Error(`${position}に未対応の見出しがあります`);
    }

    // 装飾の属性はコピーせず、空白だけの段落は出力しない
    const text = paragraph.getText();
    if (text.trim() !== '') blocks.push({ type, text });
  }

  return { schemaVersion: 1, blocks };
}
```

- たとえば見出し2と通常の段落があれば、次のJSONになる — HTMLやCSSを保存するフィールドは設けない

```json
{
  "schemaVersion": 1,
  "blocks": [
    { "type": "heading2", "text": "サービスの紹介" },
    { "type": "paragraph", "text": "サービスの内容をここに記入する" }
  ]
}
```

- この例で取得するのはタブの本文だけである — ヘッダー、フッター、コメントを記事の入力場所にはしない
- 提案モードの変更は、公開前に採用・却下を確定する運用とする — 校正中の内容をどの状態で公開するか、編集者と共有しておく
- 本文のJSONには`schemaVersion`を付け、変換ルールを変更したときにAstro側でも対応を確認できるようにする

<ol start="4">
  <li id="ref-4"><a href="https://developers.google.com/apps-script/guides/docs/tabs">Google — Work with tabs</a></li>
  <li id="ref-5"><a href="https://developers.google.com/apps-script/reference/document/paragraph">Google — Class Paragraph</a></li>
</ol>

## 更新の検知とGitHubへの同期

- Googleスプレッドシートのセル編集と異なり、Google Docsの本文編集に対応する`onEdit`トリガーは用意されていない — 定期実行や明示的な公開操作でGASを動かす<sup><a href="#ref-6">[6]</a></sup>
- 最初は、編集が完了したら管理者が同期処理を実行する運用から始めたい — 自動公開を選ぶ場合は、公開済みの記事を編集中でも次の同期で更新されることに注意する
- 定期実行する場合は、管理用シートに登録された文書IDだけを読む — アクセスできるDrive内の文書をすべて公開対象にはしない
- Driveの`getLastUpdated()`で更新日時を取得できるが、その日時だけで公開データの変更とは判断しない — 最終的には変換後のJSONで差分を確認する<sup><a href="#ref-7">[7]</a></sup>
- 色やフォントだけの変更は変換後のJSONへ含まれないため、本文データが同じならGitHubへ再送しない
- 比較には本文だけでなく、公開タイトルや`slug`、公開対象の追加・取り消しも含める — 文書を編集していなくてもサイトの更新が必要になる場合がある
- 予約公開を扱うなら、公開予定の時刻も判定する — 文書の更新日時が変わらなくても、時刻が来れば公開対象が変わる
- 変換エラーや読み取り失敗があれば、記事を削除扱いにせず同期を止める — 正常な公開データを維持し、失敗した文書IDと理由を管理者へ知らせる
- 公開用JSONはGASからGitHubのContents APIで更新し、コミットを契機に前回と同じAstroのビルド・転送処理を実行する<sup><a href="#ref-8">[8]</a></sup>
- GitHubへの同期成功とサイトの公開成功は別に記録する — 公開に失敗した場合は、保存済みのJSONとコミットから再実行できるようにする
- 未公開へ戻す場合は、次のビルドで記事を除外し、レンタルサーバーに残ったHTMLも除去する — GitHubの履歴に残った内容まで消えるわけではない

<ol start="6">
  <li id="ref-6"><a href="https://developers.google.com/apps-script/guides/triggers/installable">Google — Installable Triggers</a></li>
  <li id="ref-7"><a href="https://developers.google.com/apps-script/reference/drive/file">Google — Class File / getLastUpdated()</a></li>
  <li id="ref-8"><a href="https://docs.github.com/en/rest/repos/contents">GitHub — REST API endpoints for repository contents</a></li>
</ol>

## Astroでも許可した要素だけを描画する

- GASで検証していても、Astroのビルド時にJSONの構造を再検証する — `schemaVersion`、許可した`type`、文字列の`text`、記事IDや`slug`の重複などを確認する
- 出力するタグはサイト側で固定し、文書の値から任意のタグ名や`style`属性を生成しない
- 次の例は、検証した記事の`blocks`を1件ずつ渡して使うAstroコンポーネントである — 想定外の要素はビルドエラーにする

```astro
---
const { block } = Astro.props;
const allowedTypes = ['paragraph', 'heading2', 'heading3'];

if (
  !block ||
  !allowedTypes.includes(block.type) ||
  typeof block.text !== 'string'
) {
  throw new Error('未対応の記事ブロックです');
}
---

{block.type === 'paragraph' && <p>{block.text}</p>}
{block.type === 'heading2' && <h2>{block.text}</h2>}
{block.type === 'heading3' && <h3>{block.text}</h3>}
```

- `{block.text}`のような通常の式では、文字列がHTMLとして実行されずにエスケープされる — `set:html`は自動エスケープされないため、この構成では使わない<sup><a href="#ref-9">[9]</a></sup>
- 本文に`<script>`などの文字列を書いても、そのままテキストとして表示する — JSONに変換しただけで安全になるのではなく、表示時の扱いも固定する
- Google Docsから取得した本文を、MDXや`.astro`のソースコードとして生成しない
- リンクへ対応する場合は、許可するURLの方式も決める — 当初は`https:`だけに限定するなど、`javascript:`や任意の属性を引き継がない設計にする
- 太字、リスト、画像などを追加するたびに、許可する構造、検証、表示、エラー時の扱いを一緒に追加する
- Google Docsの編集者は記事を変更できても、同期用GASやGitHubの認証情報を編集できないように分ける — スクリプトの編集者を絞る運用は前回と同じである
- 問い合わせフォームも前回の方針どおり外部サービスへ分け、フォームの受付情報を記事の本文や公開用JSONへ混ぜない
- 確認時には、色やフォントの変更でJSONが変わらないこと、未対応の表・画像・タブで同期が止まること、HTMLのような文字列が実行されないこと、エラーでも公開中の記事が消えないことを確かめる

<ol start="9">
  <li id="ref-9"><a href="https://docs.astro.build/en/reference/directives-reference/#sethtml">Astro — Template directives reference / set:html</a></li>
</ol>

## まとめ

- Google Docsを本文の入力画面にし、記事IDや公開情報を管理用スプレッドシートへ残す構成にできる
- 最初は段落・見出し2・見出し3だけを許可し、色やフォントなどの装飾を引き継がない
- 表や画像など、未対応の内容は無視せず同期エラーとして編集者へ知らせる
- 文書内のタブも取得範囲へ影響するため、対応するタブ構成を明示する
- GASは本文を構造化したJSONへ変換し、公開情報を含む差分がある場合だけGitHubへ反映する
- Google Docsの本文編集には`onEdit`を使わず、定期実行または明示的な公開操作で同期する
- Astro側でも構造を検証し、固定したタグとエスケープされた文字列で表示する
- 同期や変換に失敗した場合は公開中の記事を維持し、フォームは記事の公開処理から分けて運用する
