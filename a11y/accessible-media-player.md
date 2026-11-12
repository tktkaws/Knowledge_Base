# アクセシブルなメディアプレイヤーの実装 — 再生操作とキャプション

## アクセシブルなメディアプレイヤーとは

メディアプレイヤーは、動画や音声を再生し、再生位置、音量、字幕などを操作するUI。アクセシブルにするには、コンテンツ自体の代替情報とプレイヤーの操作性の両方が必要になる。

- すべての操作をキーボードから実行できる
- 各ボタンの目的と状態を支援技術へ伝える
- フォーカス位置を見た目で確認できる
- 音声情報をキャプションや文字起こしで提供する
- 映像だけで伝わる情報を音声解説や説明文で提供する
- 自動再生を避け、停止、音量、再生速度を利用者が制御できるようにする
- 200%・400%拡大や狭い画面でも操作部を失わない

最初に検討するのは、ネイティブの`video`または`audio`へ`controls`を付ける方法。ブラウザが再生、一時停止、再生位置、音量、全画面、字幕などを提供するため、独自プレイヤーを作る前に対象ブラウザと支援技術で確認する。

> 参照:
> - [W3C WAI — Making Audio and Video Media Accessible](https://www.w3.org/WAI/media/av/)
> - [WHATWG HTML Standard — Media elements](https://html.spec.whatwg.org/multipage/media.html)

## 動画の基本

```html
<video
  controls
  preload="metadata"
  poster="tutorial-poster.jpg"
  width="1280"
  height="720"
>
  <source src="tutorial.webm" type="video/webm" />
  <source src="tutorial.mp4" type="video/mp4" />
  <p>
    動画を再生できません。
    <a href="tutorial.mp4">動画ファイルをダウンロード</a>してください。
  </p>
</video>
```

- `controls`でブラウザ標準の操作部を表示する
- `source`を対応させたい形式ごとに並べる
- `preload="metadata"`で長さなどのメタデータだけを先に取得する
- `poster`へ再生前の静止画を指定する
- 要素内に再生できない環境向けの案内を置く

`poster`画像が内容理解に必要な情報を持つ場合、その情報を動画の近くのテキスト、キャプション、文字起こしなどでも提供する。

## 音声の基本

```html
<figure>
  <figcaption>アクセシビリティ入門ポッドキャスト 第1回</figcaption>
  <audio controls preload="metadata">
    <source src="episode-1.ogg" type="audio/ogg" />
    <source src="episode-1.mp3" type="audio/mpeg" />
    <p>
      音声を再生できません。
      <a href="episode-1.mp3">音声ファイルをダウンロード</a>してください。
    </p>
  </audio>
</figure>

<p><a href="#transcript">第1回の文字起こしを読む</a></p>
```

`audio`には映像がないため、事前収録された音声だけのコンテンツでは文字起こしが重要になる。プレイヤーの近くに置き、利用可能であることを見つけやすくする。

## キャプションを追加する

```html
<video controls preload="metadata" poster="tutorial-poster.jpg">
  <source src="tutorial.mp4" type="video/mp4" />

  <track
    kind="captions"
    src="tutorial-ja.vtt"
    srclang="ja"
    label="日本語"
    default
  />

  <track
    kind="captions"
    src="tutorial-en.vtt"
    srclang="en"
    label="English"
  />
</video>
```

### `track`の主な属性

| 属性 | 役割 |
|---|---|
| `kind="captions"` | 会話と重要な音声情報を同期して示す |
| `src` | WebVTTなどの字幕ファイル |
| `srclang` | 字幕の言語コード |
| `label` | 字幕メニューに表示する言語名 |
| `default` | 初期候補にするトラック |

複数言語がある場合、各トラックの`srclang`と`label`を正しく設定する。同じ言語のトラックへ重複して`default`を付けない。

`default`があっても、ブラウザや利用者設定によって初期表示は異なることがある。字幕を常に見せる必要があるコンテンツでは、対象環境で実際の挙動を確認する。

## キャプションと字幕の違い

会話だけを翻訳する字幕では、音が聞こえない利用者に必要な情報が欠ける場合がある。キャプションには理解に必要な音声情報を含める。

```text
悪い例:
ドアを閉めてください。

良い例:
美咲: ドアを閉めてください。
[ドアが強く閉まる音]
[不穏な音楽]
```

- 複数人の声を区別できない場面では話者を示す
- 意味のある効果音、音楽、笑い声、無音を示す
- 発話の意味を変える声色や話し方を必要に応じて示す
- 読み切れる長さと表示時間にする
- 音声と正確に同期させる
- 映像内の重要な文字を別の手段でも伝える

自動生成字幕は下書きとして利用できるが、公開前に人が内容、固有名詞、否定表現、句読点、話者、非音声音、同期を確認する。

> 参照: [W3C WAI — Captions/Subtitles](https://www.w3.org/WAI/media/av/captions/)

## WebVTTを書く

`track`でよく使われる形式がWebVTT。

```vtt
WEBVTT

00:00:00.000 --> 00:00:03.500
<v ナレーター>この動画では、フォームの作り方を説明します。

00:00:03.700 --> 00:00:06.200
[キーボードを入力する音]

00:00:06.400 --> 00:00:10.000
<v 美咲>最初に、すべての入力欄へラベルを付けます。
```

- 先頭に`WEBVTT`を書く
- 開始時刻と終了時刻を`-->`でつなぐ
- キュー同士を空行で区切る
- `<v 話者名>`で話者を示せる
- キャプション同士を不必要に重ねない
- 画面上の重要な情報を隠さない位置を検討する

字幕ファイルを別オリジンから配信する場合は、メディア要素の`crossorigin`と配信元のCORSヘッダーを含めて検証する。動画だけ再生できて字幕だけ読み込めない状態を見落とさない。

## 文字起こしを提供する

文字起こしは音声を順番に読めるテキスト。動画では重要な映像情報も含めた「説明付き文字起こし」にすると、映像と音声の両方へアクセスできない利用者にも内容を伝えられる。

```html
<section id="transcript" aria-labelledby="transcript-heading">
  <h2 id="transcript-heading">文字起こし</h2>

  <h3>はじめに</h3>
  <p>
    <strong>ナレーター:</strong>
    この動画では、フォームの作り方を説明します。
  </p>
  <p><em>キーボードを入力する音。</em></p>

  <h3>ラベルを付ける</h3>
  <p>
    <strong>映像:</strong>
    メールアドレス入力欄と、その上にある「メールアドレス」というラベルが表示される。
  </p>
  <p>
    <strong>美咲:</strong>
    最初に、すべての入力欄へラベルを付けます。
  </p>
</section>
```

- 話者を区別する
- 会話と重要な非音声音を含める
- 動画では重要な動作、図表、画面内文字も説明する
- 長い内容は見出しと段落で分ける
- 必要なら大まかな時刻を付ける
- 動画の直後か、すぐ見つかるリンク先に置く

キャプションファイルから機械的に時刻を取り除くだけでは、映像だけにある情報が抜ける。動画を見直して説明を補う。

文字起こしはキャプションの代わりになるとは限らない。動画を見ながら同期した文字が必要な利用者にはキャプションを提供し、別の速度で読んだり検索したりしたい利用者には文字起こしを提供する。

> 参照: [W3C WAI — Transcripts](https://www.w3.org/WAI/media/av/transcripts/)

## 音声解説を提供する

音声解説は、会話の合間などに重要な映像情報を音声で伝えるもの。

```text
音声: 「保存ボタンを押します。」
映像: 画面右上に「保存しました」と表示される。

音声解説: 「画面右上に、保存しましたと表示される。」
```

次の情報が音声で伝わっているか確認する。

- 登場人物の重要な動作や表情
- 場面転換
- 図表の変化
- 画面にだけ表示される名前、数値、警告、操作結果
- 理解や操作に必要な視覚情報

新しく動画を作る場合、話者が重要な視覚情報を本編の音声内で説明する「統合された解説」を検討する。別トラックとの同期や切り替えが不要になり、より多くの環境で利用しやすい。

別の音声解説版を用意する場合は、元動画の近くに明確なリンクを置く。

```html
<ul>
  <li><a href="tutorial.html">通常版を再生</a></li>
  <li><a href="tutorial-described.html">音声解説付きで再生</a></li>
</ul>
```

`track kind="descriptions"`の読み上げ対応はプレイヤーや支援技術によって異なる。使用する場合は対象環境で確認し、必要に応じて音声解説付き動画や説明付き文字起こしも提供する。

> 参照: [W3C WAI — Description of Visual Information](https://www.w3.org/WAI/media/av/description/)

## WCAGレベルごとの主な考え方

| コンテンツ | レベルAで主に必要になるもの | レベルAAで追加される主なもの |
|---|---|---|
| 事前収録の音声のみ | 同等の情報を持つ文字起こし | — |
| 事前収録の映像のみ | 説明付き文字起こしまたは音声による説明 | — |
| 事前収録の音声付き動画 | キャプション、音声解説またはメディア代替 | 音声解説 |
| ライブの音声付き動画 | — | ライブキャプション |

適用条件や例外は、対象コンテンツと目標適合レベルに応じてWCAG本文と解説を確認する。

> 参照: [WCAG 2.2 — Guideline 1.2 Time-based Media](https://www.w3.org/TR/WCAG22/#time-based-media)

## 自動再生を避ける

```html
<!-- 推奨：利用者が再生を開始する -->
<video controls preload="metadata">
  <source src="introduction.mp4" type="video/mp4" />
</video>
```

ページを開いた直後に音声が流れると、スクリーンリーダーの音声を聞き取りにくくなり、停止操作を探すことも難しくなる。

```html
<!-- 避ける：音声付き動画の自動再生 -->
<video controls autoplay>
  <source src="introduction.mp4" type="video/mp4" />
</video>
```

自動再生が必要な背景映像では、初期状態をミュートし、停止できる操作をすぐ見つけられる位置へ置く。`muted`なら音声の問題がなくなるだけで、動きによる注意や前庭への影響は残る。

`prefers-reduced-motion`はCSSのアニメーションを減らすために使えるが、HTMLメディアの再生をブラウザが自動で止める仕組みではない。JavaScriptで設定を確認し、自動再生しない設計を優先する。

> 参照: [Understanding WCAG 2.2 SC 1.4.2 — Audio Control](https://www.w3.org/WAI/WCAG22/Understanding/audio-control.html)

## ネイティブ`controls`を優先する

ネイティブ操作部には次の利点がある。

- ブラウザやOSの操作慣例に沿う
- キーボード操作とフォーカス管理をブラウザが提供する
- 音量、全画面、ピクチャーインピクチャなどを環境に合わせて提供する
- 字幕設定や利用者の既定設定を反映できる場合がある
- JavaScriptが失敗しても操作できる

一方で、ブラウザごとに機能と見た目が異なる。採用前に対象環境で次を確認する。

- 再生、一時停止、シーク、音量、ミュート
- 字幕の表示、非表示、言語切り替え
- キーボードからすべての操作へ到達できること
- フォーカス表示とコントラスト
- 拡大時と狭い画面で操作が隠れないこと
- スクリーンリーダーが名前、役割、状態を伝えること

## カスタム操作部の基本構造

独自デザインや追加機能が必要な場合も、HTMLの`video`を再生エンジンとして使い、標準の`button`、`input type="range"`、`select`で操作部を作る。

```html
<section class="media-player" aria-labelledby="video-heading">
  <h2 id="video-heading">フォームの作り方</h2>

  <video
    id="tutorial-video"
    controls
    preload="metadata"
    poster="tutorial-poster.jpg"
  >
    <source src="tutorial.mp4" type="video/mp4" />
    <track
      id="tutorial-captions"
      kind="captions"
      src="tutorial-ja.vtt"
      srclang="ja"
      label="日本語"
      default
    />
  </video>

  <div
    id="video-controls"
    class="media-controls"
    role="group"
    aria-label="動画の操作"
    hidden
  >
    <button id="play-button" type="button">再生</button>

    <label for="seek-control">再生位置</label>
    <input
      id="seek-control"
      type="range"
      min="0"
      max="0"
      value="0"
      step="0.1"
    />
    <output id="time-output" for="seek-control">0:00 / 0:00</output>

    <button id="mute-button" type="button">ミュート</button>

    <label for="volume-control">音量</label>
    <input
      id="volume-control"
      type="range"
      min="0"
      max="1"
      value="1"
      step="0.05"
    />

    <button id="captions-button" type="button" aria-pressed="false">
      字幕
    </button>
    <button id="fullscreen-button" type="button">全画面にする</button>
  </div>

  <p id="player-status" role="status"></p>
</section>
```

HTMLでは最初に`controls`を付け、カスタム操作部を`hidden`にする。JavaScriptの初期化に成功してからネイティブ操作部を外す。JavaScriptが読み込めなければ、ネイティブ操作部が残る。

操作部全体は`role="group"`と名前でまとめる。`toolbar`にすると矢印キー移動などの追加規則が必要になるため、通常のTab移動で操作するなら`group`が単純。

## カスタム操作部を同期する

```js
const video = document.getElementById('tutorial-video');
const controls = document.getElementById('video-controls');
const playButton = document.getElementById('play-button');
const seekControl = document.getElementById('seek-control');
const timeOutput = document.getElementById('time-output');
const muteButton = document.getElementById('mute-button');
const volumeControl = document.getElementById('volume-control');
const captionsButton = document.getElementById('captions-button');
const fullscreenButton = document.getElementById('fullscreen-button');
const playerStatus = document.getElementById('player-status');
const captionsElement = document.getElementById('tutorial-captions');
const player = video.closest('.media-player');
let previousVolume = video.volume || 1;

function formatTime(seconds) {
  if (!Number.isFinite(seconds) || seconds < 0) return '0:00';

  const wholeSeconds = Math.floor(seconds);
  const minutes = Math.floor(wholeSeconds / 60);
  const remainingSeconds = String(wholeSeconds % 60).padStart(2, '0');
  return `${minutes}:${remainingSeconds}`;
}

function updateTime() {
  const current = formatTime(video.currentTime);
  const duration = formatTime(video.duration);

  seekControl.value = String(video.currentTime);
  seekControl.setAttribute(
    'aria-valuetext',
    `${current} / ${duration}`,
  );
  timeOutput.value = `${current} / ${duration}`;
}

function updateDuration() {
  if (!Number.isFinite(video.duration)) return;
  seekControl.max = String(video.duration);
  updateTime();
}

function updatePlayButton() {
  playButton.textContent = video.paused ? '再生' : '一時停止';
}

function updateMuteButton() {
  muteButton.textContent = video.muted || video.volume === 0
    ? 'ミュートを解除'
    : 'ミュート';
}

function updateVolumeControl() {
  volumeControl.value = String(video.volume);
  volumeControl.setAttribute(
    'aria-valuetext',
    video.muted ? 'ミュート' : `${Math.round(video.volume * 100)}%`,
  );
  updateMuteButton();
}

function getCaptionTrack() {
  return Array.from(video.textTracks).find(
    (track) => track.kind === 'captions',
  );
}

function updateCaptionsButton() {
  const track = getCaptionTrack();
  const showing = track?.mode === 'showing';

  captionsButton.hidden = !track;
  captionsButton.setAttribute('aria-pressed', String(showing));
}

async function togglePlayback() {
  if (!video.paused) {
    video.pause();
    return;
  }

  try {
    await video.play();
  } catch {
    playerStatus.textContent = '動画を再生できませんでした。もう一度お試しください。';
  }
}

playButton.addEventListener('click', togglePlayback);

seekControl.addEventListener('input', () => {
  video.currentTime = Number(seekControl.value);
  updateTime();
});

muteButton.addEventListener('click', () => {
  if (video.muted || video.volume === 0) {
    video.muted = false;
    video.volume = previousVolume;
  } else {
    previousVolume = video.volume;
    video.muted = true;
  }
});

volumeControl.addEventListener('input', () => {
  const nextVolume = Number(volumeControl.value);
  if (nextVolume > 0) previousVolume = nextVolume;

  video.volume = nextVolume;
  video.muted = video.volume === 0;
});

captionsButton.addEventListener('click', () => {
  const track = getCaptionTrack();
  if (!track) return;

  track.mode = track.mode === 'showing' ? 'hidden' : 'showing';
  updateCaptionsButton();
});

fullscreenButton.addEventListener('click', async () => {
  try {
    if (document.fullscreenElement) {
      await document.exitFullscreen();
    } else {
      await player.requestFullscreen();
    }
  } catch {
    playerStatus.textContent = '全画面表示へ切り替えられませんでした。';
  }
});

video.addEventListener('loadedmetadata', updateDuration);
video.addEventListener('durationchange', updateDuration);
video.addEventListener('timeupdate', updateTime);
video.addEventListener('play', updatePlayButton);
video.addEventListener('pause', updatePlayButton);
video.addEventListener('ended', updatePlayButton);
video.addEventListener('volumechange', updateVolumeControl);
video.addEventListener('error', () => {
  playerStatus.textContent = '動画を読み込めませんでした。';
});

captionsElement.addEventListener('load', updateCaptionsButton);
captionsElement.addEventListener('error', () => {
  captionsButton.hidden = true;
  playerStatus.textContent = '字幕を読み込めませんでした。';
});

document.addEventListener('fullscreenchange', () => {
  fullscreenButton.textContent = document.fullscreenElement === player
    ? '全画面を終了'
    : '全画面にする';
});

video.controls = false;
controls.hidden = false;
updatePlayButton();
updateVolumeControl();
updateCaptionsButton();

if (video.readyState >= 1) {
  updateDuration();
} else {
  updateTime();
}
```

`timeupdate`のたびに`role="status"`へ時刻を送らない。現在時刻は`output`とシークスライダーの`aria-valuetext`で確認できるようにし、再生中の連続通知を避ける。

再生や全画面の成否は、クリック直後の通常状態なら毎回ライブ通知する必要はない。失敗した場合だけ、簡潔なメッセージを`role="status"`へ送る。

## 再生位置の操作

再生位置は`input type="range"`で実装できる。

- `min="0"`にする
- メタデータ取得後に`max`へ動画の長さを設定する
- `value`と`video.currentTime`を同期する
- 数値の秒数だけでなく「1:25 / 4:30」のような値を`aria-valuetext`で示す
- 現在時刻をライブリージョンにしない
- ライブ配信など長さが確定しないメディアでは、通常の確定範囲スライダーとして扱わない

矢印キー、Page Up / Page Down、Home、Endなどはネイティブのレンジ入力が提供する。独自の`div role="slider"`に置き換える必要はない。

## 音量とミュート

音量スライダーとミュートボタンは別の機能として扱う。

- 音量スライダーへ見える`label`を付ける
- 値をパーセントとして理解できるようにする
- ミュート状態と音量0を見た目と支援技術へ伝える
- ミュート解除時に戻す音量を仕様として決める
- システム全体の音量変更を要求しない

ミュートボタンの文言を現在実行する操作に合わせて「ミュート」「ミュートを解除」と切り替える場合、`aria-pressed`は不要。文言を常に「ミュート」にするトグルボタンなら、`aria-pressed`を状態と同期する。

## 字幕の表示と言語切り替え

字幕の表示・非表示だけなら、状態を持つボタンとして`aria-pressed`を使える。

```html
<button type="button" aria-pressed="true">字幕</button>
```

ボタンの文言は「字幕」のままにし、押されている状態で表示中であることを伝える。

複数言語を選ぶ場合は、標準の`select`が単純。

```html
<label for="caption-language">字幕</label>
<select id="caption-language">
  <option value="off">表示しない</option>
  <option value="ja">日本語</option>
  <option value="en">English</option>
</select>
```

```js
const captionLanguage = document.getElementById('caption-language');

captionLanguage.addEventListener('change', () => {
  Array.from(video.textTracks).forEach((track) => {
    if (track.kind !== 'captions') return;
    track.mode = track.language === captionLanguage.value
      ? 'showing'
      : 'disabled';
  });
});
```

独自のポップアップメニューを作る場合は、開閉、フォーカス、Escape、外側クリック、選択状態なども実装する必要がある。要件を満たせるなら`select`を優先する。

## 再生速度を変更する

```html
<label for="playback-rate">再生速度</label>
<select id="playback-rate">
  <option value="0.75">0.75倍</option>
  <option value="1" selected>1倍</option>
  <option value="1.25">1.25倍</option>
  <option value="1.5">1.5倍</option>
  <option value="2">2倍</option>
</select>
```

```js
const playbackRate = document.getElementById('playback-rate');

playbackRate.addEventListener('change', () => {
  video.playbackRate = Number(playbackRate.value);
});
```

再生速度は、情報処理や聞き取りに必要な時間を利用者が調整する助けになる。速度を変えてもキャプション、音声解説、再生位置が同期することを確認する。

## キーボードショートカット

標準のボタンと入力要素だけで、Tab、Enter、Space、矢印キーによる基本操作を提供できる。独自ショートカットは必須ではない。

追加する場合は次を守る。

- プレイヤー内にフォーカスがあるときだけ有効にする
- 入力欄や`select`の操作を奪わない
- ブラウザや支援技術のショートカットと競合しない
- 画面上またはヘルプで一覧を確認できる
- 単一文字ショートカットは無効化、再割り当て、フォーカス時だけ有効化のいずれかを提供する
- キーを押し続ける正確な時間を要求しない

「Kで再生」「Mでミュート」のような慣例があっても、ボタン操作を置き換えない。

## インタラクティブな文字起こし

文字起こしの時刻から動画へ移動できると、内容を検索して該当箇所を再生しやすい。

```html
<section aria-labelledby="interactive-transcript-heading">
  <h2 id="interactive-transcript-heading">文字起こし</h2>
  <p>
    <button type="button" data-time="0">0:00へ移動</button>
    <strong>ナレーター:</strong>
    この動画では、フォームの作り方を説明します。
  </p>
  <p>
    <button type="button" data-time="6.4">0:06へ移動</button>
    <strong>美咲:</strong>
    最初に、すべての入力欄へラベルを付けます。
  </p>
</section>
```

```js
document.addEventListener('click', (event) => {
  const button = event.target.closest('[data-time]');
  if (!button) return;

  const time = Number(button.dataset.time);
  if (!Number.isFinite(time)) return;

  video.currentTime = time;
  video.play().catch(() => {
    playerStatus.textContent = '指定した位置から再生できませんでした。';
  });
});
```

現在発話中の段落を視覚的に強調する場合も、文字起こし自体を自動スクロールし続けて読書位置を奪わない。自動追従を停止できるようにし、選択したテキストやキーボードフォーカスを勝手に移さない。

## CSSで操作部を整える

```css
.media-player {
  max-inline-size: 64rem;
}

.media-player video {
  display: block;
  inline-size: 100%;
  block-size: auto;
  background: #000;
}

.media-controls {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem;
  background: Canvas;
  color: CanvasText;
  border: 0.125rem solid currentColor;
}

.media-controls button,
.media-controls select,
.media-controls input[type="range"] {
  min-block-size: 2.75rem;
  font: inherit;
}

.media-controls button:focus-visible,
.media-controls select:focus-visible,
.media-controls input[type="range"]:focus-visible {
  outline: 0.1875rem solid Highlight;
  outline-offset: 0.1875rem;
}

.media-controls input[type="range"] {
  flex: 1 1 12rem;
  min-inline-size: 8rem;
}

video::cue {
  color: white;
  background: rgb(0 0 0 / 80%);
}

@media (forced-colors: active) {
  .media-controls {
    border-color: CanvasText;
  }
}
```

キャプションの見た目を固定しすぎない。ブラウザやOSの字幕設定を利用する人がいるため、対象環境で文字サイズ、色、背景、位置の上書き可否を確認する。

操作部は横一列へ押し込めず、狭い画面で折り返せるようにする。シークや音量が短くなりすぎる場合は別の行へ分ける。

## Reactでネイティブプレイヤーを使う

```jsx
function AccessibleVideo({
  title,
  poster,
  sources,
  captions,
  transcriptHref,
}) {
  return (
    <figure>
      <figcaption>{title}</figcaption>

      <video controls preload="metadata" poster={poster}>
        {sources.map((source) => (
          <source
            key={source.src}
            src={source.src}
            type={source.type}
          />
        ))}

        {captions.map((caption) => (
          <track
            key={`${caption.src}:${caption.language}`}
            kind="captions"
            src={caption.src}
            srcLang={caption.language}
            label={caption.label}
            default={caption.default}
          />
        ))}

        <p>
          動画を再生できません。
          <a href={sources[0]?.src}>動画ファイルをダウンロード</a>
          してください。
        </p>
      </video>

      <p>
        <a href={transcriptHref}>文字起こしを読む</a>
      </p>
    </figure>
  );
}
```

Reactを使っても、ネイティブの`video`、`source`、`track`の意味は変わらない。カスタム操作部が不要なら状態管理を追加せず、ブラウザの操作部を使う。

`sources`が空の場合や字幕ファイルが取得できない場合の表示を、実際のコンポーネントでは検証する。字幕トラックの読み込みエラーも監視し、字幕が利用できないことを隠さない。

## ライブ配信

ライブ配信では、事前収録とは異なる状態を伝える。

- ライブであることをテキストで示す
- WCAGレベルAAを目標にする場合はライブキャプションを提供する
- 配信開始前、接続中、再接続中、終了を区別する
- 遅延時間を意味なく毎秒読み上げない
- 長さが確定しない場合は通常の再生位置スライダーを表示しない
- 巻き戻し可能なら利用可能な範囲を正確に示す
- キャプションが遅延または停止した場合も状態を伝える

```html
<section aria-labelledby="live-heading">
  <h2 id="live-heading">製品発表ライブ配信</h2>
  <p><strong>ライブ</strong></p>
  <video controls></video>
  <p role="status">ライブ配信へ接続しました。</p>
</section>
```

配信映像の接続とキャプション配信の接続を別々に監視する。映像が動いているだけで字幕も正常と判断しない。

## よくある失敗

### 動画にキャプションがない

事前収録の音声付き動画には、正確で同期したキャプションを提供する。自動生成の未確認字幕だけで完了にしない。

### キャプションが会話だけ

理解に必要な話者、効果音、音楽、笑い声なども含める。

### 文字起こしが見つからない

プレイヤーの直後に本文またはリンクを置き、「文字起こし」と明記する。

### 映像だけの情報を説明しない

画面内文字、図表、操作結果、重要な動作を本編音声、音声解説、説明付き文字起こしで伝える。

### 自動再生する

利用者の操作で再生を始める。背景映像でも停止手段を提供し、動きを減らす設定を考慮する。

### アイコンだけで操作を作る

再生、ミュート、字幕、全画面などへ明確なアクセシブルネームを付ける。状態も同期する。

### `div`でボタンやスライダーを再実装する

標準の`button`、`input type="range"`、`select`を優先する。

### 時刻を連続して読み上げる

再生位置はスライダーと見えるテキストで確認できるようにし、`timeupdate`をライブ通知しない。

### カスタム操作部だけがあり、JavaScript失敗時に操作できない

初期HTMLではネイティブ`controls`を残し、初期化成功後にだけカスタム操作部へ切り替える。

### キーボードショートカットが入力を奪う

プレイヤー外やフォーム入力中に単一文字キーを処理しない。ショートカットなしでも全操作を完了できるようにする。

### 全画面で字幕やフォーカスが消える

全画面要素に動画とカスタム操作部を含め、字幕、操作、フォーカス表示を実機で確認する。

## テストのチェックリスト

### メディアの代替情報

- [ ] 事前収録の音声付き動画に正確で同期したキャプションがある
- [ ] 話者と重要な非音声音がキャプションに含まれている
- [ ] 自動生成字幕を人が確認・修正している
- [ ] 音声のみのコンテンツに文字起こしがある
- [ ] 動画の重要な視覚情報を音声解説または説明文で提供している
- [ ] 文字起こしをプレイヤーの近くから見つけられる
- [ ] キャプションと文字起こしの言語が内容に合っている
- [ ] 字幕ファイルの取得失敗を検出できる

### 操作とキーボード

- [ ] 再生、一時停止、シーク、音量、ミュートをキーボードで操作できる
- [ ] 字幕、速度、全画面など提供する全機能へキーボードで到達できる
- [ ] Tab順序が見た目と操作の流れに合っている
- [ ] フォーカス位置を常に確認できる
- [ ] ボタンの名前から操作結果を予測できる
- [ ] トグル状態が`aria-pressed`などで伝わる
- [ ] スライダーの名前、現在値、最小値、最大値が伝わる
- [ ] ショートカットなしでも全操作を完了できる

### 再生状態

- [ ] ページ表示時に音声を自動再生しない
- [ ] 自動で動く背景映像を停止できる
- [ ] 再生、一時停止、終了の状態が操作部と同期する
- [ ] 音量0とミュート状態が操作部と同期する
- [ ] 連続する時刻更新をライブ通知していない
- [ ] 読み込み失敗と再試行方法をテキストで伝えている
- [ ] ライブ配信の接続中、再接続、終了を区別している

### 表示

- [ ] 操作部と背景のコントラストを確認した
- [ ] フォーカスインジケーターのコントラストを確認した
- [ ] キャプションが重要な映像情報を隠さない
- [ ] 200%・400%拡大でも操作部を使用できる
- [ ] 狭い画面で操作部が画面外へ出ない
- [ ] 強制カラーモードでボタンと状態を識別できる
- [ ] 全画面でも字幕、操作、フォーカス表示を確認できる

### 対象環境

- [ ] Chrome、Safari、Firefoxなど対象ブラウザで確認した
- [ ] Windows、macOS、iOS、Androidなど対象OSで確認した
- [ ] VoiceOver、NVDAなど対象スクリーンリーダーで確認した
- [ ] キャプションの表示・非表示と言語切り替えを確認した
- [ ] 再生速度変更後も字幕と音声解説が同期する
- [ ] JavaScriptを無効化または読み込み失敗させても再生操作が残る

## 関連して読むとよいテーマ

- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [アクセシブルなレンジスライダーの実装 — 現在値を読み上げる範囲入力](./accessible-range-slider.md)
- [アクセシブルなプログレス表示の実装 — progress と meter の違い](./accessible-progress-and-meter.md)
- [フォーカス表示の設計 — :focus-visibleとWCAG 2.4.7 / 2.4.11](./focus-visible.md)
- [prefers-reduced-motion — 動きを減らす設定への対応](./prefers-reduced-motion.md)
- [aria-live — 動的コンテンツの変更を通知する](./aria-live.md)
- [スクリーンリーダーでの手動テスト入門 — VoiceOver / NVDA](./screen-reader-manual-testing.md)

## まとめ

- ネイティブの`video` / `audio`と`controls`を最初に検討する
- 音声付き動画へ正確で同期したキャプションを提供する
- キャプションへ話者と重要な非音声音を含める
- 音声のみのコンテンツへ文字起こしを提供する
- 映像だけで伝わる情報を音声解説または説明付き文字起こしで提供する
- 自動再生を避け、停止、音量、速度を利用者が制御できるようにする
- カスタム操作部では標準のボタン、レンジ入力、セレクトを使う
- JavaScript初期化に成功するまでネイティブ操作部を残す
- 再生位置や時刻を過剰な頻度でライブ通知しない
- 字幕、全画面、再生速度、ライブ配信を対象環境で実際に確認する
