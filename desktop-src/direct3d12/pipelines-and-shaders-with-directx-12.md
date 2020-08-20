---
title: Direct3D 12 的管道和着色器
description: 相较于上一代图形编程接口，Direct3D 12 可编程管道显著提高了渲染性能。
ms.assetid: 329882F5-D2A9-4D6D-AC3B-29F370D22C97
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 7815dd08616c883e2e300aa84710acc802e4db09
ms.sourcegitcommit: 592c9bbd22ba69802dc353bcb5eb30699f9e9403
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2020
ms.locfileid: "88644103"
---
# <a name="pipelines-and-shaders-with-direct3d-12"></a>Direct3D 12 的管道和着色器

相较于上一代图形编程接口，Direct3D 12 可编程管道显著提高了渲染性能。

-   [Direct3D 12 图形管道](#direct3d-12-graphics-pipeline)
-   [管道状态对象](#pipeline-state-objects)
-   [Direct3D 12 计算管道](#direct3d-12-compute-pipeline)
-   [相关主题](#related-topics)

## <a name="direct3d-12-graphics-pipeline"></a>Direct3D 12 图形管道

下图说明了 Direct3D 12 图形管道和状态。

![说明 direct3d 12 管道和状态的图示](images/pipeline.png)

图形管道是 GPU 渲染帧时数据输入和输出的顺序流。 根据管道状态和输入，GPU 执行一系列操作来创建生成的图像。 图形管道包含着色器，后者执行可编程的渲染效果和计算，以及固定的函数操作。

参考管道状态图时，请注意以下内容：

-   描述符表和堆可以任意排列： SRVs、CBVs 和 Uav 可以按任意顺序引用和分配。
-   管道的部分操作为可配置操作。 例如，输出合并通常以“读取-修改-写入”为基础对呈现器目标和深度模具视图进行操作。 但可以配置管道，使任一视图为只读或只写。
-   静态采样器是静态的，因此它不属于根参数。

## <a name="pipeline-state-objects"></a>管道状态对象

Direct3D 12 引入了管道状态对象 (PSO)。 不是跨大量高级对象存储和表示管道状态，而是在 PSO 中存储管道组件（如输入汇编程序、光栅化程序、象素着色器和输出合并）的状态。 PSO 是统一的管道状态对象，且创建后不可改变。 当前选择的 PSO 可快速、动态地进行改变，硬件和驱动程序可直接将 PSO 转换为本机硬件指令和状态，以便 GPU 能够进行图形处理。 为应用 PSO，硬件将尽可能少的预计算状态直接复制到硬件寄存器。 这消除了由于图形驱动程序基于当前所有适用的渲染和管道设置不断重新计算硬件状态而产生的开销。 其结果是显著减少了绘制调用开销、提高了性能，并且每一帧可进行更多次绘制调用。

当前应用的 PSO 定义并连接渲染管道中使用的所有着色器。 [Microsoft 高级着色器语言 (HLSL)](/windows/desktop/direct3dhlsl/dx-graphics-hlsl) 预先编译到着色器对象中，然后在运行时后者用作管道状态对象的输入。 如需深入了解 PSO 在图形管道中的工作原理，请参阅[在 Direct3D 12 中管理图形管道状态](managing-graphics-pipeline-state-in-direct3d-12.md)。

## <a name="direct3d-12-compute-pipeline"></a>Direct3D 12 计算管道

下图说明了 Direct3D 12 计算管道和状态。

![](images/compute-pipeline.png)

此管道中没有固定函数单元，但计算时仍可使用描述符堆、采样器堆和静态采样器。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 中的工作提交](command-queues-and-command-lists.md)
</dt> </dl>

 

 