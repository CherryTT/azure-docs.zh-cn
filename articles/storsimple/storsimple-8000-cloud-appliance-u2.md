---
title: "StorSimple 云设备 Update 3 | Microsoft Docs"
description: "了解如何在 Microsoft Azure 虚拟网络中创建、部署和管理 StorSimple 云设备。 （适用于 StorSimple Update 3 及更高版本）。"
services: storsimple
documentationcenter: 
author: alkohli
manager: timlt
editor: 
ms.assetid: 
ms.service: storsimple
ms.devlang: NA
ms.topic: hero-article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/10/2017
ms.author: alkohli
ms.translationtype: HT
ms.sourcegitcommit: 2ad539c85e01bc132a8171490a27fd807c8823a4
ms.openlocfilehash: 905a70bfd37ccdb9f2944b4a9348c3b60dedda44
ms.contentlocale: zh-cn
ms.lasthandoff: 07/12/2017

---
# 在 Azure 中部署和管理 StorSimple 云设备（Update 3 及更高版本）
<a id="deploy-and-manage-a-storsimple-cloud-appliance-in-azure-update-3-and-later" class="xliff"></a>

## 概述
<a id="overview" class="xliff"></a>

StorSimple 8000 系列云设备是 Microsoft Azure StorSimple 解决方案附带的一项额外功能。 StorSimple 云设备在 Microsoft Azure 虚拟网络中的虚拟机上运行，可用于备份和克隆来自主机的数据。

本文介绍了在 Azure 中部署和管理 StorSimple 云设备的分步过程。 阅读本文后，将会：

* 了解云设备与物理设备之间的差异。
* 能够创建和配置云设备。
* 连接到云设备。
* 了解如何使用云设备。

本教程适用于运行 Update 3 及更高版本的所有 StorSimple 云设备。

#### 云设备型号比较
<a id="cloud-appliance-model-comparison" class="xliff"></a>

StorSimple 云设备以两种型号提供：标准 8010（前身为 1100）和高级 8020（在 Update 2 引入的）。 下表提供了这两种型号的对比。

| 设备型号 | 8010<sup>1</sup> | 8020 |
| --- | --- | --- |
| **最大容量** |30 TB |64 TB |
| **Azure VM** |Standard_A3（4 核，7 GB 内存）| Standard_DS3（4 核，14 GB 内存）|
| **上市区域** |所有 Azure 区域 |支持高级存储和 DS3 Azure VM 的 Azure 区域<br></br>请使用[此列表](https://azure.microsoft.com/regions/services/)，看“虚拟机”>“DS 系列”和“存储”>“磁盘存储”在你的区域是否均可用。 |
| **存储类型** |为本地磁盘使用 Azure 标准存储<br></br> 了解如何 [创建标准存储帐户](../storage/storage-create-storage-account.md) |为本地磁盘使用 Azure 高级存储<sup>2</sup> <br></br>了解如何[创建高级存储帐户](../storage/storage-premium-storage.md) |
| **工作负荷指导** |在级别从备份中检索文件 |云开发和测试方案 <br></br>低延迟和更高性能工作负载<br></br>用于灾难恢复的辅助设备 |

<sup>1</sup>*前身为 1100*。

<sup>2</sup> *8010 和 8020 为云层使用 Azure 标准存储。只有设备中的本地层存在差异*。

## 云设备与物理设备有何差异
<a id="how-the-cloud-appliance-differs-from-the-physical-device" class="xliff"></a>

StorSimple 云设备是软件形式的 StorSimple，在 Microsoft Azure 虚拟机中的单个节点上运行。 云设备支持物理设备不可用的灾难恢复方案。 云设备适用于从备份进行的项目级检索、本地灾难恢复以及云开发和测试方案。

#### 与物理设备的差异
<a id="differences-from-the-physical-device" class="xliff"></a>

下表显示了 StorSimple 云设备与 StorSimple 物理设备之间的一些主要差异。

|  | 物理设备 | 云设备 |
| --- | --- | --- |
| **位置** |驻留在数据中心。 |在 Azure 中运行。 |
| **网络接口** |有六个网络接口：DATA 0 到 DATA 5。 |只有一个网络接口：DATA 0。 |
| **注册** |在初始配置步骤中注册的。 |注册是一个单独的任务。 |
| **服务数据加密密钥** |在物理设备上重新生成，然后使用新密钥更新云设备。 |无法从云设备重新生成。 |
| **支持的卷类型** |支持本地固定卷和分层卷。 |仅支持分层卷。 |

## 云设备的先决条件
<a id="prerequisites-for-the-cloud-appliance" class="xliff"></a>

以下各部分介绍了 StorSimple 云设备的配置先决条件。 在部署云设备之前，查看有关使用云设备的安全注意事项。

[!INCLUDE [StorSimple Cloud Appliance security](../../includes/storsimple-8000-cloud-appliance-security.md)]

#### Azure 要求
<a id="azure-requirements" class="xliff"></a>

在预配云设备之前，需要在 Azure 环境中做好以下准备：

* 确保在数据中心部署并运行 StorSimple 8000 系列物理设备（型号为 8100 或 8600）。 将此设备注册到要为其创建 StorSimple 云设备的 StorSimple 设备管理器服务。
* 对于云设备，[在 Azure 中配置虚拟网络](../virtual-network/virtual-networks-create-vnet-arm-pportal.md)。 如果使用高级存储，必须在支持高级存储的 Azure 区域中创建虚拟网络。 高级存储区域是 [Azure 服务（按区域）](https://azure.microsoft.com/regions/services/)列表中与“磁盘存储”所在的行对应的区域。
* 建议使用 Azure 提供的默认 DNS 服务器，而不要指定自己的 DNS 服务器名称。 如果 DNS 服务器名称无效，或者 DNS 服务器无法正确解析 IP 地址，则创建云设备将会失败。
* 点到站点和站点到站点连接是可选的，而不是必需的。 如果需要，可以针对更高级方案配置这些选项。
* 可以在可使用云设备公开的卷的虚拟网络中创建 [Azure 虚拟机](../virtual-machines/virtual-machines-windows-quick-create-portal.md)（主机服务器）。 这些服务器必须满足以下要求：

  * 是装有 iSCSI 发起程序软件的 Windows 或 Linux VM。
  * 在云设备所在的同一个虚拟网络中运行。
  * 能够通过云设备的内部 IP 地址连接到云设备的 iSCSI 目标。
  * 确保在同一个虚拟网络中配置 iSCSI 和云流量的支持。

#### StorSimple 要求
<a id="storsimple-requirements" class="xliff"></a>

创建云设备之前，请对 StorSimple Device Manager 服务进行以下更新：

* 针对要用作云设备主机服务器的 VM 添加[访问控制记录](storsimple-8000-manage-acrs.md)。
* 使用与云设备位于同一区域中的[存储帐户](storsimple-8000-manage-storage-accounts.md#add-a-storage-account)。 使用不同区域中的存储帐户可能导致性能不佳。 可以将标准或高级存储帐户用于云设备。 如何创建[标准存储帐户](../storage/storage-create-storage-account.md)或[高级存储帐户](../storage/storage-premium-storage.md)的详细信息
* 用于创建云设备的存储帐户应该与用于存储数据的存储帐户不同。 使用相同的存储帐户可能会导致性能不佳。

在开始之前，请确保已准备好以下信息：

* Azure 门户帐户和访问凭据。
* 来自注册到 StorSimple Device Manager 服务的物理设备的服务数据加密密钥的副本。

## 创建并配置云设备
<a id="create-and-configure-the-cloud-appliance" class="xliff"></a>

在执行这些过程之前，请确保已满足[云设备先决条件](#prerequisites-for-the-cloud-appliance)。

执行以下步骤来创建 StorSimple 云设备。

### 步骤 1：创建云设备
<a id="step-1-create-a-cloud-appliance" class="xliff"></a>

执行以下步骤来创建 StorSimple 云设备。

[!INCLUDE [Create a cloud appliance](../../includes/storsimple-8000-create-cloud-appliance-u2.md)]

如果在此步骤中创建云设备失败，可能是因为没有连接到 Internet。 有关详细信息，请转到[排查创建云设备时遇到的 Internet 连接问题](#troubleshoot-internet-connectivity-errors)。

### 步骤 2：配置并注册云设备
<a id="step-2-configure-and-register-the-cloud-appliance" class="xliff"></a>

开始此过程之前，请确保已获得服务数据加密密钥的副本。 服务数据加密密钥是在向 StorSimple Device Manager 服务注册第一台 StorSimple 物理设备时创建的。 创建时会指示你将其保存在一个安全位置。 如果没有服务数据加密密钥的副本，则必须联系 Microsoft 支持部门获得帮助。

执行以下步骤来配置并注册 StorSimple 云设备。

[!INCLUDE [Configure and register a cloud appliance](../../includes/storsimple-8000-configure-register-cloud-appliance.md)]

### 步骤 3：（可选）修改设备配置设置
<a id="step-3-optional-modify-the-device-configuration-settings" class="xliff"></a>

以下部分介绍了在使用 CHAP、StorSimple Snapshot Manager 或更改设备管理员密码时，需要为 StorSimple 云设备完成的设备配置设置。

#### 配置 CHAP 发起程序
<a id="configure-the-chap-initiator" class="xliff"></a>

此参数包含云设备（目标）应该从尝试访问卷的发起端（服务器）获取的凭据。 在此身份验证过程中，发起端提供 CHAP 用户名和 CHAP 密码，在设备上验证自身的身份。 有关详细步骤，请转到 [配置设备的 CHAP](storsimple-8000-configure-chap.md#unidirectional-or-one-way-authentication)。

#### 配置 CHAP 目标
<a id="configure-the-chap-target" class="xliff"></a>

此参数包含具有 CHAP 功能的发起端请求相互或双向身份验证时，云设备使用的凭据。 在此身份验证过程中，云设备使用反向 CHAP 用户名和反向 CHAP 密码，在发起端验证自身的身份。

> [!NOTE]
> CHAP 目标设置是全局设置。 应用这些设置后，连接到云设备的所有卷都将使用 CHAP 身份验证。

有关详细步骤，请转到 [配置设备的 CHAP](storsimple-8000-configure-chap.md#bidirectional-or-mutual-authentication)。

#### 配置 StorSimple Snapshot Manager 密码
<a id="configure-the-storsimple-snapshot-manager-password" class="xliff"></a>

StorSimple Snapshot Manager 软件驻留在 Windows 主机上，可让管理员以本地和云快照的形式管理 StorSimple 设备的备份。

> [!NOTE]
> 对云设备而言，Windows 主机就是 Azure 虚拟机。

在 StorSimple Snapshot Manager 中配置设备时，系统将提示你提供 StorSimple 设备 IP 地址和密码对存储设备进行身份验证。 有关详细步骤，请转到 [配置 StorSimple Snapshot Manager 密码](storsimple-8000-change-passwords.md#set-the-storsimple-snapshot-manager-password)。

#### 更改设备管理员密码
<a id="change-the-device-administrator-password" class="xliff"></a>

使用 Windows PowerShell 接口访问云设备时，需要输入设备管理员密码。 为了确保数据的安全，必须更改此密码才能使用云设备。 有关详细步骤，请转到 [配置设备管理员密码](../storsimple/storsimple-8000-change-passwords.md#change-the-device-administrator-password)。

## 远程连接到云设备
<a id="connect-remotely-to-the-cloud-appliance" class="xliff"></a>

默认情况下，不支持通过 Windows PowerShell 接口远程访问云设备。 必须首先在云设备上启用远程管理，然后在用来访问云设备的客户端上启用远程管理。

下面的两步过程介绍了如何远程连接到云设备。

### 步骤 1：配置远程管理
<a id="step-1-configure-remote-management" class="xliff"></a>

执行以下步骤来配置 StorSimple 云设备的远程管理。

[!INCLUDE [Configure remote management via HTTP for cloud appliance](../../includes/storsimple-8000-configure-remote-management-http-device.md)]

### 步骤 2：远程访问云设备
<a id="step-2-remotely-access-the-cloud-appliance" class="xliff"></a>

在云设备上启用远程管理后，使用 Windows PowerShell 远程处理从同一虚拟网络中的另一虚拟机上连接到设备。 例如，可以从你配置的用来连接 iSCSI 的宿主 VM 进行连接。 在大多数部署中，你将打开一个公共终结点来访问可用于访问云设备的宿主 VM。

> [!WARNING]
> **为了增强安全性，强烈建议在连接到终结点时使用 HTTPS，并在完成 PowerShell 远程会话之后删除终结点。**

必须遵循[远程连接到 StorSimple 设备](storsimple-8000-remote-connect.md)中的过程来设置云设备的远程功能。

## 直接连接到云设备
<a id="connect-directly-to-the-cloud-appliance" class="xliff"></a>

还可以直接连接到云设备。 若要从虚拟网络外部或 Microsoft Azure 环境外部的另一台计算机直接连接到云设备，必须创建其他终结点。

执行以下步骤，在云设备上创建公共终结点。

[!INCLUDE [Create public endpoints on a cloud appliance](../../includes/storsimple-8000-create-public-endpoints-cloud-appliance.md)]

建议从同一虚拟网络中的另一个虚拟机进行连接，因为这种做法可将虚拟网络上的公共终结点数目减到最少。 在此情况下，请通过远程桌面会话连接到虚拟机，然后就像在局域网中配置任何其他 Windows 客户端一样配置该虚拟网络。 无需附加公共端口号，因为该端口是已知的。

## 使用 StorSimple 云设备
<a id="work-with-the-storsimple-cloud-appliance" class="xliff"></a>

现在，你已创建并配置了 StorSimple 云设备，可以开始使用它执行工作了。 可以像在物理 StorSimple 设备上一样使用卷容器、卷和备份策略。 唯一的区别是你需要确保从设备列表中选择云设备。 有关云设备的各种管理任务的分步过程，请参阅[使用 StorSimple Device Manager 服务管理云设备](storsimple-8000-manager-service-administration.md)。

以下各部分介绍了在使用云设备时会遇到的一些差异。

### 维护 StorSimple 云设备
<a id="maintain-a-storsimple-cloud-appliance" class="xliff"></a>

由于是纯软件设备，相比物理设备的维护，云设备的维护可以说非常简单。

无法更新云设备。 请使用最新版本的软件创建新的云设备。


### 云设备的存储帐户
<a id="storage-accounts-for-a-cloud-appliance" class="xliff"></a>

创建的存储帐户供 StorSimple Device Manager 服务、云设备和物理设备使用。 创建存储帐户时，建议你在友好名称中使用区域标识符。 这有助于确保区域在所有系统组件中保持一致。 对于云设备，所有组件必须位于同一区域中，防止出现性能问题。

有关分步过程，请转到 [添加存储帐户](storsimple-8000-manage-storage-accounts.md#add-a-storage-account)。

### 停用 StorSimple 云设备
<a id="deactivate-a-storsimple-cloud-appliance" class="xliff"></a>

停用某个云设备会删除在预配该设备时创建的 VM 和资源。 停用云设备之后，无法将它还原到以前的状态。 停用云设备之前，请务必停止或删除它所依赖的客户端和主机。

停用云设备会导致以下操作：

* 删除云设备。
* 删除为该云设备创建的 OS 磁盘和数据磁盘。
* 保留预配期间创建的托管服务和虚拟网络。 如果不使用该服务和网络，应该手动将其删除。
* 为云设备创建的云快照会保留。

有关分步过程，请转到 [停用和删除 StorSimple 设备](storsimple-8000-deactivate-and-delete-device.md)。

云设备在 StorSimple 设备管理器服务边栏选项卡上显示为已停用后，可以从“设备”边栏选项卡上的设备列表中删除该云设备。

### 停止、启动和重新启动云设备
<a id="start-stop-and-restart-a-cloud-appliance" class="xliff"></a>
不同于 StorSimple 物理设备，StorSimple 云设备上没有开机或关机按钮。 但是，有时可能需要停止和重新启动云设备。

若要启动、停止和重启云设备，最简单的方法是使用“虚拟机服务”边栏选项卡。 转到“虚拟机服务”。 从 VM 列表中，找出与你的云设备对应的 VM（名称相同），然后单击该 VM 名称。 在查看虚拟机边栏选项卡时，可以看到云设备状态为“正在运行”，因为它在创建之后已默认启动。 可以随时启动、停止和重新启动虚拟机。

[!INCLUDE [Stop and restart cloud appliance](../../includes/storsimple-8000-stop-restart-cloud-appliance.md)]

### 重置为出厂默认值
<a id="reset-to-factory-defaults" class="xliff"></a>
如果决定要创建全新的云设备，只需将它停用并删除，然后新建一个。

## 故障转移到云设备
<a id="fail-over-to-the-cloud-appliance" class="xliff"></a>
灾难恢复 (DR) 是 StorSimple 云设备希望解决的重要情形之一。 出现这种情况时，物理 StorSimple 设备甚至整个数据中心可能不可用。 幸运的是，可以在备用位置使用云设备来还原操作。 在 DR 期间，源设备中的卷容器将更改所有权并传输到云设备。

DR 的先决条件如下：

* 创建并配置云设备。
* 卷容器中的所有卷都处于脱机状态。
* 要故障转移的卷容器具有关联的云快照。

> [!NOTE]
> * 使用云设备作为 DR 的辅助设备时，请记住，8010 有 30 TB 的标准存储，8020 有 64 TB 的高级存储。 较高容量的 8020 云设备可能比较适合 DR 方案。

有关分步过程，请转到[故障转移到云设备](storsimple-8000-device-failover-cloud-appliance.md)。

## 删除云设备
<a id="delete-the-cloud-appliance" class="xliff"></a>
如果以前配置并使用了 StorSimple 云设备，但现在希望停止由于使用该设备而产生计算费用，则必须停止云设备。 停止云设备会解除分配 VM。 此操作将停止你的订阅上产生的费用。 不过，OS 和数据磁盘的存储费用仍然会继续产生。

若要停止所有费用，必须删除云设备。 若要删除云设备创建的备份，可以停用或删除该设备。 有关详细信息，请参阅 [停用和删除 StorSimple 设备](storsimple-8000-deactivate-and-delete-device.md)。

[!INCLUDE [Delete a cloud appliance](../../includes/storsimple-8000-delete-cloud-appliance.md)]

## 对 Internet 连接错误进行故障排除
<a id="troubleshoot-internet-connectivity-errors" class="xliff"></a>
创建云设备时，如果没有 Internet 连接，则创建步骤将会失败。 若要排查 Internet 连接故障，请在 Azure 门户中执行以下步骤：

1. [在 Azure 中创建 Windows server 2012 虚拟机](/articles/virtual-machines/windows/quick-create-portal.md)。 此虚拟机应使用与云设备相同的存储帐户、VNet 和子网。 如果 Azure 中存在使用相同存储帐户、VNet 和子网的现有 Windows Server 主机，也可以使用它来排除 Internet 连接故障。
2. 远程登录到上一步中创建的虚拟机。
3. 打开虚拟机内的命令窗口（Win + R，然后键入 `cmd`）。
4. 在提示符处运行以下 cmd。

    `nslookup windows.net`
5. 如果 `nslookup` 失败，则 Internet 连接故障会阻止云设备注册到 StorSimple Device Manager 服务。
6. 对虚拟网络进行必要的更改，以确保云设备能够访问 Azure 站点（例如 _windows.net_）。

## 后续步骤
<a id="next-steps" class="xliff"></a>
* 了解如何[使用 StorSimple Device Manager 服务管理云设备](storsimple-8000-manager-service-administration.md)。
* 了解如何 [从备份集还原 StorSimple 卷](storsimple-8000-restore-from-backup-set-u2.md)。
