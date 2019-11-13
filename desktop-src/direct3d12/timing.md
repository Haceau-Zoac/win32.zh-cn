---
title: 计时（Direct3D 12 图形）
description: 本节介绍如何查询时间戳，以及如何校准 GPU 和 CPU 时间戳计数器。
ms.assetid: CC1E5BAB-4363-43FF-BF5B-6C9AEBECD6CA
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 6de008f7e039338089e2ffa686644a8487af882e
ms.sourcegitcommit: 40a1246849dba8ececf54c716b2794b99c96ad50
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/12/2019
ms.locfileid: "73957230"
---
# <a name="timing"></a>定时

本节介绍如何查询时间戳，以及如何校准 GPU 和 CPU 时间戳计数器。

-   [时间戳频率](#timestamp-frequency)
-   [时间戳校准](#timestamp-calibration)
-   [相关主题](#related-topics)

## <a name="timestamp-frequency"></a>时间戳频率

应用程序可以基于每个命令队列查询 GPU 时间戳频率（请参阅 [ID3D12CommandQueue::GetTimestampFrequency](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-gettimestampfrequency) 方法）。

返回的频率以 Hz（时钟周期/秒）为单位。 如果指定的命令队列不支持时间戳，则此 API 失败（并返回 E\_FAIL）（请参阅[查询](queries.md)部分中的表）。

## <a name="timestamp-calibration"></a>时间戳校准

D3D12 使应用程序能够将从时间戳查询获得的结果与从调用 `QueryPerformanceCounter` 获得的结果相关联。 这是通过调用 [ID3D12CommandQueue::GetClockCalibration](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-getclockcalibration) 实现的。

[GetClockCalibration](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-getclockcalibration) 对给定命令队列的 GPU 时间戳计数器进行采样，并通过 `QueryPerformanceCounter` 对 CPU 计数器进行采样，这两项操作几乎同时进行。 如果指定的命令队列不支持时间戳，则此再次 API 失败（返回 E\_FAIL）（请参阅[查询](queries.md)部分中的表）。

请注意，GPU 和 CPU 时间戳计数器不一定与这些处理器的时钟速度直接相关，而是从时间戳时钟周期开始工作。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[计数器和查询](counters-and-queries.md)
</dt> <dt>

[**ID3D12Device::SetStablePowerState**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-setstablepowerstate)
</dt> <dt>

[**ID3D12Object::SetName**](/windows/desktop/api/d3d12/nf-d3d12-id3d12object-setname)
</dt> <dt>

[**ID3DUserDefinedAnnotation**](https://docs.microsoft.com/windows/desktop/api/d3d11_1/nn-d3d11_1-id3duserdefinedannotation)
</dt> <dt>

[性能度量](performance-measurement.md)
</dt> </dl>

 

 




