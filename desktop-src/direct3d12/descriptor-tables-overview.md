---
title: 描述符表概述
description: 每个描述符表存储一个或多个类型-SRVs、 UAVe、 CBVs 和取样器的描述符。 描述符表不是内存的分配;它只是偏移量和长度描述符堆。
ms.assetid: 4A5749CE-DD84-40E1-B67F-31828165A631
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 5e983bab45df3f8684d0ec1231f833053f232544
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224122"
---
# <a name="descriptor-tables-overview"></a>描述符表概述

每个描述符表存储一个或多个类型-SRVs、 UAVe、 CBVs 和取样器的描述符。 描述符表不是内存的分配;它只是偏移量和长度描述符堆。

-   [引用描述符表](#referencing-descriptor-tables)
-   [相关的主题](#related-topics)

## <a name="referencing-descriptor-tables"></a>引用描述符表

图形管道中的，通过根签名中，通过按索引引用到描述符表获得对资源的访问。

描述符表是实际上就是描述符堆的子范围。 描述符堆表示描述符的集合的基础内存分配。 由于内存分配是创建描述符堆的一个属性，定义描述符表从一个保证可开销识别到硬件堆中的区域。 描述符表不需要创建或销毁在 API 级别 – 它们只是作为偏移量和大小超出堆时引用标识驱动程序。

它是完全有可能使应用程序时它的着色器希望可以自由地选择从大量可用描述符 （通常引用纹理） （可能是由材料数据驱动） 动态定义非常大的描述符表。

根签名引用具有到堆的引用，表 （一个偏移量开始堆），并在表的长度 （以条目） 的开始位置的描述符表条目。 下图显示了这些概念： 描述符表指针从根签名和描述符堆引用完整纹理或缓冲区中的堆上, 传的数据中的描述符。

![描述符表](images/descriptor-table.png)

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符表](descriptor-tables.md)
</dt> </dl>

 

 




