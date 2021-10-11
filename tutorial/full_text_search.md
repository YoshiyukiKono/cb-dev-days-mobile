## 全文検索
### 全文検索
Couchbase Lite 2.0は、全文検索（FTS）をサポートするようになりました。FTSは、`match`クエリを使用して実行されます。
FTSの一致はケースセンシティブです。旅行アプリでは、FTSクエリは、アプリで事前に作成されたローカルの「旅行サンプル」ドキュメントに対して行われます。

FTSクエリを実行するには、FTSインデックスを作成する必要があります。

`app/src/android/java/…/util/DatabaseManager.java`ファイルを開きます。
`createFTSQueryIndex()`メソッドを確認します。
このコードスニペットは、`description`という名前のプロパティにFTSインデックスを作成します。

`DatabaseManager.java`

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

次に、演算子Expressionsを使用してFTSを作成しますmatch()。この特定の例では、match式desciptionStrはdescriptionプロパティ内の値を検索します。このmatch式は、論理的にANDさequalToを探し比較式locationでcountry、city、stateまたはaddressプロパティ。この式はwhere、通常の方法でクエリの句で使用されます。

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

上記のさまざまな式を使用してクエリを作成ResultSetし、List<Map<String, Object>>オブジェクトをに渡されるオブジェクトに変換しますRecyclerView。

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
- Travel Sample Mobileアプリに「デモ」ユーザーとしてログインし、パスワードを「パスワード」としてログインします
- 「ホテル」ボタンをタップします
- 説明テキストフィールドに「 `ペット」と入力します。
- [場所]テキストフィールドに「ロンドン」と入力します
- 「ノボテルロンドンウエスト」という名前のホテルが1つリストされていることを確認します

