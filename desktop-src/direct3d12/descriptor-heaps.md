---
title: 描述符堆
description: 描述符堆是一系列连续分配的描述符，每个描述符的一个资源分配。
ms.assetid: 04d3facf-21ec-45ca-ad9b-78fdcddc7136
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 6e84c1f47495c8de10eeb1fa0de3a2c286f61b08
ms.sourcegitcommit: 42cdae4d2eca84713ab3f7a5c88f583a352991a8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/23/2019
ms.locfileid: "66224476"
---
# <a name="descriptor-heaps"></a>描述符堆

描述符堆是一系列连续分配的描述符，每个描述符的一个资源分配。

## <a name="in-this-section"></a>本节内容



| 主题                                                                                             | 描述                                                                                                                                                                                                                                |
|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [描述符堆概述](descriptor-heaps-overview.md)<br/>                             | 描述符堆包含不是一部分的管道状态对象 (PSO)，如着色器资源视图 (SRVs)、 无序访问视图 (Uav)、 常量缓冲区视图 (CBVs) 和取样器的许多对象类型。 <br/>                |
| [硬件层](hardware-support.md)<br/>                                                 | 从第 1 层到第 3 层的硬件级别具有增加资源可供该管道。 <br/>                                                                                                                              |
| [着色器可见描述符堆](shader-visible-descriptor-heaps.md)<br/>                 | 着色器可见描述符堆，是描述符堆可以引用的描述符表通过着色器。<br/>                                                                                                              |
| [非着色器可见描述符堆](non-shader-visible-descriptor-heaps.md)<br/>         | 某些描述符堆着色器通过描述符表不能引用，但存在任何一个以在过渡环境中之前记录的命令列表的描述符帮助应用程序或由于没有着色器可见堆是必需的。<br/> |
| [创建描述符堆](creating-descriptor-heaps.md)<br/>                             | 若要创建和配置描述符堆，必须选择描述符堆类型、 确定多少描述符包含，并设置指示是否可见的 CPU 和/或着色器可见的标志。 <br/>                    |
| [设置和填充描述符堆](setting-descriptor-heaps.md)<br/>                | 可以设置命令列表的描述符堆类型是那些包含的描述符的描述符表可使用 （最多每一次）。 <br/>                                                        |
| [描述符堆可配置性摘要](descriptor-heap-configurability-summary.md)<br/> | 下表总结了有关着色器和非着色器可见堆支持信息。<br/>                                                                                                                                    |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符](descriptors.md)
</dt> <dt>

[描述符表](descriptor-tables.md)
</dt> <dt>

[**ID3D12DescriptorHeap**](/windows/desktop/api/D3D12/nn-d3d12-id3d12descriptorheap)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 





