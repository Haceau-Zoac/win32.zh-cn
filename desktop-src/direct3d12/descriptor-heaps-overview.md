---
title: 描述符堆概述
description: 描述符堆包含不属于管道状态对象 (PSO) 的许多对象类型，例如，着色器资源视图 (SRV)、无序访问视图 (UAV)、常量缓冲区视图 (CBV) 和取样器。
ms.assetid: 14561E77-44E0-4A58-8456-F40D59ECA175
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: a8bf720ebb71d016457fa4383a8d33aa62e2eee4
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006264"
---
# <a name="descriptor-heaps-overview"></a>描述符堆概述

描述符堆包含不属于管道状态对象 (PSO) 的许多对象类型，例如，着色器资源视图 (SRV)、无序访问视图 (UAV)、常量缓冲区视图 (CBV) 和取样器。

-   [描述符堆的用途](#the-purpose-of-descriptor-heaps)
-   [同步](#synchronization)
-   [Binding](#binding)
-   [切换堆](#switching-heaps)
-   [捆绑](#bundles)
-   [管理](#management)
-   [相关主题](#related-topics)

## <a name="the-purpose-of-descriptor-heaps"></a>描述符堆的用途

描述符堆的主要用途是包含所需的批量内存分配，用于存储着色器在尽可能大的渲染窗口（最好是在整个渲染帧或更大的窗口）中引用的对象类型的描述符规范。 如果应用程序正在切换管道通过 API 快速看到的纹理，则必须有描述符堆空间来为每个所需状态集快速定义描述符表。 例如，应用程序如果在另一个对象中再次使用资源，可以选择重复使用定义，或者仅在切换各种对象类型时按顺序分配堆空间。

描述符堆还允许各个软件组件管理彼此分开的描述符存储。

CPU 可以看到所有堆。 应用程序还可以请求描述符堆应具有哪些 CPU 访问属性（如果有）– 写入组合、回写等。 应用可以使用任何所需属性创建所需数量的描述符堆。 应用始终可以选择创建仅用于暂存目的且大小不受约束的描述符堆，并根据需要复制到用于渲染的描述符堆。

相同的描述符堆中存在一些限制。 CBV、UAV 和 SRV 条目可以位于相同的描述符堆中。 但是，取样器条目不能共享带有 CBV、UAV 或 SRV 条目的堆。 通常，有两组描述符堆，一组用于公共资源，另一组用于取样器。

Direct3D 12 使用描述符堆反映了 GPU 硬件所执行的大多数操作，即要求描述符仅位于描述符堆中，或只是在使用这些堆时需要较少的寻址位。 Direct3D 12 确实需要使用描述符堆，没有选项可以将描述符放置在内存中的任意位置。

描述符堆只能由 CPU 立即编辑，没有选项可以由 GPU 编辑描述符堆。

## <a name="synchronization"></a>同步

描述符堆内容可以在记录引用它的命令列表之前、期间和之后更改。 但是，描述符无法在提交等待执行的命令列表可能会引用该位置时更改，因为这可能会调用争用条件。

## <a name="binding"></a>绑定

任何时候最多可以绑定一个 CBV/SRV/UAV 组合堆和一个取样器堆。 这些堆在图形管道和计算管道之间共享（如其 PSO 中所述）。

## <a name="switching-heaps"></a>切换堆

可以接受应用程序使用 [**SetDescriptorHeaps**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps) 和 [**Reset**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-reset) API 在相同或不同的命令列表中切换堆。 在某些硬件上，此操作开销巨大，需要 GPU 停滞才能刷新依赖于当前绑定的描述符堆的所有工作。 因此，如果必须更改描述符堆，则在 GPU 工作负荷相对较轻时，应用程序应尝试执行此操作，这样可能会限制对命令列表开头的更改。

## <a name="bundles"></a>捆绑

使用捆绑，只能有一个对 [**SetDescriptorHeaps**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps) 方法的调用，描述符堆设置必须与调用捆绑的命令列表完全匹配。 如果捆绑不会更改描述符表，则无需设置描述符堆。

有关不能与捆绑一起使用的 API 调用的列表，请参阅[创建和记录命令列表与捆绑](recording-command-lists-and-bundles.md)。

## <a name="management"></a>管理

若要在场景中呈现所有对象，将需要多个描述符，并且可以遵循一些不同的管理策略。

最基本的策略是按照下一个绘制调用的所有要求填充描述符堆的全新区域。 因此，就在对命令列表发出绘制调用之前，描述符表指针将设置为全新填充的表的开头。 优点是无需记录任何特定描述符在堆中的位置。

此策略的缺点是描述符堆中可能有很多重复的描述符，尤其是在呈现非常相似的场景，以及描述符堆空间快要用完时。 在 GPU 上呈现的单独描述符堆和 CPU 记录的单独描述符堆可能是避免冲突所必需的。 或者，可以使用子分配系统。

此外，可以通过谨慎使用重叠的描述符表（从一个绘制调用到下一个绘制调用）来进一步优化基本系统，以便仅添加所需的新描述符。

比基本策略更有效的策略是使用部分场景已知的对象（或材料）所需的描述符预填充描述符堆。 此处的构想是仅当绘制时才需要设置描述符表，因为已提前填充描述符堆。

预填充策略的一种变体是将描述符堆视为一个大数组，包含固定已知位置中所需的所有描述符。 然后，绘制调用只需接收一组常量，这些常量是需要使用的描述符的索引数组。

进一步优化是为了确保根常量和根描述符包含最频繁更改的常量，而不是将常量放置在描述符堆中。 对于大多数硬件，这是处理常量的有效方法。

实际上，图形引擎可能会在不同情况下使用不同的策略，合并每个策略的元素以满足特定的绘制要求。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符堆](descriptor-heaps.md)
</dt> </dl>

 

 




