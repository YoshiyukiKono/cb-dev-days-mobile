# Couchbase モバイル & エッジコンピューティングワークショップ

## 事前準備

[こちら](https://github.com/YoshiyukiKono/cb-dev-days-couchbase/blob/main/labs/Lab%20-%20Prerequisite%20Steps_JP.pdf)から事前準備の手順をご参照ください。
演習に利用する環境でDockerとGitが利用できる場合は、特に追加でご準備いただく必要はありません。

## 講義
[slides](./slides)ディレクトリに、講義セッションで利用するスライドのPDFファイルを格納しています。

## ラボ/演習

### 概要
本ワークショップのラボは、公開されているチュートリアル（[Couchbase Mobile Workshop](https://docs.couchbase.com/tutorials/mobile-travel-sample/introduction.html)）の内容に基づきます。

#### 利用するCouchbaseプロダクトとそのバージョン
- Couchbase Server 6.5
- Couchbase Lite 2.8
- Sync Gateway 2.8


#### プログラミング言語

[チュートリアル](https://docs.couchbase.com/tutorials/mobile-travel-sample/introduction.html)では、下記の言語がサポートされています。

- Android Java
- Swift
- C#
- Java

演習では、基本的に、Android Javaを用いるものとします（実務との関係で他のプログラミング言語の利用を希望される場合は、[チュートリアル](https://docs.couchbase.com/tutorials/mobile-travel-sample/introduction.html)に基づいて実施することも可能です）。

Androidでのチュートリアルについて、[こちら](./tutorial)から、日本語化したもの(省略または補足あり)を参照することができます。


### 演習を始めるにあたって

まず、はじめにチュートリアルで用いる、Travel Sampleアプリケーションのリポジトリーを取得します。

このリポジトリーに含まれるSyng Gatewayの設定ファイルを環境構築で利用する必要があります。

```
git clone -b master --depth 1 https://github.com/couchbaselabs/mobile-travel-sample.git
```

（高速化のために、`--depth 1`として浅いプルを実行しています）



### Couchbase Server　/ Sync Gateway 環境構築

本ワークショップでは、基本的には、ワークショップ終了後もご利用いただけるように、ご自身のローカル環境でCouchbase Serverを動作させる事を想定しています。

[こちら](./setup_env)の手順に従って、実行してください。

下記の3つのDockerイメージを用います。

- Couchbase Server
- Sync Gateway
- Web/REST API Server(Python)

Couchbase Serverについては、チュートリアル用に公開されている、Dockerによる自動セットアップを用いて、データやユーザーがセットアップ済みの環境を構築します。
自動セットアップの内容に関心がある場合は、[こちら](https://docs.couchbase.com/tutorials/mobile-travel-sample/android/installation/manual.html#couchbase-server)で、マニュアルの設定手順を確認することができます（英語）。

### モバイル開発

ラボの目的は、モバイル開発者が自分のマシンを使用してアプリをデプロイして実行できるようにすることです。
コードの編集は、接続先の修正に止まり、コード記述による機能の実装は、演習に含まれません。

開発環境の前提条件を以下に示します。

### Android開発環境

-	Google デベロッパー サイトからダウンロード可能な Android スタジオ (4.x+) の最新バージョン
-	APIレベル22以上を実行しているAndroidデバイスまたはエミュレータ
-	Android SDK 29
-	Android ビルドツール29+
-	JDK 8

演習では、基本的に、Android Javaを用いるものとします。
実務との関係で他のプログラミング言語の利用を希望される場合は、以下の前提条件を参照してください。

### Swift開発環境
-	Xcode 12.4+: Mac App Storeからダウンロード可能な最新バージョン.

### C# 開発環境

#### Visual Studio (Xamarinフォームcsharpに必要)

- Windows：　Visual Studio (16.9.4 以降)
- Mac： Visual Studio 2019 (v8.9 以降) – 注: Mac 用の Visual Studio は UWP プロジェクトをサポートしていません。

#### OS

##### Windowsユーザー
Windows で開発している場合は、バージョン 1809 以降の Windows 10 コンピューターを使用することをお勧めします。

##### Mac（M1プロセッサ搭載）ユーザー
Visual Studio とCouchbaseは、ロゼッタ v2 変換レイヤーを介して M1 プロセッサでサポートされています。

###### Xamarin iOS 開発
iOS の開発では、XCode 12.4 以降のバージョンをお勧めします。
OS と XCode のバージョンの互換性については、Apple のドキュメントhttps://developer.apple.com/support/xcode/を参照してください。

###### Xamarin　Android開発
GoogleのIDE　Androidスタジオバージョン4.x以上をお勧めします。
AndroidスタジオとOSの互換性については、Googleのドキュメント https://developer.android.com/studio#system-requirements-a-namerequirementsa/ 参照してください。

- Androidバージョン: 常に最新バージョンの OS をエミュレーターで実行することをお勧めします。Androidネイティブサンプルは、Android4.4以上でテストされています。Xamarin の例では、Android バージョン 5.1 以降でテストされています。
- Android SDK: 最新の Android SDK をインストールすることをお勧めします。最新バージョンは次のURLから入手できます: https://docs.microsoft.com/en-us/xamarin/android/get-started/installation/android-sdk?tabs=windows


###### Xamarinホットリローデッド
ホットリローデッドを使用するには、iOS v14またはAndroid 10以上をターゲットにすることを強くお勧めします。
完全なドキュメントは、マイクロソフトのドキュメント サイト https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/hot-reload にあります。

#### Java開発

- SDK　8
- 任意のIDEを使用できますが、チュートリアルではIntelliJ IDEAを使用しています(IntelliJ IDEA 2019以降、コミュニティ版可）。





