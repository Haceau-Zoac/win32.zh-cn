---
title: 从 Direct3D 11 移植到 Direct3D 12
description: 本部分提供有关从自定义的 Direct3D 11 图形引擎移植到 Direct3D 12 的一些指导。
ms.assetid: 9EB4AC6B-AFDD-4673-8EB3-54272C151784
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 14b5bc6784d6f96c3c1599a601a57bf68b0d612d
ms.sourcegitcommit: 592c9bbd22ba69802dc353bcb5eb30699f9e9403
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2020
ms.locfileid: "88644070"
---
# <a name="porting-from-direct3d-11-to-direct3d-12"></a>从 Direct3D 11 移植到 Direct3D 12

本部分提供有关从自定义的 Direct3D 11 图形引擎移植到 Direct3D 12 的一些指导。

-   [设备创建](#device-creation)
-   [提交的资源](#committed-resources)
-   [预留的资源](#reserved-resources)
-   [上传数据](#uploading-data)
-   [着色器和着色器对象](#shaders-and-shader-objects)
-   [将工作提交到 GPU](#submitting-work-to-the-gpu)
-   [CPU/GPU 同步](#cpugpu-synchronization)
-   [资源绑定](#resource-binding)
-   [资源状态](#resource-state)
-   [交换链](#swapchains)
-   [已修复的函数渲染](#fixed-function-rendering)
-   [杂项](#odds-and-ends)
-   [相关主题](#related-topics)

## <a name="device-creation"></a>设备创建

Direct3D 11 和 Direct3D 12 均共享 (类似设备创建模式。 现有的 Direct3D 12 驱动程序全部 **D3D_FEATURE_LEVEL_11_0** 或更好，因此可以忽略较旧的功能级别和关联的 lmitations。

另外请记住，使用 Direct3D 12 时，应使用 DXGI 接口显式枚举设备信息。 在 Direct3D 11 中，你可以从 Direct3D 设备 *链接回* DXGI 设备，并且不支持 direct3d 12。

通过提供从 **IDXGIFcatory4：： EnumWarpAdapter**获取的显式适配器，可以在 Direct3D 12 上创建弯曲的软件设备。 Direct3D 12 的弯曲设备仅在启用了 **图形工具** 可选功能的系统上可用。

> [!NOTE]
> 没有等效于 **D3D11CreateDeviceAndSwapChain**。 即使在 Direct3D 11 中，我们也会反对使用此函数，因为在不同的步骤中创建设备和存在通常会更好。

## <a name="committed-resources"></a>提交的资源

使用 Direct3D 11 中以下接口创建的对象将转换为 Direct3D 12 中称作“提交的资源”的对象。 提交的资源是指具有关联的虚拟地址空间和物理页面的资源。 这是 Direct3D 12 所基于的 Microsoft Windows 设备驱动程序 2 (WDD2) 内存模型中的一个概念。

Direct3D 11 资源：

-   [**ID3D11Resource**](/windows/win32/api/d3d11/nn-d3d11-id3d11resource)
-   [**ID3D11Buffer**](/windows/win32/api/d3d11/nn-d3d11-id3d11buffer) 和 [**ID3D11Device::CreateBuffer**](/windows/win32/api/d3d11/nf-d3d11-id3d11device-createbuffer)
-   [**ID3D11Texture1D**](/windows/win32/api/d3d11/nn-d3d11-id3d11texture1d) 和 [**ID3D11Device:CreateTexture1D**](/windows/win32/api/d3d11/nf-d3d11-id3d11device-createtexture1d)
-   [**ID3D11Texture2D**](/windows/win32/api/d3d11/nn-d3d11-id3d11texture2d) 和 [**ID3D11Device::CreateTexture2D**](/windows/win32/api/d3d11/nf-d3d11-id3d11device-createtexture2d)
-   [**ID3D11Texture3D**](/windows/win32/api/d3d11/nn-d3d11-id3d11texture3d) 和 [**ID3D11Device::CreateTexture3D**](/windows/win32/api/d3d11/nf-d3d11-id3d11device-createtexture3d)

在 Direct3D 12 中，所有这些资源由 [**ID3D12Resource**](/windows/win32/api/d3d12/nn-d3d12-id3d12resource) 和 [**ID3D12Device::CreateCommittedResource**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcommittedresource) 表示。

## <a name="reserved-resources"></a>预留的资源

预留的资源是仅分配了虚拟地址空间的资源，在调用 [**ID3D12Device::CreateHeap**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createheap) 之前，不会分配物理内存。 这实质上与 Direct3D 11 中“图块化资源”的概念相同。

Direct3D 11 中使用标志 ([**D3D11\_RESOURCE\_MISC\_FLAG**](/windows/win32/api/d3d11/ne-d3d11-d3d11_resource_misc_flag)) 设置图块化资源，然后将其映射到物理内存。

-   D3D11\_RESOURCE\_MISC\_TILED
-   D3D11\_RESOURCE\_MISC\_TILE\_POOL

## <a name="uploading-data"></a>上传数据

Direct3D 11 中显示单条时间线（调用后接一个序列，例如使用 [**D3D11\_SUBRESOURCE\_DATA**](/windows/win32/api/d3d11/ns-d3d11-d3d11_subresource_data) 初始化的数据，再调用 [**ID3D11DeviceContext::UpdateSubresource**](/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-updatesubresource)，然后调用 [**ID3D11DeviceContext::Map**](/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-map)）。 Direct3D 11 开发人员无法清楚地看到所创建的数据副本数。

Direct3D 12 中有两条时间线：GPU 时间线（通过从可映射内存调用 [**CopyTextureRegion**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion) 和 [**CopyBufferRegion**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion) 来设置） 和 CPU 时间线（通过调用 [**Map**](/windows/win32/api/d3d12/nf-d3d12-id3d12resource-map) 来确定）。 提供使用共享时间线的名为 [**Updatesubresources**](updatesubresources1.md) 的帮助器函数（在 d3dx12.h 文件中）。 此帮助器函数有多个变体，其中一个变体使用 [**ID3D12Device::GetCopyableFootprints**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-getcopyablefootprints)，另一个变体使用堆分配机制，还有一个变体使用堆栈分配机制。 这些帮助器函数通过内存的中间暂存区域将资源复制到 GPU 和 CPU。

通常，GPU 和 CPU 都具有自身的资源副本，该副本与其自身的时间线相关联。 类似地，共享时间线方法维护两个副本。

## <a name="shaders-and-shader-objects"></a>着色器和着色器对象

Direct3D 11 中存在许多着色器和状态对象创建创建，以及使用 [**ID3D11Device**](/windows/win32/api/d3d11/nn-d3d11-id3d11device) 创建方法和 [**ID3D11DeviceContext**](/windows/win32/api/d3d11/nn-d3d11-id3d11devicecontext) 设置方法设置这些对象的状态的操作。 通常会对这些方法发出大量的调用，在绘制时，驱动程序会合并这些调用以设置正确的管道状态。

在 Direct3D 12 中，此管道状态设置已合并成单个对象（计算引擎的 [**CreateComputePipelineState**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcomputepipelinestate)，以及图形引擎的 [**CreateGraphicsPipelineState**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-creategraphicspipelinestate)），然后，在通过调用 [**SetPipelineState**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate) 发出绘制调用之前附加到命令列表。

在 Direct3D 11 中，这些调用会将所有调用替换为设置着色器、输入布局、blend 状态、光栅器状态、深度模具状态等。

- 设备11方法： ``CreateInputLayout`` 、 ``CreateXShader`` 、 ``CreateDepthStencilState`` 、andD ``CreateRasterizerState`` 。
- 设备上下文11方法：  ``IASetInputLayout`` 、 ``xxSetShader`` 、 ``OMSetBlendState`` 、 ``OMSetDepthStencilState`` 和 ``RSSetState`` 。

尽管 Direct3D 12 可以支持较早编译的着色器 blob，但着色器应该使用 FXC.EXE/D3DCompile Api 的着色器模型5.1 或使用 DXIL DXC 编译器的着色器模型6构建。 应通过 [**CheckFeatureSupport**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport) 和 **D3D12_FEATURE_SHADER_MODEL**验证着色器模型6支持。

## <a name="submitting-work-to-the-gpu"></a>将工作提交到 GPU

在 Direct3D 11 中，几乎无法控制工作的实际提交方式，此操作在很大程度上由驱动程序处理，不过，可以通过 [**ID3D11DeviceContext::Flush**](/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-flush) 和 [**IDXGISwapChain1::Present1**](/windows/win32/api/dxgi1_2/nf-dxgi1_2-idxgiswapchain1-present1) 调用来启用某种控制。

在 Direct3D 12 中，工作提交是非常明确的，并受应用的控制。 用于提交工作的主要构造是 [**ID3D12GraphicsCommandList**](/windows/win32/api/d3d12/nn-d3d12-id3d12graphicscommandlist)，它用于记录所有应用命令（在概念上非常类似于 ID3D11 中的延迟上下文）。 命令列表的后备存储由 [**ID3D12CommandAllocator**](/windows/win32/api/d3d12/nn-d3d12-id3d12commandallocator) 提供，它可以让应用通过实际公开 Direct3D 12 驱动程序用于存储命令列表的内存，来管理命令列表的内存利用率。

最后，[**ID3D12CommandQueue**](/windows/win32/api/d3d12/nn-d3d12-id3d12commandqueue) 是一个先入先出队列，可存储要提交到 GPU 的命令列表的正确顺序。 仅当一个命令列表已 GPU 上完成执行时，驱动程序才会提交队列中的下一个命令列表。

在 Direct3D 11 中，命令队列没有明确的概念。 在 Direct3D 12 的常见设置中，当前框架的当前打开 **D3D12_COMMAND_LIST_TYPE_DIRECT** 命令列表可视为与 Direct3D 11 立即上下文类似。 这提供了许多相同的功能。


| D3D11DeviceContext                  | ID3D12GraphicsCommand 列表     |
|-------------------------------------|--------------------------------|
| ClearDepthStencilView               | ClearDepthStencilView          |
| ClearRenderTargetView               | ClearRenderTargetView          |
| ClearUnorderedAccess*               | ClearUnorderedAccess*          |
| 绘图，DrawInstanced                 | DrawInstanced                  |
| DrawIndexed、DrawIndexedInstanced   | DrawIndexedInstanced           |
| Dispatch                            | Dispatch                       |
| IASetInputLayout、xxSetShader 等。 | SetPipelineState               |
| OMSetBlendState                     | OMSetBlendFactor               |
| OMSetDepthStencilState              | OMSetStencilRef                |
| OMSetRenderTargets                  | OMSetRenderTargets             |
| RSSetViewports                      | RSSetViewports                 |
| RSSetScissorRects                   | RSSetScissorRects              |
| IASetPrimitiveTopology              | IASetPrimitiveTopology         |
| IASetVertexBuffers                  | IASetVertexBuffers             |
| IASetIndexBuffer                    | IASetIndexBuffer               |
| ResolveSubresource                  | ResolveSubresource             |
| CopySubresourceRegion               | CopyBufferRegion               |
| UpdateSubresource                   | CopyTextureRegion              |
| CopyResource                        | CopyResource                   |

> [!NOTE]
> 使用 **D3D12_COMMAND_LIST_TYPE_BUNDLE** 创建的命令列表 (类似为延迟上下文。 Direct3D 12 还支持 abiilty 通过**D3D12_COMMAND_LIST_TYPE_COPY**和**D3D12_COMMAND_LIST_TYPE_COMPUTE**命令列表类型同时访问*直接上下文*的某些功能。

## <a name="cpugpu-synchronization"></a>CPU/GPU 同步

在 Direct3D 11 中，CPU/GPU 同步在很大程度上是自动化的，应用无需维护物理内存的状态。

在 Direct3D 12 中，应用必须显式管理两条时间线（CPU 和 GPU）。 因此，应用需要维护有关 GPU 需要哪些资源，以及需要使用多长时间的信息。 这也意味着，应用需负责确保资源内容（例如提交的资源、堆、命令分配器）在 GPU 用完它们之前不会更改。

用于同步时间线的主要对象是 [**ID3D12Fence**](/windows/win32/api/d3d12/nn-d3d12-id3d12fence) 对象。 围栏操作相当简单。围栏可让 GPU 在完成任务时发出信号。 GPU 和 CPU 都可发出信号，并且都会等待围栏。

常用的方法是，在提交某个命令列表供执行时，GPU 会在完成时（读完数据时）传输围栏信号，使 CPU 能够重复使用或销毁资源。

在 Direct3D 11 中，[**ID3D11DeviceContext::Map**](/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-map) 标志 D3D11\_MAP\_WRITE\_DISCARD 实质上会将每个资源视为应用可以写入到的内存的无尽供应（称为“重命名”的进程）。 在 Direct3D 12 中，该进程同样很明确：需要分配额外的内存，并且应该使用围栏来同步操作。 环形缓冲区（由较大的缓冲区组成）可能是解决此问题的适当方法，具体请参阅[基于围栏的资源管理](fence-based-resource-management.md)中的环形缓冲区方案。

![使用环形缓冲区](images/ring-buffer-1.png)

## <a name="resource-binding"></a>资源绑定

Direct3D 11 中的视图（着色器资源视图、渲染器目标视图等）基本上已被 Direct3D 12 中的描述符概念取代。 Direct3D 12 中仍然存在创建方法（例如 [**CreateShaderResourceView**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createshaderresourceview) 和 [**CreateRenderTargetView**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createrendertargetview)），创建描述符堆后会调用这些方法，以在堆中写入数据。 Direct3D 12 中的绑定现在由根签名中描述的描述符句柄处理，使用 [**SetGraphicsRootDescriptorTable**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootdescriptortable) 或 [**SetComputeRootDescriptorTable**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootdescriptortable) 方法提交。

根签名详细描述根签名槽数与描述符表之间的映射，其中，描述符表可以包含对顶点着色器、像素着色器和其他着色器（例如常量缓冲区、着色器资源视图和采样器）可用资源的引用。 这种灵活性可将 HLSL 寄存器空间从 Direct3D 12 中的 API 绑定空间断开连接。Direct3D 11 则与此不同，其中的这些资源存在一对一的映射。

此系统的影响之一在于，应用负责重命名描述符表，使开发人员能够了解即使在执行每个绘制调用时更改一个描述符的性能开销比。

Direct3D 12 的一项新功能是，应用可以控制哪些描述符在哪些着色器阶段之间共享。 在 Direct3D 11 中，UAV 等资源在所有着色器阶段之间共享。 通过启用要对某些着色器阶段禁用的描述符，已禁用的描述符使用的寄存器可供为特定着色器阶段启用的描述符使用。

下表显示了一个示例根签名。



| 根参数槽 | 描述符表条目         |
|---------------------|--------------------------------|
| 0                   | VS 描述符范围 b0-b13     |
| 1                   | VS 描述符范围 t0-t127    |
| 2                   | VS 描述符范围 s0-s16     |
| 3                   | PS 描述符范围 b0-b13     |
| ...                 |                                |
| 14                  | DS 描述符范围 s0-16      |
| 15                  | 共享描述符范围 u0-u63 |



 

## <a name="resource-state"></a>资源状态

在 Direct3D 11 中，资源状态由驱动程序维护，而不是由应用维护。

在 Direct3D 12 中，资源状态的维护由应用负责，以便在记录命令列表时实现完全并行度：应用必须处理命令列表的记录时间线（可以并行进行），而执行时间线必须是有序的。

资源状态转换由 [**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 方法处理。 从根本上讲，当资源使用率发生变化时，应用必须通知驱动程序。 例如，如果将某个资源用作渲染器目标，然后在下一次执行绘制调用时又将它用作顶点着色器的输入，则可能需要短暂停滞 GPU 操作，以便在处理顶点着色器之前先完成渲染器目标操作。

此系统可以实现图形管道的粒度级同步（GPU 停滞），以及缓存刷新和可能的某些内存布局更改（例如渲染器目标视图到深度模具视图的解压缩）。

这称为“转换屏障”。 Direct3D 11 中还有其他类型的屏障，两个不同的图块化资源可以通过 [**ID3D11DeviceContext2::TiledResourceBarrier**](/windows/win32/api/d3d11_2/nf-d3d11_2-id3d11devicecontext2-tiledresourcebarrier) 使用相同的物理内存。 在 Direct3D 12 中，这称为“失真屏障”。 可对 Direct3D 12 中的图块和定位资源使用失真屏障。 此外还有 UAV 屏障。 在 Direct3D 11 中，所有 UAV 调度和绘制操作都需要序列化，即使这些操作可以构成管道或并行工作。 在 Direct3D 12 中，已通过添加 UAV 屏障消除了此限制。 UAV 屏障确保 UAV 操作是有序的，因此，如果第二个操作要求第一个操作先完成，则添加屏障会强制要求第二个操作等待第一个操作完成。 UAV 的默认操作仅仅是让操作尽快继续。

很明显，如果可以并行化工作负荷，则性能将会提升。

## <a name="swapchains"></a>交换链

DXGI 交换链是 Direct3D 11 和 12 中的交换链的基础。 但两者存在一些细微的差别，在 Direct3D 11 中，三种类型的交换链为 SEQUENTIAL、DISCARD 和 FLIP\_SEQUENTIAL。 Direct3D 12 只有两种类型：翻转 \_ 顺序和翻转 \_ 放弃。 如上所述，应通过 **IDXGIFactory4**或更高版本显式创建存在，并对任何适配器枚举使用同一接口。

在 Direct3D 11 中存在自动反向缓冲区轮转：反向缓冲区 0 只需一个渲染器目标视图。 在 Direct3D 12 中，缓冲区轮转是显式的，每个反向缓冲区都需要一个渲染器目标视图。 使用 [**IDXGISwapChain3::GetCurrentBackBufferIndex**](/windows/win32/api/dxgi1_4/nf-dxgi1_4-idxgiswapchain3-getcurrentbackbufferindex) 方法选择要渲染的缓冲区。 同样，更大的这种灵活性可以实现更高的并行度。

> [!NOTE]
> 尽管有很多方法可以设置应用程序，但应用程序的每个交换链缓冲区一般都有一个 **ID3D12CommandAllocator** 。 这样，应用程序便可以继续为下一帧构建一组命令，而 GPU 呈现以前的一组命令。

## <a name="fixed-function-rendering"></a>已修复的函数渲染

在 Direct3D 11 中，有些方法简化了各种更高级别的操作，例如 [**GenerateMips**](/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-generatemips)（创建完整的 mip 链）和 [**DrawAuto**](/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-drawauto)（使用流输出作为着色器输入，且无需应用进一步提供输入）。 这些方法在 Direct3D 12 中不可用，应用需要通过创建执行这些操作的着色器来处理这些操作。

## <a name="odds-and-ends"></a>杂项

下表显示了在 Direct3D 11 和 12 中类似的，但不完全相同的一些功能。



| Direct3D 11                                                                            | Direct3D 12                                                                                                                                                                                                                                                                                                                                                                                                                              |
|----------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [**ID3D11Query**](/windows/win32/api/d3d11/nn-d3d11-id3d11query)                                              | [**ID3D12QueryHeap**](/windows/win32/api/d3d12/nn-d3d12-id3d12queryheap) 允许将查询分组在一起，以降低成本。                                                                                                                                                                                                                                                                                                                                     |
| [**ID3D11Predicate**](/windows/win32/api/d3d11/nn-d3d11-id3d11predicate)                                      | 现在，可以通过在完全透明的缓冲区中存储数据，来实现断言。 Direct3D 11 [**ID3D11Predicate**](/windows/win32/api/d3d11/nn-d3d11-id3d11predicate) 对象已由 [**ID3D12Resource::Map**](/windows/win32/api/d3d12/nf-d3d12-id3d12resource-map) 取代，后者必须在调用 [**ResolveQueryData**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvequerydata) 并完成使用围栏等待数据准备就绪的 GPU 同步操作之后执行。 请参阅[断言](predication.md)。 |
| UAV/SO 隐藏计数器                                                                  | 应用负责分配和管理 SO/UAV 计数器。 请参阅[流输出计数器](stream-output-counters.md)和 [UAV 计数器](uav-counters.md)。                                                                                                                                                                                                                                                             |
| 资源动态 MinLOD（最低详细程度）                                       | 此功能已过渡到 SRV 描述符静态 MinLOD。                                                                                                                                                                                                                                                                                                                                                                                 |
| Draw\*Indirect/[**DispatchIndirect**](/windows/win32/api/d3d11/nf-d3d11-id3d11devicecontext-dispatchindirect) | 绘制间接方法已全部合并成一个 [**ExecuteIndirect**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executeindirect) 方法。                                                                                                                                                                                                                                                                                                        |
| DepthStencil 格式是交错式                                                   | DepthStencil 格式是平面式。 例如， 24 位深度、8 位模具的格式在 Direct3D 11 中将以 24/8/24/8... 等格式存储，但在 Direct3D 12 中将以 24/24/24... 后接 8/8/8... 的格式存储。 请注意，每个平面在 D3D12 中具有自身的子资源（请参阅[子资源](subresources.md)）。                                                                                                                    |
| [**ResizeTilePool**](/windows/win32/api/d3d11_2/nf-d3d11_2-id3d11devicecontext2-resizetilepool)                   | 保留的资源可映射到多个堆。 如果图块池已在 D3D11 中增大，可以改为在 D3D12 中分配附加的堆。                                                                                                                                                                                                                                                                               |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[DirectX 高级学习视频教程： DirectX 11 到 DirectX 12 移植指南](https://www.youtube.com/watch?v=BV64mdOCgZo)
</dt> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> <dt>

[使用 Direct3D 11、Direct3D 10 和 Direct2D](direct3d-12-interop.md)
</dt> </dl>