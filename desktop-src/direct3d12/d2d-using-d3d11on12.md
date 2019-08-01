---
title: 使用 D3D11on12 的 D2D
description: D3D1211on12 示例演示了如何通过在基于 11 的设备和基于 12 的设备之间共享资源，通过 D3D12 呈现 D2D 内容。
ms.assetid: FAEF1412-053C-4B5F-80FA-85396C2586B4
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: a9c21cf439fe4e1ef7e6040c52b59e450d9c4311
ms.sourcegitcommit: 27a9dfa3ef68240fbf09f1c64dff7b2232874ef4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66725579"
---
# <a name="d2d-using-d3d11on12"></a>使用 D3D11on12 的 D2D

D3D1211on12 示例演示了如何通过在基于 11 的设备和基于 12 的设备之间共享资源，通过 D3D12 呈现 D2D 内容  。

-   [创建 ID3D11On12Device](#create-an-id3d11on12device)
-   [创建 D2D 工厂](#create-a-d2d-factory)
-   [为 D2D 创建呈现器目标](#create-a-render-target-for-d2d)
-   [创建基本 D2D 文本对象](#create-basic-d2d-text-objects)
-   [更新主呈现循环](#updating-the-main-render-loop)
-   [运行示例](#run-the-sample)
-   [相关主题](#related-topics)

## <a name="create-an-id3d11on12device"></a>创建 ID3D11On12Device

首先是在创建 [ID3D12Device](/windows/desktop/api/d3d12/nn-d3d12-id3d12device) 之后创建一个 [ID3D11On12Device](/windows/desktop/api/d3d11on12/nn-d3d11on12-id3d11on12device)，这包括创建一个通过 [API D3D11On12CreateDevice](/windows/desktop/api/d3d11on12/nf-d3d11on12-d3d11on12createdevice) 包装在 ID3D12Device 上的 [ID3D11Device](https://docs.microsoft.com/windows/desktop/api/d3d11/nn-d3d11-id3d11device)      。 该 API 还采用 [ID3D12CommandQueue](/windows/desktop/api/d3d12/nn-d3d12-id3d12commandqueue) 等参数，以便 11On12 设备能够提交其命令  。 创建 ID3D11Device 之后，可从中查询 ID3D11On12Device 接口   。 这是用于设置 D2D 的主要设备对象。

在 LoadPipeline 方法中设置设备  。

``` syntax
 // Create an 11 device wrapped around the 12 device and share
    // 12's command queue.
    ComPtr<ID3D11Device> d3d11Device;
    ThrowIfFailed(D3D11On12CreateDevice(
        m_d3d12Device.Get(),
        d3d11DeviceFlags,
        nullptr,
        0,
        reinterpret_cast<IUnknown**>(m_commandQueue.GetAddressOf()),
        1,
        0,
        &d3d11Device,
        &m_d3d11DeviceContext,
        nullptr
        ));

    // Query the 11On12 device from the 11 device.
    ThrowIfFailed(d3d11Device.As(&m_d3d11On12Device));
```



| 调用流程                                              | 参数 |
|--------------------------------------------------------|------------|
| [**ID3D11Device**](https://docs.microsoft.com/windows/desktop/api/d3d11/nn-d3d11-id3d11device)            |            |
| [**D3D11On12CreateDevice**](/windows/desktop/api/d3d11on12/nf-d3d11on12-d3d11on12createdevice) |            |



 

## <a name="create-a-d2d-factory"></a>创建 D2D 工厂

现在我们有了 11On12 设备，我们用它来创建 D2D 工厂和设备，就像使用 D3D11 时的通常做法一样。

添加到 LoadAssets 方法  。

``` syntax
 // Create D2D/DWrite components.
    {
        D2D1_DEVICE_CONTEXT_OPTIONS deviceOptions = D2D1_DEVICE_CONTEXT_OPTIONS_NONE;
        ThrowIfFailed(D2D1CreateFactory(D2D1_FACTORY_TYPE_SINGLE_THREADED, __uuidof(ID2D1Factory3), &d2dFactoryOptions, &m_d2dFactory));
        ComPtr<IDXGIDevice> dxgiDevice;
        ThrowIfFailed(m_d3d11On12Device.As(&dxgiDevice));
        ThrowIfFailed(m_d2dFactory->CreateDevice(dxgiDevice.Get(), &m_d2dDevice));
        ThrowIfFailed(m_d2dDevice->CreateDeviceContext(deviceOptions, &m_d2dDeviceContext));
        ThrowIfFailed(DWriteCreateFactory(DWRITE_FACTORY_TYPE_SHARED, __uuidof(IDWriteFactory), &m_dWriteFactory));
    }
```



| 调用流程                                                                        | 参数                                                   |
|----------------------------------------------------------------------------------|--------------------------------------------------------------|
| [**D2D1\_DEVICE\_CONTEXT\_OPTIONS**](https://docs.microsoft.com/windows/desktop/api/d2d1_1/ne-d2d1_1-d2d1_device_context_options)     |                                                              |
| [**D2D1CreateFactory**](https://docs.microsoft.com/windows/desktop/api/d2d1/nf-d2d1-d2d1createfactory)                              | [**D2D1\_FACTORY\_TYPE**](https://docs.microsoft.com/windows/desktop/api/d2d1/ne-d2d1-d2d1_factory_type)        |
| [**IDXGIDevice**](https://docs.microsoft.com/windows/desktop/api/dxgi/nn-dxgi-idxgidevice)                                      |                                                              |
| [**ID2D1Factory3::CreateDevice**](https://docs.microsoft.com/windows/desktop/api/d2d1_3/nf-d2d1_3-id2d1factory3-createdevice)           |                                                              |
| [**ID2D1Device::CreateDeviceContext**](https://docs.microsoft.com/windows/desktop/api/d2d1_1/nf-d2d1_1-id2d1device-createdevicecontext) |                                                              |
| [**DWriteCreateFactory**](https://docs.microsoft.com/windows/desktop/api/dwrite/nf-dwrite-dwritecreatefactory)                       | [**DWRITE\_FACTORY\_TYPE**](https://docs.microsoft.com/windows/desktop/api/dwrite/ne-dwrite-dwrite_factory_type) |



 

## <a name="create-a-render-target-for-d2d"></a>为 D2D 创建呈现器目标

D3D12 拥有交换链，因此如果我们想使用 11On12 设备（D2D 内容）呈现到后台缓冲区，则需要基于 [ID3D12Resource](/windows/desktop/api/d3d12/nn-d3d12-id3d12resource) 类型的后台缓冲区创建 [ID3D11Resource](https://docs.microsoft.com/windows/desktop/api/d3d11/nn-d3d11-id3d11resource) 类型的包装资源   。 这会将 ID3D12Resource 与基于 D3D11 的接口相关联，使它能够与 11On12 设备配合使用  。 在拥有包装资源后，然后就可为 D2D 创建一个要呈现到的呈现器目标，这也是在 LoadAssets 方法中创建的  。

``` syntax
 // Query the desktop's dpi settings, which will be used to create
    // D2D's render targets.
    float dpiX;
    float dpiY;
    m_d2dFactory->GetDesktopDpi(&dpiX, &dpiY);
    D2D1_BITMAP_PROPERTIES1 bitmapProperties = D2D1::BitmapProperties1(
        D2D1_BITMAP_OPTIONS_TARGET | D2D1_BITMAP_OPTIONS_CANNOT_DRAW,
        D2D1::PixelFormat(DXGI_FORMAT_UNKNOWN, D2D1_ALPHA_MODE_PREMULTIPLIED),
        dpiX,
        dpiY
        );  

    // Create frame resources.
    {
        CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart());

        // Create a RTV, D2D render target, and a command allocator for each frame.
        for (UINT n = 0; n < FrameCount; n++)
        {
            ThrowIfFailed(m_swapChain->GetBuffer(n, IID_PPV_ARGS(&m_renderTargets[n])));
            m_d3d12Device->CreateRenderTargetView(m_renderTargets[n].Get(), nullptr, rtvHandle);

            // Create a wrapped 11On12 resource of this back buffer. Since we are 
            // rendering all D3D12 content first and then all D2D content, we specify 
            // the In resource state as RENDER_TARGET - because D3D12 will have last 
            // used it in this state - and the Out resource state as PRESENT. When 
            // ReleaseWrappedResources() is called on the 11On12 device, the resource 
            // will be transitioned to the PRESENT state.
            D3D11_RESOURCE_FLAGS d3d11Flags = { D3D11_BIND_RENDER_TARGET };
            ThrowIfFailed(m_d3d11On12Device->CreateWrappedResource(
                m_renderTargets[n].Get(),
                &d3d11Flags,
                D3D12_RESOURCE_STATE_RENDER_TARGET,
                D3D12_RESOURCE_STATE_PRESENT,
                IID_PPV_ARGS(&m_wrappedBackBuffers[n])
                ));

            // Create a render target for D2D to draw directly to this back buffer.
            ComPtr<IDXGISurface> surface;
            ThrowIfFailed(m_wrappedBackBuffers[n].As(&surface));
            ThrowIfFailed(m_d2dDeviceContext->CreateBitmapFromDxgiSurface(
                surface.Get(),
                &bitmapProperties,
                &m_d2dRenderTargets[n]
                ));

            rtvHandle.Offset(1, m_rtvDescriptorSize);

            ThrowIfFailed(m_d3d12Device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&m_commandAllocators[n])));
        }
    }
```



<table>
<thead>
<tr class="header">
<th>调用流程</th>
<th>参数</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://docs.microsoft.com/windows/desktop/api/d2d1/nf-d2d1-id2d1factory-getdesktopdpi"><strong>ID2D1Factory::GetDesktopDpi</strong></a></td>

</tr>
<tr class="even">
<td><a href="https://docs.microsoft.com/windows/desktop/api/d2d1_1/ns-d2d1_1-d2d1_bitmap_properties1"><strong>D2D1_BITMAP_PROPERTIES1</strong></a></td>
<td><dl><a href="https://docs.microsoft.com/windows/desktop/api/d2d1_1helper/nf-d2d1_1helper-bitmapproperties1"><strong>BitmapProperties1</strong></a><br />
[<strong>D2D1_BITMAP_OPTIONS</strong>](https://docs.microsoft.com/windows/desktop/api/d2d1_1/ne-d2d1_1-d2d1_bitmap_options)<br />
[<strong>PixelFormat</strong>](https://docs.microsoft.com/windows/desktop/api/d2d1helper/nf-d2d1helper-pixelformat)<br />
[<strong>DXGI_FORMAT</strong>](https://docs.microsoft.com/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format)<br />
[<strong>D2D1_ALPHA_MODE</strong>](https://docs.microsoft.com/windows/desktop/api/dcommon/ne-dcommon-d2d1_alpha_mode)<br />
</dl></td>
</tr>
<tr class="odd">
<td><a href="cd3dx12-cpu-descriptor-handle.md"><strong>CD3DX12_CPU_DESCRIPTOR_HANDLE</strong></a></td>
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart"><strong>GetCPUDescriptorHandleForHeapStart</strong></a></td>
</tr>
<tr class="even">
<td><a href="https://docs.microsoft.com/windows/desktop/api/dxgi/nf-dxgi-idxgiswapchain-getbuffer"><strong>IDXGISwapChain::GetBuffer</strong></a></td>

</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createrendertargetview"><strong>CreateRenderTargetView</strong></a></td>

</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d11on12/ns-d3d11on12-d3d11_resource_flags"><strong>D3D11_RESOURCE_FLAGS</strong></a></td>
<td><a href="https://docs.microsoft.com/windows/desktop/api/d3d11/ne-d3d11-d3d11_bind_flag"><strong>D3D11_BIND_FLAG</strong></a></td>
</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-createwrappedresource"><strong>CreateWrappedResource</strong></a></td>
<td><a href="/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_states"><strong>D3D12_RESOURCE_STATES</strong></a></td>
</tr>
<tr class="even">
<td><a href="https://docs.microsoft.com/windows/desktop/api/dxgi/nn-dxgi-idxgisurface"><strong>IDXGISurface</strong></a></td>

</tr>
<tr class="odd">
<td><a href="https://docs.microsoft.com/windows/desktop/api/d2d1_1/nf-d2d1_1-id2d1devicecontext-createbitmapfromdxgisurface(idxgisurface_constd2d1_bitmap_properties1__id2d1bitmap1)"><strong>ID2D1DeviceContext::CreateBitmapFromDxgiSurface</strong></a></td>

</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandallocator"><strong>CreateCommandAllocator</strong></a></td>
<td><a href="/windows/desktop/api/d3d12/ne-d3d12-d3d12_command_list_type"><strong>D3D12_COMMAND_LIST_TYPE</strong></a></td>
</tr>
</tbody>
</table>



 

## <a name="create-basic-d2d-text-objects"></a>创建基本 D2D 文本对象

现在我们有一个 [ID3D12Device](/windows/desktop/api/d3d12/nn-d3d12-id3d12device) 用于呈现 3D 内容，一个 [ID2D1Device](https://docs.microsoft.com/windows/desktop/api/d2d1_1/nn-d2d1_1-id2d1device) 通过可用于呈现 2D 内容的 [ID3D11On12Device](/windows/desktop/api/d3d11on12/nn-d3d11on12-id3d11on12device) 与 12 设备共享内容，而且这两个都配置为呈现到同一交换链    。 此示例仅使用 D2D 设备在 3D 场景上呈现文本，类似于游戏呈现其 UI 的方式。 为此，我们需要仍然使用 LoadAssets 方法创建一些基本的 D2D 对象  。

``` syntax
 // Create D2D/DWrite objects for rendering text.
    {
        ThrowIfFailed(m_d2dDeviceContext->CreateSolidColorBrush(D2D1::ColorF(D2D1::ColorF::Black), &m_textBrush));
        ThrowIfFailed(m_dWriteFactory->CreateTextFormat(
            L"Verdana",
            NULL,
            DWRITE_FONT_WEIGHT_NORMAL,
            DWRITE_FONT_STYLE_NORMAL,
            DWRITE_FONT_STRETCH_NORMAL,
            50,
            L"en-us",
            &m_textFormat
            ));
        ThrowIfFailed(m_textFormat->SetTextAlignment(DWRITE_TEXT_ALIGNMENT_CENTER));
        ThrowIfFailed(m_textFormat->SetParagraphAlignment(DWRITE_PARAGRAPH_ALIGNMENT_CENTER));
    }
```



| 调用流程                                                                                           | 参数                                                                 |
|-----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|
| [**ID2D1RenderTarget::CreateSolidColorBrush**](https://docs.microsoft.com/windows/desktop/api/d2d1/nf-d2d1-createsolidcolorbrush)    | [**ColorF**](https://docs.microsoft.com/windows/desktop/api/d2d1helper/nl-d2d1helper-colorf)                                              |
| [**IDWriteFactory::CreateTextFormat**](https://docs.microsoft.com/windows/desktop/api/dwrite/nf-dwrite-idwritefactory-createtextformat)                 | [**DWRITE\_FONT\_WEIGHT**](https://docs.microsoft.com/windows/desktop/api/dwrite/ne-dwrite-dwrite_font_weight)                 |
| [**IDWriteTextFormat::SetTextAlignment**](https://docs.microsoft.com/windows/desktop/api/dwrite/nf-dwrite-idwritetextformat-settextalignment)           | [**DWRITE\_TEXT\_ALIGNMENT**](https://docs.microsoft.com/windows/desktop/api/dwrite/ne-dwrite-dwrite_text_alignment)           |
| [**IDWriteTextFormat::SetParagraphAlignment**](https://docs.microsoft.com/windows/desktop/api/dwrite/nf-dwrite-idwritetextformat-setparagraphalignment) | [**DWRITE\_PARAGRAPH\_ALIGNMENT**](https://docs.microsoft.com/windows/desktop/api/dwrite/ne-dwrite-dwrite_paragraph_alignment) |



 

## <a name="updating-the-main-render-loop"></a>更新主呈现循环

现已完成示例的初始化，可执行主呈现循环了。

``` syntax
// Render the scene.
void D3D1211on12::OnRender()
{
    // Record all the commands we need to render the scene into the command list.
    PopulateCommandList();

    // Execute the command list.
    ID3D12CommandList* ppCommandLists[] = { m_commandList.Get() };
    m_commandQueue->ExecuteCommandLists(_countof(ppCommandLists), ppCommandLists);

    RenderUI();

    // Present the frame.
    ThrowIfFailed(m_swapChain->Present(0, 0));

    MoveToNextFrame();
}
```



| 调用流程                                                              | 参数 |
|------------------------------------------------------------------------|------------|
| [**ID3D12CommandList**](/windows/desktop/api/d3d12/nn-d3d12-id3d12commandlist)                         |            |
| [**ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)  |            |
| [**IDXGISwapChain1::Present1**](https://docs.microsoft.com/windows/desktop/api/dxgi1_2/nf-dxgi1_2-idxgiswapchain1-present1) |            |



 

呈现循环中唯一新功能是 RenderUI 调用，它将使用 D2D 来呈现 UI  。 请注意，我们首先执行所有 D3D12 命令列表以呈现 3D 场景，然后在此基础上呈现 UI。 在深入研究 RenderUI 之前，必须查看对 PopulateCommandLists 的更改   。 在其他示例中，我们通常在关闭命令列表之前在列表上放置一个资源屏障，以便将后台缓冲区从呈现目标状态转换为当前状态。 但是，在此示例中，我们移除了该资源屏障，因为我们仍然需要使用 D2D 呈现到后台缓冲区。 注意，当我们创建后台缓冲区的包装资源时，我们将呈现目标状态指定为“IN”状态，将当前状态指定为“OUT”状态。

RenderUI 在 D2D 使用方面非常简单  。 我们设置呈现器目标并呈现文本。 但是，在 11On12 设备上使用任何包装资源（例如后台缓冲区呈现目标）之前，必须在 11On12 设备上调用 [AcquireWrappedResources](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-acquirewrappedresources) API  。 呈现后，我们在 11On12 设备上调用 [ReleaseWrappedResources](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-releasewrappedresources) API  。 通过调用 ReleaseWrappedResources，将在后台生成一个资源屏障，它将把指定的资源转换为创建时指定的“OUT”状态  。 在本例中，就是指当前状态。 最后，为了将在 11On12 设备上执行的所有命令提交到共享的 [ID3D12CommandQueue](/windows/desktop/api/d3d12/nn-d3d12-id3d12commandqueue)，必须在 [ID3D11DeviceContext](https://docs.microsoft.com/windows/desktop/api/d3d11/nn-d3d11-id3d11devicecontext) 上调用[刷新](https://docs.microsoft.com/windows/desktop/api/d3d11/nf-d3d11-id3d11devicecontext-flush)    。

``` syntax
// Render text over D3D12 using D2D via the 11On12 device.
void D3D1211on12::RenderUI()
{
    D2D1_SIZE_F rtSize = m_d2dRenderTargets[m_frameIndex]->GetSize();
    D2D1_RECT_F textRect = D2D1::RectF(0, 0, rtSize.width, rtSize.height);
    static const WCHAR text[] = L"11On12";

    // Acquire our wrapped render target resource for the current back buffer.
    m_d3d11On12Device->AcquireWrappedResources(m_wrappedBackBuffers[m_frameIndex].GetAddressOf(), 1);

    // Render text directly to the back buffer.
    m_d2dDeviceContext->SetTarget(m_d2dRenderTargets[m_frameIndex].Get());
    m_d2dDeviceContext->BeginDraw();
    m_d2dDeviceContext->SetTransform(D2D1::Matrix3x2F::Identity());
    m_d2dDeviceContext->DrawTextW(
        text,
        _countof(text) - 1,
        m_textFormat.Get(),
        &textRect,
        m_textBrush.Get()
        );
    ThrowIfFailed(m_d2dDeviceContext->EndDraw());

    // Release our wrapped render target resource. Releasing 
    // transitions the back buffer resource to the state specified
    // as the OutState when the wrapped resource was created.
    m_d3d11On12Device->ReleaseWrappedResources(m_wrappedBackBuffers[m_frameIndex].GetAddressOf(), 1);

    // Flush to submit the 11 command list to the shared command queue.
    m_d3d11DeviceContext->Flush();
}
```



| 调用流程                                                                                                                                                                                 | 参数                            |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------|
| [**D2D1\_SIZE\_F**](https://docs.microsoft.com/windows/desktop/Direct2D/d2d1-size-f)                                                                                                                                                 |                                       |
| [**D2D1\_RECT\_F**](https://docs.microsoft.com/windows/desktop/Direct2D/d2d1-rect-f)                                                                                                                                                 | [**RectF**](https://docs.microsoft.com/windows/desktop/api/d2d1helper/nf-d2d1helper-rectf)           |
| [**AcquireWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-acquirewrappedresources)                                                                                                               |                                       |
| [**ID2D1DeviceContext::SetTarget**](https://docs.microsoft.com/windows/desktop/api/d2d1_1/nf-d2d1_1-id2d1devicecontext-settarget)                                                                                                                |                                       |
| [**ID2D1RenderTarget::BeginDraw**](https://docs.microsoft.com/windows/desktop/api/d2d1/nf-d2d1-id2d1rendertarget-begindraw)                                                                                                                  |                                       |
| [**ID2D1RenderTarget::SetTransform**](https://docs.microsoft.com/windows/desktop/Direct2D/id2d1rendertarget-settransform)                                                                                                            | [**Matrix3x2F**](https://docs.microsoft.com/windows/desktop/api/d2d1helper/nl-d2d1helper-matrix3x2f) |
| [**ID2D1RenderTarget::DrawTextW**](https://docs.microsoft.com/windows/desktop/api/d2d1/nf-d2d1-id2d1rendertarget-drawtext(constwchar_uint32_idwritetextformat_constd2d1_rect_f__id2d1brush_d2d1_draw_text_options_dwrite_measuring_mode)) |                                       |
| [**ID2D1RenderTarget::EndDraw**](https://docs.microsoft.com/windows/desktop/api/d2d1/nf-d2d1-id2d1rendertarget-enddraw)                                                                                                                      |                                       |
| [**ReleaseWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-releasewrappedresources)                                                                                                               |                                       |
| [**ID3D11DeviceContext::Flush**](https://docs.microsoft.com/windows/desktop/api/d3d11/nf-d3d11-id3d11devicecontext-flush)                                                                                                                    |                                       |



 

## <a name="run-the-sample"></a>运行示例

![12 示例上的 11 最终输出](images/11on12image.png)

## <a name="related-topics"></a>相关主题

<dl> <dt>

[D3D12 代码演练](d3d12-code-walk-throughs.md)
</dt> <dt>

[Direct3D 11 on 12](direct3d-11-on-12.md)
</dt> <dt>

[Direct3D 12 互操作](direct3d-12-interop.md)
</dt> <dt>

[11on12 参考](direct3d-11on12-reference.md)
</dt> </dl>

 

 




