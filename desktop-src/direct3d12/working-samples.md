---
title: 有用的示例
description: 可供下载，其中显示了大量的 Direct3D 12 的功能的使用情况的工作示例。
ms.assetid: 4C4475D4-534F-484F-8D60-9ACEA09AC109
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 91101ae6799e463dc76ecbadf4670989752f9813
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224440"
---
# <a name="working-samples"></a>有用的示例

可供下载，其中显示了大量的 Direct3D 12 的功能的使用情况的工作示例。

## <a name="working-samples"></a>工作示例

可以从下载工作示例 （在 Visual Studio 2015 项目的窗体） [GitHub/Microsoft/DirectX 的图形的示例](https://github.com/Microsoft/DirectX-Graphics-Samples)。

> [!Note]  
> 在此位置提供的示例的确切列表而异示例添加和更新。

 



<table>
<thead>
<tr class="header">
<th>标题示例</th>
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
<td>HelloWorld 示例集包含以下简单的项目，以帮助你开始使用 Direct3D 12。<dl> 在准备呈现 Direct3D 12 内容创建一个窗口。<br />
呈现一个简单的三角形使用 Direct3D 12。<br />
演示用于呈现使用 Direct3D 12 的捆绑包的使用情况。<br />
演示如何使用常量缓冲区将数据传递到用于呈现 Direct3D 12 中的 GPU。<br />
演示如何将纹理应用于使用 Direct3D 12 的三角形。<br />
</dl></td>
<td>Y</td>
<td>Y</td>
<td><a href="creating-a-basic-direct3d-12-component">创建基本的 Direct3D 12 组件</a></td>
</tr>
<tr class="even">
<td>D3D12Bundles</td>
<td>演示帧缓冲和同步最佳做法以及呈现使用捆绑包的一个简单网格。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="odd">
<td>D3D12Multithreading</td>
<td>如何构建多线程支持应用程序的示例。</td>
<td>Y</td>
<td>N</td>

</tr>
<tr class="even">
<td>D3D12nBodyGravity</td>
<td>演示如何使用多引擎执行相同的 GPU 上的 3D 工作以及异步计算工作。</td>
<td>Y</td>
<td>Y</td>
<td><a href="multi-engine-n-body-gravity-simulation">多引擎 n 正文重力模拟</a></td>
</tr>
<tr class="odd">
<td>D3D12PredicationQueries</td>
<td>演示封闭剔除堆使用查询和断言而。</td>
<td>Y</td>
<td>Y</td>
<td><a href="predication-queries">断言而查询</a></td>
</tr>
<tr class="even">
<td>D3D12DynamicIndexing</td>
<td>演示 DirectX 12 和 HLSL 的动态索引编制的功能。</td>
<td>Y</td>
<td>Y</td>
<td><a href="dynamic-indexing-using-hlsl-5-1">动态索引使用 HLSL 5.1</a></td>
</tr>
<tr class="odd">
<td>D3D1211on12</td>
<td>演示 11on12 层的基本用法。 此示例呈现文本使用 D2D Direct3D 12 11on12 设备上使用 Direct3D 11 API。</td>
<td>Y</td>
<td>Y</td>
<td><a href="d2d-using-d3d11on12">D2D 使用 D3D11on12</a></td>
</tr>
<tr class="even">
<td>D3D12ExecuteIndirect</td>
<td>演示如何计算引擎消除与要仅呈现通过精选测试的对象的 execute 间接功能结合使用。</td>
<td>Y</td>
<td>Y</td>
<td><a href="indirect-drawing-and-gpu-culling-">间接绘制和 GPU 消除</a></td>
</tr>
<tr class="odd">
<td>D3D12PipelineStateCache</td>
<td>演示如何缓存管道状态对象 (PSO)。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="even">
<td>D3D12Fullscreen</td>
<td>演示如何处理窗口切换和重设窗口大小在 DirectX 12 的全屏幕。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="odd">
<td>D3D12HeterogeneousMultiadapter</td>
<td>演示如何共享之间使用共享的堆的多个异构 Gpu 的工作负荷。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="even">
<td>D3D12ReservedResources</td>
<td>演示如何将保留 （平铺） 资源。 在此示例中四带有包含完整的 mip 链的保留资源。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="odd">
<td>D3D12Residency</td>
<td>目的是堆管理 Direct3D 12 和已提交的资源，使用 Direct3D 11 从内存管理技术的集成的低成本解决方案。</td>
<td>Y</td>
<td>Y</td>

</tr>
<tr class="even">
<td>D3D12SmallResources</td>
<td>演示如何使用小型放置资源，其中显示内存节省获得使用通过 （使用 64k 的对齐方式） 的已提交和保留资源上放置资源 （包括 4 千字节的对齐方式）。</td>
<td>Y</td>
<td>Y</td>

</tr>
</tbody>
</table>



 

## <a name="related-topics"></a>相关主题

<dl> <dt>

[Direct3D 12 的编程指南](directx-12-programming-guide.md)
</dt> <dt>

[D3D12 代码演练](d3d12-code-walk-throughs.md)
</dt> </dl>

 

 




