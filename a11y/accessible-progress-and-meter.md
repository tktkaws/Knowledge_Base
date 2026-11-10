# アクセシブルなプログレス表示の実装 — progress と meter の違い

## プログレス表示とは

プログレス表示は、処理が進行中であることや、現在どこまで完了したかを伝えるUI。ファイルアップロード、データ変換、検索、複数ステップの処理などに使われる。

似た横棒の見た目でも、タスクの進捗と測定値では意味が異なる。

- タスクの完了状況は`progress`
- 既知の範囲内にある現在の測定値は`meter`
- 値が分からない待機状態は不確定な`progress`
- 短い状態メッセージは`role="status"`

通常はネイティブの`progress`と`meter`を使う。ブラウザが役割、現在値、最小値、最大値を支援技術へ公開するため、見た目を作る目的だけで`div`へ置き換えない。

> 参照:
> - [WHATWG HTML Standard — The `progress` element](https://html.spec.whatwg.org/multipage/form-elements.html#the-progress-element)
> - [WHATWG HTML Standard — The `meter` element](https://html.spec.whatwg.org/multipage/form-elements.html#the-meter-element)

## `progress`と`meter`の違い

| 要素 | 表すもの | 例 |
|---|---|---|
| `progress` | タスクがどこまで完了したか | アップロード、変換、インストール |
| `meter` | 既知範囲内の現在値 | ディスク使用量、バッテリー、スコア |
| `output` | 計算や操作の結果 | 合計金額、計算結果 |
| `role="status"` | フォーカスを動かさず伝える状態変化 | 「保存しました」「25%完了」 |

### `progress`を使う

```text
ファイルをアップロード中
[████████░░░░░░░░] 50%
```

処理の開始から完了までを表す。

### `meter`を使う

```text
ストレージ使用量
[████████████░░░░] 75 GB / 100 GB
```

タスクが進んでいるのではなく、現在の状態を測っている。

### 間違えやすい例

```html
<!-- 避ける：ディスク使用量はタスクの進捗ではない -->
<progress value="75" max="100">75%</progress>

<!-- 適切：既知範囲内の測定値 -->
<meter value="75" min="0" max="100">75%</meter>
```

```html
<!-- 避ける：ファイル処理の完了率は測定ゲージではない -->
<meter value="40" min="0" max="100">40%</meter>

<!-- 適切：タスクの完了状況 -->
<progress value="40" max="100">40%</progress>
```

## 値が分かる`progress`

現在値と全体量が分かる場合は、`value`と`max`を指定する。

```html
<div class="progress-field">
  <label for="upload-progress">画像をアップロード</label>
  <progress
    id="upload-progress"
    value="35"
    max="100"
  >
    35%
  </progress>
  <output id="upload-progress-value" for="upload-progress">35%</output>
</div>
```

### `progress`の主な属性

| 属性 | 意味 |
|---|---|
| `value` | 完了した量。0以上、`max`以下 |
| `max` | タスク全体の量。0より大きい値 |

`min`は指定しない。`progress`の最小値は常に`0`。

`max`を省略した場合の最大値は`1`だが、コードと表示を理解しやすくするため、パーセントなら`max="100"`を明示する。

要素内の`35%`はフォールバックテキスト。モダンブラウザではプログレスバー上に表示されるとは限らないため、正確な値を常に見せたい場合は外側にもテキストを置く。

### パーセント以外の単位

`value`と`max`の単位は任意。処理済み件数をそのまま指定できる。

```html
<label for="import-progress">顧客データを取り込み中</label>
<progress id="import-progress" value="240" max="1000">
  240 / 1000件
</progress>
<span>240 / 1000件</span>
```

支援技術は割合として伝える場合がある。件数が重要なら、見えるテキストでも現在件数と総件数を示す。

## 値が分からない`progress`

処理中だが残り時間や完了量が分からない場合は、`value`属性を付けない。

```html
<div class="progress-field">
  <label for="search-progress">検索しています</label>
  <progress id="search-progress" max="100">
    検索しています
  </progress>
  <p role="status">検索しています。しばらくお待ちください。</p>
</div>
```

不確定状態は`value="0"`ではない。

```html
<!-- 不確定：どこまで進んだか分からない -->
<progress></progress>

<!-- 確定：0%であることが分かっている -->
<progress value="0" max="100"></progress>
```

JavaScriptでは`value`へ空文字列を代入するのではなく、属性を削除する。

```js
const progress = document.getElementById('search-progress');

// 不確定状態にする
progress.removeAttribute('value');

// 全体量が判明したら確定状態へ切り替える
progress.max = 100;
progress.value = 20;
```

`progress.value = progress.value`のようにプロパティへ値を代入すると、属性がなかった不確定状態が0%の確定状態へ変わることがある。状態の切り替えを明示的に扱う。

## `progress`へ名前を付ける

### `label`を使う

`progress`はラベル付けできる要素。

```html
<label for="video-progress">動画を変換</label>
<progress id="video-progress" value="65" max="100">65%</progress>
```

### 見出しを参照する

```html
<h2 id="backup-heading">バックアップを作成</h2>
<progress
  value="3"
  max="5"
  aria-labelledby="backup-heading"
>
  3 / 5
</progress>
```

同じ画面に複数の処理がある場合、「進捗」だけでは区別できない。「画像をアップロード」「動画を変換」のように対象と処理を含む名前にする。

`progress`自体は操作コントロールではないため、通常は`tabindex="0"`を付けない。現在値を確認するためだけにTab停止位置を増やさない。

## JavaScriptで進捗を更新する

```html
<section
  id="upload-region"
  aria-labelledby="upload-heading"
  aria-busy="true"
>
  <h2 id="upload-heading">画像をアップロード</h2>

  <progress
    id="file-progress"
    max="100"
    value="0"
    aria-labelledby="upload-heading"
  >
    0%
  </progress>
  <output id="file-progress-value" for="file-progress">0%</output>
  <p id="file-progress-status" role="status"></p>
</section>
```

```js
class ProgressReporter {
  constructor({ region, progress, output, status }) {
    this.region = region;
    this.progress = progress;
    this.output = output;
    this.status = status;
    this.lastAnnouncedMilestone = 0;
  }

  setIndeterminate(message = '処理しています。') {
    this.region.setAttribute('aria-busy', 'true');
    this.progress.removeAttribute('value');
    this.output.value = '処理中';
    this.status.textContent = message;
  }

  setProgress(completed, total) {
    if (!Number.isFinite(completed) || !Number.isFinite(total)) return;
    if (total <= 0) return;

    const safeCompleted = Math.min(Math.max(completed, 0), total);
    const percent = Math.round((safeCompleted / total) * 100);

    this.region.setAttribute('aria-busy', 'true');
    this.progress.max = total;
    this.progress.value = safeCompleted;
    this.output.value = `${percent}%`;

    if (percent < this.lastAnnouncedMilestone) {
      this.lastAnnouncedMilestone = 0;
    }

    const milestone = Math.floor(percent / 25) * 25;
    if (
      milestone > this.lastAnnouncedMilestone &&
      milestone < 100
    ) {
      this.status.textContent = `${milestone}%完了しました。`;
      this.lastAnnouncedMilestone = milestone;
    }
  }

  complete(message = 'アップロードが完了しました。') {
    this.progress.value = this.progress.max;
    this.output.value = '100%';
    this.region.setAttribute('aria-busy', 'false');
    this.status.textContent = message;
    this.lastAnnouncedMilestone = 100;
  }

  fail(message = 'アップロードに失敗しました。') {
    this.region.setAttribute('aria-busy', 'false');
    this.status.textContent = message;
  }
}

const uploadProgress = new ProgressReporter({
  region: document.getElementById('upload-region'),
  progress: document.getElementById('file-progress'),
  output: document.getElementById('file-progress-value'),
  status: document.getElementById('file-progress-status'),
});

// 通信処理などから更新する
uploadProgress.setProgress(35, 100);
```

### 更新頻度を分ける

見えるバーと値は細かく更新しても、ライブリージョンで1%ごとに読み上げると操作を妨げる。例では25%ごとの節目と完了だけを`role="status"`へ送っている。

通知の間隔は処理時間と利用者の判断に必要な粒度から決める。

- 数秒で終わる処理は開始と完了だけでもよい
- 長時間処理は一定時間または意味のある節目で通知する
- 同じ文言を何度も追加しない
- 完了時にフォーカスを勝手に移動しない

> 参照: [Understanding WCAG 2.2 SC 4.1.3 — Status Messages](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html)

## `aria-busy`で更新領域を示す

```html
<section id="results" aria-busy="true" aria-labelledby="results-heading">
  <h2 id="results-heading">検索結果</h2>
  <progress aria-label="検索結果を読み込み中"></progress>
</section>
```

`aria-busy="true"`は、その領域が更新中であることを支援技術へ伝える。処理が終わったら`false`へ戻す。

```js
document.getElementById('results').setAttribute('aria-busy', 'false');
```

`aria-busy`だけでは、何をしているかや完了したことを十分に伝えられない場合がある。見える説明、名前のある`progress`、必要な状態メッセージを組み合わせる。

更新中だからという理由で、領域内のすべての操作を無効にしない。キャンセルや別画面への移動が可能なら、利用者が処理を中断できる手段を残す。

## キャンセルと再試行

長時間処理では、進捗表示だけでなく操作も必要になる。

```html
<div aria-labelledby="export-heading">
  <h2 id="export-heading">データを書き出しています</h2>
  <progress value="40" max="100" aria-labelledby="export-heading">
    40%
  </progress>
  <span>40%</span>
  <button type="button">キャンセル</button>
</div>
```

- キャンセルボタンは通常の`button`にする
- キャンセル後は処理が止まったことを伝える
- 失敗した場合は理由と再試行ボタンを示す
- 再試行後は古いエラーメッセージを整理する
- ボタンを無効にする場合は、実際の処理状態と同期する

進捗バー自体へクリックやキーボード操作を追加しない。時間位置を操作するUIはスライダーなど別のパターンになる。

## `meter`の基本

`meter`は、既知の最小値と最大値の間にある現在の測定値を表す。

```html
<div class="meter-field">
  <label for="storage-meter">ストレージ使用量</label>
  <meter
    id="storage-meter"
    min="0"
    max="100"
    low="60"
    high="85"
    optimum="0"
    value="72"
  >
    72 GB / 100 GB
  </meter>
  <span>72 GB / 100 GB</span>
</div>
```

### `meter`の主な属性

| 属性 | 意味 |
|---|---|
| `value` | 現在の測定値 |
| `min` | 範囲の最小値。省略時は0 |
| `max` | 範囲の最大値。省略時は1 |
| `low` | 低い範囲の上限 |
| `high` | 高い範囲の下限 |
| `optimum` | 最適と考える値 |

この例では使用量が少ない方が望ましいため、`optimum="0"`としている。`low`、`high`、`optimum`の関係から、ブラウザは値が最適・準最適・非最適のどの領域にあるかを表現できる。

ブラウザの色だけに「正常」「警告」「危険」の意味を任せない。値、単位、必要なら状態をテキストでも示す。

```html
<p>
  ストレージ使用量:
  <meter min="0" max="100" low="60" high="85" optimum="0" value="92">
    92%
  </meter>
  <strong>92% — 空き容量が少なくなっています</strong>
</p>
```

## `meter`が向かない値

### タスクの完了率

タスクが進んでいるなら`progress`を使う。

### 上限が決まっていない値

世界人口、任意の売上額、身長、体重など、意味のある最大値がない測定へ`meter`を使わない。通常のテキストと単位で表示する。

```html
<p>今月の売上: <strong>1,250,000円</strong></p>
```

### 利用者が操作する値

音量や価格範囲を変更するUIには`input type="range"`やスピンボタンを使う。`meter`は読み取り専用の表示であり、操作コントロールではない。

## バッテリー残量を表示する

```html
<label for="battery-meter">バッテリー残量</label>
<meter
  id="battery-meter"
  min="0"
  max="100"
  low="20"
  high="80"
  optimum="100"
  value="48"
>
  48%
</meter>
<span>48% — 約3時間使用できます</span>
```

パーセントだけでは判断しにくい場合、残り時間などの意味のある情報を見えるテキストで補足する。

カスタム`role="meter"`を使う場合は、数字だけでは不十分なときに`aria-valuetext="48%、約3時間使用できます"`を設定できる。`aria-valuetext`は数値の代わりに読み上げられることがあるため、単に同じ数字を繰り返す目的では使わない。

> 参照: [W3C WAI-ARIA APG — Meter Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/meter/)

## カスタム`progressbar`が必要な場合

ネイティブ要素では表現できないデザインが必要な場合に限り、ARIAロールで再実装する。

```html
<span id="custom-progress-label">動画を変換</span>
<div
  id="custom-progress"
  class="custom-progress"
  role="progressbar"
  aria-labelledby="custom-progress-label"
  aria-valuemin="0"
  aria-valuemax="100"
  aria-valuenow="45"
  style="--progress-value: 45%;"
>
  <span class="custom-progress__fill" aria-hidden="true"></span>
</div>
<span id="custom-progress-value">45%</span>
```

```js
function updateCustomProgress(progressbar, value) {
  const min = Number(progressbar.getAttribute('aria-valuemin') ?? 0);
  const max = Number(progressbar.getAttribute('aria-valuemax') ?? 100);
  const currentValue = Number(value);

  if (![min, max, currentValue].every(Number.isFinite) || max <= min) {
    return;
  }

  const safeValue = Math.min(Math.max(currentValue, min), max);
  const percent = ((safeValue - min) / (max - min)) * 100;

  progressbar.setAttribute('aria-valuenow', String(safeValue));
  progressbar.style.setProperty('--progress-value', `${percent}%`);
  document.getElementById('custom-progress-value').textContent =
    `${Math.round(percent)}%`;
}
```

不確定状態では`aria-valuenow`を外す。

```js
customProgress.removeAttribute('aria-valuenow');
customProgress.classList.add('custom-progress--indeterminate');
```

`aria-valuemin`と`aria-valuemax`を省略した`progressbar`の既定範囲は0から100。既定値と異なる範囲なら明示する。

ラベルや値のテキストは`progressbar`の外へ置き、`aria-labelledby`で関連付ける。カスタムロールは表示も動作も追加しないため、値、見た目、状態メッセージをJavaScriptで同期する。

## カスタム`meter`が必要な場合

```html
<span id="custom-meter-label">CPU使用率</span>
<div
  class="custom-meter"
  role="meter"
  aria-labelledby="custom-meter-label"
  aria-valuemin="0"
  aria-valuemax="100"
  aria-valuenow="68"
  aria-valuetext="68%、通常の範囲"
  style="--meter-value: 68%;"
>
  <span class="custom-meter__fill" aria-hidden="true"></span>
</div>
<span>68% — 通常の範囲</span>
```

`meter`ロールでは`aria-valuenow`が必要。現在値を必ず最小値以上、最大値以下にする。

ARIAの`meter`には、HTMLの`low`、`high`、`optimum`に直接対応するARIA属性がない。評価区分が重要なら、見えるテキストと`aria-valuetext`へ意味を含める。

> 参照: [W3C WAI-ARIA APG — Communicating Value and Limits for Range Widgets](https://www.w3.org/WAI/ARIA/apg/practices/range-related-properties/)

## CSSでネイティブ要素を装飾する

```css
.progress-field,
.meter-field {
  display: grid;
  gap: 0.5rem;
  max-inline-size: 32rem;
}

progress,
meter {
  inline-size: 100%;
  block-size: 1.25rem;
  accent-color: Highlight;
}

.custom-progress,
.custom-meter {
  overflow: hidden;
  inline-size: 100%;
  block-size: 1.25rem;
  border: 0.125rem solid CanvasText;
  border-radius: 999rem;
  background: Canvas;
}

.custom-progress__fill {
  display: block;
  inline-size: var(--progress-value);
  block-size: 100%;
  background: Highlight;
}

.custom-progress--indeterminate .custom-progress__fill {
  inline-size: 35%;
  animation: progress-indeterminate 1.2s ease-in-out infinite alternate;
}

@keyframes progress-indeterminate {
  from {
    transform: translateX(-100%);
  }

  to {
    transform: translateX(285%);
  }
}

.custom-meter__fill {
  display: block;
  inline-size: var(--meter-value);
  block-size: 100%;
  background: Highlight;
}

@media (forced-colors: active) {
  progress,
  meter {
    forced-color-adjust: auto;
  }

  .custom-progress__fill,
  .custom-meter__fill {
    background: Highlight;
  }
}
```

ネイティブ要素の内部部品を細かく装飾する疑似要素はブラウザごとに異なる。対象環境で値、境界、状態が見えることを確認する。

バーの長さや色だけに依存せず、現在値と状態をテキストでも示す。強制カラーモードや印刷でも意味を確認できるようにする。

## 不確定アニメーションと動きの軽減

不確定なプログレスバーやスピナーをアニメーションさせる場合は、動きが強すぎないようにする。

```css
@media (prefers-reduced-motion: reduce) {
  .custom-progress--indeterminate .custom-progress__fill {
    animation: none;
    inline-size: 100%;
    transform: none;
  }
}
```

- 高速な点滅を使わない
- 動きがなくても「処理中」と理解できるテキストを残す
- 装飾アニメーションだけを止め、処理状態の通知は維持する
- 長時間処理にはキャンセルや別作業へ移れる手段を検討する

スピナーだけを置き、何を待っているか説明しない状態を避ける。

## Reactで実装する

### `progress`

```jsx
import { useId } from 'react';

function TaskProgress({ label, value, max = 100 }) {
  const progressId = useId();
  const safeMax = Number.isFinite(max) && max > 0 ? max : 100;
  const determinate = Number.isFinite(value);
  const safeValue = determinate
    ? Math.min(Math.max(value, 0), safeMax)
    : undefined;
  const percent = determinate
    ? Math.round((safeValue / safeMax) * 100)
    : null;

  return (
    <div className="progress-field">
      <label htmlFor={progressId}>{label}</label>
      <progress
        id={progressId}
        max={safeMax}
        value={safeValue}
      >
        {determinate ? `${percent}%` : '処理中'}
      </progress>
      <span>{determinate ? `${percent}%` : '処理中'}</span>
    </div>
  );
}
```

`value={undefined}`ならReactは`value`属性を出力せず、不確定状態になる。`0`を渡した場合は0%の確定状態になる。

この例では`max`が有限の正数でなければ100へ戻している。利用するデータの仕様に応じて、既定値への置き換えではなくエラーとして扱う方法も検討する。

### `meter`

```jsx
import { useId } from 'react';

function StorageMeter({ used, capacity }) {
  const meterId = useId();
  const safeCapacity = Number.isFinite(capacity) && capacity > 0
    ? capacity
    : 1;
  const currentUsed = Number.isFinite(used) ? used : 0;
  const safeUsed = Math.min(Math.max(currentUsed, 0), safeCapacity);
  const percent = Math.round((safeUsed / safeCapacity) * 100);

  return (
    <div className="meter-field">
      <label htmlFor={meterId}>ストレージ使用量</label>
      <meter
        id={meterId}
        min="0"
        max={safeCapacity}
        low={safeCapacity * 0.6}
        high={safeCapacity * 0.85}
        optimum="0"
        value={safeUsed}
      >
        {percent}%
      </meter>
      <span>
        {safeUsed} GB / {safeCapacity} GB（{percent}%）
      </span>
    </div>
  );
}
```

外部データが負数、`NaN`、最大値超過にならないように検証する。画面上で丸めて表示しても、`meter`の値と説明が大きく食い違わないようにする。

## 完了、失敗、停止を伝える

進捗バーが100%になっただけでは、処理結果が成功か失敗か分からない場合がある。

### 完了

```html
<p role="status">画像3件のアップロードが完了しました。</p>
```

### 失敗

```html
<div>
  <p role="status">アップロードに失敗しました。</p>
  <p>ネットワーク接続を確認して、もう一度お試しください。</p>
  <button type="button">再試行</button>
</div>
```

### キャンセル

```html
<p role="status">アップロードをキャンセルしました。</p>
```

エラーだからという理由だけで、すべてを`role="alert"`にしない。即時対応が必要な重要メッセージだけに強い割り込みを使い、通常は`role="status"`で簡潔に伝える。

処理結果を伝えるためにフォーカスをメッセージへ移す必要はない。再試行など次の操作が必要で、見つけにくい場合はフォーカス順序と画面構成を調整する。

## 残り時間を表示する

残り時間は変動しやすいため、「12秒」のように毎秒読み上げない。

```html
<p>
  残り時間の目安:
  <span id="remaining-time">約2分</span>
</p>
```

- 「約」を付け、推定値であることを示す
- 見える値は必要な頻度で更新する
- ライブリージョンでは意味のある節目だけ通知する
- 推定不能になったら「計算中」などへ戻す
- 完了時は残り0秒より「完了しました」と伝える

時刻が延びるたびに通知すると不安や混乱につながる。利用者の判断に必要な情報へ絞る。

## 複数タスクの進捗

複数ファイルを処理する場合は、全体と個別のどちらが必要かを決める。

```html
<section aria-labelledby="uploads-heading">
  <h2 id="uploads-heading">3件のファイルをアップロード</h2>

  <label for="overall-progress">全体の進捗</label>
  <progress id="overall-progress" value="1" max="3">1 / 3</progress>
  <span>1 / 3件完了</span>

  <ul>
    <li>photo-1.jpg — 完了</li>
    <li>photo-2.jpg — 65%</li>
    <li>photo-3.jpg — 待機中</li>
  </ul>
</section>
```

各ファイルにプログレスバーを置くと、支援技術が多くの進捗情報を読むことになる。個別の値が操作判断に必要でなければ、全体バーと簡潔なテキスト一覧にする。

完了した項目をDOMから即座に消さず、結果を確認できる時間や履歴を用意する。

## よくある失敗

### `progress`と`meter`を見た目で選ぶ

横棒の色や形ではなく、タスクの進捗か測定値かで選ぶ。

### 不確定状態へ`value="0"`を使う

値が分からない場合は`value`属性を外す。0%と不確定を区別する。

### 名前がない

何の処理または測定かを`label`、`aria-label`、`aria-labelledby`で伝える。

### バーの長さだけで値を示す

現在値、総量、単位をテキストでも確認できるようにする。

### 色だけで警告を示す

`meter`が赤や黄色になっても、意味を色だけに依存しない。「空き容量が少なくなっています」などのテキストを示す。

### 進捗を1%ごとに読み上げる

見える表示の更新とライブリージョンの通知頻度を分ける。節目と完了を優先する。

### 100%になった後も`aria-busy="true"`のまま

処理終了時に進捗、状態メッセージ、更新領域の`aria-busy`をまとめて同期する。

### 進捗表示へフォーカスを移す

状態を知らせるためだけに利用者の現在位置を奪わない。`role="status"`などで通知する。

### スピナーだけを表示する

何を待っているか、可能ならどの程度かかるか、キャンセルできるかをテキストで伝える。

### カスタムロールへ値を設定しない

確定した`progressbar`と`meter`では、現在値と範囲を正しく同期する。不確定な`progressbar`だけは`aria-valuenow`を外す。

## テストのチェックリスト

### 要素の選択

- [ ] タスクの進捗には`progress`を使っている
- [ ] 既知範囲内の測定値には`meter`を使っている
- [ ] 上限がない単純な数値へ`meter`を使っていない
- [ ] 操作可能な値へ`progress`や`meter`を使っていない
- [ ] ネイティブ要素で要件を満たせない理由がある

### 名前と値

- [ ] 何の進捗・測定値か分かる名前がある
- [ ] 現在値と最大値が有効な数値である
- [ ] 現在値が最小値以上、最大値以下である
- [ ] 値と単位をテキストでも確認できる
- [ ] 複数の進捗表示を名前で区別できる
- [ ] 見える値、HTML属性、ARIA属性が一致する

### 進捗状態

- [ ] 値が分からない場合は`value`または`aria-valuenow`を外している
- [ ] 0%と不確定状態を区別している
- [ ] 完了、失敗、キャンセルをテキストで伝えている
- [ ] 更新開始と終了に合わせて`aria-busy`を同期している
- [ ] 長時間処理にキャンセルや再試行手段がある
- [ ] 完了通知のためにフォーカスを奪っていない

### 通知

- [ ] 状態メッセージをフォーカスなしで認識できる
- [ ] 1%ごとなど過剰な読み上げがない
- [ ] 開始、節目、完了の通知方針が一貫している
- [ ] 残り時間を過剰な頻度で通知していない
- [ ] 複数タスクの通知が同時に競合しない
- [ ] 緊急でないメッセージへ`role="alert"`を乱用していない

### 表示

- [ ] バーの長さや色だけに依存していない
- [ ] 警告状態をテキストでも理解できる
- [ ] 200%・400%拡大でも名前と値が重ならない
- [ ] 強制カラーモードで境界と現在値を確認できる
- [ ] 動きを減らす設定でも処理中と理解できる
- [ ] スクリーンリーダーで名前、役割、値、範囲、完了を確認した

## 関連して読むとよいテーマ

- [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)
- [アクセシブルなレンジスライダーの実装 — 現在値を読み上げる範囲入力](./accessible-range-slider.md)
- [アクセシブルなスピンボタンの実装 — 数値の増減](./accessible-spinbutton.md)
- [prefers-reduced-motion — 動きを減らす設定への対応](./prefers-reduced-motion.md)
- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)
- [スクリーンリーダーでの手動テスト入門 — VoiceOver / NVDA](./screen-reader-manual-testing.md)

## まとめ

- タスクの完了状況には`progress`、既知範囲内の測定値には`meter`を使う
- 値が分からない進捗では`value`属性または`aria-valuenow`を外す
- 0%と不確定状態を区別する
- 何の進捗・測定値かをラベルで伝える
- バーの長さや色だけに依存せず、値、単位、状態をテキストでも示す
- 見える値は細かく更新しても、ライブリージョンは節目と完了に絞る
- 更新領域の`aria-busy`を開始と終了に合わせて同期する
- 完了、失敗、キャンセルを結果メッセージとして明示する
- カスタム実装では見た目、ARIA値、状態メッセージを同じデータから更新する
- 長時間処理にはキャンセル、再試行、動きを減らす設定への対応を検討する
