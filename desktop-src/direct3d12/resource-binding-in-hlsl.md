---
title: HLSL 中的资源绑定
description: 本部分介绍 Direct3D 12 中使用高级着色语言 (HLSL) 着色器模型 5.1 的某些特定功能。
ms.assetid: 3CD4BDAD-8AE3-4DE0-B3F8-9C9F9E83BBE9
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: dde879254fd98140b754554f6aed4f98656063cc
ms.sourcegitcommit: 27a9dfa3ef68240fbf09f1c64dff7b2232874ef4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66725408"
---
# <a name="resource-binding-in-hlsl"></a>HLSL 中的资源绑定

本部分介绍 Direct3D 12 中使用高级着色语言 (HLSL) [着色器模型 5.1](https://docs.microsoft.com/windows/desktop/direct3dhlsl/shader-model-5-1) 的某些特定功能。 所有 Direct3D 12 硬件都支持着色器模型 5.1，因此对此模型的支持不依赖于硬件功能级别。

-   [资源类型和数组](#resource-types-and-arrays)
-   [描述符数组和纹理数组](#descriptor-arrays-and-texture-arrays)
-   [资源别名](#resource-aliasing)
-   [差异和派生对象](#divergence-and-derivatives)
-   [像素着色器中的 UAV](#uavs-in-pixel-shaders)
-   [常量缓冲区](#constant-buffers)
-   [SM5.1 中的字节码更改](#bytecode-changes-in-sm51)
-   [示例 HLSL 声明](#example-hlsl-declarations)
-   [相关主题](#related-topics)

## <a name="resource-types-and-arrays"></a>资源类型和数组

着色器模型 5 (SM5.0) 资源语法使用“寄存器”关键字将有关资源的重要信息中继到 HLSL 编译器。 例如，以下语句声明在槽 t3、t4、t5 和 t6 中绑定的四个纹理的数组。 t3 是唯一显示在该语句中的寄存器槽，而它仅仅是四个寄存器的数组中的第一个。

``` syntax
Texture2D<float4> tex1[4] : register(t3)
```

HLSL 中的着色器模型 5.1 (SM5.1) 资源语法基于现有的寄存器资源语法，旨在更方便地完成移植。 HLSL 中的 D3D12 资源绑定到逻辑寄存器空间中的虚拟寄存器：

-   t – 表示着色器资源视图 (SRV)
-   s – 表示采样器
-   u – 表示无序访问视图 (UAV)
-   b – 表示常量缓冲区视图 (CBV)

引用着色器的根签名必须与声明的寄存器槽兼容。 例如，根签名的以下部分将与使用的纹理槽 t3 至 t6 兼容，因为它使用槽 t0 至 t98 描述了描述符表。

``` syntax
DescriptorTable( CBV(b1), SRV(t0,numDescriptors=99), CBV(b2) )
```

资源声明可以是标量值、1D 数组或多维数组：

``` syntax
Texture2D<float4> tex1     : register(t3,  space0)
Texture2D<float4> tex2[4]     : register(t10)
Texture2D<float4> tex3[7][5][3]   : register(t20, space1)
```

SM5.1 使用的资源类型和元素类型与 SM5.0 相同。 声明限制现在更加灵活，仅受运行时/硬件限制的约束。 “空间”关键字指定声明的变量所绑定到的逻辑寄存器空间。 如果省略“空间”关键字，则会将默认空间索引 0 隐式分配到范围（因此，上述 `tex2` 范围驻留在 `space0` 中）。 `register(t3,  space0)` 永远不会与 `register(t3,  space1)` 相冲突，也不会与可能包含 t3 的另一个空间中的任何数组相冲突。

数组资源可以采用无限制的大小，该大小是通过将第一个维度指定为空或 0 声明的：

``` syntax
Texture2D<float4> tex1[]       : register(t0)
```

匹配的描述符表可能是：

``` syntax
DescriptorTable( CBV(b1), UAV(u0, numDescriptors = 4), SRV(t0, numDescriptors=unbounded)
```

HLSL 中的无限制数组匹配描述符表中使用 `numDescriptors` 设置的固定数字，HLSL 中的固定大小匹配描述符表中的无限制声明。

允许多维数组，包括无限制的大小。 这些多维数组将在寄存器空间中平展。

``` syntax
Texture2D<float4> tex3[3000][10] : register(t0,space1); // t0-t29999 in space1
Texture2D<float4> tex3[0][5][3]   : register(t5, space1)
```

不允许资源范围别名。 换而言之，对于每个资源类型（t、s、u、b），声明的寄存器范围不得重叠。 这也包括无限制的范围。 在不同寄存器空间中声明的范围不得重叠。 请注意，无限制 tex2 驻留在 space0 中，无限制 tex3 驻留在 space1 中，因此它们不会重叠。

对于已声明为数组的资源，只需通过编制其索引就能对其进行访问，例如：

``` syntax
Texture2D<float4> tex1[400] : register(t3);
sampler samp[7] : register(s0);
tex1[myMaterialID].Sample(samp[samplerID], texCoords);
```

使用索引（以上代码中的 `myMaterialID` 和 `samplerID`）时，一个重要的默认限制是不允许这些索引在绘制/调度调用中有变化。 即使是基于实例更改索引，也被视为有变化。

如果需要更改索引，请在索引中指定 `NonUniformResourceIndex` 限定符，例如：

``` syntax
tex1[NonUniformResourceIndex(myMaterialID)].Sample(samp[NonUniformResourceIndex(samplerID)], texCoords);
```

在某些硬件上，使用此限定符会生成额外的代码来强制正确性（包括跨线程），但对性能的影响非常微小。 如果在绘制/调度调用中更改索引且未指定此限定符，则结果是不确定的。

## <a name="descriptor-arrays-and-texture-arrays"></a>描述符数组和纹理数组

自 DirectX 10 以来一直可以使用纹理数组。 纹理数组需要一个描述符，但是，所有数组切片必须采用相同的格式、宽度、高度和 mip 计数。 此外，该数组必须占用虚拟地址空间中的一个连续范围。 以下代码演示了从着色器访问纹理数组的示例。

``` syntax
float3 myCoord(1.0f,1.4f,2.2f); // 2.2f is array index (rounded to int)
color = myTex2DArray.Sample(mySampler, myCoord);
```

在纹理数组中，索引可以任意变化，而无需指定 `NonUniformResourceIndex` 之类的限定符。

等效的描述符数组是：

``` syntax
Texture2D<float4> myTex2DArray[] : register(t0); // t0+
float2 myCoord(1.0f, 1.4f);
color = myTex2D[2].Sample(mySampler,myCoord); // 2 is index
```

请注意，对数组索引的隐晦式浮点用法已由 `myTex2D[2]` 取代。 此外，描述符数组在维度方面提供更大的灵活性。 此示例中的类型 `Texture2D` 不能有变化，但格式、宽度、高度和 mip 计数可随每个描述符一起变化。

使用纹理数组的描述符数组是合法的：

``` syntax
Texture2DArray<float4> myTex2DArrayOfArrays[2] : register(t0);
```

声明包含描述符的结构的数组是不合法的，例如，不支持以下代码。

``` syntax
struct myStruct {
    Texture2D                    a; 
    Texture2D                    b;
    ConstantBuffer<myConstants>  c;
};
myStruct foo[10000] : register(....);
```

这样就会允许内存布局 **abcabcabc...** ，但会造成语言限制，因此不受支持。 与此相关的一种受支持方法如下所示，不过，在此情况下，内存布局是 **aaa...bbb...ccc...** 。

``` syntax
Texture2D                     a[10000] : register(t0);
Texture2D                        b[10000] : register(t10000);
ConstantBuffer<myConstants>   c[10000] : register(b0);
```

若要实现 **abcabcabc...** 内存布局，请在不使用 `myStruct` 结构的情况下使用描述符表。

## <a name="resource-aliasing"></a>资源别名

在 HLSL 着色器中指定的资源范围是逻辑范围。 在运行时，会通过根签名机制将其绑定到具体的堆范围。 通常，逻辑范围将映射到不会与其他堆范围重叠的某个堆范围。 但是，使用根签名机制可能会产生兼容类型的堆范围的别名（重叠）。 例如，上述示例中的 `tex2` 和 `tex3` 范围可以映射到相同（或重叠）的堆范围，这会对 HLSL 程序中的别名纹理造成影响。 如果需要此类别名，必须使用 D3D10\_SHADER\_RESOURCES\_MAY\_ALIAS 选项编译着色器。该选项是使用[效应编译器工具](https://docs.microsoft.com/windows/desktop/direct3dtools/fxc) (FXC) 的 */res\_may\_alias* 选项设置的。 假设资源可以采用别名，该选项会防止特定的加载/存储优化，因此可让编译器生成正确的代码。

## <a name="divergence-and-derivatives"></a>差异和派生对象

SM5.1 不对资源索引施加限制；即 ` tex2[idx].Sample(…)` – 索引 idx 可以是文本常量、cbuffer 常量或内插的值。 尽管编程模型提供这种高灵活性，但仍需注意几个问题：

-   如果索引在 quad 中有差异，则硬件计算的派生对象和派生的数量（例如 LOD）可能是不确定的。 在此情况下，HLSL 编译器会尽最大努力发出警告，但不会阻止着色器进行编译。 此行为类似于差异控制流中的计算派生对象。
-   如果资源索引有差异，相比于索引统一的情况，性能将会下降，因为硬件需要针对多个资源执行操作。 必须在 HLSL 代码中使用 `NonUniformResourceIndex` 函数来标记可能会出现差异的资源索引。 否则结果是不确定的。

## <a name="uavs-in-pixel-shaders"></a>像素着色器中的 UAV

与 SM5.0 一样，SM5.1 不会对像素着色器中的 UAV 范围施加约束。

## <a name="constant-buffers"></a>常量缓冲区

与 SM5.0 相比，SM5.1 常量缓冲区 (cbuffer) 语法已发生更改，可让开发人员为常量缓冲区编制索引。 为了启用可编制索引的常量缓冲区，SM5.1 引入了 `ConstantBuffer`“模板”构造：

``` syntax
struct Foo
{
    float4 a;
    int2 b;
};
ConstantBuffer<Foo> myCB1[2][3] : register(b2, space1);
ConstantBuffer<Foo> myCB2 : register(b0, space1);
```

以上代码声明了类型为 `Foo` 且大小为 6 的常量缓冲区变量 `myCB1`，以及一个标量常量缓冲区变量 `myCB2`。 现在可按如下所示在着色器中为常量缓冲区变量编制索引：

``` syntax
myCB1[i][j].a.xyzw
myCB2.b.yy
```

字段“a”和“b”不会成为全局变量，必须将其视为字段。 为实现向后兼容，SM5.1 支持标量 cbuffers 的旧式 cbuffer 概念。 以下语句将“a”和“b”设置为类似于 SM5.0 中的全局只读变量。 但是，此类旧式 cbuffer 不可编制索引。

``` syntax
cbuffer : register(b1)
{
    float4 a;
    int2 b;
};
```

目前，着色器编译器仅支持对用户定义的结构使用 `ConstantBuffer` 模板。

出于兼容性原因，HLSL 编译器可能会自动为 `space0` 中声明的范围分配资源寄存器。 如果在 register 子句中省略“space”，将使用默认值 `space0`。 编译器使用“第一个孔适合”启发式算法来分配寄存器。 可以通过反射 API 检索分配。该 API 已经过扩展，添加了用于空间的 *Space* 字段，而 *BindPoint* 字段指示该资源寄存器范围的下限。

## <a name="bytecode-changes-in-sm51"></a>SM5.1 中的字节码更改

SM5.1 更改了在指令中声明和引用资源寄存器的方式。 该语法涉及到声明寄存器“变量”，类似于对组共享内存寄存器的声明方式：

``` syntax
Texture2D<float4> tex0          : register(t5,  space0);
Texture2D<float4> tex1[][5][3]  : register(t10, space0);
Texture2D<float4> tex2[8]       : register(t0,  space1);
SamplerState samp0              : register(s5, space0);

float4 main(float4 coord : COORD) : SV_TARGET
{
    float4 r = coord;
    r += tex0.Sample(samp0, r.xy);
    r += tex2[r.x].Sample(samp0, r.xy);
    r += tex1[r.x][r.y][r.z].Sample(samp0, r.xy);
    return r;
}
```

此声明可分解为：

``` syntax
// Resource Bindings:
//
// Name                                 Type  Format         Dim    ID   HLSL Bind     Count
// ------------------------------ ---------- ------- ----------- -----   --------- ---------
// samp0                             sampler      NA          NA     S0    a5            1
// tex0                              texture  float4          2d     T0    t5            1
// tex1[0][5][3]                     texture  float4          2d     T1   t10        unbounded
// tex2[8]                           texture  float4          2d     T2    t0.space1     8
//
//
//
// Input signature:
//
// Name                 Index   Mask Register SysValue  Format   Used
// -------------------- ----- ------ -------- -------- ------- ------
// COORD                    0   xyzw        0     NONE   float   xyzw
//
//
// Output signature:
//
// Name                 Index   Mask Register SysValue  Format   Used
// -------------------- ----- ------ -------- -------- ------- ------
// SV_TARGET                0   xyzw        0   TARGET   float   xyzw
//
ps_5_1
dcl_globalFlags refactoringAllowed
dcl_sampler s0[5:5], mode_default, space=0
dcl_resource_texture2d (float,float,float,float) t0[5:5], space=0
dcl_resource_texture2d (float,float,float,float) t1[10:*], space=0
dcl_resource_texture2d (float,float,float,float) t2[0:7], space=1
dcl_input_ps linear v0.xyzw
dcl_output o0.xyzw
dcl_temps 2
sample r0.xyzw, v0.xyxx, t0[0].xyzw, s0[5]
add r0.xyzw, r0.xyzw, v0.xyzw
ftou r1.x, r0.x
sample r1.xyzw, r0.xyxx, t2[r1.x + 0].xyzw, s0[5]
add r0.xyzw, r0.xyzw, r1.xyzw
ftou r1.xyz, r0.zyxz
imul null, r1.yz, r1.zzyz, l(0, 15, 3, 0)
iadd r1.y, r1.z, r1.y
iadd r1.x, r1.x, r1.y
sample r1.xyzw, r0.xyxx, t1[r1.x + 10].xyzw, s0[5]
add o0.xyzw, r0.xyzw, r1.xyzw
ret
// Approximately 12 instruction slots are used.
```

每个着色器资源范围现在包含一个对着色器字节码唯一的 ID（名称）。 例如，tex1 (t10) 纹理数组将成为着色器字节码中的“T1”。 将唯一 ID 分配到每个资源范围可以实现两个目的：

-   明确地确定要在指令（请参阅示例指令）中为哪个资源范围（请参阅 dcl\_resource\_texture2d）编制索引。
-   将一组属性（例如元素类型、步幅大小、光栅器工作模式等）附加到声明。

请注意，范围 ID 与 HLSL 下限声明无关。

反射资源绑定（在顶部列出）和着色器声明指令 (dcl\_\*) 的顺序是相同的，以帮助识别 HLSL 变量与字节码 ID 之间的对应关系。

SM5.1 中的每个声明指令使用 3D 操作数来定义范围 ID、下限和上限。 将发出一个附加的标记来指定寄存器空间。 还可能会发出其他标记来传递范围的其他属性，例如，cbuffer 或结构化缓冲区声明指令会发出 cbuffer 或结构的大小。 可以在 d3d12TokenizedProgramFormat.h 和 **D3D10ShaderBinary::CShaderCodeParser** 中找到确切的编码详细信息。

SM5.1 指令不会发出附加的资源操作数信息作为指令的一部分（与在 SM5.0 中一样）。 此信息现在会在声明指令中提供。 在 SM5.0 中，指令索引资源要求在扩展的操作码标记中描述资源属性，因为索引编制操作会模糊化与声明的关联。 在 SM5.1 中，每个 ID（例如“t1”）明确地与描述所需资源信息的单个声明相关联。 因此，不再发出在指令中用来描述资源信息的扩展操作码标记。

在非声明指令中，采样器、SRV 和 UAV 的资源操作数是一个 2D 操作数。 第一个索引是指定范围 ID 的文本常量。 第二个索引代表索引的线性化值。 该值是相对于对应寄存器空间的开头（而不是相对于逻辑范围的开头）计算的，以更好地与根签名相关联，并降低调整索引所造成的驱动程序编译器负担。

CBV 的资源操作数是一个 3D 操作数，其中包含：范围的文本 ID、常量缓冲区的索引，以及在常量缓冲区的特定实例中的偏移量。

## <a name="example-hlsl-declarations"></a>示例 HLSL 声明

HLSL 程序不需要知道有关根签名的任何信息。 它们可将绑定分配到虚拟“寄存器”绑定空间（t\# 表示 SRV，u\# 表示 UAV，b\# 表示 CBV，s\# 表示采样器），或依赖于编译器选取分配（以后可以使用着色器反射来查询生成的映射）。 根签名将描述符表、根描述符和根常量映射到此虚拟寄存器空间。

下面是 HLSL 着色器可以使用的一些示例声明。 请注意，不存在对根签名或描述符表的引用。

``` syntax
Texture2D foo[5] : register(t2);
Buffer bar : register(t7);
RWBuffer dataLog : register(u1);

Sampler samp : register(s0);

struct Data
{
    UINT index;
    float4 color;
};
ConstantBuffer<Data> myData : register(b0);

Texture2D terrain[] : register(t8); // Unbounded array
Texture2D misc[] : register(t0,space1); // Another unbounded array 
                                        // space1 avoids overlap with above t#

struct MoreData
{
    float4x4 xform;
};
ConstantBuffer<MoreData> myMoreData : register(b1);

struct Stuff
{
    float2 factor;
    UINT drawID;
};
ConstantBuffer<Stuff> myStuff[][3][8]  : register(b2, space3)
```

## <a name="related-topics"></a>相关主题

<dl> <dt>

[使用 HLSL 5.1 的动态索引](dynamic-indexing-using-hlsl-5-1.md)
</dt> <dt>

[效应编译器工具](https://docs.microsoft.com/windows/desktop/direct3dtools/fxc)
</dt> <dt>

[Direct3D 12 的 HLSL 着色器模型 5.1 功能](https://docs.microsoft.com/windows/desktop/direct3dhlsl/hlsl-shader-model-5-1-features-for-direct3d-12)
</dt> <dt>

[光栅器有序视图](rasterizer-order-views.md)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> <dt>

[着色器模型 5.1](https://docs.microsoft.com/windows/desktop/direct3dhlsl/shader-model-5-1)
</dt> <dt>

[着色器指定的模具参考值](shader-specified-stencil-reference-value.md)
</dt> <dt>

[在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)
</dt> </dl>

 

 




