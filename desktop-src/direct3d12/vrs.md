---
title: 明暗度变量速率 (VRS)
description: 明暗度变量速率&mdash;或粗略像素阴影&mdash;是一种机制，可让你分配费率因呈现图像的呈现性能/电源。
ms.topic: article
ms.date: 04/08/2019
ms.openlocfilehash: 85cb736b83bea8785146a80ccb12d672ccb48d9c
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224410"
---
# <a name="variable-rate-shading-vrs"></a>明暗度变量速率 (VRS)

## <a name="the-motivation-for-vrs"></a>VRS 动机
由于性能限制，无法始终提供图形呈现器将传送到其输出图像的每个部分的相同级别的质量。 明暗度变量速率&mdash;或粗略像素阴影&mdash;是一种机制，可让你分配费率因呈现图像的呈现性能/电源。

在某些情况下，明暗度速率可减少与可察觉的输出质量; 很少或没有下降从而导致性能的改进是实质上是免费。

## <a name="without-vrsmdashmulti-sample-anti-aliasing-with-supersampling"></a>而无需 VRS&mdash;与超级取样的多重采样抗锯齿
无变量速率阴影控制明暗度速率的唯一方法是使用多重采样抗锯齿 (MSAA) 和基于示例的执行能力 (也称为 supersampling)。

MSAA 是图像的一种机制可减少几何别名，并提高与不使用 MSAA 比较的呈现质量。 MSAA 样本计数，可以是 1 x、 2x、 4 倍，8 倍或 16 x，控制分配给每个呈现器目标像素的样本数。 分配目标，并且以后不能更改时，必须预先知道 MSAA 样本计数。

超级取样导致像素着色器中，每个样本，在更高的质量但也更高性能费用，而每个像素执行调用一次。

你的应用程序可以通过每个像素基于的执行或超级 MSAA 与取样之间进行选择来控制其明暗度速率。 这些两个选项不提供非常精确地控制。 此外，您可能希望较低的明暗度速率对于某一类的对象的比较的图像的其余部分。 此类对象可能包含后面的 HUD 元素，或透明的模糊 （字段深度、 运动等），或者由于 VR 光纤光扭曲的对象。 但这不是有可能，因为明暗度质量和成本固定的整个图像上方。

## <a name="with-variable-rate-shading-vrs"></a>明暗度变量速率 (VRS)
明暗度变量速率 (VRS) 模型扩展到相反，与 MSAA 超级取样"粗像素"、 方向，通过添加粗略的明暗度的概念。 这是其中明暗度可以执行的频率比一个像素更粗。 换而言之，可以作为单个单元，着色的像素为单位的组和结果然后广播到组中的所有示例。

粗明暗度 API 允许应用程序以指定属于阴影的组的像素数或*粗略像素*。 已分配的呈现器目标后，您可以改变粗略像素大小。 因此，不同的屏幕或不同的绘图传递部分可以有不同的明暗度速率。

下面是描述哪些 MSAA 级别支持哪些粗略像素大小的表。 在任何平台上; 不支持某些有条件地启用基于一项功能，其他人时 (*AdditionalShadingRatesSupported*)，由"Cap"。

![coarsePixelSizeSupport](images/CoarsePixelSizeSupport.PNG "粗略像素大小支持")

在下一节中讨论的功能层，则不存在 coarse-pixel-size-and-sample-count 组合需要跟踪每个像素着色器调用的 16 个以上示例硬件。 这些组合都是上表中的半色调阴影。

## <a name="feature-tiers"></a>功能级别
有两个层到 VRS 实现中，并可以查询的两个功能。 在表格之后更详细地介绍每个层。

![层](images/Tiers.PNG "层")

### <a name="tier-1"></a>第 1 层
- 可以仅在每个绘图的基础; 上指定明暗度速率没有更多粒度比的。
- 明暗度速率适用范围统一独立于其位于该呈现器目标的绘制内容。

### <a name="tier-2"></a>第 2 层
- 可以在每个绘图为基础，如第 1 层中所示指定明暗度速率。 它还可以通过组合的每个绘图为基础，以及指定：
  - 每个导致顶点，从语义和
  - 中的屏幕空间映像。
- 从三个来源的明暗度费率是使用一系列的合并器结合使用。
- 屏幕空间图像平铺大小是 16 x 16 或更小。
- 明暗度由你的应用程序请求的速率保证完全 （对于临时和其他重新构造筛选器的精度） 传递。
- 支持 SV_ShadingRate PS 输入。
- 使用一个视区时，每个人深受启发的顶点 （也称为每个基元） 明暗度速率，是有效和`SV_ViewportArrayIndex`不会写入。
- 如果可以与多个视区使用的每个人深受启发的顶点速率*SupportsPerVertexShadingRateWithMultipleViewports*功能设置为`true`。 此外，在这种情况下，该速率可能时使用`SV_ViewportArrayIndex`写入。

### <a name="list-of-capabilities"></a>功能列表
- *AdditionalShadingRatesSupported*
  - 布尔值类型。
  - 指示是否 2 x 4、 4 x 2 和 4 × 4 像素粗略大小支持单采样呈现;和 2 是否支持粗像素大小 2x4 x MSAA。
- *SupportsPerVertexShadingRateWithMultipleViewports*
  - 布尔值类型。
  - 指示是否可以使用每个顶点 （也称为每个基元） 明暗度率使用多个视区。

## <a name="specifying-shading-rate"></a>指定明暗度速率
为应用程序中的灵活性，有多种机制提供用于控制明暗度速率。 具体取决于硬件的功能层提供不同的机制。

### <a name="command-list"></a>命令列表
这是用于设置明暗度速率的最简单机制。 适用于所有层的。

你的应用程序可以指定使用粗像素大小[ **ID3D12GraphicsCommandList5::RSSetShadingRate**方法](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist5-rssetshadingrate)。 该 API 采用单个枚举参数。 该 API 提供用于呈现的质量级别的总体控件&mdash;能够在每个绘图的基础上设置的明暗度速率。

此状态的值表示通过[ **D3D12_SHADING_RATE** ](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate)枚举。

#### <a name="coarse-pixel-size-support"></a>粗像素大小支持
在所有层中支持的明暗度费率 1 x 1、 1 x 2、 2 x 2 和 2 x 2。

还有一项功能， *AdditionalShadingRatesSupported*，以指示是否在设备上支持 2 x 4、 4 x 2 和 4 × 4。

### <a name="screen-space-image-image-based"></a>屏幕空间映像 （基于映像的）
在第 2 层和更高版本，您可以使用屏幕空间映像指定的像素着色速率。

屏幕空间映像允许应用程序以创建"详细级别 (LOD) 掩码"映像，该值指示区域质量不同，如模糊运动将涵盖的方面，深度字段模糊、 透明对象或 HUD UI 元素。 图像的解析位于块效应;它不在呈现器目标的解决方法。 换而言之，明暗度数据的速率处指定的 8 x 8 或 16 x 16 像素磁贴，粒度 VRS 磁贴大小所示。

#### <a name="tile-size"></a>平铺大小
你的应用程序可以查询一个 API，用于检索其设备的受支持的 VRS 磁贴大小。

磁贴是方形，和大小是指该图块的宽度或高度以纹素。

如果硬件不支持第 2 层变量速率明暗度，磁贴大小的功能查询将返回 0。

如果硬件*does*支持第 2 层变量速率明暗度，然后平铺大小是下列值之一。

- 8
- 16
- 32

#### <a name="screen-space-image-size"></a>屏幕空间图像大小
对于大小 {rtWidth，rtHeight} 的呈现器目标，使用给定的磁贴大小命名**VRSTileSize**，将介绍它的屏幕空间映像是这些维度。

```cpp
{ ceil((float)rtWidth / VRSTileSize), ceil((float)rtHeight / VRSTileSize) }
```

屏幕空间图像的左上角 （0，0） 已锁定到呈现器目标的左上角 （0，0）。

若要查找 （x，y） 坐标的呈现器目标，除窗口空间坐标 (x，y) 按磁贴大小，忽略小数部分位数中特定位置对应的磁贴。

如果屏幕空间图像是大于它必须能够为给定的呈现器目标，则不会使用到右和/或底部的额外部分。

如果屏幕空间映像对于给定的呈现器目标太小，任何尝试的读取超出其实际的盘区的映像从生成 1 x 1 的默认底纹速度。 这是因为屏幕空间图像的左上角 （0，0） 已锁定到呈现器目标的左上角 （0，0） 和"超出了呈现器目标区阅读"x 意味着读取过的很好的值和 y。

#### <a name="format-layout-resource-properties"></a>格式、 布局、 资源属性
此图面的格式是一种单通道 8 位面 ([**DXGI_FORMAT_R8_UINT**](/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format))。

资源是维度**TEXTURE2D**。

不能数组或 mipped。 它显式必须具有一个 mip 级别。

它具有示例计数 1 和样本质量 0。

它具有纹理布局**未知**。 因为不允许跨适配器，它隐式不能为行优先布局。

在其中填充屏幕空间图像数据的预期的方法是为
1. 使用计算着色器; 将数据写入屏幕空间映像绑定为 UAV 或
2. 将数据复制到屏幕空间映像。

创建屏幕空间映像时，允许使用这些标志。

- NONE
- ALLOW_UNORDERED_ACCESS
- DENY_SHADER_RESOURCE

不允许这些标志。

- ALLOW_RENDER_TARGET
- ALLOW_DEPTH_STENCIL
- ALLOW_CROSS_ADAPTER
- ALLOW_SIMULTANEOUS_ACCESS
- VIDEO_DECODE_REFERENCE_ONLY

资源的堆类型不能为上传，也不 READBACK。

该资源不能为 SIMULTANEOUS_ACCESS。 该资源不能是跨适配器。

#### <a name="data"></a>数据
屏幕空间映像的每个字节对应的值[ **D3D12_SHADING_RATE** ](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate)枚举。

#### <a name="resource-state"></a>资源状态
转换为只读状态时的屏幕空间映像作为所需要的资源。 只读状态[ **D3D12_RESOURCE_STATE_SHADING_RATE_SOURCE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_states)，定义为此目的。

从该状态中唤醒以再次变得可写状态的图像资源。

#### <a name="setting-the-image"></a>设置图像
指定着色器的屏幕空间映像设置上命令列表。

无法读取或写入从任何着色器阶段已设置为明暗度率源的资源。

一个`null`屏幕空间映像可以设置用于指定的着色器速率。 此操作将为 1 x 1 一致地用作从屏幕空间映像所占比例。 一开始可以认为屏幕空间图像设置为`null`。

#### <a name="promotion-and-decay"></a>升级和衰减
屏幕空间图像资源不具有任何特殊影响升级或衰减。

### <a name="per-primitive-attribute"></a>每个基元属性
每个基元属性添加了指定明暗度速率术语为导致顶点中的某个属性的功能。 此属性是平面着色&mdash;即传播到当前的三角形或基元的行中的所有像素。 每个基元属性的使用可以启用更精细地控制图像质量相比其他明暗度速率说明符。

每个基元属性是一个可设置语义名为`SV_ShadingRate`。 `SV_ShadingRate` 作为的一部分而存在[HLSL 着色器模型 6.4](/windows/desktop/direct3dhlsl/hlsl-shader-model-6-4-features-for-direct3d-12)。

如果 VS 或 GS 设置`SV_ShadingRate`，但未启用 VRS，则语义设置不起作用。 如果未设置值`SV_ShadingRate`，则指定每个基元，明暗度速率值为 1 x 1 假定为每个基元贡献。

### <a name="combining-shading-rate-factors"></a>组合明暗度速率因素
使用此关系图序列中应用的明暗度速率各种来源。

![合并器](images/Combiners.PNG "的合并器")

每个对 A 和 B 使用合并器组合。

\* 当通过顶点属性指定着色器速率。

- 如果使用几何着色器，则可以通过该指定明暗度的速率。
- 如果未使用几何着色器，导致顶点由指定的明暗度速率。

#### <a name="list-of-combiners"></a>合并器的列表
支持以下的合并器。 使用 Combiner (C) 和两个输入 （A 和 B）。

- **传递**。 C.xy = A.xy.
- **重写**。 C.xy = B.xy.
- **更高的质量**。 C.xy = min （A.xy，B.xy）。
- **降低质量**。 C.xy = 最大值 （A.xy，B.xy）。
- **应用相对于一个成本 B**。C.xy = min(maxRate, A.xy + B.xy).

其中`maxRate`是粗略像素在设备上允许的最大维度。 这将是

- **D3D12_AXIS_SHADING_RATE_2X** （即，值为 1） 如果 AdditionalShadingRatesSupported `false`。
- **D3D12_AXIS_SHADING_RATE_4X** （即，值为 2） 如果 AdditionalShadingRatesSupported `true`。

所选变量速率明暗度的合并器的设置通过在命令列表[ **ID3D12GraphicsCommandList5::RSSetShadingRate**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist5-rssetshadingrate)。

如果曾经设置没有的合并器，则它们继续保留在默认情况下，这是传递。

如果向合并器源[ **D3D12_AXIS_SHADING_RATE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_axis_shading_rate)，这不允许在支持表中，然后输入净化明暗度速率的*是*支持。

如果的合并器输出不对应平台上支持的明暗度速率，则结果将进行整理到明暗度速率*是*支持。

### <a name="default-state-and-state-clearing"></a>默认状态，并声明交换
所有明暗度速率源，即

- 管道状态指定指定的费率 （上命令列表），
- 屏幕空间映像指定速率和
- 每个基元属性

具有默认值为**D3D12_SHADING_RATE_1X1**。 默认的合并器是 {传递，传递}。

如果指定没有屏幕空间图像，则为 1 x 1 的明暗度速率被推断从该源。

如果指定没有每个基元的属性，则为 1 x 1 的明暗度速率被推断从该源。

[ID3D12CommandList::ClearState](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearstate)将管道状态指定速率重置为默认值，并选择为默认值为"无屏幕空间映像"的屏幕空间映像。

## <a name="querying-shading-rate-by-using-svshadingrate"></a>通过使用 SV_ShadingRate 查询明暗度速率
它可用于了解在任何给定的像素着色器调用的硬件选择了什么明暗度速率。 这可能会使各种 PS 代码中的优化。 一个仅限 PS 的系统变量， `SV_ShadingRate`，提供有关明暗度速率的信息。

### <a name="type"></a>在任务栏的搜索框中键入
这种语义的类型是 uint。

### <a name="data-interpretation"></a>数据的解释
数据被解释为值[ **D3D12_SHADING_RATE** ](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate)枚举。

### <a name="if-vrs-is-not-being-used"></a>如果未使用 VRS
如果粗略像素明暗度未使用，然后`SV_ShadingRate`读回为 1 x 1，表示正常的像素为单位的值。

### <a name="behavior-under-sample-based-execution"></a>在基于样本的执行行为
如果其输入像素着色器编译将失败`SV_ShadingRate`并使用基于样本的执行&mdash;例如，通过输入`SV_SampleIndex`，也可以使用示例内插关键字。

> ### <a name="remarks-on-deferred-shading"></a>在延迟的明暗度的备注
>
> 延迟的明暗度应用程序的照明阶段可能需要知道什么明暗度速率使用屏幕上的哪些区域。 这是以便照明传递调度粗费率可以启动。 `SV_ShadingRate`变量可用于实现此目的，如果写出到 gbuffer。

## <a name="depth-and-stencil"></a>深度和模具
使用粗像素明暗度时，会始终计算的深度和模具和覆盖并将其发出的完整示例解析。

## <a name="using-the-shading-rate-requested"></a>使用请求的明暗度费率
对于所有层，它应该，如果明暗度速率不请求，并且它支持设备-和-MSAA-级别的组合，那么它就是由硬件提供的明暗度速率。

请求的明暗度速率意味着明暗度速率计算作为的合并器输出 (请参阅[组合明暗度速率因素](#combining-shading-rate-factors)本主题中的部分)。

受支持的明暗度速率是 1 x 1，1 x 2、 2 x 1 或 2 x 2 英寸呈现操作其中的样本数小于或等于四个。 如果*AdditionalShadingRatesSupported*功能`true`，则某些样本计数为 2 x 4、 4 x 2 和 4 × 4 是否还支持明暗度费率 (请参阅中的表[明暗度变量速率 (VRS)](#with-variable-rate-shading-vrs)本主题中的部分)。

## <a name="screen-space-derivatives"></a>屏幕空间派生类
粗像素明暗度会影响计算的相邻像素到渐变。 例如，当使用 2 x 2 粗略像素时，渐变相比大小的两倍时将不使用粗像素为单位。 你的应用程序可能要调整着色器对此进行弥补&mdash;或不是，根据所需的功能。

由于 mips 的选择依据的屏幕空间派生，粗略像素明暗度的使用情况会影响 mip 所选内容。 粗像素明暗度的使用情况会导致较少详细信息的 mips 选择相比较时不使用粗像素为单位。

## <a name="attribute-interpolation"></a>属性内插
像素着色器的输入可能会基于其源顶点内插。 由于变量速率明暗度会影响由像素着色器每次调用写入目标的区域，它与交互，属性内插。 三种类型的内插是 center、 质心和示例。

### <a name="center"></a>中心
粗像素的中心内插位置是完整粗略像素区域的几何中心。 `SV_Position` 始终在粗像素区域的中心插入。

### <a name="centroid"></a>质心
粗像素明暗度用于 MSAA 时, 为每个正常像素，但仍然会将写入到示例为目标的 MSAA 级别分配的全部数量。 因此，质心内插位置，将考虑粗略像素中的正常像素的所有示例。 话虽如此，质心内插位置被定义为第一个涵盖示例，示例索引按升序排列。 该示例的有效范围是 AND ed 与光栅器状态 SampleMask 的相应位。

> [!NOTE]
> 在第 1 层上使用粗像素明暗度时，SampleMask 始终是一个完整的掩码。 如果 SampleMask 配置为不是完整的掩码，粗略像素明暗度时禁用了第 1 层。

### <a name="sample-based-execution"></a>基于样本的执行
基于样本的执行，或*超级取样*&mdash;这导致的使用示例内插功能&mdash;可以用于粗像素明暗度，并导致像素着色器调用每个样本。 像素着色器调用的目标的样本计数 N，N 次，每个正常的像素。

### <a name="evaluateattributesnapped"></a>EvaluateAttributeSnapped
拉取模型内部函数都不符合第 1 层上的粗略像素明暗度。 如果尝试在第 1 层的粗略像素明暗度中使用拉模型内部函数，则会自动禁用粗略像素明暗度。

内部函数`EvaluateAttributeSnapped`允许用于在第 2 层的粗略像素明暗度。 始终是因为，其语法都是相同的。

```hlsl
numeric EvaluateAttributeSnapped(   
    in attrib numeric value, 
    in int2 offset);
```

有关上下文，`EvaluateAttributeSnapped`具有两个字段是偏移量的参数。 当使用没有粗略像素明暗度的情况下，只需较低序位四个出完整的 32 使用位。 这些四位代表范围 [-8，7 之内]。 此范围内像素的 16x16 网格。 范围表示这类像素的顶部和左侧边缘为包括在内，并且不是下边框和右边缘。 偏移量 （-8，-8） 位于左上角，偏移量 （7，7） 为通过右下角。 偏移量 （0，0） 是像素的中心。

与粗糙像素明暗度一起使用时`EvaluateAttributeSnapped`的偏移量参数是能够指定更多的位置。 偏移量的参数选择一个 16x16 网格，用于正常的每个像素，并且有多个正常的像素为单位。 可表示的范围和后续使用的比特数取决于粗像素大小。 粗像素的顶部和左侧边缘为包括在内，并且下边框和右边缘不。

下表描述了的解释`EvaluateAttributeSnapped`的偏移量的每个粗糙的像素大小参数。

#### <a name="evaluateattributesnappeds-offset-range"></a>EvaluateAttributeSnapped 的偏移量范围

|粗像素大小  |可编制索引的范围             |可表示的范围大小  |所需的位数 {x，y}  |可使用的位的二进制掩码          |    
|------------------:|---------------------------:|-------------------------:|-----------------------------:|-----------------------------------:|    
|1 x 1 （精确）         |{\[-8, 7\], \[-8, 7\]}      |{16, 16}                  |{4, 4}                        |{000000000000xxxx，000000000000xxxx}|    
|1x2                |{\[-8, 7\], \[-16, 15\]}    |{16, 32}                  |{4, 5}                        |{000000000000xxxx，00000000000xxxxx}|    
|2x1                |{\[-16, 15\], \[-8, 7\]}    |{32, 16}                  |{5, 4}                        |{00000000000xxxxx，000000000000xxxx}|    
|2x2                |{\[-16, 15\], \[-16, 15\]}  |{32, 32}                  |{5, 5}                        |{00000000000xxxxx，00000000000xxxxx}|    
|2x4                |{\[-16, 15\], \[-32, 31\]}  |{32, 64}                  |{5, 6}                        |{00000000000xxxxx，0000000000xxxxxx}|    
|4x2                |{\[-32, 31\], \[-16, 15\]}  |{64, 32}                  |{6, 5}                        |{0000000000xxxxxx，00000000000xxxxx}|    
|4x4                |{\[-32, 31\], \[-32, 31\]}  |{64, 64}                  |{6, 6}                        |{0000000000xxxxxx，0000000000xxxxxx}|   

下表是转换到从定点十进制和小数部分组成的表示形式的指南。 中的二进制掩码的第一个可用位是符号位和二进制掩码的其余部分包含的数值部分。

传递给四位值的数字方案`EvaluateAttributeSnapped`并不特定于变量速率明暗度。 它是此处各个出于完整性的考虑。

四位值。

| 二进制值 | 十进制  | 小数 |
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

为五位值。

| 二进制值 | 十进制  | 小数 |
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

六位值。

| 二进制值 | 十进制  | 小数 |
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

在使用正常的像素与相同的方式`EvaluateAttributeSnapped`的网格 evaluateable 位置居中粗略像素中心时使用粗像素明暗度。

## <a name="setsamplepositions"></a>SetSamplePositions
当 API [ **ID3D12GraphicsCommandList1::SetSamplePositions** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist1-setsamplepositions)用于粗明暗度，该示例将定位正常像素的 API 集。

## <a name="svcoverage"></a>SV_Coverage
如果`SV_Coverage`是声明为着色器输入或输出在第 1 层，则禁用粗略像素明暗度。

可以使用`SV_Coverage`语义与粗糙像素着色第 2 层，和它反映 MSAA 目标的示例正在编写的。

何时使用粗像素着色&mdash;允许多个源像素组成磁贴&mdash;覆盖率掩码表示来自该磁贴的所有示例。

提供与 MSAA 粗略像素明暗度的兼容性，指定所需的覆盖率比特数而异。 例如，使用 4 个 x MSAA 资源使用[ **D3D12_SHADING_RATE_2x2**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate)、 粗略的每个像素写入四个正常的像素为单位，并很好的每个像素具有四个示例。 这意味着每个粗糙像素写入 4 * 4 = 16 个示例。

### <a name="number-of-coverage-bits-needed"></a>所需的覆盖率比特数
下表指示多少个覆盖率位所需的粗略像素大小和 MSAA 级别的每个组合。

![NumberOfCoverageBits](images/NumberOfCoverageBits.PNG "NumberOfCoverageBits")

如下表所示，不能使用粗像素来写入到 16 个以上示例使用变量速率明暗度功能通过 Direct3D 12 公开一次。 此限制的原因有关的 MSAA 级别允许使用何种粗略的像素大小的 Direct3D 12 的约束 (请参阅中的表[明暗度变量速率 (VRS)](#with-variable-rate-shading-vrs)本主题中的部分)。

### <a name="ordering-and-format-of-bits-in-the-coverage-mask"></a>排序和格式中的覆盖率掩码的位数
覆盖掩码的位遵循明确定义的顺序。 掩码包含 coverages 从左到右、 自上而下的像素到下 （列优先） 的顺序。 覆盖率 bits 是语义的覆盖范围的低顺序位和密集打包在一起。

下表显示了支持的粗略像素大小和 MSAA 级别的组合覆盖率掩码格式。

![Coverage1x](images/Coverage1x.PNG "Coverage1x")

下表描述了其中每个像素都有两个示例的索引 0 和 1 2 x MSAA 个像素。

像素中的标签的位置用于说明用途，并且并不一定意味着该像素; 上的取样空间 {X，Y} 位置尤其是考虑到示例的位置，可以以编程方式更改。 示例引用通过它们从 0 开始的索引。

![Coverage2x](images/Coverage2x.PNG "Coverage2x")

下表显示 4 个 x MSAA 像素，其中每个像素都有四个示例的索引 0、 1、 2 和 3。

![Coverage4x](images/Coverage4x.PNG "Coverage4x")

## <a name="discard"></a>丢弃
当语义 HLSL`discard`使用了粗略的像素着色粗略的像素为单位将被丢弃。

## <a name="target-independent-rasterization-tir"></a>独立于目标的光栅化 (TIR)
使用粗像素明暗度时，不支持 TIR。

## <a name="raster-order-views-rovs"></a>光栅顺序视图 (ROVs)
ROV 联锁被指定为运行正常的像素的粒度。 如果明暗度执行每个样本，然后联锁操作示例粒度。

## <a name="conservative-rasterization"></a>传统型光栅化
传统型光栅化可使用变量速率明暗度。 传统型光栅化用于粗像素明暗度，粗略像素中的正常像素的保守光栅化获得完整的覆盖范围。

### <a name="coverage"></a>覆盖范围
使用传统型光栅化时，语义的覆盖范围将包含完整掩码正常像素的介绍和正常像素，且不为 0。

## <a name="bundles"></a>捆绑包
您可以在绑定上调用变量速率明暗度 Api。

## <a name="render-passes"></a>呈现阶段
可以调用变量速率明暗度 Api[呈现处理](/windows/desktop/direct3d12/direct3d-12-render-passes)。

## <a name="calling-the-vrs-apis"></a>调用 VRS Api
此下一节介绍其明暗度变量速率进行到 Direct3D 12 应用程序访问的方式。

### <a name="capability-querying"></a>查询功能

若要查询的适配器的明暗度变量速率功能，请调用[ **ID3D12Device::CheckFeatureSupport** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)与[ **D3D12_FEATURE::D3D12_FEATURE_D3D12_OPTIONS6** ](/windows/desktop/api/d3d12/ne-d3d12-d3d12_feature)，并提供[ **D3D12_FEATURE_DATA_D3D12_OPTIONS6**结构](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options6)要为你填充的函数。 **D3D12_FEATURE_DATA_D3D12_OPTIONS6**结构包含多个成员，其中一个枚举类型的[ **D3D12_VARIABLE_SHADING_RATE_TIER** ](/windows/desktop/api/d3d12/ne-d3d12-d3d12_variable_shading_rate_tier) (D3D12_FEATURE_DATA_D3D12_OPTIONS6::VariableShadingRateTier)，和一个指示是否支持后台处理 (D3D12_FEATURE_DATA_D3D12_OPTIONS6::BackgroundProcessingSupported)。

若要查询的第 1 层功能，例如，你可以执行此操作。

```cpp
D3D12_FEATURE_DATA_D3D12_OPTIONS6 options;
return 
    SUCCEEDED(m_device->CheckFeatureSupport(
        D3D12_FEATURE_D3D12_OPTIONS6, 
        &options, 
        sizeof(options))) && 
    options.ShadingRateTier == D3D12_SHADING_RATE_TIER_1;
```

### <a name="shading-rates"></a>明暗度费率

中的值[ **D3D12_SHADING_RATE**枚举](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate)组织，使明暗度的速率是到两个轴，可以轻松地分解每个轴的值简洁表示对数空间中根据[ **D3D12_AXIS_SHADING_RATE**枚举](/windows/desktop/api/d3d12/ne-d3d12-d3d12_axis_shading_rate)。

您可以编写一个宏来组合两个轴明暗度费率到如下的明暗度率。

```cpp
#define D3D12_MAKE_COARSE_SHADING_RATE(x,y) ((x) << 2 | (y))
D3D12_MAKE_COARSE_SHADING_RATE(
    D3D12_AXIS_SHADING_RATE_2X, 
    D3D12_AXIS_SHADING_RATE_1X)
```

该平台还提供了在中定义这些宏`d3d12.h`。

```cpp
#define D3D12_GET_COARSE_SHADING_RATE_X_AXIS(x) ((x) >> 2 )
#define D3D12_GET_COARSE_SHADING_RATE_Y_AXIS(y) ((y) & 3 )
```

这些可用于剖析和了解`SV_ShaderRate`。

> [!NOTE]
> 此数据解释被专门针对描述屏幕空间映像，可以由着色器操作。 讨论的前面的章节中进一步。 但没有理由不具有一致的粗略像素大小，以将定义使用无处不在包括设置命令级别明暗度速率时。

### <a name="setting-command-level-shading-rate-and-combiners"></a>设置命令级别明暗度速率和合并器
通过指定明暗度速率和 （可选） 的合并器[ **ID3D12GraphicsCommandList5::RSSetShadingRate** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist5-rssetshadingrate)方法。 您传递[ **D3D12_SHADING_RATE** ](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate)值的基的明暗度速率和的可选数组[D3D12_SHADING_RATE_COMBINER](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shading_rate_combiner)值。

### <a name="preparing-the-screen-space-image"></a>准备屏幕空间映像
指定可使用明暗度速率映像的只读资源状态指[D3D12_RESOURCE_STATES::D3D12_RESOURCE_STATE_SHADING_RATE_SOURCE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_states)。

### <a name="setting-the-screen-space-image"></a>设置屏幕空间图像
指定通过屏幕空间图像[ **ID3D12GraphicsCommandList5::RSSetShadingRateImage** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist5-rssetshadingrateimage)方法。

```cpp
m_commandList->RSSetShadingRateImage(screenSpaceImage);
```

### <a name="querying-the-tile-size"></a>查询平铺大小
您可以查询的磁贴大小[ **D3D12_FEATURE_DATA_D3D12_OPTIONS6::ShadingRateImageTileSize** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options6)成员。 请参阅[功能查询](#capability-querying)上面。

检索一个维度时，由于水平和垂直尺寸始终相同。 如果系统的功能[ **D3D12_SHADING_RATE_TIER_NOT_SUPPORTED**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_variable_shading_rate_tier)，然后平铺大小，则返回 0。
