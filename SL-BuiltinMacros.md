# 预定义的着色器预处理宏 (Predefined Shader preprocessor macros)

[TOC]

当Unity编译着色器程序时，使用了以下预处理宏

## 目标平台

| 宏                  | 目标平台                                          |
| ------------------- | ------------------------------------------------- |
| SHADER_API_D3D11    | Direct3D 11                                       |
| SHADER_API_GLCORE   | Desktop OpenGL “core” (GL 3/4)                    |
| SHADER_API_GLES     | OpenGL ES 2.0                                     |
| SHADER_API_GLES3    | OpenGL ES 3.0/3.1                                 |
| SHADER_API_METAL    | **iOS**/Mac Metal                                 |
| SHADER_API_VULKAN   | Vulkan                                            |
| SHADER_API_D3D11_9X | Direct3D 11 “feature level 9.x” target for UWP    |
| SHADER_API_PS4      | PlayStation 4. `SHADER_API_PSSL` is also defined. |
| SHADER_API_XBOXONE  | **XBox One**                                      |
| SHADER_API_PSP2     | PlayStation Vita                                  |

为所有通用移动平台(GLES、GLES3、Metal、PSP2)定义了SHADER_API_MOBILE。

另外，当目标着色语言是GLSL时(对于面向目标平台的gles来说总是正确的)，添加定义SHADER_TARGET_GLSL。

## 着色器目标模型 (Shader Target Model)

SHADER_TARGET被定义为与着色器目标编译模型匹配的数值(也就是匹配目标指令指令)。例如，当编译成ShaderMode 3.0时，SHADER_TARGET是30。您可以在着色器代码中使用它来进行条件检查。例如:

```C++
#if SHADER_TARGET < 30
    // less than Shader model 3.0:
    // very limited Shader capabilities, do some approximation
#else
    // decent capabilities, do a better thing
#endif
```

##Unity版本

UNITY_VERSION包含Unity版本的数值。例如，UNITY_VERSION是501，用于Unity 5.0.1。如果您需要编写使用不同内置着色器功能的着色器，那么可以将其用于版本比较。例如，如果UNITY_VERSION=500预处理器检查只通过版本5.0.0或更高版本。

## 着色器阶段被编译

预处理器、宏指令 SHADER_STAGE_VERTEX, SHADER_STAGE_FRAGMENT, SHADER_STAGE_DOMAIN, SHADER_STAGE_HULL, SHADER_STAGE_GEOMETRY, SHADER_STAGE_COMPUTE 在编译每个着色器阶段时都被执行。通常，当在像素着色器和计算着色器之间共享着色的代码时，它们是很有用的，以处理一些需要稍微改变一些事情的情况。

##平台差异助手

直接使用这些平台宏是不鼓励的，因为它们并不总是对代码的未来有所帮助。例如，如果您正在编写一个用于检查D3D9的着色器，那么您可能希望确保在将来，该检查被扩展为包含D3D11。相反，Unity定义了几个助手宏(HLSLSupport.cginc):

| 宏                                                           | 用法                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| UNITY_BRANCH                                                 | 在条件语句之前加上这个，告诉编译器，这个应该被编译成一个实际的分支。在HLSL平台上扩展到分支 |
| UNITY_FLATTEN                                                | 在条件语句之前添加这个语句，告诉编译器应该将其代码展开，以避免实际的分支指令。在HLSL平台上扩展为flatten |
| UNITY_NO_SCREENSPACE_SHADOWS                                 | 在不使用级联屏幕阴影映射(移动平台)的平台上定义               |
| UNITY_NO_LINEAR_COLORSPACE                                   | 在不支持线性颜色空间(移动平台)的平台上定义                   |
| UNITY_NO_RGBM                                                | 在没有使用lightmap的RGBM压缩的平台上定义(移动平台)           |
| UNITY_NO_DXT5nm                                              | 在不使用DXT5nm标准映射压缩(移动平台)的平台上定义             |
| UNITY_FRAMEBUFFER_FETCH_AVAILABLE                            | 在“framebuffer颜色取回”功能的平台上定义(通常是iOS平台——OpenGL ES 2.0、3.0和Metal) |
| UNITY_USE_RGBA_FOR_POINT_SHADOWS                             | 在点光阴影映射使用RGBA纹理和编码深度(其他平台使用单通道浮点纹理)的平台上定义 |
| UNITY_ATTEN_CHANNEL                                          | 定义光衰减纹理的哪个通道包含数据;在每个像素的照明代码中使用。定义为“r”或“a” |
| UNITY_HALF_TEXEL_OFFSET                                      | 在平台上定义，需要在将纹理映射到像素(例如Direct3D 9)的时候，需要半个纹素的偏移量调整 |
| UNITY_UV_STARTS_AT_TOP                                       | 总是定义为1或0的值。1的值在材质V坐标为0的平台上，在纹理的“顶部”。类似于direct3d的平台使用值1;类似于OpenGL平台使用值为0 |
| UNITY_MIGHT_NOT_HAVE_DEPTH_Texture                           | 如果一个平台可以通过手动渲染纹理来模拟阴影贴图或深度纹理     |
| UNITY_PROJ_COORD(a)                                          | 给定一个4个分量的向量，它返回一个纹理坐标，适用于到投影纹理的读取。在大多数平台上，它直接返回给定的值 |
| UNITY_NEAR_CLIP_VALUE                                        | 定义为接近剪切面的值。类似于direct3d的平台使用0.0，而OpenGL平台的平台使用-1.0 |
| UNITY_VPOS_TYPE                                              | 定义像素位置输入所需的数据类型(VPOS):在D3D9上是float2，在其它平台是float4 |
| UNITY_CAN_COMPILE_TESSELLATION                               | 当着色器编译器“理解”HLSL的tessellation shader时，才有定义(目前只有D3D11) |
| UNITY_INITIALIZE_OUTPUT(type,name)                           | 初始化给定类型的变量名为0                                    |
| UNITY_COMPILER_HLSL, UNITY_COMPILER_HLSL2GLSL, UNITY_COMPILER_CG | 表明哪个着色编译器被用来编译着色器——分别是:微软的HLSL，HLSL到GLSL转换器，以及NVIDIA的Cg。请参阅有关着色语言的文档，了解更多细节。如果您在编译器中遇到非常特殊的着色处理差异，并且希望为每个编译器编写不同的代码，那么就使用这个方法 |
|                                                              |                                                              |
|                                                              |                                                              |

UNITY_REVERSED_Z ----- 在plaftorms上使用反向Z缓冲区定义。存储的Z值在范围1中。0 . . 1

## 阴影映射宏(Shadow mapping macros)

根据平台的不同，声明和采样影子映射可能会非常不同。Unity有几个宏来帮助这个:

| 宏                               | 用法                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| UNITY_DECLARE_SHADOWMAP(tex)     | 用名称“tex”来声明一个阴影贴图纹理变量                        |
| UNITY_SAMPLE_SHADOW(tex,uv)      | 在给定的“uv”坐标下，采样阴影贴图纹理“tex”(XY成分是纹理位置，Z组件是可以用来比较的深度值)。一个float，在[0,1]范围内 |
| UNITY_SAMPLE_SHADOW_PROJ(tex,uv) | 类似于上面，只对投射阴影图读取。“uv”是一个float4，所有其他的部件都被w除。 |

**NOTE:** Not all graphics cards support shadowmaps. Use [SystemInfo.SupportsRenderTextureFormat](../ScriptReference/SystemInfo.SupportsRenderTextureFormat.html) to check for support.

## 常量缓冲宏(Constant buffer macros)

所有Direct3D 11 组的所有Shader变量纳入“constant缓冲池”。Unity的built-in变量已经是成组化了，已经应用在你写的shader里了。把这些变量放在不同的有赖于高频更新的constant buffer里可以带来更多的优化。

使用`CBUFFER_START`(name)和`CBUFFER_END`宏:

```C
CBUFFER_START(MyRarelyUpdatedVariables)
    float4 _SomeGlobalValue;
CBUFFER_END
```



## 纹理/采样器的宏声明(Texture/Sampler declaration macros)

通常你会在着色器代码中使用texture2D来声明纹理和采样对。但是在某些平台上(比如DX11)，纹理和采样器是独立的游戏对象，并且最大可能的采样计数是非常有限的。Unity有一些宏可以在没有采样器的情况下声明纹理，并使用来自另一个纹理的采样器来采样纹理。如果你最终进入了采样限制，你就知道你的一些纹理实际上可以共享一个采样器(Samplers定义了纹理过滤和包装模式)。

| 宏                                               | 用法                                                        |
| ------------------------------------------------ | ----------------------------------------------------------- |
| UNITY_DECLARE_TEX2D(name)                        | 声明一个纹理和采样器对                                      |
| UNITY_DECLARE_TEX2D_NOSAMPLER(name)              | 一个没有采样器的纹理                                        |
| UNITY_DECLARE_TEX2DARRAY(name)                   | 声明一个纹理数组采样器变量                                  |
| UNITY_SAMPLE_TEX2D(name,uv)                      | 从纹理和取样对中取样，使用给定的纹理坐标                    |
| UNITY_SAMPLE_TEX2D_SAMPLER( name,samplername,uv) | 从纹理(名称)中取样，使用来自另一个纹理的采样器(samplername) |
| UNITY_SAMPLE_TEX2DARRAY(name,uv)                 | 从纹理数组中获得一个浮动的UV;坐标的z分量是数组元素索引      |
| UNITY_SAMPLE_TEX2DARRAY_LOD(name,uv,lod)         | 从纹理数组中获得一个显式的mipmap级别                        |



## 表面材质通道指标(Surface Shader pass indicators)

当表面着色器被编译时，它们会生成大量的代码来进行各种各样的照明。在编译每个Pass时，以下宏定义如下:

| 宏                      | 用法                                         |
| ----------------------- | -------------------------------------------- |
| UNITY_PASS_FORWARDBASE  | 前向渲染基础通道（主方向灯、光照贴图、SH）   |
| UNITY_PASS_FORWARDADD   | 前向渲染附加通道（每个通道一个灯）           |
| UNITY_PASS_DEFERRED     | 延时着色通道 (渲染 g buffer)                 |
| UNITY_PASS_SHADOWCASTER | 阴影投射渲染通道                             |
| UNITY_PASS_PREPASSBASE  | 遗留的延时光照基础通道（渲染法线和高光指数） |
| UNITY_PASS_PREPASSFINAL | 遗留的延时光照最终通道（使用光照和纹理）     |

## 禁止自动更新(Disable Auto-Upgrade)

UNITY_SHADER_NO_UPGRADE:  允许你禁止Unity自动的更新或修改你的shader文件

