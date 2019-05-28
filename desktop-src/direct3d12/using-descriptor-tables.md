---
title: 使用描述符表
description: 在槽上命令列表的当前根签名定义绑定描述符表，每个标识描述符堆中的范围。
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

在槽上命令列表的当前根签名定义绑定描述符表，每个标识描述符堆中的范围。

-   [描述符表编制索引](#indexing-descriptor-tables)
-   [相关的主题](#related-topics)

着色器可以找到引用的构成描述符表的描述符的资源。 直接在命令列表上，而不是通过描述符完成其他资源绑定-索引缓冲区、 顶点缓冲区、 Stream 输出缓冲区、 呈现目标和深度模具。 总结：

以下资源引用可以共享同一个描述符表和堆：

-   着色器资源视图
-   无序的访问视图
-   常量缓冲区视图

以下资源引用必须在其自己的描述符堆：

-   取样器

以下资源未置于描述符表或堆的方法，但绑定在直接使用命令列表：

-   索引缓冲区
-   顶点缓冲区
-   Stream 输出缓冲区
-   呈现器目标
-   深度模具视图

## <a name="indexing-descriptor-tables"></a>描述符表编制索引

着色器不能跨描述符表边界从着色器中的给定调用站点的动态索引。 但是，描述符表中的描述符的选择被允许在相同的描述符类型 （如索引在 SRVs 的相连区域之间） 的范围内的着色器代码中动态地编制索引。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符表](descriptor-tables.md)
</dt> </dl>

 

 




