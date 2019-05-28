---
title: 设置和填充描述符堆
description: 可以设置命令列表的描述符堆类型是那些包含的描述符的描述符表可使用 （最多每一次）。
ms.assetid: F0FF3D7C-1DAC-48C3-B47D-0378BE369F37
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: f4166824b85e7588cba3ee9c781e98373acd674d
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223894"
---
# <a name="setting-and-populating-descriptor-heaps"></a>设置和填充描述符堆

可以设置命令列表的描述符堆类型是那些包含的描述符的描述符表可使用 （最多每一次）。

-   [设置描述符堆](#setting-and-populating-descriptor-heaps)
-   [填充描述符堆](#populating-descriptor-heaps)
-   [相关的主题](#related-topics)

## <a name="setting-descriptor-heaps"></a>设置描述符堆

描述符堆可以设置上命令列表的类型包括：

``` syntax
D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV
D3D12_DESCRIPTOR_HEAP_TYPE_SAMPLER
```

堆上命令列表设置必须还具有已创建为着色器可见。 有三种类型的命令列表：直接、 捆绑包，然后计算。

描述符堆集上命令列表后，请参阅当前的描述符堆定义描述符表的后续调用。 描述符表状态未定义的开头的命令列表和描述符堆命令列表更改之后。 冗余设置相同的描述符堆不会导致描述符表设置为未定义。

在绑定中，与此相反，堆只能设置一次的描述符 （两次设置同一堆的冗余调用不生成错误）;否则，该行为不确定。 描述符堆设置必须匹配状态，当任何命令列表调用捆绑;否则，该行为不确定。 这允许继承和编辑命令列表的描述符表设置的捆绑包。 不要更改描述符表的捆绑包 （仅继承这些设置） 无需设置描述符根本堆，只需将继承调用的命令列表。

描述符堆时设置 (使用[ **ID3D12GraphicsCommandList::SetDescriptorHeaps**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps))、 正在使用的所有堆中一次调用都设置 （和所有以前都集堆是通过调用取消设置）。 可以在调用中设置上述每种类型的最多一个堆。

## <a name="populating-descriptor-heaps"></a>填充描述符堆

已创建描述符堆的应用程序后，它可以然后使用方法在堆上或上命令列表直接插入堆或复制描述符从一个位置到另一个生成描述符。

描述符堆内存的初始内容未定义，因此要求要引用未初始化的内存以进行呈现的 GPU 或驱动程序可能会导致未定义的结果如设备重置。

如果应用程序配置描述符堆为 CPU 可见，则 CPU 可以调用方法来创建描述符堆到并从复制置于要放置 （包括跨堆） 直接的、 免费的线程方式。 如果已配置堆执行写入操作与将属性结合使用，不允许由 CPU 读取。

描述符还可以记录上命令列表的描述符副本调用，但在的应用程序不希望复制后，若要立即发生，而不是 GPU 执行命令列表时。 这也可以是有用或如果应用程序选择放在非 CPU-显示内存池中描述符堆，因此需要执行 GPU 操作来操作其内容，如果应用程序只是想要在 GPU 时间线上发生的更新。 命令列表描述符副本需要的副本才能在非着色器可见描述符堆 （从其命令列出快照复制到命令源描述符列出在记录时），源和目标必须是可见的着色器描述符堆 （它在命令列表执行时获取写入到由 GPU）。 复制描述符的 API 参考介绍更多详细信息。

例如，设置描述符堆时填充命令列表。


```C++
void D3D12Bundles::PopulateCommandList(FrameResource* pFrameResource)
{
    // Command list allocators can only be reset when the associated
    // command lists have finished execution on the GPU; apps should use
    // fences to determine GPU execution progress.
    ThrowIfFailed(m_pCurrentFrameResource->m_commandAllocator->Reset());

    // However, when ExecuteCommandList() is called on a particular command
    // list, that command list can then be reset at any time and must be before
    // re-recording.
    ThrowIfFailed(m_commandList->Reset(m_pCurrentFrameResource->m_commandAllocator.Get(), m_pipelineState1.Get()));

    // Set necessary state.
    m_commandList->SetGraphicsRootSignature(m_rootSignature.Get());

    ID3D12DescriptorHeap* ppHeaps[] = { m_cbvSrvHeap.Get(), m_samplerHeap.Get() };
    m_commandList->SetDescriptorHeaps(_countof(ppHeaps), ppHeaps);

    m_commandList->RSSetViewports(1, &m_viewport);
    m_commandList->RSSetScissorRects(1, &m_scissorRect);

    // Indicate that the back buffer will be used as a render target.
    m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_PRESENT, D3D12_RESOURCE_STATE_RENDER_TARGET));

    CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart(), m_frameIndex, m_rtvDescriptorSize);
    CD3DX12_CPU_DESCRIPTOR_HANDLE dsvHandle(m_dsvHeap->GetCPUDescriptorHandleForHeapStart());
    m_commandList->OMSetRenderTargets(1, &rtvHandle, FALSE, &dsvHandle);

    // Record commands.
    const float clearColor[] = { 0.0f, 0.2f, 0.4f, 1.0f };
    m_commandList->ClearRenderTargetView(rtvHandle, clearColor, 0, nullptr);
    m_commandList->ClearDepthStencilView(m_dsvHeap->GetCPUDescriptorHandleForHeapStart(), D3D12_CLEAR_FLAG_DEPTH, 1.0f, 0, 0, nullptr);

    if (UseBundles)
    {
        // Execute the prebuilt bundle.
        m_commandList->ExecuteBundle(pFrameResource->m_bundle.Get());
    }
    else
    {
        // Populate a new command list.
        pFrameResource->PopulateCommandList(m_commandList.Get(), m_pipelineState1.Get(), m_pipelineState2.Get(), m_currentFrameResourceIndex, m_numIndices, &m_indexBufferView,
            &m_vertexBufferView, m_cbvSrvHeap.Get(), m_cbvSrvDescriptorSize, m_samplerHeap.Get(), m_rootSignature.Get());
    }

    // Indicate that the back buffer will now be used to present.
    m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_RENDER_TARGET, D3D12_RESOURCE_STATE_PRESENT));

    ThrowIfFailed(m_commandList->Close());
}
```



## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符堆](descriptor-heaps.md)
</dt> </dl>

 

 




