# DockerによるCouchbase Server環境準備

「workshop」という名前のローカルDockerネットワークがまだ存在しない場合は、作成します。ターミナルウィンドウを開き、次のコマンドを実行します。

```BASH
$ docker network ls
$ docker network create -d bridge workshop
```

コンテナでアプリケーションを実行するには、最初にDockerHubからDockerイメージを取得します。新しいターミナルウィンドウを開き、以下を実行します。

```BASH
$ docker pull couchbase/server-sandbox:6.5.0
```

コマンドが完了すると、次の方法でアプリケーションを起動できます。

```BASH
$ docker run -d --name cb-server --network workshop -p 8091-8094:8091-8094 -p 11210:11210 couchbase/server-sandbox:6.5.0
```

次のコマンドを実行すると、いつでもログを表示できます。

```BASH
$ docker logs cb-server
```

サーバーが起動するまでに数秒かかる場合があります。次のコマンドを使用して、Dockerイメージが実行されていることを確認します。

```BASH
$ docker ps
```

### やってみよう

- 「http：// localhost：8091」（または「http：//127.0.0.1：8091」）を開いて、CouchbaseServerの「管理コンソール」を開きます。

- ユーザー名を「管理者」、パスワードを「パスワード」としてコンソールにログインします

- 左側のメニューから「バケット」オプションを選択します

- travel-sampleバケットに約31,000のドキュメントがあることを確認します

[目次へ戻る](./README.md)
