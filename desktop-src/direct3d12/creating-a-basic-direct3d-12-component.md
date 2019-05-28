---
title: 创建基本的 Direct3D 12 组件
description: 本主题介绍调用流，以创建基本的 Direct3D 12 组件。
ms.assetid: A0FB108B-15C1-42AD-9277-D5CB63FA8329
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 43590b5d496e1f2ef00cfb211a85930dad36fd1b
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224281"
---
# <a name="creating-a-basic-direct3d-12-component"></a>创建基本的 Direct3D 12 组件

本主题介绍调用流，以创建基本的 Direct3D 12 组件。

-   [一个简单的应用的代码流](#code-flow-for-a-simple-app)
    -   [初始化](#initialize)
    -   [Update](#update)
    -   [呈现器](#render)
    -   [销毁](#destroy)
-   [一个简单的应用的代码示例](#code-example-for-a-simple-app)
    -   [类 D3D12HelloTriangle](#class-d3d12hellotriangle)
    -   [OnInit()](#oninit)
    -   [LoadPipeline()](#loadpipeline)
    -   [LoadAssets()](#loadassets)
    -   [OnUpdate()](#onupdate)
    -   [OnRender()](#onrender)
    -   [PopulateCommandList()](#populatecommandlist)
    -   [WaitForPreviousFrame()](#waitforpreviousframe)
    -   [OnDestroy()](#ondestroy)
-   [相关的主题](#related-topics)

## <a name="code-flow-for-a-simple-app"></a>一个简单的应用的代码流

一个非常标准的图形的过程为 D3D 12 程序的最外层循环：

> [!TIP]
> Direct3D 12 的新功能后跟。

 

-   [初始化](#initialize)
-   **重复**
    -   [Update](#update)
    -   [呈现器](#render)
-   [销毁](#destroy)

### <a name="initialize"></a>Initialize

初始化涉及到首次设置全局变量和类，并将管道和资产，必须准备初始化函数。

-   初始化管道。
    -   启用调试层。
    -   创建设备。
    -   创建命令队列。
    -   创建交换链。
    -   创建呈现目标视图 (RTV) 描述符堆。
        > [!Note]  
        > 一个[描述符堆](descriptor-heaps.md)可视为数组[描述符](descriptors.md)。 其中每个描述符完全描述一个对象到 GPU。

         

    -   创建框架资源 （每个帧的渲染目标视图）。
    -   创建命令分配器。
        > [!Note]  
        > 命令分配器管理的基础存储[命令列表和捆绑包](recording-command-lists-and-bundles.md)。

         
-   初始化资产。
    -   创建空的根签名。
        > [!Note]  
        > 一种图形[根签名](root-signatures.md)定义哪些资源绑定到图形管道。

         

    -   编译着色器。
    -   创建顶点输入的布局。
    -   创建[管道状态对象](managing-graphics-pipeline-state-in-direct3d-12.md)说明，然后创建该对象。
        > [!Note]  
        > 管道状态对象维护的所有当前设置的着色器状态，以及某些固定函数状态对象 (如*输入装配器*， *tesselator*，*光栅器*并*输出合并器*)。

         

    -   创建命令列表。
    -   关闭命令列表。
    -   创建并加载顶点缓冲区。
    -   创建顶点缓冲区视图。
    -   筑起一道围墙。
        > [!Note]  
        > 一种保护是用于[同步的 CPU 和 GPU](user-mode-heap-synchronization.md)。

         

    -   创建一个事件句柄。
    -   等待 GPU 来完成。
        > [!Note]  
        > 检查 fence ！

         

请参阅[类 D3D12HelloTriangle](#class-d3d12hellotriangle)， [OnInit](#oninit)， [LoadPipeline](#loadpipeline)并[LoadAssets](#loadassets)。

### <a name="update"></a>更新

更新所有内容应该自最后一帧以来的更改。

-   修改常量、 顶点、 索引缓冲区和 else，根据需要的所有内容。

请参阅[OnUpdate](#onupdate)。

### <a name="render"></a>呈现器

绘制新世界。

-   填充命令列表。
    -   重置命令列表分配器。
        > [!Note]  
        > 重复使用与命令分配器相关联的内存。

         

    -   重置命令列表。
    -   设置图形根签名。
        > [!Note]  
        > 设置图形根签名用于当前命令列表。

         

    -   设置视区和剪刀矩形。
    -   设置资源屏障，指示呈现器目标作为后台缓冲区。
        > [!Note]  
        > 资源障碍，用于管理资源转换。

         

    -   记录命令到命令列表。
    -   指示后台缓冲区将用于显示命令列表执行后。
        > [!Note]  
        > 用于设置资源屏障的另一个调用。

         

    -   关闭到更多记录的命令列表。
-   执行命令列表。
-   提供框架。
-   等待 GPU 来完成。
    > [!Note]  
    > 请更新并检查 fence。

     

请参阅[OnRender](#onrender)。

### <a name="destroy"></a>销毁

完全关闭该应用程序。

-   等待 GPU 来完成。
    > [!Note]  
    > Fence 的最后一个检查。

     

-   关闭事件句柄。

请参阅[OnDestroy](#ondestroy)。

## <a name="code-example-for-a-simple-app"></a>一个简单的应用的代码示例

以下扩展所需的基本示例代码对上述的代码流。

### <a name="class-d3d12hellotriangle"></a>类 D3D12HelloTriangle

首先在标头文件，其中包括视区、 剪刀矩形和顶点缓冲区使用结构中定义类：

-   [**D3D12\_VIEWPORT**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_viewport)
-   [**D3D12\_RECT**](d3d12-rect.md)
-   [**D3D12\_VERTEX\_BUFFER\_VIEW**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_vertex_buffer_view)


```C++
#include "DXSample.h"

using namespace DirectX;
using namespace Microsoft::WRL;

class D3D12HelloTriangle : public DXSample
{
public:
    D3D12HelloTriangle(UINT width, UINT height, std::wstring name);

    virtual void OnInit();
    virtual void OnUpdate();
    virtual void OnRender();
    virtual void OnDestroy();

private:
    static const UINT FrameCount = 2;

    struct Vertex
    {
        XMFLOAT3 position;
        XMFLOAT4 color;
    };

    // Pipeline objects.
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

    // App resources.
    ComPtr<ID3D12Resource> m_vertexBuffer;
    D3D12_VERTEX_BUFFER_VIEW m_vertexBufferView;

    // Synchronization objects.
    UINT m_frameIndex;
    HANDLE m_fenceEvent;
    ComPtr<ID3D12Fence> m_fence;
    UINT64 m_fenceValue;

    void LoadPipeline();
    void LoadAssets();
    void PopulateCommandList();
    void WaitForPreviousFrame();
};
```



### <a name="oninit"></a>OnInit()

在项目主源代码文件中，启动初始化对象：


```C++
void D3D12HelloTriangle::OnInit()
{
    LoadPipeline();
    LoadAssets();
}
```



### <a name="loadpipeline"></a>LoadPipeline()

以下代码创建图形管道的基础知识。 创建设备和交换链的过程是非常类似于 Direct3D 11。

-   启用调试层，通过调用：<dl>

[**D3D12GetDebugInterface**](/windows/desktop/api/D3D12/nf-d3d12-d3d12getdebuginterface)  
    [**ID3D12Debug::EnableDebugLayer**](/windows/desktop/api/d3d12sdklayers/nf-d3d12sdklayers-id3d12debug-enabledebuglayer)  
    </dl>
-   创建设备：<dl>

[**CreateDXGIFactory1**](https://msdn.microsoft.com/library/windows/desktop/ff471318)  
    [**D3D12CreateDevice**](/windows/desktop/api/D3D12/nf-d3d12-d3d12createdevice)  
    </dl>
-   填写命令队列说明，然后创建命令队列：<dl>

[**D3D12\_命令\_队列\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_queue_desc)  
    [**ID3D12Device::CreateCommandQueue**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandqueue)  
    </dl>
-   填写 swapchain 说明，然后创建交换链： <dl>

[**DXGI\_SWAP\_CHAIN\_DESC**](https://msdn.microsoft.com/library/windows/desktop/bb173075)  
    [**IDXGIFactory::CreateSwapChain**](https://msdn.microsoft.com/library/windows/desktop/bb174537)  
    [**IDXGISwapChain3::GetCurrentBackBufferIndex**](https://msdn.microsoft.com/library/windows/desktop/dn903675)  
    </dl>
-   填写堆说明。 然后创建描述符堆： <dl>

[**D3D12\_DESCRIPTOR\_HEAP\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_descriptor_heap_desc)  
    [**ID3D12Device::CreateDescriptorHeap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createdescriptorheap)  
    [**ID3D12Device::GetDescriptorHandleIncrementSize**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getdescriptorhandleincrementsize)  
    </dl>
-   创建呈现目标视图： <dl>

[**CD3DX12\_CPU\_DESCRIPTOR\_HANDLE**](cd3dx12-cpu-descriptor-handle.md)  
    [**GetCPUDescriptorHandleForHeapStart**](/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart)  
    [**IDXGISwapChain::GetBuffer**](https://msdn.microsoft.com/library/windows/desktop/bb174570)  
    [**ID3D12Device::CreateRenderTargetView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrendertargetview)  
    </dl>
-   创建命令分配器：[**ID3D12Device::CreateCommandAllocator**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandallocator)。

在后续步骤中，命令列表是从命令分配器获取并提交到的命令队列。

加载呈现管道依赖项 （请注意，创建软件 WARP 设备是完全可选的）。


```C++
void D3D12HelloTriangle::LoadPipeline()
{
#if defined(_DEBUG)
    // Enable the D3D12 debug layer.
    {
        
        ComPtr<ID3D12Debug> debugController;
        if (SUCCEEDED(D3D12GetDebugInterface(IID_PPV_ARGS(&debugController))))
        {
            debugController->EnableDebugLayer();
        }
    }
#endif

    ComPtr<IDXGIFactory4> factory;
    ThrowIfFailed(CreateDXGIFactory1(IID_PPV_ARGS(&factory)));

    if (m_useWarpDevice)
    {
        ComPtr<IDXGIAdapter> warpAdapter;
        ThrowIfFailed(factory->EnumWarpAdapter(IID_PPV_ARGS(&warpAdapter)));

        ThrowIfFailed(D3D12CreateDevice(
            warpAdapter.Get(),
            D3D_FEATURE_LEVEL_11_0,
            IID_PPV_ARGS(&m_device)
            ));
    }
    else
    {
        ComPtr<IDXGIAdapter1> hardwareAdapter;
        GetHardwareAdapter(factory.Get(), &hardwareAdapter);

        ThrowIfFailed(D3D12CreateDevice(
            hardwareAdapter.Get(),
            D3D_FEATURE_LEVEL_11_0,
            IID_PPV_ARGS(&m_device)
            ));
    }

    // Describe and create the command queue.
    D3D12_COMMAND_QUEUE_DESC queueDesc = {};
    queueDesc.Flags = D3D12_COMMAND_QUEUE_FLAG_NONE;
    queueDesc.Type = D3D12_COMMAND_LIST_TYPE_DIRECT;

    ThrowIfFailed(m_device->CreateCommandQueue(&queueDesc, IID_PPV_ARGS(&m_commandQueue)));

    // Describe and create the swap chain.
    DXGI_SWAP_CHAIN_DESC swapChainDesc = {};
    swapChainDesc.BufferCount = FrameCount;
    swapChainDesc.BufferDesc.Width = m_width;
    swapChainDesc.BufferDesc.Height = m_height;
    swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
    swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
    swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
    swapChainDesc.OutputWindow = Win32Application::GetHwnd();
    swapChainDesc.SampleDesc.Count = 1;
    swapChainDesc.Windowed = TRUE;

    ComPtr<IDXGISwapChain> swapChain;
    ThrowIfFailed(factory->CreateSwapChain(
        m_commandQueue.Get(),        // Swap chain needs the queue so that it can force a flush on it.
        &swapChainDesc,
        &swapChain
        ));

    ThrowIfFailed(swapChain.As(&m_swapChain));

    // This sample does not support fullscreen transitions.
    ThrowIfFailed(factory->MakeWindowAssociation(Win32Application::GetHwnd(), DXGI_MWA_NO_ALT_ENTER));

    m_frameIndex = m_swapChain->GetCurrentBackBufferIndex();

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
    }

    ThrowIfFailed(m_device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&m_commandAllocator)));
}
```



### <a name="loadassets"></a>LoadAssets()

加载和准备资产是一个漫长的过程。 尽管一些不熟悉 D3D 12，许多这些阶段都类似于 D3D 11。

在 Direct3D 12 中，所需的管道状态附加到通过命令列表[管道状态对象](managing-graphics-pipeline-state-in-direct3d-12.md)(PSO)。 此示例演示如何创建 pso 的步骤。 可以将 PSO 存储为成员变量并将其作为根据需要多次重复使用。

描述符堆定义视图，以及如何访问资源 （例如，呈现目标视图）。

命令列表的分配器和 PSO 后，可以创建实际的命令列表中，将在稍后执行。

相继调用以下 Api 和进程。

-   创建空的根签名，使用可用的帮助器结构： <dl>

[**CD3DX12\_ROOT\_SIGNATURE\_DESC**](cd3dx12-root-signature-desc.md)  
    [**D3D12SerializeRootSignature**](/windows/desktop/api/D3D12/nf-d3d12-d3d12serializerootsignature)  
    [**ID3D12Device::CreateRootSignature**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrootsignature)  
    </dl>
-   加载和编译着色器：[**D3DCompileFromFile**](https://msdn.microsoft.com/library/windows/desktop/hh446872)。
-   创建顶点输入的布局：[**D3D12\_输入\_元素\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_input_element_desc)。
-   然后，管道状态说明，使用可用的帮助器结构，请填写创建图形管道状态： <dl>

[**D3D12\_GRAPHICS\_PIPELINE\_STATE\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_graphics_pipeline_state_desc)  
    [**CD3DX12\_RASTERIZER\_DESC**](cd3dx12-rasterizer-desc.md)  
    [**CD3DX12\_BLEND\_DESC**](cd3dx12-blend-desc.md)  
    [**ID3D12Device::CreateGraphicsPipelineState**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-creategraphicspipelinestate)  
    </dl>
-   创建，然后关闭命令列表： <dl>

[**ID3D12Device::CreateCommandList**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandlist)  
    [**ID3D12GraphicsCommandList::Close**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)  
    </dl>
-   创建顶点缓冲区：[**ID3D12Device::CreateCommittedResource**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommittedresource)。
-   将顶点数据复制到顶点缓冲区：<dl>

[**ID3D12Resource::Map**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)  
    [**ID3D12Resource::Unmap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap)  
    </dl>
-   初始化顶点缓冲区视图：[**GetGPUVirtualAddress**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-getgpuvirtualaddress)。
-   创建和初始化 fence:[**ID3D12Device::CreateFence**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createfence)。
-   创建具有帧同步使用一个事件句柄。
-   等待 GPU 来完成。


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



### <a name="onupdate"></a>OnUpdate()

对于简单的示例，没有任何更新。


```C++
void D3D12HelloTriangle::OnUpdate()

{
}
```



### <a name="onrender"></a>OnRender()

在设置完毕，成员变量**m\_commandList**用于记录和执行所有这些设置命令。 现在可以重复使用主要呈现循环中的该成员。

呈现涉及的调用来填充命令列表，则可以执行的命令列表并显示交换链中的下一个缓冲区：

-   填充命令列表。
-   执行命令列表：[**ID3D12CommandQueue::ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)。
-   [**IDXGISwapChain1::Present1** ](https://msdn.microsoft.com/library/windows/desktop/hh446797)帧。
-   若要完成在 GPU 上等待。


```C++
void D3D12HelloTriangle::OnRender()
{
    // Record all the commands we need to render the scene into the command list.
    PopulateCommandList();

    // Execute the command list.
    ID3D12CommandList* ppCommandLists[] = { m_commandList.Get() };
    m_commandQueue->ExecuteCommandLists(_countof(ppCommandLists), ppCommandLists);

    // Present the frame.
    ThrowIfFailed(m_swapChain->Present(1, 0));

    WaitForPreviousFrame();
}
```



### <a name="populatecommandlist"></a>PopulateCommandList()

您可以重复使用它们之前，则必须重置命令列表分配器和命令列表本身。 在更高级的方案中可能会有用重置分配器每隔几个帧。 内存是与不能在执行命令列表后立即释放的分配器相关联。 此示例演示如何重置后每一帧的分配器。

现在，重复使用当前帧的命令列表。 重新附加到命令列表中 （这必须重置命令列表，则只要列表执行命令前），视区指示资源会用作呈现器目标，记录命令和则指示将用于呈现器目标当前命令列表完成时正在执行。

填充命令列出以下方法的调用并反过来处理：

-   重置命令分配器和命令列表： <dl>

[**ID3D12CommandAllocator::Reset**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandallocator-reset)  
    [**ID3D12GraphicsCommandList::Reset**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-reset)  
    </dl>
-   设置根签名、 视区和剪刀矩形： <dl>

[**ID3D12GraphicsCommandList::SetGraphicsRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootsignature)  
    [**ID3D12GraphicsCommandList::RSSetViewports**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetviewports)  
    [**ID3D12GraphicsCommandList::RSSetScissorRects**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetscissorrects)  
    </dl>
-   指示要使用的呈现器目标作为后台缓冲区： <dl>

[**ID3D12GraphicsCommandList::ResourceBarrier**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)  
    [**ID3D12DescriptorHeap::GetCPUDescriptorHandleForHeapStart**](/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart)  
    [**ID3D12GraphicsCommandList::OMSetRenderTargets**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetrendertargets)  
    </dl>
-   记录命令：<dl>

[**ID3D12GraphicsCommandList::ClearRenderTargetView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearrendertargetview)  
    [**ID3D12GraphicsCommandList::IASetPrimitiveTopology**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetprimitivetopology)  
    [**ID3D12GraphicsCommandList::IASetVertexBuffers**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetvertexbuffers)  
    [**ID3D12GraphicsCommandList::DrawInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced)  
    </dl>
-   指示后台缓冲区现在将用于显示：[**ID3D12GraphicsCommandList::ResourceBarrier**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier).
-   关闭命令列表：[**ID3D12GraphicsCommandList::Close**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)。


```C++
void D3D12HelloTriangle::PopulateCommandList()
{
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
}
```



### <a name="waitforpreviousframe"></a>WaitForPreviousFrame()

下面的代码演示界定过度简化的的用法。

> [!Note]  
> 等待完成的帧是对于大多数应用程序做效率太低。

 

以下 Api 和进程的调用顺序：

-   [**ID3D12CommandQueue::Signal**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandqueue-signal)
-   [**ID3D12Fence::GetCompletedValue**](/windows/desktop/api/D3D12/nf-d3d12-id3d12fence-getcompletedvalue)
-   [**ID3D12Fence::SetEventOnCompletion**](/windows/desktop/api/D3D12/nf-d3d12-id3d12fence-seteventoncompletion)
-   等待事件。
-   更新帧索引：[**IDXGISwapChain3::GetCurrentBackBufferIndex**](https://msdn.microsoft.com/library/windows/desktop/dn903675)。


```C++
void D3D12HelloTriangle::WaitForPreviousFrame()
{
    // WAITING FOR THE FRAME TO COMPLETE BEFORE CONTINUING IS NOT BEST PRACTICE.
    // This is code implemented as such for simplicity. More advanced samples 
    // illustrate how to use fences for efficient resource usage.

    // Signal and increment the fence value.
    const UINT64 fence = m_fenceValue;
    ThrowIfFailed(m_commandQueue->Signal(m_fence.Get(), fence));
    m_fenceValue++;

    // Wait until the previous frame is finished.
    if (m_fence->GetCompletedValue() < fence)
    {
        ThrowIfFailed(m_fence->SetEventOnCompletion(fence, m_fenceEvent));
        WaitForSingleObject(m_fenceEvent, INFINITE);
    }

    m_frameIndex = m_swapChain->GetCurrentBackBufferIndex();
}
```



### <a name="ondestroy"></a>OnDestroy()

完全关闭该应用程序。

-   等待 GPU 来完成。
-   关闭事件。


```C++
void D3D12HelloTriangle::OnDestroy()
{

    // Wait for the GPU to be done with all resources.
    WaitForPreviousFrame();

    CloseHandle(m_fenceEvent);
}
```



## <a name="related-topics"></a>相关主题

<dl> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> <dt>

[有用的示例](working-samples.md)
</dt> </dl>

 

 




