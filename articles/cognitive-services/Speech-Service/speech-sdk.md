---
title: Speech Service SDK について - Speech Services
titleSuffix: Azure Cognitive Services
description: Speech Service ソフトウェア開発キット (SDK) は、Speech サービスの機能に対するネイティブ アクセス権をアプリケーションに提供します。これによりソフトウェアの開発が容易になります。 この記事では、Windows、Linux、Android 向けの SDK について、さらに詳しく取り上げます。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: speech-service
ms.topic: conceptual
ms.date: 12/18/2018
ms.author: wolfma
ms.custom: seodec18
ms.openlocfilehash: 1c29ed47e499ee23fab9f3b34e3974f479e3dbb7
ms.sourcegitcommit: 4eeeb520acf8b2419bcc73d8fcc81a075b81663a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/19/2018
ms.locfileid: "53604273"
---
# <a name="about-the-speech-service-sdk"></a>Speech Service SDK について

Speech Service ソフトウェア開発キット (SDK) は、Speech サービスの機能に対するネイティブ アクセス権をアプリケーションに提供します。これによりソフトウェアの開発が容易になります。 現時点で、SDK からは、**音声テキスト変換**、**音声翻訳**、および**意図認識**にアクセスできます。

[!INCLUDE [Speech SDK Platforms](../../../includes/cognitive-services-speech-service-speech-sdk-platforms.md)]

[!INCLUDE [License Notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

## <a name="get-the-sdk"></a>SDK の取得

### <a name="windows"></a> Windows

Windows の場合、次の言語がサポートされています。

* C# (UWP と .NET)、C++: Speech SDK NuGet パッケージの最新バージョンを参照および使用することができます。 パッケージには、32 ビットおよび 64 ビットのクライアント ライブラリとマネージ (.NET) ライブラリが含まれています。 この SDK は NuGet を使用して Visual Studio でインストールできます。 **Microsoft.CognitiveServices.Speech** を検索してください。

* Java:Speech SDK Maven パッケージの最新バージョンを参照および使用することができます。これは Windows x64 のみをサポートします。 Maven プロジェクトでは、追加のリポジトリとして `https://csspeechstorage.blob.core.windows.net/maven/` を追加し、依存関係として `com.microsoft.cognitiveservices.speech:client-sdk:1.2.0` を参照します。

### <a name="linux"></a> Linux

> [!NOTE]
> 現在は、Ubuntu 16.04 および 18.04 のみが PC 上でサポートされています (C++ 開発向けには x86 または x64、.NET Core および Java 向けには x64)。

次のシェル コマンドを実行して、必須のコンパイラおよびライブラリがインストールされていることを確認します:

```sh
sudo apt-get update
sudo apt-get install build-essential libssl1.0.0 libcurl3 libasound2
```

* C#:Speech SDK NuGet パッケージの最新バージョンを参照および使用することができます。 SDK を参照するには、プロジェクトに次のパッケージ参照を追加します。

  ```xml
  <PackageReference Include="Microsoft.CognitiveServices.Speech" Version="1.2.0" />
  ```

* Java:Speech SDK Maven パッケージの最新バージョンを参照および使用することができます。 Maven プロジェクトでは、追加のリポジトリとして `https://csspeechstorage.blob.core.windows.net/maven/` を追加し、依存関係として `com.microsoft.cognitiveservices.speech:client-sdk:1.2.0` を参照します。

* C++: [.tar パッケージ](https://aka.ms/csspeech/linuxbinary)として SDK をダウンロードし、ファイルを任意のディレクトリにアンパックします。 SDK のフォルダー構造を次の表に示します。

  |Path|説明|
  |-|-|
  |`license.md`|ライセンス|
  |`ThirdPartyNotices.md`|サード パーティに関する通知|
  |`include`|C および C++ 用のヘッダー ファイル|
  |`lib/x64`|アプリケーションとのリンクのためのネイティブ x64 ライブラリ|
  |`lib/x86`|アプリケーションとのリンクのためのネイティブ x86 ライブラリ|

  アプリケーションを作成するには、必須のバイナリ (およびライブラリ) を開発環境にコピーまたは移動し、 必要に応じてそれらをビルド プロセスに含めます。

### <a name="android"></a>Android

Android 用 Java SDK は、必要なライブラリと必要な Android アクセス許可を含む [AAR (Android ライブラリ)](https://developer.android.com/studio/projects/android-library) としてパッケージ化されます。 これは、`https://csspeechstorage.blob.core.windows.net/maven/` にある Maven リポジトリでパッケージ `com.microsoft.cognitiveservices.speech:client-sdk:1.2.0` としてホストされます。

このパッケージを Android Studio プロジェクトから使用するには、次の変更を行います。

* プロジェクト レベルの build.gradle ファイルで、`repository` セクションに次を追加します。

  ```text
  maven { url 'https://csspeechstorage.blob.core.windows.net/maven/' }
  ```

* モジュール レベルの build.gradle ファイルで、`dependencies` セクションに次を追加します。

  ```text
  implementation 'com.microsoft.cognitiveservices.speech:client-sdk:1.2.0'
  ```

Java SDK は [Speech Devices SDK](speech-devices-sdk.md) の一部でもあります。

[!INCLUDE [Get the samples](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]

## <a name="next-steps"></a>次の手順

* [Speech 試用版サブスクリプションを取得する](https://azure.microsoft.com/try/cognitive-services/)
* [C# で音声を認識する方法を確認する](quickstart-csharp-dotnet-windows.md)
