## MVPアーキテクチャ
### 概要
#### Model-View-Presenterパターン
このアプリでは、MVPパターンに従い、アプリケーションのロジックを処理し、モデルとビューの間の導管として機能するプレゼンターを介して、内部データモデルを、パッシブビューから分離します。

![](https://cl.ly/073D0j3K1d1P/92ec579c7c197eb1.png)

Android Studioプロジェクトでは、ソースファイルは機能ごとにパッケージにまとめられています。左側のナビゲーターでAndroidオプションを選択して、パッケージごとにファイルを表示できます。

![](https://cl.ly/1h080V1V2g2j/left-navigator.png)

各パッケージには、3つの種類のファイルが含まれています。

- Activity：すべてのビューロジックが存在する場所です。
- Presenter：データをフェッチしてWebサービスまたは組み込みのCouchbaseLiteデータベースに永続化するためのすべてのビジネスロジックが存在する場所です。
- Contract：PresenterとがActivityが実装するインターフェース。

このチュートリアルでは、Couchbase Lite 2.0 APIのさまざまな機能を紹介するために、以下のプレゼンターのコードをウォークスルーします：

- `BookmarksPresenter.java`
- `HotelsPresenter.java`
- `SearchFlightPresenter.java`
- `BookingsPresenter.java`


[目次へ戻る](./README.md)


# 参考情報

https://github.com/android/architecture-samples/tree/todo-mvp

https://tkhs0604.hatenablog.com/entry/android-architecture-patterns-3
