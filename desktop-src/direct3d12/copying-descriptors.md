---
title: 复制描述符
description: 设备接口上的 ID3D12Device CopyDescriptors 和 ID3D12Device CopyDescriptorsSimple 方法使用 CPU 立即复制描述符。
ms.assetid: 65AE4D96-6221-46B5-BF55-F86479FCF97C
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 7930bd43f59c03cc4e7e11936ca97610ee986dda
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296060"
---
# <a name="copying-descriptors"></a>复制描述符

设备接口上的 [ID3D12Device::CopyDescriptors](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-copydescriptors) 和 [ID3D12Device::CopyDescriptorsSimple](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-copydescriptorssimple) 方法使用 CPU 立即复制描述符   。 只要 CPU 或 GPU 上的多个线程不执行任何可能存在冲突的写入，就可以将它们称为自由线程。

## <a name="copying-descriptors-immediately-cpu-timeline"></a>立即复制描述符（CPU 时间线）

指定为一组描述符范围的源描述符（复制源）的数目必须等于指定为一组单独的描述符范围的目标描述符（复制到的）的数目。 源和目标范围不必对齐。 例如，稀疏的一组描述符可以复制到相邻的目的地，反之亦然，或者以某种组合形式复制。

在复制操作中可能会涉及到作为源和目标的多个描述符堆。 使用描述符句柄作为参数意味着复制方法不关心任何给定描述符所在的堆 - 它们都只是内存。

要从中复制以及复制到其中的描述符堆类型必须匹配，因此方法将单个描述符堆类型作为输入。 驱动程序需要知道给定复制操作中所有描述符的堆类型，以便知道复制操作涉及的数据大小。 如果给定描述符堆类型保证了实现细节，驱动程序可能还需要进行自定义复制工作。 请注意，描述符句柄本身没有标识它们所指向的类型；因此，复制操作需要一个附加参数。

如果只是将单个描述符范围从一个位置复制到另一个位置，则可使用 [CopyDescriptors](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-copydescriptors) 的替代 API - [CopyDescriptorsSimple](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-copydescriptorssimple)   。

对于这些基于设备的（CPU 时间轴）描述符复制方法，源描述符必须来自非着色器可见描述符堆。 目标描述符可以位于 CPU 可见的任何描述符堆中（着色器可见或不可见）。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符](descriptors.md)
</dt> </dl>

 

 




