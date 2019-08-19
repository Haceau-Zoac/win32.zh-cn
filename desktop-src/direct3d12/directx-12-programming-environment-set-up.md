---
title: Direct3D 12 编程环境设置
description: 描述构成多产 Direct3D 12 开发环境的安装、工具和支持库。
ms.assetid: B2288866-E95F-46B8-A7A1-19888F029C03
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: dd9ffc356ef7d9adda6862a69e1f80481ea90102
ms.sourcegitcommit: af1bedc00f1f5da3673a5095be566c076f2a51aa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/16/2019
ms.locfileid: "69561161"
---
# <a name="direct3d-12-programming-environment-setup"></a>Direct3D 12 编程环境设置

描述构成多产 Direct3D 12 开发环境的安装、工具和支持库。

-   [开发环境](#development-environment)
-   [支持的语言](#supported-languages)
-   [帮助程序结构](#helper-structures)
-   [内存管理库](#memory-management-library)
-   [支持的工具和库](#supported-tools-and-libraries)
-   [示例](#samples)
-   [调试层](#debug-layer)
-   [教育视频](#educational-videos)
-   [相关主题](#related-topics)

## <a name="development-environment"></a>开发环境

Direct3D 12 标头和库是 Windows 10 SDK 的一部分。 使用 Direct3D 12 时无需单独下载或安装。

安装 Windows 10 SDK 软件和 Visual Studio 2015 后，已完成 Direct3D 12 编程环境的设置。 建议使用 Visual Studio 2015，因为它将包括 D3D12 图形调试工具，但早期版本的 Visual Studio 将适用于程序开发。

若要使用 [Direct3D 12 API](direct3d-12-reference.md)，请包括 D3d12.h 并链接到 D3d12.lib，或直接在 D3d12.dll 中查询入口点。

提供了以下标头和库。 静态库的位置取决于计算机上运行的 Windows 10 版本（32 位或 64 位）。



| 标头或库文件名 | 描述                         | 安装位置      |
|-----------------------------|-------------------------------------|-----------------------|
| D3d12.h                     | Direct3D 12 API 标头              | %DXSDK\_DIR%\\Include |
| D3d12.lib                   | 静态 Direct3D 12 API 存根库 | %DXSDK\_DIR%\\Lib     |
| D3d12.dll                   | 动态 Direct3D 12 API 库     | %WINDIR%\\System32    |
| D3d12SDKLayers.h            | Direct3D 12 调试标头            | %DXSDK\_DIR%\\Include |
| D3d12SDKLayers.dll          | 动态 Direct3D 12 调试库   | %WINDIR%\\System32    |



 

若要正确地包括标头文件，请使用类似于以下内容的语句：

`#include <%DXSDK_DIR%Include\d3d12.h>`

若要确定开发计算机上的 %DXSDK\_DIR% 的绝对位置，请在命令窗口中键入：

`set dx`

## <a name="supported-languages"></a>支持的语言

C++ 是 Direct3D 12 开发唯一支持的语言，C# 和其他 .NET 语言不受支持。

## <a name="helper-structures"></a>帮助程序结构

具体而言，通过大量帮助程序结构可轻松地初始化大量 D3D12 结构。 这些结构和某些实用工具函数位于标头 D3dx12.h 中。 此标头是开放源代码，可由开发人员根据需要进行修改 - 从 [D3D12 帮助程序库](https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries/D3DX12)中下载该标头并参阅 [D3D12 的帮助程序结构和函数](helper-structures-and-functions-for-d3d12.md)。

## <a name="memory-management-library"></a>内存管理库

内存管理帮助程序库可供下载，你可以将其集成到你的应用中以便更接近 D3D11 内存管理行为。 作为 D3D11 样式管理库，它最适用于仍在使用“提交的资源”  样式分配策略的应用。 具体而言，当在内存受约束的情况下（例如，低端内存卡、4k、超级设置等），库应被视为很可能返回到 D3D11 性能内存管理的垫脚石。 D3D12 API 启用的技巧可使你获得比 D3D11 更好的内存效率，尽管这些技术可能颇具挑战性且需要较长时间才能实现也是如此。

请注意，此库是正在进行的工作，可能会随着时间的推移发生更改。 使用以下链接访问库和示例。

-   [D3D12 驻留初学者库](https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries)

## <a name="supported-tools-and-libraries"></a>支持的工具和库

以下库都可用于 Direct3D 12。



|                                                                                  |                                                                                                                                                                                                                                                                        |                                                                                                            |
|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|
| **库**                                                                      | **用途**                                                                                                                                                                                                                                                            | **文档**                                                                                          |
| [用于 DirectX 12 的 DirectX 工具包](https://go.microsoft.com/fwlink/?LinkID=615561) | 用于为通用 Windows 平台 (UWP) 应用、适用于 Windows 10 的 Win32 桌面应用程序和 Xbox One 独占应用编写 Direct3D 12 C++ 代码的帮助程序类的大量集合。                                                                         | [DirectX12TK wiki](https://github.com/Microsoft/DirectXTK12/wiki)                                          |
| [DirectXTex](https://go.microsoft.com/fwlink/?LinkId=248926)                      | 适用于读取和写入 DDS 文件，以及执行各种纹理内容处理操作，包括调整大小、格式转换、mip 贴图生成、Direct3D 运行时纹理资源的块压缩和高度贴图到法线贴图的转换。 | [DirectXTex wiki](https://github.com/Microsoft/DirectXTex/wiki)                                            |
| [DirectXMesh](https://go.microsoft.com/fwlink/p/?linkid=324981)                   | 适用于执行各种几何图形内容处理操作，包括生成法线和切线帧、三角形相邻计算和顶点缓存优化。                                                                                | [DirectXMesh wiki](https://github.com/Microsoft/DirectXMesh/wiki)                                          |
| [DirectXMath](https://go.microsoft.com/fwlink/?LinkID=615560)                     | 支持矢量、标量、矩阵、四元数和许多其他数学运算的大量帮助程序类和方法。                                                                                                                               | [MSDN 上的 DirectXMath 文档](https://docs.microsoft.com/windows/desktop/dxmath/ovw-xnamath-progguide) |
| [UVAtlas](https://go.microsoft.com/fwlink/?LinkID=512686)                         | 适用于创建和打包 isochart 纹理图集。                                                                                                                                                                                                           | [UVAtlas wiki](https://github.com/Microsoft/UVAtlas/wiki)                                                  |



 

## <a name="samples"></a>示例

有关工作 D3D12 示例的列表以及如何找到并运行它们的信息，请参阅[工作示例](working-samples.md)。

有关如何添加代码以启用特定功能的演练，请参阅 [D3D12 代码演练](d3d12-code-walk-throughs.md)。

## <a name="debug-layer"></a>调试层

调试层提供大量额外参数和一致性验证（例如，验证着色器链接和资源绑定、验证参数一致性和报告错误说明）。

默认情况下，d3d12.h 中包含支持调试层 D3D12SDKLayers.h 所需的标头。

当调试层列出内存泄漏时，会输出对象接口指针的列表及其友好名称。 默认友好名称是“&lt;unnamed&gt;”。 可以使用 [**ID3D12Object::SetName**](/windows/desktop/api/d3d12/nf-d3d12-id3d12object-setname) 方法设置友好名称。 通常，应在生产版本之外编译这些调用。

我们建议你使用调试层来调试应用以确保它们没有错误和警告。 调试层可帮助你编写 Direct3D 12 代码。 此外，使用调试层时可以提高工作效率，因为可以立即查看混淆呈现错误甚至在其源出现黑屏的原因。 调试层提供多个问题的警告。 例如：

-   忘记设置纹理，但在像素着色器中从该纹理中读取。
-   输出深度，但未绑定深度模具状态。
-   纹理创建失败，并出现无效参数。

设置编译器定义 D3DCOMPILE\_DEBUG 以告知 HLSL 编译器将着色器信息包括在着色器 blob 中。

``` syntax
#define D3DCOMPILE_DEBUG 1
```

有关所有调试接口和方法的详细信息，请参阅[调试层参考](direct3d-12-sdklayers-reference.md)。

有关使用调试层的概述信息，请参阅[了解 D3D12 调试层](understanding-the-d3d12-debug-layer.md)。

## <a name="educational-videos"></a>教育视频

[DirectX 高级学习视频教程](https://www.youtube.com/channel/UCiaX2B8XiXR70jaN7NK-FpA)中包含大量 Direct3D 12 和 Windows 10 相关视频，包括有关图形调试工具的视频和报告图形 bug。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[了解 Direct3D 12](directx-12-getting-started.md)
</dt> </dl>

 

 




