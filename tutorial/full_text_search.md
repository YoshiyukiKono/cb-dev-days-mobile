## 全文検索

### 概要
Couchbase Lite 2.0は、全文検索（FTS）をサポートするようになりました。FTSは、`match`クエリを使用して実行されます。
FTSの一致はケースセンシティブです。

Tracelアプリでは、FTSクエリは、アプリで事前に作成されたローカルの「travel-sample」データベースのドキュメントに対して行われます。

### インデックス作成

FTSクエリを実行するには、FTSインデックスを作成する必要があります。

`app/src/android/java/…/util/DatabaseManager.java`ファイルを開きます。`createFTSQueryIndex()`メソッドを確認します。

以下のコードスニペットは、`description`という名前のプロパティにFTSインデックスを作成します。


```JAVA
private void createFTSQueryIndex() {
    try {
        database.createIndex("descFTSIndex", IndexBuilder.fullTextIndex(FullTextIndexItem.property("description")));
    } catch (CouchbaseLiteException e) {
        e.printStackTrace();
    }
}
```

次に、インデックスを使用するFTSクエリを記述します。

`app/src/android/java/…/hotels/HotelsPresenter.java`ファイルを開きます。`queryHotels(String location, String description)`メソッドを確認します。

`HotelsPresenter.java`

```JAVA
@Override
public void queryHotels(String location, String description) {
  ...
}
```

まず、データベースのインスタンスを取得します。

```JAVA
Database database = DatabaseManager.getDatabase();
```

次に、`match()`オペレーターを使って、`FullTextExpression`を作成します。この例では、`match`式は`description`プロパティ内の値から`desciptionStr`を検索します。
この`match`式は、`country`、`city`、`state`または`address`プロパティから、`location`を探す`like`比較式と、論理的に`AND`で結合されます。
この式は、通常の方法でクエリの`where`句で使用されます。

```JAVA
Expression descExp = FullTextExpression.index("descFTSIndex").match(description) ;


Expression locationExp = Expression.property("country")
  .like(Expression.string("%" + location + "%"))
  .or(Expression.property("city").like(Expression.string("%" + location + "%")))
  .or(Expression.property("state").like(Expression.string("%" + location + "%")))
  .or(Expression.property("address").like(Expression.string("%" + location + "%")));

Expression searchExp = descExp.and(locationExp);

Query hotelSearchQuery = QueryBuilder
  .select(SelectResult.all())
  .from(DataSource.database(database))
  .where(
      Expression.property("type").equalTo(Expression.string("hotel")).and(searchExp)
);
```

上記のクエリを実行し、結果（`ResultSet`）を`mHotelView`オブジェクトに渡される`List<Map<String, Object>>`オブジェクトに変換します。

```JAVA
ResultSet rows = null;
try {
    rows = hotelSearchQuery.execute();
} catch (CouchbaseLiteException e) {
    e.printStackTrace();
    return;
}

List<Map<String, Object>> data = new ArrayList<Map<String, Object>>();
Result row = null;
while((row = rows.next()) != null) {
    Map<String, Object> properties = new HashMap<String, Object>();
    properties.put("name", row.getDictionary("travel-sample").getString("name"));
    properties.put("address", row.getDictionary("travel-sample").getString("address"));
    data.add(properties);
}
mHotelView.showHotels(data);
```

### やってみよう
- Travel Sample Mobileアプリに「demo」ユーザーとしてログインし、パスワードを「password」としてログインします
- 「ホテル」ボタンをタップします
- 「説明」テキストフィールドに「pet」と入力します。
- 「場所]」テキストフィールドに「London」と入力します
- 「Novotel London West」という名前のホテルが1つリストされていることを確認します

![](https://cl.ly/3r243s1K2600/android-advanced-query.gif)

[目次へ戻る](./README.md)
