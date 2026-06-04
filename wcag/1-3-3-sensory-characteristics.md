# 1.3.3 感覚的な特徴（A）

## 達成基準の概要

- WCAG 2.2 達成基準 1.3.3「感覚的な特徴」（Sensory Characteristics）
- レベル A（最低限の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.3「適応可能」に属する
- コンテンツの**理解・操作に関する指示**が、形状・色・サイズ・視覚的位置・向き・音などの**感覚的特徴だけ**に依存しないこと
- 視覚・聴覚に依存しない**名前・ラベル・テキスト**で操作対象を特定できるようにする

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.3 Sensory Characteristics](https://www.w3.org/TR/WCAG22/#sensory-characteristics)
> - [Understanding SC 1.3.3 Sensory Characteristics](https://www.w3.org/WAI/WCAG22/Understanding/sensory-characteristics.html)

## 達成基準の原文（要約）

- コンテンツの理解・操作のための**指示**は、コンポーネントの感覚的特徴（形状、色、サイズ、視覚的位置、向き、音）**のみ**に依存してはならない
- 色に関する要件はガイドライン 1.4 を参照（1.4.1 色の使用）

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.3](https://www.w3.org/TR/WCAG22/#sensory-characteristics)

## 感覚的特徴とは

- ユーザーが知覚・認識する**物理的・感覚的な属性**
- WCAG で明示される種類
  - **形状**（丸いボタン、三角のアイコン）
  - **色**（赤いリンク、青いアイコン）
  - **サイズ**（大きいボタン、小さい文字）
  - **視覚的位置**（右下、画面上部、左カラム）
  - **向き**（横向き、縦向き）
  - **音**（ピッという音、特定の効果音）
- これらは**補助的な手がかり**として使ってよい。**唯一の**識別手段にしてはならない

> **参照**
> - [Understanding SC 1.3.3 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/sensory-characteristics.html#intent)

## なぜ必要か

- 視覚障害者・ロービジョンは**形状・位置・色**を知覚できない場合がある
- スクリーンリーダーは**ラベル・名前・テキスト**で操作対象を特定する
- 聴覚障害者は**音**だけの指示を理解できない
- 認知障害のあるユーザーにも、**名前付きの指示**の方が理解しやすい

> **参照**
> - [Understanding SC 1.3.3 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/sensory-characteristics.html#benefits)

## 1.4.1 色の使用との関係

| 達成基準 | 焦点 |
|---|---|
| 1.3.3 感覚的な特徴 | **指示文**が感覚だけに依存しない |
| 1.4.1 色の使用 | **情報そのもの**が色だけに依存しない |

- 1.3.3：「赤いボタンをクリック」→ 指示が色だけ
- 1.4.1：必須項目が赤色だけで示されている → 情報が色だけ
- 両方違反しうる。対策は**ラベル・記号・テキスト**の併用

```html
<!-- 1.3.3 + 1.4.1 の両方に問題：色だけの指示と情報 -->
<p>赤い項目は必須です。赤い送信ボタンを押してください。</p>
<button style="background: red;">送信</button>

<!-- 良い例：名前・記号で特定 -->
<p>必須項目は <span aria-hidden="true">*</span> で示しています。</p>
<button type="submit">送信</button>
<p>フォーム下部の「送信」ボタンを押して完了してください。</p>
```

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.1 Use of Color](https://www.w3.org/TR/WCAG22/#use-of-color)
> - [wcag/1-3-1-info-and-relationships.md — 色の手がかり](./1-3-1-info-and-relationships.md)

## 形状・位置だけの指示（Failure F14）

- 「丸いボタン」「右のボタン」「下のリンク」だけでは不適合
- **アクセシブルな名前**（ラベル、`aria-label`、可視テキスト）を併記する

```html
<!-- 悪い例：形状と位置だけ（F14） -->
<p>右側の丸いボタンをクリックして送信してください。</p>
<button class="round-btn" style="float: right;" aria-label=""></button>

<!-- 悪い例：位置だけ -->
<p>画面右下の矢印を選択して次へ進んでください。</p>
<a href="/next" class="arrow-icon-bottom-right" aria-label="次へ"></a>

<!-- 良い例：ラベル名で特定。位置・色は補助 -->
<p>
  次のセクションへ進むには、画面右下にある
  「次へ」とラベル付けされた緑の矢印アイコンを選択してください。
</p>
<a href="/next" aria-label="次へ">
  <img src="arrow-next.svg" alt="">
  <span>次へ</span>
</a>
```

> **参照**
> - [Failure F14: Identifying content only by its shape or location](https://www.w3.org/WAI/WCAG22/Techniques/failures/F14)
> - [Understanding SC 1.3.3 — Example 2: Survey with green arrow](https://www.w3.org/WAI/WCAG22/Understanding/sensory-characteristics.html#examples)

## 色・形状だけの情報（Failure F26, スケジュール例）

- 凡例や指示が**色と形状のみ**でイベントを区別していると不適合
- テキストラベル・パターン・アイコンの代替テキストを追加

```html
<!-- 悪い例：色と形状だけの凡例（Understanding の Example 1） -->
<table>
  <caption>競技スケジュール</caption>
  <!-- 試合データ -->
</table>
<p>
  青いダイヤのアイコンは A グラウンド、
  緑の丸のアイコンは B グラウンドの試合です。
</p>

<!-- 良い例：テキストラベルで区別 -->
<table>
  <caption>競技スケジュール</caption>
  <tbody>
    <tr>
      <td><span aria-hidden="true">◆</span> A グラウンド — 10:00 男子100m</td>
    </tr>
    <tr>
      <td><span aria-hidden="true">●</span> B グラウンド — 10:00 女子100m</td>
    </tr>
  </tbody>
</table>
<p>凡例：◆ A グラウンド、● B グラウンド</p>
```

> **参照**
> - [Failure F26: Using a graphical symbol alone to convey information](https://www.w3.org/WAI/WCAG22/Techniques/failures/F26)

## 音だけの指示

- 「ピッという音の後に操作」「成功時のチャイムで確認」だけでは不適合
- **視覚的・テキストのフィードバック**を併用

```html
<!-- 悪い例：音だけの完了通知 -->
<p>送信が完了すると通知音が鳴ります。</p>

<!-- 良い例：テキスト + 視覚的フィードバック -->
<p>送信が完了すると、画面上部に「送信完了」と表示されます。</p>
<div role="status" aria-live="polite" id="submit-status"></div>
```

```javascript
// 良い例：音は補助、status でテキスト通知
function onSubmitSuccess() {
  document.getElementById('submit-status').textContent = '送信完了';
  // 任意：補助的な通知音
}
```

> **参照**
> - [WCAG 2.2 — Success Criterion 4.1.3 Status Messages](https://www.w3.org/TR/WCAG22/#status-messages)

## 「上」「下」「左」「右」の指示

- 読み上げ順序と一致し、**曖昧でない**場合は「下のリンク」等が許容される言語もある（日本語を含む）
- 条件
  - 参照先が**読み上げ順序上の適切な位置**にある
  - 参照が**一意**で曖昧でない
  - 可能なら**リンクテキスト・見出し名**でも特定する

```html
<!-- 許容されうる例：読み上げ順と一致、リンクテキストで特定可能 -->
<h2>関連資料</h2>
<ul>
  <li><a href="/guide">利用ガイド</a></li>
  <li><a href="/faq">よくある質問</a></li>
</ul>
<p>詳細は上記の「利用ガイド」をご覧ください。</p>

<!-- 悪い例：位置だけで複数候補がある -->
<p>右のボタンをクリックしてください。</p>
<button>キャンセル</button>
<button>送信</button>
<!-- → どちらが「右」か、視覚以外では特定できない -->

<!-- 良い例：ボタン名で特定 -->
<p>「送信」ボタンをクリックして完了してください。</p>
<button type="button">キャンセル</button>
<button type="submit">送信</button>
```

> **参照**
> - [Understanding SC 1.3.3 — Intent (above/below)](https://www.w3.org/WAI/WCAG22/Understanding/sensory-characteristics.html#intent)

## アイコンボタン・グラフィックのみの UI

- アイコンだけのボタンは**accessible name**（`aria-label`、可視テキスト、`<title>` は補助）が必須
- 指示文でもアイコンの**名前**を参照する（Technique G96）

```html
<!-- 悪い例：アイコンのみ、指示も感覚だけ（F26, F111） -->
<button>
  <svg aria-hidden="true"><!-- ゴミ箱アイコン --></svg>
</button>
<p>ゴミ箱のアイコンをクリックして削除してください。</p>

<!-- 良い例：aria-label + 指示で名前を参照 -->
<button type="button" aria-label="削除">
  <svg aria-hidden="true"><!-- ゴミ箱アイコン --></svg>
</button>
<p>「削除」ボタンをクリックして項目を削除してください。</p>
```

```html
<!-- 良い例：アイコン + 可視テキストラベル -->
<button type="button">
  <img src="search.svg" alt="">
  検索
</button>
```

> **参照**
> - [Technique G96: Providing textual identification of items that otherwise rely only on sensory information](https://www.w3.org/WAI/WCAG22/Techniques/general/G96)
> - [Failure F111: Control with visible label text but no accessible name](https://www.w3.org/WAI/WCAG22/Techniques/failures/F111)

## フォーム・操作の指示

- 入力欄の特定は**ラベル名**で行う
- エラー修正の案内も感覚だけにしない

```html
<!-- 悪い例 -->
<p>赤枠の欄を修正してください。</p>

<!-- 良い例 -->
<p>「メールアドレス」欄の入力内容を修正してください。</p>
<label for="email">メールアドレス</label>
<input type="email" id="email" aria-invalid="true" aria-describedby="email-error">
<span id="email-error">有効なメールアドレスを入力してください。</span>
```

```html
<!-- 悪い例：サイズだけの指示 -->
<p>大きいボタンで確定してください。</p>

<!-- 良い例 -->
<p>「注文を確定」ボタンを押してください。</p>
<button type="submit">注文を確定</button>
```

> **参照**
> - [a11y/accessible-form.md — フォームのアクセシビリティ](../a11y/accessible-form.md)

## 物理ハードウェアの例外

- 本基準は**画面上に表示されるコントロール**が主な対象
- **物理的なハードウェア**（キオスク、専用端末）の操作説明では、**触覚的な手がかり**（矢印型キー、右側の丸いキー等）を記述してよい
- ウェブページ上の UI には触覚手がかりは存在しないため、テキスト・ラベルでの説明が必要

> **参照**
> - [Understanding SC 1.3.3 — Intent (physical hardware)](https://www.w3.org/WAI/WCAG22/Understanding/sensory-characteristics.html#intent)

## 感覚的手がかりを併用する場合のベストプラクティス

- 形状・色・位置は**認知負荷を下げる補助**として有効
- **必ず**テキスト・ラベル・名前でも同じ情報を提供
- デザインシステムではアイコンに**常にテキストラベル**または `aria-label` を付与

| 感覚的手がかり | 併用すべき情報 |
|---|---|
| 赤色 | 「必須」テキスト、`*` 記号、`aria-required` |
| 右下の位置 | 「次へ」ラベル、`aria-label="次へ"` |
| 丸い形状 | 「送信」ボタンの accessible name |
| 大きいサイズ | 「注文を確定」等のボタン名 |
| 通知音 | `role="status"` のテキストメッセージ |

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.3.1 情報および関係性（A） | ラベルと入力欄のプログラム的関連付け |
| 1.4.1 色の使用（A） | 情報が色だけに依存しない |
| 2.5.3 名前（name）のラベル（A） | 可視ラベルと accessible name の一致 |
| 3.3.2 ラベルまたは説明（A） | フォームのラベル・説明 |
| 4.1.2 名前・役割・値（A） | コントロールの accessible name |

> **参照**
> - [WCAG 2.2 — Success Criterion 2.5.3 Label in Name](https://www.w3.org/TR/WCAG22/#label-in-name)
> - [wcag/1-3-1-info-and-relationships.md](./1-3-1-info-and-relationships.md)

## テスト・確認方法

### 自動テスト

- アイコンボタンの accessible name の有無
- リンク・ボタンの空 `aria-label`
- 一部の指示文のパターンマッチ（限定的）
- axe DevTools、WAVE 等で部分的に検出可能

### 手動テスト

- ページ内の**操作指示・凡例**を抽出し、感覚だけで特定できるか確認
- **CSS を無効化**または**モノクロ表示**で、指示に従えるか
- **スクリーンリーダー**で指示を聞き、操作対象を特定できるか
- **音量ゼロ**で、音に依存した指示がないか確認
- 複数の類似ボタンがある画面で「右のボタン」等が一意に特定できるか

```bash
# accessible name が空のボタン・リンクを検出（開発者ツール Console）
[...document.querySelectorAll('button, a[href], [role="button"]')].filter(el => {
  const name = el.getAttribute('aria-label') || el.textContent.trim();
  return !name && !el.getAttribute('aria-labelledby');
});
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.3.3 は**操作・理解の指示**が感覚的特徴**だけ**に依存しない達成基準（レベル A）
- 形状・色・サイズ・位置・向き・音は**補助**として使ってよい。**名前・ラベル・テキスト**を必ず併用
- 「右の丸い赤いボタン」→「送信」ボタン、のように**accessible name**で特定する
- 色に関する情報要件は **1.4.1** が別途担当。両方を確認
- アイコンのみの UI には `aria-label` または可視ラベル（G96）
- 読み上げ順と一致する「下のリンク」は条件付きで許容。複数候補がある場合は名前で特定
