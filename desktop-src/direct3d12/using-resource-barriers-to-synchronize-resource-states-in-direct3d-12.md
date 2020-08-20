---
title: 在 Direct3D 12 中使用资源屏障同步资源状态
description: 为了减少总体 CPU 使用率并启用驱动程序多线程和预处理，Direct3D 12 将按资源状态管理的责任从图形驱动程序转移到应用程序。
ms.assetid: 3AB3BF34-433C-400B-921A-55B23CCDA44F
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 4db730e4d0a53aad8154c75c0d482ab6f39b965b
ms.sourcegitcommit: 592c9bbd22ba69802dc353bcb5eb30699f9e9403
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2020
ms.locfileid: "88644343"
---
# <a name="using-resource-barriers-to-synchronize-resource-states-in-direct3d-12"></a>在 Direct3D 12 中使用资源屏障同步资源状态

为了减少总体 CPU 使用率并启用驱动程序多线程和预处理，Direct3D 12 将按资源状态管理的责任从图形驱动程序转移到应用程序。 按资源状态的一个例子是，某个纹理资源当前是作为着色器资源视图进行访问，还是作为渲染器目标视图进行访问。 在 Direct3D 11 中，驱动程序需要在后台跟踪此状态。 从 CPU 的角度讲，此操作的开销很大，会明显增大任何种类的多线程设计的复杂性。 在 Microsoft Direct3D 12 中，按资源状态基本上由应用程序使用单个 API [**ID3D12GraphicsCommandList::ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 进行管理。

-   [使用 ResourceBarrier API 管理按资源状态](#using-the-resourcebarrier-api-to-manage-per-resource-state)
    -   [资源状态](#using-resource-barriers-to-synchronize-resource-states-in-direct3d-12)
    -   [资源的初始状态](#initial-states-for-resources)
    -   [读/写资源状态限制](#readwrite-resource-state-restrictions)
    -   [用于呈现反向缓冲区的资源状态](#resource-states-for-presenting-back-buffers)
    -   [丢弃资源](#discarding-resources)
-   [隐式状态转换](#implicit-state-transitions)
    -   [通用状态提升](#common-state-promotion)
    -   [通用状态衰减](#state-decay-to-common)
    -   [性能影响](#performance-implications)
-   [拆分屏障](#split-barriers)
-   [资源屏障示例方案](#resource-barrier-example-scenario)
-   [通用状态提升和衰减示例](#common-state-promotion-and-decay-sample)
-   [拆分屏障示例](#example-of-split-barriers)
-   [相关主题](#related-topics)

## <a name="using-the-resourcebarrier-api-to-manage-per-resource-state"></a>使用 ResourceBarrier API 管理按资源状态

当驱动程序可能需要同步对存储资源的内存的多个访问时，[**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 会通知图形驱动程序。 将使用一个或多个资源屏障描述结构（指示所要声明的资源屏障的类型）调用该方法。

有三种类型的资源屏障：

-   **转换屏障** - 转换屏障指示不同用法之间的一组子资源转换。 使用 [**D3D12\_RESOURCE\_TRANSITION\_BARRIER**](/windows/win32/api/d3d12/ns-d3d12-d3d12_resource_transition_barrier) 结构指定正在转换的子资源，以及子资源之前和之后的状态。****

    系统将验证命令列表中的子资源转换是否与同一命令列表中以前的转换相一致。 调试层会进一步跟踪子资源状态，以查找其他错误，但这种验证是保守性的，而不是穷尽性的。

    请注意，可以使用 D3D12\_RESOURCE\_BARRIER\_ALL\_SUBRESOURCES 标志指定要转换资源中的所有子资源。

-   **失真屏障** - 失真屏障指示两个不同资源的用法之间的转换，这些资源在同一个堆中存在重叠的映射。 这适用于保留的资源和定位的资源。 使用 [**D3D12\_RESOURCE\_ALIASING\_BARRIER**](/windows/win32/api/d3d12/ns-d3d12-d3d12_resource_aliasing_barrier) 结构指定之前和之后的资源。****

    请注意，其中的一个或两个资源可为 NULL，指示任何图块化资源都可能导致失真。 有关使用图块化资源的详细信息，请参阅[图块化资源](../direct3d11/tiled-resources.md)和[立体图块化资源](volume-tiled-resources.md)。

-   **无序访问视图 (UAV) 屏障** - UAV 屏障指示对特定资源的所有 UAV 访问（读取或写入）必须在任何后续 UAV 访问（读取或写入）之间完成。 应用不需要在只读取 UAV 的两个绘制或调度调用之间放置 UAV 屏障。 此外，如果应用程序知道它能够安全地按任意顺序执行 UAV 访问，则不需要在写入同一 UAV 的两个绘制或调度调用之间放置 UAV 屏障。 使用 [**D3D12\_RESOURCE\_UAV\_BARRIER**](/windows/win32/api/d3d12/ns-d3d12-d3d12_resource_uav_barrier) 结构指定屏障应用到的 UAV 资源。 应用程序可为屏障的 UAV 指定 NULL，指示任何 UAV 访问都可能需要屏障。

如果使用资源屏障描述数组调用 [**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)，API 的行为将如同按照元素的提供顺序对每个元素调用该 API 一次。

在任意给定时间，子资源刚好处于一种状态，该状态由提供给 [**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 的 [**D3D12\_RESOURCE\_STATES**](/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) 标志集确定。 应用程序必须确保保守调用 **ResourceBarrier** 之前和之后的状态相一致。****

> [!TIP]
>
> 应用程序应尽可能地将多个转换批处理成一个 API 调用。

 

### <a name="resource-states"></a>资源状态

有关资源可转换到的资源状态的完整列表，请参阅 [**D3D12\_RESOURCE\_STATES**](/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) 枚举的参考主题。

对于拆分资源屏障，另请参阅 [**D3D12\_RESOURCE\_BARRIER\_FLAGS**](/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_barrier_flags)。

### <a name="initial-states-for-resources"></a>资源的初始状态

可以使用用户指定的任何初始状态（对资源描述有效）创建资源，但存在以下例外情况：

-   上传堆的初始状态必须是 D3D12\_RESOURCE\_STATE\_GENERIC\_READ（以下元素的按位 OR 组合）：
    -   D3D12\_RESOURCE\_STATE\_VERTEX\_AND\_CONSTANT\_BUFFER
    -   D3D12\_RESOURCE\_STATE\_INDEX\_BUFFER
    -   D3D12\_RESOURCE\_STATE\_COPY\_SOURCE
    -   D3D12\_RESOURCE\_STATE\_NON\_PIXEL\_SHADER\_RESOURCE
    -   D3D12\_RESOURCE\_STATE\_PIXEL\_SHADER\_RESOURCE
    -   D3D12\_RESOURCE\_STATE\_INDIRECT\_ARGUMENT
-   读回堆的初始状态必须是 D3D12\_RESOURCE\_STATE\_COPY\_DEST。
-   交换链反向缓冲区的初始状态自动设置为 D3D12\_RESOURCE\_STATE\_COMMON。

在某个堆可用作 GPU 复制操作的目标之前，通常该堆必须先转换为 D3D12\_RESOURCE\_STATE\_COPY\_DEST 状态。 但是，在 UPLOAD 堆中创建的资源最初必须处于 GENERIC\_READ 状态且不能从此状态更改，因为只有 CPU 执行写入。 相反，在 READBACK 中创建的已提交资源最初必须处于 COPY\_DEST 状态且不能从此状态更改。

### <a name="readwrite-resource-state-restrictions"></a>读/写资源状态限制

用于描述资源状态的资源状态用法位划分为只读和读/写状态。 [**D3D12\_RESOURCE\_STATES**](/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) 的参考主题指示了枚举中每个位的读/写访问级别。

对于任一资源，最多只能设置一个读/写位。 如果设置了写入位，则不能对该资源设置任何只读位。 如果未设置写入位，则可以设置任意数量的读取位。

### <a name="resource-states-for-presenting-back-buffers"></a>用于呈现反向缓冲区的资源状态

在呈现反向缓冲区之前，该缓冲区必须处于 D3D12\_RESOURCE\_STATE\_COMMON 状态。 请注意，资源状态 D3D12\_RESOURCE\_STATE\_PRESENT 是 D3D12\_RESOURCE\_STATE\_COMMON 的同义词，两者的值均为 0。 如果针对当前不处于此状态的资源调用 [**IDXGISwapChain::Present**](/windows/win32/api/dxgi/nf-dxgi-idxgiswapchain-present)（或 [**IDXGISwapChain1::Present1**](/windows/win32/api/dxgi1_2/nf-dxgi1_2-idxgiswapchain1-present1)），则会发出调试层警告。

### <a name="discarding-resources"></a>丢弃资源

调用 [**ID3D12GraphicsCommandList::DiscardResource**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-discardresource) 时，对于渲染器目标/深度模具资源，资源中的所有子资源必须分别处于 RENDER\_TARGET 状态或 DEPTH\_WRITE 状态。

## <a name="implicit-state-transitions"></a>隐式状态转换

只能从 D3D12\_RESOURCE\_STATE\_COMMON 状态“提升”资源。 同理，资源只能“衰减”到 D3D12\_RESOURCE\_STATE\_COMMON 状态。

### <a name="common-state-promotion"></a>通用状态提升

首次进行 GPU 访问时，所有缓冲区资源以及设置了 D3D12\_RESOURCE\_FLAG\_ALLOW\_SIMULTANEOUS\_ACCESS 标志的纹理，将从 D3D12\_RESOURCE\_STATE\_COMMON 隐式提升到相关状态（包括 GENERIC\_READ），以涵盖任何读取方案。 可以使用以下标志来访问处于 COMMON 状态的任何资源，如同它处于单一状态

<dl> 1 个 WRITE 标志，或  
1 或多个 READ 标志  
</dl>

可以根据下表从 COMMON 状态提升资源：



| 状态标志                    | 可提升状态                             |                                      |
|-------------------------------|----------------------------------------------|--------------------------------------|
|                               | **缓冲区和同时访问纹理** | **非同时访问纹理** |
| VERTEX\_AND\_CONSTANT\_BUFFER | 是                                          | 否                                   |
| INDEX\_BUFFER                 | 是                                          | 否                                   |
| RENDER\_TARGET                | 是                                          | 否                                   |
| UNORDERED\_ACCESS             | 是                                          | 否                                   |
| DEPTH\_WRITE                  | 不<sup>\*</sup>                              | 否                                   |
| DEPTH\_READ                   | 不<sup>\*</sup>                              | 否                                   |
| NON\_PIXEL\_SHADER\_RESOURCE  | 是                                          | 是                                  |
| PIXEL\_SHADER\_RESOURCE       | 是                                          | 是                                  |
| STREAM\_OUT                   | 是                                          | 否                                   |
| INDIRECT\_ARGUMENT            | 是                                          | 否                                   |
| COPY\_DEST                    | 是                                          | 是                                  |
| COPY\_SOURCE                  | 是                                          | 是                                  |
| RESOLVE\_DEST                 | 是                                          | 否                                   |
| RESOLVE\_SOURCE               | 是                                          | 否                                   |
| PREDICATION                   | 是                                          | 否                                   |



 

<sup>\*</sup>深度模具资源必须是不同时访问的纹理，因此永远不能进行隐式提升。

发生这种访问时，提升将类似于隐式资源屏障。 对于后续的访问，必须使用资源屏障来按需更改资源状态。 例如，如果将处于通用状态的资源提升为绘制调用中的 PIXEL\_SHADER\_RESOURCE，然后将其用作复制源，则需要从 PIXEL\_SHADER\_RESOURCE 到 COPY\_SOURCE 的资源状态转换屏障。

请注意，通用状态提升是“无开销的”，因为 GPU 无需执行任何同步等待。 提升表示这一事实：处于 COMMON 状态的资源不应要求使用额外的 GPU 工作或驱动程序跟踪来支持特定的访问。

### <a name="state-decay-to-common"></a>通用状态衰减

通用状态提升的对立面是衰减回到 D3D12\_RESOURCE\_STATE\_COMMON。 满足特定要求的资源被视为无状态，在 GPU 完成执行 [**ExecuteCommandLists**](/windows/win32/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists) 操作后，这些资源会恢复到通用状态。 在同一个 **ExecuteCommandLists** 调用中一起执行的命令列表之间不会发生衰减。

在 GPU 上完成 [**ExecuteCommandLists**](/windows/win32/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists) 操作后，以下资源将会衰减：

-   在复制队列中访问的资源，或**
-   任何队列类型中的缓冲区资源，或**
-   设置了 D3D12\_RESOURCE\_FLAG\_ALLOW\_SIMULTANEOUS\_ACCESS 标志的任何队列类型中的资源，或**
-   隐式提升为只读状态的任何资源。

与通用状态提升一样，衰减也是无开销的，因为不需要附加的同步。 将通用状态提升和衰减相结合有助于消除许多不必要的 [**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 转换。 在某些情况下，这可以大幅提高性能。

缓冲区和同时访问的资源将会衰减到通用状态，而不管它们是使用资源障碍显式转换还是隐式升级。

### <a name="performance-implications"></a>性能影响

记录处于通用状态的资源中的显式 [**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 转换时，正确的做法是使用 D3D12\_RESOURCE\_STATE\_COMMON 或任何可提升状态作为 D3D12\_RESOURCE\_TRANSITION\_BARRIER 结构中的 *BeforeState* 值。 这样可以实现传统状态管理，忽略缓冲区和同时访问纹理的自动衰减。 不过，这可能不是所需的结果，因为使用已知处于通用状态的资源避免转换 **ResourceBarrier** 调用可能会明显提高性能。 资源屏障的开销可能很高。 它们旨在强制执行缓存刷新、内存布局更改和其他同步，而已经处于通用状态的资源可能不需要这些操作。 使用一个非通用状态中的资源屏障转换为当前处于通用状态的资源中的另一个非通用状态的命令列表可能会造成许多不必要的开销。

此外，除非绝对必要（例如，下一次访问位于要求资源最初处于通用的 COPY 命令队列中），否则请避免显式 [**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 转换为 D3D12\_RESOURCE\_STATE\_COMMON。 过多地转换为通用状态可能会大幅降低 GPU 的性能。

总而言之，每当其语义允许在不发出 [**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 调用的情况下转换状态时，都请尝试依赖于通用状态提升和衰减。

## <a name="split-barriers"></a>拆分屏障

使用 D3D12\_RESOURCE\_BARRIER\_FLAG\_BEGIN\_ONLY 标志的资源转换屏障最初是拆分屏障，而转换屏障被视为挂起状态。 当屏障挂起时，GPU 无法读取或写入资源（子资源）。 可应用到使用挂起屏障的资源（子资源）的唯一合法转换屏障是具有相同的之前和之后状态以及 D3D12\_RESOURCE\_BARRIER\_FLAG\_END\_ONLY 标志的屏障，由该屏障完成挂起的转换。****

拆分屏障将提示 GPU，指出在以后的某个时间，处于状态 *A* 的资源将以状态 *B* 使用。 这样，GPU 便可以选择优化转换工作负荷（也许是通过减少或消除执行停滞）。 发出仅限终止的屏障可保证在转到下一个命令之前所有 GPU 转换工作已完成。

使用拆分屏障有助于提高性能，尤其是在多引擎方案中，或者在一个或多个命令列表中对资源进行稀疏读/写转换时。

## <a name="resource-barrier-example-scenario"></a>资源屏障示例方案

以下代码片段演示了 [**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 方法在多线程示例中的用法。

创建深度模具视图，并将其转换为可写状态。


```C++
// Create the depth stencil.
{
    CD3DX12_RESOURCE_DESC shadowTextureDesc(
        D3D12_RESOURCE_DIMENSION_TEXTURE2D,
        0,
        static_cast<UINT>(m_viewport.Width), 
        static_cast<UINT>(m_viewport.Height), 
        1,
        1,
        DXGI_FORMAT_D32_FLOAT,
        1, 
        0,
        D3D12_TEXTURE_LAYOUT_UNKNOWN,
        D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL | D3D12_RESOURCE_FLAG_DENY_SHADER_RESOURCE);

    D3D12_CLEAR_VALUE clearValue;    // Performance tip: Tell the runtime at resource creation the desired clear value.
    clearValue.Format = DXGI_FORMAT_D32_FLOAT;
    clearValue.DepthStencil.Depth = 1.0f;
    clearValue.DepthStencil.Stencil = 0;

    ThrowIfFailed(m_device->CreateCommittedResource(
        &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
        D3D12_HEAP_FLAG_NONE,
        &shadowTextureDesc,
        D3D12_RESOURCE_STATE_DEPTH_WRITE,
        &clearValue,
        IID_PPV_ARGS(&m_depthStencil)));

    // Create the depth stencil view.
    m_device->CreateDepthStencilView(m_depthStencil.Get(), nullptr, m_dsvHeap->GetCPUDescriptorHandleForHeapStart());
}
```



创建顶点缓冲区视图，并先将其从通用状态更改为目标，然后从目标更改为常规可读状态。


```C++
// Create the vertex buffer.
{
    ThrowIfFailed(m_device->CreateCommittedResource(
        &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
        D3D12_HEAP_FLAG_NONE,
        &CD3DX12_RESOURCE_DESC::Buffer(SampleAssets::VertexDataSize),
        D3D12_RESOURCE_STATE_COPY_DEST,
        nullptr,
        IID_PPV_ARGS(&m_vertexBuffer)));

    {
        ThrowIfFailed(m_device->CreateCommittedResource(
            &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
            D3D12_HEAP_FLAG_NONE,
            &CD3DX12_RESOURCE_DESC::Buffer(SampleAssets::VertexDataSize),
            D3D12_RESOURCE_STATE_GENERIC_READ,
            nullptr,
            IID_PPV_ARGS(&m_vertexBufferUpload)));

        // Copy data to the upload heap and then schedule a copy 
        // from the upload heap to the vertex buffer.
        D3D12_SUBRESOURCE_DATA vertexData = {};
        vertexData.pData = pAssetData + SampleAssets::VertexDataOffset;
        vertexData.RowPitch = SampleAssets::VertexDataSize;
        vertexData.SlicePitch = vertexData.RowPitch;

        PIXBeginEvent(commandList.Get(), 0, L"Copy vertex buffer data to default resource...");

        UpdateSubresources<1>(commandList.Get(), m_vertexBuffer.Get(), m_vertexBufferUpload.Get(), 0, 0, 1, &vertexData);
        commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_vertexBuffer.Get(), D3D12_RESOURCE_STATE_COPY_DEST, D3D12_RESOURCE_STATE_VERTEX_AND_CONSTANT_BUFFER));

        PIXEndEvent(commandList.Get());
    }
```



创建索引缓冲区视图，并先将其从通用状态更改为目标，然后从目标更改为常规可读状态。


```C++
// Create the index buffer.
{
    ThrowIfFailed(m_device->CreateCommittedResource(
        &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
        D3D12_HEAP_FLAG_NONE,
        &CD3DX12_RESOURCE_DESC::Buffer(SampleAssets::IndexDataSize),
        D3D12_RESOURCE_STATE_COPY_DEST,
        nullptr,
        IID_PPV_ARGS(&m_indexBuffer)));

    {
        ThrowIfFailed(m_device->CreateCommittedResource(
            &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
            D3D12_HEAP_FLAG_NONE,
            &CD3DX12_RESOURCE_DESC::Buffer(SampleAssets::IndexDataSize),
            D3D12_RESOURCE_STATE_GENERIC_READ,
            nullptr,
            IID_PPV_ARGS(&m_indexBufferUpload)));

        // Copy data to the upload heap and then schedule a copy 
        // from the upload heap to the index buffer.
        D3D12_SUBRESOURCE_DATA indexData = {};
        indexData.pData = pAssetData + SampleAssets::IndexDataOffset;
        indexData.RowPitch = SampleAssets::IndexDataSize;
        indexData.SlicePitch = indexData.RowPitch;

        PIXBeginEvent(commandList.Get(), 0, L"Copy index buffer data to default resource...");

        UpdateSubresources<1>(commandList.Get(), m_indexBuffer.Get(), m_indexBufferUpload.Get(), 0, 0, 1, &indexData);
        commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_indexBuffer.Get(), D3D12_RESOURCE_STATE_COPY_DEST, D3D12_RESOURCE_STATE_INDEX_BUFFER));

        PIXEndEvent(commandList.Get());
    }

    // Initialize the index buffer view.
    m_indexBufferView.BufferLocation = m_indexBuffer->GetGPUVirtualAddress();
    m_indexBufferView.SizeInBytes = SampleAssets::IndexDataSize;
    m_indexBufferView.Format = SampleAssets::StandardIndexFormat;
}
```



创建纹理和着色器资源视图。 纹理将从通用状态更改为目标，然后从目标更改为像素着色器资源。


```C++
    // Create each texture and SRV descriptor.
    const UINT srvCount = _countof(SampleAssets::Textures);
    PIXBeginEvent(commandList.Get(), 0, L"Copy diffuse and normal texture data to default resources...");
    for (int i = 0; i < srvCount; i++)
    {
        // Describe and create a Texture2D.
        const SampleAssets::TextureResource &tex = SampleAssets::Textures[i];
        CD3DX12_RESOURCE_DESC texDesc(
            D3D12_RESOURCE_DIMENSION_TEXTURE2D,
            0,
            tex.Width, 
            tex.Height, 
            1,
            static_cast<UINT16>(tex.MipLevels),
            tex.Format,
            1, 
            0,
            D3D12_TEXTURE_LAYOUT_UNKNOWN,
            D3D12_RESOURCE_FLAG_NONE);

        ThrowIfFailed(m_device->CreateCommittedResource(
            &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
            D3D12_HEAP_FLAG_NONE,
            &texDesc,
            D3D12_RESOURCE_STATE_COPY_DEST,
            nullptr,
            IID_PPV_ARGS(&m_textures[i])));

        {
            const UINT subresourceCount = texDesc.DepthOrArraySize * texDesc.MipLevels;
            UINT64 uploadBufferSize = GetRequiredIntermediateSize(m_textures[i].Get(), 0, subresourceCount);
            ThrowIfFailed(m_device->CreateCommittedResource(
                &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
                D3D12_HEAP_FLAG_NONE,
                &CD3DX12_RESOURCE_DESC::Buffer(uploadBufferSize),
                D3D12_RESOURCE_STATE_GENERIC_READ,
                nullptr,
                IID_PPV_ARGS(&m_textureUploads[i])));

            // Copy data to the intermediate upload heap and then schedule a copy 
            // from the upload heap to the Texture2D.
            D3D12_SUBRESOURCE_DATA textureData = {};
            textureData.pData = pAssetData + tex.Data->Offset;
            textureData.RowPitch = tex.Data->Pitch;
            textureData.SlicePitch = tex.Data->Size;

            UpdateSubresources(commandList.Get(), m_textures[i].Get(), m_textureUploads[i].Get(), 0, 0, subresourceCount, &textureData);
            commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_textures[i].Get(), D3D12_RESOURCE_STATE_COPY_DEST, D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE));
        }

        // Describe and create an SRV.
        D3D12_SHADER_RESOURCE_VIEW_DESC srvDesc = {};
        srvDesc.ViewDimension = D3D12_SRV_DIMENSION_TEXTURE2D;
        srvDesc.Shader4ComponentMapping = D3D12_DEFAULT_SHADER_4_COMPONENT_MAPPING;
        srvDesc.Format = tex.Format;
        srvDesc.Texture2D.MipLevels = tex.MipLevels;
        srvDesc.Texture2D.MostDetailedMip = 0;
        srvDesc.Texture2D.ResourceMinLODClamp = 0.0f;
        m_device->CreateShaderResourceView(m_textures[i].Get(), &srvDesc, cbvSrvHandle);

        // Move to the next descriptor slot.
        cbvSrvHandle.Offset(cbvSrvDescriptorSize);
    }
```



启动某个帧；这不仅会使用 [**ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 来指示要将反向缓冲区用作渲染器目标，而且还会初始化帧资源（针对深度模具缓冲区调用 **ResourceBarrier**）。


```C++
// Assemble the CommandListPre command list.
void D3D12Multithreading::BeginFrame()
{
    m_pCurrentFrameResource->Init();

    // Indicate that the back buffer will be used as a render target.
    m_pCurrentFrameResource->m_commandLists[CommandListPre]->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_PRESENT, D3D12_RESOURCE_STATE_RENDER_TARGET));

    // Clear the render target and depth stencil.
    const float clearColor[] = { 0.0f, 0.0f, 0.0f, 1.0f };
    CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart(), m_frameIndex, m_rtvDescriptorSize);
    m_pCurrentFrameResource->m_commandLists[CommandListPre]->ClearRenderTargetView(rtvHandle, clearColor, 0, nullptr);
    m_pCurrentFrameResource->m_commandLists[CommandListPre]->ClearDepthStencilView(m_dsvHeap->GetCPUDescriptorHandleForHeapStart(), D3D12_CLEAR_FLAG_DEPTH, 1.0f, 0, 0, nullptr);

    ThrowIfFailed(m_pCurrentFrameResource->m_commandLists[CommandListPre]->Close());
}

// Assemble the CommandListMid command list.
void D3D12Multithreading::MidFrame()
{
    // Transition our shadow map from the shadow pass to readable in the scene pass.
    m_pCurrentFrameResource->SwapBarriers();

    ThrowIfFailed(m_pCurrentFrameResource->m_commandLists[CommandListMid]->Close());
}
```



终止某个帧，指示现在会使用反向缓冲区来呈现信息。


```C++
// Assemble the CommandListPost command list.
void D3D12Multithreading::EndFrame()
{
    m_pCurrentFrameResource->Finish();

    // Indicate that the back buffer will now be used to present.
    m_pCurrentFrameResource->m_commandLists[CommandListPost]->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_RENDER_TARGET, D3D12_RESOURCE_STATE_PRESENT));

    ThrowIfFailed(m_pCurrentFrameResource->m_commandLists[CommandListPost]->Close());
}
```



初始化启动某个帧时调用的帧资源，并将深度模具缓冲区转换为可写。


```C++
void FrameResource::Init()
{
    // Reset the command allocators and lists for the main thread.
    for (int i = 0; i < CommandListCount; i++)
    {
        ThrowIfFailed(m_commandAllocators[i]->Reset());
        ThrowIfFailed(m_commandLists[i]->Reset(m_commandAllocators[i].Get(), m_pipelineState.Get()));
    }

    // Clear the depth stencil buffer in preparation for rendering the shadow map.
    m_commandLists[CommandListPre]->ClearDepthStencilView(m_shadowDepthView, D3D12_CLEAR_FLAG_DEPTH, 1.0f, 0, 0, nullptr);

    // Reset the worker command allocators and lists.
    for (int i = 0; i < NumContexts; i++)
    {
        ThrowIfFailed(m_shadowCommandAllocators[i]->Reset());
        ThrowIfFailed(m_shadowCommandLists[i]->Reset(m_shadowCommandAllocators[i].Get(), m_pipelineStateShadowMap.Get()));

        ThrowIfFailed(m_sceneCommandAllocators[i]->Reset());
        ThrowIfFailed(m_sceneCommandLists[i]->Reset(m_sceneCommandAllocators[i].Get(), m_pipelineState.Get()));
    }
}
```



屏障将交换为中间帧，将阴影映射从可写转换为可读。


```C++
void FrameResource::SwapBarriers()
{
    // Transition the shadow map from writeable to readable.
    m_commandLists[CommandListMid]->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_shadowTexture.Get(), D3D12_RESOURCE_STATE_DEPTH_WRITE, D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE));
}
```



终止某个帧时调用 Finish，并将阴影映射转换为通用状态。


```C++
void FrameResource::Finish()
{
    m_commandLists[CommandListPost]->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_shadowTexture.Get(), D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE, D3D12_RESOURCE_STATE_DEPTH_WRITE));
}
```



## <a name="common-state-promotion-and-decay-sample"></a>通用状态提升和衰减示例

``` syntax
    // Create a buffer resource using D3D12_RESOURCE_STATE_COMMON as the init state
    ID3D12Resource *pResource;
    CreateCommittedVertexBufferInCommonState(1024, &pResource);

    // Copy data to the buffer without the need for a barrier.
    // Promotes pResource state to D3D12_RESOURCE_STATE_COPY_DEST.
    pCommandList->CopyBufferRegion(pResource, 0, pOtherResource, 0, 1024); 

    // To use pResource as a vertex buffer a transition barrier is needed.
    // Note the StateBefore is D3D12_RESOURCE_STATE_COPY_DEST.
    D3D12_RESOURCE_BARRIER BarrierDesc = {};
    BarrierDesc.Type = D3D12_RESOURCE_BARRIER_TYPE_TRANSITION;
    BarrierDesc.Flags = D3D12_RESOURCE_BARRIER_FLAG_NONE;
    BarrierDesc.Transition.pResource = pResource;
    BarrierDesc.Transition.Subresource = 0;
    BarrierDesc.Transition.StateBefore = D3D12_RESOURCE_STATE_COPY_DEST;
    BarrierDesc.Transition.StateAfter = D3D12_RESOURCE_STATE_VERTEX_AND_CONSTANT_BUFFER;
    pCommandList->ResourceBarrier( 1, &BarrierDesc );

    // Use pResource as a vertex buffer
    D3D12_VERTEX_BUFFER_VIEW vbView;
    vbView.BufferLocation = pResource->GetGPUVirtualAddress();
    vbView.SizeInBytes = 1024;
    vbView.StrideInBytes = sizeof(MyVertex);

    pCommandList->IASetVertexBuffers(0, 1, &vbView);
    pCommandList->Draw();

    pCommandQueue->ExecuteCommandLists(1, &pCommandList); 
    pCommandList->Reset(pAllocator, pPipelineState);

    // Since the reset command list will be executed in a separate call to 
    // ExecuteCommandLists, the previous state of pResource
    // will have decayed to D3D12_RESOURCE_STATE_COMMON so, again, no barrier is needed
    pCommandList->CopyBufferRegion(pResource, 0, pDifferentResource, 0, 1024);

    FinishRecordingCommandList(pCommandList);
    pCommandQueue->ExecuteCommandLists(1, &pCommandList); 

    WaitForQueue(pCommandQueue);

    // The previous ExecuteCommandLists call has finished so 
    // pResource has decayed to D3D12_RESOURCE_STATE_COMMON
```

## <a name="example-of-split-barriers"></a>拆分屏障示例

以下示例演示如何使用拆分屏障来减少管道停滞。 以下代码不使用拆分屏障：

``` syntax
 D3D12_RESOURCE_BARRIER BarrierDesc = {};
    BarrierDesc.Type = D3D12_RESOURCE_BARRIER_TRANSITION;
    BarrierDesc.Flags = D3D12_RESOURCE_BARRIER_NONE;
    BarrierDesc.Transition.pResource = pResource;
    BarrierDesc.Transition.Subresource = 0;
    BarrierDesc.Transition.StateBefore = D3D12_RESOURCE_STATE_COMMON;
    BarrierDesc.Transition.StateAfter = D3D12_RESOURCE_STATE_RENDER_TARGET;

    pCommandList->ResourceBarrier( 1, &BarrierDesc );
    
    Write(pResource); // ... render to pResource
    OtherStuff(); // .. other gpu work

    // Transition pResource to PIXEL_SHADER_RESOURCE
    BarrierDesc.Transition.StateBefore = D3D12_RESOURCE_STATE_RENDER_TARGET;
    BarrierDesc.Transition.StateAfter = D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE;
    
    pCommandList->ResourceBarrier( 1, &BarrierDesc );

    Read(pResource); // ... read from pResource
```

以下代码使用拆分屏障：

``` syntax
D3D12_RESOURCE_BARRIER BarrierDesc = {};
    BarrierDesc.Type = D3D12_RESOURCE_BARRIER_TRANSITION;
    BarrierDesc.Flags = D3D12_RESOURCE_BARRIER_NONE;
    BarrierDesc.Transition.pResource = pResource;
    BarrierDesc.Transition.Subresource = 0;
    BarrierDesc.Transition.StateBefore = D3D12_RESOURCE_STATE_COMMON;
    BarrierDesc.Transition.StateAfter = D3D12_RESOURCE_STATE_RENDER_TARGET;

    pCommandList->ResourceBarrier( 1, &BarrierDesc );
    
    Write(pResource); // ... render to pResource

    // Done writing to pResource. Start barrier to PIXEL_SHADER_RESOURCE and
    // then do other work
    BarrierDesc.Flags = D3D12_RESOURCE_BARRIER_BEGIN_ONLY;
    BarrierDesc.Transition.StateBefore = D3D12_RESOURCE_STATE_RENDER_TARGET;
    BarrierDesc.Transition.StateAfter = D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE;
    pCommandList->ResourceBarrier( 1, &BarrierDesc );

    OtherStuff(); // .. other gpu work

    // Need to read from pResource so end barrier
    BarrierDesc.Flags = D3D12_RESOURCE_BARRIER_END_ONLY;

    pCommandList->ResourceBarrier( 1, &BarrierDesc );
    Read(pResource); // ... read from pResource
```

## <a name="related-topics"></a>相关主题

[DirectX 高级学习视频教程：资源障碍和状态跟踪](https://www.youtube.com/watch?v=nmB2XMasz2o)

[多引擎同步](./user-mode-heap-synchronization.md)

[Direct3D 12 中的工作提交](command-queues-and-command-lists.md)