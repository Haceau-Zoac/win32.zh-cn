---
title: 从 Direct3D 11 移植到 Direct3D 12
description: 本部分提供有关从自定义的 Direct3D 11 图形引擎移植到 Direct3D 12 的一些指导。
ms.assetid: 9EB4AC6B-AFDD-4673-8EB3-54272C151784
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 1c3eb05651aaac3a72fb0d0017999fb60fd9e4e7
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224248"
---
# <a name="porting-from-direct3d-11-to-direct3d-12"></a>从 Direct3D 11 移植到 Direct3D 12

本部分提供有关从自定义的 Direct3D 11 图形引擎移植到 Direct3D 12 的一些指导。

-   [已提交的资源](#committed-resources)
-   [保留的资源](#reserved-resources)
-   [将数据上传](#uploading-data)
-   [着色器和着色器对象](#shaders-and-shader-objects)
-   [提交到 GPU 的工作](#submitting-work-to-the-gpu)
-   [CPU/GPU 同步](#cpugpu-synchronization)
-   [资源绑定](#resource-binding)
-   [资源状态](#resource-state)
-   [Swapchains](#swapchains)
-   [修复了函数呈现](#fixed-function-rendering)
-   [几率 and 结束](#odds-and-ends)
-   [相关的主题](#related-topics)

## <a name="committed-resources"></a>已提交的资源

使用 Direct3D 11 中的以下接口创建的对象转换为所谓的"已提交的资源"在 Direct3D 12 中。 提交的资源是具有虚拟地址空间和与之关联的物理页的资源。 这是 Microsoft Windows 设备驱动程序 2 (WDD2) 内存模型，Direct3D 12 中所基于的概念。

Direct3D 11 资源：

-   [**ID3D11Resource**](https://msdn.microsoft.com/library/windows/desktop/ff476584)
-   [**ID3D11Buffer** ](https://msdn.microsoft.com/library/windows/desktop/ff476351)并[ **ID3D11Device::CreateBuffer**](https://msdn.microsoft.com/library/windows/desktop/ff476501)
-   [**ID3D11Texture1D** ](https://msdn.microsoft.com/library/windows/desktop/ff476633)并[ **ID3D11Device:CreateTexture1D**](https://msdn.microsoft.com/library/windows/desktop/ff476520)
-   [**ID3D11Texture2D** ](https://msdn.microsoft.com/library/windows/desktop/ff476635)并[ **ID3D11Device::CreateTexture2D**](https://msdn.microsoft.com/library/windows/desktop/ff476521)
-   [**ID3D11Texture3D** ](https://msdn.microsoft.com/library/windows/desktop/ff476637)并[ **ID3D11Device::CreateTexture3D**](https://msdn.microsoft.com/library/windows/desktop/ff476522)

在 Direct3D 12 中这些全都由表示[ **ID3D12Resource** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12resource)并[ **ID3D12Device::CreateCommittedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommittedresource)。

## <a name="reserved-resources"></a>保留的资源

保留的资源是仅虚拟地址空间已分配的物理内存分配的位置不调用之前的资源[ **ID3D12Device::CreateHeap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createheap)。 这是实质上是相同 Direct3D 11 中的平铺资源的概念。

标志 ([**D3D11\_资源\_杂项\_标志**](https://msdn.microsoft.com/library/windows/desktop/ff476203)) 在 Direct3D 11 中用于设置平铺资源，然后将它们映射到物理内存。

-   D3D11\_资源\_杂项\_平铺
-   D3D11\_资源\_杂项\_磁贴\_池

## <a name="uploading-data"></a>将数据上传

在 Direct3D 11 是一条时间线的外观 (调用以下序列，如使用的数据初始化[ **D3D11\_SUBRESOURCE\_数据**](https://msdn.microsoft.com/library/windows/desktop/ff476220)，然后调用[ **ID3D11DeviceContext::UpdateSubresource**](https://msdn.microsoft.com/library/windows/desktop/ff476486)，，然后调用[ **ID3D11DeviceContext::Map**](https://msdn.microsoft.com/library/windows/desktop/ff476457))。 创建数据的副本数并不明显到 Direct3D 11 开发人员。

在 Direct3D 12 中有两个时间线，GPU 时间线 (通过调用设置[ **CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)，并[ **CopyBufferRegion** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion)从可映射内存） 和 CPU 时间线 (由对[**地图**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map))。 （在 d3dx12.h 文件） 提供帮助程序函数调用[ **Updatesubresources** ](updatesubresources1.md)使用共享的时间线。 此帮助器函数，另一个使用多个变体[ **ID3D12Device::GetCopyableFootprints**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getcopyablefootprints)，另一个使用堆分配的机制，以及另一个使用堆栈分配机制。 这些帮助器函数将资源复制到 GPU 和 CPU，通过内存中间临时区域。

通常在 GPU 和 CPU 具有自己复制到其自己的时间线相关联的资源。 共享的时间线方法同样维护两个副本。

## <a name="shaders-and-shader-objects"></a>着色器和着色器对象

在 Direct3D 11 没有创建着色器和状态对象，以及使用这些对象的状态设置大量[ **ID3D11Device** ](https://msdn.microsoft.com/library/windows/desktop/ff476379)创建方法和[ **ID3D11DeviceContext** ](https://msdn.microsoft.com/library/windows/desktop/ff476385)设置方法。 通常会对这些方法，然后由组合在一起在绘制时的驱动程序，用于设置正确的管道的状态进行大量调用。

在 Direct3D 12 中此设置管道的状态已合并为单个对象 ([**CreateComputePipelineState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcomputepipelinestate)计算引擎，并且[ **CreateGraphicsPipelineState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-creategraphicspipelinestate)图形引擎) 之前的绘图调用通过调用, 然后附加到命令列表[ **SetPipelineState** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate).

这些调用将为所有单独的调用设置着色器、 输入布局、 状态、 光栅器状态、 深度模具状态等，Direct3D 11 中的 blend。

## <a name="submitting-work-to-the-gpu"></a>提交到 GPU 的工作

在 Direct3D 11 中没有几乎无法控制实际工作的提交方式，但通过启用一些控件很大程度上处理由驱动程序[ **ID3D11DeviceContext::Flush** ](https://msdn.microsoft.com/library/windows/desktop/ff476425)并[ **IDXGISwapChain1::Present1** ](https://msdn.microsoft.com/library/windows/desktop/hh446797)调用。

在 Direct3D 12 工作提交是非常显式和受控的应用。 提交工作的主要构造是[ **ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)，它用于记录应用的所有命令 （和在概念上非常类似于 ID3D11 延迟上下文）。 命令列表的后备存储提供的[ **ID3D12CommandAllocator**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandallocator)，这使应用能够通过实际公开内存管理命令列表的内存使用率的 Direct3D12 驱动程序将用于存储命令列表。

最后[ **ID3D12CommandQueue** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue)是先入先出队列，用于存储此命令列出以提交到 GPU 正确的顺序。 仅在一个命令列表完成后在 GPU 上的执行，将下一步从队列的命令列表提交驱动程序。

在 Direct3D 11 不存在命令队列的概念。

## <a name="cpugpu-synchronization"></a>CPU/GPU 同步

在 Direct3D 11 CPU/GPU 同步是很大程度上自动进行的并不需要维护的物理内存状态的应用。

在 Direct3D 12 应用程序必须显式管理两条时间线 （CPU 和 GPU）。 这要求需要由应用程序中，哪些资源是由 GPU，以及多长时间，必须维护信息。 这也意味着该应用程序负责确保资源 （已提交的资源，堆的示例命令分配器） 的内容不变，除非 GPU 已完成使用它们。

同步时间线的主要对象是[ **ID3D12Fence** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12fence)对象。 界定的操作是 failry 简单，它们使 GPU 时它已完成一个任务发出信号通知。 GPU 和 CPU 可以同时发出信号，并同时上可等待界定。

通常的方法是，提交时执行的命令列表，fence 信号传输完成后 gpu （当它完成读取的数据），启用到 CPU 重复使用或销毁资源。

在 Direct3D 11 [ **ID3D11DeviceContext::Map** ](https://msdn.microsoft.com/library/windows/desktop/ff476457)标志 D3D11\_映射\_编写\_放弃实质上是无限的内存提供作为处理每个资源应用程序无法写入 （称为"重命名"的过程）。 在 Direct3D 12 再次该过程将显式： 需分配，任何额外的内存，因此界定应进行同步操作。 环形缓冲区 （包含较大的缓冲区） 可能是一种好方法，为此，请参阅中的环缓冲区方案[Fence 基于资源管理](fence-based-resource-management.md)。

![使用环形缓冲区](images/ring-buffer-1.png)

## <a name="resource-binding"></a>资源绑定

在 Direct3D 11 （着色器资源视图、 呈现器目标视图等），视图很大程度上已替换为在 Direct3D 12 中的一个描述符概念。 在 Direct3D 12 中仍存在创建方法 (如[ **CreateShaderResourceView** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createshaderresourceview)并[ **CreateRenderTargetView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrendertargetview))，它描述符堆创建后，若要在堆中写入数据后调用。 Direct3D 12 中的绑定现在由描述符句柄所述的根签名，并使用提交[ **SetGraphicsRootDescriptorTable** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootdescriptortable)或[ **SetComputeRootDescriptorTable** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootdescriptortable)方法。

根签名的详细信息表之间映射根签名槽数和描述符，其中描述符表可以包含对资源提供给顶点着色器、 像素着色器和其他着色器，如常量缓冲区的引用着色器资源视图和取样器。 这种灵活性断开在 Direct3D 12 中，与不同的是 Direct3D 11 API 绑定空间 HLSL 寄存器空间在它们之间的一对一映射。

此系统的影响之一是应用程序是负责重命名描述符表，这使开发人员能够了解更改甚至每个绘图调用的单个描述符的性能成本。

Direct3D 12 的新功能是应用程序可以控制哪些着色器阶段之间共享的描述符。 在 Direct3D 11 中所有着色器阶段之间共享资源，例如 Uav。 通过启用描述符对于某些禁用着色器阶段，由已禁用的描述符的寄存器均可由描述符为特定的着色器阶段启用它们。

下表显示了示例根签名。



| 根参数槽 | 描述符表项         |
|---------------------|--------------------------------|
| 0                   | VS 描述符范围 b0 b13     |
| 1                   | VS 描述符范围 t0 t127    |
| 2                   | VS 描述符范围 s0 s16     |
| 3                   | PS 描述符范围 b0-b13     |
| ...                 |                                |
| 14                  | DS 描述符范围 s0-16      |
| 15                  | 共享的描述符范围 u0 u63 |



 

## <a name="resource-state"></a>资源状态

在 Direct3D 11 资源在应用程序中，但该驱动程序不保持状态。

在 Direct3D 12 中维护资源状态将变为应用，以实现完全并行命令列表的录制中的责任： 应用程序必须处理命令列表 （这可以并行进行） 的录制时间线和执行时间线这必须是连续的。

资源状态转换由[ **ResourceBarrier** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)方法。 更改资源使用情况时，主要应用程序必须通知该驱动程序。 例如，如果资源正用作呈现器目标，则要用作下一步的顶点着色器的输入绘图调用，则这可能需要在 GPU 操作完成之前处理顶点着色器呈现器目标操作短停滞。

此系统支持图形管道，以及缓存刷新数和可能是某些内存布局的更改 （如呈现深度模具视图解压缩到的目标视图） 的细粒度的同步 （的 GPU 停止）。

这被称为转换屏障。 有其他类型的障碍，在 Direct3D 11 [ **ID3D11DeviceContext2::TiledResourceBarrier** ](https://msdn.microsoft.com/library/windows/desktop/dn280507)启用相同的物理内存将由两个不同的平铺资源。 在 Direct3D 12 中这被称为"锯齿 barrier"。 障碍，别名可用于在 Direct3D 12 中的平铺和放置资源。 此外还有 UAV 屏障。 在 Direct3D 11 中所有 UAV 调度和描述的操作所需进行序列化，即使这些操作可通过管道或并行工作。 为 Direct3D 12 UAV 屏障添加并删除此限制。 UAV 屏障将确保 UAV 操作是连续的因此，如果第二个操作要求第一个完成，则第二个将被强制等待通过屏障的添加。 针对 Uav 的默认操作就是，尽可能快地将继续操作。

很明显，有性能提升如果可以并行化工作负荷。

## <a name="swapchains"></a>交换链

DXGI 交换链是 Direct3D 11 和 12 中的交换链的基础。 有一些细微的差别，Direct3D 11 中三种类型的交换链是按顺序、 放弃时和翻转\_顺序。 Direct3D 12 中有两个类型：翻转\_顺序与翻转\_放弃。

在 Direct3D 11 没有自动的后台缓冲区旋转： 只有一个呈现目标视图所需的后台缓冲区 0。 在 Direct3D 12 中缓冲区旋转是显式的需要有每个后台缓冲区的渲染目标视图。 使用[ **IDXGISwapChain3::GetCurrentBackBufferIndex** ](https://msdn.microsoft.com/library/windows/desktop/dn903675)方法来选择要呈现到哪一种。 再次此更大的灵活性使更大的并行化。

## <a name="fixed-function-rendering"></a>修复了函数呈现

在 Direct3D 11 中没有的几种方法的简化各种更高级别的操作，如[ **GenerateMips** ](https://msdn.microsoft.com/library/windows/desktop/ff476426) （创建完整的 mip 链） 和[ **DrawAuto** ](https://msdn.microsoft.com/library/windows/desktop/ff476408) （使用与着色器输入，而无需从应用的更多输入流输出）。 这些方法不可用在 Direct3D 12 中，应用程序需要通过创建着色器以执行这些处理这些操作。

## <a name="odds-and-ends"></a>几率 and 结束

下表显示了大量类似之间 Direct3D 11 和 12，但并不完全相同的功能。



| Direct3D 11                                                                            | Direct3D 12                                                                                                                                                                                                                                                                                                                                                                                                                              |
|----------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [**ID3D11Query**](https://msdn.microsoft.com/library/windows/desktop/ff476578)                                              | [**ID3D12QueryHeap** ](https://msdn.microsoft.com/en-us/library/Dn891447(v=VS.85).aspx)允许查询组合在一起，从而降低了成本。                                                                                                                                                                                                                                                                                                                                     |
| [**ID3D11Predicate**](https://msdn.microsoft.com/library/windows/desktop/ff476577)                                      | 断言而现已启用通过将数据放在完全透明的缓冲区。 Direct3D 11 [ **ID3D11Predicate** ](https://msdn.microsoft.com/library/windows/desktop/ff476577)对象被替换[ **ID3D12Resource::Map**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)，它必须遵循调用[ **ResolveQueryData** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvequerydata)和使用一种保护要等待的时间才能准备就绪的数据的 GPU 同步操作。 请参阅[断言而](predication.md)。 |
| UAV/SO 隐藏计数器                                                                  | 应用负责分配和管理等/UAV 计数器。 请参阅[Stream 输出计数器](stream-output-counters.md)并[UAV 计数器](uav-counters.md)。                                                                                                                                                                                                                                                             |
| 资源动态 MinLOD （最小级别的详细信息）                                       | 这已移至 SRV 描述符静态 MinLOD。                                                                                                                                                                                                                                                                                                                                                                                 |
| 绘制\*间接 /[**DispatchIndirect**](https://msdn.microsoft.com/library/windows/desktop/ff476406) | 绘制间接方法将合并成一个[ **ExecuteIndirect** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executeindirect)方法。                                                                                                                                                                                                                                                                                                        |
| 交错 DepthStencil 格式                                                   | DepthStencil 格式是平面的。 例如 24 位深度模具的 8 位的格式将存储在 24/8/24/8 的格式...在 Direct3D 11 中，而是作为 24/24/24 等...跟 8/8/8...在 Direct3D 12 中。 请注意，每个平面是 D3D12 中的其自身子资源 (请参阅[子](subresources.md))。                                                                                                                    |
| [**ResizeTilePool**](https://msdn.microsoft.com/library/windows/desktop/dn280505)                   | 保留的资源可以映射到多个堆。 如果将已在 D3D11 中增加了磁贴池，可以分配的附加堆，D3D12 中。                                                                                                                                                                                                                                                                               |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[DirectX 高级学习视频教程：DirectX 11 到 DirectX 12 移植指南](https://www.youtube.com/watch?v=BV64mdOCgZo)
</dt> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> <dt>

[使用 Direct3D 11，Direct3D 10 和 Direct2D](direct3d-12-interop.md)
</dt> </dl>

 

 




