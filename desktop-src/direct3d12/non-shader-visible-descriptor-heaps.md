---
title: 非着色器可见描述符堆
description: 着色器无法通过描述符表引用部分描述符堆，但描述符堆的存在是为了帮助应用在记录命令列表之前暂存描述符，或者是因为不需着色器可见堆。
ms.assetid: 85934873-8889-4564-A717-28A00614B38C
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 84cd7536ec923e33fac046ed5c5bfe020fae1a11
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71005852"
---
# <a name="non-shader-visible-descriptor-heaps"></a>非着色器可见描述符堆

着色器无法通过描述符表引用部分描述符堆，但描述符堆的存在是为了帮助应用在记录命令列表之前暂存描述符，或者是因为不需着色器可见堆。

-   [非可见视图](#non-visible-views)
-   [摘要](#summary)
-   [相关主题](#related-topics)

## <a name="non-visible-views"></a>非可见视图

前面描述的着色器可访问描述符堆等所有描述符堆，都可以由 CPU 和/或命令列表操作，具体取决于应用程序为描述符堆选择的内存池和 CPU 访问属性。

对于[着色器可见描述符堆](shader-visible-descriptor-heaps.md)，拒绝着色器访问这些描述符堆的显著原因是它们处于暂存状态。 然后这些堆设置为着色器可见，并在命令列表执行时通过描述符表进行访问。 但暂存着色器可见堆没有要求，可直接填充。

其他描述符通过将其内容直接记录到命令列表来绑定到管道。 这些描述符只用于在命令列表记录时间时转换视图参数。 这些堆始终是非着色器可见，且包含以下内容。

-   呈现目标视图 (RTV)
-   深度模具视图 (DSV)
-   流输出视图 (SOV)

索引缓冲区视图 (IBV) 和顶点缓冲区视图 (VBV) 直接传递到 API 方法，且不具有特定堆类型。

记录到命令列表（例如，通过 [OMSetRenderTargets](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetrendertargets) 等调用）之后，用于保存此调用的描述符的内存在该调用结束后可立即重新使用。

甚至描述符表也有选项，即应用可以允许实现在命令列表记录时记录表内容（而不是在执行时取消对表指针的引用）。

## <a name="summary"></a>总结



|                   |                                    |                                        |
|-------------------|------------------------------------|----------------------------------------|
|                   | **着色器可见，CPU 只写** | **非着色器可见，CPU 读/写** |
| **CBV、SRV、UAV** | 是                                | 是                                    |
| **SAMPLER**       | 是                                | 是                                    |
| **SOV**           | 否                                 | 是                                    |
| **RTV**           | 否                                 | 是                                    |
| **DSV**           | 否                                 | 是                                    |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符堆](descriptor-heaps.md)
</dt> </dl>

 

 




