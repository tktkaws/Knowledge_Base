# 認知アクセシビリティ（COGA） — 記憶・注意・読解の負荷を下げるW3Cの指針

## COGAとは

- **COGA** = Cognitive and Learning Disabilities Accessibility Task Force（W3C WAI配下）
- 認知・学習の障害がある人がWebを使えるようにするための指針・調査・ギャップ分析を進めるタスクフォース
- 対象の例：ADHD、自閉スペクトラム、ディスレクシア、知的障害、認知症、記憶や実行機能の困難など
- 認知アクセシビリティは、知覚・記憶・言語・注意・問題解決・理解のしかたに関わる障壁を減らす取り組み
- 代表ドキュメントが **Making Content Usable for People with Cognitive and Learning Disabilities**（通称 Content Usable）

> 参照:
> - [W3C WAI — Cognitive Accessibility](https://www.w3.org/WAI/cognitive/)
> - [W3C — Making Content Usable（COGA）](https://www.w3.org/TR/coga-usable/)
> - [COGA Task Force](https://www.w3.org/WAI/about/groups/task-forces/coga/)

## WCAGとの関係

| 項目 | WCAG 2.x | COGA（Content Usable / Supplemental Guidance） |
|---|---|---|
| 位置づけ | 適合の規範（normative） | 補足指針（informative） |
| 必須か | 適合宣言の対象 | WCAG適合には必須ではない |
| 強み | 検証しやすい達成基準 | わかりやすさ・記憶負荷・注意など、規範に入りきらないニーズ |
| 使い方 | 最低ラインのチェック | 設計レビュー・要件・コンテンツ指針に組み込む |

- WCAGにも認知面の手がかりはある（十分な時間、読みやすさ、予測可能、入力支援など）
- それでも「見慣れたUI」「記憶に頼らない手順」「気が散らない画面」までは規範要件だけで足りない
- COGAは「WCAGを満たしたうえで、さらに認知負荷を下げる」ための実務ガイド

> 参照:
> - [W3C WAI — Supplemental Guidance](https://www.w3.org/WAI/WCAG2/supplemental/)
> - [W3C WAI — About Supplemental Guidance](https://www.w3.org/WAI/WCAG2/supplemental/about/)

## 8つの目標（Objectives）

Content Usable / Supplemental Guidance の中心は、次の8目標。

1. **何が何か・どう使うかをわかるようにする**
2. **必要なものを見つけられるようにする**
3. **明確で理解しやすいコンテンツにする**
4. **ミスを防ぎ、直し方をわかるようにする**
5. **注意を維持できるようにする**
6. **手順を記憶に依存させない**
7. **ヘルプとサポートを提供する**
8. **適応・パーソナライズを支える**

- 各目標の下に具体的な Design Patterns（やること・避けること・例）がある
- 仕様書を通読するより、Supplemental Guidance の短いパターン一覧から入ると扱いやすい

> 参照: [W3C WAI — Cognitive Accessibility Guidance（一覧）](https://www.w3.org/WAI/WCAG2/supplemental/)

## 記憶・注意・読解に効くパターン（実装寄り）

### 記憶の負荷を下げる

- 別画面で見た内容を暗記させない（確認画面に入力内容を再表示）
- ログインを複雑な暗記だけに依存させない（パスキー、マジックリンク、SSOなどの代替）
- 計算や長いコードの暗記を求めない
- タイムアウトで入力が消えない／復元できる

```html
<!-- 悪い例：前画面の内容を覚えていないと進めない -->
<label for="confirm">確認コード</label>
<input id="confirm" name="confirm" />

<!-- 良い例：文脈をその場に残す -->
<p>メール（u***@example.com）に送った6桁のコードを入力してください。</p>
<p>有効期限は10分です。届かない場合は再送信できます。</p>
<label for="confirm">確認コード（6桁）</label>
<input id="confirm" name="confirm" inputmode="numeric" autocomplete="one-time-code" />
<button type="button">コードを再送信</button>
```

> 参照: [W3C WAI — Ensure Processes Do Not Rely on Memory](https://www.w3.org/WAI/WCAG2/supplemental/)

### 注意の負荷を下げる

- 割り込み（突然のモーダル、自動再生、同時多発の通知）を減らす／止められるようにする
- クリティカルパスを短くする
- 1画面に情報を詰め込みすぎない
- ユーザーが動きを制御できる（`prefers-reduced-motion` も尊重）

```html
<!-- 悪い例：開いた瞬間に割り込み -->
<dialog open>
  <p>今だけ！特別キャンペーン</p>
</dialog>

<!-- 良い例：本線のタスクを先に完了できる -->
<main>
  <h1>お支払い</h1>
  <p>配送先と支払い方法を入力してください。</p>
  <!-- キャンペーンは本線の外、または明示操作後に表示 -->
</main>
```

```css
@media (prefers-reduced-motion: reduce) {
  .banner {
    animation: none;
  }
}
```

> 参照: [W3C WAI — Help Users Focus](https://www.w3.org/WAI/WCAG2/supplemental/)

### 読解の負荷を下げる

- 短い文、能動態、字義どおりの表現
- 二重否定・入れ子の条件を避ける
- 手順は1つずつ分ける
- 長い文書には要約を付ける
- 余白を確保し、前景が背景に埋もれないようにする

```html
<!-- 悪い例 -->
<p>カートへの追加が完了していない場合を除き、かつ在庫がある場合に限り決済へ進めます。</p>

<!-- 良い例 -->
<p>決済に進む条件は次のとおりです。</p>
<ol>
  <li>カートに商品が入っている</li>
  <li>在庫がある</li>
</ol>
```

> 参照: [W3C WAI — Use Clear and Understandable Content](https://www.w3.org/WAI/WCAG2/supplemental/)

## 見つけやすさ・ミス防止・ヘルプ

### 見つけやすさ

- ページの目的を冒頭で示す
- 見出し・ランドマーク・一貫したナビ
- 重要タスクを目立たせ、検索を用意する

```html
<!-- 良い例：目的と主要アクションがすぐわかる -->
<main>
  <h1>配送先の変更</h1>
  <p>次回以降の配送に使う住所を変更できます。</p>
  <a href="#address-form">住所フォームへ</a>
</main>
```

### ミス防止と訂正

- ラベルを常に可視にする（プレースホルダーだけにしない）
- 入力形式をその場で示し、複数フォーマットを受け入れる
- 戻れる・取り消せる・料金はタスク開始時に明示する

```html
<!-- 悪い例 -->
<input type="tel" placeholder="電話番号" />

<!-- 良い例 -->
<label for="tel">電話番号（ハイフンありでもなしでも可）</label>
<input id="tel" type="tel" autocomplete="tel" aria-describedby="tel-hint" />
<p id="tel-hint">例: 09012345678 / 090-1234-5678</p>
```

### ヘルプとパーソナライズ

- 人的サポートへの導線を見つけやすくする
- 複雑な説明には要約・図・段階的ガイドなどの代替を用意する
- ブラウザ拡張や読み上げ支援を阻まない
- 簡易表示・動きの抑制など、ユーザー側の調整を壊さない

> 参照: [W3C WAI — Cognitive Accessibility Guidance](https://www.w3.org/WAI/WCAG2/supplemental/)

## フロントエンドの実務への落とし方

| 工程 | COGAの使い方 |
|---|---|
| 要件定義 | 8目標のうち、該当プロダクトで優先する目標を選ぶ |
| デザインシステム | 見慣れたコントロール、一貫レイアウト、可視ラベルを標準にする |
| 実装 | 記憶非依存の確認画面、止められるアニメーション、短いエラー文 |
| コンテンツ | 文言ガイドライン（短文・能動態・二重否定禁止）として共有 |
| QA | 「覚えなくても進めるか」「割り込みで止まらないか」「一文で意味が取れるか」 |

- axe / Lighthouseだけでは認知負荷は測りきれない
- チェックリスト＋実際のタスク完了テスト（できれば当事者参加）が必要

> 参照: [W3C — Including Users in Design and Testing（COGA）](https://www.w3.org/TR/coga-usable/)

## ニューロダイバーシティ記事との違い

| 記事 | 焦点 |
|---|---|
| ニューロダイバーシティとWebアクセシビリティ | 認知の多様性を設計前提にする考え方 |
| 本記事（COGA） | W3Cの補足指針・8目標・Design Patternsの使い方 |

- 考え方の導入はニューロダイバーシティ記事、規範外の具体指針は本記事、という読み分け

## 関連して読むとよいテーマ

- ニューロダイバーシティとWebアクセシビリティ
- アクセシブルなフォームの設計 — エラー表示とバリデーション
- prefers-reduced-motion — モーション設定への対応
- セマンティックHTML / ランドマーク — 構造の把握しやすさ
- WCAG 2.2の4原則 — 理解可能（Understandable）との接続

## まとめ

- COGAは、認知・学習の障害がある人向けのW3Cの取り組みと補足指針
- Content Usable / Supplemental Guidance は WCAG適合の必須条件ではないが、実務では重要
- 中心は8目標 — 特に記憶・注意・読解の負荷を下げるパターンがフロントエンドと相性が良い
- 実装では、文脈の再表示、割り込み抑制、短い文言、可視ラベル、戻れるフォームが具体的な打ち手
- 自動テストの先に、タスク完了と当事者参加の確認を置く
