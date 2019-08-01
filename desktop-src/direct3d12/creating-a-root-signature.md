---
title: 创建根签名
description: 根签名是包含嵌套结构的复杂数据结构。
ms.assetid: 565B28C1-DBD1-42B6-87F9-70743E4A2E4A
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 9a119b8e2da86d431c193828d9255221baea05b4
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296028"
---
# <a name="creating-a-root-signature"></a>创建根签名

根签名是包含嵌套结构的复杂数据结构。 这些结构可以使用以下数据结构定义（包括帮助初始化成员的方法）以编程方式定义。 或者，可以使用高级着色语言 (HLSL) 编写它们，这样编译器可以尽早验证布局是否与着色器兼容。

用于创建根签名的 API 采用下面所述布局描述的序列化（独立无指针）版本。 提供了一个方法用于从 C++ 数据结构生成此序列化版本，获取序列化根签名定义的另一种方式是从使用根签名编译的着色器中检索该定义。

若要对根签名描述符和数据利用驱动程序优化，请参阅[根签名版本 1.1](root-signature-version-1-1.md)

-   [描述符表绑定类型](#descriptor-table-bind-types)
-   [描述符范围](#descriptor-range)
-   [描述符表布局](#descriptor-table-layout)
-   [根常量](#root-constants)
-   [根描述符](#root-descriptor)
-   [着色器可见性](#shader-visibility)
-   [根签名定义](#root-signature-definition)
-   [根签名数据结构序列化/反序列化](/windows)
-   [根签名创建 API](#root-signature-creation-api)
-   [管道状态对象中的根签名](#root-signature-in-pipeline-state-objects)
-   [用于定义版本 1.1 根签名的代码](#code-for-defining-a-version-11-root-signature)
-   [相关主题](#related-topics)

## <a name="descriptor-table-bind-types"></a>描述符表绑定类型

枚举 [**D3D12\_DESCRIPTOR\_RANGE\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_descriptor_range_type) 定义可作为描述符表布局定义的一部分引用的描述符类型。

它是一个范围。举例而言，如果描述符表的某个部分包含 100 个 SRV，则可以在一个而不是 100 个条目中声明该范围。 因此，描述符表定义是范围的集合。

``` syntax
typedef enum D3D12_DESCRIPTOR_RANGE_TYPE
{
  D3D12_DESCRIPTOR_RANGE_TYPE_SRV,
  D3D12_DESCRIPTOR_RANGE_TYPE_UAV,
  D3D12_DESCRIPTOR_RANGE_TYPE_CBV,
  D3D12_DESCRIPTOR_RANGE_TYPE_SAMPLER
} D3D12_DESCRIPTOR_RANGE_TYPE;
```

## <a name="descriptor-range"></a>描述符范围

[**D3D12\_DESCRIPTOR\_RANGE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_descriptor_range) 结构定义描述符表中给定类型（例如 SRV）的描述符的范围。

通常可对 [**D3D12\_DESCRIPTOR\_RANGE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_descriptor_range) 的 `OffsetInDescriptorsFromTableStart` 参数使用 `D3D12_DESCRIPTOR_RANGE_OFFSET_APPEND` \# 定义。 这意味着会在描述符表中前一个范围的后面定义描述符范围。 如果应用程序需要别名描述符，或出于某种原因需要跳过槽，可将 `OffsetInDescriptorsFromTableStart` 设置为所需的任意偏移量。 定义不同类型的重叠范围是无效的操作。

由 `RangeType`、`NumDescriptors`、`BaseShaderRegister` 和 `RegisterSpace` 的组合指定的着色器寄存器集不能冲突或者在具有通用 [**D3D12\_SHADER\_VISIBILITY**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shader_visibility) 的根签名中的任何声明之间重叠（请参阅下面的“着色器可见性”部分）。

## <a name="descriptor-table-layout"></a>描述符表布局

[**D3D12\_ROOT\_DESCRIPTOR\_TABLE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_descriptor_table) 结构将描述符表的布局声明为在描述符堆中一个接一个地显示的描述符范围集合。 CBV/UAV/SRV 所在的同一个描述符表中不允许采样器。

将根签名槽类型设置为 `D3D12_ROOT_PARAMETER_TYPE_DESCRIPTOR_TABLE` 时，将使用此结构。

若要设置图形（CBV、SRV、UAV、采样器）描述符表，请使用 [**ID3D12GraphicsCommandList::SetGraphicsRootDescriptorTable**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootdescriptortable)。

若要设置计算描述符表，请使用 [**ID3D12GraphicsCommandList::SetComputeRootDescriptorTable**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootdescriptortable)。

## <a name="root-constants"></a>根常量

[**D3D12\_ROOT\_CONSTANTS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_constants) 结构将着色器中显示的根签名中的内联常量声明为一个常量缓冲区。

将根签名槽类型设置为 `D3D12_ROOT_PARAMETER_TYPE_32BIT_CONSTANTS` 时，将使用此结构。

## <a name="root-descriptor"></a>根描述符

[**D3D12\_ROOT\_DESCRIPTOR**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_descriptor) 结构声明根签名中内联的描述符（显示在着色器中）。

将根签名槽类型设置为 `D3D12_ROOT_PARAMETER_TYPE_CBV`、`D3D12_ROOT_PARAMETER_TYPE_SRV` 或 `D3D12_ROOT_PARAMETER_TYPE_UAV` 时，将使用此结构。

## <a name="shader-visibility"></a>着色器可见性

在 [**D3D12\_ROOT\_PARAMETER**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_parameter) 的着色器可见性参数中设置的 [**D3D12\_SHADER\_VISIBILITY**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_shader_visibility) 枚举成员确定哪些着色器能够看到给定根签名槽的内容。 计算始终使用 \_ALL（因为只有一个活动的阶段）。 图形可以选择 \_ALL，但如果使用它，所有着色器阶段会看到在根签名槽中绑定的内容。

着色器可见性的一种用途是帮助创建的着色器使用重叠的命名空间要求对每个着色器阶段使用不同的绑定。 例如，顶点着色器可以声明：

 

Texture2D foo : register(t0);"

 

像素着色器还可以声明：

 

Texture2D bar : register(t0);

如果应用程序与 t0 VISIBILITY\_ALL 建立了根签名绑定，则这两个着色器会看到相同的纹理。 如果着色器定义实际上需要每个着色器看到不同的纹理，可以使用 VISIBILITY\_VERTEX and \_PIXEL 定义 2 个根签名槽。 无论对根签名槽的可见性是什么，处理一个固定最大根签名大小的开销都是相同的（开销仅取决于 SlotType）。

在低端 D3D11 硬件上，验证根布局中描述符表的大小时，还会考虑使用 SHADER\_VISIBILITY，因为某些 D3D11 硬件可能仅支持每个阶段的最大绑定数量。 仅当在低层硬件上运行且根本不限制新式硬件时，才会施加这些限制。

如果在命名空间（着色器的注册绑定）中为根签名定义了多个相互重叠的描述符表，并且其中任何一个描述符表为可见性指定了 \_ALL，则布局是无效的（创建将会失败）。

## <a name="root-signature-definition"></a>根签名定义

[**D3D12\_ROOT\_SIGNATURE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_signature_desc) 结构可以包含描述符表和内联常量，每个插槽类型由 [**D3D12\_ROOT\_PARAMETER**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_parameter) 结构和 [**D3D12\_ROOT\_PARAMETER\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_root_parameter_type) 枚举定义。

若要启动根签名槽，请参阅 [**ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 的 **SetComputeRoot\*\*\*** 和 **SetGraphicsRoot\*\*\*** 方法。

使用 [**D3D12\_STATIC\_SAMPLER**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_static_sampler_desc) 结构在根签名中描述静态采样器。

标志数目限制特定着色器对根签名的访问，具体请参阅 [**D3D12\_ROOT\_SIGNATURE\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_root_signature_flags)。

## <a name="root-signature-data-structure-serialization--deserialization"></a>根签名数据结构序列化/反序列化

本部分所述的方法由 D3D12Core.dll 导出，提供用于序列化和反序列化根签名数据结构的方法。

序列化格式是创建根签名时传入 API 的格式。 如果编写的着色器包含根签名（添加该功能时），则编译的着色器已包含序列化的根签名。

如果应用程序遵循过程生成 [**D3D12\_ROOT\_SIGNATURE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_signature_desc) 数据结构，它必须使用 [**D3D12SerializeRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-d3d12serializerootsignature) 生成序列化格式。 它的输出可传递到 [**ID3D12Device::CreateRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createrootsignature)。

如果应用程序已包含序列化根签名或具有包含根签名的已编译着色器，并希望以编程方式发现布局定义（称为“反射”），则可以调用 [**D3D12CreateRootSignatureDeserializer**](/windows/desktop/api/d3d12/nf-d3d12-d3d12createrootsignaturedeserializer)。 这会生成 [**ID3D12RootSignatureDeserializer**](/windows/desktop/api/d3d12/nn-d3d12-id3d12rootsignaturedeserializer) 接口，其中包含用于返回反序列化 [**D3D12\_ROOT\_SIGNATURE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_signature_desc) 数据结构的方法。 该接口拥有反序列化数据结构的生存期。

## <a name="root-signature-creation-api"></a>根签名创建 API

[**ID3D12Device::CreateRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createrootsignature) API 采用根签名的序列化版本。

## <a name="root-signature-in-pipeline-state-objects"></a>管道状态对象中的根签名

用于创建管道状态（[**ID3D12Device::CreateGraphicsPipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-creategraphicspipelinestate) 和 [**ID3D12Device::CreateComputePipelineState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcomputepipelinestate)）的方法采用可选的 [**ID3D12RootSignature**](https://msdn.microsoft.com/library/Dn788714(v=VS.85).aspx) 接口作为输入参数（存储在 [**D3D12\_GRAPHICS\_PIPELINE\_STATE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_graphics_pipeline_state_desc) 结构中）。 这会重写着色器中已存在的任何根签名。

如果将根签名传入某个创建管道状态方法，将会根据 PSO 中的所有着色器验证此根签名以确认是否兼容，并提供给驱动程序以用于所有着色器。 如果任何着色器包含不同的根签名，该签名将被 API 中传入的根签名替换。 如果未传入根签名，所有传入的着色器必须包含一个根签名，并且它们必须匹配 – 此签名将提供给驱动程序。 在命令列表或捆绑中设置 PSO 不会更改根签名。 可以通过 [**SetGraphicsRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootsignature) 和 [**SetComputeRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootsignature) 方法实现此目的。 调用 draw(graphics)/dispatch(compute) 时，应用程序必须确保当前 PSO 与当前根签名匹配，否则行为是不确定的。

## <a name="code-for-defining-a-version-11-root-signature"></a>用于定义版本 1.1 根签名的代码

以下示例演示如何创建采用以下格式的根签名：



|                        |                                                |                                              |
|------------------------|------------------------------------------------|----------------------------------------------|
| **RootParameterIndex** | **内容**                                   |                                              |
| \[0\]                  | 根常量：{ b2 }                         | （1 个 CBV）                                      |
| \[1\]                  | 描述符表：{ t2-t7, u0-u3 }             | （6 个 SRV + 4 个 UAV）                            |
| \[2\]                  | 根 CBV：{ b0 }                               | （1 个 CBV，静态数据）                         |
| \[3\]                  | 描述符表：{ s0-s1 }                    | （2 个采样器）                                 |
| \[4\]                  | 描述符表：{ t8 - unbounded }           | （无限的 SRV \#，易失性描述符） |
| \[5\]                  | 描述符表：{ (t0, space1) - unbounded } | （无限的 SRV \#，易失性描述符） |
| \[6\]                  | 描述符表：{ b1 }                       | （1 个 CBV，静态数据）                         |



 

如果在大部分时间内都会使用根签名的大多数组成部分，则可能比过于频繁地切换根签名要好。 应用程序应该按更改频率从高到低的顺序将根签名中的条目排序。 当应用更改根签名任何组成部分的绑定时，驱动程序可能需要创建部分或全部根签名状态的副本，如果状态更改次数成倍增加，这可能会造成不菲的开销。

此外，根签名会定义一个在着色器寄存器 s3 中执行各向异性纹理过滤的静态采样器。

绑定此根签名后，可将描述符表、根 CBV 和常量分配到 \[0..6\] 参数空间。 例如，可在在每个根参数 \[1\] 和 \[3..6\] 中绑定描述符表（描述符堆中的范围）。

``` syntax
CD3DX12_DESCRIPTOR_RANGE1 DescRange[6];

DescRange[0].Init(D3D12_DESCRIPTOR_RANGE_SRV,6,2); // t2-t7
DescRange[1].Init(D3D12_DESCRIPTOR_RANGE_UAV,4,0); // u0-u3
DescRange[2].Init(D3D12_DESCRIPTOR_RANGE_SAMPLER,2,0); // s0-s1
DescRange[3].Init(D3D12_DESCRIPTOR_RANGE_SRV,-1,8, 0,
                  D3D12_DESCRIPTOR_RANGE_FLAG_DESCRIPTORS_VOLATILE); // t8-unbounded
DescRange[4].Init(D3D12_DESCRIPTOR_RANGE_SRV,-1,0,1,
                  D3D12_DESCRIPTOR_RANGE_FLAG_DESCRIPTORS_VOLATILE); 
                                                            // (t0,space1)-unbounded
DescRange[5].Init(D3D12_DESCRIPTOR_RANGE_CBV,1,1,
                  D3D12_DESCRIPTOR_RANGE_FLAG_DATA_STATIC); // b1

CD3DX12_ROOT_PARAMETER1 RP[7];

RP[0].InitAsConstants(3,2); // 3 constants at b2
RP[1].InitAsDescriptorTable(2,&DescRange[0]); // 2 ranges t2-t7 and u0-u3
RP[2].InitAsConstantBufferView(0, 0, 
                               D3D12_ROOT_DESCRIPTOR_FLAG_DATA_STATIC); // b0
RP[3].InitAsDescriptorTable(1,&DescRange[2]); // s0-s1
RP[4].InitAsDescriptorTable(1,&DescRange[3]); // t8-unbounded
RP[5].InitAsDescriptorTable(1,&DescRange[4]); // (t0,space1)-unbounded
RP[6].InitAsDescriptorTable(1,&DescRange[5]); // b1

CD3DX12_STATIC_SAMPLER StaticSamplers[1];
StaticSamplers[0].Init(3, D3D12_FILTER_ANISOTROPIC); // s3
CD3DX12_VERSIONED_ROOT_SIGNATURE_DESC RootSig(7,RP,1,StaticSamplers);
ID3DBlob* pSerializedRootSig;
CheckHR(D3D12SerializeVersionedRootSignature(&RootSig,pSerializedRootSig)); 

ID3D12RootSignature* pRootSignature;
hr = CheckHR(pDevice->CreateRootSignature(
    pSerializedRootSig->GetBufferPointer(),pSerializedRootSig->GetBufferSize(),
    __uuidof(ID3D12RootSignature),
    &pRootSignature));
```

以下代码演示如何在图形命令列表中使用上述根签名。

``` syntax
InitializeMyDescriptorHeapContentsAheadOfTime(); // for simplicity of the 
                                                 // example
CreatePipelineStatesAhreadOfTime(pRootSignature); // The root signature is passed into 
                                     // shader / pipeline state creation
...

ID3D12DescriptorHeap* pHeaps[2] = {pCommonHeap, pSamplerHeap};
pGraphicsCommandList->SetDescriptorHeaps(pHeaps,2);
pGraphicsCommandList->SetGraphicsRootSignature(pRootSignature);
pGraphicsCommandList->SetGraphicsRootDescriptorTable(
                        6,heapOffsetForMoreData,DescRange[5].NumDescriptors);
pGraphicsCommandList->SetGraphicsRootDescriptorTable(5,heapOffsetForMisc,5000); 
pGraphicsCommandList->SetGraphicsRootDescriptorTable(4,heapOffsetForTerrain,20000);
pGraphicsCommandList->SetGraphicsRootDescriptorTable(
                        3,heapOffsetForSamplers,DescRange[2].NumDescriptors);
pGraphicsCommandList->SetComputeRootConstantBufferView(2,pDynamicCBHeap,&CBVDesc);

MY_PER_DRAW_STUFF stuff;
InitMyPerDrawStuff(&stuff);
pGraphicsCommandList->SetSetGraphicsRoot32BitConstants(
                        0,&stuff,0,RTSlot[0].Constants.Num32BitValues);

SetMyRTVAndOtherMiscBindings();

for(UINT i = 0; i < numObjects; i++)
{
    pGraphicsCommandList->SetPipelineState(PSO[i]);
    pGraphicsCommandList->SetGraphicsRootDescriptorTable(
                    1,heapOffsetForFooAndBar[i],DescRange[1].NumDescriptors);
    pGraphicsCommandList->SetGraphicsRoot32BitConstant(0,&i,1,drawIDOffset);
    SetMyIndexBuffers(i);
    pGraphicsCommandList->DrawIndexedInstanced(...);
}
```

## <a name="related-topics"></a>相关主题

<dl> <dt>

[根签名](root-signatures.md)
</dt> <dt>

[在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)
</dt> <dt>

[使用根签名](using-a-root-signature.md)
</dt> </dl>

 

 




