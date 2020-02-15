---
title: 根签名版本 1.1
description: 根签名版本 1.1 的目的是使应用程序能够向驱动程序说明描述符堆中的描述符不会更改，或描述符指向的数据不会变化。
ms.assetid: 8FE42C1C-7F1D-4E70-A7EE-D5EC67237327
ms.localizationpriority: high
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 24d0a107b4186f164e60a6225f55c5ba3f4f06b2
ms.sourcegitcommit: f5a319899176e2df564bdb4d9eaffc32140452a2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2020
ms.locfileid: "77256922"
---
# <a name="root-signature-version-11"></a>根签名版本 1.1

根签名版本 1.1 的目的是使应用程序能够向驱动程序说明描述符堆中的描述符不会更改，或描述符指向的数据不会变化。 这样，驱动程序就知道描述符或它指向的内存在一段时间内是静态的，从而可以执行一些能够在这种情况下实现的优化。

-   [概述](#overview)
-   [静态和易失性标志](#static-and-volatile-flags)
    -   [DESCRIPTORS\_VOLATILE](#descriptors_volatile)
    -   [DATA\_VOLATILE](#data_volatile)
    -   [DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE](#data_static_while_set_at_execute)
    -   [DATA\_STATIC](#data_static)
    -   [合并标志](#combining-flags)
    -   [标志摘要](#flag-summary)
-   [版本 1.1 API 摘要](#version-11-api-summary)
    -   [枚举](#enums)
    -   [结构](#helper-structures)
    -   [函数](#functions)
    -   [方法](#methods)
    -   [帮助器结构](#helper-structures)
-   [违反静态性标志的后果](#consequences-of-violating-static-ness-flags)
-   [版本管理](#version-management)
-   [相关主题](#related-topics)

## <a name="overview"></a>概述

在引用描述符堆的命令列表/捆绑包可能正在 GPU 上运行的情况下，根签名版本 1.0 允许应用程序在任意时间自由更改描述符堆的内容及它们指向的内存。 但是，在引用描述符或内存的命令已记录后，应用程序实际上往往不需要灵活更改这些描述符或内存。

应用程序通常能够轻松地：

-   在命令列表或捆绑中绑定描述符表或根描述符之前设置描述符（有时还能设置它们指向的内存）。
-   确保这些描述符在引用它们的命令列表/捆绑最后一次完成执行之前不会发生更改。
-   确保描述符指向的数据在相同的整个持续时间内不会发生更改。

或者，应用程序只能接受数据在较短持续时间内不会更改。 具体而言，数据可能在执行命令列表期间当根参数绑定（描述符表或根描述符）当前指向数据的时间范围内保持静态。 换而言之，在 GPU 时间线上的一些时间段内，某些数据由根参数设置并会在设置后保持静态，应用程序知道这种情况并希望在这些时间段之间执行会使这些数据更新的操作。

如果描述符或描述符所指向的数据不会更改，则驱动程序可以实现的具体优化将特定于硬件供应商，重要的是，它们不会改变行为，而只可能会提高性能。 尽可能多地了解应用程序的意图不会给应用程序造成负担。

一个优化方式是，如果驱动程序知道应用程序在描述符和数据静态性方面做出的承诺，则可以使用许多驱动程序来生成可供着色器访问的更高效内存。 例如，如果特定的硬件对根参数大小不敏感，则驱动程序可以通过将堆中的某个描述符转换为根描述符，来消除访问该描述符时的间接性级别。

使用版本 1.1 的开发人员的附加任务是，尽可能地在数据易失性和静态性方面做出承诺，使驱动程序能够实现有意义的优化。 开发人员不必要在静态性方面做出任何承诺。

根签名版本 1.0 会持续按原样正常运行，但重新编译根签名的应用程序现在默认会使用根签名 1.1（有一个选项可按需强制使用版本 1.0）。

## <a name="static-and-volatile-flags"></a>静态和易失性标志

以下标志是根签名的一部分，可让驱动程序选择一种策略，以便在设置各个根参数后以最适当的方法处理这些参数；另外，可将相同的假设嵌入到最初编译好的管道状态对象 (PSO) 中 - 因为根签名是 PSO 的一部分。

以下标志由应用设置，将应用到描述符或数据。

``` syntax
typedef enum D3D12_DESCRIPTOR_RANGE_FLAGS
{
    D3D12_DESCRIPTOR_RANGE_FLAG_NONE    = 0,
    D3D12_DESCRIPTOR_RANGE_FLAG_DESCRIPTORS_VOLATILE    = 0x1,
    D3D12_DESCRIPTOR_RANGE_FLAG_DATA_VOLATILE   = 0x2,
    D3D12_DESCRIPTOR_RANGE_FLAG_DATA_STATIC_WHILE_SET_AT_EXECUTE    = 0x4,
    D3D12_DESCRIPTOR_RANGE_FLAG_DATA_STATIC = 0x8
} D3D12_DESCRIPTOR_RANGE_FLAGS;

typedef enum D3D12_ROOT_DESCRIPTOR_FLAGS
{
    D3D12_ROOT_DESCRIPTOR_FLAG_NONE = 0,
    D3D12_ROOT_DESCRIPTOR_FLAG_DATA_VOLATILE    = 0x2,
    D3D12_ROOT_DESCRIPTOR_FLAG_DATA_STATIC_WHILE_SET_AT_EXECUTE = 0x4,
    D3D12_ROOT_DESCRIPTOR_FLAG_DATA_STATIC  = 0x8
} D3D12_ROOT_DESCRIPTOR_FLAGS;
```

### <a name="descriptors_volatile"></a>DESCRIPTORS\_VOLATILE

设置此标志后，应用程序随时可以更改根描述符表指向的描述符堆中的描述符，但当绑定描述符表的命令列表/捆绑已提交和未完成执行时则不可以。 例如，在提交命令列表供执行之前，记录该命令列表并随后更改它引用的描述符堆中的描述符是有效的操作。 这是根签名版本 1.0 唯一支持的行为。

如果未设置 DESCRIPTORS\_VOLATILE 标志，则描述符是静态的。 此模式没有标志。 静态描述符表示在命令列表/捆绑中设置描述符表时（录制期间）已初始化根描述符表指向的描述符堆中的描述符，并且在命令列表/捆绑最后一次完成执行之前，描述符无法更改。 对于根签名版本 1.1，静态描述符是默认的假设，应用程序必须根据需要指定 DESCRIPTORS*VOLATILE 标志。* \_

对于使用包含静态描述符的描述符表的捆绑，记录捆绑时（而不是调用该捆绑时），描述符必须已准备好启动，并且在该捆绑最后一次完成执行之前不会更改。 必须在记录捆绑期间设置指向静态描述符的描述符表，它们不能继承到捆绑中。 命令列表可以使用包含静态描述符的、已在捆绑中设置的并返回到命令列表的描述符表。

如果描述符是静态的，另一种行为变化需要设置 DESCRIPTORS\_VOLATILE 标志。 对任何缓冲区视图（而不是 Texture1D/2D/3D/多维视图）进行界外访问都是无效的操作，会生成不确定的结果，包括可能的设备重置，而不是返回读取操作的默认值或丢弃写入操作。 去除应用程序依赖硬件界外访问权限检查的功能的目的在于，在驱动程序认为更有效的情况下，可让驱动程序选择将静态描述符访问权限提升为根描述符访问权限。 根描述符不支持任何界外检查。

如果应用程序在访问描述符时依赖于安全界外内存访问行为，则它们需要将访问这些描述符的描述符范围标记为 DESCRIPTORS\_VOLATILE。

### <a name="data_volatile"></a>DATA\_VOLATILE

设置此标志后，则 CPU 随时可以更改描述符指向的数据，但当绑定描述符表的命令列表/捆绑已提交且尚未完成执行时则不可以。 这是根签名版本 1.0 唯一支持的行为。

可在描述符范围标志和根描述符标志中使用该标志。

### <a name="data_static_while_set_at_execute"></a>DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE

设置此标志后，从在 GPU 时间线上执行期间在命令列表/捆绑中设置基础根描述符或描述符表开始，到后续绘制/调度不再引用数据为止，描述符指向的数据无法更改。

在 GPU 上设置根描述符或描述符表之前，此数据甚至可由相同的命令列表/捆绑更改。 当引用数据的根描述符或描述符表仍已在命令列表/捆绑中设置时，只要引用该数据的绘制/调度已完成，则也可以更改数据。 但是，这样做需要在下次取消引用根描述符或描述符表之前，将描述符表重新绑定到命令列表。 这样，驱动程序便知道根描述符或描述符表指向的数据已更改。

DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE 与 DATA\_VOLATILE 之间的本质差别在于，使用 DATA\_VOLATILE 时，在不执行额外的状态跟踪的情况下，驱动程序无法判断命令列表中的数据副本是否已更改描述符指向的数据。 因此，举例而言，如果驱动程序将任何类型的数据预提取命令插入到其命令列表（例如，使着色器能够更有效地访问已知数据），则 DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE 可让驱动程序知道，它只需在通过 [**SetGraphicsRootDescriptorTable**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootdescriptortable)、[**SetComputeRootDescriptorTable**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootdescriptortable)，或用于设置常量缓冲区视图、着色器资源视图或无序访问视图的某种方法设置时，才执行数据预提取。

对于捆绑，在执行时设置的数据静态性承诺将以独特方式应用到捆绑的每个执行。

可在描述符范围标志和根描述符标志中使用该标志。

### <a name="data_static"></a>DATA\_STATIC

如果设置此标志，记录期间在命令列表/捆绑中设置引用内存的根描述符或描述符表时，将初始化描述符指向的数据，并且在命令列表/捆绑最后一次完成执行之前，该数据无法更改。

对于捆绑，静态持续时间将从记录捆绑而不是记录调用方命令列表期间设置根描述符或描述符表开始。 此外，指向静态数据的描述符表必须在捆绑中设置，而不是继承的。 命令列表可以使用指向静态数据的、已在捆绑中设置的并返回到命令列表的描述符表。

可在描述符范围标志和根描述符标志中使用该标志。

### <a name="combining-flags"></a>合并标志

每次最多可以指定一个 DATA 标志，但根据不支持 DATA 标志的采样器描述符范围除外，因为采样器不指向数据。

缺少 SRV 和 CBV 描述符范围的 DATA 标志意味着采用默认的 DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE 行为。 之所以选择此默认行为而不是 DATA\_STATIC，是因为 DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE 在大多数情况下更有可能是安全的行为，同时，与默认使用 DATA\_VOLATILE 相比，它仍可以带来某种优化机会。

在通常写入到 UAV 的情况下，缺少 UAV 描述符范围的 DATA 标志意味着采用默认的 DATA\_VOLATILE 行为。

DESCRIPTORS\_VOLATILE 不能与 DATA*STATIC 合并，但可与其他 DATA 标志合并。* \_ DESCRIPTORS\_VOLATILE 之所以能够与 DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE 合并，是因为在执行命令列表/捆绑期间，易失性描述符仍需描述符准备就绪，而 DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE 只会对命令列表/捆绑执行的子集内部的静态性做出承诺。

### <a name="flag-summary"></a>标志摘要

下表汇总了可以采用的标志组合。



|                                                                |                                                                                                                                                                                                                                                                                                                                                      |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **有效的 D3D12\_DESCRIPTOR\_RANGE\_FLAGS 设置**             | **描述**                                                                                                                                                                                                                                                                                                                                      |
| 未设置标志                                                   | 描述符是静态的（默认设置）。 数据的默认假设：对于 SRV/CBV： DATA\_静态\_，同时\_在\_EXECUTE 和 UAV： DATA\_VOLATILE 设置\_。 SRV/CBV 的这些默认设置能够安全适应大多数根签名的使用模式。                                                                                              |
| DATA\_STATIC                                                   | 描述符和数据都是静态的。 这可以最大化驱动程序优化的潜力。                                                                                                                                                                                                                                                          |
| DATA\_VOLATILE                                                 | 描述符是静态的，数据是易失性的。                                                                                                                                                                                                                                                                                                     |
| DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE                          | 描述符是静态的，数据在执行期间设置时是静态的。                                                                                                                                                                                                                                                                                      |
| DESCRIPTORS\_VOLATILE                                          | 描述符是可变的，并对数据进行默认假设：对于 SRV/CBV： DATA\_静态\_，同时\_在\_EXECUTE 和 UAV： DATA\_VOLATILE 设置\_。                                                                                                                                                                                              |
| DESCRIPTORS\_VOLATILE \| DATA\_VOLATILE                        | 描述符和数据都是易失性的，等效于根签名 1.0。                                                                                                                                                                                                                                                                            |
| DESCRIPTORS\_VOLATILE \| DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE | 描述符是易失性的，但请注意，仍不允许它们在执行命令列表期间发生更改。 因此，在执行期间通过根描述符表设置时，可以合并附加的数据静态性声明 – 基础描述符保持静态的时间实际上长于承诺数据保持静态的时间。 |



 



|                                                   |                                                                                                                                                                                                                   |
|---------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **有效的 D3D12\_ROOT\_DESCRIPTOR\_FLAGS 设置** | **描述**                                                                                                                                                                                                   |
| 未设置标志                                      | 数据的默认假设：对于 SRV/CBV： DATA\_静态\_，同时\_在\_EXECUTE 和 UAV： DATA\_VOLATILE 设置\_。 SRV/CBV 的这些默认设置能够安全适应大多数根签名的使用模式。 |
| DATA\_STATIC                                      | 数据是静态的，驱动程序优化的潜力最大。                                                                                                                                                       |
| DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE             | 数据在执行期间设置时是静态的。                                                                                                                                                                              |
| DATA\_VOLATILE                                    | 等效于根签名 1.0。                                                                                                                                                                                 |



 

## <a name="version-11-api-summary"></a>版本 1.1 API 摘要

以下 API 调用支持版本 1.1。

### <a name="enums"></a>枚举

这些枚举包含用于指定描述符和数据易失性的关键标志。

-   [**D3D\_ROOT\_SIGNATURE\_VERSION**](/windows/desktop/api/d3d12/ne-d3d12-d3d_root_signature_version)：版本 ID。
-   [**D3D12\_DESCRIPTOR\_RANGE\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_descriptor_range_flags)：标志范围，确定描述符或数据是易失性的还是静态的。
-   [**D3D12\_ROOT\_DESCRIPTOR\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_root_descriptor_flags)：类似于 [**D3D12\_DESCRIPTOR\_RANGE\_FLAGS**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_descriptor_range_flags) 的标志范围，不过，只会将数据标志应用到根描述符。

### <a name="structures"></a>结构

更新的结构（版本 1.0 中）包含对易失性/静态标志的引用。

-   [**D3D12\_FEATURE\_DATA\_ROOT\_SIGNATURE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_root_signature)：将此结构传递给 [**CheckFeatureSupport**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-checkfeaturesupport) 可以检查对根签名版本 1.1 的支持。
-   [**D3D12\_VERSIONED\_ROOT\_SIGNATURE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_versioned_root_signature_desc)：可以保存根签名描述的任何版本，旨在与下面所列的序列化/反序列化函数配合使用。
-   这些结构等效于版本 1.0 中所用的结构，并添加了描述符范围和根描述符的新标志字段：

    -   [**D3D12\_ROOT\_SIGNATURE\_DESC1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_signature_desc1)
    -   [**D3D12\_DESCRIPTOR\_RANGE1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_descriptor_range1)
    -   [**D3D12\_ROOT\_DESCRIPTOR\_TABLE1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_descriptor_table1)
    -   [**D3D12\_ROOT\_DESCRIPTOR1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_descriptor1)
    -   [**D3D12\_ROOT\_PARAMETER1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_parameter1)

### <a name="functions"></a>函数

此处所列的方法取代了原始的 [**D3D12SerializeRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-d3d12serializerootsignature) 和 [**D3D12CreateRootSignatureDeserializer**](/windows/desktop/api/d3d12/nf-d3d12-d3d12createrootsignaturedeserializer) 函数，因为它们可以在任何版本的根签名中使用。 序列化格式是传入 [**CreateRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createrootsignature) API 的格式。 如果编写的着色器包含根签名，则编译的着色器已包含序列化的根签名。

-   [**D3D12SerializeVersionedRootSignature**](/windows/desktop/api/d3d12/nf-d3d12-d3d12serializeversionedrootsignature)：如果应用程序遵循过程生成 [**D3D12\_VERSIONED\_ROOT\_SIGNATURE**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_versioned_root_signature_desc) 数据结构，则它必须使用此函数生成序列化格式。
-   [**D3D12CreateVersionedRootSignatureDeserializer**](/windows/desktop/api/d3d12/nf-d3d12-d3d12createversionedrootsignaturedeserializer)：生成一个可以通过 [**GetUnconvertedRootSignatureDesc**](/windows/desktop/api/d3d12/nf-d3d12-id3d12versionedrootsignaturedeserializer-getunconvertedrootsignaturedesc) 返回反序列化数据结构的接口。

### <a name="methods"></a>方法

创建 [**ID3D12VersionedRootSignatureDeserializer**](/windows/desktop/api/d3d12/nn-d3d12-id3d12versionedrootsignaturedeserializer) 接口的目的是反序列化根签名数据结构。

-   [**GetRootSignatureDescAtVersion**](/windows/desktop/api/d3d12/nf-d3d12-id3d12versionedrootsignaturedeserializer-getrootsignaturedescatversion)：将根签名描述结构转换为请求的版本。
-   [**GetUnconvertedRootSignatureDesc**](/windows/desktop/api/d3d12/nf-d3d12-id3d12versionedrootsignaturedeserializer-getunconvertedrootsignaturedesc)：返回指向 [**D3D12\_VERSIONED\_ROOT\_SIGNATURE\_DESC**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_versioned_root_signature_desc) 结构的指针。

### <a name="helper-structures"></a>帮助器结构

已添加帮助器结构用于帮助某些版本 1.1 结构的初始化。

-   CD3DX12\_DESCRIPTOR\_RANGE1
-   CD3DX12\_ROOT\_PARAMETER1
-   CD3DX12\_STATIC\_SAMPLER1
-   CD3DX12\_VERSIONED\_ROOT\_SIGNATURE\_DESC

请参阅 [D3D12 的帮助器结构和函数](helper-structures-and-functions-for-d3d12.md)。

## <a name="consequences-of-violating-static-ness-flags"></a>违反静态性标志的后果

上述描述符和数据标志（以及缺少特定标志时隐式使用的默认设置）定义应用程序对驱动程序做出的行为承诺。 如果应用程序违反承诺，则这是无效行为：结果是不确定的，并且不同的驱动程序和硬件可能会产生不同的结果。

调试层提供用于验证应用程序是否遵守其承诺的选项，包括在未设置任何标志的情况下，使用根签名版本 1.1 时附带的默认承诺。

## <a name="version-management"></a>版本管理

编译附加到着色器的根签名时，新式 HLSL 编译器默认会编译版本 1.1 的根签名，而旧式 HLSL 编译器仅支持 1.0。 请注意，在不支持根签名 1.1 的 OS 上无法运行 1.1 根签名。

可以使用 `/force_rootsig_ver <version>` 将通过着色器编译的根签名版本强制为特定版本。 如果在强制版本时能够保留所编译的根签名的行为（例如，删除根签名中只为优化目的提供服务，但不影响行为的不受支持标志），则强制版本将会成功。

例如，这样可以在生成应用程序时让应用程序将 1.1 根签名同时编译成 1.0 和 1.1，并在运行时根据 OS 支持级别选择适当的版本。 但是，这可以最大程度地提高空间效率，使应用程序能够独立于着色器编译根签名（尤其是需要多个版本时）。 即使着色器最初是使用附加的根签名编译的，也仍可以使用 `/verifyrootsignature` 编译器选项来保留根签名编译器验证与着色器兼容的优势。 以后在运行时，可以使用不包含根签名的着色器，并传递所需的根签名（也许是 OS 支持的适当版本）作为独立的参数，来创建 PSO。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[创建根签名](creating-a-root-signature.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> <dt>

[在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)
</dt> </dl>

 

 




