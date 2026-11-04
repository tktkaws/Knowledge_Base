# CVSS v4.0スコアとは

## CVSS v4.0とは

- CVSS（Common Vulnerability Scoring System）は、脆弱性の技術的な深刻度を共通の基準で表す仕組み — FIRSTが仕様を公開している<sup>[1](#ref-1)</sup>
- スコアは0.0〜10.0で表され、9.0〜10.0は最も深刻な区分であるCriticalに該当する<sup>[1](#ref-1)</sup>
- 9.2は「攻撃が92%の確率で成功する」「92%のサイトが被害に遭う」という意味ではない — 特に基本評価だけでは、個別のサイトのリスクは判断できない<sup>[2](#ref-2)</sup>
- PHPの脆弱性「php2shell」で公表された9.2をきっかけに、スコアの読み方と対応の考え方を整理する — 情報の確認日は2026年10月22日<sup>[3](#ref-3)</sup>

**参照**

- <a id="ref-1"></a>[1] [FIRST — CVSS v4.0 Specification Document](https://www.first.org/cvss/v4.0/specification-document)
- <a id="ref-2"></a>[2] [FIRST — User Guide：CVSS Base Score Measures Severity, not Risk](https://www.first.org/cvss/v4.0/user-guide#CVSS-Base-Score-CVSS-B-Measures-Severity-not-Risk)
- <a id="ref-3"></a>[3] [GMO Flatt Security — PHPの重大な脆弱性「php2shell」の概要と対応指針](https://blog.flatt.tech/entry/php2shell)

## スコアには何が含まれているのか

- Base（基本評価）は、攻撃経路・必要な権限・被害の大きさなど、脆弱性そのものの性質を評価する<sup>[4](#ref-4)</sup>
- Threat（脅威評価）は、攻撃コードの公開や実際の悪用など、時間とともに変わる攻撃の成熟度を反映する<sup>[4](#ref-4)</sup>
- Environmental（環境評価）は、その組織の構成や、情報・サービスの重要度を反映する<sup>[4](#ref-4)</sup>
- Supplemental（補足評価）は、自動化の可否や復旧のしやすさなどを伝える追加情報 — このグループの値はスコア自体を変えない<sup>[4](#ref-4)</sup>
- 基本評価だけならCVSS-B、基本評価と脅威評価ならCVSS-BT、基本評価と環境評価ならCVSS-BE、3つを含むならCVSS-BTEと表記する<sup>[5](#ref-5)</sup>
- 点数を共有するときは、バージョンと評価グループも確認する — 同じ9.2でも、何を反映した数字かによって読み方が変わる<sup>[5](#ref-5)</sup>

**参照**

- <a id="ref-4"></a>[4] [FIRST — CVSS v4.0 Specification Document：Metric Groups](https://www.first.org/cvss/v4.0/specification-document)
- <a id="ref-5"></a>[5] [FIRST — User Guide：CVSS Nomenclature](https://www.first.org/cvss/v4.0/user-guide#CVSS-Nomenclature)

## スコアはどう計算されるのか

- 最初に各評価項目の値を決める — 基本評価には、攻撃条件を表す5項目と、影響を表す6項目がある<sup>[6](#ref-6)</sup>
- 攻撃条件は、AV（攻撃経路）、AC（攻撃の複雑さ）、AT（攻撃成立に必要な条件）、PR（必要な権限）、UI（利用者の関与）で表す<sup>[6](#ref-6)</sup>
- 影響は、脆弱なシステムのVC・VI・VAと、そこから影響を受ける別のシステムのSC・SI・SAで表す — Cは情報漏えいに関わる機密性、Iは改ざんに関わる完全性、Aは停止に関わる可用性<sup>[6](#ref-6)</sup>
- 各項目の値を並べた文字列がベクター — 点数の根拠を確認したり、計算を再現したりするために使う<sup>[6](#ref-6)</sup>
- v4.0では、各項目の点数を単純に足すのではなく、評価値の組み合わせからMacroVectorという深刻度の近いグループを求める<sup>[6](#ref-6)</sup>
- 次に、仕様の参照表にある値をもとに、グループ内での評価値の違いを補間してスコアを求める — 補間は、評価値がどの程度異なるかを点数へ反映する処理<sup>[6](#ref-6)</sup>
- 実務ではFIRSTの公式計算機へ公開ベクターを入力し、点数と選択項目を確認する — 手計算より、入力された条件が自分の環境にも当てはまるかの確認が重要になる<sup>[7](#ref-7)</sup>

**参照**

- <a id="ref-6"></a>[6] [FIRST — CVSS v4.0 Specification Document：Vector String / Scoring](https://www.first.org/cvss/v4.0/specification-document#CVSS-v4-0-Scoring)
- <a id="ref-7"></a>[7] [FIRST — CVSS v4.0 Calculator](https://www.first.org/cvss/calculator/4.0)

## php2shellの9.2を読み解く

- php2shellは、CVE-2026-109618とCVE-2026-109619の通称 — 前者はメモリ不足時の異常な処理中断、後者はリクエスト終了後に残る内部状態に関わる問題で、認証なしのリモートコード実行につながる<sup>[8](#ref-8)</sup><sup>[9](#ref-9)</sup>
- 両件の公式アドバイザリは、基本評価を9.2としている — 公開ベクターは共通で、`CVSS:4.0/AV:N/AC:H/AT:P/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N`となる<sup>[8](#ref-8)</sup><sup>[9](#ref-9)</sup>
- `AV:N`・`PR:N`・`UI:N`は、ネットワーク経由で攻撃でき、事前の権限も利用者の操作も不要という評価<sup>[8](#ref-8)</sup>
- `AC:H`・`AT:P`は、攻撃に技術的な複雑さがあり、成立に必要な条件もあるという評価 — 9.2でも「どの構成でも簡単に攻撃できる」という意味ではない<sup>[8](#ref-8)</sup>
- `VC:H`・`VI:H`・`VA:H`は、脆弱なシステムの情報漏えい・改ざん・停止への影響がいずれも大きいという評価<sup>[8](#ref-8)</sup>
- `SC:N`・`SI:N`・`SA:N`は、後続システムへの影響をNoneとして採点していることを示す — 実際の対応では、サーバが持つ認証情報や権限も別途確認する<sup>[8](#ref-8)</sup><sup>[10](#ref-10)</sup>
- この9.2は、認証や利用者操作を必要としない攻撃経路と大きな被害を、複雑さや成立条件も含めて評価した結果として読む<sup>[8](#ref-8)</sup><sup>[9](#ref-9)</sup>
- PHP公式はWebリクエストを処理するPHP環境を広く対象に挙げ、直ちに更新するよう勧告している — 今回の緊急性を判断する根拠には、この勧告も含まれる<sup>[11](#ref-11)</sup>

**参照**

- <a id="ref-8"></a>[8] [PHP公式アドバイザリ — RCE via OOM corruption](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w)
- <a id="ref-9"></a>[9] [PHP公式アドバイザリ — RCE via stale global state](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp)
- <a id="ref-10"></a>[10] [GMO Flatt Security — php2shellの影響緩和策と調査事項](https://blog.flatt.tech/entry/php2shell)
- <a id="ref-11"></a>[11] [PHP — 2026年10月21日のセキュリティ勧告](https://www.php.net/archive/2026.php)

## 高いスコアの脆弱性にどう対応するか

- 以下は、公式勧告をもとにした実務上の進め方 — スコアだけで一律の期限を決めず、影響確認と更新準備を並行して進める<sup>[12](#ref-12)</sup><sup>[13](#ref-13)</sup>
- 初動では、対象製品・バージョン・構成・外部公開の有無・担当者を洗い出す — フロントエンド担当でも、WordPressやCMS、APIなどを支えるPHP環境について、運用担当やホスティング事業者へ確認できる
- 優先順位は、攻撃者から到達できるか、成立条件があるか、悪用の情報があるか、漏えい・停止時の影響が大きいかを合わせて判断する — 不明な項目は「未確認」として担当者と確認時刻を決める<sup>[12](#ref-12)</sup>
- php2shellでは、修正を含むPHP 8.2.35・8.3.36・8.4.27・8.5.12への更新が案内されている — OSやホスティング事業者のパッケージを使う場合は、修正の取り込み状況を提供元へ確認する<sup>[13](#ref-13)</sup>
- 更新後は、稼働中のPHPワーカーへの反映と主要機能の動作を確認する — 管理画面・フォーム・アップロードなど、利用している機能を対象にする
- 更新まで時間がかかる場合は、外部からのアクセス制限や一時停止を検討する — 個別の設定変更は攻撃経路を限定する暫定策であり、修正適用の代わりにはならない<sup>[13](#ref-13)</sup><sup>[14](#ref-14)</sup>
- `session.upload_progress.enabled`を無効にしただけで対応完了としない — 公式アドバイザリは、これで防げない経路もあるとしている<sup>[15](#ref-15)</sup>
- 更新作業と並行して、アクセスログ・PHPのエラーログ・不審なファイルなどを調べる — ログに異常が見つからないことだけでは安全の証明にならない<sup>[14](#ref-14)</sup><sup>[15](#ref-15)</sup>
- 悪い例は「9.2だから全件を同じ手順で更新する」「攻撃が複雑だから後回しにする」 — 良い例は「公開中の対象環境を特定し、公式勧告に沿って更新と暫定対応の担当・期限を決める」
- 対応記録には、対象環境、確認した出典と日時、修正状況、動作確認の結果、未解決事項を残す — 「更新予定」と「稼働環境への反映済み」を区別する

**参照**

- <a id="ref-12"></a>[12] [FIRST — User Guide：深刻度とリスクの関係](https://www.first.org/cvss/v4.0/user-guide#CVSS-Base-Score-CVSS-B-Measures-Severity-not-Risk)
- <a id="ref-13"></a>[13] [PHP — 修正バージョンと更新できない場合の対応](https://www.php.net/archive/2026.php)
- <a id="ref-14"></a>[14] [GMO Flatt Security — php2shellの対応指針](https://blog.flatt.tech/entry/php2shell)
- <a id="ref-15"></a>[15] [PHP公式アドバイザリ — Mitigation / Logging and configuration audit](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp)

## まとめ

- CVSS v4.0は脆弱性の深刻度を表す共通基準 — 9.2はCriticalに該当する
- 基本評価だけでは、個別の環境でのリスクや対応順序は決められない
- スコアは評価値の組み合わせと参照表・補間から求める — 根拠はベクターで確認する
- php2shellの9.2には、認証不要のネットワーク攻撃と大きな影響に加え、攻撃の複雑さや成立条件も反映されている
- 高いスコアを見たら、対象環境の特定と公式勧告の確認を始め、更新・暫定対応・調査を進める
- 今回は設定変更だけで完了とせず、修正の反映と動作確認まで記録する

**参照**

- [FIRST — CVSS v4.0 Specification Document](https://www.first.org/cvss/v4.0/specification-document)
- [FIRST — CVSS v4.0 User Guide](https://www.first.org/cvss/v4.0/user-guide)
- [PHP — 2026年10月21日のセキュリティ勧告](https://www.php.net/archive/2026.php)
