# DNSレコードの種類と役割 — A・AAAA・CNAME・MX・TXT・NS

## DNSレコードとは

- DNSレコードは、ドメインに関する情報をDNSサーバーに登録するデータ
- レコードの種類ごとに役割が異なり、ドメインの名前解決・メール配送・サービス連携などに使われる
- ドメインの権威DNSサーバー（ネームサーバー）に設定する
- DNS管理画面（レジストラやホスティングサービスの管理パネル）で編集するのが一般的

### レコードの基本フォーマット

```
名前           TTL    クラス  タイプ   値
example.com.   3600   IN      A       93.184.216.34
```

| フィールド | 説明 |
|---|---|
| 名前（Name） | レコードの対象ドメイン名 |
| TTL | キャッシュの有効期間（秒） |
| クラス（Class） | 通常は`IN`（Internet） |
| タイプ（Type） | レコードの種類（A, AAAA, CNAME など） |
| 値（Value / Data） | レコードの内容（IPアドレス、ドメイン名など） |

- 参照: [RFC 1035 — Domain Names - Implementation and Specification](https://datatracker.ietf.org/doc/html/rfc1035)

## 主要なDNSレコード一覧

| レコード | 用途 | 値の例 |
|---|---|---|
| A | ドメインをIPv4アドレスに対応付け | `93.184.216.34` |
| AAAA | ドメインをIPv6アドレスに対応付け | `2606:2800:220:1:248:1893:25c8:1946` |
| CNAME | ドメインを別のドメインのエイリアスにする | `www.example.com` → `example.com` |
| MX | メールの配送先サーバーを指定 | `mail.example.com`（優先度付き） |
| TXT | テキスト情報を格納（SPF、DKIM、所有権確認など） | `v=spf1 include:_spf.google.com ~all` |
| NS | ドメインの権威DNSサーバーを指定 | `ns1.example.com` |
| SOA | ゾーン（ドメイン）の管理情報 | プライマリNS、管理者メール、シリアル番号など |
| PTR | IPアドレスからドメイン名への逆引き | `34.216.184.93.in-addr.arpa` → `example.com` |
| SRV | 特定サービスのサーバーとポートを指定 | `_sip._tcp.example.com` |
| CAA | SSL証明書を発行できる認証局を制限 | `0 issue "letsencrypt.org"` |

## Aレコード

- ドメイン名をIPv4アドレスに対応付ける、最も基本的なレコード
- Webサイトのホスティングで必ず設定する

```
# 基本的なAレコード
example.com.       3600  IN  A  93.184.216.34

# サブドメインのAレコード
www.example.com.   3600  IN  A  93.184.216.34
blog.example.com.  3600  IN  A  198.51.100.10

# 複数のIPアドレスを設定（DNSラウンドロビン）
example.com.       3600  IN  A  93.184.216.34
example.com.       3600  IN  A  93.184.216.35
```

- 1つのドメインに複数のAレコードを設定すると、DNSラウンドロビンによる簡易的な負荷分散が可能
- `@`はゾーン名自体（example.com）を意味する記号（DNS管理画面で使用）

## AAAAレコード

- ドメイン名をIPv6アドレスに対応付ける
- Aレコードの IPv6版（「クワッドA」と読む）

```
example.com.  3600  IN  AAAA  2606:2800:220:1:248:1893:25c8:1946
```

- IPv4の枯渇に伴い、IPv6対応のためAAAAレコードの設定が増加
- AレコードとAAAAレコードを両方設定するのが一般的（デュアルスタック）
- ブラウザはIPv6を優先し、失敗した場合にIPv4にフォールバックする（Happy Eyeballs）

## CNAMEレコード

- ドメイン名を別のドメイン名のエイリアス（別名）にする
- CNAMEの参照先のAレコードが変わっても、CNAME側は変更不要

```
# www.example.com を example.com のエイリアスにする
www.example.com.  3600  IN  CNAME  example.com.

# CDNやSaaSサービスの指定
blog.example.com.  3600  IN  CNAME  custom.cdn-provider.com.
shop.example.com.  3600  IN  CNAME  shops.myshopify.com.
```

### CNAMEの制約

- **ゾーンの頂点（apex / ルートドメイン）にはCNAMEを設定できない**

```
# ✗ NG — ルートドメインにCNAMEは設定不可
example.com.  3600  IN  CNAME  other-domain.com.

# ✓ OK — サブドメインにはCNAME設定可能
www.example.com.  3600  IN  CNAME  other-domain.com.
```

- CNAMEは他のレコード（MX, TXTなど）と共存できない
- ルートドメインには必ずSOAレコードとNSレコードがあるため、CNAMEは設置不可
- 代替策として、一部のDNSプロバイダはALIASレコード（またはANAMEレコード）を提供

| レコード | 対応サービス | 説明 |
|---|---|---|
| ALIAS | Route 53、DNSimple など | ルートドメインでCNAMEのような動作を実現 |
| ANAME | Cloudflare（Flattening）など | プロバイダ独自の仕組み |

- 参照: [RFC 1034 — 3.6.2 CNAME](https://datatracker.ietf.org/doc/html/rfc1034#section-3.6.2)

## MXレコード

- メールの配送先サーバーを指定するレコード
- 優先度（Priority）の数値が小さいほど優先される

```
example.com.  3600  IN  MX  10  mail1.example.com.
example.com.  3600  IN  MX  20  mail2.example.com.
example.com.  3600  IN  MX  30  mail3.example.com.
```

### 優先度の仕組み

```
メール送信サーバー
  │
  ├── 優先度10: mail1.example.com に配送を試みる
  │     └── 失敗した場合 ↓
  ├── 優先度20: mail2.example.com に配送を試みる
  │     └── 失敗した場合 ↓
  └── 優先度30: mail3.example.com に配送を試みる
```

### Google Workspaceの場合の設定例

```
example.com.  3600  IN  MX  1   ASPMX.L.GOOGLE.COM.
example.com.  3600  IN  MX  5   ALT1.ASPMX.L.GOOGLE.COM.
example.com.  3600  IN  MX  5   ALT2.ASPMX.L.GOOGLE.COM.
example.com.  3600  IN  MX  10  ALT3.ASPMX.L.GOOGLE.COM.
example.com.  3600  IN  MX  10  ALT4.ASPMX.L.GOOGLE.COM.
```

- MXレコードの値はIPアドレスではなくドメイン名を指定する
- 指定したドメイン名にはAレコードが設定されている必要がある

- 参照: [RFC 5321 — Simple Mail Transfer Protocol](https://datatracker.ietf.org/doc/html/rfc5321)

## TXTレコード

- ドメインに任意のテキスト情報を紐づけるレコード
- 元々は自由なメモ用だったが、現在はセキュリティや認証で多用される

### 主な用途

| 用途 | 値の例 |
|---|---|
| SPF（メール送信元認証） | `v=spf1 include:_spf.google.com ~all` |
| DKIM（メール署名検証） | `v=DKIM1; k=rsa; p=MIGfMA0GCSq...`（`selector._domainkey`に設定） |
| DMARC（メール認証ポリシー） | `v=DMARC1; p=reject; rua=mailto:...`（`_dmarc`に設定） |
| ドメイン所有権の確認 | `google-site-verification=abc123...` |
| サービス連携 | `MS=ms12345678`（Microsoft 365の確認） |

```
# SPFレコード
example.com.  3600  IN  TXT  "v=spf1 include:_spf.google.com ~all"

# Google Search Consoleのドメイン確認
example.com.  3600  IN  TXT  "google-site-verification=abcdefg1234567"

# DKIMレコード（セレクタ._domainkey サブドメインに設定）
selector1._domainkey.example.com.  3600  IN  TXT  "v=DKIM1; k=rsa; p=MIGfMA0..."

# DMARCレコード（_dmarc サブドメインに設定）
_dmarc.example.com.  3600  IN  TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"
```

- 1つのドメインに複数のTXTレコードを設定可能
- ただしSPFレコードはドメインにつき1つのみ（複数あるとエラー）
- 文字列は255文字が上限だが、複数の文字列を連結して長い値を設定可能

- 参照: [RFC 7208 — SPF](https://datatracker.ietf.org/doc/html/rfc7208)

## NSレコード

- ドメインの権威DNSサーバー（ネームサーバー）を指定するレコード
- ドメインのDNS情報がどのサーバーに問い合わせれば得られるかを示す

```
example.com.  86400  IN  NS  ns1.example.com.
example.com.  86400  IN  NS  ns2.example.com.
```

### NSレコードの役割

```
TLD DNSサーバー（.com）
  │
  │ 「example.com のDNS情報はどこにある？」
  │
  └──→ NSレコード: ns1.example.com, ns2.example.com
         │
         └──→ 権威DNSサーバーにAレコードなどを問い合わせ
```

- 通常2つ以上のNSレコードを設定する（冗長性の確保）
- NSレコードの変更はドメイン移管やDNSサーバーの変更時に行う
- レジストラ側の設定とゾーンファイル内のNSレコードの両方を揃える必要がある

### サブドメインの委任

- NSレコードを使って、サブドメインのDNS管理を別のサーバーに委任できる

```
# sub.example.com のDNSを別のサーバーに委任
sub.example.com.  86400  IN  NS  ns1.sub-dns-provider.com.
sub.example.com.  86400  IN  NS  ns2.sub-dns-provider.com.
```

- 部署ごと・サービスごとにDNS管理を分離したい場合に使用

## SOAレコード

- ゾーン（ドメイン）の管理情報を定義するレコード
- 各ゾーンに必ず1つ存在する

```
example.com.  86400  IN  SOA  ns1.example.com. admin.example.com. (
    2024010101  ; シリアル番号（ゾーンの更新を識別）
    3600        ; リフレッシュ間隔（セカンダリNSが更新確認する間隔）
    900         ; リトライ間隔（リフレッシュ失敗時の再試行間隔）
    604800      ; 有効期限（セカンダリNSがデータを保持する期間）
    86400       ; ネガティブキャッシュTTL（「レコードなし」の応答のキャッシュ時間）
)
```

| フィールド | 値の例 | 説明 |
|---|---|---|
| MNAME | `ns1.example.com.` | プライマリネームサーバー |
| RNAME | `admin.example.com.` | 管理者のメールアドレス（`@`を`.`に置換） |
| Serial | `2024010101` | ゾーンのバージョン番号。更新のたびに増加させる |
| Refresh | `3600` | セカンダリNSがプライマリに更新を問い合わせる間隔（秒） |
| Retry | `900` | Refresh失敗時の再試行間隔（秒） |
| Expire | `604800` | セカンダリNSがプライマリに到達できない場合にデータを保持する期間 |
| Minimum TTL | `86400` | NXDOMAINなどネガティブ応答のキャッシュ時間 |

- SOAレコードは通常DNS管理画面で自動管理される
- シリアル番号は`YYYYMMDDNN`形式（日付＋連番）が一般的

- 参照: [RFC 1035 — 3.3.13 SOA RDATA format](https://datatracker.ietf.org/doc/html/rfc1035#section-3.3.13)

## PTRレコード

- IPアドレスからドメイン名への逆引き（リバースDNS）に使用
- メールサーバーのなりすまし防止で重要

```
# IPv4の逆引き（in-addr.arpa ゾーン）
34.216.184.93.in-addr.arpa.  3600  IN  PTR  example.com.

# IPv6の逆引き（ip6.arpa ゾーン）
6.4.9.1...0.6.0.2.ip6.arpa.  3600  IN  PTR  example.com.
```

- IPアドレスを逆順にして`.in-addr.arpa`を付加したドメインに設定
- PTRレコードはIPアドレスの管理者（ISP、ホスティング事業者）が設定する
- 自分では設定できない場合が多く、ホスティング事業者に依頼が必要

### PTRレコードが重要な場面

- メール送信：受信サーバーがPTRレコードでIPアドレスの逆引きを確認する
- PTRレコードがないとスパム判定される可能性が高い

## SRVレコード

- 特定のサービスに対して、提供するサーバーのホスト名とポート番号を指定

```
_service._protocol.name.  TTL  IN  SRV  priority  weight  port  target

# SIPサービスの例
_sip._tcp.example.com.  3600  IN  SRV  10  60  5060  sipserver1.example.com.
_sip._tcp.example.com.  3600  IN  SRV  10  40  5060  sipserver2.example.com.

# Microsoft 365の例
_sipfederationtls._tcp.example.com.  3600  IN  SRV  100  1  5061  sipfed.online.lync.com.
```

| フィールド | 説明 |
|---|---|
| Priority | 優先度（小さいほど優先） |
| Weight | 同じ優先度内での重み（トラフィック分散） |
| Port | サービスのポート番号 |
| Target | サービスを提供するサーバーのホスト名 |

## CAAレコード

- ドメインのSSL証明書を発行できる認証局（CA）を制限するレコード
- 不正な証明書発行の防止に有効

```
# Let's Encryptのみに証明書発行を許可
example.com.  3600  IN  CAA  0  issue  "letsencrypt.org"

# ワイルドカード証明書の発行を制限
example.com.  3600  IN  CAA  0  issuewild  "letsencrypt.org"

# 不正発行時の通知先
example.com.  3600  IN  CAA  0  iodef  "mailto:security@example.com"
```

| タグ | 説明 |
|---|---|
| `issue` | 通常の証明書を発行できるCAを指定 |
| `issuewild` | ワイルドカード証明書を発行できるCAを指定 |
| `iodef` | ポリシー違反時の報告先 |

- CAAレコードが設定されていない場合、どのCAでも証明書を発行可能
- セキュリティ向上のため、CAAレコードの設定が推奨されている

- 参照: [RFC 8659 — DNS Certification Authority Authorization (CAA) Resource Record](https://datatracker.ietf.org/doc/html/rfc8659)

## 実務でよくある設定パターン

### Webサイト + メール（Google Workspace）の基本構成

```
# ルートドメインのAレコード
example.com.       3600  IN  A      93.184.216.34

# wwwのCNAME
www.example.com.   3600  IN  CNAME  example.com.

# メール（Google Workspace）
example.com.       3600  IN  MX  1   ASPMX.L.GOOGLE.COM.
example.com.       3600  IN  MX  5   ALT1.ASPMX.L.GOOGLE.COM.
example.com.       3600  IN  MX  5   ALT2.ASPMX.L.GOOGLE.COM.

# SPF
example.com.       3600  IN  TXT  "v=spf1 include:_spf.google.com ~all"

# DMARC
_dmarc.example.com.  3600  IN  TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"

# ネームサーバー
example.com.       86400  IN  NS  ns1.dns-provider.com.
example.com.       86400  IN  NS  ns2.dns-provider.com.

# CAA（Let's Encrypt限定）
example.com.       3600  IN  CAA  0  issue  "letsencrypt.org"
```

### Vercel / Netlifyなどのホスティングサービスを使う場合

```
# ルートドメイン（Aレコード）
example.com.       3600  IN  A  76.76.21.21

# wwwサブドメイン（CNAME）
www.example.com.   3600  IN  CNAME  cname.vercel-dns.com.
```

- ホスティングサービスが指定するAレコードまたはCNAMEを設定
- ルートドメインはCNAMEが使えないため、Aレコードで設定する

## レコードの確認方法

```bash
# すべてのレコードを確認
dig example.com ANY

# 特定のレコードタイプを確認
dig example.com A
dig example.com AAAA
dig example.com CNAME
dig example.com MX
dig example.com TXT
dig example.com NS
dig example.com SOA
dig example.com CAA

# ショート出力（値だけ表示）
dig +short example.com A
dig +short example.com MX
```

## まとめ

- Aレコード（IPv4）とAAAAレコード（IPv6）がドメインの名前解決の基本
- CNAMEはサブドメインのエイリアスに使用。ルートドメインには設定不可
- MXレコードでメール配送先を指定。優先度の数値が小さいほど優先
- TXTレコードはSPF・DKIM・DMARC・ドメイン所有権確認など多目的に使用
- NSレコードはドメインの権威DNSサーバーを指定。ドメイン移管時に変更
- CAAレコードでSSL証明書を発行できるCAを制限し、セキュリティを強化
- 実務ではAレコード + CNAME + MX + TXT（SPF/DKIM/DMARC）の組み合わせが頻出
