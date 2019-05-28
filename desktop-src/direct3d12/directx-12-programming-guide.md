---
title: Direct3D 12 编程指南
description: Direct3D 12 中提供的 API 和平台，可让应用可以充分利用图形和 Pc 配备一个或多个 Direct3D 12 兼容 Gpu 的计算功能。
ms.assetid: 16F78A6B-74C4-4ED1-809F-FE6DE157F368
ms.custom: 19H1
ms.topic: article
ms.date: 02/12/2019
ms.openlocfilehash: 9d26119a4d189bc5e8bb9dae5770240caf1821e6
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224266"
---
# <a name="direct3d-12-programming-guide"></a>Direct3D 12 编程指南

Direct3D 12 中提供的 API 和平台，可让应用可以充分利用图形和 Pc 配备一个或多个 Direct3D 12 兼容 Gpu 的计算功能。

## <a name="in-this-section"></a>本部分内容

| 主题 | 描述 |
|-|-|
| [Direct3D 12 是什么？](what-is-directx-12-.md) | DirectX 12 引入了下一版本的 Direct3D，DirectX 的核心的 3D 图形 API。 此版本的 Direct3D 是更快、 效率高于任何以前的版本。 Direct3D 12 启用更丰富的场景，多个对象、 更复杂的影响，以及充分利用所有最新 GPU 硬件。  |
| [新版本](new-releases.md) | 介绍了最新 SDK 版本提供的最重要的新文档。 |
| [了解 Direct3D 12](directx-12-getting-started.md) | 若要编写适用于 Windows 10 和 Windows 10 移动版的 3D 游戏和应用程序，必须了解 Direct3D 12 技术，以及如何准备您的游戏和应用中使用基础的知识。 |
| [在 Direct3D 12 中的工作提交](command-queues-and-command-lists.md) | 若要提高 Direct3D 应用的 CPU 效率，Direct3D 12 中不再支持与设备关联的即时上下文。 相反，应用程序记录，并提交*命令列出*，其中包含绘图和资源管理调用。 这些命令将列出可从多个线程提交到一个或多个命令队列，管理命令的执行。 这一基本更改应用程序可以预先计算呈现工作的更高版本重复使用，从而提高单线程效率，它利用多核系统通过呈现工作分散到多个线程。  |
| [在 Direct3D 12 资源绑定](resource-binding.md) | 绑定是链接到图形管道的着色器资源对象的过程。  |
| [在 Direct3D 12 中的内存管理](memory-management.md) | 将移动到 D3D12 涉及正确的同步和管理的物理内存。 管理内存驻留表示必须完成更多的同步。 本部分介绍内存管理策略，以及子堆中的分配和缓冲区。  |
| [多引擎和多适配器同步](multi-engine-and-multi-gpu-synchronization.md) | 概述并列出了多引擎 （3D、 计算和复制引擎） 和多适配器相关的 Api。 |
| [呈现](rendering.md) | 本部分包含有关 Direct3D 12 （和 Direct3D 11.3） 到新的呈现功能的信息。 |
| [计数器、 查询和性能度量](performance-measurement.md) | 以下部分介绍在性能测试和改进，例如查询、 计数器、 时间安排和断言而中使用的功能。 |
| [使用 Direct3D 11，Direct3D 10 和 Direct2D](direct3d-12-interop.md) | 本部分介绍了与早期版本的 Direct3D 和 Direct2D，Direct3D 11on12 API，并移植到 Direct3D 12 从 Direct3D 11 的指导原则的互操作技术。 |
| [工作示例](working-samples.md) | 可供下载，其中显示了大量的 Direct3D 12 的功能的使用情况的工作示例。 |
| [D3D12 代码演练](d3d12-code-walk-throughs.md) | 本部分提供示例方案的代码。 许多演练提供有关哪些编码是需要添加到基本示例，以避免重复每个方案的基本组件代码的详细信息。 |
| [调试和诊断与 Direct3D 12](understanding-the-d3d12-debug-layer.md) | 包含的主题介绍如何充分利用 Direct3D 12 调试层与基于 GPU 的验证 (GBV)，以及如何使用设备中删除扩展数据 （通过）。 |

## <a name="related-topics"></a>相关主题

* [Direct3D 12 图形](direct3d-12-graphics.md)
* [Direct3D 12 的引用](direct3d-12-reference.md)
* [DirectX 高级学习视频教程](https://www.youtube.com/channel/UCiaX2B8XiXR70jaN7NK-FpA)
