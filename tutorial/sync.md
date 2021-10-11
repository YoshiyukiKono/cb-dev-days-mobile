## 同期
### チャネル/データルーティング
アクセス制御レッスンでは、CouchbaseのSync Gatewayは、認証とアクセス制御機能をサポートする方法について説明しました。
このレッスンでは、データの同期とルーティングにどのように使用できるかについて説明します。

Sync Gateway構成ファイルは、サーバー構成や、Sync Gatewayインスタンスが対話できるデータベースまたはデータベースのセットなど、Sync Gatewayの実行時の動作を決定します。

Sync Gatewayはチャネルを使用して、多数のユーザー間でデータベースを簡単に共有し、データベースへのアクセスを制御できるようにします。
概念的には、チャネルはタグと見なすことができます。データベース内のすべてのドキュメントは一連のチャネルに属し、ユーザーには一連のチャネルへの適切なアクセスが許可されます。チャネルは次の目的で使用されます。

- データセットを分割します。

- ユーザーにドキュメントへのアクセスを許可します。

- デバイスに同期されるデータの量を最小限に抑えます。

で同期Gatewayインストールのセクション、我々は特定の設定ファイルと同期ゲートウェイを起動する手順を説明しました。

`https://github.com/couchbaselabs/mobile-travel-sample/blob/master/sync-gateway-config-travelsample.json`にある`sync-gateway-config-travelsample.json`ファイルを開きます。これには、sync functionソースコードがSync Gatewayのデータベース構成ファイルに保存されているJavaScript関数が含まれています。

```JAVASCRIPT
/* Routing */
// Add doc to the user's channel.
channel("channel." + username);
```

### 共有バケットアクセス
このレッスンを開始する前に、Sync Gatewayのインストールセクションの手順に従って、Sync　Gatewayが稼働していることを確認してください。

Sync Gateway1.5およびCouchbaseServer 5.0以降、モバイルアプリケーションとサーバー/ Webアプリケーションは同じバケットに対して読み取りと書き込みを行うことができるようになりました。これは、Sync　Gateway構成ファイルで有効にできるオプトイン機能です。

収束
1.5より前は、モバイルクライアントとのレプリケーションにSync Gatewayで使用される同期メタデータは、_syncプロパティの一部としてドキュメントに含まれていました。1.5では、同期メタデータは、ドキュメントに関連付けられた拡張属性またはXAttrsに移動されます。

この機能は、同期ゲートウェイ構成ファイルの構成設定を通じて有効にできます。Sync Gateway2.7のEnterpriseEditionを使用している場合、「`import_docs`」フラグはオプションであることに注意してください。「`enable_shared_bucket_access`」が「`true`」に設定されているすべてのノードは、サーバーバケットからドキュメントの変更を自動的にインポートします。

https://github.com/couchbaselabs/mobile-travel-sample/blob/master/sync-gateway-config-travelsample.jsonにあるsync-gateway-config-travelsample.jsonファイルを開きます

```JAVASCRIPT
"import_docs": "true",
"enable_shared_bucket_access": true
```

インポートフィルター機能を定義することにより、Sync Gatewayでインポートおよび処理する必要のあるCouch baseServerドキュメントを指定できます。このデモでは、「user」ドキュメントのみを同期します。したがって、他のすべてのドキュメントタイプは無視されます。

```JAVASCRIPT
function(doc) {
  /* Just ignore all the static travel-sample files */
  if (doc._deleted == true ) {
    return true;
   }
  if (doc.type == "landmark" || doc.type == "hotel" || doc.type == "airport" || doc.type =="airline" || doc.type == "route") {
    return false;
  }

  return true;
}
```

### レプリケーション
レプリケーションは、Couchbase Liteを実行しているクライアントがデータベースの変更をリモート（サーバー）データベースと同期するプロセスです。

プルレプリケーションは、Couchbase Liteダウンロードデータベースを実行しているクライアントがリモート（サーバー）ソースデータベースからローカルターゲットデータベースに変更するプロセスです。

プッシュレプリケーションは、Couchbase Liteアップロードデータベースを実行しているクライアントが、ローカルソースデータベースからリモート（サーバー）ターゲットデータベースに変更するプロセスです。

Couchbase Mobile 2.xレプリケーションプロトコルは、WebSocket上に階層化されたメッセージングプロトコルとして実装されます。

レプリケーション20

レプリケーションプロセスは、「`continuous`(継続的)」または「ワンショット」にすることができます。

「継続的」レプリケーションモードでは、変更はクライアントと同期ゲートウェイの間でリアルタイムで継続的に同期されます。

「ワンショット」モードでは、変更が1回同期され、クライアントとサーバー間の接続が切断されます。将来の変更をプッシュアップまたはプルダウンする必要がある場合、クライアントは新しいレプリケーションを開始する必要があります。

`app/src/android/java/…/util/DatabaseManager.java`ファイルを開きます。
`startPushAndPullReplicationForCurrentUser(String username, String password)`メソッドを確認します。

`DatabaseManager.java`

```JAVA
public static void startPushAndPullReplicationForCurrentUser(String username, String password) {
  ...
}
```

まず、URを指定してL同期するSyncGatewayインスタンスを指すオブジェクトを初期化します。

```JAVA
public static String mSyncGatewayEndpoint = "ws://10.0.2.2:4984/travel-sample";
URI url = null;
try {
    url = new URI(mSyncGatewayEndpoint);
} catch (URISyntaxException e) {
    e.printStackTrace();
}
```

次に、レプリケーションを構成します。`ReplicatorConfigurationSync`のゲートウェイ上のターゲットDBのローカルデータベースとURLで初期化されます。`replicatorTypeReplicator Config`のは、レプリケーションのタイプを指定します。
Travelアプリのコードスニペットでは`pushAndPull`であり、プッシュレプリケーションとプルレプリケーションの両方が有効になっていることを示しています。`continuous`モードがに設定されている`true`トラベルアプリで。


```JAVA
ReplicatorConfiguration config = new ReplicatorConfiguration(database, new URLEndpoint(url));
config.setReplicatorType(ReplicatorConfiguration.ReplicatorType.PUSH_AND_PULL);
config.setContinuous(true);
```

レプリケーターは、関連する認証資格情報で構成されます。同期ゲートウェイとのアクセス同期が許可されているユーザーのリストは、「アクセス制御」セクションで説明されているように作成されます。

```JAVA
config.setAuthenticator(new BasicAuthenticator(username, password));
```

レプリケータは指定された構成で初期化されます

```JAVA
Replicator replicator = new Replicator(config);
```

レプリケーションの変更をリッスンするために、変更リスナーのコールバックブロックが登録されます。プッシュまたはプルの変更があるたびに、コールバックが呼び出されます。

```JAVA
replicator.addChangeListener(new ReplicatorChangeListener() {
@Override
public void changed(ReplicatorChange change) {

    if (change.getReplicator().getStatus().getActivityLevel().equals(Replicator.ActivityLevel.IDLE)) {

        Log.e("Replication Comp Log", "Schedular Completed");

    }
    if (change.getReplicator().getStatus().getActivityLevel().equals(Replicator.ActivityLevel.STOPPED) || change.getReplicator().getStatus().getActivityLevel().equals(Replicator.ActivityLevel.OFFLINE)) {
        // stopReplication();
        Log.e("Rep schedular  Log", "ReplicationTag Stopped");
    }
}
});
```

レプリケーションが開始されます

```JAVA
replicator.start();
```

### 試してみてください（プッシュレプリケーション）
- Travel Sample Mobileアプリに「demo」ユーザーとしてログインし、パスワードを「password」としてログインします。このユーザーは、旅行サンプルWebバックエンドを介して作成する必要があります。

- 「航空会社」ボタンをタップして、フライトを予約します。「出発地」と「目的地」の両方の空港とフライト日はすでに設定されています。

- 「ルックアップ」ボタンをタップします

- フライトのリストから、最初のフライトリストを選択します。これにより、予約が自動的に確認されます。


- Travel Sample Python Webアプリにアクセスします（URLは http：//localhost：8080）。クラウドベースのインストールを行った場合localhostは、URLをWebアプリのクラウドインスタンスのIPアドレスに置き換えてください。

- パスワードを「password」として、「demo」ユーザーとしてWebアプリにログインします。

- 「予約済み」タブを使用して、予約済みのフライトのリストに移動します

- モバイルアプリで予約したフライトがウェブアプリのフライトリストに表示されていることを確認します


### 試してみてください（プルレプリケーション）
- Travel Sample PythonWebアプリにアクセスします（URLはhttp：//localhost：8080）。クラウドベースのインストールを行った場合localhostは、URLをWebアプリのクラウドインスタンスのIPアドレスに置き換えてください。

- パスワードを「password」として、「demo」ユーザーとしてWebアプリにログインします。

- 「フライト」タブをクリックしてフライトを予約します

- 「シアトル」として「From」空港を入力し、ドロップダウンメニューから空港を選択します。

- 「サンフランシスコ」として「To」空港を入力し、ドロップダウンメニューから空港を選択します。

- 出発日と帰国日を入力してください

- 「検索」ボタンをクリックしてください

- フライトのリストから、対応する[バスケットに追加]ボタンをクリックして、最初のフライトリストを選択します

- 「バスケット」タブをクリックしてフライトの選択を表示し、「購入」ボタンをクリックして予約を確認します

- [予約済み]タブには、確認済みのフライト予約が表示されます


- Travel Sample Mobileアプリに「デモ」ユーザーとしてログインし、パスワードを「パスワード」としてログインします

- モバイルアプリのフライトリストに、ウェブアプリで予約したフライトが表示されていることを確認します
