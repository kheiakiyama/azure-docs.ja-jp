---
title: Excel で Azure Analysis Services に接続する | Microsoft Docs
description: Azure Analysis Services サーバーに Excel を使って接続する方法を説明します。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 10/18/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 14453a1603eb414441008ef1381800009f4268e4
ms.sourcegitcommit: 2469b30e00cbb25efd98e696b7dbf51253767a05
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/06/2018
ms.locfileid: "52994423"
---
# <a name="connect-with-excel"></a>Excel を使用した接続

サーバーを作成し、それに表形式モデルをデプロイしたら、クライアントは接続してデータの調査を開始できます。 

## <a name="before-you-begin"></a>開始する前に
サインインに使用するアカウントは、少なくとも、読み取り許可を持つ model データベースのロールに属していなければなりません。 詳細については、「[認証とユーザーのアクセス許可](analysis-services-manage-users.md)」を参照してください。 

## <a name="connect-in-excel"></a>Excel での接続

Excel でのサーバーへの接続は、Excel 2016 のデータの取得を使用してサポートされます。 Power Pivot での [テーブルのインポート] ウィザードを使用した接続はサポートされません。 

**Excel 2016 で接続するには**

1. Excel 2016 では、**[データ]** リボンで **[外部データの取り込み]** > **[その他のデータ ソース]** > **[From Analysis Services] Analysis Services** をクリックします。

2. データ接続ウィザードで、**[サーバー名]** にプロトコルや URI などのサーバー名を入力します。 たとえば、「asazure://westcentralus.asazure.windows.net/advworks」のように入力します。 そして **[ログオン資格情報]** で **[以下のユーザー名とパスワードを使用する]** を選択し、組織でのユーザー名 (たとえば nancy@adventureworks.com) とパスワードを入力します。

    > [!IMPORTANT]
    > Microsoft アカウント、Live ID、Yahoo、Gmail などでサインインする場合や、多要素認証を使用してサインインする必要がある場合は、パスワード フィールドを空白のままにしておきます。 [次へ] をクリックすると、パスワードの入力を求められます。 

    ![Excel ログオンから接続する](./media/analysis-services-connect-excel/aas-connect-excel-logon.png)

3. **[データベースとテーブルの選択]** で、データベースとモデルまたはパースペクティブを選択し、**[完了]** をクリックします。
   
    ![Excel 選択モデルから接続する](./media/analysis-services-connect-excel/aas-connect-excel-select.png)


## <a name="see-also"></a>関連項目
[クライアント ライブラリ](analysis-services-data-providers.md)   
[サーバーの管理](analysis-services-manage.md)     


