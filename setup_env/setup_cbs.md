# DockerによるCouchbase Server環境準備

## ローカルDockerネットワークの作成
「workshop」という名前のローカルDockerネットワークを作成します。ターミナルを開き、次のコマンドを実行します。

```BASH
$ docker network ls
$ docker network create -d bridge workshop
```
## Dockerイメージ取得

最初にDockerHubからDockerイメージを取得します。ターミナルから、以下を実行します。

```BASH
$ docker pull couchbase/server-sandbox:6.5.0
```

## Dockerコンテナ起動

次の方法でDockerコンテナを起動します。

```BASH
$ docker run -d --name cb-server --network workshop -p 8091-8094:8091-8094 -p 11210:11210 couchbase/server-sandbox:6.5.0
```



## Docker実行確認

次のコマンドを使用して、Dockerイメージが実行されていることを確認します。

```BASH
$ docker ps
```
Dockerプロセスが開始されていても、サーバーが起動するまでに数秒かかる場合があります。



## 起動ログ確認

次のコマンドを実行すると、Couchbase Serverの起動ログを表示できます。

```BASH
$ docker logs cb-server
```

### やってみよう

- CouchbaseServerのWeb管理コンソールを開きます（URLは、次の通りです：　　http://localhost:8091)。

- ユーザー名を「Administrator」、パスワードを「password」としてコンソールにログインします

- 左側のメニューから「Buckets」オプションを選択します

- travel-sampleバケットに約31,000のドキュメントがあることを確認します

[目次へ戻る](./README.md)
