---
title: 命令队列和命令列表的设计理念
description: 为了实现渲染工作的重用和多线程缩放，需要对 Direct3D 应用向 GPU 提交渲染工作的方式进行根本性的改变。
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

为了实现渲染工作的重用和多线程缩放，需要对 Direct3D 应用向 GPU 提交渲染工作的方式进行根本性的改变。 在 Direct3D 12 中，提交渲染工作的过程在三个重要方面不同于早期版本：

<dl> 1. 消除了即时上下文。 这样可以实现多线程。 2. 应用现在拥有将渲染调用分组到图形处理单元 (GPU) 工作项中的方法。 这样可以重复使用。 3. 应用现在可以显式控制何时将工作提交给 GPU。 这可实现第 1 项和第 2 项。  
</dl>

-   [删除即时上下文](#removal-of-the-immediate-context)
-   [GPU 工作项的分组](#grouping-of-gpu-work-items)
-   [GPU 工作提交](#gpu-work-submission)
-   [相关主题](#related-topics)

## <a name="removal-of-the-immediate-context"></a>删除即时上下文

从 Microsoft Direct3D 11 到 Microsoft Direct3D 12 的最大变化是，不再存在与设备关联的单个即时上下文。 相反，为了渲染，应用创建命令列表，在其中可以调用传统的渲染 API。 命令列表看起来类似于使用即时上下文的 Direct3D 11 应用的渲染方法，因为它包含绘制基元或更改渲染状态的调用。 像即时上下文一样，每个命令列表都不是自由线程；但是，可以同时记录多个命令列表，这利用了现代的多核处理器。

命令列表通常执行一次。 但是，如果应用程序在提交新执行之前确保先前的执行完成，则可以多次执行命令列表。 有关命令列表同步的更多信息，请参阅[执行和同步命令列表](executing-and-synchronizing-command-lists.md)。

## <a name="grouping-of-gpu-work-items"></a>GPU 工作项的分组

除命令列表之外，Direct3D 12 还通过添加第二级命令列表（称为“捆绑包”）来利用当前所有硬件中的功能  。 为了帮助区分这两种类型，可以将第一级命令列表称为“直接命令列表”  。 捆绑包的目的是允许应用将少量 API 命令组合在一起，以便以后从直接命令列表中重复执行。 在创建捆绑包时，驱动程序将执行尽可能多的预处理，以使以后的执行更高效。 然后可以在多个命令列表中执行捆绑包，并在同一命令列表中多次执行捆绑包。

捆绑包的重用是单 CPU 线程提高效率的重要驱动程序。 由于捆绑包是预处理的，可以多次提交，因此对捆绑包中可以执行的操作有一定的限制。 有关详细信息，请参阅[创建和记录命令列表与捆绑包](recording-command-lists-and-bundles.md)。

## <a name="gpu-work-submission"></a>GPU 工作提交

要在 GPU 上执行工作，应用必须显式地将命令列表提交到与 Direct3D 设备关联的命令队列。 直接命令列表可以多次提交执行，但应用负责确保直接命令列表在再次提交之前已在 GPU 上完成执行。 捆绑包没有并发使用限制，可以在多个命令列表中多次执行，但是捆绑包不能直接提交给命令队列执行。

任何线程都可以随时向任何命令队列提交命令列表，运行时将自动序列化命令队列中的命令列表提交，同时保留提交顺序。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 中的工作提交](command-queues-and-command-lists.md)
</dt> </dl>

 

 




