---
title: 创建和记录命令列表与捆绑
description: 本主题介绍如何在 Direct3D 12 应用中记录命令列表和捆绑。 命令列表和捆绑都允许应用记录绘图或状态更改调用，以便稍后在图形处理单元 (GPU) 上执行。
ms.assetid: 0074B796-33A4-4AA1-A4E7-48A2A63F25B7
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: e31f0bada644639a7ad3d9fda328d9c8ca75f162
ms.sourcegitcommit: 27a9dfa3ef68240fbf09f1c64dff7b2232874ef4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66725569"
---
# <a name="creating-and-recording-command-lists-and-bundles"></a>创建和记录命令列表与捆绑

本主题介绍如何在 Direct3D 12 应用中记录命令列表和捆绑。 命令列表和捆绑都允许应用记录绘图或状态更改调用，以便稍后在图形处理单元 (GPU) 上执行。

-   [创建命令列表](#creating-command-lists)
    -   [D3D12\_COMMAND\_LIST\_TYPE](/windows)
    -   [ID3D12CommandAllocator](#id3d12commandallocator)
    -   [ID3D12PipelineState](#id3d12pipelinestate)
    -   [ID3D12DescriptorHeap](#id3d12descriptorheap)
-   [记录命令列表](#creating-and-recording-command-lists-and-bundles)
-   [示例](#example)
-   [引用计数](#reference-counting)
-   [命令列表错误](#command-list-errors)
-   [捆绑限制](#bundle-restrictions)
-   [相关主题](#related-topics)

在命令列表的外部，API 通过添加另一个级别的命令列表（称为“捆绑”）来利用 GPU 硬件中提供的功能。  捆绑旨在让应用将少量 API 命令组合在一起，供以后执行。 创建捆绑时，驱动程序会执行尽量多的预处理，以降低以后的执行开销。 捆绑可以使用和重用任意次， 而命令列表通常仅执行一次。 但是，只要应用程序在提交新执行之前确保先前的执行完成，则可以多次执行命令列表。 

通常，API 调用构成捆绑，API 调用和捆绑构成命令列表，而命令列表构成单个帧。下图演示了此结构，请注意**命令列表 1** 和**命令列表 2** 中重复使用了**捆绑 1**，图中的 API 方法名称仅用作示例，可以使用许多不同的 API 调用。

![命令、捆绑和命令列表构成帧](images/gpu-workitems.png)

在创建和执行捆绑与直接命令列表方面存在不同的限制，本主题将通篇介绍这些差异。

## <a name="creating-command-lists"></a>创建命令列表

通过调用 [**ID3D12Device::CreateCommandList**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandlist) 创建直接命令列表和捆绑。 此方法将以下参数用作输入：

### <a name="d3d12commandlisttype"></a>D3D12\_COMMAND\_LIST\_TYPE

[**D3D12\_COMMAND\_LIST\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_command_list_type) 枚举指示所要创建的命令列表的类型。 它可以是直接命令列表、捆绑、计算命令列表或复制命令列表。

### <a name="id3d12commandallocator"></a>ID3D12CommandAllocator

命令分配器可让应用管理分配给命令列表的内存。 通过调用[**CreateCommandAllocator**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandallocator) 创建命令分配器。 创建命令列表时，[**D3D12\_COMMAND\_LIST\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_command_list_type) 指定的分配器命令列表类型必须与所要创建的命令列表的类型匹配。 一个给定的分配器可同时与多个“当前正在记录”命令列表相关联，不过，可以使用一个命令分配器来创建任意数量的 [**GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 对象。 

若要回收命令分配器分配的内存，应用需调用 [**ID3D12CommandAllocator::Reset**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandallocator-reset)。 但是，在这样做之前，应用必须确保 GPU 不再执行与分配器关联的任何命令列表；否则调用将会失败。 另请注意，此 API 不是自由线程的，因此，不能同时从多个线程针对同一分配器调用此 API。

### <a name="id3d12pipelinestate"></a>ID3D12PipelineState

命令列表的初始管道状态。 在 Microsoft Direct3D 12 中，大多数图形管道状态是使用 [**ID3D12PipelineState**](/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinestate) 对象在命令列表中设置的。 应用通常在应用初始化期间创建大量此类状态，然后通过使用 [**ID3D12GraphicsCommandList::SetPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate) 更改当前绑定的状态对象来更新状态。 有关管道状态对象的详细信息，请参阅[在 Direct3D 12 中管理图形管道状态](managing-graphics-pipeline-state-in-direct3d-12.md)。

请注意，捆绑不会继承以前的调用在直接命令列表（捆绑的父级）中设置的管道状态。

如果此参数为 NULL，则使用默认状态。

### <a name="id3d12descriptorheap"></a>ID3D12DescriptorHeap

[**ID3D12DescriptorHeap**](/windows/desktop/api/d3d12/nn-d3d12-id3d12descriptorheap) 可让命令列表将资源绑定到图形管道。 直接命令列表必须指定初始描述符堆，但可以通过调用 [**ID3D12GraphicsCommandList::SetDescriptorHeap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps) 来更改命令列表中当前绑定的描述符堆。

在创建时指定描述符堆对于捆绑而言是可选操作。 但是，如果未指定描述符堆，则不允许应用程序在该捆绑中设置任何描述符表。 无论在哪种情况下，都不不允许捆绑更改捆绑中的描述符堆。 如果为捆绑指定了堆，该堆必须与正在调用父级的直接命令列表中当前绑定的堆相匹配。

有关详细信息，请参阅[描述符堆](descriptor-heaps.md)。

## <a name="recording-command-lists"></a>记录命令列表

创建后，命令列表会立即处于记录状态。 也可以通过调用 I[**D3D12GraphicsCommandList::Reset**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-reset) 来重复使用现有的命令列表，这也会使命令列表保持记录状态。 与 [**ID3D12CommandAllocator::Reset**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandallocator-reset) 不同，可以在仍在调用命令列表时调用 **Reset**。 典型的模式是提交一个命令列表，然后立即将其重置为对另一个命令列表重复使用分配的内存。 请注意，每次只能有一个与每个命令分配器关联的命令列表处于记录状态。

命令列表进入记录状态后，只需调用 [**ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 接口的方法即可将命令添加到列表。 其中的许多方法可以实现 Microsoft Direct3D 11 开发人员熟悉的常用 Direct3D 功能；其他 API 是 Direct3D 12 的新功能。

将命令添加到命令列表后，调用 [**Close**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close) 使命令列表退出记录状态。

## <a name="example"></a>示例

以下代码片段演示了命令列表的创建和记录。 请注意，此示例包括以下 Direct3D 12 功能：

-   管道状态对象 - 这些对象用于从命令列表内部设置渲染器管道的大多数状态参数。 有关详细信息，请参阅[在 Direct3D 12 中管理图形管道状态](managing-graphics-pipeline-state-in-direct3d-12.md)。
-   描述符堆 - 应用使用描述符堆来管理内存资源的管道绑定。
-   资源屏障 - 用于管理资源的不同状态转换，例如，从渲染器目标视图转换为着色器资源视图。 有关详细信息，请参阅[使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。

例如，


```C++
void D3D12HelloTriangle::LoadAssets()
{
    // Create an empty root signature.
    {
        CD3DX12_ROOT_SIGNATURE_DESC rootSignatureDesc;
        rootSignatureDesc.Init(0, nullptr, 0, nullptr, D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT);

        ComPtr<ID3DBlob> signature;
        ComPtr<ID3DBlob> error;
        ThrowIfFailed(D3D12SerializeRootSignature(&rootSignatureDesc, D3D_ROOT_SIGNATURE_VERSION_1, &signature, &error));
        ThrowIfFailed(m_device->CreateRootSignature(0, signature->GetBufferPointer(), signature->GetBufferSize(), IID_PPV_ARGS(&m_rootSignature)));
    }

    // Create the pipeline state, which includes compiling and loading shaders.
    {
        ComPtr<ID3DBlob> vertexShader;
        ComPtr<ID3DBlob> pixelShader;

#if defined(_DEBUG)
        // Enable better shader debugging with the graphics debugging tools.
        UINT compileFlags = D3DCOMPILE_DEBUG | D3DCOMPILE_SKIP_OPTIMIZATION;
#else
        UINT compileFlags = 0;
#endif

        ThrowIfFailed(D3DCompileFromFile(GetAssetFullPath(L"shaders.hlsl").c_str(), nullptr, nullptr, "VSMain", "vs_5_0", compileFlags, 0, &vertexShader, nullptr));
        ThrowIfFailed(D3DCompileFromFile(GetAssetFullPath(L"shaders.hlsl").c_str(), nullptr, nullptr, "PSMain", "ps_5_0", compileFlags, 0, &pixelShader, nullptr));

        // Define the vertex input layout.
        D3D12_INPUT_ELEMENT_DESC inputElementDescs[] =
        {
            { "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
            { "COLOR", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 }
        };

        // Describe and create the graphics pipeline state object (PSO).
        D3D12_GRAPHICS_PIPELINE_STATE_DESC psoDesc = {};
        psoDesc.InputLayout = { inputElementDescs, _countof(inputElementDescs) };
        psoDesc.pRootSignature = m_rootSignature.Get();
        psoDesc.VS = { reinterpret_cast<UINT8*>(vertexShader->GetBufferPointer()), vertexShader->GetBufferSize() };
        psoDesc.PS = { reinterpret_cast<UINT8*>(pixelShader->GetBufferPointer()), pixelShader->GetBufferSize() };
        psoDesc.RasterizerState = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);
        psoDesc.BlendState = CD3DX12_BLEND_DESC(D3D12_DEFAULT);
        psoDesc.DepthStencilState.DepthEnable = FALSE;
        psoDesc.DepthStencilState.StencilEnable = FALSE;
        psoDesc.SampleMask = UINT_MAX;
        psoDesc.PrimitiveTopologyType = D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE;
        psoDesc.NumRenderTargets = 1;
        psoDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;
        psoDesc.SampleDesc.Count = 1;
        ThrowIfFailed(m_device->CreateGraphicsPipelineState(&psoDesc, IID_PPV_ARGS(&m_pipelineState)));
    }

    // Create the command list.
    ThrowIfFailed(m_device->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, m_commandAllocator.Get(), m_pipelineState.Get(), IID_PPV_ARGS(&m_commandList)));

    // Command lists are created in the recording state, but there is nothing
    // to record yet. The main loop expects it to be closed, so close it now.
    ThrowIfFailed(m_commandList->Close());

    // Create the vertex buffer.
    {
        // Define the geometry for a triangle.
        Vertex triangleVertices[] =
        {
            { { 0.0f, 0.25f * m_aspectRatio, 0.0f }, { 1.0f, 0.0f, 0.0f, 1.0f } },
            { { 0.25f, -0.25f * m_aspectRatio, 0.0f }, { 0.0f, 1.0f, 0.0f, 1.0f } },
            { { -0.25f, -0.25f * m_aspectRatio, 0.0f }, { 0.0f, 0.0f, 1.0f, 1.0f } }
        };

        const UINT vertexBufferSize = sizeof(triangleVertices);

        // Note: using upload heaps to transfer static data like vert buffers is not 
        // recommended. Every time the GPU needs it, the upload heap will be marshalled 
        // over. Please read up on Default Heap usage. An upload heap is used here for 
        // code simplicity and because there are very few verts to actually transfer.
        ThrowIfFailed(m_device->CreateCommittedResource(
            &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
            D3D12_HEAP_FLAG_NONE,
            &CD3DX12_RESOURCE_DESC::Buffer(vertexBufferSize),
            D3D12_RESOURCE_STATE_GENERIC_READ,
            nullptr,
            IID_PPV_ARGS(&m_vertexBuffer)));

        // Copy the triangle data to the vertex buffer.
        UINT8* pVertexDataBegin;
        CD3DX12_RANGE readRange(0, 0);        // We do not intend to read from this resource on the CPU.
        ThrowIfFailed(m_vertexBuffer->Map(0, &readRange, reinterpret_cast<void**>(&pVertexDataBegin)));
        memcpy(pVertexDataBegin, triangleVertices, sizeof(triangleVertices));
        m_vertexBuffer->Unmap(0, nullptr);

        // Initialize the vertex buffer view.
        m_vertexBufferView.BufferLocation = m_vertexBuffer->GetGPUVirtualAddress();
        m_vertexBufferView.StrideInBytes = sizeof(Vertex);
        m_vertexBufferView.SizeInBytes = vertexBufferSize;
    }

    // Create synchronization objects and wait until assets have been uploaded to the GPU.
    {
        ThrowIfFailed(m_device->CreateFence(0, D3D12_FENCE_FLAG_NONE, IID_PPV_ARGS(&m_fence)));
        m_fenceValue = 1;

        // Create an event handle to use for frame synchronization.
        m_fenceEvent = CreateEvent(nullptr, FALSE, FALSE, nullptr);
        if (m_fenceEvent == nullptr)
        {
            ThrowIfFailed(HRESULT_FROM_WIN32(GetLastError()));
        }

        // Wait for the command list to execute; we are reusing the same command 
        // list in our main loop but for now, we just want to wait for setup to 
        // complete before continuing.
        WaitForPreviousFrame();
    }
}
```



创建并记录一个命令列表后，可以使用命令队列来执行该命令列表。 有关详细信息，请参阅[执行和同步命令列表](executing-and-synchronizing-command-lists.md)。

## <a name="reference-counting"></a>引用计数

大多数 D3D12 API 继续遵循 COM 约定使用引用计数。 一个值得注意的例外情况是 D3D12 图形命令列表 API。 [**ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 中的所有 API 不保留对传入这些 API 的对象的引用。 这意味着，应用程序需负责确保永远不会提交引用已销毁资源的命令列表以供执行。

## <a name="command-list-errors"></a>命令列表错误

[**ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 中的大多数 API 不会返回错误。 建命令列表期间遇到的错误将推迟到 [**ID3D12GraphicsCommandList::Close**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)。 一种例外情况是 DXGI\_ERROR\_DEVICE\_REMOVED，它会进一步推迟错误。 请注意，这不同于 D3D11，其中的许多参数验证错误将以无提示方式丢弃，而永远不会返回给调用方。

应用程序预期会在以下 API 调用中看到 DXGI\_DEVICE\_REMOVED 错误：

-   任何资源创建方法
-   [**ID3D12Resource::Map**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map)
-   [**IDXGISwapChain1::Present1**](https://docs.microsoft.com/windows/desktop/api/dxgi1_2/nf-dxgi1_2-idxgiswapchain1-present1)
-   [**GetDeviceRemovedReason**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getdeviceremovedreason)

## <a name="bundle-restrictions"></a>捆绑限制

限制可使 Direct3D 12 驱动程序在记录时执行大部分与捆绑相关的工作，因此能够以较低的开销运行 [**ExecuteBundle**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executebundle) API。 捆绑引用的所有管道状态对象必须具有相同的渲染器目标格式、深度缓冲区格式和样本说明。

不允许对使用类型创建的命令列表执行以下命令列表 API 调用：D3D12\_COMMAND\_LIST\_TYPE\_BUNDLE:

-   任何 Clear 方法
-   任何 Copy 方法
-   [**DiscardResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-discardresource)
-   [**ExecuteBundle**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executebundle)
-   [**ResourceBarrier**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)
-   [**ResolveSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvesubresource)
-   [**SetPredication**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpredication)
-   [**BeginQuery**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery)
-   [**EndQuery**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery)
-   [**SOSetTargets**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-sosettargets)
-   [**OMSetRenderTargets**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetrendertargets)
-   [**RSSetViewports**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetviewports)
-   [**RSSetScissorRects**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetscissorrects)

可以针对捆绑调用 [**SetDescriptorHeaps**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps)，但捆绑描述符堆必须与调用方命令列表描述符堆相匹配。

如果针对捆绑调用其中的任一 API，则运行时会丢弃该调用。 每当发生这种情况，调试层就会发出错误。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 中的工作提交](command-queues-and-command-lists.md)
</dt> </dl>

 

 




