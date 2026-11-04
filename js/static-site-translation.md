# 静的Webサイトに無料で翻訳機能を追加する方法

## ページ内翻訳とは

- ページ内翻訳は表示中の文章を別の言語に切り替える方式である — 言語ごとに別のHTMLやURLを用意しなくても実装できる
- HTML・CSS・JavaScriptを配信する静的サイトでも、ブラウザーで文章を置き換えれば、同じページのまま翻訳結果を表示できる
- Google以外にもEdge・Firefox・Safariの標準翻訳、MicrosoftのAzure Translator、DeepL、LibreTranslateなどの選択肢がある
- 「無料」は閲覧者のブラウザー機能を使う場合、APIの無料枠を使う場合、翻訳サーバーを自分で運用する場合で条件が異なる
- GoogleのWebサイト翻訳ウィジェットも誰でも新規導入できる前提にはしない — 公式案内では、教育機関、政府、非営利・非商用サイトなどが申請対象となっている<sup><a href="#ref-1">[1]</a></sup>
- 以下は2026年10月28日に確認した仕様をもとに、Google以外の候補と、サイト内の翻訳ボタンを実装する方法を整理したものである

参照

<ol start="1">
  <li id="ref-1"><a href="https://support.google.com/translate/answer/2534559?co=GENIE.Platform%3DDesktop&amp;hl=en">Google Translate Help — Website translation widget</a></li>
</ol>

## Google以外の無料の選択肢

- ブラウザー標準の翻訳はサイト側に翻訳APIやサーバーを追加せずに使える — 翻訳を始める操作は閲覧者がブラウザーで行う
- サイト内に独自の翻訳ボタンを置きたい場合は、翻訳APIを呼び出す処理と、結果を画面へ反映する処理を用意する

| 方法 | 無料で使える範囲 | サイト側の実装・運用 |
|---|---|---|
| Microsoft Edgeの標準翻訳 | ブラウザーの翻訳機能を利用 | 原則として追加コード不要。閲覧者が翻訳メニューを操作 |
| Firefoxの標準翻訳 | 対応言語を端末内で翻訳 | 原則として追加コード不要。言語モデルのダウンロードが必要 |
| Safariの標準翻訳 | 対応言語・地域でブラウザーの翻訳機能を利用 | 原則として追加コード不要。閲覧者が翻訳メニューを操作 |
| Azure TranslatorのF0プラン | 標準翻訳などの無料枠は月200万文字 | Azureアカウントと翻訳APIを呼ぶサーバー側の処理が必要 |
| DeepL API Developer | 無料枠は合計100万文字。使い切っても月次リセットされない | APIアカウントとサーバー側の処理が必要。継続利用では残量を確認 |
| LibreTranslateを自己ホスト | ソフトウェアは無料・オープンソース | 翻訳サーバーの計算資源、公開設定、保守が必要 |

- EdgeはMicrosoft Translatorでページを翻訳でき、元の表示へ戻す操作も備える<sup><a href="#ref-2">[2]</a></sup>
- Firefoxは必要な言語モデルをダウンロードして端末内で翻訳する — 対応言語には日本語も含まれるが、利用する言語の組み合わせは確認する<sup><a href="#ref-3">[3]</a></sup>
- Safariもページ内翻訳に対応する — 対応する言語・地域に制約があり、Mac版の公式説明ではページの本文をAppleのサーバーへ送って翻訳する<sup><a href="#ref-4">[4]</a></sup>
- AzureのF0は月200万文字の無料枠を持つプランである — 有料のS1や、Azureアカウント全体の無料クレジットとは分けて確認する<sup><a href="#ref-5">[5]</a></sup>
- DeepLの旧API Freeは月50万文字だったが、新規受付は終了している — 新しく始める場合のAPI Developerは合計100万文字の無料枠である<sup><a href="#ref-6">[6]</a></sup>
- LibreTranslateはGoogleやAzureに依存しない翻訳APIを自分で動かせる — 公式のホスト済みAPIを本番利用するには有料キーが必要であり、自己ホストのサーバー代も別に考える<sup><a href="#ref-7">[7]</a></sup>

参照

<ol start="2">
  <li id="ref-2"><a href="https://support.microsoft.com/en-au/edge/use-microsoft-translator-in-microsoft-edge-browser">Microsoft Support — Use Microsoft Translator in Microsoft Edge browser</a></li>
  <li id="ref-3"><a href="https://www.firefox.com/en-US/features/translate/">Firefox — Translate a webpage with Firefox</a></li>
  <li id="ref-4"><a href="https://support.apple.com/guide/safari/webpage-translation-in-safari-on-mac-ibrw6ea421e3/mac">Apple Support — Webpage Translation in Safari on Mac</a></li>
  <li id="ref-5"><a href="https://azure.microsoft.com/en-us/pricing/details/translator/">Microsoft Azure — TranslatorのF0無料枠</a></li>
  <li id="ref-6"><a href="https://support.deepl.com/hc/en-us/articles/360021200939-DeepL-API-plans">DeepL Help Center — DeepL API plans</a></li>
  <li id="ref-7"><a href="https://docs.libretranslate.com/guides/faq/">LibreTranslate — FAQ</a></li>
</ol>

## Azureの無料枠でページ内翻訳ボタンを作る

### 静的ページと翻訳処理を分ける

- Google以外の翻訳エンジンでサイト内のボタンを作る例として、Azure Translatorを使う
- HTMLとJavaScriptは静的に配信し、APIキーを使う部分だけCloudflare Pages Functionsで実行する — 完全な静的配信に、翻訳用のサーバー処理を追加する構成である
- AzureのTranslatorリソースをF0で作成し、APIキーを取得する — APIキーはブラウザーへ配信するファイルに書かない<sup><a href="#ref-8">[8]</a></sup>
- Pagesのプロジェクトには次の3ファイルを配置する — `functions` は公開ディレクトリの `public` の中ではなく、プロジェクト直下に置く<sup><a href="#ref-9">[9]</a></sup>

```text
functions/api/translate.js  翻訳APIを呼ぶサーバー側の処理
public/index.html          翻訳するページ
public/translate.js        ボタンと表示切り替えの処理
```

- Pagesの設定で `TRANSLATOR_KEY` をSecretとして登録する — リージョンを持つTranslatorリソースでは、`TRANSLATOR_REGION` にそのリージョンも設定する<sup><a href="#ref-10">[10]</a></sup>
- PagesのGit連携などでデプロイし、公開ディレクトリを `public` にする — 管理画面へのファイルのドラッグ＆ドロップだけではFunctionsをデプロイできない
- Pages Functionsにも実行回数などの無料枠がある — Azureの文字数と、Functions側の使用量の両方を確認する<sup><a href="#ref-11">[11]</a></sup>

### 翻訳APIを呼ぶ処理

- 次は `functions/api/translate.js` の内容である — 日本語から英語への翻訳に固定し、受け付ける文章を20個・合計5,000文字までに制限する
- この件数・文字数はサンプル独自の制限であり、Azureの無料枠やAPIの最大値ではない
- 外部APIの応答に欠落があればエラーとして返し、部分的な結果を正常な翻訳として扱わない

```js
export async function onRequestPost({ request, env }) {
  const fail = (status) => Response.json({ error: '翻訳できない' }, { status });
  if (!env.TRANSLATOR_KEY) return fail(503);

  let texts;
  try {
    ({ texts } = await request.json());
  } catch {
    return fail(400);
  }
  if (!Array.isArray(texts) || texts.length === 0 || texts.length > 20
    || texts.some((text) => typeof text !== 'string' || !text.trim())) {
    return fail(400);
  }
  if (texts.reduce((sum, text) => sum + [...text].length, 0) > 5000) {
    return fail(400);
  }

  const headers = {
    'Content-Type': 'application/json',
    'Ocp-Apim-Subscription-Key': env.TRANSLATOR_KEY,
  };
  if (env.TRANSLATOR_REGION) {
    headers['Ocp-Apim-Subscription-Region'] = env.TRANSLATOR_REGION;
  }

  try {
    const response = await fetch(
      'https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=ja&to=en',
      {
        method: 'POST',
        headers,
        body: JSON.stringify(texts.map((text) => ({ Text: text }))),
      },
    );
    if (!response.ok) return fail(502);

    const data = await response.json();
    const translated = Array.isArray(data)
      ? data.map((item) => item.translations?.[0]?.text) : [];
    if (translated.length !== texts.length
      || translated.some((text) => typeof text !== 'string' || !text.trim())) {
      return fail(502);
    }
    return Response.json({ texts: translated });
  } catch {
    return fail(502);
  }
}
```

### ページに翻訳ボタンを置く

- 次は `public/index.html` の内容である — 翻訳したい、子要素を持たないテキスト要素に `data-translate` を付ける
- `<a>` などを含む段落では翻訳対象の文章を `<span data-translate>` などに分ける — 親要素全体の `textContent` を置き換えてリンクを消さない
- `translate="no"` は翻訳しない箇所を示す属性である — 後述のJavaScriptでも、この属性がある祖先要素の内側を対象から外す<sup><a href="#ref-12">[12]</a></sup>

```html
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="utf-8">
    <title>翻訳ボタンの例</title>
    <script type="module" src="/translate.js"></script>
  </head>
  <body>
    <div translate="no">
      <button id="to-en" type="button"><span lang="en">English</span></button>
      <button id="to-ja" type="button" disabled>日本語に戻す</button>
      <p id="translation-status" role="status"></p>
    </div>
    <main id="content">
      <h1 data-translate>地域のイベント情報</h1>
      <p data-translate>今週末に駅前広場で交流会を開催する</p>
    </main>
  </body>
</html>
```

- 次は `public/translate.js` の内容である — 原文を保持し、翻訳結果がすべてそろってから表示を切り替える
- 翻訳した要素だけ `lang="en"` にする — 操作ボタンなどの日本語が残るため、ページ全体の `lang` は変更しない
- 同じページを開いている間は英訳を再利用する — 日本語へ戻してから再度英語を表示してもAPIを再呼び出ししない

```js
const toEn = document.querySelector('#to-en');
const toJa = document.querySelector('#to-ja');
const status = document.querySelector('#translation-status');
const elements = [...document.querySelectorAll('#content [data-translate]')]
  .filter((el) => el.childElementCount === 0 && !el.closest('[translate="no"]'));
const original = elements.map((el) => el.textContent);
let english = null;

toEn.addEventListener('click', async () => {
  toEn.disabled = true;
  toJa.disabled = true;
  status.textContent = '翻訳中 / Translating…';
  try {
    if (!english) {
      const response = await fetch('/api/translate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ texts: original }),
      });
      if (!response.ok) throw new Error('翻訳に失敗');
      const data = await response.json();
      if (!Array.isArray(data.texts) || data.texts.length !== elements.length
        || data.texts.some((text) => typeof text !== 'string' || !text.trim())) {
        throw new Error('翻訳結果が不正');
      }
      english = data.texts;
    }
    elements.forEach((el, i) => {
      el.textContent = english[i];
      el.lang = 'en';
    });
    status.textContent = '機械翻訳を表示中 / Machine translation';
    toJa.disabled = false;
  } catch {
    status.textContent = '翻訳できないため原文を表示 / Showing original text';
  } finally {
    toEn.disabled = false;
  }
});

toJa.addEventListener('click', () => {
  elements.forEach((el, i) => {
    el.textContent = original[i];
    el.lang = 'ja';
  });
  toJa.disabled = true;
  status.textContent = '原文を表示中 / Original text';
});
```

- この例は指定した本文だけを翻訳する — ページタイトル、画像の `alt`、フォームのラベル、後から追加した文章などは、必要に応じて対象と処理を追加する
- APIキーを隠しても、公開した翻訳用URLへのリクエスト自体は防げない — 本番公開時は呼び出し回数の制限やサーバー側のキャッシュも用意する
- 既存サイトを別のホスティングで配信する場合は、翻訳用のサーバー関数を別途配置する — 別オリジンで呼ぶならCORSも設定する

参照

<ol start="8">
  <li id="ref-8"><a href="https://learn.microsoft.com/en-us/azure/ai-services/translator/text-translation/quickstart/rest-api">Microsoft Learn — Translatorの作成・認証・REST API</a></li>
  <li id="ref-9"><a href="https://developers.cloudflare.com/pages/functions/get-started/">Cloudflare Pages — Functions: Get started</a></li>
  <li id="ref-10"><a href="https://developers.cloudflare.com/pages/functions/bindings/#secrets">Cloudflare Pages — Secrets</a></li>
  <li id="ref-11"><a href="https://developers.cloudflare.com/pages/functions/pricing/">Cloudflare Pages — Functions: Pricing</a></li>
  <li id="ref-12"><a href="https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/translate">MDN — translate HTML global attribute</a></li>
</ol>

## サーバーを追加せずに翻訳する方法

- サーバー処理を増やさず、Google以外の機械翻訳を使うなら、Edge・Firefox・Safariの標準翻訳を案内する方法がある
- 閲覧者はアドレスバーやページメニューなどから翻訳する — サイト内のボタンから各ブラウザーの標準翻訳を一律に起動できる、共通のWeb APIがある前提にはしない
- サイト内に言語切り替えボタンを置き、翻訳文をあらかじめ用意できるなら、JavaScriptの辞書やJSONから表示する文章を切り替える方法もある — 同じURLのまま動作し、翻訳APIの利用料はかからないが、文章の翻訳と更新は自分で行う
- 自動翻訳をブラウザー内で実行する別の選択肢として、`Translator` APIがある — Chromeの実装はブラウザーが提供するモデルを使う方式であり、Google以外の翻訳エンジンを選ぶ方法とは分けて考える<sup><a href="#ref-13">[13]</a></sup>
- Chromeの公式案内ではデスクトップが対象であり、モバイルには対応しない — HTTPSなどの安全なコンテキストで、APIの有無と対象言語の利用可否を確認する<sup><a href="#ref-14">[14]</a></sup>
- `Translator.create()` は利用者の操作を起点に呼び出し、必要なモデルのダウンロード中は待ち時間を伝える — Firefoxの標準翻訳と、Webページから使う `Translator` APIの対応状況は別である

```js
// 対応ブラウザーで、翻訳ボタンのクリック処理などから呼び出す
async function translateOnDevice(text) {
  if (!('Translator' in globalThis)) throw new Error('この環境では利用できない');
  const translator = await Translator.create({
    sourceLanguage: 'ja',
    targetLanguage: 'en',
  });
  try {
    return await translator.translate(text);
  } finally {
    translator.destroy();
  }
}
```

- この関数は文章を翻訳して返す部分だけである — 画面への反映、原文への復帰、モデルのダウンロード表示などはサイト側で実装する

参照

<ol start="13">
  <li id="ref-13"><a href="https://developer.chrome.com/docs/ai/translator-api">Chrome for Developers — Translation with built-in AI</a></li>
  <li id="ref-14"><a href="https://developer.mozilla.org/en-US/docs/Web/API/Translator">MDN — Translator</a></li>
</ol>

## 無料で運用するための確認点

- Azureの無料枠はサイト全体の利用量で消費する — 同じ本文を閲覧者ごとに翻訳すると、同じ文章でも繰り返し文字数に計上される
- 翻訳結果をサーバー側でも再利用する場合は、原文・翻訳先言語・記事の更新内容を区別して保存する — 原文を更新したら古い翻訳を使い続けない
- APIの文字数上限、短時間の呼び出し制限、サーバー関数の無料枠をそれぞれ確認する — 月の無料文字数が残っていても、一度に大量送信すればエラーになることがある<sup><a href="#ref-15">[15]</a></sup>
- DeepLを使う場合も、APIキーをブラウザーのJavaScriptに埋め込まない — 公式APIはブラウザーからの直接呼び出しを許可しておらず、自分のサーバーを経由させる<sup><a href="#ref-16">[16]</a></sup>
- 外部APIへ送るのは翻訳対象として指定した公開テキストに絞る — 入力中のフォームの値までまとめて送る実装にしない
- 自作のページ内翻訳では `innerHTML` 全体を翻訳結果で置き換えず、対象のテキストを更新する — リンクやボタンの構造・イベントを保つ
- 翻訳結果は機械翻訳であると示し、元の日本語へ戻せるようにする — この例では、ページを再読み込みすると原文に戻る

参照

<ol start="15">
  <li id="ref-15"><a href="https://learn.microsoft.com/en-us/azure/ai-services/translator/service-limits">Microsoft Learn — Translator service limits</a></li>
  <li id="ref-16"><a href="https://developers.deepl.com/docs/best-practices/cors-requests">DeepL Documentation — CORS requests</a></li>
</ol>

## まとめ

- 静的サイトでも、表示中の文章をJavaScriptで切り替えれば、別URLを作らずに翻訳を表示できる
- Google以外にも、Edge・Firefox・Safariの標準翻訳を使う方法がある
- サイト内の翻訳ボタンにはAzure TranslatorのF0など、翻訳APIの無料枠を利用できる
- 外部APIのキーはサーバー側で管理し、静的ページと翻訳APIを呼ぶ処理を分ける
- 新しいDeepL API Developerの無料枠は合計100万文字であり、旧API Freeの月50万文字と区別する
- 静的配信だけで完結させる場合は、ブラウザー標準翻訳、用意した翻訳文の切り替え、対応環境での端末内翻訳を検討する
- 無料枠の使用量とキャッシュを管理し、翻訳に失敗したときも原文を読めるようにする
