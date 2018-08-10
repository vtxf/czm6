---
layout: post  
title: 给Cesium增加自定义logo的方法
---

增加logo的最终效果如下：  
<!-- more -->
![logoin](https://images.gitee.com/uploads/images/2018/0808/221529_b44f2828_470194.png "logoin.png")

我们所要做的就是使用Cesium自带的ViewportQuad类，来创建一个对象，然后添加到场景中去即可。  
这种方法加载的logo和div的方式不同，是完全融入于Cesium的三维窗口当中的，这个logo也是三维的一部分。  

1 先准备好logo图片  
![logo](https://images.gitee.com/uploads/images/2018/0808/221053_67e3c1d9_470194.png "logo.png")

2 代码如下：  
```
var viewer = new Cesium.Viewer('cesiumContainer', {
    animation: false,
    baseLayerPicker: false,
    fullscreenButton: false,
    vrButton: false,
    geocoder: false,
    homeButton: false,
    infoBox: false,
    sceneModePicker: false,
    selectionIndicator: false,
    timeline: false,
    navigationHelpButton: false,
    navigationInstructionsInitiallyVisible: false
});

// 隐藏Cesium自身的logo
viewer._cesiumWidget._creditContainer.style.display="none";

// 使用ViewportQuad创建一个显示图片的区域
var viewportQuad = new Cesium.ViewportQuad();
viewportQuad.rectangle = new Cesium.BoundingRectangle(5, 5, 80, 92);
viewer.scene.primitives.add(viewportQuad);

viewportQuad.material = new Cesium.Material({
    fabric: {
        type: 'Image',
        uniforms: {
            color: new Cesium.Color(1.0, 1.0, 1.0, 1.0),
            image: 'https://images.gitee.com/uploads/images/2018/0808/221053_67e3c1d9_470194.png'
        }
    }
});
```
