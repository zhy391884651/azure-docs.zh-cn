---
title: "Azure Cosmos DB 连接器的 Power BI 教程 | Microsoft Docs"
description: "使用此 Power BI 教程来导入 JSON、创建见解深入的报表以及使用 Azure Cosmos DB 和 Power BI 连接器来可视化数据。"
keywords: "Power BI 教程, 可视化数据, Power BI 连接器"
services: cosmosdb
author: mimig1
manager: jhubbard
editor: mimig
documentationcenter: 
ms.assetid: cd1b7f70-ef99-40b7-ab1c-f5f3e97641f7
ms.service: cosmosdb
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/14/2016
ms.author: mimig
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: 25a33ea4266dd5b7874df1c642ca05fbaf6a50fa
ms.contentlocale: zh-cn
ms.lasthandoff: 05/10/2017


---
# <a name="power-bi-tutorial-for-azure-cosmos-db-visualize-data-using-the-power-bi-connector"></a>Azure Cosmos DB 的 Power BI 教程：使用 Power BI 连接器可视化数据
[PowerBI.com](https://powerbi.microsoft.com/) 是一个在线服务，在此处你可以创建和共享包含对你和你的组织来说很重要的数据的仪表板和报表。  Power BI Desktop 是一个专用的报表创作工具，使你能够从各种数据源检索数据、合并和转换数据、创建功能强大的报表和可视化效果以及将报表发布到 Power BI。  使用最新版本的 Power BI Desktop，现在可以通过 Power BI 的 Cosmos DB 连接器连接到你的 Cosmos DB 帐户。   

在此 Power BI 教程中，我们将逐步讲解各步骤 - 在 Power BI Desktop 中连接到 Cosmos DB 帐户、使用导航器导航至我们想要在其中提取数据的集合、使用 Power BI Desktop 查询编辑器将 JSON 数据转换为表格格式、以及生成报表并将其发布到 PowerBI.com。

在完成此 Power BI 教程后，你将能够回答以下问题：  

* 我可以如何使用 Power BI Desktop 生成包含 Cosmos DB 中数据的报表？
* 如何在 Power BI Desktop 中连接到 Cosmos DB 帐户？
* 如何在 Power BI Desktop 中从集合检索数据？
* 如何在 Power BI Desktop 中转换嵌套的 JSON 数据？
* 如何在 PowerBI.com 中发布和共享我的报表？

## <a name="prerequisites"></a>先决条件
在按照此 Power BI 教程中的说明操作之前，请确保已拥有：

* [最新版本的 Power BI Desktop](https://powerbi.microsoft.com/desktop)。
* 在你的 Cosmos DB 帐户中访问我们的演示帐户或数据。
  * 演示帐户使用本教程中显示的火山数据填充。 此演示帐户未被任何 SLA 绑定且只用于演示意图。  我们保留对此演示帐户进行修改的权利，包括但不限于在任何时间无需提前通知或理由而终止帐户、更改密钥、限制访问、更改和删除数据。
    * URL:   https://analytics.documents.azure.com
    * 只读密钥：MSr6kt7Gn0YRQbjd6RbTnTt7VHc5ohaAFu7osF0HdyQmfR+YhwCH2D2jcczVIR1LNK3nMPNBD31losN7lQ/fkw==
  * 或者，若要创建你自己帐户，请参阅[使用 Azure 门户创建 Azure Cosmos DB 数据库帐户](https://azure.microsoft.com/documentation/articles/documentdb-create-account/)。 然后，要获取类似于本教程中使用的示例火山数据（但不包含 GeoJSON 块），请参阅 [NOAA 站点](https://www.ngdc.noaa.gov/nndc/struts/form?t=102557&s=5&d=5)，然后使用 [Azure Cosmos DB 数据迁移工具导入数据](https://azure.microsoft.com/documentation/articles/documentdb-import-data/)。

要在 PowerBI.com 中共享你的报表，必须在 PowerBI.com 中拥有帐户。  若要了解更多有关 Power BI 免费版和 Power BI Pro 的信息，请访问 [https://powerbi.microsoft.com/pricing](https://powerbi.microsoft.com/pricing)。

## <a name="lets-get-started"></a>让我们开始吧
在本教程中，让我们假设你是一位研究世界各地的火山的地理学家。  火山数据存储在 Cosmos DB 帐户中且 JSON 文档的外观如下所示。

    {
        "Volcano Name": "Rainier",
           "Country": "United States",
          "Region": "US-Washington",
          "Location": {
            "type": "Point",
            "coordinates": [
              -121.758,
              46.87
            ]
          },
          "Elevation": 4392,
          "Type": "Stratovolcano",
          "Status": "Dendrochronology",
          "Last Known Eruption": "Last known eruption from 1800-1899, inclusive"
    }

建议从 Cosmos DB 帐户中检索火山数据并在如下交互式 Power BI 报表中将数据可视化。

![通过 Power BI 连接器完成此 Power BI 教程，你将能够使用 Power BI Desktop 火山报表对数据进行可视化](./media/documentdb-powerbi-visualize/power_bi_connector_pbireportfinal.png)

准备好尝试一下了吗？ 让我们开始吧。

1. 在你的工作站上运行 Power BI Desktop。
2. 一旦启动 Power BI Desktop 后，将显示“欢迎”屏幕。
   
    ![Power BI Desktop 欢迎屏幕 - Power BI 连接器](./media/documentdb-powerbi-visualize/power_bi_connector_welcome.png)
3. 你可以“获取数据”、查看“最近使用的源”或直接从“欢迎”屏幕“打开其他报表”。  单击右上角的 X 以关闭屏幕。 将显示 Power BI Desktop 的“报表”视图。
   
    ![Power BI Desktop 报表视图 - Power BI 连接器](./media/documentdb-powerbi-visualize/power_bi_connector_pbireportview.png)
4. 选择“主页”功能区，然后单击“获取数据”。  应出现“获取数据”窗口。
5. 单击“Azure”，选择“Microsoft Azure Cosmos DB (Beta 版本)”，然后单击“连接”。  应出现“Microsoft Azure Cosmos DB 连接”窗口。
   
    ![Power BI Desktop 获取数据 - Power BI 连接器](./media/documentdb-powerbi-visualize/power_bi_connector_pbigetdata.png)
6. 如下所示指定你想要从其中检索数据的 Cosmos DB 帐户终结点 URL，然后单击“确定”。 可以在 Azure 门户的[“密钥”](documentdb-manage-account.md#keys)边栏选项卡中的 URI 框检索 URL，或者可以使用演示帐户信息，其中 URL 为 `https://analytics.documents.azure.com`。 
   
    将数据库名称、集合名称和 SQL 声明留空，因为这些字段为可选。  相反，我们将使用导航器来选择数据库和集合以指定数据来源。
   
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 桌面连接窗口](./media/documentdb-powerbi-visualize/power_bi_connector_pbiconnectwindow.png)
7. 如果你是首次连接到此终结点，则将提示你输入帐户密钥。  可以在 Azure 门户的[“只读密钥”](documentdb-manage-account.md#keys)边栏选项卡中的“主密钥”框检索密钥，或者可以使用演示帐户信息，其中密钥为 `MSr6kt7Gn0YRQbjd6RbTnTt7VHc5ohaAFu7osF0HdyQmfR+YhwCH2D2jcczVIR1LNK3nMPNBD31losN7lQ/fkw==`。 输入帐户密钥并单击“连接”。
   
    我们建议你在生成报表时使用只读密钥。  这将防止主密钥不必要地暴露于潜在的安全风险中。 只读密钥从 Azure 门户的“密钥”[](documentdb-manage-account.md#keys)边栏选项卡中可用，或者你可以使用上文提供的演示帐户信息。
   
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 帐户密钥](./media/documentdb-powerbi-visualize/power_bi_connector_pbidocumentdbkey.png)
8. 帐户成功连接后，将出现“导航器”。  “导航器”将在帐户下显示数据库的列表。
9. 单击并展开将提供报表数据的数据库，如果正在使用演示帐户，请选择**volcanodb**。   
10. 现在，选择要从其中检索数据的集合。 如果正在使用演示帐户，请选择 **volcano1**。
    
    预览窗格显示“记录”项的列表。  文档在 Power BI 中表示为一种“记录”类型。 同样，文档内部的嵌套 JSON 块也是“记录”。
    
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 导航器窗口](./media/documentdb-powerbi-visualize/power_bi_connector_pbinavigator.png)
11. 单击“编辑”启动查询编辑器，以便我们能够转换数据。

## <a name="flattening-and-transforming-json-documents"></a>平展和转换 JSON 文档
1. 在 Power BI 查询编辑器中，你应会在中心窗格中看到“文档”列。
   ![Power BI Desktop 查询编辑器](./media/documentdb-powerbi-visualize/power_bi_connector_pbiqueryeditor.png)
2. 单击“文档”列标头右侧的扩展器。  将显示带有字段列表的上下文菜单。  选择你的报表所需的字段，例如，火山名称、国家/地区、区域、位置、海拔、类型、状态和已知的上次喷发，然后单击“确定”。
   
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 扩展文档](./media/documentdb-powerbi-visualize/power_bi_connector_pbiqueryeditorexpander.png)
3. 中心窗格将以所选字段显示结果预览。
   
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 平铺结果](./media/documentdb-powerbi-visualize/power_bi_connector_pbiresultflatten.png)
4. 在我们的示例中，“位置”属性在文档中为 GeoJSON 块。  正如你所看到的那样，“位置”在 Power BI Desktop 中表示为一种“记录”类型。  
5. 单击“位置”列标头右侧的扩展器。  将显示具有类型和坐标字段的上下文菜单。  选择坐标字段然后单击“确定”。
   
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 位置记录](./media/documentdb-powerbi-visualize/power_bi_connector_pbilocationrecord.png)
6. 中心窗格现在显示“列表”类型的坐标列。  如教程的开头所示，本教程中的 GeoJSON 数据是“点”类型，纬度值和经度值记录在坐标数组中。
   
    coordinates[0] 元素表示经度，coordinates[1] 表示纬度。
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 坐标列表](./media/documentdb-powerbi-visualize/power_bi_connector_pbiresultflattenlist.png)
7. 若要平展坐标数组，我们需创建名为“经纬度”的自定义列。  选择“添加列”功能区并单击“添加自定义列”。  应出现“添加自定义列”窗口。
8. 为新列提供一个名称，例如经纬度。
9. 接下来，为新列指定自定义公式。  对于我们的示例，我们将如下所示，使用以下公式连接逗号分隔的纬度值和经度值：`Text.From([Document.Location.coordinates]{1})&","&Text.From([Document.Location.coordinates]{0})`。 单击 **“确定”**。
   
    有关数据分析表达式 (DAX)（包括 DAX 函数）的详细信息，请访问 [Power BI Desktop 中的 DAX Basic](https://support.powerbi.com/knowledgebase/articles/554619-dax-basics-in-power-bi-desktop)。
   
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 添加自定义列](./media/documentdb-powerbi-visualize/power_bi_connector_pbicustomlatlong.png)
10. 现在，中心窗格将显示使用逗号分隔的纬度值和经度值填充的新“经纬度”列。
    
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 自定义经纬度列](./media/documentdb-powerbi-visualize/power_bi_connector_pbicolumnlatlong.png)
    
    如果新的列中出现“错误”，请确保“查询设置”中应用的步骤与下图匹配：
    
    ![应用的步骤应该为“源”、“导航”、“展开的文档”、“展开的文档位置”、“添加自定义”](./media/documentdb-powerbi-visualize/azure-documentdb-power-bi-applied-steps.png)
    
    如果步骤不同，请删除额外的步骤并再次尝试添加自定义列。 
11. 此时我们已经完成了将数据平展到表格格式中。  可以利用查询编辑器中所有可用的功能在 Cosmos DB 中形成和转换数据。  如果正在使用示例，可以通过在“主页”功能区上更改“数据类型”，将“海拔”数据类型更改为“整数”。
    
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 更改列类型](./media/documentdb-powerbi-visualize/power_bi_connector_pbichangetype.png)
12. 单击“关闭并应用”以保存数据模型。
    
    ![Azure Cosmos DB Power BI 连接器的 Power BI 教程 - 关闭和应用](./media/documentdb-powerbi-visualize/power_bi_connector_pbicloseapply.png)

## <a name="build-the-reports"></a>生成报表
你可以在 Power BI Desktop 报表视图中开始创建报表以可视化数据。  可以通过将字段拖放到“报表”画布中来创建报表。

![Power BI Desktop 报表视图 - Power BI 连接器](./media/documentdb-powerbi-visualize/power_bi_connector_pbireportview2.png)

在报表视图中，你应找到：

1. “字段”窗格中存在可用于报表的带有字段的数据模型列表。
2. “可视化”窗格。 一个报表可以包含单个或多个可视化效果。  从“可视化”窗格中选取所需视觉对象类型。
3. 可在“报表”画布中为报表生成视觉效果。
4. “报表”页。 可以在 Power BI Desktop 中添加多个报表页。

下面将演示创建简单交互地图视图报表的基本步骤。

1. 对于我们的示例，我们将创建显示每座火山的位置的地图视图。  在“可视化”窗格中，单击如上屏幕截图中突出显示的“地图”视觉对象类型。  你应该会看到绘制在“报表”画布上的地图视觉对象类型。  “可视化”窗格也应该显示一组与地图视觉对象类型相关的属性。
2. 现在，从“字段”窗格将经纬度字段拖放到“可视化”窗格中的“位置”属性。
3. 接下来，将“火山名称”字段拖放到“图例”属性。  
4. 然后，将“海拔”字段拖放到“大小”属性。  
5. 现在你应该会看到显示一组表示每座火山位置的气泡的地图视觉，气泡的大小与火山的海拔相关联。
6. 此时你已成功创建了基本报表。  你可以通过添加更多可视化效果来进一步自定义该报表。  在本例中，我们添加了火山类型切片器以使报表具有交互性。  
   
    ![Azure Cosmos DB 的 Power BI 教程完成时最终 Power BI Desktop 报表的屏幕截图](./media/documentdb-powerbi-visualize/power_bi_connector_pbireportfinal.png)

## <a name="publish-and-share-your-report"></a>发布和共享报表
要共享你的报表，必须在 PowerBI.com 中拥有帐户。

1. 在 Power BI Desktop 中，单击“主页”功能区。
2. 单击“发布” 。  系统将提示你输入你的 PowerBI.com 帐户的用户名和密码。
3. 一旦验证凭据后，就可将报表发布到所选目的地。
4. 单击“打开 Power BI 中的‘PowerBITutorial.pbix’”，查看并共享 Power BI.com 上的报表。
   
    ![成功发布到 Power BI！ 在 Power BI 中打开教程](./media/documentdb-powerbi-visualize/power_bi_connector_open_in_powerbi.png)

## <a name="create-a-dashboard-in-powerbicom"></a>在 PowerBI.com 中创建仪表板
既然有了报表，就将报表分享到 PowerBI.com 吧

从 Power BI Desktop 发布报表到 PowerBI.com 时，会在 PowerBI.com 租户中生成一个“报表”和“数据集”。 例如，将一个名为 **PowerBITutorial** 的报表发布到 PowerBI.com 后，PowerBITutorial 会出现在 PowerBI.com 的“报表”和“数据集”部分。

   ![PowerBI.com 中新“报表”和“数据集”的屏幕截图](./media/documentdb-powerbi-visualize/documentdb-powerbi-reports-datasets.png)

若要创建可共享的仪表板，请单击 PowerBI 报表上的“固定活动页”按钮。

   ![PowerBI.com 中新“报表”和“数据集”的屏幕截图](./media/documentdb-powerbi-visualize/azure-documentdb-power-bi-pin-live-tile.png)

然后按照[从报表固定磁贴](https://powerbi.microsoft.com/documentation/powerbi-service-pin-a-tile-to-a-dashboard-from-a-report/#pin-a-tile-from-a-report)中的说明创建新仪表板。 

创建仪表板之前也可以临时修改报表。 但是，建议你使用 Power BI Desktop 执行修改并将报表重新发布到 PowerBI.com。

## <a name="refresh-data-in-powerbicom"></a>在 PowerBI.com 中刷新数据
刷新数据有两种方法，临时和计划。

对于临时刷新，只需单击“数据集”旁的省略号 (…)，例如 PowerBITutorial。 应该会出现一个包括“立即刷新”的操作列表。 单击“立即刷新”刷新数据。

![PowerBI.com 中“立即刷新”的屏幕截图](./media/documentdb-powerbi-visualize/azure-documentdb-power-bi-refresh-now.png)

对于计划刷新，请执行以下操作。

1. 单击操作列表中的“计划刷新”。 
    ![PowerBI.com 中计划刷新的屏幕截图](./media/documentdb-powerbi-visualize/azure-documentdb-power-bi-schedule-refresh.png)
2. 在“设置”页上，展开“数据源凭据”。 
3. 单击“编辑凭据”。 
   
    出现配置弹出窗口。 
4. 输入密钥连接到该数据集的 Cosmos DB 帐户，然后单击“登录”。 
5. 展开“计划刷新”并设置刷新数据集的计划。 
6. 单击“应用”，就可完成计划刷新设置。

## <a name="next-steps"></a>后续步骤
* 有关 Power BI 的详细信息，请参阅 [Power BI 入门](https://powerbi.microsoft.com/documentation/powerbi-service-get-started/)。
* 若要了解有关 Cosmos DB 的详细信息，请参阅 [Azure Cosmos DB 文档登陆页](https://azure.microsoft.com/documentation/services/documentdb/)。


