---
title: Direct3D 12 是什么
description: DirectX 12 引入了下一版本的 Direct3D;3D 图形 API 的 DirectX 的核心。
ms.assetid: 09C586BF-11CE-4392-9BFD-A40B05DD0624
ms.topic: article
ms.date: 11/19/2018
ms.openlocfilehash: 62cd37a62908baac07cb33c2353198044197a2b8
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223960"
---
# <a name="what-is-direct3d-12"></a>什么是 Direct3D 12？

DirectX 12 引入了下一版本的 Direct3D&mdash;DirectX 的核心的 3D 图形 API。 Direct3D 12 是更快、 效率高于任何以前的版本。 Direct3D 12 启用更丰富的场景，多个对象、 更复杂的影响，以及充分利用所有最新 GPU 硬件。

## <a name="how-can-direct3d-12-be-so-much-faster-and-more-efficient"></a>如何 Direct3D 12 中能够更快、 更高效？

Direct3D 12 中是唯一的因为它提供较低级别的硬件抽象比以前的版本，以便您可以显著提高多核 CPU 的缩放你的标题 （或其他应用程序）。 首先，使用 Direct3D 12 中，你的标题负责自己[内存管理](memory-management.md)。 此外，通过使用 Direct3D 12，标题和应用程序受益于开销通过功能减少了 GPU 如[命令队列和列表](command-queues-and-command-lists.md)，[描述符表](descriptor-tables.md)，和简洁[管道状态对象](managing-graphics-pipeline-state-in-direct3d-12.md)。

Direct3D 12 和 Direct3D 11.3 引入了一套用于呈现管道的新功能。

- [传统型光栅化](../direct3d11/conservative-rasterization.md)以启用可靠命中的检测。
- [卷平铺资源](../direct3d11/volume-tiled-resources.md)启用流处理三维资源，就好像所有视频内存中处理。
- [光栅器排序视图](../direct3d11/volume-tiled-resources.md)若要启用可靠的透明度呈现。
- 若要启用特殊隐藏的着色器和其他效果内模具引用，来设置。
- 改进了的纹理映射和类型化的无序的访问视图 (UAV) 加载。

## <a name="how-deeply-should-i-invest-in-direct3d-12"></a>深应投入在 Direct3D 12 中？

Direct3D 12 图形开发人员 （相比于 Direct3D 11） 了四个主要好处。

- 极大地减少 CPU 开销。
- 显著减少了功率消耗。
- 最多 （大约） 20%的改进 GPU 的效率。
- Windows 10 设备 （个人计算机、 平板电脑、 控制台、 移动设备） 的跨平台开发。

Direct3D 12 专为高级的图形编程人员使用。 它调用来执行大量图形专业知识和高级别的微调。 Direct3D 12 旨在充分利用多线程处理，认真进行 CPU/GPU 同步和转换和到另一个目的从资源的重复使用。 这些是需要相当长的内存级别的编程技能的技术。

Direct3D 12 中的另一个优点是其较小的 API 资源占用量。 大约 200 个函数;约三分之一的那些执行所有繁重的任务。 这意味着，图形开发人员应能够告知有关自己称为&mdash;和主&mdash;完整 API 集而无需记住太多的 API 名称。

Direct3D 11 仍是与 Direct3D 12 的可行选项。 许多新的呈现功能的 Direct3D 12 中有[Direct3D 11.3](../direct3d11/direct3d-11-3-features.md)。 Direct3D 11.3 是一种低级别图形引擎 API;和 Direct3D 12 甚至深入。

有您的开发团队可以接触 Direct3D 12 标题的至少两种方法。

### <a name="use-direct3d-12-exclusively"></a>以独占方式使用 Direct3D 12

对于 ultimate 利用 Direct3D 12 的好处的所有项目，您应向上开发高度自定义的 Direct3D 12 引擎从零开始。

如果您，作为图形开发人员，了解使用和重复使用的资源中应用标题中，以及您可以通过尽量减少上传和复制，则可以开发和自定义这些标题的高效引擎充分利用的。 性能提升可能非常相当大，释放 CPU 时间，以增加的绘图调用的数量并因此将多个群集添加到图形。

在编程方面的投入非常重要，且调试和检测的项目从一开始应考虑。 很难线程处理、 同步和其他计时 bug。

### <a name="use-direct3d-12-in-concert-with-direct3d-11"></a>Direct3D 11 配合使用 Direct3D 12

期限较短的方法是为已知在 Direct3D 11 标题中的瓶颈的地址。 您可以解决那些通过使用[Direct3D 12 互操作和/或 D3D11On12](direct3d-12-interop.md)启用两个的 API 版本，能够协同工作的技术。 这种方法最小化现有 Direct3D 11 图形引擎所需的更改。 但是，性能提升将限于 Direct3D 12 代码解决瓶颈的缓解办法。

## <a name="microsoft-directx-12-and-graphics-education-videos"></a>Microsoft DirectX 12 （和图形教育） 视频

[增强的图形开发人员的教育](https://www.youtube.com/channel/UCiaX2B8XiXR70jaN7NK-FpA)。 这些视频涵盖了移植到 DirectX 12、 传统型光栅化、 图形工具、 角度、 Win2D，和事件，例如 GDC、 生成和的详细信息的演示文稿模式等主题。 DirectX 12 的技术内容开头，但*DirectX 12*。 通过此处了解提示和技巧，直接从 Direct3D 12 功能团队。 我们希望帮助你使用我们最新版本和工具以使它可以是的最佳的您的游戏 ！

## <a name="conclusion"></a>结论

Direct3D 12 是所有有关显著图形引擎性能。 易于开发、 较高的层次结构和编译器支持缩放回要启用此功能。 驱动程序支持和易用性调试保持等同于 Direct3D 11。

Direct3D 12 是新领域。 正在等待好学的专家，随时可以并浏览的区域。
