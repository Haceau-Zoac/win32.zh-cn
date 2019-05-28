---
title: DirectML 中的绑定
description: 本主题讨论如何调试 DirectML 设备-删除或其他错误情况。
ms.custom: 19H1
ms.topic: article
ms.date: 03/14/2019
ms.openlocfilehash: ccafb78426154818ad062760b1f694fb47593f28
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224008"
---
# <a name="handling-errors-and-device-removal-in-directml"></a>处理错误和设备-删除 DirectML 中

## <a name="device-removal"></a>设备删除

如果出现不可恢复的错误，DirectML 设备可能会进入"设备删除"状态。 发生了不可恢复错误，导致设备删除包含无效的 API 使用情况 (不返回的方法[ **HRESULT**](/windows/desktop/com/structure-of-com-error-codes))，驱动程序错误、 硬件故障或-内存不足 (OOM) 条件。

移除某个 DirectML 设备后，在设备上的所有方法都调用，并由该设备创建每个对象都将变为进行任何操作。 返回的方法[ **HRESULT**](/windows/desktop/com/structure-of-com-error-codes)即[ **DXGI_ERROR_DEVICE_REMOVED** ](/windows/desktop/direct3ddxgi/dxgi-error)返回错误代码。 可以使用[ **IDMLDevice::GetDeviceRemovedReason**方法](/windows/desktop/api/directml/nf-directml-idmldevice-getdeviceremovedreason)来检查是否已移除 DirectML 设备，并检索更详细的错误代码。

不能通过释放受影响的设备及其所有子级，然后重新创建从零开始的 DirectML 设备从此设备删除除外。

设备删除的基础 Direct3D 12 设备也会使 DirectML 设备中删除。 但是，反之不成立。 DirectML 设备删除不一定导致变得删除的基础 Direct3D 12 设备。

## <a name="debugging-directml-device-removal-and-other-errors"></a>调试 DirectML 设备-删除和其他错误

DirectML 错误的最常见原因是 API 用法无效。 无效的 API 使用情况可能会导致**E_INVALIDARG** HRESULT 错误代码，或它可能会导致设备删除。

我们强烈建议您启用[DirectML 调试层](dml-debug-layer.md)期间您的开发，以便捕获并调试此类错误。 DirectML 调试层执行大量验证的方法参数和 API 使用情况和它将发出调试输出消息，以帮助您调试。

## <a name="see-also"></a>另请参阅

* [使用 DirectML 调试层](dml-debug-layer.md)
