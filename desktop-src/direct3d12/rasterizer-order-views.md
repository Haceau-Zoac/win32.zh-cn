---
title: 光栅器有序视图
description: 像素着色器代码可通过光栅器有序视图 (ROV) 使用声明来标记无序访问视图绑定，该声明改变了 UAV 图形管道结果顺序的正常要求。
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

像素着色器代码可通过光栅器有序视图 (ROV) 使用声明来标记无序访问视图 (UAV) 绑定，该声明改变了 UAV 图形管道结果顺序的正常要求。 这可确保顺序无关透明度 (OIT) 算法能够正常工作，多个透明对象在视图中彼此对齐时，可以提供更好的渲染效果。

-   [概述](#overview)
-   [实现详细信息](#implementation-details)
-   [API 总结](#api-summary)
-   [相关主题](#related-topics)

## <a name="overview"></a>概述

标准图形管道在正确组合包含透明度的多个纹理时可能会遇到困难。 铁丝网、烟、火、植物和彩色玻璃等物体使用透明度来达到预期效果。 包含透明度的多个纹理彼此成一条线时（例如，包含植被的玻璃建筑物前面有栅栏，而栅栏前面有烟雾），将会出现问题。 光栅器有序视图 (ROV) 确保基础 OIT 算法能够使用硬件功能来尝试正确解析透明度顺序。 透明度由像素着色器处理。

像素着色器代码可通过光栅器有序视图 (ROV) 使用声明来标记 UAV 绑定，该声明改变了 UAV 图形管道结果顺序的正常要求。

ROV 确保任何一对重叠像素着色器调用的 UAV 访问顺序。 在此情况下，“重叠”是指调用由相同的绘制调用生成，且在像素频率执行模式下共享同一像素坐标，在采样频率执行模式下共享同一像素和采样坐标。

像素着色器调用的重叠 ROV 访问的执行顺序与提交几何图形的顺序相同。 这意味着，对于重叠的像素着色器调用而言，像素着色器调用所执行的 ROV 写入必须可由后续调用读取，且不能影响前一次调用的读取。 像素着色器调用所执行的 ROV 读取必须反射前一次调用的写入，且不能反射后续调用的写入。 这对 UAV 很重要，因为会将其从输出不变性保证中显式省略，而输出不变性保证通常按照图形管道结果的固定顺序设置。

## <a name="implementation-details"></a>实现详细信息

光栅器有序视图 (ROV) 是使用以下新的高级着色器语言 (HLSL) 对象声明的，并且仅用于像素着色器：

-   `RasterizerOrderedBuffer`
-   `RasterizerOrderedByteAddressBuffer`
-   `RasterizerOrderedStructuredBuffer`
-   `RasterizerOrderedTexture1D`
-   `RasterizerOrderedTexture1DArray`
-   `RasterizerOrderedTexture2D`
-   `RasterizerOrderedTexture2DArray`
-   `RasterizerOrderedTexture3D`

按使用其他 UAV 对象的方式来使用这些对象（如 `RWBuffer` 等）。

## <a name="api-summary"></a>API 总结

ROV 是仅适用于 HLSL 的结构，该结构将不同的行为语义应用到 UAV。 与 UAV 相关的所有 API 也与 ROV 相关。 注意，以下方法、结构和帮助程序类引用了光栅化器：

-   [D3D12\_RASTERIZER\_DESC](/windows/desktop/api/D3D12/ns-d3d12-d3d12_rasterizer_desc)：保存光栅器描述的结构  。
-   [D3D12\_FEATURE\_DATA\_D3D12\_OPTIONS](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_d3d12_options)：保存表示支持的布尔值的结构  。
-   [CheckFeatureSupport](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport)：访问支持功能的方法  。
-   [CD3DX12\_RASTERIZER\_DESC](cd3dx12-rasterizer-desc.md)：用于创建光栅器描述的帮助程序类  。
-   [D3D12\_GRAPHICS\_PIPELINE\_STATE\_DESC](/windows/desktop/api/D3D12/ns-d3d12-d3d12_graphics_pipeline_state_desc)：保存管道状态的结构  。

## <a name="related-topics"></a>相关主题

* [传统型光栅化](conservative-rasterization.md)
* [渲染](rendering.md)
* [HLSL 中的资源绑定](resource-binding-in-hlsl.md)
* [Shader Model 5.1](https://msdn.microsoft.com/library/windows/desktop/dn933277)（着色器模型 5.1）
