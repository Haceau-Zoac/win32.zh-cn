---
title: 使用 Direct3D 11、Direct3D 10 和 Direct2D
description: 本节介绍早期版本的 Direct3D 和 Direct2D、Direct3D 11on12 API 以及从 Direct3D 11 到 Direct3D 12 的移植指南中的互操作技术。
ms.assetid: 1AB98335-30B1-4244-B244-F8573524B38C
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 6ce3a6ec37465c117634a56cfeac0188b1f9508d
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "67394575"
---
# <a name="working-with-direct3d-11-direct3d-10-and-direct2d"></a>使用 Direct3D 11、Direct3D 10 和 Direct2D

本节介绍早期版本的 Direct3D 和 Direct2D、Direct3D 11on12 API 以及从 Direct3D 11 到 Direct3D 12 的移植指南中的互操作技术。

## <a name="in-this-section"></a>本节内容



| 主题                                                                                             | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|---------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Direct3D 12 互操作](direct3d-12-with-direct3d-11--direct-2d-and-gdi.md)<br/>             | D3D12 可以用于编写组件化的应用程序。 <br/>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| [Direct3D 11 on 12](direct3d-11-on-12.md)<br/>                                             | 开发人员可以通过 D3D11On12 机制使用 D3D11 接口和对象来驱动 D3D12 API。 借助 D3D11on12，使用 D3D11 编写的组件（如 D2D 文本和 UI）可与针对 D3D12 API 编写的组件配合工作。 D3D11on12 还可以实现应用程序从 D3D11 到 D3D12 的增量移植，它可让应用的某些组成部分继续针对 D3D11 以简化操作，同时让其他某些组成部分针对 D3D12 以保证性能，并始终提供完整且准确的渲染效果。 使用 D3D11On12 比使用互操作技术在两个 API 之间共享资源和同步工作更为简单。 <br/> |
| [从 Direct3D 移植到 Direct3D 11 12](porting-from-direct3d-11-to-direct3d-12.md)<br/> | 本部分提供有关从自定义的 Direct3D 11 图形引擎移植到 Direct3D 12 的一些指导。<br/>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 编程指南](directx-12-programming-guide.md)
</dt> </dl>

 

 





