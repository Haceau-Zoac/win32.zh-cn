---
title: DirectML 中的绑定
description: 在 DirectML 中，绑定是指将资源附加到管道，以供 GPU 在机器学习运算符初始化和执行时使用。
ms.custom: Windows 10 May 2019 Update
ms.localizationpriority: high
ms.topic: article
ms.date: 04/19/2019
ms.openlocfilehash: ab1c997c1d771c39e92688baec42b9b048e17513
ms.sourcegitcommit: 8141395d1bd1cd755d1375715538c3fe714ba179
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67465023"
---
# <a name="binding-in-directml"></a>DirectML 中的绑定

在 DirectML 中，绑定是指将资源附加到管道，以供 GPU 在机器学习运算符初始化和执行时使用。  例如，这些资源可以是输入和输出张量，也可以是运算符需要的任何临时性或永久性资源。

本主题介绍绑定的概念和过程详细信息。 此外，我们建议通篇阅读所要调用的 API 的文档，包括参数和备注。

## <a name="important-ideas-in-binding"></a>绑定中的重要概念

以下步骤列表包含绑定相关的任务的概要说明。 每次执行[可调度对象](/windows/desktop/api/directml/nn-directml-idmldispatchable)时，都需要执行这些步骤 &mdash; 可调度对象是运算符初始值设定项或编译的运算符。 这些步骤会介绍 DirectML 绑定所涉及的重要概念、结构和方法。

本主题中的后续部分将使用摘自[精简 DirectML 应用程序](dml-min-app.md)代码示例的演示性代码片段更深入、更详细地解释这些绑定任务。

- 针对可调度对象调用 [**IDMLDispatchable::GetBindingProperties**](/windows/desktop/api/directml/nf-directml-idmldispatchable-getbindingproperties)，以确定它需要多少个描述符，及其临时性/永久性资源需求。
- 创建对于描述符而言足够大的 Direct3D 12 描述符堆，并将其绑定到管道。
- 调用 [**IDMLDevice::CreateBindingTable**](/windows/desktop/api/directml/nf-directml-idmldevice-createbindingtable) 以创建一个 DirectML 绑定表来表示绑定到管道的资源。 使用 [**DML_BINDING_TABLE_DESC**](/windows/desktop/api/directml/ns-directml-dml_binding_table_desc) 结构以描述绑定表，包括它在描述符堆中指向的描述符子集。
- 创建临时性/永久性资源作为 Direct3D 12 资源，使用 [**DML_BUFFER_BINDING**](/windows/desktop/api/directml/ns-directml-dml_buffer_binding) 和 [**DML_BINDING_DESC**](/windows/desktop/api/directml/ns-directml-dml_binding_desc) 结构对其进行描述，并将其添加到绑定表。
- 如果可调度对象是编译的运算符，则创建张量元素的缓冲区作为 Direct3D 12 缓冲区资源。 填充/上传该资源，使用 **DML_BUFFER_BINDING** 和 **DML_BINDING_DESC** 结构对其进行描述，并将其添加到绑定表。
- 调用 [**IDMLCommandRecorder::RecordDispatch**](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch) 时传递绑定表作为参数。

## <a name="retrieve-the-binding-properties-of-a-dispatchable"></a>检索可调度对象的绑定属性

[**DML_BINDING_PROPERTIES**](/windows/desktop/api/directml/ns-directml-dml_binding_properties) 结构描述可调度对象（运算符初始值设定项或编译的运算符）的绑定需求。 这些绑定相关的属性包括应绑定到可调度对象的描述符数目，以及该对象所需的任何临时性和/或永久性资源的大小（以字节为单位）。

> [!NOTE]
> 即使对于同一类型的多个运算符，也不要假设它们具有相同的绑定要求。 查询所要创建的每个初始值设定项和运算符的绑定属性。

调用 [**IDMLDispatchable::GetBindingProperties**](/windows/desktop/api/directml/nf-directml-idmldispatchable-getbindingproperties) 以检索 **DML_BINDING_PROPERTIES**。

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

此处检索的 `descriptorCount` 值确定描述符堆以及在后两个步骤中创建的绑定表的（最小）大小。

**DML_BINDING_PROPERTIES** 还包含 `TemporaryResourceSize` 成员，该成员是必须绑定到此可调度对象的绑定表的临时资源的最小大小（以字节为单位）。 零值表示不需要临时性资源。

另外还有一个 `PersistentResourceSize` 成员，该成员是必须绑定到此可调度对象的绑定表的永久性资源的最小大小（以字节为单位）。 零值表示不需要永久性资源。 永久性资源（如果需要）必须在初始化编译的运算符（绑定为运算符初始值设定项的输出时）以及在执行期间提供。 本主题稍后会提供更详细的介绍。 只有编译的运算符具有永久性资源 — 运算符初始值设定项始终对此成员返回 0 值。

如果在调用 [**IDMLOperatorInitializer::Reset**](/windows/desktop/api/directml/nf-directml-idmloperatorinitializer-reset) 之前和之后针对运算符初始值设定项调用 **IDMLDispatchable::GetBindingProperties**，则不保证检索到的两个绑定属性集是相同的。

## <a name="describe-create-and-bind-a-descriptor-heap"></a>描述、创建和绑定描述符堆

在描述符方面，你的责任从描述符堆本身开始，并从其结束。 DirectML 本身负责在你提供的堆中创建和管理描述符。

因此，请使用 [**D3D12_DESCRIPTOR_HEAP_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_descriptor_heap_desc) 结构来描述对可调度对象所需描述符数目而言足够大的堆。 然后使用 [**ID3D12Device::CreateDescriptorHeap**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createdescriptorheap) 创建该堆。 最后，调用 [**ID3D12GraphicsCommandList::SetDescriptorHeaps**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps) 将描述符堆绑定到管道。

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

## <a name="describe-and-create-a-binding-table"></a>描述和创建绑定表

DirectML 绑定表表示绑定到管道供可调度对象使用的资源。 这些资源可以是运算符的输入和输出张量（或其他参数），或者是可调度对象使用的各种永久性和临时性资源。

使用 [**DML_BINDING_TABLE_DESC**](/windows/desktop/api/directml/ns-directml-dml_binding_table_desc) 结构描述绑定表，包括其绑定由绑定表表示的可调度对象，以及希望绑定表引用的（和 DirectML 可在其中写入描述符的）描述符（位于刚刚创建的描述符堆中）的范围。 `descriptorCount` 值（在第一个步骤中检索到的绑定属性之一）告知可调度对象所需的绑定表在描述符中的最小大小。 此处，我们将使用该值来指示允许 DirectML 在堆中从提供的 CPU 和 GPU 描述符句柄开头处写入的最大描述符数目。

然后调用 [**IDMLDevice::CreateBindingTable**](/windows/desktop/api/directml/nf-directml-idmldevice-createbindingtable) 创建 DirectML 绑定表。 在后续步骤中为可调度对象创建更多的资源后，我们会将这些资源添加到绑定表。

无需将 **DML_BINDING_TABLE_DESC** 传递给此调用，可以传递 `nullptr` 来指示空绑定表。

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

DirectML 在堆中写入描述符的顺序未指定，因此，应用程序必须注意不要覆盖绑定表所包装的描述符。 提供的 CPU 和 GPU 描述符句柄可能来自不同的堆，但是，在使用此绑定表执行之前，应用程序需负责确保将 CPU 描述符句柄引用的整个描述符范围复制到 GPU 描述符句柄引用的范围。 从中提供句柄的描述符堆的类型必须是 **D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV**。 此外，`GPUDescriptorHandle` 引用的堆必须是着色器可见的描述符堆。

可以重置绑定表以删除在其中添加的任何资源，同时更改在其初始 **DML_BINDING_TABLE_DESC** 中设置的任何属性（以包装新的描述符范围，或将其重复用于不同的可调度对象）。 只需更改描述结构，并调用 [**IDMLBindingTable::Reset**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-reset) 即可。

```cppwinrt
dmlBindingTableDesc.Dispatchable = pIDMLCompiledOperator.get();

winrt::check_hresult(
    pIDMLBindingTable->Reset(
        &dmlBindingTableDesc
    )
);
```

## <a name="describe-and-bind-any-temporarypersistent-resources"></a>描述和绑定任何临时性/永久性资源

[检索可调度对象的绑定属性](#retrieve-the-binding-properties-of-a-dispatchable)时填充的 **DML_BINDING_PROPERTIES** 结构包含该可调度对象所需的任何临时性和/或永久性资源的大小（以字节为单位）。 如果其中的任一大小不为零，则创建 Direct3D 12 缓冲区资源并将其添加到绑定表。

在以下代码示例中，我们将为可调度对象创建一个临时性资源（大小为 `temporaryResourceSize` 字节）。 我们将描述如何绑定资源，然后将该绑定添加到绑定表。

由于我们要绑定单个缓冲区资源，因此将使用 [**DML_BUFFER_BINDING**](/windows/desktop/api/directml/ns-directml-dml_buffer_binding) 结构描述绑定。 在该结构中，指定 Direct3D 12 缓冲区资源（该资源的维度必须是 [**D3D12_RESOURCE_DIMENSION_BUFFER**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_dimension)），以及缓冲区中的偏移量和大小。 还可以描述缓冲区数组（而不是单个缓冲区）的绑定，为此可以使用 [**DML_BUFFER_ARRAY_BINDING**](/windows/desktop/api/directml/ns-directml-dml_buffer_array_binding) 结构。

为了抽象掉缓冲区绑定与缓冲区数组绑定之间的差别，我们将使用 [**DML_BINDING_DESC**](/windows/desktop/api/directml/ns-directml-dml_binding_desc) 结构。 可将 **DML_BINDING_DESC** 的 `Type` 成员设置为 [**DML_BINDING_TYPE_BUFFER**](/windows/desktop/api/directml/ne-directml-dml_binding_type) 或 **DML_BINDING_TYPE_BUFFER_ARRAY**。 然后，可以根据 `Type`，将 `Desc` 成员设置为指向 **DML_BUFFER_BINDING** 或 **DML_BUFFER_ARRAY_BINDING**。

我们将在此示例中处理临时性资源，因此需要通过调用 [**IDMLBindingTable::BindTemporaryResource**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindtemporaryresource) 将其添加到绑定表。

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

临时性资源（如果需要）是执行运算符期间在内部使用的暂用内存，因此不需要考虑其内容。 此外，在 GPU 上完成 [**IDMLCommandRecorder::RecordDispatch**](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch) 调用后，不需要保留该资源。 这意味着，在调度编译的运算符期间，应用程序可能会释放或覆盖临时性资源。 所提供的要绑定为临时性资源的缓冲区范围的起始偏移量必须与 [**DML_TEMPORARY_BUFFER_ALIGNMENT**](/windows/desktop/direct3d12/directml_constants) 对齐。 缓冲区底层的堆的类型必须是 **D3D12_HEAP_TYPE_DEFAULT**。

不过，如果可调度对象为其生存期较长的永久性资源报告了非零大小，则过程会稍有不同。 应该根据上面所示的相同模式创建一个缓冲区并描述一个绑定。 但是，应通过调用 [**IDMLBindingTable::BindOutputs**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindoutputs) 将此资源添加到运算符初始值设定项的绑定表，因为需要由运算符初始值设定项的作业初始化永久性资源。 然后，通过调用 [**IDMLBindingTable::BindPersistentResource**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindpersistentresource) 将此资源添加到编译的运算符的绑定表。 请参阅[精简 DirectML 应用程序](dml-min-app.md)代码示例了解此工作流的运作方式。 永久性资源的内容和生存期必须与编译的运算符一样持久。 也就是说，如果某个运算符需要永久性资源，则应用程序必须在初始化期间提供该资源，以后还必须将它应用到该运算符的所有后续执行，且不能修改其内容。 DirectML 通常使用永久性资源来存储初始化运算符期间计算的，以及将来执行该运算符时重复使用的查找表或其他生存期较长的数据。 所提供的要绑定为永久性缓存区的缓冲区范围的起始偏移量必须与 [**DML_PERSISTENT_BUFFER_ALIGNMENT**](/windows/desktop/direct3d12/directml_constants) 对齐。 缓冲区底层的堆的类型必须是 **D3D12_HEAP_TYPE_DEFAULT**。

## <a name="describe-and-bind-any-tensors"></a>描述和绑定任何张量

如果处理的是编译的运算符（而不是运算符初始值设定项），则需要将输入和输出资源（适用于张量和其他参数）绑定到该运算符的绑定表。 绑定数目必须完全与运算符的输入数目（包括可选的张量）相匹配。 运算符采用的特定输入和输出张量和其他参数已在该运算符的主题（例如 [**DML_ELEMENT_WISE_IDENTITY_OPERATOR_DESC**](/windows/desktop/api/directml/ns-directml-dml_element_wise_identity_operator_desc)）中阐述。

张量资源是包含该张量的各个元素值的缓冲区。 可以使用常规的 Direct3D 12 方法（[上传资源](/windows/desktop/direct3d12/uploading-resources)和[通过缓冲区读回数据](/windows/desktop/direct3d12/readback-data-using-heaps)）向/从 GPU 上传和读回此类缓冲区。 请参阅[精简 DirectML 应用程序](dml-min-app.md)代码示例了解这些方法的运作方式。

最后，使用 **DML_BUFFER_BINDING** 和 **DML_BINDING_DESC** 结构描述输入和输出资源绑定，然后通过调用 [**IDMLBindingTable::BindInputs**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindinputs) 和 [**IDMLBindingTable::BindOutputs**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindoutputs)，将其添加到编译的运算符的绑定表。 调用 **IDMLBindingTable::Bind\*** 方法时，DirectML 会将一个或多个描述符写入 CPU 描述符的范围。

```cppwinrt
DML_BUFFER_BINDING inputBufferBinding{ inputBuffer.get(), 0, tensorBufferSize };
DML_BINDING_DESC inputBindingDesc{ DML_BINDING_TYPE_BUFFER, &inputBufferBinding };
dmlBindingTable->BindInputs(1, &inputBindingDesc);

DML_BUFFER_BINDING outputBufferBinding{ outputBuffer.get(), 0, tensorBufferSize };
DML_BINDING_DESC outputBindingDesc{ DML_BINDING_TYPE_BUFFER, &outputBufferBinding };
dmlBindingTable->BindOutputs(1, &outputBindingDesc);
```

创建 DirectML 运算符（请参阅 [**IDMLDevice::CreateOperator**](/windows/desktop/api/directml/nf-directml-idmldevice-createoperator)）的步骤之一是声明一个或多个 [**DML_BUFFER_TENSOR_DESC**](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc) 结构来描述该运算符采用和返回的张量数据缓冲区。 对于张量缓冲区的类型和大小，可以选择性地指定 [**DML_TENSOR_FLAG_OWNED_BY_DML**](/windows/desktop/api/directml/ns-directml-dml_tensor_flags) 标志。

**DML_TENSOR_FLAG_OWNED_BY_DML** 指示张量数据是否应由 DirectML 拥有和管理。 DirectML 在初始化运算符期间会创建张量数据的副本，并将其存储在永久性资源中。 这样，DirectML 便可以将张量数据的格式重新设置为其他更有效的格式。 设置此标志可以提高性能，但此设置通常只对其数据在运算符的整个生存期内不会更改的张量有用。 此外，只能针对输入张量使用该标志。 针对特定的张量描述设置该标志时，相应的张量必须在初始化运算符期间绑定到绑定表，而不能在执行期间绑定（否则会导致出错）。 这与默认行为（不使用 DML_TENSOR_FLAG_OWNED_BY_DML 标志时的行为）相反。在默认行为中，张量预期会在执行期间而不是初始化期间绑定。 将张量数据提供给运算符初始值设定项时，绑定 UPLOAD 堆（而不是 DEFAULT 堆）是合法的操作，因为 DirectML 会创建数据的副本。 在所有其他情况下，绑定到 DirectML 的所有资源必须是 DEFAULT 堆资源。

有关详细信息，请参阅 [**IDMLBindingTable::BindInputs**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindinputs) 和 [**IDMLBindingTable::BindOutputs**](/windows/desktop/api/directml/nf-directml-idmlbindingtable-bindoutputs)。

## <a name="execute-the-dispatchable"></a>执行可调度对象

调用 [**IDMLCommandRecorder::RecordDispatch**](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch) 时传递绑定表作为参数。

在调用 **IDMLCommandRecorder::RecordDispatch** 期间使用绑定表时，DirectML 会将相应的 GPU 描述符绑定到管道。 CPU 和 GPU 描述符句柄不必要指向描述符堆中的相同条目，但是，在使用此绑定表执行之前，应用程序需负责确保将 CPU 描述符句柄引用的整个描述符范围复制到 GPU 描述符句柄引用的范围。

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

最后，关闭 Direct3D 12 命令列表，并像处理任何其他命令列表一样将其提交以执行。

在 GPU 上执行 **RecordDispatch** 之前，必须将所有绑定资源转换为 **D3D12_RESOURCE_STATE_UNORDERED_ACCESS** 状态，或转换为可隐式提升到 **D3D12_RESOURCE_STATE_UNORDERED_ACCESS** 的状态，例如 **D3D12_RESOURCE_STATE_COMMON**。 此调用完成后，资源将保持 **D3D12_RESOURCE_STATE_UNORDERED_ACCESS** 状态。 只有在执行运算符初始值设定项时以及为一个或多个张量设置了 **DML_TENSOR_FLAG_OWNED_BY_DML** 标志时绑定的上传堆例外。 在这种情况下，为输入绑定的任何上传堆必须处于 **D3D12_RESOURCE_STATE_GENERIC_READ** 状态，并会根据所有上传堆的要求保持该状态。 如果编译运算符时未设置 **DML_EXECUTION_FLAG_DESCRIPTORS_VOLATILE**，则在调用 **RecordDispatch** 之前，必须在绑定表中设置所有绑定，否则行为是不确定的。 如果运算符支持[后期绑定](#optionally-specify-late-bound-operator-bindings)，则绑定资源可能会推迟至将 Direct3D 12 命令列表提交到命令队列供执行为止。

**RecordDispatch** 在逻辑上类似于调用 [**ID3D12GraphicsCommandList::Dispatch**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-dispatch。 在这种情况下，需要施加无序访问视图 (UAV) 屏障，以确保调度之间存在数据依赖关系时顺序正确。 此方法不会在输入和输出资源中插入 UAV 屏障。 应用程序必须确保当任何输入的内容依赖于上游调度时在这些输入中执行正确的 UAV 屏障，并在下游调度依赖于这任何输出时在这些输出中执行正确的 UAV 屏障。

## <a name="lifetime-and-synchronization-of-descriptors-and-binding-table"></a>描述符和绑定表的生存期与同步

DirectML 中绑定的适当心理模型是，DirectML 绑定表本身在幕后会在提供的描述符堆中创建和管理无序访问视图 (UAV) 描述符。 因此，有关访问同步的所有常用 Direct3D 12 规则将应用到该堆及其描述符。 应用程序负责在使用绑定表的 CPU 和 GPU 工作之间执行正确的同步。

使用描述符（例如，由以前的框架使用）时，绑定表无法覆盖该描述符。 因此，若要重复使用已绑定的描述符堆（例如，通过针对指向它的绑定表再次调用 Bind*，或手动覆盖描述符堆），则应该等待当前正在使用该描述符堆的可调度对象在 GPU 上完成执行。 绑定表不会在它写入到的描述符堆中保留强引用，因此，在使用该绑定表的所有工作在 GPU 上完成执行之前，不得释放后备着色器可见的描述符堆。

另一方面，尽管绑定表不会指定和管理描述符堆，但表本身不包含任何此类内存。  因此，在对绑定表调用 [**IDMLCommandRecorder::RecordDispatch**](/windows/desktop/api/directml/nf-directml-idmlcommandrecorder-recorddispatch) 之后，随时可以释放或重置该绑定表（无需等待该调用在 GPU 上完成，前提是基础描述符仍然有效）。

绑定表不会在使用它绑定的任何资源上保留强引用 &mdash; 应用程序必须确保在 GPU 仍使用这些资源时不会将其删除。 此外，绑定表不是线程安全的 &mdash; 在未同步的情况下，应用程序不得从不同的线程同时针对绑定表调用方法。

另请考虑到仅当更改要绑定的资源时才需要重新绑定的任何情况。 如果无需更改绑定的资源，则可以在启动时绑定一次，并在每次调用 **RecordDispatch** 时传递相同的绑定表。

对于交错式机器学习和渲染工作负荷，只需确保每个帧的绑定表指向尚未在 GPU 上使用的描述符堆范围。

## <a name="optionally-specify-late-bound-operator-bindings"></a>选择性地指定后期绑定的运算符绑定

如果处理的是编译的运算符（而不是运算符初始值设定项），则可以选择为运算符指定后期绑定。 如果不使用后期绑定，必须在将运算符记录到命令列表之前，在绑定表中设置所有绑定。 如果使用后期绑定，则可以先在尚未提交互命令列表的运算符中设置（或更改）绑定，然后再将其提交到命令队列。

若要指定后期绑定，请结合 [**DML_EXECUTION_FLAG_DESCRIPTORS_VOLATILE**](/windows/desktop/api/directml/ne-directml-dml_execution_flags) 的 `flags` 参数调用 [**IDMLDevice::CompileOperator**](/windows/desktop/api/directml/nf-directml-idmldevice::compileoperator)。
