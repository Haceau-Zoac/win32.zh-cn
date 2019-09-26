---
title: 多适配器系统
description: 描述在 Direct3D 12 中对安装了多个适配器的系统的支持，涵盖了应用程序显式面向多个 GPU 适配器的情况，以及驱动程序代表你的程序.
ms.assetid: CC4C6594-D48F-40C1-93EE-9F98532BC038
ms.localizationpriority: high
ms.topic: article
ms.date: 09/25/2019
ms.openlocfilehash: 7fe596688bfa551de4e5394301fa5f88a498be88
ms.sourcegitcommit: d6102d9e2b26368142fe5b006c65acb50c98be65
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/26/2019
ms.locfileid: "71306459"
---
# <a name="multi-adapter-systems"></a>多适配器系统

描述在 Direct3D 12 中对安装了多个适配器的系统的支持，涵盖了应用程序显式面向多个 GPU 适配器的情况，以及驱动程序代表你的程序.

## <a name="multi-adapter-overview"></a>多适配器概述

GPU 适配器可以是支持 Direct3D 12 的任何制造商提供的任何适配器（图形、独立或集成）。

多个适配器作为*节点*进行引用。 许多元素（如队列）适用于每个节点，因此如果有两个节点，则会有两个默认的 3D 队列。 其他元素（如管道状态、根和命令签名）可以引用一个或多个或所有节点，如图中所示。

![队列应用于每个图形适配器](images/multigpu.png)

## <a name="sharing-heaps-across-adapters"></a>跨适配器共享堆

请参阅[共享堆](shared-heaps.md)主题。

## <a name="multi-adapter-apis-and-node-masks"></a>多适配器 Api 和节点掩码

与上一个 Direct3D Api 类似，每组链接的适配器都作为单个[**IDXGIAdapter3**](/windows/win32/api/dxgi1_4/nn-dxgi1_4-idxgiadapter3)对象进行枚举。 附加到链接中任何适配器的所有输出都枚举为附加到单个 IDXGIAdapter3 对象。

应用程序可以通过调用[**ID3D12Device：： GetNodeCount**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-getnodecount)来确定与给定设备关联的物理适配器的数目。

Direct3D 12 中的许多 Api 接受*节点掩码*（位掩码），这表示 API 调用所引用的节点集。 每个节点都有一个从零开始的索引。 但在节点掩码中，零转换为第1位;1转换为第2位;依此类推。

### <a name="single-nodes"></a>单个节点

调用以下（单节点） Api 时，应用程序将指定与 API 调用关联的单个节点。 大多数情况下，这是由节点掩码指定的。 掩码中的每个位都对应一个节点。 对于本部分中所述的所有 Api，必须在节点掩码中仅设置一位。

-   [**Direct3D12\_命令\_队列DESC\_** ](/windows/win32/api/d3d12/ns-d3d12-d3d12_command_queue_desc) ：具有*NodeMask*成员。
-   [**CreateCommandQueue**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcommandqueue) ：从[**Direct3D 12\_\_命令队列\_DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_command_queue_desc)结构创建队列。
-   [**CreateCommandList**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcommandlist)：采用 nodeMask 参数。
-   [**Direct3D12\_描述符\_堆DESC\_** ](/windows/win32/api/d3d12/ns-d3d12-d3d12_descriptor_heap_desc) ：具有*NodeMask*成员。
-   [**CreateDescriptorHeap**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createdescriptorheap) ：从[**Direct3D 12\_\_描述符堆\_DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_descriptor_heap_desc)结构创建描述符堆。
-   [**Direct3D12\_查询\_堆DESC\_** ](/windows/win32/api/d3d12/ns-d3d12-d3d12_query_heap_desc) ：具有*NodeMask*成员。
-   [**CreateQueryHeap**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createqueryheap) ：从[**Direct3D 12\_\_查询堆\_的 DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_query_heap_desc)结构创建查询堆。

### <a name="multiple-nodes"></a>多个节点

调用以下（多个节点） Api 时，应用程序会指定一组将与之关联的 API 调用的节点。 将节点关联指定为节点掩码，可能会设置多个位。 如果你的应用程序为此位掩码传入0，则 Direct3D 12 驱动程序会将其转换为位掩码1（表示对象与节点0相关联）。

-   [**Direct3D 12\_跨\_节点\_共享层\_** ](/windows/win32/api/d3d12/ne-d3d12-d3d12_cross_node_sharing_tier) ：确定对跨节点共享的支持。
-   [**Direct3D12\_功能数据\_D3D12选项\_：结构引用 Direct3D 12\_** ](/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) [ **\_跨\_节点共享层\_\_** ](/windows/win32/api/d3d12/ne-d3d12-d3d12_cross_node_sharing_tier).
-   [**Direct3D12\_功能\_数据体系\_结构**](/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_architecture) ：包含*NodeIndex*成员。
-   [**Direct3D12\_图形管道\_状态DESC\_：具有 NodeMask 成员。\_** ](/windows/win32/api/d3d12/ns-d3d12-d3d12_graphics_pipeline_state_desc)
-   [**CreateGraphicsPipelineState**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-creategraphicspipelinestate) ：从[**Direct3D 12\_图形\_\_管道状态\_DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_graphics_pipeline_state_desc)结构创建图形管道状态对象。
-   [**Direct3D12\_计算管道\_状态DESC\_：具有 NodeMask 成员。\_** ](/windows/win32/api/d3d12/ns-d3d12-d3d12_compute_pipeline_state_desc)
-   [**CreateComputePipelineState**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcomputepipelinestate) ：从[**Direct3D 12\_计算\_\_管道状态\_DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_compute_pipeline_state_desc)结构创建计算管道状态对象。
-   [**CreateRootSignature**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createrootsignature)：采用 nodeMask 参数。
-   [**Direct3D12\_命令\_签名DESC\_** ](/windows/win32/api/d3d12/ns-d3d12-d3d12_command_signature_desc)：具有*NodeMask*成员。
-   [**CreateCommandSignature**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcommandsignature) ：从[**Direct3D 12\_\_命令签名\_DESC**](/windows/win32/api/d3d12/ns-d3d12-d3d12_command_signature_desc)结构创建命令签名对象。

### <a name="resource-creation-apis"></a>资源创建 API

以下 Api 引用节点掩码。

-   [**Direct3D12\_堆\_属性**](/windows/win32/api/d3d12/ns-d3d12-d3d12_heap_properties) ：同时具有*CreationNodeMask*和*VisibleNodeMask*成员。
-   [**GetResourceAllocationInfo**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-getresourceallocationinfo)：具有 visibleMask 参数。
-   [**GetCustomHeapProperties**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-getcustomheapproperties)：具有 nodeMask 参数。

创建保留资源时，未指定节点索引或掩码。 可将保留资源映射到任何节点的堆上（遵循跨节点共享规则）。

方法[**MakeResident**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-makeresident)在内部与适配器队列一起工作，无需应用程序为此指定任何内容。

调用以下[**ID3D12Device**](/windows/win32/api/d3d12/nn-d3d12-id3d12device) api 时，应用程序无需指定 api 调用将与之关联的一组节点，因为 api 调用适用于所有节点。

-   [**CreateFence**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createfence)
-   [**GetDescriptorHandleIncrementSize**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-getdescriptorhandleincrementsize)
-   [**SetStablePowerState**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-setstablepowerstate)
-   [**CheckFeatureSupport**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)
-   [**CreateSampler**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createsampler)
-   [**CopyDescriptors**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-copydescriptors)
-   [**CopyDescriptorsSimple**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-copydescriptorssimple)
-   [**CreateSharedHandle**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createsharedhandle)
-   [**OpenSharedHandleByName**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-opensharedhandlebyname)
-   [**OpenSharedHandle**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-opensharedhandle)：使用“围栏”作为参数 使用“资源”或“堆”作为参数时，此方法不接受节点作为参数，因为节点掩码继承自先前创建的对象。
-   [**CreateCommandAllocator**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createcommandallocator)
-   [**CreateConstantBufferView**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createconstantbufferview)
-   [**CreateRenderTargetView**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createrendertargetview)
-   [**CreateUnorderedAccessView**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createunorderedaccessview)
-   [**CreateDepthStencilView**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createdepthstencilview)
-   [**CreateShaderResourceView**](/windows/win32/api/d3d12/nf-d3d12-id3d12device-createshaderresourceview)

## <a name="related-topics"></a>相关主题

[Direct3D 12 编程指南](directx-12-programming-guide.md)