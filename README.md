# Fastly Load Balancing Tutorial

Fastly のロードバランス機能 (LB) を体験するためのチュートリアルです。チュートリアル1では、Fastly LB をマイクロサービス・ルーターとして、チュートリアル2ではマルチクラウド・ロードバランサーとして実装します。

## Prerequisite

このチュートリアルを実行するには、以下が必要です。

* Fastly 開発者用アカウント
* バックエンドとなる HTTP サーバー 3つ (サービス A, サービス A のバックアップ, サービス B)
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

### パラメーター

* ドメイン名: `lab0927-000.global.ssl.fastly.net`
* マイクロサービス A URL: `http://<Service_A_IP>/`
* マイクロサービス B URL: `http://<Service_B_IP>/`

### Step 1: 新サービスの作成

Fastly サービスを新たに作成します。これは、チュートリアル3までの全チュートリアルで用います。

1. Fastly コントロールパネル https://manage.fastly.com へログイン
2. ログイン後、画面の上部左から **Configure** タブをクリック
3. 同画面の上部右の **Create Service** ボタンをクリック
4. **Create a new service** 画面にて、**Name** フィールドにサービス名 (任意の文字列) を記入

### Step 2: 新サービスの作成 (つづき)

1. **Create a new service** 画面にて、**Domain** フィールドにドメイン名を記入します。ここで、ドメイン名は `lab0927-000.global.ssl.fastly.net` の形式とします
2. 同画面にて、**Address** フィールドにマイクロサービス A の IP アドレス `<Service_A_IP>` を記入
3. 同画面にて、**Enable TLS?** に対して `No, Do not enable TLS.` を選択
4. 同画面の下部から、**Create** ボタンをクリックすると、新サービスの作成と有効化が行われます
5. **確認** `https://lab0927-000.global.ssl.fastly.net` へアクセスすると、マイクロサービス A が Fastly 経由で配信されています

### Step 3: コンテンツをキャッシュしない設定にサービスを変更

1. Step 2で作成したサービスの Configure 画面にて、画面上部右の **Configuration** ボタンから **Clone Active** を選択すると、新バージョンの設定が作成され、そこから設定変更が可能です
2. 画面左のメニューから **Setting** をクリック
3. 表示された画面で Fallback TTL とある下の、**Fallback TTL (sec)** フィールドに `0` を入力し、Save ボタンをクリック
4. 同画面の右上にある **Activate** ボタンをクリックすると、新しい設定の有効化が行われます
5. 表示された画面の左側にある **Purge** ボタンから **Purge all** を選択し、表示される **Purge all** ウィンドウで **Purge** ボタンをクリック (**OK** ボタンをクリックしてウィンドウを閉じてください)
6. キャッシュ済みのコンテンツがパージされ、新しい設定が完全に有効となります

### Step 4: マイクロサービス B をオリジンとしてサービスへ追加

1. 画面上部右の **Configuration** ボタンから **Clone Active** を選択すると、新バージョンの設定が作成されます
2. 画面左のメニューから **Origins** をクリック
3. 表示された画面で Hosts とある下の、**Create host** ボタンをクリック
4. **Create a host** 画面にて、**Name** フィールドにホスト名 (任意の文字列) を記入
5. 同画面にて、**Address** フィールドにマイクロサービス B の IP アドレス `<Service_B_IP>` を記入
6. 同画面にて、**Enable TLS?** に対して `No, Do not enable TLS.` を選択
7. 同画面の下部から、**Create** ボタンをクリックすると、オリジンとなるホストが新たに作成されます

### Step 5: /product/ をマイクロサービス B にルーティング



## チュートリアル2 (マルチクラウド・ロードバランサー)



## チュートリアル3 (Big Query へのログストリーミング)

