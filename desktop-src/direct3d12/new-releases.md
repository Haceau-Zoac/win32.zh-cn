---
title: 最新发布
description: 介绍最新 SDK 发布中最重要的新文档。
ms.assetid: 38F41E05-FECB-41DE-8D30-09733FBEAC48
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 2e4cdd1011c9fe91d4d39b593c0e9a2fc2547c0f
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223717"
---
# <a name="new-releases"></a>最新发布

介绍最新 SDK 发布中最重要的新文档。

-   [Windows 10 版本 1809](#windows-10-version-1809)
-   [Windows 10 版本 1709](#windows-10-version-1709)
-   [Windows 10 版本 1703](#windows-10-version-1703)
-   [2016 年 11 月文档更新](#november-2016-documentation-update)
-   [2016 年 8 月文档更新 2](#august-2016-documentation-update-2)
-   [2016 年 8 月文档更新 1](#august-2016-documentation-update-1)
-   [Windows 10 版本 1607](#windows-10-version-1607)
-   [相关主题](#related-topics)

## <a name="windows-10-version-1809"></a>Windows 10 版本 1809

- [Direct3D 12 光线跟踪](/windows/desktop/direct3d12/direct3d-12-raytracing)
- [Direct3D 12 呈现通道](/windows/desktop/direct3d12/direct3d-12-render-passes)

## <a name="windows-10-version-1709"></a>Windows 10 版本 1709

下列接口已添加到 Windows 10 版本 1709 的 Direct3D 文档中。

-   [ID3D12Fence1](/windows/desktop/api/D3D12/nn-d3d12-id3d12fence1) 通过支持检索传入的标志来创建围栏，扩展了创建围栏的功能  。
-   [ID3D12GraphicsCommandList2](/windows/desktop/api/D3D12/nn-d3d12-id3d12graphicscommandlist2) 通过支持将即时值直接写入缓冲区，扩展了可用图形命令列表  。
-   [ID3D12Device3](/windows/desktop/api/D3D12/nn-d3d12-id3d12device3) 通过在系统内存中创建特殊用途的诊断堆来扩展虚拟适配器功能，这些诊断堆即使在出现 GPU 故障或设备被删除的情况下也能保留  。

[D3D\_SHADER\_MODEL](/windows/desktop/api/d3d12/ne-d3d12-d3d_shader_model) 枚举添加了新的 D3D\_SHADER\_MODEL\_6\_1 值来描述着色器模型 6.1   。

[D3D12\_FEATURE](/windows/desktop/api/D3D12/ne-d3d12-d3d12_feature) 枚举也具有新的 D3D12\_FEATURE\_D3D12\_OPTIONS3 和 D3D12\_FEATURE\_EXISTING\_HEAPS 值    。 顾名思义，通过这些值可查看其他 Direct3D12 选项以及现有堆的支持。

## <a name="windows-10-version-1703"></a>Windows 10 版本 1703

以下主题已添加到 Windows 10 版本 1703 的 Direct3D 文档中。

-   [ID3D12Device2::CreatePipelineState](/windows/desktop/api/d3d12/nf-d3d12-id3d12device2-createpipelinestate) 方法和 [D3D12\_Pipeline\_State\_Stream\_Desc](/windows/desktop/api/d3d12/ns-d3d12-d3d12_pipeline_state_stream_desc) 结构表示创建 PSO 的更强大的新方法，还统一了图形和计算管道的创建界面   。
-   [ID3D12Device1::CreatePipelineLibrary1](https://www.bing.com/search?q=**ID3D12Device1::CreatePipelineLibrary1**) 方法扩展了管道库接口以接受使用新的统一 [D3D12\_Pipeline\_State\_Stream\_Desc](/windows/desktop/api/d3d12/ns-d3d12-d3d12_pipeline_state_stream_desc) 结构创建的 PSO   。
-   [D3D12EnableExperimentalFeatures](/windows/desktop/api/D3D12/nf-d3d12-d3d12enableexperimentalfeatures) 函数让开发人员能够使用开发人员模式中的计算机来体验某些开发中的功能  。
-   有 5 个新接口（请参阅[接口层次结构](interface-hierarchy.md)）：
    -   [**ID3D12GraphicsCommandList1**](/windows/desktop/api/D3D12/nn-d3d12-id3d12graphicscommandlist1)
    -   [**ID3D12PipelineLibrary1**](https://msdn.microsoft.com/library/windows/desktop/mt492661)
    -   [**ID3D12Device2**](/windows/desktop/api/D3D12/nn-d3d12-id3d12device2)
    -   [**ID3D12Debug2**](/windows/desktop/api/D3D12sdklayers/nn-d3d12sdklayers-id3d12debug2)
    -   [**ID3D12Tools**](/windows/desktop/api/d3d12/nn-d3d12-id3d12tools)
-   请参阅 [HLSL 着色器模型 6.0 概述](https://msdn.microsoft.com/library/windows/desktop/mt733232)，其中介绍了多线程像素和计算着色器的批次内部操作。
-   [ID3D12Device::SetStablePowerState](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-setstablepowerstate) 的使用已更改  。
-   在 [Direct3D 11.4 功能](https://msdn.microsoft.com/library/windows/desktop/mt661819)中描述了 Direct3D 11 的一些新功能。
-   [AtomicCopyBufferUINT](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-atomiccopybufferuint) 和 [AtomicCopyBufferUINT64](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-atomiccopybufferuint64) 实现了延迟锁存来降低出现的延迟    。
-   [ID3D12Device2::CreatePipelineState](/windows/desktop/api/d3d12/nf-d3d12-id3d12device2-createpipelinestate) 和 [OMSetDepthBounds](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-omsetdepthbounds) 在支持的硬件上实现了    。
-   [ResolveSubresourceRegion](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-resolvesubresourceregion) 实现了子资源的部分解析，以帮助优化性能   。
-   [SetSamplePositions](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-setsamplepositions) 在支持的硬件上实现了可编程的示例位置   。

## <a name="november-2016-documentation-update"></a>2016 年 11 月文档更新

-   修订了 [ID3D12GraphicsCommandList::DiscardResource](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-discardresource) 的备注  。
-   阐述了“状态衰减至常见”（请参阅[在 Direct3D 12 中使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)）。
-   可直接通过 [D3D12 帮助程序库](https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries/D3DX12)下载在 [D3D12 的帮助程序结构和函数](helper-structures-and-functions-for-d3d12.md)中引用的 D3dx12.h 头文件。

## <a name="august-2016-documentation-update-2"></a>2016 年 8 月文档更新 2

-   新增了名为[了解 D3D12 调试层](understanding-the-d3d12-debug-layer.md)的指南部分。

    描述了 3 个新的调试层接口（在预览模式下）：[ID3D12Debug1](/windows/desktop/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debug1)、[ID3D12DebugCommandList1](/windows/desktop/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debugcommandlist1)、[ID3D12DebugDevice1](/windows/desktop/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debugdevice1)    。

## <a name="august-2016-documentation-update-1"></a>2016 年 8 月文档更新 1

-   修订了 [在 Direct3D 12 中使用资源屏障同步资源状态](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。
-   修订了[同步和多引擎](user-mode-heap-synchronization.md)的“多队列资源访问”部分  。

## <a name="windows-10-version-1607"></a>Windows 10 版本 1607

以下主题已添加到 Windows 10 版本 1607 的 Direct3D 文档中。

-   [根签名版本 1.1](root-signature-version-1-1.md)：概述了更新后的根签名，它们使应用能够指定静态或易失性描述符和数据的方式，这有助于图形驱动程序优化。
-   [ID3D12Device1::CreatePipelineLibrary](/windows/desktop/api/d3d12/nf-d3d12-id3d12device1-createpipelinelibrary) 方法描述了创建管道库的优势  。
-   有 3 个新接口（请参阅[接口层次结构](interface-hierarchy.md)）：
    -   [**ID3D12PipelineLibrary**](/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinelibrary)
    -   [**ID3D12Device1**](/windows/desktop/api/d3d12/nn-d3d12-id3d12device1)
    -   [**ID3D12VersionedRootSignatureDeserializer**](/windows/desktop/api/d3d12/nn-d3d12-id3d12versionedrootsignaturedeserializer)
-   请参阅 [HLSL 着色器模型 6.0 概述](https://msdn.microsoft.com/library/windows/desktop/mt733232)，其中介绍了多线程像素和计算着色器的批次内部操作。
-   [ID3D12Device::SetStablePowerState](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-setstablepowerstate) 的使用已更改  。
-   在 [Direct3D 11.4 功能](https://msdn.microsoft.com/library/windows/desktop/mt661819)中描述了 Direct3D 11 的一些新功能。
-   Direct3D 12 支持的库范围已更新，请参阅 [Direct3D 12 编程环境设置](directx-12-programming-environment-set-up.md)的“支持的工具和库”部分  。
-   [高动态范围和宽色域](https://msdn.microsoft.com/library/windows/desktop/mt742103)
-   [显示变量刷新频率](https://msdn.microsoft.com/library/windows/desktop/mt742104)
-   [DXGI 1.5 改进](https://msdn.microsoft.com/library/windows/desktop/mt661818)

## <a name="related-topics"></a>相关主题

* [Direct3D 12 编程指南](directx-12-programming-guide.md)
