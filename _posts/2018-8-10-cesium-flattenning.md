---
layout: post  
title: Cesium上的倾斜模型压平操作展示  
---

这里展示一下我在Cesium上开发的模型压平操作，参考了wish3d的压平原理。相比wish3d的压平操作做了如下改进：  
<!-- more -->
1. 可以在符合OGC标准的3dtiles格式上直接操作，而wish3d只能针对自身的私有格式数据；
2. 压平面基本不会闪烁，而wish3d的会有少量闪烁现象；
3. 理论上比wish3d的渲染性能会更快一点，wish3d的shader中需要去四次压平纹理的高度值，而我改进以后只需要获取一次即可；
4. 解决对数深度问题，wish3d应该是没有使用对数深度，包括其他特效也是一样，可能会给未来展示宏观模型带来物体被裁切或者闪面问题。

![倾斜模型压平](https://images.gitee.com/uploads/images/2018/0811/120608_1937c92a_470194.png "倾斜模型压平.png")

这里是模型压平的[演示视频](https://www.bilibili.com/video/av28905193/)

<iframe with="800px" height="600px" src="//player.bilibili.com/player.html?aid=28905193&cid=50119893&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>



