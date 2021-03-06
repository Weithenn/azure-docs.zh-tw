---
title: "開始使用 Azure Active Directory Identity Protection 和 Microsoft Graph | Microsoft Docs"
description: "了解如何查詢 Microsoft Graph 取得的風險事件和從 Azure Active Directory 的相關的資訊清單。"
services: active-directory
keywords: "azure active directory identity protection, 風險事件, 弱點, 安全性原則, Microsoft Graph"
documentationcenter: 
author: MarkusVi
manager: mtillman
ms.assetid: fa109ba7-a914-437b-821d-2bd98e681386
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/08/2017
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: fafad74f46baaf56a8220dab05028781b2f2258e
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/11/2017
---
# <a name="get-started-with-azure-active-directory-identity-protection-and-microsoft-graph"></a>開始使用 Azure Active Directory Identity Protection 和 Microsoft Graph
Microsoft Graph 是 Microsoft 統一 API 端點，也是 [Azure Active Directory Identity Protection](active-directory-identityprotection.md) API 的寄居地。 第一個 API **identityRiskEvents** 可讓您查詢 Microsoft Graph，以取得[風險事件](active-directory-identityprotection-risk-events-types.md)清單和關聯的資訊。 本文可協助您開始查詢此 API。 如需深入的簡介、完整文件以及 Graph 總管的存取權，請參閱 [Microsoft Graph 網站](https://graph.microsoft.io/)。


有四個步驟來透過 Microsoft Graph 存取的識別身分保護資料：

1. 擷取您的網域名稱。
2. 建立新的應用程式註冊。 
2. 使用此密碼和其他幾項資訊向 Microsoft Graph 驗證，以從中收到驗證權杖。 
3. 使用此權杖對 API 端點發出要求，並取回 Identity Protection 資料。

開始之前，您需要下列項目：

* 系統管理員權限，以便在 Azure AD 中建立應用程式
* 租用戶網域的名稱 (例如 contoso.onmicrosoft.com)


## <a name="retrieve-your-domain-name"></a>擷取您的網域名稱 

1. [登入](https://portal.azure.com)您身為系統管理員的 Azure 入口網站。 

2. 在左側瀏覽窗格中，按一下  **Active Directory**。 
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/41.png)


3. 在**管理**區段中，按一下**屬性**。

    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/42.png)

4. 複製您的網域名稱。


## <a name="create-a-new-app-registration"></a>建立新的應用程式註冊

1. 在**Active Directory**頁面上，於**管理**區段中，按一下**應用程式註冊**。

    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/42.png)


2. 在頂端功能表中，按一下 **新應用程式註冊**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/43.png)

3. 在**建立**頁面上，執行下列步驟：
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/44.png)

    a. 在 [名稱] 文字方塊中，輸入應用程式的名稱 (例如︰AADIP Risk Event API Application)。
   
    b. 在 [類型]，選取 [Web 應用程式和/或 Web API]。
   
    c. 在 [登入 URL] 文字方塊中，輸入 `http://localhost`。

    d. 按一下頁面底部的 [新增] 。

4. 若要開啟**設定**頁面上，在應用程式清單中，按一下您剛建立的應用程式的註冊。 

5. 複製**應用程式識別碼**。


## <a name="grant-your-application-permission-to-use-the-api"></a>授與您的應用程式使用 API 的權限

1. 在**設定**頁面上，按一下**必要的權限**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/15.png)

2. 在**必要的權限**頁面上，在頂端工具列中的按一下**新增**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/16.png)
   
3. 在**新增應用程式開發介面存取**頁面上，按一下**選取應用程式開發介面**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/17.png)

4. 在**選取應用程式開發介面**頁面上，選取**Microsoft Graph**，然後按一下 **選取**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/18.png)

5. 在**新增應用程式開發介面存取**頁面上，按一下**選取權限**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/19.png)

6. 在**啟用存取**頁面上，按一下**讀取所有識別風險資訊**，然後按一下 **選取**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/20.png)

7. 在**新增應用程式開發介面存取**頁面上，按一下**完成**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/21.png)

8. 在**必要的使用權限**頁面上，按一下**授與權限**，然後按一下 **是**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/22.png)



## <a name="get-an-access-key"></a>取得存取金鑰

1. 在**設定**頁面上，按一下**金鑰**。
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/23.png)

2. 在**金鑰**頁面上，執行下列步驟：
   
    ![建立應用程式](./media/active-directory-identityprotection-graph-getting-started/24.png)

    a. 在**金鑰描述**文字方塊中，輸入的描述 (例如， *AADIP 風險事件*)。
    
    b. 做為**持續時間**，選取**在 1 年**。

    c. 按一下 [檔案] 。
   
    d. 複製的索引鍵的值，然後將它貼到安全的位置。   
   
   > [!NOTE]
   > 如果遺失此金鑰，則必須返回本區段並建立新的金鑰。 請勿將此金鑰告知他人︰任何人只要擁有此金鑰就可以存取您的資料。
   > 
   > 

## <a name="authenticate-to-microsoft-graph-and-query-the-identity-risk-events-api"></a>向 Microsoft Graph 驗證和查詢身分識別風險事件 API
到目前為止，您應該已擁有︰

- 租用戶網域的名稱

- 用戶端識別碼 

- 索引鍵 


若要驗證，請傳送 post 要求至 `https://login.microsoft.com` ，並在主體中加入下列參數︰

- grant_type: “**client_credentials**”

-  resource: “**https://graph.microsoft.com**”

- client_id:\<用戶端識別碼\>

- client_secret:\<您的金鑰\>


如果成功，這會傳回驗證權杖。  
若要呼叫 API，請建立具有下列參數的標頭︰

    `Authorization`=”<token_type> <access_token>"


驗證時，您可以在傳回的權杖中找到權杖類型和存取權杖。

將此標頭做為要求來傳送至下列 API URL： `https://graph.microsoft.com/beta/identityRiskEvents`

回應中，如果成功，是識別風險事件和相關的 OData JSON 格式，會剖析並適當地處理資料的集合。

以下是驗證並呼叫 API 使用 PowerShell 的範例程式碼。  
只要加入用戶端識別碼、 秘密金鑰，以及租用戶網域。

    $ClientID       = "<your client ID here>"        # Should be a ~36 hex character string; insert your info here
    $ClientSecret   = "<your client secret here>"    # Should be a ~44 character string; insert your info here
    $tenantdomain   = "<your tenant domain here>"    # For example, contoso.onmicrosoft.com

    $loginURL       = "https://login.microsoft.com"
    $resource       = "https://graph.microsoft.com"

    $body       = @{grant_type="client_credentials";resource=$resource;client_id=$ClientID;client_secret=$ClientSecret}
    $oauth      = Invoke-RestMethod -Method Post -Uri $loginURL/$tenantdomain/oauth2/token?api-version=1.0 -Body $body

    Write-Output $oauth

    if ($oauth.access_token -ne $null) {
        $headerParams = @{'Authorization'="$($oauth.token_type) $($oauth.access_token)"}

        $url = "https://graph.microsoft.com/beta/identityRiskEvents"
        Write-Output $url

        $myReport = (Invoke-WebRequest -UseBasicParsing -Headers $headerParams -Uri $url)

        foreach ($event in ($myReport.Content | ConvertFrom-Json).value) {
            Write-Output $event
        }

    } else {
        Write-Host "ERROR: No Access Token"
    } 


## <a name="next-steps"></a>後續步驟

恭喜，您剛剛完成了對 Microsoft Graph 的第一次呼叫！  
現在，您可以查詢身分識別風險事件，並依照需要任意使用資料。

若要深入了解 Microsoft Graph 以及如何使用 Graph API 來建置應用程式，請查看[說明文件](https://graph.microsoft.io/docs)，而在 [Microsoft Graph 網站](https://graph.microsoft.io/)上還能找到更多資訊。 此外，請務必要將 [Azure AD Identity Protection API](https://graph.microsoft.io/docs/api-reference/beta/resources/identityprotection_root) 頁面加入書籤，因為此頁面會列出 Graph 中提供的所有 Identity Protection API。 由於我們增加了透過 API 使用 Identity Protection 的新方法，因此您會在該頁面上看到這些方法。

如需相關資訊，請參閱：

-  [Azure Active Directory Identity Protection](active-directory-identityprotection.md)

-  [Azure Active Directory Identity Protection 偵測到的風險事件類型](active-directory-identityprotection-risk-events-types.md)

- [Microsoft Graph](https://graph.microsoft.io/)

- [Microsoft Graph 概觀](https://graph.microsoft.io/docs)

- [Azure AD Identity Protection 服務根目錄](https://graph.microsoft.io/docs/api-reference/beta/resources/identityprotection_root)

