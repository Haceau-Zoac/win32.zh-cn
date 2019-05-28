---
title: 断言而
description: 断言而是一项功能，使 GPU 而不是 CPU 来确定不绘制、 复制或调度对象。
ms.assetid: 5c5138c7-f6e8-4646-961a-0e2312b5356b
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 36e9cb3d3dccce7aed2aa11856b67682d7d5991e
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224242"
---
# <a name="predication"></a>断言而

断言而是一项功能，使 GPU 而不是 CPU 来确定不绘制、 复制或调度对象。

-   [概述](#overview)
-   [SetPredication](#setpredication)
-   [相关的主题](#related-topics)

## <a name="overview"></a>概述

断言而的典型用途是与封闭;如果边界框绘制，并且封闭的像素，，是显然没有必要再绘制对象本身。 在这种情况下绘制的对象可以是"取决于所设计"，启用 gpu 的实际呈现从其删除。

与 Direct3D 11 断言而脱离了查询，以及在 Direct3D 12 中启用对基于应用程序开发人员可以决定 （而不仅仅是封闭） 任何推理的谓词对象的应用程序中扩展。

## <a name="setpredication"></a>SetPredication

断言而可以基于设置的缓冲区中的 64 位值 (请参阅[ **D3D12\_断言而\_OP**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_predication_op))。

当执行 GPU [ **SetPredication** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpredication)命令在缓冲区中，它会对齐值。 将来对缓冲区中的数据的更改不以追溯方式影响断言而状态。

如果输入的参数缓冲区为 NULL，则会禁用断言而。

断言而提示中不存在 D3D12 API，并允许断言而上直接，并在计算命令列表。 源缓冲区可以是任何堆类型 （默认值上, 传，readback）。

核心运行时将验证以下：

-   *AlignedBufferOffset*是 8 个字节的倍数
-   资源是一个缓冲区
-   该操作是枚举的有效成员
-   [**SetPredication** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpredication)捆绑包中不能从调用
-   命令列表类型支持断言而
-   偏移量不超出缓冲区大小

调试层将发出错误，如果源缓冲区不在 D3D12\_资源\_状态\_顶点\_AND\_常量\_缓冲区状态。

一组操作可以断定其是：

-   [**DrawInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced)
-   [**DrawIndexedInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawindexedinstanced)
-   [**调度**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch)
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

[**ExecuteBundle** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executebundle)不断言本身。 相反，一端捆绑包中包含从上面的列表中的各个操作都取决于所设计。

ID3D12GraphicsCommandList 方法[ **ResolveQueryData**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvequerydata)， [ **BeginQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery)并[ **EndQuery** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery)不断定。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> <dt>

[性能度量](performance-measurement.md)
</dt> <dt>

[断言而查询演练](predication-queries.md)
</dt> </dl>

 

 




