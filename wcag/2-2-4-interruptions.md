# 2.2.4 割り込み（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.2.4「割り込み」（Interruptions）
- レベル **AAA**（最高の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.2「十分な時間」に属する
- 割り込みを、ユーザーが**延期**または**抑制**できること
- 例外は、**緊急事態**に関する割り込み

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.4 Interruptions](https://www.w3.org/TR/WCAG22/#interruptions)
> - [Understanding SC 2.2.4 Interruptions](https://www.w3.org/WAI/WCAG22/Understanding/interruptions.html)

## 達成基準の原文（要約）

- 割り込みは、ユーザーが延期または抑制できること
- 例外：緊急事態に関する割り込み

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.4](https://www.w3.org/TR/WCAG22/#interruptions)

## 割り込みとは

- ユーザーの現在の操作や閲覧を妨げる、作者／サーバー側からの更新や通知
- 例
  - 自動で表示されるアラート・トースト
  - 自動更新されるニュース・チャット・通知パネル
  - 定期的なページ再読み込み
  - セッション中に突然挿入されるモーダルやバナー
- ユーザーが自分で開始した更新（「更新」ボタンの押下など）は、意図的な操作であり割り込みではない

```text
割り込みになりうる例：
  フォーム入力中にチャット通知が前面に出る
  記事読了中にヘッドラインが自動差し替わる
  meta refresh でページ全体が再読み込みされる

割り込みではない例：
  「最新情報を取得」ボタンを押した結果の更新
  ユーザーが開いたメニュー内の操作結果
```

> **参照**
> - [Understanding SC 2.2.4 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/interruptions.html#intent)

## なぜ必要か

- 注意欠陥・多動性障害など、注意の維持が難しいユーザーが、内容に集中できるようにするため
- ロービジョンのユーザーやスクリーンリーダー利用者が、読んでいる内容を途中で失わないようにするため
- 自動更新があると、読み始めの話題と読み終わりの話題が食い違い、誤解しやすい
- 支援技術の読み上げやフォーカスが、突然の更新で中断されやすい

> **参照**
> - [Understanding SC 2.2.4 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/interruptions.html#benefits)

## 基本方針：延期または抑制できること

- 割り込みを**オフ**にできる
- 割り込みを**後でまとめて表示**できる（延期）
- 自動更新ではなく、ユーザーが**要求したときだけ更新**する
- 緊急事態の割り込みは例外として残す

```text
推奨される仕組み：
1. 更新・通知をオフにする
2. セッション終了まで延期する
3. 手動更新のみにする
4. 緊急アラートだけは常に表示する
```

## 設定で延期する（G75）

- ポータルやダッシュボードで、更新・アラートをセッション終了まで延期できる設定を提供する
- 緊急アラートだけは延期対象外にする

```html
<!-- 良い例：更新の延期設定 -->
<form>
  <fieldset>
    <legend>通知設定</legend>
    <label>
      <input type="checkbox" name="postpone-updates" value="1">
      緊急事態を除き、更新とアラートをセッション終了まで延期する
    </label>
  </fieldset>
  <button type="submit">保存</button>
</form>
```

> **参照**
> - [Technique G75: Providing a mechanism to postpone any updating of content](https://www.w3.org/WAI/WCAG22/Techniques/general/G75)
> - [Understanding SC 2.2.4 — Example 1](https://www.w3.org/WAI/WCAG22/Understanding/interruptions.html#examples)

## 手動更新にする（G76）

- 自動更新の代わりに、「更新」ボタンやリンクでユーザーが内容を要求する
- ユーザーのペースで情報を取り込めるため、集中を妨げにくい

```html
<!-- 悪い例：30秒ごとに自動再読み込み -->
<meta http-equiv="refresh" content="30">

<!-- 良い例：ユーザーが更新を要求する -->
<section aria-labelledby="news-heading">
  <h2 id="news-heading">最新ニュース</h2>
  <button type="button" id="refresh-news">最新情報を取得</button>
  <div id="news-list">...</div>
</section>
```

```javascript
document.getElementById("refresh-news").addEventListener("click", async () => {
  const list = document.getElementById("news-list");
  list.textContent = "更新中...";
  const html = await fetchNews();
  list.innerHTML = html;
});
```

> **参照**
> - [Technique G76: Providing a mechanism to request an update of the content instead of updating automatically](https://www.w3.org/WAI/WCAG22/Techniques/general/G76)

## 必須でないアラートを任意にする（SCR14）

- マーケティング通知、ヒント、チャット招待など、不可欠でないアラートはオフにできる
- デフォルトでオフ、または初回に選択させる設計も有効
- エラーメッセージやフォーム検証結果など、操作の結果として必要な通知は別扱い

```html
<!-- 良い例：任意アラートの設定 -->
<label>
  <input type="checkbox" id="show-tips" checked>
  操作のヒントを表示する
</label>

<dialog id="tip-dialog">
  <p>ショートカットキーで素早く操作できます。</p>
  <button type="button" id="close-tip">閉じる</button>
</dialog>
```

```javascript
const showTips = document.getElementById("show-tips");
const tipDialog = document.getElementById("tip-dialog");

function maybeShowTip() {
  if (!showTips.checked) return;
  tipDialog.showModal();
}
```

> **参照**
> - [Technique SCR14: Using scripts to make nonessential alerts optional](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR14)

## 例外：緊急事態

- **緊急事態**：健康・安全・財産を守るために即時対応が必要な、突然の状況
- 例
  - 災害・避難などの公共緊急アラート
  - 接続切断やデータ損失の警告
  - セキュリティ侵害の可能性など、即時対応が必要な警告
- 緊急アラートは延期・抑制の対象外でよい
- 「緊急」を広く解釈し、通常のマーケティング通知まで例外にしない

```text
例外になりうる：
  津波警報
  セッション切断によるデータ損失の警告
  不正アクセス検知の即時通知

例外にならない：
  セール告知
  新機能の紹介モーダル
  おすすめ記事の自動差し替え
```

> **参照**
> - [Understanding SC 2.2.4 — emergency](https://www.w3.org/WAI/WCAG22/Understanding/interruptions.html#dfn-emergency)

## チャット・通知・ライブ更新

- チャットアプリやダッシュボードのライブ更新も、ユーザーが制御できること
- 通知をミュートする、更新頻度を下げる、手動更新にするなどの選択肢
- フォーカス中の入力欄を奪う割り込みは特に避ける

```html
<!-- 良い例：通知の抑制 -->
<label>
  <input type="checkbox" id="mute-chat" checked>
  チャット通知を表示する
</label>

<button type="button" id="check-messages">
  新着メッセージを確認
</button>
```

## meta refresh / 自動リダイレクト

- 時間付きの `meta refresh` は、ページ全体の割り込みになりやすい
- F40 / F41 は 2.2.1 だけでなく 2.2.4 の失敗にもなる
- 自動再読み込みではなく、手動更新やサーバー側の即時リダイレクト（時間制限なし）を使う

```html
<!-- 悪い例：割り込みとなる自動再読み込み -->
<meta http-equiv="refresh" content="60">

<!-- 良い例：ユーザー操作で更新 -->
<button type="button">ページを更新</button>
```

> **参照**
> - [Failure F40: using meta redirect with a time limit](https://www.w3.org/WAI/WCAG22/Techniques/failures/F40)
> - [Failure F41: using meta refresh to reload the page](https://www.w3.org/WAI/WCAG22/Techniques/failures/F41)

## 2.2.1・2.2.2・3.2.5 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.2.1 タイミング調整可能 | 時間制限のオフ・調整・延長 | 制限時間そのもの |
| 2.2.2 一時停止・停止・非表示 | 動き・点滅・自動更新の制御 | 視覚的な並行更新 |
| 2.2.4 割り込み | 更新・通知の延期・抑制 | 集中を妨げないこと |
| 3.2.5 要求による変化 | ユーザー要求なしの大きな変化 | AAA、コンテキスト変化 |

- 2.2.2 は動きや自動更新の**見た目の制御**
- 2.2.4 は、更新や通知による**作業の中断を防ぐ**こと
- `meta refresh` は 2.2.1・2.2.4・3.2.5 の複数基準に影響しうる

> **参照**
> - [wcag/2-2-1-timing-adjustable.md — 2.2.1 タイミング調整可能](./2-2-1-timing-adjustable.md)
> - [wcag/2-2-2-pause-stop-hide.md — 2.2.2 一時停止・停止・非表示](./2-2-2-pause-stop-hide.md)
> - [WCAG 2.2 — Success Criterion 3.2.5 Change on Request](https://www.w3.org/TR/WCAG22/#change-on-request)

## よくある失敗例

- 読んでいる最中にニュースや株価が自動差し替わる
- 入力中にモーダルやトーストが連続表示される
- 通知をオフ・延期する設定がない
- 緊急ではないお知らせを緊急扱いにして常時割り込ませる
- `meta refresh` で定期的にページを再読み込みする

```javascript
// 悪い例：入力中でも強制的に通知を出す
setInterval(() => {
  showModal("おすすめキャンペーンがあります");
}, 30000);

// 良い例：設定がオンのときだけ、入力中は延期
function maybeNotify(message) {
  if (!settings.allowAlerts) return;
  if (document.activeElement.matches("input, textarea, select")) {
    queueForLater(message);
    return;
  }
  showToast(message);
}
```

## テスト・確認方法

### 手動テスト

- 自動更新、トースト、モーダル、チャット通知、`meta refresh` を洗い出す
- 各割り込みをオフまたは延期できるか確認
- 緊急アラートだけが例外扱いになっているか確認
- 入力中・読み上げ中に内容が勝手に変わらないか確認

```text
チェックリスト：
1. 自動の割り込み（更新・通知）があるか
2. オフまたは延期できるか
3. 手動更新の手段があるか
4. 緊急アラートの例外は妥当か
5. meta refresh / 自動再読み込みがないか
```

### 開発者ツールでの確認

```bash
# 自動更新・割り込みになりうる実装を確認
rg "meta.*refresh|setInterval|toast|alert\\(|showModal|notification" --glob "*.html" --glob "*.js" --glob "*.tsx" --glob "*.jsx"
```

### ACT Rules

- [Meta element has no refresh delay](https://www.w3.org/WAI/standards-guidelines/act/rules/bc659a/)
- [Meta element has no refresh delay (no exception)](https://www.w3.org/WAI/standards-guidelines/act/rules/bisz58/)

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.2.4 は、割り込みをユーザーが**延期または抑制**できるように求める達成基準（レベル AAA）
- 例外は、健康・安全・財産を守るための**緊急事態**の割り込みのみ
- 設定での延期（G75）、手動更新（G76）、任意アラート（SCR14）が主な実装方針
- `meta refresh` による自動再読み込みはよくある失敗
- 2.2.1（時間制限）・2.2.2（動きの制御）・3.2.5（要求による変化）と併せて確認する
