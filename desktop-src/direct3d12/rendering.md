---
title: " (Direct3D 12 图形呈现) "
description: 本节介绍 Direct3D 12（和 Direct3D 11.3）中新增的渲染功能。
ms.assetid: 5BF1440E-E4D8-43C8-BF0E-F02FEFE79C93
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: d678d60ef9e6f5aabb59632ad5a107d820f2d59e
ms.sourcegitcommit: 592c9bbd22ba69802dc353bcb5eb30699f9e9403
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2020
ms.locfileid: "88644061"
---
# <a name="rendering"></a>渲染

本节介绍 Direct3D 12（和 Direct3D 11.3）中新增的渲染功能。

## <a name="in-this-section"></a>在本节中



| 主题                                                                                               | 说明                                                                                                                                                                                                                                                                                                                                                                                  |
|-----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [传统型光栅化](conservative-rasterization.md)<br/>                             | 保守光栅化为像素渲染增加了一定的确定性，这对碰撞检测算法特别有帮助。<br/>                                                                                                                                                                                                                                              |
| [间接绘制](indirect-drawing.md)<br/>                                                 | 通过间接绘制，某些场景遍历和精选可以从 CPU 移动到 GPU，从而可以提高性能。 可以通过 CPU 或 GPU 生成命令缓冲区。<br/>                                                                                                                                                                                              |
| [光栅器有序视图](rasterizer-order-views.md)<br/>                                   | 像素着色器代码可通过光栅器有序视图 (ROV) 使用声明来标记 UAV 绑定，该声明改变了 UAV 图形管道结果顺序的正常要求。 这可确保顺序无关透明度 (OIT) 算法能够正常工作，多个透明对象在视图中彼此对齐时，可以提供更好的渲染效果。 <br/> |
| [着色器指定的模具参考值](shader-specified-stencil-reference-value.md)<br/> | 启用像素着色器来输出模具参考值，而不是使用特定于 API 的模具参考值，可以对模具操作进行非常精细的粒度控制。<br/>                                                                                                                                                                                                              |
| [交换链](swap-chains.md)<br/>                                                           | 交换链控制反向缓冲区轮转，构成图形动画的基础。<br/>                                                                                                                                                                                                                                                                                            |



 

以下也是 Direct3D 12 和 Direct3D 11.3 中的新增主题：

-   [默认纹理映射](default-texture-mapping.md)
-   [类型化的无序访问视图加载](typed-unordered-access-view-loads.md)
-   [立体平铺资源](volume-tiled-resources.md)

## <a name="high-dynamic-range-and-wide-color-gamut"></a>高动态范围和宽色域

请参考 [DXGI 1.5 Improvements](/windows/desktop/direct3ddxgi/dxgi-1-5-improvements)（DXGI 1.5 改进）中对高动态范围（最亮的白色和最暗的黑色之间的增大的差异）和宽色域（每种颜色 10 位，而不是 8 位）的支持。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 编程指南](directx-12-programming-guide.md)
</dt> </dl>

 

