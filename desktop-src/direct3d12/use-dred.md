---
title: 通过用于诊断 GPU 错误
description: 设备中删除扩展数据 （通过） 是一组不断发展的诊断功能，旨在帮助您识别意外的设备删除错误的原因。
ms.custom: 19H1
ms.topic: article
ms.date: 02/14/2019
ms.openlocfilehash: 2427c2c8e1303e432212359e73d79e8c06fc1eda
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223888"
---
# <a name="use-dred-to-diagnose-gpu-faults"></a>通过用于诊断 GPU 错误
通过代表设备中删除扩展数据。 通过为一组不断发展的诊断功能，旨在帮助您识别意外的设备删除错误的原因。 在硬件上的支持 （按下面的定义） 所需的功能，通过提供自动的痕迹导航以及 GPU 页错误报告。

## <a name="auto-breadcrumbs"></a>自动痕迹导航
若要设置为自动痕迹导航场景，让我们首先提到手动不同。 预期在偶发性[超时检测和恢复 (TDR)](/windows-hardware/drivers/display/timeout-detection-and-recovery)，可以使用[ID3D12GraphicsCommandList2::WriteBufferImmediate 方法](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist2-writebufferimmediate)放置*的痕迹导航*到 GPU 命令流中，为了跟踪 GPU 进度。

如果你想要创建的自定义的开销较低的实现，这是合理的方法。 但它可能缺少某些标准化解决方案，如调试器扩展，或通过进行报告的通用性[Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting) (也称为 Watson)。

因此，通过的痕迹导航自动调用**WriteBufferImmediate**将进度计数器置于 GPU 命令流。 通过后每个插入痕迹*呈现 op*&mdash;这意味着 GPU 的工作会导致每个操作 (例如，**绘制**，**调度**， **副本**，**解决**，等等)。 如果 GPU 工作负荷中间删除该设备，通过痕迹导航值是实质上是在错误之前已完成呈现操作的集合。

痕迹导航历史记录环形缓冲区将保留最多 64KiB 操作在给定的命令列表。 如果有多个 65536 操作中的命令列表，则只能将最后一个 64KiB 操作存储&mdash;先覆盖最旧的操作。 但是，在痕迹导航计数器值继续计数最多`UINT_MAX`。 因此，LastOpIndex = (BreadcrumbCount-1) %65536。

通过 1.0 最初在 Windows 10，版本 1809年中提供 （Windows 10 2018 年 10 月更新），并公开基本的自动痕迹导航。 但是，没有任何 Api，并启用通过 1.0 的唯一方法是使用**反馈中心**TDR 重现 （重现） 捕获有关**应用程序和游戏** \> **游戏性能和兼容性**。 通过 1.0 的主要目的是帮助客户反馈通过根本原因分析游戏崩溃。
### <a name="caveats"></a>警告
- 由于 GPU 很大程度通过管道传递，则痕迹导航计数器指示失败的确切操作无法保证。 事实上，某些基于磁贴的延迟的呈现在设备上，很可能要为完整的资源或无序的访问视图 (UAV) 屏障背后的实际 GPU 进度的痕迹导航计数器。
- 显示驱动程序可以重新排序命令，从好之前执行命令，资源内存预提取或刷新缓存的内存也后完成的命令。 其中的任何可能产生 GPU 错误。 在这种情况下，自动痕迹导航计数器也可能不太有用或令人误解。
### <a name="performance"></a>性能
尽管自动痕迹导航设计为低开销，但它们不是免费。 经验测量结果显示在一个典型的 AAA Direct3D 12 图形游戏引擎，2-5%的性能损失。 出于此原因，自动痕迹导航默认为关闭的。
### <a name="hardware-requirements"></a>硬件要求
因为必须在设备删除后保存的痕迹导航计数器值，包含痕迹导航栏的资源必须存在于系统内存和它发生设备删除时必须保留。 这意味着需要支持显示器驱动程序[ **D3D12_FEATURE_EXISTING_HEAPS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_feature)。 幸运的是，这是大多数 Direct3D 12 中显示的驱动程序在 Windows 10，版本 1903年这种情况。
## <a name="gpu-page-fault-reporting"></a>GPU 页错误报告
通过 GPU 通过 1.1 为新的功能页上的故障报告功能。 GPU 页面错误通常会发生这些条件之一。

1. 应用程序错误地引用已删除的对象在 GPU 上执行工作。 这是意外的设备删除的主要原因之一。
2. 应用程序访问逐出的资源或将驻留的磁贴在 GPU 上错误地执行工作。
3. 着色器引用了未初始化或变得陈旧描述符。
3. 根绑定的末尾之外的着色器索引。

通过尝试解决其中一些方案通过报告的名称和任何的类型现有的或最近已释放的 API 对象的匹配 GPU 报告页面错误的虚拟地址 (VA)。

### <a name="caveat"></a>需要注意的地方
并非所有 Gpu 都支持页面错误 （尽管执行许多操作）。 对内存进行某些 Gpu 响应错误的： 位存储桶将写入;模拟的读取数据 （例如，零）;或者只挂起。 遗憾的是，在其中的 GPU 不会立即挂起的情况下[超时检测和恢复 (TDR)](/windows-hardware/drivers/display/timeout-detection-and-recovery)更高版本中的管道，使其更加困难，若要查找的根本原因，可能发生。

### <a name="performance"></a>性能
Direct3D 12 中运行时主动必须由虚拟地址 (VA) 组织现有的和最近删除可编制索引的 API 对象的集合。 这会增加系统内存开销，并引入了对对象的创建和析构的些许性能。 因此，此行为是默认情况下关闭。

### <a name="hardware-requirements"></a>硬件要求
不支持分页错误 GPU 仍可以受益于自动痕迹导航功能。

## <a name="setting-up-dred-in-code"></a>设置通过在代码中
通过设置是全局到进程，并且必须在创建 Direct3D 12 设备前进行配置。 若要执行此操作，调用[ **D3D12GetDebugInterface** ](/windows/desktop/api/d3d12/nf-d3d12-d3d12getdebuginterface)函数以检索[ **ID3D12DeviceRemovedExtendedDataSettings**](/windows/desktop/api/d3d12/nn-d3d12-id3d12deviceremovedextendeddatasettings)。

```cpp
CComPtr<ID3D12DeviceRemovedExtendedDataSettings> pDredSettings;
VERIFY_SUCCEEDED(D3D12GetDebugInterface(IID_PPV_ARGS(&pDredSettings)));

// Turn on auto-breadcrumbs and page fault reporting.
pDredSettings->SetAutoBreadcrumbsEnablement(D3D12_DRED_ENABLEMENT_FORCED_ON);
pDredSettings->SetPageFaultEnablement(D3D12_DRED_ENABLEMENT_FORCED_ON);
```

> [!NOTE]
> 对通过设置的修改具有对已创建的设备没有影响。 但随后调用[ **D3D12CreateDevice** ](/windows/desktop/api/d3d12/nf-d3d12-d3d12createdevice)使用最新的通过设置。

## <a name="accessing-dred-data-in-code"></a>访问通过在代码中的数据
设备后删除已检测到 (例如，**存在**返回[ **DXGI_ERROR_DEVICE_REMOVED**](/windows/desktop/com/com-error-codes-10))，使用的方法[ **ID3D12DeviceRemovedExtendedData** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12deviceremovedextendeddata)界面来访问已删除的设备通过数据。

若要检索**ID3D12DeviceRemovedExtendedData**界面，请调用[QueryInterface](/windows/desktop/api/unknwn/nf-unknwn-iunknown-queryinterface(refiid_void))上[ID3D12Device](/windows/desktop/api/d3d12/nn-d3d12-id3d12device.md) （或派生） 接口，传递的接口标识符(IID) 的**ID3D12DeviceRemovedExtendedData**。

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

## <a name="debugger-access-to-dred"></a>通过调试程序访问
调试器有权访问通过数据通过**d3d12 ！D3D12DeviceRemovedExtendedData**数据导出。

## <a name="dred-telemetry"></a>通过遥测
你的应用程序可以使用通过 Api 来控制通过功能并收集遥测来帮助分析问题。 这使你更广泛的 net 捕获这些硬重现 TDRs。

截至 Windows 10，版本 1903，所有用户模式设备中删除事件都报告给[Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting)，也称为 Watson。 如果应用程序、 GPU 和显示器驱动程序的特定组合生成足够数量的设备中删除事件，然后就可以通过将暂时启用启动上类似的配置的相同应用程序的客户。
