---
title: Direct3D 12 术语表
description: 这些术语是 Direct3D 12 特有的。
Robots: noindex, nofollow
ms.assetid: 46B0F055-7E4F-4F8D-9915-3D195FD695B7
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 8f54c0c1a9780ba35f3ea5d8fa0139e81ebf79ea
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71005968"
---
# <a name="direct3d-12-glossary"></a>Direct3D 12 术语表

这些术语是 Direct3D 12 特有的。

<dl> <dt>

<span id="direct3d12.directx_12_glossary_binding"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_BINDING"></span>**绑定**
</dt> <dd>

将内存附加到图形管道的过程。 例如，资源绑定涉及到将纹理等资源绑定到管道，以用于渲染对象。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_buffer"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_BUFFER"></span>**buffer**
</dt> <dd>

与连续内存分配同义的一种 D3D 资源。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_bundle"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_BUNDLE"></span>**bundle**
</dt> <dd>

一个命令缓冲区，图形处理单元 (GPU) 只能直接通过直接命令列表执行该缓冲区。 捆绑继承整个 GPU 状态（当前设置的管道状态对象和基元拓扑除外）。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_command_allocator"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_COMMAND_ALLOCATOR"></span>**命令分配器**
</dt> <dd>

用于存储 GPU 命令的基础内存分配。 命令分配器对象将应用到直接命令列表和捆绑。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_command_list"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_COMMAND_LIST"></span>**命令列表**
</dt> <dd>

命令列表对应于 GPU 执行的一组命令。 这包括设置状态、绘制、清除和复制等命令。 D3D12 命令列表接口明显不同于 D3D11 命令列表接口。 D3D12 命令列表接口包含类似于 D3D11 设备上下文渲染 API 的 API。

D3D12 命令列表不会映射或取消映射资源、更改图块映射、调整图块池的大小、获取查询数据，也不会将命令隐式提交到 GPU 供执行。

与 D3D11 延迟上下文不同，D3D12 命令列表仅支持两个间接性级别。 直接命令列表对应于 GPU 可以执行的命令缓冲区。 只能通过直接命令列表直接执行捆绑。

直接命令列表不会继承任何 GPU 状态。 捆绑继承整个 GPU 状态（当前设置的管道状态对象和基元拓扑除外）。

命令列表的内存由命令分配器设置。 命令列表可以作为单个渲染请求提交到 GPU。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_command_queue"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_COMMAND_QUEUE"></span>**命令队列**
</dt> <dd>

GPU 连续执行的命令列表队列。 应用程序必须显式将命令列表提交到命令队列以供执行。 通常有三个命令队列：3D 图形、计算和复制，分别对应于 GPU 上的 3D 图形管道、计算引擎及一个或多个复制引擎。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_conservative_rasterization"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_CONSERVATIVE_RASTERIZATION"></span>**保守光栅化**
</dt> <dd>

保守光栅化是 Direct3D 图形管道光栅器阶段的一种工作模式。 它会禁用标准的基于样本的光栅化，并改为光栅化按基元、按数量覆盖的像素。 一个重要区别在于，任何覆盖度可以完全生成光栅化的像素，但不能按硬件特征化该覆盖度，因此，在像素着色器看来，覆盖度始终是二进制的：完全覆盖或未覆盖。 由像素着色器代码负责通过分析来确定实际覆盖度。

保守光栅化可帮助解决碰撞和冲击检测、装箱和遮挡剔除等问题，其中的像素颜色更为确定，且可以消除极端情况。 请参阅[保守光栅化](conservative-rasterization.md)。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_cbv"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_CBV"></span>**常量缓冲区视图 (CBV)**
</dt> <dd>

常量缓冲区包含着色器常量数据，例如相机视图、投影矩阵和世界矩阵。 “常量缓冲区视图”是图形管道看到的格式特定的缓冲区视图。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_default_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DEFAULT_HEAP"></span>**默认堆**
</dt> <dd>

侧重于支持永久性 GPU 资源类型（包括 GPU 写入的资源）的用户模式堆。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_descriptor"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DESCRIPTOR"></span>**描述符**
</dt> <dd>

描述符是 D3D12 中单个资源的主要绑定单元。 描述符是一个相对较小的数据块，以 GPU 特定的格式完全描述提交到 GPU 的对象。 有多种不同类型的描述符：着色器资源视图 (SRV)、无序访问视图 (UAV)、常量缓冲区视图 (CBV) 和采样器就是其中的几个例子。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_descriptor_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DESCRIPTOR_HEAP"></span>**描述符堆**
</dt> <dd>

描述符堆是描述符的连续分配的集合，每个描述符有一个分配。 描述符堆的要点是包含所需的批量内存分配，用于存储着色器在尽可能大的渲染窗口（最好是在整个渲染帧或更大的窗口）中引用的对象类型的描述符规范。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_descriptor_table"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DESCRIPTOR_TABLE"></span>**描述符表**
</dt> <dd>

描述符表在逻辑上是描述符数组。 每个描述符表存储一种或多种类型的描述符，包括 SRV，UAV，CBV 和采样器。 描述符表不是内存分配；它只是描述符堆中的偏移量和长度。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_direct_command_list"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DIRECT_COMMAND_LIST"></span>**直接命令列表**
</dt> <dd>

GPU 可以执行的命令缓冲区。 直接命令列表不会继承任何 GPU 状态。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_fence"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_FENCE"></span>**fence**
</dt> <dd>

用于同步 GPU 和 CPU 的机制。 可以指示 GPU 和 CPU 等待围栏完成，这实际上是等待另一个处理器跟上进度。 请参阅[同步和多引擎](user-mode-heap-synchronization.md)。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_hazard"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_HAZARD"></span>**风险和风险跟踪**
</dt> <dd>

当某个资源已用于一种目的，而应用打算将它重用于另一种目的时，将会出现风险。 若要再次使用该资源，需要刷新中间缓存或使中间缓存失效，压缩要求需与第二种用途保持一致，并且该资源应处于所需的状态，以免在写入该资源并根据目标用途使其失效后读取该资源。

维护资源并避免这些同步问题的过程称为风险跟踪。 如果驱动程序不提供风险跟踪，则应用将负责执行此操作。 在大多数早期版本的 DirectX 中，风险跟踪由驱动程序处理。 为了提高性能，DirectX 12 中提供了不使用风险跟踪的方法。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_hlsl"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_HLSL"></span>**高级着色器语言 (HLSL)**
</dt> <dd>

一种计算机语言，它在许多方面与 C 类似但又有差别，用于编写着色器代码。 顶点、像素、几何体、计算、域和外壳着色器都是使用 HLSL 编写的。 编译器将 HLSL 源转换为二进制格式，以供 GPU 使用。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_multiengine"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_MULTIENGINE"></span>**multiengine**
</dt> <dd>

单个 GPU 中引擎的不同实例和类型。 引擎类型包括：图形、计算和复制。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_multigpu"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_MULTIGPU"></span>**MultiGPU**
</dt> <dd>

包含多个图形适配器的硬件配置。 独立的适配器有时称为节点。 与使用单个 GPU 相比，使用多个 GPU 会明显增大 GPU 与 CPU 同步以及 GPU 之间同步的任务的复杂性。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_pipeline_state_object"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_PIPELINE_STATE_OBJECT"></span>**管道状态对象 (PSO)**
</dt> <dd>

GPU 状态的重要部分。 此状态包括所有当前设置的着色器和某些固定函数状态对象。 更改管道对象中包含的状态的唯一方法是更改当前绑定的管道对象。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_predication"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_PREDICATION"></span>**断言**
</dt> <dd>

断言功能使 GPU（而不是 CPU）能够决定不绘制、复制或调度对象。 例如，如果某个对象的边界框完全由另一个对象封闭，或透视图将该对象缩减为小于一个像素的大小，则可能根本没有必要尝试绘制隐藏的对象。 请参阅[断言](predication.md)。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_rov"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_ROV"></span>**光栅器有序视图 (ROV)**
</dt> <dd>

标准图形管道在正确组合包含透明度的多个纹理时可能会遇到困难。 铁丝网、烟雾、火、植物和彩色玻璃等物体使用透明度来达到预期效果。 当包含透明度的多个纹理彼此成一条线时（例如，包含植被的玻璃建筑物前面有栅栏，而栅栏前面有烟雾），将会出现问题。 光栅器有序视图 (ROV) 使得基础顺序独立透明度 (OIT) 算法能够使用硬件功能来尝试正确解析透明度顺序。 透明度由像素着色器处理。

像素着色器代码可通过光栅器有序视图 (ROV) 使用声明来标记无序访问视图 (UAV) 绑定，该声明改变了 UAV 图形管道结果顺序的正常要求。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_readback_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_READBACK_HEAP"></span>**读回堆**
</dt> <dd>

侧重于将 GPU 中的数据传输回到 CPU 的用户模式。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_resource"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_RESOURCE"></span>**资源**
</dt> <dd>

向管道提供数据，并定义要在场景中渲染的内容的实体。 资源可以从游戏媒体加载或在运行时动态创建。 通常，资源包括纹理数据、顶点数据和着色器数据。 大多数 Direct3D 应用程序在其整个生存期内，会大量地创建和销毁资源。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_resource_barrier"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_RESOURCE_BARRIER"></span>**资源屏障**
</dt> <dd>

资源屏障通知驱动程序可能需要同步对单个资源的多次访问，例如读取和写入相同的纹理。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_resource_binding"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_RESOURCE_BINDING"></span>**资源绑定**
</dt> <dd>

资源绑定是将资源（纹理、顶点缓冲区、索引缓冲区等）链接到图形管道，使管道着色器能够处理正确的资源的过程。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_resource_heaps"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_RESOURCE_HEAPS"></span>**资源堆**
</dt> <dd>

资源堆是一个概括性的术语，表示在内存缓冲区中留出的、用于保存传入和传出 GPU 的资源的堆。 传入 GPU 需要使用上传堆，从 GPU 传入 CPU 需要使用读回堆，GPU 用来保留多个渲染帧的永久性堆称为默认堆。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_root_signature"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_ROOT_SIGNATURE"></span>**根签名**
</dt> <dd>

根签名定义要绑定到图形、计算或管道的所有资源。 根签名由应用配置，可将命令列表链接到着色器所需的资源。通常，每个应用有一个图形根签名和一个计算根签名。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_sampler"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_SAMPLER"></span>**sampler**
</dt> <dd>

采样器是从纹理读取数据的代码。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_srv"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_SRV"></span>**着色器资源视图 (SRV)**
</dt> <dd>

用于在着色器资源（例如纹理）中查找数据的格式特定的方式。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_static_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_STATIC_HEAP"></span>**静态堆**
</dt> <dd>

一种用户模式堆，侧重于通常会同时使用的且不经常更改的多个 GPU 只读资源。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_swap_chain"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_SWAP_CHAIN"></span>**交换链**
</dt> <dd>

交换链控制反向缓冲区轮转，构成图形动画的基础。 交换链由低级 API 设置的 DXGI 处理（请参阅 [DXGI 概述](https://docs.microsoft.com/windows/desktop/direct3ddxgi/d3d10-graphics-programming-guide-dxgi)）。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_swizzle"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_SWIZZLE"></span>**swizzle**
</dt> <dd>

一种在内存中定位多维数据，使附近维数的数据倾向于使用附近地址的一种技术。 具体而言，一行的所有数据不会在下一行的数据之前连续定位。 “参数化重排”描述了解释重排模式的标准化方法。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_static_texture"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_STATIC_TEXTURE"></span>**纹理**
</dt> <dd>

一种多维 D3D 资源，它采用优化的内存布局，可方便从 GPU 进行多维访问。 纹理通常包含在发生照明和混合之前，在图面上进行渲染所需的原始图像，但也可以包含其他形式的数据，例如颜色渐变和引用颜色。 Direct3D 12 支持一维、二维和三维纹理。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_tile"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_TILE"></span>**图块**
</dt> <dd>

一个视频内存页面，类似于内存的 CPU/系统页面。 图块表示法有助于将 GPU 虚拟内存子系统与 CPU 虚拟内存子系统区分开来。 GPU 提供与系统虚拟内存类似的虚拟内存功能。 某些 GPU 提供共享虚拟内存功能，可将虚拟内存子系统的某些页面与 CPU 和 GPU 共享。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_tiled_resources"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_TILED_RESOURCES"></span>**图块化资源**
</dt> <dd>

使用图块化资源可以减少 GPU 内存的浪费，这样就不必存储应用程序知道不会访问的图面区域，并且硬件知道如何对相邻的图块进行筛选。 图块化资源是较大的逻辑资源，但它们所需的物理内存量较少。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_uav"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_UAV"></span>**无序访问视图 (UAV)**
</dt> <dd>

使用资源（包括缓冲区、纹理和纹理数组 - 不包括多重采样）的无序访问视图可通过多个线程进行临时性的无序读/写访问。 这意味着此资源类型可由多个线程同时读/写，且不会产生内存冲突。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_upload_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_UPLOAD_HEAP"></span>**上传堆**
</dt> <dd>

侧重于将 CPU 中的数据传输到 GPU 的用户模式。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_user_mode_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_USER_MODE_HEAP"></span>**用户模式堆**
</dt> <dd>

在不会由任何内核组件察觉到的情况下回收的大型连续内存分配的集合：在稳定状态下，分配和销毁方法不会调用内核分配和销毁方法。 上传、读回和默认堆是用户模式堆的变体。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_volume_tiled_resources"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_VOLUME_TILED_RESOURCES"></span>**立体图块化资源**
</dt> <dd>

三维[图块化资源](/windows)。

</dd> </dl>

 

 




