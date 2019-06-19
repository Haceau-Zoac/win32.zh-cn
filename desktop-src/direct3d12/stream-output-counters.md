---
title: 流输出计数器
description: 流输出是 GPU 将顶点写入缓冲区的能力。 流输出计数器用于监视进度。
ms.assetid: 7342DA09-25E9-4154-83BA-B03ADBB8B671
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 17c3201d118eb2dda7b3ab94862c6e857b9533a0
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224089"
---
# <a name="stream-output-counters"></a>流输出计数器

流输出是 GPU 将顶点写入缓冲区的能力。 流输出计数器用于监视进度。

-   [Direct3D 11 和 Direct3D 12 中的流计数器的差异](#differences-in-stream-counters-from-direct3d-11-to-direct3d-12)
-   [BufferFilledSize](#bufferfilledsize)
-   [相关主题](#related-topics)

## <a name="differences-in-stream-counters-from-direct3d-11-to-direct3d-12"></a>Direct3D 11 和 Direct3D 12 中的流计数器的差异

在流输出过程中，GPU 必须知道它将写入的缓冲区中的最新位置。 在 Direct3D 11 中，存储此位置的内存由驱动程序分配，应用程序处理此值的唯一方法就是使用“SOSetTargets”方法  。 在 Direct3D 12 中，应用会分配内存来存储此最新位置。 没有用于处理此值的特殊方法，应用可以使用 CPU 或 GPU 自由读取/写入值。

## <a name="bufferfilledsize"></a>BufferFilledSize

应用程序负责为称为 BufferFilledSize 的 32 位数量分配存储  。 这其中包含流输出缓冲区中的数据字节数。 此存储可以放在包含流输出数据的资源中，也可以放在其他资源中。 GPU 会在流输出阶段访问该值以确定在缓冲区中添加新顶点数据的位置。 此外，GPU 还会访问此值以确定发生溢出的时间。

请参阅结构 [D3D12\_STREAM\_OUTPUT\_DESC](/windows/desktop/api/D3D12/ns-d3d12-d3d12_stream_output_desc)  。

调试层将验证 [ID3D12GraphicsCommandList::SOSetTargets](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-sosettargets) 中的以下内容  ：

-   BufferFilledSize 属于{OffsetInBytes 、SizeInBytes} 指示的范围（如果指定了非 NULL 资源）    。
-   BufferFilledSizeOffsetInBytes 是 4 的倍数  。
-   BufferFilledSizeOffsetInBytes 在包含资源的范围内  。
-   指定的资源是缓冲区。

运行时不会验证与流输出缓冲区关联的堆类型，因为所有堆类型都支持流输出。

根签名必须使用 [D3D12\_ROOT\_SIGNATURE\_FLAGS](/windows/desktop/api/D3D12/ne-d3d12-d3d12_root_signature_flags) 标志来指定是否使用流输出  。

可以为 HLSL 中创建的根签名指定 D3D12\_ROOT\_SIGNATURE\_FLAG\_ALLOW\_STREAM\_OUTPUT，指定方式与指定其他标志的方式类似。

如果几何着色器包含流输出，但根签名没有 D3D12\_ROOT\_SIGNATURE\_FLAG\_ALLOW\_STREAM\_OUTPUT 标志，则 [CreateGraphicsPipelineState](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-creategraphicspipelinestate) 将失败  。

当将资源用作流输出目标时，所使用的资源必须处于 D3D12\_RESOURCE\_STATE\_STREAM\_OUT 状态。 这适用于顶点数据和 BufferFilledSize（可以在相同或不同的资源中）  。

没有用于设置流输出缓冲区偏移量的特殊 API，因为应用程序可以直接使用 CPU 或 GPU 写入 BufferFilledSize  。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> </dl>

 

 




