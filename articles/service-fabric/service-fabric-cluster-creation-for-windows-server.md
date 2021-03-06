---
title: "创建独立的 Azure Service Fabric 群集 | Microsoft Docs"
description: "在运行 Windows Server 的任何本地或任意云计算机（物理或虚拟）上创建 Azure Service Fabric 群集。"
services: service-fabric
documentationcenter: .net
author: ChackDan
manager: timlt
editor: 
ms.assetid: 31349169-de19-4be6-8742-ca20ac41eb9e
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/24/2017
ms.author: chackdan;maburlik
translationtype: Human Translation
ms.sourcegitcommit: 432752c895fca3721e78fb6eb17b5a3e5c4ca495
ms.openlocfilehash: 3389684dca62a1b3c8297600c7c09ffef642c854
ms.lasthandoff: 03/30/2017


---
# <a name="create-a-standalone-cluster-running-on-windows-server"></a>创建在 Windows Server 上运行的独立群集
你可以使用 Azure Service Fabric 在运行 Windows Server 的任何虚拟机或计算机上创建 Service Fabric 群集。 这意味着可以在包含一组相互连接的 Windows Server 计算机的任何环境（无论是本地环境还是任何云提供商所提供的环境）中部署和运行 Service Fabric 应用程序。 Service Fabric 提供了一个安装程序包，用于创建名为“Windows Server 独立包”的 Service Fabric 群集。

本文将逐步指导你完成创建 Service Fabric 独立群集的过程。

> [!NOTE]
> 可购买此独立 Windows Server 包并将其用于生产部署。 此包可能包含“预览”形式的新 Service Fabric 功能。 向下滚动到 [此包中包括的预览功能](#previewfeatures_anchor) 预览功能列表部分。 现在，可以[下载 EULA 的副本](http://go.microsoft.com/fwlink/?LinkID=733084)。
> 
> 

<a id="getsupport"></a>

## <a name="get-support-for-the-service-fabric-standalone-package"></a>获取 Service Fabric 独立包的支持
* 在 [Azure Service Fabric 论坛](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=AzureServiceFabric?)中询问社区关于 Windows Server 的 Service Fabric 独立包的信息。
* 打开 [Service Fabric 专业支持](http://support.microsoft.com/oas/default.aspx?prid=16146)票证。  在[此处](https://support.microsoft.com/en-us/gp/offerprophone?wa=wsignin1.0)了解有关 Microsoft 专业支持的详细信息。
* 你还可以获取作为 [Microsoft 顶级支持](https://support.microsoft.com/en-us/premier)的一部分的对此包的支持。
* 有关详细信息，请参阅 [Azure Service Fabric 支持选项](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-support)。
* 若要以支持为目的收集日志，请运行 [Service Fabric 独立日志收集器](https://go.microsoft.com/fwlink/?linkid=842487)。

<a id="downloadpackage"></a>

## <a name="download-the-service-fabric-standalone-package"></a>下载 Service Fabric 独立包
若要创建群集，请使用可在此处找到的适用于 Windows Server（2012 R2 和更高版本）的 Service Fabric 独立包： <br>
[下载链接 - Service Fabric 独立包 - Windows Server](http://go.microsoft.com/fwlink/?LinkId=730690)

在[此处](service-fabric-cluster-standalone-package-contents.md)查找有关包内容的详细信息。

在创建群集时自动下载 Service Fabric 运行时包。 如果从未连接到 Internet 的计算机进行部署，请从此处下载带外的运行时包： <br>
[下载链接 - Service Fabric 运行时 - Windows Server](https://go.microsoft.com/fwlink/?linkid=839354)

在此处查找独立群集配置示例： <br>
[独立群集配置示例](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)

<a id="createcluster"></a>

## <a name="create-the-cluster"></a>创建群集
可以使用[示例](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)中包含的 *ClusterConfig.Unsecure.DevCluster.json* 文件将 Service Fabric 部署到一个计算机开发群集。

将独立包解压缩到你的计算机，将示例配置文件复制到本地计算机，然后通过管理员 PowerShell 会话从独立包文件夹中运行 *CreateServiceFabricCluster.ps1* 脚本：
### <a name="step-1a-create-an-unsecured-local-development-cluster"></a>步骤 1A：创建不受保护的本地开发群集
```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json -AcceptEULA
```

有关故障排除的详细信息，请参阅[计划和准备群集部署](service-fabric-cluster-standalone-deployment-preparation.md)中的环境设置部分。

如果你已完成运行开发方案，则可以引用下面的“[删除群集](#removecluster_anchor)”部分的步骤，从计算机中删除 Service Fabric 群集。 

### <a name="step-1b-create-a-multi-machine-cluster"></a>步骤 1B：创建多台计算机的群集
在完成下面链接中详述的计划和准备步骤后，就可以使用群集配置文件创建生产群集。 <br>
[计划和准备群集部署](service-fabric-cluster-standalone-deployment-preparation.md)

1. 从独立包文件夹中运行 *TestConfiguration.ps1* 脚本，以验证写入的配置文件：  

```powershell
.\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.json
```

可看到如下输出： 如果底部字段“Passed”的返回值为“True”，那么已通过完整性检查，并且根据输入配置群集看似可以部署。

```
Trace folder already exists. Traces will be written to existing trace folder: C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer\DeploymentTraces
Running Best Practices Analyzer...
Best Practices Analyzer completed successfully.


LocalAdminPrivilege        : True
IsJsonValid                : True
IsCabValid                 : True
RequiredPortsOpen          : True
RemoteRegistryAvailable    : True
FirewallAvailable          : True
RpcCheckPassed             : True
NoConflictingInstallations : True
FabricInstallable          : True
Passed                     : True
```

2. 创建群集：运行 *CreateServiceFabricCluster.ps1* 脚本，以便在配置中的每台计算机上部署 Service Fabric 群集。 
```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -AcceptEULA
```

> [!NOTE]
> 部署跟踪被写入到运行 CreateServiceFabricCluster.ps1 PowerShell 脚本的 VM/计算机上。 这些信息可以从运行脚本的目录中的子文件夹 DeploymentTraces 中找到。 按照群集配置文件 FabricSettings 部分中的详述，若要查看是否已将 Service Fabric 正确部署到计算机，请查找 FabricDataRoot 目录（默认为 c:\ProgramData\SF）中的已安装文件。 同样，可以看到 FabricHost.exe 和 Fabric.exe 进程在任务管理器中运行。
> 
> 

### <a name="step-2-connect-to-the-cluster"></a>步骤 2：连接到群集
若要连接到安全群集，请参阅 [Service Fabric 连接到安全群集](service-fabric-connect-to-secure-cluster.md)。

若要连接到不安全的群集，请运行以下 PowerShell 命令：

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <*IPAddressofaMachine*>:<Client connection end point port>

Connect-ServiceFabricCluster -ConnectionEndpoint 192.13.123.2345:19000
```
### <a name="step-3-bring-up-service-fabric-explorer"></a>步骤 3：打开 Service Fabric Explorer
现在可以通过 Service Fabric Explorer 连接到群集，既可以直接从装有 http://localhost:19080/Explorer/index.html 的某台计算机进行连接，也可以通过 http://<*IPAddressofaMachine*>:19080/Explorer/index.html 进行远程连接。

## <a name="add-and-remove-nodes"></a>添加和删除节点
当业务需要改变时，您可以向独立 Service Fabric 群集添加或删除节点。 参阅[向 Service Fabric 独立群集添加或删节点](service-fabric-cluster-windows-server-add-remove-nodes.md)以了解详细步骤。

<a id="removecluster" name="removecluster_anchor"></a>
## <a name="remove-a-cluster"></a>删除群集
若要删除群集，请运行包文件夹中的 *RemoveServiceFabricCluster.ps1* Powershell 脚本，并传入 JSON 配置文件的路径。 你可以选择指定删除的日志的位置。

可以在对群集配置文件中列为节点的所有计算机具有管理员访问权限的任何计算机上运行此脚本。 运行此脚本的计算机不一定是群集的一部分。

```
# Removes Service Fabric from each machine in the configuration
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -Force
```

```
# Removes Service Fabric from the current machine
.\CleanFabric.ps1
```

<a id="telemetry"></a>

## <a name="telemetry-data-collected-and-how-to-opt-out-of-it"></a>收集遥测数据以及如何选择不使用它
默认情况下，产品收集 Service Fabric 使用情况的遥测数据来改进产品。 作为安装程序一部分运行的最佳实践分析工具检查与 [https://vortex.data.microsoft.com/collect/v1](https://vortex.data.microsoft.com/collect/v1) 的连接。 如果不可访问，安装将失败，除非你选择不使用遥测。

1. 遥测管道尝试每日一次将以下数据上载到 [https://vortex.data.microsoft.com/collect/v1](https://vortex.data.microsoft.com/collect/v1)。 这是尽力而为的上载，不会影响群集功能。 仅从运行故障转移管理器的主节点发送遥测数据。 其他节点不发送遥测数据。
2. 遥测数据由以下项组成：

* 服务数量
* ServiceType 数量
* 应用程序数量
* ApplicationUpgrade 数量
* FailoverUnit 数量
* InBuildFailoverUnit 数量
* UnhealthyFailoverUnit 数量
* 副本数量
* InBuildReplica 数量
* StandByReplica 数量
* OfflineReplica 数量
* CommonQueueLength
* QueryQueueLength
* FailoverUnitQueueLength
* CommitQueueLength
* 节点数量
* IsContextComplete：True/False
* ClusterId：这是为每个群集随机生成的 GUID
* ServiceFabricVersion
* 从其上载遥测数据的虚拟机或计算机的 IP 地址

若要禁用遥测数据，请将以下内容添加到群集配置中的*属性*：*enableTelemetry: false*。

<a id="previewfeatures" name="previewfeatures_anchor"></a>

## <a name="preview-features-included-in-this-package"></a>此包中包括的预览功能
无。


> [!NOTE]
> 从[适用于 Windows Server 的独立群集的新 GA 版本（版本 5.3.204.x）](https://azure.microsoft.com/blog/azure-service-fabric-for-windows-server-now-ga/)开始，可以将群集手动或自动升级到未来版本。 请参阅[升级独立 Service Fabric 群集版本](service-fabric-cluster-upgrade-windows-server.md)文档以获取详细信息。
> 
> 

## <a name="next-steps"></a>后续步骤
* [使用 PowerShell 部署和删除应用程序](service-fabric-deploy-remove-applications.md)
* [独立 Windows 群集的配置设置](service-fabric-cluster-manifest.md)
* [在独立 Service Fabric 群集中添加或删除节点](service-fabric-cluster-windows-server-add-remove-nodes.md)
* [升级独立 Service Fabric 群集版本](service-fabric-cluster-upgrade-windows-server.md)
* [使用运行 Windows 的 Azure VM 创建独立 Service Fabric 群集](service-fabric-cluster-creation-with-windows-azure-vms.md)
* [使用 Windows 安全性保护 Windows 上的独立群集](service-fabric-windows-cluster-windows-security.md)
* [使用 X509 证书保护 Windows 上的独立群集](service-fabric-windows-cluster-x509-security.md)

<!--Image references-->
[Trusted Zone]: ./media/service-fabric-cluster-creation-for-windows-server/TrustedZone.png

