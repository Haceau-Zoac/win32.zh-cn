---
title: 复制描述符
description: 设备接口上的 ID3D12Device CopyDescriptors 和 ID3D12Device CopyDescriptorsSimple 方法使用 CPU 立即复制描述符。
ms.assetid: 65AE4D96-6221-46B5-BF55-F86479FCF97C
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: ab0e925e63a2aa3becf75128774a2144b03bc341
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224284"
---
# <a name="copying-descriptors"></a>复制描述符

[ **ID3D12Device::CopyDescriptors** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-copydescriptors)并[ **ID3D12Device::CopyDescriptorsSimple** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-copydescriptorssimple)设备上的方法的接口使用若要立即复制描述符的 CPU。 可以调用它们可用线程，只要 CPU 或 GPU 上的多个线程不执行任何可能会有冲突的写入操作。

## <a name="copying-descriptors-immediately-cpu-timeline"></a>复制立即描述符 （CPU 时间线）

（若要从复制） 的源描述符，数指定为一组的描述符范围、 目标描述符 （若要将复制到） 的数目必须相等，指定为一组单独的描述符范围。 源和目标范围否则不需要向上移动一行。 例如，稀疏描述符无法进行复制的到连续的目标，反之亦然，或某种组合中。

多个描述符堆可以参与复制操作，同时为源和目标。 使用作为参数的描述符句柄意味着复制方法并不关心哪些堆有关任何给定的描述符在于 – 它们是所有的不仅仅是内存。

要从 / 向复制的描述符堆类型必须匹配，因此方法采用单个描述符堆类型作为输入。 驱动程序需要知道的给定的复制操作中的所有描述符堆类型，因此它知道在复制操作中涉及的数据大小。 该驱动程序可能还需要执行自定义的复制工作，如果给定的描述符堆类型担保 – 实现详细信息。 请注意，描述符句柄本身不能否则标识它们指向; 哪种类型因此，还会复制操作所必需的其他参数。

到一个备用 API [ **CopyDescriptors** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-copydescriptors)提供的复制一个简单的情况下的描述符从一个位置到另一-范围[ **CopyDescriptorsSimple**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-copydescriptorssimple)。

对于这些设备基于 （CPU 时间线） 描述符复制方法，源描述符必须来自于非着色器可见描述符堆。 目标描述符可以是任何是可见的 CPU 的描述符堆中 (着色器可见或不)。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符](descriptors.md)
</dt> </dl>

 

 




