---
title: Direct3D 12 术语表
description: 这些条款是独特的 Direct3D 12。
Robots: noindex, nofollow
ms.assetid: 46B0F055-7E4F-4F8D-9915-3D195FD695B7
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 1823b2964393f9e30ac48a9aa7ef1e470e11f439
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224275"
---
# <a name="direct3d-12-glossary"></a>Direct3D 12 术语表

这些条款是独特的 Direct3D 12。

<dl> <dt>

<span id="direct3d12.directx_12_glossary_binding"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_BINDING"></span>**binding**
</dt> <dd>

附加到图形管道的内存的过程。 资源绑定，例如，涉及到用于呈现对象的某个资源例如到管道中，纹理绑定。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_buffer"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_BUFFER"></span>**buffer**
</dt> <dd>

一种是连续内存分配的同义词，D3D 资源类型。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_bundle"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_BUNDLE"></span>**bundle**
</dt> <dd>

图形处理单元 (GPU) 可以仅直接执行的命令缓冲区通过*直接的命令列表*。 捆绑包继承所有 GPU 状态 (当前设置除外*管道状态对象*和基元拓扑)。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_command_allocator"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_COMMAND_ALLOCATOR"></span>**命令分配器**
</dt> <dd>

基础内存分配命令存储在 GPU 中。 命令分配器对象所应用于这两*定向命令列出*并*捆绑包*。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_command_list"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_COMMAND_LIST"></span>**命令列表**
</dt> <dd>

命令列表对应于一系列 GPU 执行的命令。 其中包括等命令来设置状态、 绘制、 清除和复制。 D3D12 命令列表接口是明显不同于 D3D11 命令列表接口。 D3D12 命令列表接口包含类似于 D3D11 设备上下文呈现 Api 的 Api。

D3D12 命令列表不会不映射或取消映射资源、 更改磁贴映射、 调整磁贴池的大小、 获取查询数据，也不会它曾经隐式命令提交至 GPU 进行执行。

与 D3D11 延迟的上下文不同 D3D12 命令列表仅支持两个级别的间接寻址。 一个*直接的命令列表*对应于 GPU 可以执行的命令缓冲区。 一个*捆绑*可以只能直接执行通过直接的命令列表。

直接的命令列表不会继承任何 GPU 状态。 捆绑包继承 （除外的当前设置管道状态对象和基元拓扑） 的所有 GPU 状态。

设置命令列表的内存*命令分配器*。 命令列表的目的是它们可以提交到 GPU 以单个呈现请求的形式。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_command_queue"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_COMMAND_QUEUE"></span>**命令队列**
</dt> <dd>

队列*命令列出*GPU 执行连续的。 应用程序必须显式提交*命令列出*到执行的命令队列。 通常有三个命令队列：3D 图形、 计算和对应的 3D 图形管道、 计算引擎和一个或多个复制引擎，在 GPU 上的复制。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_conservative_rasterization"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_CONSERVATIVE_RASTERIZATION"></span>**传统型光栅化**
</dt> <dd>

传统型光栅化是操作的一种模式的 Direct3D 图形管道的光栅器阶段。 它禁用标准示例基于光栅化，并将改为使用栅格化涵盖的基元由任意数量的像素。 一个重要区别是，而任何覆盖率根本生成光栅化像素，该覆盖率不能有以下特征的硬件，以便覆盖范围始终显示二进制向像素着色器： 完整范围或未覆盖。 它是从左到像素着色器代码，以经过分析之后决定确定实际的覆盖率。

传统型光栅化可以帮助解决冲突所遇到的问题，并按检测、 装箱和消除，其中对像素的颜色是更特定，可以消除了边缘情况的阻挡物。 请参阅[传统型光栅化](conservative-rasterization.md)。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_cbv"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_CBV"></span>**常量缓冲区视图 (CBV)**
</dt> <dd>

常量缓冲区包含着色器常量数据，例如照相机视图、 投影矩阵和世界矩阵。 "常量缓冲区视图"是缓冲区的由图形管道所示的特定格式的视图。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_default_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DEFAULT_HEAP"></span>**默认堆**
</dt> <dd>

侧重于支持持久性 GPU 资源类型，包括 GPU 写入资源的用户模式下堆。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_descriptor"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DESCRIPTOR"></span>**descriptor**
</dt> <dd>

描述符是 D3D12 中的单个资源绑定的主计价单位。 描述符是完全描述一个对象到 GPU GPU 特定格式的数据相对较小块。 有许多不同类型的描述符：着色器资源视图 (SRVs)、 无序访问视图 (Uav)、 常量缓冲区视图 (CBVs) 和取样器是一些示例。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_descriptor_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DESCRIPTOR_HEAP"></span>**描述符堆**
</dt> <dd>

描述符堆是一系列连续分配的描述符，每个描述符的一个资源分配。 描述符堆的主要点是以包含所需的存储对象的描述符规范作为呈现尽可能的窗口的大类型对于该着色器引用的内存分配的大容量 (理想情况下的呈现整个帧或详细介绍）。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_descriptor_table"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DESCRIPTOR_TABLE"></span>**描述符表**
</dt> <dd>

描述符表在逻辑上就描述符的数组。 每个描述符表存储一个或多个类型，包括 SRVs、 UAVe、 CBVs 和取样器的描述符。 描述符表不是内存的分配，只需是一种偏移量和长度描述符堆。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_direct_command_list"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_DIRECT_COMMAND_LIST"></span>**直接的命令列表**
</dt> <dd>

GPU 可以执行命令缓冲区。 直接的命令列表不会继承任何 GPU 状态。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_fence"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_FENCE"></span>**fence**
</dt> <dd>

同步的 GPU 和 CPU 的机制。 GPU 和 CPU，可指示在防护，等待实际上等待另一个处理器来保持同步。 请参阅[同步和多引擎](user-mode-heap-synchronization.md)。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_hazard"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_HAZARD"></span>**危险，跟踪的危险**
</dt> <dd>

当资源用于一种用途，并应用要重复使用它用于其他用途，具有风险时发生。 若要再次使用该资源，中间缓存需要刷新或失效，要求将需要与第二次使用，保持一致的压缩和资源应处于所需的状态，以避免读取资源后，写入到，对于预期目的无效。

维护资源并避免这些同步问题的过程被称为危险跟踪。 如果跟踪驱动程序没有危险，则负责此应用程序。 在大多数早期版本的 DirectX，危险跟踪已处理由驱动程序。 若要提高性能，而无需跟踪的危险的方法中提供了 DirectX 12。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_hlsl"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_HLSL"></span>**高级着色器语言 (HLSL)**
</dt> <dd>

计算机类似的语言，但在许多方面与 C，用于编写着色器代码不同。 顶点、 像素、 geometry、 计算、 域和外壳着色器是所有使用编写 HLSL。 编译器将使用 GPU 的二进制格式转换为 HLSL 源代码。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_multiengine"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_MULTIENGINE"></span>**multiengine**
</dt> <dd>

不同实例和在单个 GPU 引擎类型。 引擎的类型包括： 图形中，计算，并复制。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_multigpu"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_MULTIGPU"></span>**MultiGPU**
</dt> <dd>

硬件配置的多个图形适配器上。 单独的适配器有时被称为节点。 拥有多个 Gpu 可以使其使用 CPU，以及每个其他要复杂得多比使用单个 GPU 执行同步的任务。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_pipeline_state_object"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_PIPELINE_STATE_OBJECT"></span>**管道状态对象 (PSO)**
</dt> <dd>

GPU 状态的重要部分。 此状态包括所有当前设置的着色器和某些固定函数状态对象。 更改包含在管道对象的状态的唯一方法是更改当前绑定的管道对象。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_predication"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_PREDICATION"></span>**predication**
</dt> <dd>

断言而是一项功能，使 GPU 而不是 CPU 来确定不绘制、 复制或调度对象。 例如，如果对象的边界框完全封闭的由另一个对象或透视减少了该对象为一个像素的大小小于，可能没有必要再尝试在所有绘制隐藏的对象。 请参阅[断言而](predication.md)。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_rov"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_ROV"></span>**光栅器顺序视图 (ROV)**
</dt> <dd>

标准图形管道可能有故障正确组合在一起包含透明的多个纹理。 对象，如网络界定、 烟雾、 焰火、 vegetation 和彩色的玻璃使用透明度，若要获取所需的效果。 根据彼此 （冒烟前面玻璃构建包含 vegetation，例如前面 fence） 包含透明的多个纹理时出现问题。 光栅器有序视图 (ROVs) 启用基础的顺序无关的透明度 (OIT) 算法，若要使用的硬件功能来尝试解决的透明度顺序正确。 透明度由像素着色器处理。

光栅器有序视图 (ROVs) 允许像素着色器代码将标记与更改 Uav 的常规要求图形管道结果的顺序的声明的无序访问视图 (UAV) 绑定。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_readback_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_READBACK_HEAP"></span>**readback 堆**
</dt> <dd>

用户模式下堆注重从 GPU 回 CPU 的数据传输的。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_resource"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_RESOURCE"></span>**resource**
</dt> <dd>

提供到管道的数据，并定义在您的场景要呈现的实体。 可以从游戏媒体加载或在运行时动态创建的资源。 通常情况下，资源包含纹理数据、 顶点数据和着色器数据。 大多数 Direct3D 应用程序创建和销毁其使用期限整个广泛的资源。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_resource_barrier"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_RESOURCE_BARRIER"></span>**resource barrier**
</dt> <dd>

资源屏障会通知驱动程序，对单个资源的多个访问同步可能必需的例如，读取和写入相同的纹理。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_resource_binding"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_RESOURCE_BINDING"></span>**资源绑定**
</dt> <dd>

资源绑定是与图形管道使管道的着色器处理正确的资源的链接资源 （纹理、 顶点缓冲区、 索引缓冲区等） 的过程。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_resource_heaps"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_RESOURCE_HEAPS"></span>**资源堆**
</dt> <dd>

资源堆是堆的内存缓冲区设置到某个位置来保存资源才会传输到和从 GPU 的一个通用术语。 传输到 GPU 上传的堆，需要从 CPU 到 GPU 需要 Readback 堆，并且以维护多个呈现帧的 GPU 持久堆称为默认堆。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_root_signature"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_ROOT_SIGNATURE"></span>**根签名**
</dt> <dd>

根签名定义要绑定到图形或计算，管道的所有资源。 根签名配置应用程序和链接命令将列出到着色器通常需要的资源，没有一个图形和一个计算每个应用的根签名。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_sampler"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_SAMPLER"></span>**sampler**
</dt> <dd>

采样器是从纹理中读取的代码。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_srv"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_SRV"></span>**着色器资源视图 (SRV)**
</dt> <dd>

用于查看着色器资源，如纹理中的数据的特定格式的方法。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_static_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_STATIC_HEAP"></span>**静态堆**
</dt> <dd>

侧重于用户模式下堆的多个 GPU 的只读资源通常使用在同一时间并且不经常更改。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_swap_chain"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_SWAP_CHAIN"></span>**交换链**
</dt> <dd>

交换链来控制后台缓冲区旋转，图形动画的基础。 由低级别 API 集 DXGI 交换链进行处理 (请参阅[DXGI 概述](https://msdn.microsoft.com/library/windows/desktop/bb205075))。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_swizzle"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_SWIZZLE"></span>**swizzle**
</dt> <dd>

查找在内存中多维数据，以便数据附近维数往往具有附近的地址的一种方法。 具体而言，一个行的所有数据不位于连续的下一步的行的数据之前。 "参数化 Swizzle"介绍了一种描述 swizzle 模式标准化的方法。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_static_texture"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_STATIC_TEXTURE"></span>**texture**
</dt> <dd>

D3D 资源，是多维的还具有针对从 GPU 的多维度访问权限进行了优化的内存布局的类型。 纹理通常包含一个表面上之前照明, 呈现所需的原始映像和混合采用放置，但可以包含其他形式的数据，例如颜色渐变，并引用颜色。 Direct3D 12 支持一个、 两个和三个维的纹理。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_tile"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_TILE"></span>**tile**
</dt> <dd>

视频内存，类似于一个 CPU/的系统内存页的页。 磁贴表示法有助于将 GPU 虚拟内存子系统的 CPU 虚拟内存子系统与区分开来。 Gpu 提供类似的虚拟内存功能，作为系统虚拟内存。 某些 Gpu 具有共享虚拟内存功能，允许的 CPU 和 GPU 的虚拟内存子系统的一些页面共享。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_tiled_resources"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_TILED_RESOURCES"></span>**平铺的资源**
</dt> <dd>

平铺的资源是所必需的因此更少 GPU 内存就会浪费存储的应用程序知道将不能访问的图面区域和硬件可以了解如何在相邻的图块上进行筛选。 平铺的资源是大型的逻辑资源，但需要少量的物理内存。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_uav"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_UAV"></span>**无序的访问视图 (UAV)**
</dt> <dd>

转换为资源 （其中包括缓冲区、 纹理和纹理数组-而无需多重采样），无序的访问视图允许从多个线程未按时间顺序排序的读/写访问。 这意味着，此资源类型可以是读/写同时由多个线程而不会生成内存冲突。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_upload_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_UPLOAD_HEAP"></span>**上传堆**
</dt> <dd>

用户模式下堆注重从 CPU 到 GPU 的数据传输的。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_user_mode_heap"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_USER_MODE_HEAP"></span>**用户模式下堆**
</dt> <dd>

回收而无需任何内核组件意识的大型的连续未分配的内存分配的集合： 分配和析构方法不要调用内核分配和析构的方法在稳定状态。 上传、 Readback 和默认堆堆是用户模式的变体。

</dd> <dt>

<span id="direct3d12.directx_12_glossary_volume_tiled_resources"></span><span id="DIRECT3D12.DIRECTX_12_GLOSSARY_VOLUME_TILED_RESOURCES"></span>**平铺的卷资源**
</dt> <dd>

三维[平铺资源](https://docs.microsoft.com/windows)。

</dd> </dl>

 

 




