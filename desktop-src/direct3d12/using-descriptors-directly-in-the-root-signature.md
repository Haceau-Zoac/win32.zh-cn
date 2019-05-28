---
title: 根签名中直接使用描述符
description: 应用程序可以将描述符直接置于根签名，以避免无需经历描述符堆。
ms.assetid: 033E3D8F-3003-42F7-BF77-68A7D62802E5
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 63ceafcff711d89e9b3e43c1050961225697bdb1
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224302"
---
# <a name="using-descriptors-directly-in-the-root-signature"></a>根签名中直接使用描述符

应用程序可以将描述符直接置于根签名，以避免无需经历描述符堆。 这些描述符会占用大量空间中的根签名 （请参阅根签名限制部分），因此，应用程序需要尽量少用。

示例用法是将放 CBV，更改每个根布局中的绘图，以便描述符堆空间无需分配的每个绘图应用程序 （和保存指向描述符堆中的新位置的描述符表）。 将内容放在根签名，该应用程序只处理版本控制负责向驱动程序，但这是他们已有的基础结构。

用于呈现使用极少的资源、 描述符表/堆使用可能不需要在所有如果可以直接在根签名中放置所需的所有描述符。

在根签名中受支持的描述符的唯一类型是 CBVs 和 SRV/Uav 的缓冲区资源，其中 SRV/UAV 格式包含唯一的 32 位 FLOAT/UINT/圣组件。 没有格式的转换。 Uav 根目录中的不能有与之关联的计数器。 根签名中的描述符将显示每个作为单个单独描述符-不能动态地对它们进行索引。

``` syntax
struct SceneData
{
   uint foo;
   float bar[2];
   int moo;
};
ConstantBuffer<SceneData> mySceneData : register(b6);
```

在上述示例中，`mySceneData`不能声明为一个数组，如`cbuffer mySceneData[2]`如果它要映射到根签名中的描述符，因为跨描述符索引中不支持根签名。 应用程序可以定义单独的单个常量缓冲区，并根据需要定义它们分别为根签名中的单独条目。 请注意，在`mySceneData`更高版本，没有数组`bar[2]`。 常量缓冲区中的动态索引无效 – 根签名中的描述符的行为就像相同描述符的表现如果访问通过描述符堆。 这是与直接在根签名，也会出现像常量缓冲区，但具有约束动态索引中的内联常量，不允许中的内联常量相比，`bar[2]`不允许存在。

以下 Api (从[ **ID3D12GraphicsCommandList** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)接口) 是用于直接在根签名上设置描述符：

-   [**SetComputeRootConstantBufferView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootconstantbufferview)
-   [**SetGraphicsRootConstantBufferView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootconstantbufferview)
-   [**SetComputeRootShaderResourceView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootshaderresourceview)
-   [**SetGraphicsRootShaderResourceView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootshaderresourceview)
-   [**SetComputeRootUnorderedAccessView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootunorderedaccessview)
-   [**SetGraphicsRootUnorderedAccessView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootunorderedaccessview)

> [!Note]  
> 不存在"根描述符中的数组"Direct3D 12 的概念。 描述符数组是仅支持在描述符堆。

 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




