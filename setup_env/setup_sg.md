# Sync Gateway環境準備

注：DockerコンテナーでSync Gatewayを実行している場合は、コンテナーでもCouchbaseServerが実行されていることを確認してください。そうでない場合は、Couchbase Server（上記）の指示に従ってサーバーコンテナをインストールしてください。

「workshop」という名前のローカルDockerネットワークがまだ存在しない場合は、作成します。dockerを使用してCouchbaseサーバーをデプロイする手順に従った場合、これは当てはまらないはずです。ターミナルウィンドウを開き、次のコマンドを実行します。

```BASH
$ docker network ls
$ docker network create -d bridge workshop
```

## Dockerイメージ取得

コンテナーでアプリケーションを実行するには、最初にDockerCloudからDockerイメージを取得します。

```BASH
$ docker pull couchbase/sync-gateway:2.8.0-enterprise
```

## Sync Gateway構成ファイルの編集

Sync Gatewayはsync-gateway-config-travelsample.json、WorkshopRepoステップの一部としてダウンロードする必要があるという名前の構成ファイルを使用して起動します。
設定ファイルは/path/to/mobile-travel-sampleディレクトリにあります。

任意のテキストエディタを使用してsync-gateway-config-travelsample.jsonを開きます

アプリがCouchbaseServerに接続するには、サーバーのアドレスを指定する必要があります。
Couchbase Server Dockerコンテナを起動したときにname、「cb-server」を指定したことに注意してください。

「localhost」を「cb-server」に置き換えてファイル保存します。


```JSON
"server": "couchbase://cb-server"
```

`sync-gateway-config-travelsample.json`ファイルを使用してSync Gatewayを起動します。
`sync-gateway-config-travelsample.json`ファイルが含まれているフォルダから以下のコマンドを実行する必要があります。

## Dockerコンテナの起動

### Windowsでの実行方法

```BASH
cd /path/to/mobile-travel-sample/
docker run -p 4984-4985:4984-4985 --network workshop --name sync-gateway -d -v %cd%/sync-gateway-config-travelsample.json:/etc/sync_gateway/sync_gateway.json couchbase/sync-gateway:2.8.0-enterprise -adminInterface :4985 /etc/sync_gateway/sync_gateway.json
```

### Windows以外のプラットフォームでの実行方法

```BASH
$ cd c:\path\to\mobile-travel-sample\
$ docker run -p 4984-4985:4984-4985 --network workshop --name sync-gateway -d -v `pwd`/sync-gateway-config-travelsample.json:/etc/sync_gateway/sync_gateway.json couchbase/sync-gateway:2.8.0-enterprise -adminInterface :4985 /etc/sync_gateway/sync_gateway.json
```

## Dockerプロセス確認

ターミナルウィンドウで次のコマンドを使用して、「sync-gateway」という名前のDockerコンテナが実行されていることを確認します。

```BASH
$ docker ps
　```
 
## 起動ログ確認

次のコマンドを実行すると、いつでもログを表示できます。

```BASH
$ docker logs sync-gateway
```

### やってみよう
ブラウザで下記のURLにアクセスします

http://127.0.0.1:4984

以下のようなJSON応答が返されることを確認します

```JSON
{"couchdb":"Welcome","vendor":{"name":"Couchbase Sync Gateway","version":"2.8"},"version":"Couchbase Sync Gateway/2.8.0(271;bf3ddf6) EE"}
```

[目次へ戻る](./README.md)
