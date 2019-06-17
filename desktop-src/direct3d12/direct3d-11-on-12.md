---
title: Direct3D 11 on 12
description: 开发人员可以通过 D3D11On12 机制使用 D3D11 接口和对象来驱动 D3D12 API。
ms.assetid: 8412D8BB-B6DD-471E-AAB2-A81121FB0FFA
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 5c547bfb87e00bfe9ff4cc87aabb0652f0e687c1
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223777"
---
# <a name="direct3d-11-on-12"></a>Direct3D 11 on 12

开发人员可以通过 D3D11On12 机制使用 D3D11 接口和对象来驱动 D3D12 API。 借助 D3D11on12，使用 D3D11 编写的组件（如 D2D 文本和 UI）可与针对 D3D12 API 编写的组件配合工作。 D3D11on12 还可以实现应用程序从 D3D11 到 D3D12 的增量移植，它可让应用的某些组成部分继续针对 D3D11 以简化操作，同时让其他某些组成部分针对 D3D12 以保证性能，并始终提供完整且准确的渲染效果。 使用 D3D11On12 比使用互操作技术在两个 API 之间共享资源和同步工作更为简单。

-   [初始化 D3D11On12](#initializing-d3d11on12)
-   [示例用法](#example-usage)
-   [背景](#background)
-   [清理](#cleaning-up)
-   [限制](#limitations)
-   [API](#apis)
-   [相关主题](#related-topics)

## <a name="initializing-d3d11on12"></a>初始化 D3D11On12

若要开始使用 D3D11On12，第一步是创建 D3D12 设备和命令队列。 这些对象将作为输入提供给初始化方法 [**D3D11On12CreateDevice**](/windows/desktop/api/d3d11on12/nf-d3d11on12-d3d11on12createdevice)。 可将此方法看作是使用虚构驱动程序类型 D3D\_DRIVER\_TYPE\_11ON12 创建一个 D3D11 设备，其中，D3D11 驱动程序负责创建对象并将命令列表提交到 D3D12 API。

创建 D3D11 设备和中间上下文后，可以在该设备的外部针对 [**ID3D11On12Device**](/windows/desktop/api/d3d11on12/nn-d3d11on12-id3d11on12device) 接口执行 `QueryInterface`。 这是用于在 D3D11 与 D3D12 之间实现互操作的主要接口。 要使 D3D11 设备上下文和 D3D12 命令列表都可在相同的资源上运行，需要使用 [**CreateWrappedResource**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-createwrappedresource) API 创建“包装的资源”。 此方法会“提升”某个 D3D12 资源，使其在 D3D11 中可识别。 包装的资源最初处于“acquired”状态（[**AcquireWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-acquirewrappedresources) 和 [**ReleaseWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-releasewrappedresources) 方法操作的属性）。

## <a name="example-usage"></a>示例用法

D3D11On12 的典型用法是使用 D2D 在 D3D12 反向缓冲区的顶层渲染文本或图像。 有关代码示例，请参阅 D3D11On12 示例。 下面简要概述了相关的步骤：

-   创建一个 D3D12 设备 ([**D3D12CreateDevice**](/windows/desktop/api/D3D12/nf-d3d12-d3d12createdevice)) 和一个 D3D12 交换链（使用 [**ID3D12CommandQueue**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue) 作为输入的 [**CreateSwapChain**](https://msdn.microsoft.com/library/windows/desktop/bb174537)）。
-   使用 D3D12 设备和相同的命令队列作为输入创建一个 D3D11On12 设备。
-   检索交换链反向缓冲区，并为每个缓冲区创建包装的 D3D11 资源。 使用的输入状态应是 D3D12 上次使用该输入的方式（例如 RENDER\_TARGET），输出状态应是 D3D12 在 D3D11 完成后使用该输出的方式（例如 PRESENT）。
-   初始化 D2D，并将 D3D11 已包装资源提供给 D2D 以准备渲染。

然后，对每个帧执行以下操作：

-   使用 D3D12 命令列表在当前的交换链反向缓冲区中执行渲染。
-   获取当前的反向缓冲区的已包装资源 ([**AcquireWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-acquirewrappedresources))。
-   发出 D2D 渲染命令。
-   释放已包装资源 ([**ReleaseWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-releasewrappedresources))。
-   刷新 D3D11 即时上下文。
-   呈现 ([**IDXGISwapChain1::Present1**](https://msdn.microsoft.com/library/windows/desktop/hh446797))。

## <a name="background"></a>后台

D3D11On12 有系统地工作。 每次调用 D3D11 API 都要经历典型的运行时验证，然后转到驱动程序。 在驱动程序层，特殊的 11on12 驱动程序会记录状态，并向 D3D12 命令列表发出渲染操作。 会根据需要或者按 Flush 的请求提交这些命令列表（例如，查询 `GetData` 或资源 `Map` 可能需要刷新命令）。 创建 D3D11 对象通常会导致创建相应的 D3D12 对象。 D3D11 中的某些固定函数渲染操作（例如 `GenerateMips` 或 `DrawAuto`）在 D3D12 中不受支持，因此，D3D11On12 会使用着色器和其他资源来模拟这些操作。

在互操作性方面，必须知道 D3D11On12 如何与应用创建和提供的 D3D12 对象交互。 为确保工作按正确的顺序发生，必须先刷新 D3D11 即时上下文，然后才能将其他 D3D12 工作提交到该队列。 此外，必须确保提供给 D3D11On12 的队列随时可排空。 这意味着，最终必须满足队列中的任何等待指令，即使 D3D11 渲染线程无限期阻塞。 谨慎确保不要依赖于 D3D11On12 插入内容的刷新或等待时间，因为在将来的版本中此时间可能会有变化。 此外，D3D11On12 会自行跟踪和操作资源状态。 确保状态转换一致性的唯一方法是使用获取/释放 API 根据应用的需求操作状态跟踪。

## <a name="cleaning-up"></a>清理

若要释放 D3D11On12 已包装资源，需要按以下顺序执行两项操作：

-   需要释放对资源（包括所有资源视图）的所有引用。
-   必须执行推迟销毁处理。 确保做到这一点的最简单方法是调用即时上下文 `Flush` API。

完成这两个步骤后，应释放已包装资源所用的任何引用，然后，D3D12 资源将由 D3D12 组件独占拥有。 请注意，在完全释放资源之前，D3D12 仍要求等待 GPU 完成，因此，在执行上述两个步骤之前，请务必保留资源中的引用，除非已确认 GPU 不再使用该资源。

D3D11On12 创建的所有其他资源或对象将在 GPU 用完它们后，使用 D3D11 的推迟销毁机制在适当的时间予以清理。 但是如果在 GPU 仍在执行期间尝试释放 D3D11On12 设备本身，则销毁过程可能会阻塞到 GPU 完成为止。

## <a name="limitations"></a>限制

D3D11On12 层实现很大一部分的 D3D11 API，但存在一些已知的差异（此外，实现中的 bug 可能会导致渲染错误）。

从 Windows 10 版本 1809（10.0；内部版本 17763）开始，只要 D3D11On12 在支持着色器模型 6.0 或更高版本的驱动程序中运行，则它就可以运行使用接口的着色器。 在早期版本的 Windows 中，着色器接口功能未在 D3D11On12 中实现，尝试使用该功能会导致出现错误和调试消息。

从 Windows 10 版本 1803（10.0；内部版本 17134）开始，D3D11On12 设备支持交换链。 早期版本的 Windows 不支持交换链。

D3D11On12 未在性能方面经过优化。 与 D3D11 驱动程序相比，其 CPU 开销可能适度，GPU 开销极低，但内存开销已知很高。 因此不建议将 D3D11On12 用于复杂的 3D 场景，而是建议将其用于简单场景或 2D 渲染。

## <a name="apis"></a>API

[11on12 参考](direct3d-11on12-reference.md)中介绍了构成 11on12 层的 API。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[使用 D3D11on12 的 D2D 演练](d2d-using-d3d11on12.md)
</dt> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> <dt>

[使用 Direct3D 11、Direct3D 10 和 Direct2D](direct3d-12-interop.md)
</dt> </dl>

 

 




