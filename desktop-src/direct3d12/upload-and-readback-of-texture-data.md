---
title: 通过缓冲区上传纹理数据
description: 上传 2D 或 3D 纹理数据与上传 1D 数据类似，但应用程序需要更密切地关注与行间距相关的数据对齐情况。
ms.assetid: 22A25A94-A45C-482D-853A-FA6860EE7E4E
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 4b268f02bd523e286e227cd134c1d9d41303ea15
ms.sourcegitcommit: 27a9dfa3ef68240fbf09f1c64dff7b2232874ef4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66725590"
---
# <a name="uploading-texture-data-through-buffers"></a>通过缓冲区上传纹理数据

上传 2D 或 3D 纹理数据与上传 1D 数据类似，但应用程序需要更密切地关注与行间距相关的数据对齐情况。 可以从图形管道的多个部分正交和同时使用缓冲区，缓冲区非常灵活。

-   [通过缓冲区上传纹理数据](#upload-texture-data-via-buffers)
-   [复制](#copying)
-   [映射和取消映射](#mapping-and-unmapping)
-   [缓冲区对齐](#buffer-alignment)
-   [相关主题](#related-topics)

## <a name="upload-texture-data-via-buffers"></a>通过缓冲区上传纹理数据

应用程序必须通过 [ID3D12GraphicsCommandList::CopyTextureRegion](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion) 或 [ID3D12GraphicsCommandList::CopyBufferRegion](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion) 上传数据   。 与其他资源数据相比，纹理数据更有可能变得更大、被重复访问，以及从改进非线性内存布局的缓存一致性中获益。 在 D3D12 中使用缓冲区时，只要满足内存对齐要求，应用程序就可以完全控制与复制周围资源数据相关的数据放置和排列。

该示例突出显示应用程序在将 2D 数据放入缓冲区之前简单地将其平面化为 1D 的位置。 对于 mipmap 2D 场景，应用程序可以离散地将每个子资源平面化并快速使用 1D 二次分配算法，或者使用更复杂的 2D 二次分配方法来最大程度降低视频内存利用率。 第一种方法应会更常用，因为它更简单。 将数据打包到磁盘或跨网络打包数据时，第二种方法可能很有用。 在任一种情况下，应用程序都仍必须对每个子资源调用复制 API。

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

请注意帮助程序结构 [CD3DX12\_HEAP\_PROPERTIES](cd3dx12-heap-properties.md) 和 [CD3DX12\_TEXTURE\_COPY\_LOCATION](cd3dx12-texture-copy-location.md) 以及方法 [CreateCommittedResource](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommittedresource) 和 [CopyTextureRegion](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)     的使用。

## <a name="copying"></a>复制

D3D12 方法使应用程序能够替换 D3D11 [UpdateSubresource](https://docs.microsoft.com/windows/desktop/api/d3d11/nf-d3d11-id3d11devicecontext-updatesubresource)、[CopySubresourceRegion](https://docs.microsoft.com/windows/desktop/api/d3d11/nf-d3d11-id3d11devicecontext-copysubresourceregion) 以及资源初始数据   。 行-主要纹理数据的单个 3D 子资源可能位于缓冲区资源中。 [ CopyTextureRegion](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion) 可以将纹理数据从缓冲区复制到纹理布局未知的纹理资源，反之亦然  。 应用程序应优先使用此类方式来填充频繁访问的 GPU 资源，方法是在 UPLOAD 堆中创建大型缓冲区，同时在不具有 CPU 访问权限的 DEFAULT 堆中创建频繁访问的 GPU 资源。 这种方式可有效地支持离散 GPU 及其大量的 CPU 不可访问的存储器，而不会损害 UMA 体系结构。

请注意以下两个常量：

``` syntax
const UINT D3D12_TEXTURE_DATA_PITCH_ALIGNMENT = 256;
const UINT D3D12_TEXTURE_DATA_PLACEMENT_ALIGNMENT = 512;
```

-   [**D3D12\_SUBRESOURCE\_FOOTPRINT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_subresource_footprint)
-   [**D3D12\_PLACED\_SUBRESOURCE\_FOOTPRINT**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_placed_subresource_footprint)
-   [**D3D12\_TEXTURE\_COPY\_LOCATION**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_texture_copy_location)
-   [**D3D12\_TEXTURE\_COPY\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_texture_copy_type)
-   [**ID3D12Device::GetCopyableFootprints**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getcopyablefootprints)
-   [**ID3D12GraphicsCommandList::CopyResource**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copyresource)
-   [**ID3D12GraphicsCommandList::CopyTextureRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion)
-   [**ID3D12GraphicsCommandList::CopyBufferRegion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copybufferregion)
-   [**ID3D12GraphicsCommandList::CopyTiles**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytiles)
-   [**ID3D12CommandQueue::UpdateTileMappings**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings)

## <a name="mapping-and-unmapping"></a>映射和取消映射

[Map](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map) 和 [Unmap](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-unmap) 可由多个线程安全地调用   。 首次调用 Map 可为资源分配 CPU 虚拟地址范围  。 最后一次调用 Unmap 可取消分配 CPU 虚拟地址范围  。 系统通常会向应用程序返回该 CPU 虚拟地址。

当数据通过 Readback 堆中的资源在 CPU 和 GPU 之间传递时，必须使用 [Map](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map) 和 [Unmap](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-unmap) 来支持可支持 D3D12 的所有系统   。 在需要范围的系统上，应尽可能保持较小的范围以最大限度地提高效率（请参阅 [D3D12\_RANGE](/windows/desktop/api/d3d12/ns-d3d12-d3d12_range)）  。

调试工具的性能不仅得益于对所有 [Map](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map) / [Unmap](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-unmap) 调用的范围的准确使用，还得益于应用程序在不再进行 CPU 修改时取消映射资源   。

D3D11 方法可使用 [Map](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-map)（使用 DISCARD 参数集）重命名资源，D3D12 不支持此做法  。 应用程序必须自己实现资源重命名。 所有 Map 调用都是隐式 NO\_OVERWRITE 和多线程的  。 应用程序复制确保在使用 CPU 访问数据之前完成命令列表中包含的任何相关 GPU 工作。 D3D12 调用 Map 不会隐式刷新任何命令缓冲区，也不会阻止等待 GPU 完成工作  。 因此，Map 和 [Unmap](/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-unmap) 甚至可能在某些情况下进行优化   。

## <a name="buffer-alignment"></a>缓冲区对齐

缓冲区对齐限制：

-   线性子资源复制必须对齐到 512 字节（行间距与 D3D12\_TEXTURE\_DATA\_PITCH\_ALIGNMENT 字节对齐）。
-   常量数据读取数必须是来自堆起始处的 256 字节的倍数（即仅来自 256 字节对齐的地址）。
-   索引数据读取数必须是索引数据类型大小的倍数（即仅来自与数据自然对齐的地址）。
-   [ID3D12GraphicsCommandList::DrawInstanced](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced) 和 [ID3D12GraphicsCommandList::DrawIndexedInstanced](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawindexedinstanced) 数据必须来自为 4 的倍数的偏移量（即仅来自 DWORD 对齐的地址）   。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[缓冲区中的二次分配](large-buffers.md)
</dt> </dl>

 

 




