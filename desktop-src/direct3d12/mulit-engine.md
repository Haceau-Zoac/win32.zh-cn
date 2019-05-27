---
title: 多适配器
description: 介绍 D3D12 中的多引擎适配器系统，涵盖应用程序显式面向多个 GPU 适配器的方案，以及其中的驱动程序隐式使用多个 GPU 适配器代表应用程序方案的支持。
ms.assetid: CC4C6594-D48F-40C1-93EE-9F98532BC038
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 6fb501c560273045d24ffc6cfff2a38521640354
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223879"
---
# <a name="multi-adapter"></a>多适配器

介绍 D3D12 中的多引擎适配器系统，涵盖应用程序显式面向多个 GPU 适配器的方案，以及其中的驱动程序隐式使用多个 GPU 适配器代表应用程序方案的支持。

-   [多适配器概述](#multi-adapter-overview)
-   [共享跨适配器堆](#sharing-heaps-across-adapters)
-   [多适配器 Api](#multi-adapter-apis)
    -   [单个节点](#single-nodes)
    -   [多个节点](#multiple-nodes)
    -   [资源创建 Api](#resource-creation-apis)
-   [相关的主题](#related-topics)

## <a name="multi-adapter-overview"></a>多适配器概述

GPU 适配器可以为任何制造商，支持 D3D12 （包括板载适配器），任何图形适配器。

多个适配器引用为*节点。* 在 API 中。 这些节点编制索引从零开始，但在用于引用的节点的位掩码，零将转换为 bit 1、 1 到位 2，依此类推。 大量元素，如队列、 将应用于每个节点，因此如果有两个节点，将两个默认 3D 队列。 其他元素，例如管道状态和根和命令签名，可以引用到一个或多个或所有节点，如以下关系图所示。

![队列将应用于每个图形适配器](images/multigpu.png)

## <a name="sharing-heaps-across-adapters"></a>共享跨适配器堆

请参阅[共享堆](shared-heaps.md)部分。

## <a name="multi-adapter-apis"></a>多适配器 Api

与以前的 D3D Api 类似，每个链接的适配器的枚举集时作为单个[ **IDXGIAdapter3** ](https://msdn.microsoft.com/library/windows/desktop/dn933221)对象。 连接到任何适配器的链接中的所有输出都枚举为附加到单个**IDXGIAdapter3**对象。

### <a name="single-nodes"></a>单个节点

应用程序可以确定与给定设备通过调用关联的物理适配器的数量[ **ID3D12Device::GetNodeCount**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getnodecount)。 D3D12 中的许多 Api 接受*NodeMask*，指示的 API 调用是指的节点集。

调用以下时 （单个节点） 的 Api，应用程序指定的 API 调用将与之关联的单个节点。 这指定通过大多数情况下*NodeMask*。 掩码中的每位对应于单个节点。 对于所有在本部分中所述的 Api，必须设置完全 1 的位*NodeMask*。

-   [**D3D12\_命令\_队列\_DESC** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_queue_desc) ： 具有*NodeMask*成员。
-   [**CreateCommandQueue** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandqueue) ： 创建从一个队列[ **D3D12\_命令\_队列\_DESC** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_queue_desc)结构。
-   [**CreateCommandList** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandlist) ： 采用*nodeMask*参数。
-   [**D3D12\_描述符\_堆\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_descriptor_heap_desc) ： 具有*NodeMask*成员。
-   [**CreateDescriptorHeap** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createdescriptorheap) ： 创建描述符堆[ **D3D12\_描述符\_堆\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_descriptor_heap_desc)结构。
-   [**D3D12\_查询\_堆\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_query_heap_desc) ： 具有*NodeMask*成员。
-   [**CreateQueryHeap** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createqueryheap) ： 创建从查询堆[ **D3D12\_查询\_堆\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_query_heap_desc)结构。

### <a name="multiple-nodes"></a>多个节点

在调用以下 Api 时，应用程序指定一的组 API 调用将与之关联的节点。 作为一个位掩码指定节点关联。 如果应用程序传递 0 的位掩码，然后 D3D12 驱动程序转换为这位掩码 1 （表示该对象与节点 0）。

-   [**D3D12\_跨\_节点\_共享\_层**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_cross_node_sharing_tier) ： 确定跨节点共享的支持。
-   [**D3D12\_功能\_数据\_D3D12\_选项**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_d3d12_options) ： 结构引用[ **D3D12\_跨\_节点\_共享\_层**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_cross_node_sharing_tier)。
-   [**D3D12\_功能\_数据\_体系结构**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_architecture) ： 包含*NodeIndex*成员。
-   [**D3D12\_图形\_管道\_状态\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_graphics_pipeline_state_desc) ： 具有*NodeMask*成员。
-   [**CreateGraphicsPipelineState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-creategraphicspipelinestate) ： 创建图形管道状态对象从[ **D3D12\_图形\_管道\_状态\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_graphics_pipeline_state_desc)结构。
-   [**D3D12\_计算\_管道\_状态\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_compute_pipeline_state_desc) ： 具有*NodeMask*成员。
-   [**CreateComputePipelineState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcomputepipelinestate) ： 创建计算管道状态对象从[ **D3D12\_计算\_管道\_状态\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_compute_pipeline_state_desc)结构。
-   [**CreateRootSignature**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrootsignature)： 采用*nodeMask*参数。
-   [**D3D12\_命令\_签名\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_command_signature_desc)： 具有*NodeMask*成员。
-   [**CreateCommandSignature** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandsignature) ： 创建命令签名对象从[ **D3D12\_命令\_签名\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_command_signature_desc)结构。

### <a name="resource-creation-apis"></a>资源创建 Api

以下 Api 引用节点掩码：

-   [**D3D12\_堆\_属性**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_heap_properties) ： 同时具有*CreationNodeMask*并*VisibleNodeMask*成员。
-   [**GetResourceAllocationInfo** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getresourceallocationinfo) ： 已*visibleMask*参数。
-   [**GetCustomHeapProperties** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getcustomheapproperties) ： 已*nodeMask*参数。

创建预留的资源时指定没有节点索引或蒙板。 保留的资源可以映射到堆上 （后跟跨节点共享规则） 的任何节点上。

该方法[ **MakeResident** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-makeresident)工作在内部适配器队列中，则无需为要为此指定的任何内容的应用程序。

当调用以下[ **ID3D12Device** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12device) Api，应用程序不需要指定一组将与关联的 API 调用，因为 API 调用将应用到所有节点的节点：

-   [**CreateFence**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createfence)
-   [**GetDescriptorHandleIncrementSize**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getdescriptorhandleincrementsize)
-   [**SetStablePowerState**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-setstablepowerstate)
-   [**CheckFeatureSupport**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport)
-   [**CreateSampler**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createsampler)
-   [**CopyDescriptors**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-copydescriptors)
-   [**CopyDescriptorsSimple**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-copydescriptorssimple)
-   [**CreateSharedHandle**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createsharedhandle)
-   [**OpenSharedHandleByName**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-opensharedhandlebyname)
-   [**OpenSharedHandle** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-opensharedhandle) ： 使用*fence*作为参数。 与*资源*或*堆*参数作为此方法不接受节点作为参数由于节点掩码继承自以前创建的对象。
-   [**CreateCommandAllocator**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandallocator)
-   [**CreateConstantBufferView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createconstantbufferview)
-   [**CreateRenderTargetView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrendertargetview)
-   [**CreateUnorderedAccessView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createunorderedaccessview)
-   [**CreateDepthStencilView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createdepthstencilview)
-   [**CreateShaderResourceView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createshaderresourceview)

## <a name="related-topics"></a>相关主题

<dl> <dt>

[多引擎和多适配器同步](multi-engine-and-multi-gpu-synchronization.md)
</dt> </dl>

 

 




