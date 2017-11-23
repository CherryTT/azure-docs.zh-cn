---
title: "部署使用 Azure SQL 数据库的分片多租户数据库 SaaS 应用程序 | Microsoft 文档"
description: "部署和浏览分片 Wingtip Tickets SaaS 多租户数据库应用程序，演示使用 Azure SQL 数据库的 SaaS 模式。"
keywords: "sql 数据库教程"
services: sql-database
documentationcenter: 
author: stevestein
manager: craigg
editor: billgib;MightyPen
ms.service: sql-database
ms.custom: scale out apps
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: sstein
ms.openlocfilehash: bc96221abf62677b53df43daa44a925ac5792043
ms.sourcegitcommit: 9a61faf3463003375a53279e3adce241b5700879
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2017
---
# <a name="deploy-and-explore-a-sharded-multi-tenant-application-that-uses-azure-sql-database"></a>部署和浏览使用 Azure SQL 数据库的分片多租户应用程序

在本教程中，你将部署和浏览名为 Wingtip Tickets 的示例 SaaS 多租户数据库应用程序。 Wingtip 应用旨在展示可简化 SaaS 方案实现的 Azure SQL 数据库功能。

Wingtips 的这个实现将使用分片多租户数据库模式。 分片按租户标识符进行。 租户数据根据租户标识符值分布到特定数据库。 无论任何给定数据库包含多少个租户，所有数据库都是多租户的，因为表架构包含租户标识符。 

此数据库模式允许在每个分片或数据库中存储一个或多个租户。 你可以通过让多个租户共享每个数据库来优化最低成本。 或者可以通过让每个数据库只存储一个租户来优化隔离。 可针对每个特定租户独立进行优化选择。 可以在首次存储租户时进行选择，也可以稍后再进行更改。 无论采用哪种方式，应用程序均可正常运行。

#### <a name="app-deploys-quickly"></a>应用程序快速部署

下面的部署部分提供了“部署到 Azure”按钮。 当按下此按钮时，Wingtip 应用将在 5 分钟内完全部署完成。 Wingtip 应用在 Azure 云中运行，并使用 Azure SQL 数据库。 Wingtip 部署到 Azure 订阅。 你具有完全访问权限，可以使用各个应用程序组件。

此应用程序部署了三个示例租户的数据。 这些租户一起存储在一个多租户数据库中。

任何人都可以从我们的 [Github 存储库][link-github-wingtip-multitenantdb-55g]下载 Wingtip Tickets 的 C# 和 PowerShell 源代码。

#### <a name="learn-in-this-tutorial"></a>学习本教程

> [!div class="checklist"]

> - 如何部署 Wingtip SaaS 应用程序。
> - 获取应用程序源代码和管理脚本的位置。
> - 关于构成该应用的服务器和数据库。
> - 如何通过目录将租户映射到其数据。
> - 如何预配新租户。
> - 如何在应用中监视租户的活动。

可以使用在此初始部署的基础上构建的一系列相关教程。 这些教程探讨了一系列 SaaS 设计和管理模式。 完成教程时，鼓励你逐步执行所提供的脚本，以了解各种 SaaS 模式是如何实现的。

## <a name="prerequisites"></a>先决条件

若要完成本教程，请确保已完成了以下先决条件：

- 安装最新的 Azure PowerShell。 有关详细信息，请参阅 [Azure PowerShell 入门][link-azure-get-started-powershell-41q]。

## <a name="deploy-the-wingtip-tickets-app"></a>部署 Wingtip Tickets 应用

1. 单击以下“部署到 Azure”按钮。
    - 带有 Wingtip Tickets SaaS 部署模板的 Azure 门户随即打开。 此模板要求提供两个参数值；新资源组的名称和用于区分该部署和应用中其他部署的“用户”值。 下一个步骤提供了设置这些参数值的详细信息。
        - 请务必记下你使用的确切值，因为稍后的配置文件还需要用到。

    [![“部署到 Azure”按钮。][image-deploy-to-azure-blue-48d]][link-aka-ms-deploywtp-mtapp-52k]

2. 输入该部署所需的参数值。

    > [!IMPORTANT]
    > 为了便于演示，某些身份验证和服务器防火墙设置将有意设置为不安全。 请选择“新建资源组”，不要使用现有资源组、服务器或池。 不要使用该应用程序及其创建的任何资源进行生产。 使用完该应用程序时请删除此资源组，以停止相关计费。

    最好在资源名称中仅使用小写字母、数字和连字符。

    - 对于资源组 - 选择“新建”，然后为资源组提供一个名称（区分大小写）。
        - 建议资源组名称中的字母全部采用小写形式。
        - 建议后面附加短横线，依次后跟首字母、数字，例如 wingtip-af1。
        - 从下拉列表中选择一个**位置**。
    - 对于用户 - 建议选择一个较短的用户值，例如在首字母后加一个数字：af1。

3. **部署应用程序**。

    - 单击“我同意上述条款和条件”。
    - 单击“购买”。

4. 通过单击“通知”（搜索框右侧的钟形图标）监视部署状态。 部署 Wingtip 应用大约需要 5 分钟。

   ![部署成功](media/saas-multitenantdb-get-started-deploy/succeeded.png)

## <a name="download-and-unblock-the-management-scripts"></a>下载和取消阻止管理脚本

在应用程序部署的同时，下载应用程序源代码和管理脚本。

> [!IMPORTANT]
> 从外部源下载和提取 zip 文件时，可执行内容（脚本、ddl）可能受 Windows 阻止。 从 zip 文件提取脚本时，请按照以下步骤取消阻止 .zip 文件，然后再进行提取。 通过取消阻止 .zip 文件，确保允许脚本运行。

1. 浏览到 [WingtipTicketsSaaS-MultiTenantDb github 存储库](https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDb)。
2. 单击“克隆或下载”。
3. 单击“下载 ZIP”并保存文件。
4. 右键单击“WingtipTicketsSaaS-MultiTenantDb-master.zip”文件，然后选择“属性”。
5. 在“常规”选项卡上，选择“取消阻止”，然后单击“应用”。
6. 单击“确定”。
7. 解压缩文件。

脚本位于 *..\\WingtipTicketsSaaS-MultiTenantDb-master\\Learning Modules\\* 文件夹中。

## <a name="update-the-configuration-file-for-this-deployment"></a>更新此部署的配置文件

运行任何脚本之前，在 UserConfig.psm1 中设置“资源组”和“用户”值。 将这些变量设置为在部署期间设置的相同值。

1. 在 *PowerShell ISE* 中打开 ...\\Learning Modules\\*UserConfig.psm1*
2. 使用部署（仅限 10 和 11 行）的特定值，更新 ResourceGroupName 和 Name。
3. 保存更改。

所有脚本均可使用在此文件中设置的值，因此它们的准确性非常重要。 如果重新部署应用，请确保选择不同的资源组和用户值。 然后用新值更新用户配置。

## <a name="run-the-application"></a>运行应用程序

该应用将展示举办活动的地点，如音乐厅、爵士乐俱乐部、运动俱乐部。 地点将作为 Wingtip 平台的客户（或租户）注册，以便于列出活动和销售票证。 每个地点均获得一个用于管理和列出其活动并销售票证的个性化 Web 应用，每个地点都是独立的，并且与其他租户隔离。 在封面下，每个租户的数据都默认存储在分片多租户数据库中。

中间的“事件中心”提供特定部署中租户的链接列表。

1. 在 Web 浏览器中打开“事件中心”：
    - http://events.wingtip.&lt;USER&gt;.trafficmanager.net &nbsp;（替换为部署用户值。）

    ![事件中心](media/saas-multitenantdb-get-started-deploy/events-hub.png)

2. 单击“活动中心”中的“Fabrikam 爵士乐俱乐部”。

   ![事件](./media/saas-multitenantdb-get-started-deploy/fabrikam.png)

该应用使用 [Azure 流量管理器](../traffic-manager/traffic-manager-overview.md)来控制传入请求的分配。 租户特定的事件页需要租户名称包含在 URL 中。 此 URL 还包含特定用户值，并遵循以下格式：

- http://events.wingtip.&lt;USER&gt;.trafficmanager.net/*fabrikamjazzclub*
 
事件应用将分析 URL 中的租户名，并采用哈希算法来创建密钥，以使用[分片映射管理](sql-database-elastic-scale-shard-map-management.md)来访问目录。 该目录将密钥映射到租户的数据库位置。 “事件中心”将列出在目录中注册的所有租户。 “事件中心”使用目录中扩展的元数据来检索与每个映射关联的租户名称以构造 URL。

在生产环境中，通常要创建一条 CNAME DNS 记录，以[将公司 Internet 域指向](../traffic-manager/traffic-manager-point-internet-domain.md)流量管理器配置文件。

## <a name="start-generating-load-on-the-tenant-databases"></a>开始在租户数据库上生成负载

现在已部署应用，接下来让我们投入使用！ Demo-LoadGenerator PowerShell 脚本将启动针对每个租户运行的一个工作负载。 许多 SaaS 应用上的实际负载通常是偶发性的且不可预测。 所以，为了模拟此类型的负载，负载生成器将生成分布在所有租户上的负载。 该负载包括以随机间隔期间发生的针对每个租户的随机突发。 负载模式显现会需要几分钟的时间，因此，最好让生成器先运行至少 3 或 4 分钟，然后再进行负载监测。

1. 在 PowerShell ISE 中打开 ...\\Learning Modules\\Utilities\\Demo-LoadGenerator.ps1 脚本。
2. 按 **F5** 运行该脚本并启动负载生成器（目前保留默认参数值）。

> [!IMPORTANT]
> Demo-LoadGenerator.ps1 脚本将打开运行负载生成器的另一个 PowerShell 会话。 负载生成器作为前台任务在此会话中运行，用于调用后台负载生成作业，每个租户一个。

前台任务启动后，它仍处于作业调用状态。 该任务将为随后预配的任何新租户启动额外的后台作业。

关闭 PowerShell 会话将停止所有作业

你可能需要重新启动负载生成器会话才能使用不同的参数值。 如果是这样，请关闭 PowerShell 生成会话，然后重新运行 *Demo-LoadGenerator.ps1*。

## <a name="provision-a-new-tenant-into-the-sharded-database"></a>将新租户预配到分片数据库

初始部署在 Tenants1 数据库中包括了三个示例租户。 让我们再创建一个租户，看看这会对已部署的应用程序有怎样的影响。 在此步骤中将快速创建新租户。

1. 在 PowerShell ISE 中打开 ...\\Learning Modules\Provision and Catalog\\*Demo-ProvisionTenants.ps1*。
2. 按“F5”运行该脚本（目前保留默认值）。

   > [!NOTE]
   > 许多 Wingtip Tickets SaaS 脚本都使用 $ PSScriptRoot 来允许导航文件夹、调用其他脚本或导入模块。 只有通过按“F5”完整执行脚本的情况下才能计算该变量。  突出显示和运行某个选择的内容 (F8) 时，可能会生成错误，因此请在运行脚本时按 F5。

新的 Red Maple Racing 租户已添加到 Tenants1 数据库中，并且已在目录中注册。 新租户的票证销售“事件”站点将在浏览器中打开：

![新租户](./media/saas-multitenantdb-get-started-deploy/red-maple-racing.png)

刷新事件中心，新租户将出现在列表中。

## <a name="provision-a-new-tenant-in-its-own-database"></a>在其自己数据库中预配新租户

分片多租户模型允许选择是在包含其他租户的数据库中预配新租户，还是在其自己的数据库中预配租户。 自身的租户可通过使其数据从其他租户的数据中隔离出来而受益。 通过隔离可使你独立于其他租户来管理该租户的性能。 此外，对于隔离的租户，将数据恢复到较早的时间更为容易。 你可以选择将免费试用版或常规客户放入多租户数据库，将高级客户放在单独数据库中。 如果创建大量每个当中只包含一个租户的数据库，则可以在弹性池中进行集中管理，以优化资源成本。  

现在我们预配其他租户，这次是在自己的数据库中。

1. 在 ...\\Learning Modules\\Provision and Catalog\*Demo-ProvisionTenants.ps1* 中，将 $TenantName 修改为 Salix Salsa，将 $VenueType 修改为 Dance，并将 $Scenario 修改为 2。

2. 按 F5 再次运行脚本。
    - 此次按 F5 将预配单独数据库中的新租户。 数据库和租户在目录中注册。 然后，浏览器打开到租户的“事件”页。

   ![Salix Salsa 事件页](./media/saas-multitenantdb-get-started-deploy/salix-salsa.png)

   - 滚动到页面底部。 在横幅中可以看到存储租户数据的数据库名称。

3. 刷新“事件中心”，两个新租户将会出现在列表中。



## <a name="explore-the-servers-and-tenant-databases"></a>浏览服务器和租户数据库

现在我们看一些已部署的资源：

1. 在 [Azure 门户](http://portal.azure.com)中，浏览到资源组的列表。 打开你在部署应用程序时创建的资源组。

   ![资源组](./media/saas-multitenantdb-get-started-deploy/resource-group.png)

2. 单击 **catalog-mt&lt;USER&gt;** 服务器。 目录服务器包含两个名为 tenantcatalog 和 basetenantdb 的数据库。 Basetenantdb 数据库是空的模板数据库。 复制该模板数据库来创建一个新的租户数据库，可用于多租户或仅用于一个租户。

   ![编录服务器](./media/saas-multitenantdb-get-started-deploy/catalog-server.png)

3. 返回到该资源组并选择保存租户数据库的 tenants1-mt 服务器。
    - tenants1 数据库是一个多租户数据库，其中存储有初始的三个租户以及你添加的第一个租户。 它被配置为 50 个 DTU 标准数据库。
    - Salixsalsa 数据库将 Salix Salsa 舞蹈场所保存为其唯一租户。 它被配置为标准版本数据库，默认有 50 个 DTU。

   ![租户服务器](./media/saas-multitenantdb-get-started-deploy/tenants-server.png)


## <a name="monitor-the-performance-of-the-database"></a>监测数据库的性能

如果负载生成器已运行几分钟，那么应有足够的遥测数据可用于开始了解一些内置于门户中的数据库监测功能。

1. 浏览到 **tenants1-mt&lt;USER&gt;** 服务器，然后单击“tenants1”，查看在其中具有四个租户的数据库资源利用率。 每个租户都受到来自负载生成器的偶发过大负载的影响：

   ![监视 tenants1](./media/saas-multitenantdb-get-started-deploy/monitor-tenants1.png)

   DTU 利用率图表很好地说明了多租户数据库支持多租户不可预测的工作负载的方式。 在这种情况下，负载生成器正在向每个租户应用大约 30 个 DTU 的偶发负载。 此负载相当于 50 个 DTU 数据库的 60% 的利用率。 超过 60% 的峰值是同时向多个租户应用负载的结果。

2. 浏览到 **tenants1-mt&lt;USER&gt;** 服务器，然后单击“salixsalsa”数据库，查看仅包含一个租户的数据库资源利用率。

   ![salixsalsa 数据库](./media/saas-multitenantdb-get-started-deploy/monitor-salix.png)

负载生成器正在向每个租户应用类似负载，而不管每个租户在哪个数据库中。 在 salixsalsa 数据库中只有一个租户的情况下，你可以看到该数据库可以承受比有多个租户的数据库高得多的负载。 

> [!NOTE]
> 负载生成器创建的负载仅用于说明目的。  分配给多租户和单租户数据库的资源，以及在多租户数据库中可托管的租户数量，都要取决于应用程序中的实际工作负载模式。


## <a name="next-steps"></a>后续步骤

本教程介绍了以下内容：

> [!div class="checklist"]

> - 如何部署 Wingtip Tickets SaaS 多租户数据库应用程序
> - 关于构成该应用的服务器和数据库
> - 租户通过目录映射到其数据
> - 如何将新租户预配到多租户数据库和单租户数据库。
> - 如何查看池使用率来监视租户活动
> - 如何删除示例资源以停止相关计费

现在，请尝试[预配和编录教程](sql-database-saas-tutorial-provision-and-catalog.md)。



## <a name="additional-resources"></a>其他资源

- 若要了解多租户 SaaS 应用程序，请参阅[多租户 SaaS 应用程序的设计模式](https://docs.microsoft.com/azure/sql-database/sql-database-design-patterns-multi-tenancy-saas-applications)








<!--  Link references.

A [series of related tutorials] is available that build upon this initial deployment.
[link-wtp-overivew-jumpto-saas-tutorials-97j]: saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials

-->

[link-aka-ms-deploywtp-mtapp-52k]: http://aka.ms/deploywtp-mtapp


[link-azure-get-started-powershell-41q]: https://docs.microsoft.com/powershell/azure/get-started-azureps

[link-github-wingtip-multitenantdb-55g]: https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDB/





<!--  Image references.

[image-deploy-to-azure-blue-48d]: http://aka.ms/deploywtp-mtapp "Button for Deploy to Azure."

-->

[image-deploy-to-azure-blue-48d]: media/saas-multitenantdb-get-started-deploy/deploy.png
