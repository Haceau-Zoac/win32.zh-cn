---
title: 使用根签名
description: 根签名是描述符表（包括其布局）、根常量和根描述符的任意排列集合的定义。
ms.assetid: 0131CDE4-02DB-440A-92B8-B0933C6414B0
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 3babe26dc06d4f85ce3d6fb771e18c78b54a3701
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71005721"
---
# <a name="using-a-root-signature"></a>使用根签名

根签名是描述符表（包括其布局）、根常量和根描述符的任意排列集合的定义。 每个条目都有一个接近最大限制的成本，因此应用程序可以在根签名将包含的每种类型条目的数量之间进行权衡。

-   [命令列表语义](#command-list-semantic)
-   [捆绑语义](#bundle-semantics)
-   [相关主题](#related-topics)

根签名是可在 API 中通过手动规范创建的对象。 PSO 中的所有着色器都必须与使用 PSO 指定的根布局兼容，否则各个着色器必须包括相互匹配的嵌入式根布局；否则，PSO 创建将失败。 根签名的一个属性是在编写时着色器无需对其有所了解，尽管根签名也可以直接在着色器中编写（如果需要）。 现有着色器资产不需要任何更改即可与根签名兼容。 引入了着色器模型 5.1 以提供一些额外的灵活性（着色器中的描述符的动态索引），并且可以根据需要从现有着色器资产开始以递增方式采用。

## <a name="command-list-semantic"></a>命令列表语义

在命令列表的开头，根签名是未定义的。 图形着色器具有与计算着色器不同的根签名，每个签名单独分配在命令列表中。 命令列表或捆绑中设置的根签名也必须在绘制/调度时与当前设置的 PSO 匹配；否则，行为是未定义的。 绘制/调度之前的瞬态根签名不匹配项是正常的，例如，在切换到兼容的根签名之前设置不兼容的 PSO（只要在调用绘制/调度时这些是兼容的）。 设置 PSO 不会更改根签名。 应用程序必须调用用于设置根签名的专用 API。

在命令列表中设置根签名后，布局会定义应用程序预计提供的绑定集，以及哪些 PSO（即使用相同布局编译的 PSO）可用于接下来的绘制/调度调用。 例如，根签名可由应用程序定义为具有以下条目。 每个条目称为“槽”。

-   \[0\] CBV 描述符内联（根描述符）
-   \[1\] 包含 2 个 SRV、1 个 CBV 和 1 个 UAV 的描述符表
-   \[2\] 包含 1 个采样器的描述符表
-   \[3\] 根常量的 4 x 32 位集合
-   \[4\] 包含未指定数量的 SRV 的描述符表

在这种情况下，在能够发出绘制/调度之前，应用程序预计会将相应的绑定设置为应用程序使用其当前根签名定义的每个槽 \[0..4\]。 例如，在槽 \[1\] 中，必须绑定描述符表，这是包含（或在执行时将包含）2 个 SRV、1 个 CBV 和 1 个 UAV 的描述符堆中的相连区域。 同样，必须在槽 \[2\] 和 \[4\] 中设置描述符表。

应用程序一次可以更改根签名绑定的一部分（其余部分保持不变）。 例如，如果只需在绘制之间更改的是槽 \[2\] 中的常量之一，即需要重新绑定所有应用程序。 如前文所述，所有驱动程序/硬件版本均为根签名绑定状态，因为它自动得到修改。 如果已在命令列表中更改根签名，则所有以前的根签名绑定会变得过时，并且必须在绘制/调度之前设置所有新预期的绑定；否则，行为是未定义的。 如果根签名重复设置为当前设置的同一个签名，则现有根签名绑定不会变得过时。

## <a name="bundle-semantics"></a>捆绑语义

捆绑继承命令列表的根签名绑定（绑定到上述命令列表示例中的各种槽）。 如果绑定需要更改某些继承的根签名绑定，则必须先设置要与调用的命令列表（继承的绑定不会变得过时）相同的根签名。 如果绑定将根签名设置为不同于调用的命令列表，则具有与更改上述命令列表中的根签名相同的效果：所有以前的根签名绑定均已过时，必须在绘制/调度之前设置新预期的绑定；否则，行为是未定义的。 如果绑定不需要更改任何根签名绑定，则无需设置根签名。

以下代码演示进入捆绑的调用流程示例。

``` syntax
// Command List
...
pCmdList->SetGraphicsRootSignature(pRootSig); // new parameter space
MyEngine_SetTextures(); // bundle inherits descriptor table setting
MyEngine_SetAnimationFactor(fTime); // bundle inherits root constant
pCmdList->ExecuteBundle(...);
...
// Bundle
pBundle->SetGraphicsRootSignature(pRootSig); // same as caller, in order to inherits bindings
pBundle->SetPipelineState(pPS); 
pBundle->SetGraphicsRoot32BitConstant(drawConstantsSlot,0,drawIDOffset);
pBundle->Draw(...); // using inherited textures / animation factor
pBundle->SetGraphicsRoot32BitConstant(drawConstantsSlot,1,drawIDOffset);
pBundle->Draw(...);
...
```

捆绑完成执行后，由捆绑进行的任何根布局更改和/或绑定更改继承回调用的命令列表。

有关继承的详细信息，请参阅[在 Direct3D 12 中管理图形管道状态](managing-graphics-pipeline-state-in-direct3d-12.md)的**图形管道状态继承**部分。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




