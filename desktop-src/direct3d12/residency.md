---
title: 驻留
description: 如果 GPU 可访问某个对象，则该对象将被视为常驻对象。
ms.assetid: 956F80D7-EEC8-4D88-B251-EE325614F31E
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 96ca17ada8e9e464880b202e9752ce72a5d8005f
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224194"
---
# <a name="residency"></a>驻留

如果 GPU 可访问某个对象，则该对象将被视为常驻对象。 

-   [驻留预算](#residency-budget)
-   [堆资源](#heap-resources)
-   [驻留优先级](#residency-priorities)
    -   [默认优先级算法](#default-priority-algorithm)
-   [编程驻留管理](#programming-residency-management)
-   [相关主题](#related-topics)

## <a name="residency-budget"></a>驻留预算

GPU 尚不支持分页错误，因此，当 GPU 可以访问数据时，应用程序必须将数据提交到物理内存。 此过程称为“使内容常驻”，必须针对物理系统内存和物理离散的视频内存执行。 在 D3D12 中，大多数 API 对象封装一定数量的 GPU 可访问内存。 在创建 API 对象期间，该 GPU 可访问内存被设置为常驻，在销毁 API 对象时将被逐出。

可供该进程使用的物理内存量称为视频内存预算。 该预算可能会随着后台进程的唤醒和睡眠而有明显的波动；当用户切换到另一应用程序时，这种波动会很大。 应用程序在预算更改时可以收到通知，并可以轮询当前预算以及当前已消耗的内存量。 如果应用程序不在预算范围内，该进程将会间歇性地冻结以使其他应用程序能够运行，并且/或者创建 API 将返回失败。 [**IDXGIAdapter3**](https://msdn.microsoft.com/library/windows/desktop/dn933221) 接口提供与此功能相关的方法，具体而言，是 [**QueryVideoMemoryInfo**](https://msdn.microsoft.com/library/windows/desktop/dn933223) 和 [**RegisterVideoMemoryBudgetChangeNotificationEvent**](https://msdn.microsoft.com/library/windows/desktop/dn933231)。

建议应用程序使用保留来指示它们必须获得的内存量。 理想情况下，用户指定的“低”图形设置甚至更低的设置是此类保留的适当值。 设置保留不会使应用程序的预算高于它在正常情况下获得的预算。 相反，保留信息可帮助 OS 内核快速最小化较大内存压力造成的影响。 即使应用程序不是前台应用程序，也不保证保留可供该应用程序使用。

## <a name="heap-resources"></a>堆资源

尽管有许多 API 对象可以封装某些 GPU 可访问内存，但堆和资源预期是应用程序消耗和管理物理内存的最有效方式。 堆是用于管理物理内存的最低级单元，因此，最好对其驻留属性有一定的了解。

-   无法将堆指定为部分常驻，但对于保留的资源有解决方法可用。
-   应该将堆的预算计划为特定池的一部分。 UMA 适配器有一个池，而离散的适配器有两个池。 尽管内核确实可将离散适配器中的某些堆从视频内存转移到系统内存，则它只会在迫不得已的情况下才这样做。 应用程序不应依赖于内核的超预算行为，而应该注重良好的预算管理。
-   可以从驻留位置逐出堆，使其内容可以分页到磁盘。 但是，堆销毁是一种更可靠的技术，它可以释放所有适配器体系结构中的驻留位置。 在[**D3D12\_FEATURE\_DATA\_GPU\_VIRTUAL\_ADDRESS\_SUPPORT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_gpu_virtual_address_support) 的 *theMaxGPUVirtualAddressBitsPerProcess* 字段接近预算大小的适配器中，[**Evict**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-evict) 无法可靠地回收驻留位置。
-   堆创建操作可能很缓慢；但它已针对后台线程处理进行优化。 建议在后台线程中创建堆，以避免干扰渲染线程。 在 D3D12 中，多个线程可以安全地同时调用创建例程。

D3D12 在其资源模型中引入了更大的灵活性和正交性，使应用程序支持更多的选项。 D3D12 中的资源有三个高级类型：已提交、已定位和已保留。

-   已提交资源同时创建资源和堆。 堆是隐式的，不可直接访问。 堆的大小适当，可在其中放置整个资源。
-   已定位资源允许将资源放在堆中的非零偏移位置。 偏移量通常必须经过 64KB 对齐；但两个方向存在某些例外情况。 MSAA 资源需要 4MB 偏移对齐，对于小纹理可以使用 4KB 偏移对齐。 已定位资源不能直接重新定位或重新映射到另一个堆；但是，使用它们可在堆之间方便地重新定位资源数据。 在不同的堆中创建新的已定位资源并复制资源数据后，必须对新的资源数据位置使用新的资源描述符。
-   仅当适配器支持图块式资源层 1 或更高的层时，已保留资源才可用。 如果可用，它们会提供可用的最先进驻留管理技术；但目前并非所有适配器都支持这些技术。 使用已保留资源可以重新映射资源，而无需重新生成资源描述符、部分 mip 级驻留和稀疏纹理方案等。即使已保留资源可用，也并非所有资源类型均受支持，因此，完全基于常规页面的驻留管理器尚不可行。

## <a name="residency-priorities"></a>驻留优先级

当内存压力要求将一部分资源降级时，开发人员可以使用 Windows 10 创意者更新来判断最好是将哪些堆和资源保持常驻。 这样可以帮助开发人员通过利用运行时无法从 API 的使用推断的知识来创建性能更好的应用程序。 当开发人员从已提交资源改用已保留资源和图块式资源后，有望能够更轻松且更合理地指定优先级。

应用这些优先级必定比管理两个动态内存预算，并在它们之间手动降级和升级资源更轻松，因为应用程序已经可以执行此操作。 因此，驻留优先级 API 设计的粒度级不高，但会在创建每个堆或资源时为其分配合理的默认优先级。 有关详细信息，请参阅 [**ID3D12Device1::SetResidencyPriority**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device1-setresidencypriority) 和 the [**D3D12\_RESIDENCY\_PRIORITY**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_residency_priority) 枚举。

使用优先级预期可让开发人员：

-   提高几个例外堆的优先级，以更好地缓解比以其自然访问模式需求更高的速度或频率降级的堆造成的性能影响。 从 Direct3D 11 或 OpenGL 等图形 API（其资源管理模型明显不同于 Direct3D 12）移植的应用程序预期会利用此方法。
-   以固定方式、根据程序员对访问频率的了解或以动态方式将几乎所有的堆优先级覆盖为应用程序自身的桶化方案；固定方案的管理比动态方案更容易，但可能更低效，且在开发过程中当使用模式发生变化时需要程序员的干预。 设计时考虑到了 Direct3D 12 式资源管理的应用程序预期会利用此方法，例如使用驻留库（尤其是动态方案）的应用程序。

### <a name="default-priority-algorithm"></a>默认优先级算法

在不事先了解默认优先级算法的情况下，应用程序无法为它尝试管理的任何堆指定有用的优先级。 这是因为，分配给堆的特定优化级值派生自争用同一内存的其他优先堆的相对优先级。

为生成默认优先级而选择的策略是将堆分类成两个桶，并优先使用（分配更高的优先级）GPU 频繁写入的堆（相比其他堆）。

高优先级桶包含使用标志创建的堆和资源。标志将这些堆和资源标识为渲染器目标、深度模具缓冲区或无序访问视图 (UAV)。 这是在从 **D3D12\_RESIDENCY\_PRIORITY\_HIGH** 开始的范围内分配的优先级值；若要进一步在这些堆和资源中指定优先级，需将优先级的最低 16 位设置为堆或资源的大小除以 10MB（对于极大型堆将饱和为 0xFFFF）。 这种额外的优先级非常适合较大型堆和资源。

低优先级桶包含所有其他堆和资源，为它们分配的优先级值为 **D3D12\_RESIDENCY\_PRIORITY\_NORMAL**。 系统不会尝试在这些堆和资源中进一步指定优先级。

## <a name="programming-residency-management"></a>编程驻留管理

只需创建已提交资源，直到出现内存不足错误，即可创建简单的应用程序。 失败时，应用程序可能销毁其他已提交的资源或 API 对象，使后续的资源创建能够成功。 但是，即使是对于简单应用程序，我们也强烈建议监视负面的预算更改，并在大约处理一帧后销毁未使用的 API 对象。

如果尝试针对适配器体系结构进行优化或整合驻留优先级，驻留管理设计的复杂性将会增大。 以离散方式预算和管理两个离散内存池比仅管理一个要复杂一些，以较大的规模分配固定的优先级可能会在使用模式演变时造成维护负担。 将纹理溢出到系统内存会进一步增大复杂性，因为系统内存中的错误资源可能会严重影响帧速率。 此外，没有任何简单的功能可帮助识别能够受益于更高 GPU 带宽或容忍更低 GPU 带宽的资源。

更复杂的设计需要查询当前适配器的功能。 [**D3D12\_FEATURE\_DATA\_GPU\_VIRTUAL\_ADDRESS\_SUPPORT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_gpu_virtual_address_support)、[**D3D12\_FEATURE\_DATA\_ARCHITECTURE**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_architecture)、[**D3D12\_TILED\_RESOURCES\_TIER**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_tiled_resources_tier) 和 [**D3D12\_RESOURCE\_HEAP\_TIER**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_heap_tier) 中已提供相关信息。

应用程序的多个组成部分最终可能会使用不同的技术。 例如，某些大型纹理和极少执行的代码路径可能使用已提交资源，而许多纹理中可能指定了流属性，因此会使用常规的已定位资源技术。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[**ID3D12Heap**](/windows/desktop/api/D3D12/nn-d3d12-id3d12heap)
</dt> <dt>

[内存管理](memory-management.md)
</dt> </dl>

 

 




