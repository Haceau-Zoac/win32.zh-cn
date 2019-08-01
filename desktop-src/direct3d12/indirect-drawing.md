---
title: 间接绘制
description: 通过间接绘制，某些场景遍历和精选可以从 CPU 移动到 GPU，从而可以提高性能。 可以通过 CPU 或 GPU 生成命令缓冲区。
ms.assetid: F8D6C88A-101E-4F66-999F-43206F6527B6
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: ac2ecace17cf9474cfe4548e1b09d025867dca15
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296457"
---
# <a name="indirect-drawing"></a>间接绘制

通过间接绘制，某些场景遍历和精选可以从 CPU 移动到 GPU，从而可以提高性能。 可以通过 CPU 或 GPU 生成命令缓冲区。

-   [命令签名](#command-signatures)
-   [间接参数缓冲区结构](#indirect-argument-buffer-structures)
-   [命令签名创建](#command-signature-creation)
    -   [无参数更改](#no-argument-changes)
    -   [根常量和顶点缓冲区](#root-constants-and-vertex-buffers)
-   [相关主题](#related-topics)

## <a name="command-signatures"></a>命令签名

命令签名对象 ([**ID3D12CommandSignature**](https://msdn.microsoft.com/library/Dn891446(v=VS.85).aspx)) 允许应用指定间接绘制，特别是进行以下设置：

-   间接参数缓冲区格式。
-   将使用的命令类型（来自 [**ID3D12GraphicsCommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist) 方法 [**DrawInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced)、[**DrawIndexedInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawindexedinstanced) 或 [**Dispatch**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch)）。
-   将更改每命令调用的资源绑定的集合与将继承的集合。

启动时，应用会创建一小部分的命令签名  。 运行时，应用程序会使用命令填充缓冲区（通过应用开发人员选择的任何方式）。 要为顶点缓冲区视图、索引缓冲区视图、根常量和根描述符（原始或结构化 SRV/UAV/CBV）设置的命令（可选择包含状态）。 这些参数布局不特定于硬件，因此应用可以直接生成缓冲区。 命令签名从命令列表继承剩余状态。 然后，应用调用 [**ExecuteIndirect**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executeindirect) 以指示 GPU 根据特定命令签名所定义的的格式解释间接参数缓冲区的内容。

如果命令签名更改任何根参数，则该签名作为根签名的一部分存储在命令签名中。

请注意，在执行完成后，不会有命令签名状态泄漏回命令列表。

例如，假设应用开发人员希望在每次绘制调用时间接参数缓冲区指定唯一根常量。 应用将创建允许在每次绘制调用时间接参数缓冲区指定以下参数的命令签名：

-   一个根常量的值。
-   绘制参数（顶点计数、实例计数等）。

应用程序生成的间接参数缓冲区将包含固定大小的记录的数组。 每个结构都对应于一个绘制调用。 每个结构都包含绘制参数和根常量的值。 绘制调用数在单独的 GPU 可见缓冲区中指定。

应用生成的示例命令缓冲区如下所示：

![命令缓冲区格式](images/indirect-drawing-command-buffer.png)

## <a name="indirect-argument-buffer-structures"></a>间接参数缓冲区结构

以下结构定义特定参数在间接参数缓冲区中的显示方式。 这些结构不会显示在任何 D3D12 API 中。 写入间接参数缓冲区（使用 CPU 或 GPU）时，应用程序使用这些定义：

-   [**D3D12\_DRAW\_ARGUMENTS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_draw_arguments)
-   [**D3D12\_DRAW\_INDEXED\_ARGUMENTS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_draw_indexed_arguments)
-   [**D3D12\_DISPATCH\_ARGUMENTS**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_dispatch_arguments)
-   [**D3D12\_VERTEX\_BUFFER\_VIEW**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_vertex_buffer_view)
-   [**D3D12\_INDEX\_BUFFER\_VIEW**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_index_buffer_view)
-   D3D12\_GPU\_VIRTUAL\_ADDRESS（UINT64 的 typedef'd 同义词）。
-   [**D3D12\_CONSTANT\_BUFFER\_VIEW**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_constant_buffer_view_desc)

## <a name="command-signature-creation"></a>命令签名创建

若要创建命令签名，请使用以下 API 项：

-   [**ID3D12Device::CreateCommandSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandsignature)（输出 [**ID3D12CommandSignature**](https://msdn.microsoft.com/library/Dn891446(v=VS.85).aspx)）
-   [**D3D12\_INDIRECT\_ARGUMENT\_TYPE**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_indirect_argument_type)
-   [**D3D12\_INDIRECT\_ARGUMENT\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_indirect_argument_desc)
-   [**D3D12\_COMMAND\_SIGNATURE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_signature_desc)

间接参数缓冲区中的参数顺序定义为与 [**D3D12\_COMMAND\_SIGNATURE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_command_signature_desc) 的 pArguments  参数中指定的参数顺序完全匹配。 间接参数缓冲区中的一个绘制（图形）/调度（计算）调用的所有参数紧密打包。 但是，允许应用程序指定间接参数缓冲区中的绘制/调度命令之间的任意字节步幅。

当且仅当命令签名更改其中一个根参数时，必须指定根签名。

对于根 CBV/SRV/UAV，应用程序指定的大小以字节为单位。 调试层将验证对地址的以下限制：

-   CBV – 地址必须是 256 字节的倍数。
-   原始 SRV/UAV – 地址必须是 4 字节的倍数。
-   结构化 SRV/UAV – 地址必须是结构字节步幅的倍数（已在着色器中声明）。

给定的命令签名是绘制或计算命令签名。 如果命令签名包含绘制操作，则该签名是图形命令签名。 否则，命令签名必须包含调度操作，并且该签名是计算命令签名。

以下部分显示了一些示例命令签名。

### <a name="no-argument-changes"></a>无参数更改

在此示例中，应用程序生成的间接参数缓冲区包含 36 字节结构的数组。 每个结构仅包含传递给 [**DrawIndexedInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawindexedinstanced) 的五个参数（以及填充）。

用于创建命令签名说明的代码如下所示：

``` syntax
D3D12_INDIRECT_ARGUMENT_DESC Args[1];
Args[0].Type = D3D12_INDIRECT_ARGUMENT_TYPE_DRAW_INDEXED;

D3D12_COMMAND_SIGNATURE_DESC ProgramDesc;
ProgramDesc.ByteStride = 36;
ProgramDesc.ArgumentCount = 1;
ProgramDesc.pArguments = Args;
```

间接参数缓冲区中的单个结构的布局如下所示：



| 字节 | 描述           |
|-------|-----------------------|
| 0:3   | IndexCountPerInstance |
| 4:7   | InstanceCount         |
| 8:11  | StartIndexLocation    |
| 12:15 | BaseVertexLocation    |
| 16:19 | StartInstanceLocation |
| 20:35 | 填充               |



 

### <a name="root-constants-and-vertex-buffers"></a>根常量和顶点缓冲区

在此示例中，间接参数缓冲区中的每个结构更改两个根常量、更改一个顶点缓冲区绑定并执行一个绘制非索引操作。 结构之间没有填充。

用于创建命令签名说明的代码如下所示：

``` syntax
D3D12_INDIRECT_ARGUMENT_DESC Args[4];
Args[0].Type = D3D12_INDIRECT_ARGUMENT_TYPE_CONSTANT;
Args[0].Constant.RootParameterIndex = 2;

Args[1].Type = D3D12_INDIRECT_ARGUMENT_TYPE_CONSTANT;
Args[1].Constant.RootParameterIndex = 6;

Args[2].Type = D3D12_INDIRECT_ARGUMENT_TYPE_VERTEX_BUFFER_VIEW;
Args[2].VertexBuffer.VBSlot = 3;

Args[3].Type = D3D12_INDIRECT_ARGUMENT_TYPE_DRAW_INSTANCED;

D3D12_COMMAND_SIGNATURE_DESC ProgramDesc;
ProgramDesc.ByteStride = 40;
ProgramDesc.ArgumentCount = 4;
ProgramDesc.pArguments = Args;
```

间接参数缓冲区中的单个结构的布局如下所示：



| 字节 | 描述                     |
|-------|---------------------------------|
| 0:3   | 根参数索引 2 的数据 |
| 4:7   | 根参数索引 6 的数据 |
| 8:15  | VB 的虚拟地址（64 位）  |
| 16:19 | VB 大小                         |
| 20:23 | VB 步幅                       |
| 24:27 | VertexCountPerInstance          |
| 28:31 | InstanceCount                   |
| 32:35 | StartVertexLocation             |
| 36:39 | StartInstanceLocation           |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[DirectX 高级学习视频教程：执行间接和异步 GPU 精选](https://www.youtube.com/watch?v=fKD-VKJeeds)
</dt> <dt>

[间接绘制和 GPU 精选：代码演练](indirect-drawing-and-gpu-culling-.md)
</dt> <dt>

[呈现](rendering.md)
</dt> </dl>

 

 




