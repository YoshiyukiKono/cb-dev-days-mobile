## データモデリング

[(英文)](https://docs.couchbase.com/tutorials/mobile-travel-sample/android/design/data-modeling.html)

### アプリの説明

ドキュメントスキーマの説明の前に、モバイルアプリをもう一度見てみましょう。
モバイルアプリを実行したとき、ログイン画面に2つの異なるオプションが表示されます。

![](https://cl.ly/1s2L2Q372d2m/android-login.png)

これらのオプションは、以下のような意味を持ちます。

- ログイン(同期モード)：ユーザー資格情報を使用して、データをCouchbase Serverと同期します。WEBアプリケーションや別の端末でログインした場合もデータが引き継がれます。
- ゲスト(非同期モード)：ユーザー資格情報を必要としない、ローカルのみのモードです。ローカルデータベースで作成されたドキュメントは、Couchbase Serverと同期されません。

![](https://raw.githubusercontent.com/couchbaselabs/mobile-travel-sample/master/content/assets/travel%20sample%20mobile.png)

次のレッスンでは、これら2つのモードを切り替えて、さまざまな機能をテストします。

## データモデル

データモデルは、2つのモード間で異なります。それぞれのデータモデルを確認してみましょう。

#### ログイン/同期モード
アプリケーションは、Sync GatewayおよびTravel Sample Webバックエンド（REST API）を介して、Couchbase　Serverと通信します。Couchbase　Serverのバケットに保存されているドキュメントの種類は次のとおりです。

- airline
- airport
- hotel
- route
- user

`user`ドキュメントを除く、残りのドキュメントは、ユーザー/アプリケーションから更新されることのない情報です。`user`ドキュメントをSync Gatewayを用いたデータベースレベルの同期の対象です。

`hotel`および`airport`ドキュメントは、Couchbase　Liteデータベースにあらかじめデータがバンドルされており、アプリが起動されるとロードされます。

`airline`、`airport`、`hotel`、`route`は、Sync Gatewayを用いたデータベースレベルの同期ではなく、Travel Sample Webバックエンドによって公開されているREST　WebサービスAPIを介してアプリによってフェッチされます。


![](https://cl.ly/40330Z0M1k3F/models.png)

### ゲスト/非同期モード
ゲストモードでは、モバイルアプリは匿名ユーザー用の新しいデータベースを作成します。これは、ユーザーによってブックマークされたホテルのリストをローカルに保存するための空のデータベースです。

このモードの目的は、ユーザーとしてサインアップしていなくても、ホテルの情報を検索・閲覧することができるようにすることです。
また、閲覧したホテルを、ローカルの情報として、ブックマークしておくことができます。

ゲストモードでは、Couchbase Liteデータベースは次のタイプのドキュメントをホストします。

- bookmarkedhotels
- hotel

![](https://cl.ly/2l0118183p11/guest-model.png)

### ドキュメントタイプ
Couchbase Lite(および、バージョン6.xまでのCouchbase Server)では、RDBのテーブルとは異なり、すべてのドキュメントが同じ名前空間に保存されます。
したがって、通常は追加のプロパティを使用して、各エンティティを区別します。このアプリケーションでは、このプロパティを、「type」と名付けています。

![](https://cl.ly/1w2D1Z2J0p47/document-types.png)

### やってみよう
- Couchbase ServerのWeb管理コンソールにログインします(Dockerでセットアップした場合のID/パスワードは、Administrator/passwordです)。
- 左側のメニューから「Buckets」オプションを選択します
- travel-sampleバケットの下にある「Document」をクリックします
- ID「hotel_10025」のドキュメントを検索します
- ドキュメントの「type」プロパティが「hotel」であることを確認します

### ドキュメントキー/ID

ドキュメントキーは、Couchbase Server管理コンソール上、「ID」として表示されます。（ドキュメント)キーと（ドキュメント）IDは、同意です。

Couchbaseのすべてのドキュメントは、ドキュメントの作成時にユーザーが提供する必要のある一意のキーに関連付けられています。
キーはドキュメントの一意の識別子であり、任意の形式をとることができます。
例えば、ドキュメントの内容に関するコンテキストを提供する値を指定することができます。

Travelアプリのデータセットでは、ドキュメントのキー/IDの形式として、`{doc.type}_{alphanumeric_string}`を用いています。
`{doc.type}`は、ドキュメントの種類(type)と一致しており、`{alphanumeric_string}`と組み合わせて、ドキュメントを一意に識別します。



![](https://cl.ly/0K3V1q3m3K1Z/admin-ui.png)

### やってみよう
- Couchbase Serverの「管理コンソール」にログインします(Dockerでセットアップした場合のID/パスワードは、Administrator/passwordです)。
- 左側のメニューから「Buckets」オプションを選択します
- travel-sampleバケットの下にある「Documents」をクリックします
- ID「airline_137」のドキュメントを検索します
- ドキュメントの「callsign」プロパティが「AIRFRANS」であることを確認します

### 「`_id`」プロパティ
Sync Gatewayはドキュメントを処理するときに、関連するメタデータをドキュメントに追加します。

Sync Gatewayが追加する（メタ）データには、ドキュメントIDに対応する「`_id`」プロパティが含まれています。
Sync Gateway REST APIを介してドキュメントをクエリすると、このプロパティが表示されます。

```
{
    "_id": "airline_137",
    "_rev": "1-b4e60280a1a0e3d46efad7bfd0e2068c",
    "callsign": "AIRFRANS",
    "country": "France",
    "iata": "AF",
    "icao": "AFR",
    "id": 137,
    "name": "Air France",
    "type": "airline"
}
```

Couchbase Liteを使用するモバイルアプリ開発者は、通常、`_id`プロパティを直接読み書きする必要はありません。
Couchbase Serverのメタデータへのアクセス方法である、`META().id`フィールドをクエリの中で用いて、ドキュメントIDを取得あるいは検索条件に利用することができます。

[目次へ戻る](./README.md)
