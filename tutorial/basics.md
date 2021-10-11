## 基礎
### 初期化
開始するには、関連するアプリケーションコンテキストでCouchbaseLiteを初期化する必要があり ます DatabaseManager.java。ファイルを開きます。 DatabaseManager.java

```JAVA
  public void initCouchbaseLite(Context context) {
      CouchbaseLite.init(context);
      appContext = context;
  }
```

### データベースを作成する
デバイス上で作成または開くことができるデータベースの数に制限はありません。データベースはドキュメントの名前空間と考えることができ、同じアプリで複数のデータベースを使用できます（アプリのユーザーごとに1つのデータベースが一般的なパターンです）。

以下のスニペットは、guestという名前のディレクトリにユーザー用の空のデータベースを作成しますguest。

`DatabaseManager.java`ファイルを開きます 。`OpenGuestDatabase` 方法を見直します。

`DatabaseManager.java`

```JAVA
 public void OpenGuestDatabase() {
   ...
 }
```

私たちは用のフォルダの作成guest1が存在していることを指定しない場合は、ユーザーデータベースのデータベースなどdirectoryでDatabaseConfigurationオブジェクト。

```JAVA
  DatabaseConfiguration config = new DatabaseConfiguration();
  config.setDirectory(String.format("%s/guest", appContext.getFilesDir()));
```

Couchbase Liteデータベースは、指定された名前とDatabaseConfigurationオブジェクトで作成されます。

```JAVA
 try {
      database = new Database("guest", config);
  } catch (CouchbaseLiteException e) {
      e.printStackTrace();
  }
```

#### やってみよう
旅行サンプルモバイルアプリを作成して実行する

ログイン画面で、「ゲストとして続行」オプションを選択します。

これにより、ゲストモードでアプリにログインします。ゲストとしてサインインすると、「ゲスト」アカウントの新しい空のデータベースが作成されます（データベースが存在しない場合）。

「BookmarksActivity」ページが表示されていることを確認します。初めて空になります。

### ドキュメントの作成と更新
ブックマークされたホテルは、のtypeが付いた別のドキュメントに保持されbookmarkedhotelsます。

ホテルが初めてブックマークbookmarkedhotelsされると、hotelsプロパティ内のそのホテルドキュメントのドキュメントIDを使用してドキュメントが作成されます。ホテルの情報は、別のhotelsタイプのドキュメントに保持されます。

その後、ホテルがブックマークされるたびに、このプロセスが繰り返されます。

```JSON
{
  "_id": "hotel1",
  "name": "San Francisco Hotel",
  "address": "123, Park Street, San Francisco"
}

{
  "type": "bookmarkedhotels",
  "hotels": ["hotel1", "hotel2"]
}
```

`app/src/android/java/…/hotels/HotelsPresenter.java`ファイルを開きます 。`bookmarkHotels(Map<String, Object> hotel)`メソッドを確認します。

`HotelsPresenter.java`

```JAVA
@Override
public void bookmarkHotels(Map<String, Object> hotel) {
  ...
}
```

まず、データベースのインスタンスを取得する必要があります。

```JAVA
Database database = DatabaseManager.getDatabase();
```

次のスニペットは、ホテルインスタンス（Map<String, Object>）をDocumentデータベース内の新しいものとして永続化します。これにより、オフラインでブックマークされたホテルのドキュメントにアクセスできるようになります。

```JAVA
MutableDocument hotelDoc = new MutableDocument((String) hotel.get("id"), hotel);
try {
  database.save(hotelDoc);
} catch (CouchbaseLiteException e) {
  e.printStackTrace();
}
```

次に、IDを持つドキュメントを取得するか、user::guest存在しない場合は作成します。ドキュメントは、typeプロパティをに設定し、ブックマークされたホテルのドキュメントIDを格納するbookmarkedhotelsための新しいhotels配列を使用して作成されます。

```JAVA
Document document = database.getDocument("user::guest");
MutableDocument mutableCopy = null;
if (document == null) {
	HashMap<String,Object> properties = new HashMap();
	properties.put(type, bookmarkedhotels);
	properties.put(hotels, new ArrayList());
	mutableCopy = new MutableDocument(user::guest, properties);
} else {
	mutableCopy = document.toMutable();
}
```

次に、選択したホテルのIDがhotelsアレイに追加されます。

```JAVA
MutableArray hotels =  mutableCopy.getArray(hotels).toMutable();
mutableCopy.setArray(hotels,hotels.addString((String) hotel.get(id)));
```

最後に、ドキュメントを保存します。

```JAVA
try {
  database.save(mutableCopy);
} catch (CouchbaseLiteException e) {
  e.printStackTrace();
}
```

#### やってみよう
ゲストユーザーとして、「ホテル」ボタンをタップします。

「場所」のテキストフィールドに、「ロンドン」と入力し始めたかのように「L」を入力します。ホテルのリストが表示されます。

ホテルのリストは、Travel Sample Web ServicesAPIを介してCouchbaseServerから取得されます。Python Webアプリへの接続が開いていて、全文検索インデックスがCouchbase Serverで作成されていない限り、検索結果は表示されません。

最初のホテルのセルをタップしてブックマークします。

「BookmarksActivity」画面にブックマークされたホテルが表示されていることを確認します。ブックマークされたホテルごとに個別のドキュメントを用意する動機は、同期機能を介してユーザー間でドキュメントを共有できるようになるかどうかです。

アンドロイド保存ドキュメント
### ドキュメントを削除する
このdelete方法を使用して、ドキュメントを削除できます。この操作はtombstoned、削除を他のクライアントに伝播するために、実際に新しいリビジョンを作成します。

`app/src/android/java/…/bookmarks/BookmarksPresenter.java`ファイルを開きます。`removeBookmark(Map<String, Object> bookmark)`メソッドを確認します。

ブックマークプレゼンター

```JAVA
@Override
public void removeBookmark(Map<String, Object> bookmark) {
    ...
}
```

ゲストモードでホテルを検索すると、アプリはGETリクエストをPython Webアプリに送信し、Python WebアプリはCouchbase Serverで全文検索クエリを実行します。
次に、ホテルがブックマークされている場合、オフラインアクセスのためにCouchbaseLiteデータベースに挿入されます。したがって、ユーザーがホテルのブックマークを解除するときは、ドキュメントをデータベースから削除する必要があります。それが以下のコードが行っていることです。

```JAVA
Database database = DatabaseManager.getDatabase();
Document document = database.getDocument((String) bookmark.get("id"));
try {
  database.delete(document);
} catch (CouchbaseLiteException e) {
  e.printStackTrace();
}
```

上記のようにタイプ「hotel」のドキュメントを削除することに加えて、ブックマーク解除プロセスはhotels「bookmarkedhotels」ドキュメントの配列からホテルIDを削除します。

#### やってみよう
最初のホテルセルを左にスワイプして、セルのブックマークを解除/削除します

リストに1つのホテルが表示されていることを確認します

android unbookmark
