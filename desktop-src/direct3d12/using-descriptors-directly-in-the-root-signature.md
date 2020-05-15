---
title: 在根签名中直接使用描述符
description: 应用程序可以将描述符直接放在根签名中，以避免通过描述符堆。
ms.assetid: 033E3D8F-3003-42F7-BF77-68A7D62802E5
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: cd97a7d68f5c9b51b6d15d3b71c6e30d04bb36e5
ms.sourcegitcommit: 83afb2f3e9e5d37f3f5a11e975515e9ed8b044ff
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/15/2020
ms.locfileid: "83407098"
---
# <a name="using-descriptors-directly-in-the-root-signature"></a>在根签名中直接使用描述符

应用程序可以将描述符直接放在根签名中，以避免通过描述符堆。 这些描述符在根签名中占用了大量空间（请参阅根签名限制部分），因此应用程序应尽量少用这些描述符。

示例用法是将每个绘图中正在更改的常量缓冲区视图（CBV）放在根布局中，以便每个绘图无需由应用程序分配描述符堆空间（并在描述符堆中的新位置保存指向描述符表）。 通过在根签名中放置一些内容，表示应用程序只是将版本控制职责交给驱动程序，但这是他们已经拥有的基础结构。

对于使用极少资源的呈现，可能根本不需要描述符表/堆使用，前提是所有所需的说明符都可以直接放置在根签名中。

根签名中支持的唯一说明符类型为：
- CBVs.
- 着色器资源视图（SRVs）/Unordered 访问视图（Uav），其中 SRV/UAV 格式只包含32位 FLOAT/UINT/圣马丁组件（没有格式转换）。
- SRVs raytracing 加速度结构，位于本地或全局根签名中。 

根中的 UAV 不能有与之关联的计数器。 根签名中的描述符显示为单个单独的描述符， &mdash; 它们不能动态索引。

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

以下 API（来自 [ID3D12GraphicsCommandList](/windows/win32/api/d3d12/nn-d3d12-id3d12graphicscommandlist)接口）用于直接在根签名上设置描述符****：

-   [**SetComputeRootConstantBufferView**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootconstantbufferview)
-   [**SetGraphicsRootConstantBufferView**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootconstantbufferview)
-   [**SetComputeRootShaderResourceView**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootshaderresourceview)
-   [**SetGraphicsRootShaderResourceView**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootshaderresourceview)
-   [**SetComputeRootUnorderedAccessView**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootunorderedaccessview)
-   [**SetGraphicsRootUnorderedAccessView**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootunorderedaccessview)

> [!Note]  
> Direct3D 12 中没有“根描述符数组”的概念。 仅在描述符堆中支持描述符数组。

## <a name="related-topics"></a>相关主题

[根签名](root-signatures.md)
