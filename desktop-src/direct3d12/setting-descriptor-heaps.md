---
title: 设置和填充描述符堆
description: 可在命令列表上设置的描述符堆类型包括可使用描述符表（每次最多使用一个表）的描述符。
ms.assetid: F0FF3D7C-1DAC-48C3-B47D-0378BE369F37
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 5556b4a8f7018150f9e67d8170cc4682d5e89b98
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296305"
---
# <a name="setting-and-populating-descriptor-heaps"></a>设置和填充描述符堆

可在命令列表上设置的描述符堆类型包括可使用描述符表（每次最多使用一个表）的描述符。

-   [设置描述符堆](#setting-and-populating-descriptor-heaps)
-   [填充描述符堆](#populating-descriptor-heaps)
-   [相关主题](#related-topics)

## <a name="setting-descriptor-heaps"></a>设置描述符堆

可在命令列表上设置的描述符堆类型包括：

``` syntax
D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV
D3D12_DESCRIPTOR_HEAP_TYPE_SAMPLER
```

命令列表上设置的堆还必须已创建为着色器可见。 以下是三种类型的命令列表：DIRECT、BUNDLE 和 COMPUTE。

在命令列表上设置描述符堆之后，定义描述符表的后续调用将引用当前的描述符堆。 在命令列表的开头以及描述符堆在命令列表上更改之后，描述符表状态为未定义。 重复设置同一描述符堆不会导致描述符表设置变为未定义。

相比之下，在 bundle 中，描述符堆只能设置一次（重复调用设置同一堆两次不会产生错误）；否则，行为是未定义的。 设置的描述符堆必须与任何命令列表调用 bundle 时的状态相匹配；否则，行为是未定义的。 因此，bundle 可继承和编辑命令列表的描述符表设置。 不更改（只继承）描述符表的 bundle 根本无需设置描述符堆，可直接继承自调用命令列表。

（使用 [**ID3D12GraphicsCommandList::SetDescriptorHeaps**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps)）设置描述符堆时，所用的所有堆都会通过一个调用进行设置（并且会通过该调用取消之前所设置的所有堆的设置）。 通过调用最多可设置上述每种类型的一个堆。

## <a name="populating-descriptor-heaps"></a>填充描述符堆

应用程序创建描述符堆之后，它可以使用堆上或命令列表上的方法直接将描述符生成到堆中，或者将描述符从某一位置复制到另一位置。

描述符堆内存的初始内容是未定义的，因此请求 GPU 或驱动程序引用未初始化的内存进行渲染可能产生设备重置等未定义的结果。

如果应用程序将描述符堆配置为 CPU 可见，则 CPU 可以调用方法将描述符创建到堆中，并以即时、自由线程方式从某一位置复制到另一位置（包括跨堆）。 如果堆已配置写入合并属性，则 CPU 无法读取。

如果应用程序无需立即（而是在 GPU 执行命令列表时）进行复制，则描述符还可在命令列表上记录描述符复制调用。 如果应用程序选择将描述符堆放入非 CPU 可见的内存池中，并因此需执行 GPU 操作来处理其内容，或者如果应用程序只需按 GPU 的时间线进行更新，则这也非常有效。 命令列表描述符复制需要复制源位于非着色器可见描述符堆（命令列表快照在记录时从中将源描述符复制到命令列表）中，并且目标必须是着色器可见描述符堆（在命令列表中由 GPU 将执行写入其中）。 复制描述符的 API 参考更为详细。

例如，填充命令列表时设置描述符堆。


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

 

 




