---
title: 从 Direct3D 11 到 Direct3D 12 的重要更改
description: Direct3D 12 与 Direct3D 11 编程模型之间存在显著的差异。 在 Direct3D 12 中，应用比以往任何时候更接近硬件。
ms.assetid: CE5066C9-7EA6-437D-9EB0-AACFB6CFAD9E
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: d7a3c6ff81c9dd65e4e5d81ed262acb4a2bfeced
ms.sourcegitcommit: 27a9dfa3ef68240fbf09f1c64dff7b2232874ef4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66725486"
---
# <a name="important-changes-from-direct3d-11-to-direct3d-12"></a>从 Direct3D 11 到 Direct3D 12 的重要更改

Direct3D 12 与 Direct3D 11 编程模型之间存在显著的差异。 在 Direct3D 12 中，应用比以往任何时候更接近硬件。 由于更接近硬件，Direct3D 12 速度更快且更高效。 但是，Direct3D 12 中应用速度更快且效率更高的弊端是，你需要负责完成的任务比在 Direct3D 11 中更多。

-   [显式同步](#explicit-synchronization)
-   [物理内存驻留管理](#physical-memory-residency-management)
-   [管道状态对象](#pipeline-state-objects)
-   [命令列表和捆绑](#command-lists-and-bundles)
-   [描述符堆和表](#descriptor-heaps-and-tables)
-   [从 Direct3D 11 移植](#porting-from-direct3d-11)
-   [相关主题](#related-topics)

Direct3D 12 回归到了低级编程；它引入了以下这些新的功能，可让你更好地控制游戏和应用的图形元素：用于表示管道总体状态的对象、用于提交工作的命令列表和捆绑，以及用于访问资源的描述符堆和表。

Direct3D 12 中应用的速度和效率更高，但你要负责完成的任务比在 Direct3D 11 中更多。

## <a name="explicit-synchronization"></a>显式同步

-   在 Direct3D 12 中，CPU-GPU 同步现在明确由应用负责，而不再像 Direct3D 11 中那样由运行时隐式执行。 这一事实也意味着，Direct3D 12 不会自动执行管道风险检查，因此，此工作同样由应用负责。
-   在 Direct3D 12 中，应用负责通过管道传输数据更新。 也就是说，Direct3D 11 中的“映射/锁定-丢弃”模式在 Direct3D 12 中必须手动执行。 在 Direct3D 11 中，如果当你结合 [**D3D11\_MAP\_WRITE\_DISCARD**](https://docs.microsoft.com/windows/desktop/api/d3d11/ne-d3d11-d3d11_map) 调用 [**ID3D11DeviceContext::Map**](https://docs.microsoft.com/windows/desktop/api/d3d11/nf-d3d11-id3d11devicecontext-map) 时 GPU 仍在使用缓冲区，则运行时将返回指向内存新区域（而不是旧缓冲区数据）的指针。 因此，当应用在新缓冲区中放置数据时，GPU 可继续使用旧数据。 在应用中无需进行额外的内存管理；当 GPU 用完旧缓冲区时，系统会自动重复使用或销毁旧缓冲区。
-   在 Direct3D 12 中，所有动态更新（包括常量缓冲区、动态顶点缓冲区、动态纹理等）由应用显式控制。 这些动态更新包括任何所需的 GPU 围栏或缓冲。 应用负责使内存保持可用，直到不再需要内存。
-   Direct3D 12 仅在接口的生存期内使用 COM 式的引用计数（通过使用与设备生存期关联的 Direct3D 弱引用模型）。 应用独自负责所有资源和描述内存生存期的适当持续时间，它们不会进行引用计数。 Direct3D 11 也使用引用计数来管理接口依赖项的生存期。

## <a name="physical-memory-residency-management"></a>物理内存驻留管理

Direct3D 12 应用程序必须防止多个队列、多个适配器和 CPU 线程之间出现争用状态。 D3D12 不再同步 CPU 和 GPU，也不支持用于资源重命名或多重缓冲的便利机制。 在其他处理单元用完内存之前，必须使用围栏来避免多个处理单元过度写入内存。

当 GPU 读取数据时，Direct3D 12 应用程序必须确保数据驻留在内存中。 在创建对象期间，每个对象使用的内存已设置为常驻性的。 调用这些方法的应用程序必须使用围栏来确保 GPU 不会访问已逐出的对象。

资源屏障是所需的另一种同步，用于以极高的粒度级同步资源和子资源转换。

请参阅 [Direct3D 12 中的内存管理](memory-management.md)。

## <a name="pipeline-state-objects"></a>管道状态对象

Direct3D 11 允许通过大量独立对象来操纵管道状态。 例如，输入汇编器状态、像素着色器状态、光栅器状态和输出合并器状态都可以独立修改。 此设计提供图形管道的方便且相对高级的表示形式，但不会利用新式硬件的功能，主要是因为各种状态通常是独立的。 例如，许多 GPU 会将像素着色器和输出合并器状态合并成单一硬件表示形式。 但由于 Direct3D 11 API 允许单独设置这些管道阶段，在确认状态（此时绘制时间尚未结束）之前，显示驱动程序无法解决管道状态的问题。 此方案会延迟硬件状态设置，这意味着会额外增大开销，并减少每帧的最大绘制调用次数。

Direct3D 12 将大部分管道状态统一为在创建时即已确认的不可变管道状态对象 (PSO)，并以此达成了此方案。 然后，硬件和驱动程序可立即将 PSO 转换为执行 GPU 工作所需的任何硬件本机指令和状态。 你仍可以动态更改使用的 PSO，但若要这样做，硬件只需将少量的预先计算状态直接复制到硬件寄存器，而无需匆忙计算硬件状态。 使用 PSO 可以大幅降低绘制调用开销，并且可以大幅增加每帧的绘制调用次数。 有关 PSO 的详细信息，请参阅[在 Direct3D 12 中管理图形管道状态](managing-graphics-pipeline-state-in-direct3d-12.md)。

## <a name="command-lists-and-bundles"></a>命令列表和捆绑

在 Direct3D 11 中，所有工作提交都是通过[中间上下文](https://docs.microsoft.com/windows/desktop/direct3d11/overviews-direct3d-11-render-multi-thread-render)（表示进入 GPU 的单一命令流）完成的。 为了实现多线程缩放，游戏还可以使用[延迟上下文](https://docs.microsoft.com/windows/desktop/direct3d11/overviews-direct3d-11-render-multi-thread-render)。 Direct3D 11 中的延迟上下文不能完美映射到硬件，因此可在其中完成的工作量相对较小。

Direct3D 12 中为工作提交引入了一个基于命令列表的新模型，这些命令列表包含在 GPU 上执行特定工作负荷所需的整个信息。 每个新命令列表包含要使用的 PSO、所需的纹理和缓冲区资源、所有绘制调用的参数等信息。 由于每个命令列表是独立性的且不继承任何状态，因此，驱动程序可以提前以自由线程的方式预先计算全部所需的 GPU 命令。 所需的唯一串行进程是通过命令队列将命令列表最终提交到 GPU。

除命令列表以外，Direct3D 12 还引入了另一个工作预先计算级别：捆绑。  与完全独立的并且通常是结构化的、仅提交一次且会被丢弃的命令列表不同，捆绑提供允许重复使用的状态继承形式。 例如，如果某个游戏想要使用两种不同的纹理绘制两个人物模式，一种方法是使用两组相同的绘制调用来记录某个命令列表。 另一种方法是“记录”一个捆绑来绘制单个人物模型，然后使用不同的资源在命令列表中“播放”该捆绑两次。 对于后一种情况，显示驱动程序只需计算相应的指令一次，创建命令列表实质上相当于两次低开销的函数调用。

有关命令列表和捆绑的详细信息，请参阅 [Direct3D 12 中的工作提交](command-queues-and-command-lists.md)。

## <a name="descriptor-heaps-and-tables"></a>描述符堆和表

Direct3D 11 中的资源绑定是高度抽象化的且非常方便，但会造成许多新式硬件功能的利用不足。 在 Direct3D 11 中，游戏会创建资源的视图对象，然后在管道中的不同着色器阶段将这些视图绑定到多个槽。   而着色器又会从绘制时固定的这些显式绑定槽读取数据。 此模型意味着，每当游戏使用不同的资源进行绘制时，都必须将不同的视图重新绑定到不同的槽，并再次调用绘制。 这种情况也意味着可以通过充分利用新式硬件功能来消除开销。

Direct3D 12 更改了绑定模型，以匹配新式硬件并显著提高性能。 Direct3D 12 不需要独立的资源视图和显式映射到槽，它会一个描述符堆，让游戏在其中创建各种资源视图。 此方案为 GPU 提供一种机制，让它提前将硬件本机资源描述（描述符）直接写入内存。 若要声明管道要对特定的绘制调用使用哪些资源，游戏需指定一个或多个描述符表，表示完整描述符堆的子范围。 由于描述符堆中已填充相应的硬件特定描述符数据，更改描述符表是开销极低的操作。

除了通过描述符堆和表提高性能以外，Direct3D 12 还允许在着色器中动态编制资源的索引，因此提供了前所未有的灵活性，并使新的渲染技术成为可能。 例如，新式延迟渲染引擎通常会将某种类型的材料或对象标识符编码成中间 g-buffer。 在 Direct3D 11 中，这些引擎必须小心避免使用过多的材料，因为在一个 g-buffer 中包含过多的材料可能会明显减慢最终的渲染处理。 使用可动态编制索引的资源时，可以快速实现包含 1000 个材料的场景，就如同场景中只包含 10 个材料一样。

有关描述符堆和表的详细信息，请参阅[资源绑定](resource-binding.md)和[与 Direct3D 11 中的绑定模型的区别](binding-model.md)。

## <a name="porting-from-direct3d-11"></a>从 Direct3D 11 移植

从 Direct3D 11 移植是一个复杂的过程，具体请参阅[从 Direct3D 11 移植到 Direct3D 12](porting-from-direct3d-11-to-direct3d-12.md)。 另请参阅[使用 Direct3D 11、Direct3D 10 和 Direct2D](direct3d-12-interop.md) 中的选项范围。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> </dl>

 

 




