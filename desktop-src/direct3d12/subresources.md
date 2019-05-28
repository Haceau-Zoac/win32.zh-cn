---
title: 子
description: 介绍如何将资源划分为子，以及如何引用一个、 多个或子的切片。
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
# <a name="subresources"></a>子

介绍如何将资源划分为子，以及如何引用一个、 多个或子的切片。

-   [示例子](#example-subresources)
    -   [子资源编制索引](#subresource-indexing)
    -   [Mip 切片](#mip-slice)
    -   [数组切片](#array-slice)
    -   [平面切片](#plane-slice)
    -   [多个子](#multiple-subresources)
-   [子资源 Api](#subresource-apis)
-   [相关的主题](#related-topics)

## <a name="example-subresources"></a>示例子

如果资源包含一个缓冲区，则它只需包含索引编号为 0 的一个子资源。 如果该资源包含纹理 （或纹理数组），然后引用子是更复杂。

某些 Api 访问整个资源 (如[ **ID3D12GraphicsCommandList::CopyResource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copyresource)方法)，其他人访问资源的一部分 (例如[ **ID3D12Resource::ReadFromSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource)方法)。 通常访问的资源部分的方法使用视图说明 (如[ **D3D12\_TEX2D\_数组\_SRV** ](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_srv)结构) 来指定若要访问的子。 请参阅[子资源 Api](#subresource-apis)部分有关的完整列表。

### <a name="subresource-indexing"></a>子资源编制索引

编制索引的特定子资源，请 mip 级别进行索引首先为每个数组项编制索引的索引。

![子资源编制索引](images/subresource-index.png)

### <a name="mip-slice"></a>Mip 切片

下图中所示，mip 切片数组中包括每个纹理的一个 mipmap 的级别。

![子资源 mip 切片](images/subresource-mip-slice.png)

### <a name="array-slice"></a>数组切片

给定纹理数组，具有 mipmap，数组切片的每个纹理包含一个纹理以及所有的 mip 级别，在下图中所示。

![子资源数组切片](images/subresource-array-slices.png)

### <a name="plane-slice"></a>平面切片

通常平面数据的格式不用于存储 RGBA 数据，但在其中它可能是 （24bpp RGB 数据） 的情况下，一个平面无法表示红色图像，一个绿色，一个蓝色的映像。 一个平面但不一定是一种颜色，两个或多个颜色无法合并为一个平面。 更常见的做法平面数据用于二次采样 YCbCr 和深度模具的数据。 深度模具是唯一完全支持 mipmap，数组的格式和多个平面 （通常平面 0 的深度和模具平面 1）。

为两个深度模具图像的数组编制索引子资源，每个都有三个 mip 级别，如下所示。

![深度模具编制索引](images/depth-stencil-indexing.png)

二次采样的 YCbCr 支持数组和具有平面，但不支持 mipmap。 YCbCr 映像具有两个平面，人眼是最易亮度 (Y) 和色度 （Cb 和 Cr，交错） 人眼即不易受到的影响。 此格式还支持色度值的压缩以压缩映像而不会影响亮度，并且出于此原因，是常见的视频的压缩格式，尽管它用于仍压缩映像。 下图显示了 NV12 格式，注意色度压缩到亮度，这意味着每个平面的宽度是相同的并且色度平面是亮度平面的半高分辨率的四分之一。 平面将作为子到上面的深度模具示例以相同方式编制索引。

![nv12 格式](images/ycbcr.png)

平面数据格式存在在 Direct3D 11 中，但单个平面无法不单独进行寻址，例如用于复制或映射操作。 这已更改在 Direct3D 12 中，以便每个平面接收到其自己的子资源 id。 比较以下两种方法来计算子资源 id。

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

大多数硬件假定后平面 N-1 一定会立即分配平面 N 的内存。

使用子的替代方法是应用程序无法分配每个平面完全不同的资源。 在这种情况下，应用程序可理解的数据是平面的并使用多个资源来表示它。

### <a name="multiple-subresources"></a>多个子

着色器资源视图中，可以选择子，使用其中一个查看结构中的字段的切片上文所述和明智地使用任何矩形区域 (如[ **D3D12\_TEX2D\_数组\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_srv))，如下图所示。

![多个子资源选择](images/subresource-multiple.png)

呈现目标视图只能使用一个单独的子资源或 mip 切片，并且不能包含多个 mip 切片从子。 也就是说，在呈现目标视图中的每个纹理必须大小相同。 着色器资源视图可以选择子，任何矩形区域，如图所示。

## <a name="subresource-apis"></a>子资源 Api

以下 Api 引用和使用子工作：

枚举：

-   [**D3D12\_TEXTURE\_COPY\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_texture_copy_type)

以下结构包含*PlaneSlice*索引，最包含*MipSlice*索引。

-   [**D3D12\_TEX2D\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_rtv)
-   [**D3D12\_TEX2D\_ARRAY\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_rtv)
-   [**D3D12\_TEX2D\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_srv)
-   [**D3D12\_TEX2D\_ARRAY\_SRV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_srv)
-   [**D3D12\_TEX2D\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_uav)
-   [**D3D12\_TEX2D\_ARRAY\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_array_uav)

以下结构包含*ArraySlice*索引，最包含*MipSlice*索引。

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

以下结构包含*MipSlice*的索引，但既不*ArraySlice*也不*PlaneSlice*索引。

-   [**D3D12\_TEX1D\_DSV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_dsv)
-   [**D3D12\_TEX2D\_DSV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex2d_dsv)
-   [**D3D12\_TEX1D\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_rtv)
-   [**D3D12\_TEX3D\_RTV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex3d_rtv)
-   [**D3D12\_TEX1D\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex1d_uav)
-   [**D3D12\_TEX3D\_UAV**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tex3d_uav)

以下结构还引用子：

-   [**D3D12\_放弃\_区域**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_discard_region) ： 准备对放弃资源中使用的结构。
-   [**D3D12\_PLACED\_SUBRESOURCE\_占用空间**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_placed_subresource_footprint) ： 将某一偏移量转换为资源添加到基本占用空间。
-   [**D3D12\_资源\_过渡\_屏障**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_resource_transition_barrier) ： 描述子不同用法 （着色器资源、 呈现器目标等） 之间的转换。
-   [**D3D12\_SUBRESOURCE\_数据**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_subresource_data) ： 子资源数据包括数据本身，以及行和切片间距。
-   [**D3D12\_SUBRESOURCE\_占用空间**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_subresource_footprint) ： 依旧包括格式、 宽度、 高度、 深度和行间距的子资源。
-   [**D3D12\_SUBRESOURCE\_信息**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_subresource_info) ： 包含偏移量、 行间距和深度的子资源的间距。
-   [**D3D12\_SUBRESOURCE\_平铺**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_subresource_tiling) ： 描述子资源平铺的卷 (请参阅[批量平铺资源](volume-tiled-resources.md))。
-   [**D3D12\_纹理\_副本\_位置**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_texture_copy_location) ： 介绍了用于复制的纹理的一部分。
-   [**D3D12\_平铺\_资源\_协调**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tiled_resource_coordinate) ： 描述平铺资源的坐标。

方法：

-   [**ID3D12Device::GetCopyableFootprints** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getcopyablefootprints) ： 获取资源，若要启用的副本进行的信息。
-   [**ID3D12Device::GetResourceTiling** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getresourcetiling) ： 获取有关如何将平铺的资源划分成磁贴的信息。
-   [**ID3D12GraphicsCommandList::ResolveSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvesubresource) ： 将多级采样的子资源复制到非多采样子资源。
-   [**ID3D12Resource::Map** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map) ： 将指针返回到在资源中，指定的数据和拒绝对子资源的 GPU 访问权限。
-   [**ID3D12Resource::ReadFromSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource) ： 将数据从子资源或子资源的矩形区域复制。
-   [**ID3D12Resource::Unmap** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap) ： 取消映射指定的内存范围并使对资源的指针。 恢复到的子资源的 GPU 访问权限。
-   [**ID3D12Resource::WriteToSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource) ： 将数据复制到子资源或子资源的矩形区域。

必须为纹理[ **D3D12\_资源\_状态\_常见**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)状态的 CPU 访问[ **WriteToSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource)并[ **ReadFromSubresource** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource)合法的; 但缓冲区不这样做。 对资源的 CPU 访问通常是通过[**地图**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)/[**Unmap**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap)。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 




