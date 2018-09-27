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
5. 同画面にて、**Address** フィールドにマイクロサービス B の IP アドレス `<Service_B_IP>` を記入
6. 同画面にて、**Enable TLS?** に対して `No, Do not enable TLS.` を選択
7. 同画面の下部から、**Create** ボタンをクリックすると、オリジンとなるホストが新たに作成されます

### Step 5: /product/ をマイクロサービス B にルーティング

1. Step 4で追加した Hosts の **Attache a condition** をクリック
2. **Create a new request condition** 画面にて、**Name** フィールドに条件の名前 (任意の文字列) を記入
3. 同画面にて、**Apply if...** フィールドに `req.url ~ "^/product/"` と記入
4. 同画面の下部から、**Send and Apply to (ホスト名)** ボタンをクリックすると、ホストに対して条件が設定されます

### Step 6: オリジンに対するリクエストの URL パス部を変更 (/product/* → /*)

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

### Step 7: マイクロサービス・ルーター機能の有効化と動作確認

1. 画面の上部から、**Activate** ボタンをクリックすると、変更したサービス設定の有効化が行われます
2. **確認** `https://lab0927-000.global.ssl.fastly.net/product/` へアクセスすると、マイクロサービス B が Fastly 経由で配信されています


## チュートリアル2 (マルチクラウド・ロードバランサー)

### ねらい

* 複数オリジン間のロードバランス機能が実装できる
* ヘルスチェックとフェイルオーバーが実装できる

### 必要となる設定値

* マイクロサービス A (バックアップ) URL: `http://<Service_A_Backup_IP>/`
* マイクロサービス A の VCL 内での backend 参照名: `F_addr_123_45_67_89`
* マイクロサービス A (バックアップ) の VCL 内での backend 参照名: `F_Service_A_Backup`

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
2. **確認** `https://lab0927-000.global.ssl.fastly.net/` へアクセスし、リロードを繰り返す。ロードバランス (複数オリジンへのリクエスト振り分け) が行われている
3. **さらに確認** 時間に余裕があれば、**Edit this host** から **Weight** の値を変更した上で有効化。ロードバランスの重み付けが変更できることを確認してください

### Step 3: ヘルスチェックの作成と適用

1. 画面上部右の **Configuration** ボタンから **Clone Active** を選択すると、新バージョンの設定が作成されます
2. 画面左のメニューから **Origins** をクリック
3. 表示された画面で Health checks とある下の、**Create your first health check** ボタンをクリック
4. **Create a health check** 画面にて、**Name** フィールドに設定名 (任意の文字列) を記入
5. 同画面にて、**Request** のドロップダウンには `HEAD` を選択、フィールドには `/` を記入
6. 同画面にて、**Host header** フィールドには `lab0927-000.global.ssl.fastly.net` を記入
7. 同画面にて、**Check frequency** には `Medium` を選択
8. 画面の下部から、**Create** ボタンをクリック
9. 表示された Hosts 画面から、`<Service_A_IP>` をクリック
10. **Edit this host** 画面にて、**Health check** には `HEAD / - lab0927-000.global.ssl.fastly.net` を選択
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
2. **確認** `https://lab0927-000.global.ssl.fastly.net/` へアクセスし、リロードを繰り返す。ロードバランス (複数オリジンへのリクエスト振り分け) が行われている
3. **確認** マイクロサービス A のオリジンのいずれかをダウンさせ、さらにページのリロードを繰り返す。ダウン後 30秒間ほどの間は 503 エラーが散発する。その後、ダウンしたオリジンは切り離され、稼働中のオリジンだけが参照される。

## チュートリアル3 (デモ：Dynamic Servers を用いたロードバランサー)

Dynamic Servers を用いると、より柔軟なロードバランス機能が実現できます。チュートリアル2では、ロードバランス設定の変更には、VCL の変更と有効化が必要でした。Dynamic Servers では API 呼び出しにより、VCL にふれることなく、サーバーの追加・削除等が可能です。
