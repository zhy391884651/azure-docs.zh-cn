---
title: "Linux VM 上的 NAMD 与 Microsoft HPC Pack | Microsoft Docs"
description: "在 Azure 上部署 Microsoft HPC Pack 群集，并在多个 Linux 计算节点上通过 charmrun 运行 NAMD 模拟"
services: virtual-machines-linux
documentationcenter: 
author: dlepow
manager: timlt
editor: 
tags: azure-service-management,azure-resource-manager,hpc-pack
ms.assetid: 76072c6b-ac35-4729-ba67-0d16f9443bd7
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: big-compute
ms.date: 10/13/2016
ms.author: danlep
translationtype: Human Translation
ms.sourcegitcommit: 197ebd6e37066cb4463d540284ec3f3b074d95e1
ms.openlocfilehash: f46facee3e45704f74a13db7a18274f5ce90ceff
ms.lasthandoff: 03/31/2017


---
# <a name="run-namd-with-microsoft-hpc-pack-on-linux-compute-nodes-in-azure"></a>在 Azure 中的 Linux 计算节点上使用 Microsoft HPC Pack 运行 NAMD
本文介绍在 Azure 虚拟机上运行 Linux 高性能计算 (HPC) 工作负荷的一种方式。 在学习过程中，你将在包含 Linux 计算节点的 Azure 上设置一个 [Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029) 群集，然后运行 [NAMD](http://www.ks.uiuc.edu/Research/namd/) 仿真，以计算和直观呈现大型生物分子系统的结构。  

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

* **NAMD**（用于纳米级分子动力学程序）是并行分子动力学软件包，设计用于包含数百万个原子的大型生物分子系统的高性能仿真。 这些系统的例子包括病毒、细胞结构和大蛋白。 NAMD 扩展至数百个核心进行典型仿真，扩展至 500,000 个核心进行最大型仿真。
* **Microsoft HPC Pack** 可提供在本地计算机或 Azure 虚拟机群集上运行大型 HPC 和并行应用程序的功能。 HPC Pack 最初是针对 Windows HPC 工作负荷开发的一种解决方案，现在支持在 HPC Pack 群集中部署的 Linux 计算节点 VM 上运行 Linux HPC 应用程序。 有关简介，请参阅 [Get started with Linux compute nodes in an HPC Pack cluster in Azure](hpcpack-cluster.md)（Azure 的 HPC Pack 群集中的 Linux 计算节点入门）。

有关在 Azure 中运行 Linux HPC 工作负荷的其他选项，请参阅 [Technical resources for batch and high-performance computing](../../../batch/batch-hpc-solutions.md)（适用于批处理和高性能计算的技术资源）。

## <a name="prerequisites"></a>先决条件
* **包含 Linux 计算节点的 HPC Pack 群集** - 使用 [Azure Resource Manager 模板](https://azure.microsoft.com/marketplace/partners/microsofthpc/newclusterlinuxcn/)或 [Azure PowerShell 脚本](hpcpack-cluster-powershell-script.md)，在 Azure 上部署包含 Linux 计算节点的 HPC Pack 群集。 有关任一选项的先决条件和步骤，请参阅 [Get started with Linux compute nodes in an HPC Pack cluster in Azure](hpcpack-cluster.md)（Azure 的 HPC Pack 群集中的 Linux 计算节点入门）。 如果选择 PowerShell 脚本部署选项，请参阅本文末尾的示例文件中的示例配置文件。 此文件配置一个基于 Azure 的 HPC Pack 群集，其中包含 1 个 Windows Server 2012 R2 头节点和 4 个大型 CentOS 6.6 计算节点。 请根据环境的需要自定义此文件。
* **NAMD 软件和教程文件** - 从 [NAMD](http://www.ks.uiuc.edu/Research/namd/) 站点下载适用于 Linux 的 NAMD 软件（需要注册）。 本文基于 NAMD 版本 2.10，使用 [Linux x86_64（64 位 Intel/AMD 与以太网）](http://www.ks.uiuc.edu/Development/Download/download.cgi?UserID=&AccessCode=&ArchiveID=1310)存档。 另请下载 [NAMD 教程文件](http://www.ks.uiuc.edu/Training/Tutorials/#namd)。 下载的是 .tar 文件，需要使用某种 Windows 工具在群集头节点上解压缩该文件。 若要提取文件，请遵循本文稍后的说明。 
* **VMD**（可选）- 若要查看 NAMD 作业的结果，请在所选的计算机上下载并安装分子可视化程序 [VMD](http://www.ks.uiuc.edu/Research/vmd/)。 当前版本是 1.9.2。 请参阅 VMD 下载站点入门。  

## <a name="set-up-mutual-trust-between-compute-nodes"></a>在计算节点之间建立互信关系
在多个 Linux 节点上运行跨节点作业需要节点彼此信任（通过 **rsh** 或 **ssh**）。 使用 Microsoft HPC Pack IaaS 部署脚本创建 HPC Pack 群集时，此脚本会自动为指定的管理员帐户建立永久性互信关系。 对于在群集域中创建的非管理员用户，分配作业时必须在节点之间建立临时互信关系。 作业完成后，请销毁这种关系。 若要为每个用户执行此操作，请向群集提供一个 HPC Pack 用于建立信任关系的 RSA 密钥对。 说明如下。

### <a name="generate-an-rsa-key-pair"></a>生成一个 RSA 密钥对
通过运行 Linux **ssh keygen** 命令，可以很容易生成一个 RSA 密钥对，其中包含一个公钥和一个私钥。

1. 登录到 Linux 计算机。
2. 运行以下命令：
   
   ```bash
   ssh-keygen -t rsa
   ```
   
   > [!NOTE]
   > 按 **Enter** 使用默认设置，直至命令完成。 请勿在此处输入密码；系统提示输入密码时，只需按 **Enter** 即可。
   > 
   > 
   
   ![生成一个 RSA 密钥对][keygen]
3. 将目录切换到 ~/.ssh 目录。 私钥存储在 id_rsa 中，公钥存储在 id_rsa.pub 中。
   
   ![私钥和公钥][keys]

### <a name="add-the-key-pair-to-the-hpc-pack-cluster"></a>将密钥对添加到 HPC Pack 群集中
1. 使用部署群集时提供的域凭据（例如，hpc\clusteradmin）[通过远程桌面连接到头节点 VM](../../windows/connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 可以从头节点管理群集。
2. 使用 Windows Server 标准程序，在群集的 Active Directory 域中创建一个域用户帐户。 例如，在头节点上使用 Active Directory 用户和计算机工具。 本文中的示例假设你在 hpclab 域中创建了一个名为 hpcuser 的域用户 (hpclab\hpcuser)。
3. 将域用户作为群集用户添加到 HPC Pack 群集中。 有关说明，请参阅 [Add or remove cluster users](https://technet.microsoft.com/library/ff919330.aspx)（添加或删除群集用户）。
4. 创建一个名为 C:\cred.xml 的文件，将 RSA 密钥数据复制到此文件中。 你可以在本文末尾的示例文件中找到一个示例。
   
   ```
   <ExtendedData>
     <PrivateKey>Copy the contents of private key here</PrivateKey>
     <PublicKey>Copy the contents of public key here</PublicKey>
   </ExtendedData>
   ```
5. 打开命令提示符，输入以下命令，为 hpclab\hpcuser 帐户设置凭据数据。 使用 **extendeddata** 参数传递针对密钥数据创建的 C:\cred.xml 文件的名称。
   
   ```command
   hpccred setcreds /extendeddata:c:\cred.xml /user:hpclab\hpcuser /password:<UserPassword>
   ```
   
   此命令成功完成后，没有输出。 为你需要运行作业的用户帐户设置凭据后，请将 cred.xml 文件存储在安全位置，或者删除 cred.xml 文件。
6. 如果你在一个 Linux 节点上生成 RSA 密钥对，请记住在使用完成后删除这些密钥。 如果找到一个现有 id_rsa 文件或 id_rsa.pub 文件，HPC Pack 并不会建立互信关系。

> [!IMPORTANT]
> 我们不建议在共享群集上以群集管理员的身份运行 Linux 作业，因为由管理员提交的作业会在 Linux 节点的根帐户下运行。 由非管理员用户提交的作业会在本地 Linux 用户帐户（名称与作业用户相同）下运行。 在这种情况下，HPC Pack 将在该作业分配到的所有节点中为这位 Linux 用户建立互信关系。 在运行作业之前，你可以在 Linux 节点上手动设置 Linux 用户，也可以在提交作业时由 HPC Pack 自动创建用户。 如果 HPC Pack 创建了一个用户，则作业完成后 HPC Pack 会删除此用户。 为了减少安全威胁，请在节点上完成作业后删除密钥。
> 
> 

## <a name="set-up-a-file-share-for-linux-nodes"></a>为 Linux 节点设置文件共享
现在，请设置 SMB 文件共享，并在所有 Linux 节点上装入共享文件夹，以允许 Linux 节点使用通用路径访问 NAMD 文件。 下面是在头节点上装载共享文件夹的步骤。 对于 CentOS 6.6 这种目前不支持 Azure 文件服务的分发版，建议使用共享。 如果 Linux 节点支持 Azure 文件共享，请参阅 [How to use Azure File Storage with Linux](../../../storage/storage-how-to-use-files-linux.md)（如何通过 Linux 使用 Azure 文件存储）。 有关 HPC Pack 的其他文件共享选项，请参阅 [Get started with Linux compute nodes in an HPC Pack Cluster in Azure](hpcpack-cluster.md)（Azure 的 HPC Pack 群集中的 Linux 计算节点入门）。

1. 在头节点上创建一个文件夹，然后通过设置读/写权限将其与所有人共享。 在本示例中，\\\\CentOS66HN\Namd 是文件夹的名称，其中 CentOS66HN 是头节点的主机名称。
2. 在共享文件夹中创建一个名为 namd2 的子文件夹。 在 namd2 中创建名为 namdsample 的另一个子文件夹。
3. 使用 **tar** 的一个 Windows 版本或可以操作 .tar 存档文件的其他 Windows 实用工具，解压缩文件夹中的 NAMD 文件。 
   
   * 将 NAMD tar 存档解压缩到 \\\\CentOS66HN\Namd\namd2。
   * 在 \\\\CentOS66HN\Namd\namd2\namdsample 下解压缩教程文件。
4. 打开 Windows PowerShell 窗口并运行以下命令，以在 Linux 节点上装载共享文件夹。
   
    ```command
    clusrun /nodegroup:LinuxNodes mkdir -p /namd2
   
    clusrun /nodegroup:LinuxNodes mount -t cifs //CentOS66HN/Namd/namd2 /namd2 -o vers=2.1`,username=<username>`,password='<password>'`,dir_mode=0777`,file_mode=0777
    ```

第一个命令在 LinuxNodes 组中的所有节点上创建名为 /namd2 的文件夹。 第二个命令将共享文件夹 //CentOS66HN/Namd/namd2 装载到该文件夹上，并将 dir_mode 和 file_mode 位设置为 777。 该命令中的*用户名*和*密码*应是头节点上的用户的凭据。

> [!NOTE]
> 第二个命令中的“\`”符号是 PowerShell 的转义符。 “\`,”表示“,”（逗号）是命令的一部分。
> 
> 

## <a name="create-a-bash-script-to-run-a-namd-job"></a>创建 Bash 脚本以运行 NAMD 作业
NAMD 作业需要一个适用于 *charmrun* 的 **nodelist** 文件，确定启动 NAMD 流程时要使用的节点数量。 使用 Bash 脚本生成 nodelist 文件并使用此 nodelist 文件运行 **charmrun**。 然后，你可以在 HPC 群集管理器中提交 NAMD 作业并调用此脚本。

使用你选择的文本编辑器，在包含 NAMD 程序文件的 /namd2 文件夹中创建 Bash 脚本并将其命名为 hpccharmrun.sh。 为了快速完成概念认证，将复制本文末尾提供的示例 hpccharmrun.sh 脚本并转到“提交 NAMD 作业”。[](#submit-a-namd-job)

> [!TIP]
> 将你的脚本保存为带有 Linux 换行（仅 LF，而不是 CR LF）的文本文件。 这可确保其在 Linux 节点上正常运行。
> 
> 

下面是有关此 bash 脚本的作用的详细信息。 

1. 定义一些变量。
   
   ```bash
   #!/bin/bash
   
   # The path of this script
   SCRIPT_PATH="$( dirname "${BASH_SOURCE[0]}" )"
   # Charmrun command
   CHARMRUN=${SCRIPT_PATH}/charmrun
   # Argument of ++nodelist
   NODELIST_OPT="++nodelist"
   # Argument of ++p
   NUMPROCESS="+p"
   ```
2. 从环境变量中获取节点信息。 $NODESCORES 存储一个来自 $CCP_NODES_CORES 的拆分词列表。 $COUNT 是 $NODESCORES 的大小。
   
   ```
   # Get node information from the environment variables
   NODESCORES=(${CCP_NODES_CORES})
   COUNT=${#NODESCORES[@]}
   ```    
   
   $CCP_NODES_CORES 变量的格式如下所示：
   
   ```
   <Number of nodes> <Name of node1> <Cores of node1> <Name of node2> <Cores of node2>…
   ```
   
   此变量列出节点总数、节点名称以及分配到此作业的各个节点上的核心数。 例如，如果作业需要 10 个核心才能运行，则$CCP_NODES_CORES 值将类似于：
   
   ```
   3 CENTOS66LN-00 4 CENTOS66LN-01 4 CENTOS66LN-03 2
   ```
3. 如果没有设置 $CCP_NODES_CORES 变量，请直接启动 **charmrun**。 （这只应出现于在 Linux 节点上直接运行此脚本的情况。）
   
   ```
   if [ ${COUNT} -eq 0 ]
   then
     # CCP_NODES is_CORES is not found or is empty, so just run charmrun without nodelist arg.
     #echo ${CHARMRUN} $*
     ${CHARMRUN} $*
   ```
4. 或者创建适用于 **charmrun** 的 nodelist 文件。
   
   ```
   else
     # Create the nodelist file
     NODELIST_PATH=${SCRIPT_PATH}/nodelist_$$
   
     # Write the head line
     echo "group main" > ${NODELIST_PATH}
   
     # Get every node name and number of cores and write into the nodelist file
     I=1
     while [ ${I} -lt ${COUNT} ]
     do
         echo "host ${NODESCORES[${I}]} ++cpus ${NODESCORES[$(($I+1))]}" >> ${NODELIST_PATH}
         let "I=${I}+2"
     done
   ```
5. 按 nodelist 文件运行 **charmrun**，获取其返回状态，并在结束时删除 nodelist 文件。
   
   ${CCP_NUMCPUS} 是 HPC Pack 头节点设置的另一个环境变量。 它存储了分配到此作业的核心总数。 我们使用其指定 charmrun 的流程数。
   
   ```
   # Run charmrun with nodelist arg
   #echo ${CHARMRUN} ${NUMPROCESS}${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*
   ${CHARMRUN} ${NUMPROCESS}${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*
   
   RTNSTS=$?
   rm -f ${NODELIST_PATH}
   fi
   
   ```
6. 以 **charmrun** 返回状态退出。
   
   ```
   exit ${RTNSTS}
   ```

下面是脚本将生成的 nodelist 文件中的信息：

```
group main
host <Name of node1> ++cpus <Cores of node1>
host <Name of node2> ++cpus <Cores of node2>
…
```

例如：

```
group main
host CENTOS66LN-00 ++cpus 4
host CENTOS66LN-01 ++cpus 4
host CENTOS66LN-03 ++cpus 2
```

## <a name="submit-a-namd-job"></a>提交 NAMD 作业
现在你可以在 HPC 群集管理器中提交 NAMD 作业。

1. 连接到你的群集头节点，然后启动 HPC 群集管理器。
2. 在“资源管理”中，确保 Linux 计算节点处于“联机”状态。 如果节点未处于联机状态，请选择它们并单击“联机”。
3. 在“作业管理”中，单击“新作业”。
4. 为作业输入一个名称，如 *hpccharmrun*。
   
   ![新 HPC 作业][namd_job]
5. 在“作业详细信息”页中，在“作业资源”下，将资源类型选为“节点”，并将“最小数量”设置为 3。 我们将在 3 个 Linux 节点上运行此作业，每个节点有 4 个核心。
   
   ![作业资源][job_resources]
6. 在左侧导航栏中单击“编辑任务”，然后单击“添加”将任务添加到作业。    
7. 在“任务详细信息和 I/O 重定向”页中设置以下值。
   
   * **命令行** -
     `/namd2/hpccharmrun.sh ++remote-shell ssh /namd2/namd2 /namd2/namdsample/1-2-sphere/ubq_ws_eq.conf > /namd2/namd2_hpccharmrun.log`
     
     > [!TIP]
     > 上述命令行是不带换行符的单个命令。 但在“命令行”下，它将会换行在多行上显示。
     > 
     > 
   * **工作目录** - /namd2
   * **最小值** - 3
     
     ![任务详细信息][task_details]
     
     > [!NOTE]
     > 在此处设置工作目录，因为 **charmrun** 尝试导航到各个节点上相同的工作目录。 如果没有设置工作目录，则 HPC Pack 会启动在其中一个 Linux 节点上创建的一个随机命名文件夹中的命令。 这会导致其他节点上出现以下错误：`/bin/bash: line 37: cd: /tmp/nodemanager_task_94_0.mFlQSN: No such file or directory.` 为避免发生此问题，请指定一个可作为工作目录由所有节点进行访问的文件夹路径。
     > 
     > 
8. 单击“确定”，然后单击“提交”运行此作业。
   
   默认情况下，HPC Pack 将以当前登录的用户帐户提交作业。 单击“提交”后，对话框可能会提示输入用户名和密码。
   
   ![作业凭据][creds]
   
   在某些情况下，HPC Pack 会记住以前输入的用户信息，并不会显示此对话框。 为了使 HPC Pack 再次显示此对话框，请在命令提示符下输入以下命令，然后提交作业。
   
   ```command
   hpccred delcreds
   ```
9. 作业需要几分钟时间才能完成。
10. 在 \\<headnodeName>\Namd\namd2\namd2_hpccharmrun.log 中找到作业日志，在 \\<headnodeName>\Namd\namd2\namdsample\1-2-sphere\. 中找到输出文件。
11. 也可启动 VMD 以查看你的作业结果。 可视化 NAMD 输出文件的步骤（在本例中，水中的 ubiquitin 蛋白质分子）不在本文的讨论范围之内。 有关详细信息，请参阅 [NAMD 教程](http://www.life.illinois.edu/emad/biop590c/namd-tutorial-unix-590C.pdf)。
    
    ![作业结果][vmd_view]

## <a name="sample-files"></a>示例文件
### <a name="sample-xml-configuration-file-for-cluster-deployment-by-powershell-script"></a>使用 PowerShell 脚本部署群集时可用的示例 XML 配置文件
```
<?xml version="1.0" encoding="utf-8" ?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>Subscription-1</SubscriptionName>
    <StorageAccount>mystorageaccount</StorageAccount>
  </Subscription>
      <Location>West US</Location>  
  <VNet>
    <VNetName>MyVNet</VNetName>
    <SubnetName>Subnet-1</SubnetName>
  </VNet>
  <Domain>
    <DCOption>HeadNodeAsDC</DCOption>
    <DomainFQDN>hpclab.local</DomainFQDN>
  </Domain>
  <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>CentOS66HN</VMName>
    <ServiceName>MyHPCService</ServiceName>
    <VMSize>Large</VMSize>
    <EnableRESTAPI />
    <EnableWebPortal />
  </HeadNode>
  <LinuxComputeNodes>
    <VMNamePattern>CentOS66LN-%00%</VMNamePattern>
    <ServiceName>MyLnxCNService</ServiceName>
     <VMSize>Large</VMSize>
     <NodeCount>4</NodeCount>
    <ImageName>5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-66-20150325</ImageName>
  </LinuxComputeNodes>
</IaaSClusterConfig>    
```

### <a name="sample-credxml-file"></a>示例 cred.xml 文件
```
<ExtendedData>
  <PrivateKey>-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAxJKBABhnOsE9eneGHvsjdoXKooHUxpTHI1JVunAJkVmFy8JC
qFt1pV98QCtKEHTC6kQ7tj1UT2N6nx1EY9BBHpZacnXmknpKdX4Nu0cNlSphLpru
lscKPR3XVzkTwEF00OMiNJVknq8qXJF1T3lYx3rW5EnItn6C3nQm3gQPXP0ckYCF
Jdtu/6SSgzV9kaapctLGPNp1Vjf9KeDQMrJXsQNHxnQcfiICp21NiUCiXosDqJrR
AfzePdl0XwsNngouy8t0fPlNSngZvsx+kPGh/AKakKIYS0cO9W3FmdYNW8Xehzkc
VzrtJhU8x21hXGfSC7V0ZeD7dMeTL3tQCVxCmwIDAQABAoIBAQCve8Jh3Wc6koxZ
qh43xicwhdwSGyliZisoozYZDC/ebDb/Ydq0BYIPMiDwADVMX5AqJuPPmwyLGtm6
9hu5p46aycrQ5+QA299g6DlF+PZtNbowKuvX+rRvPxagrTmupkCswjglDUEYUHPW
05wQaNoSqtzwS9Y85M/b24FfLeyxK0n8zjKFErJaHdhVxI6cxw7RdVlSmM9UHmah
wTkW8HkblbOArilAHi6SlRTNZG4gTGeDzPb7fYZo3hzJyLbcaNfJscUuqnAJ+6pT
iY6NNp1E8PQgjvHe21yv3DRoVRM4egqQvNZgUbYAMUgr30T1UoxnUXwk2vqJMfg2
Nzw0ESGRAoGBAPkfXjjGfc4HryqPkdx0kjXs0bXC3js2g4IXItK9YUFeZzf+476y
OTMQg/8DUbqd5rLv7PITIAqpGs39pkfnyohPjOe2zZzeoyaXurYIPV98hhH880uH
ZUhOxJYnlqHGxGT7p2PmmnAlmY4TSJrp12VnuiQVVVsXWOGPqHx4S4f9AoGBAMn/
vuea7hsCgwIE25MJJ55FYCJodLkioQy6aGP4NgB89Azzg527WsQ6H5xhgVMKHWyu
Q1snp+q8LyzD0i1veEvWb8EYifsMyTIPXOUTwZgzaTTCeJNHdc4gw1U22vd7OBYy
nZCU7Tn8Pe6eIMNztnVduiv+2QHuiNPgN7M73/x3AoGBAOL0IcmFgy0EsR8MBq0Z
ge4gnniBXCYDptEINNBaeVStJUnNKzwab6PGwwm6w2VI3thbXbi3lbRAlMve7fKK
B2ghWNPsJOtppKbPCek2Hnt0HUwb7qX7Zlj2cX/99uvRAjChVsDbYA0VJAxcIwQG
TxXx5pFi4g0HexCa6LrkeKMdAoGAcvRIACX7OwPC6nM5QgQDt95jRzGKu5EpdcTf
g4TNtplliblLPYhRrzokoyoaHteyxxak3ktDFCLj9eW6xoCZRQ9Tqd/9JhGwrfxw
MS19DtCzHoNNewM/135tqyD8m7pTwM4tPQqDtmwGErWKj7BaNZARUlhFxwOoemsv
R6DbZyECgYEAhjL2N3Pc+WW+8x2bbIBN3rJcMjBBIivB62AwgYZnA2D5wk5o0DKD
eesGSKS5l22ZMXJNShgzPKmv3HpH22CSVpO0sNZ6R+iG8a3oq4QkU61MT1CfGoMI
a8lxTKnZCsRXU1HexqZs+DSc+30tz50bNqLdido/l5B4EJnQP03ciO0=
-----END RSA PRIVATE KEY-----</PrivateKey>
  <PublicKey>ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDEkoEAGGc6wT16d4Ye+yN2hcqigdTGlMcjUlW6cAmRWYXLwkKoW3WlX3xAK0oQdMLqRDu2PVRPY3qfHURj0EEellpydeaSekp1fg27Rw2VKmEumu6Wxwo9HddXORPAQXTQ4yI0lWSerypckXVPeVjHetbkSci2foLedCbeBA9c/RyRgIUl227/pJKDNX2Rpqly0sY82nVWN/0p4NAyslexA0fGdBx+IgKnbU2JQKJeiwOomtEB/N492XRfCw2eCi7Ly3R8+U1KeBm+zH6Q8aH8ApqQohhLRw71bcWZ1g1bxd6HORxXOu0mFTzHbWFcZ9ILtXRl4Pt0x5Mve1AJXEKb username@servername;</PublicKey>
</ExtendedData>
```

### <a name="sample-hpccharmrunsh-script"></a>示例 hpccharmrun.sh 脚本
```
#!/bin/bash

# The path of this script
SCRIPT_PATH="$( dirname "${BASH_SOURCE[0]}" )"
# Charmrun command
CHARMRUN=${SCRIPT_PATH}/charmrun
# Argument of ++nodelist
NODELIST_OPT="++nodelist"
# Argument of ++p
NUMPROCESS="+p"

# Get node information from ENVs
# CCP_NODES_CORES=3 CENTOS66LN-00 4 CENTOS66LN-01 4 CENTOS66LN-03 4
NODESCORES=(${CCP_NODES_CORES})
COUNT=${#NODESCORES[@]}

if [ ${COUNT} -eq 0 ]
then
    # If CCP_NODES_CORES is not found or is empty, just run the charmrun without nodelist arg.
    #echo ${CHARMRUN} $*
    ${CHARMRUN} $*
else
    # Create the nodelist file
    NODELIST_PATH=${SCRIPT_PATH}/nodelist_$$

    # Write the head line
    echo "group main" > ${NODELIST_PATH}

    # Get every node name & cores and write into the nodelist file
    I=1
    while [ ${I} -lt ${COUNT} ]
    do
        echo "host ${NODESCORES[${I}]} ++cpus ${NODESCORES[$(($I+1))]}" >> ${NODELIST_PATH}
        let "I=${I}+2"
    done

    # Run the charmrun with nodelist arg
    #echo ${CHARMRUN} ${NUMPROCESS}${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*
    ${CHARMRUN} ${NUMPROCESS}${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*

    RTNSTS=$?
    rm -f ${NODELIST_PATH}
fi

exit ${RTNSTS}
```

 

<!--Image references-->
[keygen]:media/hpcpack-cluster-namd/keygen.png
[keys]:media/hpcpack-cluster-namd/keys.png
[namd_job]:media/hpcpack-cluster-namd/namd_job.png
[job_resources]:media/hpcpack-cluster-namd/job_resources.png
[creds]:media/hpcpack-cluster-namd/creds.png
[task_details]:media/hpcpack-cluster-namd/task_details.png
[vmd_view]:media/hpcpack-cluster-namd/vmd_view.png

