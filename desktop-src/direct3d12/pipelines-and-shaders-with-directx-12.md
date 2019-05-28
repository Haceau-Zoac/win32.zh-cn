---
title: 管道和着色器使用 Direct3D 12
description: Direct3D 12 可编程管道会大大提高呈现性能相比上一代图形编程接口。
ms.assetid: 329882F5-D2A9-4D6D-AC3B-29F370D22C97
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 1f196e6dbcab56059d9ff4f4ccaf9a67c69b5dd5
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223909"
---
# <a name="pipelines-and-shaders-with-direct3d-12"></a>管道和着色器使用 Direct3D 12

Direct3D 12 可编程管道会大大提高呈现性能相比上一代图形编程接口。

-   [Direct3D 12 图形管道](#direct3d-12-graphics-pipeline)
-   [管道状态对象](#pipeline-state-objects)
-   [Direct3D 12 个计算管道](#direct3d-12-compute-pipeline)
-   [相关的主题](#related-topics)

## <a name="direct3d-12-graphics-pipeline"></a>Direct3D 12 图形管道

下图说明了 Direct3D 12 图形管道和状态。

![演示如何 direct3d 12 管道和状态的关系图](images/pipeline.png)

图形管道是数据输入的顺序流，并输出 GPU 呈现的帧。 给定的管道状态和输入，GPU 执行一的系列操作创建了生成的映像。 图形管道包含执行可编程的呈现效果和计算和固定的函数操作的着色器。

当指管道状态关系图时，请注意以下：

-   可以随意排列描述符表和堆：SRVs、 CBVs 和 Uav 可以引用并按任何顺序分配。
-   管道的一些操作进行配置。 例如，输出合并器运营方式通常是读取-修改-写入按与呈现器目标和深度模具视图。 但是可以配置管道，以便其中一个视图是只读或只写。
-   静态取样器不是根自变量的一部分，因为它们是静态。

## <a name="pipeline-state-objects"></a>管道状态对象

Direct3D 12 中引入了管道状态对象 (PSO)。 而不是存储和跨大量的高级别对象表示管道状态，管道组件的状态等输入装配器光栅器、 像素着色器和输出合并器存储在 pso 的步骤。 Pso 的步骤是在创建后不可变的统一的管道状态对象。 快速、 动态地，可以更改当前所选的 PSO，硬件和驱动程序可以直接将 PSO 转换的本机硬件说明和状态，将准备就绪的图形处理 GPU。 若要将 PSO 应用，硬件将直接与硬件寄存器复制最少量的预先计算的状态。 这会删除由不断地重新计算基于所有当前适用的呈现和管道设置的硬件状态图形驱动程序引起的开销。 结果是显著降低了的绘图调用开销，提高了性能，并且多个绘图调用每个框架。

当前应用的 PSO 定义，并将在呈现管道中使用的着色器的所有连接。 [Microsoft 高级别着色器语言 (HLSL)](https://msdn.microsoft.com/library/windows/desktop/bb509561)是到着色器对象，然后使用在运行时作为输入管道状态对象的预编译。 PSO 图形管道中的运行方式的详细信息，请参阅[管理图形管道 Direct3D 12 中的状态](managing-graphics-pipeline-state-in-direct3d-12.md)。

## <a name="direct3d-12-compute-pipeline"></a>Direct3D 12 个计算管道

下图说明了管道 Direct3D 12 计算和状态。

![](images/compute-pipeline.png)

堆有在此管道，但是描述符堆，采样器没有固定的功能单位，静态取样器都在计算中仍然可用。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[提交工作在 Direct3D 12](command-queues-and-command-lists.md)
</dt> </dl>

 

 




