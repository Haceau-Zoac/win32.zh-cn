---
title: 流输出计数器、UAV 计数器、查询和预测
description: 流输出和 UAV 计数器在 Direct3D 12 中以类似于 Direct3D 11 的方法操作，尽管现在计数器的内存必须由应用分配，但驱动程序不会执行该操作。
ms.assetid: 8BDDAFEF-57D4-4EF5-BB0C-6C96AF557A45
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: bd7964a95aaf1ae7b14d09221afd7c0b6ed97952
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296046"
---
# <a name="stream-output-counters-uav-counters-queries-and-predication"></a>流输出计数器、UAV 计数器、查询和预测

流输出和 UAV 计数器在 Direct3D 12 中以类似于 Direct3D 11 的方法操作，尽管现在计数器的内存必须由应用分配，但驱动程序不会执行该操作。 Direct3D 12 中的查询与 Direct3D 11 中的查询更加不同，添加了栅栏和其他无需某些查询类型的进程。

## <a name="in-this-section"></a>本部分内容



| 主题                                                           | 描述                                                                                                                                                                                  |
|-----------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [流输出计数器](stream-output-counters.md)<br/> | 流输出是 GPU 将顶点写入缓冲区的能力。 流输出计数器用于监视进度。<br/>                                                               |
| [UAV 计数器](uav-counters.md)<br/>                     | UAV 计数器可用于将 32 位原子计数器与无序访问视图 (UAV) 关联。<br/>                                                                                |
| [查询](queries.md)<br/>                               | 在 Direct3D 12 中，查询会分组为称为查询堆的查询数组。 查询堆具有一个类型，该类型用于定义可与该堆配合使用的有效查询类型。<br/> |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[性能度量](performance-measurement.md)
</dt> </dl>

 

 





