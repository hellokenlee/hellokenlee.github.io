---
layout: post
title:  "OpenGL中真正的深度值——z-buffer探究"
date:   2017-02-24 00:00:00 +0800
categories: OpenGL
tags: 数学 图形学

---

* content
{:toc}


***原创文章，未经作者(ken_4000@qq.com)授权，禁止转载。违者视为侵权，保留追究其法律责任的权利。***

### 前言
  正如[LearnOpenGL-高级OpenGL-深度测试](https://learnopengl.com/#!Advanced-OpenGL/Depth-testing)中所述，在OpenGL的深度值可以通过在fragment shader中调用`gl_FragCoord.z`获得，其值域为[0,1]。但是物体离我们眼睛(摄像机)的真正距离是多少呢？怎么通过OpenGL中的深度反求“真正的深度”呢？

### 推导过程

  我们假设在FragmentShader出来的的深度值为$ z_b $ =`gl_FragCoord.z`，因为$ z_b $是值域为[0,1]，我们首先得转换回NDC坐标系下，假设NDC坐标系下的深度值为$ z_n $，OpenGL是这样转换的：  
  
$$
z_b=0.5z_n+0.5
$$

  这时$ z_n $的值域为[-1,1]。
  我们设视角坐标下的深度值为$z_r$(也就是实际的深度值，aka.物体到我们眼睛的距离, r for real)，同时我们假设视角截面体的近平面距离=near，远平面距离=far，OpenGL是这样转换的:  
  
$$
z_b=\frac{1/z_r-1/near}{1/far-1/near}
$$

  上面公式反求$ z_r $，下面令$ f=far,n=near $：

$$
z_b=\frac{\frac{nf}{z_r}-f}{n-f}
$$

$$
z_b(n-f)=\frac{nf}{z_r}-f
$$

$$
\frac{nf}{z_r}=z_b(n-f)+f
$$

$$
z_r=\frac{nf}{z_b(n-f)+f}
$$

  把$z_b$带入：

$$
z_r=\frac{nf}{0.5(z_n+1)(n-f)+f}
$$

$$
z_r=\frac{2nf}{(z_n+1)(n-f)+2f}
$$

$$
z_r=\frac{2nf}{f+n+z_n(n-f)}
$$

$$
z_r=\frac{2nf}{f+n-z_n(f-n)}
$$

  这就是网站上[LearnOpenGL-高级OpenGL-深度测试](https://learnopengl.com/#!Advanced-OpenGL/Depth-testing)给的公式的由来。但是很明显，网站上给的公式有错误，或者说该网站上面的表述不清楚。（后来作者评论区补充说明了）作者说希望转换到线性的z写入到颜色中显示距离，然而作者给出的Shader代码：
```glsl
#version 330 core
out vec4 color;

float near = 1.0; 
float far  = 100.0; 
  
float LinearizeDepth(float depth) 
{
    float z = depth * 2.0 - 1.0; // Back to NDC 
    return (2.0 * near * far) / (far + near - z * (far - near));	
}

void main()
{             
    float depth = LinearizeDepth(gl_FragCoord.z) / far; // divide by far for demonstration
    color = vec4(vec3(depth), 1.0f);
}
```
  函数`LinearizeDepth(float depth) `仅仅求出了$z_r$的值，从下面的验证可以看出$z_r\in[n,f]$，所以即使最后除了$far$之后$depth也是\in[n/f,1]$，而并非作者上面说的线性深度。
  要求线性深度，应该使用公式：
  
$$
F_{depth}=\frac{z-n}{f-n}
$$

  代码改为：
```glsl
...
float depth = (LinearizeDepth(gl_FragCoord.z) - near)/ (far-near); //非线性转线性
...
```

### 验证
  因为

$$
z_n\in[-1,1]
$$

  所以分母(注意$f>n$)

$$
{f+n-z_n(f-n)}\in[2f,2n]
$$

  所以

$$
z_r\in[n,f]
$$

  你问我深度值$z_r < near$的怎么办？$z_r < near$也就是不在视锥体内的值，早在光栅化之前给剔除了，根本不会传到fragment shader。对于$z_r > far$的值亦然。

  
### 参考资料
1. [https://learnopengl.com/#!Advanced-OpenGL/Depth-testing](https://learnopengl.com/#!Advanced-OpenGL/Depth-testing)
2. [http://web.archive.org/web/20130416194336/http://olivers.posterous.com/linear-depth-in-glsl-for-real](http://web.archive.org/web/20130416194336/http://olivers.posterous.com/linear-depth-in-glsl-for-real)
2. [http://stackoverflow.com/questions/6652253/getting-the-true-z-value-from-the-depth-buffer](http://stackoverflow.com/questions/6652253/getting-the-true-z-value-from-the-depth-buffer)
