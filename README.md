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

* ドメイン名: `lab0905-000.global.ssl.fastly.net`
* マイクロサービス A URL: `http://<service_a_ip>/`
* マイクロサービス B URL: `https://<name.appspot.com>/`

### Step 1: 新サービスの作成

Fastly サービスを新たに作成します。これは、チュートリアル2でも用います。

1. Fastly コントロールパネル https://manage.fastly.com へログイン
2. ログイン後、画面の上部左から **Configure** タブをクリック
3. 同画面の上部右の **Create Service** ボタンをクリック
4. 仮のサービス名 `Unnamed Service` をクリックして、ご自身で決めたサービス名 (任意の文字列) を記入

### Step 2: 新サービスに Domain と Host を設定する

1. **Domains** 画面にて、**Create Your First Domain** ボタンをクリックします。
2. **Create a domain** 画面にて、**Domain Name** フィールドにドメイン名を記入します。ここで、ドメイン名は `lab0905-000.global.ssl.fastly.net` の形式とします。
3. 動画面の **Create** ボタンをクリックして入力した **Domain Name** を確定させます。
4. 画面左のメニューから **Origins** をクリック
5. **Hosts** 画面にマイクロサービス A の IP アドレス `<service_a_ip>` を記入
6. 続いて **Add** ボタンをクリック
7. 同画面の右上にある **Activate** ボタンをクリックすると、新サービスの作成と有効化が行われます
8. **確認** `https://lab0905-000.global.ssl.fastly.net` へアクセスすると、マイクロサービス A が Fastly 経由で配信されています

### Step 3: コンテンツをキャッシュしない設定にサービスを変更

1. Step 2で作成したサービスの Version 1 の画面にて、**Clone** をクリックすると、新バージョンの設定が作成され変更可能となる
2. 画面左のメニューから **Setting** をクリック
3. 表示された画面で Fallback TTL とある下の、**Fallback TTL (sec)** フィールドに `0` を入力し、Save ボタンをクリック
4. 同画面の右上にある **Activate** ボタンをクリックすると、新しい設定の有効化が行われます
5. 表示された画面の上部左側にある矢印ボタンをクリック
6. 画面左にある **Purge** ドロップダウンより **Purge all** を選択し、表示される **Purge all** ウィンドウで **Purge** ボタンをクリック (**Okay** ボタンをクリックしてウィンドウを閉じてください)
7. キャッシュ済みのコンテンツがパージされ、新しい設定が完全に有効となります

### Step 4: マイクロサービス B をオリジンとしてサービスへ追加

1. 画面上部右の **Edit Configuration** ボタンから **Clone Version 2** を選択すると、新バージョンの設定が作成されます
2. 画面左のメニューから **Origins** をクリック
3. 表示された画面で Hosts とある下の、**Create a host** ボタンをクリック
4. **Create a host** 画面にて、**Name** フィールドに設定名 (任意の文字列) を記入
5. 同画面にて、**Address** フィールドにマイクロサービス B のドメイン `<name.appspot.com>` を記入
6. 続いて **Add** ボタンをクリックすると、オリジンとなるホストが新たに作成されます

### Step 5: /product/ をマイクロサービス B にルーティング

1. Step 4で追加した Hosts の **Attache a condition** をクリック
2. **Create a new request condition** 画面にて、**Name** フィールドに条件の名前 (任意の文字列) を記入
3. 同画面にて、**Apply if...** フィールドに `req.url ~ "^/product/"` と記入
4. 同画面の下部から、**Send and Apply to (ホスト名)** ボタンをクリックすると、ホストに対して条件が設定されます

### Step 6: Override host の設定

1. 画面左のメニューから **Settings** をクリック
2. **Override host** を On にして、**Override host header** フィールドに `<name.appspot.com>` を記入

### Step 7: マイクロサービス・ルーター機能の有効化と動作確認

1. 画面の上部から、**Activate** ボタンをクリックすると、変更したサービス設定の有効化が行われます
2. **確認** `https://lab0905-000.global.ssl.fastly.net/product/` へアクセスすると、マイクロサービス B が Fastly 経由で配信されています



## チュートリアル2 (Dynamic Serversによるマルチクラウド・ロードバランサー )

### ねらい

* 複数オリジン間のロードバランス機能を API から実装できる

### 必要となる設定値

* <api_key>
* <service_id>
* <service_a_ip>
* <service_a_backup_ip>

## Tutorial

[![Open this project in Cloud Shell](http://gstatic.com/cloudssh/images/open-btn.png)](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/toshiaizawa/fastly-lb-tutorial.git&page=editor&tutorial=tutorial.md)

## Run tutorial
Click `Open in Google Cloud Shell` above.

You can launch tutorial manually as follows:
```
cloudshell launch-tutorial -d tutorial.md
```

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

curl -sv -H "Fastly-Key: ${API_KEY}" -X DELETE \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/pool/cloudpool \
| jq
```


## チュートリアル3 (宿題) BigQuery へのログストリーミングの有効化

以下のドキュメントを参考にすすめてみてください。

* [Log streaming: Google BigQuery](https://docs.fastly.com/guides/streaming-logs/log-streaming-google-bigquery)
* [Fastly のリアルタイム ストリーミング ログを BigQuery で分析する方法](https://cloudplatform-jp.googleblog.com/2017/08/how-to-analyze-Fastly-real-time-streaming-logs-with-BigQuery.html)
