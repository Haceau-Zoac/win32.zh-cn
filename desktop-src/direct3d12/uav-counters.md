---
title: UAV 计数器
description: UAV 计数器可用来将 32 位原子计数器与无序访问的视图 (UAV) 相关联。
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

UAV 计数器可用来将 32 位原子计数器与无序访问的视图 (UAV) 相关联。

-   [从 Direct3D UAV 计数器之间的差异到 Direct3D 11 12](#differences-in-uav-counters-from-direct3d-11-to-direct3d-12)
-   [使用 UAV 计数器](#using-uav-counters)
-   [相关的主题](#related-topics)

## <a name="differences-in-uav-counters-from-direct3d-11-to-direct3d-12"></a>从 Direct3D UAV 计数器之间的差异到 Direct3D 11 12

在 Direct3D 12 应用使用相同的 HLSL 着色器功能作为 Direct3D 11 访问 UAV 计数器：

-   **IncrementCounter**
-   **DecrementCounter**
-   **Append**
-   **使用**

着色器之外 Direct3D 11 使用 API 方法来访问计数器，在 Direct3D 12 中的 32 位值分配应用，以便可以读取和写入到由 CPU 还是 GPU，就像任何其他 Direct3D 12 资源的 32 位值。

## <a name="using-uav-counters"></a>使用 UAV 计数器

应用负责 UAV 计数器分配的存储空间的 32 位。 可以将此存储在不同的资源分配与包含通过 UAV 可访问的数据。

请参阅[ **CreateUnorderedAccessView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createunorderedaccessview)， [ **D3D12\_缓冲区\_UAV\_标志**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_buffer_uav_flags)并[ **D3D12\_缓冲区\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_buffer_uav)。

如果*pCounterResource*调用中指定[ **CreateUnorderedAccessView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createunorderedaccessview)，则没有与 UAV 关联的计数器。 在这种情况下：

-   *StructureByteStride*必须是大于零
-   格式必须为 DXGI\_格式\_未知
-   不能设置原始标志
-   这两个资源必须是缓冲区
-   *CounterOffsetInBytes*必须是 4 个字节的倍数
-   *CounterOffsetInBytes*必须在计数器资源的范围内
-   *pDesc*不能为 NULL
-   *pResource*不能为 NULL

并记下以下用例：

-   如果*pCounterResource*未指定，则*CounterOffsetInBytes*必须为 0。
-   如果原始标志设置，则格式必须为 DXGI\_格式\_R32\_TYPELESS 和 UAV 资源必须是一个缓冲区。
-   如果*pCounterResource*未设置，然后*CounterOffsetInBytes*必须为 0。
-   如果未设置原始标志并*StructureByteStride* = 0，则格式必须为有效的 UAV 格式。

Direct3D 12 中删除追加和计数器 Uav （尽管两者的区别仍存在于 HLSL 字节码） 之间的区别。

核心运行时将验证这些限制的内部[ **CreateUnorderedAccessView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createunorderedaccessview)。

绘制/调度过程计数器资源必须是处于状态[ **D3D12\_资源\_状态\_无序列表\_访问**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)。 此外，在一个绘制/调度调用，是无效的应用程序访问两个单独的 UAV 计数器通过相同的 32 位内存位置。 如果检测到任何一种方法时，调试层将发出错误。

因为应用程序可以只需将数据复制到和从计数器值直接没有"SetUnorderedAccessViewCounterValue"或"CopyStructureCount"方法。

支持与计数器动态索引 Uav。

如果着色器尝试访问不具有关联的计数器 UAV 的计数器，然后调试层将发出警告，并且 GPU 页面错误会导致要删除的应用的设备。

所有堆类型 （默认值上, 传，readback） 都支持 UAV 计数器。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> </dl>

 

 




