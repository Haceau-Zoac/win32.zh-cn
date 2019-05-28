---
title: 创建和记录命令列表和捆绑包
description: 本主题介绍录制命令列表和 Direct3D 12 应用中的捆绑包。 命令列表和捆绑包允许图形处理单元 (GPU) 上绘制的记录或状态更改为更高版本执行的调用的应用。
ms.assetid: 0074B796-33A4-4AA1-A4E7-48A2A63F25B7
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 0528165307d3ce7b9fdb1c1605e9f88c719d1e45
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224203"
---
# <a name="creating-and-recording-command-lists-and-bundles"></a>创建和记录命令列表和捆绑包

本主题介绍录制命令列表和 Direct3D 12 应用中的捆绑包。 命令列表和捆绑包允许图形处理单元 (GPU) 上绘制的记录或状态更改为更高版本执行的调用的应用。

-   [创建命令列表](#creating-command-lists)
    -   [D3D12\_COMMAND\_LIST\_TYPE](https://docs.microsoft.com/windows)
    -   [ID3D12CommandAllocator](#id3d12commandallocator)
    -   [ID3D12PipelineState](#id3d12pipelinestate)
    -   [ID3D12DescriptorHeap](#id3d12descriptorheap)
-   [录制命令列表](#creating-and-recording-command-lists-and-bundles)
-   [示例](#example)
-   [引用计数](#reference-counting)
-   [命令列表错误](#command-list-errors)
-   [捆绑包限制](#bundle-restrictions)
-   [相关的主题](#related-topics)

超出命令列表 API 利用 GPU 的硬件中提供功能通过添加第二个级别的命令列表，称为*捆绑包*。 捆绑包的用途是允许组一起使用以获得更高版本执行 API 命令有少量的应用程序。 在捆绑包创建时，该驱动程序将执行太多预处理，因为可以使这些比较便宜，若要执行更高版本。 捆绑包旨在使用和重复使用任意次数。 但是，命令列表通常执行一次。 但是，命令列表*可以*多次执行 （只要该应用程序可以确保在提交新执行之前已完成上一个执行）。

通常，正在显示的捆绑包，以及 API 调用和捆绑包到的 API 调用到命令列表中，并到单个帧的命令列表向上生成在下图中，注意是重用**捆绑包 1**中**命令列表1**并**命令列表 2**，和关系图中的名称都只是作为示例的 API 方法，许多不同的 API 调用可以使用。

![生成命令，捆绑包和命令列出到帧](images/gpu-workitems.png)

有不同的创建和执行捆绑包和直接的命令列表，限制，在本主题中注明了这些差异。

## <a name="creating-command-lists"></a>创建命令列表

直接通过调用创建命令列表和捆绑包[ **ID3D12Device::CreateCommandList**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandlist)。 此方法作为输入采用以下参数：

### <a name="d3d12commandlisttype"></a>D3D12\_命令\_列表\_类型

[ **D3D12\_命令\_列表\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)枚举指示正在创建的命令列表的类型。 它可以是直接的命令列表、 捆绑包、 计算命令列表或复制命令列表。

### <a name="id3d12commandallocator"></a>ID3D12CommandAllocator

命令分配器允许此应用分配并供其命令将列出对内存进行管理。 通过调用创建命令分配器[ **CreateCommandAllocator**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandallocator)。 此命令时创建命令列表，列出的分配器指定的类型[ **D3D12\_命令\_列表\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type)，必须匹配的命令列表类型正在创建。 给定的分配器可以与多个不相关联*当前正在录制*虽然可以使用一个命令分配器来创建任意数量的但一次命令列表[ **GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)对象。

若要回收命令分配器分配的内存，应用程序调用[ **ID3D12CommandAllocator::Reset**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandallocator-reset)。 但这样做之前应用程序必须确保 GPU 不再执行与分配器; 相关联的任何命令列表否则，调用将失败。 此外，请注意，此 API 不是自由线程，因此不能调用同一分配器同时从多个线程。

### <a name="id3d12pipelinestate"></a>ID3D12PipelineState

命令列表初始管道状态。 在 Microsoft Direct3D 12 中，大多数图形管道状态设置内命令列表使用[ **ID3D12PipelineState** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12pipelinestate)对象。 应用程序将创建大量的这些选项中，通常在应用初始化期间，然后通过更改当前绑定的状态对象使用更新的状态[ **ID3D12GraphicsCommandList::SetPipelineState** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate). 有关管道状态对象的详细信息，请参阅[管理图形管道 Direct3D 12 中的状态](managing-graphics-pipeline-state-in-direct3d-12.md)。

请注意捆绑包不继承设置上一个调用是其父级的直接的命令列表中的管道状态。

如果此参数为 NULL，则使用默认状态。

### <a name="id3d12descriptorheap"></a>ID3D12DescriptorHeap

[ **ID3D12DescriptorHeap** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12descriptorheap)允许将资源绑定到图形管道的命令列表。 直接命令列表，必须指定的初始描述符堆，但可能会通过调用更改命令列表中的当前绑定描述符堆[ **ID3D12GraphicsCommandList::SetDescriptorHeap** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps).

在创建时指定描述符堆是可选的捆绑包。 如果未指定描述符堆，但是，应用程序不是允许设置该捆绑中的任何描述符表。 无论哪种方式，捆绑包不允许更改对绑定中的描述符堆。 如果为绑定指定堆，则它必须匹配调用父级的直接的命令列表中当前绑定的堆。

有关详细信息，请参阅[描述符堆](descriptor-heaps.md)。

## <a name="recording-command-lists"></a>录制命令列表

正在创建后立即命令列表是在录制状态中。 此外可以重新使用现有的命令列表通过调用我[**D3D12GraphicsCommandList::Reset**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-reset)，这也使处于录制状态的命令列表。 与不同[ **ID3D12CommandAllocator::Reset**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandallocator-reset)，可以调用**重置**仍在执行命令列表时。 典型模式是提交命令列表，然后立即重置它重复使用另一个命令列表的已分配的内存。 请注意该只能有一个命令列出与每个命令分配器相关联可能一次是在录制状态中。

将录制状态命令列表后，只需调用的方法[ **ID3D12GraphicsCommandList** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)接口将命令添加到列表。 许多这些方法都允许将是 Microsoft Direct3D 11 开发人员; 所熟悉的常用 Direct3D 功能其他 Api 是 Direct3D 12 的新增功能。

将命令添加到命令列表之后, 转换从录制状态的命令列表通过调用[**关闭**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)。

## <a name="example"></a>示例

下面的代码段演示了创建和记录的命令列表。 请注意，此示例包含以下 Direct3D 12 功能：

-   管道状态对象-使用这些设置大多数呈现管道从命令列表中的状态参数。 有关详细信息，请参阅[管理图形管道 Direct3D 12 中的状态](managing-graphics-pipeline-state-in-direct3d-12.md)。
-   描述符堆的应用程序使用描述符堆，以管理管道绑定到内存资源。
-   资源屏障-这是用于从一个状态到另一个管理资源的转换，例如，从呈现目标视图为着色器资源视图。 有关详细信息，请参阅[使用资源屏障来同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。

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



创建命令列表并将其记录后，它可以执行使用的命令队列。 有关详细信息，请参阅[Executing 和同步命令列表](executing-and-synchronizing-command-lists.md)。

## <a name="reference-counting"></a>引用计数

大多数 D3D12 Api 继续使用引用计数以下 COM 约定。 一个值得注意的例外是 D3D12 图形命令列表 Api。 上的所有 Api [ **ID3D12GraphicsCommandList** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)不具有对传递到这些 Api 的对象的引用。 这意味着应用程序负责确保命令列表永远不会提交以引用已销毁的资源的执行。

## <a name="command-list-errors"></a>命令列表错误

在大多数 Api [ **ID3D12GraphicsCommandList** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)不会返回错误。 在命令列表创建过程中遇到的错误延迟，直到[ **ID3D12GraphicsCommandList::Close**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)。 一个例外是 DXGI\_错误\_设备\_已删除，进一步延迟。 请注意，这不同于 D3D11，其中许多参数验证错误将以无提示方式删除和永远不会返回给调用方。

应用程序可能会看到 DXGI\_设备\_以下 API 调用中的已删除错误：

-   任何资源创建方法
-   [**ID3D12Resource::Map**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)
-   [**IDXGISwapChain1::Present1**](https://msdn.microsoft.com/library/windows/desktop/hh446797)
-   [**GetDeviceRemovedReason**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getdeviceremovedreason)

## <a name="bundle-restrictions"></a>捆绑包限制

限制启用 Direct3D 12 驱动程序用于执行关联的工作与捆绑包在记录时，从而使大多数[ **ExecuteBundle** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executebundle) API 来运行具有较低的开销。 引用的捆绑包的所有管道状态对象必须都具有相同的呈现目标格式、 深度缓冲区格式和示例的说明。

创建类型的命令列表上不允许使用以下命令列表 API 调用：D3D12\_命令\_列表\_类型\_捆绑包：

-   任何 Clear 方法
-   任何复制方法
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

[**SetDescriptorHeaps** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps)可以调用一个包，但绑定描述符堆必须匹配调用的命令列表描述符堆。

如果在绑定上调用这些 Api 的任何程序，运行时将删除该调用。 每当发生这种情况时，调试层将发出错误。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[提交工作在 Direct3D 12](command-queues-and-command-lists.md)
</dt> </dl>

 

 




