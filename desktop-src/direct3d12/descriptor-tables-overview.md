---
title: 描述符表概述
description: 每个描述符表可存储一种或多种类型的描述符 - SRV、UAV、CBV 和采样器。 描述符表不是内存分配；它只是描述符堆中的偏移量和长度。
ms.assetid: 4A5749CE-DD84-40E1-B67F-31828165A631
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: d446a0570cf813eacaa4d036781e8cd4b8def3c1
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006258"
---
# <a name="descriptor-tables-overview"></a>描述符表概述

每个描述符表可存储一种或多种类型的描述符 - SRV、UAV、CBV 和采样器。 描述符表不是内存分配；它只是描述符堆中的偏移量和长度。

-   [引用描述符表](#referencing-descriptor-tables)
-   [相关主题](#related-topics)

## <a name="referencing-descriptor-tables"></a>引用描述符表

通过根签名，图形管道可通过按索引引用描述符表来获得对资源的访问权限。

描述符表实际上只是描述符堆的子范围。 描述符堆表示一批描述符的基础内存分配。 由于内存分配是创建描述符堆的属性，因此可确保从描述符堆中定义描述符表的成本与在堆中向硬件标识区域的成本一样低。 不需要在 API 级别创建或销毁描述符表 - 系统仅会在堆被引用时向驱动程序将它们标识为堆的偏移量和大小。

当应用的着色器需要能够动态地（可能由材料数据驱动）从大量可用的描述符中自由进行选择（通常是引用纹理）时，应用则完全可以定义非常大的描述符表。

根签名可引用描述符表条目，其中包含对堆、表的起始位置（与堆的起始处的偏移量）以及表的长度（在条目中）的引用。 下图演示了这些概念：根签名中的描述符表指针和描述符堆中的描述符引用上传堆中的完整纹理或缓冲区数据。

![描述符表](images/descriptor-table.png)

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符表](descriptor-tables.md)
</dt> </dl>

 

 




