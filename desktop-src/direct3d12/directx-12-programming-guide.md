---
title: Direct3D 12 编程指南
description: 如果电脑配备有一个或多个与 Direct3D 12 兼容的 GPU，则应用可通过 Direct3D 12 提供的 API 和平台使用该电脑的图形和计算功能。
ms.assetid: 16F78A6B-74C4-4ED1-809F-FE6DE157F368
ms.custom: 19H1
ms.localizationpriority: high
ms.topic: article
ms.date: 04/19/2019
ms.openlocfilehash: e69563e0db09142e2f8c0cc2cf67bb313dc9e218
ms.sourcegitcommit: d6102d9e2b26368142fe5b006c65acb50c98be65
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/26/2019
ms.locfileid: "71306446"
---
# <a name="direct3d-12-programming-guide"></a>Direct3D 12 编程指南

如果电脑配备有一个或多个与 Direct3D 12 兼容的 GPU，则应用可通过 Direct3D 12 提供的 API 和平台使用该电脑的图形和计算功能。

## <a name="in-this-section"></a>本节内容

| 主题 | 描述 |
|-|-|
| [什么是 Direct3D 12？](what-is-directx-12-.md) | DirectX 12 引入了下一版本的 Direct3D，它是 DirectX 的核心 3D 图形 API。 此版本的 Direct3D 比以前的任何版本都更快更高效。 Direct3D 12 提供更丰富的场景、更多的对象、更复杂的效果，并且能全面利用现代 GPU 硬件。  |
| [最新发布](new-releases.md) | 介绍最新 SDK 发布中最重要的新文档。 |
| [了解 Direct3D 12](directx-12-getting-started.md) | 若要为 Windows 10 和 Windows 10 移动版编写 3D 游戏和应用，则必须了解 Direct3D 12 技术的基础知识，还需了解如何准备以便在游戏和应用中使用。 |
| [Direct3D 12 中的工作提交](command-queues-and-command-lists.md) | 为了提高 Direct3D 应用的 CPU 效率，Direct3D 12 不再支持与设备关联的即时上下文。 相反，应用会记录并提交“命令列表”，其中包含绘图和资源管理调用。 这些命令列表可以从多个线程提交到一个或多个命令队列，命令队列用于管理命令的执行。 这种根本性的改变通过允许应用预先计算渲染工作以供以后重用，从而提高了单线程的效率，并且它通过将渲染工作分散到多个线程来利用多核系统。  |
| [Direct3D 12 中的资源绑定](resource-binding.md) | 绑定是将资源对象链接到图形管道着色器的过程。  |
| [Direct3D 12 中的内存管理](memory-management.md) | 迁移到 D3D12 需对内存驻留进行适当同步和管理。 管理内存驻留意味着必须执行更多同步。 本节介绍内存管理策略，以及堆和缓冲区中的二次分配。  |
| [多适配器系统](multi-engine.md) | 描述在 Direct3D 12 中对安装了多个适配器的系统的支持，涵盖了应用程序显式面向多个 GPU 适配器的情况，以及驱动程序代表你的程序. |
| [多引擎同步](user-mode-heap-synchronization.md) | 本主题讨论如何同步对大多数新式 Gpu 中的多个独立引擎的访问。 |
| [渲染](rendering.md) | 本节介绍 Direct3D 12（和 Direct3D 11.3）中新增的渲染功能。 |
| [计数器、查询和性能度量](performance-measurement.md) | 以下各节描述了用于性能测试和改进的功能，例如查询、计数器、定时和预测。 |
| [使用 Direct3D 11、Direct3D 10 和 Direct2D](direct3d-12-interop.md) | 本节介绍早期版本的 Direct3D 和 Direct2D、Direct3D 11on12 API 以及从 Direct3D 11 到 Direct3D 12 的移植指南中的互操作技术。 |
| [样例](working-samples.md) | 可下载样例，了解 Direct3D 12 的许多功能的用法。 |
| [D3D12 代码演练](d3d12-code-walk-throughs.md) | 本部分介绍了示例方案的代码。 许多演练都详细介绍了以下内容，即添加到基本示例需要采取哪种编码技术以避免为每个场景重复基本组件代码。 |
| [使用 Direct3D 12 进行调试和诊断](understanding-the-d3d12-debug-layer.md) | 包括描述如何利用基于 GPU 验证 (GBV) 的 Direct3D 12 调试层以及如何使用设备删除的扩展数据 (DRED) 的主题。 |

## <a name="related-topics"></a>相关主题

* [Direct3D 12 图形](direct3d-12-graphics.md)
* [Direct3D 12 参考](direct3d-12-reference.md)
* [DirectX 高级学习视频教程](https://www.youtube.com/channel/UCiaX2B8XiXR70jaN7NK-FpA)
