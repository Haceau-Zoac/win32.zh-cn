---
title: Stream 输出计数器、 UAV 计数器、 查询和断言而
description: Stream 输出和 UAV 计数器操作在 Direct3D 12 中类似的方法，到 Direct3D 11 中，现在必须由应用程序分配内存的计数器，但该驱动程序不会执行该操作。
ms.assetid: 8BDDAFEF-57D4-4EF5-BB0C-6C96AF557A45
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: e20671fcadec92748a487f16099295eae53e743e
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224287"
---
# <a name="stream-output-counters-uav-counters-queries-and-predication"></a>Stream 输出计数器、 UAV 计数器、 查询和断言而

Stream 输出和 UAV 计数器操作在 Direct3D 12 中类似的方法，到 Direct3D 11 中，现在必须由应用程序分配内存的计数器，但该驱动程序不会执行该操作。 Direct3D 12 中的查询所更不同于那些在 Direct3D 11 中，通过添加护栏和消除为某些查询类型需要其他进程。

## <a name="in-this-section"></a>本部分内容



| 主题                                                           | 描述                                                                                                                                                                                  |
|-----------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Stream 输出计数器](stream-output-counters.md)<br/> | Stream 输出是 GPU 能够写入缓冲区的顶点。 流输出计数器监视进度。<br/>                                                               |
| [UAV 计数器](uav-counters.md)<br/>                     | UAV 计数器可用来将 32 位原子计数器与无序访问的视图 (UAV) 相关联。<br/>                                                                                |
| [查询](queries.md)<br/>                               | 在 Direct3D 12 中，查询进行分组的查询称为查询堆的数组。 查询堆有类型用于定义可用于该堆的查询的有效类型。<br/> |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[性能度量](performance-measurement.md)
</dt> </dl>

 

 





