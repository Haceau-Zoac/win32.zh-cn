---
title: 在 HLSL 中绑定的资源
description: 本部分介绍使用 Direct3D 12 的高级别着色语言 (HLSL) 着色器模型 5.1 的某些特定功能。
ms.assetid: 3CD4BDAD-8AE3-4DE0-B3F8-9C9F9E83BBE9
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 197f44aa8965168bd7aad63df2831ce2046adee7
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224179"
---
# <a name="resource-binding-in-hlsl"></a>在 HLSL 中绑定的资源

本部分介绍使用高级别着色语言 (HLSL) 的某些特定功能[着色器模型 5.1](https://msdn.microsoft.com/library/windows/desktop/dn933277) Direct3D 12。 所有 Direct3D 12 硬件支持着色器模型 5.1，因此支持此模型不依赖于硬件功能级别是什么。

-   [资源类型和数组](#resource-types-and-arrays)
-   [描述符数组和纹理数组](#descriptor-arrays-and-texture-arrays)
-   [资源别名](#resource-aliasing)
-   [分支和派生类](#divergence-and-derivatives)
-   [Uav 在像素着色器](#uavs-in-pixel-shaders)
-   [常量缓冲区](#constant-buffers)
-   [SM5.1 中的字节码更改](#bytecode-changes-in-sm51)
-   [示例 HLSL 声明](#example-hlsl-declarations)
-   [相关的主题](#related-topics)

## <a name="resource-types-and-arrays"></a>资源类型和数组

着色器模型 5 (SM5.0) 资源语法使用"注册"关键字中继到 HLSL 编译器资源有关的重要信息。 例如，以下语句声明数组槽 t3，t4，t5，和 t6 绑定的四个纹理。 t3 是在语句中，只是四个数组中的第一个出现的唯一注册槽。

``` syntax
Texture2D<float4> tex1[4] : register(t3)
```

在 HLSL 着色器模型 5.1 (SM5.1) 资源语法基于现有注册资源的语法，以允许更轻松地移植。 D3D12 资源在 HLSL 中的绑定到逻辑寄存器空间内的虚拟寄存器：

-   t – 用于着色器资源视图 (SRV)
-   s – 用于取样器
-   u – 无序的访问视图 (UAV)
-   b – 常量缓冲区视图 (CBV)

引用着色器根签名必须与声明的寄存器槽兼容。 例如，根签名的以下部分将兼容的纹理槽 t3 t6，通过使用如它具有通过 t98 槽 t0 的描述符表中所述。

``` syntax
DescriptorTable( CBV(b1), SRV(t0,numDescriptors=99), CBV(b2) )
```

资源声明可以是标量、 一维数组或多维数组：

``` syntax
Texture2D<float4> tex1     : register(t3,  space0)
Texture2D<float4> tex2[4]     : register(t10)
Texture2D<float4> tex3[7][5][3]   : register(t20, space1)
```

SM5.1 SM5.0 与使用相同的资源类型和元素类型。 声明限制是更灵活，现在，只能由运行时/硬件限制约束。 "空间"关键字指定向其逻辑寄存器声明的变量绑定到的空间。 如果省略"空间"关键字，则为 0 的默认空间索引隐式分配给范围 (因此`tex2`上述区域驻留在`space0`)。 `register(t3,  space0)` 将永远不会与冲突`register(t3,  space1)`或任何可能包括 t3 的另一个空间中的数组。

数组资源可能具有不受限制的大小，通过指定要为空或 0 的第一个维度已声明的：

``` syntax
Texture2D<float4> tex1[]       : register(t0)
```

可能是匹配的描述符表：

``` syntax
DescriptorTable( CBV(b1), UAV(u0, numDescriptors = 4), SRV(t0, numDescriptors=unbounded)
```

在 HLSL 中的不受限制的数组匹配一组固定数字与`numDescriptors`描述符中表，并在 HLSL 中的固定的大小不符描述符表中的不受限制的声明。

允许多维数组，其中包括的大小不受限制。 这些多维数组被转换为扁平寄存器空间中。

``` syntax
Texture2D<float4> tex3[3000][10] : register(t0,space1); // t0-t29999 in space1
Texture2D<float4> tex3[0][5][3]   : register(t5, space1)
```

不允许的资源范围的别名。 换而言之，对于每个资源类型 （t、 s、 u、 b），声明的寄存器范围不得重叠。 这也包括不受限制的范围。 在不同的寄存器空间中声明的范围永远不会重叠。 请注意该不受限制的 tex2 放在空间 0，而不受限制的 tex3 位于空间 1，以便它们不重叠。

访问已声明为数组的资源就像它们，例如索引一样简单：

``` syntax
Texture2D<float4> tex1[400] : register(t3);
sampler samp[7] : register(s0);
tex1[myMaterialID].Sample(samp[samplerID], texCoords);
```

使用重要的默认限制的索引 (`myMaterialID`和`samplerID`上面的代码中)，因为它们不能在绘制/调度调用方面有所不同。 即使只更改基于实例化过程中按不同的计数的索引。

如果需要不同的索引然后指定`NonUniformResourceIndex`限定符上的索引，例如：

``` syntax
tex1[NonUniformResourceIndex(myMaterialID)].Sample(samp[NonUniformResourceIndex(samplerID)], texCoords);
```

在某些硬件上使用此限定符生成附加代码以强制实施正确性 （包括跨线程），但在很小的性能成本。 如果索引已更改而无需此限定符，则结果不确定在绘制/调度。

## <a name="descriptor-arrays-and-texture-arrays"></a>描述符数组和纹理数组

纹理数组以来 DirectX 10 已可用。 纹理数组需要一个描述符，但所有数组切片必须都共享相同的格式、 宽度、 高度和 mip 计数。 此外，该数组必须占用虚拟地址空间中的连续范围。 下面的代码显示了从着色访问纹理数组的示例。

``` syntax
float3 myCoord(1.0f,1.4f,2.2f); // 2.2f is array index (rounded to int)
color = myTex2DArray.Sample(mySampler, myCoord);
```

在纹理数组中，索引可以变化公开发布，而无需限定符如`NonUniformResourceIndex`。

将等效描述符数组：

``` syntax
Texture2D<float4> myTex2DArray[] : register(t0); // t0+
float2 myCoord(1.0f, 1.4f);
color = myTex2D[2].Sample(mySampler,myCoord); // 2 is index
```

请注意，有些笨拙使用浮点数数组索引已被取代为与`myTex2D[2]`。 此外描述符数组提供更大的维度的灵活性。 类型，`Texture2D`是此示例中，不能有所变化，但所有随每个描述符格式、 宽度、 高度和 mip 计数可以。

它是合法的纹理数组描述符数组：

``` syntax
Texture2DArray<float4> myTex2DArrayOfArrays[2] : register(t0);
```

它不是合法声明结构的数组，每个结构，它包含描述符，例如不支持下面的代码。

``` syntax
struct myStruct {
    Texture2D                    a; 
    Texture2D                    b;
    ConstantBuffer<myConstants>  c;
};
myStruct foo[10000] : register(....);
```

这就允许的内存布局**abcabcabc...** ，但语言限制并不受支持。 执行此操作的一个受支持的方法是，如下所示，尽管这种情况下的内存布局**aaa...bbb...ccc...** .

``` syntax
Texture2D                     a[10000] : register(t0);
Texture2D                        b[10000] : register(t10000);
ConstantBuffer<myConstants>   c[10000] : register(b0);
```

若要实现**abcabcabc...** 内存布局中，使用描述符表而不使用的`myStruct`结构。

## <a name="resource-aliasing"></a>资源别名

HLSL 着色器中指定的资源范围是逻辑的范围。 它们是通过根签名机制绑定到在运行时的具体堆范围。 通常情况下，逻辑范围映射到使用其他堆范围不重叠的堆范围。 但是，根签名机制，可以到兼容的类型的别名 （重叠） 堆范围。 例如，`tex2`和`tex3`范围是从上面的示例可以映射到 HLSL 程序中具有别名的纹理的效果相同 （或重叠） 堆范围。 如果需要此类指定别名，则必须使用 D3D10 编译着色器\_着色器\_资源\_月\_别名选项，通过设置 */res\_可能\_别名*选项[影响编译器工具](https://msdn.microsoft.com/library/windows/desktop/bb232919)(FXC)。 选项使编译器通过防止在资源别名可能假设某些加载/存储优化生成正确的代码。

## <a name="divergence-and-derivatives"></a>分支和派生类

SM5.1 不会施加资源索引中; 的限制即，` tex2[idx].Sample(…)` – 索引 idx 可以是常量、 cbuffer 常量或插入的值。 虽然编程模型提供了很大的灵活性，有需要注意的问题：

-   如果索引与不同跨四核，硬件计算派生类和派生的数量，如 LOD 可能未定义。 HLSL 编译器尽最大努力来发出一条警告，在此情况下，但不是会阻止着色器编译。 此行为是类似于计算派生类中同名的不同控制流。
-   如果资源索引同名的不同，性能将降低比较到一个统一的索引，这种情况，因为硬件需要多个资源上执行操作。 必须使用标记可能处于不同的资源索引`NonUniformResourceIndex`HLSL 代码中的函数。 否则，结果不确定。

## <a name="uavs-in-pixel-shaders"></a>Uav 在像素着色器

SM5.1 不会施加约束中的像素着色器 UAV 范围为 SM5.0 是如此。

## <a name="constant-buffers"></a>常量缓冲区

SM5.1 常量缓冲区 (cbuffer) 语法已从 SM5.0 使开发人员能够索引常量缓冲区。 若要启用可编制索引的常量缓冲区，SM5.1 引入了`ConstantBuffer`"模板"构造：

``` syntax
struct Foo
{
    float4 a;
    int2 b;
};
ConstantBuffer<Foo> myCB1[2][3] : register(b2, space1);
ConstantBuffer<Foo> myCB2 : register(b0, space1);
```

前面的代码声明了常量缓冲区变量`myCB1`类型的`Foo`大小为 6 和标量、 常量缓冲区变量`myCB2`。 常量缓冲区变量现在可作为着色器对编制索引：

``` syntax
myCB1[i][j].a.xyzw
myCB2.b.yy
```

字段 a 和 b 是否会成为全局变量，但而是必须被视为字段。 为了向后兼容，SM5.1 支持标量 cbuffers 旧 cbuffer 概念。 使用以下语句使 a 和 b 全局，如下所示 SM5.0 只读变量。 但是，此类旧样式 cbuffer 不能为可编制索引。

``` syntax
cbuffer : register(b1)
{
    float4 a;
    int2 b;
};
```

目前，着色器编译器支持`ConstantBuffer`对于用户定义结构的模板。

出于兼容性原因，HLSL 编译器可能会自动分配资源的寄存器范围中声明为`space0`。 如果在注册子句，默认值中省略空间`space0`使用。 编译器使用第一个孔适合启发式方法来分配寄存器。 可以通过反射 API，它已扩展，添加检索赋值*空间*字段的空间，而*BindPoint*字段指示该资源注册范围的下限。

## <a name="bytecode-changes-in-sm51"></a>SM5.1 中的字节码更改

SM5.1 更改如何声明和说明中引用资源寄存器。 该语法涉及声明寄存器"变量"，则类似于如何对组进行共享的内存注册：

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

这将拆装到：

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

着色器资源的每个范围都是唯一的着色器字节码的 ID （名称）。 例如，tex1 (t10) 纹理数组可变为 T1 着色器字节码中。 唯一 Id 向每个资源范围允许两件事：

-   明确标识的资源范围 (请参阅 dcl\_资源\_texture2d) 在指令中编制索引 （请参阅示例指令）。
-   将一组属性附加到声明，例如，元素类型、 stride 大小、 光栅操作模式，等等。

请注意该范围的 ID 与 HLSL 下限声明无关。

反射资源绑定 （在顶部列出） 和着色器声明说明的顺序 (dcl\_\*) 相同，以辅助确定 HLSL 变量和字节码 Id 之间的对应关系。

每个声明指令 SM5.1 中的使用三维操作数定义： ID、 下限和上限的范围。 发出一个附加令牌，以便指定寄存器空间。 可能也发出其他令牌，以便传达的范围，例如，cbuffer 或指令发出 cbuffer 或结构的大小的结构化的缓冲区声明的其他属性。 编码的确切详细信息可在 d3d12TokenizedProgramFormat.h 并**D3D10ShaderBinary::CShaderCodeParser**。

SM5.1 说明将发出更多资源操作数信息 （如中所示 SM5.0) 指令的一部分。 此信息现在位于声明说明。 在 SM5.0，索引资源所需的资源属性的说明中所述扩展操作码的令牌，因为索引与声明关联的经过模糊处理。 在 SM5.1，每个 ID （例如 t1) 都与单个声明描述所需的资源信息的明确关联。 因此，不会再发出说明用于描述资源信息的扩展操作码令牌。

在非声明说明中，取样器、 SRVs 和 Uav 资源操作数是一个二维的操作数。 第一个索引是一个文本常量，它指定范围 id。 第二个索引表示索引的线性化的值。 为更好的关联具有根签名并减少驱动程序编译器负担调整索引的情况下，相对于相应的寄存器空间 （而不是相对于的逻辑范围的开头） 开头计算的值。

CBVs 资源操作数是一个三维的操作数，其中包含： 文本范围的偏移量常量缓冲区的特定实例的常量缓冲区索引的 ID。

## <a name="example-hlsl-declarations"></a>示例 HLSL 声明

HLSL 程序不需要知道有关根签名的任何信息。 他们可以分配绑定到虚拟"注册"绑定空间 t\# SRVs，为 u\# Uav，对于 b\# CBVs，为 s\#的取样器，或依赖于编译器选择分配 （并查询使用的生成映射着色器的反射之后）。 根签名映射到此虚拟寄存器空间描述符表、 根描述符和根常量。

以下是一些示例声明可能具有的 HLSL 着色器。 请注意，有没有对根签名或描述符表引用。

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

[动态索引使用 HLSL 5.1](dynamic-indexing-using-hlsl-5-1.md)
</dt> <dt>

[影响编译器工具](https://msdn.microsoft.com/library/windows/desktop/bb232919)
</dt> <dt>

[HLSL 着色器模型 5.1 功能对于 Direct3D 12](https://msdn.microsoft.com/library/windows/desktop/dn933267)
</dt> <dt>

[光栅器有序视图](rasterizer-order-views.md)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> <dt>

[着色器模型 5.1](https://msdn.microsoft.com/library/windows/desktop/dn933277)
</dt> <dt>

[着色器指定模具参考值](shader-specified-stencil-reference-value.md)
</dt> <dt>

[在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)
</dt> </dl>

 

 




