---
title: "找出特定的用户何时可以访问某个应用程序 | Microsoft Docs"
description: "如何找出非常重要的用户何时可以访问已使用 Azure AD 配置用户预配的应用程序"
services: active-directory
documentationcenter: 
author: ajamess
manager: femila
ms.assetid: 
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/04/2017
ms.author: asteen
translationtype: Human Translation
ms.sourcegitcommit: 0d6f6fb24f1f01d703104f925dcd03ee1ff46062
ms.openlocfilehash: b1f16079ad13c4e45f93a7e5e3d29568738e03cf
ms.lasthandoff: 04/18/2017


---

# <a name="find-out-when-a-specific-user-will-be-able-to-access-an-application"></a>找出特定的用户何时可以访问某个应用程序
当将自动化用户预配用于应用程序时，Azure AD 将根据类似[用户和组分配](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)，定期（通常每 10 分钟）自动预配并更新应用中的用户帐户。

## <a name="how-long-does-it-take"></a>需要多长时间？

给定用户的预配所需时间主要取决于是否已进行过初始“完整”同步。

Azure AD 和应用之间的首次同步可能会持续 20 分钟至几小时，具体取决于 Azure AD 目录的大小与作用域中的预配用户数量。 

在初始同步之后同步速度会变快（例如在 10 分钟内完成），因为在初始同步后，预配服务已存储代表两个系统状态的水印，这提高了后续同步的性能。

## <a name="how-to-check-the-status-of-a-user"></a>如何检查用户状态

若要查看选定用户的预配状态，请查阅 Azure AD 中的审核日志。

可在 Azure 门户中访问预配审核日志，具体位置在**“Azure Active Directory”&gt;“企业应用”&gt;“应用程序名称”\[\]“审核日志”&gt;**选项卡。 在“帐户预配”类别上筛选日志，以仅查看该应用的预配事件。 可根据在属性映射中为用户配置的“匹配 ID”搜索用户。 

例如，如果在 Azure AD 端将“用户主体名称”或“电子邮件地址”配置为匹配属性，并且尚未预配的用户的值为“audrey@contoso.com”，然后在审核日志中搜索“audrey@contoso.com”，并查看返回的条目。

预配审核日志记录了预配服务执行的全部操作，包括：

* 查询 Azure AD 以找到预配作用域中分配的用户
* 查询目标应用以验证是否存在这些用户
* 比较系统之间的用户对象
* 根据比较结果在目标系统中添加、更新或禁用用户帐户

## <a name="next-steps"></a>后续步骤
[通过 Azure Active Directory 为 SaaS 应用程序自动化用户预配和取消预配](https://docs.microsoft.com/azure/active-directory/active-directory-saas-app-provisioning)

