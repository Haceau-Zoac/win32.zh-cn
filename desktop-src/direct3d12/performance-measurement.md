---
title: 计数器、查询和性能度量
description: 以下各节描述了用于性能测试和改进的功能，例如查询、计数器、定时和预测。
ms.assetid: C7AEF1A0-36FB-4026-9CCF-ED0206961A58
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: bf316978f3dd0928692f378dd8d72b8453ad0aae
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006144"
---
# <a name="counters-queries-and-performance-measurement"></a>计数器、查询和性能度量

以下各节描述了用于性能测试和改进的功能，例如查询、计数器、定时和预测。

## <a name="in-this-section"></a>本节内容



| 主题                                                                                                 | 描述                                                                                                                                                                                                                                                                                                                                                        |
|-------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [流输出计数器、UAV 计数器、查询和预测](counters-and-queries.md)<br/> | 流输出和 UAV 计数器在 Direct3D 12 中以类似于 Direct3D 11 的方法操作，尽管现在计数器的内存必须由应用分配，但驱动程序不会执行该操作。 Direct3D 12 中的查询与 Direct3D 11 中的查询更加不同，添加了栅栏和其他无需某些查询类型的进程。<br/> |
| [定时](timing.md)<br/>                                                                       | 本节介绍如何查询时间戳，以及如何校准 GPU 和 CPU 时间戳计数器。<br/>                                                                                                                                                                                                                                                            |
| [预测](predication.md)<br/>                                                             | 预测是一种功能，使 GPU 而不是 CPU 能够决定不绘制、复制或调度对象。<br/>                                                                                                                                                                                                                                 |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 编程指南](directx-12-programming-guide.md)
</dt> </dl>

 

 





