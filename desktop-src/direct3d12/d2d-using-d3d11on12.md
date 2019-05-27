---
title: D2D 使用 D3D11on12
description: D3D1211on12 示例演示如何呈现 D2D 内容对 D3D12 的内容由 11 的基于的设备和 12 的基于的设备之间共享资源。
ms.assetid: FAEF1412-053C-4B5F-80FA-85396C2586B4
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: d0a9c9797a4120c068f29b9ee3108e658da02656
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224470"
---
# <a name="d2d-using-d3d11on12"></a>D2D 使用 D3D11on12

**D3D1211on12**示例演示如何呈现 D2D 内容对 D3D12 的内容由 11 的基于的设备和 12 的基于的设备之间共享资源。

-   [创建 ID3D11On12Device](#create-an-id3d11on12device)
-   [创建 D2D 工厂](#create-a-d2d-factory)
-   [为 D2D 创建呈现器目标](#create-a-render-target-for-d2d)
-   [创建基本 D2D 文本对象](#create-basic-d2d-text-objects)
-   [更新主呈现循环](#updating-the-main-render-loop)
-   [运行示例](#run-the-sample)
-   [相关的主题](#related-topics)

## <a name="create-an-id3d11on12device"></a>创建 ID3D11On12Device

第一步是创建[ **ID3D11On12Device** ](/windows/desktop/api/d3d11on12/nn-d3d11on12-id3d11on12device)后[ **ID3D12Device** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12device)创建后，这涉及到创建[**ID3D11Device** ](https://msdn.microsoft.com/library/windows/desktop/ff476379)包装在**ID3D12Device**通过 API [ **D3D11On12CreateDevice** ](/windows/desktop/api/d3d11on12/nf-d3d11on12-d3d11on12createdevice). 此 API 还会在中，在其他参数之间[ **ID3D12CommandQueue** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue) ，以便 11On12 设备可以提交其命令。 之后**ID3D11Device**创建后，您可以查询**ID3D11On12Device**从它的接口。 这是将用于设置 D2D 的主要设备对象。

在中**LoadPipeline**方法，设置设备。

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



| 呼叫流                                              | 参数 |
|--------------------------------------------------------|------------|
| [**ID3D11Device**](https://msdn.microsoft.com/library/windows/desktop/ff476379)            |            |
| [**D3D11On12CreateDevice**](/windows/desktop/api/d3d11on12/nf-d3d11on12-d3d11on12createdevice) |            |



 

## <a name="create-a-d2d-factory"></a>创建 D2D 工厂

现在，我们已 11On12 设备，我们使用它来创建 D2D 工厂和设备，就像通常会用 D3D11 完成。

将添加到**LoadAssets**方法。

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



| 呼叫流                                                                        | 参数                                                   |
|----------------------------------------------------------------------------------|--------------------------------------------------------------|
| [**D2D1\_DEVICE\_CONTEXT\_OPTIONS**](https://msdn.microsoft.com/library/windows/desktop/hh446998)     |                                                              |
| [**D2D1CreateFactory**](https://msdn.microsoft.com/library/windows/desktop/dd368034)                              | [**D2D1\_工厂\_类型**](https://msdn.microsoft.com/library/windows/desktop/dd368104)        |
| [**IDXGIDevice**](https://msdn.microsoft.com/library/windows/desktop/bb174527)                                      |                                                              |
| [**ID2D1Factory3::CreateDevice**](https://msdn.microsoft.com/library/windows/desktop/dn900395)           |                                                              |
| [**ID2D1Device::CreateDeviceContext**](https://msdn.microsoft.com/library/windows/desktop/hh404545) |                                                              |
| [**DWriteCreateFactory**](https://msdn.microsoft.com/library/windows/desktop/dd368040)                       | [**DWRITE\_工厂\_类型**](https://msdn.microsoft.com/library/windows/desktop/dd368057) |



 

## <a name="create-a-render-target-for-d2d"></a>为 D2D 创建呈现器目标

D3D12 拥有交换链，因此如果我们想要呈现到后台缓冲区使用我们 11On12 设备 （D2D 内容），然后我们需要创建已包装的类型的资源[ **ID3D11Resource** ](https://msdn.microsoft.com/library/windows/desktop/ff476584)从类型的后台缓冲区[ **ID3D12Resource**](/windows/desktop/api/D3D12/nn-d3d12-id3d12resource)。 此链接**ID3D12Resource**用 D3D11 基于接口，以便用于 11On12 设备。 我们已包装的资源后，我们可以创建 D2D 来呈现，也可在呈现目标图面**LoadAssets**方法。

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
<th>呼叫流</th>
<th>参数</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://msdn.microsoft.com/library/windows/desktop/dd371316"><strong>ID2D1Factory::GetDesktopDpi</strong></a></td>

</tr>
<tr class="even">
<td><a href="https://msdn.microsoft.com/library/windows/desktop/hh404275"><strong>D2D1_BITMAP_PROPERTIES1</strong></a></td>
<td><dl><a href="https://msdn.microsoft.com/library/windows/desktop/hh847934"><strong>BitmapProperties1</strong></a><br />
[<strong>D2D1_BITMAP_OPTIONS</strong>](https://msdn.microsoft.com/library/windows/desktop/hh446984)<br />
[<strong>PixelFormat</strong>](https://msdn.microsoft.com/library/windows/desktop/dd372327)<br />
[<strong>DXGI_FORMAT</strong>](https://msdn.microsoft.com/library/windows/desktop/bb173059)<br />
[<strong>D2D1_ALPHA_MODE</strong>](https://msdn.microsoft.com/library/windows/desktop/dd368058)<br />
</dl></td>
</tr>
<tr class="odd">
<td><a href="cd3dx12-cpu-descriptor-handle"><strong>CD3DX12_CPU_DESCRIPTOR_HANDLE</strong></a></td>
<td><a href="/windows/desktop/api/D3D12/nf-d3d12-id3d12descriptorheap-getcpudescriptorhandleforheapstart"><strong>GetCPUDescriptorHandleForHeapStart</strong></a></td>
</tr>
<tr class="even">
<td><a href="https://msdn.microsoft.com/library/windows/desktop/bb174570"><strong>IDXGISwapChain::GetBuffer</strong></a></td>

</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrendertargetview"><strong>CreateRenderTargetView</strong></a></td>

</tr>
<tr class="even">
<td><a href="/windows/desktop/api/d3d11on12/ns-d3d11on12-d3d11_resource_flags"><strong>D3D11_RESOURCE_FLAGS</strong></a></td>
<td><a href="https://msdn.microsoft.com/library/windows/desktop/ff476085"><strong>D3D11_BIND_FLAG</strong></a></td>
</tr>
<tr class="odd">
<td><a href="/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-createwrappedresource"><strong>CreateWrappedResource</strong></a></td>
<td><a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_resource_states"><strong>D3D12_RESOURCE_STATES</strong></a></td>
</tr>
<tr class="even">
<td><a href="https://msdn.microsoft.com/library/windows/desktop/bb174565"><strong>IDXGISurface</strong></a></td>

</tr>
<tr class="odd">
<td><a href="https://msdn.microsoft.com/library/windows/desktop/hh404482"><strong>ID2D1DeviceContext::CreateBitmapFromDxgiSurface</strong></a></td>

</tr>
<tr class="even">
<td><a href="/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createcommandallocator"><strong>CreateCommandAllocator</strong></a></td>
<td><a href="/windows/desktop/api/D3D12/ne-d3d12-d3d12_command_list_type"><strong>D3D12_COMMAND_LIST_TYPE</strong></a></td>
</tr>
</tbody>
</table>



 

## <a name="create-basic-d2d-text-objects"></a>创建基本 D2D 文本对象

现在我们有[ **ID3D12Device** ](/windows/desktop/api/D3D12/nn-d3d12-id3d12device)呈现 3D 内容， [ **ID2D1Device** ](https://msdn.microsoft.com/library/windows/desktop/hh404478)共享与我们 12 的设备通过[**ID3D11On12Device** ](/windows/desktop/api/d3d11on12/nn-d3d11on12-id3d11on12device) -而我们可以使用它们来呈现 2D 内容-它们都配置为向相同的交换链呈现。 此示例只需使用 D2D 设备对三维场景，类似于游戏呈现它们的 UI 的方式呈现文本。 我们需要为此，请创建一些基本的 D2D 对象仍处于**LoadAssets**方法。

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



| 呼叫流                                                                                           | 参数                                                                 |
|-----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|
| [**ID2D1RenderTarget::CreateSolidColorBrush**](https://msdn.microsoft.com/library/windows/desktop/dd742843)    | [**ColorF**](https://msdn.microsoft.com/library/windows/desktop/dd370907)                                              |
| [**IDWriteFactory::CreateTextFormat**](https://msdn.microsoft.com/library/windows/desktop/dd368203)                 | [**DWRITE\_字体\_权重**](https://msdn.microsoft.com/library/windows/desktop/dd368082)                 |
| [**IDWriteTextFormat::SetTextAlignment**](https://msdn.microsoft.com/library/windows/desktop/dd316709)           | [**DWRITE\_文本\_对齐方式**](https://msdn.microsoft.com/library/windows/desktop/dd368131)           |
| [**IDWriteTextFormat::SetParagraphAlignment**](https://msdn.microsoft.com/library/windows/desktop/dd316702) | [**DWRITE\_段落\_对齐方式**](https://msdn.microsoft.com/library/windows/desktop/dd368112) |



 

## <a name="updating-the-main-render-loop"></a>更新主呈现循环

现在，该示例的初始化已完成，我们可以转到主呈现循环。

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



| 呼叫流                                                              | 参数 |
|------------------------------------------------------------------------|------------|
| [**ID3D12CommandList**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandlist)                         |            |
| [**ExecuteCommandLists**](/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists)  |            |
| [**IDXGISwapChain1::Present1**](https://msdn.microsoft.com/library/windows/desktop/hh446797) |            |



 

我们呈现循环到新的唯一内容是**RenderUI**调用，它将使用 D2D 呈现我们的 UI。 请注意，我们执行所有第一次来呈现三维场景，我们 D3D12 命令列表，然后我们在最重要的是呈现我们的 UI。 在我们深入之前**RenderUI**，我们必须考虑到更改**PopulateCommandLists**。 其他示例中我们通常将资源屏障上命令列表之前关闭后从呈现器目标状态缓冲到数据库的当前状态的转换。 但是，在此示例中我们删除该资源屏障，因为我们仍然需要呈现到与 D2D 后台缓冲区。 请注意，当我们创建了我们已包装的资源的后台缓冲区，我们指定的呈现目标状态为"IN"状态和为"OUT"状态的当前状态。

**RenderUI**是相当简单-直接在 D2D 使用情况方面。 我们设置我们呈现器目标并呈现文本。 但是，在之前 11On12 设备上使用已包装的任何资源，如我们后台缓冲区呈现器目标，我们必须调用[ **AcquireWrappedResources** ](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-acquirewrappedresources) 11On12 设备上的 API。 呈现后，我们调用[ **ReleaseWrappedResources** ](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-releasewrappedresources) 11On12 设备上的 API。 通过调用**ReleaseWrappedResources**我们会产生在后台将转换为在创建时指定的"OUT"状态的指定的资源的资源障碍。 在本例中，这是数据库的当前状态。 最后，才能提交命令的所有设备上执行 11On12 共享[ **ID3D12CommandQueue**](/windows/desktop/api/D3D12/nn-d3d12-id3d12commandqueue)，我们必须调用[**刷新**](https://msdn.microsoft.com/library/windows/desktop/ff476425)上[ **ID3D11DeviceContext**](https://msdn.microsoft.com/library/windows/desktop/ff476385)。

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



| 呼叫流                                                                                                                                                                                 | 参数                            |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------|
| [**D2D1\_SIZE\_F**](https://msdn.microsoft.com/library/windows/desktop/dd368160)                                                                                                                                                 |                                       |
| [**D2D1\_RECT\_F**](https://msdn.microsoft.com/library/windows/desktop/dd368151)                                                                                                                                                 | [**RectF**](https://msdn.microsoft.com/library/windows/desktop/dd372343)           |
| [**AcquireWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-acquirewrappedresources)                                                                                                               |                                       |
| [**ID2D1DeviceContext::SetTarget**](https://msdn.microsoft.com/library/windows/desktop/hh404533)                                                                                                                |                                       |
| [**ID2D1RenderTarget::BeginDraw**](https://msdn.microsoft.com/library/windows/desktop/dd371768)                                                                                                                  |                                       |
| [**ID2D1RenderTarget::SetTransform**](https://msdn.microsoft.com/library/windows/desktop/dd742857)                                                                                                            | [**Matrix3x2F**](https://msdn.microsoft.com/library/windows/desktop/dd372275) |
| [**ID2D1RenderTarget::DrawTextW**](https://msdn.microsoft.com/library/windows/desktop/dd371916) |                                       |
| [**ID2D1RenderTarget::EndDraw**](https://msdn.microsoft.com/library/windows/desktop/dd371924)                                                                                                                      |                                       |
| [**ReleaseWrappedResources**](/windows/desktop/api/d3d11on12/nf-d3d11on12-id3d11on12device-releasewrappedresources)                                                                                                               |                                       |
| [**ID3D11DeviceContext::Flush**](https://msdn.microsoft.com/library/windows/desktop/ff476425)                                                                                                                    |                                       |



 

## <a name="run-the-sample"></a>运行示例

![在 12 示例 11 最终输出](images/11on12image.png)

## <a name="related-topics"></a>相关主题

<dl> <dt>

[D3D12 代码演练](d3d12-code-walk-throughs.md)
</dt> <dt>

[在 12 Direct3D 11](direct3d-11-on-12.md)
</dt> <dt>

[Direct3D 12 的互操作](direct3d-12-interop.md)
</dt> <dt>

[11on12 引用](direct3d-11on12-reference.md)
</dt> </dl>

 

 




