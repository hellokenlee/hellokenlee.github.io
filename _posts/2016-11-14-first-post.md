---
layout: post
title:  "第一篇博客"
date:   2016-11-14 19:23:00 +0800
categories: 杂谈
tags: 感想 环境搭建

---

* content
{:toc}

### **为什么会有这一个博客?**
&emsp;&emsp;数了数有接近一年没写博客了，这一年发生了太多事情： 毕业，找工作，升学，分手...还有很多数不清的，麻烦的事情。感觉自己也变了很多，不像以前那样无忧无虑了。以前的同学大多数都去了工作，没有能像以前那样有那么多时间聚在一起玩。现在的同学倒也还不错，大家为了前程都非常努力学习。应了一句歌词 “朋友们都去了远方，我只能每天早上和树说话。” 从前玩得最好的几个同学都开始自给自足了，开始探讨买车买房结婚生子的人生大计。我呢，还在吭老读书，结婚生子的事情似乎比读本科的时候离得更加远了。  

&emsp;&emsp;不过，最近的生活也算是过得稳定而且充实吧～就想着更新一下博客。原来的博客是用worldpress搭建在新浪云SAE上面的，结果前几天登录上去他说我余额不足，然后把我的东西给删了。同时我前几天刚刚重装了系统，之前本地的备份也都没有了...自己的东西没备份好，怪不了其他人(喂谁把SAE给黑了好不好...)。不过这倒让我发现了WorldPress博客的弊端：WorldPress的博客数据是存在他的MySQL数据库里面的，必须要导出成XML才能备份，而且编辑也得用他的编辑器来编辑，格式才不会乱(尽管有插件兼容markdown或者其他语法)。没办法，塞翁失马，就让他失呗。从新开始也是一件好事，毕竟以前的写的博文幼稚又无知。

### **为什么选择在这里搭建**
&emsp;&emsp;其实写博客就像是在浩瀚的网络中搭建一个家，出于经常玩Minecraft和饥荒的习惯，对家的选址总是非常谨慎，想了很多地方，注册了很多个账号(比如CSDN，博客园，163博客...)都觉得不太好。于是总结了一下自己对博客的需求：  

- **没有广告**：一个读者看我的文章还要强制他看广告，是我对他的不尊重。
- **免费**：之前SAE上的费用是一次他故障然后赔我的费用，一直没充值，没想到就这样没了。
- **能插入代码，数学公式，图片**：每一个程序员写博客基本上都需要这三样东西吧。
- **容易迁移和保存**：前一次博客的教训。
- **简单！简单！简单！**：只是一个记录思考的地方

&emsp;&emsp;对着上面的要求排除了一下，也就剩下Github Pages和简书这两家了，都试了一下，感觉都不错，尽管简书不能直接的插入数学公式，但是能通过第三方服务器生成图片插入的方法来hack，问题不大。后来还是觉得Github Pages的可定制性高，自己也学过一些Web的知识，所以选择在这安家。  

&emsp;&emsp;说一说Github Pages的优点和缺点吧：  

- 优点：
  - **免费，没广告。**
  - **每一次更新博客都是一次commit。**这对于一个程序员来说是最安心的了。只要Github不倒闭，你的博客就在那，而且支持版本回退。
  - **博文内容使用markdown格式。**可以非常轻而易举地插入公式和代码，支持代码高亮，支持在线编辑。
  - **容易迁移。**只需要把所有.md文件拷贝下来即可，不涉及数据库啊，服务器啊这些东西。
- 缺点：
  - **搭建麻烦而且坑多。**我之前以为只要是静态博客生成工具生成的博客都可以使用，后来发现GithubPages只支持jekyll，其他生成工具(比如[pelican]())只能本地生成，然后把html push上去。同时Github在2016年2月升级到了jekyll3，很多教程都是以前的版本，坑有点多。
  - **强制https访问。**使用http访问的话会出错。

### **怎么在这里搭建**
&emsp;&emsp;我使用的是jekyll+Github Pages，没有为什么，因为Github只支持jekyll。教程网上一搜一大堆，我不在这细说。不过我倒是想说说我遇到的坑(估计是由于我完全不会ruby的缘故，特别是估计我以后都不会用到ruby，也懒得学)：  

- jekyll需要ruby2.0: 在Ubuntu14.04上，需要

```shell
	sudo apt-get install ruby2.0
```

&emsp;&emsp;然后

```shell
	sudo gem2.0 install jekyll
```

&emsp;&emsp;才能使用，不然会出错。

- 插入数学公式： 我用的markdown公式是支持mathjax公式编辑的，但是测试jekyll的时候发现不行。解决方法是在博客目录(含有_post/的目录)下的_layout/下的default.html，在head标签中插入一Mathjax提供的JS库。
 
```html
	<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
```

&emsp;&emsp;当然你也可以把Mathjax的JS库下载下来放到某个子目录然后通过相对路径访问，不过Mathjax太大了，push到Github上面不是很道德，所以我采用的是外链的。

- 代码高亮！！！这个弄了我好久。我用到的主题是LessOrMore这个主题，但是他好像在jekyll2.x的时候写的，直接使用放在Github Pages上面显示不出来。主题自带的_config.yaml文件是这样的：  

```ruby
	highlighter: rouge
	# Build settings
	markdown: kramdown
```

&emsp;&emsp;然后push到Github上面，只能通过jekyll大括号+百分号的方法高亮，不能使用markdown自带的三个backticks的方法，所以写markdown的时候很不爽。我尝试过很多方法。包括更改_config.yaml:

```ruby
	markdown: kramdown
	highlighter: rouge
	kramdown:
		input: GFM
		syntax_highlighter: rouge
```

&emsp;&emsp;和

```ruby
	pygments: true
```

&emsp;&emsp;都不行。其实解决方法就是把关于highlighter和kramdown的几行都删了，因为现在Github只支持kramdown和rouge，所以无需在配置中声明。

- Disqus：这个也有教程。关键在于，Disqus会给你一个类似于

```js
	'http://' + disqus_shortname + '.disqus.com/embed.js'
```

&emsp;&emsp;的js脚本给你插入，但是Github只支持Https，因此我们需要把所有外链的js脚本(包括之前说的mathjax)都改成https来访问，不然会链接失败。如果你在本地的jekyll使用localhost访问正常，但是到Github Pages上面访问出错，很有可能就是由于使用Http外链js的原因。

### 感谢
&emsp;&emsp;搭建博客的时候用了很多人的东西，特别感谢。谢谢你们创造出来了这些东西，谢谢你们把这些东西开源。  

- [Github](https://www.github.com) ——提供的不止是代码托管。
- [Jekyll](http://www.jekyll.com/) ——优秀的产品(在某种意义上算是)。
- [Ruby](http://www.ruby-lang.org/en/) ——尽管我不太喜欢这种语言，写东西还是喜欢c/cpp or Python or Mathlab，大概因为我老。
- [LessOrMore](https://github.com/luoyan35714/LessOrMore) ——我用到的jekyll主题，但经过仔细修改，感谢作者[luoyan](https://github.com/luoyan35714)。
