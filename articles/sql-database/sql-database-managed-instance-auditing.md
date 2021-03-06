---
title: Azure SQL Database Managed Instance の監査 | Microsoft Docs
description: T-SQL を使って Azure SQL Database Managed Instance の監査を始める方法について説明します
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
f1_keywords:
- mi.azure.sqlaudit.general.f1
author: vainolo
ms.author: vainolo
ms.reviewer: vanto
manager: craigg
ms.date: 09/20/2018
ms.openlocfilehash: 045314980d0051e8b5ef71bdf95023084eff1880
ms.sourcegitcommit: 3ab534773c4decd755c1e433b89a15f7634e088a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2019
ms.locfileid: "54063879"
---
# <a name="get-started-with-azure-sql-database-managed-instance-auditing"></a>Azure SQL Database Managed Instance の監査の概要

[Azure SQL Database Managed Instance](sql-database-managed-instance.md) の監査は、データベース イベントを追跡し、Azure ストレージ アカウントの監査ログにイベントを書き込みます。 また、監査によって以下を行うことができます。

- 規定コンプライアンスの維持、データベース活動の理解、およびビジネス上の懸念やセキュリティ違犯の疑いを示す差異や異常に対する洞察が容易になります。
- コンプライアンスを保証するものではありませんが、標準へのコンプライアンスを強化します。 標準準拠をサポートする Azure プログラムの詳細については、 [Azure セキュリティ センター](https://azure.microsoft.com/support/trust-center/compliance/)のページを参照してください。

## <a name="set-up-auditing-for-your-server-to-azure-storage"></a>Azure Storage に対するサーバー監査の設定 

以下のセクションでは、マネージド インスタンスの監査の構成について説明します。

1. [Azure ポータル](https://portal.azure.com)にアクセスします。
2. 次の手順では、監査ログが格納される Azure Storage **コンテナー**を作成します。

   - 監査ログを格納する Azure Storage に移動します。

     > [!IMPORTANT]
     > リージョンをまたいで読み取り/書き込みが行われないように、マネージド インスタンス サーバーと同じリージョンのストレージ アカウントを使います。

   - ストレージ アカウントで **[概要]** に移動し、**[BLOB]** をクリックします。

     ![ナビゲーション ウィンドウ][1]

   - 上部のメニューで、**[+ コンテナー]** をクリックして新しいコンテナーを作成します。

     ![ナビゲーション ウィンドウ][2]

   - コンテナーの **[名前]** を指定し、パブリック アクセス レベルを **[プライベート]** に設定して、**[OK]** をクリックします。

     ![ナビゲーション ウィンドウ][3]

   - コンテナーの一覧で新しく作成されたコンテナーをクリックし、**[コンテナーのプロパティ]** をクリックします。

     ![ナビゲーション ウィンドウ][4]

   - コピー アイコンをクリックしてコンテナーの URL をコピーし、後で使えるように (メモ帳などに) URL を保存します。 コンテナー URL は、`https://<StorageName>.blob.core.windows.net/<ContainerName>` という形式になっている必要があります。

     ![ナビゲーション ウィンドウ][5]

3. 次の手順では、マネージド インスタンスの監査アクセス権をストレージ アカウントに付与するために使われる Azure Storage の **SAS トークン**を生成します。

   - 前の手順でコンテナーを作成した Azure ストレージ アカウントに移動します。

   - [ストレージの設定] メニューの **[Shared Access Signature]** をクリックします。

     ![ナビゲーション ウィンドウ][6]

   - 次に示すように SAS を構成します。
     - **使用できるサービス**:BLOB
     - **開始日**: タイム ゾーンに関連する問題を回避するため、前日の日付を使うことをお勧めします。
     - **終了日**: この SAS トークンの有効期限が切れる日付を選びます。 

       > [!NOTE]
       > 監査の失敗を避けるため、トークンの期限が切れたら更新します。

     - **[SAS の生成]** をクリックします。

       ![ナビゲーション ウィンドウ][7]

   - [SAS の生成] をクリックすると、SAS トークンが下部に表示されます。 コピー アイコンをクリックしてトークンをコピーし、後で使えるように (メモ帳などに) 保存します。

     > [!IMPORTANT]
     > トークンの先頭にある疑問符 ("?") を削除します。

     ![ナビゲーション ウィンドウ][8]

4. SQL Server Management Studio (SSMS) を使ってマネージド インスタンスに接続します。

5. 次の T-SQL ステートメントを実行し、前の手順で作成したコンテナー URL と SAS トークンを使って、**新しい資格情報を作成**します。

    ```SQL
    CREATE CREDENTIAL [<container_url>]
    WITH IDENTITY='SHARED ACCESS SIGNATURE',
    SECRET = '<SAS KEY>'
    GO
    ```

6. 次の T-SQL ステートメントを実行し、新しいサーバー監査を作成します (独自の監査名を選び、前の手順で作成したコンテナー URL を使います)。

    ```SQL
    CREATE SERVER AUDIT [<your_audit_name>]
    TO URL ( PATH ='<container_url>' [, RETENTION_DAYS =  integer ])
    GO
    ```

    `RETENTION_DAYS` を指定しないと、既定値の 0 (無制限のリテンション期間) になります。

    以下の追加情報をご覧ください。
    - [マネージド インスタンス、Azure SQL データベース、SQL Server での監査の相違点](#auditing-differences-between-managed-instance-azure-sql-database-and-sql-server)
    - [CREATE SERVER AUDIT](https://docs.microsoft.com/sql/t-sql/statements/create-server-audit-transact-sql)
    - [ALTER SERVER AUDIT](https://docs.microsoft.com/sql/t-sql/statements/alter-server-audit-transact-sql)

7. SQL Server の場合と同様に、サーバー監査仕様またはデータベース監査仕様を作成します。
    - [サーバー監査仕様の作成 T-SQL ガイド](https://docs.microsoft.com/sql/t-sql/statements/create-server-audit-specification-transact-sql)
    - [データベース監査仕様の作成 T-SQL ガイド](https://docs.microsoft.com/sql/t-sql/statements/create-database-audit-specification-transact-sql)

8. 手順 6 で作成したサーバー監査を有効にします。

    ```SQL
    ALTER SERVER AUDIT [<your_audit_name>]
    WITH (STATE=ON);
    GO
    ```

## <a name="set-up-auditing-for-your-server-to-event-hub-or-log-analytics"></a>Event Hubs または Log Analytics に対するサーバー監査の設定

マネージド インスタンスからの監査ログは、Azure Monitor を使用して Event Hubs または Log Analytics に送信できます。 このセクションでは、この動作の構成方法について説明します。

1. [Azure Portal](https://portal.azure.com/) 内で SQL マネージド インスタンスに移動します。

2. **[診断設定]** をクリックします。

3. **[Turn on diagnostics]\(診断をオンにする\)** をクリックします。 診断が既に有効になっている場合は、*[+Add diagnostic setting]\(+ 診断設定を追加する\)* が代わりに表示されます。

4. ログのリストで **[SQLSecurityAuditEvents]** を選択します。

5. 監査イベント宛先として、[イベント ハブ]、[Log Analytics]、またはその両方を選択します。 必要なパラメーター (Log Analytics ワークスペースなど) をターゲットごとに構成します。

6. **[Save]** をクリックします。

  ![ナビゲーション ウィンドウ][9]

7. **SQL Server Management Studio (SSMS)** またはサポートされるその他のクライアントを使用して、マネージド インスタンスに接続します。

8. サーバー監査を作成するために次の T-SQL ステートメントを実行します。

    ```SQL
    CREATE SERVER AUDIT [<your_audit_name>] TO EXTERNAL_MONITOR;
    GO
    ```

9. SQL Server の場合と同様に、サーバー監査仕様またはデータベース監査仕様を作成します。

   - [サーバー監査仕様の作成 T-SQL ガイド](https://docs.microsoft.com/sql/t-sql/statements/create-server-audit-specification-transact-sql)
   - [データベース監査仕様の作成 T-SQL ガイド](https://docs.microsoft.com/sql/t-sql/statements/create-database-audit-specification-transact-sql)

10. 手順 7 で作成したサーバー監査を有効にします。
 
    ```SQL
    ALTER SERVER AUDIT [<your_audit_name>] WITH (STATE=ON);
    GO
    ```

## <a name="consume-audit-logs"></a>監査ログの使用

### <a name="consume-logs-stored-in-azure-storage"></a>Azure Storage に格納されたログの使用

BLOB 監査ログを表示するには、いくつかの方法が使用できます。

- システム関数 `sys.fn_get_audit_file` (T-SQL) を使って、表形式で監査ログ データを返します。 この関数の使用方法の詳細については、[sys.fn_get_audit_file のドキュメント](https://docs.microsoft.com/sql/relational-databases/system-functions/sys-fn-get-audit-file-transact-sql)を参照してください。

- Azure Storage Explorer などのツールを使用して、監査ログを調査できます。 Azure Storage では、監査ログは sqldbauditlogs という名前のコンテナー内に BLOB ファイルのコレクションとして保存されます。 ストレージ フォルダーの階層、命名規則、およびログ形式の詳細については、BLOB 監査ログ形式のリファレンスに関する記事を参照してください。

- 監査ログの使い方の完全な一覧については、「[SQL Database 監査の使用](https://docs.microsoft.com/ azure/sql-database/sql-database-auditing)」をご覧ください。

> [!IMPORTANT]
> 現在、マネージド インスタンスについては、Azure portal ([監査レコード] ウィンドウ) から監査レコードを表示することはできません。

### <a name="consume-logs-stored-in-event-hub"></a>イベント ハブに格納されているログの使用

イベント ハブの監査ログ データを使用するには、イベントを処理し、そのイベントをターゲットに書き込むようにストリームを設定する必要があります。 詳細については、Azure Event Hubs のドキュメントを参照してください。

### <a name="consume-and-analyze-logs-stored-in-log-analytics"></a>Log Analytics に格納されているログの使用および分析

監査ログが Log Analytics に書き込まれると、それらの監査ログが Log Analytics ワークスペースで使用可能になります。Log Analytics ワークスペースでは、監査データに対して高度な検索を実行できます。 その出発点として、Log Analytics に移動し、*[全般]* セクションの下で *[ログ]* をクリックし、単純なクエリ (例: `search "SQLSecurityAuditEvents"`) を入力して、監査ログを表示してみましょう。  

Log Analytics により、統合された検索とカスタム ダッシュボードを使用してオペレーション インサイトがリアルタイムで得られるため、ワークロードやサーバー全体に散在する何百万件のレコードもすぐに分析できます。 Log Analytics 検索言語およびコマンドに関する有用な追加情報については、[Log Analytics 検索リファレンス](https://docs.microsoft.com/azure/azure-monitor/log-query/log-query-overview)に関するページをご覧ください。

## <a name="auditing-differences-between-managed-instance-azure-sql-database-and-sql-server"></a>マネージド インスタンス、Azure SQL Database、SQL Server での監査の相違点

マネージド インスタンス、Azure SQL Database、オンプレミスの SQL Server の SQL 監査の主な相違点は次のとおりです。

- マネージド インスタンスでは、SQL 監査はサーバー レベルで機能し、Azure BLOB ストレージ アカウントに `.xel` ログ ファイルが保存されます。
- Azure SQL Database では、SQL 監査はデータベース レベルで機能します。
- オンプレミス/仮想マシンの SQL Server では、SQL 監査はサーバー レベルで機能しますが、イベントはファイル システム/Windows イベント ログに保存されます。

マネージド インスタンスの XEvent 監査では、対象として Azure Blob Storage をサポートしています。 ファイル ログと Windows ログは**サポートされていません**。

Azure Blob Storage の監査の `CREATE AUDIT` 構文の主な相違点は次のとおりです。

- 新しい `TO URL` 構文が用意されています。この構文を使って、`.xel` ファイルを配置する Azure Blob Storage コンテナーの URL を指定できます。
- イベント ハブおよび Log Analytics ターゲットを有効にするための新しい構文 `TO EXTERNAL MONITOR` が用意されています。
- マネージド インスタンスは Windows ファイル共有にアクセスできないため、`TO FILE` 構文は**サポートされていません**。
- Shutdown オプションは**サポートされていません**。
- `queue_delay` の値として 0 は**サポートされていません**。

## <a name="next-steps"></a>次の手順

- 監査ログの使い方の完全な一覧については、「[SQL Database 監査の使用](https://docs.microsoft.com/azure/sql-database/sql-database-auditing)」をご覧ください。
- 標準準拠をサポートする Azure プログラムの詳細については、 [Azure セキュリティ センター](https://azure.microsoft.com/support/trust-center/compliance/)のページを参照してください。

<!--Image references-->
[1]: ./media/sql-managed-instance-auditing/1_blobs_widget.png
[2]: ./media/sql-managed-instance-auditing/2_create_container_button.png
[3]: ./media/sql-managed-instance-auditing/3_create_container_config.png
[4]: ./media/sql-managed-instance-auditing/4_container_properties_button.png
[5]: ./media/sql-managed-instance-auditing/5_container_copy_name.png
[6]: ./media/sql-managed-instance-auditing/6_storage_settings_menu.png
[7]: ./media/sql-managed-instance-auditing/7_sas_configure.png
[8]: ./media/sql-managed-instance-auditing/8_sas_copy.png
[9]: ./media/sql-managed-instance-auditing/9_mi_configure_diagnostics.png
