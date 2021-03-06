---
title: "Azure Kubernetes 群集的服务主体 | Microsoft Docs"
description: "在 Azure 容器服务中为 Kubernetes 群集创建和管理 Azure Active Directory 服务主体"
services: container-service
documentationcenter: 
author: dlepow
manager: timlt
editor: 
tags: acs, azure-container-service, kubernetes
keywords: 
ms.assetid: 
ms.service: container-service
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/08/2017
ms.author: danlep
ms.translationtype: Human Translation
ms.sourcegitcommit: e7da3c6d4cfad588e8cc6850143112989ff3e481
ms.openlocfilehash: 33a1ab822f2900fd51d801f94ad8679fe65ba21f
ms.contentlocale: zh-cn
ms.lasthandoff: 05/16/2017


---

# <a name="set-up-an-azure-ad-service-principal-for-a-kubernetes-cluster-in-container-service"></a>在容器服务中为 Kubernetes 群集设置 Azure AD 服务主体


在 Azure 容器服务中，Kubernetes 群集需要 [Azure Active Directory 服务主体](../active-directory/active-directory-application-objects.md)才能与 Azure API 交互。 需要服务主体才能动态管理相关资源，例如[用户定义路由](../virtual-network/virtual-networks-udr-overview.md)和[第 4 层 Azure 负载均衡器](../load-balancer/load-balancer-overview.md)。 


本文介绍用于为 Kubernetes 群集设置服务主体的不同选项。 例如，如果已安装并设置 [Azure CLI 2.0](/cli/azure/install-az-cli2)，则可运行 [`az acs create`](/cli/azure/acs#create) 命令，以便同时创建 Kubernetes 群集和服务主体。


## <a name="requirements-for-the-service-principal"></a>服务主体的的要求

可以使用现有的符合以下条件的 Azure AD 服务主体，也可以创建一个新的。

* **作用域**：订阅中用于部署 Kubernetes 群集的资源组，或（不太严格地来说）用于部署该群集的订阅。

* **角色**：**参与者**

* **客户端机密**：必须是密码。 目前，无法使用针对证书身份验证设置的服务主体。

> [!IMPORTANT] 
> 若要创建服务主体，你必须具有相应的权限，能够向 Azure AD 租户注册应用程序，并将应用程序分配到订阅中的角色。 若要了解你是否具有必需的权限，请[在门户中查看](../azure-resource-manager/resource-group-create-service-principal-portal.md#required-permissions)。 
>

## <a name="option-1-create-a-service-principal-in-azure-ad"></a>选项 1：在 Azure AD 中创建服务主体

如需在部署 Kubernetes 群集之前创建 Azure AD 服务主体，可以使用 Azure 提供的多种方法。 

以下示例命令演示如何通过 [Azure CLI 2.0](../azure-resource-manager/resource-group-authenticate-service-principal-cli.md) 执行此操作。 也可使用 [Azure PowerShell](../azure-resource-manager/resource-group-authenticate-service-principal.md)、[门户](../azure-resource-manager/resource-group-create-service-principal-portal.md)等方法创建服务主体。

```azurecli
az login

az account set --subscription "mySubscriptionID"

az group create -n "myResourceGroupName" -l "westus"

az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/mySubscriptionID/resourceGroups/myResourceGroupName"
```

输出如下所示（此处以密文形式显示）：

![创建服务主体](./media/container-service-kubernetes-service-principal/service-principal-creds.png)

突出显示的是“客户端 ID”(`appId`) 和“客户端机密”(`password`)，用作群集部署的服务主体参数。


### <a name="specify-service-principal-when-creating-the-kubernetes-cluster"></a>在创建 Kubernetes 群集时指定服务主体

创建 Kubernetes 群集时，提供现有服务主体的“客户端 ID”（也称为 `appId`，即应用程序 ID）和“客户端机密”(`password`) 作为参数。 请确保服务主体满足本文开头的要求。

可以在使用 [Azure 命令行界面 (CLI) 2.0](container-service-kubernetes-walkthrough.md)、[Azure 门户](./container-service-deployment.md)等方法部署 Kubernetes 群集时指定这些参数。

>[!TIP] 
>指定“客户端 ID”时，请确保使用服务主体的 `appId` 而不是 `ObjectId`。
>

以下示例说明了一种通过 Azure CLI 2.0 传递参数的方法。 此示例使用 [Kubernetes 快速启动模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-acs-kubernetes)。

1. 从 GitHub [下载](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-acs-kubernetes/azuredeploy.parameters.json)模板参数文件 `azuredeploy.parameters.json`。

2. 若要指定服务主体，请在文件中输入 `servicePrincipalClientId` 和 `servicePrincipalClientSecret` 的值。 （还需提供自己的适用于 `dnsNamePrefix` 和 `sshRSAPublicKey` 的值。 后者是用于访问群集的 SSH 公钥。）保存文件。

    ![传递服务主体参数](./media/container-service-kubernetes-service-principal/service-principal-params.png)

3. 运行以下命令，使用 `--parameters` 设置 azuredeploy.parameters.json 文件的路径。 此命令将群集部署在用户创建的名为 `myResourceGroup` 的资源组（位于“美国西部”区域）中。

    ```azurecli
    az login

    az account set --subscription "mySubscriptionID"

    az group create --name "myResourceGroup" --location "westus" 
    
    az group deployment create -g "myResourceGroup" --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-acs-kubernetes/azuredeploy.json" --parameters @azuredeploy.parameters.json
    ```


## <a name="option-2-generate-a-service-principal-when-creating-the-cluster-with-az-acs-create"></a>选项 2：在通过 `az acs create` 创建群集时生成服务主体

如果通过运行 [`az acs create`](/cli/azure/acs#create) 命令来创建 Kubernetes 群集，则可选择自动生成服务主体。

与其他 Kubernetes 群集创建选项一样，可以在运行 `az acs create` 时指定现有服务主体的参数。 不过，在省略这些参数的情况下，Azure CLI 会自动创建一个适用于容器服务的服务主体。 该操作以透明方式在部署过程中进行。 

以下命令创建 Kubernetes 群集，并生成 SSH 密钥和服务主体凭据：

```console
az acs create -n myClusterName -d myDNSPrefix -g myResourceGroup --generate-ssh-keys --orchestrator-type kubernetes
```

> [!IMPORTANT]
> 如果帐户没有创建服务主体所需的 Azure AD 和订阅权限，该命令会生成类似于`Insufficient privileges to complete the operation.`的错误
> 

## <a name="additional-considerations"></a>其他注意事项

* 如果没有在订阅中创建服务主体的权限，则可能需要请求 Azure AD 或订阅的管理员为你分配必需的权限，或者要求他们提供适用于 Azure 容器服务的服务主体。 

* Kubernetes 的服务主体是群集配置的一部分。 但是，请勿使用标识来部署群集。

* 每个服务主体都与一个 Azure AD 应用程序相关联。 Kubernetes 群集的服务主体可以与任何有效的 Azure AD 应用程序名称（例如 `https://www.contoso.org/example`）相关联。 应用程序的 URL 不一定是实际的终结点。

* 指定服务主体的“客户端 ID”时，可以使用 `appId` 的值（如本文所示）或相应的服务主体 `name`（例如，`https://www.contoso.org/example`）。

* 在 Kubernetes 群集的主 VM 和代理 VM 中，服务主体凭据存储在 /etc/kubernetes/azure.json 文件中。

* 使用 `az acs create` 命令自动生成服务主体时，会将服务主体凭据写入用于运行命令的计算机上的 ~/.azure/acsServicePrincipal.json 文件中。 

* 使用 `az acs create` 命令自动生成服务主体时，服务主体也可以使用在同一订阅中创建的 [Azure 容器注册表](../container-registry/container-registry-intro.md)进行身份验证。




## <a name="next-steps"></a>后续步骤

* [开始使用 Kubernetes](container-service-kubernetes-walkthrough.md)（位于容器服务群集中）。

* 若要排查 Kubernetes 的服务主体问题，请参阅 [ACS 引擎文档](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes.md#troubleshooting)。



