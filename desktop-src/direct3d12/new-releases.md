---
title: Direct3D 12 中的新增功能
description: 本主题介绍可用于各种版本的最重要的新 Direct3D 12 文档。
ms.assetid: 38F41E05-FECB-41DE-8D30-09733FBEAC48
ms.localizationpriority: high
ms.topic: article
ms.date: 12/05/2019
ms.openlocfilehash: ec3ecc9e68fc4711def2c364793eca32804d8d04
ms.sourcegitcommit: 592c9bbd22ba69802dc353bcb5eb30699f9e9403
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2020
ms.locfileid: "88644346"
---
# <a name="whats-new-in-direct3d-12"></a>Direct3D 12 中的新增功能

本主题介绍可用于各种版本的最重要的新 Direct3D 12 文档。

有关获取和安装 Direct3D 的信息，请参阅 [direct3d 12 编程环境设置](./directx-12-programming-environment-set-up.md)。

## <a name="direct3d-12-on-windows-7"></a>Windows 7 上的 Direct3D 12

- [基于 Windows 7 的 Direct3D 12](https://devblogs.microsoft.com/directx/porting-directx-12-games-to-windows-7/) 现在可供开发人员使用。

## <a name="windows-10-may-2019-update"></a>Windows 10 2019 年 5 月更新

为 Windows 10 （版本 1903 (10.0）添加或更新了这些功能和 Api。生成 18362) &mdash; 也称为 Windows 10 2019 更新。

- [可变速率底纹 (VRS) ](./vrs.md)。 使你能够以不同于呈现的图像的速率分配呈现性能/功率。
- [HLSL 着色器模型 6.4](../direct3dhlsl/hlsl-shader-model-6-4-features-for-direct3d-12.md)。 介绍添加到 HLSL 着色器模型6.4 的机器学习内部函数。
- [**D3D12_DRED_VERSION**](/windows/win32/api/d3d12/ne-d3d12-d3d12_dred_version) 枚举。 定义指定设备版本 (通过) 删除扩展数据的常量。
- [**D3D12_FEATURE_DATA_D3D12_OPTIONS6**](/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options6) 结构。 指示适配器为 metacommands 提供的支持级别。
- [**D3D12_FEATURE_DATA_QUERY_META_COMMAND**](/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_query_meta_command) 结构。 指示适配器为 metacommands 提供的支持级别。
- [**D3D12_VARIABLE_SHADING_RATE_TIER**](/windows/win32/api/d3d12/ne-d3d12-d3d12_variable_shading_rate_tier) 枚举。 定义用于指定可变速率底纹 (或 VRS) 的底纹速率层的常量。
- [**ID3D12Device6**](/windows/win32/api/d3d12/nn-d3d12-id3d12device6) 接口及其方法。 用于设置驱动程序后台处理优化模式。 另请参阅 [背景着色器优化](https://devblogs.microsoft.com/directx/background-shader-optimizations/)。
- [**ID3D12DeviceRemovedExtendedData**](/windows/win32/api/d3d12/nn-d3d12-id3d12deviceremovedextendeddata) 接口及其方法。 提供对设备的运行时访问 (通过) 数据的已删除扩展数据。
- [**ID3D12DeviceRemovedExtendedDataSettings**](/windows/win32/api/d3d12/nn-d3d12-id3d12deviceremovedextendeddatasettings) 接口及其方法。 控制设备 (通过) 设置中删除了扩展数据。
- [**D3D12GraphicsCommandList5**](/windows/win32/api/d3d12/nn-d3d12-id3d12graphicscommandlist5) 接口及其方法。 支持可变速率底纹 (VRS) 。

已更新 [**D3D_SHADER_MODEL**](/windows/win32/api/d3d12/ne-d3d12-d3d_shader_model) 枚举，同时 (实验级别功能) 添加了 **D3D_SHADER_MODEL_6_5** 常量。

已更新 [**D3D12_COMMAND_LIST_TYPE**](/windows/win32/api/d3d12/ne-d3d12-d3d12_command_list_type) 枚举，并增加了 **D3D12_COMMAND_LIST_TYPE_VIDEO_ENCODE** 常数。

已更新 [**D3D12_FEATURE**](/windows/win32/api/d3d12/ne-d3d12-d3d12_feature) 枚举，同时增加了 **D3D12_FEATURE_D3D12_OPTIONS6** 和 **D3D12_FEATURE_QUERY_META_COMMAND** 常数。

已更新 [**D3D12_RESOURCE_STATES**](/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) 枚举，并增加了 **D3D12_RESOURCE_STATE_SHADING_RATE_SOURCE** 常数。

## <a name="windows-10-version-1809"></a>Windows 10 版本 1809

为 Windows 10 （版本 1809 (10.0）添加或更新了这些功能和 Api。生成 17763) &mdash; 也称为 Windows 10 10 月2018更新。

- [Direct3D 12 光线跟踪](./direct3d-12-raytracing.md)
- [Direct3D 12 呈现通道](./direct3d-12-render-passes.md)

## <a name="windows-10-version-1709"></a>Windows 10 版本 1709

下列接口已添加到 Windows 10 版本 1709 的 Direct3D 文档中。

-   [ID3D12Fence1](/windows/win32/api/d3d12/nn-d3d12-id3d12fence1) 通过支持检索传入的标志来创建围栏，扩展了创建围栏的功能****。
-   [ID3D12GraphicsCommandList2](/windows/win32/api/d3d12/nn-d3d12-id3d12graphicscommandlist2) 通过支持将即时值直接写入缓冲区，扩展了可用图形命令列表****。
-   [ID3D12Device3](/windows/win32/api/d3d12/nn-d3d12-id3d12device3) 通过在系统内存中创建特殊用途的诊断堆来扩展虚拟适配器功能，这些诊断堆即使在出现 GPU 故障或设备被删除的情况下也能保留****。

[D3D\_SHADER\_MODEL](/windows/win32/api/d3d12/ne-d3d12-d3d_shader_model) 枚举添加了新的 D3D\_SHADER\_MODEL\_6\_1 值来描述着色器模型 6.1********。

[D3D12\_FEATURE](/windows/win32/api/d3d12/ne-d3d12-d3d12_feature) 枚举也具有新的 D3D12\_FEATURE\_D3D12\_OPTIONS3 和 D3D12\_FEATURE\_EXISTING\_HEAPS 值************。 顾名思义，通过这些值可查看其他 Direct3D12 选项以及现有堆的支持。

## <a name="windows-10-version-1703"></a>Windows 10 版本 1703

以下主题已添加到 Windows 10 版本 1703 的 Direct3D 文档中。

-   [ID3D12Device2::CreatePipelineState](/windows/win32/api/d3d12/nf-d3d12-id3d12device2-createpipelinestate) 方法和 [D3D12\_Pipeline\_State\_Stream\_Desc](/windows/win32/api/d3d12/ns-d3d12-d3d12_pipeline_state_stream_desc) 结构表示创建 PSO 的更强大的新方法，还统一了图形和计算管道的创建界面********。
-   [ID3D12Device1::CreatePipelineLibrary1](https://www.bing.com/search?q=**ID3D12Device1::CreatePipelineLibrary1**) 方法扩展了管道库接口以接受使用新的统一 [D3D12\_Pipeline\_State\_Stream\_Desc](/windows/win32/api/d3d12/ns-d3d12-d3d12_pipeline_state_stream_desc) 结构创建的 PSO********。
-   [D3D12EnableExperimentalFeatures](/windows/win32/api/d3d12/nf-d3d12-d3d12enableexperimentalfeatures) 函数让开发人员能够使用开发人员模式中的计算机来体验某些开发中的功能****。
-   有 5 个新接口（请参阅[接口层次结构](interface-hierarchy.md)）：
    -   [**ID3D12GraphicsCommandList1**](/windows/win32/api/d3d12/nn-d3d12-id3d12graphicscommandlist1)
    -   [**ID3D12PipelineLibrary1**](/windows/win32/api/d3d12/nn-d3d12-id3d12pipelinelibrary1)
    -   [**ID3D12Device2**](/windows/win32/api/d3d12/nn-d3d12-id3d12device2)
    -   [**ID3D12Debug2**](/windows/win32/api/D3D12sdklayers/nn-d3d12sdklayers-id3d12debug2)
    -   [**ID3D12Tools**](/windows/win32/api/d3d12/nn-d3d12-id3d12tools)
-   请参阅 [HLSL 着色器模型 6.0 概述](../direct3dhlsl/hlsl-shader-model-6-0-features-for-direct3d-12.md)，其中介绍了多线程像素和计算着色器的批次内部操作。
-   [ID3D12Device::SetStablePowerState](/windows/win32/api/d3d12/nf-d3d12-id3d12device-setstablepowerstate) 的使用已更改****。
-   在 [Direct3D 11.4 功能](../direct3d11/direct3d-11-4-features.md)中描述了 Direct3D 11 的一些新功能。
-   [AtomicCopyBufferUINT](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-atomiccopybufferuint) 和 [AtomicCopyBufferUINT64](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-atomiccopybufferuint64) 实现了延迟锁存来降低出现的延迟************。
-   [ID3D12Device2::CreatePipelineState](/windows/win32/api/d3d12/nf-d3d12-id3d12device2-createpipelinestate) 和 [OMSetDepthBounds](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-omsetdepthbounds) 在支持的硬件上实现了************。
-   [ResolveSubresourceRegion](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-resolvesubresourceregion) 实现了子资源的部分解析，以帮助优化性能********。
-   [SetSamplePositions](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-setsamplepositions) 在支持的硬件上实现了可编程的示例位置********。

## <a name="november-2016-documentation-update"></a>2016 年 11 月文档更新

-   修订了 [ID3D12GraphicsCommandList::DiscardResource](/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-discardresource) 的备注****。
-   阐述了“状态衰减至常见”（请参阅[在 Direct3D 12 中使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)）。
-   可直接通过 [D3D12 帮助程序库](https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries/D3DX12)下载在 [D3D12 的帮助程序结构和函数](helper-structures-and-functions-for-d3d12.md)中引用的 D3dx12.h 头文件。

## <a name="august-2016-documentation-update-2"></a>2016 年 8 月文档更新 2

-   新增了名为[了解 D3D12 调试层](understanding-the-d3d12-debug-layer.md)的指南部分。

    在预览模式下 (三个新的调试层接口) ： [**ID3D12Debug1**](/windows/win32/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debug1)、 [**ID3D12DebugCommandList1**](/windows/win32/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debugcommandlist1)、 [**ID3D12DebugDevice1**](/windows/win32/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debugdevice1)。

## <a name="august-2016-documentation-update-1"></a>2016 年 8 月文档更新 1

-   修订了 [在 Direct3D 12 中使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。
-   [多队列资源访问](./user-mode-heap-synchronization.md#multi-queue-resource-access)的修订版本。

## <a name="windows-10-version-1607"></a>Windows 10 版本 1607

以下主题已添加到 Windows 10 版本 1607 的 Direct3D 文档中。

-   [根签名版本 1.1](root-signature-version-1-1.md)：概述了更新后的根签名，它们使应用能够指定静态或易失性描述符和数据的方式，这有助于图形驱动程序优化。
-   [ID3D12Device1::CreatePipelineLibrary](/windows/win32/api/d3d12/nf-d3d12-id3d12device1-createpipelinelibrary) 方法描述了创建管道库的优势****。
-   有 3 个新接口（请参阅[接口层次结构](interface-hierarchy.md)）：
    -   [**ID3D12PipelineLibrary**](/windows/win32/api/d3d12/nn-d3d12-id3d12pipelinelibrary)
    -   [**ID3D12Device1**](/windows/win32/api/d3d12/nn-d3d12-id3d12device1)
    -   [**ID3D12VersionedRootSignatureDeserializer**](/windows/win32/api/d3d12/nn-d3d12-id3d12versionedrootsignaturedeserializer)
-   请参阅 [HLSL 着色器模型 6.0 概述](../direct3dhlsl/hlsl-shader-model-6-0-features-for-direct3d-12.md)，其中介绍了多线程像素和计算着色器的批次内部操作。
-   [ID3D12Device::SetStablePowerState](/windows/win32/api/d3d12/nf-d3d12-id3d12device-setstablepowerstate) 的使用已更改****。
-   在 [Direct3D 11.4 功能](../direct3d11/direct3d-11-4-features.md)中描述了 Direct3D 11 的一些新功能。
-   Direct3D 12 支持的库范围已更新，请参阅 [Direct3D 12 编程环境设置](directx-12-programming-environment-set-up.md)的“支持的工具和库”部分****。
-   [使用具有高动态范围显示的 DirectX 和高级颜色](../direct3darticles/high-dynamic-range.md)
-   [显示变量刷新频率](../direct3ddxgi/variable-refresh-rate-displays.md)
-   [DXGI 1.5 改进](../direct3ddxgi/dxgi-1-5-improvements.md)

## <a name="related-topics"></a>相关主题

* [Direct3D 12 编程指南](directx-12-programming-guide.md)