---
title: D3D12 引用中的示例代码
description: 介绍如何使用 Direct3D 12 文档中的示例代码。
ms.assetid: C2323482-D06D-43B7-9BDE-BFB9A6A6B70D
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: f291be3ace2b8d26ac3de7c20dc16d7d80ae5be8
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "68679694"
---
# <a name="example-code-in-the-d3d12-reference"></a>D3D12 引用中的示例代码

介绍如何使用 Direct3D 12 文档中的示例代码。

-   [示例代码段代码](#example-snippet-code)
-   [相关主题](#related-topics)

## <a name="example-snippet-code"></a>示例代码段代码

Direct3D 12 引用中所示的示例代码是不可以编译或运行时代码的, 它只是一个代码片段, 提供有关如何调用 API 的示例。 一些示例列出了调用使用的全局变量和类成员, 例如:

全局管道对象。


```C++
D3D12_VIEWPORT m_viewport;
D3D12_RECT m_scissorRect;
ComPtr<IDXGISwapChain3> m_swapChain;
ComPtr<ID3D12Device> m_device;
ComPtr<ID3D12Resource> m_renderTargets[FrameCount];
ComPtr<ID3D12CommandAllocator> m_commandAllocator;
ComPtr<ID3D12CommandQueue> m_commandQueue;
ComPtr<ID3D12RootSignature> m_rootSignature;
ComPtr<ID3D12DescriptorHeap> m_rtvHeap;
ComPtr<ID3D12PipelineState> m_pipelineState;
ComPtr<ID3D12GraphicsCommandList> m_commandList;
UINT m_rtvDescriptorSize;
```



正在填充命令列表。


```C++
// Command list allocators can only be reset when the associated 
// command lists have finished execution on the GPU; apps should use 
// fences to determine GPU execution progress.
ThrowIfFailed(m_commandAllocator->Reset());

// However, when ExecuteCommandList() is called on a particular command 
// list, that command list can then be reset at any time and must be before 
// re-recording.
ThrowIfFailed(m_commandList->Reset(m_commandAllocator.Get(), m_pipelineState.Get()));

// Set necessary state.
m_commandList->SetGraphicsRootSignature(m_rootSignature.Get());
m_commandList->RSSetViewports(1, &m_viewport);
m_commandList->RSSetScissorRects(1, &m_scissorRect);

// Indicate that the back buffer will be used as a render target.
m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_PRESENT, D3D12_RESOURCE_STATE_RENDER_TARGET));

CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart(), m_frameIndex, m_rtvDescriptorSize);
m_commandList->OMSetRenderTargets(1, &rtvHandle, FALSE, nullptr);

// Record commands.
const float clearColor[] = { 0.0f, 0.2f, 0.4f, 1.0f };
m_commandList->ClearRenderTargetView(rtvHandle, clearColor, 0, nullptr);
m_commandList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
m_commandList->IASetVertexBuffers(0, 1, &m_vertexBufferView);
m_commandList->DrawInstanced(3, 1, 0, 0);

// Indicate that the back buffer will now be used to present.
m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_RENDER_TARGET, D3D12_RESOURCE_STATE_PRESENT));

ThrowIfFailed(m_commandList->Close());
```



## <a name="related-topics"></a>相关主题

<dl> <dt>

[创建基本的 Direct3D 12 组件](creating-a-basic-direct3d-12-component.md)
</dt> <dt>

[Direct3D 12 参考](direct3d-12-reference.md)
</dt> <dt>

[D3D12 代码演练](d3d12-code-walk-throughs.md)
</dt> <dt>

[工作示例](working-samples.md)
</dt> </dl>

 

 




