---
title: 内存别名和数据继承
description: 放置和保留的资源可能在堆中对物理内存设置别名。 当堆具有共享标志集或当具有别名的资源已完全定义内存布局时，放置的资源启用的数据继承方案比保留的资源更多。
ms.assetid: 53C5804B-0F81-41AF-83D2-A96DCDFDC94A
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: cace5b5997e2a460406ae72abb247224886f3926
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71005976"
---
# <a name="memory-aliasing-and-data-inheritance"></a>内存别名和数据继承

放置和保留的资源可能在堆中对物理内存设置别名。 当堆具有共享标志集或当具有别名的资源已完全定义内存布局时，放置的资源启用的数据继承方案比保留的资源更多。

-   [别名](#memory-aliasing-and-data-inheritance)
-   [数据继承](#data-inheritance)
-   [相关主题](#related-topics)

## <a name="aliasing"></a>别名

必须在共享相同物理内存的两个资源的使用情况之间发出别名屏障，即使不需要数据继承也是如此。 简单的使用情况模型必须至少指明此类操作中涉及的目标资源。 有关更多详细信息和高级使用情况模型，请参阅 [**CreatePlacedResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createplacedresource)。

访问资源后，与该资源共享物理内存的任何资源将变为无效，除非允许发生数据继承。 读取无效资源导致未定义的资源内容。 写入无效资源也会导致未定义的资源内容，除非发生以下两种情况：

-   资源没有 D3D12\_RESOURCE\_FLAG\_ALLOW\_RENDER\_TARGET 或 D3D12\_RESOURCE\_FLAG\_ALLOW\_DEPTH\_STENCIL。
-   写入是对整个子资源或磁贴的复制或清除操作。 磁贴初始化功能仅适用于具有 64KB\_TILE\_UNDEFINED\_SWIZZLE 和 64KB\_TILE\_STANDARD\_SWIZZLE 的资源。

当布局提供有关纹素数据位置的信息且资源处于某些转换屏障状态时，重叠失效的作用域限定为较小的粒度。 但是，失效不能小于资源对齐粒度。

缓冲区对齐粒度为 64 KB，而更大的对齐粒度优先级更高。 这在考虑 4 KB 纹理时非常重要，因为多个 4 KB 纹理可以驻留在 64 KB 区域中，但不相互重叠。 但是，为相同的 64 KB 区域设置别名的缓冲区不能与其中任意 4 KB 纹理一起使用。 应用程序无法可靠地进入缓冲区与 4 KB 纹理相交，因为允许 GPU 在未定义的模式下对 64 KB 区域中的 4 KB 纹理数据进行重排。

64KB\_TILE\_UNDEFINED\_SWIZZLE、64KB\_TILE\_STANDARD\_SWIZZLE 和 ROW\_MAJOR 纹理布局通知应用程序哪些重叠对齐粒度已变为无效。 例如：应用程序可以使用 2 个数组切片、单个 mip 级别和 64KB\_TILE\_UNDEFINED\_SWIZZLE 布局创建 2D 呈现器目标纹理数组。 假设应用程序了解每个数组切片会占用 100 个 64 KB 磁贴。 应用程序可以放弃使用数组切片 0，并将该内存重复用于约 6 MB 的缓冲区、具有未定义布局的约 6 MB 纹理等。进一步假设应用程序不再需要数组切片 1 的第一个磁贴。 应用程序则还可以查找 64 KB 缓冲区，直到呈现操作再次需要数组切片 1 的第一个磁贴。 应用程序必须完全清除或复制磁贴，才能再次将第一个磁贴重复用于纹理数组。

但是，即使具有已定义布局的纹理也仍有问题。 纹理资源大小可能明显不同于应用程序本身可以计算的大小，因为某些适配器体系结构会为纹理分配额外内存以减少常见呈现方案过程中的有效带宽。 该额外内存区域中的任何失效导致整个资源变为失效。 有关更多详细信息，请参阅 [**GetResourceAllocationInfo**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getresourceallocationinfo)。

## <a name="data-inheritance"></a>数据继承

放置的资源启用纹理的大多数数据继承，即使具有未定义的内存布局也是如此。 应用程序可以通过查找共享堆中的同一偏移处具有相同资源属性的两个纹理来模拟共享提交资源启用的数据继承功能。 整个资源说明必须相同，包括优化的清除值和资源创建方法的类型（放置或保留）。 但是，这两个资源可能具有不同的初始转换屏障状态。

保留的资源启用每磁贴数据继承；但资源转换屏障状态通常存在限制。

若要继承数据，这两个资源必须处于兼容的资源转换屏障状态：

-   对于缓冲区、同时访问纹理和跨适配器纹理，资源转换状态并不重要，所有状态均为“兼容”。
-   对于不带之前属性的保留的纹理或通过 64KB\_TILE\_UNDEFINED\_SWIZZLE 或 64KB\_TILE\_STANDARD\_SWIZZLE 的其他每磁贴数据继承，包括磁贴的资源转换屏障状态必须处于通用状态。
-   对于资源说明完全匹配的所有其他纹理，每个相应的子资源对的转换屏障状态必须：
    -   处于通用状态。
    -   当状态具有相同的 GPU 写入标志时相等。

当 GPU 支持标准重排时，缓冲区和标准重排纹理可能会别名为相同的内存并继承它们之间的数据。 应用程序可以操作缓冲区表示形式中的纹素，因为标准重排模式描述纹素在内存中的布局方式。 CPU 可见的重排模式相当于缓冲区中看到的 GPU 可见的重排模式。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[堆中的子分配](suballocation-within-heaps.md)
</dt> </dl>

 

 




