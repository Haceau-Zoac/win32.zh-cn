---
title: 查询
description: 在 Direct3D 12 中，查询进行分组的查询称为查询堆的数组。 查询堆有类型用于定义可用于该堆的查询的有效类型。
ms.assetid: d7403b5d-7e1b-4dd2-ae45-52e1153233c6
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: f3dd307f06a1b69a2903b069196adf289be0fdfa
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224233"
---
# <a name="queries"></a>查询

在 Direct3D 12 中，查询进行分组的查询称为查询堆的数组。 查询堆有类型用于定义可用于该堆的查询的有效类型。

-   [在查询中的区别 Direct3D 11 到 Direct3D 12](#differences-in-queries-from-direct3d-11-to-direct3d-12)
-   [查询堆](#query-heaps)
-   [堆创建查询](#creating-query-heaps)
-   [从查询中提取数据](#extracting-data-from-a-query)
-   [相关的主题](#related-topics)

## <a name="differences-in-queries-from-direct3d-11-to-direct3d-12"></a>在查询中的区别 Direct3D 11 到 Direct3D 12

下列查询类型不再存在在 Direct3D 12 中，它们合并到其他进程的功能：

-   **事件查询**-事件功能现在由处理界定。
-   **不连续的时间戳查询**-GPU 时钟可以设置为在 Direct3D 12 中的稳定状态 (请参阅[计时](timing.md)部分)。 GPU 时钟比较不是 GPU 空闲根本状态之间的时间戳 （称为非连续查询） 的情况下有意义的。 与发出不同的命令从稳定电源两个时间戳查询列表是可靠地比较。 同一个命令列表中的两个时间戳始终是可靠地比较。
-   **流输出统计信息查询**-Direct3D 12 中没有任何单个流输出 (SO) 溢出查询所有输出流。 应用程序需要发出多个单一流查询，然后将结果关联起来。
-   **Stream 输出统计信息谓词和封闭谓词查询**的查询 （它写入内存） 和[断言而](predication.md)不再耦合 （从内存的读取），并因此不需要这些查询类型。

到 Direct3D 12 中添加了新的二进制封闭查询类型。

## <a name="query-heaps"></a>查询堆

查询可以是一个来自多种类型 ([**D3D12\_查询\_堆\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_query_heap_type))，并且会组合到查询堆之前提交至 GPU。

新的查询类型 D3D12\_查询\_类型\_二进制\_封闭并且可像 D3D12\_查询\_类型\_封闭，不同之处返回二进制 0/1结果：0 表示没有样本传递深度和模具测试，1 表示，至少一个示例传递深度和模具测试。 这使封闭查询以不会干扰任何与深度/模具测试关联的 GPU 性能优化。

## <a name="creating-query-heaps"></a>堆创建查询

与堆创建查询相关的 Api 是枚举[ **D3D12\_查询\_堆\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_query_heap_type)，该结构[ **D3D12\_查询\_堆\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_query_heap_desc)，和方法[ **CreateQueryHeap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createqueryheap)。

核心运行时将验证查询堆类型是有效的成员[ **D3D12\_堆\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_heap_type)枚举和 count 值大于 0。

可以启动和停止单独查询堆中的每个单个查询元素。

对于堆使用的查询的 Api 是枚举[ **D3D12\_查询\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_query_type)，和方法[ **BeginQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery)并[ **EndQuery**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery)。

D3D12\_查询\_类型\_时间戳是支持的唯一查询[ **EndQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery)仅。 所有其他查询类型需要[ **BeginQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery)并**EndQuery**。

调试层将验证以下：

-   您不能两次这些操作都无需结束 （适用于给定的元素） 开始查询。 执行的查询需要同时开始和结束时，它是非法的结束之前相应开始新的 （适用于给定的元素） 的查询。
-   查询类型传递给[ **BeginQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery)必须匹配查询类型传递给[ **EndQuery**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery)。

核心运行时将验证以下：

-   [**BeginQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery)不能调用上的时间戳查询。
-   为支持这两个的查询类型[ **BeginQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery)并[ **EndQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery) （所有除外时间戳），必须针对给定元素的查询不跨命令列表边界。
-   *ElementIndex*必须在范围内。
-   查询类型是有效的成员[ **D3D12\_查询\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_query_type)枚举。
-   查询类型必须与查询堆兼容。 下表显示了每个查询类型的所需的查询堆类型：

    

    | 查询类型                                  | 查询堆类型                                      |
    |---------------------------------------------|------------------------------------------------------|
    | D3D12\_查询\_类型\_封闭               | D3D12\_查询\_类型\_堆\_类型\_封闭            |
    | D3D12\_查询\_类型\_二进制\_封闭       | D3D12\_查询\_类型\_堆\_类型\_封闭            |
    | D3D12\_查询\_类型\_时间戳               | D3D12\_查询\_类型\_堆\_类型\_时间戳            |
    | D3D12\_查询\_类型\_管道\_统计信息    | D3D12\_查询\_类型\_堆\_类型\_管道\_统计信息 |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM0 | D3D12\_查询\_类型\_堆\_类型\_因此\_统计信息       |
    | D3D12\_查询\_类型\_因此\_统计信息\_STREAM1 | D3D12\_查询\_类型\_堆\_类型\_因此\_统计信息       |
    | D3D12\_查询\_类型\_因此\_统计信息\_STREAM2 | D3D12\_查询\_类型\_堆\_类型\_因此\_统计信息       |
    | D3D12\_查询\_类型\_因此\_统计信息\_STREAM3 | D3D12\_查询\_类型\_堆\_类型\_因此\_统计信息       |

    

     

-   命令列表类型所支持的查询类型。 下表显示了哪些命令列表类型支持的查询。

    

    | 查询类型                                  | 支持的命令列表类型 |
    |---------------------------------------------|------------------------------|
    | D3D12\_查询\_类型\_封闭               | 直接                       |
    | D3D12\_查询\_类型\_二进制\_封闭       | 直接                       |
    | D3D12\_查询\_类型\_时间戳               | 直接和计算           |
    | D3D12\_查询\_类型\_管道\_统计信息    | 直接                       |
    | D3D12\_QUERY\_TYPE\_SO\_STATISTICS\_STREAM0 | 直接                       |
    | D3D12\_查询\_类型\_因此\_统计信息\_STREAM1 | 直接                       |
    | D3D12\_查询\_类型\_因此\_统计信息\_STREAM2 | 直接                       |
    | D3D12\_查询\_类型\_因此\_统计信息\_STREAM3 | 直接                       |

    

     

## <a name="extracting-data-from-a-query"></a>从查询中提取数据

若要从查询中提取数据的方法是使用[ **ResolveQueryData** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvequerydata)方法。 **ResolveQueryData**可以处理所有堆类型 （默认、 上载和 readback）。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> <dt>

[断言而查询演练](predication-queries.md)
</dt> </dl>

 

 




