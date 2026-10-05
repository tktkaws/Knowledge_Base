# ニューロダイバーシティとWebアクセシビリティ — 認知特性の違いを設計の前提にする

## ニューロダイバーシティとは

- 脳の働き方・情報処理の仕方に多様性があって当たり前だ、という考え方
- 「正常な脳」と「障害のある脳」の二択ではなく、認知のバリエーションとして捉える立場
- 自閉スペクトラム（ASD）、ADHD、ディスレクシア、ディスカリキュリア、ディスプラキシアなどを含む広い概念
- 医療モデル（個人の欠陥）ではなく、環境とのミスマッチとして障壁を見る視点と相性が良い
- Webでは「平均的なユーザー」を前提にしたUIが、特定の認知特性を持つ人を排除しがち

> 参照:
> - [W3C WAI — Cognitive Accessibility](https://www.w3.org/WAI/cognitive/)
> - [W3C WAI — Diverse Abilities and Barriers](https://www.w3.org/WAI/people-use-web/abilities-barriers/)

## なぜフロントエンドで知る必要があるか

- コントラストやキーボード操作だけでは足りない障壁がある
- 読みにくい文言、覚えにくい手順、予測できない動きは「使えない」レベルの排除になり得る
- WCAG 2.x は認知面を一部カバーするが、すべての認知ニーズを規範要件に入れきれていない
- 補足指針として COGA（Making Content Usable）が存在する — 適合必須ではないが実務では重要
- 認知負荷を下げる設計は、疲れている人・ストレス下の人・非母語話者にも効く

> 参照:
> - [W3C — Making Content Usable for People with Cognitive and Learning Disabilities](https://www.w3.org/TR/coga-usable/)
> - [W3C WAI — Cognitive Accessibility Guidance（Supplemental）](https://www.w3.org/WAI/WCAG2/supplemental/)

## 認知特性ごとの障壁の例

| 特性の例 | Web上で起きやすい障壁 |
|---|---|
| ADHD | 自動再生・点滅・広告・通知による注意の散逸 |
| ASD | 曖昧なラベル、暗黙ルール、感覚的に強い刺激 |
| ディスレクシア | 長い行・狭い行間・装飾フォント・複雑な言い回し |
| 記憶・実行機能の困難 | 多段階ログイン、タイムアウト、前画面の情報を覚える前提 |
| 言語処理の困難 | 二重否定、専門用語だらけのエラー、比喩だらけのコピー |

- 診断名でユーザーを分類してデザインする必要はない
- 「注意・記憶・読解・時間・感覚」にかかる負荷を下げる、という機能要件として扱う

> 参照:
> - [W3C WAI — Cognitive and Learning（How People Use the Web）](https://www.w3.org/WAI/people-use-web/abilities-barriers/#cognitive)
> - [W3C — COGA User Stories](https://www.w3.org/TR/coga-usable/user_stories.html)

## WCAGでカバーされている認知まわり

| ガイドライン / 達成基準 | 認知面での意味合い |
|---|---|
| 1.3 適応可能 | レイアウトを簡略化しても情報と構造が残る |
| 1.4 判別可能 | 前景と背景を分け、読む・聞く負荷を下げる |
| 2.2 十分な時間 | 読む・操作する時間を奪わない |
| 2.4 ナビゲート可能 | 現在地・見出し・スキップ手段で迷いにくくする |
| 3.1 読みやすさ | 言語の明示、難しい語の説明など |
| 3.2 予測可能 | フォーカスや入力で不意に画面が変わらない |
| 3.3 入力支援 | ミスを防ぎ、直しやすくする |

- これらを満たしても「わかりやすい」「覚えなくてよい」までは保証されない
- だから COGA の補足パターンを設計レビューに入れる価値がある

> 参照: [W3C WAI — Cognitive Accessibility at W3C](https://www.w3.org/WAI/cognitive/)

## 設計の前提にするための8つの方針

COGA の目標を、実装寄りの言葉に言い換えたもの。

### 1. 見慣れたパターンを使う

- 独自の操作体系より、リンク・ボタン・フォームの慣習を優先
- アイコンだけでは意味を伝えない — テキストラベルを併記
- ページごとの見た目・配置・挙動を揃える

```html
<!-- 悪い例：アイコンだけで機能がわからない -->
<button aria-label="設定"><svg><!-- gear --></svg></button>

<!-- 良い例：見えるラベルがある -->
<button type="button">
  <svg aria-hidden="true"><!-- gear --></svg>
  設定
</button>
```

### 2. 何があるか・どこにいるかを一目でわかるようにする

- ページの目的を冒頭で示す
- 見出し階層・ランドマーク・パンくずで構造を示す
- 検索と主要導線を見つけやすい位置に置く

### 3. 文言を短く・明確にする

- 短い文、能動態、一義的な語を選ぶ
- 二重否定や入れ子の条件文を避ける
- 専門用語には言い換えか説明を添える

```html
<!-- 悪い例 -->
<p>カートへの追加が完了していない場合を除き、決済へ進めません。</p>

<!-- 良い例 -->
<p>先にカートへ商品を追加してください。そのあと決済に進めます。</p>
```

### 4. 記憶に頼らせない

- 別画面で見た情報を暗記させない（確認画面に入力内容を再表示）
- CAPTCHA や複雑なパスワードルールだけに依存しない代替を用意する
- セッション切れ前に十分な警告と再入力の負担軽減を行う

### 5. ミスを起きにくくし、直しやすくする

- 必須項目を減らし、入力形式のヒントをその場に出す
- エラーはどこが・なぜ・どう直すかをセットで示す
- 破壊的操作には確認と取り消しの余地を残す

```html
<!-- 悪い例：色だけ・原因不明 -->
<input type="email" class="error" />
<p style="color: red;">エラー</p>

<!-- 良い例：関連付けと具体的な直し方 -->
<label for="email">メールアドレス</label>
<input
  id="email"
  type="email"
  aria-invalid="true"
  aria-describedby="email-error"
/>
<p id="email-error">形式が正しくありません。例: name@example.com</p>
```

### 6. 気が散る要素を抑える

- 自動再生・カルーセルの自動切替・突然のモーダルを避けるか止められるようにする
- `prefers-reduced-motion` を尊重する
- 広告・通知・装飾アニメーションの同時多発を避ける

```css
@media (prefers-reduced-motion: reduce) {
  .hero-carousel {
    animation: none;
  }
}
```

### 7. 十分な時間と中断からの復帰を確保する

- 制限時間があるなら延長・解除・調整の手段を用意する
- 途中保存、下書き、ステップの進捗表示を用意する
- 「いま何をしているか」が再開時にもわかるUIにする

### 8. 助けと個人化の余地を残す

- ヘルプへの導線を見つけやすくする
- ブラウザ拡張や読み上げ支援を阻むスクリプトを書かない
- 文字サイズ・コントラスト・簡易表示など、ユーザー側の調整を壊さない

> 参照:
> - [W3C — COGA Design Guide](https://www.w3.org/TR/coga-usable/design_guide.html)
> - [W3C WAI — Supplemental Guidance](https://www.w3.org/WAI/WCAG2/supplemental/)

## フロントエンドでよくある「認知バリア」パターン

### 悪い例：手順が記憶依存

```html
<!-- 前の画面で表示した確認コードを、説明なしに再入力させる -->
<label for="code">コード</label>
<input id="code" name="code" />
```

### 良い例：文脈をその場に残す

```html
<p>メール（u***@example.com）に送った6桁のコードを入力してください。</p>
<p>届かない場合は「再送信」からやり直せます。コードの有効期限は10分です。</p>
<label for="code">確認コード（6桁）</label>
<input id="code" name="code" inputmode="numeric" autocomplete="one-time-code" />
<button type="button">コードを再送信</button>
```

### 悪い例：予測できない画面遷移

```js
// フォーカスや選択のたびに勝手に次画面へ
select.addEventListener("change", () => {
  location.href = `/category/${select.value}`;
});
```

### 良い例：ユーザーの明示操作で進む

```js
form.addEventListener("submit", (event) => {
  event.preventDefault();
  const value = select.value;
  location.href = `/category/${value}`;
});
```

- フォーカス移動や入力だけでページが変わる挙動は WCAG 3.2.1 / 3.2.2 にも抵触しやすい
- 「押した・送信した」など、意図が明確な操作で結果を出す

> 参照:
> - [WCAG 2.2 — 3.2.1 On Focus](https://www.w3.org/WAI/WCAG22/Understanding/on-focus.html)
> - [WCAG 2.2 — 3.2.2 On Input](https://www.w3.org/WAI/WCAG22/Understanding/on-input.html)

## チェック時の観点（短いリスト）

- 初見でも「このページで何ができるか」がわかるか
- 主要な操作は見慣れたコントロールか
- 読む文量・専門用語は必要最小限か
- 別画面の情報を暗記しないと進めない箇所はないか
- エラーは具体的で、その場で直せるか
- 動き・音・通知は止められるか、最初から控えめか
- 制限時間があるなら延長・解除できるか
- 実際のユーザー（認知・学習の困難がある人を含む）のフィードバックを取る計画があるか

> 参照: [W3C — Including Users in Design and Testing Activities（COGA）](https://www.w3.org/TR/coga-usable/)

## 関連して読むとよいテーマ

- 認知アクセシビリティ（COGA）そのものの指針詳細
- `prefers-reduced-motion` — 動きによる負荷の軽減
- セマンティックHTML / ランドマーク — 構造の把握しやすさ
- アクセシブルなフォーム — エラーと入力支援
- 障害の社会モデル — 「個人の不足」ではなく環境側の障壁として捉える視点

## まとめ

- ニューロダイバーシティは、認知の違いを例外ではなく設計前提に置く考え方
- 視覚・聴覚・運動の対応だけでは届かない「わかりにくさ」「覚えにくさ」「気が散る」を扱う
- WCAG の読みやすさ・予測可能性・入力支援を土台に、COGA の補足パターンで一段深くする
- フロントエンドでは、慣習的なUI、短い文言、記憶非依存、静かな画面、直しやすいエラーが具体的な打ち手
- 「平均ユーザー向けに最適化し、残りは我慢してもらう」のではなく、最初から幅のある使い方を想定する
