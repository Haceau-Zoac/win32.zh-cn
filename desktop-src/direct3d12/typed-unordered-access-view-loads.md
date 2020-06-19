---
title: 类型化无序访问视图 (UAV) 加载
description: 无序访问视图 (UAV) 类型化加载是着色器通过特定 DXGI\_FORMAT 读取 UAV 的能力。
ms.assetid: 6106D15E-EAF6-4583-B4F2-7CC7EE30DE15
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 7b6b8e8bb193ab24211d7f412a829b2cf4a815b4
ms.sourcegitcommit: f0ca63c18dc52c357d3398af7be766d2bdd40be7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84977307"
---
# <a name="typed-unordered-access-view-uav-loads"></a>类型化无序访问视图 (UAV) 加载

无序访问视图（UAV）类型化负载是指着色器使用特定[**DXGI \_ 格式**](https://docs.microsoft.com/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format)从 UAV 读取的功能。

-   [概述](#overview)
-   [支持的格式和 API 调用](#supported-formats-and-api-calls)
-   [通过 HLSL 使用类型化 UAV 加载](#using-typed-uav-loads-from-hlsl)
-   [通过 HLSL 使用 UNORM 和 SNORM 类型化 UAV 加载](#using-unorm-and-snorm-typed-uav-loads-from-hlsl)
-   [相关主题](#related-topics)

## <a name="overview"></a>概述

无序访问视图 (UAV) 是无序访问资源的视图（可包括缓冲区、纹理和纹理数组，但无需多次采样）。 使用 UAV 可通过多个线程临时进行无序读/写访问。 这意味着此资源类型可由多个线程同时读/写，且不会产生内存冲突。 这种同时访问是通过使用 [Atomic Functions](https://docs.microsoft.com/windows/desktop/direct3d11/direct3d-11-advanced-stages-cs-atomic-functions)（原子函数）来进行的。

D3D12（和 D3D11.3）扩展了可用于类型化 UAV 加载的格式列表。

## <a name="supported-formats-and-api-calls"></a>支持的格式和 API 调用

之前，以下三种格式支持类型化 UAV 加载，并且是 D3D11.0 硬件的所需格式。 所有 D3D11.3 和 D3D12 硬件都支持以上格式。

-   R32\_FLOAT
-   R32\_UINT
-   R32\_SINT

D3D12 或 D3D11.3 硬件以集合形式支持以下格式，因此如果支持其中任一格式，则支持所有格式。

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

D3D12 和 D3D11.3 硬件可选择性支持或单独支持以下格式，因此需查询每种格式来测试是否支持。

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
-   R8G8 \_ 圣马丁
-   R16\_UNORM
-   R16\_SNORM
-   R8\_SNORM
-   A8\_UNORM
-   B5G6R5\_UNORM
-   B5G5R5A1\_UNORM
-   B4G4R4A4\_UNORM

若要确定对任何其他格式的支持，请将 [D3D12\_FEATURE\_DATA\_D3D12\_OPTIONS](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) 结构用作第一个参数（参阅[功能查询](capability-querying.md)）来调用 [CheckFeatureSupport](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)********。 如果支持上述“以集合形式支持”列表，则将设置 TypedUAVLoadAdditionalFormats 字段**。 再次调用 CheckFeatureSupport，并使用 [D3D12\_FEATURE\_DATA\_FORMAT\_SUPPORT](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_format_support) 结构（根据 [D3D12\_FORMAT\_SUPPORT2](/windows/desktop/api/d3d12/ne-d3d12-d3d12_format_support2) 枚举中的 D3D12\_FORMAT\_SUPPORT2\_UAV\_TYPED\_LOAD 成员检查返回的结构）来确定上述可选支持格式列表中的支持，例如************：

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

## <a name="using-typed-uav-loads-from-hlsl"></a>通过 HLSL 使用类型化 UAV 加载

对于类型化 UAV 而言，HLSL 标志为 D3D\_SHADER\_REQUIRES\_TYPED\_UAV\_LOAD\_ADDITIONAL\_FORMATS。

以下是用于处理类型化 UAV 加载的示例着色器代码：

``` syntax
RWTexture2D<float4> uav1;
uint2 coord;
float4 main() : SV_Target
{
  return uav1.Load(coord);
}
```

## <a name="using-unorm-and-snorm-typed-uav-loads-from-hlsl"></a>通过 HLSL 使用 UNORM 和 SNORM 类型化 UAV 加载

使用类型化 UAV 加载来读取 UNORM 或 SNORM 资源时，必须将 HLSL 对象的元素类型正确声明为 `unorm` 或 `snorm`。 如果将其指定为未定义行为，则 HLSL 中声明的元素类型与基础资源数据类型会不匹配。 例如，如果在 R8\_UNORM 数据的缓冲区资源上使用类型化 UAV 加载，则必须将元素类型声明为 `unorm float`：

``` syntax
RWBuffer<unorm float> uav;
```

## <a name="related-topics"></a>相关主题

<dl> <dt>

[渲染](rendering.md)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> <dt>

[HLSL 中的资源绑定](resource-binding-in-hlsl.md)
</dt> <dt>

[Shader Model 5.1](https://docs.microsoft.com/windows/desktop/direct3dhlsl/shader-model-5-1)（着色器模型 5.1）
</dt> <dt>

[在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)
</dt> </dl>

 

 




