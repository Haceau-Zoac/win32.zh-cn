---
title: 多适配器
description: 描述 D3D12 中对多引擎适配器系统的支持，其中包括应用程序明确面向多个 GPU 适配器的场景，以及驱动程序代表应用程序隐式地使用多个 GPU 适配器的场景。
ms.assetid: CC4C6594-D48F-40C1-93EE-9F98532BC038
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: f2df7665c48b4f0a18b01b1b0be81d26ab128e3d
ms.sourcegitcommit: 27a9dfa3ef68240fbf09f1c64dff7b2232874ef4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66725467"
---
# <a name="multi-adapter"></a>多适配器

描述 D3D12 中对多引擎适配器系统的支持，其中包括应用程序明确面向多个 GPU 适配器的场景，以及驱动程序代表应用程序隐式地使用多个 GPU 适配器的场景。

-   [多适配器概述](#multi-adapter-overview)
-   [跨适配器共享堆](#sharing-heaps-across-adapters)
-   [多适配器 API](#multi-adapter-apis)
    -   [单个节点](#single-nodes)
    -   [多个节点](#multiple-nodes)
    -   [资源创建 API](#resource-creation-apis)
-   [相关主题](#related-topics)

## <a name="multi-adapter-overview"></a>多适配器概述

GPU 适配器可以是任意制造商提供的任何支持 D3D12 的图形适配器（包括车载适配器）。

多个适配器在 API 中被引用  为节点。 这些节点从零开始索引，但在用于引用节点的位掩码中，0 转换为第 1 位，1 转换为第 2 位，依此类推。 许多元素（如队列）适用于每个节点，因此如果有两个节点，则会有两个默认的 3D 队列。 其他元素（例如管道状态和根及命令签名）可引用一个或多个节点，也可引用全部节点，如下图所示。

![队列应用于每个图形适配器](images/multigpu.png)

## <a name="sharing-heaps-across-adapters"></a>跨适配器共享堆

请参阅[共享堆](shared-heaps.md)部分。

## <a name="multi-adapter-apis"></a>多适配器 API

与之前的 D3D API 类似，每组链接的适配器都枚举为单个 [ IDXGIAdapter3](https://docs.microsoft.com/windows/desktop/api/dxgi1_4/nn-dxgi1_4-idxgiadapter3) 对象  。 附加到链接中任何适配器的所有输出都枚举为附加到单个 IDXGIAdapter3 对象  。

### <a name="single-nodes"></a>单个节点

应用程序可通过调用 [ID3D12Device::GetNodeCount](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getnodecount) 来确定与给定设备关联的物理适配器的数量  。 D3D12 中的许多 API 都接受 NodeMask，后者指示 API 调用所引用的节点集  。

调用以下（单节点）API 时，应用程序会指定要与 API 调用关联的单个节点。 大多数情况下，这是由 NodeMask 指定的  。 掩码中的每个位都对应一个节点。 对于本部分中描述的所有 API，都必须在 NodeMask 中恰好设置为 1 位  。

-   [**D3D12\_COMMAND\_QUEUE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_queue_desc)：具有 NodeMask 成员  。
-   [**CreateCommandQueue**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandqueue)：基于 [D3D12\_COMMAND\_QUEUE\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_queue_desc) 结构创建一个队列  。
-   [**CreateCommandList**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandlist)：采用 nodeMask 参数  。
-   [**D3D12\_DESCRIPTOR\_HEAP\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_descriptor_heap_desc)：具有 NodeMask 成员  。
-   [**CreateDescriptorHeap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createdescriptorheap)：基于 [D3D12\_DESCRIPTOR\_HEAP\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_descriptor_heap_desc) 结构创建描述符堆  。
-   [**D3D12\_QUERY\_HEAP\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_query_heap_desc)：具有 NodeMask 成员  。
-   [**CreateQueryHeap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createqueryheap)：基于 [D3D12\_QUERY\_HEAP\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_query_heap_desc) 结构创建一个查询堆  。

### <a name="multiple-nodes"></a>多个节点

调用以下 API 时，应用程序会指定一组要与 API 调用关联的节点。 将节点相关性指定为位掩码。 如果应用程序为位掩码传递 0，则 D3D12 驱动程序将其转换为位掩码 1（表示该对象与节点 0 关联）。

-   [**D3D12\_CROSS\_NODE\_SHARING\_TIER**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_cross_node_sharing_tier)：确定对跨节点共享的支持。
-   [**D3D12\_FEATURE\_DATA\_D3D12\_OPTIONS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options)：引用 [D3D12\_CROSS\_NODE\_SHARING\_TIER](/windows/desktop/api/d3d12/ne-d3d12-d3d12_cross_node_sharing_tier) 的结构  。
-   [**D3D12\_FEATURE\_DATA\_ARCHITECTURE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_architecture)：包含 NodeIndex 成员  。
-   [**D3D12\_GRAPHICS\_PIPELINE\_STATE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_graphics_pipeline_state_desc)：具有 NodeMask 成员  。
-   [**CreateGraphicsPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-creategraphicspipelinestate)：基于 [D3D12\_GRAPHICS\_PIPELINE\_STATE\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_graphics_pipeline_state_desc) 结构创建图形管道状态对象  。
-   [**D3D12\_COMPUTE\_PIPELINE\_STATE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_compute_pipeline_state_desc)：具有 NodeMask 成员  。
-   [**CreateComputePipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcomputepipelinestate)：基于 [D3D12\_COMPUTE\_PIPELINE\_STATE\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_compute_pipeline_state_desc) 结构创建计算管道状态对象  。
-   [**CreateRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createrootsignature)：采用 nodeMask 参数  。
-   [**D3D12\_COMMAND\_SIGNATURE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_signature_desc)：具有 NodeMask 成员  。
-   [**CreateCommandSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandsignature)：基于 [D3D12\_COMMAND\_SIGNATURE\_DESC](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_signature_desc) 结构创建命令签名对象  。

### <a name="resource-creation-apis"></a>资源创建 API

以下 API 引用节点掩码：

-   [**D3D12\_HEAP\_PROPERTIES**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_heap_properties)：同时具有 CreationNodeMask 和 VisibleNodeMask 成员   。
-   [**GetResourceAllocationInfo**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getresourceallocationinfo)：具有 visibleMask 参数  。
-   [**GetCustomHeapProperties**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getcustomheapproperties)：具有 nodeMask 参数  。

创建保留的资源时，不指定节点索引或掩码。 可将保留资源映射到任何节点的堆上（遵循跨节点共享规则）。

[MakeResident](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-makeresident) 方法在内部用于适配器队列，应用程序无需为此指定任何内容  。

在调用以下 [ID3D12Device](/windows/desktop/api/d3d12/nn-d3d12-id3d12device) API 时，应用程序无需指定一组要与 API 调用关联的节点，因为该 API 调用适用于所有节点  ：

-   [**CreateFence**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createfence)
-   [**GetDescriptorHandleIncrementSize**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getdescriptorhandleincrementsize)
-   [**SetStablePowerState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-setstablepowerstate)
-   [**CheckFeatureSupport**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)
-   [**CreateSampler**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createsampler)
-   [**CopyDescriptors**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-copydescriptors)
-   [**CopyDescriptorsSimple**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-copydescriptorssimple)
-   [**CreateSharedHandle**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createsharedhandle)
-   [**OpenSharedHandleByName**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-opensharedhandlebyname)
-   [**OpenSharedHandle**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-opensharedhandle)：使用“围栏”作为参数  使用“资源”或“堆”作为参数时，此方法不接受节点作为参数，因为节点掩码继承自先前创建的对象   。
-   [**CreateCommandAllocator**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandallocator)
-   [**CreateConstantBufferView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createconstantbufferview)
-   [**CreateRenderTargetView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createrendertargetview)
-   [**CreateUnorderedAccessView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createunorderedaccessview)
-   [**CreateDepthStencilView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createdepthstencilview)
-   [**CreateShaderResourceView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createshaderresourceview)

## <a name="related-topics"></a>相关主题

<dl> <dt>

[多引擎和多适配器同步](multi-engine-and-multi-gpu-synchronization.md)
</dt> </dl>

 

 




