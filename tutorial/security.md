## セキュリティ

### ユーザーの作成
ユーザーは、TravelサンプルWebアプリを介して作成されます。ユーザーが作成されると、対応するユーザープロファイルドキュメントがユーザーに関連付けられたCouchbaseServerに作成されます。さらに、Webアプリは、SyncGatewayユーザー管理RESTエンドポイントを介してユーザーをSyncGatewayに自動的に登録します。注：Sync Gatewayユーザーは、Sync Gatewayで複製することが認証されているユーザーに対応し、CouchbaseServerで作成されたRBACユーザーとは異なります。

#### 試してみる（Webアプリ）
ブラウザでTravelWeb AppURLにアクセスします。Webアプリを手動またはDockerコンテナー経由でインストールした場合、このURLはhttp：// localhost：8080になります。クラウドインストールを使用した場合は、Webアプリのクラウドインスタンスにアクセスしてください。

ユーザー名に「demo」、パスワードに「password」を入力して、新しいユーザーを作成します。「登録」ボタンをクリックしてください

Webアプリにログインする必要があります。ユーザーのために何も作成されるべきではありません。

Webユーザーのサインアップ
#### 試してみてください（Couchbase Server）

- ブラウザでCouchbaseServerのURLにアクセスします。サーバーを手動またはDockerコンテナー経由でインストールした場合、このURLはhttp：// localhost：8091になります。クラウドインストールを使用した場合は、サーバーのクラウドインスタンスにアクセスしてください。

- CouchbaseServerのインストール中に設定した管理者の資格情報を使用してログインします。

- 左側のナビゲーションペインで[バケット]を選択します。

- 「ドキュメントID」というラベルの付いたボックスに、「user :: demo」と入力します（注：コロンは2つあります）。

- Webアプリを介してサインアップしたときに作成されたユーザードキュメントが表示されます。

- 表示される「ユーザー名」が「デモ」であることを確認してください

cbユーザー認証
- 次に、IDが「_sync：user：demo」のドキュメントを探します。これは、ユーザーを登録するときにSyncGatewayによって作成されるドキュメントです。

### アクセス制御
このレッスンでは、安全なWebゲートウェイであるSyncGatewayを紹介します。

Couchbase Sync Gatewayは、以下を提供するWebインターフェイスを公開するインターネット向けの同期メカニズムです。

- データの同期とルーティング
- 承認
- アクセス制御

この章では、承認とアクセス制御に焦点を当てます。同期レッスンでは、データの同期とルーティングについて説明します。

ではインストールガイド、我々は特定の設定ファイルと同期ゲートウェイを起動する手順を説明しました。Sync Gateway構成ファイルは、SyncGatewayの実行時の動作を決定します。

https://github.com/couchbaselabs/mobile-travel-sample/blob/master/sync-gateway-config-travelsample.jsonにあるsync-gateway-config-travelsample.jsonファイルを開きます。

このusersセクションでは、同期ゲートウェイを使用してレプリケートするためのアクセスを許可されている同期ゲートウェイユーザーのハードコードされたリストを定義します。ユーザーのリストをハードコーディングすることは、「ユーザーの作成」セクションで説明されているように、SyncGatewayユーザーを動的に作成する代わりの方法です。設定ファイルには、「*」チャネルへのアクセスが許可された「password」のパスワードを持つ「admin」という名前のハードコードされたユーザーがいます。

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
ターミナルで次のコマンドを実行します。クラウドベースのインストールを行った場合は、localhost以下のコマンドでSyncGatewayのクラウドインスタンスのIPアドレスに置き換えてください。

```BASH
curl -X GET http://localhost:4984/travel-sample/
```

サーバーから「Unauthorized」エラーが表示されることを確認します

youtターミナルで次のコマンドを実行します。authorizationヘッダは「：パスワードデモ」のbase64でエンコードされた値です。クラウドベースのインストールを行った場合は、localhost以下のコマンドでSyncGatewayのクラウドインスタンスのIPアドレスに置き換えてください。

```BASH
curl -X GET http://localhost:4984/travel-sample/ -H 'authorization: Basic ZGVtbzpwYXNzd29yZA=='
```

「travel-sample」データベースの詳細が表示され、「state」が「online」であることを確認します