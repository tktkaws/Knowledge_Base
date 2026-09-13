# 4.1.3 ステータスメッセージ（AA）

## 達成基準の概要

- WCAG 2.2 達成基準 4.1.3「ステータスメッセージ」（Status Messages）
- レベル **AA**
- 原則4「堅牢（Robust）」> ガイドライン 4.1「互換性」に属する
- マークアップ言語で実装されたコンテンツにおいて、**ステータスメッセージ**が role やプロパティでプログラムで解釈できること
- 支援技術が、**フォーカスを受け取らずに**そのメッセージをユーザーへ提示できること
- 目的は、フォーカスを奪わない重要なコンテンツ変化に、視覚以外でも気づけるようにすること
- 新しいステータスメッセージの**作成を強制する基準ではない**。出すならプログラムで識別できるようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 4.1.3 Status Messages](https://www.w3.org/TR/WCAG22/#status-messages)
> - [Understanding SC 4.1.3 Status Messages](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html)

## 達成基準の原文（要約）

- マークアップ言語で実装されたコンテンツでは、ステータスメッセージが role またはプロパティによってプログラムで解釈でき、支援技術がフォーカスを受け取らずにユーザーへ提示できること

> **参照**
> - [WCAG 2.2 — Success Criterion 4.1.3](https://www.w3.org/TR/WCAG22/#status-messages)

## なぜ必要か

- スクリーンリーダー利用者は、画面の端に出た「保存しました」などに気づきにくい
- フォーカスを動かさないトーストやインラインメッセージは、視覚利用者には見えるが、読み上げられないことがある
- 適切な role / ライブリージョンがあれば、同等の情報を得られる
- 認知障害向け AT が、メッセージを遅延・抑制・強調するなど、好みに合わせて扱える余地も生まれる
- 作業を不要に中断せず、重要な変化だけ伝えられる

> **参照**
> - [Understanding SC 4.1.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html#intent)
> - [Understanding SC 4.1.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html#benefits)

## ステータスメッセージとは

- WCAG 用語としての条件は次の2つ
  1. 次のいずれかの情報を伝える
     - 操作の成功・結果
     - アプリの待機状態
     - 処理の進捗
     - エラーの存在
  2. **コンテキストの変化**によって届けられていない

```text
ステータスメッセージの例：
  「5件の結果が見つかりました」
  「カートに 5 点」
  「郵便番号の形式が正しくありません」
  「処理中…」
  「フォームを送信しました」
  「ページに 5 件のエラーがあります」

ステータスメッセージではない例：
  検索結果の一覧そのもの（結果リスト）
  エラーをダイアログで表示しフォーカスを移す
  アコーディオンやタブの開閉で出る内容
  回答に応じて追加される新しい入力欄そのもの
```

> **参照**
> - [WCAG 2.2 — Glossary: status message](https://www.w3.org/TR/WCAG22/#dfn-status-message)
> - [Understanding SC 4.1.3 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html#examples)

## コンテキストの変化との違い

- **コンテキストの変化**：フォーカス移動、ページ遷移、新しいビューポートなど、大きな変化
- コンテキストが変わると、支援技術はすでに変化を伝えやすい
- そのため、フォーカスを取るダイアログ内のエラーなどは、この基準の対象外
- 4.1.3 が扱うのは、**今いる場所のまま**追加・更新されるステータス情報

| パターン | 対象？ | 理由 |
|---|---|---|
| インラインの「送信成功」 | ○ | フォーカスを取らない成功メッセージ |
| `role="alert"` のエラー要約 | ○ | コンテキスト変化なしのエラー通知 |
| モーダルダイアログのエラー | × | フォーカス移動＝コンテキスト変化 |
| アコーディオン展開 | × | 状態変化は 4.1.2 側。ステータスメッセージ定義外 |
| 検索結果リスト | × | 結果そのものはステータスではない |
| 「18件ヒット」の一文 | ○ | 検索の結果件数はステータス |

> **参照**
> - [Understanding SC 4.1.3 — Changes that are not status messages](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html#examples)
> - [WCAG 2.2 — Glossary: changes of context](https://www.w3.org/TR/WCAG22/#dfn-changes-of-context)

## 状況別の十分な手法

| 状況 | 内容 | 主な手法 |
|---|---|---|
| **A** | 操作の成功・結果、アプリの状態 | **ARIA22**（`role="status"`）+ **G199** など |
| **B** | 提案・警告・エラーの存在 | **ARIA19**（`role="alert"` / ライブリージョン）+ G83 / G84 / G85 など |
| **C** | 処理の進捗 | **ARIA23**（`role="log"`）、**ARIA25**（進捗バーのライブリージョン）、**ARIA22**、**ARIA27**（`ariaNotify`） |

- エラーをコンテキスト変化なしで出すときだけ `alert` が必要
- フォーカスをエラーへ移す設計なら、4.1.3 の対象外になりうる（ただし UX・他基準は別途確認）

> **参照**
> - [Understanding SC 4.1.3 — Sufficient Techniques](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html#techniques)

## 手法1：成功・結果は role="status"（ARIA22）

- `role="status"` は暗黙で `aria-live="polite"`
- 現在の読み上げを終わらせてから通知する
- 検索件数、カート点数、送信成功などが典型

```html
<!-- 良い例：検索結果件数 -->
<div role="status" aria-live="polite" aria-atomic="true">
  5件の結果が見つかりました
</div>
<!-- 下に結果リスト。リスト自体はステータスではない -->
```

```html
<!-- 良い例：送信成功のトースト領域（空で先に置く） -->
<div id="form-status" role="status" aria-live="polite"></div>

<script>
  document.getElementById('form-status').textContent =
    'フォームを送信しました';
</script>
```

```html
<!-- 悪い例：成功メッセージがただの div -->
<div class="toast">保存しました</div>
<!-- role / aria-live なし。スクリーンリーダーは気づかない（F103） -->
```

> **参照**
> - [Technique ARIA22: Using role=status to present status messages](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA22)
> - [Technique G199: Providing success feedback when data is submitted successfully](https://www.w3.org/WAI/WCAG22/Techniques/general/G199)
> - [a11y/accessible-toast-alert.md — トーストとアラート](../a11y/accessible-toast-alert.md)

## 手法2：エラー・警告は role="alert"（ARIA19）

- `role="alert"` は暗黙で `aria-live="assertive"`
- 緊急性の高いエラー・警告向け
- 重要でないメッセージに `assertive` を多用すると、読み上げがうるさくなる

```html
<!-- 良い例：フォーム上部のエラー要約 -->
<div role="alert">
  ページに 2 件のエラーがあります。
  <ul>
    <li><a href="#email">メールアドレスの形式が正しくありません</a></li>
    <li><a href="#zip">郵便番号は必須です</a></li>
  </ul>
</div>
```

```html
<!-- 良い例：フィールド近くのエラー（コンテキスト変化なし） -->
<label for="zip">郵便番号</label>
<input id="zip" name="zip" type="text" aria-invalid="true" aria-describedby="zip-error">
<p id="zip-error" role="alert">形式が正しくありません。例：100-0001</p>
```

```html
<!-- 対象外になりやすい例：ダイアログがフォーカスを取る -->
<dialog open>
  <h2>入力エラー</h2>
  <p>郵便番号の形式が正しくありません</p>
</dialog>
<!-- コンテキスト変化のため 4.1.3 の対象外。別途ダイアログの a11y は必要 -->
```

> **参照**
> - [Technique ARIA19: Using ARIA role=alert or Live Regions to Identify Errors](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA19)
> - [wcag/3-3-1-error-identification.md — 3.3.1 エラーの特定](./3-3-1-error-identification.md)

## 手法3：進捗・逐次更新（ARIA23 / ARIA25）

- 長い処理の進捗、ログの追記などに使う
- `role="log"` は逐次情報の更新向け
- 進捗バーは、ライブリージョンで断続的に状態を伝える

```html
<!-- 良い例：処理中ステータス -->
<div role="status" aria-live="polite">
  アップロード中です。しばらくお待ちください。
</div>
```

```html
<!-- 良い例：進捗の断続アナウンス -->
<div
  role="status"
  aria-live="polite"
  aria-atomic="true"
  id="upgrade-progress"
>
  アップグレード進捗：40%
</div>
<div
  role="progressbar"
  aria-valuemin="0"
  aria-valuemax="100"
  aria-valuenow="40"
  aria-labelledby="upgrade-progress"
></div>
```

```html
<!-- 良い例：チャット風の逐次ログ -->
<div role="log" aria-live="polite" aria-relevant="additions">
  <p>12:01 接続しました</p>
  <p>12:02 ファイルを受信しました</p>
</div>
```

> **参照**
> - [Technique ARIA23: Using role=log to identify sequential information updates](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA23)
> - [Technique ARIA25: Using an ARIA live region to convey the status of a progress bar](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA25)

## 実装上の注意

### ライブリージョンは先に DOM へ置く

- テキスト入りの要素を後から丸ごと追加すると、通知されないことがある
- 空の `status` / `alert` 領域を先に置き、あとからテキストを入れる

### 文言の更新はまとめて伝える

- 「0 点」→「3 点」で数字だけ更新すると、「3」だけ聞こえることがある
- `aria-atomic="true"` で領域全体を読み上げさせる
- 必要なら「カート内」など文脈語を足す

### メッセージが消えるとき

- 「処理中」が消えるだけでは、非視覚利用者は完了に気づかないことがある
- 可能なら「準備ができました」などの終了メッセージを出す

### アイコンだけのステータス

- アイコンや音だけでもステータスになりうる
- **1.1.1** のテキスト代替と、4.1.3 の適切な role の両方が必要

```html
<!-- 良い例：カート点数の全体更新 -->
<span role="status" aria-live="polite" aria-atomic="true">
  カート内 3 点
</span>
```

```html
<!-- 悪い例：数字だけ差し替え -->
<span>カート内 <span id="count">3</span> 点</span>
<!-- live なし。あっても count だけ変わると文脈不足になりやすい -->
```

> **参照**
> - [Understanding SC 4.1.3 — Modification / Removal of status text](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html#examples)
> - [a11y/aria-live.md — aria-live](../a11y/aria-live.md)

## role / aria-live の使い分け

| 用途 | 推奨 | 暗黙の live |
|---|---|---|
| 成功・件数・一般ステータス | `role="status"` | polite |
| エラー・緊急性の高い警告 | `role="alert"` | assertive |
| 逐次ログ | `role="log"` | polite |
| カスタム | `aria-live="polite"` / `"assertive"` | 明示値 |

- 重要でも時間的に急がない内容に `assertive` を乱用しない
- チャットティ（読み上げ過多）になると、かえって使いづらい

## 4.1.2・3.3.1 との関係

| 達成基準 | 関係 |
|---|---|
| **4.1.2 名前・役割・値** | UI コンポーネント自体の名前・役割・状態 |
| **4.1.3 ステータスメッセージ** | フォーカスを取らないステータス更新の通知 |
| **3.3.1 エラーの特定** | エラー内容の特定。4.1.3 はその通知手段 |

- アコーディオンの `aria-expanded` 更新は 4.1.2
- 「保存しました」トーストは 4.1.3
- エラーメッセージの文言は 3.3.1、フォーカスなしで伝えるなら 4.1.3 も絡む

> **参照**
> - [wcag/4-1-2-name-role-value.md — 4.1.2](./4-1-2-name-role-value.md)
> - [wcag/3-3-1-error-identification.md — 3.3.1](./3-3-1-error-identification.md)

## よくある失敗例（F103 など）

- ステータス文言を出すが、`role` / `aria-live` がない（F103）
- 成功トーストが単なる CSS アニメーションの `div`
- 検索後に件数が出るが、ライブリージョンではない
- スピナー画像だけで「処理中」が伝わらない
- 重要でない情報に `role="alert"` を多用して読み上げが中断され続ける
- ライブリージョンを、テキスト入りの状態で後から DOM 追加する

```html
<!-- 失敗例 -->
<div class="banner banner--success">登録が完了しました</div>
```

```html
<!-- 改善例 -->
<div role="status" aria-live="polite">登録が完了しました</div>
```

> **参照**
> - [Failure F103: Status messages that cannot be programmatically determined](https://www.w3.org/WAI/WCAG22/Techniques/failures/F103)

## テスト・確認方法

### 手動テスト

- 成功・エラー・待機・進捗のメッセージを洗い出す
- それぞれがフォーカスを取るか確認する（取るなら 4.1.3 対象外の可能性）
- 取らない場合、`status` / `alert` / `log` / `aria-live` があるか確認する
- スクリーンリーダーで、フォーカスを動かさずに読み上げられるか確認する
- 件数更新で文脈ごと聞こえるか（`aria-atomic`）を確認する

```text
チェックリスト：
1. ステータスメッセージ（成功・結果・待機・進捗・エラー）を洗い出したか
2. コンテキスト変化なしで出るものか切り分けたか
3. 適切な role / aria-live があるか
4. 成功は status（polite）、緊急エラーは alert になっているか
5. ライブリージョンを事前配置しているか
6. 更新時に文脈不足の読み上げになっていないか
7. アイコンのみのステータスにテキスト代替があるか
8. assertive の乱用でチャットティになっていないか
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)
> - [a11y/screen-reader-basics.md — スクリーンリーダー基礎](../a11y/screen-reader-basics.md)

## まとめ

- 4.1.3 は、フォーカスを取らないステータスメッセージを支援技術へ伝える達成基準（レベル AA）
- 対象は、成功・結果・待機・進捗・エラーで、かつコンテキスト変化ではないもの
- 成功は `role="status"`、エラーは `role="alert"`、進捗・ログは `log` / ライブリージョンが典型
- ダイアログやアコーディオンなど、フォーカス・状態変化で伝わるものは対象外になりやすい
- メッセージを出せ、という基準ではなく、出すならプログラムで識別できるようにする
- role なしのトースト表示は代表的な失敗（F103）
