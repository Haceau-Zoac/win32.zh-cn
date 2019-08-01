---
title: 样例
description: 可下载样例，了解 Direct3D 12 的许多功能的用法。
ms.assetid: 4C4475D4-534F-484F-8D60-9ACEA09AC109
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: dbfccb70b8f575d2f868f2678483f04a8c764945
ms.sourcegitcommit: 05483887ef8fccd79543cc1b89495f156702465a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2019
ms.locfileid: "66296500"
---
# <a name="working-samples"></a>样例

可下载样例，了解 Direct3D 12 的许多功能的用法。

## <a name="working-samples"></a>样例

可以从 [GitHub/Microsoft/DirectX-Graphics-Samples](https://github.com/Microsoft/DirectX-Graphics-Samples) 下载样例（以 Visual Studio 2015 项目的形式）。

> [!Note]  
> 随着示例的添加和更新，此位置提供的示例具体列表将有所不同。

 



<table>
<thead>
<tr class="header">
<th>示例标题</th>
<th>描述</th>
<th>桌面设备</th>
<th>UWP</th>
<th>演练</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>HelloWorld<dl> HelloWindow<br />
HelloTriangle<br />
HelloBundles<br />
HelloConstBuffers<br />
HelloTexture<br />
</dl></td>
<td>HelloWorld 示例集包含以下简单项目，可帮助你开始使用 Direct3D 12。<dl> 创建一个窗口以准备渲染 Direct3D 12 内容。<br />
使用 Direct3D 12 渲染一个简单的三角形。<br />
演示使用 Direct3D 12 渲染捆绑包的用法。<br />
演示如何使用常量缓冲区将数据传递到用于在 Direct3D 12 中渲染的 GPU。<br />
演示如何使用 Direct3D 12 将纹理应用于三角形。<br />
</dl></td>
<td>Y</td>
<td>Y</td>
<td><a href="creating-a-basic-direct3d-12-component.md">创建基本的 Direct3D 12 组件</a></td>
</tr>
<tr class="even">
<td>D3D12Bundles</td>
<td>演示帧缓冲区和同步最佳做法，以及使用捆绑包渲染简单网格。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="odd">
<td>D3D12Multithreading</td>
<td>如何构建多线程应用程序的示例。</td>
<td>Y</td>
<td>N</td>

</tr>
<tr class="even">
<td>D3D12nBodyGravity</td>
<td>演示如何使用多引擎在同一个 GPU 上与 3D 工作一起执行异步计算工作。</td>
<td>Y</td>
<td>Y</td>
<td><a href="multi-engine-n-body-gravity-simulation.md">多引擎 n 体重力模拟</a></td>
</tr>
<tr class="odd">
<td>D3D12PredicationQueries</td>
<td>使用查询堆和预测来演示封闭剔除。</td>
<td>Y</td>
<td>Y</td>
<td><a href="predication-queries.md">预测查询</a></td>
</tr>
<tr class="even">
<td>D3D12DynamicIndexing</td>
<td>演示 DirectX 12 和 HLSL 的动态索引功能。</td>
<td>Y</td>
<td>Y</td>
<td><a href="dynamic-indexing-using-hlsl-5-1.md">使用 HLSL 5.1 的动态索引</a></td>
</tr>
<tr class="odd">
<td>D3D1211on12</td>
<td>演示 11on12 层的基本用法。 此示例在 Direct3D 12 11on12 设备上使用 Direct3D 11 API 通过 D2D 呈现文本。</td>
<td>Y</td>
<td>Y</td>
<td><a href="d2d-using-d3d11on12.md">使用 D3D11on12 的 D2D</a></td>
</tr>
<tr class="even">
<td>D3D12ExecuteIndirect</td>
<td>演示计算引擎剔除与执行间接功能相结合，仅呈现通过剔除测试的对象。</td>
<td>Y</td>
<td>Y</td>
<td><a href="indirect-drawing-and-gpu-culling-.md">间接绘制和 GPU 剔除</a></td>
</tr>
<tr class="odd">
<td>D3D12PipelineStateCache</td>
<td>演示管道状态对象 (PSO) 缓存。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="even">
<td>D3D12Fullscreen</td>
<td>演示如何在 DirectX 12 中处理全屏到窗口的转换和窗口大小调整。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="odd">
<td>D3D12HeterogeneousMultiadapter</td>
<td>演示如何使用共享堆在多个异构 GPU 之间共享工作负载。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="even">
<td>D3D12ReservedResources</td>
<td>演示保留（平铺）资源的使用。 在此示例中，四色使用包含完整 mip 链的保留资源进行纹理处理。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="odd">
<td>D3D12Residency</td>
<td>这是一种低集成成本的解决方案，使用 Direct3D 11 中的内存管理技术管理 Direct3D 12 堆和提交的资源。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="even">
<td>D3D12SmallResources</td>
<td>演示如何使用放置的小型资源并介绍使用放置的资源（4K 对齐）比使用提交和保留的资源（64K 对齐）所节省的潜在内存。</td>
<td>Y</td>
<td>Y</td>

</tr>
</tbody>
</table>



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 编程指南](directx-12-programming-guide.md)
</dt> <dt>

[D3D12 代码演练](d3d12-code-walk-throughs.md)
</dt> </dl>

 

 




