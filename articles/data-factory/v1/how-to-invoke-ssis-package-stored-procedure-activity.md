---
title: "使用 Azure Data Factory 預存程序活動的 SSIS 封裝叫用 |Microsoft 文件"
description: "本文說明如何叫用 SQL Server Integration Services (SSIS) 封裝，從 Azure Data Factory 管線使用預存程序活動。"
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: spelluru
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: 
ms.devlang: powershell
ms.topic: article
ms.date: 12/07/2017
ms.author: jingwang
ms.openlocfilehash: 8aabe45a1627b1a897ca9fe4bda581c7a3f6bc03
ms.sourcegitcommit: 901a3ad293669093e3964ed3e717227946f0af96
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/21/2017
---
# <a name="invoke-an-ssis-package-using-stored-procedure-activity-in-azure-data-factory"></a>使用 Azure Data Factory 中的預存程序活動的 SSIS 封裝叫用
本文說明如何使用預存程序活動叫用 SSIS 封裝從 Azure Data Factory 管線。 

> [!NOTE]
> 本文適用於第 1 版的 Data Factory，這是正式推出。 如果您使用第 2 版的 Data Factory 服務處於公開預覽狀態，請參閱[叫用的 SSIS 封裝中第 2 版使用預存程序活動](../how-to-invoke-ssis-package-stored-procedure-activity.md)。

## <a name="prerequisites"></a>必要條件

### <a name="azure-sql-database"></a>連接字串 
這篇文章中的逐步解說會使用 Azure SQL database 主控 SSIS 目錄。 您也可以使用 Azure SQL Managed 執行個體 （私人預覽中）。

### <a name="create-an-azure-ssis-integration-runtime"></a>建立 Azure-SSIS 整合執行階段
如果您沒有依照中的逐步指示，請建立 Azure SSIS 整合執行階段[教學課程： 部署 SSIS 封裝](../tutorial-deploy-ssis-packages-azure.md)。 您必須建立 data factory 的版本來建立 Azure SSIS 整合執行階段 2。 

### <a name="azure-powershell"></a>Azure PowerShell
依照[如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)中的指示，安裝最新的 Azure PowerShell 模組。

## <a name="create-a-data-factory"></a>建立 Data Factory
下列程序提供建立 data factory 的步驟。 您建立此 data factory 中的預存程序 」 活動的管線。 預存程序活動會執行預存程序中執行 SSIS 封裝 SSISDB 資料庫。

1. 定義資源群組名稱的變數，以便稍後在 PowerShell 命令中使用。 將下列命令文字複製到 PowerShell，以雙引號指定 [Azure 資源群組](../../azure-resource-manager/resource-group-overview.md)的名稱，然後執行命令。 例如：`"adfrg"`。 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    如果資源群組已經存在，您可能不想覆寫它。 將不同的值指派給 `$ResourceGroupName` 變數，然後執行一次命令
2. 若要建立 Azure 資源群組，請執行下列命令： 

    ```powershell
    $ResGrp = New-AzureRmResourceGroup $resourceGroupName -location 'eastus'
    ``` 
    如果資源群組已經存在，您可能不想覆寫它。 將不同的值指派給 `$ResourceGroupName` 變數，然後執行一次命令。 
3. 定義 Data Factory 名稱的變數。 

    > [!IMPORTANT]
    >  將資料處理站名稱更新為全域唯一的。 

    ```powershell
    $DataFactoryName = "ADFTutorialFactory";
    ```

5. 若要建立的 data factory，執行下列命令**新增 AzureRmDataFactory**指令程式，使用從 $ResGrp 變數的位置和 ResourceGroupName 屬性： 
    
    ```powershell       
    $df = New-AzureRmDataFactory -ResourceGroupName $ResourceGroupName -Name $dataFactoryName -Location "East US"
    ```

請注意下列幾點：

* Azure Data Factory 的名稱在全域必須是唯一的。 如果發生下列錯誤，請變更名稱，並再試一次。

    ```
    The specified Data Factory name 'ADFTutorialFactory' is already in use. Data Factory names must be globally unique.
    ```
* 若要建立 Data Factory 執行個體，您用來登入 Azure 的使用者帳戶必須為**參與者**或**擁有者**角色，或是 Azure 訂用帳戶的**管理員**。

### <a name="create-an-azure-sql-database-linked-service"></a>建立 Azure SQL Database 連結服務
建立連結的服務，將連結您的 Azure SQL database 裝載您的 data factory 的 SSIS 目錄。 Data Factory 會使用這項連結服務中的資訊，來連接到 SSISDB 資料庫，並執行執行 SSIS 封裝的預存程序。 

1. 建立名為的 JSON 檔案**AzureSqlDatabaseLinkedService.json**中**C:\ADF\RunSSISPackage**資料夾具有下列內容： 

    > [!IMPORTANT]
    > 取代&lt;servername&gt;， &lt;username&gt;@&lt;servername&gt;和&lt;密碼&gt;與您的 Azure SQL Database 之前的值儲存檔案。

    ```json
    {
        "name": "AzureSqlDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=SSISDB;User ID=<username>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
        }
    ```
2. 在**Azure PowerShell**，切換至**C:\ADF\RunSSISPackage**資料夾。
3. 執行**新增 AzureRmDataFactoryLinkedService** cmdlet 來建立連結的服務： **AzureSqlDatabaseLinkedService**。 

    ```powershell
    New-AzureRmDataFactoryLinkedService $df -File ".\AzureSqlDatabaseLinkedService.json"
    ```

## <a name="create-an-output-dataset"></a>建立輸出資料表
此輸出資料集是空的資料集，磁碟機管線的排程。 請注意，頻率設定為 Hour 和 interval 設為 1。 因此，在管線執行之後，當管線內的一小時開始與結束時間。 

1. 建立 OuputDataset.json 檔案具有下列內容： 
    
    ```json
    {
        "name": "sprocsampleout",
        "properties": {
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": { },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```
2. 執行**新增 AzureRmDataFactoryDataset** cmdlet 來建立資料集。 

    ```powershell
    New-AzureRmDataFactoryDataset $df -File ".\OutputDataset.json"
    ```

## <a name="create-a-pipeline-with-stored-procedure-activity"></a>建立與預存程序活動的管線 
在此步驟中，您可以建立管線的預存程序 」 活動。 在活動叫用執行 SSIS 封裝的 sp_executesql 預存程序。 

1. 建立名為的 JSON 檔案**MyPipeline.json**中**C:\ADF\RunSSISPackage**資料夾具有下列內容：

    > [!IMPORTANT]
    > 取代&lt;資料夾名稱&gt;，&lt;專案名稱&gt;，&lt;封裝名稱&gt;與資料夾、 專案和儲存檔案之前 SSIS 目錄中的封裝的名稱。

    ```json
    {
        "name": "MyPipeline",
        "properties": {
            "activities": [{
                "name": "SprocActivitySample",
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "sp_executesql",
                    "storedProcedureParameters": {
                        "stmt": "DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<folder name>', @project_name=N'<project name>', @package_name=N'<package name>', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END"
                    }
                },
                "outputs": [{
                    "name": "sprocsampleout"
                }],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                }
            }],
            "start": "2017-10-01T00:00:00Z",
            "end": "2017-10-01T05:00:00Z",
            "isPaused": false
        }
    }    
    ```

2. 若要建立管線： **RunSSISPackagePipeline**，請執行**新增 AzureRmDataFactoryPipeline** cmdlet。

    ```powershell
    $DFPipeLine = New-AzureRmDataFactoryPipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "RunSSISPackagePipeline" -DefinitionFile ".\RunSSISPackagePipeline.json"
    ```

## <a name="monitor-the-pipeline-run"></a>監視管道執行

2. 執行**Get AzureRmDataFactorySlice**以取得詳細的輸出資料集 * *，這是輸出資料表的管線的所有配量。

    ```PowerShell
    Get-AzureRmDataFactorySlice $df -DatasetName sprocsampleout -StartDateTime 2017-10-01T00:00:00Z
    ```
    請注意，您在此處指定的 StartDateTime 與在管線 JSON 中指定的開始時間是相同的。 
3. 執行 **Get-AzureRmDataFactoryRun** ，以取得特定配量的活動執行詳細資料。

    ```PowerShell
    Get-AzureRmDataFactoryRun $df -DatasetName sprocsampleout -StartDateTime 2017-10-01T00:00:00Z
    ```

    您可以繼續執行此 Cmdlet 直到您看到配量處於**就緒**狀態或**失敗**狀態。 

    您可以執行下列查詢針對 SSISDB 資料庫可讓您確認您 Azure SQL server 中執行封裝。 

    ```sql
    select * from catalog.executions
    ```

## <a name="next-steps"></a>後續步驟
如需有關預存程序活動的詳細資訊，請參閱[預存程序活動](data-factory-stored-proc-activity.md)發行項。

