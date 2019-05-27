---
title: Direct3D 12 编程环境设置
description: 介绍安装、 工具和支持构成了工作效率的 Direct3D 12 开发环境的库。
ms.assetid: B2288866-E95F-46B8-A7A1-19888F029C03
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 9fbdcb955c01997af4dd8224fabe0e555617c30c
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224326"
---
# <a name="direct3d-12-programming-environment-setup"></a>Direct3D 12 编程环境设置

介绍安装、 工具和支持构成了工作效率的 Direct3D 12 开发环境的库。

-   [开发环境](#development-environment)
-   [支持的语言](#supported-languages)
-   [帮助程序结构](#helper-structures)
-   [内存管理库](#memory-management-library)
-   [支持的工具和库](#supported-tools-and-libraries)
-   [示例](#samples)
-   [调试层](#debug-layer)
-   [教育视频](#educational-videos)
-   [相关的主题](#related-topics)

## <a name="development-environment"></a>开发环境

Direct3D 12 标头和库是 Windows 10 SDK 的一部分。 没有单独的下载或使用 Direct3D 12 中所需的安装。

安装 Windows 10 SDK 软件和 Visual Studio 2015 后，Direct3D 12 编程环境的安装程序已完成。 建议 visual Studio 2015，因为它将包括 D3D12 图形调试工具，但 Visual Studio 的早期版本将适用于程序开发。

若要使用[Direct3D 12 API](direct3d-12-reference.md)，包括 D3d12.h 并链接到 D3d12.lib，或查询直接在 D3d12.dll 中的入口点。

提供了以下标头和库。 静态库的位置取决于您的计算机运行的 Windows 10 版本 （32 位或 64 位）。



| 标头或库文件名称 | 描述                         | 安装位置      |
|-----------------------------|-------------------------------------|-----------------------|
| D3d12.h                     | Direct3D 12 API 标头              | %DXSDK\_DIR%\\Include |
| D3d12.lib                   | Direct3D 12 API 存根 （stub） 的静态库 | %DXSDK\_DIR%\\Lib     |
| D3d12.dll                   | 动态 Direct3D 12 API 库     | %WINDIR%\\System32    |
| D3d12SDKLayers.h            | Direct3D 12 调试标头            | %DXSDK\_DIR%\\Include |
| D3d12SDKLayers.dll          | 动态 Direct3D 12 调试库   | %WINDIR%\\System32    |



 

若要正确地包括标头文件，请使用类似于下面的语句：

`#include <%DXSDK_DIR%Include\d3d12.h>`

若要确定 %dxsdk 的绝对位置\_开发计算机上，在命令窗口中键入 DIR %:

`set dx`

## <a name="supported-languages"></a>支持的语言

C++是 Direct3D 12 开发方面，唯一支持的语言C#和其他.NET 语言不受支持。

## <a name="helper-structures"></a>帮助程序结构

有大量的帮助器结构，特别是，使其可轻松地初始化的 D3D12 结构数。 这些结构和某些实用工具函数，是 D3dx12.h 的标头中。 此标头是开放源，并可由开发人员根据需要修改-从[D3D12 帮助程序库](https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries/D3DX12)，并参考[帮助程序结构和函数对 D3D12](helper-structures-and-functions-for-d3d12.md)。

## <a name="memory-management-library"></a>内存管理库

内存管理帮助程序库是可供下载，可以将集成到你的应用更密切地匹配 D3D11 内存管理行为。 为 D3D11 样式管理库，它是最有效的应用程序仍在使用*提交资源*样式分配策略。 具体而言，应看到库借鉴，有助于您大部分方式回 D3D11 中内存的约束方案时的高性能内存管理 (例如，低端内存卡时，4k、 超高设置，等等)。 D3D12 Api 确实启用技巧，以帮助您更好的内存效率克服 D3D11，尽管这些技术可能具有挑战性且长时间才能实现。

请注意，此库是正在进行的工作可能随时间变化。 使用以下链接访问的库和示例。

-   [D3D12 驻留初学者库](https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries)

## <a name="supported-tools-and-libraries"></a>支持的工具和库

以下库可以全部在 Direct3D 12。



|                                                                                  |                                                                                                                                                                                                                                                                        |                                                                                                            |
|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|
| **库**                                                                      | **用途**                                                                                                                                                                                                                                                            | **文档**                                                                                          |
| [DirectX Tool Kit for DirectX 12](https://go.microsoft.com/fwlink/?LinkID=615561) | 大量编写 Direct3D 12 的帮助器类C++通用 Windows 平台 (UWP) 应用、 Windows 10 和 Xbox One 独占应用的 Win32 桌面应用程序的代码。                                                                         | [DirectX12TK wiki](https://github.com/Microsoft/DirectXTK12/wiki)                                          |
| [DirectXTex](https://go.microsoft.com/fwlink/?LinkId=248926)                      | 使用此过程进行读取和写入 DDS 文件，以及执行各种纹理内容处理操作，包括调整大小、 格式转换、 mip 贴图生成，用于 Direct3D 运行时纹理资源和正常映射到的高度图块压缩转换。 | [DirectXTex wiki](https://github.com/Microsoft/DirectXTex/wiki)                                            |
| [DirectXMesh](https://go.microsoft.com/fwlink/p/?linkid=324981)                   | 使用此执行各种 geometry 内容处理操作，包括生成 normals 和正切帧、 三角形相邻计算和顶点缓存优化。                                                                                | [DirectXMesh wiki](https://github.com/Microsoft/DirectXMesh/wiki)                                          |
| [DirectXMath](https://go.microsoft.com/fwlink/?LinkID=615560)                     | 大量的帮助程序类和方法，以支持向量、 标量、 矩阵、 四元数和许多其他数学运算。                                                                                                                               | [在 MSDN 上的 DirectXMath 文档](https://msdn.microsoft.com/library/windows/desktop/ee415571.aspx) |
| [UVAtlas](https://go.microsoft.com/fwlink/?LinkID=512686)                         | 将其用于创建和打包 isochart 纹理地图集。                                                                                                                                                                                                           | [UVAtlas wiki](https://github.com/Microsoft/UVAtlas/wiki)                                                  |



 

## <a name="samples"></a>示例

有关使用 D3D12 示例以及如何找到并运行它们的列表，请参阅[处理示例](working-samples.md)。

有关如何添加代码以启用特定功能的演练，请参阅[D3D12 代码演练](d3d12-code-walk-throughs.md)。

## <a name="debug-layer"></a>调试层

调试层提供 （如验证着色器链接和资源绑定、 验证参数的一致性，以及报告错误说明） 大量额外参数并进行一致性验证。

默认情况下，从 d3d12.h 包含支持调试层 D3D12SDKLayers.h，所需的标头。

当调试层列出了内存泄漏时，它将输出和友好名称的对象接口指针的列表。 默认友好名称是"<unnamed>"。 可以使用设置的友好名称[ **ID3D12Object::SetName** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12object-setname)方法。 通常情况下，应将这些调用编译进行生产版本。

我们建议你使用调试层来调试应用程序以确保它们是清晰的错误和警告。 调试层可帮助你编写 Direct3D 12 中的代码。 此外，当你使用调试层，因为你可以立即查看不明显呈现错误或在其源甚至黑色屏幕的原因的可以提高工作效率。 调试层提供了很多问题的警告。 例如：

-   忘记设置纹理，但在像素着色器中读取它。
-   输出深度，但不具有深度模具状态绑定。
-   INVALIDARG 纹理创建失败。

若要设置编译器定义 D3DCOMPILE\_调试以告知 HLSL 编译器包含到着色器 blob 的调试信息。

``` syntax
#define D3DCOMPILE_DEBUG 1
```

有关所有调试接口和方法的详细信息，请参阅[调试层引用](direct3d-12-sdklayers-reference.md)。

有关使用调试层的概述信息，请参阅[了解 D3D12 调试层](understanding-the-d3d12-debug-layer.md)。

## <a name="educational-videos"></a>教育视频

有许多 Direct3D 12 和 Windows 10 相关的视频，位于[DirectX 高级学习视频教程](https://www.youtube.com/channel/UCiaX2B8XiXR70jaN7NK-FpA)，包括图形调试工具上的视频和报告图形 bug。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> </dl>

 

 




