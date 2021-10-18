## 同期

[オリジナル英文](https://docs.couchbase.com/tutorials/mobile-travel-sample/android/develop/sync.html)

### チャネルによるデータルーティング

Sync Gateway構成ファイルは、Sync Gatewayの実行時の動作を決定します。

Sync Gatewayはチャネルを使用して、多数のユーザー間でデータベースを共有し、データベースへのアクセスを制御できるようにします。
概念的には、チャネルはタグと見なすことができます。データベース内のすべてのドキュメントは一連のチャネルに属し、ユーザーに対して、チャネルへのアクセスが許可/拒否されます。
チャネルの目的は、以下のように整理することができます。

- データセットを分割する。
- ユーザーにドキュメントへのアクセスを許可する。
- デバイスに同期されるデータの量を最小限に抑える。

`sync-gateway-config-travelsample.json`ファイルを開きます。

`"sync"`属性にJavaScript関数（`function`）`sync`のコードが含まれています。

`channel`の利用例を以下に示します。

```JAVASCRIPT
/* Routing */
// Add doc to the user's channel.
channel("channel." + username);
```

### 共有バケットアクセス

モバイルアプリケーションとWebアプリケーションは同じバケットに対して読み取りと書き込みを行うことができます(Sync Gateway1.5およびCouchbase Server 5.0以降)。

これは、Sync　Gateway構成ファイルで有効にできるオプトイン機能です。

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/convergence.png)

同期メタデータは、ドキュメントに関連付けられた拡張属性/XAttrsに格納されます（1.5より前は、`_sync`プロパティの一部としてドキュメントに含まれていました）。

この機能は、Sync Gateway構成ファイルの設定を通じて有効にできます。
「`enable_shared_bucket_access`」が「`true`」に設定することで有効にされます。

関連するオプションとして、「`import_docs`」フラグがあります。


sync-gateway-config-travelsample.jsonファイルを開きます

```JAVASCRIPT
"import_docs": "true",
"enable_shared_bucket_access": true
```

インポートフィルター機能を定義することにより、Sync Gatewayでインポートおよび処理するCouchbase Serverドキュメントを指定できます。
このアプリでは、「user」ドキュメントのみを同期します。したがって、他のすべてのドキュメントタイプは無視します。

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
Travelアプリでは`pushAndPull`が用いられ、プッシュレプリケーションとプルレプリケーションの両方が有効になっていることを示しています。
また、`continuous`モードが`true`に設定されています。


```JAVA
ReplicatorConfiguration config = new ReplicatorConfiguration(database, new URLEndpoint(url));
config.setReplicatorType(ReplicatorConfiguration.ReplicatorType.PUSH_AND_PULL);
config.setContinuous(true);
```

以下のように、認証資格情報が設定されます。

```JAVA
config.setAuthenticator(new BasicAuthenticator(username, password));
```

ここまで行った構成で、レプリケータを初期化します。

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
- Travel Sampleモバイルアプリに「demo」ユーザーとしてログインし、パスワードを「password」としてログインします。このユーザーは、Travel Webバックエンドを介して作成されている必要があります。

- 「airline」ボタンをタップして、フライトを予約します。「From(出発地)」と「To(目的地)」の両方の空港とフライト日は初期値のままにしておきます。

- 「lookup」ボタンをタップします

- フライトのリストから、最初のフライトリストを選択します。これにより、予約が行われます。

- Travel Python Webアプリにアクセスします（URLは http：//localhost：8080）。必要に応じ、URLのIPアドレスを置き換えてください。

- 「demo」ユーザーとして、パスワードに「password」を使い、Webアプリにログインします。

- 「Booked」タブを使用して、予約済みのフライトのリストに移動します

- モバイルアプリで予約したフライトがウェブアプリのフライトリストに表示されていることを確認します


### やってみよう（プルレプリケーション）
- Travel Python Webアプリにアクセスします（URLはhttp：//localhost：8080）。必要に応じ、URLのIPアドレスを置き換えてください。

- 「demo」ユーザーとして、パスワードに「password」を使い、Webアプリにログインします。

- 「Flights」タブをクリックしてフライトを予約します

- 「From（出発地）」空港として「Seattle」を入力し、ドロップダウンメニューから空港を選択します。

- 「To（目的地）」空港として「To」空港「San Francisco」を入力し、ドロップダウンメニューから空港を選択します。

- 出発日と帰国日を入力します(任意の日付を用いることが可能です)。

- 「Search」ボタンをクリックします。

- フライトのリストから、対応する「Add to Basket」ボタンをクリックして、最初のフライトリストを選択します。

- 「Basket」タブをクリックしてフライトの選択を表示し、「Buy」ボタンをクリックして予約を確認します。

- 「Booked」タブには、確認済みのフライト予約が表示されます。

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/travel-app-pull.gif)

- Travel Sampleモバイルアプリに、「demo」ユーザーとして、パスワードに「password」を使い、ログインします

- モバイルアプリのフライトリストに、ウェブアプリで予約したフライトが表示されていることを確認します

[目次へ戻る](./README.md)
