---
title: Stream 输出计数器
description: Stream 输出是 GPU 能够写入缓冲区的顶点。 流输出计数器监视进度。
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
# <a name="stream-output-counters"></a>Stream 输出计数器

Stream 输出是 GPU 能够写入缓冲区的顶点。 流输出计数器监视进度。

-   [从 Direct3D Stream 计数器之间的差异到 Direct3D 11 12](#differences-in-stream-counters-from-direct3d-11-to-direct3d-12)
-   [BufferFilledSize](#bufferfilledsize)
-   [相关的主题](#related-topics)

## <a name="differences-in-stream-counters-from-direct3d-11-to-direct3d-12"></a>从 Direct3D Stream 计数器之间的差异到 Direct3D 11 12

流输出过程的一部分，GPU 必须知道它正在写入的缓冲区中的当前位置。 在 Direct3D 11 中，内存来存储此位置分配驱动程序和应用程序来处理此值的唯一方法是通过**SOSetTargets**方法。 在 Direct3D 12 中，应用程序分配内存来存储此当前的位置。 有没有特殊的方法来操作此值，并应用可以自由地读/写的值与 CPU 或 GPU。

## <a name="bufferfilledsize"></a>BufferFilledSize

应用程序负责，来为 32 位数量分配存储名为*BufferFilledSize*。 这包含流输出缓冲区中数据的字节数。 此存储可以置于相同，或不同，资源作为一个包含流输出数据。 此值是由 GPU 访问流输出阶段，以确定追加新的顶点数据缓冲区中的位置。 此外，此值是由 GPU 来确定发生溢出时访问。

结构是指[ **D3D12\_流\_输出\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_stream_output_desc)。

调试层验证在以下[ **ID3D12GraphicsCommandList::SOSetTargets**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-sosettargets):

-   *BufferFilledSize*隐含的范围内 {*OffsetInBytes*， *SizeInBytes*}，如果指定了非 NULL 资源。
-   *BufferFilledSizeOffsetInBytes*是 4 的倍数。
-   *BufferFilledSizeOffsetInBytes*包含资源的范围内。
-   指定的资源是的缓冲区。

运行时将不会验证与流输出缓冲区关联的堆类型，如所有堆类型中支持流输出。

根签名必须指定是否流输出将使用，通过使用[ **D3D12\_根\_签名\_标志**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_root_signature_flags)标志。

D3D12\_根\_签名\_标志\_允许\_流\_输出可以为指定根签名在 HLSL 中, 编写类似的方式对如何指定其他标志。

[**CreateGraphicsPipelineState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-creategraphicspipelinestate)几何着色器包含流输出但没有 D3D12 根签名。 如果将失败\_根\_签名\_标志\_允许\_流\_输出标志设置。

使用的资源时为流输出目标使用资源，必须在 D3D12\_资源\_状态\_流\_出状态。 这适用于这两个顶点数据和*BufferFilledSize* （可以是中的相同或不同的资源）。

没有特殊的 api 来设置流输出缓冲区偏移量，因为应用程序可以写入*BufferFilledSize* CPU 或 GPU 直接。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> </dl>

 

 




