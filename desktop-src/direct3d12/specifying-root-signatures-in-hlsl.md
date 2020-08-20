---
title: 在 HLSL 中指定根签名
description: 如果在 HLSL 着色器模型 5.1 中指定根签名，则无需在 C++ 代码中指定这些根签名。
ms.assetid: 399F5E91-B017-4F5E-9037-DC055407D96F
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 236876e22c3e1e0bb849ec1e1bc7d45692c900d6
ms.sourcegitcommit: 592c9bbd22ba69802dc353bcb5eb30699f9e9403
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2020
ms.locfileid: "88644055"
---
# <a name="specifying-root-signatures-in-hlsl"></a>在 HLSL 中指定根签名

如果在 HLSL 着色器模型 5.1 中指定根签名，则无需在 C++ 代码中指定这些根签名。

-   [示例 HLSL 根签名](#an-example-hlsl-root-signature)
    -   [根签名版本 1.0](#root-signature-version-10)
    -   [根签名版本 1.1](#root-signature-version-11)
-   [RootFlags](#rootflags)
-   [根常量](#root-constants)
-   [可见性](#visibility)
-   [根级 CBV](#root-level-cbv)
-   [根级 SRV](#root-level-srv)
-   [根级 UAV](#root-level-uav)
-   [描述符表](#descriptor-table)
-   [静态采样器](#static-sampler)
-   [编译 HLSL 根签名](#compiling-an-hlsl-root-signature)
-   [使用 FXC 编译器处理根签名](#manipulating-root-signatures-with-the-fxc-compiler)
-   [备注](#notes)
-   [相关主题](#related-topics)

## <a name="an-example-hlsl-root-signature"></a>示例 HLSL 根签名

可在 HLSL 中将根签名指定为字符串。 该字符串包含一个逗号分隔的子句集合，这些子句描述根签名的组成部分。 根签名在任何一个管道状态对象 (PSO) 的不同着色器中应该相同。 下面是一个示例：

### <a name="root-signature-version-10"></a>根签名版本 1.0

``` syntax
#define MyRS1 "RootFlags( ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT | " \
                         "DENY_VERTEX_SHADER_ROOT_ACCESS), " \
              "CBV(b0, space = 1), " \
              "SRV(t0), " \
              "UAV(u0, visibility = SHADER_VISIBILITY_GEOMETRY), " \
              "DescriptorTable( CBV(b0), " \
                               "UAV(u1, numDescriptors = 2), " \
                               "SRV(t1, numDescriptors = unbounded)), " \
              "DescriptorTable(Sampler(s0, numDescriptors = 2)), " \
              "RootConstants(num32BitConstants=1, b9), " \
              "DescriptorTable( UAV(u3), " \
                               "UAV(u4), " \
                               "UAV(u5, offset=1)), " \

              "StaticSampler(s2)," \
              "StaticSampler(s3, " \
                             "addressU = TEXTURE_ADDRESS_CLAMP, " \
                             "filter = FILTER_MIN_MAG_MIP_LINEAR )"
```

### <a name="root-signature-version-11"></a>根签名版本 1.1

使用[根签名版本 1.1](root-signature-version-1-1.md) 可以针对根签名描述符和数据进行驱动程序优化。

``` syntax
#define MyRS1 "RootFlags( ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT | " \
                         "DENY_VERTEX_SHADER_ROOT_ACCESS), " \
              "CBV(b0, space = 1, flags = DATA_STATIC), " \
              "SRV(t0), " \
              "UAV(u0), " \
              "DescriptorTable( CBV(b1), " \
                               "SRV(t1, numDescriptors = 8, " \
                               "        flags = DESCRIPTORS_VOLATILE), " \
                               "UAV(u1, numDescriptors = unbounded, " \
                               "        flags = DESCRIPTORS_VOLATILE)), " \
              "DescriptorTable(Sampler(s0, space=1, numDescriptors = 4)), " \
              "RootConstants(num32BitConstants=3, b10), " \
              "StaticSampler(s1)," \
              "StaticSampler(s2, " \
                             "addressU = TEXTURE_ADDRESS_CLAMP, " \
                             "filter = FILTER_MIN_MAG_MIP_LINEAR )"
```

此定义将提供以下根签名，请注意：

-   使用了默认参数。
-   b0 和 (b0, space=1) 不冲突
-   u0 仅对几何着色器可见
-   u4 和 u5 是堆中同一描述符的别名

![使用高级着色器语言指定了一个根签名](images/hlsl-root-signature.png)

HLSL 根签名语言密切对应于 C++ 根签名 API，具有同等的表达力。 根签名指定为逗号分隔的子句序列。 子句的顺序非常重要，因为分析顺序决定了根签名中的槽位置。 每个子句采用一个或多个命名参数。 但是，参数的顺序并不重要。

## <a name="rootflags"></a>RootFlags

可选的 *RootFlags* 子句采用 0（默认值，表示无标志），或一个或多个预定义的根标志值（通过 OR“\|”运算符连接）。 允许的根标志值由 [**D3D12\_ROOT\_SIGNATURE\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_root_signature_flags) 定义。

例如：

``` syntax
RootFlags(0) // default value – no flags
RootFlags(ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT)
RootFlags(ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT | DENY_VERTEX_SHADER_ROOT_ACCESS)
```

## <a name="root-constants"></a>根常量

*RootConstants* 子句指定根签名中的根常量。 两个必需的参数是：*cbuffer* 的 *num32BitConstants* 和 *bReg*（对应于 C++ API 中 *BaseShaderRegister* 的寄存器）。 space（C++ API 中的 *RegisterSpace*）和 visibility（C++ 中的 *ShaderVisibility*）参数是可选的，默认值为：

``` syntax
RootConstants(num32BitConstants=N, bReg [, space=0, 
              visibility=SHADER_VISIBILITY_ALL ])
```

例如：

``` syntax
RootConstants(num32BitConstants=3, b3)
```

## <a name="visibility"></a>能见度

Visibility 是一个可选参数，可以使用 [**D3D12\_SHADER\_VISIBILITY**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shader_visibility) 中的值之一。

SHADER\_VISIBILITY\_ALL 将根参数广播到所有着色器。 在某些硬件上，此操作不会造成开销，但在其他硬件上，将数据分叉到所有着色器阶段会造成开销。 设置其中一个选项（例如 SHADER\_VISIBILITY\_VERTEX）会将根参数限制到单个着色器阶段。

将根参数设置到单个着色器阶段可在不同的阶段使用相同的绑定名称。 例如，`t0,SHADER_VISIBILITY_VERTEX` SRV 绑定和 `t0,SHADER_VISIBILITY_PIXEL` SRV 绑定是有效的。 但是，如果某个绑定的可见性设置为 `t0,SHADER_VISIBILITY_ALL`，则根签名是无效的。

## <a name="root-level-cbv"></a>根级 CBV

`CBV`（常量缓冲区视图）子句指定根级常量缓冲区 b-register 寄存器条目。 请注意，这是一个标量条目；无法指定根级别的范围。

``` syntax
CBV(bReg [, space=0, visibility=SHADER_VISIBILITY_ALL ])    //   Version 1.0
CBV(bReg [, space=0, visibility=SHADER_VISIBILITY_ALL,      // Version 1.1
            flags=DATA_STATIC_WHILE_SET_AT_EXECUTE ])
```

## <a name="root-level-srv"></a>根级 SRV

`SRV`（着色器资源视图）子句指定根级 SRV t-register 寄存器条目。 请注意，这是一个标量条目；无法指定根级别的范围。

``` syntax
SRV(tReg [, space=0, visibility=SHADER_VISIBILITY_ALL ])    //   Version 1.0
SRV(tReg [, space=0, visibility=SHADER_VISIBILITY_ALL,      // Version 1.1
            flags=DATA_STATIC_WHILE_SET_AT_EXECUTE ])
```

## <a name="root-level-uav"></a>根级 UAV

`UAV`（无序访问视图）子句指定根级 UAV u-register 寄存器条目。 请注意，这是一个标量条目；无法指定根级别的范围。

``` syntax
UAV(uReg [, space=0, visibility=SHADER_VISIBILITY_ALL ])    //   Version 1.0
UAV(uReg [, space=0, visibility=SHADER_VISIBILITY_ALL,      // Version 1.1
            flags=DATA_VOLATILE ])
```

例如：

``` syntax
UAV(u3)
```

## <a name="descriptor-table"></a>描述符表

`DescriptorTable` 子句本身是逗号分隔的描述符表子句列表，以及可选的可见性参数。 *DescriptorTable* 子句包括 CBV、SRV、UAV 和采样器。 请注意，其参数不同于根级子句的参数。

``` syntax
DescriptorTable( DTClause1, [ DTClause2, … DTClauseN,
                 visibility=SHADER_VISIBILITY_ALL ] )
```

描述符表 `CBV` 采用以下语法：

``` syntax
CBV(bReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND ])   // Version 1.0
CBV(bReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND      // Version 1.1
          , flags=DATA_STATIC_WHILE_SET_AT_EXECUTE ])
```

例如：

``` syntax
DescriptorTable(CBV(b0),SRV(t3, numDescriptors=unbounded))
```

必需的参数 *bReg* 指定 cbuffer 范围的起始寄存器。 *numDescriptors* 参数指定连续 cbuffer 范围中的描述符数目；默认值为 1。 当 *numDescriptors* 为数字时，该条目声明 cbuffer 范围 ` [Reg, Reg + numDescriptors - 1]`。 如果 *numDescriptors* 等于“unbounded”，则范围为 `[Reg, UINT_MAX]`，这意味着，应用必须确保它不会引用界外区域。 *offset* 字段表示  C++ API 中的 *OffsetInDescriptorsFromTableStart* 参数，即，与表开头位置的偏移量（描述符中）。 如果 offset 设置为 DESCRIPTOR\_RANGE\_OFFSET\_APPEND（默认值），则表示该范围紧接在上一个范围之后。 但是，输入特定的偏移量将允许范围相互重叠，因而允许寄存器别名。

描述符表 `SRV` 采用以下语法：

``` syntax
SRV(tReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND ])    // Version 1.0
SRV(tReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND,      // Version 1.1
            flags=DATA_STATIC_WHILE_SET_AT_EXECUTE ])
```

这类似于描述符表 `CBV` 条目，但指定的范围用于着色器资源视图。

描述符表 `UAV` 采用以下语法：

``` syntax
UAV(uReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND ])    // Version 1.0
UAV(uReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND,      // Version 1.1
            flags=DATA_VOLATILE ])
```

这类似于描述符表 `CBV` 条目，但指定的范围用于无序访问视图。

描述符表 `Sampler` 采用以下语法：

``` syntax
Sampler(sReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND ])  // Version 1.0
Sampler(sReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND,    // Version 1.1
                flags=0 ])
```

这类似于描述符表 `CBV` 条目，但指定的范围用于着色器采样器。 请注意，采样器不能与同一描述符表中其他类型的描述符相混合（因为它们在独立的描述符堆中）。

## <a name="static-sampler"></a>静态采样器

静态采样器表示 [**D3D12\_STATIC\_SAMPLER\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_static_sampler_desc) 结构。 *StaticSampler* 的必需参数是一个标量采样器 s-register 寄存器。其他参数是可选的，默认值如下所示。 大多数字段接受一组预定义的枚举。

``` syntax
StaticSampler( sReg,
              [ filter = FILTER_ANISOTROPIC, 
                addressU = TEXTURE_ADDRESS_WRAP,
                addressV = TEXTURE_ADDRESS_WRAP,
                addressW = TEXTURE_ADDRESS_WRAP,
                mipLODBias = 0.f,
                maxAnisotropy = 16,
                comparisonFunc = COMPARISON_LESS_EQUAL,
                borderColor = STATIC_BORDER_COLOR_OPAQUE_WHITE,
                minLOD = 0.f,         
                maxLOD = 3.402823466e+38f,
                space = 0, 
                visibility = SHADER_VISIBILITY_ALL ])
```

例如：

``` syntax
StaticSampler(s4, filter=FILTER_MIN_MAG_MIP_LINEAR)
```

参数选项非常类似于C++ API 调用，但 *borderColor* 除外，它限制为 HLSL 中的某个枚举。

过滤器字段可以是某个 [**D3D12\_FILTER**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_filter)。

每个地址字段可以是某个 [**D3D12\_TEXTURE\_ADDRESS\_MODE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_texture_address_mode)。

比较函数可以是某个 [**D3D12\_COMPARISON\_FUNC**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_comparison_func)。

边框颜色字段可以是某个 [**D3D12\_STATIC\_BORDER\_COLOR**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_static_border_color)。

可见性可以是某个 [**D3D12\_SHADER\_VISIBILITY**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shader_visibility)。

## <a name="compiling-an-hlsl-root-signature"></a>编译 HLSL 根签名

可以使用两种机制来编译 HLSL 根签名。 首先，可以通过 *RootSignature* 属性将根签名字符串附加到特定的着色器（以下示例使用 **MyRS1** 入口点）：

``` syntax
[RootSignature(MyRS1)]
float4 main(float4 coord : COORD) : SV_Target
{
…
}
```

编译器将创建并验证着色器的根签名 Blob，并将它连同着色器字节码一起嵌入到着色器 Blob。 编译器支持着色器模型 5.0 及更高版本的根签名语法。 如果根签名嵌入在着色器模型 5.0 着色器中，而该着色器已发送到 D3D11 运行时，则不同于 D3D12，D3D11 会以无提示方式忽略根签名部分。

另一种机制是创建独立的根签名 Blob。也许可对大量的着色器重用该 Blob，以节省空间。 [效应编译器工具](/windows/desktop/direct3dtools/fxc) (FXC) 支持 **rootsig\_1\_0** 和 **rootsig\_1\_1** 着色器模型。 通过常用的 /E 参数指定定义字符串的名称。 例如：

``` syntax
fxc.exe /T rootsig_1_1 MyRS1.hlsl /E MyRS1 /Fo MyRS1.fxo
```

请注意，也可以在命令行中传递根签名字符串定义，例如 /D MyRS1 ="..."。

## <a name="manipulating-root-signatures-with-the-fxc-compiler"></a>使用 FXC 编译器处理根签名

FXC 编译器从 HLSL 源文件创建着色器字节码。 此编译器有很多的可选参数，具体请参阅[效应编译器工具](/windows/desktop/direct3dtools/fxc)。

下表提供了一些使用 FXC 的示例，供你在管理 HLSL 创作的根签名时参考。



| 行 | 命令行                                                                 | 说明                                                                                                                                                                                                                              |
|------|------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1    | `fxc /T ps_5_1 shaderWithRootSig.hlsl /Fo rs1.fxo`                           | 为像素着色器 5.1 目标编译着色器，着色器源位于包含根签名的 shaderWithRootSig.hlsl 文件中。 着色器和根签名在 rs1.fxo 二进制文件中编译为独立的 Blob。    |
| 2    | `fxc /dumpbin rs1.fxo /extractrootsignature /Fo rs1.rs.fxo`                  | 从第 1 行创建的文件中提取根签名，因此，rs1.rs.fxo 文件仅包含根签名。                                                                                                                      |
| 3    | `fxc /dumpbin rs1.fxo /Qstrip_rootsignature /Fo rs1.stripped.fxo`            | 从第 1 行创建的文件中删除根签名，因此，rs1.stripped.fxo 文件包含不带根签名的着色器。                                                                                                       |
| 4    | `fxc /dumpbin rs1.stripped.fxo /setrootsignature rs1.rs.fxo /Fo rs1.new.fxo` | 将位于不同文件中的着色器和根签名合并到包含上述两个 Blob 的二进制文件中。 在此示例中，rs1.new.fx0 与第 1 行中的 rs1.fx0 相同。                                                           |
| 5    | `fxc /T rootsig_1_0 rootSigAndMaybeShaderInHereToo.hlsl /E RS1 /Fo rs2.fxo`  | 从不仅可以包含根签名的源创建独立的根签名二进制文件。 请注意 rootsig\_1\_0 目标，RS1 是 HLSL 文件中根签名 (\#define) 宏字符串的名称。 |



 

可通过 FXC 使用的功能也可以通过 [**D3DCompile**](/windows/desktop/direct3dhlsl/d3dcompile) 函数以编程方式使用。 此调用将编译带有根签名的着色器，或独立的根签名（设置 rootsig\_1\_0 目标）。 [**D3DGetBlobPart**](/windows/desktop/direct3dhlsl/d3dgetblobpart) 和 [**D3DSetBlobPart**](/windows/desktop/direct3dhlsl/d3dsetblobpart) 可将根签名提取和附加到现有 Blob。D3D\_BLOB\_ROOT\_SIGNATURE 用于指定根签名 Blob 部分类型。 [**D3DStripShader**](/windows/desktop/direct3dhlsl/d3dstripshader) 从 Blob 中删除根签名（使用 D3DCOMPILER\_STRIP\_ROOT\_SIGNATURE 标志）。

## <a name="notes"></a>说明

> [!Note]  
> 尽管我们强烈建议对着色器进行脱机编译，但如果必须在运行时编译着色器，请参阅 [**D3DCompile2**](/windows/desktop/direct3dhlsl/d3dcompile2) 的备注。

 

> [!Note]  
> 无需更改现有的 HLSL 资产即可处理要对其使用的根签名。

 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[使用 HLSL 5.1 的动态索引](dynamic-indexing-using-hlsl-5-1.md)
</dt> <dt>

[Direct3D 12 的 HLSL 着色器模型 5.1 功能](/windows/desktop/direct3dhlsl/hlsl-shader-model-5-1-features-for-direct3d-12)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> <dt>

[HLSL 中的资源绑定](resource-binding-in-hlsl.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> <dt>

[Shader Model 5.1](/windows/desktop/direct3dhlsl/shader-model-5-1)（着色器模型 5.1）
</dt> <dt>

[着色器指定的模具参考值](shader-specified-stencil-reference-value.md)
</dt> <dt>

[类型化的无序访问视图加载](typed-unordered-access-view-loads.md)
</dt> </dl>

 

 