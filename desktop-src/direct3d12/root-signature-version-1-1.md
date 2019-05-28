---
title: 根签名版本 1.1
description: 根签名版本 1.1 的目的是使应用程序能够描述符堆中的描述符不会更改或指向的数据描述符时指示驱动程序不会更改。
ms.assetid: 8FE42C1C-7F1D-4E70-A7EE-D5EC67237327
ms.topic: article
ms.date: 05/31/2018
ms.openlocfilehash: 238da415b4163671d7e79a17895c068b5b25bfaa
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224170"
---
# <a name="root-signature-version-11"></a>根签名版本 1.1

根签名版本 1.1 的目的是使应用程序能够描述符堆中的描述符不会更改或指向的数据描述符时指示驱动程序不会更改。 这样，驱动程序进行优化，可能会知道，描述符或它指向的内存静态的某个时间段内的选项。

-   [概述](#overview)
-   [静态和易失性标志](#static-and-volatile-flags)
    -   [描述符\_易失性](https://docs.microsoft.com/windows)
    -   [DATA\_VOLATILE](https://docs.microsoft.com/windows)
    -   [DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE](https://docs.microsoft.com/windows)
    -   [DATA\_STATIC](https://docs.microsoft.com/windows)
    -   [组合标志](#combining-flags)
    -   [标记摘要](#flag-summary)
-   [版本 1.1 API 摘要](#version-11-api-summary)
    -   [枚举](#enums)
    -   [结构](#helper-structures)
    -   [函数](#functions)
    -   [方法](#methods)
    -   [帮助程序结构](#helper-structures)
-   [违反静态性标志带来的后果](#consequences-of-violating-static-ness-flags)
-   [版本管理](#version-management)
-   [相关的主题](#related-topics)

## <a name="overview"></a>概述

根签名 1.0 版将允许进行内容描述符的堆和自由地更改应用程序时，命令将列出 / 捆绑到引用它们随时在指向的内存可能正在执行中是在 GPU 上。 经常使用，但是，应用程序不实际需要灵活地更改描述符或内存后对其进行引用的命令已记录。

应用程序通常是完全可以：

-   绑定描述符表或根描述符上命令列表或捆绑包之前设置描述符 （和可能它们指向的内存）。
-   请确保引用它们命令列表 /bundles 最后一次执行完之前不会更改这些描述符。
-   请确保数据描述符点不会更改为相同的完整持续时间。

或者，应用程序可能只能接受数据不会更改的时间更短时间。 特别是数据可能是静态的窗口中当前绑定 （描述符表或根描述符） 的根参数指向的数据的命令列表执行期间的时间。 换而言之，应用程序可能想要更新的时间段之间的某些数据在 GPU 时间线上执行执行它通过根参数，了解将其设置时它将静态设置。

如果描述符或数据描述符指向，不会更改，则驱动程序可能会执行的特定优化是特定于，硬件供应商和重要的是它们不会更改行为之外有可能提高性能。 保留有关应用程序意向尽可能知识不会对应用程序中将一种负担。

一个优化是许多驱动程序可以生成更高效的着色器的内存访问，如果他们知道有关描述符和数据的静态性的承诺可以使应用程序。 例如，驱动程序无法删除一定程度的间接性如果特定的硬件不敏感到根参数大小，将其转换到根描述符访问堆中的描述符。

开发人员使用 1.1 版的其他任务是使有关波动性和静态状态数据的承诺，只要有可能，以便驱动程序可以做出有意义的优化。 开发人员无需对其做出静态性承诺。

根签名版本 1.0 将继续不变，但重新编译根签名的应用程序将默认为根签名 1.1，现在 （通过用于强制版本 1.0，如果所需的选项）。

## <a name="static-and-volatile-flags"></a>静态和易失性标志

下列标志则要允许驱动程序选择的策略如何最好地处理单个根自变量，在设置完成后，还将嵌入到管道状态对象 (Pso) 在最初编译时-由于相同的假设的根签名的一部分根签名是 PSO 的一部分。

以下标志由应用程序设置，并将应用于描述符或数据。

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

### <a name="descriptorsvolatile"></a>描述符\_易失性

设置此标志，可以通过应用程序时的命令列表之外的任何时间更改指向的根描述符表描述符堆中的描述符/绑定描述符表的捆绑包已提交和未完成执行。 例如，记录命令列表并随后更改它引用描述符堆中的描述符*之前*提交命令列表的执行就有效。 这是根签名版本 1.0 的唯一受支持的行为。

如果描述符\_易失性标志*不*设置则是静态的描述符。 没有为此模式标志。 静态描述符是指指向根描述符表描述符堆中的描述符都已初始化的描述符表设置命令列表的时间 （在录制），捆绑包和描述符之前不能更改命令列表 /捆绑包已完成最后一次执行。 *对于根签名版本 1.1 中，静态描述符是默认的假设*，并且该应用程序指定描述符\_时所需的易失性标志。

绑定描述符表使用静态描述符，描述符必须准备开始 （而不是此捆绑包调用时），记录此捆绑包的时间并不变，除非此捆绑包已完成最后一次执行。 指向静态描述符的描述符表必须在捆绑录制过程中设置并不继承到捆绑包。 它是有效的命令列表用于描述符表已在绑定中设置并返回到命令列表的静态描述符。

描述符都是静态时另一个需要描述符的行为更改\_易失性标志设置。 带到任何缓冲区边界访问视图 （而不是 Texture1D/二维/三维/多维数据集视图） 无效，并且生成未定义的结果，包括可能的设备重置，而不是返回默认值来读取或删除写入。 删除应用程序依赖于带边界访问权限检查的硬件功能的目的是允许驱动程序选择的用于升级到根描述符访问的静态描述符访问，如果他们认为，更高效。 根描述符不支持的任何带边界检查。

如果访问说明符时，应用程序依赖于退出边界内存访问行为的安全，他们需要将访问这些说明符作为描述符的描述符范围标记\_易失性。

### <a name="datavolatile"></a>数据\_易失性

设置此标志，指向描述符的数据都可以更改由 CPU 时命令列表之外的任何时间/绑定描述符表的捆绑包已提交和未完成执行。 这是根签名版本 1.0 的唯一受支持的行为。

该标志可用于描述符范围标志和根描述符标志。

### <a name="datastaticwhilesetatexecute"></a>DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE

设置此标志，指向描述符的数据不能更改从上命令列表设置基础根描述符或描述符表是开始 / 捆绑在 GPU 时间线，对执行期间和结束时后续绘制/调度将不再引用的数据。

前一个根描述符或描述符表上已设置 GPU，此数据*可以*甚至通过相同的命令列表来更改 / 捆绑在一起。 虽然根描述符或指向它的描述符表仍设置上命令列表 / 捆绑包，只要对其进行引用的绘制/调度已完成，也可以更改数据。 但是，这样做需要描述符表重新绑定到命令列表下一次取消引用的根描述符或描述符表之前再次。 这允许要指向的根描述符或描述符表的数据发生更改的驱动程序。

之间数据的重要区别\_静态\_虽然\_设置\_在\_EXECUTE 和数据\_易失性是使用数据\_易失性驱动程序无法判断是否数据命令列表中的副本已更改为指向的描述符，而无需执行额外状态跟踪的数据。 因此，如果驱动程序，可以插入任何类型的数据预提取到其命令列表 （以使着色器访问为已知的数据更有效，例如），数据的命令\_静态\_虽然\_设置\_在\_EXECUTE 允许驱动程序知道它只需执行数据预提取通过设置目前[ **SetGraphicsRootDescriptorTable**](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setgraphicsrootdescriptortable)， [ **SetComputeRootDescriptorTable** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setcomputerootdescriptortable)或一种方法来设置常量缓冲区视图、 着色器资源视图或无序的访问视图。

对于捆绑包，数据是静态的而在设置执行承诺应用唯一捆绑包的每个执行。

该标志可用于描述符范围标志和根描述符标志。

### <a name="datastatic"></a>数据\_静态

如果设置此标志，已由根描述符或描述符表引用内存已设置命令列表的时间初始化指向描述符的数据 / 录制，期间的捆绑包和数据之前不能更改命令列表 / 捆绑包具有 最后一次执行完毕。

捆绑包，静态持续时间开始根描述符或描述符表设置的捆绑包，而不是调用的命令列表的录制录制期间。 此外，还必须设置捆绑包中并不会继承指向静态数据的描述符表。 它适用于要使用指向已在绑定中设置并返回到命令列表的静态数据的描述符表的命令列表。

该标志可用于描述符范围标志和根描述符标志。

### <a name="combining-flags"></a>组合标志

可以一次，但不支持数据标志根本由于取样器未指向数据的采样器描述符范围除外指定最多的数据标志之一。

如果缺少任何数据标志 SRV 和 CBV 描述符范围意味着，默认值为数据\_静态\_虽然\_设置\_在\_假定执行行为。 此默认值而不是数据选择原因\_静态是该数据\_静态\_时\_设置\_在\_EXECUTE 是更有可能是大多数情况下，安全默认设置时仍能产生比默认值为数据更好一些优化机会\_易失性。

没有针对 UAV 描述符范围数据标志则意味着默认值为数据\_假定易失性的行为，给定通常 Uav 写入到。

描述符\_可变*不能*与数据结合\_静态的但*可以*与其他数据标志结合使用。 描述符的原因\_易失性可以与数据一起\_静态\_虽然\_设置\_在\_EXECUTE 是易失性描述符仍要求描述符是在准备就绪命令列表 / 捆绑包执行和数据\_静态\_虽然\_设置\_在\_EXECUTE 是仅使有关中命令列表的子集的静态性的承诺 / 捆绑在一起执行。

### <a name="flag-summary"></a>标记摘要

下表总结了可能采用的标志组合。



|                                                                |                                                                                                                                                                                                                                                                                                                                                      |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **有效 D3D12\_描述符\_范围\_标志设置**             | **描述**                                                                                                                                                                                                                                                                                                                                      |
| 设置任何标志                                                   | 描述符都是静态的 （默认值）。 默认的数据的假设： 对于 SRV/CBV:数据\_静态\_虽然\_设置\_在\_EXECUTE，并为 UAV:数据\_易失性。 这些默认值用于 SRV/CBV 安全地将占满根签名的大多数的使用模式。                                                                                              |
| 数据\_静态                                                   | 描述符和数据是静态的。 这可最大化驱动程序优化的可能性。                                                                                                                                                                                                                                                          |
| 数据\_易失性                                                 | 描述符是静态的数据是易失性。                                                                                                                                                                                                                                                                                                     |
| DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE                          | 描述符是静态的数据是静态的而在设置执行。                                                                                                                                                                                                                                                                                      |
| 描述符\_易失性                                          | 描述符是易失性，以及有关数据进行默认假设： 对于 SRV/CBV:数据\_静态\_虽然\_设置\_在\_EXECUTE，并为 UAV:数据\_易失性。                                                                                                                                                                                              |
| 描述符\_可变\|数据\_易失性                        | 描述符和数据是易失性，等效于根签名 1.0。                                                                                                                                                                                                                                                                            |
| 描述符\_可变\|数据\_静态\_虽然\_设置\_在\_EXECUTE | 描述符是可变的但请注意，仍不允许使用它们在命令列表执行期间更改。 因此，有效组合期间执行 – 数据是静态时通过根描述符表集的其他声明基础描述符是为长于数据正在承诺为静态有效地静态的。 |



 



|                                                   |                                                                                                                                                                                                                   |
|---------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **有效 D3D12\_根\_描述符\_标志设置** | **描述**                                                                                                                                                                                                   |
| 设置任何标志                                      | 默认的数据的假设： 对于 SRV/CBV:数据\_静态\_虽然\_设置\_在\_EXECUTE，并为 UAV:数据\_易失性。 这些默认值用于 SRV/CBV 安全地将占满根签名的大多数的使用模式。 |
| 数据\_静态                                      | 数据是静态的驱动程序优化的最佳可能性。                                                                                                                                                       |
| DATA\_STATIC\_WHILE\_SET\_AT\_EXECUTE             | 数据是静态的而在设置执行。                                                                                                                                                                              |
| 数据\_易失性                                    | 等效于根签名 1.0。                                                                                                                                                                                 |



 

## <a name="version-11-api-summary"></a>版本 1.1 API 摘要

以下 API 调用启用版本 1.1。

### <a name="enums"></a>枚举

这些枚举包含的关键标志来指定描述符和数据的波动性。

-   [**D3D\_根\_签名\_版本**](/windows/desktop/api/D3D12/ne-d3d12-d3d_root_signature_version) ： 版本 id。
-   [**D3D12\_描述符\_范围\_标志**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_descriptor_range_flags) ： 标志确定如果描述符或数据不稳定的或静态的范围。
-   [**D3D12\_根\_描述符\_标志**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_root_descriptor_flags) ： 标志，以类似范围[ **D3D12\_描述符\_范围\_标志**](/windows/desktop/api/d3d12/ne-d3d12-d3d12_descriptor_range_flags)，只不过仅数据标记应用到根描述符。

### <a name="structures"></a>结构

更新的结构 （从版本 1.0） 包含对波动性/静态标记的引用。

-   [**D3D12\_功能\_数据\_根\_签名**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_root_signature) ： 传递到此结构[ **CheckFeatureSupport** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-checkfeaturesupport)到检查根签名版本 1.1 支持。
-   [**D3D12\_VERSIONED\_根\_签名\_DESC** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_versioned_root_signature_desc) ： 可以容纳根签名说明，任何版本，专为与序列化/反序列化一起使用下面列出的函数。
-   这些结构是版本 1.0 中，添加了新的描述符范围和根描述符的标志字段中所用等效的：

    -   [**D3D12\_根\_签名\_DESC1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_signature_desc1)
    -   [**D3D12\_描述符\_RANGE1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_descriptor_range1)
    -   [**D3D12\_根\_描述符\_TABLE1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_descriptor_table1)
    -   [**D3D12\_根\_1&GT;** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_descriptor1)
    -   [**D3D12\_ROOT\_PARAMETER1**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_root_parameter1)

### <a name="functions"></a>函数

此处列出的方法取代原始[ **D3D12SerializeRootSignature** ](/windows/desktop/api/D3D12/nf-d3d12-d3d12serializerootsignature)并[ **D3D12CreateRootSignatureDeserializer** ](/windows/desktop/api/D3D12/nf-d3d12-d3d12createrootsignaturedeserializer)函数，因为它们旨在在根签名的任何版本上运行。 序列化的形式是传递给[ **CreateRootSignature** ](/windows/desktop/api/D3D12/nf-d3d12-id3d12device-createrootsignature) API。 如果是具有根签名在其中编写着色器，编译着色器将已包含在其中的序列化的根签名。

-   [**D3D12SerializeVersionedRootSignature** ](/windows/desktop/api/d3d12/nf-d3d12-d3d12serializeversionedrootsignature) ： 如果应用程序虽然产生[ **D3D12\_VERSIONED\_根\_签名**](/windows/desktop/api/d3d12/ns-d3d12-d3d12_versioned_root_signature_desc)数据结构，它必须进行序列化的形式使用此函数。
-   [**D3D12CreateVersionedRootSignatureDeserializer** ](/windows/desktop/api/d3d12/nf-d3d12-d3d12createversionedrootsignaturedeserializer) ： 一个接口，可以通过返回反序列化的数据结构中，将生成[ **GetUnconvertedRootSignatureDesc** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12versionedrootsignaturedeserializer-getunconvertedrootsignaturedesc).

### <a name="methods"></a>方法

[ **ID3D12VersionedRootSignatureDeserializer** ](/windows/desktop/api/d3d12/nn-d3d12-id3d12versionedrootsignaturedeserializer)接口创建要反序列化根签名数据结构。

-   [**GetRootSignatureDescAtVersion** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12versionedrootsignaturedeserializer-getrootsignaturedescatversion) ： 将根签名说明结构到请求的版本。
-   [**GetUnconvertedRootSignatureDesc** ](/windows/desktop/api/d3d12/nf-d3d12-id3d12versionedrootsignaturedeserializer-getunconvertedrootsignaturedesc) ： 返回一个指向[ **D3D12\_VERSIONED\_根\_签名\_DESC** ](/windows/desktop/api/d3d12/ns-d3d12-d3d12_versioned_root_signature_desc)结构。

### <a name="helper-structures"></a>帮助程序结构

添加了帮助器结构，以帮助中的某些版本 1.1 中结构的初始化。

-   CD3DX12\_DESCRIPTOR\_RANGE1
-   CD3DX12\_ROOT\_PARAMETER1
-   CD3DX12\_STATIC\_SAMPLER1
-   CD3DX12\_VERSIONED\_根\_签名\_DESC

请参阅[帮助程序结构和函数对 D3D12](helper-structures-and-functions-for-d3d12.md)。

## <a name="consequences-of-violating-static-ness-flags"></a>违反静态性标志带来的后果

描述符和数据上面所述的标志 （以及特定标志缺少权限隐含的默认值） 定义完成的 promise 到有关如何将驱动程序应用程序的行为。 如果应用程序不符合该承诺，这是无效行为： 结果是不确定和跨不同的驱动程序和硬件可能会有所不同。

该调试层的选项可用于验证应用程序遵循其承诺，包括默认承诺的使用而无需设置任何标志的根签名版本 1.1 的。

## <a name="version-management"></a>版本管理

在编译时附加到着色器根签名，较新的 HLSL 编译器将默认为编译版本 1.1 中，根签名，而旧的 HLSL 编译器仅支持 1.0。 请注意，操作系统不支持根签名 1.1 的不适用于 1.1 根签名。

使用着色器编译的根签名版本可以强制为特定版本使用`/force_rootsig_ver <version>`。 强制版本将会成功，如果编译器可以保留根签名正在编译的强制的版本，例如通过删除根签名中的不支持的标志仅出于优化目的提供服务，但不是会影响行为的行为.

例如，这种方式应用程序可以时生成应用程序编译到 1.0 和 1.1 的 1.1 根签名并选择在运行时，具体取决于操作系统支持的级别的适当版本。 它将有效，但是，对于应用程序 （尤其是当多个版本所需） 单独编译根签名的空间最多，独立于着色器。 即使使用附加的根签名不最初编译着色器，可以通过使用保留的权益与着色器根签名兼容的编译器验证`/verifyrootsignature`编译器选项。 稍后在运行时，Pso 可以使用创建着色器，其中不包含根签名的值，同时传递所需的根签名 （可能是适当版本支持的操作系统） 作为独立的形参。

## <a name="related-topics"></a>相关主题

<dl> <dt>

[创建根签名](creating-a-root-signature.md)
</dt> <dt>

[根签名](root-signatures.md)
</dt> <dt>

[在 HLSL 中指定根签名](specifying-root-signatures-in-hlsl.md)
</dt> </dl>

 

 




