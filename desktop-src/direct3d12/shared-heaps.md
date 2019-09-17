---
title: 共享堆
description: 共享对于多进程和多适配器体系结构非常有用。
ms.assetid: 67C6B1D4-BF76-45A9-BADC-7C9520C900EB
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 103dc2ff0f093461fc29e2f338cebecc70364f57
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71005671"
---
# <a name="shared-heaps"></a>共享堆

共享对于多进程和多适配器体系结构非常有用。

-   [共享概述](#sharing-overview)
-   [在进程之间共享堆](#sharing-heaps-across-processes)
-   [在适配器之间共享堆](#sharing-heaps-across-adapters)
-   [相关主题](#related-topics)

## <a name="sharing-overview"></a>共享概述

共享堆支持以下两项：在一个或多个进程之间共享堆中的数据，以及排除放置在堆中的资源的未定义纹理布局的非确定性选择。 在适配器之间共享堆也无需对数据进行 CPU 封送。

可以共享堆和提交的资源。 共享提交的资源实际上是共享隐式堆以及提交的资源说明，以便可以将兼容的资源说明从其他设备映射到堆。

所有方法都是自由线程，并继承 NT 句柄共享设计的现有 D3D11 语义。

-   [**ID3D12Device::CreateSharedHandle**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createsharedhandle)
-   [**ID3D12Device::OpenSharedHandle**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-opensharedhandle)
-   [**ID3D12Device::OpenSharedHandleByName**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-opensharedhandlebyname)

## <a name="sharing-heaps-across-processes"></a>在进程之间共享堆

共享堆使用 [**D3D12\_HEAP\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_heap_flags) 枚举的 D3D12\_HEAP\_FLAG\_SHARED 成员指定。

CPU 可访问的堆不支持共享堆：不带 D3D12\_CPU\_PAGE\_PROPERTY\_NOT\_AVAILABLE 的 D3D12\_HEAP\_TYPE\_UPLOAD、D3D12\_HEAP\_TYPE\_READBACK 和 D3D12\_HEAP\_TYPE\_CUSTOM。

排除未定义纹理布局的非确定性选择可能会显著损害某些 GPU 上的延迟呈现情况，因此这不是放置和提交的资源的默认行为。 延迟呈现在某些 GPU 体系结构上受损，因为确定性纹理布局减少在同时呈现到格式和大小相同的多个呈现器目标纹理时获得的有效内存带宽。 GPU 体系结构正在演变为不使用非确定性纹理布局，以便有效地支持延迟呈现的标准化重排模式和标准化布局。

共享堆也附带其他少量成本：

-   由于信息泄露问题，无法像进程内堆一样灵活地回收共享堆数据，因此物理内存通常会归零。
-   在创建和销毁共享堆期间，附加了少量 CPU 开销并增加了系统内存使用量。

## <a name="sharing-heaps-across-adapters"></a>在适配器之间共享堆

适配器之间的共享堆使用 [**D3D12\_HEAP\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_heap_flags) 枚举的 D3D12\_HEAP\_FLAG\_SHARED\_CROSS\_ADAPTER 成员指定。

通过跨适配器的共享堆，多个适配器可以在不对其中的数据进行 CPU 封送的情况下共享数据。 虽然不同的适配器功能可以确定适配器传递其中的数据的效率，但仅启用 GPU 副本即可增加获得的有效带宽。 跨适配器堆上允许某些纹理布局，以支持纹理数据交换，即使不支持此类纹理布局也是如此。 某些限制可能适用于此类纹理，例如仅支持复制。

跨适配器共享适用于通过调用 [**ID3D12Device::CreateHeap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createheap) 而创建的堆。 然后，应用程序可以通过 [**CreatePlacedResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createplacedresource) 创建资源。 还允许 [**CreateCommittedResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommittedresource) 创建的资源/堆，但仅适用于行主序 D3D12\_RESOURCE\_DIMENSION\_TEXTURE2D 资源（请参阅 [**D3D12\_RESOURCE\_DIMENSION**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_dimension)）。 跨适配器共享并不适用于 [**CreateReservedResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createreservedresource)。

对于跨适配器共享，所有常用跨队列资源共享规则仍适用。 应用程序必须发出相应的屏障才能确保两个适配器之间的正确同步和一致性。 应用程序应使用跨适配器围栏来协调提交到多个适配器的命令列表计划。 没有用于在 D3D API 版本之间共享跨适配器资源的机制。 系统内存中仅支持跨适配器共享资源。 D3D12\_HEAP\_TYPE\_DEFAULT 堆和 D3D12\_HEAP\_TYPE\_CUSTOM 堆中支持跨适配器共享堆/资源（具有 L0 内存池，和写入合并 CPU 页属性）。 驱动程序必须确保对跨适配器共享堆执行的 GPU 读/写操作与系统上的其他 GPU 一致。 例如，驱动程序可能需要从通常不需要在 CPU 无法直接访问堆数据时刷新的 GPU 缓存中排除堆数据。

应用程序应将跨适配器堆的使用情况限制为需要它们提供的功能的情况。 跨适配器堆位于 D3D12\_MEMORY\_POOL\_L0 中，[**GetCustomHeapProperties**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getcustomheapproperties) 并不总是建议如此。 该内存池不适用于离散/NUMA 适配器体系结构。 最高效的纹理布局并不总是可用。

以下限制也适用：

-   与堆层相关的堆标志必须为 D3D12\_HEAP\_FLAG\_ALLOW\_ALL\_BUFFERS\_AND\_TEXTURES。
-   还必须设置 D3D12\_HEAP\_FLAG\_SHARED。
-   必须设置 D3D12\_HEAP\_TYPE\_DEFAULT 或必须设置带有 D3D12\_MEMORY\_POOL\_L0 and D3D12\_CPU\_PAGE\_PROPERTY\_NOT\_AVAILABLE 的 D3D12\_HEAP\_TYPE\_CUSTOM。
-   只有带有 D3D12\_RESOURCE\_FLAG\_ALLOW\_CROSS\_ADAPTER 的资源才有可能放置在跨适配器堆上。

有关使用多个适配器的详细信息，请参阅[多适配器](multi-engine.md)部分。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[堆中的子分配](suballocation-within-heaps.md)
</dt> </dl>

 

 




