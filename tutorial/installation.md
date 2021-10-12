# インストール

[(オリジナル英文)](https://docs.couchbase.com/tutorials/mobile-travel-sample/android/installation/travel-mobile-app.html)

## 前提条件
- Google Developerサイトからダウンロード可能な最新バージョンのAndroid Studio（4.x+）
- APIレベル22以上を実行しているAndroidエミュレーター(またはデバイス)
- Android SDK 29
- Androidビルドツール29+
- JDK 8

#### Windowsユーザー：
Windowsで開発している場合は、Windows10マシンを使用する必要があります。

## 旅行サンプルモバイルアプリ

GitHubから「master」ブランチのクローンを作成します。クローン作成プロセスを高速化するために、`--depth 1`として浅いプルを実行しています。

```
git clone -b master --depth 1 https://github.com/couchbaselabs/mobile-travel-sample.git
```

Android　Studioを使用してプロジェクトを開きます。build.gradleは`mobile-travel-sample/android/TravelSample`にあります。

### バックエンドに接続するようにアプリを構成する

バックエンドに接続するには、アプリで指定されたURLを更新する必要があります。

#### WebバックエンドURLの更新
`util`フォルダー内の`DatabaseManager.java`ファイルを開きます。PythonWebサーバーを指す`APPLICATION_ENDPOINT`定数を更新する必要があります。

ここで、指定するURLは、バックエンドをデプロイするために選択したインストールオプションによって異なります。以下はローカルホストに（Dockerまたは直接）インストールした例です。

```
public static String APPLICATION_ENDPOINT = "http://10.0.2.2:8080/api/";
```

#### 同期ゲートウェイURLの更新
次に、SyncGatewayエンドポイントを更新します。

`util`フォルダー内の`DatabaseManager.java`ファイルを開きます。`SGW_ENDPOINT`定数を更新する必要があります。

ここで、指定するURLは、バックエンドをデプロイするために選択したインストールオプションによって異なります。以下はローカルホストに（Dockerまたは直接）インストールした例です。

```
    public static String SGW_ENDPOINT = "ws://10.0.2.2:4984/travel-sample";
```

#### やってみよう

- プロジェクトをビルドしてAndroidエミュレーターを使用して実行します。
- ログイン画面がemuatorに表示されることを確認します。

[目次へ戻る](./README.md)
