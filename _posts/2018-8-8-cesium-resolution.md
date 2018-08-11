---
layout: post  
title: 解决Cesium显示画面模糊的问题  
---

有的时候，在某些笔记本或者显示器查看Cesium的三维窗口时，会发现图像非常模糊，锯齿非常明显。
<!-- more -->

![模糊图片](https://images.gitee.com/uploads/images/2018/0808/222029_14824194_470194.png "模糊图片.png")


如果遇到这种情况，创建viewer以后，增加以下代码即可：  
```
viewer._cesiumWidget._supportsImageRenderingPixelated = Cesium.FeatureDetection.supportsImageRenderingPixelated();
viewer._cesiumWidget._forceResize = true;
if (Cesium.FeatureDetection.supportsImageRenderingPixelated()) {
    var vtxf_dpr = window.devicePixelRatio;
    // 适度降低分辨率
    while (vtxf_dpr >= 2.0) {
        vtxf_dpr /= 2.0;
    }
    //alert(dpr);
    viewer.resolutionScale = vtxf_dpr;
}
```
之后就会发现三维窗口又恢复的清晰的状态。如下图所示：

![清洗图片](https://images.gitee.com/uploads/images/2018/0808/222157_4bb326d6_470194.png "清晰图片.png")

为什么会这样呢？

其实这和笔记本的设置有关系。如果你的win10电脑的显示设置中 缩放和布局大小 此项的值 不是100%，而是125%或者别的数字时，就会造成Cesium三维窗口的模糊。

![显示设置](https://images.gitee.com/uploads/images/2018/0808/222300_ad7eacb8_470194.png "显示设置.png")

当然还有另外一种不改代码变清晰的方法，就是把这个缩放大小调整为100%。  

点击[czm6.com](https://www.czm6.com)，查看更多内容。
