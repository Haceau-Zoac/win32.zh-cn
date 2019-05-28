---
title: 资源绑定
description: 绑定是链接到图形管道的着色器资源对象的过程。
ms.assetid: CB0065EF-4E53-4E16-9F7E-09A261C360FB
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 728de87ec59fb1b3fd19383d560beeeb0fa713a1
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224173"
---
# <a name="resource-binding"></a>资源绑定

绑定是链接到图形管道的着色器资源对象的过程。

## <a name="in-this-section"></a>本部分内容

| 主题 | 描述 |
|-|-|
| [资源绑定概述](resource-binding-flow-of-control.md) | DirectX 12 中的资源绑定的关键是两个的概念*描述符*，*描述符表*，*描述符堆*，和一个*根签名*. |
| [绑定模型中的区别 Direct3D 11](binding-model.md) | DirectX12 绑定背后的主要设计决策之一是将它与其他管理任务。 这会应用来管理某些潜在危险的一些要求。 |
| [描述符](descriptors.md) | 描述符是 D3D12 中的单个资源绑定的主计价单位。 |
| [描述符堆](descriptor-heaps.md) | 描述符堆是一系列连续分配的描述符，每个描述符的一个资源分配。 |
| [描述符表](descriptor-tables.md) | 描述符表在逻辑上就描述符的数组。 |
| [根签名](root-signatures.md) | 根签名定义的资源类型绑定到图形管道。 |
| [查询功能](capability-querying.md) | 你的应用程序可以发现对资源绑定和许多其他功能，通过调用的支持级别[ **ID3D12Device::CheckFeatureSupport**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport)。 |
| [在 HLSL 中绑定的资源](resource-binding-in-hlsl.md) | 本部分介绍使用高级别着色语言 (HLSL) 的某些特定功能[着色器模型 5.1](https://msdn.microsoft.com/library/windows/desktop/dn933277) Direct3D 12。 所有 Direct3D 12 硬件支持着色器模型 5.1，因此支持此模型不依赖于硬件功能级别是什么。 |
| [UMA 优化：CPU 可访问纹理和标准 Swizzle](default-texture-mapping.md) | 通用内存体系结构 (UMA) Gpu 相比，具有一些效率优点离散 Gpu，尤其是在针对移动设备进行优化。 当 GPU UMA 可以减少的复制，为资源指定 CPU 访问 CPU 和 GPU 之间发生。 虽然我们不 t 建议应用程序会盲目地让 UMA 设计上的所有资源的 CPU 访问，有机会来提高效率，通过提供 CPU 访问的适当的资源。 与离散 Gpu 不同 CPU 从技术上讲可以有一个指针指向 GPU 可以访问的所有资源。 |
| [类型化无序的访问视图加载](typed-unordered-access-view-loads.md) | 无序访问视图 (UAV) 类型的负载是从具有特定 UAV 读取的着色器的功能[ **DXGI\_格式**](https://msdn.microsoft.com/library/windows/desktop/bb173059)。 |
| [卷平铺资源](volume-tiled-resources.md) | 卷 (3D) 纹理可以用作平铺资源，注意磁贴解析为三维。 |
| [子](subresources.md) | 介绍如何将资源划分为子，以及如何引用一个、 多个或子的切片。 |

## <a name="related-topics"></a>相关主题

* [Direct3D 12 的编程指南](directx-12-programming-guide.md)
* [DirectX 高级学习视频教程：DirectX 12 中的绑定的资源](https://www.youtube.com/watch?v=Uwhhdktaofg)
* [内存管理](memory-management.md)
