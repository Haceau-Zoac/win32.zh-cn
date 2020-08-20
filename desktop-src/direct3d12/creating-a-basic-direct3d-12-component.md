---
title: 创建基本的 Direct3D 12 组件
description: 本主题介绍用于创建基本 Direct3D 12 组件的调用流。
ms.assetid: A0FB108B-15C1-42AD-9277-D5CB63FA8329
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 8c00656cf5630b8743240268a91a51aa2e466c97
ms.sourcegitcommit: 592c9bbd22ba69802dc353bcb5eb30699f9e9403
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2020
ms.locfileid: "88644232"
---
# <a name="creating-a-basic-direct3d-12-component"></a>创建基本的 Direct3D 12 组件

本主题介绍用于创建基本 Direct3D 12 组件的调用流。

-   [简单应用的代码流](#code-flow-for-a-simple-app)
    -   [初始化](#initialize)
    -   [更新](#update)
    -   [Render](#render)
    -   [破坏](#destroy)
-   [简单应用的代码示例](#code-example-for-a-simple-app)
    -   [类 D3D12HelloTriangle](#class-d3d12hellotriangle)
    -   [OnInit()](#oninit)
    -   [LoadPipeline ( # B1 ](#loadpipeline)
    -   [LoadAssets()](#loadassets)
    -   [OnUpdate ( # B1 ](#onupdate)
    -   [OnRender()](#onrender)
    -   [PopulateCommandList()](#populatecommandlist)
    -   [WaitForPreviousFrame()](#waitforpreviousframe)
    -   [OnDestroy ( # B1 ](#ondestroy)
-   [相关主题](#related-topics)

## <a name="code-flow-for-a-simple-app"></a>简单应用的代码流

D3D 12 程序的最外层循环遵循一个极其标准的图形过程：

> [!TIP]
> Direct3D 12 中的新功能附有注释。

 

-   [初始化](#initialize)
-   **以上**
    -   [更新](#update)
    -   [Render](#render)
-   [破坏](#destroy)

### <a name="initialize"></a>初始化

初始化涉及到首次设置全局变量和类，initialize 函数必须准备管道和资产。

-   初始化管道。
    -   启用调试层。
    -   创建设备。
    -   创建命令队列。
    -   创建交换链。
    -   创建渲染器目标视图 (RTV) 描述符堆。
        > [!Note]  
        > 可将[描述符堆](descriptor-heaps.md)视为[描述符](descriptors.md)的数组。 其中的每个描述符完全描述 GPU 的对象。

         

    -   创建帧资源（每个帧的渲染器目标视图）。
    -   创建命令分配器。
        > [!Note]  
        > 命令分配器管理[命令列表和捆绑](recording-command-lists-and-bundles.md)的基础存储。

         
-   初始化资产。
    -   创建空的根签名。
        > [!Note]  
        > 图形[根签名](root-signatures.md)定义哪些资源要绑定到图形管道。

         

    -   编译着色器。
    -   创建顶点输入布局。
    -   创建[管道状态对象](managing-graphics-pipeline-state-in-direct3d-12.md)说明，然后创建对象。
        > [!Note]  
        > 管道状态对象保留所有当前设置的着色器以及某些固定函数状态对象（例如输入汇编器、细分器、光栅器和输出合并器）的状态。********

         

    -   创建命令列表。
    -   关闭命令列表。
    -   创建并加载顶点缓冲区。
    -   创建顶点缓冲区视图。
    -   创建围栏。
        > [!Note]  
        > 使用 GPU 将 CPU 与 GPU 同步 (参阅 [多引擎同步](./user-mode-heap-synchronization.md)) 。

         

    -   创建事件处理程序。
    -   等待 GPU 完成。
        > [!Note]  
        > 检查围栏！

         

请参阅[类 D3D12HelloTriangle](#class-d3d12hellotriangle)、[OnInit](#oninit)、[LoadPipeline](#loadpipeline) 和 [LoadAssets](#loadassets)。

### <a name="update"></a>更新

更新自完成上一帧以来应该更改的所有内容。

-   根据需要修改常量、顶点、索引缓冲区和其他任何内容。

请参阅 [OnUpdate](#onupdate)。

### <a name="render"></a>呈现

绘制新世界。

-   填充命令列表。
    -   重置命令列表分配器。
        > [!Note]  
        > 重复使用与命令分配器关联的内存。

         

    -   重置命令列表。
    -   设置图形根签名。
        > [!Note]  
        > 设置用于当前命令列表的图形根签名。

         

    -   设置视区和裁切矩形。
    -   设置资源屏障，指示将反向缓冲区用作渲染器目标。
        > [!Note]  
        > 资源屏障用于管理资源转换。

         

    -   将命令记录到命令列表中。
    -   指示在执行命令列表后将使用反向缓冲区来呈现信息。
        > [!Note]  
        > 用于设置资源屏障的另一个调用。

         

    -   关闭命令列表以进一步记录。
-   执行命令列表。
-   呈现帧。
-   等待 GPU 完成。
    > [!Note]  
    > 持续更新并检查围栏。

     

请参阅 [OnRender](#onrender)。

### <a name="destroy"></a>Destroy

彻底关闭应用。

-   等待 GPU 完成。
    > [!Note]  
    > 对围栏进行最终检查。

     

-   关闭事件句柄。

请参阅 [OnDestroy](#ondestroy)。

## <a name="code-example-for-a-simple-app"></a>简单应用的代码示例

以下内容扩展了上述代码流，包含基本示例所需的代码。

### <a name="class-d3d12hellotriangle"></a>类 D3D12HelloTriangle

首先使用以下结构在标头文件中定义该类，包括视区、剪切矩形和顶点缓冲区：

-   [**D3D12 \_ 视区**](/windows/win32/api/d3d12/ns-d3d12-d3d12_viewport)
-   [**D3D12 \_ RECT**](d3d12-rect.md)
-   [**D3D12 \_ 顶点 \_ 缓冲区 \_ 视图**](/windows/win32/api/d3d12/ns-d3d12-d3d12_vertex_buffer_view)


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

在项目的主源代码文件中，开始初始化对象：


```C++
void D3D12HelloTriangle::OnInit()
{
    LoadPipeline();
    LoadAssets();
}
```



### <a name="loadpipeline"></a>LoadPipeline()

以下代码创建图形管道的基本组件。 创建设备和交换链的过程与 Direct3D 11 非常类似。

-   调用以下方法启用调试层：<dl>

[**D3D12GetDebugInterface**](/windows/win32/api/d3d12/nf-d3d12-d3d12getdebuginterface)  
    [**ID3D12Debug::EnableDebugLayer**](/windows/win32/api/d3d12sdklayers/nf-d3d12sdklayers-id3d12debug-enabledebuglayer)  
    </dl>
-   创建设备：<dl>

[**CreateDXGIFactory1**](/windows/win32/api/dxgi/nf-dxgi-createdxgifactory1)  
    [**D3D12CreateDevice**](/windows/win32/api/d3d12/nf-d3d12-d3d12createdevice)  
    </dl>
-   填写命令队列说明，然后创建命令队列：<dl>

[**D3D12 \_ 命令 \_ 队列 \_ DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_command_queue_desc)  
    [**ID3D12Device::CreateCommandQueue**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcommandqueue)  
    </dl>
-   填写交换链说明，然后创建交换链： <dl>

[**DXGI\_SWAP\_CHAIN\_DESC**](/windows/win32/api/dxgi/ns-dxgi-dxgi_swap_chain_desc)  
    [**IDXGIFactory::CreateSwapChain**](/windows/win32/api/dxgi/nf-dxgi-idxgifactory-createswapchain)  
    [**IDXGISwapChain3::GetCurrentBackBufferIndex**](/windows/win32/api/dxgi1_4/nf-dxgi1_4-idxgiswapchain3-getcurrentbackbufferindex)  
    </dl>
-   填写堆说明。 然后创建描述符堆： <dl>

[**D3D12 \_ 描述符 \_ 堆 \_ DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_descriptor_heap_desc)  
    [**ID3D12Device::CreateDescriptorHeap**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createdescriptorheap)  
    [**ID3D12Device::GetDescriptorHandleIncrementSize**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-getdescriptorhandleincrementsize)  
    </dl>
-   创建渲染器目标视图： <dl>

[**CD3DX12 \_ CPU \_ 描述符 \_ 句柄**](cd3dx12-cpu-descriptor-handle.md)  
    [**GetCPUDescriptorHandleForHeapStart**](/windows/win32/api/d3d12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart)  
    [**IDXGISwapChain::GetBuffer**](/windows/win32/api/dxgi/nf-dxgi-idxgiswapchain-getbuffer)  
    [**ID3D12Device::CreateRenderTargetView**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createrendertargetview)  
    </dl>
-   创建命令分配器： [**ID3D12Device：： CreateCommandAllocator**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcommandallocator)。

在后续步骤中，将从命令分配器获取命令列表并将其提交到命令队列。

加载渲染管道依赖项（请注意，创建软件 WARP 设备是完全可选的操作）。


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

加载和准备资产是一个漫长的过程。 其中的许多阶段类似于 D3D 11，不过有些阶段是 D3D 12 中的新阶段。

在 Direct3D 12 中，所需管道状态通过[管道状态对象](managing-graphics-pipeline-state-in-direct3d-12.md) (PSO) 附加到命令列表。 此示例演示如何创建 PSO。 可将 PSO 存储为成员变量，并重复使用任意次。

描述符堆定义视图，以及如何访问资源（例如渲染器目标视图）。

使用命令列表分配器和 PSO 可以创建稍后要执行的实际命令列表。

相继调用以下 API 和进程。

-   使用可用的帮助器结构创建空的根签名： <dl>

[**CD3DX12 \_ 根 \_ 签名 \_ DESC**](cd3dx12-root-signature-desc.md)  
    [**D3D12SerializeRootSignature**](/windows/win32/api/d3d12/nf-d3d12-d3d12serializerootsignature)  
    [**ID3D12Device::CreateRootSignature**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createrootsignature)  
    </dl>
-   加载和编译着色器： [**D3DCompileFromFile**](/windows/win32/api/d3dcompiler/nf-d3dcompiler-d3dcompilefromfile)。
-   创建顶点输入布局： [**D3D12 \_ input \_ 元素 \_ DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_input_element_desc)。
-   填写管道状态说明，然后使用可用的帮助器结构创建图形管道状态： <dl>

[**D3D12 \_ 图形 \_ 管道 \_ 状态 \_ DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_graphics_pipeline_state_desc)  
    [**CD3DX12 \_ 光栅化 \_ DESC**](cd3dx12-rasterizer-desc.md)  
    [**CD3DX12 \_ BLEND \_ DESC**](cd3dx12-blend-desc.md)  
    [**ID3D12Device::CreateGraphicsPipelineState**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-creategraphicspipelinestate)  
    </dl>
-   创建然后关闭命令列表： <dl>

[**ID3D12Device::CreateCommandList**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcommandlist)  
    [**ID3D12GraphicsCommandList：： Close**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)  
    </dl>
-   创建顶点缓冲区： [**ID3D12Device：： CreateCommittedResource**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcommittedresource)。
-   将顶点数据复制到顶点缓冲区：<dl>

[**ID3D12Resource::Map**](/windows/win32/api/d3d12/nf-d3d12-id3d12resource-map)  
    [**ID3D12Resource::Unmap**](/windows/win32/api/d3d12/nf-d3d12-id3d12resource-unmap)  
    </dl>
-   初始化顶点缓冲区视图： [**GetGPUVirtualAddress**](/windows/win32/api/d3d12/nf-d3d12-id3d12resource-getgpuvirtualaddress)。
-   创建和初始化防护： [**ID3D12Device：： CreateFence**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createfence)。
-   创建事件句柄用于帧同步。
-   等待 GPU 完成。


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

在简单的示例中无需更新任何内容。


```C++
void D3D12HelloTriangle::OnUpdate()

{
}
```



### <a name="onrender"></a>OnRender()

在设置期间，已使用成员变量 **m\_commandList** 来记录和执行所有设置的命令。 现在可以在主渲染循环中重复使用该成员。

渲染涉及到发出一个调用来填充命令列表，然后可以执行该命令列表，并呈现交换链中的下一个缓冲区：

-   填充命令列表。
-   执行命令列表： [**ID3D12CommandQueue：： ExecuteCommandLists**](/windows/win32/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)。
-   对帧执行 [**IDXGISwapChain1::Present1**](/windows/win32/api/dxgi1_2/nf-dxgi1_2-idxgiswapchain1-present1)。
-   等待 GPU 完成。


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

必须先重置命令列表分配器和命令列表本身，然后才能重复使用它们。 在更高级的方案中，可能有效的做法是每隔几个帧就重置分配器。 内存与执行命令列表后无法立即释放的分配器相关联。 此示例演示如何在完成每一帧后重置分配器。

现在，对当前帧重复使用命令列表。 将视区重新附加到命令列表（必须在每次重置命令列表之后、执行命令之前完成），指示资源将用作渲染器目标，记录命令，然后指示在执行完命令列表后，将使用渲染器目标来呈现信息。

填充命令列表的过程会依次调用以下方法和进程：

-   重置命令分配器和命令列表： <dl>

[**ID3D12CommandAllocator::Reset**](/windows/win32/api/d3d12/nf-d3d12-id3d12commandallocator-reset)  
    [**ID3D12GraphicsCommandList::Reset**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-reset)  
    </dl>
-   设置根签名、视区和裁切矩形： <dl>

[**ID3D12GraphicsCommandList::SetGraphicsRootSignature**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootsignature)  
    [**ID3D12GraphicsCommandList::RSSetViewports**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetviewports)  
    [**ID3D12GraphicsCommandList::RSSetScissorRects**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetscissorrects)  
    </dl>
-   指示要将反向缓冲区用作渲染器目标： <dl>

[**ID3D12GraphicsCommandList::ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)  
    [**ID3D12DescriptorHeap::GetCPUDescriptorHandleForHeapStart**](/windows/win32/api/d3d12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart)  
    [**ID3D12GraphicsCommandList::OMSetRenderTargets**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetrendertargets)  
    </dl>
-   记录命令：<dl>

[**ID3D12GraphicsCommandList::ClearRenderTargetView**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearrendertargetview)  
    [**ID3D12GraphicsCommandList::IASetPrimitiveTopology**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetprimitivetopology)  
    [**ID3D12GraphicsCommandList::IASetVertexBuffers**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetvertexbuffers)  
    [**ID3D12GraphicsCommandList::DrawInstanced**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced)  
    </dl>
-   指示现在将使用后台缓冲区显示： [**ID3D12GraphicsCommandList：： ResourceBarrier**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)。
-   关闭命令列表： [**ID3D12GraphicsCommandList：： Close**](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)。


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

以下代码演示围栏的一种过度简化的用法。

> [!Note]  
> 对于大多数应用而言，等待帧完成的做法过于低效。

 

依次调用以下 API 和进程：

-   [**ID3D12CommandQueue::Signal**](/windows/win32/api/d3d12/nf-d3d12-id3d12commandqueue-signal)
-   [**ID3D12Fence::GetCompletedValue**](/windows/win32/api/d3d12/nf-d3d12-id3d12fence-getcompletedvalue)
-   [**ID3D12Fence::SetEventOnCompletion**](/windows/win32/api/d3d12/nf-d3d12-id3d12fence-seteventoncompletion)
-   等待事件。
-   更新帧索引： [**IDXGISwapChain3：： GetCurrentBackBufferIndex**](/windows/win32/api/dxgi1_4/nf-dxgi1_4-idxgiswapchain3-getcurrentbackbufferindex)。


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

彻底关闭应用。

-   等待 GPU 完成。
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

 

 