# 2.2.5 再認証（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 2.2.5「再認証」（Re-authenticating）
- レベル **AAA**（最高の適合レベル）
- 原則2「操作可能（Operable）」> ガイドライン 2.2「十分な時間」に属する
- 認証済みセッションが切れたあと、再認証すれば**データを失わずに活動を続行**できること
- セッションタイムアウトや再ログインがあっても、入力内容や進捗を復元する

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.5 Re-authenticating](https://www.w3.org/TR/WCAG22/#re-authenticating)
> - [Understanding SC 2.2.5 Re-authenticating](https://www.w3.org/WAI/WCAG22/Understanding/re-authenticating.html)

## 達成基準の原文（要約）

- 認証済みセッションの有効期限が切れた場合、ユーザーは再認証後に、**データの損失なく**活動を続行できること

> **参照**
> - [WCAG 2.2 — Success Criterion 2.2.5](https://www.w3.org/TR/WCAG22/#re-authenticating)

## 対象となる状況

- 無操作によるセッションタイムアウト
- セキュリティ上の理由による強制ログアウト
- 別端末からのログイン検知によるセッション切断
- 不審な活動を検知したときの再認証要求
- 取引や入力の途中でログイン状態が切れるケース全般

```text
対象の例：
  購入手続きの途中でセッション切れ
  長いアンケート入力中のタイムアウト
  メール作成中の再認証要求

対象外になりやすい例：
  最初から未ログインで行う閲覧のみのページ
  再認証後に復元すべき入力データがない単純な閲覧
```

> **参照**
> - [Understanding SC 2.2.5 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/re-authenticating.html#intent)

## なぜ必要か

- 障害のあるユーザーは、活動完了により長い時間がかかることがある
- 認知障害のあるユーザーは、質問票の読解・回答に時間がかかる
- スクリーンリーダー利用者は、複雑なフォームの移動・入力に時間がかかる
- 運動障害のあるユーザーや代替入力装置の利用者は、入力完了に時間がかかる
- 途中でログアウトされてデータが消えると、最初からやり直しになり負担が大きい

> **参照**
> - [Understanding SC 2.2.5 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/re-authenticating.html#benefits)

## 基本方針：再認証後に同じ状態へ戻す

- セッション切れ前の**入力データ**を保持する
- セッション切れ前の**手順・画面位置**を保持する
- 再ログイン後に、同じ段階から続行できること
- ユーザーに「最初からやり直してください」とさせないこと

```text
望ましい流れ：
1. ユーザーがフォームや購入手続きを入力中
2. セッションが切れる / 再認証が必要になる
3. 再認証を完了する
4. 同じ入力内容・同じ段階へ復帰する
5. データの損失なく続行できる
```

## データを保存して再認証後に使う（G105）

- サーバー側に一時保存する
- 安全な方法でクライアント側に保持する
- セッション切れ後も、再認証完了までデータを破棄しない
- センシティブな情報は暗号化し、保存期間と削除方針を明確にする

```javascript
// 良い例：入力途中を下書きとして保存
form.addEventListener("input", () => {
  const draft = Object.fromEntries(new FormData(form));
  saveDraftToServer(draft); // サーバーに一時保存
});

// 再認証後
async function restoreAfterLogin() {
  const draft = await fetchDraft();
  if (!draft) return;
  for (const [name, value] of Object.entries(draft)) {
    const field = form.elements.namedItem(name);
    if (field) field.value = value;
  }
}
```

```html
<!-- 良い例：再認証後に同じチェックアウト段階へ戻る -->
<p>セッションが切れました。再ログイン後、購入手続きを続行できます。</p>
<a href="/login?return=/checkout/step-2">ログインして続行</a>
```

> **参照**
> - [Technique G105: Saving data so that it can be used after a user re-authenticates](https://www.w3.org/WAI/WCAG22/Techniques/general/G105)

## 再認証ページにデータを引き継ぐ（G181）

- 再認証フォームへ、ユーザーデータを hidden や暗号化データとして渡す手法
- 再ログイン後に同じページ状態を復元するために使う
- クライアントへ渡す場合は改ざん防止・暗号化が必要
- 可能な限りサーバー側セッション／下書き ID で管理する方が安全

```html
<!-- 概念例：再認証ページへ状態を引き継ぐ -->
<form action="/reauth" method="post">
  <input type="hidden" name="draft_id" value="abc123">
  <input type="hidden" name="step" value="2">
  <label for="password">パスワード</label>
  <input id="password" name="password" type="password" autocomplete="current-password">
  <button type="submit">再認証して続行</button>
</form>
```

> **参照**
> - [Technique G181: Encoding user data as hidden or encrypted data in a re-authorization page](https://www.w3.org/WAI/WCAG22/Techniques/general/G181)

## 購入手続きの例

- チェックアウト途中で席を外し、セッションが切れる
- 戻って再ログインすると、同じ商品・同じ段階・同じ入力内容で再開できる
- サーバーが一時的に送信内容を保持し、再認証後に状態を復元する

```text
良い例：
  カート内容・配送先・支払い方法の途中入力が残る
  ステップ2（配送先）から再開できる

悪い例：
  再ログイン後にカートが空になる
  配送先入力が消え、最初のステップからやり直し
```

> **参照**
> - [Understanding SC 2.2.5 — shopping site checkout example](https://www.w3.org/WAI/WCAG22/Understanding/re-authenticating.html#examples)

## メール作成などの進行中作業

- 作成中の下書きウィンドウを残したまま、別ウィンドウで再認証する設計も有効
- 再認証後、作成中の内容を失わずに送信できること
- SPA でも、ルート遷移や再ログイン後に下書きを復元する

```html
<!-- 良い例：再認証を別ウィンドウで行い、下書きを維持 -->
<p>セッションがまもなく切れます。</p>
<a href="/reauth" target="_blank" rel="noopener">
  別ウィンドウで再認証する
</a>
<textarea id="mail-body">作成中のメール本文...</textarea>
```

> **参照**
> - [Understanding SC 2.2.5 — email program example](https://www.w3.org/WAI/WCAG22/Understanding/re-authenticating.html#examples)

## セキュリティとアクセシビリティの両立

- セッションタイムアウト自体は禁止されない
- セキュリティ上必要な再認証も許容される
- 要件は「再認証後にデータを失わないこと」
- センシティブデータの保存は、暗号化・短期間保持・再認証後の厳密な紐付けが必要
- 2.2.1「タイミング調整可能」も併せて満たす必要がある場合がある

```text
両立のポイント：
  セッション切れは許容
  入力データの安全な一時保存は必須
  再認証後の復元先を明確にする
  不要になった下書きは削除する
```

> **参照**
> - [wcag/2-2-1-timing-adjustable.md — 2.2.1 タイミング調整可能](./2-2-1-timing-adjustable.md)

## 2.2.1・2.2.3・2.2.6 との関係

| 達成基準 | 焦点 | 関係 |
|---|---|---|
| 2.2.1 タイミング調整可能 | 時間制限のオフ・調整・延長 | セッション制限そのものの扱い |
| 2.2.3 タイミング非依存 | タイミングを不可欠にしない | AAA、時間制限を避ける |
| 2.2.5 再認証 | 再認証後のデータ保持 | セッション切れ後の続行 |
| 2.2.6 タイムアウト | タイムアウトの事前告知 | AAA、ユーザーデータの消失リスク |

- 2.2.5 は「切れたあとにどう戻るか」
- 2.2.1 は「切れる前に延長できるか」
- 2.2.6 は「切れる可能性があることを事前に知らせるか」
- 理想は、切れにくく（2.2.1 / 2.2.3）、切れても復元できる（2.2.5）設計

> **参照**
> - [wcag/2-2-3-no-timing.md — 2.2.3 タイミング非依存](./2-2-3-no-timing.md)
> - [WCAG 2.2 — Success Criterion 2.2.6 Timeouts](https://www.w3.org/TR/WCAG22/#timeouts)

## よくある失敗例（F12）

- セッションタイムアウト後、再ログインしても入力内容が消える
- 購入・申請の進捗が最初からになる
- 再認証ページへ遷移した時点で POST データやフォーム状態を破棄する
- 「もう一度やり直してください」だけを表示し、復元手段がない

```javascript
// 悪い例：タイムアウト時に下書きを削除
function onSessionTimeout() {
  clearDraft();
  location.href = "/login";
}

// 良い例：下書きを残して再認証へ誘導
function onSessionTimeout() {
  preserveDraft();
  location.href = "/login?return=/form&draft=1";
}
```

> **参照**
> - [Failure F12: session time limit without a mechanism to continue after re-authentication](https://www.w3.org/WAI/WCAG22/Techniques/failures/F12)

## 実装時の注意点

- パスワードやクレジットカード番号など、保存すべきでない項目は扱いに注意
- 再認証後に復元する範囲を明確にする（例：配送先は復元、CVV は再入力）
- 複数タブ・複数端末での下書き競合を考慮する
- 再認証後のフォーカス位置や画面位置も、可能なら元の段階へ戻す
- ユーザーに「下書きが保存されている」ことを伝えると安心につながる

```html
<!-- 良い例：復元できることを明示 -->
<p role="status">
  入力内容は下書きとして保存されています。再ログイン後に続きから再開できます。
</p>
```

## テスト・確認方法

### 手動テスト

- ログインが必要な長いフォーム・購入・申請フローを開始する
- 入力途中でセッションを切る（待機、別端末ログイン、手動ログアウト相当）
- 再認証する
- 入力内容・手順・状態が復元されるか確認
- センシティブ項目の扱いが妥当かも確認

```text
チェックリスト：
1. 認証セッションが切れるケースがあるか
2. 再認証の手段があるか
3. 再認証後に入力データが残るか
4. 同じ段階・同じ画面から続行できるか
5. 最初からやり直しになっていないか
```

### 開発者ツールでの確認

```bash
# セッション・下書き・再認証まわりの実装を確認
rg "session|timeout|reauth|draft|localStorage|sessionStorage" --glob "*.js" --glob "*.tsx" --glob "*.jsx" --glob "*.php"
```

> **参照**
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 2.2.5 は、認証セッション切れ後に再認証すれば、**データを失わず活動を続行**できることを求める達成基準（レベル AAA）
- セッションタイムアウト自体は禁止されないが、入力・進捗の破棄は不適合になりやすい
- サーバー側の下書き保存（G105）や、再認証ページへの状態引き継ぎ（G181）が主な手法
- 購入・申請・メール作成など、途中入力があるフローが重点対象
- 2.2.1（延長）・2.2.3（時間非依存）・2.2.6（タイムアウト告知）と併せて設計する
