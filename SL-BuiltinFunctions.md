#　内置的着色器辅助函数(Built-in shader helper functions)

[TOC]

Unity有许多内置的实用功能，可以让编写着色器变得更简单和简单。

## 函数（UnityCG.cginc）

查看Built-in shader include files，用于对着色器的概述，其中包括提供Unity的文件。

## 顶点转换函数（Unitycg.cginc）

| 函数                                    | 描述                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| float4 UnityObjectToClipPos(float3 pos) | 将一个点从物体空间转换为在齐次坐标下的摄像机的剪辑空间。这相当于mul(**UNITY_MATRIX_MVP, float4(pos, 1.0)**)，应该在合适的地方 |
| float3 UnityObjectToViewPos(float3 pos) | 将一个点从对象空间转换为视图空间。这相当于mul(**UNITY_MATRIX_MV, float4(pos, 1.0)**))。xyz，应该在合适的地方 |

## 通用辅助函数（在unitycg.cginc中）

| 函数                                                       | 描述                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| float3 WorldSpaceViewDir (float4 v)                        | 给定的对象空间顶点位置，返回到相机的世界空间方向（not normalized） |
| float3 ObjSpaceViewDir (float4 v)                          | 给定的对象空间顶点位置，返回到相机的对象空间方向(not normalized) |
| float2 ParallaxOffset (half h, half height, half3 viewDir) | 计算视差贴图的UV偏移量                                       |
| fixed Luminance (fixed3 c)                                 | 把颜色转换成亮度(灰度)                                       |
| fixed3 DecodeLightmap (fixed4 color)                       | 从unity的lightmap(RGBM或dLDR根据平台)来解码颜色              |
| float4 EncodeFloatRGBA (float v)                           | 编码0.1)范围float到RGBA颜色，用于低精度渲染目标的存储        |
| float DecodeFloatRGBA (float4 enc)                         | 将RGBA颜色转换为一个浮点数                                   |
| float2 EncodeFloatRG (float v)                             | 编码0.1)范围float到float2                                    |
| float DecodeFloatRG (float2 enc)                           | 解码先前编码的RG浮点数                                       |
| float2 EncodeViewNormalStereo (float3 n)                   | 将视图空间的法线值编码为0。1范围                             |
| float3 DecodeViewNormalStereo (float4 enc4)                | 解码从enc4.xy中法线空间                                      |

## 前向渲染辅助函数（Unitycg.cginc）

这些函数只有在使用前向渲染来表达时才有用(往前向渲染里加入代码或前向渲染的参数里添加传递参数  )

| 函数                                 | 描述                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| float3 WorldSpaceLightDir (float4 v) | 计算世界空间中朝向光源的方向(未单位话)，给定物体空间顶点位置 |
| float3 ObjSpaceLightDir (float4 v)   | 计算对象空间中朝向光源的方向(未单位话)，给定对象空间顶点位置 |
| float3 Shade4PointLights (...)       | 从四个点光源计算照明，将光数据紧密地压缩到矢量中。前向渲染利用这个来计算每个顶点的照明 |

## 屏幕空间辅助函数（Unitycg.cginc）

下面的函数是用来计算用于采样屏幕空间纹理的坐标的辅助工具。这些函数返回float4，这个值是到采样纹理的最终坐标，它可以通过透视除法(例如xy/w)来再计算。

这些函数还可以处理渲染纹理坐标的`平台差异`。

| 函数                                         | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| float4 ComputeScreenPos (float4 clipPos)     | 计算纹理坐标用于做一个屏幕空间的纹理采样。输入是裁剪空间位置 |
| float4 ComputeGrabScreenPos (float4 clipPos) | 计算纹理坐标，以对一个GrabPass材质进行采样。输入是裁剪空间位置 |

## Vertex-lit的辅助函数 (UnithCG.cginc)

当使用顶点的光照着色器(Vertex Pass)时，这些函数才有用。

| 函数                                                    | 描述                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| float3 ShadeVertexLights (float4 vertex, float3 normal) | 根据给定的物体空间位置和法线的情况，计算每个顶点灯光和环境的光照 |

