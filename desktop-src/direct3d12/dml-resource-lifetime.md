---
title: 资源生存期和同步
description: DirectML 应用程序必须准确地管理对象生存期和 CPU 与 GPU 之间的同步，以避免未定义的行为。
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

与 Direct3D 12 一样，DirectML 应用程序必须准确地管理对象生存期和 CPU 与 GPU 之间的同步，以避免未定义的行为。 DirectML 与 Direct3D 12 采用的资源生存期模型相同。

- DirectML 使用强引用计数维护两个 CPU 对象之间的生存期依赖关系。 应用程序无需手动管理 CPU 生存期依赖关系。 例如，每个子设备包含对其父设备的强引用。
- 无法自动管理 GPU 对象之间的生存期依赖关系或 CPU 和 GPU 之间的依赖关系。 应用程序负责确保 GPU 资源的生存期至少延续至 GPU 上使用该资源的所有作业完成执行。

## <a name="directml-devices"></a>DirectML 设备

DirectML 设备是线程安全的无状态工厂对象。 每个子设备（请参阅 [IDMLDeviceChild  ](/windows/desktop/api/directml/nn-directml-idmldevicechild)）都包含对其父 DirectML 设备（请参阅 [IDMLDevice  ](/windows/desktop/api/directml/nn-directml-idmldevice)）的强引用。 这意味着，你始终可以从任何子设备接口检索父设备接口。

相应地，DirectML 设备包含对用于创建它的 Direct3D 12 设备的强引用（请参阅 [ID3D12Device  ](/windows/desktop/api/d3d12/nn-d3d12-id3d12device) 和派生接口）。

由于 DirectML 设备是无状态设备，因此它属于隐式线程安全设备。 你可以同时从多个线程调用 DirectML 设备上的方法，而无需执行外部同步。

但是，不同于 Direct3D 12 设备的是，DirectML 设备不是单一实例对象。 你可根据需要创建多个 DirectML 设备。 不过，你不能混合和匹配属于不同设备的子设备。 例如，[IDMLBindingTable  ](/windows/desktop/api/directml/nn-directml-idmlbindingtable) 和 [IDMLCompiledOperator  ](/windows/desktop/api/directml/nn-directml-idmlcompiledoperator) 属于两种子设备类型（两个接口均直接或间接派生自 IDMLDeviceChild  ）。 此外，当运算符和绑定表属于不同的 DirectML 设备实例时，不可以使用绑定表 (  IDMLBindingTable) 来绑定运算符 (  IDMLCompiledOperator)。

由于 DirectML 设备不是单一实例对象，因此要基于每台设备执行设备删除，而非如同使用 Direct3D 12 设备时那样作为进程范围事件执行删除。 有关详细信息，请参阅[在 DirectML 中处理错误和设备删除](dml-errors.md)。

## <a name="lifetime-requirements-of-gpu-resources"></a>GPU 资源的生存期要求

与 Direct3D 12 一样，DirectML 无法自动在 CPU 和 GPU 之间同步；同时，当 GPU 使用资源时，它也无法自动使它们保持活动状态。 相反，应用程序要承担上述职责。

执行包含 DirectML 调度的命令列表时，应用程序必须确保，在 GPU 上使用这些资源的所有作业完成执行前 GPU 资源保持活动状态。

对于 DirectML 运算符的 [IDMLCommandRecorder::RecordDispatch  ](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch)，其中包含以下对象。

- 正在执行的 [IDMLCompiledOperator  ](/windows/desktop/api/directml/nn-directml-idmlcompiledoperator)（若正在执行运算符初始化，则为 [IDMLOperatorInitializer  ](/windows/desktop/api/directml/nn-directml-idmloperatorinitializer)）。
- 支持用于绑定运算符的绑定表的 IDMLCompiledOperator  。
- 绑定为运算符输入/输出的 [ID3D12Resource  ](/windows/desktop/api/d3d12/nn-d3d12-id3d12resource) 对象。
- 绑定为永久资源和临时资源的 ID3D12Resource 对象（如果适用）。 
- 支持命令列表本身的 [ID3D12CommandAllocator  ](/windows/desktop/api/d3d12/nn-d3d12-id3d12commandallocator)。

只有部分 DirectML 接口表示 GPU 资源。 例如，当 GPU 上使用绑定表的所有调度都完成执行时，它才需要保持活动状态。  因为绑定表本身不拥有任何 GPU 资源。 而描述符堆拥有这些。 因此，在执行完成前，是基础描述符堆对象而非绑定表本身必须保持活动状态。 

Direct3D 12 包含类似的概念。 在 GPU 上使用命令分配器的所有执行完成前，它必须保持活动状态；因为它拥有 GPU 内存。  但是，命令列表本身不拥有 GPU 内存，因此在提交以执行它时即可重置或释放它。 

在 DirectML 中，编译的运算符 ([IDMLCompiledOperator  ](/windows/desktop/api/directml/nn-directml-idmlcompiledoperator)) 和运算符初始值 ([IDMLOperatorInitializer  ](/windows/desktop/api/directml/nn-directml-idmloperatorinitializer)) 均直接拥有 GPU 资源，因此在 GPU 上使用这些资源的所有调度完成执行前，它们必须保持活动状态。 此外，应用程序必须同样要使所使用的任何 Direct3D 12 资源（如命令分配器、描述符堆、缓冲）保持活动状态。

若在 GPU 使用对象期间提前释放了该对象，结果将是未定义的行为，并且可能会删除设备或引发其他错误。

## <a name="cpu-and-gpu-synchronization"></a>CPU 和 GPU 同步

DirectML 本身不会提交任何作业来让 GPU 执行。 相反，[IDMLCommandRecorder::RecordDispatch  方法](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch)会将该作业调度记录到命令列表，以便稍后执行它。  然后，如同对待任何 Direct3D 12 命令列表一般，应用程序必须调用 [ID3D12CommandQueue::ExecuteCommandLists  ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists) 来关闭并提交其命令列表，以便执行它。

由于 DirectML 本身不会提交任何作业来让 GPU 执行，因此它也不会创建任何围墙，并且也不会以你的身份执行任何形式的 CPU/GPU 同步。 必要时，由应用程序负责使用适当的 Direct3D 12 基元，来等待 GPU 上已提交的作业完成执行。 有关详细信息，请参阅 [ID3D12Fence  ](/windows/desktop/api/d3d12/nn-d3d12-id3d12fence) 和 [ID3D12CommandQueue::Signal  ](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-signal)。

## <a name="see-also"></a>另请参阅

* [执行和同步命令列表](/windows/desktop/direct3d12/executing-and-synchronizing-command-lists)
* [在 DirectML 中处理错误和设备删除](dml-errors.md)