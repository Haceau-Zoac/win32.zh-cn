---
title: 同步和多引擎
description: 大多数现代 Gpu 包含多个独立引擎提供特定的功能的。
ms.assetid: 93903F50-A6CA-41C2-863D-68D645586B4C
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 6e1ca85f1bb590a8579ee7da88a7aea94a80e731
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223744"
---
# <a name="synchronization-and-multi-engine"></a>同步和多引擎

大多数现代 Gpu 包含多个独立引擎提供特定的功能的。 有许多的一个或多个专用的复制引擎和计算引擎，通常不同于 3D 引擎。 每个这些引擎可以彼此并行执行命令。 Direct3D 12 提供精细访问权限的三维效果、 计算和复制引擎，使用队列和命令列表。

-   [GPU 引擎](#gpu-engines)
-   [多引擎方案](#multi-engine-scenarios)
-   [同步 Api](#synchronization-apis)
    -   [设备和队列](#devices-and-queues)
    -   [复制和计算命令列表](#copy-and-compute-command-lists)
-   [通过管道传递的计算和图形示例](#pipelined-compute-and-graphics-example)
-   [异步计算和图形示例](#asynchronous-compute-and-graphics-example)
-   [多队列资源访问权限](#multi-queue-resource-access)
-   [相关的主题](#related-topics)

## <a name="gpu-engines"></a>GPU 引擎

下图显示了标题的 CPU 线程，每个填充一个或多个副本、 计算和 3D 队列。 3D 队列可以驱动所有三个 GPU 引擎，计算队列可以提高计算和复制引擎和复制队列只需复制引擎。

不同的线程填充队列，如时，可能会顺序执行，因此不简单保证的同步机制-需要标题需要它们。

![将命令发送到三个队列的四个线程](images/gpu-engines.png)

下图演示了如何标题可能跨多个 GPU 引擎，在必要时包括间引擎同步安排工作： 它显示了具有间引擎依赖项的每个引擎工作负荷。 在此示例中，复制引擎首先将复制所需呈现一些几何图形。 3D 引擎等待这些副本来完成，并通过几何图形呈现预传递。 这之后可供计算引擎。 计算引擎的结果[**调度**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch)，以及多个纹理的复制操作复制引擎，由最终的三维引擎[**绘制**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced)调用。

![复制、 图形和计算引擎进行通信](images/gpu-sync.png)

下面的伪代码说明了如何标题可能会提交此类工作负荷。

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

下面的伪代码说明了要完成环形缓冲区通过类似于堆的内存分配的副本和 3D 引擎之间的同步。 标题可以灵活地选择最大化的并行度 （通过较大的缓冲区），并减少内存占用量和延迟 （通过一个较小的缓冲区） 之间的最佳平衡。

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

D3D12 允许开发人员避免意外地遇到意外的同步延迟所导致的效率低下。 它还允许开发人员能够引入其中可以更明确地确定所需的同步较高级别上的同步。 第二个问题的多引擎地址是使成本高昂的操作更显式的其中包括由于多个内核上下文之间的同步了传统上成本高昂的三维和视频之间转换。

具体而言，可以使用 D3D12 解决以下方案：

-   异步和低优先级 GPU 的工作。 这使低优先级 GPU 的工作并启用一个 GPU 线程，而不会阻止使用另一个未同步线程的结果的原子操作的并发执行。
-   高优先级的计算工作。 后台计算就可以中断 3D 呈现为少量的高优先级计算工作。 可以进行其他处理在 CPU 上提前获得这项工作的结果。
-   后台计算工作。 计算工作负荷的单独的低优先级队列，应用程序以利用空闲 GPU 周期来执行后台计算，而无需在主呈现 （或其他） 的负面影响的任务。 后台任务可能包括资源或更新模拟或加速结构的解压缩。 后台任务应同步在 CPU 上很少 (大约每一帧的一次) 以避免拖延症或加快前台工作。
-   流式处理和上载数据。 单独的复制队列将替换初始数据和更新资源的 D3D11 概念。 尽管应用程序负责 D3D12 模型中的更多详细信息，但此职责附带了电源。 应用程序可以控制缓冲的数据上传专门介绍系统内存量。 应用程序可以选择何时以及如何 （CPU 和 GPU，阻止与非阻塞） 同步，并可以跟踪进度和控制的已排队的工作。
-   提高的并行度。 应用程序可将更深入的队列用于后台工作负荷 （例如视频解码） 时，它们具有单独的前景色工作队列。

在 D3D12 命令队列的概念是大致串行序列中的提交的应用程序工作的 API 表示形式。 障碍和其他技术允许在管道或无序，要执行此工作，但应用程序只能看到单独的完成时间线。 这对应于在 D3D11 的即时上下文。

## <a name="synchronization-apis"></a>同步 Api

### <a name="devices-and-queues"></a>设备和队列

D3D 12 设备都有用于创建和检索不同的类型和优先级的命令队列的方法。 大多数应用程序应使用默认命令队列，因为这些行为允许共享使用情况的其他组件。 提高并发性要求的应用程序可以创建额外的队列。 队列指定的命令列表类型，它们消耗。

请参阅以下的创建方法[ **ID3D12Device**](/windows/desktop/api/D3D12/nn-d3d12-id3d12device):

-   [**CreateCommandQueue** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandqueue) ： 创建基于中信息的命令队列[ **D3D12\_命令\_队列\_DESC** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_queue_desc)结构。
-   [**CreateCommandList** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandlist) ： 创建类型的命令列表[ **D3D12\_命令\_列表\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)。
-   [**CreateFence** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createfence) ： 创建的防护，注意是中的标志[ **D3D12\_FENCE\_标志**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_fence_flags)。 界定用于同步队列。

所有类型 （三维、 计算和复制） 共享相同接口，并将所有命令列表基于的队列。

下列方法之一是指[ **ID3D12CommandQueue**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue):

-   [**ExecuteCommandLists** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists) ： 提交执行的命令列表的数组。 正在定义的每个命令列表[ **ID3D12CommandList**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandlist)。
-   [**信号**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandqueue-signal) : （在 GPU 上运行） 的队列达到特定的时间点时设置围栏值。
-   [**等待**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandqueue-wait) ： 队列等待，直到指定的 fence 达到指定的值。

请注意捆绑包未被任何队列，因此此类型不能用于创建队列。

### <a name="fences"></a>界定

多引擎 API 提供了显式 Api 创建和同步使用界定。 Fence 为单调递增的 UINT64 值由一个同步构造。 应用程序会设置围栏值。 信号操作会增加围栏值并等待操作阻止直到 fence 已达到请求的值。 一种保护达到特定值时，会激发事件。

引用的方法[ **ID3D12Fence** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12fence)接口：

-   [**GetCompletedValue** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12fence-getcompletedvalue) ： 返回在界定的当前值。
-   [**SetEventOnCompletion** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12fence-seteventoncompletion) ： 导致 fence 达到的给定的值时引发该事件。
-   [**信号**](/windows/desktop/api/D3D12/nf-d3d12-id3d12fence-signal) : fence 设置为给定的值。

界定允许 CPU 访问当前的围栏值和 CPU 等待和信号。 独立组件可以共享默认队列，但创建其自己护栏和控制其自己的围栏值和同步。

[**信号**](/windows/desktop/api/D3D12/nf-d3d12-id3d12fence-signal)方法[ **ID3D12Fence** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12fence)接口更新从 CPU 端隔离。 [**信号**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandqueue-signal)方法[ **ID3D12CommandQueue** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue)更新从 GPU 端隔离。

多引擎安装程序中的所有节点可以读取和响应任何 fence 到达正确的值。

应用程序将设置其自己的围栏值，很好的起点可能增加一次每一帧 fence。

Fence Api 提供功能强大的同步功能，但可以创建可能导致难以调试的问题。

### <a name="copy-and-compute-command-lists"></a>复制和计算命令列表

所有三种类型的命令列表使用[ **ID3D12GraphicsCommandList** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)接口，但是这些方法的一个子集复制支持，以及计算。

复制并计算命令列出了可以使用以下方法：

-   [**关闭**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)
-   [**CopyBufferRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion)
-   [**CopyResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copyresource)
-   [**CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)
-   [**CopyTiles**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytiles)
-   [**Reset**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-reset)
-   [**ResourceBarrier**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)

计算命令列表还可以使用以下方法：

-   [**ClearState**](/windows/desktop/api/D3D12/nf-d3d12-id3d12graphicscommandlist-clearstate)
-   [**ClearUnorderedAccessViewFloat**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearunorderedaccessviewfloat)
-   [**ClearUnorderedAccessViewUint**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearunorderedaccessviewuint)
-   [**DiscardResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-discardresource)
-   [**调度**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch)
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

计算调用时，命令列表必须设置计算 PSO [ **SetPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate)。

捆绑包不能用于计算或复制命令列表或队列。

## <a name="pipelined-compute-and-graphics-example"></a>通过管道传递的计算和图形示例

此示例演示如何使用 fence 同步在队列上创建的计算工作的管道 (所引用的`pComputeQueue`) 由图形工作队列`pGraphicsQueue`。 计算和图形工作通过管道传递与图形队列使用从多个帧，计算工作的结果，以限制对所有排队的总工作用于 CPU 事件。

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

若要支持此管道操作必须有的缓冲区`ComputeGraphicsLatency+1`传递的数据的不同副本形成计算队列到图形队列。 此命令列出必须使用 Uav 和间接寻址读取和写入从缓冲区中的数据的相应"版本"。 计算队列必须等待，直到图形队列已完成从框架 N 的数据进行读取，它可以写入帧之前`N+ComputeGraphicsLatency`。

请注意，计算量排队相对于 CPU 工作不依赖于所需的缓冲，但是超出可用缓冲空间量的 GPU 工作排队，则属性不太重要量直接。

使用备用机制来避免间接寻址就是创建多个对应于每个"重命名为"版本的数据的命令列表。 下一个示例继续前面的示例时使用此技术允许计算和图形队列，更以异步方式运行。

## <a name="asynchronous-compute-and-graphics-example"></a>异步计算和图形示例

下一个示例允许图形从计算队列以异步方式呈现。 仍然是固定的两个阶段之间缓冲的数据量，但是现在结合使用图形独立地继续，并为已知在 CPU 上排队的图形工作时使用个计算阶段的最新结果。 这会非常有用，如果另一个源，例如用户输入更新的图形工作了。 必须有多个命令列表，以允许`ComputeGraphicsLatency`帧的图形工作时间和该函数在处于飞行`UpdateGraphicsCommandList`表示更新包括最新的输入的数据并读取中的计算数据中的命令列表适当的缓冲区。

计算队列必须仍等待图形队列从头到尾管道缓冲区，但第三个围墙 (`pGraphicsComputeFence`) 引入的以便可以跟踪图形一般情况下读取与图形进度的计算工作的进度。 这反映了这一事实，现在连续图形帧未能读取，相同计算结果，或者可以跳过的计算结果。 更有效，但稍微复杂一些设计将使用只是单个图形界定，并将存储到每个图形帧使用的计算帧的映射。

``` syntax
void AsyncPipelinedComputeGraphics()
{
    const UINT CpuLatency = 3;
    const UINT ComputeGraphicsLatency = 2;

    // Compute is 0, graphics is 1
    ID3D12Fence *rgpFences[] = { pComputeFence, pGraphicsFence };
    HANDLE handles[2];
    handles[0] = CreateEvent(nullptr, FALSE, TRUE, nullptr);
    handles[1] = CreateEvent(nullptr, FALSE, TRUE, nullptr);
    UINT FrameNumbers[] = { 0, 0 };

    ID3D12GraphicsCommandList *rgpGraphicsCommandLists[CpuLatency];
    CreateGraphicsCommandLists(ARRAYSIZE(rgpGraphicsCommandLists),
        rgpGraphicsCommandLists);

    // Graphics needs to wait for the first compute frame to complete, this is the
    // only wait that the graphics queue will perform.
    pGraphicsQueue->Wait(pComputeFence, 1);

    while (1)
    {
        for (auto i = 0; i < 2; ++i)
        {
            if (FrameNumbers[i] > CpuLatency)
            {
                rgpFences[i]->SetEventOnFenceCompletion(
                    FrameNumbers[i] - CpuLatency,
                    handles[i]);
            }
            else
            {
                SetEvent(handles[i]);
            }
        }

        auto WaitResult = WaitForMultipleObjects(2, handles, FALSE, INFINITE);
        auto Stage = WaitResult = WAIT_OBJECT_0;
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
                pComputeFence->GetCurrentFenceValue());
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

## <a name="multi-queue-resource-access"></a>多队列资源访问权限

若要访问多个队列上的资源应用程序必须遵守以下规则。

-   资源访问权限 (请参阅[ **D3D12\_资源\_状态**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)) 由队列类型类不队列对象。 有两种类型类的队列：计算/三维队列是一个类型的类，复制是第二个类型的类。 因此某个资源具有非的障碍\_像素\_着色器\_，排队上任何 3D 或计算的状态，请遵循需要大多数写入同步要求，可以使用 3D 队列上的资源状态序列化。 两个类之间共享的资源状态 (副本\_源和副本\_DEST) 被视为不同类型的每个类的状态。 因此，如果资源转换为复制\_DEST 复制队列不从 3D 或计算队列复制目标的可访问性，反之亦然。

    总结：

    -   "对象"的队列是任何单个队列。
    -   "Type"的队列是这三个字段的任何一个：计算、 3D、 和副本。
    -   队列"类型 class"是这两个任何一个：计算/三维和复制。

-   复制标志 (副本\_DEST 和复制\_源) 用作初始状态表示 3D/计算类型类中的状态。 复制队列中的常见状态开始时间上最初使用资源。 常见的状态可以用于复制队列使用隐式状态转换的所有用法。 
-   尽管在所有计算和 3D 队列共享资源状态，但它不允许在不同的队列上同时写入资源。 "同时"此处意味着未同步，注意未同步的执行无法实现的某些硬件。 以下规则适用。

    -   只有一个队列可以一次写入到的资源。
    -   只要它们不读取 （阅读正在同时写入的字节数产生未定义的结果） 的编写器正在修改的字节数，可以从资源读取多个队列。
    -   一种保护必须用于写入另一个队列可以读取写入的字节数或使任何写访问权限之前后同步。

-   后台的缓冲区显示必须在 D3D12\_资源\_状态\_常见的状态。 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[多引擎和多适配器同步](multi-engine-and-multi-gpu-synchronization.md)
</dt> <dt>

[使用资源屏障来同步资源状态在 Direct3D 12](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)
</dt> </dl>

 

 




