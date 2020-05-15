---
date: 2020-05-06 12:26:40
layout: post
title: Shader·积雪
subtitle: 
description: 
image: /assets/img/graphics/shader/rim.gif
#optimized_image: /assets/img/opengl/spongebob.jpg
category: tech
published: true
tags:
  - graphics
  - unity
  - shader
author: llapuras13
---

![](/assets/img/line.png)

### 雪的颜色

3D模型由一般由众多三角面构成，通过计算落雪方向与各个面的法线间点积可获得这两个向量间的夹角关系，然后设置阈值决定满足条件的面被染色。

```csharp
void surf(Input IN, inout SurfaceOutput o) {
	if (dot( o.Normal, normalize(_SnowDir.xyz)) >= -(_SnowSize-1)) { 
		o.Albedo = _SnowColor;
	}
}
```
![](/assets/img/graphics/shader/color.png)

![](/assets/img/line.png)

### 雪的厚度

通过**顶点移位**(Vertex Displacement)营造出积雪的「厚度」。在Unity中，可以通过surface shader中的自定义顶点函数修改传入shader的原始顶点数据。积雪厚度朝竖轴正方向增高，给顶点一个正竖直方向的增量即可。

```csharp

#pragma surface surf ToonRamp vertex:vert  
	
void vert(inout appdata_full v)	//custom vertex modifier fucntion
{
	if (dot(v.normal, normalize(_SnowDir.xyz)) >= _SnowSize )
	{
		v.vertex.xyz +=  * _SnowHeight;// scale vertices along up direction
	}
}

```

![](/assets/img/graphics/shader/height.png)

![](/assets/img/line.png)


### 光晕

现在的雪看起来一片惨白，缺少自然光下的颜色差异，最后给shader加上光晕效果。

```c++
void surf (Input IN, inout SurfaceOutput o) {
    //...
    half rim = 1.0 - saturate(dot(normalize(IN.viewDir), o.Normal));
    o.Emission = _RimColor.rgb * pow(rim, _RimPower);
}
```
![](/assets/img/graphics/shader/rim.gif)

![](/assets/img/line.png)

### Shader code

```


```

### 参考

1.[Quick Game Art Tips - Unity Toon Snow](https://www.patreon.com/posts/15944770)·Toon积雪效果<br>
2.[Shader Showcase Saturday #6: Dynamic Snow](https://www.alanzucconi.com/2018/08/18/shader-showcase-saturday-6/)·提到了用noise map制作平面雪地效果