---
title: 使用 Azure Log Analytics 跨资源进行搜索 | Microsoft Docs
description: 本文介绍了如何在订阅中跨多个工作区以及从特定的 App Insights 应用查询资源。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/17/2018
ms.author: magoedte
ms.component: ''
ms.openlocfilehash: e06b9ff2134c0bd1fb1ee8515827e9e8c06a3108
ms.sourcegitcommit: 00dd50f9528ff6a049a3c5f4abb2f691bf0b355a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/05/2018
ms.locfileid: "51008464"
---
# <a name="perform-cross-resource-log-searches-in-log-analytics"></a>在 Log Analytics 中执行跨资源日志搜索  

之前通过 Azure Log Analytics，只可以分析来自当前工作区内的数据，这限制了跨订阅中定义的多个工作区查询数据的能力。  此外，之前只能直接在 Application Insights 中或从 Visual Studio 中使用 Application Insights 搜索通过基于 web 的应用程序收集的遥测数据项。  这还使得难以采用本机方式将操作数据和应用程序数据一起分析。   

现在不但可以跨多个 Log Analytics 工作区进行查询，而且还可以查询同一资源组、另一资源组或另一订阅中特定 Application Insights 应用的数据。 这可以提供数据的系统级视图。  你只能在 [Log Analytics](log-analytics-log-search-portals.md#log-analytics-page) 中执行这些类型的查询。 可以在单个查询中包含的资源（Log Analytics 工作区和 Application Insights 应用）数量限制为 100。 

## <a name="querying-across-log-analytics-workspaces-and-from-application-insights"></a>跨 Log Analytics 工作区以及从 Application Insights 进行查询
若要在查询中引用另一个工作区，请使用 [*workspace*](https://docs.microsoft.com/azure/log-analytics/query-language/workspace-expression) 标识符，对于 Application Insights 中的应用，请使用 [*app*](https://docs.microsoft.com/azure/log-analytics/query-language/app-expression) 标识符。  

### <a name="identifying-workspace-resources"></a>标识工作区资源
以下示例演示了跨 Log Analytics 工作区进行查询从名为 *contosoretail-it* 的工作区的 Update 表中返回记录的汇总计数。 

可以通过以下任一方式来标识工作区：

* 资源名称 - 用户可读的工作区名称，有时称为“组件名称”。 

    `workspace("contosoretail-it").Update | count`
 
    >[!NOTE]
    >按名称标识工作区会假设它在所有可访问订阅中是唯一的。 如果具有多个采用指定名称的应用程序，查询将因多义性而失败。 在这种情况下，必须使用其他标识符之一。

* 限定名称 - 工作区的“全名”，由订阅名称、资源组和组件名称组成，并采用以下格式：*subscriptionName/resourceGroup/componentName*。 

    `workspace('contoso/contosoretail/contosoretail-it').Update | count `

    >[!NOTE]
    >因为 Azure 订阅名称不唯一，所以此标识符可能不明确。 
    >

* 工作区 ID - 工作区 ID 是分配给每个工作区的唯一不可变标识符，表示为全局唯一标识符 (GUID)。

    `workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update | count`

* Azure 资源 ID - Azure 定义的工作区唯一标识。 当资源名称不明确时需使用资源 ID。  对于工作区，此格式为：/subscriptions/subscriptionId/resourcegroups/resourceGroup/providers/microsoft.OperationalInsights/workspaces/componentName。  

    例如：
    ``` 
    workspace("/subscriptions/e427519-5645-8x4e-1v67-3b84b59a1985/resourcegroups/ContosoAzureHQ/providers/Microsoft.OperationalInsights/workspaces/contosoretail-it").Update | count
    ```

### <a name="identifying-an-application"></a>标识应用程序
以下示例返回针对 Application Insights 中名为 *fabrikamapp* 的应用发出的请求的汇总计数。 

可以使用 *app(Identifier)* 表达式标识 Application Insights 中的应用程序。  *Identifier* 参数使用下列项之一来指定应用：

* 资源名称 - 人类可读的应用名称，有时称为“组件名称”。  

    `app("fabrikamapp")`

* 限定名称 - 应用的“全名”，由订阅名称、资源组和组件名称组成，并采用以下格式：*subscriptionName/resourceGroup/componentName*。 

    `app("AI-Prototype/Fabrikam/fabrikamapp").requests | count`

     >[!NOTE]
    >因为 Azure 订阅名称不唯一，所以此标识符可能不明确。 
    >

* ID - 应用程序的应用 GUID。

    `app("b459b4f6-912x-46d5-9cb1-b43069212ab4").requests | count`

* Azure 资源 ID - Azure 定义的应用唯一标识。 当资源名称不明确时需使用资源 ID。 格式为：*/subscriptions/subscriptionId/resourcegroups/resourceGroup/providers/microsoft.OperationalInsights/components/componentName*。  

    例如：
    ```
    app("/subscriptions/b459b4f6-912x-46d5-9cb1-b43069212ab4/resourcegroups/Fabrikam/providers/microsoft.insights/components/fabrikamapp").requests | count
    ```

### <a name="performing-a-query-across-multiple-resources"></a>跨多个资源执行查询
可以从任何资源实例查询多个资源，这些资源实例可以是工作区和应用程序的组合。
    
跨两个工作区进行查询的示例：    

```
union Update, workspace("contosoretail-it").Update, workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update
| where TimeGenerated >= ago(1h)
| where UpdateState == "Needed"
| summarize dcount(Computer) by Classification
```

## <a name="next-steps"></a>后续步骤

请参阅 [Log Analytics 日志搜索引用](https://docs.microsoft.com/azure/log-analytics/query-language/kusto)，了解 Log Analytics 中所有可用的查询语法选项。    
