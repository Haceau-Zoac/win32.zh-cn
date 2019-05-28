---
title: 执行和同步命令列表
description: 在 Microsoft Direct3D 12 中，早期版本的即时模式不再存在。
ms.assetid: D5013102-2302-4D66-8F59-079C03BA65FD
keywords:
- 命令列表
- 同步
- 执行
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: f714a8d7ad00be8b1aced88618df90993766769a
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224368"
---
# <a name="executing-and-synchronizing-command-lists"></a>执行和同步命令列表

在 Microsoft Direct3D 12 中，早期版本的即时模式不再存在。 相反，应用程序创建命令列表和捆绑包，然后记录 GPU 命令集。 命令队列用于提交要执行的命令列表。 此模型允许开发人员能够有效地使用这两个图形处理单元 (GPU) 的更好地控制和 CPU。

-   [命令队列概述](#command-queue-overview)
-   [初始化命令队列](#initializing-a-command-queue)
-   [正在执行的命令列表](#executing-command-lists)
-   [从多个命令队列访问资源](#accessing-resources-from-multiple-command-queues)
-   [使用命令队列界定的同步命令列表执行](#synchronizing-command-list-execution-using-command-queue-fences)
-   [同步访问的命令队列资源](#synchronizing-resources-accessed-by-command-queues)
-   [平铺资源的命令队列支持](#command-queue-support-for-tiled-resources)
-   [相关的主题](#related-topics)

## <a name="command-queue-overview"></a>命令队列概述

Direct3D 12 命令队列将为运行时和驱动程序同步的即时模式工作提交，以前为开发人员使用 Api 进行显式管理并发性、 并行度和同步公开 not。 命令队列提供了开发人员的以下改进：

-   允许开发人员避免意外不足引起的意外的同步。
-   允许开发人员引入更有效地准确地确定所需的同步可以较高级别上的同步。 这意味着在运行时和图形驱动程序将花费更少时间被动工程并行度。
-   使成本高昂的操作更为明确。

这些改进实现或增强以下方案：

-   提高的并行度的应用程序可以使用后台工作负荷，如视频解码，它们具有单独的队列，为前台工作时更深层次的队列。
-   异步和低优先级的 GPU 工作的命令队列模型可并发执行的低优先级的 GPU 工作并启用一个 GPU 线程，而不会阻止使用另一个未同步线程的结果的原子操作。
-   高优先级计算工作-此设计使需要中断以执行少量的高优先级计算工作，以便可以进行其他处理在 CPU 上提前获取结果的 3D 渲染方案。

## <a name="initializing-a-command-queue"></a>初始化命令队列

可以通过调用创建命令队列[ **ID3D12Device::CreateCommandQueue**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandqueue)。 此方法采用[ **D3D12\_命令\_列表\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type) ，该值指示应创建队列的类型，并因此，可在哪种类型的命令提交到该队列。 请记住，捆绑包仅可从直接的命令列表调用，并且不能直接提交到队列。 受支持的队列类型包括：

-   [**D3D12\_命令\_列表\_类型\_直接**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)
-   [**D3D12\_命令\_列表\_类型\_计算**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)
-   [**D3D12\_命令\_列表\_类型\_复制**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)

一般情况下，直接队列和命令列表接受任何命令、 计算队列和列表接受计算和复制的命令关联的命令，并复制队列和命令列表接受仅复制命令。

## <a name="executing-command-lists"></a>正在执行的命令列表

在记录后命令列表，并检索默认命令队列或创建一个新的在执行命令列表通过调用[ **ID3D12CommandQueue::ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)。

应用程序可从多个线程可以提交到任何命令队列命令列表。 在运行时将执行序列化顺序提交这些请求的工作。

运行时将验证提交的命令列表，并将删除对[ **ExecuteCommandLists** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)如果违反了任何限制。 调用将删除原因如下：

-   提供的命令列表是一个包，而不是直接的命令列表。
-   [**ID3D12GraphicsCommandList::Close** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)尚未调用到完成录制在提供的命令列表。
-   [**ID3D12CommandAllocator::Reset** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandallocator-reset)已调用了与命令列表，因为它已记录的命令分配器。 有关命令分配器的详细信息，请参阅[创建和记录命令列表和捆绑包](recording-command-lists-and-bundles.md)。
-   命令队列 fence 指示发出的命令列表的上一次执行尚未完成。 下面将详细讨论了命令队列界定。
-   通过调用设置之前和之后的查询的状态， [ **ID3D12GraphicsCommandList::BeginQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery)并[ **ID3D12GraphicsCommandList::EndQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery)，未正确匹配。
-   之前和之后的资源的状态转换障碍不正确匹配。 有关详细信息，请参阅[使用资源屏障来同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。

## <a name="accessing-resources-from-multiple-command-queues"></a>从多个命令队列访问资源

有几个施加的运行时的规则，以从多个命令队列限制对资源的访问。 这些规则如下所示：

<dl> 1. 资源不能是从多个命令队列同时写入。 当资源已转换为可写入状态队列时，被视为由该队列，以独占方式拥有，必须转换到读取或常见的状态 (请参阅<a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states"> **D3D12\_资源\_状态** </a>) 另一个队列可以访问它之前。 2. 在读取状态下，资源可以同时从读取多个命令队列，包括跨进程，基于其读取状态。  
</dl>

有关资源的访问限制和使用资源屏障来同步资源的访问权限的详细信息，请参阅[使用资源屏障来同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。

## <a name="synchronizing-command-list-execution-using-command-queue-fences"></a>使用命令队列界定的同步命令列表执行

在 Direct3D 12 中的多个并行命令队列的支持提供更多灵活性和控制在 GPU 上的异步工作的优先顺序。 这种设计还意味着应用程序需要显式管理同步的工作，尤其是在一个队列中的命令列表依赖于由另一个命令队列进行操作的资源。 其中的一些示例包括等待在计算队列来完成，以便结果可用于在三维的队列中，一个呈现操作的操作，并等待 3D 操作来完成，以便计算队列上的操作可以访问有关编写资源运算结果。 若要启用工作队列之间的同步，Direct3D 12 中使用的界定，表示在通过 API 概念[ **ID3D12Fence** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12fence)接口。

Fence 为一个整数，表示当前正在处理的工作单元。 应用程序何时前移的防护，通过调用[ **ID3D12CommandQueue::Signal**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandqueue-signal)，更新的整数。 应用可以检查起一道围墙的值，并确定是否以确定是否可以启动后续的操作已完成的工作单元。

## <a name="synchronizing-resources-accessed-by-command-queues"></a>同步访问的命令队列资源

在 Direct3D 12 中，同步某些资源的状态来实现与资源的障碍。 在每个资源屏障，应用程序声明之前和之后的某个资源的状态。 常见示例适用于呈现目标视图的着色器资源视图之间进行过渡的资源。 大多数情况下，命令列表内部管理这些资源的障碍。 如果启用了调试层，系统强制执行的之前和之后的所有资源的状态相匹配，保证的资源处于屏障转换在某一特定操作的正确状态。

有关同步资源状态的详细信息，请参阅[使用资源屏障来同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。

## <a name="command-queue-support-for-tiled-resources"></a>平铺资源的命令队列支持

用于管理方法平铺资源，这通过公开[ **ID3D11DeviceContext2** ](https://msdn.microsoft.com/library/windows/desktop/dn280498)接口在 Direct3D 11 中，通过提供[ **ID3D12CommandQueue** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue) Direct3D 12 中的接口。 这些方法包括：



| 方法                                                              | 描述                                                                                              |
|---------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| [**CopyTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-copytilemappings)     | 来自源的副本映射平铺到平铺的目标资源的资源。<br/>                 |
| [**UpdateTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings) | 磁贴中的位置平铺资源映射更新到资源堆中的内存位置。<br/> |



 

有关在 Direct3D 12 应用中使用平铺的资源的详细信息，请参阅[Direct3D11 平铺资源](https://msdn.microsoft.com/library/windows/desktop/dn786477)。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[提交工作在 Direct3D 12](command-queues-and-command-lists.md)
</dt> </dl>

 

 





