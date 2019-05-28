---
title: 从 Direct3D 的重要更改到 Direct3D 11 12
description: Direct3D 12 表示从 Direct3D 11 编程模型大不同之处。 Direct3D 12 可以让应用获取比以往任何时候更接近于硬件。
ms.assetid: CE5066C9-7EA6-437D-9EB0-AACFB6CFAD9E
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: d50aa09bc5951a2fe5ad60f48b697eae4877da7a
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223816"
---
# <a name="important-changes-from-direct3d-11-to-direct3d-12"></a>从 Direct3D 的重要更改到 Direct3D 11 12

Direct3D 12 表示从 Direct3D 11 编程模型大不同之处。 Direct3D 12 可以让应用获取比以往任何时候更接近于硬件。 通过成为更接近于硬件，Direct3D 12 是更快、 更高效。 但是，应用程序具有更高的速度和效率与 Direct3D 12 的弊端是你负责比以前使用 Direct3D 11 的更多任务。

-   [显式同步](#explicit-synchronization)
-   [物理内存驻留管理](#physical-memory-residency-management)
-   [管道状态对象](#pipeline-state-objects)
-   [命令列表和捆绑包](#command-lists-and-bundles)
-   [描述符堆和表](#descriptor-heaps-and-tables)
-   [从 Direct3D 移植 11](#porting-from-direct3d-11)
-   [相关的主题](#related-topics)

Direct3D 12 是返回到低级编程;这样可以更好地控制游戏和应用程序的图形元素通过引入这些新功能： 对象来表示管道、 列表和捆绑包以处理提交和描述符堆的命令和表来访问资源的总体状态。

速度和效率与 Direct3D 12 中，增加了您的应用程序，但你有责任比以前使用 Direct3D 11 的更多任务。

## <a name="explicit-synchronization"></a>显式同步

-   在 Direct3D 12 中，CPU GPU 同步现在是显式应用的责任，因此不再隐式执行由运行时，因为它是在 Direct3D 11 中。 这一事实也意味着，没有自动检查执行管道危险的 Direct3D 12 中，这同样是应用程序负责。
-   在 Direct3D 12 中，应用负责流水线处理的数据更新。 也就是说，Direct3D 11 中的"锁定/映射-丢弃"模式必须手动执行在 Direct3D 12 中。 在 Direct3D 11 中，如果 GPU 在调用时仍然使用缓冲区[ **ID3D11DeviceContext::Map** ](https://msdn.microsoft.com/library/windows/desktop/ff476457)与[ **D3D11\_映射\_写\_放弃**](https://msdn.microsoft.com/library/windows/desktop/ff476181#d3d11-map-write-discard)，运行时返回一个指向内存而不是旧的缓冲区数据的新区域。 这样，若要继续使用旧数据，而该应用将数据放在新的缓冲区中的 GPU。 没有更多的内存管理被必需的应用程序中旧的缓冲区是重复使用或与之完成 GPU 时自动销毁。
-   在 Direct3D 12 中，通过应用程序进行显式控制 （包括常量缓冲区、 动态顶点缓冲区、 动态纹理等） 的所有动态更新。 这些动态更新包括任何所需的 GPU 界定或缓冲。 应用负责直到不再需要保留的可用内存。
-   Direct3D 12 中使用 COM 样式引用计数仅为接口的生存期 （通过使用 Direct3D 取决于设备的生存期的弱引用模型）。 所有资源和描述内存生存期是负责任地的应用程序以正确的持续时间，维护的唯一且不引用计数。 Direct3D 11 使用引用计数来管理接口的依赖项的生存期。

## <a name="physical-memory-residency-management"></a>物理内存驻留管理

Direct3D 12 应用程序必须阻止多个队列、 多个适配器和 CPU 线程之间的争用条件。 D3D12 不再同步 CPU 和 GPU，也不支持资源重命名或多缓冲的方便的机制。 必须使用界定，以避免覆盖内存另一个处理单元完成使用它之前的多个处理单元。

Direct3D 12 应用程序必须确保数据是驻留在内存中，而在 GPU 上读取。 对象的创建过程中使用的每个对象的内存是进行驻留。 调用这些方法的应用程序必须使用界定来确保 GPU 不会访问已逐出的对象。

资源障碍是另一种同步需要用来同步以非常高粒度级别的资源和子资源转换。

请参阅[内存管理在 Direct3D 12](memory-management.md)。

## <a name="pipeline-state-objects"></a>管道状态对象

Direct3D 11 允许在大量独立的对象通过管道状态操作。 例如，输入装配器状态、 像素着色器状态、 光栅器状态和输出合并器状态全都可以独立地修改。 此设计提供了图形管道的方便、 相对较高层次的表示形式，但它不会利用现代硬件的功能，主要是因为各种状态通常是独立。 例如，多个 Gpu 将像素着色器和输出合并器状态合并为单个硬件表示形式。 但由于 Direct3D 11 API 允许单独设置这些管道阶段，直到状态确定，这并不到绘图时，显示器驱动程序无法解析管道状态的问题。 此方案会延迟硬件状态设置，这意味着额外的开销和较少的最大值绘制每个框架调用。

Direct3D 12 通过统一的大部分管道状态为不可变的管道状态对象 (Pso)，在创建时已完成了解决了此方案。 硬件和驱动程序可以然后立即转换为 PSO 的任何硬件本机指令和状态所需执行 GPU 的工作。 你可以仍动态更改该 PSO 正在使用中，但为此，请在硬件只需少量的预先计算的状态将直接复制到硬件寄存器，而不是计算动态的硬件状态。 通过使用 Pso，绘图调用开销会降低大的差异，并且许多的多个绘图调用可以在每一帧。 Pso 详细信息，请参阅[管理图形管道 Direct3D 12 中的状态](managing-graphics-pipeline-state-in-direct3d-12.md)。

## <a name="command-lists-and-bundles"></a>命令列表和捆绑包

在 Direct3D 11 中，所有工作提交都是通过[即时上下文](https://msdn.microsoft.com/library/windows/desktop/ff476892#immediate)，它表示一个流命令，请转到 GPU。 若要实现多线程缩放，游戏还拥有[延迟上下文](https://msdn.microsoft.com/library/windows/desktop/ff476892#deferred)提供给他们。 Direct3D 11 中的延迟的环境不完全映射到硬件，因此，相对较小的工作可以在它们中完成。

Direct3D 12 中引入了针对工作提交基于包含在 GPU 上执行特定工作负荷所需的信息的完整的命令列表的新模型。 每个新的命令列表包含要使用的 PSO，例如纹理和缓冲区的资源所需的信息，并为所有参数绘图调用。 由于每个命令列表是自包含且继承没有状态，该驱动程序可以预先计算所有必要的 GPU 命令前期和自由线程的方式。 必要仅串行的过程是通过命令队列 GPU 向最终提交的命令列表。

除了命令列表，Direct3D 12 中还引入了一个第二个级别的工作预先计算：*捆绑包*。 与不同的命令列表，这些是完全独立且通常构造的提交一次，并被丢弃，捆绑包提供允许重复使用的状态继承的窗体。 例如，如果游戏想要绘制以下两种不同的纹理的字符模式，一种方法是记录具有相同的绘图调用的两个集的命令列表。 但另一种方法是将单个字符模型，然后绘制的"记录"一个捆绑包"播放"捆绑包在使用不同的资源的命令列表中两次。 在后一种情况下，显示器驱动程序仅有一次，计算相应的说明和创建命令列表实质上相当于两个低成本函数调用。

有关命令列表和捆绑包的详细信息，请参阅[Direct3D 12 中的工作提交](command-queues-and-command-lists.md)。

## <a name="descriptor-heaps-and-tables"></a>描述符堆和表

在 Direct3D 11 中的资源绑定是高度抽象的方便，但保留未被充分利用的许多现代硬件功能。 在 Direct3D 11 中创建游戏*视图*对象的资源，然后将这些视图绑定到多个*槽*在管道中的各种着色器阶段。 着色器，进而读取固定为那些显式绑定槽中的数据绘制时间。 此模型意味着每当将绘制一个游戏，使用不同的资源，它必须重新绑定到不同的插槽，不同的视图并调用重新绘制。 这种情况下还表示可以充分利用现代硬件功能消除了开销。

Direct3D 12 更改要与现代硬件匹配的绑定模型，并显著提高性能。 而不是要求独立资源视图和显式映射到槽，Direct3D 12 中提供的游戏在其中创建其各种资源视图描述符堆。 此方案提供 GPU 将直接写入内存前期硬件本机资源说明 （描述） 的机制。 若要声明哪些资源是用于特定的绘图调用的管道，游戏指定一个或多个表示完整的描述符堆的子范围的描述符表。 描述符堆已填入适当的特定于硬件的描述符数据，如更改描述符表是一个极低成本操作。

除了改进了性能提供的描述符堆和表，Direct3D 12 还允许资源进行动态索引在着色器，它提供了前所未有的灵活性并解锁新的呈现技术。 例如，现代延迟的呈现引擎通常编码到中间 g 缓冲区某种类型的材料或对象标识符。 在 Direct3D 11 中，这些引擎必须小心避免使用过多的材料，与包含太多在一个 g-缓冲区可能会明显下降的最终呈现处理。 动态可编制索引的资源，可以只需尽快为一个具有唯一的十个完成使用千位材料场景。

有关详细信息，有关描述符堆和表，请参阅[资源绑定](resource-binding.md)，并[绑定模型中的区别 Direct3D 11](binding-model.md)。

## <a name="porting-from-direct3d-11"></a>从 Direct3D 移植 11

从 Direct3D 移植 11 是一个复杂的过程中所述[从 Direct3D 11 移植到 Direct3D 12](porting-from-direct3d-11-to-direct3d-12.md)。 此外对范围中选项的引用[使用 Direct3D 11、 Direct3D 10 和 Direct2D](direct3d-12-interop.md)。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> </dl>

 

 




