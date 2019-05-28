---
title: 光栅器有序视图
description: 光栅器排序视图 (ROVs) 允许像素着色器代码将标记与更改 Uav 的常规要求图形管道结果的顺序的声明的无序的访问视图绑定。
ms.assetid: D308BF3E-8CBE-4DF0-B020-4D202E858D99
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 58c93f5c0b2c62eb6e50040e4425cc9b9cb13a7f
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224236"
---
# <a name="rasterizer-ordered-views"></a>光栅器有序视图

光栅器排序视图 (ROVs) 允许像素着色器代码将标记与更改 Uav 的常规要求图形管道结果的顺序的声明的无序的访问视图 (UAV) 绑定。 这样，独立于订单的透明度 (OIT) 算法来工作，从而提供更好的呈现结果时多个透明对象是根据每个其他视图中。

-   [概述](#overview)
-   [实现详细信息](#implementation-details)
-   [API 摘要](#api-summary)
-   [相关的主题](#related-topics)

## <a name="overview"></a>概述

标准图形管道可能有故障正确组合在一起包含透明的多个纹理。 对象，如网络界定、 烟雾、 焰火、 vegetation 和彩色的玻璃使用透明度，若要获取所需的效果。 根据彼此 （冒烟前面玻璃构建包含 vegetation，例如前面 fence） 包含透明的多个纹理时出现问题。 光栅器排序视图 (ROVs) 使基础的 OIT 算法，若要使用的硬件功能来尝试解决的透明度顺序正确。 透明度由像素着色器处理。

光栅器排序视图 (ROVs) 允许像素着色器代码将标记 UAV 绑定与更改 Uav 的常规要求图形管道结果的顺序的声明。

ROVs 保证任何一对重叠像素着色器调用 UAV 访问的顺序。 在这种情况下进行的调用都由相同的"重叠"表示绘图调用和共享在像素频率执行模式下，相同的像素坐标和相同的像素和示例协调在采样频率模式下。

重叠 ROV 访问的像素着色器执行调用的顺序与在其中提交几何图形的顺序完全相同。 这意味着，对于重叠像素着色器调用，像素着色器调用所执行的 ROV 写入必须有可供读取的后续调用不能通过在前一个调用影响读取。 像素着色器调用所执行的 ROV 读取必须反映在前一个调用写入，并且必须通过后续调用反映写入。 这是对于 Uav 重要，因为显式省略，则从由图形管道结果的固定顺序通常设置的输出不变性保证。

## <a name="implementation-details"></a>实现详细信息

光栅器排序视图 (ROVs) 以下列新的高级别着色器语言 (HLSL) 对象声明，并且仅对像素着色器是可用：

-   `RasterizerOrderedBuffer`
-   `RasterizerOrderedByteAddressBuffer`
-   `RasterizerOrderedStructuredBuffer`
-   `RasterizerOrderedTexture1D`
-   `RasterizerOrderedTexture1DArray`
-   `RasterizerOrderedTexture2D`
-   `RasterizerOrderedTexture2DArray`
-   `RasterizerOrderedTexture3D`

在与其他 UAV 对象相同的方式使用这些对象 (如`RWBuffer`等。)。

## <a name="api-summary"></a>API 摘要

ROVs 是向 Uav 应用不同的行为语义的仅限 HLSL 的构造。 Uav 相关的所有 Api 都都还与 ROVs 相关。 请注意下面的方法、 结构和帮助器类参考光栅器：

-   [**D3D12\_光栅器\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_rasterizer_desc) ： 保存光栅器说明的结构。
-   [**D3D12\_功能\_数据\_D3D12\_选项**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_d3d12_options) ： 存放一个布尔值，该值指示支持结构。
-   [**CheckFeatureSupport** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport) ： 方法访问受支持的功能。
-   [**CD3DX12\_光栅器\_DESC** ](cd3dx12-rasterizer-desc.md) ： 帮助器类，用于创建光栅器说明。
-   [**D3D12\_图形\_管道\_状态\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_graphics_pipeline_state_desc) ： 存放管道状态结构。

## <a name="related-topics"></a>相关主题

* [传统型光栅化](conservative-rasterization.md)
* [呈现](rendering.md)
* [在 HLSL 中绑定的资源](resource-binding-in-hlsl.md)
* [着色器模型 5.1](https://msdn.microsoft.com/library/windows/desktop/dn933277)
