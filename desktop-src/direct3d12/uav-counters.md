---
title: UAV 计数器
description: 可以使用无序访问视图（UAV）计数器将32位原子计数器与无序访问视图（UAV）相关联。
ms.assetid: 0B77E238-E8CF-466B-9188-3DE96AF97F42
ms.localizationpriority: high
ms.topic: article
ms.date: 02/10/2020
ms.openlocfilehash: 94bc1492e3b984d96c76788430d2e63c0672ca76
ms.sourcegitcommit: 927b9c371f75f52b8011483edf3a4ba37d11ebe4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/11/2020
ms.locfileid: "77128116"
---
# <a name="uav-counters"></a>UAV 计数器
可以使用无序访问视图（UAV）计数器将32位原子计数器与无序访问视图（UAV）相关联。

## <a name="differences-in-uav-counters-from-direct3d-11-to-direct3d-12"></a>从 Direct3D 11 到 Direct3D 12 的 UAV 计数器的差异
Direct3D 12 应用和 Direct3D 11 应用都使用相同的高级着色器语言（HLSL）着色器函数来访问 UAV 计数器。

-   **IncrementCounter**
-   **DecrementCounter**
-   **Append**
-   **Consume**

### <a name="direct3d-12"></a>Direct3D 12
在 Direct3D 12 中，32位值由应用程序分配，因此 CPU 或 GPU 可以读取和写入32位值，就像其他任何 Direct3D 12 资源一样。

### <a name="direct3d-11"></a>Direct3D 11
在着色器外，使用 Direct3D 11 时，需要调用 API 方法才能访问计数器（例如， [ID3D11DeviceContext：： CopyStructureCount](/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-copystructurecount)）。

## <a name="using-uav-counters"></a>使用 UAV 计数器
应用负责为 UAV 计数器分配32位的存储。 此存储可分配在不同的资源中，类似于包含可通过 UAV 访问的数据的存储。

请参阅 [CreateUnorderedAccessView](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createunorderedaccessview)、[D3D12\_BUFFER\_UAV\_FLAGS](/windows/desktop/api/d3d12/ne-d3d12-d3d12_buffer_uav_flags) 和 [D3D12\_BUFFER\_UAV](/windows/desktop/api/d3d12/ns-d3d12-d3d12_buffer_uav)。

如果在 [CreateUnorderedAccessView](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createunorderedaccessview) 调用中指定了 pCounterResource，则会有一个与 UAV 关联的计数器。 在此情况下：

-   StructureByteStride 必须大于零
-   格式必须为 DXGI\_FORMAT\_UNKNOWN
-   不得设置 RAW 标志
-   两个资源必须是缓冲区
-   CounterOffsetInBytes 必须是 4 字节的倍数
-   CounterOffsetInBytes 必须在计数器资源范围内
-   pDesc 不能为 NULL
-   pResource 不能为 NULL

注意以下用例：

-   如果未指定 pCounterResource，则 CounterOffsetInBytes 必须为 0。
-   如果已设置 RAW 标志，则格式必须为 DXGI\_\_R32\_TYPELESS 且 UAV 资源必须是缓冲区。
-   如果未设置 pCounterResource，则 CounterOffsetInBytes 必须为 0。
-   如果未设置 RAW 标志且 StructureByteStride = 0，则格式必须为有效的 UAV 格式。

Direct3D 12 消除了追加和计数器 UAV 之间的区别（尽管 HLSL 字节码中这种区别仍然存在）。

核心运行时将验证 [CreateUnorderedAccessView](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createunorderedaccessview) 中的这些限制。

绘制/调度期间，计数器资源必须处于 [D3D12\_RESOURCE\_STATE\_UNORDERED\_ACCESS](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_states) 状态。 此外，在单个绘制/调度调用中，应用程序通过两个独立的 UAV 计数器访问同一 32 位内存位置是无效的。 如果检测到以上任一状况，调试层将发出错误。

由于应用只需直接将数据复制到计数器值以及从计数器值复制数据，因此没有“SetUnorderedAccessViewCounterValue”或“copystructu”方法。

支持使用计数器动态编制 UAV 索引。

如果着色器试图访问的 UAV 计数器没有关联的计数器，则调试层将发出警告，并将产生导致删除应用设备的 GPU 页面错误。

UAV 计数器支持所有堆类型（默认、上传、回读）。

## <a name="related-topics"></a>相关主题

* [计数器和查询](counters-and-queries.md)