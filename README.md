# Fastly Load Balancing Tutorial

Fastly のロードバランス機能 (LB) を体験するためのチュートリアルです。チュートリアル1では、Fastly LB をマイクロサービス・ルーターとして、チュートリアル2ではマルチクラウド・ロードバランサーとして実装します。

## Prerequisite

このチュートリアルを実行するには、以下が必要です。

* Fastly 開発者用アカウント
* バックエンドとなる HTTP サーバー 3つ (マイクロサービス A, 同バックアップ, マイクロサービス B)
* Google Cloud Platform アカウント

## Suggested Reading

以下の資料の併読をお薦めします。

Fastly documentation:

* [API documentation](https://docs.fastly.com/api/)
* [Dynamic servers documentation](https://docs.fastly.com/guides/dynamic-servers/)
* [Working with services](https://docs.fastly.com/api/config#service)
* [Working with service versions](https://docs.fastly.com/api/config#version)
* [Conditions documentation](https://docs.fastly.com/guides/conditions/)
* [Creating health checks via the API](https://docs.fastly.com/api/config#healthcheck)

## チュートリアル1 (マイクロサービス・ルーター)

### ねらい

* Fastly サービスの作成・変更の手順に慣れる
* マイクロサービス・ルーター機能が実装できる

### 必要となる設定値

* ドメイン名: `lab1010-000.global.ssl.fastly.net`
* マイクロサービス A URL: `http://<service_a_ip>/`
* マイクロサービス B URL: `https://<name.appspot.com>/`

### Step 1: 新サービスの作成

Fastly サービスを新たに作成します。これは、チュートリアル3までの全チュートリアルで用います。

1. Fastly コントロールパネル https://manage.fastly.com へログイン
2. ログイン後、画面の上部左から **Configure** タブをクリック
3. 同画面の上部右の **Create Service** ボタンをクリック
4. **Create a new service** 画面にて、**Name** フィールドにサービス名 (任意の文字列) を記入

### Step 2: 新サービスの作成 (つづき)

1. **Create a new service** 画面にて、**Domain** フィールドにドメイン名を記入します。ここで、ドメイン名は `lab1010-000.global.ssl.fastly.net` の形式とします
2. 同画面にて、**Address** フィールドにマイクロサービス A の IP アドレス `<service_a_ip>` を記入
3. 同画面にて、**Enable TLS?** に対して `No, Do not enable TLS.` を選択
4. 同画面の下部から、**Create** ボタンをクリックすると、新サービスの作成と有効化が行われます
5. **確認** `https://lab1010-000.global.ssl.fastly.net` へアクセスすると、マイクロサービス A が Fastly 経由で配信されています

### Step 3: コンテンツをキャッシュしない設定にサービスを変更

1. Step 2で作成したサービスの Configure 画面にて、画面上部右の **Configuration** ボタンから **Clone Active** を選択すると、新バージョンの設定が作成され変更可能となる
2. 画面左のメニューから **Setting** をクリック
3. 表示された画面で Fallback TTL とある下の、**Fallback TTL (sec)** フィールドに `0` を入力し、Save ボタンをクリック
4. 同画面の右上にある **Activate** ボタンをクリックすると、新しい設定の有効化が行われます
5. 表示された画面の左側にある **Purge** ボタンから **Purge all** を選択し、表示される **Purge all** ウィンドウで **Purge** ボタンをクリック (**OK** ボタンをクリックしてウィンドウを閉じてください)
6. キャッシュ済みのコンテンツがパージされ、新しい設定が完全に有効となります

### Step 4: マイクロサービス B をオリジンとしてサービスへ追加

1. 画面上部右の **Configuration** ボタンから **Clone Active** を選択すると、新バージョンの設定が作成されます
2. 画面左のメニューから **Origins** をクリック
3. 表示された画面で Hosts とある下の、**Create host** ボタンをクリック
4. **Create a host** 画面にて、**Name** フィールドに設定名 (任意の文字列) を記入
5. 同画面にて、**Address** フィールドにマイクロサービス B のドメイン `<name.appspot.com>` を記入
6. 同画面にて、**Enable TLS?** に対して `No, Do not enable TLS.` を選択
7. 同画面の下部から、**Create** ボタンをクリックすると、オリジンとなるホストが新たに作成されます

### Step 5: /product/ をマイクロサービス B にルーティング

1. Step 4で追加した Hosts の **Attache a condition** をクリック
2. **Create a new request condition** 画面にて、**Name** フィールドに条件の名前 (任意の文字列) を記入
3. 同画面にて、**Apply if...** フィールドに `req.url ~ "^/product/"` と記入
4. 同画面の下部から、**Send and Apply to (ホスト名)** ボタンをクリックすると、ホストに対して条件が設定されます

### Step 6: Override host の設定

1. 画面左のメニューから **Settings** をクリック
2. **Override host** を On にして、**Override host header** フィールドに `<name.appspot.com>` を記入

### Step 7: オリジンに対するリクエストの URL パス部を変更 (/product/* → /*)

1. 画面左のメニューから **Content** をクリック
2. 表示された画面で Headers とある下の、**Create your first header** ボタンをクリック
3. **Create a header** 画面にて、**Name** フィールドに設定名 (任意の文字列) を記入
4. 同画面にて、**Type / Action** フィールドに `Request` と `Regex` をそれぞれ選択
5. **Destination** フィールドに `url` と記入
6. **Source** フィールドに `req.url` と記入
7. **Regex** フィールドに `^/product` と記入し、Substitution** フィールドは空欄のママ
8. 画面の下部から、**Create** ボタンをクリックすると、リクエストされる URL の書き換えが設定されます
9. 追加した Headers 設定の **Attache a condition** をクリック
10. **Add a condition to rewrite** 画面にて、ドロップダウンより作成済みの条件である `req.url ~ "^/product/"` を選択

### Step 8: マイクロサービス・ルーター機能の有効化と動作確認

1. 画面の上部から、**Activate** ボタンをクリックすると、変更したサービス設定の有効化が行われます
2. **確認** `https://lab1010-000.global.ssl.fastly.net/product/` へアクセスすると、マイクロサービス B が Fastly 経由で配信されています



## チュートリアル2 (Dynamic Serversによるマルチクラウド・ロードバランサー )

### ねらい

* 複数オリジン間のロードバランス機能を API から実装できる

### 必要となる設定値

* <api_key>
* <service_id>
* <service_a_ip>
* <service_a_backup_ip>

### Step 1: 環境設定

```
API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
SERVICE_ID=xxxxxxxxxxxxxxxxxxxxxx
VERSION=3
```

### Step 2: API を用いたサービス内容確認

```
curl sv -H "Fastly-Key: ${API_KEY}" \
https://api.fastly.com/service/${SERVICE_ID}/details | jq
```

### Step 3: (API を用いた) Clone による新バージョンの作成

```
curl -sv -H "Fastly-Key: ${API_KEY}" -X PUT \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/clone \
| jq
```

### Step 4: マイクロサービス A 用オリジンの設定から削除 (UI経由)

1. マイクロサービス A のオリジン <service_a_ip> を、**Origins** の **Host** から消去します。


### Step 5: Custom VCL のアップロード

```
VERSION=4

curl -vs -H "Fastly-Key: ${API_KEY}" -X POST -H \
"Content-Type: application/x-www-form-urlencoded" \
--data "name=main&main=true" \
--data-urlencode "content@main.vcl" \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/vcl | jq
```


### Step 6: サーバープールの作成

```
curl -sv -H "Fastly-Key: ${API_KEY}" -X POST \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/pool -d \
'name=cloudpool&comment=cloudpool' | jq

POOL_ID_1=<id>
```

### Step 7: サーバープールへサーバーの追加

```
curl -vs -H "Fastly-Key: ${API_KEY}" -X POST \
https://api.fastly.com/service/${SERVICE_ID}/pool/${POOL_ID_1}/server -d \
'address=<service_a_ip>' | jq

SERVER_ID_1=<id>
```

### Step 8: サービスバージョンの有効化

```
curl -vs -H "Fastly-Key: ${API_KEY}" -X PUT \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/activate | jq
```

### Step 9: サーバープールへサーバーの追加

```
curl -vs -H "Fastly-Key: ${API_KEY}" -X POST \
https://api.fastly.com/service/${SERVICE_ID}/pool/${POOL_ID_1}/server -d \
'address=<service_a_backup_ip>' | jq

SERVER_ID_2=<id>
```

### Step 10: サーバープール内のサーバーの weight を変更

```
curl -vs -H "Fastly-Key: ${API_KEY}" -X PUT \
https://api.fastly.com/service/${SERVICE_ID}/pool/${POOL_ID_1}/server/${SERVER_ID_1} \
-d 'weight=10' | jq
```

### Step 11: (後片付け) サーバープールの削除

```
curl -sv -H "Fastly-Key: ${API_KEY}" -X PUT \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/clone \
| jq

SERVER=6

curl -sv -H "Fastly-Key: ${API_KEY}" -X DELETE \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/pool/cloudpool \
| jq
```


## チュートリアル3 (Directorによるマルチクラウド・ロードバランサー )

### ねらい

* 複数オリジン間のロードバランス機能が実装できる
* ヘルスチェックとフェイルオーバーが実装できる

### 必要となる設定値

* マイクロサービス A (バックアップ) URL: `http://<Service_A_Backup_IP>/`
* マイクロサービス A の VCL 内での backend 参照名: `F_addr_123_45_67_89`
* マイクロサービス A (バックアップ) の VCL 内での backend 参照名: `F_Service_A_Backup`

### Step 0: マイクロサービス A をオリジンとしてサービスへ追加

チュートリアル2 Step 4 にて削除したオリジン <service_a_ip> を、再度サービスのオリジンとして追加してください。

### Step 1: マイクロサービス A のバックアップをオリジンとしてサービスへ追加

1. チュートリアル1で作成したサービスの Configure 画面にて、画面上部右の **Configuration** ボタンから **Clone Active** を選択すると、新バージョンの設定が作成されます
2. 画面左のメニューから **Origins** をクリック
3. 表示された画面で Hosts とある下の、**Create host** ボタンをクリック
4. **Create a host** 画面にて、**Name** フィールドに設定名 (任意の文字列) を記入
5. 同画面にて、**Address** フィールドにマイクロサービス A (バックアップ) の IP アドレス `<Service_A_Backup_IP>` を記入
6. 同画面にて、**Enable TLS?** に対して `No, Do not enable TLS.` を選択
7. 同画面にて、**Auto load balance** に対して `Yes` を選択
8. 同画面の下部から、**Create** ボタンをクリックすると、オリジンとなるホストが新たに作成されます
9. 表示された Hosts 画面から、`<Service_A_IP>` をクリック
10. **Edit this host** 画面にて、**Auto load balance** に対して `Yes` を選択
11. 同画面の下部から、**Update** ボタンをクリック

### Step 2: ロードバランス機能の有効化と動作確認

1. 画面の上部から、**Activate** ボタンをクリックすると、変更したサービス設定の有効化が行われます
2. **確認** `https://lab1010-000.global.ssl.fastly.net/` へアクセスし、リロードを繰り返す。ロードバランス (複数オリジンへのリクエスト振り分け) が行われている
3. **さらに確認** 時間に余裕があれば、**Edit this host** から **Weight** の値を変更した上で有効化。ロードバランスの重み付けが変更できることを確認してください

### Step 3: ヘルスチェックの作成と適用

1. 画面上部右の **Configuration** ボタンから **Clone Active** を選択すると、新バージョンの設定が作成されます
2. 画面左のメニューから **Origins** をクリック
3. 表示された画面で Health checks とある下の、**Create your first health check** ボタンをクリック
4. **Create a health check** 画面にて、**Name** フィールドに設定名 (任意の文字列) を記入
5. 同画面にて、**Request** のドロップダウンには `HEAD` を選択、フィールドには `/` を記入
6. 同画面にて、**Host header** フィールドには `lab1010-000.global.ssl.fastly.net` を記入
7. 同画面にて、**Check frequency** には `Medium` を選択
8. 画面の下部から、**Create** ボタンをクリック
9. 表示された Hosts 画面から、`<Service_A_IP>` をクリック
10. **Edit this host** 画面にて、**Health check** には `HEAD / - lab1010-000.global.ssl.fastly.net` を選択
11. 画面の下部から、**Update** ボタンをクリック
12. `<Service_A_Backup_IP>` についても、9〜11 の手順を繰り返す
13. 画面の右上にある **Activate** ボタンをクリックすると、新しい設定の有効化が行われます

### Step 4: フェイルオーバー機能の明示的な実装

1. 画面上部右の **Options** ボタンから **Show VCL** を選択
2. 表示された VCL の冒頭部より、各オリジンを指す Backend の参照名 (**F_** で始まる文字列) を[確認](#コード1-手順-2-の-vcl-表示例)
3. 画面上部右の **Configuration** ボタンから **Clone Active** を選択すると、新バージョンの設定が作成されます
4. 画面左のメニューから **VCL snippets** をクリック
5. 表示された画面で Health checks とある下の、**Create your first VCL snippet** ボタンをクリック
6. **Create a VCL snippet** 画面にて、**Name** フィールドに設定名 (任意の文字列) を記入
7. 同画面にて、**Type** に対して `within subroutine` を、続いて **Select subroutine...** に対して `recv (vcl_recv)` をそれぞれ選択
8. **VCL** フィールドには下記[「コード2」](#コード2-手順-8-にて入力する-vcl-コード)を記入。ただし、*F_addr_123_45_67_89* と *F_Service_A_backup* の部分は手順2で確認した Backend 参照名へ置き換えること
9. 同画面の下部から、**Create** ボタンをクリック

#### コード1: 手順 2 の VCL 表示例
```
backend F_addr_123_45_67_89 {
    .first_byte_timeout = 15s;
    .connect_timeout = 1s;
    .max_connections = 200;
    .between_bytes_timeout = 10s;
    .share_key = "xxxxxxxxxxxxxxxxxxxxxx";
    .port = "80";
    .host = "<Service_A_IP>";
```

#### コード2: 手順 8 にて入力する VCL コード
```
# primary to backup failover
if(req.backend == F_addr_123_45_67_89 && (!req.backend.healthy || req.restarts > 0)) {
  set req.backend = F_Service_A_backup;
# backup to primary failover
} else if(req.backend == F_Service_A_backup && (!req.backend.healthy || req.restarts > 0)) {
  set req.backend = F_addr_123_45_67_89;
}
return(pass);
```

### Step 5: フェイルオーバー機能の有効化と動作確認

1. 画面の上部から、**Activate** ボタンをクリックすると、変更したサービス設定の有効化が行われます
2. **確認** `https://lab1010-000.global.ssl.fastly.net/` へアクセスし、リロードを繰り返す。ロードバランス (複数オリジンへのリクエスト振り分け) が行われている
3. **確認** マイクロサービス A のオリジンのいずれかをダウンさせ、さらにページのリロードを繰り返す。ダウン後 30秒間ほどの間は 503 エラーが散発する。その後、ダウンしたオリジンは切り離され、稼働中のオリジンだけが参照される。


## チュートリアル4 (宿題) BigQuery へのログストリーミングの有効化

以下のドキュメントを参考にすすめてみてください。

* [Log streaming: Google BigQuery](https://docs.fastly.com/guides/streaming-logs/log-streaming-google-bigquery)
* [Fastly のリアルタイム ストリーミング ログを BigQuery で分析する方法](https://cloudplatform-jp.googleblog.com/2017/08/how-to-analyze-Fastly-real-time-streaming-logs-with-BigQuery.html)
