# 1.3.6 目的の特定（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 1.3.6「目的の特定」（Identify Purpose）
- レベル **AAA**（最高の適合レベル）
- WCAG **2.1** で追加された達成基準
- 原則1「知覚可能（Perceivable）」> ガイドライン 1.3「適応可能」に属する
- マークアップ言語で実装されたコンテンツにおいて、**UI コンポーネント・アイコン・領域（region）** の**目的**をプログラム的に特定可能にする
- パーソナライゼーション・シンボル置換・UI 簡素化など、**認知障害**向けの適応を支援

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.6 Identify Purpose](https://www.w3.org/TR/WCAG22/#identify-purpose)
> - [Understanding SC 1.3.6 Identify Purpose](https://www.w3.org/WAI/WCAG22/Understanding/identify-purpose.html)

## 達成基準の原文（要約）

- マークアップ言語で実装されたコンテンツにおいて、**ユーザーインターフェースコンポーネント**、**アイコン**、**領域（region）** の目的を、**プログラム的に特定**できること

> **参照**
> - [WCAG 2.2 — Success Criterion 1.3.6](https://www.w3.org/TR/WCAG22/#identify-purpose)

## 1.3.5 との違い

| 達成基準 | レベル | 対象 | 主な手段 |
|---|---|---|---|
| 1.3.5 入力目的の特定 | AA | **ユーザー情報**を収集する**入力欄** | `autocomplete` 属性 |
| 1.3.6 目的の特定 | AAA | **UI 全体**（コンポーネント・アイコン・領域） | ランドマーク、メタデータ、WAI-Adapt 等 |

- 1.3.5 はフォーム入力に**限定**
- 1.3.6 は**ページ全体**の操作要素・ナビゲーション・領域の目的を対象
- 1.3.5 を満たしても 1.3.6 は**自動的には満たさない**

> **参照**
> - [wcag/1-3-5-identify-input-purpose.md — 1.3.5 入力目的の特定](./1-3-5-identify-input-purpose.md)

## 4.1.2 との違い

| 達成基準 | 焦点 | 例 |
|---|---|---|
| 4.1.2 名前・役割・値 | **何であるか**（role） | ボタン、リンク、画像 |
| 1.3.6 目的の特定 | **何を表すか**（purpose） | ホームへのリンク、検索、メインコンテンツ領域 |

- 4.1.2：`role="button"` → これはボタンである
- 1.3.6：目的のメタデータ → これは**ホームへ戻る**ボタンである
- 支援技術が**ユーザーに馴染みのあるシンボル**に置き換えるための情報

> **参照**
> - [WCAG 2.2 — Success Criterion 4.1.2 Name, Role, Value](https://www.w3.org/TR/WCAG22/#name-role-value)

## なぜ必要か

- 語彙が限られたユーザー、認知・学習障害のあるユーザーが、**馴染みのある用語・シンボル**で UI を理解できるようにするため
- 作者が目的を明示すれば、ユーザーエージェントが**パーソナライズ**（アイコン置換、用語変更、領域の非表示）を提供できる
- 非言語コミュニケーションでシンボルを使うユーザー向けに、**シンボルの相互運用性**を支援
- 記憶・注意・実行機能・言語処理に困難があるユーザーへの**認知負荷軽減**

> **参照**
> - [Understanding SC 1.3.6 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/identify-purpose.html#benefits)
> - [Making Content Usable for People with Cognitive and Learning Disabilities](https://www.w3.org/WAI/coga/)

## 対象となる要素

### UI コンポーネント

- ボタン、リンク、フォームコントロール等
- ユーザーが**単一の機能**として知覚する操作要素

### アイコン

- ナビゲーション、操作、状態を示すグラフィック
- テキストラベルがない、またはラベルだけでは目的が伝わらないアイコン

### 領域（region）

- ページ内の**知覚可能な区画**
- ヘッダー、ナビゲーション、メインコンテンツ、サイドバー、フッター等
- HTML ではランドマーク role が region に該当

> **参照**
> - [Understanding SC 1.3.6 — Intent](https://www.w3.org/WAI/WCAG22/Understanding/identify-purpose.html#intent)

## 実装方法：ARIA ランドマーク（Technique ARIA11）

- ページの**領域**に目的を示すランドマーク role を付与
- スクリーンリーダー利用者が領域間を**ジャンプ**できる
- ユーザーエージェントが `main` 以外の領域を**非表示**にする等の適応が可能

| ランドマーク | 目的 |
|---|---|
| `banner` | サイト全体のヘッダー（ロゴ、サイト名） |
| `navigation` | ナビゲーションリンク群 |
| `main` | ページ固有のメインコンテンツ |
| `search` | 検索機能 |
| `complementary` | メインを補完するサイドバー等 |
| `contentinfo` | フッター（著作権、プライバシーポリシー） |
| `form` | フォーム（accessible name が必要） |
| `region` | その他の重要な区画（accessible name が必要） |

```html
<!-- 悪い例：div のみで領域の目的が不明 -->
<div class="header">...</div>
<div class="sidebar">...</div>
<div class="content">...</div>

<!-- 良い例：HTML5 セマンティック要素 + ランドマーク -->
<header>...</header>
<nav aria-label="メインナビゲーション">...</nav>
<main>...</main>
<aside aria-label="関連記事">...</aside>
<footer>...</footer>
```

```html
<!-- 良い例：同一 role が複数ある場合は aria-label で区別 -->
<nav aria-label="メインメニュー">...</nav>
<nav aria-label="フッターメニュー">...</nav>

<!-- 良い例：検索フォーム -->
<form role="search">
  <label for="q">検索</label>
  <input type="search" id="q" name="q">
  <button type="submit">検索</button>
</form>
```

- ページ上の**すべてのコンテンツ**をランドマーク内に含めるのがベストプラクティス
- ランドマーク外のコンテンツは支援技術利用者が**見失う**可能性がある

> **参照**
> - [Technique ARIA11: Using ARIA landmarks to identify regions of a page](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA11)
> - [W3C WAI — Page Regions](https://www.w3.org/WAI/tutorials/page-structure/regions/)

## UI コンポーネントの目的

- **accessible name**（ラベル、`aria-label`）で操作の目的を伝える
- 4.1.2 の name と重なるが、1.3.6 では**パーソナライズ可能な意味**の提供が焦点
- 将来は WAI-Adapt 等のメタデータで、より標準化された目的トークンを付与

```html
<!-- 悪い例：アイコンのみ、目的不明 -->
<a href="/">
  <img src="home.svg" alt="">
</a>

<!-- 良い例：目的が名前として特定可能 -->
<a href="/" aria-label="ホーム">
  <img src="home.svg" alt="" aria-hidden="true">
  <span>ホーム</span>
</a>

<!-- 良い例：操作目的が明確なボタン -->
<button type="button" aria-label="カートに追加">
  <img src="cart.svg" alt="" aria-hidden="true">
  カートに追加
</button>
```

> **参照**
> - [wcag/1-3-3-sensory-characteristics.md — アイコンボタン](./1-3-3-sensory-characteristics.md)

## WAI-Adapt とパーソナライゼーション

- **WAI-Adapt**（旧 Personalization Semantics）は、UI 要素の目的を**標準化されたメタデータ**でマークアップする仕組
- ユーザーエージェントが、ユーザー設定に応じて**シンボル・用語・レイアウト**を置き換え
- 1.3.6 の**理想的な実装**だが、エコシステムは発展途上
- 現時点では **ARIA ランドマーク + 明確な accessible name** が実用的な sufficient technique

```html
<!-- 将来の WAI-Adapt による目的の明示（概念例） -->
<button data-purpose="save">
  保存
</button>
<!-- ユーザーエージェントが馴染みのシンボルセットに置換 -->
```

- シンボルユーザー向け：**同一ノードにマッピング**されれば、異なるアプリ間でシンボルを共有可能
- 現状は各社のシンボルセットが**相互運用不可**な場合が多い

> **参照**
> - [WAI-Adapt Overview](https://www.w3.org/WAI/standards-guidelines/wai-adapt/)
> - [WAI-Adapt: Symbols Module](https://www.w3.org/WAI/WAI-Adapt/)

## COGA 向けの Advisory Techniques

- **必須項目**のプログラム的特定：`required` / `aria-required` + `aria-invalid`
- **重要機能**のマークアップ：`coga-simplification` 等（仕様発展中）
- ユーザーが**必要なバージョン**のコンテンツを見つけられるようにする

```html
<!-- Advisory：必須・エラーをプログラム的に特定 -->
<label for="email">メールアドレス <span aria-hidden="true">*</span></label>
<input
  type="email"
  id="email"
  required
  aria-required="true"
  aria-invalid="true"
  aria-describedby="email-error"
>
<span id="email-error">有効なメールアドレスを入力してください</span>
```

> **参照**
> - [Understanding SC 1.3.6 — Advisory Techniques](https://www.w3.org/WAI/WCAG22/Understanding/identify-purpose.html#techniques)

## 1.3.5 との組み合わせ

- フォーム領域では **1.3.5（autocomplete）** + **1.3.6（ランドマーク・目的）** の両方を検討
- `role="form"` + `aria-label` でフォーム領域の目的を特定
- 各入力欄に `autocomplete` で個別の入力目的を特定

```html
<!-- 1.3.5 + 1.3.6 を満たすフォーム例 -->
<form role="form" aria-label="会員登録">
  <fieldset>
    <legend>基本情報</legend>
    <label for="email">メールアドレス</label>
    <input type="email" id="email" autocomplete="email">
    <label for="name">氏名</label>
    <input type="text" id="name" autocomplete="name">
  </fieldset>
  <button type="submit">登録</button>
</form>
```

## 他の達成基準との関係

| 関連基準 | 関係 |
|---|---|
| 1.3.1 情報および関係性（A） | ランドマークは 1.3.1 でも sufficient（ARIA11） |
| 1.3.5 入力目的の特定（AA） | フォーム入力の autocomplete |
| 2.4.1 ブロックスキップ（A） | ランドマークによる領域ジャンプ |
| 4.1.2 名前・役割・値（A） | role / name（「何か」vs「何のためか」） |

- ARIA11 は **1.3.1・1.3.6・2.4.1** の sufficient technique として共通
- AAA 適合では、ランドマークに加え**コンポーネント・アイコン**の目的も網羅的に検討

> **参照**
> - [wcag/1-3-1-info-and-relationships.md — 1.3.1](./1-3-1-info-and-relationships.md)
> - [WCAG 2.2 — Success Criterion 2.4.1 Bypass Blocks](https://www.w3.org/TR/WCAG22/#bypass-blocks)

## AAA 達成の現実的な考え方

- 1.3.6 は**エコシステム全体**（ブラウザ、支援技術、パーソナライズ設定）の成熟を前提とする部分がある
- **現時点で確実に実装できる**内容
  - セマンティック HTML + ARIA ランドマーク（ARIA11）
  - すべての UI コンポーネントへの**明確な accessible name**
  - フォーム入力への **autocomplete**（1.3.5 も兼ねる）
- **将来の拡張**
  - WAI-Adapt メタデータ
  - microdata による UI コンポーネントのマークアップ
  - ユーザー設定に基づくシンボル・用語の置換

> **参照**
> - [Understanding SC 1.3.6 — Examples](https://www.w3.org/WAI/WCAG22/Understanding/identify-purpose.html#examples)

## テスト・確認方法

### 自動テスト

- ランドマーク role の有無・重複時の accessible name
- ボタン・リンクの accessible name の有無
- axe DevTools の landmark ルール等

### 手動テスト

- スクリーンリーダーの**ランドマーク一覧**で領域の目的が識別できるか
- 同一 role（`navigation` 等）が複数ある場合、**区別可能な名前**があるか
- アイコンボタン・リンクに**目的が伝わる名前**があるか
- ページ上のコンテンツが**ランドマーク外に漏れていない**か
- フォーム入力に **autocomplete** があるか（1.3.5 も確認）

```bash
# ランドマーク role の一覧（開発者ツール Console）
[...document.querySelectorAll('[role="banner"],[role="navigation"],[role="main"],[role="complementary"],[role="contentinfo"],[role="search"],header,nav,main,aside,footer')].map(el => ({
  tag: el.tagName,
  role: el.getAttribute('role') || '(implicit)',
  label: el.getAttribute('aria-label') || el.getAttribute('aria-labelledby') || ''
}));
```

> **参照**
> - [a11y/a11y-automated-testing.md — 自動テストツール](../a11y/a11y-automated-testing.md)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 1.3.6 は **UI コンポーネント・アイコン・領域**の**目的**をプログラム的に特定する達成基準（レベル AAA、WCAG 2.1 追加）
- 1.3.5（入力欄）より**広い範囲**。パーソナライゼーション・認知支援が目的
- 4.1.2 は「何であるか」、1.3.6 は「**何のためか**」
- **ARIA ランドマーク**（ARIA11）が現時点の主要な sufficient technique
- すべての操作要素に**明確な accessible name**、領域は**ランドマークで網羅**
- WAI-Adapt 等のメタデータは将来の拡張。現状はランドマーク + name + autocomplete で基盤を整える
