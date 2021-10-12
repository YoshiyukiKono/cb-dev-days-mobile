旅行モバイルアプリ
前提条件
GoogleDeveloperサイトからダウンロード可能な最新バージョンのAndroidStudio（4.x +）

APIレベル22以上を実行しているAndroidデバイスまたはエミュレーター

Android SDK 29

Androidビルドツール29+

JDK 8

Windowsユーザー：Windowsで開発している場合は、Windows10マシンを使用する必要があります。また、手動またはDockerインストールモードを選択した場合は、必要な実行可能ファイルのインストールと実行を許可できるように、Windowsボックスの管理者権限も必要であることに注意してください。

旅行サンプルモバイルアプリ
GitHubからTravelSampleアプリの「マスター」ブランチのクローンを作成します。depthクローン作成プロセスを高速化するために、1として浅いプルを実行しています。

BASH
コピー
git clone -b master --depth 1 https://github.com/couchbaselabs/mobile-travel-sample.git
AndroidStudioを使用してプロジェクトを開きます。build.gradleは/パス/に/モバイル・旅行・サンプル/アンドロイド/ TravelSample /フォルダ/ディレクトリにあります。

バックエンドに接続するようにアプリを構成する
バックエンドに接続するには、アプリで指定されたURLを更新する必要があります。まだ更新していない場合は、「バックエンドのインストール」で概説されている手順を実行して、Couchbase Server、Sync Gateway、およびPythonWebバックエンドアプリをインストールします。

WebバックエンドURLの更新
DatabaseManager.javautilフォルダー内のファイルを開きます。APPLICATION_ENDPOINTPythonWebサーバーを指す定数を更新する必要があります。

これで、指定するURLは、バックエンドをデプロイするために選択したインストールオプションによって異なります。

マニュアル

Docker

クラウド

JAVA
コピー
public static String APPLICATION_ENDPOINT = "http://10.0.2.2:8080/api/";
同期ゲートウェイURLの更新
次に、SyncGatewayエンドポイントを更新します。

DatabaseManager.javautilフォルダー内のファイルを開きますSGW_ENDPOINT。定数を更新する必要があります。

これで、指定するURLは、バックエンドをデプロイするために選択したインストールオプションによって異なります。

マニュアル

Docker

クラウド

JAVA
コピー
    public static String SGW_ENDPOINT = "ws://10.0.2.2:4984/travel-sample";
やってみて
Androidエミュレーターを使用してプロジェクトをビルドして実行します。

ビルドとして
ログイン画面がemuatorに表示されることを確認します。

