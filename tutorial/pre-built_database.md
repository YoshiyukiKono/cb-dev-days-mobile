## 構築済みデータベース

[オリジナル英文](https://docs.couchbase.com/tutorials/mobile-travel-sample/android/develop/pre-built-database.html)

### 構築済みデータベースからスタートする

このセクションでは、アプリケーションにバンドルされた構築済みのCouchbase　Liteデータベースを利用する方法を紹介します。

アプリケーションに静的または半静的コンテンツデータベースをバンドルし、最初の起動時にインストールする方が、全てのデータをサーバーから取得するよりも、はるかに効率的です。
アプリの作成後にサーバー上のコンテンツの一部が変更された場合でも、アプリの最初のプルレプリケーションによってデータベースを最新の状態にすることができます。

ここでは、空港とホテルのドキュメントを含む事前に構築済みのデータベースを使用します。以下のコードは、ビルド済みのデータベースをバンドルされた場所からアプリケーションが利用するディレクトリに移動します。

ファイル `app/src/android/java/…/util/DatabaseManager.java`を開き、データベースコンストラクターに移動します。

このメソッドは、最初に特定のユーザーのデータベースファイルがすでに存在するかどうかを確認します。
存在しない場合は、アセットディレクトリからデータベースをロードし、解凍して、ユーザー用に作成されたフォルダにコピーします。

`DatabaseManager.java`

```java
    public void OpenDatabaseForUser(String username) {
        File dbFile = new File(appContext.getFilesDir()+"/"+ username, "travel-sample.cblite2");
        DatabaseConfiguration config = new DatabaseConfiguration();
        config.setDirectory(String.format("%s/%s", appContext.getFilesDir(),username));
        currentUser = username;

        if (!dbFile.exists()) {
            AssetManager assetManager = appContext.getAssets();
            try {
                File path = new File(appContext.getFilesDir()+"");
                unzip(assetManager.open("travel-sample.cblite2.zip"),path);
                Database.copy(new File(appContext.getFilesDir(),"travel-sample.cblite2"), "travel-sample", config);

            }
            catch (IOException e) {
                e.printStackTrace();
            }
            catch (CouchbaseLiteException e) {
                e.printStackTrace();
            }

        }
        try {
            database = new Database("travel-sample", config);
            createFTSQueryIndex();
        } catch (CouchbaseLiteException e) {
            e.printStackTrace();
        }
    }
```

### やってみよう
- Travelモバイルアプリに「demo」ユーザーとしてログインし、パスワードを「password」としてログインします
- 「Airplane」ボタンをタップしてフライトを予約します
- 「From(目的地)」空港のテキストフィールドに「London」と入力します
- ドロップダウンリストの最初の項目が「London St Pancras」であることを確認します。

![](https://cl.ly/3V3h151g0x19/android-prebuilt-db.gif)

[目次へ戻る](./README.md)
