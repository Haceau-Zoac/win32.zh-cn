---
title: 执行和同步命令列表
description: 在 Microsoft Direct3D 12 中，以前版本的即时模式不再存在。
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

在 Microsoft Direct3D 12 中，以前版本的即时模式不再存在。 应用将创建命令列表和捆绑，然后记录 GPU 命令集。 命令队列用于提交要执行的命令列表。 此模型可让开发人员更好地控制图形处理单元 (GPU) 和 CPU 的有效使用。

-   [命令队列概述](#command-queue-overview)
-   [初始化命令队列](#initializing-a-command-queue)
-   [执行命令列表](#executing-command-lists)
-   [从多个命令队列访问资源](#accessing-resources-from-multiple-command-queues)
-   [使用命令队列围栏同步命令列表的执行](#synchronizing-command-list-execution-using-command-queue-fences)
-   [同步命令队列访问的资源](#synchronizing-resources-accessed-by-command-queues)
-   [图块式资源的命令队列支持](#command-queue-support-for-tiled-resources)
-   [相关主题](#related-topics)

## <a name="command-queue-overview"></a>命令队列概述

Direct3D 12 命令队列将以前不会向开发人员公开的即时模式工作提交内容的运行时和驱动程序同步，替换为可显式管理并发性、并行度和同步的 API。 命令队列为开发人员提供以下方面的改进：

-   可让开发人员避免意外的同步导致效率意外下降的问题。
-   可让开发人员引入更高级别的同步，以便更有效、更准确地确定所需的同步。 这意味着，运行时和图形驱动程序可以减少被动式工程并行度的时间。
-   使高开销的操作更为明确。

这些改进实现或增强了以下方案：

-   提高并行度 - 如果应用程序为前台工作提供独立的队列，则可以使用更深层的队列来完成后台工作负荷（例如视频解码）。
-   异步和低优先级 GPU 工作 - 命令队列模型可以实现低优先级 GPU 工作和原子操作的并发执行，这些操作支持在不阻塞的情况下，通过一个 GPU 线程来使用另一个未同步线程的结果。
-   高优先级计算工作 - 使用此设计可实现以下方案：中断 3D 渲染来执行少量的高优先级计算工作，可以提前获取结果，以便在 CPU 上进行其他处理。

## <a name="initializing-a-command-queue"></a>初始化命令队列

可以调用 [**ID3D12Device::CreateCommandQueue**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandqueue) 来创建命令队列。 此方法采用 [**D3D12\_COMMAND\_LIST\_TYPE**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)，指示要创建哪种类型的队列，因此也会指示可将哪种类型的命令提交到该队列。 请记住，只能从直接命令列表调用捆绑，而不能直接将捆绑提交到队列。 支持的队列类型为：

-   [**D3D12\_COMMAND\_LIST\_TYPE\_DIRECT**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)
-   [**D3D12\_COMMAND\_LIST\_TYPE\_COMPUTE**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)
-   [**D3D12\_COMMAND\_LIST\_TYPE\_COPY**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)

一般情况下，DIRECT 队列和命令列表接受任何命令，COMPUTE 队列和命令列表接受计算和复制相关的命令，COPY 队列和命令列表仅接受复制命令。

## <a name="executing-command-lists"></a>执行命令列表

记录命令列表并检索默认命令队列或创建新的命令队列后，通过调用 [**ID3D12CommandQueue::ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists) 来执行命令列表。

应用程序可从多个线程将命令列表提交到任何命令队列。 运行时将按提交顺序执行序列化这些请求的工作。

运行时将验证提交的命令列表，如果违反了任何限制，它会丢弃对 [**ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists) 的调用。 丢弃调用的原因如下：

-   提供的命令列表是捆绑，而不是直接命令列表。
-   尚未对提供的命令列表调用 [**ID3D12GraphicsCommandList::Close**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)，因此无法完成记录。
-   尚未对与命令列表关联的命令分配器调用 [**ID3D12CommandAllocator::Reset**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandallocator-reset)（因为已记录该命令列表）。 有关命令分配器的详细信息，请参阅[创建和记录命令列表与捆绑](recording-command-lists-and-bundles.md)。
-   命令队列围栏指示尚未完成命令列表的上一次执行。 下面将详细介绍命令队列围栏。
-   通过调用 [**ID3D12GraphicsCommandList::BeginQuery**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery) 和 [**ID3D12GraphicsCommandList::EndQuery**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery) 设置的查询之前和之后状态不适当匹配。
-   资源转换屏障的之前和之后状态不适当匹配。 有关详细信息，请参阅[使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。

## <a name="accessing-resources-from-multiple-command-queues"></a>从多个命令队列访问资源

运行时会施加几条规则来限制从多个命令队列对资源进行访问。 这些规则如下：

<dl> 1. 不能同时从多个命令队列写入一个资源。 当某个资源在队列中转换为可写状态后，该资源被视为由该队列独占拥有，在可由另一个队列访问之前，它必须转换到读取或 COMMON 状态（请参阅 <a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states">**D3D12\_RESOURCE\_STATES**</a>）。 2. 进入读取状态下，可以根据资源的读取状态，同时从多个命令队列（包括跨进程）读取该资源。  
</dl>

有关资源访问限制和使用资源屏障来同步资源访问的详细信息，请参阅[使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。

## <a name="synchronizing-command-list-execution-using-command-queue-fences"></a>使用命令队列围栏同步命令列表的执行

Direct3D 12 中对多个并行命令队列的支持提供更高的灵活性，可让你控制 GPU 中异步工作的优先顺序。 这种设计也意味着应用需要显式管理工作同步，尤其是当一个队列中的命令列表依赖于另一个命令队列操作的资源时。 部分示例包括等待计算队列中的操作完成，使其结果可用于 3D 队列中的渲染操作；等待 3D 操作完成，使计算队列中的某个操作可以访问用于写入操作的资源。 为了实现队列之间的工作同步，Direct3D 12 使用了 [**ID3D12Fence**](/windows/desktop/api/D3D12/nn-d3d12-id3d12fence) 接口的 API 表示的围栏的概念。

围栏是一个整数，表示当前正在处理的工作单位。 当应用通过调用 [**ID3D12CommandQueue::Signal**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandqueue-signal) 推进围栏时，该整数将会更新。 应用可以检查围栏的值，并确定某个工作单位是否已完成，从而确定是否可以启动后续的操作。

## <a name="synchronizing-resources-accessed-by-command-queues"></a>同步命令队列访问的资源

在 Direct3D 12 中，同步某些资源的状态是使用资源屏障实现的。 在每个资源屏障中，应用声明资源的之前和之后状态。 一个常见示例是资源在着色器资源视图与渲染器目标视图之间转换。 这些资源屏障主要在命令列表内部进行管理。 启用调试层后，系统会强制所有资源的之前和之后状态匹配，保证资源在屏障转换时处于正确的状态，可执行特定的操作。

有关同步资源状态的详细信息，请参阅[使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。

## <a name="command-queue-support-for-tiled-resources"></a>图块式资源的命令队列支持

在 Direct3D 11 中通过 [**ID3D11DeviceContext2**](https://msdn.microsoft.com/library/windows/desktop/dn280498) 接口公开的用于管理图块式资源的方法在 Direct3D 12 中由 [**ID3D12CommandQueue**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue) 接口提供。 这些方法包括：



| 方法                                                              | 描述                                                                                              |
|---------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| [**CopyTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-copytilemappings)     | 将映射从源图块式资源复制到目标图块式资源。<br/>                 |
| [**UpdateTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings) | 将图块式资源中的图块位置映射更新为资源堆中的内存位置。<br/> |



 

有关在 Direct3D 12 应用中使用图块式资源的详细信息，请参阅 [Direct3D11 图块式资源](https://msdn.microsoft.com/library/windows/desktop/dn786477)。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 中的工作提交](command-queues-and-command-lists.md)
</dt> </dl>

 

 





