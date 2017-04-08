---
layout: post
title:  "在MSAA中关于 glBlitFramebuffer 函数出现 Invalid Operation 的解析"
date:   2017-04-08 00:12:00 +0800
categories: OpenGL
tags: 图形学 编程

---

* content
{:toc}


***原创文章，未经作者(ken_4000@qq.com)授权，禁止转载。违者视为侵权，保留追究其法律责任的权利。***

### 前言
  跟着[LearnOpenGL-AntiAliasing](https://learnopengl.com/#!Advanced-OpenGL/Anti-Aliasing)章节做了一下多重采样抗据齿，使用第一种方法，即调用GLFW的`glfwWindowHint(GLFW_SAMPLES, 4)`来使用多重采样抗据齿是没有问题的。然而，当我做到第二种方法的时候，即现在一个Multisample的Framebuffer中绘制出我想绘制的图形，然后调用`glBlitFramebuffer(...)`函数来块传输我的MSAA帧缓冲到默认缓冲(显示帧缓冲)的时候，屏幕出现黑屏，我调用`glGetError()`查看错误的时候，出现了`Invalid Operation`的错误。  
  主要错误代码如下(不相关代码被省略)：  
```cpp
//绑定块缓冲代码如下 屏幕大小800x600 4xMSAA
GLuint MSAAfbo;
glGenFramebuffers(1, &MSAAfbo);
glBindFramebuffer(GL_FRAMEBUFFER, MSAAfbo);
	// 创建多重采样纹理
	GLuint MSAAtex;
	glGenTextures(1, &MSAAtex);
	glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, MSAAtex);
		glTexImage2DMultisample(GL_TEXTURE_2D_MULTISAMPLE, 1, GL_RGB, 800, 600, GL_TRUE);
	glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, 0);
	// 附加到FB上
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D_MULTISAMPLE, MSAAtex, 0);
	// 生产Render Buffer Object
	GLuint MSAArbo;
	glGenRenderbuffers(1, &MSAArbo);
	glBindRenderbuffer(GL_RENDERBUFFER, MSAArbo);
		glRenderbufferStorageMultisample(GL_RENDERBUFFER, 1, GL_DEPTH24_STENCIL8, 800, 600);
	glBindRenderbuffer(GL_RENDERBUFFER, 0);
	// 附加到FB上
	glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, MSAArbo);
	//错误检查
	if(glCheckFramebufferStatus(GL_FRAMEBUFFER)!=GL_FRAMEBUFFER_COMPLETE){
		cout<<"ERROR:: FRAMEBUFFER init faild!"<<endl;
		return;
	}
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

```cpp
//主循环代码(绘制循环)如下：
while(!glfwWindowShouldClose(window)){
	glfwPollEvents();
	// 在MSAAFB上绘制
	glBindFramebuffer(GL_FRAMEBUFFER, MSAAfbo);
	glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	cube.draw();//draw call
	// 转移结果到默认FB
	glBindFramebuffer(GL_READ_FRAMEBUFFER, MSAAfbo);
	glBindFramebuffer(GL_DRAW_FRAMEBUFFER, 0);
	glBlitFramebuffer(0, 0, 800, 600, 0, 0, 800, 600, GL_COLOR_BUFFER_BIT, GL_NEAREST);
	checkError();//此处出现了 GL_INVALID_OPERATION
	// 交换
	glfwSwapBuffers(window);
}
```

### 错误分析
  结果是我Debug了一个晚上，也没有找出我哪里写错了。 更加神奇的是，之后我换了一台PC， 用编译运行同样的代码， 居然成功显示了。因而我觉得是显卡驱动的问题，我手头上有3块GPU，对于上述代码，运行结果如下：
  
| GPU | OS | Status |  
| --- | -- | ------ |  
| Intel® Haswell i5 4570 | Ubuntu 14.04 | 失败： GL_INVALID_OPERATION|  
| Intel® Ivy Bridge i7 3630QM | Windows 7 | 成功 | 
| nVidia GT650M | Windows 7 | 成功 | 

  很明显，坑在Intel的Linux显卡驱动上。下面的几个小哥好像也遇到了类似的问题：  
  [https://learnopengl.com/#!Advanced-Lighting/Deferred-Shading](https://learnopengl.com/#!Advanced-Lighting/Deferred-Shading)  
  ![](/images/q1.png)  
  [https://www.opengl.org/discussion_boards/showthread.php/184723-Question-about-FBO-and-16Bit-RGB-Half-Float-textures](https://www.opengl.org/discussion_boards/showthread.php/184723-Question-about-FBO-and-16Bit-RGB-Half-Float-textures)  
  根据官方给出的文档，GL_INVALID_OPERATION出现的原因如下：  
  ![](/images/doc1.png)
  我个人的猜测是在Intel HD4600显卡驱动上，`glBlitFramebuffer(...)`MSAA Framebuffer中的值的类型和默认FrameBuffer中的类型不一样， 其驱动实现的函数不能自动转换，因此抛出一个`GL_ERROR`。

### 解决方案
  尝试了几次之后发现了一个简单而且神奇的解决方案，我们需要定义2块帧缓冲，第一块缓冲是MSAA的缓冲，第二块是常规的帧缓冲。在每次绘制的时候我们做2次Blit操作， 第一次从 MSAA FBO -> Intermediate FBO； 第二次从 Intermediate FBO -> Default FBO(0)。 至于为什么work我也不太清楚...具体代码如下：
```cpp
// FBO声明代码
// 创建MSAA缓冲
GLuint MSAAfbo;
glGenFramebuffers(1, &MSAAfbo);
glBindFramebuffer(GL_FRAMEBUFFER, MSAAfbo);
	// 创建多重采样纹理
	GLuint MSAAtex;
	glGenTextures(1, &MSAAtex);
	glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, MSAAtex);
		glTexImage2DMultisample(GL_TEXTURE_2D_MULTISAMPLE, 1, GL_RGB, 800, 600, GL_TRUE);
	glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, 0);
	// 附加到FB上
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D_MULTISAMPLE, MSAAtex, 0);
	// 生产Render Buffer Object
	GLuint MSAArbo;
	glGenRenderbuffers(1, &MSAArbo);
	glBindRenderbuffer(GL_RENDERBUFFER, MSAArbo);
		glRenderbufferStorageMultisample(GL_RENDERBUFFER, 1, GL_DEPTH24_STENCIL8, 800, 600);
	glBindRenderbuffer(GL_RENDERBUFFER, 0);
	// 附加到FB上
	glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, MSAArbo);
glBindFramebuffer(GL_FRAMEBUFFER, 0);

//创建中间帧缓冲对象
GLuint fbo;
glGenFramebuffers(1, &fbo);
glBindFramebuffer(GL_FRAMEBUFFER, fbo);
	//创建纹理缓冲附件对象
	GLuint texColorBuffer;
	glGenTextures(1, &texColorBuffer);
	glBindTexture(GL_TEXTURE_2D, texColorBuffer);
		glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);
		glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
		glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glBindTexture(GL_TEXTURE_2D, 0);
	//附着到帧缓冲对象上
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texColorBuffer, 0);
	//创建深度测试和模板测试缓冲附件，以便于GL在绘制该FB的时候可以进行深度测试和模板测试
	GLuint renderBuffer;
	glGenRenderbuffers(1, &renderBuffer);
	glBindRenderbuffer(GL_RENDERBUFFER, renderBuffer);
		//分配内存
		glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, 800, 600);
	glBindRenderbuffer(GL_RENDERBUFFER, 0);
	//附着到帧缓冲对象上
	glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, renderBuffer);
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

```cpp
// 绘制循环
while(!glfwWindowShouldClose(window)){
	glfwPollEvents();
	// 在MSAAFB上绘制
	glBindFramebuffer(GL_FRAMEBUFFER, MSAAfbo);
	glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	cube.draw();// draw call

	// 转移结果到中间
	glBindFramebuffer(GL_READ_FRAMEBUFFER, MSAAfbo);
	glBindFramebuffer(GL_DRAW_FRAMEBUFFER, fbo);
	glBlitFramebuffer(0, 0, 800, 600, 0, 0, 800, 600, GL_COLOR_BUFFER_BIT, GL_NEAREST);
	checkError();// no error
	// 将中间转移到默认缓冲
	glBindFramebuffer(GL_READ_FRAMEBUFFER, fbo);
	glBindFramebuffer(GL_DRAW_FRAMEBUFFER, 0);
	glBlitFramebuffer(0, 0, 800, 600, 0, 0, 800, 600, GL_COLOR_BUFFER_BIT, GL_NEAREST);
	checkError();// no error
	// 显示
	glfwSwapBuffers(window);
}
```
  
### 参考资料
1. [https://learnopengl.com/#!Advanced-OpenGL/Anti-Aliasing](https://learnopengl.com/#!Advanced-OpenGL/Anti-Aliasing)
2. [http://stackoverflow.com/questions/11915267/glblitframebuffer-invalid-operation](http://stackoverflow.com/questions/11915267/glblitframebuffer-invalid-operation)
2. [https://www.opengl.org/discussion_boards/showthread.php/179909-MSAA-with-maginification-s-simple-question](https://www.opengl.org/discussion_boards/showthread.php/179909-MSAA-with-maginification-s-simple-question)
3. [http://opengl.developpez.com/docs/man/man/glBlitFramebuffer](http://opengl.developpez.com/docs/man/man/glBlitFramebuffer)
