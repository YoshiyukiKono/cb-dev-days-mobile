# Sync Gateway環境準備

注：DockerコンテナーでSync Gatewayを実行している場合は、コンテナーでもCouchbaseServerが実行されていることを確認してください。そうでない場合は、Couchbase Server（上記）の指示に従ってサーバーコンテナをインストールしてください。

「workshop」という名前のローカルDockerネットワークがまだ存在しない場合は、作成します。dockerを使用してCouchbaseサーバーをデプロイする手順に従った場合、これは当てはまらないはずです。ターミナルウィンドウを開き、次のコマンドを実行します。

```BASH
$ docker network ls
$ docker network create -d bridge workshop
```

コンテナーでアプリケーションを実行するには、最初にDockerCloudからDockerイメージを取得します。

```BASH
$ docker pull couchbase/sync-gateway:2.8.0-enterprise
```

Sync Gatewayはsync-gateway-config-travelsample.json、WorkshopRepoステップの一部としてダウンロードする必要があるという名前の構成ファイルを使用して起動する必要があります。設定ファイルは/path/to/mobile-travel-sampleディレクトリ/フォルダにあります。

sync-gateway-config-travelsample.json任意のテキストエディタを使用してファイルを開きます

アプリがCouchbaseServerに接続するには、サーバーのアドレスを指定する必要があります。Couchbase Server Dockerコンテナを起動したときにname、「cb-server」を指定したことに注意してください。

ファイルをに置き換えlocalhostて保存します。`sync-gateway-config-travelsample.json`cb-server

+

```JSON
"server": "couchbase://cb-server"
```

`sync-gateway-config-travelsample.json`ファイルを使用してSync Gatewayを起動します。
`sync-gateway-config-travelsample.json`ファイルが含まれているフォルダから以下のコマンドを実行する必要があります。

### ウィンドウズ

```BASH
cd /path/to/mobile-travel-sample/
docker run -p 4984-4985:4984-4985 --network workshop --name sync-gateway -d -v %cd%/sync-gateway-config-travelsample.json:/etc/sync_gateway/sync_gateway.json couchbase/sync-gateway:2.8.0-enterprise -adminInterface :4985 /etc/sync_gateway/sync_gateway.json
```

### Windows以外のプラットフォーム

```BASH
$ cd c:\path\to\mobile-travel-sample\
$ docker run -p 4984-4985:4984-4985 --network workshop --name sync-gateway -d -v `pwd`/sync-gateway-config-travelsample.json:/etc/sync_gateway/sync_gateway.json couchbase/sync-gateway:2.8.0-enterprise -adminInterface :4985 /etc/sync_gateway/sync_gateway.json
```

次のコマンドを実行すると、いつでもログを表示できます。

```BASH
$ docker logs sync-gateway
```

ターミナルウィンドウで次のコマンドを使用して、「sync-gateway」という名前のDockerコンテナが実行されていることを確認します。

```BASH
$ docker ps
　```

### やってみよう
http://127.0.0.1:4984ブラウザでこのURLにアクセスします

以下のようなJSON応答が返されることを確認します

```JSON
{"couchdb":"Welcome","vendor":{"name":"Couchbase Sync Gateway","version":"2.8"},"version":"Couchbase Sync Gateway/2.8.0(271;bf3ddf6) EE"}
```

[目次へ戻る](./README.md)
