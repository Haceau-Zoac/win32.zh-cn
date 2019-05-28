---
title: 交换链
description: 交换链来控制后台缓冲区旋转，图形动画的基础。
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

交换链来控制后台缓冲区旋转，图形动画的基础。

-   [概述](#overview)
    -   [缓冲区的生存期](#buffer-lifetime)
    -   [交换效果](#swap-effects)
    -   [开窗和全屏幕模式之间转换](#transitioning-between-windowed-and-full-screen-modes)
    -   [示例](#example)
    -   [创建交换链](#creating-swap-chains)
-   [相关的主题](#related-topics)

## <a name="overview"></a>概述

D3D12 中的交换链的编程模型不是在早期版本的 D3D 相同的。 编程更加方便，例如，支持自动资源旋转 D3D10 和 D3D11 中出现过的不现在支持。 自动资源旋转启用应用程序来呈现相同的 API 对象，而实际的图面上所呈现的更改的每个帧。 交换链的行为更改与 D3D12 若要启用其他功能的 D3D12 具有较低的 CPU 开销。

### <a name="buffer-lifetime"></a>缓冲区的生存期

允许应用将预先创建的描述符引用启用此功能通过确保的一组拥有的交换链缓冲区永远不会更改的生存期内的交换链后台缓冲区。 一组返回的缓冲区[ **IDXGISwapChain::GetBuffer** ](https://msdn.microsoft.com/library/windows/desktop/bb174570)不会更改之前调用某些 Api:

-   [**IDXGISwapChain::ResizeTarget**](https://msdn.microsoft.com/library/windows/desktop/bb174578)
-   [**IDXGISwapChain::ResizeBuffers**](https://msdn.microsoft.com/library/windows/desktop/bb174577)

返回的缓冲区的顺序[ **GetBuffer** ](https://msdn.microsoft.com/library/windows/desktop/bb174570)永远不会更改。

[**IDXGISwapChain3::GetCurrentBackBufferIndex** ](https://msdn.microsoft.com/library/windows/desktop/dn903675)返回应用到当前的后台缓冲区的索引。

### <a name="swap-effects"></a>交换效果

唯一受支持的交换效果，FLIP\_顺序，需要为大于 1 的缓冲区计数。

### <a name="transitioning-between-windowed-and-full-screen-modes"></a>开窗和全屏幕模式之间转换

D3D12 维护应用程序必须调用的限制[ **ResizeBuffers** ](https://msdn.microsoft.com/library/windows/desktop/bb174577)后 （D3D11 翻转模型交换链受到相同的限制） 的开窗和全屏幕模式之间转换。

[ **IDXGISwapChain::SetFullscreenState** ](https://msdn.microsoft.com/library/windows/desktop/bb174579)转换不会更改的一组在交换链中的应用程序可见缓冲区。 仅[ **ResizeBuffers** ](https://msdn.microsoft.com/library/windows/desktop/bb174577)并[ **ResizeTarget** ](https://msdn.microsoft.com/library/windows/desktop/bb174578)调用创建或销毁应用程序可见的缓冲区。

时[ **IDXGISwapChain1::Present1** ](https://msdn.microsoft.com/library/windows/desktop/hh446797)调用后要显示缓冲区必须采用[ **D3D12\_资源\_状态\_存在**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states)状态。 存在将因 DXGI\_错误\_无效\_如果这不是这种情况，请调用。

全屏幕交换链仍存在限制， [ **SetFullscreenState**](https://msdn.microsoft.com/library/windows/desktop/bb174579)交换链在最终发行前必须调用 （FALSE、 NULL）。 **SetFullscreenState**D3D12 设备上运行的交换链上成功 (FALSE)。

与该设备，关联的默认 3D 队列上发生存在操作和应用程序可以自由地同时存在多个交换链，并记录并执行命令列表。

### <a name="example"></a>示例

下面的示例代码也会在主要的渲染循环出现：

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

使用时[ **CreateSwapChainForHwnd**](https://msdn.microsoft.com/library/windows/desktop/hh404557)， [ **CreateSwapChainForCoreWindow**](https://msdn.microsoft.com/library/windows/desktop/hh404559)，或[ **CreateSwapChainForComposition** ](https://msdn.microsoft.com/library/windows/desktop/hh404558)调用，请注意， *pDevice*参数实际需要的指针到 Direct3D 12 中，并不是设备中的直接的命令队列。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[**D3D12\_HEAP\_FLAGS**](/windows/desktop/api/D3D12/ne-d3d12-d3d12_heap_flags)
</dt> <dt>

[DirectX 高级学习视频教程：不受阻止的帧速率](https://www.youtube.com/watch?v=wn02zCXa9IU)
</dt> <dt>

[呈现](rendering.md)
</dt> </dl>

 

 




