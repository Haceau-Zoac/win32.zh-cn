---
title: 绑定模型中的区别 Direct3D 11
description: DirectX12 绑定背后的主要设计决策之一是将它与其他管理任务。 这会应用来管理某些潜在危险的一些要求。
ms.assetid: 3EE7E9AE-203D-40D4-9F12-4313ED179035
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: da9874d5b3e664051d8367b8692a1d643a4333f9
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224428"
---
# <a name="differences-in-the-binding-model-from-direct3d-11"></a>绑定模型中的区别 Direct3D 11

DirectX12 绑定背后的主要设计决策之一是将它与其他管理任务。 这会应用来管理某些潜在危险的一些要求。

D3D12 绑定模型的主要优点是它使应用纹理绑定经常变化，而无需非常大的 CPU 性能成本。 其他优点是和着色器有权访问大量的资源、 着色器不需要知道将绑定提前多少资源，统一的资源绑定模型，可以使用而不考虑硬件或应用程序内容流。

若要提高性能，绑定模型不需要系统来跟踪哪些绑定应用程序请求 GPU，若要使用，并且没有绑定和多线程的命令列表之间的全新集成。

以下各节列出了以来 D3D11 一些对资源绑定模型的更改。

-   [内存驻留管理分开绑定](#memory-residency-management-separated-from-binding)
-   [从绑定分隔对象生存期管理](#object-lifetime-management-separated-from-binding)
-   [跟踪驱动程序资源状态与绑定分离](#driver-resource-state-tracking-separated-from-binding)
-   [CPU GPU 映射分隔从绑定的内存同步](#cpu-gpu-mapped-memory-synchronization-separated-from-binding)
-   [相关的主题](#related-topics)

## <a name="memory-residency-management-separated-from-binding"></a>内存驻留管理分开绑定

应用程序具有显式控制他们需要的图面上可供直接使用 GPU （称为正在"居民"）。 相反，它们可以应用资源，例如显式使其不驻留，或允许操作系统选择某些类需要最小内存占用量的应用程序的其他状态。 这里的重要一点是如何为资源添加到着色器访问完全分离的常驻的应用程序的管理。

分离驻留授予着色器对资源的访问权限的机制管理可减少呈现由于操作系统就不必不断地检查要知道该如何面对驻留的本地绑定状态的系统/硬件成本。 此外，着色器不再需要知道它们可能需要引用，如有可能访问的资源的整个集已提前常驻的确切图面。

## <a name="object-lifetime-management-separated-from-binding"></a>从绑定分隔对象生存期管理

系统无法再到管道跟踪资源的绑定与以前的 Api 不同。 GPU 工作这用来使系统能够保持活动状态已发布应用程序，因为它们仍由未完成引用的资源。

之前释放任何资源，例如纹理，应用程序现在必须确保 GPU 已完成对其进行引用。 这意味着之前应用程序可以安全地释放资源 GPU 必须已完成执行的命令列表引用的资源。

## <a name="driver-resource-state-tracking-separated-from-binding"></a>跟踪驱动程序资源状态与绑定分离

系统不会再检查资源绑定，以了解当资源转换发生需要其他驱动程序或 GPU 工作。 一个常见示例为多个 Gpu 和驱动程序不必知道图面时从正在使用作为呈现目标视图 (RTV) 到着色器资源视图 (SRV) 转换。 时系统可能会关心任何资源转换发生通过专用 Api，现在必须确定应用程序本身。

## <a name="cpu-gpu-mapped-memory-synchronization-separated-from-binding"></a>CPU GPU 映射分隔从绑定的内存同步

系统不会再检查资源绑定，以了解呈现需要会延迟，因为它依赖于已映射的 CPU 访问但还没有未映射的资源。 应用程序现在有责任同步 CPU 和 GPU 内存访问。 为了帮助实现此目的，系统提供了要请求睡眠 CPU 线程，直到工作完成的应用程序的机制。 轮询也可以实现，但可以是效率较低。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 




