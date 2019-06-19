---
title: 上传不同类型的资源
description: 演示如何使用缓冲区将常量缓冲区数据和顶点缓冲区数据上载到 GPU，以及如何在缓冲区中正确地二次分配和放置数据。
ms.assetid: 255B0482-21D6-4276-8009-3F3891879CA1
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: b325fed790a2242d14655d0ed1778cbdec937d47
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224308"
---
# <a name="uploading-different-types-of-resources"></a>上传不同类型的资源

演示如何使用缓冲区将常量缓冲区数据和顶点缓冲区数据上载到 GPU，以及如何在缓冲区中正确地二次分配和放置数据。 使用单个缓冲区可提高内存使用灵活性，并为应用程序提供更严格的内存使用控制。 还显示了 D3D11 和 D3D12 模型之间在上传不同类型资源方面的差异。

-   [上传不同类型的资源](#upload-different-types-of-resources)
    -   [代码示例：D3D11](#code-example-d3d11)
    -   [代码示例：D3D12](#code-example-d3d12)
-   [常量](#constants)
-   [资源](#uploading-different-types-of-resources)
-   [资源大小反射](#resource-size-reflection)
-   [缓冲区对齐](#buffer-alignment)
-   [相关主题](#related-topics)

## <a name="upload-different-types-of-resources"></a>上传不同类型的资源

在 D3D12 中，应用程序创建一个缓冲区以容纳不同类型的资源数据上传，并按不同资源数据的相似方式将资源数据复制到同一个缓冲区。 然后，创建单个视图以将这些资源数据绑定到新资源绑定模型中的图形管道。

在 D3D11 中，应用程序为不同类型的资源数据创建单独的缓冲区（请注意，以下 D3D11 代码示例中使用的不同 `BindFlags`），将每个资源缓冲区显式绑定到图形管道，并基于不同的资源类型使用不同的方法更新资源数据。

在 D3D12 和 D3D11 中，应用程序应仅使用 CPU 将写入一次数据和 GPU 将读取一次数据的上传资源。 如果 GPU 将多次读取数据，则 GPU 不会线性读取数据，或呈现已显著受到 GPU 限制。 最好使用 [**ID3D12GraphicsCommandList::CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion) 或 [**ID3D12GraphicsCommandList::CopyBufferRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion) 将上传缓冲区数据复制到默认资源。 默认资源可以驻留在离散 GPU 上的物理视频内存中。

### <a name="code-example-d3d11"></a>代码示例：D3D11

``` syntax
// D3D11: Separate buffers for each resource type.

void main()
{
    // ...

    // Create a constant buffer.
    float constantBufferData[] = ...;

    D3D11_BUFFER_DESC constantBufferDesc = {0};  
    constantBufferDesc.ByteWidth = sizeof(constantBufferData);  
    constantBufferDesc.Usage = D3D11_USAGE_DYNAMIC;  
    constantBufferDesc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;  
    constantBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;  

    ComPtr<ID3D11Buffer> constantBuffer;
    d3dDevice->CreateBuffer(  
        &constantBufferDesc,  
        NULL,
        &constantBuffer  
        );

    // Create a vertex buffer.
    float vertexBufferData[] = ...;

    D3D11_BUFFER_DESC vertexBufferDesc = { 0 };
    vertexBufferDesc.ByteWidth = sizeof(vertexBufferData);
    vertexBufferDesc.Usage = D3D11_USAGE_DYNAMIC;
    vertexBufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;

    ComPtr<ID3D11Buffer> vertexBuffer;
    d3dDevice->CreateBuffer(
        &vertexBufferDesc,
        NULL,
        &vertexBuffer
        );

    // ...
}

void DrawFrame()
{
    // ...

    // Bind buffers to the graphics pipeline.
    d3dDeviceContext->VSSetConstantBuffers(0, 1, constantBuffer.Get());
    d3dDeviceContext->IASetVertexBuffers(0, 1, vertexBuffer.Get(), ...);

    // Update the constant buffer.
    D3D11_MAPPED_SUBRESOURCE mappedResource;  
    d3dDeviceContext->Map(
        constantBuffer.Get(),
        0, 
        D3D11_MAP_WRITE_DISCARD,
        0,
        &mappedResource
        );
    memcpy(mappedResource.pData, constantBufferData,
        sizeof(contatnBufferData));
    d3dDeviceContext->Unmap(constantBuffer.Get(), 0);  

    // Update the vertex buffer.
    d3dDeviceContext->UpdateSubresource(
        vertexBuffer.Get(),
        0,
        NULL,
        vertexBufferData,
        sizeof(vertexBufferData),
        0
    );

    // ...
}
```

### <a name="code-example-d3d12"></a>代码示例：D3D12

``` syntax
// D3D12: One buffer to accommodate different types of resources

ComPtr<ID3D12Resource> m_spUploadBuffer;
UINT8* m_pDataBegin = nullptr;    // starting position of upload buffer
UINT8* m_pDataCur = nullptr;      // current position of upload buffer
UINT8* m_pDataEnd = nullptr;      // ending position of upload buffer

void main()
{
    //
    // Initialize an upload buffer
    //

    InitializeUploadBuffer(64 * 1024);

    // ...
}

void DrawFrame()
{
    // ...

    // Set vertices data to the upload buffer.

    float vertices[] = ...;
    UINT verticesOffset = 0;
    ThrowIfFailed(
        SetDataToUploadBuffer(
            vertices, sizeof(float), sizeof(vertices) / sizeof(float),
            sizeof(float), 
            verticesOffset
            ));

    // Set constant data to the upload buffer.

    float constants[] = ...;
    UINT constantsOffset = 0;
    ThrowIfFailed(
        SetDataToUploadBuffer(
            constants, sizeof(float), sizeof(constants) / sizeof(float), 
            D3D12_CONSTANT_BUFFER_DATA_PLACEMENT_ALIGNMENT, 
            constantsOffset
            ));

    // Create vertex buffer views for the new binding model.

    D3D12_VERTEX_BUFFER_VIEW vertexBufferViewDesc = {
        m_spUploadBuffer->GetGPUVirtualAddress() + verticesOffset,
        sizeof(vertices), // size
        sizeof(float) * 4,  // stride
    };

    commandList->IASetVertexBuffers( 
        0,
        1,
        &vertexBufferViewDesc,
        ));

    // Create constant buffer views for the new binding model.

    D3D12_CONSTANT_BUFFER_VIEW_DESC constantBufferViewDesc = {
        m_spUploadBuffer->GetGPUVirtualAddress() + constantsOffset,
        sizeof(constants) // size
         };

    d3dDevice->CreateConstantBufferView(
        &constantBufferViewDesc,
        ...
        ));

    // Continue command list building and execution ...
}

//
// Create an upload buffer and keep it always mapped.
//

HRESULT InitializeUploadBuffer(SIZE_T uSize)
{
    HRESULT hr = d3dDevice->CreateCommittedResource(
         &CD3DX12_HEAP_PROPERTIES( D3D12_HEAP_TYPE_UPLOAD ),    
               D3D12_HEAP_FLAG_NONE, 
               &CD3DX12_RESOURCE_DESC::Buffer( uSize ), 
               D3D12_RESOURCE_STATE_GENERIC_READ, nullptr,  
               IID_PPV_ARGS( &m_spUploadBuffer ) );

    if (SUCCEEDED(hr))
    {
        void* pData;
        //
        // No CPU reads will be done from the resource.
        //
        CD3DX12_RANGE readRange(0, 0);
        m_spUploadBuffer->Map( 0, &readRange, &pData ); 
        m_pDataCur = m_pDataBegin = reinterpret_cast< UINT8* >( pData );
        m_pDataEnd = m_pDataBegin + uSize;
    }
    return hr;
}

//
// Sub-allocate from the buffer, with offset aligned.
//

HRESULT SuballocateFromBuffer(SIZE_T uSize, UINT uAlign)
{
    m_pDataCur = reinterpret_cast< UINT8* >(
        Align(reinterpret_cast< SIZE_T >(m_pDataCur), uAlign)
        );

    return (m_pDataCur + uSize > m_pDataEnd) ? E_INVALIDARG : S_OK;
}

//
// Place and copy data to the upload buffer.
//

HRESULT SetDataToUploadBuffer(
    const void* pData, 
    UINT bytesPerData, 
    UINT dataCount, 
    UINT alignment, 
    UINT& byteOffset
    )
{
    SIZE_T byteSize = bytesPerData * dataCount;
    HRESULT hr = SuballocateFromBuffer(byteSize, alignment);
    if (SUCCEEDED(hr))
    {
        byteOffset = UINT(m_pDataCur - m_pDataBegin);
        memcpy(m_pDataCur, pData, byteSize); 
        m_pDataCur += byteSize;
    }
    return hr;
}

//
// Align uLocation to the next multiple of uAlign.
//

UINT Align(UINT uLocation, UINT uAlign)
{
    if ( (0 == uAlign) || (uAlign & (uAlign-1)) )
    {
        ThrowException("non-pow2 alignment");
    }

    return ( (uLocation + (uAlign-1)) & ~(uAlign-1) );
}
```

请注意帮助程序结构 [**CD3DX12\_HEAP\_PROPERTIES**](cd3dx12-heap-properties.md) 和 [**CD3DX12\_RESOURCE\_DESC**](cd3dx12-resource-desc.md) 的用法。

## <a name="constants"></a>常量

若要设置上传或读回堆中的常量、顶点和索引，请使用以下 API：

-   [**ID3D12Device::CreateConstantBufferView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createconstantbufferview)
-   [**ID3D12GraphicsCommandList::IASetVertexBuffers**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetvertexbuffers)
-   [**ID3D12GraphicsCommandList::IASetIndexBuffer**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetindexbuffer)

## <a name="resources"></a>资源

资源是提取 GPU 物理内存使用情况的 D3D 概念。 资源需要 GPU 虚拟地址空间来访问物理内存。 资源创建是自由线程。

对于 D3D12 中的虚拟地址创建和灵活性，有以下三种类型的资源：

-   提交的资源

    提交的资源是各代 D3D 资源的最常见想法。 创建此类资源会分配虚拟地址范围（足以容纳整个资源的隐式堆），并将虚拟地址范围提交由堆封装的物理内存。 必须传递隐式堆属性以将功能奇偶校验与以前的 D3D 版本匹配。 请参阅 [**ID3D12Device::CreateCommittedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommittedresource)。

-   预留的资源

    预留的资源等效于 D3D11 平铺资源。 创建时，仅分配虚拟地址范围，并且不会将其映射到任何堆。 应用程序稍后会将此类资源映射到堆。 此类资源的功能当前在 D3D11 中保持不变，因为可以使用 [**UpdateTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings) 将这些功能映射到磁贴粒度为 64 KB 的堆。 请参阅 [**ID3D12Device::CreateReservedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createreservedresource)

-   放置的资源

    这是 D3D12 的新增资源，应用程序可能会创建独立于资源的堆。 然后，应用程序可能会查找单个堆中的多个资源。 无需创建平铺或预留的资源即可完成此操作，从而启用能够通过应用程序直接创建的所有资源类型的功能。 多个资源可能会重叠，应用程序必须使用 `TiledResourceBarrier` 来正确地重复使用物理内存。 请参阅 [**ID3D12Device::CreatePlacedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createplacedresource)

## <a name="resource-size-reflection"></a>资源大小反射

应用程序必须使用资源大小反射来了解具有未知纹理布局的纹理在堆中需要多少空间。 还支持缓冲区，但主要是为了方便起见。

应用程序应注意主要对齐差异，以便更密集地打包资源。

例如，具有单字节缓冲区的单元素数组返回 64 KB 的大小和 64 KB 的对齐，因为缓冲区当前只有 64 KB 对齐。

此外，具有两个单纹素 64 KB 对齐纹理和单纹素 4 MB 对齐纹理的三元素数组根据数组顺序报告了不同的大小。 如果有 4 MB 对齐的纹理位于中间，则生成的大小为 12 MB。 否则，生成的大小为 8 MB。 返回的对齐将始终为 4 MB（资源数组中所有对齐的超集）。

引用以下 API：

-   [**D3D12\_RESOURCE\_ALLOCATION\_INFO**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_resource_allocation_info)
-   [**GetResourceAllocationInfo**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getresourceallocationinfo)

## <a name="buffer-alignment"></a>缓冲区对齐

值得注意的是，缓冲区对齐限制自 D3D11 起未发生更改：

-   对于多重采样纹理，则为 4 MB。
-   对于单采样纹理和缓冲区，则为 64 KB。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[缓冲区中的二次分配](large-buffers.md)
</dt> </dl>

 

 




