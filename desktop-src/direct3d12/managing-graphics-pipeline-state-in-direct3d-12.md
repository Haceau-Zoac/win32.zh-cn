---
title: 在 Direct3D 12 中管理图形管道状态
description: 本主题介绍如何在 Direct3D 12 中设置图形管道状态。
ms.assetid: AAF97BD2-D532-469D-9242-CC94C06727C3
keywords:
- 管道状态
- 管道状态对象
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 7c45c56250ed4af855917d0de44de36ce31269f6
ms.sourcegitcommit: 27a9dfa3ef68240fbf09f1c64dff7b2232874ef4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66725468"
---
# <a name="managing-graphics-pipeline-state-in-direct3d-12"></a>在 Direct3D 12 中管理图形管道状态

本主题介绍如何在 Direct3D 12 中设置图形管道状态。

-   [管道状态概述](#pipeline-state-overview)
-   [使用管道状态对象设置的图形管道状态](#graphics-pipeline-states-set-with-pipeline-state-objects)
-   [在管道状态对象之外设置的图形管道状态](#graphics-pipeline-states-set-outside-of-the-pipeline-state-object)
-   [图形管道状态继承](#graphics-pipeline-state-inheritance)
-   [相关主题](#related-topics)

## <a name="pipeline-state-overview"></a>管道状态概述

当几何图形提交到要绘制的图形处理单元 (GPU) 时，有各种硬件设置可用来确定如何解释和呈现输入数据。 这些设置统称为图形管道状态，并包括光栅器状态、混合状态和深度模具状态以及提交的 几何图形的基元拓扑类型和将用于呈现的着色器等常见设置。 在 Microsoft Direct3D 12 中，大多数图形管道状态是使用管道状态对象 (PSO) 设置的。 应用可以创建无限数量的这些对象，由 [**ID3D12PipelineState**](/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinestate) 接口表示（通常在初始化时）。 然后，在呈现时，命令列表可以通过调用直接命令列表或捆绑中的 [**ID3D12GraphicsCommandList::SetPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate) 来设置活动 PSO，以便快速切换管道状态的多个设置。

在 Direct3D 11 中，图形管道状态已绑定到大型、粗粒度状态对象（如 [**ID3D11BlendState**](https://docs.microsoft.com/windows/desktop/api/d3d11/nn-d3d11-id3d11blendstate)），可以在呈现时使用类似 [**ID3D11DeviceContext::OMSetBlendState**](https://docs.microsoft.com/windows/desktop/api/d3d10/nf-d3d10-id3d10device-omsetblendstate) 的方法在即时上下文中创建和设置这些对象。 这背后的构想是 GPU 可以通过同时设置相关设置（例如，混合状态设置）来提高效率。 但是，在如今的图形硬件中，不同的硬件单元之间存在依赖项。 例如，硬件混合状态可能会存在光栅状态以及混合状态的依赖项。 Direct3D 12 中的 PSO 旨在允许 GPU 在每个管道状态中预处理所有依赖设置（通常在初始化时），以便在呈现时尽可能高效地在状态之间切换。

请注意，虽然大多数管道状态设置是使用 PSO 设置的，但有一些状态设置是使用 [**ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 提供的 API 单独设置的。 下面详细介绍了这些设置和关联的 API。 此外，由直接命令列表和捆绑继承图形管道状态的方式和从直接命令列表和捆绑继承保留图形管道状态的方式存在差异。 本主题提供有关这两种方式的详细信息。

## <a name="graphics-pipeline-states-set-with-pipeline-state-objects"></a>使用管道状态对象设置的图形管道状态

查看可使用管道状态对象设置的所有不同管道状态的最简单方法是查看在初始化对象时传递到 [**ID3D12Device::CreateGraphicsPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-creategraphicspipelinestate) 的 [**D3D12\_GRAPHICS\_PIPELINE\_STATE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_graphics_pipeline_state_desc) 的参考主题。 可以设置的状态的快速摘要包括：

-   所有着色器（包括顶点、像素、域、外壳和几何着色器）的字节码。
-   输入顶点格式。
-   基元拓扑类型。 请注意，输入组装器基元拓扑类型（点、线条、三角形、修补程序）是使用 [**D3D12\_PRIMITIVE\_TOPOLOGY\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_primitive_topology_type) 枚举在 PSO 中设置的。 基元邻近度和排序（线列表、线条带、带邻近度数据的线条带等）是使用 [**ID3D12GraphicsCommandList::IASetPrimitiveTopology**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetprimitivetopology) 方法在命令列表中设置的。
-   混合状态、光栅器状态、深度模具状态。
-   深度模具和呈现器目标格式，以及呈现器目标计数。
-   多采样参数。
-   流式处理输出缓冲区。
-   根签名。 有关详细信息，请参阅[根签名](root-signatures.md)。

## <a name="graphics-pipeline-states-set-outside-of-the-pipeline-state-object"></a>在管道状态对象之外设置的图形管道状态

大多数图形管道状态是使用 PSO 设置的。 但是，有一组管道状态参数是通过调用命令列表中的 [**ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 接口的方法设置的。 下表显示了通过此方式和相应的方法设置的状态。



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
所有 <strong>SetGraphicsRoot...</strong> 方法<br />
所有 <strong>SetComputeRoot...</strong> 方法<br />
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
<td>混合系数</td>
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetblendfactor"><strong>OMSetBlendFactor</strong></a></td>
</tr>
<tr class="odd">
<td>深度模具参考值</td>
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetstencilref"><strong>OMSetStencilRef</strong></a></td>
</tr>
<tr class="even">
<td>输入组装器基元拓扑顺序和邻近度类型。</td>
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-iasetprimitivetopology"><strong>IASetPrimitiveTopology</strong></a></td>
</tr>
</tbody>
</table>



 

## <a name="graphics-pipeline-state-inheritance"></a>图形管道状态继承

由于直接命令列表通常用于一次使用一个，捆绑用于多次同时使用，因此如何继承以前的命令列表或捆绑设置的图形管道状态有不同的规则。

对于使用 PSO 设置的图形管道状态，直接命令列表或捆绑都不会继承其中任何一种状态。 直接命令列表和捆绑的初始图形管道状态在创建时使用 [**ID3D12PipelineState**](/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinestate) 参数设置为 [**ID3D12Device::CreateCommandList**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandlist)。 如果调用中未指定 PSO，则使用默认初始状态。 可以通过调用 [**ID3D12GraphicsCommandList::SetPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setpipelinestate) 更改命令列表中的当前 PSO。

直接命令列表也不会继承使用命令列表方法（例如，[**RSSetViewports**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetviewports)）设置的状态。 有关非 PSO 状态的默认初始值的详细信息，请参阅 [**ID3D12GraphicsCommandList::ClearState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearstate)。

捆绑继承未使用 PSO（基元拓扑类型除外）设置的所有图形管道状态。 当捆绑开始执行时，基元拓扑始终设置为 [**D3D12\_PRIMITIVE\_TOPOLOGY\_TYPE\_UNDEFINED**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_primitive_topology_type)。 在捆绑（PSO 本身、基于非 PSO 的状态和资源绑定）中设置的任何状态都会影响其父直接命令列表的状态。 例如，如果从捆绑中调用 [**RSSetViewports**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-rssetviewports)，则将在父直接的命令列表中为设置视区的 [**ExecuteBundle**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executebundle) 调用之后的调用设置指定的视区。

保留在命令列表或捆绑中设置的资源绑定。 因此，将仍在后续子捆绑执行中设置在直接命令列表中修改的资源绑定。 将仍为父直接命令列表中的后续调用设置在捆绑中修改的资源绑定。

有关绑定的详细信息，请参阅[使用根签名](using-a-root-signature.md)的**捆绑语义**部分。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 中的工作提交](command-queues-and-command-lists.md)
</dt> </dl>

 

 




