---
title: 根签名概述
description: 根签名配置应用程序和链接命令将列出到着色器需要的资源。
ms.assetid: 2E649DA2-6CAC-4C2A-A420-D4EC0DD6EA73
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 19261caac1266152c45d16346f7a1ebd203e2023
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224167"
---
# <a name="root-signatures-overview"></a>根签名概述

根签名配置应用程序和链接命令将列出到着色器需要的资源。 图形命令列表具有的图形和计算的根签名。 一个计算命令列表将只具有一个计算根签名。 这些根签名是相互独立。

-   [根参数和自变量](#root-parameters-and-arguments)
-   [根常量、 描述符和表](#root-constants-descriptors-and-tables)
-   [相关的主题](#related-topics)

## <a name="root-parameters-and-arguments"></a>根参数和自变量

一个**根签名**是类似的 API 函数签名，它确定的着色器期望的那样，但未定义的实际内存或数据的数据类型。 一个**根参数**根签名中的一个条目。 根参数设置并在运行时更改的实际值称为**根参数**。 更改根自变量会更改着色器读取的数据。

## <a name="root-constants-descriptors-and-tables"></a>根常量、 描述符和表

根签名可以包含三种类型的参数;根常量 （常量内联根自变量中）、 根描述符 （描述符内联根自变量中） 和描述符表 （指向一系列描述符堆中的描述符的指针）。

根常量是内联 32 位值常量缓冲区用作着色器中显示的。

内联的根描述符应包含的最常访问，但仅限于 CBVs 和原始或结构化 UAV 或 SRV 缓冲区描述符。 一个更复杂的类型，例如 2D 纹理 SRV，不能用作根描述符。 根描述符不包括的大小限制，因此可以对任何超出边界检查，与不同的描述符中的描述符堆，其中包含一个大小。

根签名中的描述符表条目还包含描述符、 HLSL 着色器绑定名称和可见性标志。 请参阅[着色器模型 5.1](https://msdn.microsoft.com/library/windows/desktop/dn933277)的着色器名称的详细信息。 在某些硬件上，可以仅使需要它们的着色器阶段对可见描述符可以获得性能提升 (请参阅[ **D3D12\_着色器\_可见性**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_shader_visibility))。

![根描述符表项](images/root-descriptor-table.png)

根签名的布局是非常灵活，有一些性能较差的硬件上施加的约束。 而不考虑硬件级别，应用程序应始终尝试使根签名尽可能小所需的最大效率。 应用程序可以具有多个描述符表中的根签名但不为根常量留出空间太权衡，反之亦然。

应用程序已自动绑定根签名 （描述符表、 根常量和根描述符） 的内容获取 D3D12 驱动程序版本控制，只要内容的任何部分之间绘制 （图形） 更改 / 调度 （计算）调用。 因此，每个绘制/调度获取一组唯一完整的根签名状态。

理想情况下，没有管道状态对象 (Pso) 共享相同的根签名的组。 对管道设置根签名后，它定义 （描述符表，描述符，常量） 的所有绑定可每个单独设置或更改，包括捆绑的继承。

应用可以进行多少描述符表它想与内联描述符 （这需要更多空间但删除间接寻址） 之间自己权衡与内联常量 （它具有非间接） 他们希望在根签名。 应用程序应使用根签名为可能，为堆依赖于此类应用程序控制内存和描述符堆指针到它们尽量少表示大容量数据。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




