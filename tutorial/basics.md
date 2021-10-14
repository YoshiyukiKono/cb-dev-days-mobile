## データベース操作の基本

### 初期化
データベースの利用を開始するには、関連するアプリケーションコンテキストでCouchbase Liteを初期化する必要があります。 確認のため、`DatabaseManager.java`ファイルを開きます。 


```JAVA
  public void initCouchbaseLite(Context context) {
      CouchbaseLite.init(context);
      appContext = context;
  }
```

### データベース作成

デバイス上で作成またはオープンすることができるデータベースの数に制限はありません。
データベースはドキュメントの名前空間と考えることができ、同じアプリで複数のデータベースを使用できます（アプリのユーザーごとに1つのデータベースが一般的なパターンです）。

以下のスニペットは、`guest`という名前のディレクトリにゲストユーザー用の空のデータベースを作成します。

`DatabaseManager.java`ファイルを開きます 。`OpenGuestDatabase`メソッドを確認します。

`DatabaseManager.java`

```JAVA
 public void OpenGuestDatabase() {
   ...
 }
```

`DatabaseConfiguration`オブジェクトを作成し、`guest`ディレクトリーを設定します。

```JAVA
  DatabaseConfiguration config = new DatabaseConfiguration();
  config.setDirectory(String.format("%s/guest", appContext.getFilesDir()));
```

Couchbase Liteデータベースは、指定された名前と`DatabaseConfiguration`オブジェクトで作成されます。

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

これにより、ゲストモードでアプリにログインします。ゲストとしてサインインすると、「guest」アカウントの新しい空のデータベースが作成されます（データベースが存在しない場合）。

「BookmarksActivity」ページが表示されていることを確認します。初めて空になります。

### ドキュメントの作成と更新

ブックマークされたホテルは、`bookmarkedhotels`という`type`のドキュメントに保持されます。

ホテルが初めてブックマークされると、`hotels`プロパティ内のそのホテルドキュメントのドキュメントIDを使用して`bookmarkedhotels`ドキュメントが作成されます。ホテルの情報は、別の`hotels`タイプのドキュメントに保持されます。

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


```JAVA
@Override
public void bookmarkHotels(Map<String, Object> hotel) {
  ...
}
```

まず、データベースのインスタンスを取得します。

```JAVA
Database database = DatabaseManager.getDatabase();
```

次のスニペットは、「ホテル」ドキュメントのインスタンス（インスタンスのは、キーと、JSONドキュメントに対応する`Map<String, Object>`構造を持ちます）をデータベース内の新しいドキュメントとして永続化します。
これにより、オフラインでブックマークされたホテルのドキュメントにアクセスできるようになります。

```JAVA
MutableDocument hotelDoc = new MutableDocument((String) hotel.get("id"), hotel);
try {
  database.save(hotelDoc);
} catch (CouchbaseLiteException e) {
  e.printStackTrace();
}
```

次に、IDを持つドキュメントを取得するか、存在しない場合は作成します（ID/キー：`user::guest`）。
ドキュメントは、`type`プロパティを「`"bookmarkedhotels"`」に設定し、ブックマークされたホテルのドキュメントIDを格納するのための新しい`hotels`配列を作成されます。

```JAVA
if (document == null) {
    HashMap<String, Object> properties = new HashMap<>();
    properties.put("type", "bookmarkedhotels");
    properties.put("hotels", new ArrayList<>());
    mutableCopy = new MutableDocument("user::guest", properties);
}
else {
    mutableCopy = document.toMutable();
}
```

次に、選択したホテルのIDが`hotels`配列に追加されます。

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
- ゲストユーザーとして、「ホテル」ボタンをタップします。

- 「場所」のテキストフィールドに、「ロンドン」と入力し始めたかのように「L」を入力します。ホテルのリストが表示されます。

- ホテルのリストは、Travel Sample WebサービスAPIを介してCouchbase Serverから取得されます。Python Webアプリへの接続ができなかったり、Couchbase Serverで全文検索インデックスが作成されていない場合、検索結果は表示されません。

- 最初のホテルのセルをタップしてブックマークします。

- 「BookmarksActivity」画面にブックマークされたホテルが表示されていることを確認します。ブックマークされたホテルごとに個別のドキュメントを用意する動機は、同期機能を介してユーザー間でドキュメントを共有できるようにすることです。

![](https://cl.ly/1t38050A1T40/android-save-doc.gif)

### ドキュメントを削除する

`delete`メソッドを使用して、ドキュメントを削除できます。この操作は、削除を他のクライアントに伝播するために、内部的に、`tombstone`(墓石)としてマークされた新しいリビジョンを作成します。

`app/src/android/java/…/bookmarks/BookmarksPresenter.java`ファイルを開きます。`removeBookmark(Map<String, Object> bookmark)`メソッドを確認します。

ブックマークプレゼンター

```JAVA
@Override
public void removeBookmark(Map<String, Object> bookmark) {
    ...
}
```

ゲストモードでホテルを検索すると、アプリはGETリクエストをPython Webアプリに送信し、Python WebアプリはCouchbase Serverで全文検索クエリを実行します。
次に、ホテルがブックマークした場合、オフラインアクセスのためにCouchbase　Liteデータベースに挿入されます。したがって、ユーザーがホテルのブックマークを解除するときは、ドキュメントをデータベースから削除する必要があります。それを以下のコードで行っています。

```JAVA
Database database = DatabaseManager.getDatabase();
Document document = database.getDocument((String) bookmark.get("id"));
try {
  database.delete(document);
} catch (CouchbaseLiteException e) {
  e.printStackTrace();
}
```

ブックマーク解除プロセスは、タイプ「hotel」のドキュメントを削除することに加えて、「bookmarkedhotels」ドキュメントの配列「hotels」からホテルIDを削除します。

#### やってみよう

- 「ホテル」セルを左にスワイプして、セルのブックマークを解除/削除します。

![](https://cl.ly/0A0D363w3R1g/android-unbookmark.gif)

[目次へ戻る](./README.md)
