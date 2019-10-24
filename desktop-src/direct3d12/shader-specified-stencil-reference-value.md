---
title: 着色器指定的模具参考值
description: 启用像素着色器来输出模具参考值，而不是使用特定于 API 的模具参考值，可以对模具操作进行非常精细的粒度控制。
ms.assetid: F58B1930-F12E-4FA4-A15C-A3C2B8705033
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 0ea50bad820b6395131f219351e493a24bf70c1b
ms.sourcegitcommit: 2f07c0e03693b7807042440bb7a7ef027c1d2a42
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/22/2019
ms.locfileid: "72776787"
---
# <a name="shader-specified-stencil-reference-value"></a>着色器指定的模具参考值

启用像素着色器来输出模具参考值，而不是使用特定于 API 的模具参考值，可以对模具操作进行非常精细的粒度控制。

模具参考值通常由[**ID3D12GraphicsCommandList：： OMSetStencilRef**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetstencilref)方法指定。 此方法根据每个绘制粒度设置模具引用值。 但是，此值可以由像素着色器覆盖。

使用此 D3D12 （和 D3D 11.3）功能，开发人员可以读取和使用从像素着色器输出的模具引用值（*SV \_StencilRef*），从而启用每像素或每样本粒度。

着色器指定的值将替换该调用的 API 指定的引用值，这意味着更改会影响模具测试，当模具操作 D3D12 \_STENCIL \_OP \_REPLACE （D3D12 的一个成员[ **\_STENCIL \_OP**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_stencil_op)）用于向模具缓冲区写入引用值。

此功能在 D3D12 和 D3D 11.3 中是可选的。 若要测试其支持，请使用[**CheckFeatureSupport**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)查看 D3D12 \_FEATURE 的*PSSpecifiedStencilRefSupported*布尔值字段[ **\_DATA \_D3D12 \_OPTIONS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) 。

下面是在像素着色器中使用*SV \_StencilRef*的示例：

``` syntax
uint main2(float4 c : COORD) : SV_StencilRef
{
    return uint(c.x);
}
```

## <a name="related-topics"></a>相关主题

<dl> <dt>

[渲染](rendering.md)
</dt> <dt>

[HLSL 中的资源绑定](resource-binding-in-hlsl.md)
</dt> <dt>

[着色器模型 5.1](https://docs.microsoft.com/windows/desktop/direct3dhlsl/shader-model-5-1)
</dt> <dt>

[在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)
</dt> </dl>

 

 




