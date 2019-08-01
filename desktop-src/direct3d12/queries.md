---
title: 查询
description: 在 Direct3D 12 中，查询会分组为称为查询堆的查询数组。 查询堆具有一个类型，该类型用于定义可与该堆配合使用的有效查询类型。
ms.assetid: d7403b5d-7e1b-4dd2-ae45-52e1153233c6
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 6c56b3f2583b46272ae837a47ae085abd751f47d
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296354"
---
# <a name="queries"></a>查询

在 Direct3D 12 中，查询会分组为称为查询堆的查询数组。 查询堆具有一个类型，该类型用于定义可与该堆配合使用的有效查询类型。

-   [从 Direct3D 11 到 Direct3D 12 的查询差异](#differences-in-queries-from-direct3d-11-to-direct3d-12)
-   [查询堆](#query-heaps)
-   [创建查询堆](#creating-query-heaps)
-   [从查询中提取数据](#extracting-data-from-a-query)
-   [相关主题](#related-topics)

## <a name="differences-in-queries-from-direct3d-11-to-direct3d-12"></a>从 Direct3D 11 到 Direct3D 12 的查询差异

Direct3D 12 不再使用以下查询类型，而是将其功能合并到其他进程：

-   **事件查询** - 事件功能现由栅栏处理。
-   **不连续的时间戳查询** - GPU 时钟可在 Direct3D 12 中设置为稳定状态（请参见[计时](timing.md)一节）。 如果 GPU 在时间戳之间完全空闲（称为不连续查询），那么 GPU 时钟比较就没有意义。 通过稳定的电源，可以可靠地比较从不同命令列表发出的两个时间戳查询。 同一命令列表中的两个时间戳始终可以可靠地进行比较。
-   **流输出统计信息查询** - Direct3D 12 中没有适用于所有输出流的单个流输出 (SO) 溢出查询。 应用需要发出多个单流查询，然后关联结果。
-   **流输出统计信息预测和封闭预测查询** - 查询（写入内存）和[预测](predication.md)（读取内存）不再耦合，因此不需要这些查询类型。

新的二进制封闭查询类型已添加到 Direct3D 12。

## <a name="query-heaps"></a>查询堆

查询可以是多种类型（[D3D12\_QUERY\_HEAP\_TYPE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_query_heap_type)）中的一种，并先分组为查询堆，然后再提交到 GPU  。

新查询类型 D3D12\_QUERY\_TYPE\_BINARY\_OCCLUSION 可用，除会返回二进制 0/1 结果以外，其作用类似于 D3D12\_QUERY\_TYPE\_OCCLUSION：0 表示没有示例通过深度和模板测试，1 表示至少一个示例通过深度和模具测试。 这确保封闭查询不会干扰任何与深度/模具测试相关的 GPU 性能优化。

## <a name="creating-query-heaps"></a>创建查询堆

与创建查询堆相关的 API 包括 [D3D12\_QUERY\_HEAP\_TYPE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_query_heap_type) 枚举、[D3D12\_QUERY\_HEAP\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_query_heap_desc) 结构和 [CreateQueryHeap](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createqueryheap) 方法    。

核心运行时将验证查询堆类型是否为 [D3D12\_HEAP\_TYPE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_heap_type) 枚举的有效成员，以及计数是否大于 0  。

查询堆中的每个独立查询元素都可以分别启动和停止。

使用查询堆的 API 包括 [D3D12\_QUERY\_TYPE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_query_type) 枚举以及 [BeginQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery) 和 [EndQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery) 方法    。

D3D12\_QUERY\_TYPE\_TIMESTAMP 是仅支持 [EndQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery) 的唯一查询  。 其他所有查询类型都需要 [BeginQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery) 和 EndQuery   。

调试层将验证以下内容：

-   在不结束查询（对于给定元素而言）的情况下开始两次查询是非法的。 对于需要开始和结束的查询而言，在相应的开始（对于给定元素而言）之前结束查询是非法的。
-   传递到 [BeginQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery) 的查询类型必须与传递到 [EndQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery) 的查询类型相匹配   。

核心运行时将验证以下内容：

-   无法在时间戳查询上调用 [BeginQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery)  。
-   对于支持 [BeginQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery) 和 [EndQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery)（除时间戳之外的所有）的查询类型，给定元素的查询不得跨越命令列表边界   。
-   ElementIndex 必须在范围内  。
-   查询类型是 [D3D12\_QUERY\_TYPE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_query_type) 枚举的有效成员  。
-   查询类型必须与查询堆兼容。 下表显示每个查询类型所需的查询堆类型：

    

    | 查询类型                                  | 查询堆类型                                      |
    |---------------------------------------------|------------------------------------------------------|
    | D3D12\_QUERY\_TYPE\_OCCLUSION               | D3D12\_QUERY\_TYPE\_HEAP\_TYPE\_OCCLUSION            |
    | D3D12\_QUERY\_TYPE\_BINARY\_OCCLUSION       | D3D12\_QUERY\_TYPE\_HEAP\_TYPE\_OCCLUSION            |
    | D3D12\_QUERY\_TYPE\_TIMESTAMP               | D3D12\_QUERY\_TYPE\_HEAP\_TYPE\_TIMESTAMP            |
    | D3D12\_QUERY\_TYPE\_PIPELINE\_STATISTICS    | D3D12\_QUERY\_TYPE\_HEAP\_TYPE\_PIPELINE\_STATISTICS |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM0 | D3D12\_QUERY\_TYPE\_HEAP\_TYPE\_SO\_STATISTICS       |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM1 | D3D12\_QUERY\_TYPE\_HEAP\_TYPE\_SO\_STATISTICS       |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM2 | D3D12\_QUERY\_TYPE\_HEAP\_TYPE\_SO\_STATISTICS       |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM3 | D3D12\_QUERY\_TYPE\_HEAP\_TYPE\_SO\_STATISTICS       |

    

     

-   命令列表类型支持查询类型。 下表显示哪些命令列表类型支持哪些查询。

    

    | 查询类型                                  | 支持的命令列表类型 |
    |---------------------------------------------|------------------------------|
    | D3D12\_QUERY\_TYPE\_OCCLUSION               | Direct                       |
    | D3D12\_QUERY\_TYPE\_BINARY\_OCCLUSION       | Direct                       |
    | D3D12\_QUERY\_TYPE\_TIMESTAMP               | Direct 和 Compute           |
    | D3D12\_QUERY\_TYPE\_PIPELINE\_STATISTICS    | Direct                       |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM0 | Direct                       |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM1 | Direct                       |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM2 | Direct                       |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM3 | Direct                       |

    

     

## <a name="extracting-data-from-a-query"></a>从查询中提取数据

从查询中提取数据的方法是使用 [ResolveQueryData](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvequerydata) 方法  。 **ResolveQueryData** 可使用所有堆类型（默认、上传和回读）。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> <dt>

[预测查询演练](predication-queries.md)
</dt> </dl>

 

 




