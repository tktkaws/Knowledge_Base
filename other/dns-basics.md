# DNSの基礎 — ドメイン名解決の仕組みをゼロから理解する

## DNSとは

- DNS（Domain Name System）は、ドメイン名をIPアドレスに変換する仕組み
- インターネットの「電話帳」に例えられる
- 人間が覚えやすいドメイン名（例: `example.com`）を、コンピュータが通信に使うIPアドレス（例: `93.184.216.34`）に変換する
- DNSがなければ、WebサイトにアクセスするたびにIPアドレスを直接入力する必要がある

```
ブラウザ: 「example.com にアクセスしたい」
DNS:     「example.com のIPアドレスは 93.184.216.34 です」
ブラウザ: 「93.184.216.34 に接続します」
```

- 参照: [RFC 1034 — Domain Names - Concepts and Facilities](https://datatracker.ietf.org/doc/html/rfc1034)

## ドメイン名の構造

- ドメイン名は右から左に読む階層構造
- 各階層はドット（`.`）で区切られる

```
www.example.com.
 │    │      │  │
 │    │      │  └─ ルートドメイン（通常省略される「.」）
 │    │      └──── トップレベルドメイン（TLD）: .com, .jp, .org など
 │    └─────────── セカンドレベルドメイン（SLD）: example
 └──────────────── サブドメイン: www
```

### ドメイン階層の種類

| 階層 | 例 | 説明 |
|---|---|---|
| ルートドメイン | `.`（ドット） | DNS階層の最上位。通常は省略される |
| TLD（トップレベルドメイン） | `.com`, `.jp`, `.org` | ICANN（国際機関）が管理 |
| gTLD（汎用TLD） | `.com`, `.net`, `.info` | 用途や国に限定されないTLD |
| ccTLD（国別TLD） | `.jp`, `.uk`, `.de` | 国・地域に割り当てられたTLD |
| SLD（セカンドレベルドメイン） | `example` | ドメイン登録者が取得する部分 |
| サブドメイン | `www`, `mail`, `blog` | ドメイン所有者が自由に設定できる |

- `co.jp`のように、TLDの下にさらに区分がある場合もある（属性型JP）
- FQDN（完全修飾ドメイン名）は末尾にルートドメインの`.`を含む表記（例: `www.example.com.`）

- 参照: [ICANN — ドメイン名の仕組み](https://www.icann.org/resources/pages/what-2012-02-25-ja)

## DNSの名前解決の流れ

- ブラウザにURLを入力してからWebページが表示されるまでに、DNSによる名前解決が行われる
- 名前解決には複数のDNSサーバーが関与する

### 登場するDNSサーバー

| サーバー | 役割 | 例 |
|---|---|---|
| DNSリゾルバ（フルリゾルバ） | クライアントの代わりに名前解決を行う | ISPのDNSサーバー、Google Public DNS（`8.8.8.8`） |
| ルートDNSサーバー | TLDのDNSサーバーの場所を教える | 世界に13系統（a〜m.root-servers.net） |
| TLD DNSサーバー | 各TLD配下の権威DNSサーバーの場所を教える | `.com`のDNSサーバー、`.jp`のDNSサーバー |
| 権威DNSサーバー | ドメインの実際のDNSレコード（IPアドレスなど）を保持・応答する | ドメイン管理者が設定するDNSサーバー |

### 名前解決のステップ

```
ユーザー                DNSリゾルバ         ルートDNS      TLD DNS       権威DNS
  │                       │                  │              │             │
  │── example.com は？ ──→│                  │              │             │
  │                       │── .com は？ ────→│              │             │
  │                       │←─ .comのTLD DNS ─│              │             │
  │                       │                  │              │             │
  │                       │── example.com のNSは？ ────────→│             │
  │                       │←─ example.com の権威DNS ────────│             │
  │                       │                  │              │             │
  │                       │── example.com のAレコードは？ ──────────────→│
  │                       │←─ 93.184.216.34 ────────────────────────────│
  │                       │                  │              │             │
  │←── 93.184.216.34 ────│                  │              │             │
```

1. ユーザーがブラウザに`example.com`を入力
2. ブラウザはOSに名前解決を依頼
3. OSはDNSリゾルバ（通常ISPのDNSサーバー）に問い合わせ
4. リゾルバはルートDNSサーバーに「`.com`のTLD DNSはどこか」を問い合わせ
5. ルートDNSがTLD DNSサーバーのアドレスを返す
6. リゾルバはTLD DNSサーバーに「`example.com`の権威DNSはどこか」を問い合わせ
7. TLD DNSが権威DNSサーバーのアドレスを返す
8. リゾルバは権威DNSサーバーに「`example.com`のIPアドレスは何か」を問い合わせ
9. 権威DNSがIPアドレスを返す
10. リゾルバがIPアドレスをOSに返し、ブラウザがそのIPに接続

- この再帰的な問い合わせプロセスを**再帰クエリ**（recursive query）と呼ぶ
- リゾルバが各DNSサーバーに順に問い合わせることを**反復クエリ**（iterative query）と呼ぶ

- 参照: [Cloudflare — DNSとは](https://www.cloudflare.com/ja-jp/learning/dns/what-is-dns/)

## DNSキャッシュ

- 毎回ルートDNSから問い合わせると時間がかかるため、DNSの応答はキャッシュされる
- キャッシュにより2回目以降の名前解決は高速化される

### キャッシュが保持される場所

| 場所 | 説明 |
|---|---|
| ブラウザキャッシュ | ブラウザが直近のDNS応答を保持 |
| OSキャッシュ | OSがDNS応答をメモリに保持 |
| DNSリゾルバのキャッシュ | ISPやパブリックDNSがキャッシュを保持 |

### TTL（Time To Live）

- DNSレコードにはTTL（生存時間）が設定されている
- TTLの秒数が経過すると、キャッシュは破棄され再度問い合わせが行われる

```
example.com.  300  IN  A  93.184.216.34
              ^^^
              TTL = 300秒（5分）
```

| TTL値 | 意味 | 用途 |
|---|---|---|
| 300（5分） | 短いTTL | DNS切り替え予定がある場合。変更が素早く反映される |
| 3600（1時間） | 中程度 | 一般的な設定 |
| 86400（24時間） | 長いTTL | 変更頻度が低いレコード。DNSサーバーへの負荷を軽減 |

- TTLが長いほどキャッシュが効くが、DNS変更時の反映が遅くなる
- DNS切り替え前にTTLを短く設定しておくのがベストプラクティス

- 参照: [RFC 1035 — Domain Names - Implementation and Specification](https://datatracker.ietf.org/doc/html/rfc1035)

## パブリックDNS

- ISP以外が提供するDNSリゾルバ
- 速度やプライバシーの改善、フィルタリング機能を提供するものもある

| サービス | IPアドレス | 特徴 |
|---|---|---|
| Google Public DNS | `8.8.8.8` / `8.8.4.4` | 高速・高可用性。最も広く使われている |
| Cloudflare DNS | `1.1.1.1` / `1.0.0.1` | プライバシー重視。ログを24時間で削除 |
| OpenDNS | `208.67.222.222` / `208.67.220.220` | フィルタリング機能あり。家庭向け |
| Quad9 | `9.9.9.9` | マルウェアドメインのブロック機能あり |

- ISPのDNSが遅い・不安定な場合にパブリックDNSへの切り替えが有効
- 設定はOSのネットワーク設定またはルーターで変更可能

## DNSの確認・デバッグ

### よく使うコマンド

```bash
# ドメインのIPアドレスを調べる
nslookup example.com

# 詳細なDNS情報を調べる（推奨）
dig example.com

# Aレコードを指定して調べる
dig example.com A

# 特定のDNSサーバーを指定して調べる
dig @8.8.8.8 example.com

# 名前解決の全過程を表示する（トレース）
dig +trace example.com

# MXレコードを調べる
dig example.com MX
```

### digコマンドの出力の読み方

```bash
$ dig example.com

;; ANSWER SECTION:
example.com.        300     IN      A       93.184.216.34
│                   │       │       │       │
│                   │       │       │       └─ IPアドレス（レコードの値）
│                   │       │       └───────── レコードタイプ（A = IPv4）
│                   │       └───────────────── クラス（IN = Internet）
│                   └───────────────────────── TTL（秒）
└───────────────────────────────────────────── ドメイン名

;; Query time: 12 msec        ← 応答にかかった時間
;; SERVER: 8.8.8.8#53         ← 応答したDNSサーバー
;; MSG SIZE  rcvd: 56         ← 応答メッセージのサイズ
```

### Windowsでの確認

```bash
# Windowsではnslookupが標準
nslookup example.com

# DNSキャッシュの確認
ipconfig /displaydns

# DNSキャッシュのクリア
ipconfig /flushdns
```

### macOS / Linuxでのキャッシュクリア

```bash
# macOS
sudo dscacheutil -flushcache && sudo killall -HUP mDNSResponder

# Linux（systemd-resolved）
sudo systemd-resolve --flush-caches
```

- 参照: [dig manページ](https://linux.die.net/man/1/dig)

## DNSに関連するセキュリティ

### DNSキャッシュポイズニング

- DNSリゾルバのキャッシュに偽の情報を注入する攻撃
- ユーザーが正しいドメインにアクセスしても、攻撃者のサーバーに誘導される

```
正常: example.com → 93.184.216.34（正規サーバー）
攻撃: example.com → 203.0.113.99（攻撃者のサーバー）
```

### DNSSEC

- DNSの応答にデジタル署名を付与し、改ざんを検知する仕組み
- 権威DNSサーバーがレコードに署名し、リゾルバが検証する
- すべてのドメインが対応しているわけではない

### DNS over HTTPS（DoH）/ DNS over TLS（DoT）

| プロトコル | ポート | 説明 |
|---|---|---|
| 通常のDNS | 53（UDP/TCP） | 平文。ISPや第三者が通信内容を傍受可能 |
| DNS over TLS（DoT） | 853（TCP） | TLSで暗号化。専用ポートを使用 |
| DNS over HTTPS（DoH） | 443（TCP） | HTTPSで暗号化。通常のHTTPS通信と見分けがつかない |

- DoH/DoTにより、どのドメインにアクセスしたかを第三者が知ることが困難になる
- 主要ブラウザ（Chrome、Firefox）はDoHに対応済み

- 参照: [RFC 8484 — DNS Queries over HTTPS (DoH)](https://datatracker.ietf.org/doc/html/rfc8484)

## hostsファイル

- OSにはDNSより優先される名前解決の仕組みとしてhostsファイルがある
- ローカル開発やテスト時にドメインを特定のIPに向けるために使用

```bash
# hostsファイルの場所
# macOS / Linux: /etc/hosts
# Windows: C:\Windows\System32\drivers\etc\hosts

# 記述例
127.0.0.1   mysite.local
192.168.1.10  staging.example.com
```

- hostsファイルの設定はDNSよりも優先される
- 開発環境でのテスト、一時的なDNS変更の確認に便利

## まとめ

- DNSはドメイン名をIPアドレスに変換するインターネットの基盤技術
- ドメイン名はルートドメインを頂点とする階層構造
- 名前解決はDNSリゾルバがルートDNS→TLD DNS→権威DNSの順に問い合わせて行う
- TTLによりDNS応答がキャッシュされ、2回目以降の名前解決が高速化される
- `dig`コマンドはDNSのデバッグに最も有用なツール
- DNSSEC、DoH/DoTなどのセキュリティ技術により、DNSの安全性が向上している
