# Travel Sample Webバックエンド

注：DockerコンテナーでWebアプリを実行している場合は、Couchbase ServerとSync Gatewayも同じDockerネットワークで実行されていることを確認してください。
ない場合は、の指示に従ってくださいCouchbase　Serverにドッキングウィンドウと命令を使用して、サーバーコンテナをインストールするには、セクション同期ゲートウェイ同期ゲートウェイ・コンテナをインストールするセクション。

「workshop」という名前のローカルDockerネットワークがまだ存在しない場合は、作成します。ターミナルウィンドウを開き、次のコマンドを実行します。

```BASH
$ docker network ls
$ docker network create -d bridge workshop
```

コンテナーでアプリケーションを実行するには、最初にDockerCloudからDockerイメージを取得します。ターミナルウィンドウを開き、以下を実行します。

```BASH
$ docker pull connectsv/try-cb-python-v2:6.5.0-server
```

コマンドが完了すると、次の方法でアプリケーションを起動できます。

```BASH
$ docker run -it -p 8080:8080 --network workshop connectsv/try-cb-python-v2:6.5.0-server
```

次に、コンソール出力に次のように表示されます。

BASH
コピー
Running on http://127.0.0.1:8080/ (Press CTRL+C to quit)


### やってみよう

Webブラウザでhttp://127.0.0.1:8080/を開きます。

以下に示すように、Travel SampleWebアプリのログイン画面が表示されることを確認します。


![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/try-cb-login-2.png)

[目次へ戻る](./README.md)
