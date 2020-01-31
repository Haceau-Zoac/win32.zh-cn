---
title: 使用 DirectML 调试层
description: DirectML 调试层是可选的开发时组件，可帮助调试 DirectML 代码。
ms.custom: Windows 10 May 2019 Update
ms.localizationpriority: high
ms.topic: article
ms.date: 04/19/2019
ms.openlocfilehash: 75bc7f1bf7297a971efe33ee8e19e7b81403c131
ms.sourcegitcommit: cba7f424a292fd7f3a8518947b9466439b455419
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/23/2019
ms.locfileid: "74420390"
---
# <a name="using-the-directml-debug-layer"></a>使用 DirectML 调试层

DirectML 调试层是可选的开发时组件，可帮助调试 DirectML 代码。 调试层是独立图形工具包的一部分，以[按需功能](/windows-hardware/manufacture/desktop/features-on-demand-v2--capabilities) (FOD) 形式分发。 若要使用调试层，必须在系统上安装图形工具 FOD。

强烈建议使用 DirectML 开发应用程序时启用调试层，因为它可以在 API 用法无效的情况下提供宝贵信息。

## <a name="installing-the-directml-debug-layer"></a>安装 DirectML 调试层

若要安装可选图形工具按需功能 (FOD) 包，请从管理员 Powershell 提示符运行以下命令。

```powershell
Add-WindowsCapability -Online -Name "Tools.Graphics.DirectX~~~~0.0.1.0"
```

也可以在 Windows 10 设置中安装图形工具包。 导航到“设置” > “应用” > “应用和功能” > “可选功能” > “添加功能”，然后选择“图形工具”。

## <a name="enabling-the-directml-debug-layer"></a>启用 DirectML 调试层

安装图形工具包之后，可以在调用 [DMLCreateDevice](/windows/desktop/api/directml/nf-directml-dmlcreatedevice.md) 时通过提供 [DML_CREATE_DEVICE_FLAG_DEBUG](/windows/desktop/api/directml/ne-directml-dml_create_device_flag) 来启用 DirectML 调试层。

> [!IMPORTANT]
> 必须先启用 Direct3D 12 调试层。 然后，通过调用 DMLCreateDevice 来启用 DirectML 调试层。

启用 DirectML 调试层之后，任何 DirectML 错误或无效 API 调用都会导致调试信息以调试输出形式发出。 下面提供了一个示例。

```console
DML_OPERATOR_CONVOLUTION: invalid D3D12_HEAP_TYPE. DirectML requires all bound buffers to be D3D12_HEAP_TYPE_DEFAULT.
```

## <a name="see-also"></a>另请参阅

* [DMLCreateDevice 函数](/windows/desktop/api/directml/nf-directml-dmlcreatedevice.md)
* [可用的按需功能](/windows-hardware/manufacture/desktop/features-on-demand-non-language-fod)
* [配合使用基于 GPU 的验证和 Direct3D 12 调试层](/windows/desktop/direct3d12/using-d3d12-debug-layer-gpu-based-validation)
* [Direct3D 12 调试层参考](/windows/desktop/direct3d12/direct3d-12-sdklayers-reference)
