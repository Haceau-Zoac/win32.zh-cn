---
title: 类型化无序的访问视图 (UAV) 加载
description: 无序访问视图 (UAV) 类型的负载是从使用特定 DXGI UAV 读取的着色器的功能\_格式。
ms.assetid: 6106D15E-EAF6-4583-B4F2-7CC7EE30DE15
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 8b8bea53ca05ee3d1cbfcbf98bdc2905d8eea41d
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223921"
---
# <a name="typed-unordered-access-view-uav-loads"></a>类型化无序的访问视图 (UAV) 加载

无序访问视图 (UAV) 类型的负载是从具有特定 UAV 读取的着色器的功能[ **DXGI\_格式**](https://msdn.microsoft.com/library/windows/desktop/bb173059)。

-   [概述](#overview)
-   [支持的格式和 API 调用](#supported-formats-and-api-calls)
-   [使用类型化 HLSL 的 UAV 负载](#using-typed-uav-loads-from-hlsl)
-   [使用 UNORM 和 SNORM 类型从 HLSL UAV 加载](#using-unorm-and-snorm-typed-uav-loads-from-hlsl)
-   [相关的主题](#related-topics)

## <a name="overview"></a>概述

无序的访问视图 (UAV) 是一种无序的访问资源 （这包括缓冲区、 纹理和纹理数组，但无需多重采样） 的视图。 UAV 允许从多个线程未按时间顺序排序的读/写访问。 这意味着，此资源类型可以是读/写同时由多个线程而不会生成内存冲突。 使用处理此同时访问[原子函数](https://msdn.microsoft.com/library/windows/desktop/ff476334.aspx)。

D3D12 （和 D3D11.3） 扩展的格式的可与列表上键入 UAV 加载。

## <a name="supported-formats-and-api-calls"></a>支持的格式和 API 调用

以前，支持以下三种格式键入 UAV 加载和所需的 D3D11.0 硬件。 所有 D3D11.3 和 D3D12 硬件都支持它们。

-   R32\_FLOAT
-   R32\_UINT
-   R32\_SINT

以下格式支持作为一组在 D3D12 或 D3D11.3 硬件上，因此如果任何一个受支持，支持所有。

-   R32G32B32A32\_FLOAT
-   R32G32B32A32\_UINT
-   R32G32B32A32\_SINT
-   R16G16B16A16\_FLOAT
-   R16G16B16A16\_UINT
-   R16G16B16A16\_SINT
-   R8G8B8A8\_UNORM
-   R8G8B8A8\_UINT
-   R8G8B8A8\_SINT
-   R16\_FLOAT
-   R16\_UINT
-   R16\_SINT
-   R8\_UNORM
-   R8\_UINT
-   R8\_SINT

支持以下格式 （可选） 并分别在 D3D12 和 D3D11.3 硬件上，因此单个查询需要对每种格式来测试其支持。

-   R16G16B16A16\_UNORM
-   R16G16B16A16\_SNORM
-   R32G32\_FLOAT
-   R32G32\_UINT
-   R32G32\_SINT
-   R10G10B10A2\_UNORM
-   R10G10B10A2\_UINT
-   R11G11B10\_FLOAT
-   R8G8B8A8\_SNORM
-   R16G16\_FLOAT
-   R16G16\_UNORM
-   R16G16\_UINT
-   R16G16\_SNORM
-   R16G16\_SINT
-   R8G8\_UNORM
-   R8G8\_UINT
-   R8G8\_SNORM
-   8G8\_SINT
-   R16\_UNORM
-   R16\_SNORM
-   R8\_SNORM
-   A8\_UNORM
-   B5G6R5\_UNORM
-   B5G5R5A1\_UNORM
-   B4G4R4A4\_UNORM

若要确定对任何其他格式的支持，请调用[ **CheckFeatureSupport** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport)与[ **D3D12\_功能\_数据\_D3D12\_选项**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_d3d12_options)结构的第一个参数 (请参阅[功能查询](capability-querying.md))。 *TypedUAVLoadAdditionalFormats*支持上面的"支持作为一组"列表时，将设置字段。 第二个调用**CheckFeatureSupport**，并使用[ **D3D12\_功能\_数据\_格式\_支持**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_format_support)结构 (检查返回的结构对 D3D12\_格式\_SUPPORT2\_UAV\_类型化\_负载隶属[ **D3D12\_格式化\_SUPPORT2** ](/windows/desktop/api/D3D12/ne-d3d12-d3d12_format_support2)枚举) 来确定可以选择支持的格式，如以上所列的列表中的支持：

``` syntax
D3D12_FEATURE_DATA_D3D12_OPTIONS FeatureData;
ZeroMemory(&FeatureData, sizeof(FeatureData));
HRESULT hr = pDevice->CheckFeatureSupport(D3D12_FEATURE_D3D12_OPTIONS, &FeatureData, sizeof(FeatureData));
if (SUCCEEDED(hr))
{
    // TypedUAVLoadAdditionalFormats contains a Boolean that tells you whether the feature is supported or not
    if (FeatureData.TypedUAVLoadAdditionalFormats)
    {
        // Can assume “all-or-nothing” subset is supported (e.g. R32G32B32A32_FLOAT)
        // Cannot assume other formats are supported, so we check:
        D3D12_FEATURE_DATA_FORMAT_SUPPORT FormatSupport = {DXGI_FORMAT_R32G32_FLOAT, D3D12_FORMAT_SUPPORT1_NONE, D3D12_FORMAT_SUPPORT2_NONE};
        hr = pDevice->CheckFeatureSupport(D3D12_FEATURE_FORMAT_SUPPORT, &FormatSupport, sizeof(FormatSupport));
        if (SUCCEEDED(hr) && (FormatSupport.Support2 & D3D12_FORMAT_SUPPORT2_UAV_TYPED_LOAD) != 0)
        {
            // DXGI_FORMAT_R32G32_FLOAT supports UAV Typed Load!
        }
    }
}
```

## <a name="using-typed-uav-loads-from-hlsl"></a>使用类型化 HLSL 的 UAV 负载

对于类型化 Uav HLSL 标志是 D3D\_着色器\_REQUIRES\_类型化\_UAV\_负载\_其他\_格式。

下面是示例着色器代码来处理类型化的 UAV 负载：

``` syntax
RWTexture2D<float4> uav1;
uint2 coord;
float4 main() : SV_Target
{
  return uav1.Load(coord);
}
```

## <a name="using-unorm-and-snorm-typed-uav-loads-from-hlsl"></a>使用 UNORM 和 SNORM 类型从 HLSL UAV 加载

使用类型化的 UAV 加载读取从 UNORM 或 SNORM 资源，必须正确声明的 HLSL 对象的元素类型`unorm`或`snorm`。 它指定为未定义的行为与基础资源的数据类型在 HLSL 中声明的元素类型不匹配。 例如，如果您具有 R8 的缓冲区资源上使用类型化的 UAV 加载\_UNORM 数据，则必须声明为元素类型`unorm float`:

``` syntax
RWBuffer<unorm float> uav;
```

## <a name="related-topics"></a>相关主题

<dl> <dt>

[呈现](rendering.md)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> <dt>

[在 HLSL 中绑定的资源](resource-binding-in-hlsl.md)
</dt> <dt>

[着色器模型 5.1](https://msdn.microsoft.com/library/windows/desktop/dn933277)
</dt> <dt>

[在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)
</dt> </dl>

 

 




