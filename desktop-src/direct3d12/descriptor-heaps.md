---
title: 描述符堆
description: 描述符堆是描述符的连续分配的集合，每个描述符都有一个分配。
ms.assetid: 04d3facf-21ec-45ca-ad9b-78fdcddc7136
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 5f19821cff5e8730c376e5c80fef07e67974d31d
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006263"
---
# <a name="descriptor-heaps"></a>描述符堆

描述符堆是描述符的连续分配的集合，每个描述符都有一个分配。

## <a name="in-this-section"></a>本节内容



| 主题                                                                                             | 描述                                                                                                                                                                                                                                |
|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [描述符堆概述](descriptor-heaps-overview.md)<br/>                             | 描述符堆包含不属于管道状态对象 (PSO) 的许多对象类型，例如，着色器资源视图 (SRV)、无序访问视图 (UAV)、常量缓冲区视图 (CBV) 和取样器。 <br/>                |
| [硬件层](hardware-support.md)<br/>                                                 | 硬件级别从第 1 层到第 3 层可供管道使用的资源越来越多。 <br/>                                                                                                                              |
| [着色器可见描述符堆](shader-visible-descriptor-heaps.md)<br/>                 | 着色器可见描述符堆是着色器可通过描述符表引用的描述符堆。<br/>                                                                                                              |
| [非着色器可见描述符堆](non-shader-visible-descriptor-heaps.md)<br/>         | 着色器无法通过描述符表引用部分描述符堆，但描述符堆的存在是为了帮助应用在记录命令列表之前暂存描述符，或者是因为不需着色器可见堆。<br/> |
| [创建描述符堆](creating-descriptor-heaps.md)<br/>                             | 若要创建和配置描述符堆，必须选择描述符堆类型，确定所包含的描述符数，并设置指示 CPU 是否可见和/或着色器是否可见的标志。 <br/>                    |
| [设置和填充描述符堆](setting-descriptor-heaps.md)<br/>                | 可在命令列表上设置的描述符堆类型包括可使用描述符表（每次最多使用一个表）的描述符。 <br/>                                                        |
| [描述符堆可配置性摘要](descriptor-heap-configurability-summary.md)<br/> | 下表总结了有关着色器和非着色器可见堆支持的信息。<br/>                                                                                                                                    |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符](descriptors.md)
</dt> <dt>

[描述符表](descriptor-tables.md)
</dt> <dt>

[**ID3D12DescriptorHeap**](/windows/desktop/api/d3d12/nn-d3d12-id3d12descriptorheap)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 





