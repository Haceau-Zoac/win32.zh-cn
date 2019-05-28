---
title: 直接在根签名中使用常量
description: 应用程序可以在根签名中，每个作为 32 位值的一组定义根常量。 它们显示在高级别着色语言 (HLSL) 为常量缓冲区。 请注意出于历史原因的常量缓冲区被视为 4 x 32 位值的集。
ms.assetid: F9A2640F-D1FA-481C-BDF1-B15372E3C512
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 81091c7224e0f177c7803c34aedc399aca318273
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224446"
---
# <a name="using-constants-directly-in-the-root-signature"></a>直接在根签名中使用常量

应用程序可以在根签名中，每个作为 32 位值的一组定义根常量。 它们显示在高级别着色语言 (HLSL) 为常量缓冲区。 请注意出于历史原因的常量缓冲区被视为 4 x 32 位值的集。

每个组的用户常量被视为 32-位值，动态可编制索引和着色器的只读的标量数组。 超出界限编制索引一组给定的根常量生成未定义的结果。 在 HLSL，可以为用户的常数为他们提供类型提供数据结构定义。 例如，如果根签名定义一组的 4 根常量，HLSL 可以覆盖其上的以下结构。

``` syntax
struct DrawConstants
{
    uint foo;
    float2 bar;
    int moo;
};
ConstantBuffer<DrawConstants> myDrawConstants : register(b1, space0);
```

获取映射到根常量，因为不支持动态索引根签名空间中的常量缓冲区中不允许使用数组。 例如，它是无效常量缓冲区的方式与在中有一个条目`float myArray[2];`。 映射到根常量的常量缓冲区本身不能为数组; 例如：因此，它是无效的映射`cbuffer myCBArray[2]`到根常量。

常量可以部分设置。 例如，如果根签名定义四个 32 位值在*RootTableBindSlot* （其他人保持不变） 一次可以设置四个常量的 2 个，则任何子集。 这可以是继承根签名状态，并且可以部分更改它的捆绑包中很有用。

设置常量时要小心的所需的着色器的常量缓冲区布局。 可以常量填充到`vec4`边界，例如。 若要验证预期的布局，请检查 HLSL 着色器的反射信息。

以下 Api (从[ **ID3D12GraphicsCommandList** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)接口) 是用于直接在根签名上设置常量：

-   [**SetGraphicsRoot32BitConstant**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsroot32bitconstant)
-   [**SetGraphicsRoot32BitConstants**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsroot32bitconstants)
-   [**SetComputeRoot32BitConstant**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputeroot32bitconstant)
-   [**SetComputeRoot32BitConstants**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputeroot32bitconstants)

另请参阅[ **D3D12\_根\_常量**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_root_constants)结构。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




