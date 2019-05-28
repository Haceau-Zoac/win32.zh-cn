---
title: UMA 优化 CPU 可访问纹理和标准 Swizzle
description: 通用内存体系结构 (UMA) Gpu 相比，具有一些效率优点离散 Gpu，尤其是在针对移动设备进行优化。
ms.assetid: 26C41948-9625-4786-BBDF-552D1F8A2437
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 3899b8b9150927779b922a7948721e301839675b
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224152"
---
# <a name="uma-optimizations-cpu-accessible-textures-and-standard-swizzle"></a>UMA 优化：CPU 可访问纹理和标准 Swizzle

通用内存体系结构 (UMA) Gpu 相比，具有一些效率优点离散 Gpu，尤其是在针对移动设备进行优化。 当 GPU UMA 可以减少的复制，为资源指定 CPU 访问 CPU 和 GPU 之间发生。 虽然我们不建议应用程序会盲目地授予对 UMA 设计上的所有资源的 CPU 访问权限，有机会来提高效率，通过提供 CPU 访问的适当的资源。 与离散 Gpu 不同 CPU 从技术上讲可以有一个指针指向 GPU 可以访问的所有资源。

-   [CPU 可访问纹理的概述](#overview-of-cpu-accessible-textures)
-   [标准 Swizzle 的概述](#overview-of-standard-swizzle)
-   [Api](#apis)
-   [相关的主题](#related-topics)

## <a name="overview-of-cpu-accessible-textures"></a>CPU 可访问纹理的概述

CPU 可访问的纹理相同，在图形管道中，是一项功能的 UMA 体系结构，使 Cpu 读取和写入访问权限的纹理。 在更常见的离散 Gpu，CPU 没有访问纹理图形管道中。

纹理的常规最佳做法建议是为了满足离散 Gpu，这通常涉及以下在进程[将纹理数据上传通过缓冲区](upload-and-readback-of-texture-data.md)、 汇总为：

-   无法对大部分纹理任何 CPU 访问。
-   将纹理布局设置为 D3D12\_纹理\_布局\_未知。
-   将纹理上传到与 GPU [ **CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)。

但是，对于某些情况下，CPU 和 GPU 可能相当频繁，因此上交互的相同数据，贴图纹理就很有用，以节约电源，或以特定的适配器或体系结构上加快特定设计。 应用程序应检测这种情况下，并优化掉不必要的副本。 在这种情况下，为了获得最佳性能，请考虑以下：

-   仅启动幽默映射的更好的性能纹理何时[ **D3D12\_功能\_数据\_体系结构**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_architecture):: UMA 为 TRUE。 然后请注意*CacheCoherentUMA*如果决定要选择在堆上的 CPU 缓存属性。

-   利用纹理的 CPU 访问是比要复杂一些的缓冲区。 Gpu 的最有效纹理布局很少行\_主要。 事实上，某些 Gpu 只能支持行\_复制时的主要纹理纹理周围的数据。

-   UMA Gpu 普遍将受益于简单的优化以便减少级别加载时间。 在确认 UMA 之后, 应用程序可以优化掉初始[ **CopyTextureRegion** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)来填充将不会修改 GPU 的纹理。 而不是使用 D3D12 堆中创建纹理\_堆\_类型\_默认情况下，和封送处理，通过将纹理数据应用程序可以使用[ **WriteToSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource)以免了解实际纹理布局。

-   在 D3D12，纹理使用创建 D3D12\_纹理\_布局\_未知和无 CPU 访问是最有效的频繁的 GPU 呈现和采样。 性能测试，这些纹理应在何时进行比较 D3D12\_纹理\_布局\_未知和 CPU 访问权限 D3D12\_纹理\_布局\_标准\_SWIZZLE 和 CPU 访问权限 D3D12\_纹理\_布局\_行\_主要跨适配器支持。

-   使用 D3D12\_纹理\_布局\_CPU 访问未知使方法[ **WriteToSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource)， [ **ReadFromSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource)， [**映射**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map) （不能进行应用程序访问到指针），以及[ **Unmap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap); 但可以选择了牺牲效率的 GPU 访问权限。

-   使用 D3D12\_纹理\_布局\_标准\_SWIZZLE CPU 访问使[ **WriteToSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource)， [ **ReadFromSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource)， [**映射**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map) （返回有效的指针到应用程序），以及[ **Unmap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap). 它还可以牺牲效率的 GPU 访问多个 D3D12\_纹理\_布局\_CPU 访问未知。

## <a name="overview-of-standard-swizzle"></a>标准 Swizzle 的概述

D3D12 （和 D3D11.3） 引入了标准多维数据布局。 这样做是为了启用多个处理单元来操作相同的数据而不复制数据或 swizzling 多个布局之间的数据。 标准化的布局使通过网络效果的效率提高，并允许算法利用快捷方式的假设的特定模式。

有关纹理布局的详细说明，请参阅[ **D3D12\_纹理\_布局**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_texture_layout)。

不过请注意此标准 swizzle 是一种硬件功能，并且可能不支持的所有 Gpu。

有关 swizzling 的背景信息，请参阅[Z 顺序曲线](https://en.wikipedia.org/wiki/Z-order_curve)。

## <a name="apis"></a>API

与不同 D3D11.3，D3D12 支持纹理映射的默认值，因此无需查询[ **D3D12\_功能\_数据\_D3D12\_选项**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_d3d12_options)。 但是 D3D12 不总是支持标准 swizzle-此功能将需要通过调用查询[ **CheckFeatureSupport** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport)并检查*StandardSwizzle64KBSupported*字段**D3D12\_功能\_数据\_D3D12\_选项**。

以下 Api 引用纹理映射：

枚举

-   [**D3D12\_纹理\_布局**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_texture_layout) ： 控制默认纹理的 swizzle 模式并启用对 CPU 可访问纹理映射支持。

结构

-   [**D3D12\_资源\_DESC** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_resource_desc) ： 描述资源，如纹理，这是一个广泛使用的结构。
-   [**D3D12\_堆\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_heap_desc) ： 描述堆。

方法

-   [**ID3D12Device::CreateCommittedResource** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommittedresource) ： 创建单个资源和适当的大小和对齐方式的后备堆。
-   [**ID3D12Device::CreateHeap** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createheap) ： 创建一个堆的缓冲区或纹理。
-   [**ID3D12Device::CreatePlacedResource** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createplacedresource) ： 创建的资源的特定堆，通常创建于资源的速度更快的方法中放置[ **CreateHeap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createheap)。
-   [**ID3D12Device::CreateReservedResource** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createreservedresource) ： 创建保留，但尚未提交或放置在堆中的资源。
-   [**ID3D12CommandQueue::UpdateTileMappings** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings) ： 磁贴中的位置平铺资源映射更新到资源堆中的内存位置。
-   [**ID3D12Resource::Map** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map) ： 获取一个指向指定的数据在资源中，并拒绝对子资源的 GPU 访问权限。
-   [**ID3D12Resource::GetDesc** ](id3d12resource-getdesc.md) ： 获取资源属性。
-   [**ID3D12Heap::GetDesc** ](id3d12heap-getdesc.md)获取堆属性。
-   [**ReadFromSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource) ： 将数据从其使用映射的纹理复制[**映射**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)。
-   [**WriteToSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource) ： 将数据复制到其中使用映射的纹理[**映射**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)。

资源和父堆具有对齐要求：

-   D3D12\_默认\_MSAA\_资源\_放置\_多重采样的纹理的对齐方式 (4 MB)。
-   D3D12\_默认\_资源\_放置\_单一样本纹理和缓冲区的对齐方式 (64 KB)。
-   线性 subresource 复制必须对齐到 D3D12\_纹理\_数据\_放置\_（512 字节为单位），符合所对齐到 D3D12 行间距\_纹理\_数据\_间距\_对齐方式 （256 个字节）。
-   常量缓冲区视图必须符合 D3D12\_常量\_缓冲区\_数据\_放置\_对齐方式 （256 个字节）。

纹理小于 64 KB 应通过处理[ **CreateCommittedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommittedresource)。

使用动态纹理 （更改每一帧的纹理） CPU 将对上传堆，跟 GPU 复制操作以线性方式编写。

通常要创建动态资源创建较大的缓冲区上传堆中 (请参阅[缓冲区内的子分配](large-buffers.md))。 若要创建过渡资源，请在 readback 堆中创建较大的缓冲区。 若要创建默认静态资源，请在默认堆中创建相邻资源。 若要创建默认别名资源，请在默认堆中创建重叠的资源。

[**WriteToSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource)并[ **ReadFromSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource)重新排列行优先布局和未定义的资源布局之间的纹理数据。 操作是同步的因此应用程序应保留 CPU 计划记住。 应用程序始终可以分解成较小的区域复制或计划在另一个任务中此操作。 MSAA 资源和深度模具资源使用不透明的资源布局不受这些 CPU 复制操作，并将导致失败。 没有 power 的两个元素大小的格式也不受支持，还将导致失败。 内存不足可能发生的返回代码。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[**核心常量**](constants.md)
</dt> <dt>

[**ID3D12Heap**](/windows/desktop/api/D3D12/nn-d3d12-id3d12heap)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 




