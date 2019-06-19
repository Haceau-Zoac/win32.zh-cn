---
title: 使用描述符表
description: 描述符表（每个表标识描述符堆中的一个范围）绑定在由命令列表上的当前根签名所定义的槽中。
ms.assetid: 4ECEC07A-7ABC-4C5F-B23C-36F0D92643FE
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 6da625011ae28f4973e485f25c88e46989e20057
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223858"
---
# <a name="using-descriptor-tables"></a>使用描述符表

描述符表（每个表标识描述符堆中的一个范围）绑定在由命令列表上的当前根签名所定义的槽中。

-   [编制描述符表索引](#indexing-descriptor-tables)
-   [相关主题](#related-topics)

着色器可以定位由构成描述符表的描述符所引用的资源。 其他资源绑定 - 索引缓冲区、顶点缓冲区、流输出缓冲区、呈现器目标和深度模具直接在命令列表上（而不是通过描述符）完成。 总结：

以下资源引用可以共享相同的描述符表和堆：

-   着色器资源视图
-   无序访问视图
-   常量缓冲区视图

以下资源引用必须位于自己的描述符堆：

-   采样器

以下资源不在描述符表或堆中，而是使用命令列表直接绑定：

-   索引缓冲区
-   顶点缓冲区
-   流输出缓冲区
-   呈现器目标
-   深度模具视图

## <a name="indexing-descriptor-tables"></a>编制描述符表索引

着色器不能通过着色器中的给定调用站点跨描述符表边界动态编制索引。 但是，描述符表中描述符的选择可以在相同描述符类型（例如跨 SRV 的连续区域编制索引）范围内的着色器代码中动态编制索引。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符表](descriptor-tables.md)
</dt> </dl>

 

 




