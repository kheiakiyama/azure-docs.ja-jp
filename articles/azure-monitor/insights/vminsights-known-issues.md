---
title: Azure Monitor for VMs (プレビュー) の既知の問題 | Microsoft Docs
description: この記事では Azure Monitor for VMs の既知の問題について説明します。Azure Monitor for VMs は、Azure VM オペレーティング システムの正常性とパフォーマンスの監視が組み合わされた Azure のソリューションです。 Azure Monitor for VMs ではまた、アプリケーション コンポーネント、および他のリソースとの依存関係が自動的に検出され、その間の通信がマップされます。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/26/2018
ms.author: magoedte
ms.openlocfilehash: c329e1fa80c6647bb78b11917ecd012461e62ea4
ms.sourcegitcommit: 295babdcfe86b7a3074fd5b65350c8c11a49f2f1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/27/2018
ms.locfileid: "53790505"
---
# <a name="known-issues-with-azure-monitor-for-vms-preview"></a>Azure Monitor for VMs (プレビュー) の既知の問題

この記事では Azure Monitor for VMs の既知の問題について説明します。Azure Monitor for VMs は、Azure VM オペレーティング システムの正常性とパフォーマンスの監視が組み合わされた Azure のソリューションです。 

正常性機能に関する既知の問題を次に示します。

- 正常性機能は、Log Analytics ワークスペースに接続されているすべての VM に対して有効化されます。 これは、アクションが単一の VM から開始されて完了した場合にも当てはまります。
- Linux VM に関して、単一 VM ビューの正常性基準を一覧表示するページのタイトルに、ユーザー定義の VM 名ではなく VM のドメイン名全体が表示されます。  
- サポートされている方法を使用して VM の監視を無効にした後、それを再びデプロイする場合は、前と同じワークスペースにデプロイする必要があります。 新しいワークスペースを使用する場合は、その VM の正常性状態を表示する際に、異常な動作が見られる場合があります。
- Azure VM は、削除または移動された場合でも、しばらくの間、VM リスト ビューに表示されます。 さらに、移動または削除された VM の状態をクリックすると、**正常性診断**ビューが開き、読み込みループが開始されます。 削除された VM の名前を選択すると、ウィンドウが開き、VM が削除されたことを示すメッセージが表示されます。
- このリリースでは、正常性基準となる期間と頻度を変更できません。 
- 正常性基準は無効にできません。 
- デプロイの後、**[Azure Monitor]** > **[仮想マシン]** > **[正常性]** ウィンドウまたは **[VM resource]\(VM のリソース\)** > **[分析情報]** ウィンドウにデータが表示されるまでに時間がかかる場合があります。
- 正常性診断エクスペリエンスは他のどのビューよりも迅速に更新されます。 ビューの切り替えを行う場合に、情報の遅延が発生する可能性があります。 
- Azure Monitor for VM に含まれているアラートの概要には、監視対象の Azure VM で検出された正常性の問題に関するアラートのみが表示されます。
- VM をシャットダウンすると、正常性基準が "*クリティカル*" に更新されるものと、"*正常*" に更新されるものがあります。 VM の最終的な状態は、"*クリティカル*" になります。
- 正常性アラートの重大度を変更することはできません。有効または無効にのみ変更できます。 また、一部の重大度は、正常性の基準の状態に基づいて更新されます。   
- 正常性基準インスタンスの設定を変更すると、VM 上の同じ種類の正常性基準インスタンスすべてが変更されます。 たとえば、論理ディスク C: に対応する空きディスク領域の正常性基準インスタンスのしきい値を変更すると、このしきい値は、同じ VM において検出および監視されている他の論理ディスクすべてに適用されます。  
- Windows VM を対象とする正常性基準のしきい値は変更できません。これらの正常性状態は、"*実行中*" または "*使用可能*" に設定されているためです。 [Workload Monitor API](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/workloadmonitor/resource-manager) から正常性状態に対してクエリを実行すると、以下に該当する場合、サービスまたはエンティティに対して、**LessThan** または **GreaterThan** の *comparisonOperator* 値と、"*しきい*" 値 **4** が表示されます。
   - DNS クライアント サービスの正常性 – サービスが実行されていない。 
   - DHCP クライアント サービスの正常性 – サービスが実行されていない。 
   - RPC サービスの正常性 – サービスが実行されていない。 
   - Windows ファイアウォール サービスの正常性 – サービスが実行されていない。
   - Windows イベント ログ サービスの正常性 – サービスが実行されていない。 
   - サーバー サービスの正常性 – サービスが実行されていない。 
   - Windows リモート管理サービスの正常性 – サービスが実行されていない。 
   - ファイル システム エラーまたは破損 – 論理ディスクが使用できない。

- 次の Linux の正常性基準のしきい値は変更できません。これらの正常性状態は、既に *true* に設定されているためです。 Workload Monitoring API からエンティティに対するクエリが実行されると、コンテキストに応じて、正常性状態には、値 **LessThan** を使用した *comparisonOperator* と "*しきい*" 値 **1** が表示されます。
   - 論理ディスクの状態 – 論理ディスクがオンライン/使用可能でない
   - ディスクの状態 – ディスクがオンライン/使用可能でない
   - ネットワーク アダプターの状態 - ネットワーク アダプターが無効  

- しきい値の更新などの構成の変更は、ポータルまたは Workload Monitor API でそれらがすぐに更新される場合でも、最大 30 分かかります。 
- 個々のプロセッサおよび論理プロセッサ レベルの正常性基準は Windows では使用できません。 Windows VM では CPU 使用率の合計のみを使用できます。 
- 各正常性基準に対して定義されているアラート ルールは、Azure portal には表示されません。 [Workload Monitor API](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/components) でのみ、正常性アラート ルールを有効または無効にすることができます。 
- 正常性アラート用の [Azure Monitor アクション グループ](../../azure-monitor/platform/action-groups.md)を Azure portal 内で割り当てることはできません。 正常性アラートが発生するたびにトリガーされるようにアクション グループを構成するには、通知設定 API を使用する必要があります。 現在、VM に対して発生したすべての*正常性アラート*によって同じアクション グループがトリガーされるように、VM に対してアクション グループを割り当てることができます。 従来の Azure アラートとは異なり、正常性アラート ルールごとに個別のアクション グループが存在するという概念はありません。 さらに、正常性アラートがトリガーされたときに、電子メールまたは SMS 通知を提供するように構成されたアクション グループのみがサポートされます。 

## <a name="next-steps"></a>次の手順
ご利用の仮想マシンの監視を有効にするための要件と方法については、[Azure Monitor for VMs のデプロイ](vminsights-onboard.md)に関するページを確認してください。
