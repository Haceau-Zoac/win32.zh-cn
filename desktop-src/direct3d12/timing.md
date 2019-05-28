---
title: 定时
description: 本节介绍了查询时间戳，并校准 GPU 和 CPU 时间戳计数器。
ms.assetid: CC1E5BAB-4363-43FF-BF5B-6C9AEBECD6CA
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 4039bd08faf15b15f1d4214463de3f4d77ec837e
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224341"
---
# <a name="timing"></a>定时

本节介绍了查询时间戳，并校准 GPU 和 CPU 时间戳计数器。

-   [时间戳频率](#timestamp-frequency)
-   [时间戳校准](#timestamp-calibration)
-   [相关的主题](#related-topics)

## <a name="timestamp-frequency"></a>时间戳频率

应用程序可以查询每个命令队列的基础上的 GPU 时间戳频率 (请参阅[ **ID3D12CommandQueue::GetTimestampFrequency** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-gettimestampfrequency)方法)。

返回的频率以赫兹 （计时周期数/秒）。 此 API 将失败 (并返回电子\_失败) 如果指定的命令队列不支持时间戳 (请参阅中的表[查询](queries.md)部分)。

## <a name="timestamp-calibration"></a>时间戳校准

D3D12 使应用程序可以将结果从时间戳查询中获取与调用获得的结果相互关联`QueryPerformanceCounter`。 启用此功能通过调用[ **ID3D12CommandQueue::GetClockCalibration**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-getclockcalibration)。

[**GetClockCalibration** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-getclockcalibration)提供的示例为给定的命令队列的 GPU 时间戳计数器和示例通过 CPU 计数器`QueryPerformanceCounter`在几乎在同一时间。 此 API 再次失败 (返回电子\_失败) 如果指定的命令队列不支持时间戳 (请参阅中的表[查询](queries.md)部分)。

请注意，GPU 和 CPU 时间戳计数器不一定直接与这些处理器的时钟速度，但改为工作由时间戳刻度数。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> <dt>

[**ID3D12Device::SetStablePowerState**](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-setstablepowerstate)
</dt> <dt>

[**ID3D12Object::SetName**](/windows/desktop/api/D3D12/nf-d3d12-id3d12object-setname)
</dt> <dt>

[**ID3DUserDefinedAnnotation**](https://msdn.microsoft.com/library/windows/desktop/hh446881)
</dt> <dt>

[性能度量](performance-measurement.md)
</dt> </dl>

 

 




