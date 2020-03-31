#　ShaderLab: Pass

[TOC]



Pass块会导致一个游戏对象的几何图形被渲染一次。

##　语法（Syntax）

```C++
Pass { [Name and Tags] [RenderSetup] }
```

基本的Pass命令包含一个呈现状态设置命令的列表。

##　Name and tags

Pass可以定义它的名称（Name)和任意数量的标记(Tags)。这里是指与Pass里驱动渲染引擎的通信用的字符串化的名称/值。

##　Render state set-up

通过设置图形硬件的各种状态，比如是否应该开启alpha blending，或者是否应该使用深度测试。
以下命令如下:

###　Cull

> Cull Back | Front | Off

Set polygon culling mode.

###　ZTest

> ZTest (Less | Greater | LEqual | GEqual | Equal | NotEqual | Always)
>
> // Less == 0, Greater == 1, LEqual == 2, GEqual == 3, Equal == 4, NotEqual == 5, Always == 6
>
> ZTest On | Off	// 这里On==2, Off==0

设置深度缓冲测试模式。

###　ZWrite

> ZWrite On | Off

设置深度缓冲写入模式。

### Offset

> Offset OffsetFactor, OffsetUnits

设置Z缓冲深度偏移量。有关详细信息，请参阅关于Cull and Depth页面的文档。

有关Cull、ZTest、ZWrite和偏移的详细信息，请参阅关于Cull and Depth的文档。

### 混合(Blend)

```C++
Blend sourceBlendMode destBlendMode
Blend sourceBlendMode destBlendMode, alphaSourceBlendMode alphaDestBlendMode
BlendOp colorOp
BlendOp colorOp, alphaOp
AlphaToMask On | Off
```

设置alpha blend、alpha操作和alpha-to-coverage模式。参见关于blend的文档，了解更多细节。

### ColorMask

> ColorMask RGB | A | 0 | any combination of R, G, B, A

设置彩色通道的书写蒙版。写ColorMask 0可以关闭到所有颜色通道的写入。默认模式是写入所有通道(RGBA)，但是对于某些特殊效果，您可能希望保留某些通道未修改，或者完全禁用颜色写。

当使用多个渲染目标(MRT)渲染时，可以通过添加索引(0-7)来为每个渲染目标设置不同的颜色掩码。例如，ColorMask RGB 3将使target #3只写入RGB通道。

### 传统固定功能材质命令(Legacy fixed-function Shader commands )

许多命令用于编写遗留的“固定函数样式”着色器。这被认为是不赞成的功能，因为写表面着色或着色程序可以让更多的灵活性。然而，对于非常简单的着色器，在固定函数样式中编写它们有时会更容易一些，因此这里提供了命令。注意，如果您不使用固定函数的着色器，那么所有下面的命令都将被忽略。

#### 固定功能照明和材料(Fixed-function Lighting and Material)
```C++
Lighting On | Off
Material { Material Block }
SeparateSpecular On | Off
Color Color-value
ColorMaterial AmbientAndDiffuse | Emission
```
所有这些控制固定功能的每个顶点照明:它们打开，设置材质颜色，开启高光部分，提供默认颜色(如果顶点照明关闭)，并控制网格顶点颜色对光线的影响。有关更多细节的材料，请参阅材料文档。

#### Fixed-function Fog 固定功能雾

> Fog { Fog Block }

设置固定功能雾参数。有关详细信息，请参阅文档。

#### Fixed-function AlphaTest 固定功能AlphaTest

> AlphaTest (Less | Greater | LEqual | GEqual | Equal | NotEqual | Always) CutoffValue

开启固定功能的alpha testing。有关alpha测试的文档了解更多细节。

#### Fixed-function Texture combiners 固定功能结构组合器
在呈现状态设置之后，使用SetTexture命令来指定一些纹理和它们的组合模式:

> SetTexture textureProperty { combine options }

## 参数化写法
```C++
Shader "Custom/Sprite" {
    Properties {
        // TIPS: モバイル環境でScaleとOffsetは使えないのでNoScaleOffsetで封印する
        [NoScaleOffset] _MainTex ("Base (RGB), Alpha (A)", 2D) = "white" {}

        [Enum(UnityEngine.Rendering.CullMode)]
        _Cull("Cull", Float) = 0                // Off

        [Enum(UnityEngine.Rendering.CompareFunction)]
        _ZTest("ZTest", Float) = 4              // LEqual

        [Enum(Off, 0, On, 1)]
        _ZWrite("ZWrite", Float) = 0            // Off

        [Enum(UnityEngine.Rendering.BlendMode)]
        _SrcFactor("Src Factor", Float) = 5     // SrcAlpha

        [Enum(UnityEngine.Rendering.BlendMode)]
        _DstFactor("Dst Factor", Float) = 10    // OneMinusSrcAlpha
    }

    SubShader {
        Tags {
            "Queue" = "Transparent"
            "IgnoreProjector" = "True"
            "RenderType" = "Transparent"
            "PreviewType" = "Plane"
        }

        Cull [_Cull]
        ZTest [_ZTest]
        ZWrite [_ZWrite]
        Blend [_SrcFactor][_DstFactor]

        Pass {
            CGPROGRAM
            #include "UnityCG.cginc"

            #pragma vertex vert
            #pragma fragment frag

            struct appdata_t {
                float4 vertex : POSITION;
                half2 texcoord : TEXCOORD0;
                fixed4 color : COLOR;
            };

            struct v2f {
                float4 pos : SV_POSITION;
                half2 uv : TEXCOORD0;
                fixed4 color : COLOR;
            };

            sampler2D _MainTex;

            v2f vert(appdata_t v) {
                v2f o;
                o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
                o.uv = v.texcoord;
                o.color = v.color;
                return o;
            }

            fixed4 frag(v2f i) : SV_Target {
                return tex2D(_MainTex, i.uv) * i.color;
            }
            ENDCG
        }
    }
}
```



## 详情(Details)

Shader通过多个方式与Unity的渲染管道进行交互;例如，一个pass可以表明，它应该只用于使用标签命令的延迟着色。某些Pass也可以在相同的游戏对象上多次执行;例如，在前向渲染中，“ForwardAdd” pass类型被多次执行，这取决于有多少光源影响了游戏对象。请参阅渲染管道上的文档，了解更多细节。