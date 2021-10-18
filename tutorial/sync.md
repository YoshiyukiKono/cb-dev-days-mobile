## 同期

[オリジナル英文](https://docs.couchbase.com/tutorials/mobile-travel-sample/android/develop/sync.html)

### チャネル/データルーティング

Sync Gateway構成ファイルは、サーバー構成や、Sync Gatewayインスタンスが対話できるデータベース(のセット)など、Sync Gatewayの実行時の動作を決定します。

Sync Gatewayはチャネルを使用して、多数のユーザー間でデータベースを共有し、データベースへのアクセスを制御できるようにします。
概念的には、チャネルはタグと見なすことができます。データベース内のすべてのドキュメントは一連のチャネルに属し、ユーザーには一連のチャネルへの適切なアクセスが許可されます。チャネルは次の目的で使用されます。

- データセットを分割します。
- ユーザーにドキュメントへのアクセスを許可します。
- デバイスに同期されるデータの量を最小限に抑えます。

`sync-gateway-config-travelsample.json`ファイルを開きます。

`"sync"`属性にJavaScript関数（`function`）`sync`のコードが含まれています。

```JAVASCRIPT
/* Routing */
// Add doc to the user's channel.
channel("channel." + username);
```

### 共有バケットアクセス
このレッスンを開始する前に、Sync Gatewayのインストールセクションの手順に従って、Sync　Gatewayが稼働していることを確認してください。

モバイルアプリケーションとサーバー/Webアプリケーションは同じバケットに対して読み取りと書き込みを行うことができます(Sync Gateway1.5およびCouchbase Server 5.0以降)。
これは、Sync　Gateway構成ファイルで有効にできるオプトイン機能です。

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/convergence.png)

同期メタデータは、ドキュメントに関連付けられた拡張属性またはXAttrsに格納されます（1.5より前は、`_sync`プロパティの一部としてドキュメントに含まれていました）。

この機能は、Sync Gateway構成ファイルの構成設定を通じて有効にできます。
「`enable_shared_bucket_access`」が「`true`」に設定されているすべてのノードは、サーバーバケットからドキュメントの変更を自動的にインポートします。
(Sync Gateway2.7のEnterprise Editionを使用している場合、「`import_docs`」フラグはオプションです。)


sync-gateway-config-travelsample.jsonファイルを開きます

```JAVASCRIPT
"import_docs": "true",
"enable_shared_bucket_access": true
```

インポートフィルター機能を定義することにより、Sync Gatewayでインポートおよび処理する必要のあるCouchbase Serverドキュメントを指定できます。
このアプリでは、「user」ドキュメントのみを同期します。したがって、他のすべてのドキュメントタイプは無視されます。

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

- プルレプリケーションは、Couchbase Liteデータベースを実行しているクライアントがリモート（サーバー）ソースデータベースからローカルターゲットデータベースに変更をダウンロードするプロセスです。

- プッシュレプリケーションは、Couchbase Liteデータベースを実行しているクライアントが、ローカルソースデータベースからリモート（サーバー）ターゲットデータベースに変更をアップロードするプロセスです。

Couchbase Mobile 2.xレプリケーションプロトコルは、WebSocket上に階層化されたメッセージングプロトコルとして実装されています。

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/replication-2-0.png)

レプリケーションプロセスは、「`continuous`(継続的)」または「`one shot`(ワンショット)」にすることができます。

- 「継続的」レプリケーションモードでは、変更はクライアントと同期ゲートウェイの間でリアルタイムで継続的に同期されます。

- 「ワンショット」モードでは、変更が1回同期され、クライアントとサーバー間の接続が切断されます。将来の変更をプッシュアップまたはプルダウンする必要がある場合、クライアントは新しいレプリケーションを開始する必要があります。

`app/src/android/java/…/util/DatabaseManager.java`ファイルを開きます。
`startPushAndPullReplicationForCurrentUser(String username, String password)`メソッドを確認します。

`DatabaseManager.java`

```JAVA
public static void startPushAndPullReplicationForCurrentUser(String username, String password) {
  ...
}
```

まず、URLを指定して同期するSync Gatewayインスタンスを指すオブジェクトを初期化します。

```JAVA
public static String mSyncGatewayEndpoint = "ws://10.0.2.2:4984/travel-sample";
URI url = null;
try {
    url = new URI(mSyncGatewayEndpoint);
} catch (URISyntaxException e) {
    e.printStackTrace();
}
```

次に、レプリケーションを構成します。`ReplicatorConfiguration`を、ローカルデータベースとターゲットDBのURLで初期化します。
さらに、レプリケーションのタイプを指定します。
Travelアプリのコードスニペットでは`pushAndPull`であり、プッシュレプリケーションとプルレプリケーションの両方が有効になっていることを示しています。
また、`continuous`モードが`true`に設定されています。


```JAVA
ReplicatorConfiguration config = new ReplicatorConfiguration(database, new URLEndpoint(url));
config.setReplicatorType(ReplicatorConfiguration.ReplicatorType.PUSH_AND_PULL);
config.setContinuous(true);
```

レプリケーターには、認証資格情報が設定されます。

```JAVA
config.setAuthenticator(new BasicAuthenticator(username, password));
```

レプリケータは指定された構成で初期化されます。

```JAVA
Replicator replicator = new Replicator(config);
```

レプリケーションの変更をリッスンするために、チェンジリスナーのコールバックブロックが登録されます。プッシュまたはプルの変更があるたびに、コールバックが呼び出されます。

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

レプリケーションを開始します。

```JAVA
replicator.start();
```

### やってみよう（プッシュレプリケーション）
- Travel Sample Mobileアプリに「demo」ユーザーとしてログインし、パスワードを「password」としてログインします。このユーザーは、旅行サンプルWebバックエンドを介して作成する必要があります。

- 「航空会社」ボタンをタップして、フライトを予約します。「出発地」と「目的地」の両方の空港とフライト日はすでに設定されています。

- 「ルックアップ」ボタンをタップします

- フライトのリストから、最初のフライトリストを選択します。これにより、予約が自動的に確認されます。


- Travel Sample Python Webアプリにアクセスします（URLは http：//localhost：8080）。必要に応じ、URLのIPアドレスを置き換えてください。

- 「demo」ユーザーとして、パスワードに「password」を使い、Webアプリにログインします。

- 「予約済み」タブを使用して、予約済みのフライトのリストに移動します

- モバイルアプリで予約したフライトがウェブアプリのフライトリストに表示されていることを確認します


### やってみよう（プルレプリケーション）
- Travel Python Webアプリにアクセスします（URLはhttp：//localhost：8080）。必要に応じ、URLのIPアドレスを置き換えてください。

- 「demo」ユーザーとして、パスワードに「password」を使い、Webアプリにログインします。

- 「Flights」タブをクリックしてフライトを予約します

- 「Seattle」として「From」空港を入力し、ドロップダウンメニューから空港を選択します。

- 「San Francisco」として「To」空港を入力し、ドロップダウンメニューから空港を選択します。

- 出発日と帰国日を入力します(任意の日付を用いることが可能です)。

- 「Search」ボタンをクリックします。

- フライトのリストから、対応する「Add to Basket」ボタンをクリックして、最初のフライトリストを選択します。

- 「Basket」タブをクリックしてフライトの選択を表示し、「Buy」ボタンをクリックして予約を確認します。

- 「Booked」タブには、確認済みのフライト予約が表示されます。

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/travel-app-pull.gif)

- Travel Sample Mobileアプリに、「demo」ユーザーとして、パスワードに「password」を使い、ログインします

- モバイルアプリのフライトリストに、ウェブアプリで予約したフライトが表示されていることを確認します

[目次へ戻る](./README.md)
