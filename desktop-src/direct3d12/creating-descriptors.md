---
title: 创建描述符
description: 描述并显示创建以下内容的示例：索引、顶点和常量缓冲区视图；着色器资源、呈现器目标、无序访问、流输出和深度模具视图；以及采样器。 创建描述符的所有方法都是自由线程。
ms.assetid: 0D360A7C-8A2F-49E1-A5CC-98C958B59D1C
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 0b6016dbe8a15d24e392377a607592c0ae982831
ms.sourcegitcommit: fc240ac77d4c40a9f3a27714d7b852abbd234774
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2019
ms.locfileid: "72310684"
---
# <a name="creating-descriptors"></a>创建描述符

描述并显示创建以下内容的示例：索引、顶点和常量缓冲区视图；着色器资源、呈现器目标、无序访问、流输出和深度模具视图；以及采样器。 创建描述符的所有方法都是自由线程。

-   [索引缓冲区视图](#index-buffer-view)
-   [顶点缓冲区视图](#vertex-buffer-view)
-   [着色器资源视图](#shader-resource-view)
-   [常量缓冲区视图](#constant-buffer-view)
-   [采样器](#sampler)
-   [无序访问视图](#unordered-access-view)
-   [流输出视图](#stream-output-view)
-   [呈现器目标视图](#render-target-view)
-   [深度模具视图](#depth-stencil-view)
-   [相关主题](#related-topics)

## <a name="index-buffer-view"></a>索引缓冲区视图

若要创建索引缓冲区视图，请填写 [D3D12\_INDEX\_BUFFER\_VIEW](/windows/desktop/api/d3d12/ns-d3d12-d3d12_index_buffer_view) 结构：

``` syntax
typedef struct D3D12_INDEX_BUFFER_VIEW
{
    D3D12_GPU_VIRTUAL_ADDRESS BufferLocation;
    UINT SizeInBytes;
    DXGI_FORMAT Format;
}   D3D12_INDEX_BUFFER_VIEW;
```

设置缓冲区的位置（调用 [GetGPUVirtualAddress](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-getgpuvirtualaddress)）和大小，并请注意，D3D12\_GPU\_VIRTUAL\_ADDRESS 的定义如下所示：

`typedef UINT64 D3D12_GPU_VIRTUAL_ADDRESS;`

请参阅 [DXGI\_FORMAT](https://docs.microsoft.com/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format) 枚举。 通常，索引缓冲区可能会采用以下定义：

`const DXGI_FORMAT StandardIndexFormat = DXGI_FORMAT_R32_UINT;`

最后调用 [ID3D12GraphicsCommandList::IASetIndexBuffer](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetindexbuffer)。

例如，应用于对象的


```C++
// Initialize the index buffer view.
m_indexBufferView.BufferLocation = m_indexBuffer->GetGPUVirtualAddress();
m_indexBufferView.SizeInBytes = SampleAssets::IndexDataSize;
m_indexBufferView.Format = SampleAssets::StandardIndexFormat;
```



## <a name="vertex-buffer-view"></a>顶点缓冲区视图

若要创建顶点缓冲区视图，请填写 [D3D12\_\_BUFFER\_VIEW](/windows/desktop/api/d3d12/ns-d3d12-d3d12_vertex_buffer_view) 结构：

``` syntax
typedef struct D3D12_VERTEX_BUFFER_VIEW {
    D3D12_GPU_VIRTUAL_ADDRESS BufferLocation;  
    UINT SizeInBytes;  
    UINT StrideInBytes;  
} D3D12_VERTEX_BUFFER_VIEW;
```

设置缓冲区的位置（调用 [GetGPUVirtualAddress](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-getgpuvirtualaddress)）和大小，并请注意，D3D12\_GPU\_VIRTUAL\_ADDRESS 的定义如下所示：

`typedef UINT64 D3D12_GPU_VIRTUAL_ADDRESS;`

步幅通常是 `sizeof(Vertex);` 等单个顶点数据结构的大小，然后调用 [ID3D12GraphicsCommandList::IASetVertexBuffers](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetvertexbuffers)。

例如，应用于对象的


```C++
// Initialize the vertex buffer view.
m_vertexBufferView.BufferLocation = m_vertexBuffer->GetGPUVirtualAddress();
m_vertexBufferView.SizeInBytes = SampleAssets::VertexDataSize;
m_vertexBufferView.StrideInBytes = SampleAssets::StandardVertexStride;
```



## <a name="shader-resource-view"></a>着色器资源视图

若要创建着色器资源视图，请填写 [ D3D12\_SHADER\_RESOURCE\_VIEW\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_shader_resource_view_desc) 结构：

``` syntax
typedef struct D3D12_SHADER_RESOURCE_VIEW_DESC  
    {  
    DXGI_FORMAT Format;  
    D3D12_SRV_DIMENSION ViewDimension;  
    UINT Shader4ComponentMapping;  
    union   
        {  
        D3D12_BUFFER_SRV Buffer;  
        D3D12_TEX1D_SRV Texture1D;  
        D3D12_TEX1D_ARRAY_SRV Texture1DArray;  
        D3D12_TEX2D_SRV Texture2D;  
        D3D12_TEX2D_ARRAY_SRV Texture2DArray;  
        D3D12_TEX2DMS_SRV Texture2DMS;  
        D3D12_TEX2DMS_ARRAY_SRV Texture2DMSArray;  
        D3D12_TEX3D_SRV Texture3D;  
        D3D12_TEXCUBE_SRV TextureCube;  
        D3D12_TEXCUBE_ARRAY_SRV TextureCubeArray;  
        }   ;  
    }   D3D12_SHADER_RESOURCE_VIEW_DESC; 
```

`ViewDimension` 字段设置为零，或者设置为 [D3D12\_BUFFER\_SRV\_FLAGS](/windows/desktop/api/d3d12/ne-d3d12-d3d12_buffer_srv_flags) 枚举的一个值。

[D3D12\_SHADER\_RESOURCE\_VIEW\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_shader_resource_view_desc) 引用的枚举和结构为：

-   [DXGI\_FORMAT](https://docs.microsoft.com/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format)
-   [**D3D12\_BUFFER\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_buffer_srv)
-   [**D3D12\_TEX1D\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex1d_srv)
-   [**D3D12\_TEX1D\_ARRAY\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex1d_array_srv)
-   [**D3D12\_TEX2D\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2d_srv)
-   [**D3D12\_TEX2D\_ARRAY\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2d_array_srv)
-   [**D3D12\_TEX2DMS\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2dms_srv)
-   [**D3D12\_TEX2DMS\_ARRAY\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2dms_array_srv)
-   [**D3D12\_TEX3D\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex3d_srv)
-   [**D3D12\_TEXCUBE\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_texcube_srv)
-   [**D3D12\_TEXCUBE\_ARRAY\_SRV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_texcube_array_srv)

注意下方的 float `ResourceMinLODClamp` 已添加到 SRV，用于 Tex1D/2D/3D/Cube。 在 D3D11 中，它是一个资源属性，但这与它在硬件中的实现方式不匹配。 `StructureByteStride` 已添加到缓冲区 SRV，而在 D3D11 中，它是一个资源属性。 如果步幅为非零值，这表示结构化的缓冲区视图，则格式必须设置为 DXGI\_FORMAT\_UNKNOWN。

最后，若要创建着色器资源视图，请调用 [ID3D12Device::CreateShaderResourceView](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createshaderresourceview)。

例如，应用于对象的


```C++
// Describe and create an SRV.
D3D12_SHADER_RESOURCE_VIEW_DESC srvDesc = {};
srvDesc.ViewDimension = D3D12_SRV_DIMENSION_TEXTURE2D;
srvDesc.Shader4ComponentMapping = D3D12_DEFAULT_SHADER_4_COMPONENT_MAPPING;
srvDesc.Format = tex.Format;
srvDesc.Texture2D.MipLevels = tex.MipLevels;
srvDesc.Texture2D.MostDetailedMip = 0;
srvDesc.Texture2D.ResourceMinLODClamp = 0.0f;
m_device->CreateShaderResourceView(m_textures[i].Get(), &srvDesc, cbvSrvHandle);
```



## <a name="constant-buffer-view"></a>常量缓冲区视图

若要创建常量缓冲区视图，请填写 [ D3D12\_CONSTANT\_BUFFER\_VIEW\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_constant_buffer_view_desc) 结构：

``` syntax
typedef struct D3D12_CONSTANT_BUFFER_VIEW_DESC {
  D3D12_GPU_VIRTUAL_ADDRESS BufferLocation;
  UINT   SizeInBytes;
} D3D12_CONSTANT_BUFFER_VIEW_DESC;
```

然后调用 [ID3D12Device::CreateConstantBufferView](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createconstantbufferview)。

例如，应用于对象的


```C++
// Describe and create a constant buffer view.
D3D12_CONSTANT_BUFFER_VIEW_DESC cbvDesc = {};
cbvDesc.BufferLocation = m_constantBuffer->GetGPUVirtualAddress();
cbvDesc.SizeInBytes = (sizeof(ConstantBuffer) + 255) & ~255;    // CB size is required to be 256-byte aligned.
m_device->CreateConstantBufferView(&cbvDesc, m_cbvHeap->GetCPUDescriptorHandleForHeapStart());
```



## <a name="sampler"></a>取样器

若要创建示例，请填写 [D3D12\_SAMPLER\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_sampler_desc) 结构：

``` syntax
typedef struct D3D12_SAMPLER_DESC
{
    D3D12_FILTER Filter;
    D3D12_TEXTURE_ADDRESS_MODE AddressU;
    D3D12_TEXTURE_ADDRESS_MODE AddressV;
    D3D12_TEXTURE_ADDRESS_MODE AddressW;
    FLOAT MipLODBias;
    UINT MaxAnisotropy;
    D3D12_COMPARISON_FUNC ComparisonFunc;
    FLOAT BorderColor[4]; // RGBA
    FLOAT MinLOD;
    FLOAT MaxLOD;
} D3D12_SAMPLER_DESC;
```

若要填写此结构，参阅以下枚举：

-   [**D3D12\_FILTER**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_filter)
-   [**D3D12\_FILTER\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_filter_type)
-   [D3D12\_FILTER\_REDUCTION\_TYPE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_filter_reduction_type)
-   [D3D12\_TEXTURE\_ADDRESS\_MODE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_texture_address_mode)
-   [D3D12\_COMPARISON\_FUNC](/windows/desktop/api/d3d12/ne-d3d12-d3d12_comparison_func)

最后，调用 [ID3D12Device::CreateSampler](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createsampler)。

例如，应用于对象的


```C++
// Describe and create a sampler.
D3D12_SAMPLER_DESC samplerDesc = {};
samplerDesc.Filter = D3D12_FILTER_MIN_MAG_MIP_LINEAR;
samplerDesc.AddressU = D3D12_TEXTURE_ADDRESS_MODE_WRAP;
samplerDesc.AddressV = D3D12_TEXTURE_ADDRESS_MODE_WRAP;
samplerDesc.AddressW = D3D12_TEXTURE_ADDRESS_MODE_WRAP;
samplerDesc.MinLOD = 0;
samplerDesc.MaxLOD = D3D12_FLOAT32_MAX;
samplerDesc.MipLODBias = 0.0f;
samplerDesc.MaxAnisotropy = 1;
samplerDesc.ComparisonFunc = D3D12_COMPARISON_FUNC_ALWAYS;
m_device->CreateSampler(&samplerDesc, m_samplerHeap->GetCPUDescriptorHandleForHeapStart());
```



## <a name="unordered-access-view"></a>无序访问视图

若要创建无序访问视图，请填写 [ D3D12\_UNORDERED\_ACCESS\_VIEW\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_unordered_access_view_desc) 结构：

``` syntax
typedef struct D3D12_UNORDERED_ACCESS_VIEW_DESC
{
    DXGI_FORMAT Format;
    D3D12_UAV_DIMENSION ViewDimension;

    union
    {
        D3D12_BUFFER_UAV Buffer;
        D3D12_TEX1D_UAV Texture1D;
        D3D12_TEX1D_ARRAY_UAV Texture1DArray;
        D3D12_TEX2D_UAV Texture2D;
        D3D12_TEX2D_ARRAY_UAV Texture2DArray;
        D3D12_TEX3D_UAV Texture3D;
    };
} D3D12_UNORDERED_ACCESS_VIEW_DESC;
```

`ViewDimension` 字段设置为零，或者设置为 [D3D12\_BUFFER\_UAV\_FLAGS](/windows/desktop/api/d3d12/ne-d3d12-d3d12_buffer_uav_flags) 枚举的一个值。

请参阅以下枚举和结构：

-   [DXGI\_FORMAT](https://docs.microsoft.com/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format)
-   [D3D12\_BUFFER\_UAV](/windows/desktop/api/d3d12/ns-d3d12-d3d12_buffer_uav)
-   [**D3D12\_TEX1D\_UAV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex1d_uav)
-   [**D3D12\_TEX1D\_ARRAY\_UAV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex1d_array_uav)
-   [**D3D12\_TEX2D\_UAV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2d_uav)
-   [**D3D12\_TEX2D\_ARRAY\_UAV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2d_array_uav)
-   [**D3D12\_TEX3D\_UAV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex3d_uav)

`StructureByteStride` 已添加到缓冲区 UAV，而在 D3D11 中，它是一个资源属性。 如果步幅为非零值，这表示结构化的缓冲区视图，则格式必须设置为 DXGI\_FORMAT\_UNKNOWN。

最后调用 [ID3D12Device::CreateUnorderedAccessView](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createunorderedaccessview)。

例如，应用于对象的


```C++
// Create the unordered access views (UAVs) that store the results of the compute work.
CD3DX12_CPU_DESCRIPTOR_HANDLE processedCommandsHandle(m_cbvSrvUavHeap->GetCPUDescriptorHandleForHeapStart(), ProcessedCommandsOffset, m_cbvSrvUavDescriptorSize);
for (UINT frame = 0; frame < FrameCount; frame++)
{
    // Allocate a buffer large enough to hold all of the indirect commands
    // for a single frame as well as a UAV counter.
    commandBufferDesc = CD3DX12_RESOURCE_DESC::Buffer(CommandBufferSizePerFrame + sizeof(UINT), D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS);
    ThrowIfFailed(m_device->CreateCommittedResource(
        &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
        D3D12_HEAP_FLAG_NONE,
        &commandBufferDesc,
        D3D12_RESOURCE_STATE_COPY_DEST,
        nullptr,
        IID_PPV_ARGS(&m_processedCommandBuffers[frame])));

    D3D12_UNORDERED_ACCESS_VIEW_DESC uavDesc = {};
    uavDesc.Format = DXGI_FORMAT_UNKNOWN;
    uavDesc.ViewDimension = D3D12_UAV_DIMENSION_BUFFER;
    uavDesc.Buffer.FirstElement = 0;
    uavDesc.Buffer.NumElements = TriangleCount;
    uavDesc.Buffer.StructureByteStride = sizeof(IndirectCommand);
    uavDesc.Buffer.CounterOffsetInBytes = CommandBufferSizePerFrame;
    uavDesc.Buffer.Flags = D3D12_BUFFER_UAV_FLAG_NONE;

    m_device->CreateUnorderedAccessView(            
        m_processedCommandBuffers[frame].Get(),
        m_processedCommandBuffers[frame].Get(),
        &uavDesc,
        processedCommandsHandle);

    processedCommandsHandle.Offset(CbvSrvUavDescriptorCountPerFrame, m_cbvSrvUavDescriptorSize);
}

// Allocate a buffer that can be used to reset the UAV counters and initialize it to 0.
ThrowIfFailed(m_device->CreateCommittedResource(
    &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
    D3D12_HEAP_FLAG_NONE,
    &CD3DX12_RESOURCE_DESC::Buffer(sizeof(UINT)),
    D3D12_RESOURCE_STATE_GENERIC_READ,
    
    nullptr,
    IID_PPV_ARGS(&m_processedCommandBufferCounterReset)));

UINT8* pMappedCounterReset = nullptr;
CD3DX12_RANGE readRange(0, 0);        // We do not intend to read from this resource on the CPU.
ThrowIfFailed(m_processedCommandBufferCounterReset->Map(0, &readRange, reinterpret_cast<void**>(&pMappedCounterReset)));
ZeroMemory(pMappedCounterReset, sizeof(UINT));
m_processedCommandBufferCounterReset->Unmap(0, nullptr);
```



## <a name="stream-output-view"></a>流输出视图

若要创建流输出视图，请填写 [D3D12\_STREAM\_OUTPUT\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_stream_output_desc) 结构。

``` syntax
typedef struct D3D12_STREAM_OUTPUT_DESC  
    {  
    _Field_size_full_(NumEntries)  const D3D12_SO_DECLARATION_ENTRY *pSODeclaration;  
    UINT NumEntries;  
    _Field_size_full_(NumStrides)  const UINT *pBufferStrides;  
    UINT NumStrides;  
    UINT RasterizedStream;  
    }   D3D12_STREAM_OUTPUT_DESC;  
```

然后调用 [ID3D12GraphicsCommandList::SOSetTargets](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-sosettargets)。

## <a name="render-target-view"></a>呈现器目标视图

若要创建呈现器目标视图，请填写 [D3D12\_RENDER\_TARGET\_VIEW\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_render_target_view_desc) 结构。

``` syntax
typedef struct D3D12_RENDER_TARGET_VIEW_DESC
{
    DXGI_FORMAT Format;
    D3D12_RTV_DIMENSION ViewDimension;

    union
    {
        D3D12_BUFFER_RTV Buffer;
        D3D12_TEX1D_RTV Texture1D;
        D3D12_TEX1D_ARRAY_RTV Texture1DArray;
        D3D12_TEX2D_RTV Texture2D;
        D3D12_TEX2D_ARRAY_RTV Texture2DArray;
        D3D12_TEX2DMS_RTV Texture2DMS;
        D3D12_TEX2DMS_ARRAY_RTV Texture2DMSArray;
        D3D12_TEX3D_RTV Texture3D;
    };
} D3D12_RENDER_TARGET_VIEW_DESC;
```

`ViewDimension` 字段设置为零，或者设置为 [D3D12\_RTV\_DIMENSION](/windows/desktop/api/d3d12/ne-d3d12-d3d12_rtv_dimension) 枚举的一个值。

请参阅以下枚举和结构：

-   [DXGI\_FORMAT](https://docs.microsoft.com/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format)
-   [D3D12\_BUFFER\_RTV](/windows/desktop/api/d3d12/ns-d3d12-d3d12_buffer_rtv)
-   [**D3D12\_TEX1D\_RTV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex1d_rtv)
-   [**D3D12\_TEX1D\_ARRAY\_RTV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex1d_array_rtv)
-   [**D3D12\_TEX2D\_RTV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2d_rtv)
-   [**D3D12\_TEX2DMS\_RTV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2dms_rtv)
-   [**D3D12\_TEX2D\_ARRAY\_RTV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2d_array_rtv)
-   [**D3D12\_TEX2DMS\_ARRAY\_RTV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2dms_array_rtv)
-   [**D3D12\_TEX3D\_RTV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex3d_rtv)

最后，调用 [ID3D12Device::CreateRenderTargetView](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createrendertargetview)。

例如，应用于对象的

```C++
// Create descriptor heaps.
{
    // Describe and create a render target view (RTV) descriptor heap.
    D3D12_DESCRIPTOR_HEAP_DESC rtvHeapDesc = {};
    rtvHeapDesc.NumDescriptors = FrameCount;
    rtvHeapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_RTV;
    rtvHeapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
    ThrowIfFailed(m_device->CreateDescriptorHeap(&rtvHeapDesc, IID_PPV_ARGS(&m_rtvHeap)));

    m_rtvDescriptorSize = m_device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);
}

// Create frame resources.
{
    CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart());

    // Create a RTV for each frame.
    for (UINT n = 0; n < FrameCount; n++)
    {
        ThrowIfFailed(m_swapChain->GetBuffer(n, IID_PPV_ARGS(&m_renderTargets[n])));
        m_device->CreateRenderTargetView(m_renderTargets[n].Get(), nullptr, rtvHandle);
        rtvHandle.Offset(1, m_rtvDescriptorSize);
    }
```



## <a name="depth-stencil-view"></a>深度模具视图

若要创建深度模具视图，请填写 [ D3D12\_DEPTH\_STENCIL\_VIEW\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_depth_stencil_view_desc) 结构：

``` syntax
typedef struct D3D12_DEPTH_STENCIL_VIEW_DESC  
    {  
    DXGI_FORMAT Format;  
    D3D12_DSV_DIMENSION ViewDimension;  
    D3D12_DSV_FLAGS Flags;  
    union   
        {  
        D3D12_TEX1D_DSV Texture1D;  
        D3D12_TEX1D_ARRAY_DSV Texture1DArray;  
        D3D12_TEX2D_DSV Texture2D;  
        D3D12_TEX2D_ARRAY_DSV Texture2DArray;  
        D3D12_TEX2DMS_DSV Texture2DMS;  
        D3D12_TEX2DMS_ARRAY_DSV Texture2DMSArray;  
        }   ;  
    }   D3D12_DEPTH_STENCIL_VIEW_DESC;  
```

`ViewDimension` 字段设置为零，或者设置为 [D3D12\_DSV\_DIMENSION](/windows/desktop/api/d3d12/ne-d3d12-d3d12_dsv_dimension) 枚举的一个值。 请参阅标志设置的 [D3D12\_DSV\_FLAGS](/windows/desktop/api/d3d12/ne-d3d12-d3d12_dsv_flags) 枚举。

请参阅以下枚举和结构：

-   [DXGI\_FORMAT](https://docs.microsoft.com/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format)
-   [**D3D12\_TEX1D\_DSV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex1d_dsv)
-   [**D3D12\_TEX1D\_ARRAY\_DSV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex1d_array_dsv)
-   [**D3D12\_TEX2D\_DSV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2d_dsv)
-   [**D3D12\_TEX2D\_ARRAY\_DSV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2d_array_dsv)
-   [**D3D12\_TEX2DMS\_DSV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2dms_dsv)
-   [**D3D12\_TEX2DMS\_ARRAY\_DSV**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tex2dms_array_dsv)

最后，调用 [ID3D12Device::CreateDepthStencilView](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createdepthstencilview)。

例如，应用于对象的


```C++
// Create the depth stencil view.
{
    D3D12_DEPTH_STENCIL_VIEW_DESC depthStencilDesc = {};
    depthStencilDesc.Format = DXGI_FORMAT_D32_FLOAT;
    depthStencilDesc.ViewDimension = D3D12_DSV_DIMENSION_TEXTURE2D;
    depthStencilDesc.Flags = D3D12_DSV_FLAG_NONE;

    D3D12_CLEAR_VALUE depthOptimizedClearValue = {};
    depthOptimizedClearValue.Format = DXGI_FORMAT_D32_FLOAT;
    depthOptimizedClearValue.DepthStencil.Depth = 1.0f;
    depthOptimizedClearValue.DepthStencil.Stencil = 0;

    ThrowIfFailed(m_device->CreateCommittedResource(
        &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
        D3D12_HEAP_FLAG_NONE,
        &CD3DX12_RESOURCE_DESC::Tex2D(DXGI_FORMAT_D32_FLOAT, m_width, m_height, 1, 0, 1, 0, D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL),
        D3D12_RESOURCE_STATE_DEPTH_WRITE,
        &depthOptimizedClearValue,
        IID_PPV_ARGS(&m_depthStencil)
        ));

    m_device->CreateDepthStencilView(m_depthStencil.Get(), &depthStencilDesc, m_dsvHeap->GetCPUDescriptorHandleForHeapStart());
}
```



## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符](descriptors.md)
</dt> <dt>

[描述符堆](descriptor-heaps.md)
</dt> </dl>

 

 




