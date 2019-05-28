---
title: 新版本
description: 介绍了最新 SDK 版本提供的最重要的新文档。
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
# <a name="new-releases"></a>新版本

介绍了最新 SDK 版本提供的最重要的新文档。

-   [Windows 10，版本 1809](#windows-10-version-1809)
-   [Windows 10 版本 1709](#windows-10-version-1709)
-   [Windows 10，版本 1703](#windows-10-version-1703)
-   [2016 年 11 月文档更新](#november-2016-documentation-update)
-   [2016 年 8 月文档更新 2](#august-2016-documentation-update-2)
-   [2016 年 8 月文档更新 1](#august-2016-documentation-update-1)
-   [Windows 10，版本 1607](#windows-10-version-1607)
-   [相关的主题](#related-topics)

## <a name="windows-10-version-1809"></a>Windows 10，版本 1809

- [Direct3D 12 Raytracing](/windows/desktop/direct3d12/direct3d-12-raytracing)
- [Direct3D 12 呈现阶段](/windows/desktop/direct3d12/direct3d-12-render-passes)

## <a name="windows-10-version-1709"></a>Windows 10 版本 1709

这些接口已添加到 Windows 10 版本 1709年的 Direct3D 文档。

-   [**ID3D12Fence1** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12fence1)扩展通过支持检索传入的要创建隔离的标志创建界定的功能。
-   [**ID3D12GraphicsCommandList2** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12graphicscommandlist2)扩展通过支持即时值直接写入缓冲区的可用的图形命令列表。
-   [**ID3D12Device3** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12device3)扩展虚拟适配器功能通过创建专用的诊断堆在系统内存中保留即使 GPU 容错或设备中删除方案。

[ **D3D\_着色器\_模型**](/windows/desktop/api/d3d12/ne-d3d12-d3d_shader_model)枚举有一个新的**D3D\_着色器\_模型\_6\_1**值增加了描述的着色器模型 6.1。

[ **D3D12\_功能**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_feature)枚举还具有新**D3D12\_功能\_D3D12\_OPTIONS3**和**D3D12\_功能\_现有\_堆**值。 顾名思义，这些值将允许你检查 Direct3D12 的其他选项，以及查找所支持的现有堆。

## <a name="windows-10-version-1703"></a>Windows 10 版本 1703

这些主题已添加到 Windows 10，版本 1703年的 Direct3D 文档。

-   [ **ID3D12Device2::CreatePipelineState** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12device2-createpipelinestate)方法和[ **D3D12\_管道\_状态\_Stream\_Desc** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_pipeline_state_stream_desc)结构表示新、 更有效地创建 Pso，和统一界面用于创建图形和计算管道。
-   [ **ID3D12Device1::CreatePipelineLibrary1** ](https://www.bing.com/search?q=**ID3D12Device1::CreatePipelineLibrary1**)方法会扩展管道库接口接受使用新创建的 Pso 统一[ **D3D12\_管道\_状态\_Stream\_Desc** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_pipeline_state_stream_desc)结构。
-   [ **D3D12EnableExperimentalFeatures** ](/windows/desktop/api/D3D12/nf-d3d12-d3d12enableexperimentalfeatures)函数允许开发人员尝试使用在开发人员模式下使用计算机特定中开发功能。
-   有五个新接口 (请参阅[界面层次结构](interface-hierarchy.md)):
    -   [**ID3D12GraphicsCommandList1**](/windows/desktop/api/D3D12/nn-d3d12-id3d12graphicscommandlist1)
    -   [**ID3D12PipelineLibrary1**](https://msdn.microsoft.com/library/windows/desktop/mt492661)
    -   [**ID3D12Device2**](/windows/desktop/api/D3D12/nn-d3d12-id3d12device2)
    -   [**ID3D12Debug2**](/windows/desktop/api/D3D12sdklayers/nn-d3d12sdklayers-id3d12debug2)
    -   [**ID3D12Tools**](/windows/desktop/api/d3d12/nn-d3d12-id3d12tools)
-   请参阅[HLSL 着色器模型 6.0 概述](https://msdn.microsoft.com/library/windows/desktop/mt733232)，其中描述了多线程像素和计算着色器的批内部操作。
-   利用[ **ID3D12Device::SetStablePowerState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-setstablepowerstate)已更改。
-   中介绍了一些新功能用于 Direct3D 11 [Direct3D 11.4 功能](https://msdn.microsoft.com/library/windows/desktop/mt661819)。
-   [**AtomicCopyBufferUINT** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-atomiccopybufferuint)并[ **AtomicCopyBufferUINT64** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-atomiccopybufferuint64)启用**后期闩锁**以减少 pervieved 延迟。
-   [**ID3D12Device2::CreatePipelineState** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12device2-createpipelinestate)并[ **OMSetDepthBounds** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-omsetdepthbounds)启用**深度边界测试**在受支持的硬件。
-   [**ResolveSubresourceRegion** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-resolvesubresourceregion)使**部分解析**的子来帮助优化性能。
-   [**SetSamplePositions** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-setsamplepositions)使**programable 示例位置**在受支持的硬件。

## <a name="november-2016-documentation-update"></a>2016 年 11 月文档更新

-   修订的备注[ **ID3D12GraphicsCommandList::DiscardResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-discardresource)。
-   "为常见状态 decay"说明 (请参阅[使用 Direct3D 12 中的同步资源状态资源障碍](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md))。
-   中 D3dx12.h 标头文件引用[帮助程序结构和函数对 D3D12](helper-structures-and-functions-for-d3d12.md)，可以直接从下载[D3D12 帮助程序库](https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries/D3DX12)。

## <a name="august-2016-documentation-update-2"></a>2016 年 8 月文档更新 2

-   标题为新指南部分[了解 D3D12 调试层](understanding-the-d3d12-debug-layer.md)。

    描述了三个新的调试层接口 （在预览模式下）：[**ID3D12Debug1**](/windows/desktop/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debug1)， [ **ID3D12DebugCommandList1**](/windows/desktop/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debugcommandlist1)， [ **ID3D12DebugDevice1**](/windows/desktop/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debugdevice1)。

## <a name="august-2016-documentation-update-1"></a>2016 年 8 月文档更新 1

-   修订[使用资源屏障来同步资源状态在 Direct3D 12](using-resource-barriers-to-synchronize-resource-states-in-direct3d-12.md)。
-   修订**多队列资源访问权限**一部分[同步和多引擎](user-mode-heap-synchronization.md)。

## <a name="windows-10-version-1607"></a>Windows 10 版本 1607

这些主题已添加到 Windows 10，版本 1607年的 Direct3D 文档。

-   [根签名版本 1.1](root-signature-version-1-1.md) ： 是更新的根签名，因而应用程序可以指定如何静态的还是可变的描述符和数据的概述，这可帮助图形驱动程序优化。
-   [ **ID3D12Device1::CreatePipelineLibrary** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12device1-createpipelinelibrary)方法描述创建管道库的优势。
-   有三个新接口 (请参阅[界面层次结构](interface-hierarchy.md)):
    -   [**ID3D12PipelineLibrary**](/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinelibrary)
    -   [**ID3D12Device1**](/windows/desktop/api/d3d12/nn-d3d12-id3d12device1)
    -   [**ID3D12VersionedRootSignatureDeserializer**](/windows/desktop/api/d3d12/nn-d3d12-id3d12versionedrootsignaturedeserializer)
-   请参阅[HLSL 着色器模型 6.0 概述](https://msdn.microsoft.com/library/windows/desktop/mt733232)，其中描述了多线程像素和计算着色器的批内部操作。
-   利用[ **ID3D12Device::SetStablePowerState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-setstablepowerstate)已更改。
-   中介绍了一些新功能用于 Direct3D 11 [Direct3D 11.4 功能](https://msdn.microsoft.com/library/windows/desktop/mt661819)。
-   支持的库的范围有关 Direct3D 12 中进行了更新，请参阅**支持工具和库**一部分[Direct3D 12 编程环境设置](directx-12-programming-environment-set-up.md)。
-   [高动态范围和宽颜色域](https://msdn.microsoft.com/library/windows/desktop/mt742103)
-   [显示变量的刷新频率](https://msdn.microsoft.com/library/windows/desktop/mt742104)
-   [DXGI 1.5 改进](https://msdn.microsoft.com/library/windows/desktop/mt661818)

## <a name="related-topics"></a>相关主题

* [Direct3D 12 的编程指南](directx-12-programming-guide.md)
