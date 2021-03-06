---
title: Direct3D 12 中的内存管理
description: 迁移到 D3D12 需对内存驻留进行适当同步和管理。
ms.assetid: 94D47EBB-8060-49F6-A1FF-8B7B98AD5363
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 037a2d75adcde6ff03adf2ccee8ce30d33d090c7
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71005861"
---
# <a name="memory-management-in-direct3d-12"></a>Direct3D 12 中的内存管理

迁移到 D3D12 需对内存驻留进行适当同步和管理。 管理内存驻留意味着必须执行更多同步。 本节介绍内存管理策略，以及堆和缓冲区中的二次分配。

-   [本部分内容](#in-this-section)
-   [相关主题](#related-topics)

## <a name="in-this-section"></a>本节内容



| 主题                                                                       | 描述                                                                                                                                                                                                                                                                                                                                                                          |
|-----------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [内存管理策略](memory-management-strategies.md)<br/> | Direct3D 12 的内存管理器可能会迅速变得非常复杂，原因在于所有对 UMA 或离散（非 UMA）适配器的各层级支持，以及 GPU 适配器之间相当多的体系结构差异。<br/> 本部分所述的建议 Direct3D 12 内存管理策略是“分类、预算和流式传输”。<br/> |
| [缓冲区中的二次分配](large-buffers.md)<br/>                | 缓冲区具有 D3D12 所需的所有功能，可供应用程序将大量瞬态数据从 CPU 传输到 GPU。 本节介绍使用和管理资源及缓冲区的四种常见场景。<br/>                                                                                                                                     |
| [堆中的二次分配](suballocation-within-heaps.md)<br/>     | 资源堆将数据从 CPU 传输到 GPU（上传），并从 GPU 传输到 CPU（回读）。 <br/>                                                                                                                                                                                                                                                                  |
| [驻留](residency.md)<br/>                                       | 如果某个对象可被 GPU 访问，则该对象被视为“常驻”。<br/>                                                                                                                                                                                                                                                                                                |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 编程指南](directx-12-programming-guide.md)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 





