---
title: 在 HLSL 中指定根签名
description: HLSL 着色器模型 5.1 中指定根签名并指定它们中的替代方法C++代码。
ms.assetid: 399F5E91-B017-4F5E-9037-DC055407D96F
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 31703f87b28f3f5eaddc8889aaac96a7c5da0fae
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223765"
---
# <a name="specifying-root-signatures-in-hlsl"></a>在 HLSL 中指定根签名

HLSL 着色器模型 5.1 中指定根签名并指定它们中的替代方法C++代码。

-   [示例 HLSL 根签名](#an-example-hlsl-root-signature)
    -   [根签名版本 1.0](#root-signature-version-10)
    -   [根签名版本 1.1](#root-signature-version-11)
-   [RootFlags](#rootflags)
-   [根常量](#root-constants)
-   [可见性](#visibility)
-   [Root-level CBV](#root-level-cbv)
-   [Root-level SRV](#root-level-srv)
-   [Root-level UAV](#root-level-uav)
-   [描述符表](#descriptor-table)
-   [静态取样器](#static-sampler)
-   [编译 HLSL 根签名](#compiling-an-hlsl-root-signature)
-   [操作使用 FXC 编译器根签名](#manipulating-root-signatures-with-the-fxc-compiler)
-   [注意](#notes)
-   [相关的主题](#related-topics)

## <a name="an-example-hlsl-root-signature"></a>示例 HLSL 根签名

根签名可以在 HLSL 中指定为字符串。 该字符串包含一个以逗号分隔的子句描述根签名构成组件的集合。 任何一个管道状态对象 (PSO) 的着色器根签名应相同。 下面是一个示例：

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

[根签名版本 1.1](root-signature-version-1-1.md)启用根签名描述符和数据驱动程序优化。

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

此定义将得到以下根签名，注意：

-   使用默认参数。
-   b0 和 (b0，空间 = 1) 不会发生冲突
-   u0 时才会显示为几何着色器
-   u4 和 u5 是化名为堆中的相同描述符

![使用高级别着色器语言指定根签名](images/hlsl-root-signature.png)

HLSL 根签名语言密切对应于C++根签名 Api 并具有等效的表达能力。 根签名被指定为一系列子句，用逗号分隔的。 子句的顺序至关重要，因为分析的顺序决定的插槽中的根签名。 每个子句采用一个或多个命名的参数。 参数的顺序不重要，但是使用。

## <a name="rootflags"></a>RootFlags

可选*RootFlags*子句将为 0 （默认值以不指示任何标志），或一个或多个预定义的根的标志值，通过或连接\|运算符。 允许的根的标志值定义由[ **D3D12\_根\_签名\_标志**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_root_signature_flags)。

例如：

``` syntax
RootFlags(0) // default value – no flags
RootFlags(ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT)
RootFlags(ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT | DENY_VERTEX_SHADER_ROOT_ACCESS)
```

## <a name="root-constants"></a>根常量

*RootConstants*子句指定根常量中的根签名。 有两个必需参数： *num32BitConstants*并*bReg* (注册对应于*BaseShaderRegister*在C++Api) 的*cbuffer*。 空间 (*RegisterSpace*中C++Api) 和可见性 (*ShaderVisibility*中C++) 参数是可选的并且默认值：

``` syntax
RootConstants(num32BitConstants=N, bReg [, space=0, 
              visibility=SHADER_VISIBILITY_ALL ])
```

例如：

``` syntax
RootConstants(num32BitConstants=3, b3)
```

## <a name="visibility"></a>可见性

可见性是一个可选参数，可以从值之一[ **D3D12\_着色器\_可见性**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_shader_visibility)。

着色器\_可见性\_所有广播到所有着色器根自变量。 某些硬件上这样做产生的费用，但在其他硬件上没有进行分叉到所有着色器阶段的数据的成本。 设置其中一个选项，如着色器\_可见性\_顶点，限制到单个的着色器阶段的根参数。

将根自变量设置为单一的着色器阶段允许在不同阶段使用的相同绑定名称。 例如，SRV 绑定`t0,SHADER_VISIBILITY_VERTEX`和 SRV 绑定`t0,SHADER_VISIBILITY_PIXEL`会有效。 但是，如果可见性设置以前`t0,SHADER_VISIBILITY_ALL`根签名将无效的绑定之一。

## <a name="root-level-cbv"></a>Root-level CBV

`CBV` （常量缓冲区视图） 子句指定的根级别常量缓冲区 b 登记注册表条目。 请注意，这是该标量项;不能指定根级别的范围。

``` syntax
CBV(bReg [, space=0, visibility=SHADER_VISIBILITY_ALL ])    //   Version 1.0
CBV(bReg [, space=0, visibility=SHADER_VISIBILITY_ALL,      // Version 1.1
            flags=DATA_STATIC_WHILE_SET_AT_EXECUTE ])
```

## <a name="root-level-srv"></a>Root-level SRV

`SRV` （着色器资源视图） 子句指定的根级别 SRV t 登记注册表条目。 请注意，这是该标量项;不能指定根级别的范围。

``` syntax
SRV(tReg [, space=0, visibility=SHADER_VISIBILITY_ALL ])    //   Version 1.0
SRV(tReg [, space=0, visibility=SHADER_VISIBILITY_ALL,      // Version 1.1
            flags=DATA_STATIC_WHILE_SET_AT_EXECUTE ])
```

## <a name="root-level-uav"></a>Root-level UAV

`UAV` （无序的访问视图） 子句指定的根级别 UAV u 登记注册表条目。 请注意，这是该标量项;不能指定根级别的范围。

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

`DescriptorTable`子句本身就是一系列以逗号分隔的描述符表子句，以及可选的可见性参数。 *DescriptorTable*子句包括 CBV、 SRV、 UAV 和采样器。 请注意，其参数不同于那些根级别子句。

``` syntax
DescriptorTable( DTClause1, [ DTClause2, … DTClauseN,
                 visibility=SHADER_VISIBILITY_ALL ] )
```

描述符表`CBV`具有以下语法：

``` syntax
CBV(bReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND ])   // Version 1.0
CBV(bReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND      // Version 1.1
          , flags=DATA_STATIC_WHILE_SET_AT_EXECUTE ])
```

例如：

``` syntax
DescriptorTable(CBV(b0),SRV(t3, numDescriptors=unbounded))
```

必需的参数*bReg*指定开始 Reg cbuffer 范围。 *NumDescriptors*参数中连续 cbuffer 指定说明符的数目范围; 默认值为 1。 该条目声明 cbuffer 范围` [Reg, Reg + numDescriptors - 1]`，当*numDescriptors*是一个数字。 如果*numDescriptors*是等于"unbounded"，范围是`[Reg, UINT_MAX]`，这意味着应用程序必须确保它未引用超出边界区域。 *偏移量*字段表示*OffsetInDescriptorsFromTableStart*中的参数C++Api，即中的偏移量 （描述符） 从表开始。 如果偏移量设置为描述符\_范围\_偏移量\_追加 （默认值），它表示的范围是直接在上一个范围之后。 但是，输入特定偏移量允许的范围重叠，从而允许注册别名。

描述符表`SRV`具有以下语法：

``` syntax
SRV(tReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND ])    // Version 1.0
SRV(tReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND,      // Version 1.1
            flags=DATA_STATIC_WHILE_SET_AT_EXECUTE ])
```

这是类似于描述符表`CBV`条目，但指定的范围是用于着色器资源视图。

描述符表`UAV`具有以下语法：

``` syntax
UAV(uReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND ])    // Version 1.0
UAV(uReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND,      // Version 1.1
            flags=DATA_VOLATILE ])
```

这是类似于描述符表`CBV`条目，但指定的范围是无序的访问视图。

描述符表`Sampler`具有以下语法：

``` syntax
Sampler(sReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND ])  // Version 1.0
Sampler(sReg [, numDescriptors=1, space=0, offset=DESCRIPTOR_RANGE_OFFSET_APPEND,    // Version 1.1
                flags=0 ])
```

这是类似于描述符表`CBV`条目指定的范围是为着色器取样器除外。 请注意，（因为它们是单独描述符堆中） 不能与其他类型的同一个描述符表中的描述符混合取样器。

## <a name="static-sampler"></a>静态取样器

静态取样器表示[ **D3D12\_静态\_采样器\_DESC** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_static_sampler_desc)结构。 必需参数*StaticSampler*是标量，采样器的寄存器地区其他参数是可选的如下所示的默认值。 大多数字段接受一组预定义的枚举。

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

参数选项为非常类似于C++API 调用除外*borderColor*，这仅限于在 HLSL 中枚举。

筛选器字段可以是之一[ **D3D12\_筛选器**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_filter)。

地址字段可以每个是之一[ **D3D12\_纹理\_地址\_模式**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_texture_address_mode)。

比较函数可以是之一[ **D3D12\_比较\_FUNC**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_comparison_func)。

边框颜色字段可以是之一[ **D3D12\_静态\_边框\_颜色**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_static_border_color)。

可见性可以是之一[ **D3D12\_着色器\_可见性**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_shader_visibility)。

## <a name="compiling-an-hlsl-root-signature"></a>编译 HLSL 根签名

有两种机制来编译 HLSL 根签名。 首先，就可以将根签名字符串附加到通过特定的着色器*RootSignature*属性 (在以下示例中，使用**MyRS1**入口点):

``` syntax
[RootSignature(MyRS1)]
float4 main(float4 coord : COORD) : SV_Target
{
…
}
```

编译器将创建和验证为着色器根签名 blob 并将它与着色器字节代码一起嵌入到着色器 blob。 编译器支持的着色器模型 5.0 及更高版本的根签名语法。 如果根签名嵌入在着色器模型 5.0 着色器和该着色器发送到 D3D11 运行时，而不是 D3D12，根签名部分将获取以无提示方式忽略 D3D11。

其他机制是创建一个独立根签名 blob，可能要重用与大量的着色器，节省空间。 [效果编译器工具](https://msdn.microsoft.com/library/windows/desktop/bb232919)(FXC) 同时支持两者**rootsig\_1\_0**并**rootsig\_1\_1**着色器模型。 通过常用 /E 参数指定的定义字符串的名称。 例如：

``` syntax
fxc.exe /T rootsig_1_1 MyRS1.hlsl /E MyRS1 /Fo MyRS1.fxo
```

请注意，根签名字符串定义还可以传递命令行，例如、 /D MyRS1 ="..."。

## <a name="manipulating-root-signatures-with-the-fxc-compiler"></a>操作使用 FXC 编译器根签名

FXC 编译器从 HLSL 源文件创建着色器字节代码。 有很多此编译器的可选参数，请参阅[效果编译器工具](https://msdn.microsoft.com/library/windows/desktop/bb232919)。

用于管理 HLSL 创作根签名下, 表提供了使用 FXC 的一些示例。



| Line | 命令行                                                                 | 描述                                                                                                                                                                                                                              |
|------|------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1    | `fxc /T ps_5_1 shaderWithRootSig.hlsl /Fo rs1.fxo`                           | 编译的着色器对像素着色器 5.1 目标，着色器源处于 shaderWithRootSig.hlsl 文件，其中包含的根签名。 作为单独的 blob rs1.fxo 二进制文件中编译着色器和根签名。    |
| 2    | `fxc /dumpbin rs1.fxo /extractrootsignature /Fo rs1.rs.fxo`                  | 从创建第 1 行，因此 rs1.rs.fxo 文件包含只是根签名的文件中提取根签名。                                                                                                                      |
| 3    | `fxc /dumpbin rs1.fxo /Qstrip_rootsignature /Fo rs1.stripped.fxo`            | 删除创建的第 1 行，因此 rs1.stripped.fxo 文件包含与任何根签名的着色器文件的根签名。                                                                                                       |
| 4    | `fxc /dumpbin rs1.stripped.fxo /setrootsignature rs1.rs.fxo /Fo rs1.new.fxo` | 将合并到包含这两个 blob 的二进制文件的单独文件中的着色器和根签名。 在此示例中 rs1.new.fx0 将具有与在第 1 行 rs1.fx0 相同。                                                           |
| 5    | `fxc /T rootsig_1_0 rootSigAndMaybeShaderInHereToo.hlsl /E RS1 /Fo rs2.fxo`  | 从可能包含不止是根签名的源创建一个独立根签名二进制文件。 请注意 rootsig\_1\_0 目标和 RS1 是根签名的名称 (\#定义) 宏 HLSL 文件中的字符串。 |



 

可通过 FXC 功能也是可以以编程方式使用[ **D3DCompile** ](https://msdn.microsoft.com/library/windows/desktop/dd607324)函数。 此调用将编译的着色器根签名或独立根签名 (设置 rootsig\_1\_0 目标)。 [**D3DGetBlobPart** ](https://msdn.microsoft.com/library/windows/desktop/ff728674)并[ **D3DSetBlobPart** ](https://msdn.microsoft.com/library/windows/desktop/hh446880)可以提取并将根签名附加到现有 blob。  D3D\_BLOB\_根\_签名用于指定根签名 blob 部分类型。 [**D3DStripShader** ](https://msdn.microsoft.com/library/windows/desktop/dd607335)移除根签名 (使用 D3DCOMPILER\_条带\_根\_签名标志) 从 blob。

## <a name="notes"></a>注释

> [!Note]  
> 强烈建议脱机的着色器编译，如果着色器必须在运行时编译，而是指的备注[ **D3DCompile2**](https://msdn.microsoft.com/library/windows/desktop/hh446869)。

 

> [!Note]  
> HLSL 的现有资产不需要更改以处理根签名与它们一起使用。

 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[动态索引使用 HLSL 5.1](dynamic-indexing-using-hlsl-5-1.md)
</dt> <dt>

[HLSL 着色器模型 5.1 功能对于 Direct3D 12](https://msdn.microsoft.com/library/windows/desktop/dn933267)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> <dt>

[在 HLSL 中绑定的资源](resource-binding-in-hlsl.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> <dt>

[着色器模型 5.1](https://msdn.microsoft.com/library/windows/desktop/dn933277)
</dt> <dt>

[着色器指定模具参考值](shader-specified-stencil-reference-value.md)
</dt> <dt>

[类型化无序的访问视图加载](typed-unordered-access-view-loads.md)
</dt> </dl>

 

 




