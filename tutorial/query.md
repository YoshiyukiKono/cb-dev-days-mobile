## クエリ
### 概要
Couchbase Lite 2.0には、クエリインターフェイスのようなN1QLのサポートが含まれています。データベースは、クエリビルダーを使用してクエリを作成し、そのクエリを実行することでクエリできます。

Couchbase Lite 2.0のクエリインターフェイスは強力で、とりわけ以下のサポートが含まれています

- パターンマッチング
- 正規表現マッチング
- 数学関数
- 文字列操作関数
- 集計関数
- グループ化
- 結合（単一のデータベース内）
- 並べ替え
- NilOrMissingプロパティ

### 単純なクエリ
旅行アプリには、データベースにクエリを実行するインスタンスが多数あります。ここでは簡単な例について説明します。

`app/src/android/java/…/searchflight/SearchFlightPresenter.java`ファイルを開きます。`startsWith(String prefix, String tag)`メソッドを確認します。

`SearchFlightPresenter.java`

```java
@Override
public void startsWith(String prefix, String tag) {
  ...
}
```

以下のクエリは、データベースからドキュメント内の「name」プロパティを選択します。ここで、typeプロパティはairportと等しく、「airportname」プロパティは検索語と等しくなります。

```java
Database database = DatabaseManager.getDatabase();
Query searchQuery = QueryBuilder
  .select(SelectResult.expression(Expression.property("airportname")))
  .from(DataSource.database(database))
  .where(
    Expression.property("type").equalTo(Expression.string("airport"))
      .and(Expression.property("airportname").like(Expression.string(prefix + "%")))
);
```

次に、execute()メソッドを使用してクエリが実行されます。結果の各行には、「airportname」と呼ばれる単一のプロパティが含まれます。最終結果はshowAirports、結果がに表示されるメソッドに渡されRecyclerViewます。

```java
ResultSet rows = null;
try {
    rows = searchQuery.execute();
} catch (CouchbaseLiteException e) {
    Log.e("app", "Failed to run query", e);
    return;
}

Result row;
List<String> data = new ArrayList<>();
while ((row = rows.next()) != null) {
    data.add(row.getString("airportname"));
}
mSearchView.showAirports(data, tag);
```
  
### やってみよう
- Travel Sampleモバイルアプリに「demo」ユーザーとしてログインし、パスワードを「password」としてログインします
- 「フライト」ボタンをタップしてフライトを予約します
- 「From」空港のテキストフィールドに「Detroit」と入力します
- ドロップダウンリストの最初の項目が「DetroitMetroWayneCo」であることを確認します

![](https://cl.ly/0b3q2T2t1R1J/android-simple-query.gif)

### 高度なクエリ
  
このセクションでは、JOINクエリについて説明します。Couchbase Lite 2.0のJOINクエリは、データベース内の結合です。

「データモデリング」セクションを思い出すと、「bookmarkedhotels」に等しいタイプのドキュメントには、ブックマークされたホテルのIDの配列であるhotelsプロパティが含まれています。

```
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

\_idタイプ「bookmarkedhotels」のドキュメントの「hotels」プロパティ配列に含まれているドキュメントをフェッチするクエリを確認します。

`app/src/android/java/…/hotes/BookmarksPresenter.java`ファイルを開きます。`fetchBookmarks()`メソッドを確認します。

`BookmarksPresenter.java`

```java
public void fetchBookmarks() {
  ...
}
```

まず、結合クエリの両側に対応する2つのデータソースをインスタンス化します。

```java
DataSource bookmarkDS = DataSource.database(database).as("bookmarkDS");
DataSource hotelsDS = DataSource.database(database).as("hotelDS");
```

次に、クエリ式を記述します。1つ目は、hotelsブックマークデータソースのプロパティを取得します。秒は、ホテルのデータソースのドキュメントIDを取得します。

```java
Expression hotelsExpr = Expression.property("hotels").from("bookmarkDS");
Expression hotelIdExpr = Meta.id.from("hotelDS");
```

次に、関数式を使用して、`_id`プロパティが`hotels`配列内にあるドキュメントを検索します。そして、結合式を作成します。

```java
Expression joinExpr = ArrayFunction.contains(hotelsExpr, hotelIdExpr);
Join join = Join.join(hotelsDS).on(joinExpr);
```

最後に、クエリオブジェクトはその結合式を使用して、ブックマークドキュメントの「hotels」配列で参照されているすべてのホテルドキュメントを検索します。

```java
Expression typeExpr = Expression.property("type").from("bookmarkDS");

SelectResult bookmarkAllColumns = SelectResult.all().from("bookmarkDS");
SelectResult hotelsAllColumns = SelectResult.all().from("hotelDS");

Query query = QueryBuilder
  .select(bookmarkAllColumns, hotelsAllColumns)
  .from(bookmarkDS)
  .join(join)
  .where(typeExpr.equalTo(Expression.string("bookmarkedhotels")));
```

この`execute()`メソッドを使用して結果を取得し、ビューに渡します。

```java
query.addChangeListener(new QueryChangeListener() {
    @Override
    public void changed(QueryChange change) {
        ResultSet rows = change.getRows();

        List<Map<String, Object>> data = new ArrayList<>();
        Result row = null;
        while((row = rows.next()) != null) {
            Map<String, Object> properties = new HashMap<>();
            properties.put("name", row.getDictionary("hotelDS").getString("name"));
            properties.put("address", row.getDictionary("hotelDS").getString("address"));
            properties.put("id", row.getDictionary("hotelDS").getString("id"));
            data.add(properties);
        }
        mBookmarksView.showBookmarks(data);
    }
});

try {
    query.execute();
} catch (CouchbaseLiteException e) {
    e.printStackTrace();
}
```

### やってみよう

- 「ゲストとして続行」を選択して、「guest」ユーザーとして旅行サンプルモバイルアプリにログインします
- 「ホテル」ボタンをタップ
- [場所]テキストフィールドに「ロンドン」と入力します
- 「説明」テキストフィールドに「ペット」と入力します
- 「ノボテルロンドンウエスト」がリストされていることを確認します
- タップしてホテルを「ブックマーク」します
- ノボテルホテルが「BookmarksActivity」ページのリストに表示されていることを確認します

![](https://cl.ly/3r243s1K2600/android-advanced-query.gif)

[目次へ戻る](./README.md)
