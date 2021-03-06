## <a name="general-considerations-for-n-series-vms"></a>N 系列 VM 的一般注意事项

* 有关 N 系列 VM 的可用性，请查看[可用产品(按区域)](https://azure.microsoft.com/en-us/regions/services/)。

* N 系列 VM 只能按 Resource Manager 部署模型部署。

* 使用 Azure 门户创建 N 系列的 VM 时，请在“基本信息”边栏选项卡中选择“HDD”作为“VM 磁盘类型”。 若要选择可用的 N 系列大小，请在“大小”边栏选项卡中单击“全部查看”。

* N 系列 VM 不支持 Azure 高级存储所支持的 VM 磁盘。

* 如果需要部署的 N 系列 VM 较多，请考虑使用即用即付订阅或其他购买选项。 如果使用的是 [Azure 免费帐户](https://azure.microsoft.com/free/)，则仅可以使用有限数量的 Azure 计算核心。

* 你可能需要提高 Azure 订阅中的核心配额（按区域）以及单独针对 NC 或 NV 核心的配额。 若要请求提高配额，可免费[提出在线客户支持请求](../articles/azure-supportability/how-to-create-azure-support-request.md)。 默认限制可能因订阅类别而异。







