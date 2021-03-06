---
title: "教程：Azure Active Directory 与 Bime 集成 | Microsoft Docs"
description: "了解如何将 Bime 与 Azure Active Directory 结合使用以启用单一登录、自动化预配和其他功能！"
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: bdcf0729-c880-4c95-b739-0f6345b17dd8
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/10/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: bd3dd077bef87a78904ffd5d2be469b6b8bc8959
ms.openlocfilehash: 7857480d033e4d570aa48569e08bb30846b280f6
ms.lasthandoff: 02/17/2017


---
# <a name="tutorial-azure-active-directory-integration-with-bime"></a>教程：Azure Active Directory 与 Bime 集成
本教程旨在展示 Azure 与 Bime 集成。  
在本教程中概述的方案假定您已具有以下各项：

* 一个有效的 Azure 订阅
* Bime 租户

完成本教程后，分配到 Bime 的 Azure AD 用户将能够在 Bime 公司站点（服务提供商启动的登录）或通过阅读[访问面板简介](active-directory-saas-access-panel-introduction.md)单一登录应用程序。

在本教程中概述的方案由以下构建基块组成：

* 支持 Bime 的应用程序集成
* 配置单一登录 (SSO)
* 配置用户设置
* 分配用户

![方案](./media/active-directory-saas-bime-tutorial/IC775552.png "方案")

## <a name="enable-the-application-integration-for-bime"></a>为 Bime 启用应用程序集成
本部分旨在概述如何为 Bime 启用应用程序集成。

**若要为 Bime 启用应用程序集成，请执行以下步骤：**

1. 在 Azure 经典门户的左侧导航窗格中，单击“Active Directory”。
   
   ![Active Directory](./media/active-directory-saas-bime-tutorial/IC700993.png "Active Directory")
2. 在“目录”列表中，选择要启用目录集成的目录。
3. 若要打开应用程序视图，请在目录视图的顶部菜单中，单击“应用程序”。
   
   ![应用程序](./media/active-directory-saas-bime-tutorial/IC700994.png "应用程序")
4. 在页面底部单击“添加”。
   
   ![添加应用程序](./media/active-directory-saas-bime-tutorial/IC749321.png "添加应用程序")
5. 在“要执行什么操作”对话框中，单击“从库中添加应用程序”。
   
   ![从库添加应用程序](./media/active-directory-saas-bime-tutorial/IC749322.png "从库添加应用程序")
6. 在“搜索框”中，键入“Bime”。
   
   ![应用程序库](./media/active-directory-saas-bime-tutorial/IC775553.png "应用程序库")
7. 在“结果”窗格中，选择“Bime”，然后单击“完成”，添加该应用程序。
   
   ![Bime](./media/active-directory-saas-bime-tutorial/IC775554.png "Bime")
   
## <a name="configure-single-sign-on"></a>配置单一登录

本部分旨在概述如何使用户使用基于 SAML 协议的联合身份验证通过他们在 Azure AD 中的帐户对 Bime 进行身份验证。  

配置 Bime 的 SSO 需要检索证书的指纹值。 如果不熟悉此步骤，请参阅 [How to retrieve a certificate's thumbprint value](http://youtu.be/YKQF266SAxI)（如何检索证书的指纹值）。

**若要配置单一登录，请执行以下步骤：**

1. 在 Azure 经典门户中的“Bime”应用程序集成页上，单击“配置单一登录”，打开“配置单一登录”对话框。
   
   ![配置单一登录](./media/active-directory-saas-bime-tutorial/IC771709.png "配置单一登录")
2. 在“你希望用户如何登录 Bime”页上，选择“Microsoft Azure AD 单一登录”，然后单击“下一步”。
   
   ![配置单一登录](./media/active-directory-saas-bime-tutorial/IC775555.png "配置单一登录")
3. 在“配置应用 URL”页上的“Bime 登录 URL”文本框中，使用以下模式“*https://\<tenant-name\>.Bimeapp.com*”键入 URL，然后单击“下一步”。
   
   ![配置应用 URL](./media/active-directory-saas-bime-tutorial/IC775556.png "配置应用 URL")
4. 在“配置 Bime 的单一登录”页上，若要下载证书，请单击“下载证书”，然后将证书文件本地保存为“c:\\Bime.cer”。
   
   ![配置单一登录](./media/active-directory-saas-bime-tutorial/IC775557.png "配置单一登录")
5. 在其他 Web 浏览器窗口中，以管理员身份登录 Bime 公司站点。
6. 在工具栏中，单击“管理员”，然后单击“帐户”。
   
   ![管理员](./media/active-directory-saas-bime-tutorial/IC775558.png "管理员")
7. 在帐户配置页面上，执行以下步骤：
   
   ![配置单一登录](./media/active-directory-saas-bime-tutorial/IC775559.png "配置单一登录")
   
   1. 选择“启用 SAML 身份验证”。
   2. 在 Azure 经典门户的“配置 Bime 的单一登录”对话框页面上，复制“远程登录 URL”值，然后将其粘贴到“远程登录 URL”文本框。
   3. 从导出的证书中复制“指纹”值，然后将其粘贴到“证书指纹”文本框。       
      
      >[!TIP]
      >有关更多详细信息，请参阅[如何检索证书的指纹值](http://youtu.be/YKQF266SAxI)。 
      > 
   4. 单击“保存” 。
8. 在 Azure 经典门户中，选择“单一登录配置确认”，然后单击“完成”，关闭“配置单一登录”对话框。
   
   ![配置单一登录](./media/active-directory-saas-bime-tutorial/IC775560.png "配置单一登录")
   
## <a name="configure-user-provisioning"></a>配置用户设置

为了使 Azure AD 用户能够登录 Bime，必须对其进行预配才能使其登录 Bime。  

* 就 Bime 来说，预配任务需要手动完成。

**若要配置用户预配，请执行以下步骤：**

1. 登录 **Bime** 租户。
2. 在工具栏中，单击“管理员”，然后单击“用户”。
   
   ![管理员](./media/active-directory-saas-bime-tutorial/IC775561.png "管理员")
3. 在“用户列表”中，单击“添加新用户”(“+”)。
   
   ![用户](./media/active-directory-saas-bime-tutorial/IC775562.png "用户")
4. 在“用户详细信息”对话框页上，执行以下步骤：
   
   ![用户详细信息](./media/active-directory-saas-bime-tutorial/IC775563.png "用户详细信息")   
  1. 输入想要预配的有效 AAD 帐户的“名字”、“姓氏”、“登录名”和“电子邮件”。
  2. 单击“保存”。

>[!NOTE]
>可使用其他任何 Bime 用户帐户创建工具或 Bime 提供的 API 预配 AAD 用户帐户。
>  

## <a name="assign-users"></a>分配用户
若要测试配置，需要通过分配权限的方式向希望其使用应用程序的 Azure AD 用户授予该配置的访问权限。

**若要将用户分配到 Bime，请执行以下步骤：**

1. 在 Azure 经典门户中，创建测试帐户。
2. 在“Bime”应用程序集成页上，单击“分配用户”。
   
   ![分配用户](./media/active-directory-saas-bime-tutorial/IC775564.png "分配用户")
3. 选择测试用户，单击“分配”，然后单击“是”确认分配。
   
   ![是](./media/active-directory-saas-bime-tutorial/IC767830.png "是")

如果要测试单一登录设置，请打开访问面板。 有关访问面板的详细信息，请参阅 [Introduction to the Access Panel](active-directory-saas-access-panel-introduction.md)（访问面板简介）。


