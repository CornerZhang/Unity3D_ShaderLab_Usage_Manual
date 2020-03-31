# 内置Shader的变量（Built-in shader variables）

[TOC]

Unity提供了非常有用的内置全局变量，比如：当前Object的变换矩阵、光照参数、当前时间等。内置变量与其他变量一样能在Shader程序中使用，唯一不同的是，不需要再重新定义它们，这些变量都定义在UnityShaderVariables.cginc(unity安装目录下查找)中。

## 变换 (Transformations)

以下所有Matrice是float4x4类型

| Name               | Value                                       |
| ------------------ | ------------------------------------------- |
| UNITY_MATRIX_MVP   | 当前 模型x视图x投影矩阵                     |
| UNITY_MATRIX_MV    | 当前 模型x视图矩阵                          |
| UNITY_MATRIX_V     | 当前 视图矩阵                               |
| UNITY_MATRIX_P     | 当前 投影矩阵                               |
| UNITY_MATRIX_VP    | 当前 视图x投影矩阵                          |
| UNITY_MATRIX_T_MV  | 模型x视图矩阵的转置                         |
| UNITY_MATRIX_IT_MV | 模型x视图矩阵的逆转置                       |
| _Object2World      | 模型世界坐标矩阵（本地坐标-->世界坐标）     |
| _World2Object      | 模型世界坐标矩阵的逆（世界坐标-->本地坐标） |

## 摄像机和屏幕(Camera and Screen)

这些变量是对渲染相机而言的。比如：在shadowmap渲染期间，会涉及到相机组件的变量，而不是用于阴影投影的虚拟相机。

| Name                           | Type     | Value                                                        |
| ------------------------------ | -------- | ------------------------------------------------------------ |
| _WorldSpaceCameraPos           | float3   | 相机的世界空间位置                                           |
| _ProjectionParams              | float4   | x是1.0(或-1.0，如果当前渲染的是一个翻转的投影矩阵)，y是摄像机的近平面，z是摄像机的远平面，w是1/far平面 |
| _ScreenParams                  | float4   | x是相机的渲染目标宽度以像素为单位，y是相机的渲染目标高度为像素，z为1.0+1.0/宽度，w为1.0+1.0/高度 |
| _ZBufferParams                 | float4   | 用于线性化Z缓冲区的值。x是(1-far/near), `y` 是(far/near), `z`是(x/far) ， `w`是(y/far) |
| unity_OrthoParams              | float4   | x是正交相机的宽度，y是正相机的高度，z是未使用的，当相机正交的时候，w是1.0，当透视时是0.0 |
| unity_CameraProjection         | float4x4 | 相机投影矩阵                                                 |
| unity_CameraInvProjection      | float4x4 | 相机投影矩阵的逆                                             |
| unity_CameraWorldClipPlanes[6] | float4   | 摄像机平截头体平面世界空间方程，按这个顺序:左、右、下、上、近、远 |

## 时间(Time)

| Name            | Type   | Value                                                        |
| --------------- | ------ | ------------------------------------------------------------ |
| _Time           | float4 | 来自游戏场景加载时(t/20，t，t 2，t 3)，用在着色器里面的动画的东东 |
| _SinTime        | float4 | 正弦的时间: (t / 8，t/4，t/2,   t)                           |
| _CosTime        | float4 | 余弦的时间: (t / 8，t/4，t/2，t)                             |
| unity_DeltaTime | float4 | 时间差:(dt，1/dt，smoothDt，1/smoothDt)                      |

## 光照(Lighting)

根据使用[Rendering Path](https://docs.unity3d.com/Manual/RenderingPaths.html)的不同，以及Shader中的Pass内的LightMode[标签（Pass Tag）](https://docs.unity3d.com/Manual/SL-PassTags.html)的不同，光照相关参数通过不同的方式被传递给Shader

| Name                                    | Type        | Value                                                        |
| --------------------------------------- | ----------- | ------------------------------------------------------------ |
| _LightColor0 (在Lighting.cginc中声明)   | fixed4      | 灯光颜色                                                     |
| _WorldSpaceLightPos0                    | float4      | 分为两种情况：平行光时：float4的xyzw最后一个w分量为0，表示这是一个平行光，xyz代表其光源方向；点光源时：最后一个分量为1，表示这是一个点光源，xyz代表点光源在世界坐标系内的位置 |
| _LightMatrix0 (在AutoLight.cginc中声明) | float4x4    | 这是一个由世界空间变换至光源空间（即光源的局部坐标系）的矩阵。可用于计算光照衰减。变换原理可以参考《3D Math Primer for Graphics and Game Development》或大学线性代数。在Shader中使用mul(param1 , param1)函数进行变换 |
| unity_4LightPosX0                       | float4      | 仅在ForwardBase pass 有效，代表的是世界空间内的前四个非重要点光源的位置。有人可能会问为什么明明只有三个参数，却能代表四个位置，是因为如下规则：unity_4LightPosX0 的四个分量xyzw分别代表的是四个点光源的x坐标，也就是说这个值的xyzw值都代表着四个点光源各自的x坐标，另外两个参数同理。这样就得到四个点光源的位置信息 |
| unity_4LightPosY0                       | float4      | (同上)                                                       |
| unity_4LightPosZ0                       | float4      | (同上)                                                       |
| unity_4LightAtten0                      | float4      | 仅在ForwardBase pass 有效，代表着前四个非重要点光源的衰减系数。注：当没有数据时的值是1而不是0 |
| unity_LightColor                        | half4[4]    | 仅在ForwardBase pass 有效，这是一个half4类型的4分量数组，里面的4个参数代表上述的四个点光源的颜色信息 |
| unity_WorldToShadow                     | float4x4[4] | World-to-shadow矩阵数组，一个矩阵对应一个spotlight，或是4个全对应级联的平行光照 |

通常用于光照pass着色器中的延迟着色和延迟光照（它们被声明在UnityDeferredLibrary.cginc)

| Name                | Type        | Value                                                        |
| ------------------- | ----------- | ------------------------------------------------------------ |
| _LightColor         | float4      | 灯光颜色                                                     |
| _LightMatrix0       | float4x4    | 这是一个由世界空间变换至光源空间（即光源的局部坐标系）的矩阵。可用于计算光照衰减 |
| unity_WorldToShadow | float4x4[4] | World-to-shadow矩阵数组，一个矩阵对应一个spotlight，或是4个全对应级联的平行光照 |

**球谐系数（Spherical harmonics coefficients：用于环境与光照探头（以后有机会再做讲解））**适用于ForwardBase, PrePassFinal和Deferred光照模式的Pass。包含了由世界空间法线计算而得的第三阶球谐函数(3rd order SH) [参考 UnityCG.cginc文件中的ShaderSH9函数；此部分内容需要一定的数学知识，球谐相关涉及傅里叶变换，3rd order SH即third order Spherical Harmonics，球谐函数相关资料]。这些参数都是half4类型，具有和unity_SHAr相似的命名。

**顶点光照渲染(Vertex-lit rendering)**，LightMode为Vertex及Vertex相关的Pass。

顶点光照模式可高达8个光源，并且总是从最亮的光源排序（也就是说第一个元素就是最亮的光源的数据）。如果你想要利用两个光源来对物体进行渲染，则可以直接使用数组内的前两个数据，如果实际光源数量不满8个，则空闲的数据都为0，即代表着颜色为黑色。

| Name                | Type      | Value                                                        |
| ------------------- | --------- | ------------------------------------------------------------ |
| unity_LightColor    | half4[8]  | 灯光颜色                                                     |
| unity_LightPosition | float4[8] | 视图空间内的灯光位置。<br>w分量为0则代表是平行光，前三分量为光源方向的负值<br>w分量为1则代表为点光源或聚光灯，前三分量代表光源所有的位置 |
| unity_LightAtten    | half4[8]  | 光源衰减系数，非聚光灯：<br>X = cos(spotAngle/2)或-1<br/>Y = 1/cos(spotAngle/4)或1<br>Z ,W表示光线按距离以二次函数衰减（暂时理解如是，有待实验） |
| unity_SpotDirection | float4[8] | 聚光灯在视点空间内的位置， (0,0,1,0) 表示非聚光灯            |



## 雾和环境光(Fog and Ambient)

| Name                     | Type   | Value                                                        |
| ------------------------ | ------ | ------------------------------------------------------------ |
| unity_AmbientSky         | fixed4 | 在梯度环境照明(Gradient ambient lighting)时，天空的环境光颜色 |
| unity_AmbientEquator     | fixed4 | 在梯度环境照明时，赤道环境光颜色                             |
| unity_AmbientGround      | fixed4 | 在梯度环境照明时，地面环境光颜色                             |
| UNITY_LIGHTMODEL_AMBIENT | fixed4 | 环境照明颜色(在梯度环境中的天空颜色)。遗留的变量             |
| unity_FogColor           | fixed4 | 雾的颜色                                                     |
| unity_FogParams          | float4 | 雾计算的参数:(密度/sqrt(ln(2))、密度/ln(2)、-1/(结束开始)、结束/(结束开始))。<br>x用于Exp2 雾效模式(Fog Mode)<br>y用于Exp模式<br>z和w用于Linear模式 |



## 杂项(Various)

| Name          | Type   | Value                                                        |
| ------------- | ------ | ------------------------------------------------------------ |
| unity_LODFade | float4 | 当使用LODGroup时，细节级别会降低。x是淡色(0.1)，y被逐渐量化为16个级别，z和w未被使用 |

