---
title: DirectML 中的绑定
description: 本主题论述如何调试 DirectML 设备删除和其他错误条件。
ms.custom: Windows 10 May 2019 Update
ms.localizationpriority: high
ms.topic: article
ms.date: 04/19/2019
ms.openlocfilehash: 14045601ccdc237679ac312fd705033b4f226c51
ms.sourcegitcommit: 8141395d1bd1cd755d1375715538c3fe714ba179
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67465002"
---
# <a name="handling-errors-and-device-removal-in-directml"></a>在 DirectML 中处理错误和设备删除

## <a name="device-removal"></a>设备删除

如果发生不可恢复的错误，DirectML 设备可能会进入“设备删除”状态。 导致设备删除的不可恢复错误包括无效的 API 用法（对于不返回 [HRESULT](/windows/desktop/com/structure-of-com-error-codes) 的方法）、驱动程序错误、硬件故障或内存不足 (OOM) 条件  。

删除 DirectML 设备时，设备上的所有方法调用以及该设备创建的每个对象都变为 no-op。 对于返回 [HRESULT](/windows/desktop/com/structure-of-com-error-codes) 的方法，会返回[DXGI_ERROR_DEVICE_REMOVED](/windows/desktop/direct3ddxgi/dxgi-error) 错误代码   。 可以使用 [IDMLDevice::GetDeviceRemovedReason 方法](/windows/desktop/api/directml/nf-directml-idmldevice-getdeviceremovedreason)来检查是否已删除 DirectML 设备，并检索更为详细的错误代码  。

除非释放受影响的设备及其所有子设备，然后从头重新创建 DirectML 设备，否则无法从设备删除中恢复。

基础 Direct3D 12 设备的设备删除也会导致删除 DirectML 设备。 但反之却不成立。 DirectML 设备删除不一定会导致删除基础 Direct3D 12 设备。

## <a name="debugging-directml-device-removal-and-other-errors"></a>调试 DirectML 设备删除和其他错误

DirectML 错误最常见的原因是无效的 API 用法。 无效的 API 用法可能导致 E_INVALIDARG HRESULT 错误代码，或者可能导致设备删除  。

我们强烈建议在开发期间启用 [DirectML 调试层](dml-debug-layer.md)，以便捕获并调试此类错误。 DirectML 调试层执行方法参数和 API 用法的广泛验证，它将发出调试输出消息来帮助进行调试。

## <a name="see-also"></a>另请参阅

* [使用 DirectML 调试层](dml-debug-layer.md)
