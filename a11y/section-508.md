# Section 508について — 米国でVPATが求められる法的な背景

## Section 508とは

- **Section 508** は、米国 Rehabilitation Act of 1973（リハビリテーション法）第508条の通称
- 法令上の条文は **29 U.S.C. § 794d**
- 米国の連邦機関がICTを **開発、調達、保守、利用** するときに適用される
- 障害のある連邦職員と、連邦機関の情報・サービスを利用する障害のある一般市民に、障害のない人と同等・同程度の情報アクセスを提供することが目的
- 対象はWebサイトだけではない。ソフトウェア、モバイルアプリ、電子文書、動画、ハードウェア、キオスク、サポート文書・サービスなどのICTを含む
- 技術要件は、U.S. Access Boardが定める **Revised 508 Standards（36 CFR Part 1194）** に置かれている
- 連邦調達では Federal Acquisition Regulation（FAR）Part 39が、Section 508とRevised 508 Standardsを調達手続きへ接続する

> 参照:
> - [Section508.gov — Section 508 of the Rehabilitation Act](https://www.section508.gov/manage/laws-and-policies/section-508-law/)
> - [U.S. Access Board — Revised 508 Standards and 255 Guidelines](https://www.access-board.gov/ict/)
> - [Acquisition.gov — FAR Part 39](https://www.acquisition.gov/far/part-39)

## 法律・技術基準・調達文書を分ける

Section 508への対応は、一つの文書だけで決まらない。

| 層 | 文書・主体 | 役割 |
|---|---|---|
| 法律 | 29 U.S.C. § 794d | 連邦機関に、アクセシブルなICTを開発・調達・保守・利用する義務を置く |
| 技術基準 | 36 CFR Part 1194 | 対象範囲、WCAG、ハードウェア、ソフトウェアなどの具体的要件を定める |
| 調達規則 | FAR Part 39 | 調達時に適用要件を特定し、例外・免除を記録する手続きを定める |
| 募集・契約 | Solicitation / Contract | 製品固有の要件、提出証拠、試験、検収、修正義務を定める |
| 適合報告 | ACR | 製品が各要件をどこまで満たすかを、ベンダーが説明する |

- ベンダーにSection 508が直接「VPATという書式を使え」と命じる構造ではない
- 連邦機関にはアクセシブルなICTを選ぶ義務があるため、製品を比較できる適合情報が必要になる
- その情報を共通形式で出す手段として、VPATを使ったACRが調達文書で要求される
- 最後に適用範囲、例外・免除、調達判断を決める責任は連邦機関側にある。ベンダーが自社判断で免除を宣言するものではない

> 参照:
> - [Acquisition.gov — FAR 39.201–39.205](https://www.acquisition.gov/far/part-39)
> - [Section508.gov — Buy or Sell Accessible ICT](https://www.section508.gov/buy-sell/)

## 誰に適用されるか

| 対象 | Section 508の扱い |
|---|---|
| 米国の連邦機関 | 開発・調達・保守・利用するICTに適用される |
| 連邦機関へICTを販売する企業 | 契約要件を通じて、適合情報の提出と製品の対応を求められる |
| 州・地方政府 | Section 508が直接適用されるとは限らない。ADA Title IIや州法などを別に確認する |
| 民間企業の一般サイト | Section 508の直接対象ではない。ADA、契約、州法、業界要件などを別に確認する |
| 連邦資金を受ける組織 | Section 504など別の法的義務が関係し得る |

- 「米国向けサイトだからSection 508が一律に適用される」わけではない
- 連邦政府との契約、助成、州法、業界規制によって、同じWCAGを参照していても法的根拠は変わる
- Section 501、504、508は目的が異なる。508の例外があっても、雇用差別やプログラムの差別禁止に関する別の義務が消えるわけではない
- 適用法の判断と、技術的にアクセシブルであるかの判断を分ける

> 参照:
> - [Section508.gov — Do Section 508 Standards Apply to My Website?](https://www.section508.gov/blog/do-section-508-accessibility-standards-apply-to-mywebsite/)
> - [Section508.gov — Relationship Between Sections 501, 504, and 508](https://www.section508.gov/manage/relationship-between-sections-508-504-and-501/)

## Revised 508 Standards

U.S. Access Boardは2017年1月18日に基準の改訂版を公表し、調達以外のICTには2018年1月18日から準拠が求められた。FARは2021年に改訂基準を正式に取り込み、2021年9月10日に施行した。

| 箇所 | 内容 |
|---|---|
| Appendix A, Chapter 1 | Section 508の適用と管理 |
| Appendix A, Chapter 2 | 対象範囲、例外、電子コンテンツ、ハードウェア、ソフトウェア、サポート |
| Appendix C, Chapter 3 | 機能的パフォーマンス基準 |
| Appendix C, Chapter 4 | ハードウェア |
| Appendix C, Chapter 5 | ソフトウェアと支援技術との相互運用、オーサリングツール |
| Appendix C, Chapter 6 | サポート文書とサポートサービス |
| Appendix C, Chapter 7 | WCAG 2.0などの引用規格 |
| Appendix D | 変更されていないレガシーICTを判断するための旧基準 |

- Webだけを見る場合でも、製品にソフトウェア、文書、動画、サポートが含まれれば対応する章を追加で見る
- 技術要件だけで機能を扱えない場合は、視覚、聴覚、発話、操作、認知などに関するChapter 3の機能的パフォーマンス基準を使う
- 代替設計や技術で、基準と実質的に同等以上のアクセシビリティとユーザビリティを提供する **Equivalent Facilitation** も認められる

> 参照:
> - [U.S. Access Board — Revised 508 Standards](https://www.access-board.gov/ict/)
> - [U.S. Access Board — FAR Updated to Incorporate Revised 508 Standards](https://www.access-board.gov/news/2021/09/13/federal-acquisition-regulation-far-updated-to-incorporate-revised-508-standards/)

## WCAGとの関係

Revised 508 Standardsは、Webコンテンツ、一定の電子文書、ソフトウェアについて **WCAG 2.0のレベルA・AAの達成基準と適合要件** を引用している。

| ICT | 主な規定 | WCAGの扱い |
|---|---|---|
| 公開Webコンテンツ | E205.2 / E205.4 | WCAG 2.0 A・AAと適合要件 |
| 対象となる内部の公式通信 | E205.3 / E205.4 | WCAG 2.0 A・AAと適合要件 |
| 非Web文書 | E205.4 | WCAG 2.0を「document」に読み替え、一部基準を除外 |
| ソフトウェア | E207 | WCAG 2.0に加え、Chapter 5の要件 |
| 電子的なサポート文書 | 602.3 | WCAG 2.0 A・AAと適合要件 |

- 2026年10月22日時点でも、Section 508が規範として引用するのは **WCAG 2.0**。WCAG 2.1や2.2へ自動更新されない
- WCAG 2.1 / 2.2を追加の契約要件や設計目標にすることはできるが、「Section 508がWCAG 2.2を要求する」とは書かない
- WCAG 2.1 / 2.2の追加達成基準には、リフロー、非テキストのコントラスト、ステータスメッセージ、ドラッグ操作、ターゲットサイズ、アクセシブルな認証などがある
- 現行法の最低線と、新規開発で採用する現在のアクセシビリティ基準を分けて記録する

> 参照:
> - [U.S. Access Board — E205 Electronic Content / E207 Software](https://www.access-board.gov/ict/)
> - [Section508.gov — Beyond Section 508: WCAG 2.1 and 2.2](https://www.section508.gov/event/best-practices-webinar-jul2024/)
> - [ITI — VPAT editions and WCAG versions](https://www.itic.org/policy/accessibility/vpat)

## Webサイトだけの基準ではない

連邦機関向けSaaSを納品するとき、ログイン後のWeb画面だけが対象とは限らない。

```html
<section aria-labelledby="support-heading">
  <h2 id="support-heading">サポート</h2>
  <a href="/manual.pdf">利用者マニュアル（PDF）</a>
  <a href="/accessibility">アクセシビリティ機能の説明</a>
  <a href="/contact">サポートへの問い合わせ</a>
</section>
```

- Web画面はWCAG 2.0のA・AAを確認する
- `manual.pdf` は電子文書として確認する
- ネイティブアプリやデスクトップクライアントにはChapter 5も適用する
- マニュアルには、製品のアクセシビリティ機能と支援技術との互換性、その使い方を記載する
- ヘルプデスクは、障害のある人のコミュニケーションニーズに対応できる連絡方法を提供する

### オーサリングツールは出力も見る

CMS、レポート生成、フォームビルダーなど、電子コンテンツを作るツールにはChapter 5のオーサリング要件が関係する。

```html
<label for="alt">画像の代替テキスト</label>
<input id="alt" name="alt">

<label>
  <input type="checkbox" name="decorative">
  装飾画像として扱う
</label>
```

- 編集画面自体をアクセシブルにする
- アクセシブルな出力を作れる機能を用意する
- 変換時に代替テキストなどのアクセシビリティ情報を残す
- テンプレートを提供する場合は、アクセシブルなテンプレートを用意する

> 参照: [U.S. Access Board — Revised 508 Standards, Chapters 5 and 6](https://www.access-board.gov/ict/)

## 調達での流れ

Section 508がVPAT/ACRの要求へつながる流れは次のようになる。

1. 連邦機関が、調達するICTの機能と障害のある利用者のニーズを特定する
2. Revised 508 Standardsから、適用される要件を選ぶ
3. 募集要項に、要件、対象、提出する適合情報、試験・検収条件を書く
4. ベンダーが製品を試験し、ACRなどで要件ごとの対応状況を回答する
5. 機関がACR、追加証拠、デモ、独自試験を使って製品を比較する
6. 契約後も、更新、受入試験、不具合修正、再評価を管理する

```md
## ICTアクセシビリティ要求事項

- 基準: Revised Section 508 Standards（36 CFR Part 1194）
- 対象: Webアプリ、生成するPDF、利用者向け文書、サポートサービス
- 追加基準: WCAG 2.2 レベルA・AA
- 提出物: VPAT 2.5 Rev 508を用いた製品版ごとのACR
- ACRの記載: 適合レベル、評価方法、具体的な根拠、既知の不適合
- 検収: 発注者による自動・手動・支援技術テスト
- 更新: UIまたは機能を変更したリリースごとにACRへの影響を確認する
```

- 追加のWCAG 2.2は契約上の要件であり、Revised 508 Standardsそのものと混同しない
- ACR提出だけを検収にせず、主要タスクをキーボードと支援技術で確認する
- 製品名だけでなく、対象バージョン、構成、プラットフォーム、対象機能を固定する

> 参照:
> - [Section508.gov — Request Accessibility Information from Vendors & Contractors](https://www.section508.gov/buy/request-accessibility-information/)
> - [Acquisition.gov — FAR Part 39](https://www.acquisition.gov/far/part-39)

## VPATとACRの違い

| 用語 | 意味 |
|---|---|
| **VPAT** | ITIが提供する、アクセシビリティ要件を行ごとに並べた未記入のテンプレート |
| **ACR** | 製品を評価し、VPATへ結果と説明を記入して完成させたAccessibility Conformance Report |

- 「VPATを提出する」という会話では、実際には記入済みの **ACR** を指すことが多い
- VPATは法律や政府が作った様式ではなく、Information Technology Industry Council（ITI）が提供する登録サービスマーク付きのテンプレート
- VPATを使うこと自体は唯一の法定方法ではないが、Section 508の要件をまとめた共通形式として広く使われる
- Section508.govは、政府に製品を検討してもらう場合、通常はベンダーにACRが求められると説明している
- **VPAT認証という制度はない**。完成したACRを政府やITIへ提出して認証ロゴを得る仕組みでもない
- ACRは第三者認証とは限らず、製品所有者やベンダーが自ら作成できる。第三者監査が必要かは募集・契約条件による

> 参照:
> - [Section508.gov — ACR/VPAT FAQ](https://www.section508.gov/sell/acr-vpat-faq/)
> - [ITI — VPAT](https://www.itic.org/policy/accessibility/vpat)

## なぜ「Voluntary」なのに求められるのか

**Voluntary** は、VPATという特定テンプレートの採用が任意であることを指す。連邦機関のSection 508対応や、調達で要求された適合情報の提出まで任意という意味ではない。

- 連邦機関は、購入候補が適用要件をどこまで満たすか評価しなければならない
- Section508.govの調達案内は、標準的なICT品目ごとに完全なACRを要求するよう勧めている
- 募集要項がACRを提出条件にすれば、ベンダーは審査対象になるために提出する必要がある
- VPATは、Section 508の行番号、評価用語、説明欄が揃っているため、独自書式より比較しやすい
- 法的義務の主体は連邦機関でも、契約条件を通じてベンダーの設計、試験、報告、修正へ影響する

つまり、**法律が直接VPATを義務化するのではなく、法律に従って調達判断を行うために、ACRが要求される**。

> 参照:
> - [Section508.gov — ACR/VPAT FAQ「Isn't the VPAT voluntary?」](https://www.section508.gov/sell/acr-vpat-faq/)
> - [Section508.gov — Request Accessibility Information](https://www.section508.gov/buy/request-accessibility-information/)

## VPATの版を選ぶ

2026年10月22日時点でITIが公開する最新版は **VPAT 2.5Rev（2025年4月）**。

| Edition | 主な基準 | 主な用途 |
|---|---|---|
| 508 | Revised Section 508、WCAG 2.0 | 米国連邦政府向け |
| EU | EN 301 549、WCAG 2.1 | EU向け |
| WCAG | WCAG 2.2 | WCAGだけを報告する場合 |
| INT | 508、EN 301 549、WCAG 2.2 | 複数市場を一つの報告で扱う場合 |

- 米国連邦向けなら、最低でもRevised 508 Standardsを含む **Rev 508** または **Rev INT** を選ぶ
- `VPAT 2.x` だけで判断せず、Editionと、その中で参照されるWCAGの版を見る
- EU Editionと508 Editionでは、同じ行番号に見えても対象規格とWCAGの版が異なる
- 募集要項が指定したテンプレート、版、追加資料を優先する

> 参照: [ITI — VPAT 2.5Rev editions](https://www.itic.org/policy/accessibility/vpat)

## ACRの読み方

VPAT 2.5は、各要件について四つの適合レベルを使う。

| 表記 | 読み方 |
|---|---|
| Supports | 要件を満たす |
| Partially Supports | 一部で満たすが、例外や不適合がある |
| Does Not Support | 要件を満たさない |
| Not Applicable | 製品にその要件を適用する機能・条件がない |

- `Supports` の数だけを集計しない。利用者の主要タスクを止める一つの不適合が、製品選定では重大になる
- `Partially Supports` には、影響する画面、操作、プラットフォーム、回避策、修正予定が必要
- `Not Applicable` は未試験ではない。適用されない理由を説明する
- 製品名、版、公開日、評価範囲、試験環境、評価方法を確認する
- 「コードレビューをした」「自動ツールを使った」だけでは、キーボード操作やスクリーンリーダーの結果は分からない
- ACRが古い製品版のものなら、現在のUIを評価した証拠にならない。製品の変更・更新に応じてACRも見直す
- 発注者は、ACRの記載を独自試験やデモで検証できる

> 参照:
> - [ITI — VPAT 2.5Rev conformance levels](https://www.itic.org/policy/accessibility/vpat)
> - [Section508.gov — ACR/VPAT FAQ](https://www.section508.gov/sell/acr-vpat-faq/)

## 例外・免除はベンダーが決めない

FARは、適用除外と免除を区別し、連邦機関側の文書化を求める。

| 区分 | 例 |
|---|---|
| Exception | 国家安全保障システム、契約遂行のためだけに請負業者が内部利用する付随的ICT、保守員専用空間の一部 |
| Exemption | 過度な負担、ICTの本質的な変更、市場に完全適合する製品・サービスがない場合 |

- 過度な負担や本質的変更は、該当する範囲と理由を連邦機関が記録する
- 完全適合品が市場にない場合も、機関は業務ニーズに合い、基準を最もよく満たす製品を選ぶ
- 免除が適用されても、障害のある利用者へ代替手段によるアクセスを提供する
- ベンダーは不適合を隠して「免除予定」とせず、ACRで具体的に報告する

> 参照: [Acquisition.gov — FAR 39.204 Exceptions / 39.205 Exemptions](https://www.acquisition.gov/far/part-39)

## フロントエンドの実務への落とし方

| 工程 | 行うこと |
|---|---|
| 対象確認 | 連邦機関向けか、契約でSection 508やVPATが指定されているか確認する |
| 機能分解 | Web、ソフトウェア、PDF、動画、ハードウェア、サポートに分ける |
| 要件整理 | Revised 508 Standardsの適用条項と、追加のWCAG 2.2要件を分ける |
| 実装 | コンポーネントだけでなく、主要タスクと生成コンテンツをアクセシブルにする |
| 試験 | 自動、キーボード、拡大、スクリーンリーダー、文書・メディア試験を組み合わせる |
| 報告 | 製品版を固定し、要件ごとの結果、根拠、不適合、回避策をACRへ書く |
| リリース後 | UI・機能・依存サービスの変更がACRへ与える影響を確認する |

- 開発の最後に法務や営業だけでACRを埋めない。各行の根拠になる試験記録を開発中から残す
- `Supports` を増やすことより、申請、検索、認証、支払いなど主要タスクを完了できることを優先する
- VPATがWCAG 2.0を基礎にしていても、新しいコンポーネントはWCAG 2.2まで確認する
- サードパーティ製の決済、チャット、認証を組み込む場合、その部分も製品の実際の利用範囲として評価する

## JIS X 8341-3・EN 301 549との違い

| 観点 | JIS X 8341-3:2016 | EN 301 549 | Revised Section 508 |
|---|---|---|---|
| 地域・制度 | 日本の国家規格 | 欧州規格とEU法令 | 米国連邦法と連邦調達 |
| 主な範囲 | Webコンテンツ | ICT製品・サービス全体 | 連邦機関が開発・調達・保守・利用するICT |
| WCAG | WCAG 2.0 | V3.2.1は2.1、V4.1.1は2.2 | WCAG 2.0 |
| 報告 | WAICの方針・試験結果 | 要件ごとの適合評価 | ACRが調達で広く使われる |
| 代表的なVPAT Edition | 固有のEditionなし | EU / INT | 508 / INT |

- 同じ製品を日本、EU、米国連邦へ出す場合、WCAGの共通試験を再利用しても、適用条項と報告書は市場ごとに分ける
- INT Editionを使えば一つのACRへ複数規格を含められるが、各規格の版差と独自要件は残る

## 関連して読むとよいテーマ

- [JIS X 8341-3について — 日本のウェブアクセシビリティ規格とWCAGの対応](./jis-x-8341-3.md)
- [EN 301 549について — 欧州の調達で参照されるアクセシビリティ規格](./en-301-549.md)
- [WCAG 2.2の4原則 — 知覚可能・操作可能・理解可能・堅牢](./wcag-four-principles.md)
- VPATについて — 製品のアクセシビリティ適合を報告する書式
- WCAG2ICTについて — Web以外の文書やソフトウェアへWCAGを当てはめる考え方

## まとめ

- Section 508は、米国連邦機関が開発・調達・保守・利用するICTをアクセシブルにする連邦法上の要件
- Revised 508 Standardsが技術要件を、FAR Part 39が連邦調達での適用・例外・免除を定める
- Webだけでなく、電子文書、ソフトウェア、ハードウェア、動画、サポート文書・サービスまで対象になる
- 現行基準が引用するのはWCAG 2.0のレベルA・AA。WCAG 2.1 / 2.2は追加要件として分ける
- VPATはITIの未記入テンプレートで、記入済みの報告書がACR。VPAT認証は存在しない
- 法律がVPATという書式を直接義務化するのではなく、連邦機関が調達候補の適合性を評価するためACRを要求する
- ACRは証拠の入口であり、製品版、評価範囲、試験方法、具体的な不適合を確認し、必要に応じて発注者が試験する
