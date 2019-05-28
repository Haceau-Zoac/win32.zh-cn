---
title: 非着色器可见描述符堆
description: 某些描述符堆着色器通过描述符表不能引用，但存在任何一个以在过渡环境中之前记录的命令列表的描述符帮助应用程序或由于没有着色器可见堆是必需的。
ms.assetid: 85934873-8889-4564-A717-28A00614B38C
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 7ed1bce7653e31cb798d0300ed2dc6e51d90b51d
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223876"
---
# <a name="non-shader-visible-descriptor-heaps"></a>非着色器可见描述符堆

某些描述符堆着色器通过描述符表不能引用，但存在任何一个以在过渡环境中之前记录的命令列表的描述符帮助应用程序或由于没有着色器可见堆是必需的。

-   [非可见视图](#non-visible-views)
-   [摘要](#summary)
-   [相关的主题](#related-topics)

## <a name="non-visible-views"></a>非可见视图

所有描述符堆的方法，包括堆前面所述的着色器可访问描述符可以操作由 CPU 和/或命令列出了具体取决于内存池和 CPU 访问属性，该应用程序选择的描述符堆。

有关[着色器可见描述符堆](shader-visible-descriptor-heaps.md)，最明显的原因来拒绝着色器访问这些描述符堆是，而它们进入临时状态。 然后这些堆进行着色器可见，并访问通过描述符表处执行命令列表。 但是，没有任何要求暂存着色器可见的堆，它们可以直接填充。

其他描述符获取绑定到管道具有直接到命令列表中记录其内容。 这些描述符只是为了在命令列表记录时转换视图参数。 这些堆始终是非着色器可见，并且包含以下。

-   呈现目标视图 (RTVs)
-   深度模具视图 (Dsv)
-   Stream 输出视图 (SOVs)

索引缓冲区视图 (IBVs) 和顶点缓冲区视图 (VBVs) 直接传递到 API 方法，不是具有特定的堆类型。

到命令列表中的录制完后 (如拨打[ **OMSetRenderTargets**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-omsetrendertargets)，例如) 用来保存描述符，此调用是立即可供在调用后重复使用的内存。

甚至描述符表具有选项应用可实现来选择记录命令列表录制中的表内容 （而非在执行时表指针取消引用）。

## <a name="summary"></a>摘要



|                   |                                    |                                        |
|-------------------|------------------------------------|----------------------------------------|
|                   | **着色器可见，CPU 只写的** | **非着色器可见，CPU 读/写** |
| **CBV, SRV, UAV** | 是                                | 是                                    |
| **SAMPLER**       | 是                                | 是                                    |
| **SOV**           | 否                                 | 是                                    |
| **RTV**           | 否                                 | 是                                    |
| **DSV**           | 否                                 | 是                                    |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符堆](descriptor-heaps.md)
</dt> </dl>

 

 




