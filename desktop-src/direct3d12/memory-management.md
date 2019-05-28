---
title: 内存管理在 Direct3D 12
description: 将移动到 D3D12 涉及正确的同步和管理的物理内存。
ms.assetid: 94D47EBB-8060-49F6-A1FF-8B7B98AD5363
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 6cc1cb438c689e19aab97db44137856d64a76e08
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224038"
---
# <a name="memory-management-in-direct3d-12"></a>内存管理在 Direct3D 12

将移动到 D3D12 涉及正确的同步和管理的物理内存。 管理内存驻留表示必须完成更多的同步。 本部分介绍内存管理策略，以及子堆中的分配和缓冲区。

-   [在本部分中](#in-this-section)
-   [相关的主题](#related-topics)

## <a name="in-this-section"></a>本部分内容



| 主题                                                                       | 描述                                                                                                                                                                                                                                                                                                                                                                          |
|-----------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [内存管理策略](memory-management-strategies.md)<br/> | Direct3D 12 的内存管理器无法获取快速与技术支持、 UMA 或合理的 (非 UMA) 适配器，以及使用 GPU 适配器之间的体系结构差异相当大范围的所有不同层非常复杂。<br/> Direct3D 12 内存管理，在本部分中所述的建议的策略是"分类、 预算和流式传输"。<br/> |
| [在缓冲区中的子分配](large-buffers.md)<br/>                | 缓冲区具有必要的应用程序将从 CPU 的很大变化的暂时性数据传输到 GPU D3D12 中的所有功能。 本部分介绍的使用和管理资源和缓冲区的四个常见方案。<br/>                                                                                                                                     |
| [子中堆分配](suballocation-within-heaps.md)<br/>     | 资源堆从到 GPU （上载）、 CPU 和 GPU （读回） 的 CPU 到传输数据。 <br/>                                                                                                                                                                                                                                                                  |
| [驻留](residency.md)<br/>                                       | 一个对象被视为*常驻*时由 GPU 可访问。<br/>                                                                                                                                                                                                                                                                                                |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 的编程指南](directx-12-programming-guide.md)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 





