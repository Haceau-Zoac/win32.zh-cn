---
title: 平铺的卷资源
description: 卷 (3D) 纹理可以用作平铺资源，注意磁贴解析为三维。
ms.assetid: F670D15D-BC0F-4F90-99C1-A35192FE8980
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: d3f84f50a6463763098fbc94b6dbad560b8b03d2
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223801"
---
# <a name="volume-tiled-resources"></a>平铺的卷资源

卷 (3D) 纹理可以用作平铺资源，注意磁贴解析为三维。

-   [概述](#overview)
-   [平铺的资源 Api](#tiled-resource-apis)
-   [相关的主题](#related-topics)

## <a name="overview"></a>概述

平铺的资源将从其备份的 D3D 资源对象中分离出来 （过去中的资源必须与它们支持的内存的 1 对 1 关系） 的内存。 这允许各种有趣的情况下，如纹理数据中的流式处理和重复使用或减少内存使用量。

D3D11.2 支持 2D 纹理平铺资源。 D3D12 和 D3D11.3 提供了对三维平铺纹理的可选支持 (请参阅[ **D3D12\_平铺\_资源\_层**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_tiled_resources_tier))。

使用平铺中的典型资源维度是 2D 纹理的 4 × 4 磁贴，3D 纹理的 4 x 4 x 4 平铺。



| 位/像素 （1 示例/像素） | 磁贴维度 （像素，w x h x d） |
|-----------------------------|-------------------------------------|
| 8                           | 64x32x32                            |
| 16                          | 32x32x32                            |
| 32                          | 32x32x16                            |
| 64                          | 32x16x16                            |
| 128                         | 16x16x16                            |
| BC 1,4                      | 128x64x16                           |
| BC 2,3,5,6,7                | 64x64x16                            |

请注意使用平铺资源不支持以下格式：96bpp 格式，视频格式 R1\_UNORM、 R8G8\_B8G8\_UNORM、 R8R8\_G8B8\_UNORM。

下面的深灰色的关系图中表示 NULL 磁贴。

-   [纹理三维平铺资源默认映射 (最详细 mip)](#texture-3d-tiled-resource-default-mapping-most-detailed-mip)
-   [纹理三维平铺资源默认映射 （第二个最详细的 mip）](#texture-3d-tiled-resource-default-mapping-second-most-detailed-mip)
-   [纹理三维平铺资源 (最详细 mip)](#texture-3d-tiled-resource-most-detailed-mip)
-   [纹理三维平铺资源 （第二个最详细的 mip）](#texture-3d-tiled-resource-second-most-detailed-mip)
-   [纹理三维平铺的资源 （单个磁贴）](#texture-3d-tiled-resource-single-tile)
-   [纹理三维平铺的资源 （统一框）](#texture-3d-tiled-resource-uniform-box)

### <a name="texture-3d-tiled-resource-default-mapping-most-detailed-mip"></a>纹理三维平铺资源默认映射 (最详细 mip)

![平铺的 3 个维资源的默认映射](images/vtr-tex3d-default-1.png)

### <a name="texture-3d-tiled-resource-default-mapping-second-most-detailed-mip"></a>纹理三维平铺资源默认映射 （第二个最详细的 mip）

![显示第二个最详细的 mip](images/vtr-tex3d-default-2.png)

### <a name="texture-3d-tiled-resource-most-detailed-mip"></a>纹理三维平铺资源 (最详细 mip)

下面的代码设置的最详细的 mip 三维平铺资源。

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

![大多数详述 mip 的三个三维纹理](images/vtr-tex3d-default-1b.png)

### <a name="texture-3d-tiled-resource-second-most-detailed-mip"></a>纹理三维平铺资源 （第二个最详细的 mip）

下面的代码将三维平铺资源设置，第二个最详细的 mip:

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

![第二个最详细的三个 mip 三维纹理](images/vtr-tex3d-default-2b.png)

### <a name="texture-3d-tiled-resource-single-tile"></a>纹理三维平铺的资源 （单个磁贴）

下面的代码设置了单个磁贴资源：

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

![一个平铺的三个维的资源](images/vtr-tex3d-single.png)

### <a name="texture-3d-tiled-resource-uniform-box"></a>纹理三维平铺的资源 （统一框）

下面的代码设置平铺的统一框资源 （请注意该语句 `trSize.bUseBox = true;) :`

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

![一个统一的框](images/vtr-tex3d-uniform.png)

## <a name="tiled-resource-apis"></a>平铺的资源 Api

对于 2D 和 3D 平铺资源使用相同的 API 调用：

枚举

-   [**D3D12\_平铺\_资源\_层**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_tiled_resources_tier) ： 确定平铺的资源支持的级别。
-   [**D3D12\_格式\_SUPPORT2** ](/windows/desktop/api/D3D12/ne-d3d12-d3d12_format_support2) ： 用于测试的平铺的资源支持。
-   [**D3D12\_MULTISAMPLE\_质量\_级别\_标志**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_multisample_quality_level_flags) ： 确定多采样资源中的平铺的资源支持。
-   [**D3D12\_磁贴\_副本\_标志**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_tile_copy_flags) ： 包含用于从 swizzled 平铺的资源和线性缓冲区复制到和的标志。

结构

-   [**D3D12\_平铺\_资源\_协调**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tiled_resource_coordinate) ： 保存 x、 y 和 z 坐标和子资源引用。 请注意，有一个帮助器结构：[**CD3DX12\_平铺\_资源\_协调**](cd3dx12-tiled-resource-coordinate.md)。
-   [**D3D12\_磁贴\_区域\_大小**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tile_region_size) ： 指定的大小和数量的平铺区域磁贴。
-   [**D3D12\_磁贴\_形状**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_tile_shape) ： 磁贴形状作为宽度、 高度和纹素中的深度。
-   [**D3D12\_功能\_数据\_D3D12\_选项**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_feature_data_d3d12_options) ： 包含受支持的磁贴资源层级别和一个布尔值， *VolumeTiledResourcesSupported*，指示是否支持批量平铺资源。

方法

-   [**ID3D12Device::CheckFeatureSupport** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport) ： 用于确定什么功能，以及在哪个层支持当前的硬件。
-   [**ID3D12GraphcisCommandList::CopyTiles** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytiles) ： 将磁贴从缓冲区复制到平铺资源，反之亦然。
-   [**ID3D12CommandQueue::UpdateTileMappings** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings) ： 磁贴中的位置平铺资源映射更新到资源堆中的内存位置。
-   [**ID3D12CommandQueue::CopyTileMappings** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-copytilemappings) ： 将映射从平铺的源资源复制到平铺的目标资源。
-   [**ID3D12CommandQueue::GetResourceTiling** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getresourcetiling) ： 获取有关如何将平铺的资源划分成磁贴的信息。

## <a name="related-topics"></a>相关主题
* [资源绑定](resource-binding.md)
