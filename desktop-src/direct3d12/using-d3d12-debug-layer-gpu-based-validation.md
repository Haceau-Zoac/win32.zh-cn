---
title: 在 Direct3D 12 调试层中使用基于 GPU 的验证
description: 本主题介绍如何充分利用 Direct3D 12 调试层。 基于 GPU 的验证（GBV）可在 CPU 上的 API 调用期间无法执行的 GPU 时间线上启用验证方案。
ms.assetid: 01D1F94F-4DD4-4781-86EF-6C639E8B1069
ms.localizationpriority: high
ms.topic: article
ms.date: 02/12/2019
ms.openlocfilehash: cd7b46ef5fb424f24f9d043d0c89bef7acc38786
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71005723"
---
# <a name="use-gpu-based-validation-with-the-direct3d-12-debug-layer"></a>在 Direct3D 12 调试层中使用基于 GPU 的验证

本主题介绍如何充分利用 Direct3D 12 调试层。 基于 GPU 的验证（GBV）可在 CPU 上的 API 调用期间无法执行的 GPU 时间线上启用验证方案。 从适用于 Windows 10 周年更新的图形工具开始，可以使用 GBV。

## <a name="purpose-of-gpu-based-validation"></a>基于 GPU 的验证的用途

基于 GPU 的验证有助于识别以下错误：

- 在着色器中使用未初始化或不兼容的说明符。
- 在着色器中使用引用已删除资源的描述符。
- 验证已升级的资源状态和资源状态衰减。
- 在着色器中超出描述符堆末尾的索引。
- 资源的着色器访问处于不兼容状态。
- 在着色器中使用未初始化或不兼容的采样器。

GBV 通过创建具有直接添加到着色器的验证的已修补着色器来工作。 修补后的着色器检查在着色器执行期间访问的根参数和资源，并将错误报告给日志缓冲区。 GBV 还会注入额外的操作并将调用调度到应用程序命令列表，以验证和跟踪对 GPU 时间线上的命令列表施加的资源状态所做的更改。

由于 GBV 需要执行着色器的功能，因此，复制命令列表由计算命令列表模拟。 这可能会改变硬件执行副本的方式，但最终结果不应更改。 应用程序仍会发现这些是复制命令列表，调试层会对其进行验证。

## <a name="turning-on-gpu-based-validation"></a>启用基于 GPU 的验证

通过在 Direct3D 12 调试层上强制，并对基于 GPU 的验证（控制面板中的 "新建" 选项卡）强制执行，可以强制 GBV 使用 DirectX 控制面板（DXCPL）。 启用后，GBV 将保持启用状态，直到 Direct3D 12 设备发布。 另外，还可以在创建 Direct3D 12 设备之前以编程方式启用 GBV：

```cpp
void EnableShaderBasedValidation()
{
    CComPtr<ID3D12Debug> spDebugController0;
    CComPtr<ID3D12Debug1> spDebugController1;
    VERIFY(D3D12GetDebugInterface(IID_PPV_ARGS(&spDebugController0)));
    VERIFY(spDebugController0->QueryInterface(IID_PPV_ARGS(&spDebugController1)));
    spDebugController1->SetEnableGPUBasedValidation(true);
}
```

## <a name="recommended-usage"></a>推荐用法

通常情况下，你应在大多数时间启用了调试层的情况下运行代码。 但是，GBV 可能会降低工作效率。 开发人员可能会考虑通过较小的数据集（例如，具有较少 PSO 和资源的引擎演示或小型游戏级别）或在早期应用程序中引入以减少性能问题时启用 GBV。 使用较大内容时，请考虑在夜间测试过程中打开一台或两台测试计算机上的 GBV。

## <a name="debug-output"></a>调试输出

对[**ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)的调用在 GPU 上完成执行后，GBV 将生成调试输出。 由于这是在 GPU 时间线上，调试输出可能与其他 CPU 时间线验证异步。 应用程序开发人员可能需要注入自己的 "执行后的等待" 来同步调试输出。

GBV output 标识着色器中发生错误的位置，以及当前的绘图/调度计数和相关对象的标识（例如，命令列表、队列、PSO，等等）。

## <a name="example-debug-message"></a>示例调试消息

以下错误消息指示在着色器中以着色器资源的形式访问了名为 "主颜色缓冲区" 的资源，但当着色器在 GPU 上运行时该资源处于无序访问状态。 还提供其他信息，如着色器源中的位置、命令列表的名称和绘制计数（绘图索引）以及相关 D3D 接口对象的名称。

```cmd
D3D12 ERROR: Incompatible resource state: Resource: 0x0000016F61A6EA80:'Main Color Buffer', 
Subresource Index: [0], 
Descriptor heap index: [0], 
Binding Type In Descriptor: SRV, 
Resource State: D3D12_RESOURCE_STATE_UNORDERED_ACCESS(0x8), 
Shader Stage: PIXEL, 
Root Parameter Index: [0], 
Draw Index: [0], 
Shader Code: E:\FileShare\MiniEngine_GitHub_160128\MiniEngine_GitHub\Core\Shaders\SharpeningUpsamplePS.hlsl(37,2-59), 
Asm Instruction Range: [0x138-0x16b], 
Asm Operand Index: [3], 
Command List: 0x0000016F6F75F740:'CommandList', SRV/UAV/CBV Descriptor Heap: 0x0000016F6F76F280:'Unnamed ID3D12DescriptorHeap Object', 
Sampler Descriptor Heap: <not set>, 
Pipeline State: 0x0000016F572C89F0:'Unnamed ID3D12PipelineState Object',  
[ EXECUTION ERROR #942: GPU_BASED_VALIDATION_INCOMPATIBLE_RESOURCE_STATE]
```

## <a name="debug-layer-apis"></a>调试层 Api

若要启用调试层，请调用[**EnableDebugLayer**](/windows/desktop/api/d3d12sdklayers/nf-d3d12sdklayers-id3d12debug-enabledebuglayer)。

若要启用基于 GPU 的验证，请调用[**SetEnableGPUBasedValidation**](/windows/desktop/api/d3d12sdklayers/nf-d3d12sdklayers-id3d12debug1-setenablegpubasedvalidation)，并引用以下接口的方法：

- [**ID3D12Debug1**](/windows/desktop/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debug1)
- [**ID3D12DebugCommandList1**](/windows/desktop/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debugcommandlist1)
- [**ID3D12DebugDevice1**](/windows/desktop/api/d3d12sdklayers/nn-d3d12sdklayers-id3d12debugdevice1)

请参阅以下枚举和结构：

- [**D3D12\_调试\_命令\_列表参数类型\_\_** ](/windows/desktop/api/d3d12sdklayers/ne-d3d12sdklayers-d3d12_debug_command_list_parameter_type)
- [**D3D12\_调试\_设备参数\_类型\_** ](/windows/desktop/api/d3d12sdklayers/ne-d3d12sdklayers-d3d12_debug_device_parameter_type)
- [**D3D12\_基于\_GPU的\_验证管道\_状态创建\_标志\_\_** ](/windows/desktop/api/d3d12sdklayers/ne-d3d12sdklayers-d3d12_gpu_based_validation_pipeline_state_create_flags)
- [**D3D12\_基于\_GPU的验证着色器\_修补程序模式\_\_\_** ](/windows/desktop/api/d3d12sdklayers/ne-d3d12sdklayers-d3d12_gpu_based_validation_shader_patch_mode)
- [**D3D12\_调试\_命令列表基于\_GPU 的验证\_设置\_\_\_** ](/windows/desktop/api/d3d12sdklayers/ns-d3d12sdklayers-d3d12_debug_command_list_gpu_based_validation_settings)
- [**D3D12\_调试\_设备基于\_GPU的验证\_设置\_\_** ](/windows/desktop/api/d3d12sdklayers/ns-d3d12sdklayers-d3d12_debug_device_gpu_based_validation_settings)

## <a name="related-topics"></a>相关主题

* [了解 Direct3D 12 调试层](understanding-the-d3d12-debug-layer.md)
