---
title: 通过缓冲区读取后数据
description: 若要从 （例如，若要捕获屏幕快照） GPU 读取后的数据，您使用 readback 堆。
ms.assetid: 2F515B2C-3317-4AA8-81E1-7762ED895F77
ms.topic: article
ms.date: 12/17/2018
ms.openlocfilehash: 9b2cf9c62019df23cdf10ece7231454ac865ef5b
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224209"
---
# <a name="read-back-data-via-a-buffer"></a>通过缓冲区读取后数据

若要从 （例如，若要捕获屏幕快照） GPU 读取后的数据，您使用 readback 堆。 此技术与相关[将通过一个缓冲区的纹理数据上传](upload-and-readback-of-texture-data.md)，有一些区别。

- 若要读取数据后，您创建与堆**D3D12_HEAP_TYPE**设置为[D3D12_HEAP_TYPE_READBACK](/windows/desktop/api/D3D12/ne-d3d12-d3d12_heap_type)，而不是 D3D12_HEAP_TYPE_UPLOAD。
- 一种保护用于检测 GPU 完成处理帧 （完成后将数据写入到输出缓冲区）。 这很重要，因为[ **ID3D12Resource::Map** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)方法不会同步与 GPU (相反，Direct3D 11 等效*does*同步)。 Direct3D 12**映射**呼叫行为就像调用 NO_OVERWRITE 标志的 Direct3D 11 等效项。
- 数据 （包括任何必需的资源屏障） 准备就绪后，调用[ **ID3D12Resource::Map** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)使 readback 数据对 CPU 可见。

## <a name="code-example"></a>代码示例

下面的代码示例显示从 GPU 的后数据读取到缓冲区通过 CPU 的过程的概要。

```cppwinrt

// The output buffer (created below) is on a default heap, so only the GPU can access it.

D3D12_HEAP_PROPERTIES defaultHeapProperties{ CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT) };
D3D12_RESOURCE_DESC outputBufferDesc{ CD3DX12_RESOURCE_DESC::Buffer(outputBufferSize, D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS) };
winrt::com_ptr<::ID3D12Resource> outputBuffer;
winrt::check_hresult(d3d12Device->CreateCommittedResource(
    &defaultHeapProperties,
    D3D12_HEAP_FLAG_NONE,
    &outputBufferDesc,
    D3D12_RESOURCE_STATE_COPY_DEST,
    nullptr,
    __uuidof(outputBuffer),
    outputBuffer.put_void()));

// The readback buffer (created below) is on a readback heap, so that the CPU can access it.

D3D12_HEAP_PROPERTIES readbackHeapProperties{ CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_READBACK) };
D3D12_RESOURCE_DESC readbackBufferDesc{ CD3DX12_RESOURCE_DESC::Buffer(outputBufferSize) };
winrt::com_ptr<::ID3D12Resource> readbackBuffer;
winrt::check_hresult(d3d12Device->CreateCommittedResource(
    &readbackHeapProperties,
    D3D12_HEAP_FLAG_NONE,
    &readbackBufferDesc,
    D3D12_RESOURCE_STATE_COPY_DEST,
    nullptr,
    __uuidof(readbackBuffer),
    readbackBuffer.put_void()));

{
    D3D12_RESOURCE_BARRIER outputBufferResourceBarrier
    {
        CD3DX12_RESOURCE_BARRIER::Transition(
            outputBuffer.get(),
            D3D12_RESOURCE_STATE_COPY_DEST,
            D3D12_RESOURCE_STATE_COPY_SOURCE)
    };
    commandList->ResourceBarrier(1, &outputBufferResourceBarrier);
}

commandList->CopyResource(readbackBuffer.get(), outputBuffer.get());

// Code goes here to close, execute (and optionally reset) the command list, and also
// to use a fence to wait for the command queue.

// The code below assumes that the GPU wrote FLOATs to the buffer.

D3D12_RANGE readbackBufferRange{ 0, outputBufferSize };
FLOAT * pReadbackBufferData{};
winrt::check_hresult(
    readbackBuffer->Map
    (
        0,
        &readbackBufferRange,
        reinterpret_cast<void**>(&pReadbackBufferData)
    )
);

// Code goes here to access the data via pReadbackBufferData.

D3D12_RANGE emptyRange{ 0, 0 };
readbackBuffer->Unmap
(
    0,
    &emptyRange
);
```

## <a name="related-topics"></a>相关主题

* [缓冲区中的子分配](large-buffers.md)
