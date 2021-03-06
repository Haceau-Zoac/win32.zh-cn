---
title: 堆中的二次分配
description: 资源堆将数据从 CPU 传输到 GPU（上传），并从 GPU 传输到 CPU（回读）。
ms.assetid: 7E319BCF-FF3F-43CB-9C48-A9DD2A043592
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 701e68e31e698bbf2c955a252bd46876f45d6b7c
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006216"
---
# <a name="suballocation-within-heaps"></a>堆中的二次分配

资源堆将数据从 CPU 传输到 GPU（上传），并从 GPU 传输到 CPU（回读）。

## <a name="in-this-section"></a>本节内容



| 主题                                                                                       | 描述                                                                                                                                                                                                                                                              |
|---------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [内存别名和数据继承](memory-aliasing-and-data-inheritance.md)<br/> | 放置和保留的资源在堆中可能会使用物理内存别名。 堆具有共享标志集或当具有别名的资源已完全定义内存布局时，放置的资源启用的数据继承方案比保留的资源更多。 <br/> |
| [共享堆](shared-heaps.md)<br/>                                                 | 共享对于多进程和多适配器体系结构非常有用。<br/>                                                                                                                                                                                          |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[**ID3D12Device::CreateHeap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createheap)
</dt> <dt>

[**ID3D12Heap**](/windows/desktop/api/d3d12/nn-d3d12-id3d12heap)
</dt> <dt>

[内存管理](memory-management.md)
</dt> </dl>

 

 





