---
Description: 传递到 TraceRay 函数，以定义射线的原点、方向和范围。
ms.assetid: ''
title: RayDesc 结构
ms.topic: structure
ms.date: 05/31/2018
topic_type:
- APIRef
- kbSyntax
api_name:
- RAY_FLAG
api_type:
- NA
ms.openlocfilehash: 8987c825f84d411616ddd1701d715e388f9f4044
ms.sourcegitcommit: 1fbe7572f20938331e9c9bd6cccd098fa1c6054d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/27/2019
ms.locfileid: "66224230"
---
# <a name="raydesc-structure"></a>RayDesc 结构

传递到 [TraceRay](traceray-function.md) 函数，以定义射线的原点、方向和范围  。

## <a name="syntax"></a>语法


```
struct RayDesc
{
    float3 Origin;
    float  TMin;
    float3 Direction;
    float  TMax;
};

```



## <a name="fields"></a>字段

<dl> <dt>

<span id="Origin"></span><span id="origin"></span>原点 
</dt> <dd>

射线的原点。

</dd> <dt>

<span id="TMin"></span><span id="tmin"></span>**TMin**
</dt> <dd>

射线的最小范围。


</dd> <dt>

<span id="Direction"></span><span id="direction"></span>方向 
</dt> <dd>

射线的方向。


</dd> <dt>

<span id="TMax"></span><span id="tmax"></span>**TMax**
</dt> <dd>

射线的最大范围。


</dd>

## <a name="requirements"></a>要求



## <a name="see-also"></a>另请参阅

<dl> <dt>

[Direct3D 12 光线跟踪 HLSL 参考](direct3d-12-raytracing-hlsl-reference.md)
</dt> </dl>

 

 




