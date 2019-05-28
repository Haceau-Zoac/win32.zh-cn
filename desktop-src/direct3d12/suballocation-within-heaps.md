---
title: 子中堆分配
description: 资源堆从到 GPU （上载）、 CPU 和 GPU （读回） 的 CPU 到传输数据。
ms.assetid: 7E319BCF-FF3F-43CB-9C48-A9DD2A043592
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: b011d6f3dda501e87df9da90d2b856bed379a824
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224080"
---
# <a name="suballocation-within-heaps"></a>子中堆分配

资源堆从到 GPU （上载）、 CPU 和 GPU （读回） 的 CPU 到传输数据。

## <a name="in-this-section"></a>本部分内容



| 主题                                                                                       | 描述                                                                                                                                                                                                                                                              |
|---------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [内存别名和数据继承](memory-aliasing-and-data-inheritance.md)<br/> | 放置和保留的资源可能别名堆中的物理内存。 当堆有设置了共享的标志或使用别名资源时已完全定义内存布局，放置资源启用比保留资源的更多数据继承方案。 <br/> |
| [共享的堆](shared-heaps.md)<br/>                                                 | 共享可用于多进程和多适配器体系结构。<br/>                                                                                                                                                                                          |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[**ID3D12Device::CreateHeap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createheap)
</dt> <dt>

[**ID3D12Heap**](/windows/desktop/api/D3D12/nn-d3d12-id3d12heap)
</dt> <dt>

[内存管理](memory-management.md)
</dt> </dl>

 

 





