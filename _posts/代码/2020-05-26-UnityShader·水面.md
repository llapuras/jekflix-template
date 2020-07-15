---
date: 2020-05-26 12:26:40
layout: post
title: UnityShader·水面
subtitle: 
description: 
image: /assets/img/graphics/shader/edge_shine.gif
#optimized_image: /assets/img/opengl/spongebob.jpg
category: article
published: false
tags:
  - graphics
  - unity
  - shader
author: llapuras13
---

静水效果主要由波纹，折射和边缘效果这三个部分组成。

### 动态波纹

使用**vertex displacement**营造水面的波浪浮动。可控波浪幅度、高度和速度。

> CPU保留原始顶点数据，但是在将这些数据传向GPU前对顶点数据进行修改——这个技巧被称为**Vertex Displacement**。常用于制造水面、地形、爆炸和扭曲效果。简单来说就是在shaderlab的vertex function通过数学公式或者位移贴图修改顶点位置。

```cpp
v2f vert(appdata v)
{
	v2f o;
	v.vertex.y += sin(_Time.z * _Speed + (v.vertex.x * v.vertex.z * _Amount)) * _Height; //用sin函数制造波浪
	o.vertex = UnityObjectToClipPos(v.vertex);
	return o;
}
```
![](/assets/img/graphics/shader/water001.gif)

这一步的完整代码戳[这里](https://github.com/llapuras/ShaderLib/blob/master/Water/Water001.shader)。

### 水的透明度

Blend的默认状态是``Blend Off``，不设置Blend参数只会显示非透明颜色。修改Blend参数使贴图透明看起来更像水，注意**水体的颜色透明度不能为1**。

```cpp
Blend SrcAlpha OneMinusSrcAlpha
```

![](/assets/img/graphics/shader/water002.gif)

### 边缘浮沫

接下来添加接触物边缘效果。因为阳光照射的原因，深处的水颜色会较深，而近处的水颜色则较浅。如果有物体浸入水中，则物体边缘处的水颜色较浅，甚至会积累白色浮沫。

这里需要用到深度值(Depth)计算。**根据“深度”计算水应呈现的颜色。**

Camera可以生成深度贴图(depth texture)，深度贴图是一张G-Buffer贴图，储存了每个像素的深度值，常用于后期效果和自定义光照制作。这里需要介绍几个内置变量/方法：

- **_CameraDepthTexture：**Unity内置的变量。用于表示camera的深度贴图。
- **SAMPLE_DEPTH_TEXTURE_PROJ：**对深度贴图进行采样。该方法接受深度纹理和纹理坐标这两个参数。纹理坐标通常是由顶点着色器输出插值而得的屏幕坐标。
- **LinearEyeDepth：**通过纹理采样得到深度值后，这些深度值往往是非线性的，这种非线性来自于透视投影使用的裁剪矩阵。通过**LinearEyeDepth**把投影后的深度值变换到线性空间下。

```cpp
Pass
{
  //...
  sampler2D _CameraDepthTexture;//unity内置变量，无需在Properties中声明
  //...
  fixed4 frag(v2f i) : SV_Target
  {
      float4 depthSample = SAMPLE_DEPTH_TEXTURE_PROJ(_CameraDepthTexture, (i.screenPos));
	    float depth = LinearEyeDepth(depthSample);
	    float foamLine = 1 - saturate(_FoamThickness * (depth - i.screenPos.w));
	    half4 col = _Tint + foamLine * _EdgeColor * 0.5;
	    return col;
  }
}
```

![](/assets/img/graphics/shader/water003.gif)

>注意：要打开灯光的shadow效果，否则无法获取depth值。

这一步的完整代码戳[这里](https://github.com/llapuras/ShaderLib/blob/master/Water/Water002.shader)。

### 添加折射

水下物体因为折射和水面波动会呈现出扭曲的效果。这里使用GrabPass和一张扭曲贴图来制造这种折射效果。

**[GrabPass](https://docs.unity3d.com/Manual/SL-GrabPass.html)**是一个特殊的pass类型，它可以抓取屏幕上显示的内容然后将这个对象绘制到一个纹理中。在这里可以抓取物体成纹理，然后对其扭曲处理形成水下折射的图像。

![](/assets/img/graphics/shader/grabtex.gif)

```cpp
GrabPass{}
//...
v2f vert(appdata v)
{
  //...
  o.grabtex = ComputeGrabScreenPos(o.vertex);		
  return o;
}

fixed4 frag(v2f i) : SV_Target
{   
  float2 dism = UnpackNormal(tex2D(_DistortionMap, i.dismap + (_Time.x * 0.2)));
	float2 offset = dism * (_DistortionFactor * 10) * _GrabTexture_TexelSize.xy;
	i.grabtex.xy = offset+ i.grabtex.xy;;
	float4 dis = tex2Dproj(_GrabTexture, UNITY_PROJ_COORD(i.grabtex)) * _EdgeColor;
  //...
}
```

- **ComputeGrabScreenPos：**输入顶点坐标获得grab texture的正确贴图坐标。

- **UnpackNormal：**UnpackNormal是定义在UnityCG.cginc文件中的方法，这个文件中包含了一系列常用的CG变量以及方法。UnpackNormal接受一个fixed4的输入，并将其转换为所对应的法线值（fixed3）。在解包得到这个值之后，将其赋给输出的Normal，就可以参与到光线运算中完成接下来的渲染工作了。

- **_GrabTexture_TexelSize：**内置变量，grabtex的坐标位置。

添加distortion贴图后的效果：

![](/assets/img/graphics/shader/water004.gif)

这里用到的noise贴图：

![](/assets/img/graphics/shader/WaterNoise.jpg)

### 加上贴图

最后一步加上一张贴图，可以营造各种风格的水面。

![](/assets/img/graphics/shader/water005.gif)

这里例子用的贴图↓

![](/assets/img/graphics/shader/WaterTexture.png)

这一步的完整代码戳[这里](https://github.com/llapuras/ShaderLib/blob/master/Water/Water003.shader)。

### 其他

ps1：非常偶然地试了一张毫无关联的图，乘了一个没什么道理的参数，画面却意外还不错（感觉比水酷炫多了啊喂...戳[这里](https://github.com/llapuras/ShaderLib/blob/master/Water/edge_shine.shader)。

![](/assets/img/graphics/shader/edge_shine.gif)

ps2：颜色调成非透明白色可以获得blingbling的发光片...戳[这里](https://github.com/llapuras/ShaderLib/blob/master/Water/blingbling.shader)。

![](/assets/img/graphics/shader/blingbling.png)


![](/assets/img/line.png)

### 参考

1. [Unity3D中的深度纹理和法线纹理](https://www.jianshu.com/p/98aa7d5de675)

3. [Foam Line using Depth](https://lindenreid.wordpress.com/2017/12/15/simple-water-shader-in-unity/)

4. [Unity Stylized Water](https://www.patreon.com/posts/15121329)
