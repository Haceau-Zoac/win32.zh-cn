---
title: 资源绑定
description: 绑定是将资源对象链接到图形管道着色器的过程。
ms.assetid: CB0065EF-4E53-4E16-9F7E-09A261C360FB
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 7ca1bb5b9bb747bd315de9901d8d8e2257f13e23
ms.sourcegitcommit: 27a9dfa3ef68240fbf09f1c64dff7b2232874ef4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66725401"
---
# <a name="resource-binding"></a>资源绑定

绑定是将资源对象链接到图形管道着色器的过程。

## <a name="in-this-section"></a>本部分内容

| 主题 | 描述 |
|-|-|
| [资源绑定概述](resource-binding-flow-of-control.md) | 对于 DirectX 12 中的资源绑定，关键是要了解描述符、描述符表、描述符堆和根签名的概念。     |
| [与 Direct3D 11 中的绑定模型的区别](binding-model.md) | Directx12 绑定背后的主要设计决策之一是将其与其他管理任务分离。 这对应用提出了一些要求，以管理某些潜在的危险。 |
| [描述符](descriptors.md) | 描述符是 D3D12 中单个资源的主要绑定单元。 |
| [描述符堆](descriptor-heaps.md) | 描述符堆是描述符的连续分配的集合，每个描述符都有一个分配。 |
| [描述符表](descriptor-tables.md) | 描述符表在逻辑上是描述符数组。 |
| [根签名](root-signatures.md) | 根签名定义绑定到图形管道的资源类型。 |
| [功能查询](capability-querying.md) | 通过调用 [ID3D12Device::CheckFeatureSupport](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)，应用程序可以发现对资源绑定和许多其他功能的支持级别  。 |
| [HLSL 中的资源绑定](resource-binding-in-hlsl.md) | 本部分介绍 Direct3D 12 中使用高级着色语言 (HLSL) [着色器模型 5.1](https://docs.microsoft.com/windows/desktop/direct3dhlsl/shader-model-5-1) 的某些特定功能。 所有 Direct3D 12 硬件都支持着色器模型 5.1，因此对此模型的支持不依赖于硬件功能级别。 |
| [UMA 优化：CPU 可访问纹理和标准重排](default-texture-mapping.md) | 通用内存体系结构 (UMA) GPU 与离散 GPU 相比具有一定的效率优势，尤其是在针对移动设备进行优化时。 当 GPU 是 UMA 时，授予资源 CPU 访问权限可以减少 CPU 和 GPU 之间发生的复制量。 虽然我们不建议盲目地让 CPU 访问 UMA 设计上的所有资源，但仍有机会通过提供正确的 CPU 资源访问来提高效率。 与离散 GPU 不同，CPU 在技术上可以指向 GPU 可访问的所有资源。 |
| [类型化无序访问视图加载](typed-unordered-access-view-loads.md) | 无序访问视图 (UAV) 类型化加载是着色器通过特定 [DXGI\_FORMAT](https://docs.microsoft.com/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format) 读取 UAV 的能力  。 |
| [立体平铺资源](volume-tiled-resources.md) | 立体 (3D) 纹理可以用作平铺资源，请注意，平铺分辨率是三维的。 |
| [子资源](subresources.md) | 介绍如何将资源划分为子资源，以及如何引用单个、多个或切片子资源。 |

## <a name="related-topics"></a>相关主题

* [Direct3D 12 编程指南](directx-12-programming-guide.md)
* [DirectX 高级学习视频教程：DirectX 12 中的资源绑定](https://www.youtube.com/watch?v=Uwhhdktaofg)
* [内存管理](memory-management.md)
