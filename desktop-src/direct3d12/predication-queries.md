---
title: 断言而查询
description: D3D12PredicationQueries 示例演示封闭剔除堆使用 DirectX 12 查询和断言而。 本演练介绍扩展 HelloConstBuffer 示例处理断言而查询所需的附加代码。
ms.assetid: F61817BB-45BC-4977-BE4A-EE0FDAFBCB57
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: acbcc718093e0303d9c47a42bfa1bd37028059db
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223732"
---
# <a name="predication-queries"></a>断言而查询

**D3D12PredicationQueries**示例演示封闭剔除堆使用 DirectX 12 查询和断言而。 本演练介绍了扩展所需的附加代码**HelloConstBuffer**处理断言而查询的示例。

-   [创建深度模具描述符堆和封闭查询堆](#create-a-depth-stencil-descriptor-heap-and-an-occlusion-query-heap)
-   [启用 alpha 值混合处理](#enable-alpha-blending)
-   [禁用颜色和深度写入](#disable-color-and-depth-writes)
-   [创建用于存储查询的结果的缓冲区](#create-a-buffer-to-store-the-results-of-the-query)
-   [绘制四边形以及执行并解决在封闭查询](#draw-the-quads-and-perform-and-resolve-the-occlusion-query)
-   [运行示例](#run-the-sample)
-   [相关的主题](#related-topics)

## <a name="create-a-depth-stencil-descriptor-heap-and-an-occlusion-query-heap"></a>创建深度模具描述符堆和封闭查询堆

在中**LoadPipeline**方法创建深度模具描述符堆。

``` syntax
              // Describe and create a depth stencil view (DSV) descriptor heap.
              D3D12_DESCRIPTOR_HEAP_DESC dsvHeapDesc = {};
              dsvHeapDesc.NumDescriptors = 1;
              dsvHeapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_DSV;
              dsvHeapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
              ThrowIfFailed(m_device->CreateDescriptorHeap(&dsvHeapDesc, IID_PPV_ARGS(&m_dsvHeap)));
```



<table>
<thead>
<tr class="header">
<th>呼叫流</th>
<th>参数</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/windows/desktop/api/D3D12/ns-d3d12-d3d12_descriptor_heap_desc"><strong>D3D12_DESCRIPTOR_HEAP_DESC</strong></a></td>
<td><dl><a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_descriptor_heap_type"><strong>D3D12_DESCRIPTOR_HEAP_TYPE</strong></a><br />
[<strong>D3D12_DESCRIPTOR_HEAP_FLAG</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_descriptor_heap_flags)<br />
</dl></td>
</tr>
<tr class="even">
<td><a href="/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createdescriptorheap"><strong>CreateDescriptorHeap</strong></a></td>

</tr>
</tbody>
</table>



 

在中**LoadAssets**方法为创建堆封闭查询。

``` syntax
     // Describe and create a heap for occlusion queries.
              D3D12_QUERY_HEAP_DESC queryHeapDesc = {};
              queryHeapDesc.Count = 1;
              queryHeapDesc.Type = D3D12_QUERY_HEAP_TYPE_OCCLUSION;
              ThrowIfFailed(m_device->CreateQueryHeap(&queryHeapDesc, IID_PPV_ARGS(&m_queryHeap)));
```



| 呼叫流                                                 | 参数                                                |
|-----------------------------------------------------------|-----------------------------------------------------------|
| [**D3D12\_查询\_堆\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_query_heap_desc) | [**D3D12\_查询\_堆\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_query_heap_type) |
| [**CreateQueryHeap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createqueryheap)   |                                                           |



 

## <a name="enable-alpha-blending"></a>启用 alpha 值混合处理

此示例绘制两个四边形，并说明了二进制封闭查询。 在前四动态显示在屏幕上，一个后退将有时封闭的像素。 在中**LoadAssets**方法，以便我们可以看到在哪个点 D3D 会考虑到底层封闭的四 alpha 值混合处理为此示例启用。

``` syntax
     // Enable alpha blending so we can visualize the occlusion query results.
              CD3DX12_BLEND_DESC blendDesc(CD3DX12_DEFAULT);
              blendDesc.RenderTarget[0] =
              {
                     TRUE, FALSE,
                     D3D12_BLEND_SRC_ALPHA, D3D12_BLEND_INV_SRC_ALPHA, D3D12_BLEND_OP_ADD,
                     D3D12_BLEND_ONE, D3D12_BLEND_ZERO, D3D12_BLEND_OP_ADD,
                     D3D12_LOGIC_OP_NOOP,
                     D3D12_COLOR_WRITE_ENABLE_ALL,
              };
```



<table>
<thead>
<tr class="header">
<th>呼叫流</th>
<th>参数</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="cd3dx12-blend-desc"><strong>CD3DX12_BLEND_DESC</strong></a></td>
<td><dl><a href="cd3dx12-default"><strong>CD3DX12_DEFAULT</strong></a><br />
[<strong>D3D12_BLEND</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_blend)<br />
[<strong>D3D12_BLEND_OP</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_blend_op)<br />
[<strong>D3D12_LOGIC_OP</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_logic_op)<br />
[<strong>D3D12_COLOR_WRITE_ENABLE</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_color_write_enable)<br />
</dl></td>
</tr>
</tbody>
</table>



 

## <a name="disable-color-and-depth-writes"></a>禁用颜色和深度写入

封闭查询由呈现四涵盖四我们想要测试其可见性的区域执行。 在更复杂场景中，查询可能是边界的卷，而不是简单的四核。 在任一情况下，将创建新的管道状态，以禁用，以便在封闭查询本身不会影响呈现处理过程的可见输出写入到呈现器目标和 z 缓冲区。

在中**LoadAssets**方法、 禁用颜色写入和深度封闭查询的状态的写入。

``` syntax
 // Disable color writes and depth writes for the occlusion query's state.
              psoDesc.BlendState.RenderTarget[0].RenderTargetWriteMask = 0;
              psoDesc.DepthStencilState.DepthWriteMask = D3D12_DEPTH_WRITE_MASK_ZERO;

              ThrowIfFailed(m_device->CreateGraphicsPipelineState(&psoDesc, IID_PPV_ARGS(&m_queryState)));
```



| 呼叫流                                                                            | 参数                                                  |
|--------------------------------------------------------------------------------------|-------------------------------------------------------------|
| [**D3D12\_GRAPHICS\_PIPELINE\_STATE\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_graphics_pipeline_state_desc) | [**D3D12\_DEPTH\_WRITE\_MASK**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_depth_write_mask) |
| [**CreateGraphicsPipelineState**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-creategraphicspipelinestate)      |                                                             |



 

## <a name="create-a-buffer-to-store-the-results-of-the-query"></a>创建用于存储查询的结果的缓冲区

在中**LoadAssets**缓冲区需要创建用于存储查询的结果的方法。 每个查询需要 8 个字节的 GPU 内存空间。 此示例仅执行一个查询，并对于简单性和可读性创建缓冲区大小完全 （即使此函数调用将分配 64k 页的 GPU 内存-最真实的应用程序可能会创建较大的缓冲区）。

``` syntax
 // Create the query result buffer.
              ThrowIfFailed(m_device->CreateCommittedResource(
                     &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
                     D3D12_HEAP_FLAG_NONE,
                     &CD3DX12_RESOURCE_DESC::Buffer(8),
                     D3D12_RESOURCE_STATE_GENERIC_READ,
                     nullptr,
                     IID_PPV_ARGS(&m_queryResult)
                     ));
```



<table>
<thead>
<tr class="header">
<th>呼叫流</th>
<th>参数</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommittedresource"><strong>CreateCommittedResource</strong></a></td>
<td><dl><a href="cd3dx12-heap-properties"><strong>CD3DX12_HEAP_PROPERTIES</strong></a><br />
[<strong>D3D12_HEAP_TYPE</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_heap_type)<br />
[<strong>D3D12_HEAP_FLAG</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_heap_flags)<br />
[<strong>CD3DX12_RESOURCE_DESC</strong>](cd3dx12-resource-desc.md)<br />
[<strong>D3D12_RESOURCE_STATES</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)<br />
</dl></td>
</tr>
</tbody>
</table>



 

## <a name="draw-the-quads-and-perform-and-resolve-the-occlusion-query"></a>绘制四边形以及执行并解决在封闭查询

完成后安装程序，在中更新的主循环**PopulateCommandLists**方法。

<dl> 1. 从备份以面对以便正常工作的透明度效果绘制四边形。 顶层后退绘制四都基于上一帧的查询的结果和是对此相当常见技术。 2. 更改要禁用的 PSO 呈现目标和深度模具写入。 3. 执行封闭查询。 4. 解析封闭查询。  
</dl>

``` syntax
       // Draw the quads and perform the occlusion query.
       {
              CD3DX12_GPU_DESCRIPTOR_HANDLE cbvFarQuad(m_cbvHeap->GetGPUDescriptorHandleForHeapStart(), m_frameIndex * CbvCountPerFrame, m_cbvSrvDescriptorSize);
              CD3DX12_GPU_DESCRIPTOR_HANDLE cbvNearQuad(cbvFarQuad, m_cbvSrvDescriptorSize);

              m_commandList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLESTRIP);
              m_commandList->IASetVertexBuffers(0, 1, &m_vertexBufferView);

              // Draw the far quad conditionally based on the result of the occlusion query
              // from the previous frame.
              m_commandList->SetGraphicsRootDescriptorTable(0, cbvFarQuad);
              m_commandList->SetPredication(m_queryResult.Get(), 0, D3D12_PREDICATION_OP_EQUAL_ZERO);
              m_commandList->DrawInstanced(4, 1, 0, 0);

              // Disable predication and always draw the near quad.
              m_commandList->SetPredication(nullptr, 0, D3D12_PREDICATION_OP_EQUAL_ZERO);
              m_commandList->SetGraphicsRootDescriptorTable(0, cbvNearQuad);
              m_commandList->DrawInstanced(4, 1, 4, 0);

              // Run the occlusion query with the bounding box quad.
              m_commandList->SetGraphicsRootDescriptorTable(0, cbvFarQuad);
              m_commandList->SetPipelineState(m_queryState.Get());
              m_commandList->BeginQuery(m_queryHeap.Get(), D3D12_QUERY_TYPE_BINARY_OCCLUSION, 0);
              m_commandList->DrawInstanced(4, 1, 8, 0);
              m_commandList->EndQuery(m_queryHeap.Get(), D3D12_QUERY_TYPE_BINARY_OCCLUSION, 0);

              // Resolve the occlusion query and store the results in the query result buffer
              // to be used on the subsequent frame.
              m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_queryResult.Get(), D3D12_RESOURCE_STATE_GENERIC_READ, D3D12_RESOURCE_STATE_COPY_DEST));
              m_commandList->ResolveQueryData(m_queryHeap.Get(), D3D12_QUERY_TYPE_BINARY_OCCLUSION, 0, 1, m_queryResult.Get(), 0);
              m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_queryResult.Get(), D3D12_RESOURCE_STATE_COPY_DEST, D3D12_RESOURCE_STATE_GENERIC_READ));
       }
```



<table>
<thead>
<tr class="header">
<th>呼叫流</th>
<th>参数</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="cd3dx12-gpu-descriptor-handle"><strong>CD3DX12_GPU_DESCRIPTOR_HANDLE</strong></a></td>
<td><a href="/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getgpudescriptorhandleforheapstart"><strong>GetGPUDescriptorHandleForHeapStart</strong></a></td>
</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetprimitivetopology"><strong>IASetPrimitiveTopology</strong></a></td>
<td><a href="https://msdn.microsoft.com/library/windows/desktop/ff728726"><strong>D3D_PRIMITIVE_TOPOLOGY</strong></a></td>
</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetvertexbuffers"><strong>IASetVertexBuffers</strong></a></td>

</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootdescriptortable"><strong>SetGraphicsRootDescriptorTable</strong></a></td>

</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpredication"><strong>SetPredication</strong></a></td>
<td><a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_predication_op"><strong>D3D12_PREDICATION_OP</strong></a></td>
</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced"><strong>DrawInstanced</strong></a></td>

</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpredication"><strong>SetPredication</strong></a></td>
<td><a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_predication_op"><strong>D3D12_PREDICATION_OP</strong></a></td>
</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootdescriptortable"><strong>SetGraphicsRootDescriptorTable</strong></a></td>

</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced"><strong>DrawInstanced</strong></a></td>

</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootdescriptortable"><strong>SetGraphicsRootDescriptorTable</strong></a></td>

</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate"><strong>SetPipelineState</strong></a></td>

</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-beginquery"><strong>BeginQuery</strong></a></td>
<td><a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_query_type"><strong>D3D12_QUERY_TYPE</strong></a></td>
</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced"><strong>DrawInstanced</strong></a></td>

</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-endquery"><strong>EndQuery</strong></a></td>
<td><a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_query_type"><strong>D3D12_QUERY_TYPE</strong></a></td>
</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier"><strong>ResourceBarrier</strong></a></td>
<td><dl><a href="cd3dx12-resource-barrier"><strong>CD3DX12_RESOURCE_BARRIER</strong></a><br />
[<strong>D3D12_RESOURCE_STATES</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)<br />
</dl></td>
</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvequerydata"><strong>ResolveQueryData</strong></a></td>
<td><a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_query_type"><strong>D3D12_QUERY_TYPE</strong></a></td>
</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier"><strong>ResourceBarrier</strong></a></td>
<td><dl><a href="cd3dx12-resource-barrier"><strong>CD3DX12_RESOURCE_BARRIER</strong></a><br />
[<strong>D3D12_RESOURCE_STATES</strong>](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)<br />
</dl></td>
</tr>
</tbody>
</table>



 

## <a name="run-the-sample"></a>运行示例

不封闭的像素：

![两个框不封闭的像素](images/not-occluded.png)

封闭的像素：

![一个框完全封闭的像素](images/occluded.png)

部分封闭的像素：

![一个框部分封闭的像素](images/partially-occluded.png)

## <a name="related-topics"></a>相关主题

<dl> <dt>

[D3D12 代码演练](d3d12-code-walk-throughs.md)
</dt> <dt>

[断言而](predication.md)
</dt> <dt>

[查询](queries.md)
</dt> </dl>

 

 




