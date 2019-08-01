---
title: 什么是 Direct3D 12
description: DirectX 12 引入了下一版本的 Direct3D；它是 DirectX 的核心 3D 图形 API。
ms.assetid: 09C586BF-11CE-4392-9BFD-A40B05DD0624
ms.localizationpriority: high
ms.topic: article
ms.date: 11/19/2018
ms.openlocfilehash: b7d7b653397be96ea6c83b736c817408f17a4e65
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296499"
---
# <a name="what-is-direct3d-12"></a>什么是 Direct3D 12？

DirectX 12 引入了下一版本的 Direct3D&mdash;它是 DirectX 的核心 3D 图形 API。 Direct3D 12 比以前的任何版本都更快更高效。 Direct3D 12 提供更丰富的场景、更多的对象、更复杂的效果，并且能全面利用现代 GPU 硬件。

## <a name="how-can-direct3d-12-be-so-much-faster-and-more-efficient"></a>如何使 Direct3D 12 更快更高效？

Direct3D 12 提供的硬件抽象级别比以前的版本低，可显著提高标题（或其他应用程序）的多核 CPU 缩放。 一方面，使用 Direct3D 12，你的标题负责自己的[内存管理](memory-management.md)。 另一方面，使用 Direct3D 12，你的标题和应用程序可通过[命令队列和列表](command-queues-and-command-lists.md)、[描述符表](descriptor-tables.md)和简洁的[管道状态对象](managing-graphics-pipeline-state-in-direct3d-12.md)等功能减少 GPU 开销。

Direct3D 12 和 Direct3D 11.3 引入了呈现管道的一组新功能。

- [传统型光栅化](../direct3d11/conservative-rasterization.md)用于启用可靠的命中检测。
- [立体平铺资源](../direct3d11/volume-tiled-resources.md)用于启用被视为均位于视频内存中的流式处理的三维资源。
- [光栅器有序视图](../direct3d11/volume-tiled-resources.md)用于启用可靠的透明度呈现。
- 设置着色器中的模具引用来启用特殊阴影和其他效果。
- 改进的纹理映射和类型化无序访问视图 (UAV) 加载。

## <a name="how-deeply-should-i-invest-in-direct3d-12"></a>在 Direct3D 12 中的投入程度如何？

Direct3D 12 为图形开发人员提供四个主要好处（与 Direct3D 11 相比）。

- 极大地减少了 CPU 开销。
- 显著减少了电源消耗。
- GPU 效率最多（大约）改进了 20%。
- Windows 10 设备（个人电脑、平板电脑、控制台、移动设备）的跨平台开发。

Direct3D 12 专供高级图形程序员使用。 它需要大量图形专业知识和高级别的微调。 Direct3D 12 旨在充分利用多线程，仔细的 CPU/GPU 同步以及资源从一个目的到另一个目的的转换和重复使用。 这些技术需要相当多的内存级别的编程技能。

Direct3D 12 的另一个优点是其 API 资源占用空间较小。 大约有 200 个函数；约三分之一的函数负责繁重的工作。 这意味着，图形开发人员应能够向自己提供相关信息&mdash;并了解&mdash;完整的 API 集，而无需记住太多的 API 名称。

除了 Direct3D 12 之外，Direct3D 11 仍是一个可行选项。 Direct3D 12 的许多新呈现功能在 [Direct3D 11.3](../direct3d11/direct3d-11-3-features.md) 中可用。 Direct3D 11.3 是一个低级别的图形引擎 API；而 Direct3D 12 的级别更高。

至少有两种方法可供你的开发团队处理 Direct3D 12 标题。

### <a name="use-direct3d-12-exclusively"></a>以独占方式使用 Direct3D 12

对于充分利用 Direct3D 12 的所有好处的项目，应从头开始开发高度自定义的 Direct3D 12 引擎。

如果图形开发人员了解标题中资源的使用和重复使用情况，并且可以通过最大限度地减少上传和复制来充分利用这一点，则可以为这些标题开发和自定义高效引擎。 性能改进可能非常多，释放 CPU 时间来增加绘制调用数量，从而使你的图形更加光彩。

在编程方面的投入非常重要，从一开始就应该考虑项目的调试和检测。 线程处理、同步和其他计时 bug 可能颇具挑战性。

### <a name="use-direct3d-12-in-concert-with-direct3d-11"></a>将 Direct3D 12 与 Direct3D 11 一起使用

短期方法是解决 Direct3D 11 标题中的已知瓶颈。 可以通过使用 [Direct3D 12 互操作和/或 D3D11On12](direct3d-12-interop.md) 技术（使两个 API 版本协同工作）来解决这些问题。 此方法最大限度地减少现有 Direct3D 11 图形引擎所需的更改。 但是，性能增益将仅限于解除 Direct3D 12 代码处理的瓶颈。

## <a name="microsoft-directx-12-and-graphics-education-videos"></a>Microsoft DirectX 12（和图形教育）视频

[面向图形开发人员的增强教育](https://www.youtube.com/channel/UCiaX2B8XiXR70jaN7NK-FpA)。 这些视频涵盖以下主题：呈现模式、移植到 DirectX 12、传统型光栅化、图形工具、Angle、Win2D 以及 GDC、生成等事件。 DirectX 12 技术内容以 DirectX 12  开头。 单击此处直接从 Direct3D 12 功能团队获得提示和技巧。 我们希望帮助你使用最新版本和工具来尽量改进你的游戏！

## <a name="conclusion"></a>结论

Direct3D 12 的关键在于显著的图形引擎性能。 已缩减易于开发、高级别构造和编译器支持功能来实现这一目的。 驱动程序支持和易于调试功能保持为 Direct3D 11 的一部分。

Direct3D 12 是一个新领域。 正等待好追根究底的专家前来探索。
