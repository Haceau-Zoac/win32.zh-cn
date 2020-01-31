---
title: 设置和填充描述符堆
description: 可在命令列表上设置的描述符堆类型包括可使用描述符表（每次最多使用一个表）的描述符。
ms.assetid: F0FF3D7C-1DAC-48C3-B47D-0378BE369F37
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: c0e3a1b6c4863827e1d8e1bdfb33a0a64420211d
ms.sourcegitcommit: 584b35959515bc5e4a104e8cf5270513f146f590
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/03/2019
ms.locfileid: "74709680"
---
# <a name="setting-and-populating-descriptor-heaps"></a>设置和填充描述符堆

可在命令列表上设置的描述符堆类型包括可使用描述符表（每次最多使用一个表）的描述符。

-   [设置描述符堆](#setting-and-populating-descriptor-heaps)
-   [填充描述符堆](#populating-descriptor-heaps)
-   [相关主题](#related-topics)

## <a name="setting-descriptor-heaps"></a>设置描述符堆

可在命令列表上设置的描述符堆类型包括：

``` syntax
D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV
D3D12_DESCRIPTOR_HEAP_TYPE_SAMPLER
```

命令列表上设置的堆还必须已创建为着色器可见。 有三种类型的命令列表：直接、绑定和计算。

在命令列表上设置描述符堆之后，定义描述符表的后续调用将引用当前的描述符堆。 在命令列表的开头以及描述符堆在命令列表上更改之后，描述符表状态为未定义。 重复设置同一描述符堆不会导致描述符表设置变为未定义。

相比之下，在 bundle 中，描述符堆只能设置一次（重复调用设置同一堆两次不会产生错误）；否则，行为是未定义的。 设置的描述符堆必须与任何命令列表调用 bundle 时的状态相匹配；否则，行为是未定义的。 因此，bundle 可继承和编辑命令列表的描述符表设置。 不更改（只继承）描述符表的 bundle 根本无需设置描述符堆，可直接继承自调用命令列表。

（使用 [**ID3D12GraphicsCommandList::SetDescriptorHeaps**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps)）设置描述符堆时，所用的所有堆都会通过一个调用进行设置（并且会通过该调用取消之前所设置的所有堆的设置）。 通过调用最多可设置上述每种类型的一个堆。

## <a name="populating-descriptor-heaps"></a>填充描述符堆

应用程序创建了描述符堆后，就可以使用设备上的方法直接在堆中生成描述符或将描述符从一个位置复制到另一个位置。

描述符堆内存的初始内容是未定义的，因此请求 GPU 或驱动程序引用未初始化的内存进行渲染可能产生设备重置等未定义的结果。

如果应用程序将描述符堆配置为 CPU 可见，则 CPU 可以调用方法将描述符创建到堆中，并以即时、自由线程方式从某一位置复制到另一位置（包括跨堆）。 如果堆已配置为 SHADER_VISIBLE，则不允许由 CPU 读取。



## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符堆](descriptor-heaps.md)
</dt> </dl>

 

 




