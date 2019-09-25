---
title: 同步和多引擎
description: 大多数新式 GPU 包含多个用于提供专用功能的独立引擎。
ms.assetid: 93903F50-A6CA-41C2-863D-68D645586B4C
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 0e02af4565914a7c03b0d71a8958e6606987dd79
ms.sourcegitcommit: 4c947260f53655bc7a8a4670ec1b66766413baf1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2019
ms.locfileid: "71262328"
---
# <a name="synchronization-and-multi-engine"></a>同步和多引擎

大多数新式 GPU 包含多个用于提供专用功能的独立引擎。 许多 GPU 拥有一个或多个专用复制引擎和一个计算引擎，通常与 3D 引擎不同。 这些引擎可以彼此并行执行命令。 Direct3D 12 使用队列和命令列表提供对 3D、计算和复制引擎的精细访问。

-   [GPU 引擎](#gpu-engines)
-   [多引擎方案](#multi-engine-scenarios)
-   [同步 API](#synchronization-apis)
    -   [设备和队列](#devices-and-queues)
    -   [复制和计算命令列表](#copy-and-compute-command-lists)
-   [管道化计算和图形示例](#pipelined-compute-and-graphics-example)
-   [异步计算和图形示例](#asynchronous-compute-and-graphics-example)
-   [多队列资源访问](#multi-queue-resource-access)
-   [相关主题](#related-topics)

## <a name="gpu-engines"></a>GPU 引擎

下图显示了某个资产的 CPU 线程，其中的每个线程填充一个或多个复制、计算和 3D 队列。 3D 队列可以驱动所有三个 GPU 引擎，计算队列可以驱动计算和复制引擎，复制队列只是复制引擎。

由于不同的线程会填充队列，且不能简单地保证执行顺序，因此，需要根据资产的需要提供同步机制。

![四个线程正在向三个队列发送命令](images/gpu-engines.png)

下图演示了资产如何在多个 GPU 引擎之间安排工作，包括必要的引擎间同步：其中显示了每个引擎的工作负荷与引擎间的依赖关系。 在此示例中，复制引擎首先复制渲染所需的一些几何体。 3D 引擎等待这些复制操作完成，并基于该几何体渲染预先通道。 然后，计算引擎可以使用此通道。 计算引擎[**调度**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch)以及复制引擎中多个纹理复制操作的结果供 3D 引擎用于最终的[**绘制**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced)调用。

![复制、图形和计算引擎的通信](images/gpu-sync.png)

以下伪代码演示资产如何提交此类工作负荷。

``` syntax
// Get per-engine contexts.  Note that multiple queues may be exposed
// per engine, however that design is not reflected here.
copyEngine = device->GetCopyEngineContext();
renderEngine = device->GetRenderEngineContext();
computeEngine = device->GetComputeEngineContext();
copyEngine->CopyResource(geometry, ...); // copy geometry
copyEngine->Signal(copyFence, 101);
copyEngine->CopyResource(tex1, ...); // copy textures
copyEngine->CopyResource(tex2, ...); // copy more textures
copyEngine->CopyResource(tex3, ...); // copy more textures
copyEngine->CopyResource(tex4, ...); // copy more textures
copyEngine->Signal(copyFence, 102);
renderEngine->Wait(copyFence, 101); // geometry copied
renderEngine->Draw(); // pre-pass using geometry only into rt1
renderEngine->Signal(renderFence, 201);
computeEngine->Wait(renderFence, 201); // prepass completed
computeEngine->Dispatch(); // lighting calculations on pre-pass (using rt1 as SRV)
computeEngine->Signal(computeFence, 301);
renderEngine->Wait(computeFence, 301); // lighting calculated into buf1
renderEngine->Wait(copyFence, 102); // textures copied
renderEngine->Draw(); // final render using buf1 as SRV, and tex[1-4] SRVs
```

以下伪代码演示通过环形缓冲区完成类似于堆的内存分配的复制引擎与 3D 引擎之间的同步。 资产能够灵活地在最大化并行度（通过大型缓冲区）与降低内存消耗和延迟（通过小型缓冲区）之间选择最佳的平衡。

``` syntax
device->CreateBuffer(&ringCB);
for(int i=1;i++){
  if(i > length) copyEngine->Wait(fence1, i - length);
  copyEngine->Map(ringCB, value%length, WRITE, pData); // copy new data
  copyEngine->Signal(fence2, i);
  renderEngine->Wait(fence2, i);
  renderEngine->Draw(); // draw using copied data
  renderEngine->Signal(fence1, i);
}

// example for length = 3:
// copyEngine->Map();
// copyEngine->Signal(fence2, 1); // fence2 = 1  
// copyEngine->Map();
// copyEngine->Signal(fence2, 2); // fence2 = 2
// copyEngine->Map();
// copyEngine->Signal(fence2, 3); // fence2 = 3
// copy engine has exhausted the ring buffer, so must wait for render to consume it
// copyEngine->Wait(fence1, 1); // fence1 == 0, wait
// renderEngine->Wait(fence2, 1); // fence2 == 3, pass
// renderEngine->Draw();
// renderEngine->Signal(fence1, 1); // fence1 = 1, copy engine now unblocked
// renderEngine->Wait(fence2, 2); // fence2 == 3, pass
// renderEngine->Draw();
// renderEngine->Signal(fence1, 2); // fence1 = 2
// renderEngine->Wait(fence2, 3); // fence2 == 3, pass
// renderEngine->Draw();
// renderEngine->Signal(fence1, 3); // fence1 = 3
// now render engine is starved, and so must wait for the copy engine
// renderEngine->Wait(fence2, 4); // fence2 == 3, wait
```

## <a name="multi-engine-scenarios"></a>多引擎方案

D3D12 可让开发人员避免意外的同步延迟导致效率意外下降的问题。 它还可让开发人员引入更高级别的同步，以更有把握地确定所需的同步。 多引擎解决的另一个问题是使高开销的操作变得更明确，包括 3D 与视频之间的转换（在传统上，这种转换需要在多个内核上下文之间同步，因此开销很高）。

具体而言，D3D12 可以实现以下方案：

-   异步和低优先级 GPU 工作。 可以实现低优先级 GPU 工作和原子操作的并发执行，这些操作支持在不阻塞的情况下，通过一个 GPU 线程来使用另一个未同步线程的结果。
-   高优先级计算工作。 使用后台计算可以中断 3D 渲染，以执行少量的高优先级计算工作。 可以提前获得此工作的结果，以便在 CPU 上进行其他处理。
-   后台计算工作。 计算工作负荷的独立低优先级队列可让应用程序利用空闲的 GPU 周期来执行后台计算，而不会对主要渲染（或其他）任务造成负面影响。 后台任务可能包括资源解压缩、更新模拟或加速结构。 应该以较低的频率在 CPU 上同步后台任务（大约每帧同步一次），以避免停滞或减慢前台工作。
-   流式处理和上传数据。 独立复制队列取代了 D3D11 中初始数据和更新资源的概念。 尽管应用程序负责 D3D12 模型中的更多细节，但此责任与算力相关。 应用程序可以控制专用于缓冲上传数据的系统内存量。 应用程序可以选择同步时间和方式（CPU 或 GPU，阻塞或非阻塞），并可以跟踪排队工作的进度和工作量。
-   提高并行度。 如果应用程序为前台工作提供独立的队列，则可以使用更深层的队列来完成后台工作负荷（例如视频解码）。

在 D3D12 中，命令队列概念是应用程序所提交工作的大致连续序列的 API 表示形式。 屏障和其他技术允许此工作在管道中或以无序方式执行，但应用程序只会看到单个完成时间线。 这对应于 D3D11 中的即时上下文。

## <a name="synchronization-apis"></a>同步 API

### <a name="devices-and-queues"></a>设备和队列

D3D 12 设备提供相应的方法用于创建和检索不同的类型与优先级的命令队列。 大多数应用程序应使用默认的命令队列，因为这些队列允许其他组件共享使用。 并发性要求更高的应用程序可以创建额外的队列。 队列按它们使用的命令列表类型指定。

请参阅以下 [**ID3D12Device**](/windows/desktop/api/d3d12/nn-d3d12-id3d12device) 创建方法：

-   [**CreateCommandQueue**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandqueue)：基于 [**D3D12\_COMMAND\_QUEUE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_queue_desc) 结构中的信息创建命令队列。
-   [**CreateCommandList**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandlist)：创建 [**D3D12\_COMMAND\_LIST\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_command_list_type) 类型的命令列表。
-   [**CreateFence**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createfence)：创建围栏。请注意 [**D3D12\_FENCE\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_fence_flags) 中的标志。 围栏用于同步队列。

所有类型（3D、计算和复制）的队列共享同一个接口，全部基于命令列表。

请参阅以下 [**ID3D12CommandQueue**](/windows/desktop/api/d3d12/nn-d3d12-id3d12commandqueue) 方法：

-   [**ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)：提交命令列表的数组供执行。 [**ID3D12CommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12commandlist) 定义的每个命令列表。
-   [**Signal**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-signal)：当（正在 GPU 上运行的）队列达到特定的点时设置围栏值。
-   [**Wait**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-wait)：队列等到指定的围栏达到指定的值。

请注意，捆绑不会由任何队列使用，因此无法使用此类型创建队列。

### <a name="fences"></a>围栏

多引擎 API 提供显式 API 用于通过围栏创建和同步队列。 围栏是通过单调递增某个 UINT64 值确定的同步构造。 围栏值由应用程序设置。 信号操作会增大围栏值，等待操作会阻塞到围栏达到请求的值为止。 当围栏达到特定的值时，可以激发事件。

请参阅 [**ID3D12Fence**](/windows/desktop/api/d3d12/nn-d3d12-id3d12fence) 接口的方法：

-   [**GetCompletedValue**](/windows/desktop/api/d3d12/nf-d3d12-id3d12fence-getcompletedvalue)：返回围栏的当前值。
-   [**SetEventOnCompletion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12fence-seteventoncompletion)：导致在围栏达到给定的值时激发事件。
-   [**Signal**](/windows/desktop/api/d3d12/nf-d3d12-id3d12fence-signal)：将围栏设置为给定的值。

围栏允许 CPU 访问当前围栏值，CPU 将会等待并发出信号。 独立的组件可以共享默认队列，但会创建自身的围栏并控制自身的围栏值和同步。

[**ID3D12Fence**](/windows/desktop/api/d3d12/nn-d3d12-id3d12fence) 接口中的 [**Signal**](/windows/desktop/api/d3d12/nf-d3d12-id3d12fence-signal) 方法从 CPU 端更新围栏。 [**ID3D12CommandQueue**](/windows/desktop/api/d3d12/nn-d3d12-id3d12commandqueue) 中的 [**Signal**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-signal) 方法从 GPU 端更新围栏。

多引擎设置中的所有节点可以读取正确的值，并在任何围栏达到该值时做出反应。

应用程序会设置自身的围栏值，良好的起点可能是为每个帧增大围栏一次。

围栏 API 提供强大的同步功能，但可能会产生难以调试的问题。

### <a name="copy-and-compute-command-lists"></a>复制和计算命令列表

命令列表的所有三个类型都使用 [**ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 接口，但是，只有一部分方法支持复制和计算。

复制和计算命令列表可使用以下方法：

-   [**Close**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)
-   [**CopyBufferRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion)
-   [**CopyResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copyresource)
-   [**CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)
-   [**CopyTiles**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytiles)
-   [**Reset**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-reset)
-   [**ResourceBarrier**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)

计算命令列表还可使用以下方法：

-   [**ClearState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearstate)
-   [**ClearUnorderedAccessViewFloat**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearunorderedaccessviewfloat)
-   [**ClearUnorderedAccessViewUint**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearunorderedaccessviewuint)
-   [**DiscardResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-discardresource)
-   [**Dispatch**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch)
-   [**ExecuteIndirect**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executeindirect)
-   [**SetComputeRoot32BitConstant**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputeroot32bitconstant)
-   [**SetComputeRoot32BitConstants**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputeroot32bitconstants)
-   [**SetComputeRootConstantBufferView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootconstantbufferview)
-   [**SetComputeRootDescriptorTable**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootdescriptortable)
-   [**SetComputeRootShaderResourceView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootshaderresourceview)
-   [**SetComputeRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootsignature)
-   [**SetComputeRootUnorderedAccessView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootunorderedaccessview)
-   [**SetDescriptorHeaps**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps)
-   [**SetPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate)
-   [**SetPredication**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpredication)

调用 [**SetPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate) 时，计算命令列表必须设置计算 PSO。

捆绑不能与计算或复制命令列表或队列配合使用。

## <a name="pipelined-compute-and-graphics-example"></a>管道化计算和图形示例

此示例演示如何使用围栏同步在队列上创建由图形工作`pComputeQueue` `pGraphicsQueue`使用的计算工作管道（由引用）。 计算和图形工作是使用图形队列的流水线操作，该队列从几个帧返回计算工作的结果，并使用 CPU 事件来限制整个排队的工作总量。

``` syntax
void PipelinedComputeGraphics()
{
    const UINT CpuLatency = 3;
    const UINT ComputeGraphicsLatency = 2;

    HANDLE handle = CreateEvent(nullptr, FALSE, FALSE, nullptr);

    UINT64 FrameNumber = 0;

    while (1)
    {
        if (FrameNumber > ComputeGraphicsLatency)
        {
            pComputeQueue->Wait(pGraphicsFence,
                FrameNumber - ComputeGraphicsLatency);
        }

        if (FrameNumber > CpuLatency)
        {
            pComputeFence->SetEventOnFenceCompletion(
                FrameNumber - CpuLatency,
                handle);
            WaitForSingleObject(handle, INFINITE);
        }

        ++FrameNumber;

        pComputeQueue->ExecuteCommandLists(1, &pComputeCommandList);
        pComputeQueue->Signal(pComputeFence, FrameNumber);
        if (FrameNumber > ComputeGraphicsLatency)
        {
            UINT GraphicsFrameNumber = FrameNumber - ComputeGraphicsLatency;
            pGraphicsQueue->Wait(pComputeFence, GraphicsFrameNumber);
            pGraphicsQueue->ExecuteCommandLists(1, &pGraphicsCommandList);
            pGraphicsQueue->Signal(pGraphicsFence, GraphicsFrameNumber);
        }
    }
}
```

若要支持此管道，必须有从计算队列`ComputeGraphicsLatency+1`向图形队列传递数据的不同副本的缓冲区。 命令列表必须使用 UAV 和间接性从该缓冲区中的相应数据“版本”读取和写入。 计算队列必须等到图形队列完成从数据中读取帧 N，然后才能写入帧 `N+ComputeGraphicsLatency`。

请注意，相对于 CPU 的计算队列量不会直接依赖于所需的缓冲量，然而，队列 GPU 工作超出了可用的缓冲区空间量也不太重要。

避免间接性的替代机制是创建对应于数据的每个“重命名”版本的多个命令列表。 以下示例使用此技术，同时扩展了前一个示例，允许以更高的异步性运行计算和图形队列。

## <a name="asynchronous-compute-and-graphics-example"></a>异步计算和图形示例

以下示例允许图形从计算队列以异步方式进行渲染。 两个阶段之间仍然存在固定的缓冲数据量，但现在图形工作会独立继续，并使用在将图形工作排队时，CPU 上已知的计算阶段的最新结果。 如果图形工作过去正在由另一个源（例如用户输入）更新，则此方法非常有用。 必须使用多个命令列表才能使图形工作的 `ComputeGraphicsLatency` 帧同时同步，`UpdateGraphicsCommandList` 函数表示更新命令列表以包含最新的输入数据，并从相应的缓冲区读取计算数据。

计算队列仍必须等待图形队列使用管道缓冲区完成，但引入了第三个围栏 (`pGraphicsComputeFence`)，以便可以跟踪读取计算工作的图形的进度，以及一般的图形进度。 这反映了这样一个事实：连续的图形帧现在可以读取相同的计算结果，或者可以跳过计算结果。 更有效但略微复杂一些的设计是仅使用单个图形围栏，并存储对每个图形帧使用的计算帧的映射。

```cpp
void AsyncPipelinedComputeGraphics()
{
    const UINT CpuLatency{ 3 };
    const UINT ComputeGraphicsLatency{ 2 };

    // The compute fence is at index 0; the graphics fence is at index 1.
    ID3D12Fence* rgpFences[]{ pComputeFence, pGraphicsFence };
    HANDLE handles[2];
    handles[0] = CreateEvent(nullptr, FALSE, TRUE, nullptr);
    handles[1] = CreateEvent(nullptr, FALSE, TRUE, nullptr);
    UINT FrameNumbers[]{ 0, 0 };

    ID3D12GraphicsCommandList* rgpGraphicsCommandLists[CpuLatency];
    CreateGraphicsCommandLists(ARRAYSIZE(rgpGraphicsCommandLists),
        rgpGraphicsCommandLists);

    // Graphics needs to wait for the first compute frame to complete; this is the
    // only wait that the graphics queue will perform.
    pGraphicsQueue->Wait(pComputeFence, 1);

    while (true)
    {
        for (auto i = 0; i < 2; ++i)
        {
            if (FrameNumbers[i] > CpuLatency)
            {
                rgpFences[i]->SetEventOnCompletion(
                    FrameNumbers[i] - CpuLatency,
                    handles[i]);
            }
            else
            {
                ::SetEvent(handles[i]);
            }
        }


        auto WaitResult = ::WaitForMultipleObjects(2, handles, FALSE, INFINITE);
        if (WaitResult > WAIT_OBJECT_0 + 1) continue;
        auto Stage = WaitResult - WAIT_OBJECT_0;
        ++FrameNumbers[Stage];

        switch (Stage)
        {
        case 0:
        {
            if (FrameNumbers[Stage] > ComputeGraphicsLatency)
            {
                pComputeQueue->Wait(pGraphicsComputeFence,
                    FrameNumbers[Stage] - ComputeGraphicsLatency);
            }
            pComputeQueue->ExecuteCommandLists(1, &pComputeCommandList);
            pComputeQueue->Signal(pComputeFence, FrameNumbers[Stage]);
            break;
        }
        case 1:
        {
            // Recall that the GPU queue started with a wait for pComputeFence, 1
            UINT64 CompletedComputeFrames = min(1,
                pComputeFence->GetCompletedValue());
            UINT64 PipeBufferIndex =
                (CompletedComputeFrames - 1) % ComputeGraphicsLatency;
            UINT64 CommandListIndex = (FrameNumbers[Stage] - 1) % CpuLatency;
            // Update graphics command list based on CPU input and using the appropriate
            // buffer index for data produced by compute.
            UpdateGraphicsCommandList(PipeBufferIndex,
                rgpGraphicsCommandLists[CommandListIndex]);

            // Signal *before* new rendering to indicate what compute work
            // the graphics queue is DONE with
            pGraphicsQueue->Signal(pGraphicsComputeFence, CompletedComputeFrames - 1);
            pGraphicsQueue->ExecuteCommandLists(1,
                rgpGraphicsCommandLists + PipeBufferIndex);
            pGraphicsQueue->Signal(pGraphicsFence, FrameNumbers[Stage]);
            break;
        }
        }
    }
}
```

## <a name="multi-queue-resource-access"></a>多队列资源访问

若要访问多个队列中的资源，应用程序必须遵守以下规则。

-   资源访问权限（请参阅 [**D3D12\_RESOURCE\_STATES**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_states)）按队列类型类而不是队列对象确定。 队列有两个类型类：计算/3D 队列是一个类型类，复制是另一个类型类。 因此，对一个 3D 队列中的 NON\_PIXEL\_SHADER\_RESOURCE 状态设置了屏障的资源可以在任何 3D 或计算队列中以该状态使用，但需要遵守序列化大多数写入操作的同步要求。 在两个类型类之间（COPY\_SOURCE 和 COPY\_DEST）共享的资源状态被视为每个类型类的不同状态。 因此，如果资源转换为复制队列中的 COPY\_DEST，则无法从 3D 或计算队列将其作为复制目标进行访问，反之亦然。

    总结：

    -   队列“对象”是任何单一队列。
    -   队列“类型”是以下三项中的任何一个：“计算”、“3D”和“复制”。
    -   队列“类型类”是以下两项中的任何一个：“计算/3D”和“复制”。

-   用作初始状态的 COPY 标志（COPY\_DEST 和 COPY\_SOURCE）表示 3D/计算类型类中的状态。 最初若要在复制队列中使用某个资源，该资源最初应处于 COMMON 状态。 对于使用隐式状态转换的所有复制队列用法，可以使用 COMMON 状态。 
-   尽管资源状态在所有计算和 3D 队列之间共享，但不允许在不同队列上同时写入资源。 此处的“同时”表示未同步。请注意在某些硬件上无法实现未同步的执行。 以下规则适用。

    -   每次只能有一个队列写入资源。
    -   多个队列可以读取资源，前提是它们不会读取写入方正在修改的字节（读取同时正在写入的字节会产生不确定的结果）。
    -   写入之前，必须使用一个围栏进行同步，然后，另一个队列才能读取写入的字节并进行任何写入访问。

-   提供的反向缓冲区必须处于 D3D12\_RESOURCE\_STATE\_COMMON 状态。 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[多引擎和多适配器同步](multi-engine-and-multi-gpu-synchronization.md)
</dt> <dt>

[在 Direct3D 12 中使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)
</dt> </dl>

 

 




