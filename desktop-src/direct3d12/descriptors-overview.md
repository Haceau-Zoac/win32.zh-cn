---
title: 描述符概述
description: 描述符由 API 调用创建并标识资源。
ms.assetid: 64721226-5533-4816-865E-9429032FCC86
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: f45b2aafd85ed43396508bfb6852a0f862da9e15
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296241"
---
# <a name="descriptors-overview"></a>描述符概述

描述符由 API 调用创建并标识资源。

-   [描述符数据](#descriptor-data)
-   [描述符句柄](#descriptor-handles)
-   [Null 描述符](#null-descriptors)
-   [默认描述符](#default-descriptors)
-   [相关主题](#related-topics)

## <a name="descriptor-data"></a>描述符数据

描述符是一个相对较小的数据块，以 GPU 特定的不透明格式完全描述提交到 GPU 的对象。 有多种不同类型的描述符：着色器资源视图 (SRV)、无序访问视图 (UAV)、常量缓冲区视图 (CBV) 和采样器就是其中的几个例子。

描述符具有不同的大小（对于 SRV、UAV 或 CBV 通常为 32 到 64 个字节（具体取决于 GPU 硬件）），并在本文档中显示为不可分割的单元，例如：

![srv、cbv、uav 和采样器](images/single-descriptor.png)

描述符由 API 调用创建，并且将包括希望描述符包含的资源和 mip-map 等信息。

驱动程序不会跟踪或保存对描述符的引用，由应用来确保正在使用正确的描述符类型，并且信息是最新的。 此情况有一个小例外；驱动程序会检查呈现器目标绑定以确保交换链正常工作。

无需释放对象描述符。 驱动程序不会为描述符创建附加任何分配。 但是，描述符可能会对应用程序永久拥有的其他分配的引用进行编码。 例如，SRV 的描述符必须包含 SRV 引用的 D3D 资源（例如纹理）的虚拟地址。 由应用来确保在 SRV 描述符所依赖的基础 D3D 资源已被销毁或修改（例如，声明为非常驻）时不使用该描述符。

使用描述符的主要方法是将它们放置在描述符堆中，这些描述符堆是描述符的后备内存。

## <a name="descriptor-handles"></a>描述符句柄

描述符句柄是描述符的唯一地址。 它类似于指针，但不透明，因为其实现特定于硬件。 句柄在描述符堆唯一，例如，句柄数组可以引用多个堆中的描述符。

CPU 句柄可供立即使用，例如，需要同时确定源和目标的复制。

GPU 句柄不可供立即使用，它们从命令列表中确定位置，以便在 GPU 执行时使用。

若要在创建描述符堆本身之后为堆的开头创建描述符句柄，请调用以下方法之一：

-   [**ID3D12DescriptorHeap::GetCPUDescriptorHandleForHeapStart**](/windows/desktop/api/d3d12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart)
-   [**ID3D12DescriptorHeap::GetGPUDescriptorHandleForHeapStart**](/windows/desktop/api/d3d12/nf-d3d12-id3d12descriptorheap-getgpudescriptorhandleforheapstart)

这些方法返回以下结构：

-   [**D3D12\_CPU\_DESCRIPTOR\_HANDLE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_cpu_descriptor_handle)
-   [**D3D12\_GPU\_DESCRIPTOR\_HANDLE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_gpu_descriptor_handle)

由于描述符大小因硬件而异，因此若要获得堆中的每个描述符之间的增量，请使用：

-   [**ID3D12Device::GetDescriptorHandleIncrementSize**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-getdescriptorhandleincrementsize)

可以安全地使用若干增量偏移起始位置，复制句柄并将句柄传递到 API 调用数中。 取消引用句柄（如同有效的 CPU 指针一样）不安全，分析句柄中的位数也不安全。

已添加一些具有初始化成员的帮助程序结构，以便更易于管理句柄。

-   [**CD3DX12\_CPU\_DESCRIPTOR\_HANDLE**](cd3dx12-cpu-descriptor-handle.md)
-   [**CD3DX12\_GPU\_DESCRIPTOR\_HANDLE**](cd3dx12-gpu-descriptor-handle.md)

## <a name="null-descriptors"></a>Null 描述符

使用 API 调用创建描述符时，应用程序针对描述符定义中的资源指针传递 NULL 以达到被着色器访问时未绑定的效果。

必须尽可能多地填充描述符的其余部分。 例如，如果是着色器资源视图 (SRV)，描述符可用于区分视图的类型（Texture1D、Texture2D 等）。 视图描述符中的数值参数（例如 mipmap 数）必须全部设置为适用于资源的值。

在许多情况下，有访问未绑定资源的已定义行为，例如返回默认值的 SRV。 访问 NULL 描述符时支持这些情况，前提是着色器访问的类型与描述符类型兼容。 例如，如果着色器需要 Texture2D SRV 并访问定义为 Texture1D 的 NULL SRV，则该行为是未定义的，并且可能会导致设备重置。

总之，若要创建 null 描述符，请在使用 [**CreateShaderResourceView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createshaderresourceview) 等方法创建视图时针对 pResource  参数传递 `null`。 对于视图描述参数 pDesc  ，设置在资源不是 null 时可行的配置（否则，某些硬件可能会崩溃）。

但是，根描述符不应设置为 null。

## <a name="default-descriptors"></a>默认描述符

若要创建特定视图的默认描述符，请将有效的 pResource  参数传递到创建视图方法（例如，[**CreateShaderResourceView**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createshaderresourceview)），但针对 pDesc  参数传递 null。 例如，如果资源包含 14 个 mip，则视图将包含 14 个 mip。 默认情况包含资源到视图的最明显映射。 这需要资源分配有完全限定的格式名称（例如，DXGI\_FORMAT\_R8G8B8A8\_UNORM\_SRGB，而不是 DXGI\_FORMAT\_R8G8B8A8\_TYPELESS）。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符](descriptors.md)
</dt> </dl>

 

 




