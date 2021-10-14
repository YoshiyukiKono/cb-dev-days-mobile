## セキュリティ

### ユーザーの作成
ユーザーは、TravelサンプルWebアプリを介して作成されます。ユーザーが作成されると、対応するユーザープロファイルドキュメントがユーザーに関連付けられたCouchbase　Serverに作成されます。さらに、Webアプリは、Sync　Gatewayユーザー管理RESTエンドポイントを介してユーザーをSync　Gatewayに自動的に登録します。

注：Sync Gatewayユーザーは、Sync Gatewayで複製することが認証されているユーザーに対応し、CouchbaseServerで作成されたRBACユーザーとは異なります。

Tips: Syng Gatewayユーザーは、対象バケットのドキュメントとして管理されます。`_sync:user:<ユーザー名>`というキー/IDを持ちます（例：`_sync:user:demo`）。この演習で用いるアプリケーションでは、アプリケーションのユーザーとSync Gatewayのユーザーとして、同じ名前（とパスワード）を用いていますが、それぞれ異なるものです。Syng Gatewayユーザーは、同期の際のアクセス許可などのコントロールのために必要とされます。

#### やってみよう（Webアプリ）

- ブラウザでTravel Web App　URLにアクセスします（Dockerコンテナーとして実行している場合、URLはhttp：//localhost：8080）。

- ユーザー名に「demo」、パスワードに「password」を入力して、新しいユーザーを作成します。「登録」ボタンをクリックしてください。

- Webアプリにログインします。

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/web_user_signup.gif)

#### やってみよう（Couchbase Server）

- ブラウザでCouchbase　ServerのURLにアクセスします（Dockerコンテナーとして実行している場合、URLはhttp：//localhost：8091)。

- Couchbase　Serverのインストール中に設定した管理者アカウントを使用してログインします。

- 左側のナビゲーションペインで「Buckets」を選択します。

- 「ドキュメントID」というラベルの付いたボックスに、「user::demo」と入力します（注：コロンは2つあります）。

- Webアプリを介してサインアップしたときに作成されたユーザードキュメントが表示されます。

- 表示される「ユーザー名」が「demo」であることを確認してください

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/cb_user_auth.gif)

- 次に、IDが「_sync：user：demo」のドキュメントを探します。これは、ユーザーを登録するときにSyncGatewayによって作成されるドキュメントです。

### アクセス制御
このレッスンでは、セキュアなWebゲートウェイであるSync　Gatewayを紹介します。

Couchbase Sync Gatewayは、以下を提供するWebインターフェイスを公開するインターネット向けの同期メカニズムです。

- データの同期とルーティング
- 承認
- アクセス制御

この章では、承認とアクセス制御に焦点を当てます。同期レッスンでは、データの同期とルーティングについて説明します。

ではインストールガイド、我々は特定の設定ファイルと同期ゲートウェイを起動する手順を説明しました。Sync Gateway構成ファイルは、SyncGatewayの実行時の動作を決定します。

https://github.com/couchbaselabs/mobile-travel-sample/blob/master/sync-gateway-config-travelsample.jsonにあるsync-gateway-config-travelsample.jsonファイルを開きます。

この`users`セクションでは、Sync　Gatewayを使用してレプリケートするためのアクセスを許可されているSync　Gatewayユーザーのハードコードされたリストを定義します。
ユーザーのリストをハードコーディングすることは、「ユーザーの作成」セクションで説明されているように、Sync　Gatewayユーザーを動的に作成する代わりの方法です。
設定ファイルには、「*」チャネルへのアクセスが許可された「password」のパスワードを持つ「admin」という名前のハードコードされたユーザーがいます。

```JAVASCRIPT
"users": {
  "admin": {"password": "password", "admin_channels": ["*"]}
}
```

設定ファイルの同期関数は、アクセス制御ロジックを実装するJavaScript関数です。このaccessメソッドは、現在のユーザーに特定のチャネルへのアクセスを許可するために使用されます。「同期」セクションでチャネルについて詳しく説明します。今のところ、ドキュメントはチャネルに関連付けられていることに注意してください。したがって、ドキュメントへのアクセスは、チャネルへのアクセス権を制御することによって制御されます。

```JAVASCRIPT
  // Give user read access to channel
  if (!isDelete()) {
  // Deletion of user document is essentially deletion of user
  access(username,"channel." + username)
}
```

#### やってみよう

- ターミナルで次のコマンドを実行します。（ローカルホスト以外のインストールを行った場合は、コマンドの「localhost」の部分をSync GatewayをホストするサーバーのIPアドレスに置き換えてください)

```BASH
curl -X GET http://localhost:4984/travel-sample/
```

- サーバーから「Unauthorized」エラーが表示されることを確認します

- youtターミナルで次のコマンドを実行します。`authorization`ヘッダは「demo:password」(<ユーザー名>:<パスワード>)のbase64でエンコードされた値です。
（ローカルホスト以外のインストールを行った場合は、コマンドの「localhost」の部分をSync GatewayをホストするサーバーのIPアドレスに置き換えてください)

```BASH
curl -X GET http://localhost:4984/travel-sample/ -H 'authorization: Basic ZGVtbzpwYXNzd29yZA=='
```

- 「travel-sample」データベースの詳細が表示され、「state」が「online」であることを確認します

[目次へ戻る](./README.md)
