---
title: 根签名
description: 根签名定义绑定到图形管道的资源类型。
ms.assetid: ee32a222-8469-4af5-b688-afa70cf77c6a
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 21dc673869c5af7085858287ddef4cc0b0c47871
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296302"
---
# <a name="root-signatures"></a>根签名

根签名定义绑定到图形管道的资源类型。

## <a name="in-this-section"></a>本部分内容



| 主题                                                                                                               | 描述                                                                                                                                                                                                                                                                                                                                                                                                       |
|---------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [根签名概述](root-signatures-overview.md)<br/>                                                 | 根签名由应用配置，并将命令列表链接到着色器所需的资源。 图形命令列表同时具有图形和计算根签名。 计算命令列表只具有一个计算根签名。 这些根签名彼此独立。<br/>                                                                                             |
| [使用根签名](using-a-root-signature.md)<br/>                                                     | 根签名是描述符表（包括其布局）、根常量和根描述符的任意排列集合的定义。 每个条目都有一个接近最大限制的成本，因此应用程序可以在根签名将包含的每种类型条目的数量之间进行权衡。<br/>                                                                     |
| [创建根签名](creating-a-root-signature.md)<br/>                                               | 根签名是包含嵌套结构的复杂数据结构。 这些可以使用下面的数据结构定义（包括帮助初始化成员的方法）以编程方式定义。 或者，可以使用高级着色语言 (HLSL) 编写它们，这样编译器可以尽早验证布局是否与着色器兼容。 <br/> |
| [根签名限制](root-signature-limits.md)<br/>                                                       | 根签名是优质资产，需考虑严格的限制和成本。<br/>                                                                                                                                                                                                                                                                                                            |
| [直接在根签名中使用常量](using-constants-directly-in-the-root-signature.md)<br/>     | 应用程序可在根签名中定义根常量，每个常量都是一组 32 位值。 它们按高级着色语言 (HLSL) 显示为常量缓冲区。 请注意，由于历史原因，常量缓冲区被视为 4x32 位值的集合。 <br/>                                                                                                                                        |
| [直接在根签名中使用描述符](using-descriptors-directly-in-the-root-signature.md)<br/> | 应用程序可以将描述符直接放在根签名中，以避免通过描述符堆。 这些描述符在根签名中占用了大量空间（请参阅根签名限制部分），因此应用程序应尽量少用这些描述符。 <br/>                                                                                                                                     |
| [示例根签名](example-root-signatures.md)<br/>                                                   | 下一节介绍从空到完全填充的复杂性各异的根签名。<br/>                                                                                                                                                                                                                                                                                                       |
| [在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)<br/>                             | 如果在 HLSL 着色器模型 5.1 中指定根签名，则无需在 C++ 代码中指定这些根签名。<br/>                                                                                                                                                                                                                                                                                                  |
| [根签名版本 1.1](root-signature-version-1-1.md)<br/>                                             | 根签名版本 1.1 的目的是使应用程序能够在描述符堆中的描述符不更改或数据描述符指向不变的情况下向驱动程序指示。 这使得驱动程序可以选择进行可能的优化，因为知道描述符或它指向的内存在一段时间内是静态的。 <br/>                                  |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[**ID3D12RootSignature**](https://msdn.microsoft.com/library/Dn788714(v=VS.85).aspx)
</dt> <dt>

[**ID3D12RootSignatureDeserializer**](/windows/desktop/api/d3d12/nn-d3d12-id3d12rootsignaturedeserializer)
</dt> <dt>

[资源绑定](resource-binding.md)
</dt> </dl>

 

 





