---
title: 硬件层
description: 硬件级别从第 1 层到第 3 层可供管道使用的资源越来越多。
ms.assetid: 5A640BA9-3914-4481-9A0C-E18B52BD8AFE
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: f48c64a569fd47bb58a2ad2d26d1818e2cda41f2
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006014"
---
# <a name="hardware-tiers"></a>硬件层

硬件级别从第 1 层到第 3 层可供管道使用的资源越来越多。

-   [依赖于硬件的限制](#limits-dependant-on-hardware)
-   [固定限制](#invariable-limits)
-   [相关主题](#related-topics)

## <a name="limits-dependant-on-hardware"></a>依赖于硬件的限制



| 可供管道使用的资源                                                                                                              | 第 1 层                                                                   | 第 2 层        | 第 3 层        |
|--------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|---------------|---------------|
| 功能级别                                                                                                                                   | 高于 11.0                                                                    | 高于 11.0         | 11.1+         |
| 常量缓冲区视图 (CBV)、着色器资源视图 (SRV) 或无序访问视图 (UAV) 堆中用于呈现的描述符的最大数量 | 1,000,000                                                                | 1,000,000     | 超过 1,000,000    |
| 每个着色器阶段中所有描述符表中常量缓冲区视图的最大数量                                                                | 14                                                                       | 14            | **完整的堆** |
| 每个着色器阶段所有描述符表中着色器资源视图的最大数量                                                                | 128                                                                      | **完整的堆** | 完整的堆     |
| 所有阶段中全部描述符表内无序访问视图的最大数量                                                              | 64（针对高于 11.1 的功能级别）<br/> 8（针对功能级别 11）<br/> | 64            | **完整的堆** |
| 每个着色器阶段在所有描述符表中采样器的最大数量                                                                             | 16                                                                       | **完整的堆** | 完整的堆     |



 

粗体条目突出了超越上一层的重大改进。

适用于所有堆的第 1 层硬件以及适用于 CBV 和 UAV 堆的第 2 层硬件还有一个额外限制，即根签名中的描述符表所涵盖的所有描述符堆条目都必须在着色器执行前使用描述符进行填充，即使着色器（可能由于分支）不需要描述符。 第 3 层硬件没有此类限制。 缓解这种限制的一种方法是大量使用 [Null 描述符](descriptors.md)。

## <a name="invariable-limits"></a>固定限制

着色器可见描述符堆中的最大采样器数为 2048。

跨活动根签名的唯一静态采样器的最大数量是 2032（其余 16 个用于需自带采样器的驱动程序）。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符堆](descriptor-heaps.md)
</dt> <dt>

[硬件功能级别](hardware-feature-levels.md)
</dt> </dl>

 

 




