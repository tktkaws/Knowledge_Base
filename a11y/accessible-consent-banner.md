# アクセシブルな同意バナーの実装 — フォーカス順と拒否できること

## 同意バナーとは

同意バナーは、Cookie、アクセス解析、広告、外部サービスへのデータ送信などについて説明し、利用者の選択を受け取るUI。Cookieだけでなく、ローカルストレージや端末識別子を使う場合もあるため、この記事では「同意バナー」と呼ぶ。

アクセシブルな同意バナーでは、単にボタンをキーボードで押せるだけでは足りない。

- 何に同意するのかを、選択前に理解できる
- 「すべて許可」と「任意項目を拒否」を同じ階層で選べる
- Tabキーで見た目と一致する順序に移動できる
- バナーが背後のフォーカスを隠さない
- 詳細設定を閉じた後のフォーカス位置を予測できる
- 一度決めた設定を後から変更できる
- 同意前に任意のスクリプトを読み込まない
- 保存に失敗したとき、成功したように見せない

同意を得る法的要件は、地域、データの種類、利用目的、サービスの立場によって異なる。この記事はアクセシビリティとUI実装を扱うものであり、法的助言ではない。実際の文言、分類、保存期間、同意が必要な処理は、対象地域の法令と組織の方針を専門家と確認する。

EUのデータ保護当局で構成されるEDPBのCookie Banner Taskforceは、最初の階層に拒否ボタンがないこと、任意項目が事前選択されていること、誤解を招く色やコントラスト、同意を撤回する手段がないことなどを検討対象としている。適用される法令の判断とは別に、これらは利用者が対等に選べるUIを設計するための重要な確認点になる。

> 参照: [EDPB — Report of the work undertaken by the Cookie Banner Taskforce](https://www.edpb.europa.eu/documents/task-force-report/report-of-the-work-undertaken-by-the-cookie-banner-taskforce_en)

## 最初に選択肢を整理する

一般的な同意バナーでは、次の操作を用意する。

| 操作 | 結果 |
|---|---|
| 任意項目を拒否 | 必須の処理だけを有効にする |
| 設定を選ぶ | 目的ごとの説明と選択肢を開く |
| すべて許可 | 表示したすべての任意項目を有効にする |
| 設定を変更 | 保存後も同じ選択を見直せるようにする |

「拒否」を詳細設定の奥へ隠し、「すべて許可」だけを最初の画面に置くと、操作量が対等にならない。最初の階層で許可できるなら、同じ階層で任意項目を拒否できるようにする。

ボタンの見た目も一方へ誘導しない。

```html
<div class="consent-actions">
  <button type="button" class="consent-choice">
    任意Cookieを拒否
  </button>
  <button type="button">
    設定を選ぶ
  </button>
  <button type="button" class="consent-choice">
    すべて許可
  </button>
</div>
```

- 「許可」だけを大きく、明るくしない
- 「拒否」を小さなテキストリンクにしない
- 「同意しない場合はこちら」のような遠回しな名前を避ける
- 二重否定を使わず、実行結果をボタン名で示す
- 閉じるアイコンを同意として扱わない
- 何も選ばず閉じることと、任意項目を拒否することを混同しない

## 非モーダルとモーダルを使い分ける

同意バナーが表示されていてもページを閲覧できる設計なら、初回の選択領域を非モーダルにする。`section`と通常のボタンで実装し、フォーカストラップを作らない。

目的ごとの説明やチェックボックスを開く「詳細設定」は、利用者が自分で開くモーダルダイアログにできる。

| UI | 適する場面 | フォーカスの扱い |
|---|---|---|
| ページ内の非モーダル領域 | バナー表示中も本文を利用できる | 自動で移動しない、トラップしない |
| モーダルダイアログ | 利用者が詳細設定を開いた | 内部へ移し、閉じるまで内部に保つ |
| ページ全体を遮るモーダル | 背後を本当に操作させない要件がある | 背後を不活性にし、内部だけを操作可能にする |

見た目が画面に重なっているだけで`role="dialog"`や`aria-modal="true"`を付けない。`aria-modal="true"`を付けるなら、ポインター、キーボード、支援技術のすべてで背後を操作できない、本当のモーダルとして動作させる。

WAI-ARIA APGのモーダルダイアログパターンでは、開いたときに内部へフォーカスを移し、Tabキーの移動を内部に保ち、Escapeキーで閉じ、閉じた後は原則として開いた要素へフォーカスを戻すとしている。

> 参照: [WAI-ARIA APG — Dialog (Modal) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

## 基本のHTML構造

初回のバナーをヘッダーの後、メインコンテンツの前に置く例。

```html
<header>
  <!-- サイトのヘッダー -->
</header>

<section
  id="consent-banner"
  class="consent-banner"
  aria-labelledby="consent-title"
>
  <div class="consent-banner__inner">
    <div>
      <h2 id="consent-title">プライバシー設定</h2>
      <p>
        サイトの動作に必要なCookieに加えて、アクセス解析と広告の
        Cookieを使用するか選べます。
      </p>
      <p>
        <a href="/privacy/cookies">Cookieの利用目的を確認する</a>
      </p>
    </div>

    <div class="consent-actions">
      <button id="consent-reject" type="button" class="consent-choice">
        任意Cookieを拒否
      </button>
      <button
        id="consent-customize"
        type="button"
        aria-haspopup="dialog"
        aria-controls="consent-settings"
      >
        設定を選ぶ
      </button>
      <button id="consent-accept" type="button" class="consent-choice">
        すべて許可
      </button>
    </div>
  </div>
</section>

<main id="main-content" tabindex="-1">
  <h1>ページのタイトル</h1>
  <!-- ページの本文 -->
</main>

<p
  id="consent-status"
  class="visually-hidden"
  role="status"
  aria-atomic="true"
></p>

<footer>
  <button id="privacy-settings" type="button">
    プライバシー設定を変更
  </button>
</footer>
```

### `section`を使う理由

見出しを持つ`section`なら、ページ内の名前付き領域として内容を理解できる。初回バナーは緊急警告ではないため、`role="alert"`を付けて強制的に割り込ませない。

サーバーが最初のHTMLにバナーを含めれば、スクリーンリーダーの読み上げ順でもヘッダーの後、本文の前に現れる。ページを読んでいる途中で遅れて挿入し、現在の読み上げを突然中断させる実装を避ける。

### フォーカスを自動で移さない

非モーダルなバナーでは、ページ読み込み時に`focus()`しない。利用者がアドレスバー、スキップリンク、見出しなどから操作を始めようとしているとき、同意ボタンへ強制的に移すと予測できない。

HTMLの並びを次の順にすれば、通常のTab順序も同じになる。

1. 「任意Cookieを拒否」
2. 「設定を選ぶ」
3. 「すべて許可」
4. メインコンテンツ内の操作要素
5. フッターの「プライバシー設定を変更」

CSSの`order`で見た目だけを並べ替えない。`tabindex="1"`のような正の値で順序を修正せず、DOMの順序そのものを論理的にする。

> 参照: [Understanding WCAG 2.2 SC 2.4.3 — Focus Order](https://www.w3.org/WAI/WCAG22/Understanding/focus-order.html)

## 固定バナーでフォーカスを隠さない

画面下部に`position: fixed`で置いたバナーは、背後のリンクやボタンを覆うことがある。Tabキーで背後へ移動できても、フォーカスされた要素がバナーに完全に隠れると、WCAG 2.2の「2.4.11 フォーカスが隠れない（最低限）」に抵触する。

W3Cの解説でも、固定されたCookieバナーがフォーカス対象を完全に覆う例が示されている。対策には次の選択肢がある。

- バナーを通常フローに置き、本文を覆わない
- バナーの高さ分だけ本文末尾へ余白を確保する
- `scroll-padding`を設定し、フォーカス対象がバナーの背後へスクロールしないようにする
- 本当にモーダルなら、正しくフォーカスを内部へ閉じ込める

> 参照: [Understanding WCAG 2.2 SC 2.4.11 — Focus Not Obscured (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html)

固定表示が必須でなければ、通常フローが最も単純。固定する場合は、表示中の高さを測って余白へ反映する。

```css
:root {
  --consent-banner-offset: 0px;
}

html {
  scroll-padding-block-end: calc(var(--consent-banner-offset) + 1rem);
}

body.has-consent-banner {
  padding-block-end: var(--consent-banner-offset);
}

.consent-banner {
  position: fixed;
  z-index: 1000;
  inset-inline: 0;
  inset-block-end: 0;
  max-block-size: min(80dvh, 40rem);
  overflow: auto;
  border-block-start: 0.125rem solid currentColor;
  background: Canvas;
  color: CanvasText;
}

.consent-banner[hidden] {
  display: none;
}
```

```js
const banner = document.querySelector('#consent-banner');

function updateBannerOffset() {
  const height = banner.hidden
    ? 0
    : Math.ceil(banner.getBoundingClientRect().height);

  document.documentElement.style.setProperty(
    '--consent-banner-offset',
    `${height}px`,
  );
  document.body.classList.toggle('has-consent-banner', height > 0);
}

const bannerObserver = new ResizeObserver(updateBannerOffset);
bannerObserver.observe(banner);
window.addEventListener('resize', updateBannerOffset);
updateBannerOffset();
```

テキスト拡大、翻訳、狭い画面ではバナーの高さが変わるため、固定値の`padding-bottom`だけに依存しない。`ResizeObserver`で実際の高さを反映する。

バナー自体が画面より高くなる可能性もある。`max-block-size`と`overflow: auto`を指定し、200%・400%拡大でもすべての説明とボタンへ到達できるようにする。

## 選択肢を見やすくするCSS

```css
.consent-banner__inner {
  inline-size: min(100% - 2rem, 75rem);
  margin-inline: auto;
  padding-block: 1rem;
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  gap: 1.5rem;
  align-items: center;
}

.consent-actions {
  display: grid;
  grid-template-columns: repeat(3, minmax(10rem, 1fr));
  gap: 0.75rem;
}

.consent-actions button,
#privacy-settings,
.consent-dialog button {
  min-block-size: 2.75rem;
  padding: 0.625rem 1rem;
  border: 0.125rem solid currentColor;
  border-radius: 0.375rem;
  background: Canvas;
  color: CanvasText;
  font: inherit;
}

.consent-actions .consent-choice {
  background: ButtonFace;
  color: ButtonText;
}

.consent-actions button:focus-visible,
#privacy-settings:focus-visible,
.consent-dialog button:focus-visible,
.consent-dialog input:focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

@media (max-width: 60rem) {
  .consent-banner__inner {
    grid-template-columns: 1fr;
  }

  .consent-actions {
    grid-template-columns: 1fr;
  }
}

@media (forced-colors: active) {
  .consent-banner,
  .consent-actions button,
  .consent-dialog {
    border-color: CanvasText;
  }
}
```

- 許可と拒否に同じ`.consent-choice`を使い、視覚的な強さをそろえる
- ボタン名を省略せず、折り返しを許可する
- 狭い画面ではDOM順序を保ったまま縦に並べる
- フォーカス表示を消さない
- 強制カラーモードでも境界を確認できるようにする
- 色だけで選択肢の重要度や状態を区別しない

## 詳細設定はネイティブの`dialog`を使う

利用者が「設定を選ぶ」を押した後は、目的ごとの説明をモーダルダイアログに表示する。

```html
<dialog
  id="consent-settings"
  class="consent-dialog"
  aria-labelledby="consent-settings-title"
>
  <form id="consent-settings-form">
    <h2 id="consent-settings-title" tabindex="-1">
      Cookieの詳細設定
    </h2>

    <p>
      任意Cookieは初期状態でオフです。目的ごとに変更できます。
    </p>

    <section aria-labelledby="necessary-title">
      <h3 id="necessary-title">必須Cookie</h3>
      <p>
        セッション維持や同意設定の保存に必要です。常に使用します。
      </p>
    </section>

    <fieldset>
      <legend>任意Cookie</legend>

      <label>
        <input id="consent-analytics" type="checkbox" name="analytics" />
        <span>
          <strong>アクセス解析</strong><br />
          ページの利用状況を集計し、改善に使用します。
        </span>
      </label>

      <label>
        <input id="consent-advertising" type="checkbox" name="advertising" />
        <span>
          <strong>広告</strong><br />
          関心に合わせた広告の表示と効果測定に使用します。
        </span>
      </label>
    </fieldset>

    <p
      id="consent-dialog-status"
      class="consent-dialog__status"
      role="status"
      aria-atomic="true"
    ></p>

    <div class="consent-dialog__actions">
      <button id="consent-settings-close" type="button">
        変更せず閉じる
      </button>
      <button type="submit">選択した設定を保存</button>
    </div>
  </form>
</dialog>
```

### 必須項目を無効なチェックボックスにしない

必須Cookieが変更できないなら、`disabled`のチェック済みチェックボックスで選択肢のように見せず、見出しと説明で示す。操作できる任意項目だけをチェックボックスにする。

```html
<!-- 避ける：変更できそうに見える -->
<label>
  <input type="checkbox" checked disabled />
  必須Cookie
</label>
```

### 任意項目を事前に選択しない

初めて開いたとき、任意項目のチェックボックスはオフにする。保存済みの設定を後から開いたときだけ、現在の選択状態を復元する。

### ダイアログのタイトルへ初期フォーカスを置く

詳細設定には複数の説明と選択肢がある。最初のチェックボックスだけへフォーカスすると、その前の説明を読み飛ばす可能性があるため、タイトルへ`tabindex="-1"`を付け、開いた直後だけプログラムでフォーカスする。

`tabindex="-1"`は通常のTab順序には追加せず、`focus()`の移動先にできる。ダイアログコンテナ自体へフォーカスを置かない。

## 設定を開閉するJavaScript

ネイティブの`showModal()`を使うと、ダイアログ外は不活性になり、Tabキーの移動もダイアログ内に保たれる。対応環境と製品要件を確認し、古い環境を対象にする場合は十分にテストしたポリフィルを使う。

```js
const settingsDialog = document.querySelector('#consent-settings');
const settingsTitle = document.querySelector('#consent-settings-title');
const customizeButton = document.querySelector('#consent-customize');
const privacySettingsButton = document.querySelector('#privacy-settings');
const closeSettingsButton = document.querySelector('#consent-settings-close');
const analyticsCheckbox = document.querySelector('#consent-analytics');
const advertisingCheckbox = document.querySelector('#consent-advertising');

let dialogOpener = null;
let savedPreferences = null;

function fillSettingsForm(preferences) {
  analyticsCheckbox.checked = preferences?.analytics ?? false;
  advertisingCheckbox.checked = preferences?.advertising ?? false;
}

function openSettings(opener) {
  dialogOpener = opener;
  fillSettingsForm(savedPreferences);
  settingsDialog.showModal();
  settingsTitle.focus();
}

function closeSettingsWithoutSaving() {
  settingsDialog.close();
  dialogOpener?.focus();
}

customizeButton.addEventListener('click', () => {
  openSettings(customizeButton);
});

privacySettingsButton.addEventListener('click', () => {
  openSettings(privacySettingsButton);
});

closeSettingsButton.addEventListener('click', closeSettingsWithoutSaving);

settingsDialog.addEventListener('cancel', (event) => {
  event.preventDefault();
  closeSettingsWithoutSaving();
});
```

`cancel`イベントはEscapeキーで発生する。`preventDefault()`して共通の閉じる処理を呼ぶことで、フォーカスの戻り先を明示している。

閉じるボタンには「×」だけでなく、「変更せず閉じる」のように結果が分かる名前を付ける。アイコンを使う場合も、見えるテキストまたは適切なアクセシブルネームを用意する。

## 設定を保存する

次の例では、説明を簡潔にするため`localStorage`へ保存する。実際には、サーバーでの同意記録、複数ドメイン、ログイン状態との同期、保存期間、同意文面のバージョン管理が必要になる場合がある。

```js
const STORAGE_KEY = 'consent-preferences-v1';
const acceptButton = document.querySelector('#consent-accept');
const rejectButton = document.querySelector('#consent-reject');
const settingsForm = document.querySelector('#consent-settings-form');
const mainContent = document.querySelector('#main-content');
const status = document.querySelector('#consent-status');
const dialogStatus = document.querySelector('#consent-dialog-status');

function isPreferences(value) {
  return Boolean(
    value
      && typeof value === 'object'
      && typeof value.analytics === 'boolean'
      && typeof value.advertising === 'boolean',
  );
}

function readPreferences() {
  try {
    const value = JSON.parse(localStorage.getItem(STORAGE_KEY));
    return isPreferences(value) ? value : null;
  } catch {
    return null;
  }
}

function writePreferences(preferences) {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(preferences));
    return true;
  } catch {
    return false;
  }
}

function hideBanner() {
  banner.hidden = true;
  updateBannerOffset();
}

function makePreferences({ analytics, advertising }) {
  return {
    necessary: true,
    analytics,
    advertising,
    updatedAt: new Date().toISOString(),
  };
}

function savePreferences(nextPreferences, focusTarget) {
  if (!writePreferences(nextPreferences)) {
    const message =
      '設定を保存できませんでした。ブラウザの保存設定を確認して、もう一度お試しください。';

    if (settingsDialog.open) {
      dialogStatus.textContent = message;
    } else {
      status.textContent = message;
    }
    return false;
  }

  savedPreferences = nextPreferences;
  applyPreferences(savedPreferences);
  dialogStatus.textContent = '';

  if (settingsDialog.open) {
    settingsDialog.close();
  }

  hideBanner();
  status.textContent = 'プライバシー設定を保存しました。';
  focusTarget?.focus();
  return true;
}

rejectButton.addEventListener('click', () => {
  savePreferences(
    makePreferences({ analytics: false, advertising: false }),
    mainContent,
  );
});

acceptButton.addEventListener('click', () => {
  savePreferences(
    makePreferences({ analytics: true, advertising: true }),
    mainContent,
  );
});

settingsForm.addEventListener('submit', (event) => {
  event.preventDefault();

  const nextPreferences = makePreferences({
    analytics: analyticsCheckbox.checked,
    advertising: advertisingCheckbox.checked,
  });

  const openedFromBanner = dialogOpener === customizeButton;
  const focusTarget = openedFromBanner ? mainContent : dialogOpener;
  savePreferences(nextPreferences, focusTarget);
});

savedPreferences = readPreferences();

if (savedPreferences) {
  hideBanner();
  applyPreferences(savedPreferences);
} else {
  updateBannerOffset();
}
```

この例では、バナー内のボタンを押した後にバナーを非表示にする。フォーカス中のボタンを消すとフォーカスが`body`へ失われることがあるため、次の論理的な位置である`main`へ移している。

`main`の`tabindex="-1"`はプログラムによる移動先のためであり、通常のTab停止位置は増やさない。

フッターの「プライバシー設定を変更」から開いた場合は、保存後にそのボタンへ戻す。ダイアログを開いた要素が存在しなくなる場合だけ、ワークフロー上の次の位置へ移すというAPGの原則に沿っている。

### 保存失敗を成功扱いしない

ブラウザ設定、容量制限、プライベートモードなどにより、保存APIが失敗する可能性がある。失敗時は次を行う。

- バナーまたはダイアログを閉じない
- 選択したチェック状態を残す
- バナーからの操作はページ側、モーダルからの操作はダイアログ内部の`role="status"`で失敗を伝える
- 再試行方法を示す
- 保存していない任意処理を有効にしない

モーダル表示中は背後のライブリージョンも不活性になるため、ダイアログ内で発生した失敗をダイアログ外の`role="status"`だけへ書き込まない。エラーが詳しい確認を必要とする場合は、問題のある領域とも関連付ける。

## 同意前に任意スクリプトを読み込まない

バナーだけを先に表示しても、その背後ですでにアクセス解析や広告スクリプトを読み込んでいれば選択が機能していない。

```html
<!-- 避ける：同意状態を確認する前に読み込まれる -->
<script src="/assets/analytics.js"></script>
```

同意状態を確認してから、必要な処理だけを開始する。

```js
function loadScriptOnce(src, category) {
  if (document.querySelector(`script[data-consent-category="${category}"]`)) {
    return;
  }

  const script = document.createElement('script');
  script.src = src;
  script.async = true;
  script.dataset.consentCategory = category;
  document.head.append(script);
}

function disableAnalytics() {
  // ベンダーが提供する同意撤回APIやCookie削除処理を呼ぶ。
}

function disableAdvertising() {
  // 新しい送信を止め、必要に応じて保存済み識別子を削除する。
}

function applyPreferences(preferences) {
  if (preferences.analytics) {
    loadScriptOnce('/assets/analytics.js', 'analytics');
  } else {
    disableAnalytics();
  }

  if (preferences.advertising) {
    loadScriptOnce('/assets/advertising.js', 'advertising');
  } else {
    disableAdvertising();
  }
}
```

一度読み込んだ`script`要素をDOMから削除しても、すでに実行されたコード、送信済みリクエスト、作成済みCookieは元に戻らない。設定変更時は、利用するサービスの同意API、停止方法、Cookie削除方法を確認する。次のページ読み込みから反映する設計なら、そのことを利用者へ明示する。

必須処理と任意処理の分類は、フロントエンドの都合だけで決めない。「すでに読み込んでいるから必須」とはならない。

## 後から変更できる導線を残す

同意後にバナーを消したまま、設定変更手段まで消さない。フッターなど、サイト全体で一貫した場所に「プライバシー設定を変更」ボタンを置く。

```html
<footer>
  <nav aria-label="フッター">
    <ul>
      <li><a href="/privacy">プライバシーポリシー</a></li>
      <li><a href="/privacy/cookies">Cookieの利用について</a></li>
    </ul>
  </nav>

  <button id="privacy-settings" type="button">
    プライバシー設定を変更
  </button>
</footer>
```

ページを移動する文書は`a`、現在のページでダイアログを開く操作は`button`にする。すべてを「Cookie」という言葉だけで表さず、利用者が目的を予測できる名前にする。

保存済み設定を開いたら、現在の値をチェックボックスへ反映する。変更せず閉じた場合は保存値を変えない。変更を保存したら、新しい設定と実際の処理状態を一致させる。

## 閉じる操作の意味を決める

右上の「×」でバナーを閉じられるデザインでは、閉じた結果を明確にする必要がある。

```html
<!-- 意味が不明確 -->
<button type="button" aria-label="閉じる">×</button>
```

「×」を押しただけで「すべて許可」と記録してはいけない。設計として閉じる操作を用意するなら、少なくとも次を決める。

- 未選択のまま一時的に閉じるのか
- 任意項目を拒否した状態として保存するのか
- 次回いつ再表示するのか
- 閉じた後も設定導線をどこへ残すのか

拒否として扱うなら、ボタン名も「任意Cookieを拒否して閉じる」のように結果を示す。単なる「閉じる」を、見えない同意や拒否へ変換しない。

## Reactで実装する

表示と保存処理を分離した最小例。親コンポーネントが保存の成功・失敗と任意スクリプトの制御を担当する。

```jsx
function ConsentBanner({ onAcceptAll, onRejectAll, onCustomize }) {
  return (
    <section
      className="consent-banner"
      aria-labelledby="consent-title"
    >
      <div className="consent-banner__inner">
        <div>
          <h2 id="consent-title">プライバシー設定</h2>
          <p>
            サイトの動作に必要なCookieに加えて、アクセス解析と広告の
            Cookieを使用するか選べます。
          </p>
          <a href="/privacy/cookies">Cookieの利用目的を確認する</a>
        </div>

        <div className="consent-actions">
          <button
            type="button"
            className="consent-choice"
            onClick={onRejectAll}
          >
            任意Cookieを拒否
          </button>
          <button
            type="button"
            aria-haspopup="dialog"
            aria-controls="consent-settings"
            onClick={onCustomize}
          >
            設定を選ぶ
          </button>
          <button
            type="button"
            className="consent-choice"
            onClick={onAcceptAll}
          >
            すべて許可
          </button>
        </div>
      </div>
    </section>
  );
}
```

条件付きレンダリングでバナーを消す場合も、クリックされたボタンと一緒にフォーカスが消える。保存成功後に、`ref`で保持した`main`など論理的な次の位置へフォーカスを移す。

```jsx
const mainRef = useRef(null);

async function rejectAll() {
  const saved = await saveConsent({
    analytics: false,
    advertising: false,
  });

  if (!saved) {
    setStatus('設定を保存できませんでした。もう一度お試しください。');
    return;
  }

  setShowBanner(false);
  setStatus('プライバシー設定を保存しました。');

  requestAnimationFrame(() => {
    mainRef.current?.focus();
  });
}

return (
  <>
    {showBanner && (
      <ConsentBanner
        onRejectAll={rejectAll}
        onAcceptAll={acceptAll}
        onCustomize={openSettings}
      />
    )}
    <main ref={mainRef} tabIndex={-1}>
      {/* ページの本文 */}
    </main>
    <p role="status" aria-atomic="true">{status}</p>
  </>
);
```

初期レンダリング後に同意状態を取得する構成では、任意スクリプトを先に実行しない。サーバーレンダリング、ハイドレーション、タグマネージャーを含め、同意状態が確定するまで任意処理を保留する。

## よくある失敗

### 「すべて許可」しか最初の画面にない

同じ階層に「任意Cookieを拒否」を置き、同程度の操作量と見つけやすさにする。

### 拒否だけが小さなリンクになっている

許可と拒否を同等のボタンとして示す。色、サイズ、コントラストで一方だけを強調しない。

### 任意項目が最初からチェック済み

初回はオフにする。保存済み設定を再編集するときだけ現在値を復元する。

### 非モーダルなのにフォーカスを閉じ込める

ページを利用できる設計なら、バナー外へTabキーで移動できるようにする。キーボードだけ背後へ移動できず、ポインターでは操作できる状態を作らない。

### モーダルなのに背後を操作できる

`aria-modal="true"`だけを追加しない。`dialog.showModal()`などを使い、すべての入力方法で背後を不活性にする。

### ページ読み込み時にバナーへフォーカスする

非モーダルなら自動で移さない。DOMの早い位置に置き、利用者が通常の読み上げとTab順序で到達できるようにする。

### 固定バナーが背後のフォーカスを隠す

通常フローへ戻すか、実際の高さに合わせた余白と`scroll-padding`を確保する。拡大と狭い画面でも確認する。

### バナーを消してフォーカスも失う

選択後に、メインコンテンツなど次の論理的な位置へフォーカスを移す。設定ダイアログを閉じるだけなら、開いたボタンへ戻す。

### `role="alert"`でバナー全体を読み上げる

通常の同意確認は緊急警告ではない。見出し付きの領域としてHTMLへ含め、保存結果など短い変更だけを`role="status"`で伝える。

### 同意前にスクリプトを読み込む

初期HTML、タグマネージャー、サードパーティー埋め込みを含め、任意処理の開始条件を確認する。

### 設定を後から変更できない

全ページで見つけられる場所に、同じ名前の設定ボタンを残す。現在値を表示し、許可と拒否の両方を変更できるようにする。

### 保存失敗でもバナーを閉じる

失敗を見える形とステータスメッセージで伝え、再試行できる状態を保つ。

## テストのチェックリスト

### 内容と選択肢

- [ ] 何のためにデータを使うかを選択前に説明している
- [ ] 必須項目と任意項目を区別している
- [ ] 最初の階層に「すべて許可」と「任意項目を拒否」がある
- [ ] 許可と拒否の操作量が同程度である
- [ ] 許可だけを色、サイズ、位置で過度に強調していない
- [ ] 任意項目が初期状態でオフになっている
- [ ] ボタン名から実行結果を予測できる
- [ ] 閉じる操作を同意として扱っていない

### キーボードとフォーカス

- [ ] TabキーでDOMと見た目が一致する順序に移動できる
- [ ] すべてのボタンをEnterキーとSpaceキーで操作できる
- [ ] 非モーダルなバナーでフォーカスを閉じ込めていない
- [ ] ページ読み込み時に非モーダルなバナーへ強制移動していない
- [ ] 詳細設定を開くとダイアログ内部へフォーカスが移る
- [ ] モーダル内でTabキーとShift + Tabキーが循環する
- [ ] Escapeキーで変更せず詳細設定を閉じられる
- [ ] 閉じた後、開いたボタンまたは次の論理的な位置へ戻る
- [ ] バナーを非表示にした後、フォーカスが失われない
- [ ] 正の`tabindex`で順序を作っていない

### 表示と拡大

- [ ] 固定バナーが背後のフォーカス対象を完全に隠さない
- [ ] バナーの高さ変更を本文余白と`scroll-padding`へ反映している
- [ ] 200%・400%拡大ですべての説明と操作を利用できる
- [ ] 狭い画面でボタンが重ならず、横スクロールを強制しない
- [ ] バナーが画面より高い場合に内部をスクロールできる
- [ ] フォーカスインジケーターが背景と十分に区別できる
- [ ] 強制カラーモードでも境界、選択、フォーカスが分かる
- [ ] ボタンの押下範囲が十分にあり、隣と重ならない

### スクリーンリーダー

- [ ] バナーの見出しと説明を読み上げ順で理解できる
- [ ] バナーを不要な`alert`として割り込ませていない
- [ ] 各ボタンの名前が見える文言と一致している
- [ ] 詳細設定が名前のあるダイアログとして伝わる
- [ ] チェックボックスの名前、状態、説明が伝わる
- [ ] 必須項目を操作できる選択肢のように見せていない
- [ ] 保存成功と失敗を短いステータスメッセージで確認できる
- [ ] 設定変更ボタンをランドマークやボタン一覧から見つけられる

### 状態とデータ処理

- [ ] 同意状態を確認する前に任意スクリプトを読み込んでいない
- [ ] 拒否時に任意の通信と保存を開始しない
- [ ] 保存した値と実際に有効な処理が一致している
- [ ] 設定を再び開くと現在値が復元される
- [ ] 変更せず閉じても保存値が変わらない
- [ ] 設定変更時に停止処理と識別子削除を確認している
- [ ] 保存失敗時にバナーまたはダイアログを閉じない
- [ ] 保存期間と同意文面のバージョン変更を考慮している
- [ ] 後から設定を変更する導線が全ページで一貫している

### 入力方法と実機確認

- [ ] マウス、タッチ、キーボードで同じ選択ができる
- [ ] 音声入力で見えるボタン名を使って操作できる
- [ ] スクリーンリーダーとブラウザの組み合わせで確認した
- [ ] JavaScriptエラー時に任意処理が許可状態へ倒れない
- [ ] 低速回線でもバナー表示前に任意処理が始まらない
- [ ] ページ遷移後も設定と処理状態が一致する

## 関連して読むとよいテーマ

- [アクセシブルなモーダルダイアログの実装](./accessible-modal-dialog.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカスインジケーターのカスタマイズ — :focus-visibleの活用](./focus-visible.md)
- [アクセシブルなフォームの設計 — エラー表示とバリデーション](./accessible-form.md)
- [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)
- [手動テストのチェックリスト — 最低限確認すべき項目](./manual-testing-checklist.md)

## まとめ

- 初回バナーを非モーダルにするなら、`section`と通常のボタンで実装する
- ページを利用できる設計では、バナーへフォーカスを閉じ込めない
- 最初の階層に許可、拒否、詳細設定を置き、拒否を隠さない
- 許可と拒否の文言、サイズ、コントラスト、操作量を対等にする
- DOM順序と見た目の順序を一致させ、正の`tabindex`を使わない
- 固定バナーの高さを本文余白と`scroll-padding`へ反映し、背後のフォーカスを隠さない
- 詳細設定は本当のモーダルとして実装し、内部への移動、Escape、フォーカス復帰を管理する
- 任意項目は初期状態でオフにし、必須項目を変更可能な選択肢に見せない
- バナーを消すときは、消えるボタンから次の論理的な位置へフォーカスを移す
- 同意状態を確認する前に任意スクリプトを読み込まない
- 保存失敗時はUIを閉じず、失敗と再試行方法を伝える
- フッターなど一貫した場所に設定変更の導線を残す
