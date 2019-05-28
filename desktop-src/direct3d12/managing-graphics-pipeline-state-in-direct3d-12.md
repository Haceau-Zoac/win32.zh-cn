---
title: 管理图形管道状态在 Direct3D 12
description: 本主题介绍如何在 Direct3D 12 中设置的图形管道状态。
ms.assetid: AAF97BD2-D532-469D-9242-CC94C06727C3
keywords:
- 管道状态
- 管道状态对象
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 4bc7d9fd6dd1a7ba4005f27cee8644a1eb9b4d6f
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224419"
---
# <a name="managing-graphics-pipeline-state-in-direct3d-12"></a>管理图形管道状态在 Direct3D 12

本主题介绍如何在 Direct3D 12 中设置的图形管道状态。

-   [管道状态概述](#pipeline-state-overview)
-   [图形管道设置管道状态对象的状态](#graphics-pipeline-states-set-with-pipeline-state-objects)
-   [图形管道外部管道状态对象设置的状态](#graphics-pipeline-states-set-outside-of-the-pipeline-state-object)
-   [图形管道状态继承](#graphics-pipeline-state-inheritance)
-   [相关的主题](#related-topics)

## <a name="pipeline-state-overview"></a>管道状态概述

Geometry 提交到图形处理单元 (GPU) 要绘制，有各种硬件设置，以确定如何解释和呈现输入的数据。 这些设置统称为称为图形管道状态和包含如光栅器状态的常见设置，blend 状态，和深度模具状态以及已提交的 geometry 和将用于着色器的原型拓扑类型呈现。 在 Microsoft Direct3D 12 中，大多数图形管道状态使用管道状态对象 (PSO) 设置。 应用程序可以创建无限的数量的这些对象，表示[ **ID3D12PipelineState** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12pipelinestate)接口，通常在初始化时。 然后，在呈现时，命令列表可以快速切换管道状态的多个设置通过调用[ **ID3D12GraphicsCommandList::SetPipelineState** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate)中直接的命令列表或捆绑包设置活动的 PSO。

在 Direct3D 11 中，图形管道状态已捆绑到大型的、 粗粒度的状态对象，如[ **ID3D11BlendState** ](https://msdn.microsoft.com/library/windows/desktop/ff476349) ，可以创建和使用方法设置在呈现时，在当前上下文中像[ **ID3D11DeviceContext::OMSetBlendState**](https://msdn.microsoft.com/library/windows/desktop/bb173595)。 这背后的想法是 GPU 可以通过设置相关的设置，例如，blend 状态设置，同时提高效率。 但是，现在的图形硬件之间有依赖项不同的硬件单位。 例如，硬件 blend 状态可能会在光栅状态，以及 blend 状态上具有依赖项。 在 Direct3D 12 的 Pso 旨在允许 GPU 在预处理中的所有依赖设置每个管道状态，通常初始化过程中，以便在呈现时尽可能高效的状态之间切换。

请注意，虽然可使用 Pso 设置的大多数管道状态设置，有一些状态设置单独使用提供的 Api 设置[ **ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)。 下面将详细讨论了这些设置和关联的 Api。 此外，有图形管道状态是由继承和保存从直接的命令列表和捆绑包的方式的差异。 本主题提供有关这两项的详细信息如下。

## <a name="graphics-pipeline-states-set-with-pipeline-state-objects"></a>图形管道设置管道状态对象的状态

若要查看的所有不同的管道状态可以使用管道状态对象设置的最简单方法是查看的参考主题[ **D3D12\_图形\_管道\_状态\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_graphics_pipeline_state_desc)传递给[ **ID3D12Device::CreateGraphicsPipelineState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-creategraphicspipelinestate)时初始化对象。 可以设置的状态的快速摘要包括：

-   所有包括的着色器、 顶点、 像素、 域、 外壳，和几何着色器字节码。
-   输入的顶点格式。
-   原型拓扑类型中。 请注意，输入装配器基元拓扑类型 （点、 线、 三角形、 修补程序） 设置在 PSO 内使用[ **D3D12\_基元\_拓扑\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_primitive_topology_type)枚举。 基元相邻页和排序 （行列表，行条带，行条带与相邻的数据，等等） 从设置中的命令列表 using [ **ID3D12GraphicsCommandList::IASetPrimitiveTopology** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetprimitivetopology)方法。
-   Blend 状态、 光栅器状态、 深度模具状态中。
-   深度模具和呈现目标格式，以及呈现器目标计数。
-   多采样参数。
-   流式处理的输出缓冲区。
-   根签名。 有关详细信息，请参阅[根签名](root-signatures.md)。

## <a name="graphics-pipeline-states-set-outside-of-the-pipeline-state-object"></a>图形管道外部管道状态对象设置的状态

可使用 Pso 设置大多数图形管道状态。 但是，有一组管道状态参数设置的调用的方法[ **ID3D12GraphicsCommandList** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)接口从命令列表中。 下表显示了这种方式和相应的方法设置的状态。



<table>
<thead>
<tr class="header">
<th>状态</th>
<th>方法</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>资源绑定</td>
<td><dl><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetindexbuffer"><strong>IASetIndexBuffer</strong></a><br />
[<strong>IASetVertexBuffers</strong>](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetvertexbuffers)<br />
[<strong>SOSetTargets</strong>](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-sosettargets)<br />
[<strong>OMSetRenderTargets</strong>](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetrendertargets)<br />
[<strong>SetDescriptorHeaps</strong>](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps)<br />
所有<strong>SetGraphicsRoot...</strong>方法<br />
所有<strong>SetComputeRoot...</strong>方法<br />
</dl></td>
</tr>
<tr class="even">
<td>视区</td>
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetviewports"><strong>RSSetViewports</strong></a></td>
</tr>
<tr class="odd">
<td>剪刀矩形</td>
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetscissorrects"><strong>RSSetScissorRects</strong></a></td>
</tr>
<tr class="even">
<td>Blend 系数</td>
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetblendfactor"><strong>OMSetBlendFactor</strong></a></td>
</tr>
<tr class="odd">
<td>深度模具参考值</td>
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetstencilref"><strong>OMSetStencilRef</strong></a></td>
</tr>
<tr class="even">
<td>输入装配器原型拓扑顺序和相邻类型。</td>
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetprimitivetopology"><strong>IASetPrimitiveTopology</strong></a></td>
</tr>
</tbody>
</table>



 

## <a name="graphics-pipeline-state-inheritance"></a>图形管道状态继承

因为直接的命令列表通常是每次捆绑包的一种用途用于的目的同时多次使用，有关它们如何继承前面的命令列表或捆绑包所设置的图形管道状态不同的规则。

使用 Pso 设置的图形管道状态，任何一种状态由继承直接的命令列表或捆绑包。 初始图形管道状态这两个直接的命令列表，在创建时设置捆绑包[ **ID3D12PipelineState** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12pipelinestate)参数[ **ID3D12Device::CreateCommandList**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandlist)。 如果没有 PSO 的调用中指定，则使用默认的初始状态。 可以通过调用更改命令列表中的当前 PSO [ **ID3D12GraphicsCommandList::SetPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate)。

直接命令列表也不会继承等命令列表方法设置的状态[ **RSSetViewports**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetviewports)。 有关非 PSO 状态的默认初始值的详细信息，请参阅[ **ID3D12GraphicsCommandList::ClearState**](/windows/desktop/api/D3D12/nf-d3d12-id3d12graphicscommandlist-clearstate)。

捆绑包继承原型拓扑类型除外 Pso 具有未设置的所有图形管道状态。 原型拓扑始终设置为[ **D3D12\_基元\_拓扑\_类型\_UNDEFINED** ](/windows/desktop/api/D3D12/ne-d3d12-d3d12_primitive_topology_type)捆绑包开始执行。 设置在绑定 （PSO 本身、 非基于 PSO 的状态和资源绑定） 内的任何状态会影响其父直接的命令列表的状态。 例如，如果[ **RSSetViewports** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetviewports)称为从捆绑包中指定的视区将继续在调用之后的父直接的命令列表中设置[ **ExecuteBundle** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executebundle)设置视区的调用。

设置命令列表或捆绑包中的资源绑定持久保存。 因此仍将被直接的命令列表中修改的资源绑定设置中后续子包执行。 并仍为父直接的命令列表中的后续调用将设置捆绑包中修改的资源绑定。

有关绑定的详细信息，请参阅**捆绑包语义**一部分[使用根签名](using-a-root-signature.md)。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[提交工作在 Direct3D 12](command-queues-and-command-lists.md)
</dt> </dl>

 

 




