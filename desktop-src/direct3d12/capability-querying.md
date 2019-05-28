---
title: 查询功能
description: 你的应用程序可以发现对资源绑定和许多其他功能，通过调用 ID3D12Device 的支持级别\:\:CheckFeatureSupport。
ms.assetid: ECBAF8EF-5D91-46D8-9D6E-A7FA4203B9F8
ms.topic: article
ms.date: 11/26/2018
ms.openlocfilehash: 5d966fcae1894e1d4662b8172e0d56a4747264a1
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223864"
---
# <a name="capability-querying"></a>查询功能

你的应用程序可以通过调用发现对资源绑定 （以及对很多其他功能的支持级别） 的支持级别[ **ID3D12Device::CheckFeatureSupport**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport)。

## <a name="how-to-query-for-the-resource-binding-tier"></a>如何查询的资源绑定层

第一个示例重点介绍资源绑定。 每个资源绑定层是较低层的功能中的一个超集，因此任何更高的层上的适用于给定的级别中的工作原理的代码保持不变。

资源绑定层是中的常量[ **D3D12_RESOURCE_BINDING_TIER** ](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_binding_tier)枚举。

若要查询的资源绑定层，请使用如下代码。 此代码示例演示了各种类型的功能支持的任何查询的常规模式。

```cppwinrt
D3D12_RESOURCE_BINDING_TIER get_resource_binding_tier(::ID3D12Device* pIDevice)
{
    D3D12_FEATURE_DATA_D3D12_OPTIONS featureSupport{};
    winrt::check_hresult(
        pIDevice->CheckFeatureSupport(D3D12_FEATURE_D3D12_OPTIONS, &featureSupport, sizeof(featureSupport))
    );

    switch (featureSupport.ResourceBindingTier)
    {
    case D3D12_RESOURCE_BINDING_TIER_1:
        // Tier 1 is supported.
        break;

    case D3D12_RESOURCE_BINDING_TIER_2:
        // Tiers 1 and 2 are supported.
        break;

    case D3D12_RESOURCE_BINDING_TIER_3:
        // Tiers 1, 2, and 3 are supported.
        break;
    }

    return featureSupport.ResourceBindingTier;
}
```

请注意任何枚举传递的常量 ([**D3D12_FEATURE_D3D12_OPTIONS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_feature)，在这种情况下) 有一个相应的数据结构，它接收有关该功能或功能集的信息 ([ **D3D12_FEATURE_DATA_D3D12_OPTIONS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options)，在这种情况下)。 始终将指针传递给匹配枚举的常数，它将传递的结构。

## <a name="how-to-query-for-any-feature-level"></a>如何查询的任何功能级别

资源绑定层，以及有许多其他功能，可以使用在上面的代码示例所示的相同模式查询其级别的支持。 只需传递从采用不同的常量[ **D3D12_FEATURE** ](/windows/desktop/api/d3d12/ne-d3d12-d3d12_feature)枚举[ **ID3D12Device::CheckFeatureSupport** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport) （来告诉该 API 的哪项功能请求的支持信息上），并将指针传递给 （要在其中接收请求的信息） 的匹配结构的实例。

- 传递**D3D12_FEATURE_ARCHITECTURE**并[ **D3D12_FEATURE_DATA_ARCHITECTURE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_architecture)。
- 传递**D3D12_FEATURE_ARCHITECTURE1**并[ **D3D12_FEATURE_DATA_ARCHITECTURE1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_architecture1)。
- 传递**D3D12_FEATURE_COMMAND_QUEUE_PRIORITY**并[ **D3D12_FEATURE_DATA_COMMAND_QUEUE_PRIORITY**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_command_queue_priority)。
- 传递**D3D12_FEATURE_CROSS_NODE**并[ **D3D12_FEATURE_DATA_CROSS_NODE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_cross_node)。
- 传递**D3D12_FEATURE_D3D12_OPTIONS**并[ **D3D12_FEATURE_DATA_D3D12_OPTIONS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options)。
- 传递**D3D12_FEATURE_D3D12_OPTIONS1**并[ **D3D12_FEATURE_DATA_D3D12_OPTIONS1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options1)。
- 传递**D3D12_FEATURE_D3D12_OPTIONS2**并[ **D3D12_FEATURE_DATA_D3D12_OPTIONS2**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options2)。
- 传递**D3D12_FEATURE_D3D12_OPTIONS3**并[ **D3D12_FEATURE_DATA_D3D12_OPTIONS3**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options3)。
- 传递**D3D12_FEATURE_D3D12_OPTIONS4**并[ **D3D12_FEATURE_DATA_D3D12_OPTIONS4**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options4)。
- 传递**D3D12_FEATURE_D3D12_OPTIONS5**并[ **D3D12_FEATURE_DATA_D3D12_OPTIONS5**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options5)。
- 传递**D3D12_FEATURE_EXISTING_HEAPS**并[ **D3D12_FEATURE_DATA_EXISTING_HEAPS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_existing_heaps)。
- 传递**D3D12_FEATURE_FEATURE_LEVELS**并[ **D3D12_FEATURE_DATA_FEATURE_LEVELS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_feature_levels)。
- 传递**D3D12_FEATURE_FORMAT_INFO**并[ **D3D12_FEATURE_DATA_FORMAT_INFO**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_format_info)。
- 传递**D3D12_FEATURE_FORMAT_SUPPORT**并[ **D3D12_FEATURE_DATA_FORMAT_SUPPORT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_format_support)。
- 传递**D3D12_FEATURE_GPU_VIRTUAL_ADDRESS_SUPPORT**并[ **D3D12_FEATURE_DATA_GPU_VIRTUAL_ADDRESS_SUPPORT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_gpu_virtual_address_support)。
- 传递**D3D12_FEATURE_MULTISAMPLE_QUALITY_LEVELS**并[ **D3D12_FEATURE_DATA_MULTISAMPLE_QUALITY_LEVELS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_multisample_quality_levels)。
- 传递**D3D12_FEATURE_PROTECTED_RESOURCE_SESSION_SUPPORT**并[ **D3D12_FEATURE_DATA_PROTECTED_RESOURCE_SESSION_SUPPORT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_protected_resource_session_support)。
- 传递**D3D12_FEATURE_ROOT_SIGNATURE**并[ **D3D12_FEATURE_DATA_ROOT_SIGNATURE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_root_signature)。
- 传递**D3D12_FEATURE_SERIALIZATION**并[ **D3D12_FEATURE_DATA_SERIALIZATION**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_serialization)。
- 传递**D3D12_FEATURE_SHADER_CACHE**并[ **D3D12_FEATURE_DATA_SHADER_CACHE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_shader_cache)。
- 传递**D3D12_FEATURE_SHADER_MODEL**并[ **D3D12_FEATURE_DATA_SHADER_MODEL**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_shader_model)。

## <a name="hardware-support-for-dxgi-formats"></a>DXGI 格式的硬件支持

若要查看的 DXGI 格式和硬件功能的表，请参阅这些主题。

- [对 Direct3D 功能级别 12.1 硬件的 DXGI 格式支持](https://msdn.microsoft.com/library/windows/desktop/mt426648)
- [对 Direct3D 功能级别 12.0 硬件的 DXGI 格式支持](https://msdn.microsoft.com/library/windows/desktop/mt426647)
- [对 Direct3D 功能级别 11.1 硬件的 DXGI 格式支持](https://msdn.microsoft.com/library/windows/desktop/mt427456)
- [对 Direct3D 功能级别 11.0 硬件的 DXGI 格式支持](https://msdn.microsoft.com/library/windows/desktop/mt427455)
- [硬件支持 Direct3D 10Level9 格式](https://msdn.microsoft.com/library/windows/desktop/ff471324)
- [硬件支持的 Direct3D 10.1 格式](https://msdn.microsoft.com/library/windows/desktop/cc627091)
- [硬件支持的 Direct3D 10 格式](https://msdn.microsoft.com/library/windows/desktop/cc627090)

## <a name="related-topics"></a>相关主题

* [在 Direct3D 12 资源绑定](resource-binding.md)
* [硬件功能级别](hardware-feature-levels.md)
* [根签名](root-signatures.md)
