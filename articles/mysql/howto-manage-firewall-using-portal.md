---
title: "使用 Azure 门户创建和管理 Azure Database for MySQL 防火墙规则 | Microsoft Docs"
description: "使用 Azure 门户创建和管理 Azure Database for MySQL 防火墙规则"
services: mysql
author: v-chenyh
ms.author: v-chenyh
manager: jhubbard
editor: jasonh
ms.assetid: 
ms.service: mysql-database
ms.tgt_pltfrm: portal
ms.topic: article
ms.date: 05/10/2017
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: 744f3503d2abc618095351e1b018cab22adcf091
ms.contentlocale: zh-cn
ms.lasthandoff: 05/10/2017

---
# <a name="create-and-manage-azure-database-for-mysql-firewall-rules-using-the-azure-portal"></a>使用 Azure 门户创建和管理 Azure Database for MySQL 防火墙规则
使用服务器级防火墙规则，管理员可以从指定的 IP 地址或某个范围的 IP 地址访问 Azure Database for MySQL 服务器。 

## <a name="create-a-server-level-firewall-rule-in-the-azure-portal"></a>在 Azure 门户中创建服务器级防火墙规则

1. 在 MySQL 服务器边栏选项卡上的“设置”标题下，单击“连接安全性”，以打开 Azure Database for MySQL 的“连接安全性”边栏选项卡。
![Azure 门户 - 单击“连接安全性”](./media/howto-manage-firewall-using-portal/1-connection-security.png)

2. 在工具栏上单击“添加我的 IP”。 该操作会自动使用计算机的 IP 地址（Azure 系统所识别的地址）创建一个规则。
![Azure 门户 - 单击“添加我的 IP”](./media/howto-manage-firewall-using-portal/2-add-my-ip.png)

3. 验证 IP 地址，然后保存配置。 在某些情况下，Azure 门户识别出的 IP 地址与访问 Internet 和 Azure 服务器时所使用的 IP 地址不同。 因此，可能需要更改起始 IP 和结束 IP，以使规则正常工作。

使用搜索引擎或其他联机工具查看自己的 IP 地址（例如，搜索“我的 IP 地址是多少”）。
![用必应搜索“我的 IP 是多少”](./media/howto-manage-firewall-using-portal/3-what-is-my-ip.png)

4. 在工具栏上单击“保存”以保存此服务器级防火墙规则。 等待出现有关防火墙规则更新已成功的确认消息。
![Azure 门户 - 单击“保存”](./media/howto-manage-firewall-using-portal/4-save-firewall-rule.png)

> [!NOTE]
> 在 Azure Database for MySQL 防火墙规则中，可以指定单个 IP 地址，也可以指定某个范围的地址。 如果希望将规则限制为单个 IP 地址，请在起始 IP 和结束 IP 字段中输入相同的地址。 打开防火墙后，管理员和用户可以登录到 MySQL 服务器上他们拥有有效凭据的任何数据库。
![Azure 门户 - 防火墙规则](./media/howto-manage-firewall-using-portal/5-specify-addresses.png)

## <a name="manage-existing-server-level-firewall-rules-through-the-azure-portal"></a>通过 Azure 门户管理现有的服务器级别防火墙规则
重复这些步骤来管理防火墙规则。
* 若要添加当前计算机，请单击“+ 添加我的 IP”。
* 若要添加其他 IP 地址，请键入“规则名称”、“起始 IP”和“结束 IP”。
* 若要修改现有规则，单击规则中的任意字段并修改。
* 若要删除现有规则，请单击省略号 […]，然后单击“删除”。
* 单击“保存”以保存更改。

## <a name="next-steps"></a>后续步骤
- 有关连接到 Azure Database for MySQL 服务器的帮助，请参阅 [Azure Database for MySQL 的连接库](./concepts-connection-libraries.md)
