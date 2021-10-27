## セキュリティ

[オリジナル原文](https://docs.couchbase.com/tutorials/mobile-travel-sample/android/develop/security.html)

**※ソースコードに関する解説は、プログラムと処理の関係を理解していただけるように、「確認のため」記載されています。この演習では、接続先の変更以外では、ソースコードを変更する必要はありません。演習として、「やってみよう」セクションのアプリケーションの操作をご実施ください。**

### ユーザーの作成
ユーザーは、Travel　Webアプリを介して作成されます。ユーザーが作成されると、対応するユーザープロファイルドキュメントがユーザーに関連付けられたCouchbase　Serverに作成されます。さらに、Webアプリは、Sync　Gatewayユーザー管理RESTエンドポイントを介してユーザーをSync　Gatewayに自動的に登録します。

注：Sync Gatewayのユーザーは、CouchbaseServerのユーザーとは異なります。

Tips: Syng Gatewayユーザー情報は、同期対象バケットの中のドキュメントとして管理されます。`_sync:user:<ユーザー名>`というキー/IDを持ちます（例：`_sync:user:demo`）。この演習で用いるアプリケーションでは、アプリケーションのユーザーとSync Gatewayのユーザーとして、同じ名前（とパスワード）を用いていますが、それぞれ異なるものです。Syng Gatewayユーザーは、同期の際のアクセス許可などのコントロールのために必要とされます。

#### やってみよう（Webアプリ）

- ブラウザでTravel Web App　URLにアクセスします。Dockerコンテナーとして実行している場合、URLは次の通りです。: http://localhost:8080

- ユーザー名に「demo」、パスワードに「password」を入力して、新しいユーザーを作成します。「Register(登録)」ボタンをクリックしてください。

- Webアプリにログインします。

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/web_user_signup.gif)

#### やってみよう（Couchbase Server）

- ブラウザでCouchbase　ServerのURLにアクセスします。Dockerコンテナーとして実行している場合、URLは次の通りです。: http://localhost:8091

- Couchbase　Serverのインストール中に設定した管理者アカウントを使用してログインします。Dockerで構築した場合、ID/パスワードは、次の通りです。: Administrator/password

- 左側のナビゲーションペインで「Buckets」を選択します。

- 「Document ID」というラベルの付いた入力欄に、「user::demo」と入力します（注：コロンは2つあります）。

- Webアプリを介してサインアップしたときに作成されたユーザードキュメントが表示されます。

- 「username」が「demo」であることを確認してください

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/cb_user_auth.gif)

- 次に、IDが「`_sync：user：demo`」のドキュメントを探します。これは、ユーザーを登録するときにSync Gatewayによって作成されるドキュメントです。

### アクセス制御

Sync Gatewayは、以下のセキュリティ関連機能を提供します。

- 承認
- アクセス制御


Sync Gatewayの実行時の動は、Sync Gateway構成ファイルによって決定されます。

`sync-gateway-config-travelsample.json`ファイルを開きます。

`users`セクションでは、Sync　Gatewayを使用してレプリケートするためのアクセスを許可されているSync　Gatewayユーザーのハードコードされたリストを定義することができます
。ユーザーを定義する他の方法としてREST APIがあります。Sync　Gatewayユーザーを動的に作成する代わりに、構成ファイルに、ユーザーのリストをハードコーディングすることができます。


設定ファイルには、「`*`(全ての)」チャネルへのアクセスが許可された「password」というパスワードを持つ「admin」という名前のハードコードされたユーザーが定義されています。

```JAVASCRIPT
"users": {
  "admin": {"password": "password", "admin_channels": ["*"]}
}
```

設定ファイルには、同期（"sync"）関数を設定することができ、JavaScript関数として記載します。この中で、アクセス制御ロジックを実装します。
`access`メソッドは、現在のユーザーに特定のチャネルへのアクセスを許可するために使用されます。チャネルについては、別に詳しく説明します。

ドキュメントはチャネルに関連付けられています。ドキュメントへのアクセスは、チャネルへのアクセス権によって制御されます。

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

- 今度は、ターミナルで次のコマンドを実行します。`authorization`ヘッダは「demo:password」(<ユーザー名>:<パスワード>)のbase64でエンコードされた値です。
（ローカルホスト以外のインストールを行った場合は、コマンドの「localhost」の部分をSync GatewayをホストするサーバーのIPアドレスに置き換えてください)

```BASH
curl -X GET http://localhost:4984/travel-sample/ -H 'authorization: Basic ZGVtbzpwYXNzd29yZA=='
```

- 「travel-sample」データベースの詳細が表示され、「state」が「online」であることを確認します

[目次へ戻る](./README.md)
