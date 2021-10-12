## MVPアーキテクチャ
### 
概要
Model-View-Presenterパターン
このアプリでは、MVPパターンに従い、内部データモデルを、アプリケーションのロジックを処理し、モデルとビューの間の導管として機能するプレゼンターを介してパッシブビューから分離します。

92ec579c7c197eb1
Android Studioプロジェクトでは、コードは機能ごとに構造化されています。左側のナビゲーターでAndroidオプションを選択して、パッケージごとにファイルを表示できます。

左ナビゲーター
各パッケージには、3つの異なるファイルが含まれています。

- Activity：すべてのビューロジックが存在する場所です。
- Presenter：データをフェッチしてWebサービスまたは組み込みのCouchbaseLiteデータベースに永続化するためのすべてのビジネスロジックが存在する場所です。
- Contract：PresenterとがActivityが実装するインターフェース。

このチュートリアルでは、Couchbase Lite 2.0 APIのさまざまな機能を紹介するために、様々なプレゼンターのコードをウォークスルーします：
`BookmarksPresenter.java`、`HotelsPresenter.java`、`SearchFlightPresenter.java`と`BookingsPresenter.java`。


[目次へ戻る](./README.md)
