---
title: 间接绘图
description: 间接绘图，一些场景遍历和消除从 CPU 移动到 GPU，这可以提高性能。 可以通过 CPU 或 GPU 生成命令缓冲区。
ms.assetid: F8D6C88A-101E-4F66-999F-43206F6527B6
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 0adb3da743d3bb155c08335ef6ac8689afc83ebf
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223813"
---
# <a name="indirect-drawing"></a>间接绘图

间接绘图，一些场景遍历和消除从 CPU 移动到 GPU，这可以提高性能。 可以通过 CPU 或 GPU 生成命令缓冲区。

-   [命令签名](#command-signatures)
-   [间接参数缓冲区结构](#indirect-argument-buffer-structures)
-   [命令创建签名](#command-signature-creation)
    -   [任何参数更改](#no-argument-changes)
    -   [根常量和顶点缓冲区](#root-constants-and-vertex-buffers)
-   [相关的主题](#related-topics)

## <a name="command-signatures"></a>命令签名

命令签名对象 ([**ID3D12CommandSignature**](https://msdn.microsoft.com/en-us/library/Dn891446(v=VS.85).aspx)) 支持使用应用程序指定间接绘图，特别是设置以下：

-   间接参数缓冲区格式。
-   将使用的命令类型 (从[ **ID3D12GraphicsCommandList** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12graphicscommandlist)方法[ **DrawInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawinstanced)， [ **DrawIndexedInstanced**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawindexedinstanced)，或[**调度**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch))。
-   这将更改集将继承与每个命令调用资源绑定集。

在启动时，应用程序创建一小部分**命令签名**。 在运行时，应用程序填充具有命令 （通过任何方式应用开发人员选择） 的缓冲区。 可以选择性地包含状态的命令，将顶点缓冲区视图、 索引缓冲区视图、 根常量和根描述符 (原始或结构化 SRV/UAV/CBVs)。 这些参数布局不是特定于硬件，因此应用程序可以直接生成缓冲区。 命令签名将剩余状态继承命令列表。 然后应用程序调用[ **ExecuteIndirect** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-executeindirect)以指示 GPU 来解释所定义的特定命令签名的格式根据间接参数缓冲区的内容。

如果命令签名更改任何根自变量，该对象存储命令签名内为根签名的子集。

请注意，当执行完成时，没有命令签名状态泄漏返回到命令列表。

例如，假设应用程序开发人员想要为间接参数缓冲区中指定的每个绘图调用的唯一根常量。 应用程序将创建使间接参数缓冲区指定以下参数，每个绘图调用的命令签名：

-   一个根常量的值。
-   绘制自变量 （顶点计数、 实例计数等）。

应用程序生成的间接参数缓冲区将包含固定大小的记录的数组。 每个结构都对应一个绘图调用。 每个结构包含绘制自变量和根常量的值。 在单独的 GPU 可见缓冲区中指定的绘图调用数。

由应用生成的示例命令缓冲区如下所示：

![命令缓冲区格式](images/indirect-drawing-command-buffer.png)

## <a name="indirect-argument-buffer-structures"></a>间接参数缓冲区结构

以下结构定义的方式特定自变量显示在间接参数缓冲区中。 这些结构中任何 D3D12 API 不会出现。 写入 （使用 CPU 或 GPU 中） 的间接参数缓冲区时，应用程序使用这些定义：

-   [**D3D12\_绘制\_参数**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_draw_arguments)
-   [**D3D12\_绘制\_索引\_参数**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_draw_indexed_arguments)
-   [**D3D12\_调度\_参数**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_dispatch_arguments)
-   [**D3D12\_VERTEX\_BUFFER\_VIEW**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_vertex_buffer_view)
-   [**D3D12\_INDEX\_BUFFER\_VIEW**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_index_buffer_view)
-   D3D12\_GPU\_虚拟\_地址 （typedef 的 UINT64 的同义词）。
-   [**D3D12\_常量\_缓冲区\_视图**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_constant_buffer_view_desc)

## <a name="command-signature-creation"></a>命令创建签名

若要创建命令签名，请使用以下 API 各项：

-   [**ID3D12Device::CreateCommandSignature** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandsignature) (输出[ **ID3D12CommandSignature**](https://msdn.microsoft.com/en-us/library/Dn891446(v=VS.85).aspx))
-   [**D3D12\_间接\_自变量\_类型**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_indirect_argument_type)
-   [**D3D12\_间接\_自变量\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_indirect_argument_desc)
-   [**D3D12\_命令\_签名\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_command_signature_desc)

间接参数缓冲区中的参数的顺序定义完全匹配的参数中指定的顺序*pArguments*的参数[ **D3D12\_命令\_签名\_DESC**](/windows/desktop/api/D3D12/ns-d3d12-d3d12_command_signature_desc)。 一个参数的所有绘制 （图形）/紧密打包调度 （计算） 调用间接参数缓冲区中。 但是，允许应用程序指定任意字节 stride 之间绘制/调度间接参数缓冲区中的命令。

当且仅当命令签名更改其中一个根自变量，则必须指定根签名。

对于根 CBV/SRV/UAV，将大小是以字节为单位指定的应用程序。 调试层将验证该地址的以下限制：

-   CBV – 地址必须是 256 个字节的倍数。
-   原始 SRV/UAV – 地址必须是 4 个字节的倍数。
-   结构化的 SRV/UAV – 地址必须是结构字节 stride （在着色器中声明） 的倍数。

给定的命令签名是绘制或计算命令签名。 如果命令签名包含绘制操作，它是图形命令签名。 否则为命令签名必须包含一个调度的操作，并且它是计算命令签名。

以下部分显示了一些示例命令签名。

### <a name="no-argument-changes"></a>任何参数更改

在此示例中，间接参数缓冲区的应用程序生成包含 36 个字节结构的数组。 每个结构仅包含五个参数传递给[ **DrawIndexedInstanced** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-drawindexedinstanced) （加上填充）。

若要创建命令签名说明的代码如下所示：

``` syntax
D3D12_INDIRECT_ARGUMENT_DESC Args[1];
Args[0].Type = D3D12_INDIRECT_ARGUMENT_TYPE_DRAW_INDEXED;

D3D12_COMMAND_SIGNATURE_DESC ProgramDesc;
ProgramDesc.ByteStride = 36;
ProgramDesc.ArgumentCount = 1;
ProgramDesc.pArguments = Args;
```

间接参数缓冲区中的单个结构的布局是：



| 字节 | 描述           |
|-------|-----------------------|
| 0:3   | IndexCountPerInstance |
| 4:7   | InstanceCount         |
| 8:11  | StartIndexLocation    |
| 12:15 | BaseVertexLocation    |
| 16:19 | StartInstanceLocation |
| 20:35 | 填充               |



 

### <a name="root-constants-and-vertex-buffers"></a>根常量和顶点缓冲区

在此示例中，间接参数缓冲区中的每个结构更改两个根常量、 更改绑定，一个顶点缓冲区和执行一个绘制非索引操作。 没有结构之间无需进行填充。

若要创建命令签名说明的代码是：

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

以下是间接的参数缓冲区中的单个结构的布局：



| 字节 | 描述                     |
|-------|---------------------------------|
| 0:3   | 根参数的数据编制索引 2 |
| 4:7   | 根参数的数据编制索引 6 |
| 8:15  | 虚拟地址的 VB （64 位）  |
| 16:19 | VB 大小                         |
| 20:23 | VB stride                       |
| 24:27 | VertexCountPerInstance          |
| 28:31 | InstanceCount                   |
| 32:35 | StartVertexLocation             |
| 36:39 | StartInstanceLocation           |



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[DirectX 高级学习视频教程：执行间接和消除异步 GPU](https://www.youtube.com/watch?v=fKD-VKJeeds)
</dt> <dt>

[间接绘图和 GPU 剔除： 代码演练](indirect-drawing-and-gpu-culling-.md)
</dt> <dt>

[呈现](rendering.md)
</dt> </dl>

 

 




