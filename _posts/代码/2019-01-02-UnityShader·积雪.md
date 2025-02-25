---
date: 2019-02-02 12:26:40
layout: post
title: Shader·积雪
subtitle: 
description: 
image: /assets/img/graphics/shader/rim.gif
#optimized_image: /assets/img/opengl/spongebob.jpg
category: article
published: true
tags:
  - graphics
  - unity
  - shader
author: llapuras13
---

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

```c++
Shader "Lapu/ToonSnow"{
    Properties{
        _ModelColor("Model Color", Color) = (0.5,0.5,0.5,1)
        _ModelTex("Model Tex", 2D) = "white" {}
	    _SnowRamp("Snow Ramp", 2D) = "white" {}
        _SnowColor("Snow Color", Color) = (0.5,0.5,0.5,1)
		_SnowRimColor("Snow Rim Color", Color) = (0.5,0.5,0.5,1)
		_SnowRimIntensity("Snow Rim Intensity", Range(0,10)) = 3
        _SnowDir("Snow Direction", Vector) = (0,1,0)
        _SnowSize("Snow Size", Range(0,2)) = 1
        _SnowHeight("Snow Height",Range(0,10)) = 0 
		_Atten("Atten",Range(0,10)) = 1.5 
		_Phong ("Phong Strengh", Range(0,1)) = 0.5
		_EdgeLength ("Edge length", Range(2,50)) = 5
    }


SubShader{
    Tags{"RenderType" = "Opaque"}
    LOD 100
    Cull off
    
    CGPROGRAM

    #pragma surface surf Lambert vertex:vert 
    #include "Tessellation.cginc"


 	struct Input {
		float2 uv_ModelTex : TEXCOORD0; 
		float3 worldPos;
		float3 viewDir;
		float3 lightDir;
	};

	struct appdata {                                                                                   
		float4 vertex : POSITION;
		float3 normal : NORMAL;
	};

    float4 _ModelColor;
    sampler2D _ModelTex;  
    sampler2D _SnowRamp;  
    float4 _SnowColor;
	float4 _SnowRimColor;
	float _SnowRimIntensity;
	float4 _SnowDir;
	float  _SnowSize;
	float  _SnowHeight;
	float  _Atten;
	float _Phong;
	float _EdgeLength;
	
    float4 tessEdge (appdata_full v0, appdata_full v1, appdata_full v2)
    {
        return UnityEdgeLengthBasedTess (v0.vertex, v1.vertex, v2.vertex, _EdgeLength);
	}

	void vert(inout appdata_full v)
	{
		float3 up = float3(0,1,0);
		if (dot(v.normal, normalize(_SnowDir.xyz)) > -(_SnowSize-1) )
		{
			v.vertex.xyz += up.xyz * _SnowHeight * 0.0001;
		}
	}


	void surf(Input IN, inout SurfaceOutput o) {
		float3 localPos = (IN.worldPos - mul(unity_ObjectToWorld, float4(0, 0, 0, 1)).xyz); 
		half d = dot(o.Normal, IN.lightDir)*0.5 + 0.5; 
		half4 c = tex2D(_ModelTex, IN.uv_ModelTex) * _ModelColor; 
		half3 snowRamp = tex2D(_SnowRamp, float2(d, d)).rgb;
		o.Albedo = _Atten * c.rgb * _ModelColor;
		half rim = 1.0 - saturate(dot(normalize(IN.viewDir), o.Normal)); 
		if (dot( o.Normal, normalize(_SnowDir.xyz)) >= -(_SnowSize-1)) { 
			o.Albedo = _Atten * _SnowColor * snowRamp; 
			o.Emission = _SnowRimColor.rgb *pow(rim, _SnowRimIntensity);
		}
	}

    ENDCG
    }

    Fallback "Diffuse"
}

```

### 参考

1.[Quick Game Art Tips - Unity Toon Snow](https://www.patreon.com/posts/15944770)·Toon积雪效果<br>
2.[Shader Showcase Saturday #6: Dynamic Snow](https://www.alanzucconi.com/2018/08/18/shader-showcase-saturday-6/)·提到了用noise map制作平面雪地效果