---
title: 使用 Azure 诊断跟踪云服务应用程序中的流 | Microsoft Docs
description: 将跟踪消息添加到 Azure 应用程序中，以协作进行调试、性能度量、监视、流量分析等。
services: cloud-services
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 09934772-cc07-4fd2-ba88-b224ca192f8e
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 02/20/2016
ms.author: jeconnoc
ms.openlocfilehash: 2ba97e43616386a0ff8459316bfc4d3ddfe241a0
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/13/2018
ms.locfileid: "39000889"
---
# <a name="trace-the-flow-of-a-cloud-services-application-with-azure-diagnostics"></a>使用 Azure 诊断跟踪云服务应用程序的流
跟踪是在应用程序运行时监视其执行情况的一种方式。 可以使用 [System.Diagnostics.Trace](https://msdn.microsoft.com/library/system.diagnostics.trace.aspx)、[System.Diagnostics.Debug](https://msdn.microsoft.com/library/system.diagnostics.debug.aspx) 和 [System.Diagnostics.TraceSource](https://msdn.microsoft.com/library/system.diagnostics.tracesource.aspx) 类在日志、文本文件或其他设备中记录与错误及应用程序执行情况相关的信息，供以后进行分析。 有关跟踪的详细信息，请参阅[跟踪和检测应用程序](https://msdn.microsoft.com/library/zs6s4h68.aspx)。

## <a name="use-trace-statements-and-trace-switches"></a>使用 Trace 语句和 Trace 开关
在云服务应用程序中实施跟踪时，可以将 [DiagnosticMonitorTraceListener](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.diagnostics.diagnosticmonitortracelistener.aspx) 添加到应用程序配置中，并在应用程序代码中调用 System.Diagnostics.Trace 或 System.Diagnostics.Debug。 对辅助角色使用配置文件 *app.config*，对 Web 角色使用配置文件 *web.config*。 使用 Visual Studio 模板创建新的托管服务时，系统会针对你所添加的角色将 Azure 诊断自动添加到项目中，并将 DiagnosticMonitorTraceListener 添加到相应的配置文件中。

有关如何放置 Trace 语句的信息，请参阅[如何：向应用程序代码添加 Trace 语句](https://msdn.microsoft.com/library/zd83saa2.aspx)。

可以将 [Trace 开关](https://msdn.microsoft.com/library/3at424ac.aspx)放置到代码中，从而控制是否进行跟踪以及跟踪的范围。 这样即可在生产环境中监视应用程序的状态。 这在业务应用程序中特别重要，因为业务应用程序会在多个计算机中使用多个运行的组件。 有关详细信息，请参阅[如何：配置 Trace 开关](https://msdn.microsoft.com/library/t06xyy08.aspx)。

## <a name="configure-the-trace-listener-in-an-azure-application"></a>在 Azure 应用程序中配置跟踪侦听器
Trace、Debug 和 TraceSource 都要求设置“侦听器”来收集和记录已发送的消息。 侦听器将收集、存储和路由跟踪消息。 它们会将跟踪输出传输到适当的目标，如日志、窗口或文本文件。 Azure 诊断使用 [DiagnosticMonitorTraceListener](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.diagnostics.diagnosticmonitortracelistener.aspx) 类。

完成以下过程之前，必须初始化 Azure 诊断监视器。 若要执行此操作，请参阅[在 Microsoft Azure 中启用诊断](cloud-services-dotnet-diagnostics.md)。

请注意，如果使用的模板是 Visual Studio 提供的，则会自动添加侦听器的配置。

### <a name="add-a-trace-listener"></a>添加跟踪侦听器
1. 打开针对角色的 web.config 或 app.config 文件。
2. 将以下代码添加到文件。 更改 Version 属性，以使用引用的程序集的版本号。 除非有所更新，否则程序集的版本不一定随着每个 Azure SDK 发行版发生变化。
   
    ```
    <system.diagnostics>
        <trace>
            <listeners>
                <add type="Microsoft.WindowsAzure.Diagnostics.DiagnosticMonitorTraceListener,
                  Microsoft.WindowsAzure.Diagnostics,
                  Version=2.8.0.0,
                  Culture=neutral,
                  PublicKeyToken=31bf3856ad364e35"
                  name="AzureDiagnostics">
                    <filter type="" />
                </add>
            </listeners>
        </trace>
    </system.diagnostics>
    ```
   > [!IMPORTANT]
   > 确保与 Microsoft.WindowsAzure.Diagnostics 程序集建立项目引用。 更新上述 xml 中的版本号，以便与引用的 Microsoft.WindowsAzure.Diagnostics 程序集的版本匹配。
   > 
   > 
3. 保存 config 文件。

有关侦听器的详细信息，请参阅[跟踪侦听器](https://msdn.microsoft.com/library/4y5y10s7.aspx)。

完成添加侦听器的步骤后，即可将 Trace 语句添加到代码中。

### <a name="to-add-trace-statement-to-your-code"></a>将 Trace 语句添加到代码中
1. 打开应用程序的源文件。 例如用于辅助角色或 Web 角色的 <RoleName>.cs 文件。
2. 添加以下 using 语句（如果尚未添加）：
    ```
        using System.Diagnostics;
    ```
3. 添加 Trace 语句，以便捕获有关应用程序状态的信息。 可以使用多种方法来来格式化 Trace 语句的输出。 有关详细信息，请参阅[如何：向应用程序代码添加 Trace 语句](https://msdn.microsoft.com/library/zd83saa2.aspx)。
4. 保存源文件。

