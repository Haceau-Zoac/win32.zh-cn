---
title: Direct3D 12 render pass
description: 渲染器通道功能通过减少与芯片外内存之间的内存流量，帮助渲染器提高 GPU 效率；它通过使应用程序更好地识别资源渲染排序要求和数据依赖项来实现此目的。
ms.localizationpriority: high
ms.topic: article
ms.date: 11/15/2018
ms.openlocfilehash: f776729f17ac0017d713c6f37bc71de7302a7c08
ms.sourcegitcommit: 780d4b1601c45658ef0b799b80d13f45a53d808d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/26/2020
ms.locfileid: "77635083"
---
# <a name="direct3d-12-render-passes"></a>Direct3D 12 render pass

呈现器通过功能是 Windows 10 版本1809（10.0;版本17763），并引入了 Direct3D 12 render pass 的概念。 呈现传递包含您记录到命令列表中的命令的子集。

若要声明每个呈现过程的开始和结束位置，请将包含的命令嵌套在对[**ID3D12GraphicsCommandList4：： BeginRenderPass**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist4-beginrenderpass)和[**EndRenderPass**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist4-endrenderpass)的调用内。 因此，任何命令列表都包含零个、一个或多个呈现阶段。

## <a name="scenarios"></a>方案

呈现器通过基于磁贴延迟呈现（TBDR）和其他技术，可以提高呈现器的性能。 更具体地说，通过使应用程序能够更好地确定资源呈现顺序要求和数据依赖关系，该方法有助于呈现器提高 GPU 效率。

专门编写的显示驱动程序可利用呈现刀路功能，提供最佳结果。 但是，即使在预先存在的驱动程序上，呈现传递 Api 也可运行（但不一定会提高性能）。

在这些方案中，呈现阶段旨在提供价值。

### <a name="allow-your-application-to-avoid-unnecessary-loadsstores-of-resources-fromto-main-memory-on-a-tile-based-deferred-rendering-tbdr-architecture"></a>允许你的应用程序避免在基于磁贴的延迟渲染（TBDR）体系结构上对主内存进行不必要的负载/存储资源

Render pass 的价值主张之一是，它为你提供了一个中心位置，用于指示应用程序对一组呈现操作的数据依赖关系。 这些数据依存关系允许显示驱动程序在绑定/关卡时间检查此数据，并发出说明，将资源负载/存储的最小化/存储到主内存中。

### <a name="allow-your-tbdr-architecture-to-opportunistically-persistent-resources-in-on-chip-cache-across-render-passes-even-in-separate-command-lists"></a>允许你的 TBDR 体系结构跨呈现器（即使在单独的命令列表中）在芯片缓存中找机会持久性资源

> [!NOTE]
> 具体而言，此方案仅限于在多个命令列表中写入同一呈现目标的情况。

常见的呈现模式适用于你的应用程序，以串行方式跨多个命令列表呈现相同的呈现器目标，即使呈现命令是并行生成的。 在此方案中使用渲染器会使这些刀路组合在一起（因为应用程序知道它将在立即后续的命令列表中恢复呈现），显示驱动程序可以避免刷新到命令列表中的主内存边界线.

## <a name="your-applications-responsibilities"></a>应用程序的责任

即使使用呈现刀路功能，Direct3D 12 运行时和显示驱动程序也不会承担推断出机会来重新排序/避免负载和存储。 若要正确利用 render pass 功能，你的应用程序将具有这些责任。

- 正确确定数据/排序依赖项的操作。
- 以最大程度地减少刷新的方式对其提交进行排序（因此，最大程度地减少对 **_PRESERVE**标志的使用）。
- 正确利用资源障碍并跟踪资源状态。
- 避免不需要的副本/清除。 为了帮助识别这些信息，你可以从[Windows PIX 上的 PIX 工具](https://devblogs.microsoft.com/pix/)使用自动性能警告。

## <a name="using-the-render-pass-feature"></a>使用呈现器处理功能

### <a name="what-is-a-render-pass"></a>什么是*呈现阶段*？

呈现过程通过这些元素来定义。

- 在呈现处理过程中固定的一组输出绑定。 这些绑定指向一个或多个呈现目标视图（RTVs）和/或深度模具视图（DSV）。
- 针对此输出绑定集的 GPU 操作的列表。
- 描述呈现器传递所针对的所有输出绑定的负载/存储依赖项的元数据。

### <a name="declare-your-output-bindings"></a>声明输出绑定

在呈现阶段开始时，声明将绑定到您的呈现器目标和/或您的深度/模具缓冲区。 绑定到呈现目标是可选的，并且可以将其绑定到深度/模具缓冲区。 但必须至少绑定到这两个中的一个，而在下面的代码示例中，我们绑定到这两个。

在对[**ID3D12GraphicsCommandList4：： BeginRenderPass**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist4-beginrenderpass)的调用中声明这些绑定。

```cppwinrt
void render_passes(::ID3D12GraphicsCommandList4 * pIGCL4,
    D3D12_CPU_DESCRIPTOR_HANDLE const& rtvCPUDescriptorHandle,
    D3D12_CPU_DESCRIPTOR_HANDLE const& dsvCPUDescriptorHandle)
{
    const float clearColor4[]{ 0.f, 0.f, 0.f, 0.f };
    CD3DX12_CLEAR_VALUE clearValue{ DXGI_FORMAT_R32G32B32_FLOAT, clearColor4 };

    D3D12_RENDER_PASS_BEGINNING_ACCESS renderPassBeginningAccessClear{ D3D12_RENDER_PASS_BEGINNING_ACCESS_TYPE_CLEAR, { clearValue } };
    D3D12_RENDER_PASS_ENDING_ACCESS renderPassEndingAccessPreserve{ D3D12_RENDER_PASS_ENDING_ACCESS_TYPE_PRESERVE, {} };
    D3D12_RENDER_PASS_RENDER_TARGET_DESC renderPassRenderTargetDesc{ rtvCPUDescriptorHandle, renderPassBeginningAccessClear, renderPassEndingAccessPreserve };

    D3D12_RENDER_PASS_BEGINNING_ACCESS renderPassBeginningAccessNoAccess{ D3D12_RENDER_PASS_BEGINNING_ACCESS_TYPE_NO_ACCESS, {} };
    D3D12_RENDER_PASS_ENDING_ACCESS renderPassEndingAccessNoAccess{ D3D12_RENDER_PASS_ENDING_ACCESS_TYPE_NO_ACCESS, {} };
    D3D12_RENDER_PASS_DEPTH_STENCIL_DESC renderPassDepthStencilDesc{ dsvCPUDescriptorHandle, renderPassBeginningAccessNoAccess, renderPassBeginningAccessNoAccess, renderPassEndingAccessNoAccess, renderPassEndingAccessNoAccess };

    pIGCL4->BeginRenderPass(1, &renderPassRenderTargetDesc, &renderPassDepthStencilDesc, D3D12_RENDER_PASS_FLAG_NONE);
    // Record command list.
    pIGCL4->EndRenderPass();
    // Begin/End further render passes and then execute the command list(s).
}
```

将[**D3D12_RENDER_PASS_RENDER_TARGET_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_render_pass_render_target_desc)结构的第一个字段设置为与一个或多个呈现器目标视图（RTVs）对应的 CPU 描述符句柄。 同样， [**D3D12_RENDER_PASS_DEPTH_STENCIL_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_render_pass_depth_stencil_desc)包含对应于深度模具视图（DSV）的 CPU 描述符句柄。 这些 CPU 描述符句柄与你要传递给[**ID3D12GraphicsCommandList：： OMSetRenderTargets**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetrendertargets)的句柄相同。 与**OMSetRenderTargets**一样，在调用**BEGINRENDERPASS**时，CPU 描述符会与其各自的（CPU 描述符）堆*对齐*。

不能将 RTVs 和 DSV 继承到呈现处理过程中。 相反，它们必须设置。 也不会将**BeginRenderPass**中声明的 RTVS 和 DSV 传播到命令列表。 相反，在呈现器传递后，它们处于未定义状态。

### <a name="render-passes-and-workloads"></a>呈现刀路和工作负荷

不能嵌套呈现器，也不能让 render pass 跨多个命令列表（当记录到单个命令列表时，它们必须开始和结束）。 下面的 "[呈现传递标志](#render-pass-flags)" 一节中讨论旨在实现呈现器传递的高效多线程生成的优化。

从呈现处理过程中执行的写入操作对你来说是*无效*的，因为你可以在后续的呈现阶段中进行读取。 这将从呈现处理过程中排除某些类型的障碍&mdash;例如，barriering 从**RENDER_TARGET**到当前绑定的呈现器目标上的**SHADER_RESOURCE** 。 有关详细信息，请参阅下面的[呈现阶段和资源障碍](#render-passes-and-resource-barriers)部分。

刚才提到的写读约束的一个例外涉及作为深度测试和呈现目标混合的一部分发生的隐式读取。
因此，在呈现过程中不允许使用这些 Api （如果在记录过程中调用了任何这些 Api，则核心运行时将删除它）。

- [**ID3D12GraphicsCommandList1::AtomicCopyBufferUINT**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-atomiccopybufferuint)
- [**ID3D12GraphicsCommandList1::AtomicCopyBufferUINT64**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-atomiccopybufferuint64)
- [**ID3D12GraphicsCommandList4::BeginRenderPass**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist4-beginrenderpass)
- [**ID3D12GraphicsCommandList::ClearDepthStencilView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-cleardepthstencilview)
- [**ID3D12GraphicsCommandList::ClearRenderTargetView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearrendertargetview)
- [**ID3D12GraphicsCommandList::ClearState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearstate)
- [**ID3D12GraphicsCommandList::ClearUnorderedAccessViewFloat**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearunorderedaccessviewfloat)
- [**ID3D12GraphicsCommandList::ClearUnorderedAccessViewUint**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearunorderedaccessviewuint)
- [**ID3D12GraphicsCommandList::CopyBufferRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion)
- [**ID3D12GraphicsCommandList::CopyResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copyresource)
- [**ID3D12GraphicsCommandList::CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)
- [**ID3D12GraphicsCommandList::CopyTiles**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytiles)
- [**ID3D12GraphicsCommandList：:D iscardResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-discardresource)
- [**ID3D12GraphicsCommandList：:D ispatch**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch)
- [**ID3D12GraphicsCommandList::OMSetRenderTargets**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetrendertargets)
- [**ID3D12GraphicsCommandList::ResolveQueryData**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvequerydata)
- [**ID3D12GraphicsCommandList::ResolveSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvesubresource)
- [**ID3D12GraphicsCommandList1::ResolveSubresourceRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-resolvesubresourceregion)
- [**ID3D12GraphicsCommandList3::SetProtectedResourceSession**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist3-setprotectedresourcesession)

### <a name="render-passes-and-resource-barriers"></a>呈现阶段和资源障碍

你不能读取或使用同一呈现处理过程中发生的写入。 某些障碍不符合此约束，例如，从**D3D12_RESOURCE_STATE_RENDER_TARGET**到\*当前绑定的呈现器目标上的 **_SHADER_RESOURCE** （调试层会错误）。 但在当前渲染传递*外部*编写的呈现器目标上的相同屏障是相容的，因为写入操作将在当前的呈现处理开始之前完成。
你可能会从了解显示器驱动程序可以在这方面做出的某些优化中获益。 在给定符合工作负荷的情况下，显示驱动程序可能会将呈现传递过程中遇到的任何障碍移动到呈现器传递的开头。 在这里，它们可以合并（而不会干扰任何平铺/装箱操作）。 这是一种有效的优化条件，前提是在当前的呈现开始之前，所有写入操作都已完成。

下面是一个更完整的驱动程序优化示例，假设你有一个呈现引擎，该引擎具有一个预 Direct3D 12 样式资源绑定设计，&mdash;根据资源的绑定方式*在需求上*进行屏障。 当向框架的末尾写入无序访问视图（UAV）时，在帧结束时，可能会出现引擎将资源置于**D3D12_RESOURCE_STATE_UNORDERED_ACCESS**状态的情况。 在以下框架中，当引擎转到将资源绑定为着色器资源视图（SRV）时，它会发现资源未处于正确的状态，并且将从**D3D12_RESOURCE_STATE_UNORDERED_ACCESS**向**D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE**发出屏障。 如果该关卡出现在呈现处理过程中，则显示驱动程序将两端对齐，假定所有写入已经在此当前呈现器传递过程*之外*进行，并且因此（并且此处是优化操作的位置），则显示驱动程序可能会*将关卡移*到呈现处理过程的开始处。 同样，这是有效的，前提是你的代码符合本部分中所述的写读约束和最后一个。


以下是一致屏障的示例。
- **D3D12_RESOURCE_STATE_UNORDERED_ACCESS** **D3D12_RESOURCE_STATE_INDIRECT_ARGUMENT**。
- **D3D12_RESOURCE_STATE_COPY_DEST** **\*_SHADER_RESOURCE**。

这些是不相容障碍的示例。

- **D3D12_RESOURCE_STATE_RENDER_TARGET**当前绑定的 RTVs/dsv 上的任何读取状态。
- **D3D12_RESOURCE_STATE_DEPTH_WRITE**当前绑定的 RTVs/dsv 上的任何读取状态。
- 任何别名屏障。
- 无序访问视图（UAV）屏障。 

### <a name="resource-access-declaration"></a>资源访问声明

在**BeginRenderPass**时，还必须指定在该经历中充当 RTVs 和/或 DSV 的所有资源，还必须指定其开始和终止*访问*特性。 正如上面的 "[声明输出绑定](#declare-your-output-bindings)" 一节中的代码示例所示，可以通过[**D3D12_RENDER_PASS_RENDER_TARGET_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_render_pass_render_target_desc)和[**D3D12_RENDER_PASS_DEPTH_STENCIL_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_render_pass_depth_stencil_desc)结构来实现此目的。

有关更多详细信息，请参阅[**D3D12_RENDER_PASS_BEGINNING_ACCESS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_render_pass_beginning_access)和[**D3D12_RENDER_PASS_ENDING_ACCESS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_render_pass_ending_access)结构，以及[**D3D12_RENDER_PASS_BEGINNING_ACCESS_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_render_pass_beginning_access_type)和[**D3D12_RENDER_PASS_ENDING_ACCESS_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_render_pass_ending_access_type)枚举。

### <a name="render-pass-flags"></a>呈现传递标志

传递给**BeginRenderPass**的最后一个参数是一个呈现器传递标志（ [**D3D12_RENDER_PASS_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_render_pass_flags)枚举中的值）。

```cppwinrt
enum D3D12_RENDER_PASS_FLAGS
{
    D3D12_RENDER_PASS_FLAG_NONE = 0,
    D3D12_RENDER_PASS_FLAG_ALLOW_UAV_WRITES = 0x1,
    D3D12_RENDER_PASS_FLAG_SUSPENDING_PASS = 0x2,
    D3D12_RENDER_PASS_FLAG_RESUMING_PASS = 0x4
};
```

#### <a name="uav-writes-within-a-render-pass"></a>呈现处理过程中的 UAV 写入

在呈现处理过程中允许无序访问视图（UAV）写入，但你必须特别指出，你将通过指定**D3D12_RENDER_PASS_FLAG_ALLOW_UAV_WRITES**在呈现器中发出 UAV 写入，以便显示驱动程序可以在必要时选择禁用平铺。

UAV 访问必须遵循上文所述的写读约束（在呈现传递过程中的写入无效，无法读取，直到后续的呈现通过）。 不允许在呈现处理过程中出现 UAV 障碍。

UAV 绑定（通过根表或根描述符）继承到 render pass，并传播到 render pass。

#### <a name="suspending-passes-and-resuming-passes"></a>挂起-传递和恢复-通过

你可以将整个呈现传递视为挂起传递和/或恢复通过。 暂停后继续对必须在两次传递间具有相同的视图/访问标志，并且可能不会在挂起的呈现过程和恢复呈现过程之间具有任何干预 GPU 操作（例如，绘制、调度、放弃、清除、复制、更新磁贴映射、immediates、查询、查询解析）。

目标用例是多线程呈现，其中，假设有四个命令列表（每个命令列表都具有其自己的呈现阶段）可面向相同的呈现器目标。 当跨命令列表暂停/恢复 render 传递时，必须在对[**ID3D12CommandQueue：： ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)的相同调用中执行命令列表。

呈现过程可以是继续和挂起的。 在刚才给定的多线程示例中，命令列表2和3将分别从1和2恢复。 同时，2和3将分别挂起为3和4。

## <a name="query-for-render-passes-feature-support"></a>针对呈现器传递功能的查询支持

可以调用[**ID3D12Device：： CheckFeatureSupport**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)来查询设备驱动程序和/或硬件有效支持呈现刀路的范围。

```cppwinrt
D3D12_RENDER_PASS_TIER get_render_passes_tier(::ID3D12Device * pIDevice)
{
    D3D12_FEATURE_DATA_D3D12_OPTIONS5 featureSupport{};
    winrt::check_hresult(
        pIDevice->CheckFeatureSupport(D3D12_FEATURE_D3D12_OPTIONS5, &featureSupport, sizeof(featureSupport))
    );
    return featureSupport.RenderPassesTier;
}
...
    D3D12_RENDER_PASS_TIER renderPassesTier{ get_render_passes_tier(pIDevice) };
```

由于运行时的映射逻辑，render 始终会起作用。 但根据功能支持，它们并不会始终提供权益。 你可以使用类似于上面的代码示例的代码来确定是否需要你在将命令作为呈现器传递来发出命令时，以及当它确实不是权益时（即，当运行时只是映射到现有的 API 图面）。 如果使用的是[D3D11On12](/windows/desktop/direct3d12/direct3d-11-on-12)，则执行此检查尤其重要。

有关三个支持层的说明，请参阅[**D3D12_RENDER_PASS_TIER**](/windows/win32/api/d3d12/ne-d3d12-d3d12_render_pass_tier)枚举。
