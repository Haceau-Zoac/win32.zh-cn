---
title: DirectML 中的绑定
description: 描述障碍，以及如何可以与它们配合中 DirectML 正确性的好处。
ms.custom: 19H1
ms.topic: article
ms.date: 03/12/2019
ms.openlocfilehash: 52397120378f19868bb832f8b976ec7d504e7b9f
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224017"
---
# <a name="uav-barriers-and-resource-state-barriers-in-directml"></a>UAV 障碍和资源状态中 DirectML 的障碍

## <a name="unordered-access-view-uav-barrier-requirements"></a>无序的访问视图 (UAV) 屏障要求

### <a name="uav-barriers-in-direct3d-12"></a>Direct3D 12 中的 UAV 障碍

在 Direct3D 12 中，同一个命令列表中相邻的计算着色器调度有权在 GPU 上并行执行，除非它们干扰的无序的访问视图 (UAV) 屏障的同步。 这可以通过增加 GPU 硬件的利用率提高性能。 但是，默认情况下，无需 UAV 屏障，使用两个相邻调度的并行执行可能导致争用条件存在两个调度; 之间的数据依赖项或如果这两个调度执行 UAV 将写入到同一区域的内存。

UAV 屏障强制所有先前所提交的调度到在 GPU 上完成执行之前可能会开始后续调度。 UAV 障碍，用于在相同的命令列表，以避免数据争用调度之间的同步。 可以通过使用发出 UAV 屏障[ **ID3D12GraphicsCommandList::ResourceBarrier**方法](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier)。

### <a name="uav-barriers-in-directml"></a>在 DirectML UAV 障碍

在 DirectML，运算符将被分派是在 Direct3D 12 中调度计算着色器的方式类似的方式。 也就是说，运算符的相邻调度有权在 GPU 上并行执行，除非存在它们之间干预 UAV 屏障。 典型的机器学习模型包含其运算符; 之间的数据依赖项例如，一个运算符的输出源到另一个输入。 因此，它是必须使用 UAV 屏障来正确同步调度。

它将只会从 （永远不会对读写） 的 DirectML 保证输入 tensors。 它还确保它将永远不会制造写入到输出 tensor tensor 的范围之外[ **DML_BUFFER_TENSOR_DESC::TotalTensorSizeInBytes** ](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc)成员。 这意味着，可以通过只查看操作员的输入和输出绑定想到有关数据中 DirectML 运算符之间的依赖关系。

例如，通过这些保证可以调度绑定同一区域的资源作为输入，而无需干预的 UAV 屏障发出的两个运算符。 这是始终安全，因为 DirectML 永远不会写入到输入 tensors。 另举一例，它始终是安全将两个并发运算符调度输出 tensors 绑定到同一 Direct3D 12 资源 （只要其 tensors 不重叠），这是因为 DirectML 永远不会写入到的 tensor 边界之外 （如所 tensor 的定义**DML_BUFFER_TENSOR_DESC::TotalTensorSizeInBytes**)。

UAV 障碍是同步的一种形式，因为不必要地使用 UAV 屏障可能对性能产生负面影响。 因此，它是最适合您要使用 UAV 屏障正确同步调度命令列表中的所需的最小数目。

### <a name="example-1"></a>示例 1

在以下示例中，一个卷积运算符输出送入 ReLU 激活，跟批处理规范化。

```console
    CONVOLUTION (conv1)
         |
  ACTIVATION_RELU (relu1)
         |
BATCH_NORMALIZATION (batch1)
```

由于所有三个运算符之间存在数据依赖项，你将需要每个连续的调度之间 UAV 屏障 (请参阅[ **IDMLCommandRecorder::RecordDispatch**](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch))。

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

此处的池的输出会提供给两个卷积，其输出然后将其连接在一起使用 JOIN 运算符。 数据依赖项之间存在`pool1`并且两个`conv1`和`conv2`; 以及如之间同时`conv1`并`conv2`和`join1`。 下面是一个有效的方式执行此图形。

1. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**pool1**`)`
2. `d3d12CommandList->ResourceBarrier(`**UAV 屏障**`)`
3. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**conv1**`)`
4. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**conv2**`)`
5. `d3d12CommandList->ResourceBarrier(`**UAV 屏障**`)`
6. `dmlCommandRecorder->RecordDispatch(d3d12CommandList, `**join1**`)`

在这种情况下，`conv1`和`conv2`可以并发执行 GPU，这样可能会改善性能上。

## <a name="resource-barrier-state-requirements"></a>障碍状态的资源要求

调用方，它是由你负责确保 Direct3D 12 的所有资源都都处于正确的资源屏障状态在 GPU 上执行 DirectML 调度之前。 DirectML 不会代表你执行任何转换的障碍。

在执行前[ **IDMLCommandRecorder::RecordDispatch** ](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch)必须在 GPU 上转换到的所有绑定的资源**D3D12_RESOURCE_STATE_UNORDERED_ACCESS**状态，或为隐式提升为状态**D3D12_RESOURCE_STATE_UNORDERED_ACCESS**，如**D3D12_RESOURCE_STATE_COMMON**。 此调用完成后，资源将保留在**D3D12_RESOURCE_STATE_UNORDERED_ACCESS**状态。 有关更多详细信息，请参阅[绑定中 DirectML](dml-binding.md)。

## <a name="see-also"></a>另请参阅

* [使用资源屏障来同步在 Direct3D 12 中的资源状态](/windows/desktop/direct3d12/using-resource-barriers-to-synchronize-resource-states-in-direct3d-12)
* [DirectML 中的绑定](dml-binding.md)
