---
title: DirectML 中的绑定
description: 基础 DirectML 帮助程序函数的部分代码列表。
ms.custom: Windows 10 May 2019 Update
ms.localizationpriority: high
ms.topic: article
ms.date: 04/19/2019
ms.openlocfilehash: 52303618c146bddc0318a01e28abeb92aea92745
ms.sourcegitcommit: 8141395d1bd1cd755d1375715538c3fe714ba179
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67465016"
---
# <a name="directml-helper-functions"></a>DirectML 帮助程序函数

## <a name="dmlcalcbuffertensorsize"></a>DMLCalcBufferTensorSize

该帮助程序函数计算存储具有指定类型、大小和步幅的缓冲区张量所需的最小字节数。 公式可如下表示。

```cpp
IndexOfLastElement = dot(Sizes - 1, Strides);
MinimumImpliedSizeInBytes = roundup((IndexOfLastElement + 1) * ElementSizeInBytes, 4)
```

换言之，张量的最小大小是乘以元素大小（例如，FLOAT16 张量的 2 字节）的超尾后的任何一个元素的索引  。 此外，DirectML 要求所有绑定的缓冲区的总大小必须是 DWORD 对齐，因此以字节为单位的最小隐含大小必须上调取整为最接近的 4 字节边界  。

```cppwinrt
inline UINT64 DMLCalcBufferTensorSize(
    DML_TENSOR_DATA_TYPE dataType,
    UINT dimensionCount,
    _In_reads_(dimensionCount) const UINT* sizes,
    _In_reads_opt_(dimensionCount) const UINT* strides
    )
{
    UINT elementSizeInBytes = 0;
    switch (dataType)
    {
    case DML_TENSOR_DATA_TYPE_FLOAT32:
    case DML_TENSOR_DATA_TYPE_UINT32:
    case DML_TENSOR_DATA_TYPE_INT32:
        elementSizeInBytes = 4;
        break;

    case DML_TENSOR_DATA_TYPE_FLOAT16:
    case DML_TENSOR_DATA_TYPE_UINT16:
    case DML_TENSOR_DATA_TYPE_INT16:
        elementSizeInBytes = 2;
        break;

    case DML_TENSOR_DATA_TYPE_UINT8:
    case DML_TENSOR_DATA_TYPE_INT8:
        elementSizeInBytes = 1;
        break;

    default:
        return 0; // Invalid data type
    }

    UINT64 minimumImpliedSizeInBytes = 0;
    if (!strides)
    {
        minimumImpliedSizeInBytes = sizes[0];
        for (UINT i = 1; i < dimensionCount; ++i)
        {
            minimumImpliedSizeInBytes *= sizes[i];
        }
        minimumImpliedSizeInBytes *= elementSizeInBytes;
    }
    else
    {
        UINT indexOfLastElement = 0;
        for (UINT i = 0; i < dimensionCount; ++i)
        {
            indexOfLastElement += (sizes[i] - 1) * strides[i];
        }

        minimumImpliedSizeInBytes = (indexOfLastElement + 1) * elementSizeInBytes;
    }

    // Round up to the nearest 4 bytes.
    minimumImpliedSizeInBytes = (minimumImpliedSizeInBytes + 3) & ~3ui64;

    return minimumImpliedSizeInBytes;
}
```

## <a name="calculatestrides"></a>CalculateStrides

该帮助程序函数计算具有 NCHW 或 NHWC 布局以及可选广播的 4D 张量的步幅。

```cppwinrt
enum class Layout
{
    NCHW,
    NHWC
};

// Given dimension sizes (in NCHW order), calculates the strides to achieve a desired layout.
std::array<uint32_t, 4> CalculateStrides(
        Layout layout, 
        std::array<uint32_t, 4> sizes, 
        std::array<bool, 4> broadcast)
{
    enum DML_ORDER { N, C, H, W };

    uint32_t n = broadcast[N] ? 1 : sizes[N];
    uint32_t c = broadcast[C] ? 1 : sizes[C];
    uint32_t h = broadcast[H] ? 1 : sizes[H];
    uint32_t w = broadcast[W] ? 1 : sizes[W];

    uint32_t nStride = 0, cStride = 0, hStride = 0, wStride = 0;

    switch (layout)
    {
    case Layout::NCHW:
        nStride = broadcast[N] ? 0 : c * h * w;
        cStride = broadcast[C] ? 0 : h * w;
        hStride = broadcast[H] ? 0 : w;
        wStride = broadcast[W] ? 0 : 1;
        break;

    case Layout::NHWC:
        nStride = broadcast[N] ? 0 : h * w * c;
        hStride = broadcast[H] ? 0 : w * c;
        wStride = broadcast[W] ? 0 : c;
        cStride = broadcast[C] ? 0 : 1;
        break;
    }

    return { nStride, cStride, hStride, wStride };
}
```
