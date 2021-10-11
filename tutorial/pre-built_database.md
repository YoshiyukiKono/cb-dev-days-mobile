## 構築済みのデータベース

### 構築済みデータベースから開始
このセクションでは、構築済みのCouchbase　Liteデータベースをアプリケーションにバンドルする方法を学習します。
アプリケーションに静的または半静的コンテンツデータベースをバンドルし、最初の起動時にインストールする方がはるかに効率的です。
アプリの作成後にサーバー上のコンテンツの一部が変更された場合でも、アプリの最初のプルレプリケーションによってデータベースが最新の状態になります。
ここでは、空港とホテルのドキュメントのみを含む事前に作成されたデータベースを使用します。以下のコードは、ビルド済みのデータベースをバンドルされた場所からアプリケーションサポートディレクトリに移動します。

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

### やってみて
- Travel Sample Mobileアプリに「デモ」ユーザーとしてログインし、パスワードを「パスワード」としてログインします
- 「飛行機」ボタンをタップしてフライトを予約します
- 「From」空港のテキストフィールドに「London」と入力します
- ドロップダウンリストの最初の項目が「ロンドンセントパンクラス」（駅です！）であることを確認します。
