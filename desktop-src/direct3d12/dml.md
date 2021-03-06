---
title: 直接机器学习 (DirectML)
description: 直接机器学习 (DirectML) 是机器学习的低级 API。 它具有常见的（本机 C++、nano-COM）编程接口和 DirectX 12 样式的工作流。
ms.custom: Windows 10 May 2019 Update
ms.localizationpriority: high
ms.topic: article
ms.date: 04/19/2019
ms.openlocfilehash: 25bbd169a1ad0467ed56135c31c8c2a0b70b57a8
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006026"
---
# <a name="direct-machine-learning-directml"></a>直接机器学习 (DirectML)

直接机器学习 (DirectML) 是机器学习的低级 API。 它具有常见的（本机 C++、nano-COM）编程接口和 DirectX 12 样式的工作流。 可将机器学习推断工作负荷集成到游戏、引擎、中间件、后端或其他应用程序中。 所有与 DirectX 12 兼容的硬件都支持 DirectML。

DirectML 是在 Windows 10 版本 1903 和相应版本的 Windows SDK 中引入的。

## <a name="in-this-section"></a>本节内容

| 主题 | 描述 |
|-|-|
| [DirectML 简介](dml-intro.md) | 直接机器学习 (DirectML) 是机器学习 (ML) 的低级 API。 |
| [DirectML 中的绑定](dml-binding.md) | 在 DirectML 中，绑定是指将资源附加到管道，以供 GPU 在机器学习运算符初始化和执行时使用。 例如，这些资源可以是输入和输出张量，也可以是运算符需要的任何临时或持久资源。 |
| [DirectML 中的 UAV 屏障和资源状态屏障](dml-barriers.md) | 描述屏障的正确性好处，以及在 DirectML 中的使用方式。 |
| [使用步幅来表示填充和内存布局](dml-strides.md) | DirectML 张量用张量的“大小”和“步幅”等属性进行描述。 |
| [资源生存期和同步](dml-resource-lifetime.md) | DirectML 应用程序必须准确地管理对象生存期和 CPU 与 GPU 之间的同步，以避免未定义的行为。 |
| [使用 DirectML 调试层](dml-debug-layer.md) | DirectML 调试层是可选的开发时组件，可帮助调试 DirectML 代码。 |
| [处理错误和设备删除](dml-errors.md) | 本主题论述如何调试 DirectML 设备删除和其他错误条件。 |
| [DirectML 帮助程序函数](dml-helper-functions.md) | 基础 DirectML 帮助程序函数的代码列表。 |
| [DirectML 示例应用程序](dml-min-app.md) | 链接到 DirectML 示例应用程序，其中包括最小的 DirectML 应用程序示例。 |

## <a name="related-topics"></a>相关主题

* [Direct3D 12 编程指南](directx-12-programming-guide.md)
