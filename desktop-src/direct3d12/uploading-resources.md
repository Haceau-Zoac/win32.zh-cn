---
title: 上传不同类型的资源
description: 演示如何使用一个缓冲区以将常量缓冲区数据和顶点缓冲区数据上载到 GPU，以及如何正确二次分配，并将缓冲区中的数据。
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

演示如何使用一个缓冲区以将常量缓冲区数据和顶点缓冲区数据上载到 GPU，以及如何正确二次分配，并将缓冲区中的数据。 一个单一的缓冲区的使用会增加内存使用情况的灵活性，并提供具有更严格的控制的内存使用情况的应用程序。 此外显示了 D3D11 和 D3D12 模型用于上传不同类型的资源之间的差异。

-   [上传不同类型的资源](#upload-different-types-of-resources)
    -   [代码示例：D3D11](#code-example-d3d11)
    -   [代码示例：D3D12](#code-example-d3d12)
-   [常量](#constants)
-   [资源](#uploading-different-types-of-resources)
-   [资源大小反射](#resource-size-reflection)
-   [缓冲区对齐方式](#buffer-alignment)
-   [相关的主题](#related-topics)

## <a name="upload-different-types-of-resources"></a>上传不同类型的资源

D3D12，在应用程序创建一个缓冲区以容纳不同类型的资源数据的上传，并按不同的资源数据类似的方式将资源数据复制到同一缓冲区。 然后，创建单个视图，以便将这些资源数据绑定到新的资源绑定模型中的图形管道。

D3D11，应用程序创建不同类型的资源数据的单独缓冲区 (请注意不同`BindFlags`D3D11 下面的代码示例中使用)、 显式绑定到图形管道的每个资源缓冲区和更新使用的资源数据基于不同的资源类型的不同方法。

在 D3D12 和 D3D11，应用程序只应使用上传资源的 CPU 会将数据写入一次和 GPU 将一次读取。 如果 GPU 将读取数据多次，GPU 将线性增长，读取的数据或呈现已显著 GPU 受限。 更好的选择可能是使用[ **ID3D12GraphicsCommandList::CopyTextureRegion** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)或[ **ID3D12GraphicsCommandList::CopyBufferRegion** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion)将缓冲区中上载的数据复制到默认资源。 默认资源可以驻留在离散的 Gpu 上的物理视频内存。

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

请注意使用的帮助器结构[ **CD3DX12\_堆\_属性**](cd3dx12-heap-properties.md)并[ **CD3DX12\_资源\_DESC**](cd3dx12-resource-desc.md)。

## <a name="constants"></a>常量

若要设置常量、 顶点和上传或 Readback 堆中的索引，请使用以下 Api:

-   [**ID3D12Device::CreateConstantBufferView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createconstantbufferview)
-   [**ID3D12GraphicsCommandList::IASetVertexBuffers**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetvertexbuffers)
-   [**ID3D12GraphicsCommandList::IASetIndexBuffer**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetindexbuffer)

## <a name="resources"></a>资源

资源是 D3D 概念，它将提取的 GPU 物理内存使用情况。 资源需要 GPU 虚拟地址空间访问物理内存。 创建资源是自由线程。

有三种类型的相对虚拟地址创建而又灵活的 D3D12 于的资源：

-   已提交的资源

    已提交的资源是通过代 D3D 资源的最常见的想法。 创建此类资源分配的虚拟地址范围的隐式堆，足以容纳整个资源，并提交到堆中的物理内存的虚拟地址范围。 必须传递隐式堆属性以匹配与以前的 D3D 版本相同的功能。 请参阅[ **ID3D12Device::CreateCommittedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommittedresource)。

-   保留的资源

    等效于 D3D11 平铺资源预留的资源。 在其创建时，只是虚拟的地址范围分配和未映射到任何堆。 应用程序将更高版本将此类资源映射到堆中。 此类资源的功能为当前未更改 D3D11，通过将它们映射到某一堆在使用 64 KB 磁贴粒度[ **UpdateTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings)。 请参阅[ **ID3D12Device::CreateReservedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createreservedresource)

-   放置的资源

    新的 D3D12，应用程序可能会创建堆单独从资源。 然后，应用程序可能会查找单个堆中的多个资源。 这可以而无需创建平铺或保留资源，使能够直接由应用程序创建的所有资源类型的功能。 多个资源可能会重叠，并且该应用程序必须使用`TiledResourceBarrier`若要重新正确使用物理内存。 请参阅[ **ID3D12Device::CreatePlacedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createplacedresource)

## <a name="resource-size-reflection"></a>资源大小反射

应用程序必须使用资源大小反射来了解多少空间纹理具有未知的纹理布局需要在堆中。 缓冲区也支持，但主要是为方便起见。

应用程序应注意的主要的对齐方式的差异，以便更密集地帮助资源的包。

例如，一个字节缓冲区的单个元素数组的大小为 64 KB 和 64 KB 的对齐方式作为返回的缓冲区当前只能是 64KB 对齐。

此外，两个单纹素 64KB 对齐纹理和纹素的单 4MB 对齐纹理的三个元素数组报告根据顺序数组的大小不同。 中间对齐 4 MB 纹理时，所生成的大小为 12 MB。 否则，所生成的大小为 8 MB。 返回的对齐方式将始终为 4 MB，所有的资源数组中的对齐方式的超级集。

参考以下 Api:

-   [**D3D12\_资源\_分配\_信息**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_resource_allocation_info)
-   [**GetResourceAllocationInfo**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getresourceallocationinfo)

## <a name="buffer-alignment"></a>缓冲区对齐方式

缓冲区对齐限制未更改从 D3D11，值得注意的是：

-   对于多重采样的纹理的 4 MB。
-   单示例纹理和缓冲区的 64 KB。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[在缓冲区中的子分配](large-buffers.md)
</dt> </dl>

 

 




