---
title: 基于围栏的资源管理
description: 演示如何利用通过围栏跟踪 GPU 进度来管理资源数据的生存期。 可有效地重复使用内存，用围栏来仔细管理内存中可用空间的可用性，例如针对“上传”堆环形缓冲区实现。
ms.assetid: A7AB6569-EC6B-4B1B-9266-D05B6DB3A27B
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: d870785d8e16225d17aced4c7442869cd855e696
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296388"
---
# <a name="fence-based-resource-management"></a>基于围栏的资源管理

演示如何利用通过围栏跟踪 GPU 进度来管理资源数据的生存期。 可有效地重复使用内存，用围栏来仔细管理内存中可用空间的可用性，例如针对“上传”堆环形缓冲区实现。

-   [环形缓冲区方案](#ring-buffer-scenario)
-   [环形缓冲区示例](#ring-buffer-sample)
-   [相关主题](#related-topics)

## <a name="ring-buffer-scenario"></a>环形缓冲区方案

下面的示例展示了一个应用罕见地需要使用“上传”堆内存。

管理上传堆的一种方法是环形缓冲区。 环形缓冲区会保存下几帧所需的数据。 应用会维护当前数据输入指针和帧偏移量队列，用于记录每一帧和该帧资源数据的起始偏移量。

应用会基于缓冲区创建环形缓冲区，针对每个帧将数据上传到 GPU。 目前第 2 帧已呈现，环形缓冲区将第 4 帧的数据包含在内，第 5 帧所需的全部数据均已显示，第 6 帧所需的大型常量缓冲区需要进行二次分配。

**图 1**：应用尝试为常量缓冲区进行二次分配，但发现可用内存不足。

![此环形缓冲区中的可用内存不足](images/ring-buffer-1.png)

**图 2**：通过围栏轮询，应用发现第 3 帧已呈现，帧偏移量队列随后已更新，环形缓冲区的当前状态也随之更新；然而，空闲内存仍不足以无法容纳常量缓冲区。

![第 3 帧呈现后，内存仍然不够](images/ring-buffer-2.png)

**图 3**：鉴于情况，CPU 阻止自身（通过栅栏等待），直到第 4 帧呈现，这释放了次级分配给第 4 帧的内存。

![呈现第 4 帧释放出环形缓冲区的更多空间](images/ring-buffer-3.png)

**图 4**：现在，空闲内存已足以容纳常量缓冲区，且二次分配成功；应用将大型常量缓冲区数据复制到之前由第 3 帧和第 4 帧的资源数据所占用的内存中。 最后更新当前输入指针。

![现在环形缓冲区中有第 6 帧的空间](images/ring-buffer-4.png)

如果应用实现了环形缓冲区，则该缓冲区必须足够大，以应对资源数据大小最糟糕的情况。

## <a name="ring-buffer-sample"></a>环形缓冲区示例

以下示例代码显示了如何管理环形缓冲区，请注意处理围栏轮询和等待的二次分配例程。 为简单起见，该示例使用 NOT\_SUFFICIENT\_MEMORY 隐藏“堆中没有足够的可用内存”的详细信息，因为该逻辑（基于 m\_pDataCur 以及 FrameOffsetQueue 中的偏移量）与堆或围栏联系不紧密   。 简化了示例，牺牲的是帧速率而不是内存利用率。

请注意，环缓冲区支持有望成为常用方案；但是，堆设计并不排除其他用途，比如命令列表参数化和重用。

``` syntax
struct FrameResourceOffset
{
    UINT frameIndex;
    UINT8* pResourceOffset;
};
std::queue<FrameResourceOffset> frameOffsetQueue;

void DrawFrame()
{
    float vertices[] = ...;
    UINT verticesOffset = 0;
    ThrowIfFailed(
        SetDataToUploadHeap(
            vertices, sizeof(float), sizeof(vertices) / sizeof(float), 
            4, // Max alignment requirement for vertex data is 4 bytes.
            verticesOffset
            ));

    float constants[] = ...;
    UINT constantsOffset = 0;
    ThrowIfFailed(
        SetDataToUploadHeap(
            constants, sizeof(float), sizeof(constants) / sizeof(float), 
            D3D12_CONSTANT_BUFFER_DATA_PLACEMENT_ALIGNMENT,
            constantsOffset
            ));

    // Create vertex buffer views for the new binding model. 
    // Create constant buffer views for the new binding model. 
    // ...

    commandQueue->Execute(commandList);
    commandQueue->AdvanceFence();
}

HRESULT SuballocateFromHeap(SIZE_T uSize, UINT uAlign)
{
    if (NOT_SUFFICIENT_MEMORY(uSize, uAlign))
    {
        // Free up resources for frames processed by GPU; see Figure 2.
        UINT lastCompletedFrame = commandQueue->GetLastCompletedFence();
        FreeUpMemoryUntilFrame( lastCompletedFrame );

        while ( NOT_SUFFICIENT_MEMORY(uSize, uAlign)
            && !frameOffsetQueue.empty() )
        {
            // Block until a new frame is processed by GPU, then free up more memory; see Figure 3.
            UINT nextGPUFrame = frameOffsetQueue.front().frameIndex;
            commandQueue->SetEventOnFenceCompletion(nextGPUFrame, hEvent);
            WaitForSingleObject(hEvent, INFINITE);
            FreeUpMemoryUntilFrame( nextGPUFrame );
        }
    }

    if (NOT_SUFFICIENT_MEMORY(uSize, uAlign))
    {
        // Apps need to create a new Heap that is large enough for this resource.
        return E_HEAPNOTLARGEENOUGH;
    }
    else
    {
        // Update current data pointer for the new resource.
        m_pDataCur = reinterpret_cast<UINT8*>(
            Align(reinterpret_cast<SIZE_T>(m_pHDataCur), uAlign)
            );

        // Update frame offset queue if this is the first resource for a new frame; see Figure 4.
        UINT currentFrame = commandQueue->GetCurrentFence();
        if ( frameOffsetQueue.empty()
            || frameOffsetQueue.back().frameIndex < currentFrame )
        {
            FrameResourceOffset offset = {currentFrame, m_pDataCur};
            frameOffsetQueue.push(offset);
        }

        return S_OK;
    }
}

void FreeUpMemoryUntilFrame(UINT lastCompletedFrame)
{
    while ( !frameOffsetQueue.empty() 
        && frameOffsetQueue.first().frameIndex <= lastCompletedFrame )
    {
        frameOffsetQueue.pop();
    }
}
```

## <a name="related-topics"></a>相关主题

<dl> <dt>

[**ID3D12Fence**](/windows/desktop/api/d3d12/nn-d3d12-id3d12fence)
</dt> <dt>

[缓冲区中的二次分配](large-buffers.md)
</dt> </dl>

 

 




