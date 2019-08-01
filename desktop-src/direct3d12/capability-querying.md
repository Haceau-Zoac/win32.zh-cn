---
title: 功能查询
description: 通过调用 ID3D12Device\:\:CheckFeatureSupport，应用程序可以发现对资源绑定和许多其他功能的支持级别。
ms.assetid: ECBAF8EF-5D91-46D8-9D6E-A7FA4203B9F8
ms.date: 11/26/2018
ms.localizationpriority: high
ms.topic: article
ms.openlocfilehash: 3764244bfe3b83e0df9670318585bd0091364e98
ms.sourcegitcommit: 27a9dfa3ef68240fbf09f1c64dff7b2232874ef4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66725452"
---
# <a name="capability-querying"></a>功能查询

通过调用 [ID3D12Device::CheckFeatureSupport](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)，应用程序可以发现对资源绑定的支持级别（以及对许多其他功能的支持级别）  。

## <a name="how-to-query-for-the-resource-binding-tier"></a>如何查询资源绑定层

第一个示例重点介绍资源绑定。 每个资源绑定层在功能上都是较低层的父集，因此给定层上正常运行的代码在任何较高层上也可正常运行。

资源绑定层是 [D3D12_RESOURCE_BINDING_TIER](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_binding_tier) 枚举中的常量  。

若要查询资源绑定层，请使用如下代码。 该代码示例演示了查询各种功能支持的一般模式。

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

注意，传递的任何枚举常量（本例中为 [D3D12_FEATURE_D3D12_OPTIONS](/windows/desktop/api/d3d12/ne-d3d12-d3d12_feature)）都具有相应数据结构，可接收关于该功能或该功能集合（本例中为 [D3D12_FEATURE_DATA_D3D12_OPTIONS](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options)）的信息   。 始终传递指向与所传递枚举常量匹配的结构的指针。

## <a name="how-to-query-for-any-feature-level"></a>如何查询任一功能级别

除资源绑定层之外，还有许多可使用上述代码示例中的同一模式查询其支持级别的其他功能。 只需将 [D3D12_FEATURE](/windows/desktop/api/d3d12/ne-d3d12-d3d12_feature) 枚举中的另一个常量传递到 [ID3D12Device::CheckFeatureSupport](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)（告诉 API 请求哪个功能的支持信息），并传递指向匹配结构实例（在其中接收请求的信息）的指针   。

- 传递 D3D12_FEATURE_ARCHITECTURE 和 [D3D12_FEATURE_DATA_ARCHITECTURE](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_architecture)   。
- 传递 D3D12_FEATURE_ARCHITECTURE1 和 [D3D12_FEATURE_DATA_ARCHITECTURE1](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_architecture1)   。
- 传递 D3D12_FEATURE_COMMAND_QUEUE_PRIORITY 和 [D3D12_FEATURE_DATA_COMMAND_QUEUE_PRIORITY](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_command_queue_priority)   。
- 传递 D3D12_FEATURE_CROSS_NODE 和 [D3D12_FEATURE_DATA_CROSS_NODE](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_cross_node)   。
- 传递 D3D12_FEATURE_D3D12_OPTIONS 和 [D3D12_FEATURE_DATA_D3D12_OPTIONS](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options)   。
- 传递 D3D12_FEATURE_D3D12_OPTIONS1 和 [D3D12_FEATURE_DATA_D3D12_OPTIONS1](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options1)   。
- 传递 D3D12_FEATURE_D3D12_OPTIONS2 和 [D3D12_FEATURE_DATA_D3D12_OPTIONS2](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options2)   。
- 传递 D3D12_FEATURE_D3D12_OPTIONS3 和 [D3D12_FEATURE_DATA_D3D12_OPTIONS3](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options3)   。
- 传递 D3D12_FEATURE_D3D12_OPTIONS4 和 [D3D12_FEATURE_DATA_D3D12_OPTIONS4](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options4)   。
- 传递 D3D12_FEATURE_D3D12_OPTIONS5 和 [D3D12_FEATURE_DATA_D3D12_OPTIONS5](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options5)   。
- 传递 D3D12_FEATURE_EXISTING_HEAPS 和 [D3D12_FEATURE_DATA_EXISTING_HEAPS](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_existing_heaps)   。
- 传递 D3D12_FEATURE_FEATURE_LEVELS 和 [D3D12_FEATURE_DATA_FEATURE_LEVELS](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_feature_levels)   。
- 传递 D3D12_FEATURE_FORMAT_INFO 和 [D3D12_FEATURE_DATA_FORMAT_INFO](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_format_info)   。
- 传递 D3D12_FEATURE_FORMAT_SUPPORT 和 [D3D12_FEATURE_DATA_FORMAT_SUPPORT](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_format_support)   。
- 传递 D3D12_FEATURE_GPU_VIRTUAL_ADDRESS_SUPPORT 和 [D3D12_FEATURE_DATA_GPU_VIRTUAL_ADDRESS_SUPPORT](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_gpu_virtual_address_support)   。
- 传递 D3D12_FEATURE_MULTISAMPLE_QUALITY_LEVELS 和 [D3D12_FEATURE_DATA_MULTISAMPLE_QUALITY_LEVELS](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_multisample_quality_levels)   。
- 传递 D3D12_FEATURE_PROTECTED_RESOURCE_SESSION_SUPPORT 和 [D3D12_FEATURE_DATA_PROTECTED_RESOURCE_SESSION_SUPPORT](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_protected_resource_session_support)   。
- 传递 D3D12_FEATURE_ROOT_SIGNATURE 和[D3D12_FEATURE_DATA_ROOT_SIGNATURE](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_root_signature)   。
- 传递 D3D12_FEATURE_SERIALIZATION 和[D3D12_FEATURE_DATA_SERIALIZATION](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_serialization)   。
- 传递 D3D12_FEATURE_SHADER_CACHE 和 [D3D12_FEATURE_DATA_SHADER_CACHE](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_shader_cache)   。
- 传递 D3D12_FEATURE_SHADER_MODEL 和 [D3D12_FEATURE_DATA_SHADER_MODEL](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_shader_model)   。

## <a name="hardware-support-for-dxgi-formats"></a>DXGI 格式的硬件支持

若要查看 DXGI 格式和硬件功能的表，请参阅以下主题。

- [DXGI Format Support for Direct3D Feature Level 12.1 Hardware](https://docs.microsoft.com/windows/desktop/direct3ddxgi/hardware-support-for-direct3d-12-1-formats)（Direct3D 功能级别 12.1 硬件的 DXGI 格式支持）
- [DXGI Format Support for Direct3D Feature Level 12.0 Hardware](https://docs.microsoft.com/windows/desktop/direct3ddxgi/hardware-support-for-direct3d-12-0-formats)（Direct3D 功能级别 12.0 硬件的 DXGI 格式支持）
- [DXGI Format Support for Direct3D Feature Level 11.1 Hardware](https://docs.microsoft.com/windows/desktop/direct3ddxgi/format-support-for-direct3d-11-1-feature-level-hardware)（Direct3D 功能级别 11.1 硬件的 DXGI 格式支持）
- [DXGI Format Support for Direct3D Feature Level 11.0 Hardware](https://docs.microsoft.com/windows/desktop/direct3ddxgi/format-support-for-direct3d-11-0-feature-level-hardware)（Direct3D 功能级别 11.0 硬件的 DXGI 格式支持）
- [Hardware Support for Direct3D 10Level9 Formats](https://docs.microsoft.com/previous-versions//ff471324(v=vs.85))（Direct3D 10Level9 格式的硬件支持）
- [Hardware Support for Direct3D 10.1 Formats](https://docs.microsoft.com/previous-versions//cc627091(v=vs.85))（Direct3D 10.1 格式的硬件支持）
- [Hardware Support for Direct3D 10 Formats](https://docs.microsoft.com/previous-versions//cc627090(v=vs.85))（Direct3D 10 格式的硬件支持）

## <a name="related-topics"></a>相关主题

* [Direct3D 12 中的资源绑定](resource-binding.md)
* [硬件功能级别](hardware-feature-levels.md)
* [根签名](root-signatures.md)
