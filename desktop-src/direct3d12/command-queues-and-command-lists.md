---
title: Direct3D 12 中的工作提交
description: 为了提高 Direct3D 应用的 CPU 效率，Direct3D 12 不再支持与设备关联的即时上下文。
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
# <a name="work-submission-in-direct3d-12"></a>Direct3D 12 中的工作提交

为了提高 Direct3D 应用的 CPU 效率，从版本 12 开始，Direct3D 不再支持与设备关联的即时上下文。 相反，应用程序会记录并提交“命令列表”，其中包含绘图和资源管理调用  。 可以将这些命令列表从多个线程提交到一个或多个命令队列，命令队列用于管理命令的执行。 这种根本性的改变通过允许应用程序预先计算渲染工作以供以后重用，从而提高了单线程的效率，并且它通过将渲染工作分散到多个线程来利用多核系统。

## <a name="in-this-section"></a>本部分内容

| 主题 | 描述 |
|-|-|
| [命令队列和命令列表的设计理念](design-philosophy-of-command-queues-and-command-lists.md) | 为了实现渲染工作的重用和多线程缩放，需要对 Direct3D 应用向 GPU 提交渲染工作的方式进行根本性的改变。 |
| [创建和记录命令列表与捆绑包](recording-command-lists-and-bundles.md) | 本主题描述在 Direct3D 12 应用中记录命令列表和捆绑包。 命令列表和捆绑都允许应用记录绘图或状态更改调用，以便稍后在图形处理单元 (GPU) 上执行。 |
| [执行和同步命令列表](executing-and-synchronizing-command-lists.md) | 在 Microsoft Direct3D 12 中，以前版本的即时模式不再存在。 应用将创建命令列表和捆绑，然后记录 GPU 命令集。 命令队列用于提交要执行的命令列表。 该模型可让开发人员更好地控制 GPU 和 CPU 的有效使用。 |
| [在 Direct3D 12 中管理图形管道状态](managing-graphics-pipeline-state-in-direct3d-12.md) | 本主题描述如何在 Direct3D 12 中设置图形管道状态。 |
| [在 Direct3D 12 中使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md) | 为了减少总体 CPU 使用率并启用驱动程序多线程和预处理，Direct3D 12 将按资源状态管理的责任从图形驱动程序转移到应用程序。 |
| [Direct3D 12 的管道和着色器](pipelines-and-shaders-with-directx-12.md) | 相较于上一代图形编程接口，Direct3D 12 可编程管道显著提高了渲染性能。 |
| [可变速率着色 (VRS)](vrs.md) | 可变速率着色 &mdash; 或粗略像素着色 &mdash; 是一种机制，可让你以不同渲染图像的速率分配渲染性能/算力。 |
| [渲染器通道](direct3d-12-render-passes.md) | 渲染器通道功能通过减少与芯片外内存之间的内存流量，帮助渲染器提高 GPU 效率；它通过使应用程序更好地识别资源渲染排序要求和数据依赖项来实现此目的。 |

## <a name="related-topics"></a>相关主题

* [Direct3D 12 编程指南](directx-12-programming-guide.md)
