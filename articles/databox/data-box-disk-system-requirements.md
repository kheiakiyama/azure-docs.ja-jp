---
title: Microsoft Azure Data Box Disk のシステム要件| Microsoft Docs
description: Azure Data Box Disk のソフトウェア要件とネットワーキング要件について説明します。
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: article
ms.date: 01/09/2019
ms.author: alkohli
ms.openlocfilehash: b2f8a299dfc52bcc1f2eaa17e658b841d27bf62c
ms.sourcegitcommit: 33091f0ecf6d79d434fa90e76d11af48fd7ed16d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2019
ms.locfileid: "54154931"
---
# <a name="azure-data-box-disk-system-requirements"></a>Azure Data Box Disk のシステム要件

この記事では、Microsoft Azure Data Box Disk ソリューション、および Data Box Disk に接続するクライアントのシステム要件のうち、重要なものについて説明します。 この情報は、Data Box Disk をデプロイする前によく確認し、デプロイ時やそれ以降の操作時にも、必要に応じて繰り返し参照することをお勧めします。

システム要件には、ディスクに接続するクライアントでサポートされるプラットフォーム、サポートされるストレージ アカウント、およびストレージの種類が含まれます。


## <a name="supported-operating-systems-for-clients"></a>クライアントでサポートされるオペレーティング システム

Data Box Disk に接続するクライアント経由でディスクのロック解除、およびデータのコピー操作を行う場合にサポートされるオペレーティング システムの一覧を次に示します。

| **オペレーティング システム** | **テスト済みのバージョン** |
| --- | --- |
| Windows Server |2008 R2 SP1 <br> 2012 <br> 2012 R2 <br> 2016 |
|  Windows |7、8、10 |
| Linux <br> <li> Ubuntu </li><li> Debian </li><li> Red Hat Enterprise Linux (RHEL) </li><li> CentOS| <br>14.04、16.04、18.04 <br> 8.11、9 <br> 7.0 <br> 6.5、6.9、7.0、7.5 |  

## <a name="other-required-software-for-windows-clients"></a>Windows クライアントに必要なその他のソフトウェア

Windows クライアントでは、次のものもインストールする必要があります。

| **ソフトウェア**| **バージョン** |
| --- | --- |
| Windows PowerShell |5.0 |
| .NET Framework |4.5.1 |
| Windows Management Framework |5.0|
| BitLocker| - |

## <a name="other-required-software-for-linux-clients"></a>Linux クライアントに必要なその他のソフトウェア

Linux クライアントでは、Data Box Disk のツールセットが次の必要なソフトウェアをインストールします:

- dislocker
- OpenSSL

## <a name="supported-storage-accounts"></a>サポートされるストレージ アカウント

Data Box Disk でサポートされるストレージの種類の一覧を次に示します。

| **ストレージ アカウント** | **メモ** |
| --- | --- |
| クラシック | 標準 |
| 汎用  |標準。V1 と V2 の両方がサポートされます。 また、ホット層とクール層の両方がサポートされます。 |

>[!NOTE]
> Azure Data Lake Storage Gen 2 アカウントはサポートされていません。


## <a name="supported-storage-types"></a>サポートされるストレージの種類

Data Box Disk でサポートされるストレージの種類の一覧を次に示します。

| **ファイル形式** | **メモ** |
| --- | --- |
| Azure ブロック BLOB | |
| Azure ページ BLOB  | |


## <a name="next-step"></a>次のステップ

* [Azure Data Box Disk をデプロイする](data-box-disk-deploy-ordered.md)

