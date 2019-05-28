---
title: 将通过缓冲区的纹理数据上传
description: 上传 2D 或 3D 纹理数据类似于上传一维数据只是应用程序需要更密切地关注与行间距相关的数据对齐方式。
ms.assetid: 22A25A94-A45C-482D-853A-FA6860EE7E4E
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 2c02ace68728dc56b8b4753550d5a093e0ecf970
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224311"
---
# <a name="uploading-texture-data-through-buffers"></a>将通过缓冲区的纹理数据上传

上传 2D 或 3D 纹理数据类似于上传一维数据只是应用程序需要更密切地关注与行间距相关的数据对齐方式。 缓冲区可以用于以正交方式同时从图形管道的多个部分，非常灵活。

-   [将通过缓冲区的纹理数据上传](#upload-texture-data-via-buffers)
-   [复制](#copying)
-   [映射和取消映射](#mapping-and-unmapping)
-   [缓冲区对齐方式](#buffer-alignment)
-   [相关的主题](#related-topics)

## <a name="upload-texture-data-via-buffers"></a>将通过缓冲区的纹理数据上传

应用程序必须将通过数据上传[ **ID3D12GraphicsCommandList::CopyTextureRegion** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)或[ **ID3D12GraphicsCommandList::CopyBufferRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion). 纹理数据是更有可能会更大，重复，访问并从中获益的非线性内存比其他资源的数据布局改进缓存一致性。 D3D12 中使用的缓冲区时, 应用程序具有完全控制数据放置和排列方式，只要满足的内存对齐需求与将资源数据，复制相关联。

其中应用程序只需将平展 2D 数据到 1d 之前将其放在缓冲区中示例突出显示。 对于 mipmap 2D 方案，应用程序可以具有离散平展每个子资源，快速使用一个一维的二次分配算法，或者，使用更复杂的 2D 二次分配技术以最小化视频内存使用率。 第一项技术被预计使用频率会更简单。 打包到磁盘上或在网络上的数据时，第二种方法可能很有用。 在任一情况下，应用程序仍必须为每个子资源调用复制 Api。

``` syntax
// Prepare a pBitmap in memory, with bitmapWidth, bitmapHeight, and pixel format of DXGI_FORMAT_B8G8R8A8_UNORM. 
//
// Sub-allocate from the buffer for texture data.
//

D3D12_SUBRESOURCE_FOOTPRINT pitchedDesc = { 0 };
pitchedDesc.Format = DXGI_FORMAT_B8G8R8A8_UNORM;
pitchedDesc.Width = bitmapWidth;
pitchedDesc.Height = bitmapHeight;
pitchedDesc.Depth = 1;
pitchedDesc.RowPitch = Align(bitmapWidth * sizeof(DWORD), D3D12_TEXTURE_DATA_PITCH_ALIGNMENT);

//
// Note that the helper function UpdateSubresource in D3DX12.h, and ID3D12Device::GetCopyableFootprints 
// can help applications fill out D3D12_SUBRESOURCE_FOOTPRINT and D3D12_PLACED_SUBRESOURCE_FOOTPRINT structures.
//
// Refer to the D3D12 Code example for the previous section "Uploading Different Types of Resources"
// for the code for SuballocateFromBuffer.
//

SuballocateFromBuffer(
    pitchedDesc.Height * pitchedDesc.RowPitch,
    D3D12_TEXTURE_DATA_PLACEMENT_ALIGNMENT
    );

D3D12_PLACED_SUBRESOURCE_FOOTPRINT placedTexture2D = { 0 };
placedTexture2D.Offset = m_pDataCur – m_pDataBegin;
placedTexture2D.Footprint = pitchedDesc;

//
// Copy texture data from DWORD* pBitmap->pixels to the buffer
//

for (UINT y = 0; y < bitmapHeight; y++)
{
  UINT8 *pScan = m_pDataBegin + placedTexture2D.Offset + y * pitchedDesc.RowPitch;
  memcpy( pScan, &(pBitmap->pixels[y * bitmapWidth]), sizeof(DWORD) * bitmapWidth );
}

//
// Create default texture2D resource.
//

D3D12_RESOURCE_DESC  textureDesc { ... };

CComPtr<ID3D12Resource> texture2D;
d3dDevice->CreateCommittedResource( 
        &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT), 
        D3D12_HEAP_FLAG_NONE, &textureDesc, 
        D3D12_RESOURCE_STATE_COPY_DEST, 
        nullptr, 
        IID_PPV_ARGS(&texture2D) );

//
// Copy heap data to texture2D.
//

commandList->CopyTextureRegion( 
        &CD3DX12_TEXTURE_COPY_LOCATION( texture2D, 0 ), 
        0, 0, 0, 
        &CD3DX12_TEXTURE_COPY_LOCATION( m_spUploadHeap, placedTexture2D ), 
        nullptr );
```

请注意使用的帮助器结构[ **CD3DX12\_堆\_属性**](cd3dx12-heap-properties.md)并[ **CD3DX12\_纹理\_复制\_位置**](cd3dx12-texture-copy-location.md)，和方法[ **CreateCommittedResource** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommittedresource)并[ **CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion).

## <a name="copying"></a>复制

D3D12 方法使应用程序以替换 D3D11 [ **UpdateSubresource**](https://msdn.microsoft.com/library/windows/desktop/ff476486)， [ **CopySubresourceRegion**](https://msdn.microsoft.com/library/windows/desktop/ff476394)，和初始资源数据。 单个 3D 的子资源值得行优先纹理数据可能会出现在缓冲区资源。 [**CopyTextureRegion** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)可以将该纹理数据从缓冲区复制到的未知的纹理布局中，使用的纹理资源，反之亦然。 应用程序应优先使用这种类型的方法来填充经常访问的 GPU 资源，通过在上传堆中创建具有无 CPU 访问默认堆中经常访问的 GPU 资源时创建较大的缓冲区。 这种方法有效地支持离散 Gpu 和 CPU 访问内存中，其大量不通常会影响到 UMA 体系结构。

请注意以下两个常量：

``` syntax
const UINT D3D12_TEXTURE_DATA_PITCH_ALIGNMENT = 256;
const UINT D3D12_TEXTURE_DATA_PLACEMENT_ALIGNMENT = 512;
```

-   [**D3D12\_SUBRESOURCE\_FOOTPRINT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_subresource_footprint)
-   [**D3D12\_PLACED\_SUBRESOURCE\_FOOTPRINT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_placed_subresource_footprint)
-   [**D3D12\_纹理\_复制\_位置**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_texture_copy_location)
-   [**D3D12\_TEXTURE\_COPY\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_texture_copy_type)
-   [**ID3D12Device::GetCopyableFootprints**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getcopyablefootprints)
-   [**ID3D12GraphicsCommandList::CopyResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copyresource)
-   [**ID3D12GraphicsCommandList::CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)
-   [**ID3D12GraphicsCommandList::CopyBufferRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion)
-   [**ID3D12GraphicsCommandList::CopyTiles**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytiles)
-   [**ID3D12CommandQueue::UpdateTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings)

## <a name="mapping-and-unmapping"></a>映射和取消映射

[**地图**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)并[ **Unmap** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap)可以安全地调用由多个线程。 首次调用**映射**资源分配的 CPU 虚拟地址范围。 上次调用**Unmap**解除分配的 CPU 虚拟地址范围。 CPU 虚拟地址通常返回到应用程序。

通过 readback 中的资源时的 CPU 和 GPU 之间传递数据堆[**地图**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)并[ **Unmap** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap)必须用于支持所有支持系统 D3D12。 保留范围严格尽可能最大化需要范围的系统上的效率 (请参阅[ **D3D12\_范围**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_range))。

调试工具好处不仅上所有范围的精确使用情况的性能[**地图**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map) / [**Unmap** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap)调用，而且还从应用程序时将无法再进行 CPU 修改中取消映射的资源。

使用 D3D11 方法[**地图**](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map) （使用放弃参数设置） 来重命名的资源中不支持 D3D12。 应用程序必须实现重命名本身的资源。 所有**地图**调用都是隐式无\_覆盖和多线程。 它是应用程序负责确保，命令列表中包含任何相关的 GPU 工作完成后再访问数据，cpu。 调用 D3D12**映射**执行不隐式刷新任何命令缓冲区，也不在阻止等待 GPU 完成工作。 因此，**地图**并[ **Unmap** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-unmap)可能甚至被优化出在某些情况下。

## <a name="buffer-alignment"></a>缓冲区对齐方式

缓冲区对齐限制：

-   线性 subresource 复制必须为 512 个字节对齐 (与行间距对齐到 D3D12\_纹理\_数据\_间距\_对齐字节)。
-   常量数据读取必须是从一堆开始的 256 个字节的倍数 (即仅从 256 字节对齐的地址)。
-   索引数据读取必须是索引的数据类型大小的倍数 (即仅从自然对齐的数据的地址)。
-   [**ID3D12GraphicsCommandList::DrawInstanced** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced)并[ **ID3D12GraphicsCommandList::DrawIndexedInstanced** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawindexedinstanced)数据必须从偏移量的 （即以 4 的倍数仅从地址是双字节对齐）。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[在缓冲区中的子分配](large-buffers.md)
</dt> </dl>

 

 




