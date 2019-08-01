---
title: Direct3D 12 互操作
description: D3D12 可用于编写组件化应用程序。
ms.assetid: 51F7E715-82B6-48D8-A06A-CBBEDF6968F5
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 81bb106f33cf6f6996ef11f4d023e34ab989710c
ms.sourcegitcommit: 6bed25d64276d500eb21b789f788a75ee229b4d6
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "68679730"
---
# <a name="direct3d-12-interop"></a>Direct3D 12 互操作

D3D12 可用于编写组件化应用程序。

-   [互操作概述](#interop-overview)
-   [使用互操作的原因](#reasons-for-using-interop)
    -   [共享命令列表](#sharing-a-command-list)
    -   [共享命令队列](#sharing-a-command-queue)
    -   [共享同步基元](#sharing-sync-primitives)
    -   [共享资源](#sharing-resources)
    -   [选择互操作模型](#choosing-an-interop-model)
-   [互操作 Api](#interop-apis)
-   [相关主题](#related-topics)

## <a name="interop-overview"></a>互操作概述

D3D12 可以提供非常强大的功能, 使应用程序能够以类似控制台的效率编写图形代码, 但并不是每个应用程序都需要重新生成滚轮并从头开始编写其整个呈现引擎。 在某些情况下, 另一个组件或库已经更好地执行了此操作, 或者在其他情况下, 一部分代码的性能并不像它的正确性和可读性那么重要。

本部分介绍以下互操作技术:

-   D3D12 和 D3D12, 位于同一设备上
-   不同设备上的 D3D12 和 D3D12
-   同一设备上的 D3D12 和 D3D11、D3D10 或 D2D 的任意组合
-   不同设备上的 D3D12 和 D3D11、D3D10 或 D2D 的任意组合
-   D3D12 和 GDI, 或 D3D12 和 D3D11 和 GDI

## <a name="reasons-for-using-interop"></a>使用互操作的原因

应用程序需要使用其他 Api D3D12 互操作的原因有多种。 一些示例:

-   增量移植: 希望将整个应用程序从 D3D10 或 D3D11 移植到 D3D12, 同时使其在迁移过程的中间阶段正常运行 (以启用测试和调试)。
-   黑色框代码: 想要将应用程序的特定部分保留原样, 同时移植代码的其余部分。 例如, 可能无需移植游戏的 UI 元素。
-   不可进行的组件: 需要使用不属于应用程序的组件, 这些组件不会写入目标 D3D12。
-   新组件: 不希望移植整个应用程序, 而是想要使用通过 D3D12 编写的新组件。

D3D12 中有四种用于互操作的主要方法:

-   应用程序可以选择向组件提供打开的命令列表, 这会将一些其他呈现命令记录到已绑定的呈现器目标。 这等效于向 D3D11 中的另一个组件提供已准备好的设备上下文, 并且非常适用于将 UI/文本添加到已绑定的后台缓冲区的情况。
-   应用可以选择向组件提供命令队列以及所需的目标资源。 这等效于使用 D3D11 中的[**ClearState**](https://docs.microsoft.com/windows/desktop/api/d3d11/nf-d3d11-id3d11devicecontext-clearstate)或[**DeviceContextState**](https://docs.microsoft.com/windows/desktop/api/d3d11_1/nn-d3d11_1-id3ddevicecontextstate) api 向另一个组件提供干净的设备上下文。 这是 D2D 等组件的工作方式。
-   组件可以选择一个模型, 在此模型中, 它会生成一个命令列表 (可能并行), 应用负责稍后提交。 必须跨组件边界提供至少一个资源。 这一技术在使用延迟上下文的 D3D11 中可用, 但 D3D12 的性能更适合。
-   每个组件都有其自己的队列和/或设备, 应用程序和组件需要跨组件边界共享资源和同步信息。 这类似于旧版`ISurfaceQueue`, 更新式的[**IDXGIKeyedMutex**](https://docs.microsoft.com/windows/desktop/api/dxgi/nn-dxgi-idxgikeyedmutex)。

这两种情况之间的区别是在组件边界之间共享的内容完全相同。 设备被认为是共享的, 但由于它基本上是无状态的, 因此它并不真正相关。 关键对象包括命令列表、命令队列、同步对象和资源。 其中每个都在共享它们时有其自身的复杂问题。

### <a name="sharing-a-command-list"></a>共享命令列表

最简单的互操作方法要求仅共享包含部分引擎的命令列表。 完成呈现操作后, 命令列表所有权将返回到调用方。 可以通过堆栈跟踪命令列表的所有权。 由于命令列表是单线程的, 因此应用程序无法使用此方法执行独特或创新性操作。

### <a name="sharing-a-command-queue"></a>共享命令队列

对于在同一进程中共享设备的多个组件, 可能使用最常见的方法。

如果命令队列是共享的单元, 则需要对组件进行调用, 以使其知道需要将所有未完成的命令列表立即提交到命令队列 (并且需要同步任何内部命令队列)。 这等效于 D3D11 [**Flush**](https://docs.microsoft.com/windows/desktop/api/d3d11/nf-d3d11-id3d11devicecontext-flush) API, 它是应用程序可以提交其自己的命令列表或同步基元的唯一方法。

### <a name="sharing-sync-primitives"></a>共享同步基元

在其自己的设备和/或命令队列上操作的组件的预期模式将是接受[**ID3D12Fence**](/windows/desktop/api/d3d12/nn-d3d12-id3d12fence)或 shared 句柄, 并在开始其工作时接受 UINT64 配对, 它将等待, 然后是另一个 ID3D12Fence 或共享句柄,完成所有工作后将发出信号的 UINT64 对。 此模式匹配[**IDXGIKeyedMutex**](https://docs.microsoft.com/windows/desktop/api/dxgi/nn-dxgi-idxgikeyedmutex)和 DWM/DXGI 翻转模型同步设计的当前实现。

### <a name="sharing-resources"></a>共享资源

到目前为止, 编写利用多个组件的 D3D12 应用的最复杂的部分是如何处理跨组件边界共享的资源。 这主要是因为资源状态的概念。 尽管资源状态设计的某些方面旨在处理命令列表同步, 但其他方面却会对命令列表产生影响, 影响资源布局以及访问的有效操作集或性能特征资源数据。

有两种处理这一复杂的模式, 这两种模式都涉及组件之间的协定。

-   协定可由组件开发人员定义并记录在文档中。 这可能很简单, 如 "工作开始时资源必须处于默认状态, 并将在工作完成时被恢复为默认状态"; 或者可以具有更复杂的规则, 以允许在不强制中间深度的情况下共享深度缓冲区解析ves.
-   当资源在组件边界之间共享时, 应用程序可以在运行时定义协定。 它包含相同的两条信息, 即, 当组件开始使用资源时, 该资源将处于的状态, 以及组件在完成后应将其保留的状态。

### <a name="choosing-an-interop-model"></a>选择互操作模型

对于大多数 D3D12 应用程序而言, 共享命令队列可能是理想的模型。 它允许完成工作创建和提交的全部所有权, 无需额外的内存开销, 因为具有冗余队列, 而且不会对处理 GPU 同步基元产生任何影响。

一旦组件需要处理不同的队列属性 (例如类型或优先级), 或者一旦共享需要跨进程边界, 就需要共享同步基元。

第三方组件不会广泛地使用共享或生成命令列表, 但可能会广泛地用于游戏引擎的内部组件。

## <a name="interop-apis"></a>互操作 Api

[Direct3D 11 on 12](/windows/win32/direct3d12/direct3d-11-on-12)主题介绍了与本主题中所述的互操作类型相关的大部分 API 图面。

另请参阅[ID3D12Device:: CreateSharedHandle](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createsharedhandle)方法, 该方法可用于在 Windows 图形 api 之间共享表面。

## <a name="related-topics"></a>相关主题

* [了解 Direct3D 12](directx-12-getting-started.md)
* [使用 Direct3D 11、Direct3D 10 和 Direct2D](direct3d-12-interop.md)
* [Direct3D 11 on 12](/windows/win32/direct3d12/direct3d-11-on-12)