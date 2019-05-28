---
title: 创建根签名
description: 根签名是一个包含嵌套的结构的复杂数据结构。
ms.assetid: 565B28C1-DBD1-42B6-87F9-70743E4A2E4A
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 9003856d610c2e0877ea3ecfb3dbc91aa06c504d
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223846"
---
# <a name="creating-a-root-signature"></a>创建根签名

根签名是一个包含嵌套的结构的复杂数据结构。 这些都可以使用以下数据结构定义 （其中包括方法，以帮助初始化成员） 以编程方式定义。 或者，他们可以编写在高级别着色语言 (HLSL) – 提供的一个优势在于，编译器将验证早期的布局与着色器兼容。

用于在序列化 （自包含，免费的指针） 中创建的根签名所需的 API 版本如下所述的布局说明。 用于生成此序列化的版本中提供一个方法C++数据结构，但获取序列化的根签名定义另一个方法是从根签名进行编译的着色器中检索它。

如果你想要利用的驱动程序的根签名描述符和数据的优化，请参阅[根签名版本 1.1](root-signature-version-1-1.md)

-   [描述符表绑定类型](#descriptor-table-bind-types)
-   [描述符范围](#descriptor-range)
-   [描述符表布局](#descriptor-table-layout)
-   [根常量](#root-constants)
-   [根描述符](#root-descriptor)
-   [着色器可见性](#shader-visibility)
-   [根签名定义](#root-signature-definition)
-   [根签名数据结构序列化 / 反序列化](https://docs.microsoft.com/windows)
-   [根签名创建 API](#root-signature-creation-api)
-   [管道状态对象中的根签名](#root-signature-in-pipeline-state-objects)
-   [用于定义版本 1.1 根签名的代码](#code-for-defining-a-version-11-root-signature)
-   [相关的主题](#related-topics)

## <a name="descriptor-table-bind-types"></a>描述符表绑定类型

枚举[ **D3D12\_描述符\_范围\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_descriptor_range_type)作为描述符表布局定义的一部分定义的描述符可以引用的类型。

它是一个范围，因此，例如，如果描述符的一部分表描述符表有 100 SRVs，可以在一个条目，而不是 100 中声明该范围。 因此描述符表定义是范围的集合。

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

[ **D3D12\_描述符\_范围**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_descriptor_range)结构定义一系列 （例如 SRVs) 描述符表中的给定类型的描述符。

`D3D12_DESCRIPTOR_RANGE_OFFSET_APPEND` \#定义通常可以用于`OffsetInDescriptorsFromTableStart`参数[ **D3D12\_描述符\_范围**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_descriptor_range)。 这意味着追加后描述符表中的前一个定义描述符范围。 如果应用程序希望向别名描述符或由于某种原因想要跳过槽时，它可以设置`OffsetInDescriptorsFromTableStart`到所需任意偏移量。 定义不同类型的重叠的范围无效。

着色器寄存器的组合所指定的一套`RangeType`， `NumDescriptors`， `BaseShaderRegister`，和`RegisterSpace`不能具有常见的根签名中的任何声明之间的重叠或冲突[ **D3D12\_着色器\_可见性**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_shader_visibility) （请参阅下面的着色器可见性部分）。

## <a name="descriptor-table-layout"></a>描述符表布局

[ **D3D12\_根\_描述符\_表**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_root_descriptor_table)结构作为一系列描述符范围出现一次是在声明的描述符表布局其他描述符堆中。 CBV/UAV/SRVs 与相同的描述符表中不允许取样器。

如果根签名槽类型设置为使用此结构`D3D12_ROOT_PARAMETER_TYPE_DESCRIPTOR_TABLE`。

若要设置的图形 (CBV，SRV UAV，采样器) 描述符表，请使用[ **ID3D12GraphicsCommandList::SetGraphicsRootDescriptorTable**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootdescriptortable)。

若要设置计算描述符表，请使用[ **ID3D12GraphicsCommandList::SetComputeRootDescriptorTable**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootdescriptortable)。

## <a name="root-constants"></a>根常量

[ **D3D12\_根\_常量**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_root_constants)结构声明在着色器中显示为一个常量缓冲区的根签名中的常量以内联方式。

如果根签名槽类型设置为使用此结构`D3D12_ROOT_PARAMETER_TYPE_32BIT_CONSTANTS`。

## <a name="root-descriptor"></a>根描述符

[ **D3D12\_根\_描述符**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_root_descriptor)结构声明描述符 （即显示在着色器） 中的根签名以内联方式。

如果根签名槽类型设置为使用此结构`D3D12_ROOT_PARAMETER_TYPE_CBV`，`D3D12_ROOT_PARAMETER_TYPE_SRV`或`D3D12_ROOT_PARAMETER_TYPE_UAV`。

## <a name="shader-visibility"></a>着色器可见性

成员[ **D3D12\_着色器\_可见性**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_shader_visibility)枚举设置到的着色器可见性参数[ **D3D12\_根\_参数**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_root_parameter)确定哪些着色器查看给定的根签名槽的内容。 始终计算使用\_所有 （因为只有一个活动的阶段）。 可以选择图形，但如果它使用\_所有，则所有着色器阶段，请参阅无论在根签名槽绑定。

着色器可见性的一种用途是帮助创作应为每个使用重叠的命名空间的着色器阶段的不同绑定的着色器。 例如，可能会声明顶点着色器：

 

Texture2D foo : register(t0);"

 

和像素着色器还可能会声明：

 

Texture2D bar : register(t0);

如果应用程序发出根签名绑定到 t0 可见性\_所有，这两个着色器，请参阅同一纹理。 如果着色器定义实际要将每个着色器以查看不同的纹理，它可以具有可见性定义 2 根签名槽\_顶点和\_像素。 无论什么可见性是根签名插槽上，它始终针对一个固定的最大根签名大小具有相同的成本 （仅根据 SlotType 是成本）。

在低端 D3D11 硬件上，着色器\_可见性还考虑到验证在根布局中，描述符表的大小，因为某些 D3D11 硬件仅支持绑定每个阶段的最大数量时使用的帐户。 这些限制仅施加低上运行时层硬件和根本不限制更现代硬件。

如果根签名具有多个描述符表定义的命名空间 （注册绑定到着色器） 中将相互重叠，并且其中任何一个指定\_的可见性，所有的布局是无效 （创建将失败）。

## <a name="root-signature-definition"></a>根签名定义

[ **D3D12\_根\_签名\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_root_signature_desc)结构可以包含描述符表和内联常量，每个插槽的类型由定义[**D3D12\_根\_参数**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_root_parameter)结构和枚举[ **D3D12\_根\_参数\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_root_parameter_type)。

若要启动的根签名槽，请参阅**SetComputeRoot\* \* \*** 并**SetGraphicsRoot\* \* \*** 的方法[ **ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)。

静态取样器使用在根签名中所述[ **D3D12\_静态\_采样器**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_static_sampler_desc)结构。

标志数限制到根签名的某些着色器的访问权限，请参阅[ **D3D12\_根\_签名\_标志**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_root_signature_flags)。

## <a name="root-signature-data-structure-serialization--deserialization"></a>根签名数据结构序列化 / 反序列化

在本部分中所述的方法由 D3D12Core.dll 导出，并提供用于序列化和反序列化根签名数据结构的方法。

序列化的形式就传递到 API 时创建的根签名。 如果是具有根签名 （当添加该功能） 中编写着色器，然后编译着色器将包含在其中的序列化的根签名已。

如果应用程序虽然产生[ **D3D12\_根\_签名\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_root_signature_desc)数据结构，它必须进行序列化的形式使用[ **D3D12SerializeRootSignature**](/windows/desktop/api/D3D12/nf-d3d12-d3d12serializerootsignature)。 该输出可以传递到[ **ID3D12Device::CreateRootSignature**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrootsignature)。

如果应用程序已序列化的根签名，或具有包含根签名，并且想要以编程方式发现 （称为"反射"），到 layout 定义的已编译着色器[ **D3D12CreateRootSignatureDeserializer** ](/windows/desktop/api/D3D12/nf-d3d12-d3d12createrootsignaturedeserializer)可以调用。 这将生成[ **ID3D12RootSignatureDeserializer** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12rootsignaturedeserializer)接口，它包含要返回的反序列化的方法[ **D3D12\_根\_签名\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_root_signature_desc)数据结构。 该接口拥有反序列化的数据结构的生存期。

## <a name="root-signature-creation-api"></a>根签名创建 API

[ **ID3D12Device::CreateRootSignature** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrootsignature) API 采用根签名的序列化版本。

## <a name="root-signature-in-pipeline-state-objects"></a>管道状态对象中的根签名

若要创建管道状态的方法 ([**ID3D12Device::CreateGraphicsPipelineState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-creategraphicspipelinestate)并[ **ID3D12Device::CreateComputePipelineState** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcomputepipelinestate) )使用可选[ **ID3D12RootSignature** ](https://msdn.microsoft.com/en-us/library/Dn788714(v=VS.85).aspx)作为输入参数的接口 (存储在[ **D3D12\_图形\_管道\_状态\_DESC** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_graphics_pipeline_state_desc)结构)。 这将覆盖已在着色器中的任何根签名。

如果根签名传递到创建管道状态方法之一，此根签名是验证中的 PSO 的兼容性的所有着色器，并提供给该驱动程序，以使用所有着色器。 如果任何着色器中包含不同的根签名，它在 API 中传递的根签名被替换。 如果根签名不传递中，所有传入的着色器必须具有根签名并且它们必须匹配 – 这将赋予该驱动程序。 设置 PSO 上命令列表或捆绑包不会更改根签名。 这由方法来实现[ **SetGraphicsRootSignature** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootsignature)并[ **SetComputeRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootsignature)。 Draw(graphics)/dispatch(compute) 调用时，，应用程序必须确保当前 PSO 与当前的根签名。否则，该行为不确定。

## <a name="code-for-defining-a-version-11-root-signature"></a>用于定义版本 1.1 根签名的代码

下面的示例演示如何创建具有以下格式的根签名：



|                        |                                                |                                              |
|------------------------|------------------------------------------------|----------------------------------------------|
| **RootParameterIndex** | **内容**                                   |                                              |
| \[0\]                  | 根常量: {b2}                         | (1 CBV)                                      |
| \[1\]                  | 描述符表: {t2 t7、 u0 u3}             | （6 SRVs + 4 Uav）                            |
| \[2\]                  | 根 CBV: {b0}                               | (1 CBV，静态数据)                         |
| \[3\]                  | 描述符表: {s0 s1}                    | （2 个取样器）                                 |
| \[4\]                  | 描述符表: {t8-无限}           | (不受限制\#SRVs，易失性描述符的) |
| \[5\]                  | 描述符表: {(t0，空间 1)-不受限制} | (不受限制\#SRVs，易失性描述符的) |
| \[6\]                  | 描述符表: {b1}                       | (1 CBV，静态数据)                         |



 

如果根签名的大多数部件使用大多数情况下它可能会比无需太过频繁切换根签名。 应用程序应该对大多数经常更改到最小的根签名中的条目进行排序。 应用更改时绑定到根签名的任何部分，该驱动程序可能需要制作一份部分或全部根签名状态，这可能会变得重要的成本，乘以跨多个状态更改时。

此外，根签名将定义静态取样器执行 anisotropic 纹理过滤在着色器寄存器 s3。

绑定此根签名后，描述符表根 CBV 和常量可以分配给\[0..6\]参数空间。 例如在每个根参数绑定描述符表 （描述符堆中的范围） \[1\]并\[3..6\]。

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

以下代码说明可以如何使用上述的根签名上图形命令列表。

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

 

 




