---
title: 卷磁贴资源（Direct3D 12）
description: 立体 (3D) 纹理可以用作平铺资源，请注意，平铺分辨率是三维的。
ms.assetid: F670D15D-BC0F-4F90-99C1-A35192FE8980
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: e3eac806ad29151cd49c894e47846c7bebd048a6
ms.sourcegitcommit: cba7f424a292fd7f3a8518947b9466439b455419
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/23/2019
ms.locfileid: "74418227"
---
# <a name="volume-tiled-resources-direct3d-12"></a>卷磁贴资源（Direct3D 12）

立体 (3D) 纹理可以用作平铺资源，请注意，平铺分辨率是三维的。

-   [概述](#overview)
-   [平铺资源 API](#tiled-resource-apis)
-   [相关主题](#related-topics)

## <a name="overview"></a>概述

平铺资源将 D3D 资源对象与其后备内存分离（过去的资源与其后备内存存在一对一的关系）。 这可实现各种有趣的场景，例如纹理数据中的流式传输以及重用或减少内存使用。

D3D11.2 支持 2D 纹理平铺资源。 D3D12 和 D3D11.3 可选择性地支持 3D 平铺纹理（请参阅 [D3D12\_TILED\_RESOURCES\_TIER](/windows/desktop/api/d3d12/ne-d3d12-d3d12_tiled_resources_tier)）。

平铺中使用的典型资源尺寸是 4 x 4 个图块（针对 2D 纹理）和 4 x 4 x 4 个图块（针对 3D 纹理）。



| 位数/像素（每像素 1 个示例） | 磁贴尺寸（像素，宽 x 高 x 深） |
|-----------------------------|-------------------------------------|
| 8                           | 64x32x32                            |
| 16                          | 32x32x32                            |
| 32                          | 32x32x16                            |
| 64                          | 32x16x16                            |
| 128                         | 16x16x16                            |
| BC 1、4                      | 128x64x16                           |
| BC 2、3、5、6、7                | 64x64x16                            |

注意平铺资源不支持以下格式：96bpp 格式、视频格式、R1\_UNORM、R8G8\_B8G8\_UNORM、R8R8\_G8B8\_UNORM。

在下面的图表中，深灰色表示 NULL 平铺。

-   [纹理 3D 平铺资源默认映射（最详细的 MIP）](#texture-3d-tiled-resource-default-mapping-most-detailed-mip)
-   [纹理 3D 平铺资源默认映射（第二详细的 MIP）](#texture-3d-tiled-resource-default-mapping-second-most-detailed-mip)
-   [纹理 3D 平铺资源（最详细的 MIP）](#texture-3d-tiled-resource-most-detailed-mip)
-   [纹理 3D 平铺资源（第二详细的 MIP）](#texture-3d-tiled-resource-second-most-detailed-mip)
-   [纹理 3D 平铺资源（单平铺）](#texture-3d-tiled-resource-single-tile)
-   [纹理 3D 平铺资源（统一框）](#texture-3d-tiled-resource-uniform-box)

### <a name="texture-3d-tiled-resource-default-mapping-most-detailed-mip"></a>纹理 3D 平铺资源默认映射（最详细的 MIP）

![平铺三维资源的默认映射](images/vtr-tex3d-default-1.png)

### <a name="texture-3d-tiled-resource-default-mapping-second-most-detailed-mip"></a>纹理 3D 平铺资源默认映射（第二详细的 MIP）

![显示第二详细的 MIP](images/vtr-tex3d-default-2.png)

### <a name="texture-3d-tiled-resource-most-detailed-mip"></a>纹理 3D 平铺资源（最详细的 MIP）

下面的代码会在最详细的 MIP 上设置 3D 平铺资源。

``` syntax
D3D12_TILED_RESOURCE_COORDINATE trCoord;
trCoord.X = 1;
trCoord.Y = 0;
trCoord.Z = 0;
trCoord.Subresource = 0;

D3D12_TILE_REGION_SIZE trSize;
trSize.bUseBox = false;
trSize.NumTiles = 63;
```

![三维纹理的最详细的 MIP](images/vtr-tex3d-default-1b.png)

### <a name="texture-3d-tiled-resource-second-most-detailed-mip"></a>纹理 3D 平铺资源（第二详细的 MIP）

下面的代码会设置一个 3D 平铺资源和第二详细的 MIP：

``` syntax
D3D12_TILED_RESOURCE_COORDINATE trCoord;
trCoord.X = 1;
trCoord.Y = 0;
trCoord.Z = 0;
trCoord.Subresource = 1;

D3D12_TILE_REGION_SIZE trSize;
trSize.bUseBox = false;
trSize.NumTiles = 6;
```

![三维纹理的第二详细的 MIP](images/vtr-tex3d-default-2b.png)

### <a name="texture-3d-tiled-resource-single-tile"></a>纹理 3D 平铺资源（单平铺）

下面的代码会设置单平铺资源：

``` syntax
D3D12_TILED_RESOURCE_COORDINATE trCoord;
trCoord.X = 1;
trCoord.Y = 1;
trCoord.Z = 1;
trCoord.Subresource = 0;

D3D12_TILE_REGION_SIZE trSize;
trSize.bUseBox = true;
trSize.NumTiles = 27;
trSize.Width = 3;
trSize.Height = 3;
trSize.Depth = 3;
```

![单个平铺三维资源](images/vtr-tex3d-single.png)

### <a name="texture-3d-tiled-resource-uniform-box"></a>纹理 3D 平铺资源（统一框）

以下代码会设置统一框平铺资源（请注意语句 `trSize.bUseBox = true;) :`

``` syntax
D3D12_TILED_RESOURCE_COORDINATE trCoord;
trCoord.X = 0;
trCoord.Y = 1;
trCoord.Z = 0;
trCoord.Subresource = 0;

D3D12_TILE_REGION_SIZE trSize;
trSize.bUseBox = true;
trSize.NumTiles = 27;
trSize.Width = 3;
trSize.Height = 3;
trSize.Depth = 3;
```

![统一框](images/vtr-tex3d-uniform.png)

## <a name="tiled-resource-apis"></a>平铺资源 API

对 2D 和 3D 平铺资源使用相同的 API 调用：

枚举

-   [**D3D12\_TILED\_RESOURCES\_TIER**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_tiled_resources_tier)：确定平铺资源支持的级别。
-   [**D3D12\_FORMAT\_SUPPORT2**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_format_support2)：用于测试平铺资源支持。
-   [**D3D12\_MULTISAMPLE\_QUALITY\_LEVEL\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_multisample_quality_level_flags)：确定多采样资源中的平铺资源支持。
-   [**D3D12\_TILE\_COPY\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_tile_copy_flags)：保存用于在重排平铺资源和线性缓冲区之间复制的标志。

结构

-   [**D3D12\_TILED\_RESOURCE\_COORDINATE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tiled_resource_coordinate)：保存x、y 和 z 坐标以及子资源引用。 请注意，有一个帮助器结构： [**CD3DX12\_平铺\_资源\_坐标**](cd3dx12-tiled-resource-coordinate.md)。
-   [**D3D12\_TILE\_REGION\_SIZE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tile_region_size)：指定平铺区域的平铺大小和数量。
-   [**D3D12\_TILE\_SHAPE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_tile_shape)：平铺的宽度、高度和深度（以纹素为单位）。
-   [**D3D12\_FEATURE\_DATA\_D3D12\_OPTIONS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options)：包含支持的平铺资源级别和布尔值 VolumeTiledResourcesSupported，该值指示是否支持立体平铺资源。

方法

-   [**ID3D12Device::CheckFeatureSupport**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport)：用于确定当前硬件在哪些层支持哪些功能。
-   [**ID3D12GraphcisCommandList::CopyTiles**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytiles)：将平铺从缓冲区复制到平铺资源，反之亦然。
-   [**ID3D12CommandQueue::UpdateTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings)：将平铺资源中的平铺位置映射更新到资源堆中的内存位置。
-   [**ID3D12CommandQueue::CopyTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-copytilemappings)：将映射从源平铺资源复制到目标平铺资源。
-   [**ID3D12CommandQueue::GetResourceTiling**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getresourcetiling)：获取有关平铺资源如何分解为平铺的信息。

## <a name="related-topics"></a>相关主题
* [资源绑定](resource-binding.md)
