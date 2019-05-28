---
title: 描述符概述
description: 描述符创建的 API 调用和标识的资源。
ms.assetid: 64721226-5533-4816-865E-9429032FCC86
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 9fc60357398b8360536dfc6f1d18e3e7557ff3f7
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224101"
---
# <a name="descriptors-overview"></a>描述符概述

描述符创建的 API 调用和标识的资源。

-   [描述符数据](#descriptor-data)
-   [描述符句柄](#descriptor-handles)
-   [Null 描述符](#null-descriptors)
-   [默认描述符](#default-descriptors)
-   [相关的主题](#related-topics)

## <a name="descriptor-data"></a>描述符数据

描述符是完全描述一个对象到 GPU GPU 特定不透明格式的数据相对较小块。 有许多不同类型的描述符：着色器资源视图 (SRVs)、 无序访问视图 (Uav)、 常量缓冲区视图 (CBVs) 和取样器是一些示例。

描述符具有不同大小的通常 SRV、 UAV 或 CBV 32 到 64 字节 （具体取决于 GPU 硬件），并且显示在本文档中作为不可分的单元，例如：

![srv、 cbv、 uav 和采样器](images/single-descriptor.png)

描述符创建的 API 调用，将包括如资源的信息并且想要包含的描述符 mip 贴图。

该驱动程序不会跟踪或具有对描述符的引用，它最多是应用程序以确保正确描述符类型正在使用，并且信息是最新。 还有一个小例外;该驱动程序 does 检查呈现器目标绑定，以确保交换链工作正常。

对象描述符不必释放或释放。 驱动程序不附加任何分配的描述符创建。 描述符可能，但是，对为其应用程序拥有生存期其他分配对引用进行编码。 例如，SRV 的说明符必须包含 SRV 指 D3D 资源 （例如纹理） 的虚拟地址。 它是应用程序的责任，以确保它不会使用 SRV 描述符，这取决于基础 D3D 资源已被销毁或正在修改 （如被声明为外来） 时。

使用描述符的主要方法是将它们放在描述符堆，这支持的内存用于描述符。

## <a name="descriptor-handles"></a>描述符句柄

描述符句柄是描述符的唯一地址。 它类似于一个指针，但不透明的因为其实现是特定于硬件。 句柄是唯一的描述符堆，因此，例如，一个句柄数组可以引用多个堆中的描述符。

立即使用，如将复制的源和目标需要确定是 CPU 句柄。

GPU 句柄不立即使用，它们确定从命令列表，以便在 GPU 执行时间使用的位置。

若要创建堆，开头的描述符句柄创建描述符堆本身后，请调用以下方法之一：

-   [**ID3D12DescriptorHeap::GetCPUDescriptorHandleForHeapStart**](/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart)
-   [**ID3D12DescriptorHeap::GetGPUDescriptorHandleForHeapStart**](/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getgpudescriptorhandleforheapstart)

这些方法返回了以下结构：

-   [**D3D12\_CPU\_DESCRIPTOR\_HANDLE**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_cpu_descriptor_handle)
-   [**D3D12\_GPU\_DESCRIPTOR\_HANDLE**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_gpu_descriptor_handle)

随着硬件，以获取每个描述符堆使用之间的增量的说明符的大小变化：

-   [**ID3D12Device::GetDescriptorHandleIncrementSize**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-getdescriptorhandleincrementsize)

它可以安全地递增，来复制句柄，并将句柄传递到 API 调用数与起始位置的偏移。 不安全，若要取消引用句柄，就像它是有效的 CPU 指针，也不分析中一个句柄的位。

已添加一些帮助器结构，与初始化成员，以更简单管理图柄。

-   [**CD3DX12\_CPU\_DESCRIPTOR\_HANDLE**](cd3dx12-cpu-descriptor-handle.md)
-   [**CD3DX12\_GPU\_DESCRIPTOR\_HANDLE**](cd3dx12-gpu-descriptor-handle.md)

## <a name="null-descriptors"></a>Null 描述符

在创建时，通过调用 API 的描述符，应用程序传递要达到的效果的任何内容时访问的着色器绑定的描述符定义中的资源指针为 NULL。

必须尽可能多地填充描述符的其余部分。 例如，在着色器资源视图 (SRVs) 的情况下描述符可用来区分类型的视图是 （Texture1D、 Texture2D，等）。 中的视图描述符，例如 mipmap，数的数字参数必须全部设置为有效的资源的值。

在许多情况下，没有用于访问未绑定的资源，如 SRVs 返回默认值已定义的行为。 访问 NULL 描述符，前提是与描述符类型兼容的着色器访问类型时，这些将起作用。 例如，如果着色器期望 Texture2D SRV 和 NULL SRV 定义为 Texture1D 的访问，该行为将是不确定，可能会导致设备重置。

总之，若要创建空的描述符，请将传递`null`有关*pResource*参数，如使用方法创建视图时[ **CreateShaderResourceView** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createshaderresourceview). 为视图说明参数*pDesc*，设置配置的那样如果资源不是 null （否则崩溃可能会出现一些硬件上）。

根描述符但是，不应设置为 null。

## <a name="default-descriptors"></a>默认描述符

若要创建的特定视图的默认描述符，请将传递中的有效*pResource* create 视图方法的参数 (如[ **CreateShaderResourceView**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createshaderresourceview))，但在传递为 null *pDesc*参数。 例如，如果该资源包含 14 mips，视图将包含 14 mips。 默认情况下包括到视图资源的最明显的映射。 这需要资源的分配具有完全限定的格式名称 (例如 DXGI\_格式\_R8G8B8A8\_UNORM\_SRGB 而不是 DXGI\_格式\_R8G8B8A8\_无类型）。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符](descriptors.md)
</dt> </dl>

 

 




