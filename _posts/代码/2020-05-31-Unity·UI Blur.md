---
date: 2020-05-31 12:26:40
layout: post
title: UnityShader·UI Blur
subtitle: 
description: 
image: /assets/img/graphics/shader/blur_009.gif
#optimized_image: /assets/img/opengl/spongebob.jpg
category: tech
published: false
tags:
  - unity
  - ui
author: llapuras13
---

### Blur的原理

模糊(Blur)的原理很简单。对于一张图片中的每个像素点，对它和其周围像素的颜色求和再取均值，设为该像素点经过blur处理后的颜色，可以达到类似毛玻璃的模糊效果。

在shaderlab中的实现思路为，使用**GrabPass**获取场景图像，根据**模糊半径**对范围内每个像素进行上述处理。（ps：GrabPass对性能消耗较大，在移动端表现可能不佳）

模糊效果有很多种算法，这篇文里只实现了平均权重的普通模糊。

### 初始模板

首先，做好准备工作，先准备一个可以改变颜色的GrabPass空模板。效果如下。（看起来像是半透明的板子，其实上面显示的是GrabPass抓取的场景纹理）

![](/assets/img/graphics/shader/blur_000.png)

```cpp

  Tags{ "Queue" = "Transparent"}
  GrabPass {}
  //...

	//GrabPass相关变量
	sampler2D _GrabTexture;
	float4 _GrabTexture_TexelSize;

  //..
  
  half4 frag(v2f i) : SV_TARGET
  {
    float4 result = tex2Dproj(_GrabTexture, i.screenPos) * _Color;			
    return result;
  }
```

注意[Queue](https://docs.unity3d.com/Manual/SL-SubShaderTags.html)需要设置为``Transparent``，这样透明通道相关渲染会放在非透明物体渲染后。

前一篇[UnityShader·水面](../UnityShader-水面/)中也用到了GrabPass，这里一样——声明``_GrabTexture``和``_GrabTexture_TexelSize``。

这一步的代码戳[这里](https://github.com/llapuras/ShaderLib/blob/master/UIBlur/BlurUI_000.shader)。

### Blur ver.1

新增模糊半径``_Radius``，通过_GrabTexture_TexelSize获得Grab纹理每个像素的颜色，遍历模糊范围内的每个像素点，累加他们的颜色并取均值。

```cpp

half4 frag(v2f i) : SV_TARGET
{
	float4 result = tex2Dproj(_GrabTexture, i.screenPos);
	
  float4 grabPos;
	grabPos.zw = i.screenPos.zw;

  //遍历范围内的每个像素点，累加它们的颜色
	for (int rangeX = 1; rangeX <= _Radius; rangeX++)
	{
		for (int rangeY = 1; rangeY < _Radius; rangeY++)
		{
			float2 w1 = i.screenPos.xy + float2(_GrabTexture_TexelSize.x * rangeX, _GrabTexture_TexelSize.y * rangeY);
			grabPos.xy = w1;
			result += tex2Dproj(_GrabTexture, i.screenPos + grabPos);

			float2 w2 = i.screenPos.xy + float2(_GrabTexture_TexelSize.x * rangeX, _GrabTexture_TexelSize.y * -rangeY);
			grabPos.xy = w2;
			result += tex2Dproj(_GrabTexture, i.screenPos + grabPos);
  	}
	}

  //取均值
	result /= _Radius * _Radius * 2 + 1; 

  //混合模糊后的GrabTex和自定义颜色
  float4 col = half4(_Color.a * _Color.rgb + (1 - _Color.a) * result.rgb, 1.0f);
	return col;
}

```

![](/assets/img/graphics/shader/blur_001.gif)

这一步的代码戳[这里](https://github.com/llapuras/ShaderLib/blob/master/UIBlur/BlurUI_001.shader)。

### 优化Sample次数

到这里效果看起来还蛮凑合了，但是上面那样的模糊程度已经是我电脑的极限了。

上述代码中，**重复采样纹理**给GPU带来了极大的负担，每个像素颜色需要经过_Radius*_Radius次的采样才能获得，采样总次数还要乘上一整张图片包含的总像素数，是相当可怕的计算量，_Radius取到300时就开始出现轻微卡顿了。

需要改进的是，在达到相同效果的前提下，**降低采样的次数**。

一个替换思路是，**分两个Pass对横向和竖向的像素分别进行计算**，即对于每个像素而言，只需要采样横向一条和竖向一条，共计**_Radius×2×2**个像素，而非**_Radius×_Radius×2×2**这一片正方形范围内的所有像素，这样大大减少了计算量，但最终效果与修改前相同。

![](/assets/img/graphics/shader/blur_002.gif)

代码改动不大，戳[这里](https://github.com/llapuras/ShaderLib/blob/master/UIBlur/BlurUI_002.shader)。

![](/assets/img/graphics/shader/blur_009.gif)

阔以开始做UI了w(ﾟДﾟ)w

![](/assets/img/line.png)

### 参考

1. [LEXDEV Tutorials·UI Blur](https://lexdev.net/tutorials/ui/blur.html#section_2)

2. [阮一峰的网络日志·高斯模糊的算法](http://www.ruanyifeng.com/blog/2012/11/gaussian_blur.html)

3. [Unity Shader之磨砂玻璃与水雾玻璃效果](https://blog.uwa4d.com/archives/UWALab_UnityShader.html)
