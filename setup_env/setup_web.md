# Travel Sample Webバックエンド


## Dockerネットワーク作成確認

Dockerコンテナーで環境を構築している場合は、Couchbase ServerとSync Gatewayも同じDockerネットワークで実行されている必要があります。

「workshop」という名前のローカルDockerネットワークが存在していることを確認します。

ターミナルで、次のコマンドを実行します。

```BASH
$ docker network ls
```

Dockerを使用してCouchbaseサーバーをデプロイする手順に従った場合、既に作成されているはずですが、まだ存在作成されていない場合は、作成します。

```BASH
$ docker network create -d bridge workshop
```
## Dockerイメージ取得

最初にDocker CloudからDockerイメージを取得します。ターミナルを開き、以下を実行します。

```BASH
$ docker pull connectsv/try-cb-python-v2:6.5.0-server
```

## Dockerコンテナ起動

次の方法でDockerコンテナを起動します。

```BASH
$ docker run -it -p 8080:8080 --network workshop connectsv/try-cb-python-v2:6.5.0-server
```

## Docker コンテナ起動確認

コンソールに次のように出力されます。

```BASH
Running on http://127.0.0.1:8080/ (Press CTRL+C to quit)
```

### やってみよう

ブラウザでWebアプリケーションを開きます。URLは、次の通りです: http://127.0.0.1:8080/

以下に示すように、Travel SampleWebアプリのログイン画面が表示されることを確認します。


![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/try-cb-login-2.png)

[目次へ戻る](./README.md)
