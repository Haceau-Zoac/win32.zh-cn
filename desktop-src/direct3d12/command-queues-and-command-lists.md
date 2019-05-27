---
title: 在 Direct3D 12 中的工作提交
description: 若要提高 Direct3D 应用的 CPU 效率，Direct3D 12 中不再支持与设备关联的即时上下文。
ms.assetid: BE2F46EA-D4A9-47F7-A2D1-6A486DD4DC1A
ms.topic: article
ms.date: 11/15/2018
ms.openlocfilehash: def9f327148489bca5f9f80a804a002b9afcd025
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224377"
---
# <a name="work-submission-in-direct3d-12"></a>在 Direct3D 12 中的工作提交

若要提高 CPU 效率的 Direct3D 应用，自版本 12，Direct3D 不再支持与设备关联的即时上下文。 相反，你的应用程序记录，然后提交*命令列出*，其中包含绘图和资源管理调用。 您可以提交到一个或多个命令队列，管理命令执行这些命令将列出从多个线程。 这一基本更改应用程序以预先计算的重复更高版本使用的呈现工作，从而提高了单线程的效率，它利用多核系统通过呈现工作分散到多个线程。

## <a name="in-this-section"></a>本部分内容

| 主题 | 描述 |
|-|-|
| [设计理念的命令队列和命令列表](design-philosophy-of-command-queues-and-command-lists.md) | 启用重复使用的呈现工作，并且多线程缩放的目标所需如何 Direct3D 应用提交到 GPU 呈现工作基础上的更改。 |
| [创建和记录命令列表和捆绑包](recording-command-lists-and-bundles.md) | 本主题介绍录制命令列表和 Direct3D 12 应用中的捆绑包。 命令列表和捆绑包允许图形处理单元 (GPU) 上绘制的记录或状态更改为更高版本执行的调用的应用。 |
| [执行和同步命令列表](executing-and-synchronizing-command-lists.md) | 在 Microsoft Direct3D 12 中，早期版本的即时模式不再存在。 相反，应用程序创建命令列表和捆绑包，然后记录 GPU 命令集。 命令队列用于提交要执行的命令列表。 此模型允许开发人员能够更好地控制有效地使用 GPU 和 CPU。 |
| [管理图形管道 Direct3D 12 中的状态](managing-graphics-pipeline-state-in-direct3d-12.md) | 本主题介绍如何在 Direct3D 12 中设置的图形管道状态。 |
| [使用资源屏障来同步在 Direct3D 12 中的资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md) | 若要减少总体 CPU 使用率，并启用驱动程序的多线程处理和预处理，Direct3D 12 移动从图形驱动程序的每个资源状态管理对应用程序的责任。 |
| [管道和 Direct3D 12 的着色器](pipelines-and-shaders-with-directx-12.md) | Direct3D 12 可编程管道会大大提高呈现性能相比上一代图形编程接口。 |
| [明暗度变量速率 (VRS)](vrs.md) | 明暗度变量速率&mdash;或粗略像素阴影&mdash;是一种机制，可让你分配费率因呈现图像的呈现性能/电源。 |
| [呈现阶段](direct3d-12-render-passes.md) | 呈现阶段功能可帮助您通过减少与关闭芯片内存; 内存流量，提高 GPU 效率的呈现器通过启用应用程序以更好地识别资源呈现排序要求和数据依赖项执行此操作。 |

## <a name="related-topics"></a>相关主题

* [Direct3D 12 编程指南](directx-12-programming-guide.md)
