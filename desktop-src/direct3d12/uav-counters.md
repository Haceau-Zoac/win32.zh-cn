---
title: UAV 计数器
description: UAV 计数器可用于将 32 位原子计数器与无序访问视图 (UAV) 关联。
ms.assetid: 0B77E238-E8CF-466B-9188-3DE96AF97F42
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 4e10130b3aeb006b6a0193a42a29b6b9388b8462
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223918"
---
# <a name="uav-counters"></a>UAV 计数器

UAV 计数器可用于将 32 位原子计数器与无序访问视图 (UAV) 关联。

-   [从 Direct3D 11 到 Direct3D 12 的 UAV 计数器的差异](#differences-in-uav-counters-from-direct3d-11-to-direct3d-12)
-   [使用 UAV 计数器](#using-uav-counters)
-   [相关主题](#related-topics)

## <a name="differences-in-uav-counters-from-direct3d-11-to-direct3d-12"></a>从 Direct3D 11 到 Direct3D 12 的 UAV 计数器的差异

在 Direct3D 12 中，应用使用与 Direct3D 11 相同的 HLSL 着色器函数来访问 UAV 计数器：

-   **IncrementCounter**
-   **DecrementCounter**
-   **Append**
-   **Consume**

除着色器之外，Direct3D 11 还使用 API 方法访问计数器，而在 Direct3D 12 中，32 位值是由应用分配的，因此 CPU 或 GPU 可以读写 32 位值（类似于其他 Direct3D 12 资源）。

## <a name="using-uav-counters"></a>使用 UAV 计数器

应用负责分配 UAV 计数器的 32 位存储。 此存储可分配在不同的资源中，类似于包含可通过 UAV 访问的数据的存储。

请参阅 [CreateUnorderedAccessView](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createunorderedaccessview)、[D3D12\_BUFFER\_UAV\_FLAGS](/windows/desktop/api/D3D12/ne-d3d12-d3d12_buffer_uav_flags) 和 [D3D12\_BUFFER\_UAV](/windows/desktop/api/D3D12/ns-d3d12-d3d12_buffer_uav)    。

如果在 [CreateUnorderedAccessView](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createunorderedaccessview) 调用中指定了 pCounterResource，则会有一个与 UAV 关联的计数器   。 在此情况下：

-   StructureByteStride 必须大于零 
-   格式必须为 DXGI\_FORMAT\_UNKNOWN
-   不得设置 RAW 标志
-   两个资源必须是缓冲区
-   CounterOffsetInBytes 必须是 4 字节的倍数 
-   CounterOffsetInBytes 必须在计数器资源范围内 
-   pDesc 不能为 NULL 
-   pResource 不能为 NULL 

注意以下用例：

-   如果未指定 pCounterResource，则 CounterOffsetInBytes 必须为 0   。
-   如果已设置 RAW 标志，则格式必须为 DXGI\_\_R32\_TYPELESS 且 UAV 资源必须是缓冲区。
-   如果未设置 pCounterResource，则 CounterOffsetInBytes 必须为 0   。
-   如果未设置 RAW 标志且 StructureByteStride = 0，则格式必须为有效的 UAV 格式  。

Direct3D 12 消除了追加和计数器 UAV 之间的区别（尽管 HLSL 字节码中这种区别仍然存在）。

核心运行时将验证 [CreateUnorderedAccessView](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createunorderedaccessview) 中的这些限制  。

绘制/调度期间，计数器资源必须处于 [D3D12\_RESOURCE\_STATE\_UNORDERED\_ACCESS](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states) 状态  。 此外，在单个绘制/调度调用中，应用程序通过两个独立的 UAV 计数器访问同一 32 位内存位置是无效的。 如果检测到以上任一状况，调试层将发出错误。

由于应用只需直接将数据复制到计数器值以及从计数器值复制数据，因此没有“SetUnorderedAccessViewCounterValue”或“copystructu”方法。

支持使用计数器动态编制 UAV 索引。

如果着色器试图访问的 UAV 计数器没有关联的计数器，则调试层将发出警告，并将产生导致删除应用设备的 GPU 页面错误。

UAV 计数器支持所有堆类型（默认、上传、回读）。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> </dl>

 

 




