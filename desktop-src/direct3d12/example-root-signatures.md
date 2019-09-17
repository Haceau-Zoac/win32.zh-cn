---
title: 示例根签名
description: 以下部分显示了从空到完全填充的复杂程度不同的根签名。
ms.assetid: 493D35C9-2A90-4688-BD7E-74B9EB2B4E72
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 19d09d355cc1c96d77670c5536400f0b3f93c097
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006035"
---
# <a name="example-root-signatures"></a>示例根签名

以下部分显示了从空到完全填充的复杂程度不同的根签名。

-   [空根签名](#an-empty-root-signature)
-   [一个常量](#one-constant)
-   [添加根常量缓冲区视图](#adding-a-root-constant-buffer-view)
-   [绑定描述符表](#binding-descriptor-tables)
-   [更复杂的根签名](#a-more-complex-root-signature)
-   [流式传递着色器资源视图](#streaming-shader-resource-views)
-   [相关主题](#related-topics)

## <a name="an-empty-root-signature"></a>空根签名

![空根签名没有绑定](images/root-tables-0.png)

空根签名不太可能有用，但可以用于普通的呈现传递，只需使用输入装配器、最小顶点以及不访问任何描述符的像素着色器。 此外，混合阶段、呈现器目标和深度模具阶段都可用，即使具有空根签名也是如此。

## <a name="one-constant"></a>一个常量

![单个根常量](images/root-tables-constant.png)

API 绑定槽是将在记录命令列表时绑定此参数的根参数的位置。 基于根签名中的参数顺序（第一个始终为零），API 绑定槽数是隐式的。 HLSL 绑定槽是着色器会看到根参数出现的位置。 类型（上述示例中的“uint”）未被硬件识别，但只是图像中的注释，硬件只会将单个 DWORD 视为内容。

若要在记录命令列表时绑定常量，将使用类似于以下内容的命令：

``` syntax
pCmdList->SetComputeRoot32BitConstant(0,seed); // 0 is the parameter index, seed is used by the shaders
```

## <a name="adding-a-root-constant-buffer-view"></a>添加根常量缓冲区视图

![将常量缓冲区视图添加到根签名](images/root-tables-cbv.png)

此示例演示两个根常量，以及需要两个 DWORD 槽的根常量缓冲区视图 (CBV)。

若要绑定常量缓冲区视图，请使用如下命令。 请注意，第一个参数 (2) 是图中所示的槽。 通常，常量数组将进行设置，然后可作为 CBV 用于 b0 处的着色器。

``` syntax
pCmdList->SetGraphicsRootConstantBufferView(2,GPUVAForCurrDynamicConstants);
```

## <a name="binding-descriptor-tables"></a>绑定描述符表

![将描述符表添加到根签名](images/root-tables-2.png)

此示例演示如何使用两个描述符表；一个声明将在执行时可用于 CBV\_SRV\_UAV 描述符堆的表（包含五个描述符），另一个声明将在执行时显示在取样器描述符堆中的表（包含两个描述符）。

在记录命令列表时绑定描述符表。

``` syntax
pCmdList->SetComputeRootDescriptorTable(1, handleToCurrentMaterialDataInHeap);
pCmdList->SetComputeRootDescriptorTable(2, handleToCurrentMaterialDataInSamplerHeap);
```

根签名的另一个功能是 float4 根常量，其大小为四个 DWORD。 以下命令仅绑定四个 DWORD 中的中间两个。

``` syntax
pCmdList->SetComputeRoot32BitConstants(0,2,myFloat2Array,1);  // 2 constants starting at offset 1 (middle 2 values in float4)
```

## <a name="a-more-complex-root-signature"></a>更复杂的根签名

![具有多个元素的复杂根签名](images/root-tables-3.png)

此示例演示具有大多数类型的条目的密集根签名。 两个描述符表（在插槽 3 和 6 处）包括未绑定的大小数组。 对应用程序增加此处的负担，以便仅触摸堆中的有效描述符。 未绑定或非常大的数组需要资源绑定支持的硬件层 2+。

有两个静态取样器（已绑定，而无需根签名槽）。

在槽 9 处，将在相同的描述符表偏移处声明 UAV u4 和 UAV u5。 这会使用别名描述符，内存中的一个描述符将作为 u4 和 u5 显示在 HLSL 着色器中。 在这种情况下，必须使用 D3D10\_SHADER\_RESOURCES\_MAY\_ALIAS 选项或 FXC 中的 `/res_may_alias` 选项编译着色器。 别名描述符允许一个描述符绑定到多个绑定点，而无需对着色器进行任何更改。

## <a name="streaming-shader-resource-views"></a>流式传递着色器资源视图

![流式传递此根签名中的着色器资源视图](images/root-tables-4.png)

此根签名说明了所有 SRV 将流式传入和传出到一个大型数组的情况。 在执行时，可以在设置根签名后立即设置描述符表。 然后，所有纹理读取都通过常量（通过前几个根参数提供）编入索引到数组来完成。 只需单个描述符堆，并且仅在纹理流式传入或传出可用描述符槽时才进行更新。

大型堆中的描述符偏移使用常量缓冲区视图中的常量由着色器标识。 例如，如果为着色器提供了材料 ID，则可以使用常量将该着色器编入索引到一个大型数组中来访问所需的描述符（引用所需的纹理）。

此方案需要具有资源绑定层 2+ 的硬件。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[资源绑定硬件层](hardware-support.md)
</dt> <dt>

[HLSL 中的资源绑定](resource-binding-in-hlsl.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




