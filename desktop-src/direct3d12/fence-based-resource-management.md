---
title: Fence 基于资源管理
description: 演示如何管理资源数据生存期内的 GPU 进度跟踪通过界定。 与界定仔细管理可用性的可用空间在内存中，如上载堆的环形缓冲区实现中，可以有效地重复使用内存。
ms.assetid: A7AB6569-EC6B-4B1B-9266-D05B6DB3A27B
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: faec81958b2283bdd17a335fae98525acd9309f0
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224458"
---
# <a name="fence-based-resource-management"></a>Fence 基于资源管理

演示如何管理资源数据生存期内的 GPU 进度跟踪通过界定。 与界定仔细管理可用性的可用空间在内存中，如上载堆的环形缓冲区实现中，可以有效地重复使用内存。

-   [环形缓冲区方案](#ring-buffer-scenario)
-   [环形缓冲区示例](#ring-buffer-sample)
-   [相关的主题](#related-topics)

## <a name="ring-buffer-scenario"></a>环形缓冲区方案

下面是在其中应用内体验对上传的堆内存的极少数要求的示例。

环形缓冲区是一种方法来管理上载堆。 环形缓冲区包含所需的数据在接下来的几个帧。 该应用程序维护当前数据输入的指针，和帧偏移量的队列，用于记录每个帧和该框架的资源数据的起始偏移量。

应用程序创建基于一个缓冲区以将数据上传到每个帧 GPU 环形缓冲区。 当前呈现帧 2，环形缓冲区绕框架 4 中，所有数据所需的帧，5，且较大的常量缓冲区所需的框架 6 需要二次分配的数据。

**图 1** ： 此应用尝试二次分配的常量缓冲区，但查找不足，无法释放内存。

![在此环形缓冲区中的可用内存不足](images/ring-buffer-1.png)

**图 2** ： 通过 fence 轮询应用程序发现已呈现该框架 3，帧偏移量的队列然后更新，并遵循环形缓冲区的当前状态-，可用内存但仍不足够大以容纳该常量缓冲区。

![已呈现帧 3 后仍没有足够的内存](images/ring-buffer-2.png)

**图 3** ： 给定情况下，CPU 块本身 （通过隔离正在等待） 之前呈现帧 4，并释放为框架 4 二次分配的内存。

![呈现框架 4 释放更多的环形缓冲区](images/ring-buffer-3.png)

**图 4** ： 现在可用内存过大的常量缓冲区和二次分配成功，应用将大的常量缓冲区数据复制到内存以前所用的资源数据的这两个框架 3 和 4。 最后更新当前输入的指针。

![现在没有聊天室从环形缓冲区中的第 6 帧](images/ring-buffer-4.png)

如果应用程序实现环形缓冲区，环形缓冲区必须足够大，以应对资源数据的大小的最坏情况。

## <a name="ring-buffer-sample"></a>环形缓冲区示例

下面的示例代码演示如何管理环形缓冲区，并关注的二次分配例程的处理 fence 轮询和等待。 为简单起见，此示例不使用\_SUFFICIENT\_内存，因为该逻辑隐藏的"在堆中找到不足够的可用内存"的详细信息 (基于*m\_pDataCur*和内部偏移量*FrameOffsetQueue*) 不与堆紧密相关或界定。 该示例简化以牺牲而不是内存使用率的帧速率。

请注意，应使用环形缓冲区支持是一种常用的方案;但是，堆设计不会阻止其他使用情况，例如命令列表参数化和重新使用。

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

[**ID3D12Fence**](/windows/desktop/api/D3D12/nn-d3d12-id3d12fence)
</dt> <dt>

[在缓冲区中的子分配](large-buffers.md)
</dt> </dl>

 

 




