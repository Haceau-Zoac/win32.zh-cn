---
title: 交换链
description: 交换链控制后台缓冲区旋转，构成图形动画的基础。
ms.assetid: AABF5FDE-DB49-4B29-BC0E-032E0C7DF9EB
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: a0f251020fbd9df1e917f0ad56b2d49b81452e64
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224056"
---
# <a name="swap-chains"></a>交换链

交换链控制后台缓冲区旋转，构成图形动画的基础。

-   [概述](#overview)
    -   [缓冲区生存期](#buffer-lifetime)
    -   [交换效果](#swap-effects)
    -   [窗口模式和全屏模式之间转换](#transitioning-between-windowed-and-full-screen-modes)
    -   [示例](#example)
    -   [创建交换链](#creating-swap-chains)
-   [相关主题](#related-topics)

## <a name="overview"></a>概述

D3D12 中交换链的编程模型不同于 D3D 早期版本中的编程模型。 例如，D3D10 和 D3D11 中曾具有支持自动资源旋转的编程便利性，但现已不支持。 自动资源旋转确保应用能够呈现相同的 API 对象，而实际呈现的表面会改变每一帧。 D3D12 改变了交换链的行为，使 D3D12 的其他功能具有较低的 CPU 开销。

### <a name="buffer-lifetime"></a>缓冲区生存期

应用可以存储引用后台缓冲区的预创建描述符。其实现方式为确保交换链拥有的缓冲区集在交换链的生存期内不会更改。 [IDXGISwapChain::GetBuffer](https://msdn.microsoft.com/library/windows/desktop/bb174570) 返回的缓冲区集合在调用某些 API 之前不会更改  ：

-   [**IDXGISwapChain::ResizeTarget**](https://msdn.microsoft.com/library/windows/desktop/bb174578)
-   [**IDXGISwapChain::ResizeBuffers**](https://msdn.microsoft.com/library/windows/desktop/bb174577)

[GetBuffer](https://msdn.microsoft.com/library/windows/desktop/bb174570) 返回的缓冲区顺序永远不会更改  。

[IDXGISwapChain3::GetCurrentBackBufferIndex](https://msdn.microsoft.com/library/windows/desktop/dn903675) 将当前后台缓冲区的索引返回给应用  。

### <a name="swap-effects"></a>交换效果

唯一支持的交换效果是 FLIP\_ SEQUENTIAL，它要求缓冲区计数大于 1。

### <a name="transitioning-between-windowed-and-full-screen-modes"></a>窗口模式和全屏模式之间转换

D3D12 保留了应用程序在窗口模式和全屏模式之间切换后必须调用 [ResizeBuffers](https://msdn.microsoft.com/library/windows/desktop/bb174577) 的限制（D3D11 翻转模型交换链也有相同限制）  。

[IDXGISwapChain::SetFullscreenState](https://msdn.microsoft.com/library/windows/desktop/bb174579) 切换不会更改交换链中的应用可见缓冲区集合  。 仅 [ResizeBuffers](https://msdn.microsoft.com/library/windows/desktop/bb174577) 和 [ResizeTarget](https://msdn.microsoft.com/library/windows/desktop/bb174578) 调用可创建或销毁应用可见缓冲区   。

调用 [IDXGISwapChain1::Present1](https://msdn.microsoft.com/library/windows/desktop/hh446797) 时，要显示的后台缓冲区必须处于 [D3D12\_RESOURCE\_STATE\_PRESENT](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states) 状态   。 如果实际情况并非如此，DXGI\_ERROR\_INVALID\_CALL 将无法显示。

全屏交换链仍具有这样的限制：必须在交换链最终发布之前调用 [SetFullscreenState](https://msdn.microsoft.com/library/windows/desktop/bb174579) (FALSE、NULL)  。 SetFullscreenState(FALSE) 的成功取决于在 D3D12 设备上运行的交换链  。

当前操作发生在与设备关联的默认 3D 队列上，应用可同时呈现多个交换链，并记录和执行命令列表。

### <a name="example"></a>示例

以下示例代码将在主渲染循环中显示：

``` syntax
        CComPtr<IDXGISwapChain3> spSwapChain3;
        m_spSwapChain->QueryInterface(&spSwapChain3);
        UINT backBufferIndex = spSwapChain3->GetCurrentBackBufferIndex();

        CComPtr<ID3D12Resource>         spBackBuffer;
        m_spSwapChain->GetBuffer(backBufferIndex, IID_PPV_ARGS(&spBackBuffer));

        // record and execute a command list referencing spBackBuffer
        m_spSwapChain->Present1(0, 0);
```

### <a name="creating-swap-chains"></a>创建交换链

使用 [CreateSwapChainForHwnd](https://msdn.microsoft.com/library/windows/desktop/hh404557)、[CreateSwapChainForCoreWindow](https://msdn.microsoft.com/library/windows/desktop/hh404559) 或 [CreateSwapChainForComposition](https://msdn.microsoft.com/library/windows/desktop/hh404558) 调用时，请注意，pDevice 参数实际上需要指向 Direct3D 12 中直接命令队列（而不是设备）的指针     。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[D3D12\_HEAP\_FLAGS](/windows/desktop/api/D3D12/ne-d3d12-d3d12_heap_flags) 
</dt> <dt>

[DirectX 高级学习视频教程：不受制约的帧速率](https://www.youtube.com/watch?v=wn02zCXa9IU)
</dt> <dt>

[渲染](rendering.md)
</dt> </dl>

 

 




