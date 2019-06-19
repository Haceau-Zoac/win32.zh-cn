---
title: 子资源
description: 介绍如何将资源划分为子资源，以及如何引用单个、多个或切片子资源。
ms.assetid: C4F92F8A-DBF0-412B-8707-CC4C1BF2D88F
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 4ddcec532b870802cbbc7d2beeb9a9c998f00a32
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224065"
---
# <a name="subresources"></a>子资源

介绍如何将资源划分为子资源，以及如何引用单个、多个或切片子资源。

-   [子资源示例](#example-subresources)
    -   [子资源索引](#subresource-indexing)
    -   [Mip 切片](#mip-slice)
    -   [数组切片](#array-slice)
    -   [平面切片](#plane-slice)
    -   [多个子资源](#multiple-subresources)
-   [子资源 API](#subresource-apis)
-   [相关主题](#related-topics)

## <a name="example-subresources"></a>子资源示例

如果某个资源包含缓冲区，则它仅包含索引编号为 0 的一个子资源。 如果该资源包含纹理（或纹理数组），则引用子资源更为复杂。

某些 API 会访问整个资源（例如，[**ID3D12GraphicsCommandList::CopyResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copyresource) 方法），而其他 API 会访问一部分资源（例如，[**ID3D12Resource::ReadFromSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource) 方法）。 访问一部分资源的方法通常使用视图描述（例如，[**D3D12\_TEX2D\_ARRAY\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_srv) 结构）来指定要访问的子资源。 有关完整列表，请参阅[子资源 API](#subresource-apis) 部分。

### <a name="subresource-indexing"></a>子资源索引

若要编制特定子资源的索引，请先编制 mip 级别的索引，因为已编制每个数组项的索引。

![子资源索引](images/subresource-index.png)

### <a name="mip-slice"></a>Mip 切片

Mip 切片包括数组中的每个纹理的一个 mipmap 级别，如下图所示。

![子资源 mip 切片](images/subresource-mip-slice.png)

### <a name="array-slice"></a>数组切片

假设一个纹理数组，每个纹理包含 mipmap，则一个数组切片将包括一个纹理及其所有 mip 级别，如下图所示。

![子资源数组切片](images/subresource-array-slices.png)

### <a name="plane-slice"></a>平面切片

通常，平面格式不用于存储 RGBA 数据，但在（可能是 24bpp RGB 数据）的情况下，一个平面可以表示红色图像，一个平面可以表示绿色图像，一个平面可以表示蓝色图像。 虽然一个平面不一定是一种颜色，但两种或更多颜色可以组合到一个平面中。 更为典型的是，平面数据用于子采样的 YCbCr 和深度模具数据。 深度模具是完全支持 mipmap、数组和多个平面的唯一格式（通常，平面 0 用于深度，而平面 1 用于模具）。

下面显示了包含两个深度模具图像的数组的子资源索引，每个都有三个 mip 级别。

![深度模具索引](images/depth-stencil-indexing.png)

子采样的 YCbCr 支持数组且具有平面，但不支持 mipmap。 YCbCr 图像具有两个平面，一个平面用于人眼最敏感的亮度 (Y)，一个平面用于人眼不太敏感的色度（Cb 和 Cr，交错）。 此格式支持压缩色度值，以便在不影响亮度的情况下压缩图像，并且是出于此原因的常见视频压缩格式（尽管它用于压缩静态映像）。 下图显示了 NV12 格式，注意色度已压缩为亮度分辨率的四分之一，这意味着每个平面的宽度是相同的，并且色度平面是亮度平面高度的一半。 平面将作为子资源按照相同的方式编入索引到上述深度模具示例中。

![nv12 格式](images/ycbcr.png)

平面格式存在于 Direct3D 11 中，但无法单独处理各个平面，例如复制或映射操作。 已在 Direct3D 12 中对此进行更改，以便每个平面收到自己的子资源 ID。 比较以下两种方法来计算子资源 ID。

Direct3D 11

``` syntax
inline UINT D3D11CalcSubresource( UINT MipSlice, UINT ArraySlice, UINT MipLevels )
{
    return MipSlice + (ArraySlice * MipLevels); 
}
```

Direct3D 12

``` syntax
inline UINT D3D12CalcSubresource( UINT MipSlice, UINT ArraySlice, UINT PlaneSlice, UINT MipLevels, UINT ArraySize )
{ 
    return MipSlice + (ArraySlice * MipLevels) + (PlaneSlice * MipLevels * ArraySize); 
}
```

大多数硬件假设始终在平面 N-1 后立即分配平面 N 的内存。

使用子资源的替代方法是应用可以在每个平面上分配完全不同的资源。 在这种情况下，应用程序了解到数据是平面的，并且使用多个资源来表示它。

### <a name="multiple-subresources"></a>多个子资源

着色器资源视图可以使用上述切片之一并谨慎使用视图结构（例如，[**D3D12\_TEX2D\_ARRAY\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_srv)）中的字段来选择子资源的任何矩形区域，如下图所示。

![多个子资源选择](images/subresource-multiple.png)

呈现器目标视图只能使用单个子资源或 mip 切片，并且不能包含多个 mip 切片中的子资源。 也就是说，呈现器目标视图中的每个纹理的大小必须相同。 着色器资源视图可以选择子资源的任何矩形区域，如下图所示。

## <a name="subresource-apis"></a>子资源 API

以下 API 引用并使用子资源：

枚举：

-   [**D3D12\_TEXTURE\_COPY\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_texture_copy_type)

以下结构包含 PlaneSlice  索引，大多数包含 MipSlice  索引。

-   [**D3D12\_TEX2D\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_rtv)
-   [**D3D12\_TEX2D\_ARRAY\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_rtv)
-   [**D3D12\_TEX2D\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_srv)
-   [**D3D12\_TEX2D\_ARRAY\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_srv)
-   [**D3D12\_TEX2D\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_uav)
-   [**D3D12\_TEX2D\_ARRAY\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_uav)

以下结构包含 ArraySlice  索引，大多数包含 MipSlice  索引。

-   [**D3D12\_TEX1D\_ARRAY\_DSV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_array_dsv)
-   [**D3D12\_TEX2D\_ARRAY\_DSV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_dsv)
-   [**D3D12\_TEX2DMS\_ARRAY\_DSV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2dms_array_dsv)
-   [**D3D12\_TEX1D\_ARRAY\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_array_rtv)
-   [**D3D12\_TEX2D\_ARRAY\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_rtv)
-   [**D3D12\_TEX2DMS\_ARRAY\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2dms_array_rtv)
-   [**D3D12\_TEX1D\_ARRAY\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_array_srv)
-   [**D3D12\_TEX2D\_ARRAY\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_srv)
-   [**D3D12\_TEX2DMS\_ARRAY\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2dms_array_srv)
-   [**D3D12\_TEX1D\_ARRAY\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_array_uav)
-   [**D3D12\_TEX2D\_ARRAY\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_uav)

以下结构包含 MipSlice  索引，但既不包含 ArraySlice  ，也不包含 PlaneSlice  索引。

-   [**D3D12\_TEX1D\_DSV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_dsv)
-   [**D3D12\_TEX2D\_DSV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_dsv)
-   [**D3D12\_TEX1D\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_rtv)
-   [**D3D12\_TEX3D\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex3d_rtv)
-   [**D3D12\_TEX1D\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_uav)
-   [**D3D12\_TEX3D\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex3d_uav)

以下结构也引用子资源：

-   [**D3D12\_DISCARD\_REGION**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_discard_region)：准备丢弃资源的结构。
-   [**D3D12\_PLACED\_SUBRESOURCE\_FOOTPRINT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_placed_subresource_footprint)：将进入资源的偏移量添加到基本占用情况。
-   [**D3D12\_RESOURCE\_TRANSITION\_BARRIER**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_resource_transition_barrier)：介绍不同用法（着色器资源、呈现器目标等）之间的子资源转换。
-   [**D3D12\_SUBRESOURCE\_DATA**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_subresource_data)：子资源数据包括数据本身，以及行和切片间距。
-   [**D3D12\_SUBRESOURCE\_FOOTPRINT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_subresource_footprint)：占用情况包含子资源的格式、宽度、高度、深度和行间距。
-   [**D3D12\_SUBRESOURCE\_INFO**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_subresource_info)：包含子资源的偏移量、行间距和深度间距。
-   [**D3D12\_SUBRESOURCE\_TILING**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_subresource_tiling)：介绍立体平铺子资源（请参阅[立体平铺资源](volume-tiled-resources.md)）。
-   [**D3D12\_TEXTURE\_COPY\_LOCATION**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_texture_copy_location)：介绍用于复制的纹理的一部分。
-   [**D3D12\_TILED\_RESOURCE\_COORDINATE**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tiled_resource_coordinate)：介绍平铺资源的坐标。

方法：

-   [**ID3D12Device::GetCopyableFootprints**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getcopyablefootprints)：获取有关资源的信息，以便启用复制操作。
-   [**ID3D12Device::GetResourceTiling**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getresourcetiling)：获取有关平铺资源如何分解为磁贴的信息。
-   [**ID3D12GraphicsCommandList::ResolveSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvesubresource)：将多采样的子资源复制到非多采样的子资源中。
-   [**ID3D12Resource::Map**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)：将指针返回到资源中的指定数据，并拒绝对子资源的 GPU 访问。
-   [**ID3D12Resource::ReadFromSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource)：从子资源或子资源的矩形区域复制数据。
-   [**ID3D12Resource::Unmap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap)：取消映射指定的内存范围并使指向资源的指针失效。 恢复对子资源的 GPU 访问。
-   [**ID3D12Resource::WriteToSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource)：将数据复制到子资源或子资源的矩形区域。

纹理必须处于 [**D3D12\_RESOURCE\_STATE\_COMMON**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states) 状态，才能使通过 [**WriteToSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource) 和 [**ReadFromSubresource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource) 的 CPU 访问合法；但缓冲区不是如此。 对资源的 CPU 访问通常是通过 [**Map**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)/[**Unmap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap) 完成的。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 




