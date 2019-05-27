---
title: 呈现
description: 本部分包含有关 Direct3D 12 （和 Direct3D 11.3） 到新的呈现功能的信息。
ms.assetid: 5BF1440E-E4D8-43C8-BF0E-F02FEFE79C93
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 5b1083a733c641f92fbf3b2fb3a5b348d30ee996
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224206"
---
# <a name="rendering"></a>呈现

本部分包含有关 Direct3D 12 （和 Direct3D 11.3） 到新的呈现功能的信息。

## <a name="in-this-section"></a>本部分内容



| 主题                                                                                               | 描述                                                                                                                                                                                                                                                                                                                                                                                  |
|-----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [传统型光栅化](conservative-rasterization.md)<br/>                             | 传统型光栅化将某些颇有自信地添加到像素呈现，冲突检测算法尤其是有帮助。<br/>                                                                                                                                                                                                                                              |
| [间接绘图](indirect-drawing.md)<br/>                                                 | 间接绘图，一些场景遍历和消除从 CPU 移动到 GPU，这可以提高性能。 可以通过 CPU 或 GPU 生成命令缓冲区。<br/>                                                                                                                                                                                              |
| [光栅器有序视图](rasterizer-order-views.md)<br/>                                   | 光栅器有序视图 (ROVs) 允许像素着色器代码将标记 UAV 绑定与更改 Uav 的常规要求图形管道结果的顺序的声明。 这使顺序独立透明度 (OIT) 算法来工作，从而提供更好的呈现结果时多个透明对象是根据每个其他视图中。 <br/> |
| [着色器指定模具参考值](shader-specified-stencil-reference-value.md)<br/> | 启用到输出模具参考值，像素着色器，而不是使用 API 指定一个允许对模具操作非常精确地精确控制。<br/>                                                                                                                                                                                                              |
| [交换链](swap-chains.md)<br/>                                                           | 交换链来控制后台缓冲区旋转，图形动画的基础。<br/>                                                                                                                                                                                                                                                                                            |



 

以下主题还不熟悉 Direct3D 12 和 Direct3D 11.3:

-   [默认纹理映射](default-texture-mapping.md)
-   [类型化无序的访问视图加载](typed-unordered-access-view-loads.md)
-   [卷平铺资源](volume-tiled-resources.md)

## <a name="high-dynamic-range-and-wide-color-gamut"></a>高动态范围和宽颜色域

对高动态范围 （最聪明的人员的白色和最深黑之间增加差异） 和宽颜色域 （10 位，而非 8 位，每个颜色） 中所述的支持是指[DXGI 1.5 改进](https://msdn.microsoft.com/library/windows/desktop/mt661818)。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 的编程指南](directx-12-programming-guide.md)
</dt> </dl>

 

 





