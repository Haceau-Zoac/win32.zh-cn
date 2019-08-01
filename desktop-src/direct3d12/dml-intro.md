---
title: DirectML 简介
description: 直接机器学习 (DirectML) 是机器学习 (ML) 的低级 API。
ms.custom: Windows 10 May 2019 Update
ms.localizationpriority: high
ms.topic: article
ms.date: 04/19/2019
ms.openlocfilehash: 6ce5dd0a1710e1625d93a12536fa9c7fac0a2e43
ms.sourcegitcommit: 8141395d1bd1cd755d1375715538c3fe714ba179
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67465012"
---
# <a name="introduction-to-directml"></a>DirectML 简介

## <a name="summary"></a>摘要

直接机器学习 (DirectML) 是机器学习 (ML) 的低级 API。 硬件加速的机器学习基元（称为“运算符”）是 DirectML 的构建基块。 在这些构建基块中，可以开发纵向扩展、抗锯齿和样式转移等机器学习技术。 例如，使用噪声抑制和超解析度，可以实现令人印象深刻的光线跟踪效果且可以减少每个像素的光线。

可将机器学习推断工作负荷集成到游戏、引擎、中间件、后端或其他应用程序中。 DirectML 提供用户熟悉的（本机C++、nano-COM）DirectX 12 式编程接口和工作流，且受所有 DirectX 12 兼容硬件的支持。 有关 DirectML 示例应用程序（包括精简 DirectML 应用程序的示例），请参阅 [DirectML 示例应用程序](dml-min-app.md)。

DirectML 是在 Windows 10 版本 1903 和相应版本的 Windows SDK 中引入的。

## <a name="is-directml-appropriate-for-my-project"></a>DirectML 是否适合我的项目？

DirectML 是 [Windows 机器学习](/windows/ai)涵盖的一个组件。 更高级别的 WinML API 主要面向模型及其加载-绑定-评估工作流。 但是，游戏和引擎等领域通常需要更低级别的抽象和更高的开发人员控制度，以充分利用芯片。 如果你追求的是毫秒级延迟和极短的帧时间，DirectML 可满足你的机器学习需求。

对于可靠、实时、高性能、低延迟和/或资源受限的方案，请使用 DirectML（而非 WinML）。 可将 DirectML 直接集成到现有的引擎或渲染管道中。 或者，在更高级别的自定义机器学习框架和中间件上，DirectML 可在 Windows 中提供一个高性能的后端。

WinML 本身是使用 DirectML 作为后端之一实现的。

## <a name="what-work-does-directml-do-and-what-work-must-i-do-as-the-developer"></a>DirectML 可完成哪些工作？作为开发人员，我必须完成哪些工作？ 

DirectML 在 GPU 上（或者在 AI 加速核心上，如果存在）有效执行推理模型的各个层。 每个层是一个运算符，DirectML 提供低级别、硬件加速机器学习基元运算符的库。 这些运算符采用硬件特定的体系结构特定优化设计（[DirectML 的表现为何如此出众？](#why-does-directml-perform-so-well)部分提供了更多相关信息）。 同时，开发人员会看到单一与供应商无关的界面，可在其中执行这些运算符。

DirectML 中的运算符库提供预期可在机器学习工作负荷中使用的所有常用操作。

- 激活运算符，例如 **linear**、**ReLU**、**sigmoid**、**tanh**，等等。
- 元素范围的运行符，例如 **add**、**exp**、**log**、**max**、**min**、**sub**，等等。
- 卷积运算符，例如 2D 和 3D **convolution**，等等。
- 化减运算符，例如 **argmin**、**average**、**l2**、**sum**，等等。
- 池运算符，例如 **average**、**lp** 和 **max**。
- 神经网络 (NN) 运算符，例如 **gemm**、**gru**、**lstm** 和 **rnn**。
- 还有其他许多运算符。

为获得最大性能，并避免针对未使用的资源付费，DirectML 允许开发人员亲手控制机器学习工作负荷在硬件上的执行方式。 开发人员需负责确定要执行的运算符，以及何时执行。 由你自行决定的任务包括：听录模型；简化和优化层；加载权重；资源分配、绑定和内存管理（如同在 Direct3D 12 中一样）；图形执行。

你需要对图形具有深入的了解（可以直接将模型硬编码，或者编写自己的模型加载程序）。 例如，可以使用每个**过采样**、**卷积**、**规范化**和**激活**运算符设计纵向扩展模块。 凭借对技术的熟悉、精心的计划和屏障管理，可以从硬件中获得最高并行度和性能。 开发游戏时，精心的资源管理和计划控制可以错开机器学习工作负荷和传统的渲染工作，以充分利用 GPU。

## <a name="whats-the-high-level-directml-workflow"></a>什么是高级 DirectML 工作流？

以下高级方案演示了 DirectML 的期望用法。 在初始化和执行的两个主要阶段中，将工作记录到命令列表，然后在队列上执行它们。

### <a name="initialization"></a>初始化

1. 创建 Direct3D 12 资源 &mdash; Direct3D 12 设备、命令队列、命令列表和资源（例如描述符堆）。
2. 由于你要执行机器学习推断和渲染工作负荷，因此需要创建 DirectML 资源 &mdash; DirectML 设备和运算符实例。 如果需要在机器学习模型中使用特定数据类型的特定大小的筛选器张量执行特定类型的卷积运算，则所有这些参数都是要加入 DirectML **卷积**运算符的参数。
3. DirectML 将工作记录到 Direct3D 12 命令列表中。 因此，完成初始化后，需要将（例如）卷积运算符的绑定和初始化记录到命令列表中。 然后，像平时一样在队列中关闭和执行该命令列表。

### <a name="execution"></a>执行

1. 将权重张量上传到资源中。 DirectML 中的张量是使用常规的 Direct3D 12 资源表示的。 例如，若要将权重数据上传到 GPU，请像处理任何其他 Direct3D 12 资源一样执行该操作（使用上传堆或复制队列）。
2. 接下来，需要将这些 Direct3D 12 资源绑定为输入和输出张量。 记录到运算符的绑定和执行的命令列表。
3. 关闭并执行命令列表。

与在 Direct3D 12 中一样，资源生存期和同步由你负责。 例如，在 DirectML 对象最起码已在 GPU 上完成执行之前不要释放这些对象。

## <a name="why-does-directml-perform-so-well"></a>DirectML 的表现为何如此出众？

有一个很好的理由可以解释为何不只是在[计算着色器](/windows/desktop/direct3d12/pipelines-and-shaders-with-directx-12#direct3d-12-compute-pipeline)中编写自己的（例如）卷积运算符作为 HLSL。 使用 DirectML 的优势在于 &mdash; 除了节省自行编写解决方案所要耗费的精力以外 &mdash; 与对**卷积**或 **lstm** 等运算符使用手工编写的通用计算着色器相比，它可以提供高得多的性能。

DirectML 在一定程度上借助了 Direct3D 12 元命令功能实现了这项优势。 元命令可直接向 DirectML 公开功能的黑盒，可让硬件供应商为 DirectML 提供对供应商硬件特定的和体系结构特定的优化的访问权限。 多个运算符 &mdash; 例如卷积后接激活&mdash; 可以融合在一起构成一个元命令。  由于这些因素，DirectML 的性能甚至可以超过妥善编写的且经过手工优化的、可在各种硬件上运行的计算着色器。

元命令属于 Direct3D 12 API，不过，两者是松散耦合的。 元命令按固定 GUID 标识，有关它的其他任何信息（从其行为和语义，到其签名和名称）在严格意义上几乎都不是 Direct3D 12 API 的一部分。 元命令是在其作者与实现它的驱动程序之间指定的。 在这种情况下，作者是 DirectML。 元命令是 Direct3D 12 基元（与绘制和调度一样），因此，可将它们记录到命令列表，并将其计划为一起执行。

DirectML 使用整套机器学习元命令加速机器学习工作负荷。 因此，你无需编写供应商特定的代码路径即可在推理过程中实现硬件加速。 如果你正好在 AI 加速的芯片上运行，则 DirectML 会使用该硬件来大幅加速操作，例如卷积运算。 可以采用自己编写的相同代码且不对其进行修改，并在未经 AI 加速的芯片（也许是笔记本电脑中集成的 GPU）上运行它，这样仍可获得强大的 GPU 硬件加速。 如果没有可用的 GPU，则 DirectML 会回退到 CPU。
