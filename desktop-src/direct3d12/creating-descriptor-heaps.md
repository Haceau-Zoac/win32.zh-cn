---
title: 创建描述符堆
description: 若要创建和配置描述符堆，必须选择描述符堆类型、 确定多少描述符包含，并设置指示是否可见的 CPU 和/或着色器可见的标志。
ms.assetid: 58677023-692C-4BA4-90B7-D568F3DD3F73
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 78404af10597bc9691cd4fa8401d66c7b8220c1f
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223843"
---
# <a name="creating-descriptor-heaps"></a>创建描述符堆

若要创建和配置描述符堆，必须选择描述符堆类型、 确定多少描述符包含，并设置指示是否可见的 CPU 和/或着色器可见的标志。

-   [描述符堆类型](#descriptor-heap-types)
-   [描述符堆属性](#descriptor-heap-properties)
-   [描述符句柄](#descriptor-handles)
-   [描述符堆方法](#descriptor-heap-methods)
-   [最小描述符堆包装器](#minimal-descriptor-heap-wrapper)
-   [相关的主题](#related-topics)

## <a name="descriptor-heap-types"></a>描述符堆类型

堆的类型由一个成员[ **D3D12\_描述符\_堆\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_descriptor_heap_type)枚举：

``` syntax
typedef enum D3D12_DESCRIPTOR_HEAP_TYPE
{
    D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV,    // Constant buffer/Shader resource/Unordered access views
    D3D12_DESCRIPTOR_HEAP_TYPE_SAMPLER,        // Samplers
    D3D12_DESCRIPTOR_HEAP_TYPE_RTV,            // Render target view
    D3D12_DESCRIPTOR_HEAP_TYPE_DSV,            // Depth stencil view
    D3D12_DESCRIPTOR_HEAP_TYPE_NUM_TYPES       // Simply the number of descriptor heap types
} D3D12_DESCRIPTOR_HEAP_TYPE;
```

## <a name="descriptor-heap-properties"></a>描述符堆属性

堆属性在设置[ **D3D12\_描述符\_堆\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_descriptor_heap_desc)结构，它将同时引用[ **D3D12\_描述符\_堆\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_descriptor_heap_type)并[ **D3D12\_描述符\_堆\_标志**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_descriptor_heap_flags)枚举。

标志 D3D12\_描述符\_堆\_标志\_着色器\_VISIBLE 可以选择性地设置上一个描述符堆，以指示它是由着色器绑定上命令列表，以便引用。 描述符堆创建*而无需*此标志允许应用程序阶段描述符选项在 CPU 内存中之前将其复制到着色器可见描述符堆，为方便起见。 但它也是相当不错的应用程序直接创建描述符到着色器可见描述符堆而不需要分阶段在 CPU 上的任何内容。

此标志仅适用于 CBV、 SRV、 UAV 和取样器。 它不适用于其他描述符堆类型由于着色器不能直接引用其他类型。

例如，描述并创建取样器描述符堆。


```C++
// Describe and create a sampler descriptor heap.
D3D12_DESCRIPTOR_HEAP_DESC samplerHeapDesc = {};
samplerHeapDesc.NumDescriptors = 1;
samplerHeapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_SAMPLER;
samplerHeapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;
ThrowIfFailed(m_device->CreateDescriptorHeap(&samplerHeapDesc, IID_PPV_ARGS(&m_samplerHeap)));
```



描述和创建常量缓冲区视图 (CBV)、 着色器资源视图 (SRV) 和无序的访问视图 (UAV) 描述符堆。


```C++
// Describe and create a shader resource view (SRV) and unordered
// access view (UAV) descriptor heap.
D3D12_DESCRIPTOR_HEAP_DESC srvUavHeapDesc = {};
srvUavHeapDesc.NumDescriptors = DescriptorCount;
srvUavHeapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;
srvUavHeapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;
ThrowIfFailed(m_device->CreateDescriptorHeap(&srvUavHeapDesc, IID_PPV_ARGS(&m_srvUavHeap)));

m_rtvDescriptorSize = m_device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);
m_srvUavDescriptorSize = m_device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);
```



## <a name="descriptor-handles"></a>描述符句柄

[ **D3D12\_GPU\_描述符\_处理**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_gpu_descriptor_handle)并[ **D3D12\_CPU\_描述符\_处理**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_cpu_descriptor_handle)结构标识描述符堆中的特定描述符。 句柄是有点类似于指针，但应用程序必须未对其取消引用手动;否则，该行为不确定。 使用句柄必须经过 API。 可以自由地复制或传递到对 / 使用描述符的 Api 本身的句柄。 没有无引用计数，因此应用程序必须确保它不删除基础描述符堆后使用一个句柄。

应用程序可以找出给定的描述符堆类型描述符的增量大小，以便它们可以在手动启动从该句柄到基描述符堆中生成的句柄的任何位置。 应用程序必须永远不会进行硬编码描述符句柄增量大小，并应始终将这些查询为给定的设备实例;否则，该行为不确定。 应用程序还必须不使用增量大小和句柄来执行其自己的检查或操作描述符堆数据，因为执行此操作的结果是未定义。 不可能实际使用句柄，作为指针，而是而不是作为代理以避免意外取消引用指针。

> [!Note]
>
> 没有帮助器结构，CD3DX12\_GPU\_描述符\_句柄，在标头 d3dx12.h，继承中定义[ **D3D12\_GPU\_描述符\_处理**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_gpu_descriptor_handle)结构并提供了初始化和其他有用的操作。 同样 CD3DX12\_CPU\_描述符\_为定义句柄帮助器结构[ **D3D12\_CPU\_描述符\_句柄**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_cpu_descriptor_handle)结构。

 

这两个这些帮助器结构时，使用填充命令列表。


```C++
// Fill the command list with all the render commands and dependent state.
void D3D12nBodyGravity::PopulateCommandList()
{
    // Command list allocators can only be reset when the associated
    // command lists have finished execution on the GPU; apps should use
    // fences to determine GPU execution progress.
    ThrowIfFailed(m_commandAllocators[m_frameIndex]->Reset());

    // However, when ExecuteCommandList() is called on a particular command
    // list, that command list can then be reset at any time and must be before
    // re-recording.
    ThrowIfFailed(m_commandList->Reset(m_commandAllocators[m_frameIndex].Get(), m_pipelineState.Get()));

    // Set necessary state.
    m_commandList->SetPipelineState(m_pipelineState.Get());
    m_commandList->SetGraphicsRootSignature(m_rootSignature.Get());

    m_commandList->SetGraphicsRootConstantBufferView(RootParameterCB, m_constantBufferGS->GetGPUVirtualAddress() + m_frameIndex * sizeof(ConstantBufferGS));

    ID3D12DescriptorHeap* ppHeaps[] = { m_srvUavHeap.Get() };
    m_commandList->SetDescriptorHeaps(_countof(ppHeaps), ppHeaps);

    m_commandList->IASetVertexBuffers(0, 1, &m_vertexBufferView);
    m_commandList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_POINTLIST);
    m_commandList->RSSetScissorRects(1, &m_scissorRect);

    // Indicate that the back buffer will be used as a render target.
    m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_PRESENT, D3D12_RESOURCE_STATE_RENDER_TARGET));

    CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart(), m_frameIndex, m_rtvDescriptorSize);
    m_commandList->OMSetRenderTargets(1, &rtvHandle, FALSE, nullptr);

    // Record commands.
    const float clearColor[] = { 0.0f, 0.0f, 0.1f, 0.0f };
    m_commandList->ClearRenderTargetView(rtvHandle, clearColor, 0, nullptr);

    // Render the particles.
    float viewportHeight = static_cast<float>(static_cast<UINT>(m_viewport.Height) / m_heightInstances);
    float viewportWidth = static_cast<float>(static_cast<UINT>(m_viewport.Width) / m_widthInstances);
    for (UINT n = 0; n < ThreadCount; n++)
    {
        const UINT srvIndex = n + (m_srvIndex[n] == 0 ? SrvParticlePosVelo0 : SrvParticlePosVelo1);

        D3D12_VIEWPORT viewport;
        viewport.TopLeftX = (n % m_widthInstances) * viewportWidth;
        viewport.TopLeftY = (n / m_widthInstances) * viewportHeight;
        viewport.Width = viewportWidth;
        viewport.Height = viewportHeight;
        viewport.MinDepth = D3D12_MIN_DEPTH;
        viewport.MaxDepth = D3D12_MAX_DEPTH;
        m_commandList->RSSetViewports(1, &viewport);

        CD3DX12_GPU_DESCRIPTOR_HANDLE srvHandle(m_srvUavHeap->GetGPUDescriptorHandleForHeapStart(), srvIndex, m_srvUavDescriptorSize);
        m_commandList->SetGraphicsRootDescriptorTable(RootParameterSRV, srvHandle);

        m_commandList->DrawInstanced(ParticleCount, 1, 0, 0);
    }

    m_commandList->RSSetViewports(1, &m_viewport);

    // Indicate that the back buffer will now be used to present.
    m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_RENDER_TARGET, D3D12_RESOURCE_STATE_PRESENT));

    ThrowIfFailed(m_commandList->Close());
}
```



## <a name="descriptor-heap-methods"></a>描述符堆方法

描述符堆 ([**ID3D12DescriptorHeap**](/windows/desktop/api/D3D12/nn-d3d12-id3d12descriptorheap)) 继承[ **ID3D12Pageable**](https://msdn.microsoft.com/en-us/library/Dn788704(v=VS.85).aspx)。 此堆施加堆上的应用程序，就像资源描述符驻留管理的责任。 驻留管理方法仅适用于着色器可见堆由于非的着色器可见堆看不到 GPU 直接。

[ **ID3D12Device::GetDescriptorHandleIncrementSize** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getdescriptorhandleincrementsize)方法使应用程序来手动偏移到堆的句柄 （生成处理到任意位置描述符堆中）。 来自于堆开始位置的句柄[ **ID3D12DescriptorHeap::GetCPUDescriptorHandleForHeapStart**](/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart)/[**ID3D12DescriptorHeap::GetGPUDescriptorHandleForHeapStart**](/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getgpudescriptorhandleforheapstart)。 偏移通过添加增量大小\*到描述符堆开始的偏移的说明符的数目。 请注意，增量大小不能被视为字节大小由于应用程序必须取消引用句柄如同它们是内存 – 内存指向具有非标准化的布局和即使对于给定的设备而异。

[**GetCPUDescriptorHandleForHeapStart** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart)返回 CPU 可见描述符的 CPU 句柄堆。 它将返回 NULL 句柄 （和将报告错误的调试层） 如果描述符堆不可见的 CPU。

[**GetGPUDescriptorHandleForHeapStart** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getgpudescriptorhandleforheapstart)返回着色器可见描述符的 GPU 句柄堆。 它将返回 NULL 句柄 （和将报告错误的调试层） 如果描述符堆不可见的着色器。

例如，创建呈现目标视图以显示使用 11on12 设备 D2D 文本。


```C++
    // Create descriptor heaps.
    {
        // Describe and create a render target view (RTV) descriptor heap.
        D3D12_DESCRIPTOR_HEAP_DESC rtvHeapDesc = {};
        rtvHeapDesc.NumDescriptors = FrameCount;
        rtvHeapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_RTV;
        rtvHeapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
        ThrowIfFailed(m_d3d12Device->CreateDescriptorHeap(&rtvHeapDesc, IID_PPV_ARGS(&m_rtvHeap)));

        m_rtvDescriptorSize = m_d3d12Device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);
    }

    // Create frame resources.
    {
        CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart());

        // Create a RTV, D2D render target, and a command allocator for each frame.
        for (UINT n = 0; n < FrameCount; n++)
        {
            ThrowIfFailed(m_swapChain->GetBuffer(n, IID_PPV_ARGS(&m_renderTargets[n])));
            m_d3d12Device->CreateRenderTargetView(m_renderTargets[n].Get(), nullptr, rtvHandle);

            // Create a wrapped 11On12 resource of this back buffer. Since we are 
            // rendering all D3D12 content first and then all D2D content, we specify 
            // the In resource state as RENDER_TARGET - because D3D12 will have last 
            // used it in this state - and the Out resource state as PRESENT. When 
            // ReleaseWrappedResources() is called on the 11On12 device, the resource 
            // will be transitioned to the PRESENT state.
            D3D11_RESOURCE_FLAGS d3d11Flags = { D3D11_BIND_RENDER_TARGET };
            ThrowIfFailed(m_d3d11On12Device->CreateWrappedResource(
                m_renderTargets[n].Get(),
                &d3d11Flags,
                D3D12_RESOURCE_STATE_RENDER_TARGET,
                D3D12_RESOURCE_STATE_PRESENT,
                IID_PPV_ARGS(&m_wrappedBackBuffers[n])
                ));

            // Create a render target for D2D to draw directly to this back buffer.
            ComPtr<IDXGISurface> surface;
            ThrowIfFailed(m_wrappedBackBuffers[n].As(&surface));
            ThrowIfFailed(m_d2dDeviceContext->CreateBitmapFromDxgiSurface(
                surface.Get(),
                &bitmapProperties,
                &m_d2dRenderTargets[n]
                ));

            rtvHandle.Offset(1, m_rtvDescriptorSize);

            ThrowIfFailed(m_d3d12Device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&m_commandAllocators[n])));
        }
    
    }
```



## <a name="minimal-descriptor-heap-wrapper"></a>最小描述符堆包装器

应用程序开发人员可能会想要构建自己的管理描述符句柄和堆的帮助器代码。 基本示例如下所示。 例如，更复杂的包装器无法跟踪描述符类型是在堆中的和存储描述符创建参数。

``` syntax
class CDescriptorHeapWrapper
{
public:
    CDescriptorHeapWrapper() { memset(this, 0, sizeof(*this)); }

    HRESULT Create(
        ID3D12Device* pDevice, 
        D3D12_DESCRIPTOR_HEAP_TYPE Type, 
        UINT NumDescriptors, 
        bool bShaderVisible = false)
    {
        D3D12_DESCRIPTOR_HEAP_DESC Desc;
        Desc.Type = Type;
        Desc.NumDescriptors = NumDescriptors;
        Desc.Flags = (bShaderVisible ? D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE : 0);
       
        HRESULT hr = pDevice->CreateDescriptorHeap(&Desc, 
                               __uuidof(ID3D12DescriptorHeap), 
                               (void**)&pDH);
        if (FAILED(hr)) return hr;

        hCPUHeapStart = pDH->GetCPUDescriptorHandleForHeapStart();
        hGPUHeapStart = pDH->GetGPUDescriptorHandleForHeapStart();

        HandleIncrementSize = pDevice->GetDescriptorHandleIncrementSize(Desc.Type);
        return hr;
    }
    operator ID3D12DescriptorHeap*() { return pDH; }

    D3D12_CPU_DESCRIPTOR_HANDLE hCPU(UINT index)
    {
        return hCPUHeapStart.MakeOffsetted(index,HandleIncrementSize); 
    }
    D3D12_GPU_DESCRIPTOR_HANDLE hGPU(UINT index) 
    {
        assert(Desc.Flags&D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE); 
        return hGPUHeapStart.MakeOffsetted(index,HandleIncrementSize); 
    }
    D3D12_DESCRIPTOR_HEAP_DESC Desc;
    CComPtr<ID3D12DescriptorHeap> pDH;
    D3D12_CPU_DESCRIPTOR_HANDLE hCPUHeapStart;
    D3D12_GPU_DESCRIPTOR_HANDLE hGPUHeapStart;
    UINT HandleIncrementSize;
};
```

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符堆](descriptor-heaps.md)
</dt> </dl>

 

 




