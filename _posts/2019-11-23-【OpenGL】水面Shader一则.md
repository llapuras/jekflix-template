---
date: 2019-11-23 12:26:40
layout: post
title: 【OpenGL】水面Shader一则
subtitle: 
description: 
optimized_image: /assets/img/opengl/waterwave.gif
image: /assets/img/opengl/waterwave.gif
category: tech
tags:
  - opengl
  - graphics
author: llapuras13
---

出于对水的执著，我寻思着要加点水面效果到作业里。

主要参考了Shadertoy上这位大佬的水面，还用上了光追的样子，有空研究一下。[Water Waves Drag](https://www.shadertoy.com/view/4dBcRD)

Shadertoy和OpenGL用的都是GLSL着色器语言，只有少许差异。

在原有项目的基础上添加了水面vertex和frag shader文件。

碎碎念，直接搬运的话效果如下图——完全一样!

![](../assets/img/opengl/water_opgl.jpg)

但是！就是张贴图而已，只是挑了个特殊角度截了个看起来很像样的图(╯·_·)╯︵┻━┻

修改后把水面效果应用到一张Plane Mesh上，但是最终还是没能加上高度差变化，Fake Water实锤。经过了一点魔改，水波系数过大的时候看起来会莫名像是云朵在水里的倒影emmmm

![](../assets/img/opengl/waterwave.gif)

俯瞰的视觉效果其实还凑合啦。

