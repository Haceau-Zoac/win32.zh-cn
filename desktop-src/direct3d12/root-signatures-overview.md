---
title: 根签名概述
description: 根签名由应用配置，并将命令列表链接到着色器所需的资源。
ms.assetid: 2E649DA2-6CAC-4C2A-A420-D4EC0DD6EA73
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 84ae85f6d0455152ef90601f17d5e8ce4619ad6e
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71005785"
---
# <a name="root-signatures-overview"></a>根签名概述

根签名由应用配置，并将命令列表链接到着色器所需的资源。 图形命令列表同时具有图形和计算根签名。 计算命令列表只具有一个计算根签名。 这些根签名彼此独立。

-   [根形参和根实参](#root-parameters-and-arguments)
-   [根常量、描述符和表](#root-constants-descriptors-and-tables)
-   [相关主题](#related-topics)

## <a name="root-parameters-and-arguments"></a>根形参和根实参

“根签名”类似于 API 函数签名，它确定着色器应期望的数据类型，但不定义实际的内存或数据。 “根形参”是根签名中的一个条目。 在运行时设置和更改的根形参的实际值称为“根实参”。 更改根实参会更改着色器读取的数据。

## <a name="root-constants-descriptors-and-tables"></a>根常量、描述符和表

根签名可以包含三种类型的形参；根常量（根实参中内联的常量），根描述符（根实参中内联的描述符）和描述符表（指向描述符堆中一系列描述符的指针）。

根常量是内联 32 位值，在着色器中显示为常量缓冲区。

内联根描述符应该包含访问频率最高的描述符，但仅限于 CBV、原始或结构化 UAV 或 SRV 缓冲区。 更复杂的类型（如 2D texture SRV）不能用作根描述符。 根描述符不包含大小限制，因此不能进行越界检查，这与描述符堆中包含大小的描述符不同。

根签名中的描述符表条目包含描述符、HLSL 着色器绑定名称和可见性标志。 有关着色器名称的详细信息，请参阅 [Shader Model 5.1](https://docs.microsoft.com/windows/desktop/direct3dhlsl/shader-model-5-1)（着色器模型 5.1）。 在某些硬件上，只有使描述符对需要它们的着色器阶段可见才能获得性能提升（请参见 [D3D12\_SHADER\_VISIBILITY](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shader_visibility)）。

![根描述符表条目](images/root-descriptor-table.png)

根签名的布局非常灵活，对功能有限的硬件施加了一些限制。 无论硬件级别如何，应用程序都应尽量使根签名尽可能小，以获得最大的效率。 应用程序可以在根签名中权衡使用更多描述符表，而为根常量预留更少空间，反之亦然。

应用程序绑定的根签名的内容（描述符表、根常量和根描述符）在绘制（图形）/调度（计算）调用之间的任何部分发生更改时，都会自动由 D3D12 驱动程序进行版本控制。 因此，每个绘制/调度都会获得一组唯一的根签名状态。

理想情况下，有多组管共享相同根签名的管道状态对象 (PSO)。 在管道上设置根签名后，它所定义的所有绑定（描述符表、描述符、常量）都可以单独进行设置或更改，包括捆绑的继承。

应用可以在它所需的描述符表与在根签名中所需的内联描述符（占用更多空间但会删除间接）和内联常量（没有间接）之间自行进行权衡。 应用程序应尽可能少地使用根签名，并依赖其控制的内存（如指向根签名的堆和描述符堆）来表示批量数据。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 




