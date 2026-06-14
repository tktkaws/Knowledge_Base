# 1.4.1 色の使用（A）

## 達成基準の概要

- WCAG 2.2 達成基準 1.4.1「色の使用」（Use of Color）
- レベル A（最低限の適合レベル）
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.4「判別可能」に属する
- **色だけ**を視覚的手段として、情報の伝達・操作の指示・応答の促し・要素の区別に使わない
- 色の使用自体は**推奨**。色以外の視覚的手がかりと**併用**する

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.1 Use of Color](https://www.w3.org/TR/WCAG22/#use-of-color)
> - [Understanding SC 1.4.1 Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html)

## 達成基準の原文（要約）

- **色**を、情報の伝達・操作の指示・応答の促し・視覚要素の区別における**唯一の視覚的手段**として使用してはならない
- 色の知覚に特化した要件。プログラム的アクセスは 1.3.1 等が担当

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.1](https://www.w3.org/TR/WCAG22/#use-of-color)

## なぜ必要か

- **色覚特性**（色弱・色盲）のあるユーザーは、特定の色の区別が困難
- 高齢者は色の識別能力が低下する場合がある
- **モノクロ・限色ディスプレイ**では色情報が失われる
- スクリーンリーダー利用者には**視覚的な色**は伝わらない（本基準は**視力のある**色識別困難者向け）

> **参照**
> - [Understanding SC 1.4.1 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html#benefits)

## 1.3.3・1.4.3 との関係

| 達成基準 | 焦点 | 例 |
|---|---|---|
| 1.3.3 感覚的な特徴 | **指示文**が感覚だけに依存 | 「赤いボタンを押す」 |
| 1.4.1 色の使用 | **情報・UI**が色だけに依存 | 必須項目が赤ラベルのみ |
| 1.4.3 コントラスト（最低限） | **読みやすさ**（輝度差） | テキスト 4.5:1 |

- 1.4.1 と 1.4.3 は**独立**。コントラスト比を満たしても、色だけが識別手段なら 1.4.1 違反
- 1.4.1 は**視覚的な代替**が必要。1.1.1 / 4.1.2 は支援技術向け（別要件）

> **参照**
> - [wcag/1-3-3-sensory-characteristics.md — 1.3.3 感覚的な特徴](./1-3-3-sensory-characteristics.md)
> - [a11y/color-contrast.md — 色だけで情報を伝える](../a11y/color-contrast.md)

## 色相と輝度（明度）の違い

- **色相（hue）だけ**の区別 → 追加の視覚的手がかりが必要
- **輝度（lightness）も大きく異なる**場合 → 3:1 以上のコントラスト比があれば、**追加の視覚的区別**として認められる場合がある
- 例：薄い緑と濃い赤（色相 + 輝度の両方が異なる）→ 3:1 以上なら可

```
色相のみの区別（NG の可能性）：
  赤 #FF0000 vs 緑 #00FF00 → 輝度が近いと区別困難

色相 + 輝度の区別（追加手がかりとして可の可能性）：
  薄い緑 #90EE90 vs 濃い赤 #8B0000 → 輝度差が大きい
```

- **例外**：有効/無効の枠線が「緑=OK・赤=NG」だけ、のように**特定の色の知覚**に依存する場合は、コントラスト比に関わらず**追加の視覚的手がかり**が必要

> **参照**
> - [Understanding SC 1.4.1 — Note on lightness difference](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html#intent)

## 対象となる4つの用途

| 用途 | 説明 | 例 |
|---|---|---|
| 情報の伝達 | 色で意味を持たせる | グラフの系列色、凡例 |
| 操作の指示 | 色でアクションを示す | 「赤いボタンをクリック」 |
| 応答の促し | 色で入力を促す | 未入力欄のハイライト |
| 視覚要素の区別 | 色で要素を区別 | 必須/任意、エラー/正常 |

## フォーム：必須項目・エラー（Failure F81）

- 必須項目やエラー欄を**色の違いだけ**で示すのは不適合
- **記号**（`*`）、**アイコン**、**テキストメッセージ**を併用

```html
<!-- 悪い例：赤色ラベルのみで必須を示す（F81） -->
<style>.required { color: red; }</style>
<label class="required" for="email">メールアドレス</label>
<input type="email" id="email">

<!-- 悪い例：赤枠のみでエラー（F81） -->
<input type="email" style="border: 2px solid red;">

<!-- 良い例：色 + 記号 + テキスト + プログラム的属性（G205, G182） -->
<p>必須項目は <span aria-hidden="true">*</span> で示しています。</p>
<label for="email">
  メールアドレス <span aria-hidden="true">*</span>
  <span class="sr-only">（必須）</span>
</label>
<input type="email" id="email" required aria-required="true">

<label for="email2">メールアドレス</label>
<input
  type="email"
  id="email2"
  aria-invalid="true"
  aria-describedby="email-error"
  style="border: 2px solid #d32f2f;"
>
<p id="email-error">
  <span aria-hidden="true">⚠</span>
  メールアドレスの形式が正しくありません
</p>
```

> **参照**
> - [Failure F81: Identifying required or error fields using color differences only](https://www.w3.org/WAI/WCAG22/Techniques/failures/F81)
> - [Technique G205: Including a text cue for colored form control labels](https://www.w3.org/WAI/WCAG22/Techniques/general/G205)
> - [Technique G182: Additional visual cues when text color differences convey information](https://www.w3.org/WAI/WCAG22/Techniques/general/G182)
> - [a11y/accessible-form.md — 必須フィールドの示し方](../a11y/accessible-form.md)

## リンク（Failure F73, Technique G183）

- リンクを**色の違いだけ**で通常テキストと区別するのは不適合（F73）
- 対策
  - **下線**を付ける（最も一般的）
  - 下線なしの場合：リンク色と通常テキスト色のコントラスト比 **3:1 以上** + **ホバー時の追加手がかり**（G183）

```html
<!-- 悪い例：色だけでリンクを区別（F73） -->
<p>
  詳細は<a href="/about" style="color: #0066cc; text-decoration: none;">会社概要</a>をご覧ください。
  <!-- 通常テキスト #333 とリンク #0066cc の色差が小さい、下線なし -->
</p>

<!-- 良い例：下線で区別 -->
<p>
  詳細は<a href="/about">会社概要</a>をご覧ください。
</p>

<!-- 良い例：下線なし + 3:1 色差 + ホバーで下線（G183） -->
<style>
  .content a {
    color: #0056b3; /* 通常テキスト #333333 との比 3:1 以上 */
    text-decoration: none;
  }
  .content a:hover,
  .content a:focus {
    text-decoration: underline;
  }
</style>
```

### `:visited` リンクについて

- 訪問済みリンクの色は**ユーザーエージェント**が制御（プライバシー制約）
- 作者が `:visited` のスタイルを色のみで設定しても、**本基準の違反とはみなされない**
- ただし通常リンクと背景の **1.4.3 コントラスト**は引き続き必要

> **参照**
> - [Failure F73: Links that are not visually evident without color vision](https://www.w3.org/WAI/WCAG22/Techniques/failures/F73)
> - [Technique G183: 3:1 contrast + hover cue for links](https://www.w3.org/WAI/WCAG22/Techniques/general/G183)
> - [Understanding SC 1.4.1 — Visited links note](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html#intent)

## 画像・グラフ内の色（Situation B）

- グラフ・図表で**色だけ**で系列を区別するのは不適合
- **パターン**、**数値ラベル**、**凡例のテキスト**を併用（G111, G14）

```html
<!-- 悪い例：色だけでグラフ系列を区別 -->
<!-- 青=売上A、赤=売上B のみ。色覚特性で区別不可 -->

<!-- 良い例：色 + パターン + 数値 + 凡例テキスト（G111, G14） -->
<figure>
  <img
    src="sales-chart.svg"
    alt="2024年四半期別売上。Q1: 120万（斜線）、Q2: 150万（点線）、Q3: 180万（実線）"
  >
  <figcaption>
    凡例：<span aria-hidden="true">▨</span> Q1（斜線）、
    <span aria-hidden="true">▧</span> Q2（点線）、
    <span aria-hidden="true">▩</span> Q3（実線）
  </figcaption>
</figure>
```

```html
<!-- 悪い例：alt に色情報だけ（F13） -->
<img src="chart.png" alt="青い棒がA、赤い棒がB">

<!-- 良い例：alt に色に依存しない情報（G14） -->
<img src="chart.png" alt="棒グラフ。A社 120万、B社 95万。A社が上回る。">
```

> **参照**
> - [Technique G111: Using color and pattern](https://www.w3.org/WAI/WCAG22/Techniques/general/G111)
> - [Technique G14: Information conveyed by color also available in text](https://www.w3.org/WAI/WCAG22/Techniques/general/G14)
> - [Failure F13: Text alternative does not include color information](https://www.w3.org/WAI/WCAG22/Techniques/failures/F13)

## 状況別対応（Sufficient Techniques）

### Situation A：テキスト・UI で色を使用

| 手法 | 内容 |
|---|---|
| G14 | 色の違いで伝える情報を**テキスト**でも提供 |
| G205 | 色付きフォームラベルに**テキストの手がかり**を追加 |
| G182 | テキスト色の違いに**追加の視覚的手がかり**（太字、記号、アイコン） |
| G183 | リンク：3:1 色差 + **ホバー時の追加手がかり** |

### Situation B：画像内で色を使用

| 手法 | 内容 |
|---|---|
| G111 | **色 + パターン**で区別 |
| G14 | 色情報を**テキスト**（alt、凡例、キャプション）でも提供 |

## 適用されないケース

- 色で**意味を持たせていない**場合は本基準の対象外
- 例：リンクが通常テキストと**見た目も同じ**（色も下線もなし）→ 1.4.1 ではなく**ユーザビリティ**の問題
- 装飾目的の色使い（ブランドカラー等）で**情報を伝えていない**場合

> **参照**
> - [Understanding SC 1.4.1 — Note on undifferentiated links](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html#intent)

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.3.1 情報および関係性（A） | 色 + プログラム的関連付け（G138） |
| 1.3.3 感覚的な特徴（A） | 指示文が色だけに依存しない |
| 1.4.3 コントラスト（最低限）（AA） | 輝度コントラスト（別要件） |
| 1.4.11 非テキストのコントラスト（AA） | UI コンポーネントの視認性 |

> **参照**
> - [WCAG 2.2 — Success Criterion 1.4.3 Contrast (Minimum)](https://www.w3.org/TR/WCAG22/#contrast-minimum)

## テスト・確認方法

### 手動テスト

- **グレースケール**（モノクロ）表示で、色情報なしに内容が理解できるか
- 必須項目・エラー・リンク・グラフ系列が**色なし**でも区別できるか
- 下線のないリンク：通常テキストとの**色差 3:1** + ホバー手がかりがあるか
- 色覚シミュレーター（Chrome DevTools、Vischeck 等）で確認

```bash
# 開発者ツール：Rendering > Emulate vision deficiencies
# protanopia, deuteranopia, tritanopia, achromatopsia
```

### 自動テスト

- リンクの視認性（下線、色差）の一部を検出
- axe DevTools、WAVE 等（限定的）
- **完全な判定は手動**が必要

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)
> - [Vischeck](http://www.vischeck.com/vischeck/)

## まとめ

- 1.4.1 は**色だけ**を視覚的識別手段にしない達成基準（レベル A）
- 色の使用は**推奨**。`*`・下線・アイコン・テキスト・パターンと**併用**
- フォーム：必須（`*`）、エラー（メッセージ + `aria-invalid`）。F81 に注意
- リンク：**下線**、または 3:1 色差 + ホバー手がかり。F73 に注意
- グラフ：色 + パターン + 数値 + 凡例。F13 に注意
- 1.4.3（コントラスト）と**独立**。両方を満たす必要がある
- モノクロ表示での確認が最も効果的な手動テスト
