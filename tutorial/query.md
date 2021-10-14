## クエリ
### 概要
Couchbase Liteでは、データベースに対するクエリを利用することができます。

クエリの構造は、SQLに類似しています。ただし、モバイルアプリケーションという性格上、クエリは文字列で表現するのではなく、オブジェクトとして構築します。
クエリの構築には、ビルダークラス（`QueryBuilder`）を使用します。

注：一般に、クエリが文字列として扱われる場合には、文字列を解析して実行するための機能を持つか、プログラミング言語のコンパイラーがソースコードを解析する前に、クエリ文字列をプログラム言語に書き換える（プリコンパイルする）必要があります。モバイルアプリが前者の機能をカバーすることは、フットプリント(アプリのサイズ)を考えると不適切であることがわかります。後者の機能は、提供されていません。

Couchbase Lite 2.0のクエリインターフェイスには、以下の機能が含まれています

- パターンマッチング
- 正規表現マッチング
- 数学関数
- 文字列操作関数
- 集計関数
- グループ化
- 結合（単一のデータベース内）
- 並べ替え
- `NilOrMissing`プロパティ

### 単純なクエリ
Travel Sampleアプリには、データベースにクエリを実行している箇所が多数あります。ここでは簡単な例について説明します。

`app/src/android/java/…/searchflight/SearchFlightPresenter.java`ファイルを開きます。`startsWith(String prefix, String tag)`メソッドを確認します。

```java
@Override
public void startsWith(String prefix, String tag) {
  ...
}
```

以下のクエリは、データベースから、`type`プロパティが`airport`と等しく、`airportname`プロパティが検索語と等しい、ドキュメントの`airportname`プロパティを取得しています。

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

次に、execute()メソッドを使用してクエリが実行されます。結果の各行には、`airportname`と呼ばれるプロパティの値が含まれます。最終結果は、結果表示のために`mSearchView`の`showAirports`メソッドに渡されます。

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
- ドロップダウンリストの最初の項目が「Detroit　Metro　Wayne　Co」であることを確認します

![](https://cl.ly/0b3q2T2t1R1J/android-simple-query.gif)

### 高度なクエリ
  
このセクションでは、JOINクエリについて説明します。JOINクエリは、データベース内の複数のドキュメントを結合します。

「bookmarkedhotels」タイプのドキュメントには、ブックマークされたホテルのIDの配列であるhotelsプロパティが含まれています。

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

`bookmarkedhotels`ドキュメントの`hotels`プロパティ配列に含まれているドキュメントをフェッチするクエリを確認します。

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

次に、クエリ式を記述します。1行目は、`bookmark`データソースのプロパティ`hotels`を取得します。２行目で、`hotel`データソースのドキュメントIDを取得します。

```java
Expression hotelsExpr = Expression.property("hotels").from("bookmarkDS");
Expression hotelIdExpr = Meta.id.from("hotelDS");
```

次に、`ArrayFunction`関数式を使用して、`_id`プロパティが`hotels`配列内に含まれるドキュメントを検索します。そして、結合式を作成します。

```java
Expression joinExpr = ArrayFunction.contains(hotelsExpr, hotelIdExpr);
Join join = Join.join(hotelsDS).on(joinExpr);
```

最後に、クエリオブジェクトはその結合式を使用して、ブックマークドキュメントの`hotels`配列で参照されているすべてのホテルドキュメントを検索します。

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
- 「場所」テキストフィールドに「London」と入力します
- 「説明」テキストフィールドに「pets」と入力します
- 「ノボテルロンドンウエスト」がリストされていることを確認します
- タップしてホテルを「ブックマーク」します
- ノボテルホテルが「BookmarksActivity」ページのリストに表示されていることを確認します

![](https://cl.ly/3r243s1K2600/android-advanced-query.gif)

[目次へ戻る](./README.md)
