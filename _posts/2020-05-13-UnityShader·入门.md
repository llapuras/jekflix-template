---
date: 2020-05-12 12:26:40
layout: post
title: UnityShader·入门
subtitle: shaderlab速查手册
description: shaderlab速查手册
image: /assets/img/graphics/shader/rim.gif
#optimized_image: /assets/img/opengl/spongebob.jpg
category: tech
published: false
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
  - **Queue：**决定物体渲染的顺序。比如先渲染透明物体还是先渲染不透明物体。
  - **[RenderType](https://docs.unity3d.com/Manual/SL-ShaderReplacement.html)：** 一个分类tag，需要结合方法``camera.SetReplacementShader(Shader, "RenderType")``使用。用来批量替换同RenderType物体的Shader。比如物体A的RenderType是``awsl``，B的RenderType是``keke``，用来替换的shader X中有RenderType是``awsl``的SubShader，但是没有``keke``的SubShader，那么这个替换方法执行后，A物体按照shader X渲染，物体B则不被渲染。
- **Pass：**

- **[Fallback](https://docs.unity3d.com/Manual/SL-Fallback.html)：**定义在SubShader之后，如果没有一个SubShader能被执行，U3D就会执行Fallback的函数

### ToonBasic


