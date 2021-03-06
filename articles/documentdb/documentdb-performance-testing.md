---
title: "Azure Cosmos DB 缩放和性能测试 | Microsoft Docs"
description: "了解如何执行 Azure Cosmos DB 缩放和性能测试"
keywords: "性能测试"
services: cosmosdb
author: arramac
manager: jhubbard
editor: 
documentationcenter: 
ms.assetid: f4c96ebd-f53c-427d-a500-3f28fe7b11d0
ms.service: cosmosdb
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/19/2017
ms.author: arramac
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: d662af5fcd78d88b02e7f21c4d2d2c6add1270bc
ms.contentlocale: zh-cn
ms.lasthandoff: 05/10/2017


---
# <a name="performance-and-scale-testing-with-azure-cosmos-db"></a>执行 Azure Cosmos DB 缩放和性能测试
性能和规模测试是应用程序开发过程中的关键步骤。 对许多应用程序而言，数据库层对整体性能和可缩放性具有相当重大的影响，因此是性能测试的关键组件。 [Azure Cosmos DB](https://azure.microsoft.com/services/documentdb/) 是为了实现弹性缩放和性能可预测而构建的，因此非常适合需要高性能数据库层的应用程序。 

要针对其 Cosmos DB 工作负荷实施性能测试套件或针对高性能应用程序方案评估 Cosmos DB 的开发人员可以参考本文。 本文重点演示隔离的数据库性能测试，但也提供适用于生产应用程序的最佳实践。

阅读本文后，你将能够回答以下问题：   

* 在哪里可以找到可用于 Cosmos DB 性能测试的示例 .NET 客户端应用程序？ 
* 如何从客户端应用程序实现 Cosmos DB 的高吞吐量级别？

若要开始处理代码，请从 [Azure Cosmos DB 性能测试示例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)下载项目。 

> [!NOTE]
> 此应用程序的目标是演示使用少数客户端计算机从 Cosmos DB 中提取更好性能的最佳做法。 生成此应用程序的目的不是为了演示服务的峰值容量如何可以无限扩展。
> 
> 

如果正在寻找用于提高 Cosmos DB 性能的客户端配置选项，请参阅 [Azure Cosmos DB 性能提示](documentdb-performance-tips.md)。

## <a name="run-the-performance-testing-application"></a>运行性能测试应用程序
最快的入门方法是根据以下步骤中所述，编译并运行下面的 .NET 示例。 你也可以查看源代码，然后在自己的客户端应用程序中实施类似的配置。

**步骤 1：**从 [Azure Cosmos DB 性能测试示例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)下载项目，或创建 GitHub 存储库分支。

**步骤 2：**在 App.config 中修改 EndpointUrl、AuthorizationKey、CollectionThroughput 和 DocumentTemplate（可选）的设置。

> [!NOTE]
> 为集合预配高吞吐量之前，请参阅[定价页](https://azure.microsoft.com/pricing/details/documentdb/)以估算每个集合的成本。 Cosmos DB 根据存储和吞吐量单独按小时计费，因此你可以通过在测试后删除或降低 DocumentDB 集合的吞吐量来节省成本。
> 
> 

**步骤 3：**从命令行编译并运行控制台应用。 你应会看到如下输出：

    Summary:
    ---------------------------------------------------------------------
    Endpoint: https://docdb-scale-demo.documents.azure.com:443/
    Collection : db.testdata at 50000 request units per second
    Document Template*: Player.json
    Degree of parallelism*: 500
    ---------------------------------------------------------------------

    DocumentDBBenchmark starting...
    Creating database db
    Creating collection testdata
    Creating metric collection metrics
    Retrying after sleeping for 00:03:34.1720000
    Starting Inserts with 500 tasks
    Inserted 661 docs @ 656 writes/s, 6860 RU/s (18B max monthly 1KB reads)
    Inserted 6505 docs @ 2668 writes/s, 27962 RU/s (72B max monthly 1KB reads)
    Inserted 11756 docs @ 3240 writes/s, 33957 RU/s (88B max monthly 1KB reads)
    Inserted 17076 docs @ 3590 writes/s, 37627 RU/s (98B max monthly 1KB reads)
    Inserted 22106 docs @ 3748 writes/s, 39281 RU/s (102B max monthly 1KB reads)
    Inserted 28430 docs @ 3902 writes/s, 40897 RU/s (106B max monthly 1KB reads)
    Inserted 33492 docs @ 3928 writes/s, 41168 RU/s (107B max monthly 1KB reads)
    Inserted 38392 docs @ 3963 writes/s, 41528 RU/s (108B max monthly 1KB reads)
    Inserted 43371 docs @ 4012 writes/s, 42051 RU/s (109B max monthly 1KB reads)
    Inserted 48477 docs @ 4035 writes/s, 42282 RU/s (110B max monthly 1KB reads)
    Inserted 53845 docs @ 4088 writes/s, 42845 RU/s (111B max monthly 1KB reads)
    Inserted 59267 docs @ 4138 writes/s, 43364 RU/s (112B max monthly 1KB reads)
    Inserted 64703 docs @ 4197 writes/s, 43981 RU/s (114B max monthly 1KB reads)
    Inserted 70428 docs @ 4216 writes/s, 44181 RU/s (115B max monthly 1KB reads)
    Inserted 75868 docs @ 4247 writes/s, 44505 RU/s (115B max monthly 1KB reads)
    Inserted 81571 docs @ 4280 writes/s, 44852 RU/s (116B max monthly 1KB reads)
    Inserted 86271 docs @ 4273 writes/s, 44783 RU/s (116B max monthly 1KB reads)
    Inserted 91993 docs @ 4299 writes/s, 45056 RU/s (117B max monthly 1KB reads)
    Inserted 97469 docs @ 4292 writes/s, 44984 RU/s (117B max monthly 1KB reads)
    Inserted 99736 docs @ 4192 writes/s, 43930 RU/s (114B max monthly 1KB reads)
    Inserted 99997 docs @ 4013 writes/s, 42051 RU/s (109B max monthly 1KB reads)
    Inserted 100000 docs @ 3846 writes/s, 40304 RU/s (104B max monthly 1KB reads)

    Summary:
    ---------------------------------------------------------------------
    Inserted 100000 docs @ 3834 writes/s, 40180 RU/s (104B max monthly 1KB reads)
    ---------------------------------------------------------------------
    DocumentDBBenchmark completed successfully.


**步骤 4（如有必要）：**工具报告的吞吐量（RU/秒）应该等于或大于预配的集合吞吐量。 如果情况并非如此，以较小的增量提高 DegreeOfParallelism 可帮助达到该限制。 如果客户端应用的吞吐量达到持平状态，则在相同或不同计算机上启动多个应用程序实例可帮助在各个不同的实例中达到预配的限制。 如果需要此步骤的帮助，请撰写电子邮件并将其发送到 askdocdb@microsoft.com 或填写来自 [Azure 门户](https://portal.azure.com)的支持票证。

应用处于运行状态后，可以尝试不同的[编制索引策略](documentdb-indexing-policies.md)和[一致性级别](documentdb-consistency-levels.md)，以了解它们对吞吐量和延迟的影响。 你也可以查看源代码，然后在自己的测试套件或生产应用程序中实施类似的配置。

## <a name="next-steps"></a>后续步骤
本文介绍了如何使用 .NET 控制台应用对 Cosmos DB 执行性能和缩放测试。 有关使用 Cosmos DB 的其他信息，请参阅下面的链接。

* [Azure Cosmos DB 性能测试示例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)
* [用于提高 Azure Cosmos DB 性能的客户端配置选项](documentdb-performance-tips.md)
* [Azure Cosmos DB 中的服务器端分区](documentdb-partition-data.md)
* [DocumentDB 集合和性能级别](documentdb-performance-levels.md)
* [MSDN 上的 DocumentDB .NET SDK 文档](https://msdn.microsoft.com/library/azure/dn948556.aspx)
* [DocumentDB .NET samples](https://github.com/Azure/azure-documentdb-net)（DocumentDB .NET 示例）
* [Azure Cosmos DB blog on performance tips](https://azure.microsoft.com/blog/2015/01/20/performance-tips-for-azure-documentdb-part-1-2/)（Azure Cosmos DB 性能提示博客）


