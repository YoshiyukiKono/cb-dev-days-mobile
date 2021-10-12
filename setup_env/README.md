# 環境セットアップ

## 目次
1. [Couchbase Server](./setup_cbs.md)
1. [Sync Gateway](./setup_sg.md)
1. [Travel Sample Webバックエンド](./setup_web.md)

## アーキテクチャー

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/travelsampleapp-arch.png)


次のコンポーネントにより構成されています。

### クライアント側

- モバイルデバイスまたはデスクトップで実行されているCouchbaseLite対応の旅行アプリ（iOS、Android、UWP、Xamarin（iOSおよびAndroid）、およびJavaSwingアプリをサポートします）。

### バックエンド/サーバー側

- Couchbase Server Enterprise v6.5.x

- Sync Gateway Enterprise v2.8.x

- Travel Webアプリ（バックエンド/フロントエンド）： Travel Sample Webアプリには、Couchbase Python SDK 3.0.xで実装されたPythonベースのWebバックエンドと、vue.jsベースのWebフロントエンドが含まれています。
