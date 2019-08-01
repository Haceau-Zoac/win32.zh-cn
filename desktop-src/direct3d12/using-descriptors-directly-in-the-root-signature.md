---
title: 直接在根签名中使用描述符
description: 应用程序可以将描述符直接放在根签名中，以避免通过描述符堆。
ms.assetid: 033E3D8F-3003-42F7-BF77-68A7D62802E5
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 9ddbdaac859183b20245e7f4164a85c14f940f6a
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296454"
---
# <a name="using-descriptors-directly-in-the-root-signature"></a>直接在根签名中使用描述符

应用程序可以将描述符直接放在根签名中，以避免通过描述符堆。 这些描述符在根签名中占用了大量空间（请参阅根签名限制部分），因此应用程序应尽量少用这些描述符。

一个示例用法是，在根布局中放置每次绘制都会发生更改的 CBV，这样应用程序就不必为每次绘制分配描述符堆空间（并将描述符表保存在描述符堆的新位置）。 通过在根签名中放置一些内容，表示应用程序只是将版本控制职责交给驱动程序，但这是他们已经拥有的基础结构。

对于使用极少资源的渲染，如果所有需要的描述符都可以直接放在根签名中，则可能根本不需要使用描述符表/堆。

根签名中唯一支持的描述符类型是缓冲区区资源的 CBV 和 SRV/UAV，其中 SRV/UAV 格式仅包含 32 位 FLOAT/UINT/SINT 组件。 无需格式转换。 根中的 UAV 不能有与之关联的计数器。 根签名中的描述符每个都显示为单独的独立描述符 - 它们不能进行动态索引。

``` syntax
struct SceneData
{
   uint foo;
   float bar[2];
   int moo;
};
ConstantBuffer<SceneData> mySceneData : register(b6);
```

在上面的示例中，如 `cbuffer mySceneData[2]` 中所述，如果 `mySceneData` 要映射到根签名中的描述符，则不能声明为数组，因为根签名不支持跨描述符索引。 应用程序可以定义单独的单个常量缓冲区，并根据需要将它们定义为根签名中的单独条目。 请注意，在以上的 `mySceneData` 中，有一个数组 `bar[2]`。 常量缓冲区中的动态索引是有效的 - 根签名中的描述符的行为与通过描述符堆访问时相同的描述符的行为相同。 这与直接在根签名中使用内联常量形成了对比，根签名也像一个常量缓冲区，但有一个限制，即不允许在内联常量中进行动态索引，因此不允许使用 `bar[2]`。

以下 API（来自 [ID3D12GraphicsCommandList](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)接口）用于直接在根签名上设置描述符  ：

-   [**SetComputeRootConstantBufferView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootconstantbufferview)
-   [**SetGraphicsRootConstantBufferView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootconstantbufferview)
-   [**SetComputeRootShaderResourceView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootshaderresourceview)
-   [**SetGraphicsRootShaderResourceView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootshaderresourceview)
-   [**SetComputeRootUnorderedAccessView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootunorderedaccessview)
-   [**SetGraphicsRootUnorderedAccessView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootunorderedaccessview)

> [!Note]  
> Direct3D 12 中没有“根描述符数组”的概念。 仅在描述符堆中支持描述符数组。

 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




