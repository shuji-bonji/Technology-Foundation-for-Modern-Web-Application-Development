# DNS プロトコル

## DNSプロトコルスタック操作
```mermaid
flowchart TB
    subgraph Application Layer
        DNS1[DNS]
        DNS2[DNS]
        DoH[DoH]
        DoT[DoT]
        TLS1[[TLS]]
        TLS2[[TLS]]
    end

    subgraph Transport Layer
        UDP53[UDP 53]
        TCP53[TCP 53]
        TCP443[TCP 443]
        TCP853[TCP 853]
    end

    subgraph Network Layer
        IP[IP]
    end

    subgraph Data Link Layer
        Ethernet[Ethernet]
    end

    subgraph Physical Layer
        PHY[PHY]
    end

    DNS1 ---> UDP53
    DNS2 ---> TCP53
    DoH --> TLS1 --> TCP443
    DoT --> TLS2 --> TCP853

    UDP53 --> IP
    TCP53 --> IP
    TCP443 --> IP
    TCP853 --> IP

    IP --> Ethernet --> PHY
```

- DNSはネームサーバーの階層構造に基づいています。
- DNSはホスト名をIPアドレスに解決します。
- 逆DNSはIPアドレスをホスト名に解決します。
- ルーティングにはIPアドレスが必要です。
- UDP 53 - 512バイト未満のほとんどのDNSクエリで高速です。
- TCP 53 - DNSゾーン転送/IPv6 AAAAレコード
- DNSはUDPペイロードを512バイトに制限します。
- 切り捨てエラーが発生した場合は、TCP 53経由で再送信します。
- DoHはDNSクエリをHTTPSリクエストにカプセル化します。
- DoTはDNSクエリをTCPにカプセル化します。



## DNSルックアッププロセス



### DNSルックアップ構成図

以下は、DNSルックアップの実際の問い合わせフローに基づいた構成図です。各構成要素がどの順序でどのようにやり取りを行うかを矢印で示しています。

```mermaid
flowchart LR
    Client[Client<br>（ブラウザ・OS・ルーターキャッシュ確認）]
    Resolver[DNS Resolver<br>（ISP）]
    Root[Root Server]
    TLD["TLD Server (.com)"]
    Auth[Authoritative Server<br>（webapp.com）]

    Client -->|1\. DNSクエリ送信| Resolver
    Resolver -->|2\. キャッシュなければ問合せ| Root
    Root -->|3\. TLDサーバのアドレス返却| Resolver
    Resolver -->|4\. TLDに問合せ| TLD
    TLD -->|5\. 権威サーバのアドレス返却| Resolver
    Resolver -->|6\. 権威サーバに問合せ| Auth
    Auth -->|7\. Aレコード（IP）返却| Resolver
    Resolver -->|8\. クライアントへ返却（キャッシュ保存）| Client
```


### DNSルックアップシーケンス図
```mermaid
sequenceDiagram
    participant Client
    participant Resolver as DNS Resolver (ISP)
    participant Root as Root Server
    participant TLD as TLD Server (.com)
    participant Auth as Authoritative Server

    Note over Client: Step 1: クライアントはまず<br>ブラウザ → OS → ルータのキャッシュを確認
    Client->>Resolver: www.mywebapp.com のIPは？
    
    Note over Resolver: Step 2: DNSキャッシュになければ<br>ルートサーバへ問い合わせ
    Resolver->>Root: www.mywebapp.com?
    
    Note over Root: Step 3: TLD (.com) のアドレスを返す
    Root-->>Resolver: TLDサーバのアドレス

    Note over Resolver: Step 4: TLDに問い合わせ
    Resolver->>TLD: www.mywebapp.com?

    TLD-->>Resolver: 権威サーバのアドレス

    Note over Resolver: Step 5: 権威サーバに最終問い合わせ
    Resolver->>Auth: www.mywebapp.com?

    Auth-->>Resolver: Aレコード（例：200.200.1.1）

    Resolver-->>Client: IPアドレスを返す（キャッシュにも保存）
```

1. クライアントはブラウザキャッシュ、OSキャッシュ、ルーターキャッシュの順に確認し、ISPのDNSリゾルバにクエリを送信します。
2. ISPのDNSリゾルバはDNSキャッシュ参照後、ルートサーバにクエリを送信します。
3. ルートサーバは(.com)|のトップレベルドメインサーバのアドレスを返します。
4. TLDサーバはwebapp.comの権威サーバのアドレスを返します。
5. 権威サーバはwebapp.comのIPアドレスを解決し、次のサーバに送信します。

## DNS レコードタイプ

### Aレコード
ホスト名をIPv4アドレスに解決

#### 例
|hostname|record type|value|TTL|
|---|---|---|---|
|webapp.com|A|200.200.1.1|3200|

### CNAMEレコード
エイリアスまたはサブドメインをAレコードに解決

#### 例
|hostname|record type|value|TTL|
|---|---|---|---|
|blog.webapp.com|CNAME|webapp.com|3200|

### MXレコード
ルーティング用のメールサーバーのホスト名を解決

#### 例
|hostname|record type|value|TTL|
|---|---|---|---|
|webapp.com|MX|mail.server.com|3200|
### NSレコード
ルーティング用のメールサーバーのホスト名を解決

#### 例
|hostname|record type|value|TTL|
|---|---|---|---|
|webapp.com|NS|ns1.server.com|3200|

### AAAAレコード
ホスト名をIPv6アドレスに解決

#### 例
|hostname|record type|value|TTL|
|---|---|---|---|
|webapp.com|AAAA|2001:db8:3c4d: 1::1|3200|



*TTL = キャッシュをフラッシュするまでの時間間隔


## 補足事項

### DNSキャッシュとTTLの重要性

DNSは名前解決のたびに毎回問い合わせを行うとパフォーマンスが低下するため、ブラウザやOS、ルーター、ISPのDNSリゾルバなど多層にわたるキャッシュ機構を備えています。このキャッシュの保持時間はDNSレコードのTTL（Time To Live）に依存します。TTLが短すぎると頻繁な再問合せでパフォーマンスが落ち、逆に長すぎるとIP変更が即時反映されないといった影響があります。

### DoH（DNS over HTTPS）とDoT（DNS over TLS）の違い

- **DoH (DNS over HTTPS)** は、通常のHTTPS通信（TCP 443）を利用し、DNSトラフィックをHTTPリクエストにカプセル化します。Webブラウザとの親和性が高く、ファイアウォールやプロキシを回避しやすい特徴があります。
- **DoT (DNS over TLS)** は、専用ポートTCP 853でTLS暗号化通信を行うため、明示的にDNSトラフィックを保護する意図が明確です。専用アプリケーションやネットワーク機器での利用に適しています。

### 名前解決失敗時の挙動

DNSリゾルバがいずれのサーバからも応答を得られない場合、名前解決に失敗し、ブラウザでは「DNS_PROBE_FINISHED_NXDOMAIN」や「サーバーが見つかりませんでした」といったエラーが表示されます。

### セキュリティ観点での留意点

- DNSキャッシュポイズニングや中間者攻撃を防ぐため、信頼できるDNSサーバやDoH/DoTの活用が推奨されます。
- DNSSEC（DNS Security Extensions）を導入することで、DNS応答の整合性検証が可能になります（ただし対応していないサーバもあります）。

## Basic認証のシーケンス図

以下は、HTTP Basic認証における典型的な通信の流れを示すシーケンス図です。クライアントが保護されたリソースにアクセスしようとすると、サーバは `401 Unauthorized` 応答とともに認証を要求します。クライアントは `Authorization` ヘッダーにユーザー名とパスワードを Base64 でエンコードした文字列を付加して再リクエストを送り、認証に成功すればサーバは対象リソースを返します。

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: GET /protected-resource
    Server-->>Client: 401 Unauthorized<br>WWW-Authenticate: Basic realm="example"
    Client->>Server: GET /protected-resource<br>Authorization: Basic dXNlcjpwYXNz
    Server-->>Client: 200 OK<br>(リソースを返却)
```

- `Authorization: Basic dXNlcjpwYXNz` は、`user:pass` を Base64 エンコードした文字列です（この形式は容易に復号可能なため注意が必要です）。
- Basic認証は**暗号化されていないため、通信経路の盗聴を防ぐためにも必ず HTTPS と併用する必要があります**。
- 多くのAPIではセキュリティ上の理由から Basic認証ではなく BearerトークンやOAuth2が推奨されます。

#### Base64エンコード例

```
元の文字列: user:password
↓
Base64エンコード: dXNlcjpwYXNzd29yZA==
```