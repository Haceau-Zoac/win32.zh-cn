---
title: 描述符
description: 描述符是 D3D12 中单个资源的主要绑定单元。
ms.assetid: f35b5776-46b0-4659-9e61-c6ebfdb55f87
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: bf3745acfd2ac1c16eab108c296a52a373756413
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224098"
---
# <a name="descriptors"></a>描述符

描述符是 D3D12 中单个资源的主要绑定单元。

## <a name="in-this-section"></a>本部分内容



| 主题                                                       | 描述                                                                                                                                                                                                                                                                                                                                                                               |
|-------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [描述符概述](descriptors-overview.md)<br/> | 描述符由 API 调用创建，用于标识资源。<br/>                                                                                                                                                                                                                                                                                                                   |
| [创建描述符](creating-descriptors.md)<br/> | 描述并显示创建以下内容的示例：索引、顶点和常量缓冲区视图；着色器资源、呈现器目标、无序访问、流输出和深度模具视图；以及采样器。 创建描述符的所有方法都是自由线程。<br/>                                                                                                                             |
| [复制描述符](copying-descriptors.md)<br/>   | 设备接口上的 [ID3D12Device::CopyDescriptors](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-copydescriptors) 和 [ID3D12Device::CopyDescriptorsSimple](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-copydescriptorssimple) 方法使用 CPU 立即复制描述符   。 只要 CPU 或 GPU 上的多个线程不执行任何可能存在冲突的写入，就可以将它们称为自由线程。<br/> |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[描述符堆](descriptor-heaps.md)
</dt> <dt>

[描述符表](descriptor-tables.md)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> </dl>

 

 





