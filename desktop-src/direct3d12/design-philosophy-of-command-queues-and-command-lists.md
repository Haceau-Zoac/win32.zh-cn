---
title: 命令队列和命令列表的设计理念
description: 启用重复使用的呈现工作，并且多线程缩放的目标所需如何 Direct3D 应用提交到 GPU 呈现工作基础上的更改。
ms.assetid: C85C8C41-2306-4568-8FAE-F57EDA624298
keywords:
- 命令列表
- bundle
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 83f29e203568adc84ecd8b2957382dcddf3566ac
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224110"
---
# <a name="design-philosophy-of-command-queues-and-command-lists"></a>命令队列和命令列表的设计理念

启用重复使用的呈现工作，并且多线程缩放的目标所需如何 Direct3D 应用提交到 GPU 呈现工作基础上的更改。 在 Direct3D 12 中，提交呈现工作的过程不同于早期版本在三个重要方面：

<dl> 1. 消除了即时上下文。 这使多线程处理。 2. 应用现在拥有呈现调用到图形处理单元 (GPU) 工作项的分组方式。 这样，重复使用。 3. 现在应用显式控制在将工作提交到 GPU。 这可以使项 1 和 2。  
</dl>

-   [删除的即时上下文](#removal-of-the-immediate-context)
-   [分组的 GPU 工作项](#grouping-of-gpu-work-items)
-   [GPU 工作提交](#gpu-work-submission)
-   [相关的主题](#related-topics)

## <a name="removal-of-the-immediate-context"></a>删除的即时上下文

从 Microsoft Direct3D 11 到 Microsoft Direct3D 12 的最大变化是不再与设备关联的单一、 即时上下文。 相反，若要呈现，应用创建命令列表中的传统可调用呈现 Api。 命令列表看起来类似于使用即时上下文，其中包含调用绘制基元或更改的呈现状态的 Direct3D 11 应用的呈现方法。 如即时上下文中，每个命令列表不是自由线程;但是，多个命令将列出可以记录，其中充分利用现代、 多核处理器。

命令列表通常执行一次。 但是，命令列表可以执行多个时间如果应用程序可确保在提交新执行之前确保以前的执行都完成。 有关命令列表同步的详细信息，请参阅[Executing 和同步命令列表](executing-and-synchronizing-command-lists.md)。

## <a name="grouping-of-gpu-work-items"></a>分组的 GPU 工作项

除了命令列表，Direct3D 12 今天充分利用所有硬件中提供功能通过添加第二个级别的命令列表，称为*捆绑包*。 为了帮助区分这两种类型，第一个级别的命令列表可以称为*定向命令列表*。 捆绑包的用途是允许组少量的 API 命令一起为更高版本时，重复执行从直接的命令列表中的应用程序。 创建一个捆绑包时，驱动程序将执行，以使更高版本执行高效尽可能预处理。 在多个命令列表和多个时间内相同的命令列表，然后可以从执行捆绑包。

重复使用捆绑包是效率的提升以及单个 CPU 线程的大型驱动程序。 捆绑包进行了预先处理，并可以多次提交，因为有一定限制可以对绑定中执行哪些操作。 有关详细信息，请参阅[创建和记录命令列表和捆绑包](recording-command-lists-and-bundles.md)。

## <a name="gpu-work-submission"></a>GPU 工作提交

若要在 GPU 上执行工作，应用程序必须显式提交到与 Direct3D 设备相关联的命令队列的命令列表。 直接的命令列表都可以提交要执行多次，但该应用程序负责确保直接的命令列表已完成重新提交之前在 GPU 上执行。 捆绑包没有并发使用限制和可执行多个时间的多个命令列表，但是捆绑包不能直接提交给执行的命令队列。

任何线程可能在任何时候，提交到任何命令队列的命令列表，然后运行时将自动序列化时保留的提交顺序提交命令队列中的命令列表。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[提交工作在 Direct3D 12](command-queues-and-command-lists.md)
</dt> </dl>

 

 




