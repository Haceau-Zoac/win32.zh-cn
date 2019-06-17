---
title: DirectML 中的绑定
description: DirectML 张量由张量的大小和步幅属性描述   。
ms.custom: 19H1
ms.topic: article
ms.date: 03/12/2019
ms.openlocfilehash: fdd1655dca41f9b3c79d5975f2f1e5721b7634fb
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66223972"
---
# <a name="using-strides-to-express-padding-and-memory-layout"></a>使用步幅来表示填充和内存布局

Direct3D 12 缓冲区支持的 DirectML 张量由张量的大小和步幅属性描述   。 张量的大小描述张量的逻辑维度。  例如，2D 张量的高度为 2，宽度为 3。 从逻辑上讲，张量具有 6 个相异的元素，不过，大小并不指定这些元素在内存中的存储方式。 张量的步幅描述张量元素的物理内存布局。 

## <a name="two-dimensional-2d-arrays"></a>二维 (2D) 数组

假设某个 2D 张量的高度为 2，宽度为 3；数据由文本字符组成。 在 C/C++ 中，可以使用多维数组表示此张量。

```cpp
constexpr int rows = 2;
constexpr int columns = 3;
char tensor[rows][columns];
tensor[0][0] = 'A';
tensor[0][1] = 'B';
tensor[0][2] = 'C';
tensor[1][0] = 'D';
tensor[1][1] = 'E';
tensor[1][2] = 'F';
```

下面是上述张量的可视化逻辑视图。

```console
A B C
D E F
```

在 C /C++ 中，按行主序存储多维数组。 换言之，沿宽度维分布的连续元素将连续存储在线性内存空间中。

偏移量：|0|1|2|3|4|5
-|-|-|-|-|-|-
值：|A|B|C|D|E|F

维度的步幅是为了访问该维度中的下一个元素而要跳过的元素数。  步幅表示内存中的张量布局。 使用行主序时，宽度维度的步幅始终为 1，因为维度上的相邻元素是连续存储的。 高度维度的步幅依赖于宽度维度的大小；在上述示例中，高度维度上的连续元素之间的距离（例如 A 到 D）等于张量的宽度（在本示例中为 3）。

若要演示不同的布局，请考虑列主序。 换言之，沿高度维分布的连续元素将连续存储在线性内存空间中。 在本例中，高度步幅始终为 1，宽度步幅为 2（高度维的大小）。

偏移量：|0|1|2|3|4|5
-|-|-|-|-|-|-
值：|A|D|B|E|C|F

## <a name="higher-dimensions"></a>更高的维度

超过两个维度时，以行主序或列主序方式引用布局会很不方便。 因此，本主题的余下部分使用了如下所述的术语和标签。

- 2D：“HW”&mdash; 高度是最高序维度（行主序）。
- 2D：“WH”&mdash; 宽度是最高序维度（列主序）。
- 3D：“DHW”&mdash; 深度是最高序维度，依次后接高度和宽度。
- 3D：“WDH”&mdash; 宽度是最高序维度，依次后接高度和深度。
- 4D：“NCHW”&mdash; 依次为图像数目（批大小）、通道数、高度、宽度。

一般情况下，维度的打包步幅等于低序维度大小的乘积。  例如，对于“DHW”布局，D 步幅等于高度 * 宽度；H 步幅等于宽度；W 步幅等于 1。 如果张量的总物理大小等于张量的总逻辑大小，则认为步幅是打包的；换言之，不存在任何额外的空间，也不存在重叠的元素。 

让我们将 2D 示例扩展为三维，使张量的深度、高度和宽度分别为 2、2、3（总共 12 个逻辑元素）。

```console
A B C
D E F

G H I
J K L
```

对于“DHW”布局，此张量将按以下方式存储。

偏移量：|0|1|2|3|4|5|6|7|8|9|10|11|
-|-|-|-|-|-|-|-|-|-|-|-|-|
值：|A|B|C|D|E|F|G|H|I|J|K|L|

- D 步幅 = 高度 (2) * 宽度 (3) = 6（例如，“A”到“G”的距离）。
- H 步幅 = 宽度 (3) = 3（例如，“A”到“G”的距离）。
- W 步幅 = 1（例如，“A”到“B”的距离）。

元素的索引/坐标和步幅的点积提供了该元素在缓冲区中的偏移量。 例如，H 元素 (d=1, h=0, w=1) 的偏移量为 7。

{1, 0, 1} ⋅ {6, 3, 1} = 1 * 6 + 0 * 3 + 1 * 1 = 7

## <a name="packed-tensors"></a>打包的张量

以上示例演示了打包的张量。  如果（元素中）张量的逻辑大小等于（元素中）缓冲区的物理大小，并且每个元素具有唯一的地址/偏移量，则认为该张量是打包的。  例如，如果缓冲区的长度为 12 个元素，并且没有任何一对元素在缓冲区中使用相同的偏移量，则 2x2x3 张量是打包的。 打包的张量是最常见的情况；但步幅允许更复杂的内存布局。

## <a name="broadcasting-with-strides"></a>使用步幅进行广播

如果（元素中）张量的缓冲区大小小于其逻辑维度的积，则必然存在一些重叠的元素。 这种常见情况称为“广播”；其中某个维度的元素是另一个维度的重复项。  例如，让我们重新探讨 2D 示例。 假设我们想要获得一个逻辑维度为 2x3 的张量，但第二行与第一行相同。 布局如下。

```console
A B C
A B C
```

此张量可存储为打包的 HW/行主序张量。 但是，更紧凑的存储只包含 3 个元素（A、B 和 C），并使用高度步幅 0，而不是 3。 在这种情况下，张量的物理大小为 3 个元素，但逻辑大小为 6 个元素。

一般而言，如果维度的步幅为 0，则较低序维度中的所有元素将沿广播维度重复；例如，如果张量为 NCHW，C 步幅为 0，则每个通道在 H 和 W 维上的值相同。

## <a name="padding-with-strides"></a>使用步幅进行填充

如果某个张量的物理大小大于拟合其元素所需的最小大小，则认为该张量是填充的。  如果既不存在广播也不存在重叠的元素，则（元素中）张量的最小大小仅是其维度的积。 可以使用帮助器函数 `DMLCalcBufferTensorSize`（有关该函数的列表，请参阅 [DirectML 帮助器函数](dml-helper-functions.md)）来计算 DirectML 张量的最小缓冲区大小。 

假设某个缓冲区包含以下值（“x”元素指示填充值）。

0|1|2|3|4|5|6|7|8|9|
-|-|-|-|-|-|-|-|-|-|
A|B|C|x|x|D|E|F|x|x

可以使用高度步幅 5 而不是 3 来描述填充的张量。 不是按 3 个元素步进到下一行，而步幅为 5（3 个真实元素加上 2 个填充元素）。  例如，填充在计算机图形中很常见，目的是确保图像采用 2 次幂对齐。

```console
A B C
D E F
```

## <a name="directml-buffer-tensor-descriptions"></a>DirectML 缓冲区张量描述

DirectML 可以使用各种物理张量布局，因为 [**DML_BUFFER_TENSOR_DESC** 结构](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc)包含 `Sizes` 和 `Strides` 成员。 使用特定的布局时，某些运算符实现可能更高效，为了提高性能，用户经常会改变张量数据的存储方式。

大多数 DirectML 运算符需要 4D 或 5D 张量，而大小和步幅值的阶是固定的。 通过在张量描述中固定大小和步幅值的阶，可让 DirectML 推理出不同的物理布局。

**4D**
- [**DML_BUFFER_TENSOR_DESC::Sizes**](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc) = { N 大小, C 大小, H 大小, W 大小 }
- [**DML_BUFFER_TENSOR_DESC::Strides**](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc) = { N 步幅, C 步幅, H 步幅, W 步幅 }

**5D**
- **DML_BUFFER_TENSOR_DESC::Sizes** = { N 大小, C 大小, D 大小, H 大小, W 大小 }
- **DML_BUFFER_TENSOR_DESC::Strides** = { N 步幅, C 步幅, D 步幅, H 步幅, W 步幅 }

如果 DirectML 运算符需要 4D 或 5D 张量，但实际数据的秩更小（例如 2D），则应使用 1s 填充前导维度。 例如，使用 **DML_BUFFER_TENSOR_DESC::Sizes** = { 1, 1, H, W } 设置“HW”张量。

如果张量数据存储在 NCHW/NCDHW 中，则除非要进行广播或填充，否则不需要设置 **DML_BUFFER_TENSOR_DESC::Strides**。 可将步幅字段设置为 `nullptr`。 但是，如果张量数据存储在另一个布局中（例如 NHWC），则需要使用步幅来表示从 NCHW 到该布局的转换。

举个简单的例子，假设使用高度 3 和宽度 5 描述某个 2D 张量。

**打包的 NCHW（隐式步幅）**
- **DML_BUFFER_TENSOR_DESC::Sizes** = { 1, 1, 3, 5 }
- **DML_BUFFER_TENSOR_DESC::Strides** = `nullptr`

**打包的 NCHW（显式步幅）**
- N 步幅 = C 大小 * H 大小 * W 大小 = 1 * 3 * 5 = 15
- C 步幅 = H 大小 * W 大小 = 3 * 5 = 15
- H 步幅 = W 大小 = 5
- W 步幅 = 1
- **DML_BUFFER_TENSOR_DESC::Sizes** = { 1, 1, 3, 5 }
- **DML_BUFFER_TENSOR_DESC::Strides** = { 15, 15, 5, 1 }

**打包的 NHWC**
- N 步幅 = H 大小 * W 大小 * C 大小 = 3 * 5 * 1 = 15
- H 步幅 = W 大小 * C 大小 = 5 * 1 = 5
- W 步幅 = C 大小 = 1
- C 步幅 = 1
- **DML_BUFFER_TENSOR_DESC::Sizes** = { 1, 1, 3, 5 }
- **DML_BUFFER_TENSOR_DESC::Strides** = { 15, 1, 5, 1 }

## <a name="see-also"></a>另请参阅

* [DirectML 帮助器函数](dml-helper-functions.md)
* [DML_BUFFER_TENSOR_DESC 结构](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc)
