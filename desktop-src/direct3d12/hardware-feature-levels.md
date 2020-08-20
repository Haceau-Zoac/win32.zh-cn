---
title: 硬件功能级别
description: 描述 11\_0 到 12\_1 硬件功能级别的功能。
ms.assetid: B8304E29-A532-464E-8FAF-075228D8FB73
keywords:
- DX 功能级别
- DirectX 功能级别
- 功能级别, DX
- 功能级别, DirectX
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 3f44103227397b57139a92921a2c1b5ca2d10f2f
ms.sourcegitcommit: 592c9bbd22ba69802dc353bcb5eb30699f9e9403
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2020
ms.locfileid: "88644067"
---
# <a name="hardware-feature-levels"></a>硬件功能级别

描述 11\_0 到 12\_1 硬件功能级别的功能。

-   [编号系统](#numbering-systems)
-   [功能级别支持](#feature-level-support)
-   [DXGI 格式的硬件支持](#hardware-support-for-dxgi-formats)
-   [相关主题](#related-topics)

若要处理新的或现有计算机中的各类视频卡，Microsoft Direct3D 11 引入了功能级别的概念。 每个视频卡实现某个级别的 Microsoft DirectX (DX) 功能，具体取决于所安装的图形处理单元 (GPU)。 功能级别是明确定义的 GPU 功能的集合。 例如，11\_0 功能级别实现在 Direct3D 11 中实现的功能。

现在，在创建设备时，你可以尝试为想要请求的功能级别创建设备。 如果设备创建成功，该功能级别将存在，如果失败，硬件将不支持该功能级别。 你可以尝试在更低的功能级别重新创建设备，也可以选择退出应用程序。

功能级别的基本属性包括：

-   所有 Direct3D 12 驱动程序将是功能级别 11.0 或更好。
-   允许创建设备的 GPU 达到或超过该功能级别的功能。
-   功能级别始终包括上一个或更低功能级别的功能。
-   功能级别并不意味着性能，而只是功能。 性能依赖于硬件实现。
-   调用 [**D3D12CreateDevice**](/windows/desktop/api/d3d12/nf-d3d12-d3d12createdevice) 时将选择功能级别。
-   有关支持的功能（尤其是下表中标记为“可选”** 的功能，这意味着硬件可能支持功能，但不是必需的）的更多详细信息，请调用 [**CheckFeatureSupport**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)。

有关在特定功能级别创建非硬件类型的设备的限制的信息，请参阅[有关创建 WARP 和引用设备的限制](/windows/desktop/direct3d11/overviews-direct3d-11-devices-limitations)。 有关引入功能级别的详细信息，请参阅 [Direct3D 功能级别](/windows/desktop/direct3d11/overviews-direct3d-11-devices-downlevel-intro)上的 Direct3D 11 文档。

## <a name="numbering-systems"></a>编号系统

硬件功能级别与 API 版本不同**。 例如，有一个 D3D11.3 API，但没有 11\_3 硬件功能级别。 功能级别在 [**D3D\_FEATURE\_LEVEL**](/windows/desktop/api/d3dcommon/ne-d3dcommon-d3d_feature_level) 枚举中定义。

有三个不同的编号系统：

-   Direct3D 版本使用句点；例如，Direct3D 12.0。
-   着色器模型使用句点；例如，着色器模型 5.1。
-   功能级别使用下划线；例如，功能级别 12\_0。

## <a name="feature-level-support"></a>功能级别支持

以下功能可用于每个 Direct3D 功能级别。

首行标题是 Direct3D 功能级别。 左侧列标题是功能。



| 功能 \\ 功能级别                                                                                                 | 12\_1⁰                    | 12\_0⁰                    | 11\_1¹                   | 11\_0                    |
|--------------------------------------------------------------------------------------------------------------------------|---------------------------|---------------------------|--------------------------|--------------------------|
| 着色器模型                                                                                                             | 5.1                       | 5.1                       | 5.1²                     | 5.1²                     |
| [资源绑定层](hardware-support.md)                                                                            | Tier2³                    | Tier2³                    | Tier1³                   | Tier1³                   |
| [平铺资源](/windows/desktop/api/d3d12/ne-d3d12-d3d12_tiled_resources_tier)                                                                        | Tier2³                    | Tier2³                    | 可选                 | 可选                 |
| [传统型光栅化](conservative-rasterization.md)                                                             | Tier1³                    | 可选                  | 可选                 | 否                       |
| [光栅器有序视图](rasterizer-order-views.md)                                                                   | “是”                       | 可选                  | 可选                 | 否                       |
| [最小/最大筛选器](/windows/desktop/api/d3d12/ne-d3d12-d3d12_filter)                                                                                      | 是                       | “是”                       | 可选                 | 否                       |
| 映射默认缓冲区                                                                                                       | 可选                  | 可选                  | 可选                 | 可选                 |
| [着色器指定的模具参考值](shader-specified-stencil-reference-value.md)                                 | 可选                  | 可选                  | 可选                 | 否                       |
| [类型化的无序访问视图加载](typed-unordered-access-view-loads.md)                                               | 18 种格式，多个可选 | 18 种格式，多个可选 | 3 种格式，多个可选 | 3 种格式，多个可选 |
| [几何着色器](/previous-versions//bb205146(v=vs.85)) | 是                       | 是                       | 是                      | 是                      |
| [流输出](/windows/desktop/direct3d11/d3d10-graphics-programming-guide-output-stream-stage)                                            | 是                       | 是                       | 是                      | 是                      |
| [DirectCompute/计算着色器](/windows/desktop/direct3d11/direct3d-11-advanced-stages-compute-shader)                                  | 是                       | 是                       | 是                      | 是                      |
| [外壳着色器和域着色器](/windows/desktop/direct3d11/direct3d-11-advanced-stages-tessellation)                                           | 是                       | 是                       | 是                      | 是                      |
| [纹理资源数组](/windows/desktop/direct3d11/overviews-direct3d-11-resources-textures-intro)                                     | 是                       | 是                       | 是                      | 是                      |
| [立方体贴图资源数组](/windows/desktop/direct3d11/overviews-direct3d-11-resources-textures-intro)                                     | 是                       | 是                       | 是                      | 是                      |
| [BC1 到 BC7 压缩](/windows/desktop/direct3d10/d3d10-graphics-programming-guide-resources-block-compression)                        | 是                       | 是                       | 是                      | 是                      |
| [Alpha 覆盖范围](/windows/desktop/direct3d11/d3d10-graphics-programming-guide-blend-state)         | 是                       | 是                       | 是                      | 是                      |
| [逻辑操作（输出合并器）](/windows/desktop/api/d3d11/ns-d3d11-d3d11_feature_data_d3d11_options)                                          | 是                       | 是                       | “是”                      | 可选                 |
| 独立于目标的光栅化                                                                                         | 是                       | 是                       | 是                      | 否                       |
| [带有 ForcedSampleCount 1 的多呈现器目标 (MRT)](/windows/desktop/api/d3d11/ns-d3d11-d3d11_feature_data_d3d11_options)                      | 是                       | 是                       | “是”                      | 可选                 |
| [仅 UAV 呈现的最大强制采样计数](/windows/desktop/api/d3d11/ns-d3d11-d3d11_feature_data_d3d11_options)                            | 16                        | 16                        | 16                       | 8                        |
| 最大纹理维度                                                                                                    | 16384                     | 16384                     | 16384                    | 16384                    |
| 最大立方体贴图维度                                                                                                    | 16384                     | 16384                     | 16384                    | 16384                    |
| 最大卷范围                                                                                                        | 2048                      | 2048                      | 2048                     | 2048                     |
| 最大纹理重复                                                                                                       | 16384                     | 16384                     | 16384                    | 16384                    |
| 最大各向异性                                                                                                           | 16                        | 16                        | 16                       | 16                       |
| 最大基元计数                                                                                                      | 2^32 – 1                  | 2^32 – 1                  | 2^32 – 1                 | 2^32 – 1                 |
| 最大顶点索引                                                                                                         | 2^32 – 1                  | 2^32 – 1                  | 2^32 – 1                 | 2^32 – 1                 |
| 最大输入槽                                                                                                          | 32                        | 32                        | 32                       | 32                       |
| 同时呈现器目标                                                                                              | 8                         | 8                         | 8                        | 8                        |
| 封闭查询                                                                                                        | 是                       | 是                       | 是                      | 是                      |
| 单独的 Alpha 混合                                                                                                     | 是                       | 是                       | 是                      | 是                      |
| 镜像一次                                                                                                              | 是                       | 是                       | 是                      | 是                      |
| 重叠顶点元素                                                                                              | 是                       | 是                       | 是                      | 是                      |
| 独立写掩码                                                                                                  | 是                       | 是                       | 是                      | 是                      |
| 实例化                                                                                                               | 是                       | 是                       | 是                      | 是                      |



 

-   ⁰ 需要 Direct3D 11.3 或 Direct3D 12 运行时。
-   ¹ 需要 Direct3D 11.1 运行时。
-   ² 着色器模型 5.0 可以选择性地支持双精度着色器、扩展的双精度着色器、SAD4**** 着色器说明和部分精度着色器。 若要确定可用的着色器模型 5.0 选项，请调用 [**ID3D12Device::CheckFeatureSupport**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)。 某些兼容性取决于运行的硬件：着色器模型5.1 仅在支持 DirectX 12 API 的硬件上受支持，而不考虑所使用的功能级别。 DirectX 11 硬件最多仅支持着色器模型 5.0。 DirectX 12 API 仅延续至功能级别 11\_0。
-   ³ 更高的层是可选的。
-   功能级别 12.0 和 12.1 需要 Direct3D 11.3 或 Direct3D 12 运行时。
-   功能级别 11.1 需要 Direct3D 11.1 运行时。
-   功能级别 11.0 需要 Direct3D 11.0 运行时。

## <a name="hardware-support-for-dxgi-formats"></a>DXGI 格式的硬件支持

若要查看 DXGI 格式和硬件功能的表，请参阅：

-   [Direct3D 功能级别 12.1 硬件的 DXGI 格式支持](/windows/desktop/direct3ddxgi/hardware-support-for-direct3d-12-1-formats)
-   [DXGI Format Support for Direct3D Feature Level 12.0 Hardware](/windows/desktop/direct3ddxgi/hardware-support-for-direct3d-12-0-formats)（Direct3D 功能级别 12.0 硬件的 DXGI 格式支持）
-   [DXGI Format Support for Direct3D Feature Level 11.1 Hardware](/windows/desktop/direct3ddxgi/format-support-for-direct3d-11-1-feature-level-hardware)（Direct3D 功能级别 11.1 硬件的 DXGI 格式支持）
-   [DXGI Format Support for Direct3D Feature Level 11.0 Hardware](/windows/desktop/direct3ddxgi/format-support-for-direct3d-11-0-feature-level-hardware)（Direct3D 功能级别 11.0 硬件的 DXGI 格式支持）
-   [Hardware Support for Direct3D 10Level9 Formats](/previous-versions//ff471324(v=vs.85))（Direct3D 10Level9 格式的硬件支持）
-   [Hardware Support for Direct3D 10.1 Formats](/previous-versions//cc627091(v=vs.85))（Direct3D 10.1 格式的硬件支持）
-   [Hardware Support for Direct3D 10 Formats](/previous-versions//cc627090(v=vs.85))（Direct3D 10 格式的硬件支持）

## <a name="related-topics"></a>相关主题

<dl> <dt>

[功能查询](capability-querying.md)
</dt> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> </dl>

 

 