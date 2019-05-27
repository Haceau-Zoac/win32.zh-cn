---
title: 使用资源屏障来同步资源状态在 Direct3D 12
description: 若要减少总体 CPU 使用率，并启用驱动程序的多线程处理和预处理，Direct3D 12 移动从图形驱动程序的每个资源状态管理对应用程序的责任。
ms.assetid: 3AB3BF34-433C-400B-921A-55B23CCDA44F
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 48313fa4065a9224ad69b29d09e05d94800a055f
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224299"
---
# <a name="using-resource-barriers-to-synchronize-resource-states-in-direct3d-12"></a>使用资源屏障来同步资源状态在 Direct3D 12

若要减少总体 CPU 使用率，并启用驱动程序的多线程处理和预处理，Direct3D 12 移动从图形驱动程序的每个资源状态管理对应用程序的责任。 每个资源状态的一个示例是是否纹理资源当前正在访问通过着色器资源视图或呈现目标视图。 在 Direct3D 11 中，驱动程序需要跟踪在后台中的此状态。 这是极 CPU 角度来看，明显会增加任何类型的多线程设计的复杂性。 在 Microsoft Direct3D 12 中，大多数的每个资源状态使用单个 API，在应用程序管理[ **ID3D12GraphicsCommandList::ResourceBarrier**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)。

-   [使用 ResourceBarrier API 来管理每个资源状态](#using-the-resourcebarrier-api-to-manage-per-resource-state)
    -   [资源状态](#using-resource-barriers-to-synchronize-resource-states-in-direct3d-12)
    -   [资源的初始状态](#initial-states-for-resources)
    -   [读/写资源状态限制](#readwrite-resource-state-restrictions)
    -   [用于呈现的后台缓冲区资源状态](#resource-states-for-presenting-back-buffers)
    -   [放弃资源](#discarding-resources)
-   [隐式状态转换](#implicit-state-transitions)
    -   [常见状态升级](#common-state-promotion)
    -   [为常见状态衰减](#state-decay-to-common)
    -   [性能影响](#performance-implications)
-   [拆分障碍](#split-barriers)
-   [资源屏障示例方案](#resource-barrier-example-scenario)
-   [常见的状态升级和衰减示例](#common-state-promotion-and-decay-sample)
-   [拆分屏障的示例](#example-of-split-barriers)
-   [相关的主题](#related-topics)

## <a name="using-the-resourcebarrier-api-to-manage-per-resource-state"></a>使用 ResourceBarrier API 来管理每个资源状态

[**ResourceBarrier** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)通知的情况下，驱动程序可能需要在其中同步对在其中存储资源的内存的多个访问图形驱动程序。 具有一个或多个资源屏障描述结构，它指示资源屏障所声明的类型调用方法。

有三种类型的资源的障碍：

-   **转换屏障**-转换屏障指示一系列子转换之间不同的用法。 一个[ **D3D12\_资源\_过渡\_屏障**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_resource_transition_barrier)结构用于指定转换的子资源，以及*之前*并*后*子资源的状态。

    系统将验证命令列表中的子资源转换是与以前相同的命令列表中的转换一致。 调试层进一步跟踪子资源状态，以查找其他错误，但此验证是保守并不详尽。

    请注意，您可以使用 D3D12\_资源\_屏障\_所有\_子标志，用于指定正在过渡所有子资源中的。

-   **别名屏障**-锯齿屏障指示的两个不同的资源具有重叠映射到同一个堆使用实例之间的转换。 这适用于保留和放置资源。 一个[ **D3D12\_资源\_锯齿\_屏障**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_resource_aliasing_barrier)结构用于同时指定*之前*资源和*后*资源。

    请注意，一个或两个资源可以是 NULL，表示任何平铺的资源可能会导致别名。 有关使用平铺的资源的详细信息，请参阅[平铺资源](https://msdn.microsoft.com/library/windows/desktop/dn786477)并[批量平铺资源](volume-tiled-resources.md)。

-   **无序的访问视图 (UAV) 屏障**-UAV 屏障指示 UAV 的所有访问，同时读取或写入，到特定资源必须都完成之间任何将来的 UAV 访问，同时读取或写入。 不需要的应用将只能从 UAV 读取的两个绘制或调度调用之间的 UAV 障碍。 此外，不需要将写入到同一 UAV 如果应用程序知道它是安全地按任意顺序执行 UAV 访问的两个绘制或调度调用之间的 UAV 障碍。 一个[ **D3D12\_资源\_UAV\_屏障**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_resource_uav_barrier)结构用于指定屏障适用的 UAV 资源。 应用程序可以指定对于屏障的 UAV，指示任何 UAV 访问可能需要屏障，为 NULL。

当[ **ResourceBarrier** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)称为资源屏障说明的数组，该 API 的行为就像对于每个元素，在其中提供的顺序调用一次。

在任何给定时间，子资源处于一个状态，由的一套[ **D3D12\_资源\_状态**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)标志提供给[ **ResourceBarrier**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)。 应用程序必须确保*之前*并*后*的连续调用的状态**ResourceBarrier**即表示同意。

> [!TIP]
>
> 应用程序应批处理多个转换成一个 API 调用，如有可能。

 

### <a name="resource-states"></a>资源状态

有关资源的完整列表状态资源可以之间进行转换，请参阅的参考主题[ **D3D12\_资源\_状态**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)枚举。

有关拆分资源障碍，另请参阅[ **D3D12\_资源\_屏障\_标志**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_barrier_flags)。

### <a name="initial-states-for-resources"></a>资源的初始状态

资源可能会创建与任何用户指定初始状态 （适用于资源说明），但存在以下例外：

-   上传堆必须的初始状态 D3D12\_资源\_状态\_泛型\_读取它是按位 OR 组合的：
    -   D3D12\_RESOURCE\_STATE\_VERTEX\_AND\_CONSTANT\_BUFFER
    -   D3D12\_资源\_状态\_索引\_缓冲区
    -   D3D12\_资源\_状态\_副本\_源
    -   D3D12\_资源\_状态\_非\_像素\_着色器\_资源
    -   D3D12\_RESOURCE\_STATE\_PIXEL\_SHADER\_RESOURCE
    -   D3D12\_资源\_状态\_间接\_参数
-   Readback 堆必须在启动 D3D12\_资源\_状态\_副本\_DEST 状态。
-   交换链后台缓冲区在自动启动 D3D12\_资源\_状态\_常见的状态。

堆可以 GPU 复制操作的目标之前，通常在堆必须先转换到 D3D12\_资源\_状态\_副本\_DEST 状态。 但是，堆创建上传的资源必须在启动，并且不能更改从泛型\_READ 状态，因为仅 CPU 将执行写入操作。 相反，已提交堆在 READBACK 中创建的资源必须在启动和从副本不能更改\_DEST 状态。

### <a name="readwrite-resource-state-restrictions"></a>读/写资源状态限制

资源状态使用情况用于描述资源状态的位被划分到只读和读/写状态。 参考主题[ **D3D12\_资源\_状态**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)指示枚举中的每个位的读/写访问权限级别。

最多只能有一个读/写位可以设置的任何资源。 如果将设置写入位，没有只读位可能为该资源。 如果设置未写入位，则可能会设置任意数量的读取位。

### <a name="resource-states-for-presenting-back-buffers"></a>用于呈现的后台缓冲区资源状态

它显示后台缓冲区之前，必须为 D3D12\_资源\_状态\_常见的状态。 请注意，资源状态 D3D12\_资源\_状态\_存在是对 D3D12 的同义词\_资源\_状态\_常见，并且它们都具有值为 0。 如果[ **IDXGISwapChain::Present** ](https://msdn.microsoft.com/library/windows/desktop/bb174576) (或[ **IDXGISwapChain1::Present1**](https://msdn.microsoft.com/library/windows/desktop/hh446797)) 当前不在此状态下，对资源调用发出调试层警告。

### <a name="discarding-resources"></a>放弃资源

在资源中的所有子都必须位于呈现\_目标状态或深度\_分别写入的呈现器目标/深度模具资源状态，当[ **ID3D12GraphicsCommandList::DiscardResource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-discardresource)调用。

## <a name="implicit-state-transitions"></a>隐式状态转换

资源只能"提升"带 D3D12\_资源\_状态\_常见。 同样，资源将仅"decay"到 D3D12\_资源\_状态\_常见。

### <a name="common-state-promotion"></a>常见状态升级

所有缓冲的资源，以及使用 D3D12 纹理\_资源\_标志\_允许\_同时\_从 D3D12 隐式提升访问标志设置\_资源\_状态\_普遍适用于在第一个 GPU 访问权限，包括泛型的相关状态\_读取以覆盖任何只读的方案。 通过它已处于与一个单一状态，因此可以访问的常见状态中的任何资源

<dl> 1 写标志，或  
1 或更多读取标志  
</dl>

可以从下表所基于的常见状态提升资源：



| 状态标志                    | 不能提升状态                             |                                      |
|-------------------------------|----------------------------------------------|--------------------------------------|
|                               | **缓冲区，并同时访问纹理** | **非同时访问纹理** |
| 顶点\_AND\_常量\_缓冲区 | 是                                          | 否                                   |
| 索引\_缓冲区                 | 是                                          | 否                                   |
| 呈现\_目标                | 是                                          | 否                                   |
| 无序\_访问             | 是                                          | 否                                   |
| 深度\_编写                  | 不<sup>\*</sup>                              | 否                                   |
| 深度\_读取                   | 不<sup>\*</sup>                              | 否                                   |
| 非\_像素\_着色器\_资源  | 是                                          | 是                                  |
| 像素\_着色器\_资源       | 是                                          | 是                                  |
| 流\_出                   | 是                                          | 否                                   |
| 间接\_参数            | 是                                          | 否                                   |
| 复制\_DEST                    | 是                                          | 是                                  |
| 复制\_源                  | 是                                          | 是                                  |
| 解决\_DEST                 | 是                                          | 否                                   |
| 解决\_源               | 是                                          | 否                                   |
| 断言而                   | 是                                          | 否                                   |



 

<sup>\*</sup>深度模具资源必须非同时访问纹理，并因此永远无法隐式升级。

当此访问权限时升级行为像隐式资源屏障。 对于后续访问资源障碍，将需要更改资源状态，如有必要。 例如，如果中的资源的常见状态被提升为像素\_着色器\_绘图调用中的资源，然后用作复制源，从像素的资源状态转换障碍\_着色器\_复制到的资源\_需要源。

请注意，常见状态升级"免费"，因为 GPU 执行任何同步等待时间，不需要。 提升表示常见的状态中的资源不应要求其他的 GPU 工作或驱动程序跟踪，以支持某些访问这一事实。

### <a name="state-decay-to-common"></a>为常见状态衰减

常见状态升级另外一点是回 D3D12 衰减\_资源\_状态\_常见。 满足某些要求的资源被视为无状态服务和常见的状态都有效返回 GPU 完成执行后[ **ExecuteCommandLists** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)操作。 在同一个一起执行的命令列表之间不会发生衰减**ExecuteCommandLists**调用。

以下资源将 decay 何时[ **ExecuteCommandLists** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)在 GPU 上完成操作：

-   要访问复制队列资源*或*
-   在任何队列类型，缓冲区资源*或*
-   纹理上任何队列类型的资源具有 D3D12\_资源\_标志\_允许\_同时\_访问标志设置，*或*
-   隐式提升为只读状态的任何资源。

常见状态促销优惠，如衰减是免费的需要任何其他同步。 结合使用常见状态升级和衰减有助于消除不必要的许多[ **ResourceBarrier** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)转换。 在某些情况下，这可以提供显著的性能改进。

将为常见状态而不考虑是否它们已并显式转换使用资源屏障 decay 或隐式提升的资源。

### <a name="performance-implications"></a>性能影响

当记录显式[ **ResourceBarrier** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)转换中的常见状态的资源，它是正确的以便使用任一 D3D12\_资源\_状态\_常见或任何可提升状态作为*BeforeState* D3D12 中的值\_资源\_转换\_屏障结构。 这样，将忽略自动衰减的缓冲区，并同时访问纹理的传统状态管理。 这可能不需要为避免转换**ResourceBarrier**已知要处于通用状态的资源的调用可以显著提高性能。 资源屏障可能成本高昂。 它们旨在强制缓存刷新数、 内存布局更改和其他可能不是必需的资源已在常见的状态的同步。 为当前中常见的状态的资源上的另一个非常见状态从非常见状态使用资源屏障的命令列表可以引入大量不必要的开销。

此外，避免显式[ **ResourceBarrier** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)转换为 D3D12\_资源\_状态\_常见除非绝对必要 (例如下, 一步的访问是上复制命令队列需要资源中的常见状态开始）。 过多将转换为常见状态可以极大地降低 GPU 性能。

在摘要，尝试依赖于常见状态升级并减慢其语义允许您获取消失而无需发出每当[ **ResourceBarrier** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)调用。

## <a name="split-barriers"></a>拆分障碍

使用 D3D12 资源转换屏障\_资源\_屏障\_标志\_开始\_仅标志开始拆分屏障，转换屏障则称其处于挂起状态。 屏障挂起时无法读取或写入由 GPU (sub) 资源。 可以应用于具有挂起的屏障的 (sub) 资源的唯一合法转换屏障是一个具有相同*之前*并*后*状态和 D3D12\_资源\_屏障\_标志\_最终\_仅标志，哪些屏障完成挂起的转换。

拆分障碍，提供对 GPU 的提示的状态中的资源*A*接下来将使用状态*B*一段时间后。 这为 GPU 提供的选项来优化工作负荷转移，这可能会减少或消除执行延迟。 发出仅限最终的屏障保证所有 GPU 转换工作已经都完成之后才转移到下一个命令。

使用拆分障碍，可以帮助提高性能，尤其是在多引擎方案或资源的读/写在一个或多个命令将列出整个稀疏转换。

## <a name="resource-barrier-example-scenario"></a>资源屏障示例方案

以下代码片段显示了如何使用[ **ResourceBarrier** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)多线程处理的示例中的方法。

创建深度模具视图，其转换为可写状态。


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



创建顶点缓冲区视图，第一次更改其从常见的状态为目标，然后从目标添加到通用的可读状态。


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



创建索引缓冲区视图，第一次更改其从常见的状态为目标，然后从目标添加到通用的可读状态。


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



创建纹理和着色器资源视图。 纹理将更改从到目标，常见状态，然后从目标添加到一个像素着色器资源。


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



从帧的值。这不仅利用[ **ResourceBarrier** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)表明后台缓冲区是要用作呈现器目标，但还可以初始化框架资源 (它调用**ResourceBarrier**深度模具缓冲区上)。


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



结束一个帧，后台缓冲区，该值指示现在用于呈现。


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



正在初始化名为时开始一个帧的帧资源转换为可写的深度模具缓冲区。


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



屏障会交换中间帧，转换从可写卷影映射到可读。


```C++
void FrameResource::SwapBarriers()
{
    // Transition the shadow map from writeable to readable.
    m_commandLists[CommandListMid]->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_shadowTexture.Get(), D3D12_RESOURCE_STATE_DEPTH_WRITE, D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE));
}
```



完成时结束一个帧，调用转换为通用状态的卷影映射。


```C++
void FrameResource::Finish()
{
    m_commandLists[CommandListPost]->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_shadowTexture.Get(), D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE, D3D12_RESOURCE_STATE_DEPTH_WRITE));
}
```



## <a name="common-state-promotion-and-decay-sample"></a>常见的状态升级和衰减示例

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

## <a name="example-of-split-barriers"></a>拆分屏障的示例

下面的示例演示如何使用拆分屏障来降低管道停止。 下面的代码不使用拆分障碍：

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

以下代码使用拆分障碍：

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

<dl> <dt>

[DirectX 高级学习视频教程：资源障碍和状态的跟踪](https://www.youtube.com/watch?v=nmB2XMasz2o)
</dt> <dt>

[同步和多引擎](user-mode-heap-synchronization.md)
</dt> <dt>

[提交工作在 Direct3D 12](command-queues-and-command-lists.md)
</dt> </dl>

 

 




