---
title: 直接机器学习 (DirectML)
description: TBD
ms.custom: 19H1
ms.topic: article
ms.date: 02/01/2019
ms.openlocfilehash: 9ab5233bc9520b2575914c28b968da929c965c89
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223963"
---
# <a name="direct-machine-learning-directml"></a>直接机器学习 (DirectML)

直接机器学习 (DirectML) 是用于机器学习的低级别 API。 它具有熟悉 (本机C++，nano COM) 编程接口和 DirectX 12 的样式中的工作流。 您可以将机器学习集成到您的游戏、 引擎、 中间件、 后端或其他应用程序的推断工作负荷。 DirectML 受所有 DirectX 12 兼容硬件。

DirectML 是在 Windows 10，版本 1903，和 Windows SDK 的相应版本中引入的。

## <a name="in-this-section"></a>本部分内容

| 主题 | 描述 |
|-|-|
| [DirectML 简介](dml-intro.md) | 直接机器学习 (DirectML) 是用于机器学习 (ML) 的低级别 API。 |
| [DirectML 中的绑定](dml-binding.md) | 在 DirectML，*绑定*是指资源连接到要在初始化期间使用 GPU 的管道和你的机器学习运算符的执行。 这些资源可以是输入和输出 tensors，以及需要的任何临时或永久资源。 |
| [UAV 障碍和资源状态中 DirectML 的障碍](dml-barriers.md) | 描述障碍，以及如何可以与它们配合中 DirectML 正确性的好处。 |
| [使用进步表示填充和内存布局](dml-strides.md) | 由称为属性描述 DirectML tensors*大小*并*进步*的 tensor。 |
| [资源生存期和同步](dml-resource-lifetime.md) | 为避免出现未定义的行为，DirectML 应用程序必须正确管理对象生存期和 CPU 和 GPU 之间的同步。 |
| [使用 DirectML 调试层](dml-debug-layer.md) | DirectML 调试层是一个可选的开发时间组件，可帮助你在调试 DirectML 代码。 |
| [处理错误和设备-删除](dml-errors.md) | 本主题讨论如何调试 DirectML 设备-删除或其他错误情况。 |
| [DirectML 帮助器函数](dml-helper-functions.md) | 基本 DirectML 帮助程序函数的代码列表。 |
| [DirectML 示例应用程序](dml-min-app.md) | DirectML 示例应用程序，包括最小 DirectML 应用程序的示例的链接。 |

## <a name="related-topics"></a>相关主题

* [Direct3D 12 编程指南](directx-12-programming-guide.md)
