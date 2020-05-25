---
date: 2020-05-12 12:26:40
layout: post
title: UnityShader·入门
subtitle: shaderlab速查手册
description: shaderlab速查手册
image: /assets/img/graphics/shader/cover0.png
#optimized_image: /assets/img/opengl/spongebob.jpg
category: tech
published: true
tags:
  - graphics
  - unity
  - shader
author: llapuras13
---

### Shader概况

Unity支持三种Shader，[surface shader](https://docs.unity3d.com/Manual/SL-SurfaceShaders.html)，[fragment and vertex shader](https://docs.unity3d.com/Manual/SL-ShaderPrograms.html)，以及[fixed function shader](https://docs.unity3d.com/Manual/ShaderTut1.html)。

Unity所用的shader语言为Cg/HLSL，相关函数可在[Nvida](https://developer.download.nvidia.cn/cg/index_stdlib.html)查阅。

![](/assets/img/line.png)

### Shader代码结构

一个Shader文件大致长下面这样：

```c++
Shader "ShaderName"
{
 Properties
 {
  //...
 }
 
 SubShader
 {
  Tags { }

  CGPROGRAM
  // Cg / HLSL code of the shader
  // ...
  ENDCG
 }
}

```

- **SubShader：**SubShader中包含的是实际上会被GPU执行的指令。一个shader文件中可以包含多个SubShader，一般是针对不同平台的多个版本的shader代码，U3D会挨个尝试执行它们，直到找到能用的那一个为止。

- **Properties：**在[Properties](https://docs.unity3d.com/Manual/SL-Properties.html)中声明需要调控的变量，这些变量会先是在U3D的面板上

- **[Tags](https://docs.unity3d.com/Manual/SL-SubShaderTags.html):**用来告知引擎渲染顺序。不写也行，有默认模式。以下是比较常用的：
  - **[Queue](https://docs.unity3d.com/Manual/SL-SubShaderTags.html)：**决定物体渲染的顺序。比如先渲染透明物体还是先渲染不透明物体。
  - **[RenderType](https://docs.unity3d.com/Manual/SL-ShaderReplacement.html)：** 一个分类tag，需要结合方法``camera.SetReplacementShader(Shader, "RenderType")``使用。用来批量替换同RenderType物体的Shader。比如物体A的RenderType是``awsl``，B的RenderType是``keke``，用来替换的shader X中有RenderType是``awsl``的SubShader，但是没有``keke``的SubShader，那么这个替换方法执行后，A物体按照shader X渲染，物体B则不被渲染。
- **Pass：**一个SubShader中可以有多个Pass块，当某个SubShader被执行时，它的所有Pass都会被执行一遍。Pass中会包含一系列有关图形硬件的状态设置，常见的几种如下：
  - **[Cull](https://docs.unity3d.com/Manual/SL-CullAndDepth.html)：**剔除。Back/Front/Off
  - **[ZWrite](https://docs.unity3d.com/Manual/SL-CullAndDepth.html)：**决定pixel是否被写入depth buffer。On/Off
  - **[ZTest](https://docs.unity3d.com/Manual/SL-CullAndDepth.html)：**决定depth绘制顺序。默认为``LEqual``
  - 注意surface shader不能写在Pass块内，只能直接写在SubShader块中，surface shader会自动被编译为多Pass。

```cpp
 SubShader
 {
  Tags { }
  Pass {
    CGPROGRAM
    // Cg / HLSL code of the shader
    // ...
    ENDCG
  }
 }
```

- **[Fallback](https://docs.unity3d.com/Manual/SL-Fallback.html)：**定义在SubShader之后，如果没有一个SubShader能被执行，U3D就会执行Fallback的函数。

![](/assets/img/line.png)

### Surf vs. Vert/Frag

两者的主要区别在于vert/frag shader没有物理语义，包括albedo，gloss，specular都不会在这个层面呈现。而surface shader的输出结构（``SurfaceOutput``等）则包含了这些内容。

Surface shader**可以指定光照模型**，shader中通常会有一个叫surf的函数，在此计算输出颜色，再将其输入光照模型获得最后的结果。所有的Surface shader被编译时都会转化成vert/frag shader，即surface shader是Unity提供的一种简易shader写法。

Vertex/Fragment则没有内置光照模型的概念，更适合用在非现实渲染、2D图形和后期处理上。（当然也可以在里头写光照只是相对不大方便）

![](https://www.alanzucconi.com/wp-content/uploads/2015/06/Surface-shader.png)

![](https://www.alanzucconi.com/wp-content/uploads/2015/06/Vertex-and-Fragment-shader.png)

![](/assets/img/line.png)

### Surface Shader

声明surface function，将需要的数据输入shader并进行计算，最终打包以``SurfaceOutput``格式输出。``SurfaceOutput``会定义有关表面的各项属性，包括albedo、normal、emission、specularity等。

一个[surf shader](https://docs.unity3d.com/Manual/SL-SurfaceShaders.html)中必须包含的内容有二：
- **surfaceFunction：**这个func必须包含一个``void surf(Input IN, inout SurfaceOutput o)``。
- **lightModel：**可以使用built-in或[自己写](https://docs.unity3d.com/Manual/SL-SurfaceShaderLighting.html)的光照模型。常用的内置模型有``Standard``、``StandardSpecular ``、``Lambert ``、``BlinnPhong ``。例如``#pragma surface surf Lambert``声明了该shader是``surf``shader，光照模型用的是Lambert。


其他常用到但不必须的：
- **Custom modifier functions：****自定义修饰函数**，用来修改输入的顶点数据或最终输出的片段颜色。之前[积雪](/UnityShader-积雪/)那篇就用到了vert修改(vertex displacement)
- **Shadows and Tessellation：**给出控制shadow和tessellation的额外指令。[Tessellation](https://docs.unity3d.com/Manual/SL-SurfaceShaderTessellation.html)
- **Code generation options：**默认情况下surf shader会将所有的光照、阴影等场景信息纳入计算，但某些情况下可能可以跳过其中一些计算，比如不应用shadow、ambient、lightmap、fog等。这样可以加速shader的加载。

在``void surf``中，输入变量之一的``IN``是自定义的结构，通常会包含shader所需的变量信息。这个结构的中的**[变量命名](https://docs.unity3d.com/Manual/SL-SurfaceShaders.html)**是有要求的。
- 注意：[贴图坐标](https://docs.unity3d.com/Manual/SL-ShaderSemantics.html)的命名需要以``uv_``(或者``uv2_``)开头，后接变量名，比如``float2 uv_ModelTex : TEXCOORD0;``。一个模型的uv是可以有好几套的，冒号后的``TEXCOORDn``中的n即意为第n套uv。

![](/assets/img/line.png)

### Vert/Frag Shader

TBC...

![](/assets/img/line.png)

### 参考

1. [A gentle introduction to shaders in Unity3D](https://www.alanzucconi.com/2015/06/10/a-gentle-introduction-to-shaders-in-unity3d/)

2. [Vertex and fragment shaders in Unity3D](https://www.alanzucconi.com/2015/07/01/vertex-and-fragment-shaders-in-unity3d/)

3. [Surface shaders in Unity3D](https://www.alanzucconi.com/2015/06/17/surface-shaders-in-unity3d/)

4. [Unity·Writing Surface Shaders](https://docs.unity3d.com/Manual/SL-SurfaceShaders.html)

5. [Unity·Vertex and fragment shader examples](https://docs.unity3d.com/Manual/SL-VertexFragmentShaderExamples.html)