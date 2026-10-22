# ARIA APGの読み方 — ウィジェット実装パターン集の公式ガイド

## ARIA APGとは

**ARIA Authoring Practices Guide（APG）** は、WAI-ARIAを使ってアクセシブルなUIを設計・実装するためのW3Cの公式ガイド。

- タブ、ダイアログ、メニュー、コンボボックス、ツリーなど、一般的なUIパターンを探せる
- パターンの目的、キーボード操作、ロール・ステート・プロパティの組み合わせを確認できる
- 動作するサンプルとソースコードから、フォーカス管理や状態更新を学べる
- アクセシブルネーム、キーボードインターフェース、ランドマーク、ハイコントラスト対応などの横断的な実践も扱う

APGの価値は、ARIA属性の一覧ではなく、**あるロールを名乗るUIが、利用者に対して約束する操作と状態をまとめて示すこと** にある。

> 参照:
> - [W3C — ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
> - [W3C — APG Introduction](https://www.w3.org/WAI/ARIA/apg/about/introduction/)

## APGは仕様でも適合基準でもない

APGはW3Cが公開する公式資料だが、WAI-ARIAやWCAGのような規範的な標準ではない。

| 資料 | 主な役割 | 読むと分かること |
|---|---|---|
| HTML | ネイティブ要素の意味と動作を定める | `button`、`input`、`dialog`などが元から持つ機能 |
| WAI-ARIA | ロール、ステート、プロパティの要件を定める | `tab` に必要な状態や、各属性を使えるロール |
| WCAG | Webコンテンツが満たす成果を定める | キーボード操作、名前・役割・値、フォーカスなどの達成基準 |
| APG | 仕様と一般的な操作慣習を実装パターンへまとめる | タブならどのキーで移動し、どのARIAをどう同期するか |

- APGに「適合」という判定モデルはない
- APGの例と同じ属性を付けただけで、WCAGへ自動的に適合するわけではない
- APGのキーボード操作には、ARIA仕様の要件に加え、デスクトップUIで定着した慣習も含まれる
- `Optional` と明記された操作は任意。それ以外は、相互運用性のため実装要件として扱うのが安全
- APGから外れる設計も直ちに不正ではないが、仕様への適合、予測可能な操作、支援技術との相互運用を自分で説明・検証する必要がある

2026年10月23日時点では、通常は更新が続く **APGのWebサイト** を参照する。過去の技術報告書「WAI-ARIA Authoring Practices 1.2」は2022年にRetiredとなっているため、古いURLや記事だけを実装根拠にしない。

> 参照:
> - [W3C — APG is Not a Normative Standard](https://www.w3.org/WAI/ARIA/apg/about/introduction/#apg-is-not-a-normative-standard)
> - [W3C — WAI-ARIA Authoring Practices 1.2 publication history](https://www.w3.org/standards/history/wai-aria-practices-1.2/)

## 最初に「Read Me First」を読む

パターンを探す前に、APGの **Read Me First** を読む。重要なのは「ARIAを付ければアクセシブルになる」のではないという前提。

### ロールは振る舞いの約束

```html
<!-- roleを付けても、ボタンの動作は追加されない -->
<div role="button">注文を確定</div>
```

この要素は支援技術にはボタンとして伝わるが、ブラウザは次の機能を自動では追加しない。

- Tabキーでのフォーカス
- EnterキーとSpaceキーでの起動
- 無効状態の扱い
- ボタンとして見分けられるフォーカス表示

同じ目的なら、ネイティブのHTMLを使う。

```html
<button type="button">注文を確定</button>
```

APGは自作ウィジェットを始める許可証ではない。まずネイティブHTMLで実現できるかを確認し、不足する意味や状態だけをARIAで補う。

### ARIAは元の意味を上書きできる

ARIAは意味を追加できる一方、HTMLの意味や内容を別のものとして支援技術へ伝えることもできる。誤ったロールやアクセシブルネームは、画面に見えるUIと支援技術が認識するUIを食い違わせる。

- ロールに対応する操作を実装できないなら、そのロールを付けない
- ネイティブ要素が持つ意味を、不必要な `role` で上書きしない
- `aria-label` で、画面に表示された分かりやすい文言を隠さない
- 対象とするブラウザと支援技術の組み合わせでテストする

> 参照:
> - [W3C — APG Read Me First](https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/)
> - [W3C — Using ARIA](https://www.w3.org/TR/using-aria/)

## 見た目ではなく、目的と操作でパターンを選ぶ

似た外観でも、UIの目的が違えば使うパターンと操作方法も変わる。検索結果から最初にサンプルコードを開くのではなく、各ページの **About This Pattern** を先に読む。

| 見た目 | 確認すること | 候補になるパターン |
|---|---|---|
| 見出しを押すと内容が開く | 一つのボタンが一つの領域を表示・非表示にするか | Disclosure |
| 横並びの項目で内容が切り替わる | 一度に一つの関連パネルを表示するか | Tabs |
| ボタンから選択肢が開く | コマンドを実行するか、値を選ぶか | Menu Button、Listbox、Combobox |
| ヘッダーからリンク一覧が開く | OS風のコマンドメニューか、サイト内の移動か | MenuではなくDisclosure Navigationの場合が多い |
| 行と列が並ぶ | 読むための表か、セルを矢印キーで操作する複合ウィジェットか | Table、Grid |
| 補足が浮かぶ | 操作できない短い説明か、フォーカス可能な内容を含むか | Tooltip、非モーダルDialog |

特に `menu` は「開閉するリンク一覧」の総称ではない。APGのMenuは、デスクトップアプリのメニューのように矢印キーで項目間を移動し、Tabキーで項目間を巡回しない複合ウィジェット。一般的なサイトナビゲーションに安易に適用すると、リンク利用者が予想する操作を変えてしまう。

> 参照:
> - [W3C — APG Patterns](https://www.w3.org/WAI/ARIA/apg/patterns/)
> - [W3C — Menu and Menubar Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/menubar/)
> - [W3C — Disclosure Navigation Menu Example](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/examples/disclosure-navigation/)

## パターンページを読む順番

APGのパターンページは、主に四つの区画で構成される。上から順に読むと、コードだけを切り取る失敗を避けやすい。

### 1. About This Patternで用途を確定する

- そのUIが何をするものか
- 似たパターンと何が違うか
- 適切でない利用場面や注意事項があるか
- フォーカスと選択、展開と選択など、状態をどう区別するか

ここで自分のUIと目的が一致しなければ、別のパターンかネイティブHTMLへ戻る。

### 2. Examplesで自分の要件に近い変種を選ぶ

同じパターンにも複数の例がある。たとえばTabsには自動選択と手動選択、Comboboxには編集可能・選択のみ・候補の自動選択などがある。

- 見た目ではなく、入力方法と状態遷移が近い例を選ぶ
- 複数の例を混ぜるときは、ロール、フォーカス、選択状態の整合を取り直す
- Exampleページ冒頭のアクセシビリティ機能とサポート上の注意を読む
- ブラウザ上で、マウスより先にキーボードだけで操作してみる

### 3. Keyboard Interactionを操作仕様にする

キー一覧を参考情報で終わらせず、実装とテストの受け入れ条件へ変換する。

```md
## Tabsの受け入れ条件

- Tabでタブリストへ入ると、選択中のタブにフォーカスする
- 左右矢印で前後のタブへフォーカスを移す
- 末尾の次は先頭、先頭の前は末尾へ移る
- 手動選択ではEnterまたはSpaceでパネルを切り替える
- Tabでタブリストを抜け、表示中のパネル側へ進める
- フォーカス位置を常に視覚的に確認できる
```

`Optional`、方向、折り返し、無効な項目、削除後のフォーカスなど、注記も含めて仕様を決める。

### 4. Roles, States, and Propertiesで意味を実装する

最後に、操作と状態を支援技術へ伝えるためのARIAを確認する。

- コンテナと子要素に必要なロール
- 必須の親子関係
- `aria-labelledby` や `aria-controls` が参照するID
- `aria-selected`、`aria-expanded`、`aria-checked` などの初期値
- 操作に応じて更新する値
- 横向き・縦向きなどで省略時の値と異なる属性

属性を付けた時点ではなく、**見た目、DOM、フォーカス、ARIAの状態が同じ事実を示し続けているか** を確認する。

> 参照:
> - [W3C — Tabs Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/)
> - [W3C — Combobox Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/)

## キーボード操作は属性とは別に実装する

ARIAは支援技術へ意味と状態を伝えるが、JavaScriptの動作を追加しない。複合ウィジェットでは、ページ内のTab移動と、部品内の矢印キー移動を分けて考える。

| 操作 | 一般的な役割 |
|---|---|
| `Tab` / `Shift + Tab` | コンポーネント間を移動する |
| 矢印キー | タブ、メニュー、リストボックスなどの内部を移動する |
| `Enter` / `Space` | 項目の起動、選択、開閉を行う |
| `Escape` | ポップアップや一時的なUIを閉じ、必要なら起点へ戻す |
| `Home` / `End` | 複合ウィジェット内の先頭・末尾へ移動する場合がある |

複合ウィジェット内のフォーカス管理には、主に次の方法がある。

### ロービングtabindex

- 子要素の一つだけを `tabindex="0"` にする
- 残りを `tabindex="-1"` にする
- 矢印キーで値を入れ替え、対象要素へ `focus()` する
- 実際のDOMフォーカスが移るため、フォーカス表示を合わせやすい

### aria-activedescendant

- DOMフォーカスはコンテナや入力欄に置いたままにする
- `aria-activedescendant` へ現在の項目のIDを設定する
- 見た目のアクティブ表示、スクロール、選択状態は実装側で同期する
- 対象ロールと所有関係がARIA仕様の条件を満たすか確認する

どちらを使う場合も、フォーカスと選択は同じとは限らない。たとえば手動選択のTabsでは、矢印キーでフォーカスだけを移し、EnterまたはSpaceを押したときに `aria-selected` と表示パネルを変更する。

> 参照:
> - [W3C — Developing a Keyboard Interface](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)
> - [ロービングタブインデックスパターン](./roving-tabindex.md)

## 小さなパターンで読み方を試す

Disclosureは、一つのボタンで一つの領域を開閉するパターン。ネイティブの `button` を使えばEnterとSpaceの動作はブラウザが提供するため、実装側は展開状態と表示状態の同期に集中できる。

```html
<button
  type="button"
  id="filter-trigger"
  aria-expanded="false"
  aria-controls="filter-panel"
>
  詳細条件
</button>

<div id="filter-panel" hidden>
  <label>
    価格の上限
    <input type="number" name="max-price">
  </label>
</div>
```

```js
const trigger = document.querySelector('#filter-trigger');
const panel = document.querySelector('#filter-panel');

trigger.addEventListener('click', () => {
  const willOpen = trigger.getAttribute('aria-expanded') === 'false';

  trigger.setAttribute('aria-expanded', String(willOpen));
  panel.hidden = !willOpen;
});
```

APGの記述と照らすと、次の対応になる。

| APGの区画 | この実装で確認すること |
|---|---|
| About This Pattern | ボタンと、そのボタンが開閉する領域の二要素で構成される |
| Keyboard Interaction | フォーカス中にEnterまたはSpaceで開閉できる |
| Roles | `button` 要素の暗黙のbuttonロールを使う |
| States | 閉じているとき `aria-expanded="false"`、開いているとき `true` |
| Properties | `aria-controls` は開閉対象のIDを参照する。APGでは任意 |

`aria-expanded` だけを変えてパネルを隠さない、またはパネルだけを表示して `aria-expanded` を更新しない実装は、画面と支援技術へ異なる状態を伝える。

> 参照: [W3C — Disclosure (Show/Hide) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/)

## サンプルコードをそのまま本番へ持ち込まない

APGの例は、アクセシビリティの概念を説明する **例示** であり、完成したUIライブラリやデザインシステムではない。

- 自分のDOM構造、データ更新、ルーティングに合わせて状態管理を設計する
- 入力方式、エラー処理、非同期読み込み、項目の追加・削除後にもフォーカスを失わないようにする
- 日本語入力、右から左へ書く言語、長いラベル、翻訳後の文字量など、製品の言語要件を確認する
- レスポンシブ表示、ズーム、タッチ、画面回転でも操作できるか確認する
- forced-colorsなど、OSの配色設定でも状態とフォーカスを見分けられるようにする
- 対象とするOS、ブラウザ、スクリーンリーダーの組み合わせでテストする
- APGサイトの例が更新される可能性を考え、参照日や採用した挙動を設計資料に残す

APG自身も、サンプルの目的はアクセシビリティ概念の学習であり、本番用コード、ローカライズ、広範なプラットフォーム相互運用まで提供するものではないと説明している。また、モバイルとタッチの互換性は例ごとに網羅されていない。

> 参照:
> - [W3C — APG is not a UI Design System](https://www.w3.org/WAI/ARIA/apg/about/introduction/#apg-is-not-a-ui-design-system)
> - [W3C — APG Read Me First: Mobile and Touch Support](https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/#mobile-and-touch-support)

## よくある読み違い

### 「ARIA属性が一致したから完成」

ロールと属性は実装の一層にすぎない。キーボード操作、フォーカス、表示、アクセシブルネーム、状態更新が揃って初めてパターンになる。

### 「APGの全キー操作は任意」

APG全体は参考情報だが、パターンには仕様由来の要件と、利用者が期待する確立した操作慣習が含まれる。任意の箇所は `Optional` と示される。根拠なく省略すると、同じロールなのに操作できないUIになる。

### 「見た目がドロップダウンだからmenu」

ドロップダウンは外観の呼び名。中身がサイト内リンク、値の選択、入力候補、コマンドのどれかでパターンは変わる。先に利用目的と操作を言語化する。

### 「矢印キーで移動した項目は必ず選択する」

フォーカス移動と選択は別の状態。Tabsの自動選択はパネルを遅延なく表示できる場合に適し、読み込みで待たせるなら手動選択を検討する。ListboxやComboboxでも、選択がいつ確定するかを例ごとに確認する。

### 「APGどおりならWCAG試験は不要」

APGはコンポーネントの意味と操作を強く支援するが、コントラスト、拡大、エラー、動き、タッチターゲット、ページ全体のフォーカス順などを一つのパターンだけで保証しない。WCAG評価と利用者の主要タスクのテストは別に行う。

## 実装へ落とす手順

1. UIが達成する目的と、利用者が行う操作を文章にする
2. ネイティブHTMLだけで実現できないか確認する
3. APGのAbout This Patternを読み、目的が一致するパターンを選ぶ
4. 要件に近いExampleを選び、異なる例を混ぜる箇所を記録する
5. Keyboard Interactionを受け入れ条件とテストケースへ変換する
6. Roles, States, and Propertiesを初期状態と状態遷移へ割り当てる
7. フォーカス、選択、展開、表示、読み込み中などの状態を一つのモデルで管理する
8. キーボード、自動検査、スクリーンリーダー、拡大・配色・タッチで確認する
9. 実装したパターン、APG参照URL、意図的な差分、試験環境を残す

### レビューで使うチェックリスト

- [ ] UIの目的は選んだパターンのAbout This Patternと一致する
- [ ] ネイティブHTMLを優先している
- [ ] すべての操作をキーボードだけで完了できる
- [ ] Tabキーと矢印キーの役割を混同していない
- [ ] フォーカス位置を常に視覚的に確認できる
- [ ] 操作後にフォーカスが `body` や削除済み要素へ失われない
- [ ] 表示状態と `aria-expanded`、選択状態と `aria-selected` などが同期する
- [ ] ID参照、親子関係、アクセシブルネームが正しい
- [ ] Optionalな操作を採用するか、採用しないか決めている
- [ ] APGの例をコピーした部分も製品の対象環境で再テストした
- [ ] WCAGと主要タスクの評価を別途実施した

## 関連して読むとよいテーマ

- [WAI-ARIAのファーストルール — ARIAより先にネイティブHTMLを使う](./aria-first-rule.md)
- [ARIAロールの基本 — role属性の種類と使い方](./aria-roles.md)
- [キーボードナビゲーションの基本 — Tab, Enter, Escape](./keyboard-navigation-basics.md)
- [フォーカス管理 — tabindex, focus(), フォーカストラップ](./focus-management.md)
- [ロービングタブインデックスパターン](./roving-tabindex.md)
- [アクセシブルなタブUIの実装](./accessible-tabs.md)
- [アクセシブルなドロップダウンメニューの実装](./accessible-dropdown-menu.md)

## まとめ

- APGは、ARIAを使うUIの目的、操作、フォーカス、ロール、状態をパターンとして学ぶ公式ガイド
- WAI-ARIAやWCAGのような規範的標準ではなく、APGの例と同じにしただけで適合が保証されるわけではない
- 最初にRead Me Firstを読み、ネイティブHTMLを優先し、ロールが約束する動作を理解する
- パターンページは、About This Pattern、Examples、Keyboard Interaction、Roles, States, and Propertiesの順に読む
- パターンは外観ではなく目的と操作で選び、フォーカス、選択、表示、ARIAの状態を同期させる
- サンプルは学習用の例示。製品の言語、非同期処理、タッチ、配色、ブラウザ・支援技術環境に合わせて設計とテストを行う
