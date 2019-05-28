---
title: 示例根签名
description: 以下部分显示不同的程度不一到完全填充从空的根签名。
ms.assetid: 493D35C9-2A90-4688-BD7E-74B9EB2B4E72
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 5fba92270c0593cb07437ab91b691f5ed1e308a0
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223930"
---
# <a name="example-root-signatures"></a>示例根签名

以下部分显示不同的程度不一到完全填充从空的根签名。

-   [空的根签名](#an-empty-root-signature)
-   [一个常量](#one-constant)
-   [添加根常量缓冲区视图](#adding-a-root-constant-buffer-view)
-   [绑定描述符表](#binding-descriptor-tables)
-   [更复杂的根签名](#a-more-complex-root-signature)
-   [流式处理着色器资源视图](#streaming-shader-resource-views)
-   [相关的主题](#related-topics)

## <a name="an-empty-root-signature"></a>空的根签名

![空的根签名具有未绑定](images/root-tables-0.png)

空的根签名不太可能会很有用，但无法与使用输入装配器，并最小顶点和像素着色器未访问任何描述符的普通的呈现处理过程中使用。 Blend 阶段中，呈现器目标和深度模具阶段都会提供，甚至具有空的根签名。

## <a name="one-constant"></a>一个常量

![单个根常量](images/root-tables-constant.png)

API 绑定槽是此参数的根参数在命令列表记录时的绑定位置。 API 绑定槽数是隐式的基于根签名中参数顺序 （第一个始终为零）。 HLSL 绑定槽是着色器会显示根参数。 类型 (如上述示例中的"uint") 未知的硬件但是只需在映像中的注释，硬件将只看到单个 DWORD，作为内容。

若要将一个常量绑定在命令列表记录时，会使用类似于以下的命令：

``` syntax
pCmdList->SetComputeRoot32BitConstant(0,seed); // 0 is the parameter index, seed is used by the shaders
```

## <a name="adding-a-root-constant-buffer-view"></a>添加根常量缓冲区视图

![将常量缓冲区视图添加到根签名](images/root-tables-cbv.png)

此示例演示两个根常量和常量缓冲区视图 (CBV)，这两个 DWORD 槽的根。

若要将绑定一个常量缓冲区视图，请使用如下所示的命令。 请注意图中所示的槽的第一个参数 (2)。 通常将设置的常量数组并将其然后可供在作为 CBV b0 着色器。

``` syntax
pCmdList->SetGraphicsRootConstantBufferView(2,GPUVAForCurrDynamicConstants);
```

## <a name="binding-descriptor-tables"></a>绑定描述符表

![将描述符表添加到根签名](images/root-tables-2.png)

此示例演示如何使用两个描述符表;一个声明的表将出现在 CBV 中的执行时间的五个描述符\_SRV\_UAV 描述符堆，另一个声明的两个描述符表将会显示在取样器描述符堆中的执行时间。

若要记录的命令列表时，将绑定描述符表。

``` syntax
pCmdList->SetComputeRootDescriptorTable(1, handleToCurrentMaterialDataInHeap);
pCmdList->SetComputeRootDescriptorTable(2, handleToCurrentMaterialDataInSamplerHeap);
```

根签名的另一个功能是 float4 根常量，它是四个 DWORD 的大小。 以下命令将绑定只需四个中间两个 DWORD。

``` syntax
pCmdList->SetComputeRoot32BitConstants(0,2,myFloat2Array,1);  // 2 constants starting at offset 1 (middle 2 values in float4)
```

## <a name="a-more-complex-root-signature"></a>更复杂的根签名

![具有多个元素的复杂根签名](images/root-tables-3.png)

此示例演示使用大多数类型的条目的密集根签名。 两个 （在插槽 3 和 6） 的描述符表包括大小不受限制的数组。 应用程序只能接触到堆中的有效描述符是此处的负担。 不受限制，或非常大数组需要硬件层 2 + 资源绑定支持。

有两个静态取样器 （而不需要根签名槽绑定）。

在槽 9，UAV u4 和 UAV u5 相同描述符表偏移量位置处声明。 这是使用别名描述符的使用，在内存中的一个描述符将显示为 u4 和 u5 HLSL 着色器中。 必须在这种情况下使用 D3D10 编译着色器\_着色器\_资源\_可能\_别名选项或或`/res_may_alias`FXC 中的选项。 使用别名描述符启用一个描述符来绑定到多个绑定点，而无需对着色器进行任何更改。

## <a name="streaming-shader-resource-views"></a>流式处理着色器资源视图

![此根签名中的流式处理着色器资源视图](images/root-tables-4.png)

此根签名显示所有 SRVs 入和移出一个大型数组流式都传输的其中一个方案。 在执行时，设置根签名后，可以设置描述符表。 然后由为通过提供第一个通过几个根自变量的常量数组编制索引完成所有纹理读取。 单个描述符堆需要并且只会更新纹理流入加入或退出免费描述符槽。

着色器在常量缓冲区视图中使用常量标识描述符的偏移量大的堆中。 例如，如果着色器材料的 ID，则它可以使用常量来访问所需的描述符 （即在引用所需的纹理） 在一个大型数组中的索引。

此方案需要使用资源绑定第 2 层 + 硬件。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[资源绑定硬件层](hardware-support.md)
</dt> <dt>

[在 HLSL 中绑定的资源](resource-binding-in-hlsl.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




