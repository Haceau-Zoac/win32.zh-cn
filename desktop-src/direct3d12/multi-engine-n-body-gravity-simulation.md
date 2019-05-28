---
title: 多引擎 n 正文重力模拟
description: D3D12nBodyGravity 示例演示如何执行操作以异步方式计算工作。
ms.assetid: B20C5575-0616-43F7-9AC9-5F802E5597B5
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 890abfee1f18b68850877aa8f49f3817e9cae7c0
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223855"
---
# <a name="multi-engine-n-body-gravity-simulation"></a>多引擎 n 正文重力模拟

**D3D12nBodyGravity**示例演示如何执行操作以异步方式计算工作。 该示例会启动的线程每个与计算命令队列数和计划计算执行 n 正文重力模拟在 GPU 上的工作。 每个线程对两个缓冲区的位置和速度数据进行操作。 每次迭代时，计算着色器从一个缓冲区中读取的当前的位置和速度数据，并写入另一个缓冲区的下一个迭代。 完成迭代后，计算着色器交换的缓冲区是用于读取位置/速度数据 SRV，哪个是用于通过更改每个缓冲区上的资源状态写入位置/速度更新 UAV。

-   [创建根签名](#create-the-root-signatures)
-   [创建 SRV 和 UAV 缓冲区](#create-the-srv-and-uav-buffers)
-   [创建 CBV 和顶点缓冲区](#create-the-cbv-and-vertex-buffers)
-   [同步呈现和计算线程](#synchronize-the-rendering-and-compute-threads)
-   [运行示例](#run-the-sample)
-   [相关的主题](#related-topics)

## <a name="create-the-root-signatures"></a>创建根签名

我们首先创建图形和计算根签名，在**LoadAssets**方法。 这两个根签名具有根常量缓冲区视图 (CBV) 和着色器资源视图 (SRV) 描述符表。 计算根签名还具有无序的访问视图 (UAV) 描述符表。

``` syntax
 // Create the root signatures.
       {
              CD3DX12_DESCRIPTOR_RANGE ranges[2];
              ranges[0].Init(D3D12_DESCRIPTOR_RANGE_TYPE_SRV, 1, 0);
              ranges[1].Init(D3D12_DESCRIPTOR_RANGE_TYPE_UAV, 1, 0);

              CD3DX12_ROOT_PARAMETER rootParameters[RootParametersCount];
              rootParameters[RootParameterCB].InitAsConstantBufferView(0, 0, D3D12_SHADER_VISIBILITY_ALL);
              rootParameters[RootParameterSRV].InitAsDescriptorTable(1, &ranges[0], D3D12_SHADER_VISIBILITY_VERTEX);
              rootParameters[RootParameterUAV].InitAsDescriptorTable(1, &ranges[1], D3D12_SHADER_VISIBILITY_ALL);

              // The rendering pipeline does not need the UAV parameter.
              CD3DX12_ROOT_SIGNATURE_DESC rootSignatureDesc;
              rootSignatureDesc.Init(_countof(rootParameters) - 1, rootParameters, 0, nullptr, D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT);

              ComPtr<ID3DBlob> signature;
              ComPtr<ID3DBlob> error;
              ThrowIfFailed(D3D12SerializeRootSignature(&rootSignatureDesc, D3D_ROOT_SIGNATURE_VERSION_1, &signature, &error));
              ThrowIfFailed(m_device->CreateRootSignature(0, signature->GetBufferPointer(), signature->GetBufferSize(), IID_PPV_ARGS(&m_rootSignature)));

              // Create compute signature. Must change visibility for the SRV.
              rootParameters[RootParameterSRV].ShaderVisibility = D3D12_SHADER_VISIBILITY_ALL;

              CD3DX12_ROOT_SIGNATURE_DESC computeRootSignatureDesc(_countof(rootParameters), rootParameters, 0, nullptr);
              ThrowIfFailed(D3D12SerializeRootSignature(&computeRootSignatureDesc, D3D_ROOT_SIGNATURE_VERSION_1, &signature, &error));

              ThrowIfFailed(m_device->CreateRootSignature(0, signature->GetBufferPointer(), signature->GetBufferSize(), IID_PPV_ARGS(&m_computeRootSignature)));
       }
```



| 呼叫流                                                             | 参数                                                            |
|-----------------------------------------------------------------------|-----------------------------------------------------------------------|
| [**CD3DX12\_DESCRIPTOR\_RANGE**](cd3dx12-descriptor-range.md)        | [**D3D12\_描述符\_范围\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_descriptor_range_type) |
| [**CD3DX12\_ROOT\_PARAMETER**](cd3dx12-root-parameter.md)            | [**D3D12\_着色器\_可见性**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_shader_visibility)          |
| [**CD3DX12\_ROOT\_SIGNATURE\_DESC**](cd3dx12-root-signature-desc.md) | [**D3D12\_根\_签名\_标志**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_root_signature_flags)   |
| [**ID3DBlob**](https://msdn.microsoft.com/library/windows/desktop/ff728743)                                   |                                                                       |
| [**D3D12SerializeRootSignature**](/windows/desktop/api/D3D12/nf-d3d12-d3d12serializerootsignature)    | [**D3D\_根\_签名\_版本**](/windows/desktop/api/D3D12/ne-d3d12-d3d_root_signature_version)   |
| [**CreateRootSignature**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrootsignature)       |                                                                       |
| [**CD3DX12\_ROOT\_SIGNATURE\_DESC**](cd3dx12-root-signature-desc.md) |                                                                       |
| [**D3D12SerializeRootSignature**](/windows/desktop/api/D3D12/nf-d3d12-d3d12serializerootsignature)    | [**D3D\_根\_签名\_版本**](/windows/desktop/api/D3D12/ne-d3d12-d3d_root_signature_version)   |
| [**CreateRootSignature**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrootsignature)       |                                                                       |



 

## <a name="create-the-srv-and-uav-buffers"></a>创建 SRV 和 UAV 缓冲区

SRV 和 UAV 缓冲区包含的数组位置和速度数据。

``` syntax
 // Position and velocity data for the particles in the system.
       // Two buffers full of Particle data are utilized in this sample.
       // The compute thread alternates writing to each of them.
       // The render thread renders using the buffer that is not currently
       // in use by the compute shader.
       struct Particle
       {
              XMFLOAT4 position;
              XMFLOAT4 velocity;
       };
```



| 呼叫流                       | 参数 |
|---------------------------------|------------|
| [**XMFLOAT4**](https://msdn.microsoft.com/library/windows/desktop/ee419608) |            |



 

## <a name="create-the-cbv-and-vertex-buffers"></a>创建 CBV 和顶点缓冲区

对于图形管道是 CBV**结构**包含几何着色器使用的两个矩阵。 几何着色器在系统中使用的每个粒子的位置，并生成四来表示它使用这些矩阵。

``` syntax
 struct ConstantBufferGS
       {
              XMMATRIX worldViewProjection;
              XMMATRIX inverseView;

              // Constant buffers are 256-byte aligned in GPU memory. Padding is added
              // for convenience when computing the struct's size.
              float padding[32];
       };
```



| 呼叫流                       | 参数 |
|---------------------------------|------------|
| [**XMMATRIX**](https://msdn.microsoft.com/library/windows/desktop/ee419959) |            |



 

因此，使用顶点着色器的顶点缓冲区实际上不包含任何位置的数据。

``` syntax
 // "Vertex" definition for particles. Triangle vertices are generated 
       // by the geometry shader. Color data will be assigned to those 
       // vertices via this struct.
       struct ParticleVertex
       {
              XMFLOAT4 color;
       };
```



| 呼叫流                       | 参数 |
|---------------------------------|------------|
| [**XMFLOAT4**](https://msdn.microsoft.com/library/windows/desktop/ee419608) |            |



 

计算管道是 CBV**结构**包含由计算着色器中的 n 正文重力模拟一些常量。

``` syntax
 struct ConstantBufferCS
       {
              UINT param[4];
              float paramf[4];
       };
```

## <a name="synchronize-the-rendering-and-compute-threads"></a>同步呈现和计算线程

缓冲区都初始化后，将开始呈现和计算工作。 计算线程将 SRV 与 UAV 之间来回的两个位置/速度缓冲区的状态更改它会循环访问上模拟，以及呈现线程必须确保它计划对 SRV 在图形管道上的工作。 界定用于同步对两个缓冲区的访问。

在呈现线程：

``` syntax
// Render the scene.
void D3D12nBodyGravity::OnRender()
{
       // Let the compute thread know that a new frame is being rendered.
       for (int n = 0; n < ThreadCount; n++)
       {
              InterlockedExchange(&m_renderContextFenceValues[n], m_renderContextFenceValue);
       }

       // Compute work must be completed before the frame can render or else the SRV 
       // will be in the wrong state.
       for (UINT n = 0; n < ThreadCount; n++)
       {
              UINT64 threadFenceValue = InterlockedGetValue(&m_threadFenceValues[n]);
              if (m_threadFences[n]->GetCompletedValue() < threadFenceValue)
              {
                     // Instruct the rendering command queue to wait for the current 
                     // compute work to complete.
                     ThrowIfFailed(m_commandQueue->Wait(m_threadFences[n].Get(), threadFenceValue));
              }
       }

       // Record all the commands we need to render the scene into the command list.
       PopulateCommandList();

       // Execute the command list.
       ID3D12CommandList* ppCommandLists[] = { m_commandList.Get() };
       m_commandQueue->ExecuteCommandLists(_countof(ppCommandLists), ppCommandLists);

       // Present the frame.
       ThrowIfFailed(m_swapChain->Present(0, 0));

       MoveToNextFrame();
}
```



| 呼叫流                                                              | 参数 |
|------------------------------------------------------------------------|------------|
| [**InterlockedExchange**](https://msdn.microsoft.com/library/windows/hardware/ff547892)                  |            |
| [**InterlockedGetValue**](https://msdn.microsoft.com/library/windows/hardware/ff547853)           |            |
| [**GetCompletedValue**](/windows/desktop/api/D3D12/nf-d3d12-id3d12fence-getcompletedvalue)             |            |
| [**等待**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandqueue-wait)                                |            |
| [**ID3D12CommandList**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandlist)                         |            |
| [**ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)  |            |
| [**IDXGISwapChain1::Present1**](https://msdn.microsoft.com/library/windows/desktop/hh446797) |            |



 

若要稍微简化示例，计算线程等待 GPU 完成每个迭代计划任何更多的计算工作之前。 在实践中，应用程序可能会想要保留计算队列 full 可实现从 GPU 的最大性能。

在计算线程：

``` syntax
DWORD D3D12nBodyGravity::AsyncComputeThreadProc(int threadIndex)
{
       ID3D12CommandQueue* pCommandQueue = m_computeCommandQueue[threadIndex].Get();
       ID3D12CommandAllocator* pCommandAllocator = m_computeAllocator[threadIndex].Get();
       ID3D12GraphicsCommandList* pCommandList = m_computeCommandList[threadIndex].Get();
       ID3D12Fence* pFence = m_threadFences[threadIndex].Get();

       while (0 == InterlockedGetValue(&m_terminating))
       {
              // Run the particle simulation.
              Simulate(threadIndex);

              // Close and execute the command list.
              ThrowIfFailed(pCommandList->Close());
              ID3D12CommandList* ppCommandLists[] = { pCommandList };

              pCommandQueue->ExecuteCommandLists(1, ppCommandLists);

              // Wait for the compute shader to complete the simulation.
              UINT64 threadFenceValue = InterlockedIncrement(&m_threadFenceValues[threadIndex]);
              ThrowIfFailed(pCommandQueue->Signal(pFence, threadFenceValue));
              ThrowIfFailed(pFence->SetEventOnCompletion(threadFenceValue, m_threadFenceEvents[threadIndex]));
              WaitForSingleObject(m_threadFenceEvents[threadIndex], INFINITE);

              // Wait for the render thread to be done with the SRV so that
              // the next frame in the simulation can run.
              UINT64 renderContextFenceValue = InterlockedGetValue(&m_renderContextFenceValues[threadIndex]);
              if (m_renderContextFence->GetCompletedValue() < renderContextFenceValue)
              {
                     ThrowIfFailed(pCommandQueue->Wait(m_renderContextFence.Get(), renderContextFenceValue));
                     InterlockedExchange(&m_renderContextFenceValues[threadIndex], 0);
              }

              // Swap the indices to the SRV and UAV.
              m_srvIndex[threadIndex] = 1 - m_srvIndex[threadIndex];

              // Prepare for the next frame.
              ThrowIfFailed(pCommandAllocator->Reset());
              ThrowIfFailed(pCommandList->Reset(pCommandAllocator, m_computeState.Get()));
       }

       return 0;
}
```



| 呼叫流                                                                   | 参数 |
|-----------------------------------------------------------------------------|------------|
| [**ID3D12CommandQueue**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue)                            |            |
| [**ID3D12CommandAllocator**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandallocator)                    |            |
| [**ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)              |            |
| [**ID3D12Fence**](/windows/desktop/api/D3D12/nn-d3d12-id3d12fence)                                          |            |
| [**InterlockedGetValue**](https://msdn.microsoft.com/library/windows/hardware/ff547853)                |            |
| [**关闭**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-close)                            |            |
| [**ID3D12CommandList**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandlist)                              |            |
| [**ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)       |            |
| [**InterlockedIncrement**](https://msdn.microsoft.com/library/windows/hardware/ff547910)                     |            |
| [**信号**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandqueue-signal)                                 |            |
| [**SetEventOnCompletion**](/windows/desktop/api/D3D12/nf-d3d12-id3d12fence-seteventoncompletion)            |            |
| [**WaitForSingleObject**](https://msdn.microsoft.com/library/windows/desktop/ms687032)                         |            |
| [**InterlockedGetValue**](https://msdn.microsoft.com/library/windows/hardware/ff547853)                |            |
| [**GetCompletedValue**](/windows/desktop/api/D3D12/nf-d3d12-id3d12fence-getcompletedvalue)                  |            |
| [**等待**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandqueue-wait)                                     |            |
| [**InterlockedExchange**](https://msdn.microsoft.com/library/windows/hardware/ff547892)                       |            |
| [**ID3D12CommandAllocator::Reset**](/windows/desktop/api/D3D12/nf-d3d12-id3d12commandallocator-reset)       |            |
| [**ID3D12GraphicsCommandList::Reset**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-reset) |            |



 

## <a name="run-the-sample"></a>运行示例

![最后一个 n 正文重力模拟的屏幕截图](images/nbodygravity.png)

## <a name="related-topics"></a>相关主题

<dl> <dt>

[D3D12 代码演练](d3d12-code-walk-throughs.md)
</dt> <dt>

[同步和多引擎](user-mode-heap-synchronization.md)
</dt> </dl>

 

 




