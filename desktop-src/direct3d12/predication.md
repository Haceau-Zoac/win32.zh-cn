---
title: 预测
description: 预测是一种功能，使 GPU 而不是 CPU 能够决定不绘制、复制或调度对象。
ms.assetid: 5c5138c7-f6e8-4646-961a-0e2312b5356b
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: a06757d14ac381da0a5c66b620c86cda1ab0a590
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006007"
---
# <a name="predication"></a>预测

预测是一种功能，使 GPU 而不是 CPU 能够决定不绘制、复制或调度对象。

-   [概述](#overview)
-   [SetPredication](#setpredication)
-   [相关主题](#related-topics)

## <a name="overview"></a>概述

预测的典型用途是封闭；如果绘制了一个边界框并对其进行了封闭，则在绘制对象本身时显然没有任何意义。 在这种情况下，对象的绘制可以“预测”，从而使 GPU 能够将其从实际渲染中删除。

与 Direct3D 11 不同，预测与查询分离，并在 Direct3D 12 中进行扩展，以使应用程序能够根据应用开发人员可能决定的任何推理（而不仅仅是封闭）来预测对象。

## <a name="setpredication"></a>SetPredication

可以基于缓冲区内的 64 位值来设置预测（请参考 [D3D12\_PREDICATION\_OP](/windows/desktop/api/d3d12/ne-d3d12-d3d12_predication_op)）。

当 GPU 执行 [SetPredication](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpredication) 命令时，它会提取缓冲区中的值快照。 缓冲区中数据的未来更改不会追溯性地影响预测状态。

如果输入参数缓冲区为 NULL，则禁用预测。

D3D12 API 中不存在预测提示，但在 Direct 和 Compute 命令列表中允许进行预测。 源缓冲区可以是任何堆类型（默认、上传、回读）。

核心运行时将验证以下内容：

-   AlignedBufferOffset 是 8 个字节的倍数
-   资源是一个缓冲区
-   操作是枚举的有效成员
-   无法从捆绑包中调用 [SetPredication](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpredication)
-   命令列表类型支持预测
-   偏移量不超过缓冲区大小

如果源缓冲区不处于 D3D12\_RESOURCE\_STATE\_VERTEX\_AND\_CONSTANT\_BUFFER 状态，则调试层将发出错误。

可以预测的操作集是：

-   [**DrawInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced)
-   [**DrawIndexedInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawindexedinstanced)
-   [**Dispatch**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch)
-   [**CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)
-   [**CopyBufferRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion)
-   [**CopyResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copyresource)
-   [**CopyTiles**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytiles)
-   [**ResolveSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvesubresource)
-   [**ClearDepthStencilView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-cleardepthstencilview)
-   [**ClearRenderTargetView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearrendertargetview)
-   [**ClearUnorderedAccessViewUint**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearunorderedaccessviewuint)
-   [**ClearUnorderedAccessViewFloat**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearunorderedaccessviewfloat)
-   [**ExecuteIndirect**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executeindirect)

[ExecuteBundle](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executebundle) 本身不可预测。 相反，上面列表中包含在捆绑包一侧的各个操作都可预测。

ID3D12GraphicsCommandList 方法 [ResolveQueryData](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvequerydata)[BeginQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery) 和 [EndQuery](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery) 不可预测。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> <dt>

[性能度量](performance-measurement.md)
</dt> <dt>

[预测查询演练](predication-queries.md)
</dt> </dl>

 

 




