---
title: 硬件功能级别
description: 描述的功能的 11\_0 到 12\_1 的硬件功能级别。
ms.assetid: B8304E29-A532-464E-8FAF-075228D8FB73
keywords:
- DX 功能级别
- DirectX 功能级别
- DX 的功能级别
- DirectX 的功能级别
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: d039c0efaae8bd94a8b0d4ca42f5b92809d9b7b0
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223750"
---
# <a name="hardware-feature-levels"></a>硬件功能级别

描述的功能的 11\_0 到 12\_1 的硬件功能级别。

-   [编号系统](#numbering-systems)
-   [功能级别支持](#feature-level-support)
-   [DXGI 格式的硬件支持](#hardware-support-for-dxgi-formats)
-   [相关的主题](#related-topics)

若要处理新的和现有机中视频卡的多样性，Microsoft Direct3D 11 引入的功能级别的概念。 每个视频卡实现一定的具体取决于图形处理单元 (Gpu) 安装的 Microsoft DirectX (DX) 功能。 功能级别是一组定义完善的 GPU 功能。 例如，11\_0 功能级别实现在 Direct3D 11 中实现的功能。

现在时创建的设备，你可以尝试创建你想要请求的功能级别的设备。 如果设备创建的工作原理，该功能级别存在，如果没有，硬件不支持该功能级别。 您可以尝试重新创建在较低功能级别的设备也可以选择退出该应用程序。

功能级别的基本属性包括：

-   Direct3D 12 的所有驱动程序将功能级别 11.0 或更好的。
-   允许在设备创建 GPU 达到或超过该功能级别的功能。
-   功能级别始终包括的上一个或更低功能级别的功能。
-   功能级别并不表示性能，仅功能。 性能是依赖于硬件实现。
-   在调用时，选择功能级别[ **D3D12CreateDevice**](/windows/desktop/api/D3D12/nf-d3d12-d3d12createdevice)。
-   有关更多详细信息上支持的功能 (尤其是那些标记*可选*在下表中，这意味着，硬件可能支持的功能但不需要对) 调用[ **CheckFeatureSupport**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport)。

有关限制在特定的功能级别上创建非硬件类型的设备的信息，请参阅[限制创建 WARP 和引用设备](https://msdn.microsoft.com/library/windows/desktop/ff728764)。 有关引入的功能级别的详细信息，请参阅 Direct3D 11 文档上[Direct3D 功能级别](https://msdn.microsoft.com/library/windows/desktop/ff476876)。

## <a name="numbering-systems"></a>编号系统

硬件功能级别*不*API 版本相同。 例如，一个 D3D11.3 的 API，但没有任何 11\_3 硬件功能级别。 在中定义功能级别[ **D3D\_功能\_级别**](https://msdn.microsoft.com/library/windows/desktop/ff476329)枚举。

有三个不同的编号系统：

-   Direct3D 版本使用时间;例如，Direct3D 12.0。
-   着色器模型使用时间;例如，着色器的模型 5.1。
-   功能级别使用下划线;例如，功能级别 12\_0。

## <a name="feature-level-support"></a>功能级别支持

以下功能是可用于每个 Direct3D 功能级别。

在顶行标题是 Direct3D 功能级别。 在左侧的列标题是功能。



| 功能\\功能级别                                                                                                 | 12\_1⁰                    | 12\_0⁰                    | 11\_1¹                   | 11\_0                    |
|--------------------------------------------------------------------------------------------------------------------------|---------------------------|---------------------------|--------------------------|--------------------------|
| 着色器模型                                                                                                             | 5.1                       | 5.1                       | 5.1²                     | 5.1²                     |
| [资源绑定层](hardware-support.md)                                                                            | Tier2³                    | Tier2³                    | Tier1³                   | Tier1³                   |
| [平铺的资源](/windows/desktop/api/D3D12/ne-d3d12-d3d12_tiled_resources_tier)                                                                        | Tier2³                    | Tier2³                    | 可选                 | 可选                 |
| [传统型光栅化](conservative-rasterization.md)                                                             | Tier1³                    | 可选                  | 可选                 | 否                       |
| [光栅器有序视图](rasterizer-order-views.md)                                                                   | 是                       | 可选                  | 可选                 | 否                       |
| [最小/最大的筛选器](/windows/desktop/api/D3D12/ne-d3d12-d3d12_filter)                                                                                      | 是                       | 是                       | 可选                 | 否                       |
| 映射默认缓冲区                                                                                                       | 可选                  | 可选                  | 可选                 | 可选                 |
| [着色器指定模具参考值](shader-specified-stencil-reference-value.md)                                 | 可选                  | 可选                  | 可选                 | 否                       |
| [类型化无序的访问视图加载](typed-unordered-access-view-loads.md)                                               | 多个可选的 18 格式 | 多个可选的 18 格式 | 多个可选的 3 个格式 | 多个可选的 3 个格式 |
| [几何着色器](https://msdn.microsoft.com/library/windows/desktop/bb205146#geometry-shader-stage) | 是                       | 是                       | 是                      | 是                      |
| [扩展 Stream](https://msdn.microsoft.com/library/windows/desktop/bb205121)                                            | 是                       | 是                       | 是                      | 是                      |
| [DirectCompute 计算着色器 /](https://msdn.microsoft.com/library/windows/desktop/ff476331)                                  | 是                       | 是                       | 是                      | 是                      |
| [包和域着色器](https://msdn.microsoft.com/library/windows/desktop/ff476340)                                           | 是                       | 是                       | 是                      | 是                      |
| [纹理资源数组](https://msdn.microsoft.com/library/windows/desktop/ff476906)                                     | 是                       | 是                       | 是                      | 是                      |
| [立方体贴图资源数组](https://msdn.microsoft.com/library/windows/desktop/ff476906)                                     | 是                       | 是                       | 是                      | 是                      |
| [BC1 BC7 压缩](https://msdn.microsoft.com/library/windows/desktop/bb694531)                        | 是                       | 是                       | 是                      | 是                      |
| [Alpha 覆盖率](https://msdn.microsoft.com/library/windows/desktop/bb205072#alpha-to-coverage)         | 是                       | 是                       | 是                      | 是                      |
| [逻辑运算 （输出合并器）](https://msdn.microsoft.com/library/windows/desktop/hh404457)                                          | 是                       | 是                       | 是                      | 可选                 |
| 独立于目标的光栅化                                                                                         | 是                       | 是                       | 是                      | 否                       |
| [多个呈现 target(MRT) ForcedSampleCount 1](https://msdn.microsoft.com/library/windows/desktop/hh404457)                      | 是                       | 是                       | 是                      | 可选                 |
| [最大强制仅 UAV 呈现的样本数](https://msdn.microsoft.com/library/windows/desktop/hh404457)                            | 16                        | 16                        | 16                       | 8                        |
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
| 一次镜像                                                                                                              | 是                       | 是                       | 是                      | 是                      |
| 重叠的顶点元素                                                                                              | 是                       | 是                       | 是                      | 是                      |
| 独立于编写掩码                                                                                                  | 是                       | 是                       | 是                      | 是                      |
| 实例化                                                                                                               | 是                       | 是                       | 是                      | 是                      |



 

-   ⁰ 需要 Direct3D 11.3 或 Direct3D 12 的运行时。
-   ¹ 需要 Direct3D 11.1 运行时。
-   ² 着色器模型 5.0 可以选择性地支持扩展双精度着色器的双精度着色器**SAD4**着色器的说明和部分精度着色器。 若要确定可用的着色器模型 5.0 选项，请调用[ **ID3D12Device::CheckFeatureSupport**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport)。 某些兼容性取决于哪些硬件上运行：支持 DirectX 12 API，而不考虑正在使用的功能级别的硬件上仅支持着色器模型 5.1。 着色器模型 5.0 多仅支持 DirectX 11 硬件。 DirectX 12 API 仅关闭到功能级别为 11\_0。
-   ³ 更高的层是可选的。
-   功能级别为 12.0 和 12.1 需要 Direct3D 11.3 或 Direct3D 12 的运行时。
-   功能级别 11.1 需要 Direct3D 11.1 运行时。
-   功能级别 11.0 需要 Direct3D 11.0 运行时。

## <a name="hardware-support-for-dxgi-formats"></a>DXGI 格式的硬件支持

若要查看的 DXGI 格式和硬件功能的表，请参阅：

-   [对 Direct3D 功能级别 12.1 硬件的 DXGI 格式支持](https://msdn.microsoft.com/library/windows/desktop/mt426648)
-   [对 Direct3D 功能级别 12.0 硬件的 DXGI 格式支持](https://msdn.microsoft.com/library/windows/desktop/mt426647)
-   [对 Direct3D 功能级别 11.1 硬件的 DXGI 格式支持](https://msdn.microsoft.com/library/windows/desktop/mt427456)
-   [对 Direct3D 功能级别 11.0 硬件的 DXGI 格式支持](https://msdn.microsoft.com/library/windows/desktop/mt427455)
-   [硬件支持 Direct3D 10Level9 格式](https://msdn.microsoft.com/library/windows/desktop/ff471324)
-   [硬件支持的 Direct3D 10.1 格式](https://msdn.microsoft.com/library/windows/desktop/cc627091)
-   [硬件支持的 Direct3D 10 格式](https://msdn.microsoft.com/library/windows/desktop/cc627090)

## <a name="related-topics"></a>相关主题

<dl> <dt>

[查询功能](capability-querying.md)
</dt> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> </dl>

 

 




