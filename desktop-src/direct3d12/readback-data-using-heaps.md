---
title: 通过缓冲区读回数据
description: 若要从 GPU 回读数据（例如，捕获屏幕截图），需使用回读堆。
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
# <a name="read-back-data-via-a-buffer"></a>通过缓冲区读回数据

若要从 GPU 回读数据（例如，捕获屏幕截图），需使用回读堆。 这种技术与[通过缓冲区上传纹理数据](upload-and-readback-of-texture-data.md)有关，但有一些不同。

- 若要回读数据，请创建一个堆，将 D3D12_HEAP_TYPE 设置为 [D3D12_HEAP_TYPE_READBACK](/windows/desktop/api/D3D12/ne-d3d12-d3d12_heap_type)，而不是设置为 D3D12_HEAP_TYPE_UPLOAD  。
- 使用栅栏来检测 GPU 完成帧处理的时间（当它完成将数据写入输出缓冲区时）。 这很重要，因为 [ID3D12Resource::Map](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map) 方法不与 GPU 同步（相反，Direct3D 11 的对应方法会与之同步）   。 Direct3D 12 映射调用的运行状况类似于使用 NO_OVERWRITE 标志调用 Direct3D 11 对应映射  。
- 数据准备就绪后（包括任何必要的资源屏障），调用 [ID3D12Resource::Map](/windows/desktop/api/D3D12/nf-d3d12-id3d12resource-map)，使回读数据对 CPU 可见  。

## <a name="code-example"></a>代码示例

下方的代码示例显示通过缓冲区将数据从 GPU 回读到 CPU 的过程的概要。

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

* [缓冲区中的二次分配](large-buffers.md)
