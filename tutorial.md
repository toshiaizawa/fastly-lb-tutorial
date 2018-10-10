# Tutorial

## Step 1: 環境設定
### API KEY の設定
```bash
API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
### Service ID の設定
```bash
SERVICE_ID=xxxxxxxxxxxxxxxxxxxxxx
```
### サービスの対象バージョンの設定
```bash
VERSION=3
```

## Step 2: API を用いたサービス内容確認
```bash
curl sv -H "Fastly-Key: ${API_KEY}" \
https://api.fastly.com/service/${SERVICE_ID}/details | jq
```

## Step 3: Clone による新バージョンの作成
```bash
curl -sv -H "Fastly-Key: ${API_KEY}" -X PUT \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/clone \
| jq
```

## Step 4: マイクロサービス A 用オリジンの設定から削除 (UI経由)

UIでの操作

## Step 5: Custom VCL のアップロード
### サービスの対象バージョンの更新
```bash
VERSION=4
```
### アップロード 
```bash
curl -vs -H "Fastly-Key: ${API_KEY}" -X POST -H \
"Content-Type: application/x-www-form-urlencoded" \
--data "name=main&main=true" \
--data-urlencode "content@main.vcl" \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/vcl | jq
```

## Step 6: サーバープールの作成
```bash
curl -sv -H "Fastly-Key: ${API_KEY}" -X POST \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/pool -d \
'name=cloudpool&comment=cloudpool' | jq
```
### サーバープールのIDを設定
```bash
POOL_ID_1=<id>
```

## Step 7: サーバープールへサーバーの追加
```bash
curl -vs -H "Fastly-Key: ${API_KEY}" -X POST \
https://api.fastly.com/service/${SERVICE_ID}/pool/${POOL_ID_1}/server -d \
'address=<service_a_ip>' | jq
```
### サーバーのIDを設定
```bash
SERVER_ID_1=<id>
```

## Step 8: サービスバージョンの有効化
```bash
curl -vs -H "Fastly-Key: ${API_KEY}" -X PUT \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/activate | jq
```
### サービスの対象バージョンの更新
```bash
VERSION=5
```

## Step 9: サーバープールへサーバーの追加
```bash
curl -vs -H "Fastly-Key: ${API_KEY}" -X POST \
https://api.fastly.com/service/${SERVICE_ID}/pool/${POOL_ID_1}/server -d \
'address=<service_a_backup_ip>' | jq
```
### サーバーのIDを設定
```bash
SERVER_ID_2=<id>
```

## Step 10: サーバープール内のサーバーの weight を変更
```bash
curl -vs -H "Fastly-Key: ${API_KEY}" -X PUT \
https://api.fastly.com/service/${SERVICE_ID}/pool/${POOL_ID_1}/server/${SERVER_ID_1} \
-d 'weight=10' | jq
```

## Step 11: (後片付け) サーバープールの削除
###クローン
```bash
curl -sv -H "Fastly-Key: ${API_KEY}" -X PUT \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/clone \
| jq
```

### サービスの対象バージョンの更新
```bash
VERSION=6
```

### サーバープールの削除
```bash
curl -sv -H "Fastly-Key: ${API_KEY}" -X DELETE \
https://api.fastly.com/service/${SERVICE_ID}/version/${VERSION}/pool/cloudpool \
| jq
```
