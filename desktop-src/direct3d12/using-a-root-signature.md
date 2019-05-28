---
title: 使用根签名
description: 根签名是随意排列的描述符表 （包括其布局），集合的定义根常量和根描述符。
ms.assetid: 0131CDE4-02DB-440A-92B8-B0933C6414B0
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 982052e360821726969630cf15d47f7993505931
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224263"
---
# <a name="using-a-root-signature"></a>使用根签名

根签名是随意排列的描述符表 （包括其布局），集合的定义根常量和根描述符。 每个条目具有针对最大限制，成本，因此应用程序可以占用较根签名将包含多少个项的每个类型之间的平衡。

-   [命令列表语义](#command-list-semantic)
-   [捆绑包语义](#bundle-semantics)
-   [相关的主题](#related-topics)

根签名是通过手动规范的 api，可以创建的对象。 必须是兼容 PSO，使用指定的根布局的 pso 的步骤中的所有着色器，否则单个着色器必须包括相互; 匹配的嵌入的根布局否则，PSO 创建将失败。 根签名的一个属性是着色器无需了解它时编写的虽然也根签名创作直接在着色器中，如果所需的。 着色器的现有资产不需要为与根签名兼容的任何更改。 着色器模型 5.1 引入为了提供一些额外的灵活性 （动态索引的描述符从着色器中），并可从现有的着色器资产，根据需要开始增量采用。

## <a name="command-list-semantic"></a>命令列表语义

在命令列表的开头，根签名是未定义。 图形着色器具有来自计算着色器的单独根签名，每个单独分配上命令列表。 根签名设置上命令列表或捆绑包也必须匹配当前设置在绘制/调度; PSO否则，该行为不确定。 暂时性根签名不匹配项前绘制/调度是没问题-例如，切换到兼容的根签名 （只要它们是兼容调用绘制/调度时） 之前设置不兼容的 PSO。 设置 pso 的步骤不会更改根签名。 应用程序必须调用专用的 API 来设置根签名。

一旦对命令列表设置了根签名，布局定义的绑定的应用程序应该提供，并可以是 Pso 集 （即具有相同布局编译） 用于下一步的绘制/调度调用。 例如，可能由应用程序具有以下项定义的根签名。 每一项称为"槽"。

-   \[0\] CBV 描述符内联 （根描述符）
-   \[1\]包含 2 个 SRVs、 1 CBVs 和 1 UAV 描述符表
-   \[2\]描述符表包含 1 采样器
-   \[3\]根常量的 4 x 32 位集合
-   \[4\]包含 SRVs 未指定的数量的描述符表

在这种情况下，然后才能发出绘制/调度，应用程序应该将适当的绑定设置为每个插槽\[0..4\]用其当前的根签名定义应用程序。 例如，在槽\[1\]，必须绑定描述符表，这是包含 （或将包含在执行时） 2 SRVs、 1 CBVs 和 1 UAV 描述符堆中的相连区域。 同样，必须在槽中设置描述符表\[2\]并\[4\]。

应用程序可以 （其余部分保持不变） 一次更改根签名绑定的一部分。 例如，如果需要更改之间的唯一事情绘制是在槽常量之一\[2\]，即所有应用程序需要重新绑定。 如前文所述，所有根签名的驱动程序/硬件版本将自动得到修改状态。 如果根签名更改上命令列表中，所有以前的根签名绑定会变得陈旧和绘制/调度; 之前，必须设置所有的新预期的绑定否则，该行为不确定。 如果根签名冗余设置为相同的其中一个当前设置，现有根签名绑定不会过时。

## <a name="bundle-semantics"></a>捆绑包语义

捆绑包继承命令列表根签名绑定 （绑定到在上面的命令列表示例的各种槽）。 如果绑定需要更改某些继承的根签名绑定，它必须先设置要调用的命令列表 （继承的绑定不会过时） 相同的根签名。 如果绑定设置根签名不同于调用的命令列表，具有与更改上面所述的命令列表中的根签名相同的效果： 所有以前的根签名绑定已过时，必须设置新预期的绑定之前绘制/调度;否则，该行为不确定。 如果绑定不需要更改根签名的任何绑定，它不需要设置根签名。

下面的代码演示转变为捆绑包的示例调用流。

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

来自一个捆绑包，一个捆绑包进行任何根布局更改和/或绑定更改继承返回到调用的命令列表绑定完成执行时。

有关继承的详细信息，请参阅**图形管道状态继承**一部分[管理图形管道 Direct3D 12 中的状态](managing-graphics-pipeline-state-in-direct3d-12.md)。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




