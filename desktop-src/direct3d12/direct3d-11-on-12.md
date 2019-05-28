---
title: Direct3D 11 on 12
description: D3D11On12 是依据开发人员可以使用 D3D11 接口和对象来驱动 D3D12 API 的机制。
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

D3D11On12 是依据开发人员可以使用 D3D11 接口和对象来驱动 D3D12 API 的机制。 D3D11on12 使使用 D3D11 （如 D2D 文本和 UI） 与针对 D3D12 API 编写的组件一起编写的组件。 D3D11on12 还可以通过启用的应用才能继续以 D3D11 目标为简单起见，而其他人的性能，同时始终具有完整且正确呈现目标 D3D12 部分增量移植应用程序到 D3D12，D3D11。 D3D11On12 可以比使用互操作技术来共享资源和同步两个 Api 之间的工作更简单。

-   [初始化 D3D11On12](#initializing-d3d11on12)
-   [示例用法](#example-usage)
-   [背景](#background)
-   [清理](#cleaning-up)
-   [限制](#limitations)
-   [Api](#apis)
-   [相关的主题](#related-topics)

## <a name="initializing-d3d11on12"></a>初始化 D3D11On12

若要开始使用 D3D11On12，第一步是创建 D3D12 设备和命令队列。 这些对象作为输入提供给初始化方法[ **D3D11On12CreateDevice**](/windows/desktop/api/d3d11on12/nf-d3d11on12-d3d11on12createdevice)。 您可以将此方法作为使用 D3D 的虚部驱动程序类型创建 D3D11 设备\_驱动程序\_类型\_11ON12，D3D11 驱动程序负责创建对象和提交命令将列出对 D3D12 API。

D3D11 设备和即时上下文后，你可以`QueryInterface`遗失的设备[ **ID3D11On12Device** ](/windows/desktop/api/d3d11on12/nn-d3d11on12-id3d11on12device)接口。 这是用于 D3D11 和 D3D12 之间的互操作的主要接口。 为了获得 D3D11 设备上下文和 D3D12 命令列表对相同的资源执行操作，它是创建"包装资源"使用所需[ **CreateWrappedResource** ](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-createwrappedresource) API。 此方法"提升"D3D12 资源是在 D3D11 易于理解。 在"获取"状态下，一个属性，它所操作的已包装的资源开始[ **AcquireWrappedResources** ](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-acquirewrappedresources)并[ **ReleaseWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-releasewrappedresources)方法。

## <a name="example-usage"></a>示例用法

D3D11On12 的典型用法是使用 D2D 呈现文本或图像 D3D12 后台缓冲区。 请参阅 D3D11On12 示例有关代码示例。 下面是执行此操作所采取的步骤大体概述：

-   创建 D3D12 设备 ([**D3D12CreateDevice**](/windows/desktop/api/D3D12/nf-d3d12-d3d12createdevice)) 和 D3D12 交换链 ([**CreateSwapChain** ](https://msdn.microsoft.com/library/windows/desktop/bb174537)与[ **ID3D12CommandQueue** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue)作为输入)。
-   创建作为输入使用 D3D12 设备和相同的命令队列 D3D11On12 设备。
-   检索交换链后台缓冲区，并为每个创建包装的 D3D11 资源。 输入的状态应为 D3D12 使用它的最后一个方法 (例如呈现\_目标) 和输出状态应使用的方式，D3D12 将它 D3D11 完毕后 （如存在）。
-   初始化 D2D，和 D3D11 包装资源向 D2D 准备进行呈现。

然后，在每个帧上，执行以下操作：

-   呈现到当前的交换链后台缓冲区使用 D3D12 命令列表，并执行它。
-   获取当前的后台缓冲区的已包装的资源 ([**AcquireWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-acquirewrappedresources))。
-   发出 D2D 呈现命令。
-   释放已包装的资源 ([**ReleaseWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-releasewrappedresources))。
-   刷新 D3D11 即时上下文。
-   存在 ([**IDXGISwapChain1::Present1**](https://msdn.microsoft.com/library/windows/desktop/hh446797))。

## <a name="background"></a>后台

D3D11On12 系统地工作。 每个 D3D11 API 调用经历典型的运行时验证并使其向驱动程序的方式。 在驱动程序层的特殊 11on12 驱动程序记录状态和问题呈现到 D3D12 命令列表的操作。 这些命令列表提交在必要时 (例如，查询`GetData`或资源`Map`可能需要刷新的命令) 或刷新的要求。 创建 D3D11 对象通常会导致正在创建的相应 D3D12 对象。 某些函数渲染操作中修复 D3D11 如`GenerateMips`或`DrawAuto`D3D12，不支持，因此 D3D11On12 模拟它们使用着色器和其他资源。

互操作，务必了解 D3D11On12 与 D3D12 对象创建并提供应用程序交互的方式。 为了确保工作的发生顺序正确，然后将其他 D3D12 工作提交到该队列，必须刷新 D3D11 即时上下文。 它还是必须确保，提供给 D3D11On12 队列必须是 drainable 在所有时间。 这意味着，最终必须满足在队列上的任何等待，即使 D3D11 呈现线程块无限期。 谨防无法依赖于 D3D11On12 时插入刷新或等待，因为这可能会更改与将来的版本。 此外，D3D11On12 跟踪和操作自己的资源状态。 若要确保一致性状态转换的唯一方法是使用获取/释放 Api 的操作状态跟踪，以匹配应用程序的需求。

## <a name="cleaning-up"></a>清理

以释放 D3D11On12 包装资源，需要按以下顺序执行两项操作：

-   需要释放的资源，包括资源，任何视图的所有引用。
-   延迟的析构必须进行处理。 若要确保这一点的最简单方法进行调用即时上下文是`Flush`API。

这两个步骤完成后，应释放已包装的资源所执行的任何引用，并 D3D12 资源变得以独占方式拥有 D3D12 组件。 请注意 D3D12 仍为 GPU 完成完全释放资源前需要等待，因此请确保保留的引用的资源才能执行这两个步骤更高版本，除非您已经确认 GPU 不能再使用该资源。

所有其他资源或对象创建的 D3D11On12 将清理在适当的时间，GPU 结束时使用它们，使用 D3D11 的延迟的析构机制。 但是如果您尝试释放 D3D11On12 设备本身，GPU 仍在执行时，GPU 完成之前，可能会阻止析构。

## <a name="limitations"></a>限制

D3D11On12 层实现了很大一部分 D3D11 API，但有一些已知的差异 （除了可能会导致不正确呈现的实现中的 bug)。

为 Windows 10，版本 1809 (10.0;生成 17763），只要 D3D11On12 正在运行的驱动程序支持着色器模型 6.0 或更高版本，然后它可以运行使用接口的着色器。 在早期版本的 Windows，着色器接口功能尚未实现在 D3D11On12，并尝试使用该功能会导致错误和调试消息。

为 Windows 10，版本 1803 (10.0;内部版本 17134），交换链支持 D3D11On12 设备。 在早期版本的 Windows 中，它们不是。

D3D11On12 不已针对性能进行优化。 可能有中等程度的 CPU 开销相比标准 D3D11 驱动程序，最少 GPU 开销，并存在一个已知为大量的内存开销。 因此不建议用于 D3D11On12 进行复杂的三维场景，而是建议将其用于简单的场景或 2D 呈现。

## <a name="apis"></a>API

介绍了 Api 11on12 层构成[11on12 引用](direct3d-11on12-reference.md)。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[D2D 使用 D3D11on12 演练](d2d-using-d3d11on12.md)
</dt> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> <dt>

[使用 Direct3D 11，Direct3D 10 和 Direct2D](direct3d-12-interop.md)
</dt> </dl>

 

 




