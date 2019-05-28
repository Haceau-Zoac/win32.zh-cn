---
title: 资源生存期和同步
description: 为避免出现未定义的行为，DirectML 应用程序必须正确管理对象生存期和 CPU 和 GPU 之间的同步。
ms.custom: 19H1
ms.topic: article
ms.date: 03/14/2019
ms.openlocfilehash: c16d3d6fb889afb15e33c2d2fdd8ef1b2f08b9e1
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223975"
---
# <a name="resource-lifetime-and-synchronization"></a>资源生存期和同步

就像你 DirectML 的应用程序 （目的是避免未定义的行为） 与 Direct3D 12 中，必须正确管理对象生存期和 CPU 和 GPU 之间的同步。 DirectML 到 Direct3D 12 的遵循相同的资源生命周期模型。

- 使用强引用计数的 DirectML 维护两个 CPU 对象之间的生存期依赖项。 你的应用程序无需手动管理 CPU 生存期依赖项。 例如，每个设备子级保存到其父设备的强引用。
- 生存期 GPU 对象之间的依赖项&mdash;或跨越多 CPU 和 GPU 的依赖项&mdash;不自动进行管理。 它是应用程序有责任确保资源已完成在 GPU 上的执行的所有工作使用之前至少实时 GPU 资源。

## <a name="directml-devices"></a>DirectML 设备

DirectML 设备是线程安全的无状态工厂对象。 每个设备子 (请参阅[ **IDMLDeviceChild**](/windows/desktop/api/directml/nn-directml-idmldevicechild)) 保存到其父 DirectML 设备的强引用 (请参阅[ **IDMLDevice**](/windows/desktop/api/directml/nn-directml-idmldevice))。 这意味着，可以随时从任何设备子接口检索父设备接口。

DirectML 设备又包含对用于创建它的 Direct3D 12 中设备的强引用 (请参阅[ **ID3D12Device**](/windows/desktop/api/d3d12/nn-d3d12-id3d12device)，并且派生接口)。

DirectML 设备是无状态，因为它是隐式线程安全的。 可以从多个线程同时无需外部同步 DirectML 设备上调用方法。

但是，不同于 Direct3D 12 设备，DirectML 设备不是单一实例对象。 您可以随意创建任意数量的 DirectML 设备根据需要。 但是，您可能不混合和匹配属于不同的设备的设备子级。 例如， [ **IDMLBindingTable** ](/windows/desktop/api/directml/nn-directml-idmlbindingtable)并[ **IDMLCompiledOperator** ](/windows/desktop/api/directml/nn-directml-idmlcompiledoperator)两种类型的设备 （这两个接口派生自的子级直接或间接从**IDMLDeviceChild**)。 和可能不会使用绑定表 (**IDMLBindingTable**) 要绑定的运算符 (**IDMLCompiledOperator**) 如果的操作员以及绑定表属于不同 DirectML 设备实例。

由于 DirectML 设备不是单一实例，在每台设备，而不是正在进程级事件，因为它是 Direct3D 12 设备会发生设备删除。 有关详细信息，请参阅[处理错误和设备删除中 DirectML](dml-errors.md)。

## <a name="lifetime-requirements-of-gpu-resources"></a>GPU 资源的生存期要求

Direct3D 12 中，如 DirectML 不会自动同步之间的 CPU 和 GPU;也不会它自动使资源保持活动状态，而不是由 GPU 使用中。 相反，这些是你的应用程序的职责。

当执行包含 DirectML 调度的命令列表，你的应用程序必须确保，GPU 资源将保持活动状态，直到使用这些资源的所有工作已都完成在 GPU 上的执行。

情况下[ **IDMLCommandRecorder::RecordDispatch** ](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch) DirectML 运算符，包括以下对象。

- [ **IDMLCompiledOperator** ](/windows/desktop/api/directml/nn-directml-idmlcompiledoperator)正在执行 (或[ **IDMLOperatorInitializer** ](/windows/desktop/api/directml/nn-directml-idmloperatorinitializer)相反，如果执行运算符初始化).
- **IDMLCompiledOperator**备份用于绑定运算符的绑定表。
- [ **ID3D12Resource** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12resource)为运算符的输入/输出绑定的对象。
- **ID3D12Resource**绑定的对象为持久性和临时资源，如果适用。
- [ **ID3D12CommandAllocator** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12commandallocator)备份命令列表本身。

并非所有 DirectML 接口都表示 GPU 资源。 例如，绑定表 does*不*需要使用它的所有调度都完成在 GPU 上的执行之前保持活动状态。 这是因为绑定表本身不拥有任何 GPU 资源。 相反，会执行描述符堆。 因此，基础*描述符堆*是必须要保持活动状态，直到执行完毕后，该对象并不是绑定表本身。

在 Direct3D 12 中存在类似的概念。 命令*分配器*必须将保持活动状态，直到由于拥有 GPU 内存使用它的所有执行都已都完成; 在 GPU 上。 但是，命令*列表*本身不必拥有 GPU 内存，因此它可以因此重置或已为执行提交后，即可释放。

在 DirectML，编译运算符 ([**IDMLCompiledOperator**](/windows/desktop/api/directml/nn-directml-idmlcompiledoperator)) 和运算符初始值设定项 ([**IDMLOperatorInitializer**](/windows/desktop/api/directml/nn-directml-idmloperatorinitializer)) 同时直接，拥有 GPU 资源，因此必须将它们保持活动状态之前使用它们的所有调度已都完成在 GPU 上的执行。 此外，使用任何 Direct3D 12 资源 （命令分配器描述符堆、 缓冲，作为示例） 必须同样会保持活动状态，你的应用程序。

如果仍在使用由 GPU 时，过早地释放对象，则结果将是未定义的行为，这有可能会造成设备删除或其他错误。

## <a name="cpu-and-gpu-synchronization"></a>CPU 和 GPU 同步

DirectML 本身不会提交在 GPU 上执行任何工作。 相反， [ **IDMLCommandRecorder::RecordDispatch**方法](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch)*记录*调度到更高版本执行的命令列表的工作。 你的应用程序必须先关闭然后通过调用提交执行其命令列表然后[ **ID3D12CommandQueue::ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)，如同处理任何 Direct3D 12 命令列表。

DirectML 本身不会提交在 GPU 上执行任何工作，因为它也不会创建任何界定，也不代表你执行任何形式的 CPU/GPU 同步。 它负责应用程序以使用适当的 Direct3D 12 基元要等待的提交的工作与在 GPU 上完成执行必要。 有关详细信息，请参阅[ **ID3D12Fence** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12fence)并[ **ID3D12CommandQueue::Signal**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-signal)。

## <a name="see-also"></a>另请参阅

* [执行和同步命令列表](/windows/desktop/direct3d12/executing-and-synchronizing-command-lists)
* [处理错误和 DirectML 中的删除设备](dml-errors.md)