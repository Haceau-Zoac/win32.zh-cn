---
title: UMA 优化 CPU 可访问纹理和标准重排
description: 通用内存体系结构 (UMA) GPU 与离散 GPU 相比具有一定的效率优势，尤其是在针对移动设备进行优化时。
ms.assetid: 26C41948-9625-4786-BBDF-552D1F8A2437
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: fd0f0c636e21f28dc3c77212470579463766a15e
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006270"
---
# <a name="uma-optimizations-cpu-accessible-textures-and-standard-swizzle"></a>UMA 优化：CPU 可访问纹理和标准重排

通用内存体系结构 (UMA) GPU 与离散 GPU 相比具有一定的效率优势，尤其是在针对移动设备进行优化时。 当 GPU 是 UMA 时，授予资源 CPU 访问权限可以减少 CPU 和 GPU 之间发生的复制量。 虽然我们不建议盲目地让 CPU 访问 UMA 设计上的所有资源，但仍有机会通过提供正确的 CPU 资源访问来提高效率。 与离散 GPU 不同，CPU 在技术上可以指向 GPU 可访问的所有资源。

-   [CPU 可访问纹理的概述](#overview-of-cpu-accessible-textures)
-   [标准重排的概述](#overview-of-standard-swizzle)
-   [API](#apis)
-   [相关主题](#related-topics)

## <a name="overview-of-cpu-accessible-textures"></a>CPU 可访问纹理的概述

图形管道中的 CPU 可访问纹理是 UMA 体系结构的一项功能，允许 CPU 对纹理进行读写访问。 在更常见的离散 GPU 上，CPU 没有权限访问图形管道中的纹理。

纹理的一般最佳做法建议是容纳离散 GPU，这通常涉及执行[通过缓冲区上传纹理数据](upload-and-readback-of-texture-data.md)中的过程，总结如下：

-   没有对大部分纹理的 CPU 访问权限。
-   将纹理布局设置为 D3D12\_TEXTURE\_LAYOUT\_UNKNOWN。
-   使用 [**CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion) 将纹理上传到 GPU。

但是，在某些情况下，CPU 和 GPU 可能会在相同的数据上非常频繁地交互，以便于映射纹理节约电量，或加速有关特定适配器或体系结构的特定设计。 应用程序应检测这些情况，并优化掉不必要的副本。 在这种情况下，为了获得最佳性能，请考虑以下事项：

-   当 [**D3D12\_FEATURE\_DATA\_ARCHITECTURE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_architecture)::UMA 为 TRUE 时，仅开始实现映射纹理的更好性能。 然后注意 CacheCoherentUMA（如果决定在堆上选择哪些 CPU 缓存属性）。

-   利用纹理的 CPU 访问权限比利用缓冲区的 CPU 访问权限更复杂。 GPU 的最高效纹理布局很少是行主序布局\_。 事实上，当复制周围的纹理数据时，某些 GPU 只能支持行主序纹理\_。

-   UMA GPU 应通常受益于简单的优化以便减少级别加载时间。 识别 UMA 之后，应用程序可以优化掉初始 [**CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion) 来填充 GPU 不会修改的纹理。 应用程序可以使用 [**WriteToSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource) 以免了解实际纹理布局，而不是使用 D3D12\_HEAP\_TYPE\_DEFAULT 在堆中创建纹理和封送纹理数据。

-   在 D3D12 中，使用 D3D12\_TEXTURE\_LAYOUT\_UNKNOWN 创建且无 CPU 访问权限的纹理最适用于频繁的 GPU 呈现和采样。 进行性能测试时，这些纹理应与具有 CPU 访问权限的 D3D12\_TEXTURE\_LAYOUT\_UNKNOWN、具有 CPU 访问权限的 D3D12\_TEXTURE\_LAYOUT\_STANDARD\_SWIZZLE 和 D3D12\_TEXTURE\_LAYOUT\_ROW\_MAJOR 进行比较以获得跨适配器支持。

-   使用具有 CPU 访问权限的 D3D12\_TEXTURE\_LAYOUT\_UNKNOWN 可启用方法 [**WriteToSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource)、[**ReadFromSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource)、[**Map**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map)（排除对指针的应用程序访问），以及 [**Unmap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-unmap)；但会影响 GPU 访问的效率。

-   使用具有 CPU 访问权限的 D3D12\_TEXTURE\_LAYOUT\_STANDARD\_SWIZZLE 可启用 [**WriteToSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource)、[**ReadFromSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource)、[**Map**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map)（返回指向应用程序的有效指针），以及 [**Unmap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-unmap)。 它还会比具有 CPU 访问权限的 D3D12\_TEXTURE\_LAYOUT\_UNKNOWN 更影响 GPU 访问的效率。

## <a name="overview-of-standard-swizzle"></a>标准重排的概述

D3D12（和 D3D11.3）引入了标准多维数据布局。 这样做是为了对相同数据运行多个处理单元，而不在多个布局之间复制数据或重排数据。 使用标准化的布局，可以通过网络效果提高效率，并允许算法利用快捷方式假设特定模式。

有关纹理布局的详细说明，请参阅 [**D3D12\_TEXTURE\_LAYOUT**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_texture_layout)。

请注意，此标准重排是一种硬件功能，并非所有 GPU 都支持该功能。

有关重排的背景信息，请参阅 [Z 顺序曲线](https://en.wikipedia.org/wiki/Z-order_curve)。

## <a name="apis"></a>API

与 D3D11.3 不同，D3D12 默认支持纹理映射，因此无需查询 [**D3D12\_FEATURE\_DATA\_D3D12\_OPTIONS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options)。 但是，D3D12 并不总是支持标准重排，将需要通过调用 [**CheckFeatureSupport**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport) 并选中 **D3D12\_FEATURE\_DATA\_D3D12\_OPTIONS** 的 StandardSwizzle64KBSupported 字段来查询此功能。

以下 API 引用纹理映射：

枚举

-   [**D3D12\_TEXTURE\_LAYOUT**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_texture_layout)：控制默认纹理的重排模式并启用对 CPU 可访问纹理的映射支持。

结构

-   [**D3D12\_RESOURCE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_resource_desc)：描述资源（如纹理），这是一个广泛使用的结构。
-   [**D3D12\_HEAP\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_heap_desc)：描述堆。

方法

-   [**ID3D12Device::CreateCommittedResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommittedresource)：创建单个资源以及具有适当大小和对齐方式的后备堆。
-   [**ID3D12Device::CreateHeap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createheap)：为缓冲区或纹理创建堆。
-   [**ID3D12Device::CreatePlacedResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createplacedresource)：创建放置在特定堆中的资源，这通常是比 [**CreateHeap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createheap) 更快的创建资源方法。
-   [**ID3D12Device::CreateReservedResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createreservedresource)：创建预留，但尚未提交或放置在堆中的资源。
-   [**ID3D12CommandQueue::UpdateTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings)：将平铺资源中的磁贴位置映射更新到资源堆中的内存位置。
-   [**ID3D12Resource::Map**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map)：获取指向资源中的指定数据的指针，并拒绝对子资源的 GPU 访问。
-   [**ID3D12Resource::GetDesc**](id3d12resource-getdesc.md)：获取资源属性。
-   [**ID3D12Heap::GetDesc**](id3d12heap-getdesc.md)：获取堆属性。
-   [**ReadFromSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource)：从使用 [**Map**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map) 映射的纹理中复制数据。
-   [**WriteToSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource)：将数据复制到使用 [**Map**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map) 映射的纹理中。

资源和父堆具有对齐要求：

-   对于多重采样纹理，则为 D3D12\_DEFAULT\_MSAA\_RESOURCE\_PLACEMENT\_ALIGNMENT (4MB)。
-   对于单采样纹理和缓冲区，则为 D3D12\_DEFAULT\_RESOURCE\_PLACEMENT\_ALIGNMENT (64KB)。
-   线性子资源复制必须与 D3D12\_TEXTURE\_DATA\_PLACEMENT\_ALIGNMENT（512 个字节）对齐，使行间距与 D3D12\_TEXTURE\_DATA\_PITCH\_ALIGNMENT（256 个字节）对齐。
-   常量缓冲区视图必须与 D3D12\_CONSTANT\_BUFFER\_DATA\_PLACEMENT\_ALIGNMENT（256 个字节）对齐。

应通过 [**CreateCommittedResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommittedresource) 处理小于 64 KB 的纹理。

使用动态纹理（更改每个帧的纹理），CPU 将以线性方式写入上传堆，然后执行 GPU 复制操作。

通常，若要创建动态资源，请在上传堆中创建较大的缓冲区（请参阅[缓冲区中的子分配](large-buffers.md)）。 若要创建暂存资源，请在读回堆中创建较大的缓冲区。 若要创建默认静态资源，请在默认堆中创建相邻资源。 若要创建默认别名资源，请在默认堆中创建重叠资源。

[**WriteToSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource) 和 [**ReadFromSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource) 会在行主序布局和未定义的资源布局之间重新排列纹理数据。 操作是同步的，因此应用程序应记住 CPU 计划。 应用程序始终可以将复制拆分为较小的区域或在另一个任务中计划此操作。 具有不透明资源布局的 MSAA 资源和深度模具资源不受这些 CPU 复制操作支持，并且会导致失败。 没有二次幂元素大小的格式也不受支持，并且也会导致失败。 可能会出现内存不足的返回代码。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[**核心常量**](constants.md)
</dt> <dt>

[**ID3D12Heap**](/windows/desktop/api/d3d12/nn-d3d12-id3d12heap)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 




