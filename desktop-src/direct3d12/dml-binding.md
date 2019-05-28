---
title: DirectML 中的绑定
description: 在 DirectML，绑定引用资源连接到要在初始化期间使用 GPU 的管道和机器学习运算符的执行。
ms.custom: 19H1
ms.topic: article
ms.date: 02/01/2019
ms.openlocfilehash: 76a2b0632c90a3a5d0e7afea2d87971436a83bc4
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224014"
---
# <a name="binding-in-directml"></a>DirectML 中的绑定

在 DirectML，*绑定*是指资源连接到要在初始化期间使用 GPU 的管道和你的机器学习运算符的执行。 这些资源可以是输入和输出 tensors，以及需要的任何临时或永久资源。

本主题介绍绑定的概念性和程序性详细的信息。 我们建议您也完全读取的文档的 Api 的调用，包括参数和备注。

## <a name="important-ideas-in-binding"></a>在绑定中的重要观点

下面的步骤的列表包含与绑定相关的任务的高级说明。 你需要每次执行的时请按照下列步骤[调度](/windows/desktop/api/directml/nn-directml-idmldispatchable)&mdash;可调度是运算符初始值设定项或已编译的运算符。 这些步骤会介绍重要的想法、 结构和 DirectML 绑定中所涉及的方法。

本主题中的后续部分更深入发掘和说明中更多详细信息，这些绑定的任务和说明的代码片段摘自[最小 DirectML 应用程序](dml-min-app.md)的代码示例。

- 调用[ **IDMLDispatchable::GetBindingProperties** ](/windows/desktop/api/directml/nf-directml-idmldispatchable-getbindingproperties)上可调度来确定多少描述符它需求，而且还需要其临时/持久资源。
- 创建 Direct3D 12 描述符堆大小不足以描述符，并将其绑定到管道。
- 调用[ **IDMLDevice::CreateBindingTable** ](/windows/desktop/api/directml/nf-directml-idmldevice-createbindingtable)创建 DirectML 表来表示资源绑定绑定到管道。 使用[ **DML_BINDING_TABLE_DESC** ](/windows/desktop/api/directml/ns-directml-dml_binding_table_desc)结构来描述绑定表，它指向的描述符的子集包括描述符堆中。
- 创建 Direct3D 12 资源作为临时/持久资源，描述它们与[ **DML_BUFFER_BINDING** ](/windows/desktop/api/directml/ns-directml-dml_buffer_binding)并[ **DML_BINDING_DESC** ](/windows/desktop/api/directml/ns-directml-dml_binding_desc)结构，并将其添加到绑定表。
- 如果可调度是一个已编译的运算符，然后作为 Direct3D 12 缓冲区资源创建 tensor 元素的缓冲区。 填充/上传它，描述与该**DML_BUFFER_BINDING**并**DML_BINDING_DESC**结构，并将其添加到绑定表。
- 在调用时作为参数传递绑定表[ **IDMLCommandRecorder::RecordDispatch**](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch)。

## <a name="retrieve-the-binding-properties-of-a-dispatchable"></a>检索可调度的绑定属性

[ **DML_BINDING_PROPERTIES** ](/windows/desktop/api/directml/ns-directml-dml_binding_properties)结构描述可调度的绑定需要 （运算符初始值设定项或已编译的运算符）。 这些绑定相关的属性包括应绑定到可调度，以及以字节为单位的所需的任何临时和/或永久资源大小的说明符的数目。

> [!NOTE]
> 即使对于相同类型的多个运算符，不要进行假设它们具有相同的绑定要求。 每个初始值设定项和您创建的运算符的查询的绑定属性。

调用[ **IDMLDispatchable::GetBindingProperties** ](/windows/desktop/api/directml/nf-directml-idmldispatchable-getbindingproperties)检索**DML_BINDING_PROPERTIES**。

```cppwinrt
winrt::com_ptr<::IDMLCompiledOperator> dmlCompiledOperator;
// Code to create and compile a DirectML operator goes here.

DML_BINDING_PROPERTIES executeDmlBindingProperties{
    dmlCompiledOperator->GetBindingProperties()
};

winrt::com_ptr<::IDMLOperatorInitializer> dmlOperatorInitializer;
// Code to create a DirectML operator initializer goes here.

DML_BINDING_PROPERTIES initializeDmlBindingProperties{
    dmlOperatorInitializer->GetBindingProperties()
};

UINT descriptorCount = ...
```

`descriptorCount`此处检索的值确定 （最小值） 的大小和在接下来两个步骤中创建的绑定表的描述符堆。

**DML_BINDING_PROPERTIES**还包含`TemporaryResourceSize`成员，它是以字节为单位必须绑定到此可调度对象的绑定表的临时资源的最小大小。 值为零意味着不需要临时资源。

和一个`PersistentResourceSize`成员，它是以字节为单位的持久资源，必须绑定到此可调度对象的绑定表的最小大小。 值为零意味着持久资源不是必需的。 持久资源，如果需要必须提供在已编译的运算符 （它绑定到的位置作为运算符初始值设定项的输出） 的初始化过程中以及在执行过程。 还有更多有关这本主题中的更高版本。 仅编译的运算符具有持久性资源 — 运算符初始值设定项将始终返回此成员的值为 0。

如果您调用**IDMLDispatchable::GetBindingProperties**之前和之后调用的运算符初始值设定项[ **IDMLOperatorInitializer::Reset**](/windows/desktop/api/directml/nf-directml-idmloperatorinitializer-reset)，然后两个绑定检索到的属性集并非一定要相同。

## <a name="describe-create-and-bind-a-descriptor-heap"></a>描述、 创建和绑定描述符堆

根据描述符，由你负责开始和结束与描述符堆本身。 DirectML 本身负责创建和管理在你提供的堆内的描述符。

因此，使用[ **D3D12_DESCRIPTOR_HEAP_DESC** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_descriptor_heap_desc)结构来描述堆大小不足以说明符的数目，可调度的需求。 然后创建与它[ **ID3D12Device::CreateDescriptorHeap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createdescriptorheap)。 和最后，调用[ **ID3D12GraphicsCommandList::SetDescriptorHeaps** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps)要绑定到管道的描述符堆。

```cppwinrt
winrt::com_ptr<::ID3D12DescriptorHeap> d3D12DescriptorHeap;

D3D12_DESCRIPTOR_HEAP_DESC descriptorHeapDescription{};
descriptorHeapDescription.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;
descriptorHeapDescription.NumDescriptors = descriptorCount;
descriptorHeapDescription.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;

winrt::check_hresult(
    d3D12Device->CreateDescriptorHeap(
        &descriptorHeapDescription,
        _uuidof(d3D12DescriptorHeap),
        d3D12DescriptorHeap.put_void()
    )
);

std::array<ID3D12DescriptorHeap*, 1> d3D12DescriptorHeaps{ d3D12DescriptorHeap.get() };
d3D12GraphicsCommandList->SetDescriptorHeaps(
    static_cast<UINT>(d3D12DescriptorHeaps.size()),
    d3D12DescriptorHeaps.data()
);
```

## <a name="describe-and-create-a-binding-table"></a>描述并创建一个绑定表

DirectML 绑定表表示绑定到的管道调度要使用的资源。 这些资源可能是输入和输出 tensors （或其他参数） 的运算符来说，或者它们可能是持久性和临时的各种资源的使用中的调度工作原理。

使用[ **DML_BINDING_TABLE_DESC** ](/windows/desktop/api/directml/ns-directml-dml_binding_table_desc)结构来描述绑定表，其中包括可调度绑定表将为其表示绑定和描述符的范围 (从你刚刚创建的描述符堆） 想要请参阅 （和写入其 DirectML 可能描述符） 的绑定表。 `descriptorCount`值 （我们在第一步中检索到的绑定属性之一） 告诉我们哪些最小大小是，在描述符的调度对象所需的绑定表。 在这里，我们使用值，以指示 DirectML 允许将写入我们堆，从这两个提供 CPU 和 GPU 描述符开头的说明符的最大数目，处理。

然后调用[ **IDMLDevice::CreateBindingTable** ](/windows/desktop/api/directml/nf-directml-idmldevice-createbindingtable)创建 DirectML 绑定表。 在后续步骤中，我们创建了进一步的调度资源后我们将添加这些资源到绑定表。

而不是传入**DML_BINDING_TABLE_DESC**到此调用时，可以将传递`nullptr`，指示一个空绑定表。

```cppwinrt
DML_BINDING_TABLE_DESC dmlBindingTableDesc{};
dmlBindingTableDesc.Dispatchable = dmlOperatorInitializer.get();
dmlBindingTableDesc.CPUDescriptorHandle = d3D12DescriptorHeap->GetCPUDescriptorHandleForHeapStart();
dmlBindingTableDesc.GPUDescriptorHandle = d3D12DescriptorHeap->GetGPUDescriptorHandleForHeapStart();
dmlBindingTableDesc.SizeInDescriptors = descriptorCount;

winrt::com_ptr<::IDMLBindingTable> dmlBindingTable;
winrt::check_hresult(
    dmlDevice->CreateBindingTable(
        &dmlBindingTableDesc,
        __uuidof(dmlBindingTable),
        dmlBindingTable.put_void()
    )
);
```

在其中 DirectML 写入描述符堆的顺序是未指定的因此你的应用程序必须注意不要覆盖由绑定表包装的描述符。 提供的 CPU 和 GPU 描述符句柄可能来自不同的堆，但是这样，它将应用程序负责确保，整个描述符范围引用了 CPU 描述符句柄被复制到由 GPU 引用的范围在执行使用该绑定表之前的描述符句柄。 描述符堆句柄提供从其必须具有类型**D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV**。 此外，引用堆`GPUDescriptorHandle`必须是一个着色器可见描述符堆。

你可以重置用于删除已添加到它，同时更改在其初始设置任何属性在所有资源的绑定表**DML_BINDING_TABLE_DESC** (要包装新的描述符，范围，或要重新将其用于不同可调度)。 只需进行更改描述的结构，并调用[ **IDMLBindingTable::Reset**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-reset)。

```cppwinrt
dmlBindingTableDesc.Dispatchable = pIDMLCompiledOperator.get();

winrt::check_hresult(
    pIDMLBindingTable->Reset(
        &dmlBindingTableDesc
    )
);
```

## <a name="describe-and-bind-any-temporarypersistent-resources"></a>描述并将任何临时/持久资源绑定

**DML_BINDING_PROPERTIES**结构我们时填充我们[检索的绑定属性](#retrieve-the-binding-properties-of-a-dispatchable)的我们可调度包含以字节为单位的任何临时和/或永久资源的大小，可调度的需求。 如果这些大小的任何一个为非零，则创建 Direct3D 12 缓冲区资源并将其添加到绑定表。

在下面的代码示例中，我们将创建临时资源 (`temporaryResourceSize`大小字节) 为可调度。 我们将介绍我们想要将绑定资源，然后我们将该绑定添加到绑定表。

由于我们要绑定一个缓冲区资源，我们将介绍与绑定[ **DML_BUFFER_BINDING** ](/windows/desktop/api/directml/ns-directml-dml_buffer_binding)结构。 在该结构中，我们指定 Direct3D 12 缓冲区资源 (资源必须具有维度[ **D3D12_RESOURCE_DIMENSION_BUFFER**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_dimension))，以及一个偏移量-和-大小的缓冲区。 还有可能用于描述绑定的缓冲区数组 （而不是单个缓冲区），并[ **DML_BUFFER_ARRAY_BINDING** ](/windows/desktop/api/directml/ns-directml-dml_buffer_array_binding)结构存在用于此目的。

若要抽象出缓冲区绑定和缓冲区数组绑定之间的区别，我们使用[ **DML_BINDING_DESC** ](/windows/desktop/api/directml/ns-directml-dml_binding_desc)结构。 可以设置`Type`的成员**DML_BINDING_DESC**至任一[ **DML_BINDING_TYPE_BUFFER** ](/windows/desktop/api/directml/ne-directml-dml_binding_type)或**DML_BINDING_TYPE_BUFFER_ARRAY**. 然后，可以设置`Desc`成员为指向任一**DML_BUFFER_BINDING**或设置为**DML_BUFFER_ARRAY_BINDING**，具体取决于`Type`。

我们正在处理临时资源在此示例中，因此我们将其添加到绑定表通过调用[ **IDMLBindingTable::BindTemporaryResource**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindtemporaryresource)。

```cppwinrt
D3D12_HEAP_PROPERTIES defaultHeapProperties{ CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT) };
winrt::com_ptr<::ID3D12Resource> temporaryBuffer;

D3D12_RESOURCE_DESC temporaryBufferDesc{ CD3DX12_RESOURCE_DESC::Buffer(temporaryResourceSize) };
winrt::check_hresult(
    d3D12Device->CreateCommittedResource(
        &defaultHeapProperties,
        D3D12_HEAP_FLAG_NONE,
        &temporaryBufferDesc,
        D3D12_RESOURCE_STATE_COMMON,
        nullptr,
        __uuidof(temporaryBuffer),
        temporaryBuffer.put_void()
    )
);

DML_BUFFER_BINDING bufferBinding{ temporaryBuffer.get(), 0, temporaryResourceSize };
DML_BINDING_DESC bindingDesc{ DML_BINDING_TYPE_BUFFER, &bufferBinding };
dmlBindingTable->BindTemporaryResource(&bindingDesc);
```

临时资源 （如果需要） 是运算符的在内部执行期间使用，因此不需要关心其内容的临时内存。 也需要在调用后将其保留[ **IDMLCommandRecorder::RecordDispatch** ](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch)在 GPU 上完成。 这意味着你的应用程序可能会发布或覆盖之间的已编译的运算符的调度临时资源。 提供的缓冲区范围要绑定为临时资源必须具有对齐到其起始偏移量[ **DML_TEMPORARY_BUFFER_ALIGNMENT**](/windows/desktop/direct3d12/directml_constants)。 堆基础缓冲区的类型必须是**D3D12_HEAP_TYPE_DEFAULT**。

如果可调度报告其多生存期较长的持久资源的非零大小，不过，该过程则稍有不同。 应创建一个缓冲区，并描述绑定遵循相同模式，如上所示。 但将其添加到通过调用运算符初始值设定项的绑定表[ **IDMLBindingTable::BindOutputs**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindoutputs)，因为它是运算符初始值设定项的作业，以初始化持久资源。 然后将其添加到已编译的操作员所在的绑定表通过调用[ **IDMLBindingTable::BindPersistentResource**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindpersistentresource)。 请参阅[最小 DirectML 应用程序](dml-min-app.md)若要查看此工作流操作中的代码示例。 永久资源的内容和生存期必须保留，只要编译的运算符执行。 也就是说，如果操作员需要持久资源，则你的应用程序在初始化过程中，并随后必须提供它还提供它为所有将来都执行的运算符，但不修改其内容。 DirectML 通常使用持久资源来查找表或重复使用的运算符的初始化过程中计算其他长期数据存储上以后再执行该运算符。 提供的缓冲区范围要绑定的持久缓冲区必须拥有对齐到其起始偏移量[ **DML_PERSISTENT_BUFFER_ALIGNMENT**](/windows/desktop/direct3d12/directml_constants)。 堆基础缓冲区的类型必须是**D3D12_HEAP_TYPE_DEFAULT**。

## <a name="describe-and-bind-any-tensors"></a>描述并将绑定任何 tensors

如果您正在处理使用已编译的运算符 （而不使用运算符初始值设定项)，然后需要将 （适用于 tensors 和其他参数） 的输入和输出资源绑定到操作员的绑定表。 绑定数必须完全匹配运算符，包括可选 tensors 的输入的数目。 特定输入和输出 tensors 和运算符采用其他参数主题中记录为该运算符 (例如， [ **DML_ELEMENT_WISE_IDENTITY_OPERATOR_DESC**](/windows/desktop/api/directml/ns-directml-dml_element_wise_identity_operator_desc))。

Tensor 资源是包含 tensor 的单个元素值的缓冲区。 上传并读取返回此类缓冲区向/从 GPU 使用正则 Direct3D 12 技术 ([上载资源](/windows/desktop/direct3d12/uploading-resources)并[通过缓冲区读取后的数据](/windows/desktop/direct3d12/readback-data-using-heaps))。 请参阅[最小 DirectML 应用程序](dml-min-app.md)若要查看操作中的这些技术的代码示例。

最后，介绍你的输入和输出资源绑定与**DML_BUFFER_BINDING**并**DML_BINDING_DESC**结构，并将它们添加到具有对的调用已编译的运算符绑定表[**IDMLBindingTable::BindInputs** ](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindinputs)并[ **IDMLBindingTable::BindOutputs**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindoutputs)。 当您调用**IDMLBindingTable::Bind\*** 方法，则 DirectML 会将一个或多个描述符写入到的范围内的 CPU 描述符。

```cppwinrt
DML_BUFFER_BINDING inputBufferBinding{ inputBuffer.get(), 0, tensorBufferSize };
DML_BINDING_DESC inputBindingDesc{ DML_BINDING_TYPE_BUFFER, &inputBufferBinding };
dmlBindingTable->BindInputs(1, &inputBindingDesc);

DML_BUFFER_BINDING outputBufferBinding{ outputBuffer.get(), 0, tensorBufferSize };
DML_BINDING_DESC outputBindingDesc{ DML_BINDING_TYPE_BUFFER, &outputBufferBinding };
dmlBindingTable->BindOutputs(1, &outputBindingDesc);
```

创建一个 DirectML 运算符中的步骤之一 (请参阅[ **IDMLDevice::CreateOperator**](/windows/desktop/api/directml/nf-directml-idmldevice-createoperator)) 是声明一个或多个[ **DML_BUFFER_TENSOR_DESC** ](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc)结构来描述的 tensor 数据缓冲区的运算符采用并返回。 可以选择指定 tensor 缓冲区的类型和大小，以及[ **DML_TENSOR_FLAG_OWNED_BY_DML** ](/windows/desktop/api/directml/ns-directml-dml_tensor_flags)标志。

**DML_TENSOR_FLAG_OWNED_BY_DML**指示应拥有和管理的 DirectML tensor 数据。 DirectML 运算符，初始化期间创建的 tensor 数据副本，并将其存储在持久资源。 这允许 DirectML 执行到窗体中其他、 更高效的 tensor 数据重新格式化。 设置此标志可能会提高性能，但它通常仅适用于 tensors 运算符 (例如，权重 tensors) 的生存期内不会更改其数据。 该标志仅用于输入 tensors。 当特定 tensor 说明上设置了标志时，相应 tensor 必须绑定到绑定表中，初始化过程中运算符，而不是在执行 （这将导致错误）。 这就是其中 tensor 应在执行期间，而不是在初始化绑定的默认行为 （如果没有 DML_TENSOR_FLAG_OWNED_BY_DML 标志的行为），相反。 时提供的运算符初始值设定项将 tensor 的数据，它是合法的绑定上传而不是默认堆，因为 DirectML 使数据的副本。 在所有其他情况下，绑定到 DirectML 的所有资源必须都是默认堆资源。

有关详细信息，请参阅[ **IDMLBindingTable::BindInputs** ](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindinputs)并[ **IDMLBindingTable::BindOutputs**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindoutputs)。

## <a name="execute-the-dispatchable"></a>执行可调度

在调用时作为参数传递绑定表[ **IDMLCommandRecorder::RecordDispatch**](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch)。

当你使用的绑定表在调用**IDMLCommandRecorder::RecordDispatch**，DirectML 将相应的 GPU 描述符绑定到管道。 CPU 和 GPU 描述符句柄不需要指向描述符堆中相同的条目，但是这样，它将应用程序负责确保，整个描述符范围引用了 CPU 描述符句柄被复制到范围通过 GPU 描述符处理之前执行，使用该绑定表引用。

```cppwinrt
winrt::com_ptr<::ID3D12GraphicsCommandList> d3D12GraphicsCommandList;
// Code to create a Direct3D 12 command list goes here.

winrt::com_ptr<::IDMLCommandRecorder> dmlCommandRecorder;
// Code to create a DirectML command recorder goes here.

dmlCommandRecorder->RecordDispatch(
    d3D12GraphicsCommandList.get(),
    dmlOperatorInitializer.get(),
    dmlBindingTable.get()
);
```

最后，关闭在 Direct3D 12 命令列表中，并将其提交进行执行，就像任何其他命令列表。

在执行前**RecordDispatch**必须在 GPU 上转换到的所有绑定的资源**D3D12_RESOURCE_STATE_UNORDERED_ACCESS**状态，或为隐式提升为状态**D3D12_RESOURCE_STATE_UNORDERED_ACCESS**，如**D3D12_RESOURCE_STATE_COMMON**。 此调用完成后，资源将保留在**D3D12_RESOURCE_STATE_UNORDERED_ACCESS**状态。 唯一的例外用于上传堆绑定执行运算符初始值设定项时，在一个或多个 tensors 同时还提供**DML_TENSOR_FLAG_OWNED_BY_DML**标志设置。 在这种情况下，任何上载堆绑定中必须是输入**D3D12_RESOURCE_STATE_GENERIC_READ**状态，并将保持的状态，根据需要由所有上传堆。 如果**DML_EXECUTION_FLAG_DESCRIPTORS_VOLATILE**编译运算符，则所有绑定必须绑定表之前设置时，未设置**RecordDispatch**是调用，否则该行为是未定义。 否则为如果支持操作员[后期绑定](#optionally-specify-late-bound-operator-bindings)，则可能会延迟绑定的资源，直到 Direct3D 12 命令列表提交给执行命令队列。

**RecordDispatch**逻辑上的作用类似于调用 [**ID3D12GraphicsCommandList::Dispatch**] （/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-调度。 在这种情况下，无序的访问视图 (UAV) 障碍是确保正确顺序调度之间的数据依赖关系是否所必需的。 此方法不会插入 UAV 障碍，对输入和输出资源。 你的应用程序必须确保正确 UAV 障碍任何输入上执行，如果它们的内容依赖于上游的调度，以及任何输出是否存在取决于这些输出的下游调度。

## <a name="lifetime-and-synchronization-of-descriptors-and-binding-table"></a>生存期和描述符和绑定表的同步

DirectML 中绑定的良好的心理模型是在后台 DirectML 绑定表本身是创建和管理在你提供的描述符堆内的无序的访问视图 (UAV) 描述符。 因此，所有常用的 Direct3D 12 规则适用围绕同步访问该堆和其描述符。 它负责应用程序的执行之间使用的绑定表的 CPU 和 GPU 工作的正确同步。

绑定表不能覆盖描述符描述符正在使用中 （由以前的框架，例如）。 因此，如果你想要重用已绑定到描述符堆 （例如，通过调用 Bind * 再次绑定表指向它，或通过手动覆盖描述符堆），然后你应等待调度的当前正在使用到描述符堆完成在 GPU 上执行。 绑定表不会继续在它写入描述符堆上的强引用，因此直到使用该绑定表的所有工作已都完成在 GPU 上的执行，不能释放后备着色器可见描述符堆。

另一方面，绑定表有指定，并且管理描述符堆，而表不本身*包含*任何内存。 因此，可能会释放或重置绑定表后调用的任何时间[ **IDMLCommandRecorder::RecordDispatch** ](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch)与其 (无需等待上 GPU，因此长时间以完成该调用基础描述符仍将有效）。

绑定表不会保留强引用上使用该绑定的任何资源&mdash;你的应用程序必须确保资源不会由 GPU 删除仍在使用时。 此外，绑定表不是线程安全&mdash;你的应用程序不得调用的方法绑定表上同时从不同的线程，而不进行同步。

并考虑，在任何情况下重新绑定时是必需的仅更改绑定的资源。 如果无需更改绑定的资源，则可以在启动时，一次绑定并传递相同的绑定表每次调用**RecordDispatch**。

为交错执行机器学习和呈现的工作负荷，只需确保每个帧的绑定表指向不是在 GPU 上使用描述符堆的范围。

## <a name="optionally-specify-late-bound-operator-bindings"></a>（可选） 指定后期绑定运算符绑定

如果您正在处理使用已编译的运算符 （而不使用运算符初始值设定项)，则必须指定运算符的后期绑定的选项。 而无需后期绑定，必须绑定表上设置的所有绑定到命令列表记录运算符之前。 凭借后期绑定，可以设置 （或更改） 您已录制到命令列表之前已提交到命令队列, 的运算符上的绑定。

若要指定后期绑定，请调用[ **IDMLDevice::CompileOperator** ](/windows/desktop/api/directml/nf-directml-idmldevice::compileoperator)与`flags`参数[ **DML_EXECUTION_FLAG_DESCRIPTORS_VOLATILE**](/windows/desktop/api/directml/ne-directml-dml_execution_flags).
