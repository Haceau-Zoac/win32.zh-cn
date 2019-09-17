---
title: 直接在根签名中使用常量
description: 应用程序可在根签名中定义根常量，每个常量都是一组 32 位值。 它们按高级着色语言 (HLSL) 显示为常量缓冲区。 请注意，由于历史原因，常量缓冲区被视为 4x32 位值的集合。
ms.assetid: F9A2640F-D1FA-481C-BDF1-B15372E3C512
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 0a3cd3980bede72d6e2f4b163abe11b249970eb7
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006003"
---
# <a name="using-constants-directly-in-the-root-signature"></a>直接在根签名中使用常量

应用程序可在根签名中定义根常量，每个常量都是一组 32 位值。 它们按高级着色语言 (HLSL) 显示为常量缓冲区。 请注意，由于历史原因，常量缓冲区被视为 4x32 位值的集合。

每组用户常量被视为 32 位值的一个标量数组，可从着色器进行动态索引和只读操作。 如果超出范围地索引一组给定的根常量，会产生未定义的结果。 在 HLSL 中，可为用户常量提供数据结构定义，从而为其提供类型。 例如，如果根签名定义了一个由 4 个根常量组成的集合，则 HLSL 可在其上覆盖以下结构。

``` syntax
struct DrawConstants
{
    uint foo;
    float2 bar;
    int moo;
};
ConstantBuffer<DrawConstants> myDrawConstants : register(b1, space0);
```

由于不可在根签名空间中进行动态索引，因此不允许在映射到根常量的常量缓冲区中使用数组。 例如，不可在常量缓冲区中拥有 `float myArray[2];` 之类的条目。 映射到根常量的常量缓冲区本身不能是数组；因此，不能将 `cbuffer myCBArray[2]` 映射到根常量。

可部分设置常量。 例如，如果根签名在 RootTableBindSlot 2 上定义了四个 32 位值，则每次可设置这四个常量的任何子集（其他常量不变）。 这在继承根签名状态且可部分更改它的捆绑中非常有用。

设置常量时，请注意着色器预期的常量缓冲区布局。 例如，常量可填充到 `vec4` 边界。 要验证预期的布局，请检查 HLSL 着色器的反射信息。

以下 API（来自 [ID3D12GraphicsCommandList](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 接口）用于直接在根签名上设置常量：

-   [**SetGraphicsRoot32BitConstant**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsroot32bitconstant)
-   [**SetGraphicsRoot32BitConstants**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsroot32bitconstants)
-   [**SetComputeRoot32BitConstant**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputeroot32bitconstant)
-   [**SetComputeRoot32BitConstants**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputeroot32bitconstants)

另请参阅 [D3D12\_ROOT\_CONSTANTS](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_constants) 结构。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




