---
title: '仮想ネットワークのパフォーマンスのトラブルシューティング: Azure | Microsoft Docs'
description: このページでは、Azure ネットワーク リンクのパフォーマンスをテストする標準化された方法について説明します。
services: expressroute
author: tracsman
ms.service: expressroute
ms.topic: article
ms.date: 12/20/2017
ms.author: jonor
ms.custom: seodec18
ms.openlocfilehash: 2572ff3711fb86cda88a86744192980a5b2d5361
ms.sourcegitcommit: 7fd404885ecab8ed0c942d81cb889f69ed69a146
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/12/2018
ms.locfileid: "53277626"
---
# <a name="troubleshooting-network-performance"></a>ネットワーク パフォーマンスのトラブルシューティング
## <a name="overview"></a>概要
Azure では、オンプレミスのネットワークから Azure に接続するための高速で安定した方法を提供します。 大小を問わずすべての企業のお客様が、サイト間 VPN や ExpressRoute などの方法を上手に利用して Azure でビジネスを運営しています。 一方で、パフォーマンスが期待通りではなかったり、これまでのエクスペリエンスを下回った場合にはどうなるでしょうか。 このドキュメントは、ご自分の特定の環境をテストして基準に照らし合わせる方法を標準化するのに役立ちます。

このドキュメントでは、一貫性を保ちつつ簡単に、2 つのホスト間のネットワーク待機時間と帯域幅をテストする方法を説明します。 またこのドキュメントでは、Azure ネットワークの状態を確認し、問題点を特定する上で役立つ方法についてアドバイスを提供します。 ここで取り上げる PowerShell スクリプトとツールでは、ネットワーク上 (テストするリンクの両端) に 2 つのホストが必要です。 一方のホストは Windows Server または Windows デスクトップの必要があります。もう一方のホストには Windows または Linux のいずれかを使用できます。 

>[!NOTE]
>ここで使用するトラブルシューティングのアプローチ、ツール、および方法は、個人的な好みです。 このドキュメントでは、筆者が通常使用しているアプローチとツールについて説明します。 お読みいただいている方の中には別のアプローチをとる方もいらっしゃるでしょうが、問題解決のアプローチが違っていても問題はありません。 ただし、確立されたアプローチをお持ちでない場合は、このドキュメントを読んでネットワークの問題をトラブルシューティングする独自の方法、ツール、および好みを確立させていくことができます。
>
>

## <a name="network-components"></a>ネットワーク コンポーネント
トラブルシューティングの詳細を説明する前に、一般的な用語とコンポーネントについて見ていきましょう。 これにより、Azure の接続性を実現するエンドツーエンド チェーン内の各コンポーネントについて考えていくことができます。
[![1]][1]

最上位のレベルでは、次のネットワークの 3 つの主要なルーティング ドメインについて説明します。

- Azure ネットワーク (右の青色の雲)
- インターネットまたは WAN (中央の緑色の雲)
- 企業ネットワーク (左のピンク色の雲)

図を右から左へ見ていきながら、各コンポーネントについて簡単に説明します。
 - **仮想マシン** - サーバーに複数の NIC がある場合、すべての静的ルート、既定ルート、およびオペレーティング システムの設定が想定されたとおりにトラフィックを送受信しているかどうかを確認します。 また、各 VM の SKU には、帯域幅制限があります。 SKU のサイズが小さい VM を使用している場合、NIC で使用可能な帯域幅によってトラフィックが制限されます。 筆者の場合、テストには通常 DS5v2 を使用して VM に必要な帯域幅を確保し、テストが完了したら削除してコストを節約しています。
 - **NIC** - 問題の NIC に割り当てられているプライベート IP を把握していることを確認します。
 - **NIC の NSG** - NIC レベルで適用されている特定の NSG がある場合、渡そうとしているトラフィックに対して NSG のルール セットが適切であることを確認します。 たとえば、iPerf 用のポート 5201、RDP 用のポート 3389、または SSH 用のポート 22 が開いていて、テスト トラフィックを渡すことが許可されていることを確認します。
 - **VNet のサブネット** - NIC は特定のサブネットに割り当てられます。NIC がどのサブネットに割り当てられているか、およびそのサブネットに関連付けられているルールについて把握していることを確認します。
 - **サブネットの NSG** - NIC と同様に、サブネットにも NSG を適用できます。 渡そうとしているトラフィックに対して NSG のルール セットが適切であることを確認します  (NIC への受信トラフィックには、最初にサブネットの NSG、次に NIC の NSG が適用されます。一方、VM からの送信トラフィックには、最初に NIC の NSG、次にサブネットの NSG が適用されます)。
 - **サブネットの UDR** - ユーザー定義ルートでは、中間のホップ (ファイアウォールやロードバランサーなど) にトラフィックを向けることができます。 トラフィックに UDR があるかどうかを把握し、UDR がある場合にはどのようなルートをとるか、また UDR の次のホップがトラフィックに対してどのような働きをするかを把握するようにします  (たとえば、同じ 2 つのホスト間のトラフィックでもファイアウォールに許可されるトラフィックもあれば、拒否されるトラフィックもあります)。
 - **ゲートウェイ サブネット、NSG、UDR** - VM サブネットと同様に、ゲートウェイ サブネットも NSG と UDR を持つことができます。 ゲートウェイ サブネットに NSG と UDR があるかどうか、また NSG と UDR がトラフィックに対してどのような影響が与えるかを把握するようにします。
 - **VNet ゲートウェイ (ExpressRoute)** - ピアリング (ExpressRoute) または VPN を有効にすると、トラフィックのルーティングの有無や方法に影響を与える設定はあまりありません。 同じ VNet ゲートウェイに接続している ExpressRoute 回線や VPN トンネルが複数ある場合は、接続優先度設定に注意する必要があります。それは、この設定が接続の設定とトラフィックのパスに影響を与えるためです。
 - **ルート フィルター** (表示されていません) - ルート フィルターは、ExpressRoute で Microsoft ピアリングにのみ適用されますが、Microsoft ピアリングで期待するルートが表示されないときに確認する際に重要です。 

ここで、リンクの WAN の部分に来ました。 このルーティング ドメインは、サービス プロバイダー、会社の WAN、またはインターネットの場合があります。 これらのリンクには多くのホップ、テクノロジ、および企業が関与しているため、トラブルシューティングが困難になる場合があります。 多くの場合、この一連の企業とホップに取り組む前に、まず Azure ネットワークと企業ネットワークの両方に問題がないかを確認する作業をします。

上の図の一番左にあるのが企業ネットワークです。 会社の規模によって、このルーティング ドメインはユーザーと WAN との間のいくつかのネットワーク デバイスである場合もあれば、キャンパスまたは企業ネットワーク内の複数レイヤーのデバイスである場合もあります。

これら 3 つの異なる上位ネットワーク環境の複雑さを考慮すると、通常はネットワークの端から開始して、パフォーマンスが良好な部分やパフォーマンスの低下がみられる部分を明らかにしていくことが最適と言えます。 このアプローチを使用すると、3 つのうちどのルーティング ドメインに問題があるかを特定し、その特定の環境に的を絞ってトラブルシューティングを実行できます。

## <a name="tools"></a>ツール
ネットワークのほとんどの問題は、ping や traceroute などの基本的なツールを使用して分析し、特定できます。 Wireshark などのパケット分析にまで進む必要があることはほとんどありません。 トラブルシューティングに役立つように、Azure Connectivity Toolkit (AzureCT) が開発されて、これらのツールの一部が使いやすいパッケージにまとめられました。 筆者はパフォーマンスのテストに、iPerf と PSPing を使用します。 iPerf は一般的に使用されるツールであり、ほとんどの OS で動作します。 また基本的なパフォーマンス テストに適しており、簡単に使えます。 PSPing は、SysInternals によって開発された ping ツールです。 PSPing を使用すると、ICMP と TCP の ping をまとめて簡単に実行したり、コマンドを簡単に使ったりすることができます。 これらのツールはどちらも軽量で、ホスト上のディレクトリにファイルをコピーするだけで "インストール" できます。

これらのツールと方法をすべて PowerShell モジュール (AzureCT) にまとめましたので、インストールしてご利用いただけます。

### <a name="azurect---the-azure-connectivity-toolkit"></a>AzureCT - Azure Connectivity Toolkit
AzureCT の PowerShell モジュールには、[可用性テスト][Availability Doc]と[パフォーマンス テスト][Performance Doc]の 2 つのコンポーネントが含まれています。 このドキュメントで取り上げるのはパフォーマンス テストのみのため、この PowerShell モジュールの 2 つの LinkPerformance コマンドに注目しましょう。

このツールキットをパフォーマンス テストに使用する基本的な手順は次の 3 つです。 1) PowerShell モジュールのインストール、2) iPerf と PSPing の関連アプリケーションのインストール、3) パフォーマンス テストの実行です。

1. PowerShell モジュールのインストール

    ```powershell
    (new-object Net.WebClient).DownloadString("https://aka.ms/AzureCT") | Invoke-Expression
    
    ```

    このコマンドは、PowerShell モジュールをダウンロードしてローカルにインストールします。

2. 関連アプリケーションのインストール
    ```powershell
    Install-LinkPerformance
    ```
    AzureCT のこのコマンドにより、iPerf と PSPing が新しいディレクトリ "C:\ACTTools" にインストールされます。また、Windows ファイアウォールのポートが開き、ICMP とポート 5201 (iPerf) のトラフィックが許可されます。

3. パフォーマンス テストの実行

    まず、リモート ホストに iPerf をインストールし、サーバー モードで実行する必要があります。 また、リモート ホストはポート 3389 (Windows の RDP) またはポート 22 (Linux の SSH) のいずれかでリッスンし、iPerf 用のポート 5201 でトラフィックが許可されていることを確認します。 リモート ホストが Windows の場合は、AzureCT をインストールし、Install-LinkPerformance コマンドを実行して、iPerf の設定、および iPerf をサーバー モードで正常に起動させるために必要なファイアウォール規則の設定を行います。 
    
    リモート マシンの準備ができたら、ローカル コンピューターで PowerShell を開き、テストを開始します。
    ```powershell
    Get-LinkPerformance -RemoteHost 10.0.0.1 -TestSeconds 10
    ```

    このコマンドにより、負荷と待機時間に関する一連のテストが同時に実行されます。これらのテストは、ネットワーク リンクの帯域幅容量と待機時間の評価に役立ちます。

4. テストの出力の確認

    PowerShell の出力形式は次のようになります。

    [![4]][4]

    iPerf と PSPing のすべてのテストの詳細な結果は、AzureCT ツールのディレクトリ (C:\ACTTools) 内の、個々のテキスト ファイルで確認できます。

## <a name="troubleshooting"></a>トラブルシューティング
パフォーマンス テストで期待した結果が得られない場合、その理由を明らかにするには手順を踏んだ段階的なプロセスが必要です。 パス内のコンポーネント数を考慮すると、あちこちの疑わしい箇所について不必要かもしれないテストを何度も繰り返すよりも、体系的なアプローチをとるほうが速やかな解決につながります。

>[!NOTE]
>ここで紹介するシナリオはパフォーマンスの問題であり、接続の問題ではありません。 トラフィックがまったく流れない場合は手順が異なります。
>
>

まず、想定を見直します。 期待は妥当でしょうか。 たとえば、ExpressRoute 回線が 1 Gbps で待機時間が 100 ミリ秒の場合、待機時間が長いリンクでの TCP のパフォーマンス特性を考慮すると、1 Gbps の完全なトラフィックを期待することは妥当とは言えません。 パフォーマンスの想定については、「[参照](#references)」をご覧ください。

次に、ルーティング ドメイン間の端の部分から始めて、企業ネットワーク、WAN、または Azure ネットワークのうち、1 つの主要なルーティング ドメインに問題を絞り込むことをお勧めします。 パス内の "ブラック ボックス" が原因だと見なされがちですが、ブラック ボックスのせいにするのは簡単なものの、実は改善できる領域に問題がある場合は特に、解決を大幅に遅らせる可能性があります。 サービス プロバイダーや ISP に任せる前に、しかるべき努力を払うようにしましょう。

問題が含まれるとみられる主要なルーティング ドメインを特定したら、問題の領域を表す図を作成する必要があります。 ホワイトボード、メモ帳、Visio のいずれでも結構です。この図を具体的な "作戦図" として使用することで、問題をさらに絞り込む系統的なアプローチをとることができます。 テスト ポイントを計画し、テストが進むにつれて、問題がないことが明らかになった領域や詳細なテスト結果をもとに作戦図を更新できます。

図を作成したら、まずネットワークをセグメントに分けて、問題を絞り込みます。 機能している箇所と機能していない箇所を調べます。 テスト ポイントを移動させて、問題となっているコンポーネントを特定します。

また、OSI モデルの他の層も忘れずに確認します。 ネットワークと 1 - 3 層 (物理、データ、およびネットワーク層) に的を絞りがちですが、アプリケーション レイヤーの 7 層にも問題が見つかる場合があります。 先入観を抱かずに、想定を確認します。

## <a name="advanced-expressroute-troubleshooting"></a>ExpressRoute の高度なトラブルシューティング
クラウドの端の部分が具体的にどこなのかわからないと、Azure コンポーネントの特定が難しくなります。 ExpressRoute を使用している場合は、Microsoft Enterprise Edge (MSEE) と呼ばれるネットワーク コンポーネントが端になります。 **ExpressRoute を使用している場合**、MSEE は Microsoft ネットワークへの最初のコンタクト ポイントであり、また Microsoft ネットワークから離れる最後のホップでもあります。 VNet ゲートウェイと ExpressRoute 回線の間に接続オブジェクトを作成すると、実際には MSEE への接続を作成することになります。 MSEE を (進行方向がどちらかに応じて) 最初または最後のホップと認識することは、Azure ネットワークの問題を特定し、問題が Azure 内にあるのか、それともダウンストリームの WAN や企業ネットワーク内にあるのかを明らかにする上で極めて重要です。 

[![2]][2]

>[!NOTE]
> MSEE は Azure クラウド内にないことに注意してください。 ExpressRoute は実際には Microsoft ネットワークの端に位置し、Azure 内にはありません。 ExpressRoute を使用して MSEE に接続すると、Microsoft のネットワークに接続します。そこから Office 365 (Microsoft ピアリング経由) や Azure (プライベート ピアリングや Microsoft ピアリング経由) などのクラウド サービスのどれにでも移動できます。
>
>

2 つの VNet (図の VNet A と VNet B) を**同じ** ExpressRoute 回線に接続すると、一連のテストを実行して Azure 内の問題を特定したり、Azure 内には問題がないことを明らかにしたりすることができます。
 
### <a name="test-plan"></a>テスト計画
1. VM1 と VM2 の間で Get-LinkPerformance テストを実行します。 このテストは、問題がローカルかそうでないかについての分析情報を提供します。 このテストで待機時間と帯域幅について許容できる結果が得られた場合、ローカルの VNet ネットワークの状態は良好だと見なすことができます。
2. ローカルの VNet トラフィックの状態は良好だと想定し、VM1 と VM3 の間で Get-LinkPerformance テストを実行します。 このテストでは、Microsoft ネットワークから MSEE を経由し Azure に戻るまでの接続が実行されます。 このテストで待機時間と帯域幅について許容できる結果が得られた場合、Azure ネットワークの状態は良好だと見なすことができます。
3. Azure に問題がないことが明らかになったら、企業ネットワークでも同様の一連のテストを実行できます。 このテストでも問題がない場合には、サービス プロバイダーや ISP と連携して WAN の接続を診断します。 例:2 つのブランチ オフィス間、またはデスクとデータ センター サーバーの間でこのテストを実行します。 何をテストするかに応じて、そのパスを実行できるエンドポイント (サーバー、PC など) を見つけます。

>[!IMPORTANT]
> テストごとにそのテストを実行した日時を記録し、結果を共通の場所 (OneNote や Excel がお勧めです) に保存しておくことが重要です。 各テストの実行の出力形式を統一することで、テストの実行間の結果データを比較でき、データに "漏れ" がないようにする必要があります。 複数のテスト間で一貫性を維持できることが、トラブルシューティングに AzureCT を使用する主な理由です。 実行する負荷シナリオそのものにマジックがあるわけではありませんが、どのテストからも*一貫したテストおよびデータの出力*が得られるという事実こそが*マジック*なのです。 毎回日時が記録されて一貫したデータが得られることは、後になって問題が散発的であることが明らかになった場合に特に役立ちます。 前もってきちんとデータを収集しておくと、何時間もかけて同じシナリオをテストし直す手間を省けます (筆者は何年も前にこの苦労を味わいました)。
>
>

## <a name="the-problem-is-isolated-now-what"></a>問題特定後の手順
問題を絞り込むとより簡単に解決できますが、トラブルシューティングでこれ以上は進めないという段階に達することがよくあります。 例: サービス プロバイダーのリンクがヨーロッパを経由するホップを通っているが、期待されるパスはすべてアジアにある場合。 このような段階では、サポートを求める必要があります。 サポートを求める相手は、問題が特定されたルーティング ドメイン (または特定のコンポーネントに問題を絞り込めることが望ましい) によって異なります。

企業ネットワークの問題については、ネットワークをサポートする社内の IT 部門またはサービス プロバイダー (ハードウェアの製造元の場合もあります) が、デバイスの構成やハードウェアの修理をサポートできるかもしれません。

WAN では、テスト結果をサービス プロバイダーや ISP と共有することが彼らの取り組みに役立ち、すでにテスト済みの領域を再度調べる手間を省くことができます。 ただし、彼らが結果を自分たちで確認しようとしても気を悪くしないでください。 他の人の報告結果をもとにトラブルシューティングを行う場合、"信頼しても確認する" ことが優れた方法です。

Azure では、可能な限り詳しく問題を特定したら、[Azure ネットワークのドキュメント][Network Docs]をご覧ください。その上でなおサポートが必要な場合は[サポート チケット][Ticket Link]を開いてください。

## <a name="references"></a>参照
### <a name="latencybandwidth-expectations"></a>待機時間と帯域幅の期待値
>[!TIP]
> 待機時間の最大の要素は、テストするエンドポイント間の地理的要因による待機時間 (マイルまたはキロメートル) です。 機器的要因による待機時間 (物理および仮想コンポーネント、ホップ数など) も関係するものの、WAN 接続の処理では、地理的要因が待機時間全体における最大の要素であることが明らかになっています。 また、ここでいう距離とは、直線距離や道路地図上の距離ではなく、設置されるファイバーの距離であることに注意する必要があります。 この距離を正確に取得することは極めて困難です。 そのため、筆者は通常インターネットで都市間の距離計算機を使用しています。この方法では測定精度が非常に低くなりますが、大まかな期待値を設定するには十分です。
>
>

筆者は米国ワシントン州シアトルで ExpressRoute をセットアップしました。 次の表は、Azure のさまざまな場所に対するテストで確認された待機時間と帯域幅を示しています。 テストする各エンドポイント間の地理的距離は、筆者が推定したものです。

テストの設定
 - Windows Server 2016 を稼働している物理サーバーには、10 Gbps の NIC が 1 枚搭載され、ExpressRoute 回線に接続されています。
 - 特定された場所の ExpressRoute 回線は 10Gbps の Premium で、プライベート ピアリングが有効になっています。
 - 指定されたリージョンには、UltraPerformance ゲートウェイを備えた Azure VNet があります。
 - VNet では、DS5v2 の VM で Windows Server 2016 が稼働しています。 この VM はドメインに参加しておらず、最適化もカスタマイズもされていない既定の Azure イメージから作成され、AzureCT がインストールされています。
 - すべてのテストでは AzureCT の Get-LinkPerformance コマンドが使用され、6 回のテストの実行それぞれで、5 分間のロード テストが行われました。 例: 

    ```powershell
    Get-LinkPerformance -RemoteHost 10.0.0.1 -TestSeconds 300
    ```
 - 各テストのデータ フローには、オンプレミスの物理サーバー (シアトルの iPerf クライアント) から Azure VM (一覧表示されている Azure リージョンの iPerf サーバー) までフローする負荷がありました。
 - "待機時間" 列のデータは、ロードなしテスト (iPerf を実行しない TCP の待機時間テスト) から得られたデータです。
 - "最大帯域幅" 列のデータは、ウィンドウ サイズが 1 MB の、16 の TCP フローのロード テストから得られたデータです。

[![3]][3]

### <a name="latencybandwidth-results"></a>待機時間と帯域幅の結果
>[!IMPORTANT]
> これらの数値は一般的な参照用です。 待機時間には多くの要因が影響を与えます。これらの値は時間が経過しても概ね一貫しているものの、Azure やサービス プロバイダーのネットワーク内の条件ではいつでも別のパスを使用してトラフィックを送信できるため、待機時間と帯域幅が影響を受ける可能性があります。 一般に、このような変更による影響では大きな違いは生じません。
>
>

| | | | | | |
|-|-|-|-|-|-|
|ExpressRoute<br/>場所|Azure<br/>リージョン|推定<br/>距離 (km)|Latency|1 セッションの<br/>帯域幅|最大値<br/>帯域幅|
| シアトル | 米国西部 2        |    191 km |   5 ミリ秒 | 262.0 メガビット/秒 |  3.74 ギガビット/秒 | 21
| シアトル | 米国西部          |  1,094 km |  18 ミリ秒 |  82.3 メガビット/秒 |  3.70 ギガビット/秒 | 20
| シアトル | 米国中央部       |  2,357 km |  40 ミリ秒 |  38.8 メガビット/秒 |  2.55 ギガビット/秒 | 17
| シアトル | 米国中南部 |  2,877 km |  51 ミリ秒 |  30.6 メガビット/秒 |  2.49 ギガビット/秒 | 19
| シアトル | 米国中北部 |  2,792 km |  55 ミリ秒 |  27.7 メガビット/秒 |  2.19 ギガビット/秒 | 18
| シアトル | 米国東部 2        |  3,769 km |  73 ミリ秒 |  21.3 メガビット/秒 |  1.79 ギガビット/秒 | 16
| シアトル | 米国東部          |  3,699 km |  74 ミリ秒 |  21.1 メガビット/秒 |  1.78 ギガビット/秒 | 15
| シアトル | 東日本       |  7,705 km | 106 ミリ秒 |  14.6 メガビット/秒 |  1.22 ギガビット/秒 | 28
| シアトル | 英国南部         |  7,708 km | 146 ミリ秒 |  10.6 メガビット/秒 |   896 メガビット/秒 | 24
| シアトル | 西ヨーロッパ      |  7,834 km | 153 ミリ秒 |  10.2 メガビット/秒 |   761 メガビット/秒 | 23
| シアトル | オーストラリア東部   | 12,484 km | 165 ミリ秒 |   9.4 メガビット/秒 |   794 メガビット/秒 | 26
| シアトル | 東南アジア   | 12,989 km | 170 ミリ秒 |   9.2 メガビット/秒 |   756 メガビット/秒 | 25
| シアトル | ブラジル南部*   | 10,930 km | 189 ミリ秒 |   8.2 メガビット/秒 |   699 メガビット/秒 | 22
| シアトル | インド南部      | 12,918 km | 202 ミリ秒 |   7.7 メガビット/秒 |   634 メガビット/秒 | 27

\* ブラジルの待機時間は、直線距離と設置されたファイバーの距離が大きく異なることを表す好例です。 160 ミリ秒前後の待機時間が予想されましたが、実際には 189 ミリ秒でした。 予想に対するこの違いは、ネットワークのどこかに問題があることを示す場合もありますが、ほとんどの場合は設置されたファイバーがブラジルまで直線で通っておらず、シアトルからブラジルに到達するまでに 1,000 km ほど余分にかかっていることを示しています。

## <a name="next-steps"></a>次の手順
1. Azure Connectivity Toolkit を GitHub ([http://aka.ms/AzCT][ACT]) からダウンロードします
2. [リンクのパフォーマンス テスト][Performance Doc]の手順に従います

<!--Image References-->
[1]: ./media/expressroute-troubleshooting-network-performance/network-components.png "Azure ネットワーク コンポーネント"
[2]: ./media/expressroute-troubleshooting-network-performance/expressroute-troubleshooting.png "ExpressRoute のトラブルシューティング"
[3]: ./media/expressroute-troubleshooting-network-performance/test-diagram.png "パフォーマンス テスト環境"
[4]: ./media/expressroute-troubleshooting-network-performance/powershell-output.png "PowerShell の出力"

<!--Link References-->
[Performance Doc]: https://github.com/Azure/NetworkMonitoring/blob/master/AzureCT/PerformanceTesting.md
[Availability Doc]: https://github.com/Azure/NetworkMonitoring/blob/master/AzureCT/AvailabilityTesting.md
[Network Docs]: https://docs.microsoft.com/azure/index#pivot=services&panel=network
[Ticket Link]: https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview
[ACT]: http://aka.ms/AzCT











