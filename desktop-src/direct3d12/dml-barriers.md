---
title: DirectML 中的绑定
description: 描述屏障的正确性好处，以及在 DirectML 中的使用方式。
ms.custom: Windows 10 May 2019 Update
ms.localizationpriority: high
ms.topic: article
ms.date: 04/19/2019
ms.openlocfilehash: 09107959853cd91b47c99e94fe6e63e47ae36280
ms.sourcegitcommit: 2d531328b6ed82d4ad971a45a5131b430c5866f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2019
ms.locfileid: "71006061"
---
# <a name="uav-barriers-and-resource-state-barriers-in-directml"></a>DirectML 中的 UAV 屏障和资源状态屏障

## <a name="unordered-access-view-uav-barrier-requirements"></a>无序访问视图 (UAV) 屏障要求

### <a name="uav-barriers-in-direct3d-12"></a>Direct3D 12 中的 UAV 屏障

在 Direct3D 12 中，可在 GPU 上并行执行同一命令列表内的相邻计算着色器调度，除非它们与干预无序访问视图 (UAV) 屏障同步。 这可通过提高 GPU 硬件的利用率来提高性能。 但在默认情况下，如果不使用 UAV 屏障，而两个调度之间存在数据依赖关系，或者两个调度都执行 UAV 写入到相同的内存区域，则两个相邻调度的并行执行会产生争用条件。

在随后的调度开始之前，UAV 屏障会强制之前提交的所有调度在 GPU 上完成执行。 UAV 屏障用于在同一命令列表上的调度之间进行同步，以避免数据争用。 可使用 [ID3D12GraphicsCommandList::ResourceBarrier方法](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) 发出 UAV 屏障。

### <a name="uav-barriers-in-directml"></a>DirectML 中的 UAV 屏障

在 DirectML 中，调度运算符的方式与在 Direct3D 12 中调度计算着色器的方式类似。 也就是说，可在 GPU 上并行执行运算符的相邻调度，除非它们之间存在干预性的 UAV 屏障。 典型的机器学习模型包含其运算符之间的数据依赖关系；例如，一个运算符的输出会馈送到另一个运算符的输入中。 因此，使用 UAV 屏障来正确同步调度非常重要。

DirectML 可保证它只能从输入张量读取（且永远不会写入这些张量）。 此外，它还保证永不会写入到张量 [DML_BUFFER_TENSOR_DESC::TotalTensorSizeInBytes](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc) 成员范围之外的输出张量。 这意味着可通过仅查看运算符的输入和输出绑定来推断 DirectML 中运算符之间的数据依赖关系。

例如，通过这些保证可调度两个绑定资源的相同区域的运算符作为输入，而不必发出干预性的 UAV 屏障。 这始终安全，因为 DirectML 永不会写入输入张量。 另外一个例子是，将两个并发运算符调度的输出张量绑定到同一个 Direct3D 12 资源（只要它们的张量不重叠）总是安全的，因为 DirectML 永不会在张量的边界外写入（由张量的 DML_BUFFER_TENSOR_DESC::TotalTensorSizeInBytes 定义）。

由于 UAV 屏障是一种同步形式，不必要地使用 UAV 屏障可能会对性能产生负面影响。 因此，最好使用必要的最小数量的 UAV 屏障来正确同步命令列表中的调度。

### <a name="example-1"></a>示例 1

在下面的示例中，将卷积运算符的输出输入 ReLU 激活，然后进行批处理标准化。

```console
    CONVOLUTION (conv1)
         |
  ACTIVATION_RELU (relu1)
         |
BATCH_NORMALIZATION (batch1)
```

由于所有三个运算符之间都存在数据依赖关系，因此每次后续调度之间都需要一个 UAV 屏障（请参阅 [**IDMLCommandRecorder::RecordDispatch**](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch)）。

1. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**conv1**`)`
2. `d3d12CommandList->ResourceBarrier(`**UAV 屏障**`)`
3. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**relu1**`)`
4. `d3d12CommandList->ResourceBarrier(`**UAV 屏障**`)`
5. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**batch1**`)`

### <a name="example-2"></a>示例 2

```console
     MAX_POOLING (pool1)
        /    \
CONVOLUTION  CONVOLUTION
  (conv1)      (conv2)
        \    /
         JOIN (join1)
```

在此处，池的输出被输入到两个卷积中，然后使用 JOIN 运算符将两个卷积的输出串联在一起。 `pool1` 与 `conv1` 和 `conv2` 之间存在数据依赖关系；`conv1` 和 `conv2` 与 `join1` 之间也存在数据依赖关系。 这是执行此图表的一种有效方法。

1. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**pool1**`)`
2. `d3d12CommandList->ResourceBarrier(`**UAV 屏障**`)`
3. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**conv1**`)`
4. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**conv2**`)`
5. `d3d12CommandList->ResourceBarrier(`**UAV 屏障**`)`
6. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**join1**`)`

在这种情况下，`conv1` 和 `conv2` 能够在 GPU 上同时执行，从而提高性能。

## <a name="resource-barrier-state-requirements"></a>资源屏障状态要求

作为调用方，在 GPU 上执行 DirectML 调度之前，你有责任确保所有 Direct3D 12 资源的资源屏障状态均正确。 DirectML 不会代表你执行任何转换屏障。

在 GPU 上执行 [IDMLCommandRecorder::RecordDispatch](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch) 之前，必须将所有绑定资源都转换为 D3D12_RESOURCE_STATE_UNORDERED_ACCESS 状态，或者转换为可隐式升级到 D3D12_RESOURCE_STATE_UNORDERED_ACCESS 的状态（例如 D3D12_RESOURCE_STATE_COMMON）。 此调用完成后，资源仍然处于 D3D12_RESOURCE_STATE_UNORDERED_ACCESS 状态。 有关详细信息，请参阅 [DirectML 中的绑定](dml-binding.md)。

## <a name="see-also"></a>请参阅

* [在 Direct3D 12 中使用资源屏障同步资源状态](/windows/desktop/direct3d12/using-resource-barriers-to-synchronize-resource-states-in-direct3d-12)
* [DirectML 中的绑定](dml-binding.md)
