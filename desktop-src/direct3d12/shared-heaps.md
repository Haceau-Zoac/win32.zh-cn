---
title: 共享的堆
description: 共享可用于多进程和多适配器体系结构。
ms.assetid: 67C6B1D4-BF76-45A9-BADC-7C9520C900EB
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 5a5bd93846c8e6550238bb30f43a9c84c0aab9f0
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223906"
---
# <a name="shared-heaps"></a>共享的堆

共享可用于多进程和多适配器体系结构。

-   [共享概述](#sharing-overview)
-   [共享进程间的堆](#sharing-heaps-across-processes)
-   [共享跨适配器堆](#sharing-heaps-across-adapters)
-   [相关的主题](#related-topics)

## <a name="sharing-overview"></a>共享概述

共享的堆启用两件事： 在一个或多个进程之间共享数据在堆中的并不能进行资源放置在堆中未定义的纹理布局的一个非确定性的选择。 共享适配器之间的堆也无需 CPU 封送处理的数据。

这两个堆，并且可以共享已提交的资源。 实际上共享提交的资源共享以及提交的资源说明中，隐式堆，以便兼容资源说明可以从另一台设备映射到堆。

所有方法是自由线程，继承现有的共享设计的 NT 句柄 D3D11 语义。

-   [**ID3D12Device::CreateSharedHandle**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createsharedhandle)
-   [**ID3D12Device::OpenSharedHandle**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-opensharedhandle)
-   [**ID3D12Device::OpenSharedHandleByName**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-opensharedhandlebyname)

## <a name="sharing-heaps-across-processes"></a>共享进程间的堆

使用 D3D12 指定共享的堆\_堆\_标志\_的共享成员[ **D3D12\_堆\_标志**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_heap_flags)枚举。

共享的堆不支持的可访问 CPU 的堆：D3D12\_堆\_类型\_将上传，D3D12\_堆\_类型\_READBACK 和 D3D12\_堆\_类型\_自定义而无需 D3D12\_CPU\_页上\_属性\_不\_可用。

不能进行未定义的纹理布局的一个非确定性的选择可以情况下显著降低某些 Gpu 上的延迟的呈现方案以便它不是放置和提交资源的默认行为。 延迟的呈现某些 GPU 体系结构上是被削弱因为确定性纹理布局降低实现同时向多个相同的格式和大小的呈现器目标纹理呈现时的有效内存带宽。 从以延迟呈现有效地支持标准化的 swizzle 模式和标准化的布局利用非确定性纹理布局不断演变 GPU 体系结构。

共享的堆附带其他次要的成本：

-   原样灵活过程中由于信息泄露问题的堆，因此物理内存 zero'ed 频率不能回收共享的堆数据。
-   次要的额外 CPU 开销和更高的系统内存使用情况期间没有共享的堆的创建和析构。

## <a name="sharing-heaps-across-adapters"></a>共享跨适配器堆

使用 D3D12 指定适配器跨共享的堆\_堆\_标志\_共享\_跨\_适配器隶属[ **D3D12\_堆\_标志**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_heap_flags)枚举。

适配器跨共享的堆启用多个适配器没有封送处理它们之间的数据的 CPU 的情况下共享数据。 而不同适配器功能确定如何有效地适配器它们只启用 GPU 副本之间传递数据可以提高来实现的有效带宽。 某些纹理布局上允许跨适配器堆，以支持纹理数据的交换，即使不支持此类纹理布局。 某些限制可能适用于这种纹理，如仅支持复制。

跨适配器与堆通过调用创建共享的工作原理[ **ID3D12Device::CreateHeap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createheap)。 然后，应用程序可以创建资源通过[ **CreatePlacedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createplacedresource)。 它还允许的资源/堆创建[ **CreateCommittedResource** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommittedresource)但仅对行优先 D3D12\_资源\_维度\_TEXTURE2D资源 (请参阅[ **D3D12\_资源\_维度**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_dimension))。 跨适配器共享并不适用于[ **CreateReservedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createreservedresource)。

用于跨适配器共享，所有常用的跨队列资源共享规则仍适用。 应用程序必须发出相应的屏障，以确保正确同步，并在两个适配器之间的一致性。 应用程序应使用跨适配器界定来协调命令列表提交到多个适配器的计划。 没有任何机制来在 D3D API 版本之间共享跨适配器资源。 在系统内存中仅支持跨适配器共享的资源。 跨适配器共享堆 / 资源支持 D3D12\_堆\_类型\_默认堆和 D3D12\_堆\_类型\_堆 （与 L0 内存池，并写入-将自定义CPU 页属性）。 必须确保 GPU 读/写操作到跨适配器共享堆是一致的系统上的其他 Gpu 驱动程序。 例如，驱动程序可能需要从驻留在 GPU 通常不需要时 CPU 不能直接访问堆数据刷新的缓存中排除了堆数据。

应用程序应限制跨堆到需要它们提供的功能的方案的适配器的使用情况。 跨适配器堆都位于 D3D12\_内存\_池\_L0，这并不总是什么[ **GetCustomHeapProperties** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getcustomheapproperties)建议。 该内存池不是有效的离散 / NUMA 适配器体系结构。 和最高效的纹理布局并不总是可用。

以下限制也适用：

-   与堆层相关的堆标志必须为 D3D12\_堆\_标志\_允许\_所有\_缓冲区\_AND\_纹理。
-   D3D12\_堆\_标志\_还必须设置共享。
-   任一 D3D12\_堆\_类型\_默认值必须为组或 D3D12\_堆\_类型\_D3D12 与自定义\_内存\_池\_L0 和 D3D12\_CPU\_页面\_属性\_不\_必须设置可用。
-   仅使用 D3D12 资源\_资源\_标志\_允许\_跨\_适配器可能会将其置于跨适配器堆。

有关使用多个适配器的详细信息，请参阅[多适配器](multi-engine.md)部分。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[子中堆分配](suballocation-within-heaps.md)
</dt> </dl>

 

 




