---
title: DirectML 中的绑定
description: DirectML 调试层是一个可选的开发时间组件，可帮助你在调试 DirectML 代码。
ms.custom: 19H1
ms.topic: article
ms.date: 03/13/2019
ms.openlocfilehash: bc57d1cad3c51b5e1f237c9e18076df7bfc3d973
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224011"
---
# <a name="using-the-directml-debug-layer"></a>使用 DirectML 调试层

DirectML 调试层是一个可选的开发时间组件，可帮助你在调试 DirectML 代码。 调试层是一个单独的图形工具软件包，以的形式分发的一部分[功能按需](/windows-hardware/manufacture/desktop/features-on-demand-v2--capabilities)(FOD)。 若要使用调试层，必须在系统上安装图形工具 FOD。

我们强烈建议您开发使用 DirectML，应用程序，因为它可以提供保护无效 API 使用情况的宝贵信息时启用调试层。

## <a name="installing-the-directml-debug-layer"></a>安装 DirectML 调试层

若要安装可选的图形工具功能按需 (FOD) 包，请运行以下命令从管理员 Powershell 提示符。

```powershell
Add-WindowsCapability -Online -Name "Tools.Graphics.DirectX~~~~0.0.1.0"
```

或者，可以安装在 Windows 10 设置从图形工具包。 导航到**设置** > **应用** > **应用和功能** > **可选功能**  > **添加功能**> 然后选择**图形工具**。

## <a name="enabling-the-directml-debug-layer"></a>启用 DirectML 调试层

安装图形工具包后，可以通过提供启用 DirectML 调试层[ **DML_CREATE_DEVICE_FLAG_DEBUG** ](/windows/desktop/api/directml/ne-directml-dml_create_device_flag)当您调用[ **DMLCreateDevice**](/windows/desktop/api/directml/nf-directml-dmlcreatedevice.md).

> [!IMPORTANT]
> 必须先启用 Direct3D 12 调试层。 并*然后*通过调用启用 DirectML 调试层**DMLCreateDevice**。

启用 DirectML 调试层后，任何 DirectML 错误或无效的 API 调用将导致调试输出作为发出调试信息。 下面提供了一个示例。

```console
DML_OPERATOR_CONVOLUTION: invalid D3D12_HEAP_TYPE. DirectML requires all bound buffers to be D3D12_HEAP_TYPE_DEFAULT.
```

## <a name="see-also"></a>另请参阅

* [DMLCreateDevice 函数](/windows/desktop/api/directml/nf-directml-dmlcreatedevice.md)
* [可用功能按需](/windows-hardware/manufacture/desktop/features-on-demand-non-language-fod)
* [使用基于 GPU 的 Direct3D 12 调试层验证](/windows/desktop/direct3d12/using-d3d12-debug-layer-gpu-based-validation)
* [Direct3D 12 调试层引用](/windows/desktop/direct3d12/direct3d-12-sdklayers-reference)
