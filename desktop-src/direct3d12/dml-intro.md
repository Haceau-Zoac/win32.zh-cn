---
title: DirectML 简介
description: 直接机器学习 (DirectML) 是用于机器学习 (ML) 的低级别 API。
ms.custom: 19H1
ms.topic: article
ms.date: 02/01/2019
ms.openlocfilehash: a444edecd071d35b0a1525b3ae6dd18d6962edbf
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224002"
---
# <a name="introduction-to-directml"></a>DirectML 简介

## <a name="summary"></a>摘要

直接机器学习 (DirectML) 是用于机器学习 (ML) 的低级别 API。 （称为运算符） 的硬件加速机器学习基元是 DirectML 的构建基块。 这些构建基块，可以开发此类机器学习技术，放大、 抗锯齿和到名称，但有一些样式传输。 噪声抑制和超级解决方法，例如，允许您将能够实现令人印象深刻 raytraced 效果与较少大气每像素。

您可以将机器学习集成到您的游戏、 引擎、 中间件、 后端或其他应用程序的推断工作负荷。 DirectML 已熟悉 (本机C++，nano COM) DirectX 12 样式编程接口和工作流，并且它支持所有的 DirectX 12 兼容硬件。 有关 DirectML 示例应用程序，包括最小的 DirectML 应用程序的示例，请参阅[DirectML 示例应用程序](dml-min-app.md)。

DirectML 是在 Windows 10，版本 1903，和 Windows SDK 的相应版本中引入的。

## <a name="is-directml-appropriate-for-my-project"></a>是 DirectML 适合我的项目？

DirectML 是下的一个组件[Windows 机器学习](/windows/ai)涵盖性。 更高级别的 WinML API 是主要模型为中心，其负载绑定评估工作流使用。 但域例如游戏和引擎通常需要较低级别的抽象和更高版本的开发人员控制度，以便充分利用芯片。 如果您是计数毫秒，并且榨出帧时间，DirectML 将满足机器学习的需求。

对于可靠的实时、 高性能、 低延迟和/或资源约束方案，请使用 DirectML （而非 WinML）。 您可以将 DirectML 直接到现有的引擎或呈现管道。 或者，在自定义机器学习框架和中间件较高级别，DirectML 可提供高效的后端在 Windows 上。

WinML 是本身作为一个其后端使用 DirectML 实现。

## <a name="what-work-does-directml-do-and-what-work-must-i-do-as-the-developer"></a>哪些工作 DirectML？;和哪些工作必须*我*作为开发人员执行操作？

DirectML 有效地执行您的推理模型的各个层，在 GPU 上 （或 AI 加速个内核，如果存在）。 每一层是运算符，并 DirectML 提供低级别、 硬件加速的机器学习基元运算符的库。 这些运算符具有与其设计中的特定于硬件和特定于体系结构的优化 (部分中将详细介绍[为什么会 DirectML 执行得很好？](#why-does-directml-perform-so-well))。 在同一时间，您作为开发人员看到执行这些运算符的单一、 供应商无关的接口。

DirectML 中运算符的库提供的所有期望能够在机器学习工作负荷中使用的常用操作。

- 激活运算符，如**线性**， **ReLU**， **sigmoid**， **tanh**，和的详细信息。
- 点运算符，如**添加**， **exp**，**日志**，**最大**， **min**， **sub**，和的详细信息。
- 卷积运算符，如 2D 和 3D**卷积**，和的详细信息。
- 减少运算符，如**argmin**，**平均**， **l2**，**总和**，和的详细信息。
- 池运算符，如**平均**， **lp**，并**最大**。
- 神经网络 (NN) 运算符，如**gemm**， **gru**， **lstm**，以及**rnn**。
- 及更多。

为了最大性能，以便无需付费不使用，DirectML 使控件进入手作为开发人员通过机器学习工作负荷的硬件上的执行方式。 找出哪些运算符上执行，并且为时，您作为开发人员有责任。 留给您自行决定的任务包括： 转录模型;简化和优化您的层;正在加载权重;资源分配，绑定，内存管理 （就像使用 Direct3D 12）;和关系图的执行。

保留的关系图的高级知识 （可以硬编码您的模型直接，也可以编写您自己的模型加载程序）。 可能设计 upscaling 模型中，例如，使用多个层的每个**取样**，**卷积**，**规范化**，和**激活**运算符。 使用该熟悉程度、 仔细计划和屏障管理，可以从硬件中提取最多并行性和性能。 如果要开发游戏，然后注意资源管理和控制计划可以交错为了使饱和 GPU 的机器学习工作负荷和传统呈现工作。

## <a name="whats-the-high-level-directml-workflow"></a>什么是高级 DirectML 工作流？

下面是高级方案，我们期望 DirectML 要使用的方式。 中的初始化和执行的两个主要阶段，记录工作到命令列表，然后在队列上执行它们。

### <a name="initialization"></a>初始化

1. 创建 Direct3D 12 资源&mdash;Direct3D 12 设备、 命令队列、 命令列表和资源，例如描述符堆。
2. 要执行机器学习推断，以及你的渲染工作负载，因为创建 DirectML 资源&mdash;DirectML 设备和运算符实例。 如果有，您需要执行特定类型的卷积的筛选器 tensor 特定大小与特定的数据类型，则这些是到 DirectML 的所有参数的机器学习模型**卷积**运算符。
3. DirectML 记录到 Direct3D 12 命令列表的工作原理。 因此，完成初始化后，你记录的绑定和初始化 （例如） 在卷积运算符到命令列表。 然后，关闭并像往常一样在您的队列上执行命令列表。

### <a name="execution"></a>执行

1. 将权重 tensors 上传到资源。 使用常规的 Direct3D 12 资源表示 tensor DirectML 中。 例如，如果你想要将权重数据上载到 GPU，则您执行的操作相同的方式就像使用任何其他 Direct3D 12 资源 （使用上载堆或复制队列）。
2. 接下来，需要将绑定为输入和输出 tensors 这些 Direct3D 12 资源。 记录到命令列表的绑定和你的运算符的执行。
3. 关闭并执行命令列表。

与相同 Direct3D 12 资源生存期和同步由你负责。 例如，不发布 DirectML 对象至少直到它们已完成在 GPU 上的执行。

## <a name="why-does-directml-perform-so-well"></a>为什么会 DirectML 执行得很好？

没有充分的理由为什么你不应只需编写自己的卷积运算符 （例如） 作为中的 HLSL[计算着色器](/windows/desktop/direct3d12/pipelines-and-shaders-with-directx-12#direct3d-12-compute-pipeline)。 使用 DirectML 的优点在于&mdash;除了保存您的工作量 homebrewing 你自己的解决方案&mdash;已为您提供更好的性能不是您可以使用手写的常规用途计算来实现的功能着色器对一些喜欢**卷积**，或**lstm**。

DirectML 来实现此部分由于 Direct3D 12 metacommands 功能。 Metacommands 公开的功能最多 DirectML，允许硬件供应商提供对供应商特定于硬件和特定于体系结构的优化 DirectML 访问一个黑色框。 多个运算符&mdash;例如，卷积跟激活&mdash;可以是*融合在*合为一个 metacommand。 由于这些因素，DirectML 都具有可以超过甚至编写非常良好手工调整计算着色器编写范围广泛的硬件上运行的性能。

Metacommands 是 Direct3D 12 API 的一部分，尽管它们松散地耦合到它。 Metacommand 是由固定 GUID 标识，而几乎所有它的其他内容 （从其行为和语义为其签名和名称） 不严格 Direct3D 12 API 的一部分。 相反，它的作者和实现该驱动程序之间指定 metacommand。 在这种情况下，作者是 DirectML。 Metacommands 是 Direct3D 12 基元 （只需如绘制和调度），以便他们可以记录到命令列表并一起计划执行。

DirectML 加速机器学习工作负荷使用机器学习 metacommands 整个套件。 因此，不需要编写特定于供应商的代码路径来实现您推断的硬件加速。 如果您碰巧在 AI 加速的芯片上运行，DirectML 将使用该硬件来极大地加快操作，如卷积。 可能需要你编写了，而无需修改它，不是 AI 加速的芯片上运行它 (可能是集成中的 GPU 便携式计算机)，并仍获得强大的 GPU 硬件加速的相同代码。 并且如果没有 GPU 可用，然后 DirectML 回退到 CPU。
