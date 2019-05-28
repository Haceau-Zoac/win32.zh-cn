---
title: 根签名
description: 根签名定义的资源类型绑定到图形管道。
ms.assetid: ee32a222-8469-4af5-b688-afa70cf77c6a
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: a18f470e590f00a7b7d3a6e43d525596c90580af
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224104"
---
# <a name="root-signatures"></a>根签名

根签名定义的资源类型绑定到图形管道。

## <a name="in-this-section"></a>本部分内容



| 主题                                                                                                               | 描述                                                                                                                                                                                                                                                                                                                                                                                                       |
|---------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [根签名概述](root-signatures-overview.md)<br/>                                                 | 根签名配置应用程序和链接命令将列出到着色器需要的资源。 图形命令列表具有的图形和计算的根签名。 一个计算命令列表将只具有一个计算根签名。 这些根签名是相互独立。<br/>                                                                                             |
| [使用根签名](using-a-root-signature.md)<br/>                                                     | 根签名是随意排列的描述符表 （包括其布局），集合的定义根常量和根描述符。 每个条目具有针对最大限制，成本，因此应用程序可以占用较根签名将包含多少个项的每个类型之间的平衡。<br/>                                                                     |
| [创建根签名](creating-a-root-signature.md)<br/>                                               | 根签名是一个包含嵌套的结构的复杂数据结构。 这些都可以使用以下数据结构定义 （其中包括方法，以帮助初始化成员） 以编程方式定义。 或者，他们可以编写在高级别着色语言 (HLSL) 提供的一个优势在于，编译器将验证早期的布局与着色器兼容。 <br/> |
| [根签名限制](root-signature-limits.md)<br/>                                                       | 根签名是素数房地产和有严格的限制和成本考虑。<br/>                                                                                                                                                                                                                                                                                                            |
| [直接在根签名中使用常量](using-constants-directly-in-the-root-signature.md)<br/>     | 应用程序可以在根签名中，每个作为 32 位值的一组定义根常量。 它们显示在高级别着色语言 (HLSL) 为常量缓冲区。 请注意出于历史原因的常量缓冲区被视为 4 x 32 位值的集。 <br/>                                                                                                                                        |
| [根签名中直接使用描述符](using-descriptors-directly-in-the-root-signature.md)<br/> | 应用程序可以将描述符直接置于根签名，以避免无需经历描述符堆。 这些描述符会占用大量空间中的根签名 （请参阅根签名限制部分），因此，应用程序需要尽量少用。 <br/>                                                                                                                                     |
| [示例根签名](example-root-signatures.md)<br/>                                                   | 以下部分显示不同的程度不一到完全填充从空的根签名。<br/>                                                                                                                                                                                                                                                                                                       |
| [在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)<br/>                             | HLSL 着色器模型 5.1 中指定根签名并指定它们中的替代方法C++代码。<br/>                                                                                                                                                                                                                                                                                                  |
| [根签名版本 1.1](root-signature-version-1-1.md)<br/>                                             | 根签名版本 1.1 的目的是使应用程序描述符堆中的描述符结束-赢得 t 更改或数据描述符指向获胜的 t 更改时指示驱动程序。 这样，驱动程序进行优化，可能会知道，描述符或它指向的内存静态的某个时间段内的选项。 <br/>                                  |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[**ID3D12RootSignature**](https://msdn.microsoft.com/en-us/library/Dn788714(v=VS.85).aspx)
</dt> <dt>

[**ID3D12RootSignatureDeserializer**](/windows/desktop/api/D3D12/nn-d3d12-id3d12rootsignaturedeserializer)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 





