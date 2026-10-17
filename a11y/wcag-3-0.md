# WCAG 3.0について — 次期ガイドラインの方針とWCAG 2との違い

## いまの位置づけ

- **WCAG 3** の正式名は W3C Accessibility Guidelines（WCAG）3.0
- WCAG 2 の正式名は Web Content Accessibility Guidelines。略称は同じでも、3は「コンテンツ」に限らない名前
- 2026年10月時点の公開版は **Working Draft（2026年9月10日）**。勧告（完成した標準）ではない
- 草案は数年単位で変わり、未完成のまま引用して適合宣言の根拠にはしない
- WCAG 3は WCAG 2 の後継として作られているが、**WCAG 2を廃止しない**
- 各国の法令がすぐに WCAG 3 へ切り替わる前提ではない。WCAG 2 は完成後も数年は残る、とWAIは説明している
- 今の実務で足場にするのは **WCAG 2.2**。WAIも「WCAG 3への準備は、今 WCAG 2.2 の達成基準を満たすこと」としている
- 以前のプロジェクト名は Silver（おおよそ2017〜2023年）。最初の公開草案は2021年1月21日

> 参照:
> - [W3C WAI — WCAG 3 Introduction](https://www.w3.org/WAI/standards-guidelines/wcag/wcag3-intro/)
> - [W3C — WCAG 3.0 Working Draft（2026-09-10）](https://www.w3.org/TR/wcag-3.0/)
> - [W3C — WCAG 3.0 publication history](https://www.w3.org/standards/history/wcag-3.0/)

## WCAG 2との違い

| 項目 | WCAG 2.x | WCAG 3（2026年9月草案） |
|---|---|---|
| 状態 | 2.2まで勧告。適合宣言に使える | 作業草案。大幅に変わる |
| 単位 | 達成基準（Success Criteria）と A / AA / AAA | ガイドライン（結果の文）と、その下の要件 |
| 適合 | 選んだレベルで、適用される達成基準をすべて満たす | 単一の適合＝**Core Requirements をすべて満たす**、という案 |
| 範囲 | Webコンテンツが中心 | Webコンテンツに加え、アプリ、出版、XR、関連ツール（ブラウザ、CMS、オーサリング、検査）も視野 |
| 書き方 | 技術非依存の達成基準＋Techniques / Understanding | 結果文、要件、アサーション、推奨プラクティス。技術別の Method と How to は参考 |
| 粒度 | 1つの達成基準に複数の判断が含まれることが多い | 要件をより細かく分割する方向 |

- WCAG 2.2 のレベルAとAAを満たすと、WCAG 3の最低適合に近い、と草案は述べる
- 近いだけで同一ではない。追加のテストと、別の適合の仕組みがあるため、移行には追加作業が要る
- WCAG 3が勧告になるのは、少なくとも WCAG 2.2 がカバーする範囲を含んでから、という方針
- 非Web技術そのもの、UIを持たないプラットフォームやOSへの規範要件は、Explainer上は対象外

> 参照:
> - [W3C — WCAG 3.0 Introduction（plain language summary）](https://www.w3.org/TR/wcag-3.0/)
> - [Explainer for WCAG 3.0](https://w3c.github.io/wcag3/explainer/)
> - [W3C Blog — Crafting WCAG 3（2026-09-25）](https://www.w3.org/blog/2026/crafting-wcag-3-for-more-accessible-user-experiences/)

## 文書の層

- **ガイドライン**: 平易な「利用者にとっての結果」。規範。例: 画像には同等の代替がある
- **Core Requirements**: 適合に必須とする規範の要件。WCAG 2.2 のAおよびAAに近いが同一ではない、という位置づけ
- **Supplemental Requirements**: 適合の必須にはしない規範。政策や報告で追加できる
- **Assertions（アサーション）**: 第三者がコードだけ見て検証しにくい手続きを、組織が文書で声明する仕組み。研修、支援技術での確認、利用者テストなど。Coreの代わりにはならない
- **Recommended practices**: 参考情報。客観的に測れない、状況判断が要るものはここ。適合の合否には影響しない
- **Methods / How to**: WCAG 2 の Techniques / Understanding に近い参考資料。技術別のやり方やテストを置く

> 参照: [W3C WAI — WCAG 3 Introduction（Structure）](https://www.w3.org/WAI/standards-guidelines/wcag/wcag3-intro/)

## 成熟度のラベル

草案の規範セクションには、進み具合のラベルが付く。

| ラベル | 意味 |
|---|---|
| Placeholder | 仮置き。中身は置き換わる |
| Exploratory | 方向を探している段階 |
| Developing | 必要なものは大筋で合意。詳細は未確定 |
| Refining | 広く試し、実装可能性への意見を求める段階 |
| Mature | 勧告に載せられる、と作業部会が考える段階 |

- 2026年9月10日の Working Draft に載るのは、Developing 以上の要件とアサーション
- Exploratory の一覧は Editor's Draft 側
- ガイドライン一覧自体は、まだ題目のリストに近く、最終版で統合・削除・追加される

> 参照: [W3C — WCAG 3.0（Section status levels）](https://www.w3.org/TR/wcag-3.0/)

## 適合と報告ティア（2026年9月草案の案）

- 適合は二値。スコープ内のコンテンツが **Core Requirements をすべて満たす**こと
- A / AA / AAA の三段階適合は、この草案では採らない
- その代わり、適合の手前と先を示す **報告ティア** を検討している
- Core要件にはタグを付ける案がある

| タグ | 満たさないときの影響（草案の説明） |
|---|---|
| Physical harm | 障害のある人に、すぐ身体的危害が起き得る |
| Risk | 金銭・医療・法的・プライバシー・セキュリティの不当なリスクが増える |
| Barrier | 先へ進めなくなる |
| Friction | 妨げになる。重なると先へ進めなくなる |

WAIの紹介ページが示す報告ティアの案:

| ティア | 中身 |
|---|---|
| Tier 1 | Physical harm と Risk |
| Tier 2 | それに Barrier を加える |
| Tier 3（適合） | すべての Core Requirements（Friction まで） |
| Tier 4〜6 | 適合に加え、補足要件とアサーションを段階的に増やす。件数は未定 |

- Explainer には、適合の上を Bronze / Silver / Gold と呼ぶ説明や、タグに重みを付けた別案のスコアもある
- 別案のスコアは Working Draft には入っていない、と Explainer は注記している
- 政策（法令・調達）がどのティアやタグを義務にするかは、W3Cではなく規制側が決める、という切り分け
- 件数も名称も未確定。この表は2026年9月時点の案

> 参照:
> - [W3C WAI — WCAG 3 Introduction（Conformance approach）](https://www.w3.org/WAI/standards-guidelines/wcag/wcag3-intro/)
> - [Explainer — Conformance approach](https://w3c.github.io/wcag3/explainer/)

## テストの単位

- **定量テスト**: 属性の有無や数値など、テスター間で結果がぶれにくいもの
- **定性テスト**: 代替テキストが目的と同等か、など基準を読んだうえでの判断。説明を具体化してぶれを減らす
- スコープの語も WCAG 2 の「ページ」から広がる案
  - **Item**: ドロップダウン、リンク、画像、エラーメッセージなど最小単位
  - **View**: 大きな遷移なしで見られるまとまり。モーダルやアプリの1画面を含む
  - **Task flow**: ログイン、購入、申告など、一連の操作
  - **Product**: それらの全体
- 要件は「Applies when / Except when」で適用条件を書く方向

> 参照: [Explainer — Types of tests / Scope](https://w3c.github.io/wcag3/explainer/)

## コードで見る粒度の違い

WCAG 2.2 の 1.1.1（非テキストコンテンツ）は、装飾の除外と、同等の代替テキストを一つの達成基準で扱う。2026年9月草案の「Image alternatives」は、検出・装飾の隠蔽・代替の存在・代替の同等性を分けて書いている。文言は Developing であり、最終形ではない。

```html
<!-- 悪い例：ファイル名は代替にならず、装飾アイコンも読み上げられる -->
<a href="/cart">
  <img src="/icon-bag.svg" alt="icon-bag.svg">
  カート
</a>
<img src="/divider.svg" alt="区切り線の画像">
```

```html
<!-- 良い例：意味のある画像は検出でき、目的と同等のテキストがある。装飾はプログラムから隠す -->
<a href="/cart">
  <img src="/icon-bag.svg" alt="">
  カート
</a>
<img src="/divider.svg" alt="">
```

- 空の `alt` は「代替がある」こととは別問題。リンク名は隣のテキスト「カート」が担う
- 自動検査は「空でない alt があるか」までは見やすい。「目的と同等か」は人が見る定性の判断
- この分け方は、草案が達成基準を細かくしている例。番号や必須／補足の割り当てはまだ動く

> 参照:
> - [W3C — WCAG 3.0（Image alternatives）](https://www.w3.org/TR/wcag-3.0/)
> - [W3C WAI — Understanding Non-text Content（WCAG 2.2）](https://www.w3.org/WAI/WCAG22/Understanding/non-text-content.html)

## フロントエンドが今やること

| 状況 | 扱い |
|---|---|
| 案件の適合目標 | WCAG 2.2 のレベルAAを基本にする。WCAG 3適合とは書かない |
| 提案書・アクセシビリティ方針 | 「WCAG 3対応済み」は草案段階では不正確 |
| 実装 | セマンティクス、名前、キーボード、コントラストなど WCAG 2.2 の達成基準をテンプレートで満たす |
| 認知・状況依存の配慮 | 草案が広げようとしている領域。COGAなど参考指針と、タスクが完了できるかの確認を併用する |
| 情報の追う場所 | 一次情報は WCAG 3 Introduction。TRの本文より先に、状態が草案であることを確認する |

- 利用者投稿（コメント、アップロード画像）の扱いは、Explainerではまだ Placeholder
- 投稿UIを作るときの足場は、完成している ATAG 2.0 の代替テキスト案内

> 参照:
> - [W3C WAI — WCAG 3 Introduction（Timeline）](https://www.w3.org/WAI/standards-guidelines/wcag/wcag3-intro/)
> - [Explainer — User-generated content](https://w3c.github.io/wcag3/explainer/)

## 関連して読むとよいテーマ

- WCAG 2.2の4原則 — 今の適合の枠組み
- ATAG 2.0 — 入稿ツール側の指針。WCAG 3は一部を取り込む方針
- 認知アクセシビリティ（COGA） — 達成基準に入りきらない認知面。WCAG 3が広げようとしている領域と重なる
- 画像のalt属性 — 1.1.1を実装に落とすとき
- [Beyond WCAGとは — 達成基準のチェックを超えた配慮](./beyond-wcag.md)

## まとめ

- WCAG 3.0は、W3C Accessibility Guidelines という次期ガイドラインの作業草案
- WCAG 2を置き換えず、今の実装と適合宣言の基準は WCAG 2.2
- 違いは、結果文と細かい要件、単一の Core 適合、報告ティア、アプリやXRまで含む範囲
- 2026年9月草案のティアやタグは案であり、Bronzeなどの呼称やスコア別案も併存している
- フロントエンドは WCAG 2.2 のAAをテンプレートで満たし、WCAG 3は方針の変化として追う
