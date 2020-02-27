---
title: 使用 DRED 诊断 GPU 故障
description: 设备删除扩展数据 (DRED) 是一组不断发展的诊断功能，专门用于帮助确定意外设备删除错误的原因。
ms.custom: 19H1
ms.localizationpriority: high
ms.topic: article
ms.date: 04/19/2019
ms.openlocfilehash: bbc754239210899e804d41a294e8c9f47967fb25
ms.sourcegitcommit: 780d4b1601c45658ef0b799b80d13f45a53d808d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/26/2020
ms.locfileid: "77635089"
---
# <a name="use-dred-to-diagnose-gpu-faults"></a>使用 DRED 诊断 GPU 故障
DRED 表示设备删除扩展数据。 DRED 是一组不断发展的诊断功能，专门用于帮助确定意外设备删除错误的原因。 在支持必要功能（如下面的定义所示）的硬件上，DRED 传送自动痕迹导航以及 GPU 页面错误报告。

## <a name="auto-breadcrumbs"></a>自动痕迹导航
在设置自动导航的场景前，我们先介绍手动差异。 预测[超时检测和恢复 (TDR)](/windows-hardware/drivers/display/timeout-detection-and-recovery) 的可能情况时，可以使用 [ID3D12GraphicsCommandList2::WriteBufferImmediate 方法](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist2-writebufferimmediate)将痕迹导航置于 GPU 命令流中，来跟踪 GPU 进度。

若要创建自定义低开销实现，这是一种合理的方法。 但是它可能会缺少标准化解决方案的部分功能，例如调试器扩展，或通过 [Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting)（又称为 Watson）报告。

因此，自动痕迹导航调用WriteBufferImmediate 来将进度计数器置于 GPU 命令流中。 DRED 在每个呈现器运行（即引发 GPU 作业的各个运行，如绘制、调度、复制、解析等）后面插入痕迹导航。&mdash; 如果设备是在 GPU 工作负荷的中间删除的，则通过痕迹值实质上是在发生错误之前完成的 render 操作的集合。

痕迹导航历史记录环形缓冲区最多保留给定命令列表中 64KB 的运行。 若命令列表中的运行数超过 65536，则仅存储最近的 64KB 运行&mdash;首先将重写时间最早的运行。 但是，痕迹导航计数器值继续计数，最高计数为 `UINT_MAX`。 因此，最后的运行索引 =（痕迹导航计数 - 1）% 65536。

Windows 10 版本 1809（Windows 10 2018 年 10 月更新）首次发布了 DRED 1.0，并且它公开了初步的自动痕迹导航。 但是，没有适用于它的 Api，启用通过1.0 的唯一方法是使用**反馈中心**来捕获**应用程序**的 TDR 复制（重现） & 游戏 \>**游戏性能和兼容性**。 DRED 1.0 主要用于借助客户反馈帮助执行游戏崩溃根本原因分析。
### <a name="caveats"></a>注意事项
- 由于 GPU 占用大量管道，因此无法确保痕迹导航计数器准确地指示已失败的运行。 实际上，在一些基于磁贴的延迟呈现器设备上，痕迹导航计数器可以是实际 GPU 进程后面的完整资源或无序访问视图 (UAV) 屏障。
- 显示驱动程序可以在执行命令很久之前将命令重新排序、从资源内存预提取，或在完成命令很久之后刷新缓存内存。 上述任意操作均可以引发 GPU 错误。 在这些情况下，自动痕迹导航计数器的用途较小，或会误导用户。
### <a name="performance"></a>性能
尽管自动痕迹导航专门用于降低开销，但它们不是免费功能。 在典型的 AAA Direct3D 12 图形游戏引擎上，经验测量值性能下降幅度为 2-5%。 出于此原因，默认为关闭自动痕迹导航。
### <a name="hardware-requirements"></a>硬件要求
由于在删除设备后必须保留痕迹导航计数器值，因此包含痕迹导航的资源必须位于系统内存，并且在删除设备后必须保留它。 这意味着，显示驱动程序需要支持 [D3D12_FEATURE_EXISTING_HEAPS](/windows/desktop/api/d3d12/ne-d3d12-d3d12_feature)。 庆幸的是，Windows 10 版本 1903 上的多数 Direct3D 12 显示驱动程序均是如此。
## <a name="gpu-page-fault-reporting"></a>GPU 页错误报告
DRED 1.1 的新增功能是 DRED GPU 页面错误报告。 GPU 页面错误通常在下列情况之一下发生。

1. 应用程序在 GPU 上错误地执行了应用已删除的对象的作业。 这是意外删除设备的主要原因之一。
2. 应用程序错误地在 GPU 上执行了访问已逐出的资源或非驻留磁贴的作业。
3. 着色器引用未初始化的或过时的描述符。
3. 着色器索引超出根绑定末尾。

DRED 尝试通过报告与 GPU 报告页面错误虚拟地址 (VA) 匹配的任何现有或最新释放的 API 对象的名称和类型，来解决上述某些场景。

### <a name="caveat"></a>警告
尽管许多 GPU 都支持页面错误，但并非所有都支持。 某些 Gpu 通过以下方式响应内存错误：位桶写入;读取模拟数据（例如零）;或者只是挂起。 遗憾的是，若 GPU 未立即挂起，管道稍后可能会执行[超时检测和恢复(TDR)](/windows-hardware/drivers/display/timeout-detection-and-recovery)，这样将更加难以找到根本原因。

### <a name="performance"></a>性能
Direct3D 12 运行时必须主动策展可通过虚拟地址 (VA) 索引的现有和最近删除的 API 对象集合。 这将增加系统内存开销，并对对象创建和析构性能产生轻微的不利影响。 鉴于此原因，默认为禁用此行为。

### <a name="hardware-requirements"></a>硬件要求
不支持页面错误的 GPU 仍然可以从自动痕迹导航功能受益。

## <a name="setting-up-dred-in-code"></a>在代码中安装 DRED
DRED 设置全局到进程，必须先配置它们，才能创建 Direct3D 12 设备。 若要配置它们，请调用 [D3D12GetDebugInterface](/windows/desktop/api/d3d12/nf-d3d12-d3d12getdebuginterface) 函数，来检索 [ID3D12DeviceRemovedExtendedDataSettings](/windows/desktop/api/d3d12/nn-d3d12-id3d12deviceremovedextendeddatasettings)。

```cpp
CComPtr<ID3D12DeviceRemovedExtendedDataSettings> pDredSettings;
VERIFY_SUCCEEDED(D3D12GetDebugInterface(IID_PPV_ARGS(&pDredSettings)));

// Turn on auto-breadcrumbs and page fault reporting.
pDredSettings->SetAutoBreadcrumbsEnablement(D3D12_DRED_ENABLEMENT_FORCED_ON);
pDredSettings->SetPageFaultEnablement(D3D12_DRED_ENABLEMENT_FORCED_ON);
```

> [!NOTE]
> DRED 设置修改不影响已创建的设备。 但对 [D3D12CreateDevice](/windows/desktop/api/d3d12/nf-d3d12-d3d12createdevice) 的后续调用使用最新的 DRED 设置。

## <a name="accessing-dred-data-in-code"></a>在代码中访问 DRED 数据
检测到设备删除后（如 Present返回 [DXGI_ERROR_DEVICE_REMOVED](/windows/desktop/com/com-error-codes-10)），请使用 [ID3D12DeviceRemovedExtendedData](/windows/desktop/api/d3d12/nn-d3d12-id3d12deviceremovedextendeddata) 接口的方法访问已删除设备的 DRED 数据。

若要检索 ID3D12DeviceRemovedExtendedData 接口，请在 [ID3D12Device](/windows/desktop/api/unknwn/nf-unknwn-iunknown-queryinterface(refiid_void)) （或派生的）接口上调用 [QueryInterface](/windows/win32/api/d3d12/nn-d3d12-id3d12device)，并传送 ID3D12DeviceRemovedExtendedData 的接口标识符 (IID)。

```cpp
void MyDeviceRemovedHandler(ID3D12Device * pDevice)
{
    CComPtr<ID3D12DeviceRemovedExtendedData> pDred;
    VERIFY_SUCCEEDED(pDevice->QueryInterface(IID_PPV_ARGS(&pDred)));
    D3D12_DRED_AUTO_BREADCRUMBS_OUTPUT DredAutoBreadcrumbsOutput;
    D3D12_DRED_PAGE_FAULT_OUTPUT DredPageFaultOutput;
    VERIFY_SUCCEEDED(pDred->GetAutoBreadcrumbsOutput(&DredAutoBreadcrumbsOutput));
    VERIFY_SUCCEEDED(pDred->GetPageFaultAllocationOutput(&DredPageFaultOutput));
    // Custom processing of DRED data can be done here.
    // Produce telemetry...
    // Log information to console...
    // break into a debugger...
}
```

## <a name="debugger-access-to-dred"></a>调试器的 DRED 访问权限
调试器可以通过 d3d12!D3D12DeviceRemovedExtendedData 数据导出访问 DRED 数据。

## <a name="dred-telemetry"></a>DRED 遥测
应用程序可以使用 DRED API 控制 DRED 功能，并收集遥测来帮助分析问题。 这样即可从更广阔的范围捕捉这些难以重现的 TDR。

自 Windows 10 版本 1903 起，将所有用户模式设备删除事件上报给 [Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting)（又称为 Watson）。 若特定应用程序组合、GPU 和显示驱动程序引发了大量设备删除事件，则可能会为在相似配置上启动同个应用程序的客户暂时启用 DRED。
