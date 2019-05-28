---
title: DirectML 中的绑定
description: 由称为属性描述 DirectML tensors*大小*并*进步*的 tensor。
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
# <a name="using-strides-to-express-padding-and-memory-layout"></a>使用进步表示填充和内存布局

DirectML tensors&mdash;哪些受 Direct3D 12 缓冲区&mdash;由名为的属性描述*大小*并*进步*的 tensor。 Tensor*大小*描述 tensor 的逻辑维度。 例如，2D tensor 可能高度为 2 和 3 的宽度。 在逻辑上，tensor 具有 6 个非重复元素，虽然大小未指定这些元素在内存中的存储方式。 Tensor*进步*描述 tensor 的元素的物理内存布局。

## <a name="two-dimensional-2d-arrays"></a>二维 (2D) 数组

请考虑具有 2 的高度和宽度为 3; 2D tensor数据由文本字符组成。 在 C /C++，这可能表示使用多维数组。

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

下面被可视化上述 tensor 的逻辑视图。

```console
A B C
D E F
```

在 C /C++，按行优先顺序存储多维数组。 换而言之，沿宽度维度的连续元素线性内存空间中存储连续。

偏移量：|0|1|2|3|4|5
-|-|-|-|-|-|-
值：|A|B|C|D|E|F

*Stride*维度的是要跳过才能访问该维度中的下一个元素的元素数。 进步 express tensor 在内存中的布局。 具有行优先顺序，跨距宽度维度始终为 1，因为连续存储维度上的相邻元素。 跨距的高度维度取决于宽度维度; 的大小在上述示例中，此过程的高度维度 (例如，从 A 到 D) 的连续元素之间的距离等于 tensor （这是在此示例中的 3） 的宽度。

为了说明不同的布局，请考虑列优先顺序。 换而言之，此过程的高度维度的连续元素线性内存空间中存储连续。 在这种情况下，高度 stride 始终为 1，且宽度 stride 是 2 （的高度维度的大小）。

偏移量：|0|1|2|3|4|5
-|-|-|-|-|-|-
值：|A|D|B|E|C|F

## <a name="higher-dimensions"></a>更高的维度

谈大于两个维度，它是难以处理来指代布局为行优先还是列优先。 因此，本主题的其余部分使用条款和这些标签。

- 2D:"硬件"&mdash;高度是最高序位维度 （行优先）。
- 2D:"以下值时"&mdash;宽度是最高序位维度 （列优先）。
- 3D:"DHW"&mdash;深度是最高序位维度后, 跟高度和宽度。
- 3D:"WHD"&mdash;宽度是最高序位维度后, 跟高度和深度。
- 4D:"NCHW"&mdash;映像 （批大小），该数字然后通道，然后高度，宽度的数量。

一般情况下，*打包*stride 维度是否等于该产品的较低序位维数的大小。 例如，具有"DHW"布局，D stride 等于 H * W;H stride 是相等到 W;和 W stride 值等于 1。 进步被称为*打包*tensor 的总物理大小等于 tensor; 的总逻辑大小时换而言之，没有任何额外的空间也不重叠的元素。

让我们 2D 将示例扩展到三个维度，它使我们能够 tensor 深度 2、 2、 高度和宽度 3 （共 12 个逻辑元素）。

```console
A B C
D E F

G H I
J K L
```

具有"DHW"布局，此 tensor 存储中，如下所示。

偏移量：|0|1|2|3|4|5|6|7|8|9|10|11|
-|-|-|-|-|-|-|-|-|-|-|-|-|
值：|A|B|C|D|E|F|G|H|I|J|K|L|

- D stride = 高度 (2) * 宽度 (3) = 6 （例如，A 和 G 之间的距离）。
- H stride = (3) = 3 (例如，A 之间的距离和具有)。
- W stride = 1 （例如，A 和 B 之间的距离）。

元素和进步的索引/坐标点积提供到缓冲区中该元素的偏移量。 例如，H 元素的偏移量 (d = 1，h = 0，w = 1) 为 7。

{1, 0, 1} ⋅ {6, 3, 1} = 1 * 6 + 0 * 3 + 1 * 1 = 7

## <a name="packed-tensors"></a>已打包的 tensors

上面的示例演示了*打包*tensors。 Tensor 称为*打包*时 tensor （元素） 中的逻辑大小相同的物理大小的缓冲区 （在元素），且每个元素具有唯一的地址/偏移量。 例如，如果缓冲区是 12 个元素的长度，没有一对元素共享相同的偏移量在缓冲区中打包 3 tensor x 2 x 2。 已打包的 tensors 是最常见的情况;但进步允许更复杂的内存布局。

## <a name="broadcasting-with-strides"></a>使用进步的广播

如果 tensor 的缓冲区的大小 （以元素） 小于其逻辑的维度的产品，它遵循必须有一些重叠的元素。 此常见情况通常所说*广播*; 其中维度的元素是重复的另一个维度。 例如，让我们重新探讨 2D 示例。 假设我们想要在逻辑上就 2 x 3 个，tensor 但第二行是相同的第一行。 下面是这看起来。

```console
A B C
A B C
```

这可以存储为已打包的 HW/行优先 tensor。 但更紧凑的存储将包含只有 3 个元素 （A、 B 和 C） 和使用高度的跨距的值为 0，而不是 3。 在这种情况下，tensor 的物理大小为 3 个元素，但逻辑大小为 6 个元素。

一般情况下，如果跨距维度为 0，则较低序位维度中的所有元素被都重复广播维度;例如，如果 tensor NCHW，并且 C stride，然后每个通道具有相同的值沿 H 和 w.

## <a name="padding-with-strides"></a>这时将需要使用进步

Tensor 称为*填充*如果其物理大小大于所需以适合其元素的最小大小。 如果有不广播也不重叠的元素，tensor （元素） 中的最小大小是只是其维度的产品。 可以使用 helper 函数`DMLCalcBufferTensorSize`(请参阅[DirectML 帮助器函数](dml-helper-functions.md)有关该函数的列表) 来计算*最小*缓冲区 DirectML tensors 的大小。

让我们假设一个缓冲区包含以下值 （'x' 元素表示填充的值）。

0|1|2|3|4|5|6|7|8|9|
-|-|-|-|-|-|-|-|-|-|
A|B|C|x|x|D|E|F|x|x

填充的 tensor 可以通过使用 5 高度 stride，而不 3 所述。 而不是单步执行由 3 个元素以转到下一行，该步骤是 5 个元素 (3*真实*元素加上 2 填充元素)。 填充常见于计算机图形中，例如，若要确保映像具有 2 电源的对齐方式。

```console
A B C
D E F
```

## <a name="directml-buffer-tensor-descriptions"></a>DirectML 缓冲区 tensor 说明

DirectML 可使用各种物理 tensor 布局，因为[ **DML_BUFFER_TENSOR_DESC**结构](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc)同时具有`Sizes`和`Strides`成员。 某些运算符实现可能更高效地使用一个特定的布局，因此并不少见，若要更改 tensor 数据以提高性能的存储方式。

大多数 DirectML 运算符要求 4 D 或 5d tensors 和大小的顺序并且进步值固定不变。 通过修复 tensor 说明中的大小和 stride 值的顺序，很可能 DirectML 推断出不同的物理布局。

**4D**
- [**DML_BUFFER_TENSOR_DESC::Sizes**](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc) = { N-size, C-size, H-size, W-size }
- [**DML_BUFFER_TENSOR_DESC::Strides**](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc) = { N-stride, C-stride, H-stride, W-stride }

**5D**
- **DML_BUFFER_TENSOR_DESC::Sizes** = { N-size, C-size, D-size, H-size, W-size }
- **DML_BUFFER_TENSOR_DESC::Strides** = { N-stride, C-stride, D-stride, H-stride, W-stride }

如果 DirectML 运算符要求 4 D 或 5d tensor，但实际数据具有较小排名 （例如，2D），则前导维度应填充为 1。 例如，使用设置"HW"tensor **DML_BUFFER_TENSOR_DESC::Sizes** = {1，1，H、 W}。

如果 tensor 数据存储在 NCHW/NCDHW，则不需要设置**DML_BUFFER_TENSOR_DESC::Strides**，除非您希望广播或填充。 可以将进步字段设置为`nullptr`。 但是，如果 tensor 数据存储在另一个布局，如 NHWC，则需要进步为了表示从 NCHW 到该布局的转换。

一个简单的示例，请考虑使用 3 高度和宽度 5 2D tensor 的说明。

**已打包的 NCHW （隐式进步）**
- **DML_BUFFER_TENSOR_DESC::Sizes** = { 1, 1, 3, 5 }
- **DML_BUFFER_TENSOR_DESC::Strides** = `nullptr`

**已打包的 NCHW （显式进步）**
- N stride = C 大小 * H 大小 * W 大小 = 1 * 3 * 5 = 15 个
- C stride = H 大小 * W 大小 = 3 * 5 = 15 个
- H stride = W 大小 = 5
- W stride = 1
- **DML_BUFFER_TENSOR_DESC::Sizes** = { 1, 1, 3, 5 }
- **DML_BUFFER_TENSOR_DESC::Strides** = { 15, 15, 5, 1 }

**已打包的 NHWC**
- N stride = H 大小 * W 大小 * C 大小 = 3 * 5 * 1 = 15 个
- H stride = W 大小 * C 大小 = 5 * 1 = 5
- W stride = C 大小 = 1
- C stride = 1
- **DML_BUFFER_TENSOR_DESC::Sizes** = { 1, 1, 3, 5 }
- **DML_BUFFER_TENSOR_DESC::Strides** = { 15, 1, 5, 1 }

## <a name="see-also"></a>另请参阅

* [DirectML 帮助器函数](dml-helper-functions.md)
* [DML_BUFFER_TENSOR_DESC 结构](/windows/desktop/api/directml/ns-directml-dml_buffer_tensor_desc)
