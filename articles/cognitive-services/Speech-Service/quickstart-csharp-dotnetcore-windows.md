---
title: クイック スタート:音声を認識する (C# (.NET Core Windows)) - Speech Services
titleSuffix: Azure Cognitive Services
description: Windows の .NET Core で C# と Speech Service SDK を使用して音声を認識する方法について説明します。
services: cognitive-services
author: wolfma61
manager: cgronlun
ms.service: cognitive-services
ms.component: speech-service
ms.topic: quickstart
ms.date: 12/13/2018
ms.author: wolfma
ms.openlocfilehash: a5a04fdede498d404a00d666e4042337b4dc675b
ms.sourcegitcommit: 549070d281bb2b5bf282bc7d46f6feab337ef248
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/21/2018
ms.locfileid: "53727838"
---
# <a name="quickstart-recognize-speech-with-the-speech-sdk-for-net-core"></a>クイック スタート:.NET Core 用 Speech SDK を使用して音声を認識する

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

この記事では、Cognitive Services [Speech SDK](speech-sdk.md) を使用してWindows の .NET Core 用の C# コンソール アプリケーションを作成します。 PC のマイクからリアルタイムで音声をテキストに変換します。 このアプリケーションの構築には、[Speech SDK NuGet パッケージ](https://aka.ms/csspeech/nuget)と Microsoft Visual Studio 2017 (任意のエディション) を使用します。

> [!NOTE]
> .NET Core は、[.NET Standard](https://docs.microsoft.com/dotnet/standard/net-standard) 仕様を実装した、オープン ソース、クロスプラットフォームの .NET プラットフォームです。

このクイック スタートを完了するには、Speech サービス サブスクリプション キーが必要です。 1 つ無料で取得できます。 詳しくは、[Sppech サービスを無料で試す](get-started.md)ための記事を参照してください。

## <a name="prerequisites"></a>前提条件

このクイック スタートでは以下が必要です。

* [.NET コア SDK](https://dotnet.microsoft.com/download)
* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)
* Speech Service の Azure サブスクリプション キー。 [無料で 1 つ取得できます](get-started.md)。

## <a name="create-a-visual-studio-project"></a>Visual Studio プロジェクトを作成する

[!INCLUDE [](../../../includes/cognitive-services-speech-service-quickstart-dotnetcore-create-proj.md)]

## <a name="add-sample-code"></a>サンプル コードを追加する

1. `Program.cs` を開き、その中のすべてのコードを次の内容に置き換えます。

    [!code-csharp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/csharp-dotnetcore/helloworld/Program.cs#code)]

1. 同じファイル内で、文字列 `YourSubscriptionKey` をサブスクリプション キーに置き換えます。

1. 文字列 `YourServiceRegion` を、サブスクリプションに関連付けられた[リージョン](regions.md) (たとえば、無料試用版サブスクリプションでは `westus`) に置き換えます。

1. プロジェクトへの変更内容を保存します。

## <a name="build-and-run-the-app"></a>アプリのビルドと実行

1. アプリケーションをビルドします。 メニュー バーから、**[ビルド]** > **[ソリューションのビルド]** を選択します。 コードはエラーなくコンパイルされるはずです。

    ![[ソリューションのビルド] オプションを強調表示した Visual Studio アプリケーションのスクリーンショット](media/sdk/qs-csharp-dotnetcore-windows-05-build.png "成功したビルド")

1. アプリケーションを起動します。 メニュー バーから、**[デバッグ]** > **[デバッグの開始]** を選択するか、**F5** キーを押します。

    ![[デバッグの開始] オプションを強調表示した Visual Studio アプリケーションのスクリーンショット](media/sdk/qs-csharp-dotnetcore-windows-06-start-debugging.png "アプリのデバッグの開始")

1. コンソール ウィンドウが表示され、何か言うように求められます。 英語の語句または文を読み上げます。 音声が Speech サービスに送信されてテキストに変換され、同じウィンドウに表示されます。

    ![正常に認識された後のコンソールの出力のスクリーンショット](media/sdk/qs-csharp-dotnetcore-windows-07-console-output.png "正常に認識された後のコンソールの出力")

## <a name="next-steps"></a>次の手順

オーディオ ファイルから音声を読み取る方法など、追加のサンプルは GitHub で入手できます。

> [!div class="nextstepaction"]
> [GitHub で C# のサンプル コードを詳しく見てみる](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>関連項目

- [音響モデルをカスタマイズする](how-to-customize-acoustic-models.md)
- [言語モデルをカスタマイズする](how-to-customize-language-model.md)
