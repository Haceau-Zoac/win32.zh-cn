---
title: 与 Direct3D 11 中的绑定模型的区别
description: Directx12 绑定背后的主要设计决策之一是将其与其他管理任务分离。 这对应用提出了一些要求，以管理某些潜在的危险。
ms.assetid: 3EE7E9AE-203D-40D4-9F12-4313ED179035
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 43b2785da6497fd4e775d9f88847928e7c4c08e8
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71005699"
---
# <a name="differences-in-the-binding-model-from-direct3d-11"></a>与 Direct3D 11 中的绑定模型的区别

Directx12 绑定背后的主要设计决策之一是将其与其他管理任务分离。 这对应用提出了一些要求，以管理某些潜在的危险。

D3D12 绑定模型的主要优点是，它使应用能够经常更改纹理绑定，而不会产生巨大的 CPU 性能成本。 其他优点是，着色器可以访问大量资源，着色器不需要预先知道将绑定多少资源，并且无论硬件或应用的内容流如何，都可以使用统一的资源绑定模型。

为了提高性能，绑定模型不需要系统跟踪应用请求 GPU 使用的绑定，并且绑定和多线程命令列表之间存在干净的集成。

以下部分列出了自 D3D11 以来资源绑定模型的一些更改。

-   [内存驻留管理与绑定分离](#memory-residency-management-separated-from-binding)
-   [对象生命周期管理与绑定分离](#object-lifetime-management-separated-from-binding)
-   [驱动程序资源状态跟踪与绑定分离](#driver-resource-state-tracking-separated-from-binding)
-   [CPU GPU 映射内存同步与绑定分离](#cpu-gpu-mapped-memory-synchronization-separated-from-binding)
-   [相关主题](#related-topics)

## <a name="memory-residency-management-separated-from-binding"></a>内存驻留管理与绑定分离

应用程序可以显式控制 GPU 直接使用哪些表明（称为“常驻”）。 相反，它们可以在资源上应用其他状态，比如显式地使其不驻留，或者让操作系统选择某些需要最小内存占用的应用程序类。 这里重要的一点是，应用程序对驻留内容的管理与它如何向着色器提供对资源的访问完全分离。

由于操作系统不必经常检查本地绑定状态以了解如何实现驻留，因此将驻留管理与授予着色器对资源的访问权限的机制分离可以降低渲染的系统/硬件成本。 此外，只要可能的可访问资源集都提前驻留，着色器就不再需要知道它们可能需要引用的确切表面。

## <a name="object-lifetime-management-separated-from-binding"></a>对象生命周期管理与绑定分离

与以前的 API 不同，系统不再跟踪资源到管道的绑定。 这种情况用于使系统能够保留应用程序释放的可用资源，因为它们仍然被未完成的 GPU 工作引用。

在释放任何资源（如纹理）之前，应用程序现在必须确保 GPU 已完成对该资源的引用。 这意味着在应用程序可以安全释放资源之前，GPU 必须完成引用该资源的命令列表的执行。

## <a name="driver-resource-state-tracking-separated-from-binding"></a>驱动程序资源状态跟踪与绑定分离

系统不再检查资源绑定以了解何时发生了需要额外驱动程序或 GPU 工作的资源转换。 对于许多 GPU 和驱动程序来说，一个常见的例子是必须知道表面何时从用作呈现目标视图 (RTV) 过渡到着色器资源视图 (SRV)。 应用程序本身现在必须确定系统可能关心的任何资源转换何时通过专用 API 进行。

## <a name="cpu-gpu-mapped-memory-synchronization-separated-from-binding"></a>CPU GPU 映射内存同步与绑定分离

系统不再检查资源绑定以了解呈现是否需要延迟，因为它依赖于已为 CPU 访问映射但尚未取消映射的资源。 应用程序现在有责任同步 CPU 和 GPU 内存访问。 为了解决这个问题，系统为应用程序提供了相关机制，以请求 CPU 线程休眠，直到工作完成。 也可以进行轮询，但可能会降低效率。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 




