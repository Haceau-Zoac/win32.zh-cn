---
title: 可变速率着色 (VRS)
description: 可变速率着色 &mdash; 或粗略像素着色 &mdash; 是一种机制，可让你以不同渲染图像的速率分配渲染性能/算力。
ms.localizationpriority: high
ms.topic: article
ms.date: 04/08/2019
ms.openlocfilehash: cd871fcd72234af6c18df416886822b37777aabb
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006013"
---
# <a name="variable-rate-shading-vrs"></a>可变速率着色 (VRS)

## <a name="the-motivation-for-vrs"></a>VRS 的动机
由于性能限制，图形渲染器不能一直为其输出图像的每个组成部分提供相同的质量级别。 可变速率着色 &mdash; 或粗略像素着色 &mdash; 是一种机制，可让你以不同渲染图像的速率分配渲染性能/算力。

在某些情况下，着色速率可能会轻微降低，或者在可察觉的输出质量方面不会有任何降低，因此可以提高性能，且基本上不会额外增加费用。

## <a name="without-vrsmdashmulti-sample-anti-aliasing-with-supersampling"></a>不使用 VRS &mdash; 结合超采样实现多重采样抗锯齿
如果不使用可变速率着色，控制着色速率的唯一方法是结合采样执行（也称为超采样）使用多重采样抗锯齿 (MSAA)。

MSAA 机制可以减少几何失真，与不使用 MSAA 相比，它可以改善图像的渲染质量。 MSAA 样本计数（可为 1x、2x、4x、8x 或 16x）控制分配给每个渲染器目标像素的样本数。 分配目标时必须预先知道 MSAA 样本计数，以后无法更改此计数。

超采样导致以更高的质量为每个样本调用像素着色器中一次，但与按像素的执行相比，还可以提高性价比。

应用程序可以通过选择基于像素的执行或结合超采样的 MSAA，来控制其着色速率。 这些两个选项不提供极精细的控制。 此外，相比于图像的剩余部分，可以较低对象特定类的着色速率。 此类对象可能包括 HUD 元素后面的对象、透明度、模糊度（景深度、动作等），或者 VR 光学产生的光学畸变。 但这种情况不可能出现，因为整个图像的着色质量和成本是固定的。

## <a name="with-variable-rate-shading-vrs"></a>使用可变速率着色 (VRS)
可变速率着色 (VRS) 模型增加了“粗略着色”的概念，可将结合 MSAA 的超采样扩展为相反的“粗略像素”方向。 在此模型中，可以使用比像素更粗略的频率执行着色。 换而言之，可将一组像素作为一个单位进行着色，然后，结果将广播到该组中的所有样本。

应用程序可以使用粗略着色 API 来指定属于某个着色组的像素（粗略像素）数目。 分配渲染器目标后，可以改变粗略像素大小。 因此，不同的屏幕部分或不同的绘制通道可以采用不同的着色速率。

下表描述了哪个 MSAA 级别支持哪种粗略像素大小。 有些大小并非在任何平台上均受支持；有些大小只能根据“上限”指示的功能条件 (*AdditionalShadingRatesSupported*) 受到支持。

![coarsePixelSizeSupport](images/CoarsePixelSizeSupport.PNG "粗略像素大小支持")

对于下一部分所述的功能层，不存在粗略像素大小与样本计数的组合，每次调用像素着色器，硬件都需要跟踪 16 个以上的样本。 在上表中，这些组合已进行半色调着色。

## <a name="feature-tiers"></a>功能层
VRS 实现有两个层，你可以查询两项功能。 表格后面更详细地描述了每个层。

![层](images/Tiers.PNG "层")

### <a name="tier-1"></a>第 1 层
- 只能按绘制指定着色速率；不存在更高的粒度。
- 无论绘制的内容位于渲染器目标中的哪个位置，着色速率都会统一应用于该内容。

### <a name="tier-2"></a>第 2 层
- 与第 1 层一样，可以按绘制指定着色速率。 也可以按绘制的组合以及以下条件的组合指定着色速率：
  - 每个诱发顶点的语义，以及
  - 屏幕空间图像。
- 使用一组合并器来合并三个源的着色速率。
- 屏幕空间图像的图块大小为 16x16 或更小。
- 保证准确提供应用程序所请求的着色速率（可实现临时过滤器和其他重构过滤器的精度）。
- 支持 SV_ShadingRate PS 输入。
- 使用一个视区且未写入 `SV_ViewportArrayIndex` 时，按诱发顶点（也称为“按基元”）的着色速率是有效的。
- 如果 *SupportsPerVertexShadingRateWithMultipleViewports* 功能设置为 `true`，则可以对多个视区使用按诱发顶点的速率。 此外，在这种情况下，写入 `SV_ViewportArrayIndex` 时可以使用该速率。

### <a name="list-of-capabilities"></a>功能列表
- *AdditionalShadingRatesSupported*
  - 布尔类型。
  - 指示是否支持使用 2x4、4x2 和 4x4 粗略像素大小进行单一采样渲染，以及 2x MSAA 是否支持粗略像素大小 2x4。
- *SupportsPerVertexShadingRateWithMultipleViewports*
  - 布尔类型。
  - 指示是否可对多个视区使用按顶点（也称为“按基元”）的着色速率。

## <a name="specifying-shading-rate"></a>指定着色速率
为提高应用程序的灵活性，可以使用多种机制来控制着色速率。 可用的机制根据硬件功能层而异。

### <a name="command-list"></a>命令列表
这是用于设置着色速率的最简单机制。 它适用于所有层。

应用程序可以使用 [**ID3D12GraphicsCommandList5::RSSetShadingRate** 方法](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist5-rssetshadingrate)指定粗略像素大小。 该 API 采用单个枚举参数。 使用该 API 可对渲染质量级别进行整体控制 &mdash; 可按绘制设置着色速率。

此状态的值通过 [**D3D12_SHADING_RATE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate) 枚举来表示。

#### <a name="coarse-pixel-size-support"></a>粗略像素大小支持
所有层都支持着色速率 1x1、1x2、2x2 和 2x2。

*AdditionalShadingRatesSupported* 功能指示设备是否支持 2x4、4x2 和 4x4。

### <a name="screen-space-image-image-based"></a>屏幕空间图像（基于图像）
在第 2 层和更高的层上，可以使用屏幕空间图像指定像素着色速率。

屏幕空间图像可让应用程序创建“详细级别 (LOD) 掩码”图像用于指示质量可变的区域，例如运动模糊、景深度模糊、透明对象或 HUD UI 元素覆盖的区域。 图像分辨率以宏块表示；不采用渲染器目标的分辨率。 换而言之，着色速率数据是以 VRS 图块大小指示的 8x8 或 16x16 像素图块粒度指定的。

#### <a name="tile-size"></a>图块大小
应用程序可以查询某个 API 来检索其设备支持的 VRS 图块大小。

图块是方形的，大小是指该图块的宽度或高度（以纹素表示）。

如果硬件不支持第 2 层可变速率着色，则针对图块大小的功能查询将返回 0。

如果硬件支持第 2 层可变速率着色，则图块大小是以下值之一。

- 8
- 16
- 32

#### <a name="screen-space-image-size"></a>屏幕空间图像大小
对于大小为 {rtWidth, rtHeight}、使用名为 **VRSTileSize** 的给定图块大小的渲染器目标，覆盖该目标的屏幕空间图像将采用这些维度。

```cpp
{ ceil((float)rtWidth / VRSTileSize), ceil((float)rtHeight / VRSTileSize) }
```

屏幕空间图像的左上坐标 (0, 0) 将锁定为渲染器目标的左上坐标 (0, 0)。

若要查找某个图块的、对应于渲染器目标中特定位置的 (x,y) 坐标，请将窗口空间坐标 (x, y) 除以图块大小，并忽略分数位。

如果屏幕空间图像大于给定渲染器目标所需的大小，则不会使用右侧和/或底部的附加部分。

如果屏幕空间图像对于给定的渲染器目标而言太小，尝试从超出目标实际范围的图像读取数据会生成默认的着色速率 1x1。 这是因为，屏幕空间图像的左上坐标 (0, 0) 锁定为渲染器目标的左上坐标 (0, 0)，“超出渲染器目标范围读取”意味着读取的 x 和 y 值太大。

#### <a name="format-layout-resource-properties"></a>格式、布局和资源属性
此图面的格式为单通道 8 位图面 ([**DXGI_FORMAT_R8_UINT**](/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format))。

资源的维度为 **TEXTURE2D**。

无法对此图面进行阵列处理或 mip 处理。 它必须明确地采用一个 mip 级别。

它的样本计数为 1，样本质量为 0。

它的纹理布局为 **UNKNOWN**。 由于不允许跨适配器，这意味着它不能采用行主序布局。

填充屏幕空间图像数据的预期方式是
1. 使用计算着色器写入数据；将屏幕空间图像绑定为 UAV，或
2. 将数据复制到屏幕空间图像。

创建屏幕空间图像时允许使用这些标志。

- NONE
- ALLOW_UNORDERED_ACCESS
- DENY_SHADER_RESOURCE

不允许使用这些标志。

- ALLOW_RENDER_TARGET
- ALLOW_DEPTH_STENCIL
- ALLOW_CROSS_ADAPTER
- ALLOW_SIMULTANEOUS_ACCESS
- VIDEO_DECODE_REFERENCE_ONLY

资源的堆类型不能是 UPLOAD 或 READBACK。

资源不能是 SIMULTANEOUS_ACCESS。 不允许资源跨适配器。

#### <a name="data"></a>Data
屏幕空间图像的每个字节对应于 [**D3D12_SHADING_RATE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate) 枚举的值。

#### <a name="resource-state"></a>资源状态
将资源用作屏幕空间图像时，需将其转换为只读状态。 只读状态 [**D3D12_RESOURCE_STATE_SHADING_RATE_SOURCE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_states) 是为了实现此目的而定义的。

转换出该状态的图像资源将再次可写。

#### <a name="setting-the-image"></a>设置图像
在命令列表中设置用于指定着色器速率的屏幕空间图像。

无法从任何着色器阶段读取或写入已设置为着色速率源的资源。

可以设置一个 `null` 屏幕空间图像用于指定着色器速率。 这样就可以持续使用 1x1 来实现屏幕空间图像的价值。 最初可以考虑将屏幕空间图像设置为 `null`。

#### <a name="promotion-and-decay"></a>提升和衰减
在提升或衰减方面，屏幕空间图像资源不提供任何特别的暗示。

### <a name="per-primitive-attribute"></a>按基元的属性
使用按基元的属性可将某个着色速率条件指定为诱发顶点中的属性。 此属性将平面着色 &mdash; 也就是说，它会传播到当前三角形或线条基元中的所有像素。 与其他着色速率说明符相比，使用按基元的属性可以更精细地控制图像质量。

按基元的属性是名为 `SV_ShadingRate` 的可设置语义。 `SV_ShadingRate` 作为 [HLSL 着色器模型 6.4](/windows/desktop/direct3dhlsl/hlsl-shader-model-6-4-features-for-direct3d-12) 的一部分存在。

如果 VS 或 GS 设置了 `SV_ShadingRate`，但 VRS 未启用，则语义设置不起作用。 如果未按基元指定 `SV_ShadingRate` 的值，则会采用着色速率值 1x1 作为按基元贡献值。

### <a name="combining-shading-rate-factors"></a>合并着色速率因素
着色速率的各种源是使用此示意图按顺序应用的。

![合并器](images/Combiners.PNG "合并器")

使用合并器来合并每个 A-B 对。

\* 按顶点属性指定着色器速率时。

- 如果使用几何着色器，可以通过该着色器指定着色速率。
- 如果未使用几何着色器，将按诱发顶点指定着色速率。

#### <a name="list-of-combiners"></a>合并器列表
支持以下合并器。 使用合并器 (C) 和两个输入（A 和 B）。

- **直通**。 C.xy = A.xy。
- **重写**。 C.xy = B.xy。
- **更高的质量**。 C.xy = min(A.xy, B.xy)。
- **更低的质量**。 C.xy = max(A.xy, B.xy)。
- **应用相对于 A 的成本 B**。C.xy = min(maxRate, A.xy + B.xy)。

其中，`maxRate` 是设备上允许的最大粗略像素维度。 这将是

- **D3D12_AXIS_SHADING_RATE_2X**（即值 1）（如果 AdditionalShadingRatesSupported 为 `false`）。
- **D3D12_AXIS_SHADING_RATE_4X**（即值 2）（如果 AdditionalShadingRatesSupported 为 `true`）。

通过 [**ID3D12GraphicsCommandList5::RSSetShadingRate**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist5-rssetshadingrate) 在命令列表中设置所选的可变速率着色合并器。

如果未曾设置过合并器，则合并器将保留默认值，即 PASSTHROUGH。

如果合并器的源为支持表中不允许的 [**D3D12_AXIS_SHADING_RATE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_axis_shading_rate)，则输入将净化为受支持的着色速率。

如果合并器的输出不对应于平台支持的着色速率，则结果将净化为受支持的着色速率。

### <a name="default-state-and-state-clearing"></a>默认状态和状态清除
所有着色速率源，即

- 管道状态指定的速率（在命令列表中指定）、
- 屏幕空间图像指定的速率和
- 按基元的属性

都具有默认值 **D3D12_SHADING_RATE_1X1**。 默认的合并器为 {PASSTHROUGH, PASSTHROUGH}。

如果未指定屏幕空间图像，则从该源推断出着色速率 1x1。

如果未指定按基元的属性，则从该源推断出着色速率 1x1。

[ID3D12CommandList::ClearState](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearstate) 将管道状态指定的速率重置为默认值，并将所选的屏幕空间图像重置为默认值“无屏幕空间图像”。

## <a name="querying-shading-rate-by-using-sv_shadingrate"></a>使用 SV_ShadingRate 查询着色速率
调用任意给定的像素着色器时，知道硬件选择了哪种着色速率会很有用。 这样可以在 PS 代码中实现各种优化。 仅限 PS 的系统变量 `SV_ShadingRate` 提供有关着色速率的信息。

### <a name="type"></a>类型
此语义的类型为 uint。

### <a name="data-interpretation"></a>数据解释
数据将解释为 [**D3D12_SHADING_RATE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate) 枚举的值。

### <a name="if-vrs-is-not-being-used"></a>如果未使用 VRS
如果未使用粗略像素着色，则读回值为 1x1 的 `SV_ShadingRate`（表示精细像素）。

### <a name="behavior-under-sample-based-execution"></a>基于样本的执行的行为
如果像素着色器输入了 `SV_ShadingRate` 并使用基于样本的执行 &mdash; 例如，输入 `SV_SampleIndex` 或使用样本内插关键字，则该像素着色器将无法编译。

> ### <a name="remarks-on-deferred-shading"></a>有关延迟着色的备注
>
> 延迟着色应用程序的照明通道可能需要知道对屏幕的哪个区域使用了哪种着色速率。 这样，便能够以更粗略的速率启动照明通道调度。 如果将 `SV_ShadingRate` 变量写出到 gbuffer，可以使用该变量来实现此目的。

## <a name="depth-and-stencil"></a>深度和模具
使用粗略像素着色时，始终会计算深度、模具和覆盖度，并以完全样本分辨率发出此信息。

## <a name="using-the-shading-rate-requested"></a>使用请求的着色速率
对于所有层，如果请求了某种着色速率，并且该速率受设备和 MSAA 级别组合的支持，则该速率预期是硬件提供的着色速率。

请求的着色速率表示计算为合并器的输出的着色速率（请参阅本主题的[合并着色速率因素](#combining-shading-rate-factors)部分）。

在样本数小于或等于 4 的渲染操作中，支持的着色速率为 1x1、1x2、2x1 或 2x2。 如果 *AdditionalShadingRatesSupported* 功能为 `true`，则 2x4、4x2 和 4x4 也是某些样本计数支持的着色速率（请参阅本主题的[使用可变速率着色 (VRS)](#with-variable-rate-shading-vrs) 部分中的表格）。

## <a name="screen-space-derivatives"></a>屏幕空间派生对象
相邻像素渐变的计算受粗略像素着色的影响。 例如，使用 2x2 粗略像素时，渐变大小是不使用粗略像素时的两倍。 应用程序可能需要调整着色器以进行相应的补偿 &mdash; 根据所需的功能，也可以不调整。

由于 mip 是根据屏幕空间派生对象选择的，因此，使用粗略像素着色会影响 mip 的选择。 与不使用粗略像素相比，使用粗略像素着色会导致选择细节更少的 mip。

## <a name="attribute-interpolation"></a>属性内插
可以根据源顶点内插像素着色器的输入。 由于可变速率着色会影响每次调用像素着色器时写入的目标区域，因此，着色器将与属性内插交互。 有三种类型的内插：中点、质心和样本。

### <a name="center"></a>中点
粗略像素的中点内插位置是完整粗略像素区域的几何中心。 `SV_Position` 始终在粗略像素区域的中心内插。

### <a name="centroid"></a>质心
将粗略像素着色与 MSAA 配合使用时，对于每个精细像素，仍会写入到为目标 MSAA 级别分配的完整数量的样本。 因此，质心内插位置会考虑粗略像素中的精细像素的所有样本。 话虽如此，质心内插位置将按样本索引的升序定义为第一个覆盖的样本。 该样本的有效覆盖范围将通过 AND 运算符与光栅器状态 SampleMask 的相应位合并。

> [!NOTE]
> 在第 1 层上使用粗略像素着色时，SampleMask 始终是完整掩码。 如果 SampleMask 未配置为完整掩码，则会在第 1 层上禁用粗略像素着色。

### <a name="sample-based-execution"></a>基于样本的执行
通过样本内插功能进行的基于样本的执行（或“超采样”）可与粗略像素着色配合使用，它会导致按样本调用像素着色器。 对于样本计数 N 的目标，将按精细像素调用像素着色器 N 次。

### <a name="evaluateattributesnapped"></a>EvaluateAttributeSnapped
提取模型内部函数与第 1 层上的粗略像素着色不兼容。 如果尝试在第 1 层上将提取模型内部函数与粗略像素着色配合使用，将会自动禁用粗略像素着色。

允许在第 2 层上将内部函数 `EvaluateAttributeSnapped` 与粗略像素着色配合使用。 其语法一直未有变化。

```hlsl
numeric EvaluateAttributeSnapped(   
    in attrib numeric value, 
    in int2 offset);
```

在上下文方面，`EvaluateAttributeSnapped` 提供一个带有两个字段的偏移量参数。 在不使用粗略像素着色的情况下使用该函数时，只会使用 32 个位中的 4 个低序位。 这 4 个位表示范围 [-8, 7]。 此范围跨越像素中的一个 16x16 网格。 该范围覆盖该像素的顶部和左侧边缘，但不覆盖底部和右侧边缘。 偏移量 (-8, -8) 位于左上角，偏移量 (7, 7) 靠近右下角。 偏移量 (0, 0) 位于像素的中心。

与粗略像素着色配合使用时，`EvaluateAttributeSnapped` 的偏移量参数能够指定更多的位置。 偏移量参数选择每个精细像素的 16x16 网格，并且存在多个精细像素。 可表示的范围和使用的相应位数取决于粗略像素大小。 覆盖粗略像素的顶部和左侧边缘，但不覆盖底部和右侧边缘。

下表描述了每个粗略像素大小的 `EvaluateAttributeSnapped` 偏移量参数的内插。

#### <a name="evaluateattributesnappeds-offset-range"></a>EvaluateAttributeSnapped 的偏移范围

|粗略像素大小  |可编制索引的范围             |可表示的范围大小  |所需的位数 {x, y}  |可用位的二进制掩码          |    
|------------------:|---------------------------:|-------------------------:|-----------------------------:|-----------------------------------:|    
|1x1（精细）         |{\[-8, 7\], \[-8, 7\]}      |{16, 16}                  |{4, 4}                        |{000000000000xxxx, 000000000000xxxx}|    
|1x2                |{\[-8, 7\], \[-16, 15\]}    |{16, 32}                  |{4, 5}                        |{000000000000xxxx, 00000000000xxxxx}|    
|2x1                |{\[-16, 15\], \[-8, 7\]}    |{32, 16}                  |{5, 4}                        |{00000000000xxxxx, 000000000000xxxx}|    
|2x2                |{\[-16, 15\], \[-16, 15\]}  |{32, 32}                  |{5, 5}                        |{00000000000xxxxx, 00000000000xxxxx}|    
|2x4                |{\[-16, 15\], \[-32, 31\]}  |{32, 64}                  |{5, 6}                        |{00000000000xxxxx, 0000000000xxxxxx}|    
|4x2                |{\[-32, 31\], \[-16, 15\]}  |{64, 32}                  |{6, 5}                        |{0000000000xxxxxx, 00000000000xxxxx}|    
|4x4                |{\[-32, 31\], \[-32, 31\]}  |{64, 64}                  |{6, 6}                        |{0000000000xxxxxx, 0000000000xxxxxx}|   

下表提供了从定点数到十进制数和分数表示形式的转换指南。 二进制掩码中的第一个可用位是符号位，剩余的二进制掩码由数字部分构成。

传入 `EvaluateAttributeSnapped` 的 4 位值的数字方案并不特定于可变速率着色。 出于完整性，此处的方案是反复迭代的。

对于 4 位值。

| 二进制值 | 十进制  | 分数 |
|-------------:|---------:|-----------:|
|         1000 |-0.5f     |-8 / 16     |
|         1001 |-0.4375f  |-7 / 16|    |
|         1010 |-0.375f   |-6 / 16|    |
|         1011 |-0.3125f  |-5 / 16     |
|         1100 |-0.25f    |-4 / 16     |
|         1101 |-0.1875f  |-3 / 16     |
|         1110 |-0.125f   |-2 / 16     |
|         1111 |-0.0625f  |-1 /16      |
|         0000 |0.0f      |0 / 16      |
|         0001 |-0.0625f  |1 / 16      |
|         0010 |-0.125f   |2 / 16      |
|         0011 |-0.1875f  |3 / 16      |
|         0100 |-0.25f    |4 / 16      |
|         0101 |-0.3125f  |5 / 16      |
|         0110 |-0.375f   |6 / 16      |
|         0111 |-0.4375f  |7 / 16      |

对于 5 位值。

| 二进制值 | 十进制  | 分数 |
|-------------:|---------:|-----------:|
|        10000 |-1        |-16 / 16    |
|        10001 |-0.9375   |-15 / 16    |
|        10010 |-0.875    |-14 / 16    |
|        10011 |-0.8125   |-13 / 16    |
|        10100 |-0.75     |-12 / 16    |
|        10101 |-0.6875   |-11 / 16    |
|        10110 |-0.625    |-10 / 16    |
|        10111 |-0.5625   |-9 / 16     |
|        11000 |-0.5      |-8 / 16     |
|        11001 |-0.4375   |-7 / 16     |
|        11010 |-0.375    |-6 / 16     |
|        11011 |-0.3125   |-5 / 16     |
|        11100 |-0.25     |-4 / 16     |
|        11101 |-0.1875   |-3 / 16     |
|        11110 |-0.125    |-2 / 16     |
|        11111 |-0.0625   |-1 / 16     |
|        00000 |0         |0 / 16      |
|        00001 |0.0625    |1 / 16      |
|        00010 |0.125     |2 / 16      |
|        00011 |0.1875    |3 / 16      |
|        00100 |0.25      |4 / 16      |
|        00101 |0.3125    |5 / 16      |
|        00110 |0.375     |6 / 16      |
|        00111 |0.4375    |7 / 16      |
|        01000 |0.5       |8 / 16      |
|        01001 |0.5625    |9 / 16      |
|        01010 |0.625     |10 / 16     |
|        01011 |0.6875    |11 / 16     |
|        01100 |0.75      |12 / 16     |
|        01101 |0.8125    |13 / 16     |
|        01110 |0.875     |14 / 16     |
|        01111 |0.9375    |15 / 16     |

对于 6 位值。

| 二进制值 | 十进制  | 分数 |
|-------------:|---------:|-----------:|
|       100000 |-2        |-32 / 16    |
|       100001 |-1.9375   |-31 / 16    |
|       100010 |-1.875    |-30 / 16    |
|       100011 |-1.8125   |-29 / 16    |
|       100100 |-1.75     |-28 / 16    |
|       100101 |-1.6875   |-27 / 16    |
|       100110 |-1.625    |-26 / 16    |
|       100111 |-1.5625   |-25 / 16    |
|       101000 |-1.5      |-24 / 16    |
|       101001 |-1.4375   |-23 / 16    |
|       101010 |-1.375    |-22 / 16    |
|       101011 |-1.3125   |-21 / 16    |
|       101100 |-1.25     |-20 / 16    |
|       101101 |-1.1875   |-19 / 16    |
|       101110 |-1.125    |-18 / 16    |
|       101111 |-1.0625   |-17 / 16    |
|       110000 |-1        |-16 / 16    |
|       110001 |-0.9375   |-15 / 16    |
|       110010 |-0.875    |-14 / 16    |
|       110011 |-0.8125   |-13 / 16    |
|       110100 |-0.75     |-12 / 16    |
|       110101 |-0.6875   |-11 / 16    |
|       110110 |-0.625    |-10 / 16    |
|       110111 |-0.5625   |-9 / 16     |
|       111000 |-0.5      |-8 / 16     |
|       111001 |-0.4375   |-7 / 16     |
|       111010 |-0.375    |-6 / 16     |
|       111011 |-0.3125   |-5 / 16     |
|       111100 |-0.25     |-4 / 16     |
|       111101 |-0.1875   |-3 / 16     |
|       111110 |-0.125    |-2 / 16     |
|       111111 |-0.0625   |-1 / 16     |
|       000000 |0         |0 / 16      |
|       000001 |0.0625    |1 / 16      |
|       000010 |0.125     |2 / 16      |
|       000011 |0.1875    |3 / 16      |
|       000100 |0.25      |4 / 16      |
|       000101 |0.3125    |5 / 16      |
|       000110 |0.375     |6 / 16      |
|       000111 |0.4375    |7 / 16      |
|       001000 |0.5       |8 / 16      |
|       001001 |0.5625    |9 / 16      |
|       001010 |0.625     |10 / 16     |
|       001011 |0.6875    |11 / 16     |
|       001100 |0.75      |12 / 16     |
|       001101 |0.8125    |13 / 16     |
|       001110 |0.875     |14 / 16     |
|       001111 |0.9375    |15 / 16     |
|       010000 |1         |16 / 16     |
|       010001 |1.0625    |17 / 16     |
|       010010 |1.125     |18 / 16     |
|       010011 |1.1875    |19 / 16     |
|       010100 |1.25      |20 / 16     |
|       010101 |1.3125    |21 / 16     |
|       010110 |1.375     |22 / 16     |
|       010111 |1.4375    |23 / 16     |
|       011000 |1.5       |24 / 16     |
|       011001 |1.5625    |25 / 16     |
|       011010 |1.625     |26 / 16     |
|       011011 |1.6875    |27 / 16     |
|       011100 |1.75      |28 / 16     |
|       011101 |1.8125    |29 / 16     |
|       011110 |1.875     |30 / 16     |
|       011111 |1.9375    |31 / 16     |

与使用精细像素时一样，在使用粗略像素着色时，可评估位置的 `EvaluateAttributeSnapped` 网格的中点位于粗略像素的中心。

## <a name="setsamplepositions"></a>SetSamplePositions
如果将 API [**ID3D12GraphicsCommandList1::SetSamplePositions**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-setsamplepositions) 用于粗略着色，该 API 会设置精细像素的样本位置。

## <a name="sv_coverage"></a>SV_Coverage
如果在第 1 层上将 `SV_Coverage` 声明为着色器的输入或输出，则会禁用粗略像素着色。

在第 2 层上可对粗略像素着色使用 `SV_Coverage` 语义，该语义反映正在写入 MSAA 目标的哪些样本。

如果使用粗略像素着色（允许使用多个源像素来构成一个图块），覆盖掩码表示来自该图块的所有样本。

假设粗略像素着色与 MSAA 兼容，需要指定的覆盖位数可能不同。 例如，如果某个 4x MSAA 资源使用 [**D3D12_SHADING_RATE_2x2**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate)，则每个粗略像素将写入 4 个精细像素，而每个像素具有 4 个样本。 这意味着，每个粗略像素总共会写入 4 * 4 = 16 个样本。

### <a name="number-of-coverage-bits-needed"></a>所需的覆盖位数
下表指出了粗略像素大小与 MSAA 级别的每种组合需要多少个覆盖位。

![NumberOfCoverageBits](images/NumberOfCoverageBits.PNG "NumberOfCoverageBits")

如下表所示，无法使用粗略像素通过 Direct3D 12 公开的可变速率着色功能一次性写入 16 个以上的样本。 施加此限制的原因在于，Direct3D 12 对不同的 MSAA 级别允许的粗略像素大小施加了约束（请参阅本主题的[使用可变速率着色 (VRS)](#with-variable-rate-shading-vrs) 部分中的表格）。

### <a name="ordering-and-format-of-bits-in-the-coverage-mask"></a>覆盖掩码中的位的排序和格式
覆盖掩码的位遵循明确定义的顺序。 掩码由像素中从左到右、自上到下（列主序）顺序的覆盖范围组成。 覆盖位是覆盖语义的低序位，它们密集打包在一起。

下表显示了支持的粗略像素大小与 MSAA 级别组合的覆盖掩码格式。

![Coverage1x](images/Coverage1x.PNG "Coverage1x")

下表描述了 2x MSAA 像素，其中的每个像素具有索引为 0 和 1 的两个样本。

像素中样本标签的位置用于演示目的，不一定表示该像素中的样本的空间 {X, Y} 位置，尤其是能够以编程方式更改该样本位置的情况下。 样本按其从 0 开始的索引引用。

![Coverage2x](images/Coverage2x.PNG "Coverage2x")

下表显示了 4x MSAA 像素，其中的每个像素具有索引为 0、1、2 和 3 的四个样本。

![Coverage4x](images/Coverage4x.PNG "Coverage4x")

## <a name="discard"></a>丢弃
对粗略像素着色使用 HLSL 语义 `discard` 时，会丢弃粗略像素。

## <a name="target-independent-rasterization-tir"></a>目标独立的光栅化 (TIR)
使用粗略像素着色时，不支持 TIR。

## <a name="raster-order-views-rovs"></a>光栅顺序视图 (ROV)
ROV 联锁指定为以精细像素粒度运行。 如果按样本执行着色，则联锁将以样本粒度运行。

## <a name="conservative-rasterization"></a>保守光栅化
可对可变速率着色使用保守光栅化。 对粗略像素着色使用保守光栅化时，将按给定的完整覆盖范围，以保守方式将不包含精细像素的精细像素光栅化。

### <a name="coverage"></a>覆盖范围
使用保守光栅化时，覆盖语义将包含所覆盖的精细像素的完整掩码，并为未覆盖的精细像素包含 0。

## <a name="bundles"></a>捆绑
可以针对捆绑调用可变速率着色 API。

## <a name="render-passes"></a>渲染器通道
可以在[渲染器通道](/windows/desktop/direct3d12/direct3d-12-render-passes)中调用可变速率着色 API。

## <a name="calling-the-vrs-apis"></a>调用 VRS API
以下部分介绍应用程序通过 Direct3D 12 访问可变速率着色的方式。

### <a name="capability-querying"></a>功能查询

若要查询适配器的可变速率着色功能，请结合 [**D3D12_FEATURE::D3D12_FEATURE_D3D12_OPTIONS6**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_feature) 调用 [**ID3D12Device::CheckFeatureSupport**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)，并提供函数要为你填充的 [**D3D12_FEATURE_DATA_D3D12_OPTIONS6** 结构](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options6)。 **D3D12_FEATURE_DATA_D3D12_OPTIONS6** 结构包含多个成员，这些成员包括一个 [**D3D12_VARIABLE_SHADING_RATE_TIER**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_variable_shading_rate_tier) (D3D12_FEATURE_DATA_D3D12_OPTIONS6::VariableShadingRateTier) 枚举类型的成员，以及一个指示是否支持后台处理的成员 (D3D12_FEATURE_DATA_D3D12_OPTIONS6::BackgroundProcessingSupported)。

例如，若要查询第 1 层功能，可以执行此操作。

```cpp
D3D12_FEATURE_DATA_D3D12_OPTIONS6 options;
return 
    SUCCEEDED(m_device->CheckFeatureSupport(
        D3D12_FEATURE_D3D12_OPTIONS6, 
        &options, 
        sizeof(options))) && 
    options.ShadingRateTier == D3D12_SHADING_RATE_TIER_1;
```

### <a name="shading-rates"></a>着色速率

[**D3D12_SHADING_RATE** 枚举](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate)中的值将经过适当的组织，使着色速率可轻松分解成两个轴，其中每个轴的值将会根据 [**D3D12_AXIS_SHADING_RATE** 枚举](/windows/desktop/api/d3d12/ne-d3d12-d3d12_axis_shading_rate)以对数空间的形式精简表示。

可以编写一个宏，以将两个轴着色速率组合成如下所示的着色速率。

```cpp
#define D3D12_MAKE_COARSE_SHADING_RATE(x,y) ((x) << 2 | (y))
D3D12_MAKE_COARSE_SHADING_RATE(
    D3D12_AXIS_SHADING_RATE_2X, 
    D3D12_AXIS_SHADING_RATE_1X)
```

平台也提供这些宏（在 `d3d12.h` 中定义）。

```cpp
#define D3D12_GET_COARSE_SHADING_RATE_X_AXIS(x) ((x) >> 2 )
#define D3D12_GET_COARSE_SHADING_RATE_Y_AXIS(y) ((y) & 3 )
```

这些宏可用于剖析和了解 `SV_ShaderRate`。

> [!NOTE]
> 此数据解释专门用于描述屏幕空间图像，可由着色器操纵。 前面的部分进一步讨论了此功能。 但是，没有理由不在任何位置使用一致的粗略像素大小定义，包括何时设置命令级着色速率。

### <a name="setting-command-level-shading-rate-and-combiners"></a>设置命令级着色速率与合并器
着色速率和（可选的）合并器是通过 [**ID3D12GraphicsCommandList5::RSSetShadingRate**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist5-rssetshadingrate) 方法指定的。 为基本着色速率传递 [**D3D12_SHADING_RATE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate) 值，并传递可选的 [D3D12_SHADING_RATE_COMBINER](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate_combiner) 值数组。

### <a name="preparing-the-screen-space-image"></a>准备屏幕空间图像
指定可用着色速率图像的只读资源状态将定义为 [D3D12_RESOURCE_STATES::D3D12_RESOURCE_STATE_SHADING_RATE_SOURCE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_states)。

### <a name="setting-the-screen-space-image"></a>设置屏幕空间图像
通过 [**ID3D12GraphicsCommandList5::RSSetShadingRateImage**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist5-rssetshadingrateimage) 方法指定屏幕空间图像。

```cpp
m_commandList->RSSetShadingRateImage(screenSpaceImage);
```

### <a name="querying-the-tile-size"></a>查询图块大小
可以从 [**D3D12_FEATURE_DATA_D3D12_OPTIONS6::ShadingRateImageTileSize**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options6) 成员查询图块大小。 请参阅前面的[功能查询](#capability-querying)。

将检索一个维度，因为水平和垂直维度始终相同。 如果系统的功能为 [**D3D12_SHADING_RATE_TIER_NOT_SUPPORTED**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_variable_shading_rate_tier)，则返回的图块大小为 0。
