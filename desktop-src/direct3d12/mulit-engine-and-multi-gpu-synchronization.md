---
title: 多引擎和多适配器同步
description: 概述并列出了多引擎 （3D、 计算和复制引擎） 和多适配器相关的 Api。
ms.assetid: D81BE0E6-D7A4-4EB4-8267-2C97E4216A9D
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 51ccfaa246ea96d765f55cd84e76aecb46dc7892
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224323"
---
# <a name="multi-engine-and-multi-adapter-synchronization"></a>多引擎和多适配器同步

概述并列出了多引擎 （3D、 计算和复制引擎） 和多适配器相关的 Api。

## <a name="in-this-section"></a>本部分内容



| 主题                                                                             | 描述                                                                                                                                                                                                                                                                                                                                                                                         |
|-----------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [同步和多引擎](user-mode-heap-synchronization.md)<br/> | 大多数现代 Gpu 包含多个独立引擎提供特定的功能的。 有许多的一个或多个专用的复制引擎和计算引擎，通常不同于 3D 引擎。 每个这些引擎可以彼此并行执行命令。 Direct3D 12 提供精细访问权限的三维效果、 计算和复制引擎，使用队列和命令列表。<br/> |
| [Multi-Adapter](multi-engine.md)<br/>                                      | 介绍 D3D12 中的多引擎适配器系统，涵盖应用程序显式面向多个 GPU 适配器的方案，以及其中的驱动程序隐式使用多个 GPU 适配器代表应用程序方案的支持。<br/>                                                                                                                                                |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 的编程指南](directx-12-programming-guide.md)
</dt> <dt>

[内存管理](memory-management.md)
</dt> </dl>

 

 





