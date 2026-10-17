# Beyond WCAGとは — 達成基準のチェックを超えた配慮

## Beyond WCAGとは

- **Beyond WCAG** は規格名ではない。WCAGの達成基準を満たしたうえで、利用者が目的の作業を終えられるところまで見る、という実務の呼び方
- WCAGは、障害のある人にとって比例して大きな障壁のうち、テストできるものを達成基準にしたガイドライン
- 適合は「その基準を満たした」という判定。すべての障害、すべての支援技術、すべての作業が快適になる保証ではない
- W3Cは、達成基準をすべて満たしても、幅広い障害のある人にとって常に使えるとは限らない、と適合の解説に書いている
- そのうえで、機能的な確認に加えてユーザビリティテストを推奨し、障害のある人を参加者に含めるよう案内している
- 達成基準を外して「体験だけ見る」ことではない。床はWCAG適合、その上に作業の完了しやすさを足す

> 参照:
> - [W3C WAI — Understanding Conformance（WCAG 2.2）](https://www.w3.org/WAI/WCAG22/Understanding/conformance.html)
> - [W3C WAI — Accessibility, Usability, and Inclusion](https://www.w3.org/WAI/fundamentals/accessibility-usability-inclusion/)

## 達成基準がすべてを書かない理由

- 達成基準は、機械と人の評価を組み合わせて、満たしたかどうかを高い確度で判定できる必要がある
- 対象は、障害のない人にもある使いにくさのうち、障害のある人により大きく効くもの
- 判定できない好み、文脈依存のわかりやすさ、作業全体の効率は、達成基準に入りにくい
- レベルAAAはサイト全体への適合宣言が現実的でないものも含む。多くの案件の目標はAとAAで、AAAの一部は「任意」のまま残る
- 認知の負荷、平易な言葉、記憶に頼らない手順は、COGAなどの補足指針側に厚い。WCAG適合の必須条件ではない

> 参照:
> - [W3C WAI — Understanding Conformance](https://www.w3.org/WAI/WCAG22/Understanding/conformance.html)
> - [W3C WAI — Cognitive Accessibility Guidance](https://www.w3.org/WAI/WCAG2/supplemental/)

## 三層に分ける

| 層 | 見ているもの | 足りないこと |
|---|---|---|
| 自動検査 | コントラスト、名前の欠落、一部のARIA | 代替テキストの意味、操作の成否、作業の流れ |
| 達成基準の適合 | 適用されるA / AAを、ページとプロセスで判定 | ページをまたぐ面倒さ、同じ達成基準の「最低限」と「わかりやすい」の差 |
| Beyond WCAG | 障害のある人が、自分の支援技術で作業を終えられるか | 適合の代わりにはならない。参加者の障害の種類を超えて一般化できない |

- チェックリストだけに寄せると、人の操作が落ちる。W3Cはこれを、技術面に寄ってインタラクションが抜ける状態として説明している
- 利用者参加だけでもアクセシビリティの判定にはならない。少数の参加者では、障害の多様性と支援技術の組み合わせを覆えない
- 両方を使う状態を、W3Cは **usable accessibility**（使えるアクセシビリティ）や accessible UX と呼んでいる
- 適合評価の方法である WCAG-EM も、評価の中で障害のある利用者に関わることを推奨している

> 参照:
> - [W3C WAI — Accessibility, Usability, and Inclusion](https://www.w3.org/WAI/fundamentals/accessibility-usability-inclusion/)
> - [W3C WAI — Involving Users in Evaluating Web Accessibility](https://www.w3.org/WAI/test-evaluate/involving-users/)
> - [W3C WAI — WCAG-EM Overview](https://www.w3.org/WAI/test-evaluate/conformance/wcag-em/)

## コードで見る「基準は通るが、操作が残る」

### リンクの目的は文の中では足りる

WCAG 2.2 の 2.4.4（リンクの目的、コンテキスト内、A）は、リンク文字単体か、同じ文・段落・リスト項目などのプログラムから決まる文脈で目的が分かればよい。スクリーンリーダーのリンク一覧は、文脈を読まずリンク文字だけを並べることがある。

```html
<!-- 同じ文で目的が分かるので、2.4.4は満たしやすい -->
<p>料金プランの比較は<a href="/plans">こちら</a>。</p>
<p>解約手続きは<a href="/cancel">こちら</a>。</p>
```

```html
<!-- リンク一覧でも目的が残る -->
<p><a href="/plans">料金プランを比較する</a></p>
<p><a href="/cancel">解約手続きへ進む</a></p>
```

- 2.4.9（リンクの目的、リンクのみ、AAA）は、リンク文字だけで目的が分かることを求める。AA適合のチェックでは必須にならないことが多い
- フロントの実装では、AAAを宣言しなくても、リンク文字を行き先の動詞にする

> 参照: [W3C WAI — Understanding Link Purpose (In Context)](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-in-context.html)

### 各画面は合格でも、手続きが記憶頼りになる

ページ単体にラベル、エラー、見出しがあっても、前の画面の数字を覚えていないと次へ進めない手続きは残る。達成基準のページ検査では主作業の負荷が見えない。

```html
<!-- 悪い例：前画面の金額を覚えていないと確認できない -->
<h1>支払い確認</h1>
<label for="amount">請求額を再入力</label>
<input id="amount" name="amount" inputmode="numeric">
<button type="submit">支払う</button>
```

```html
<!-- 良い例：確認に必要な値を、その画面に再表示する -->
<h1>支払い確認</h1>
<p>請求額は <strong>12,800円</strong>（税込）です。</p>
<button type="submit">12,800円を支払う</button>
```

- 再表示は、達成基準のチェック欄を増やす作業というより、手続きを記憶から外す配慮
- 詳細なパターンは認知アクセシビリティ（COGA）の補足指針側

> 参照: [W3C WAI — Supplemental Guidance](https://www.w3.org/WAI/WCAG2/supplemental/)

## フロントエンドの実務への落とし方

| 工程 | 達成基準 | その先 |
|---|---|---|
| 実装 | 名前、キーボード、コントラスト、エラーの特定をコンポーネントで満たす | リンク文字、ボタン名、再表示を、検査の最低限より具体的にする |
| セルフレビュー | axeとキーボードで主要画面を見る | 主要タスクを最初から最後まで、キーボードだけで完了する |
| 適合判定 | 目標レベルの適用基準を、対象範囲について記録する | 「AA適合」を、作業が楽であることの宣言にしない |
| 利用者参加 | 適合の代替にしない | 支援技術の利用者に、本番と同じタスクを完了してもらう。障害の種類は報告書に書く |

- 開発中の確認は、早期の短い観察が、リリース直前の一回より効く。W3Cは開発の途中で非公式に利用者評価へ入ることを勧めている
- 参加者は「障害のある人」で一括りにしない。全盲のスクリーンリーダー利用と、拡大やキーボードのみでは、見つかる問題が違う
- 小さなテストは統計的な証明にならない。範囲と、見ていない障害を報告書に残す

> 参照: [W3C WAI — Involving Users in Evaluating Web Accessibility](https://www.w3.org/WAI/test-evaluate/involving-users/)

## WCAG 3との関係

- WCAG 3の草案は、適合（ものさし）と、よりアクセシブルな体験を分ける方向を探している
- 2026年10月時点では作業草案であり、Beyond WCAGの実装根拠にはしない
- 今の床は WCAG 2.2。その外側の配慮は、補足指針とタスク確認で足す

> 参照: [W3C WAI — WCAG 3 Introduction](https://www.w3.org/WAI/standards-guidelines/wcag/wcag3-intro/)

## 関連して読むとよいテーマ

- WCAG 2.2の4原則 — 達成基準の枠
- WCAG 3.0について — 適合の先を報告に含めようとする草案
- 認知アクセシビリティ（COGA） — 記憶・注意・読解の補足指針
- 支援技術ユーザーによる利用テスト — 自動・手動の先の確認（チェックリスト上の別テーマ）
- スクリーンリーダーでの手動テスト入門 — リンク一覧など、実際の操作

## まとめ

- Beyond WCAGは、達成基準の適合を床にしたうえで、作業が完了できるかを見る呼び方
- 適合はテスト可能な障壁への判定であり、すべての人にとって使いやすいことの証明ではない
- 自動検査、達成基準、利用者のタスク確認は別の層。どれか一つで残りを代替しない
- 実装では、文脈依存の「こちら」や、前画面の記憶を要する確認を、その画面で完結させる
- 宣言できる目標は引き続き WCAG 2.2。体験の確認は適合宣言とは分けて残す
