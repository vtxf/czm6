---
layout: post  
title: 模型压平代码操作步骤和修改文件说明
---

模型压平代码操作步骤和修改文件说明  
<!-- more -->

### 模型压平效果图  
![模型压平效果图 ](https://images.gitee.com/uploads/images/2018/0810/180222_6e42beee_470194.png "模型压平效果图 .png")

### 快速使用说明
![快速使用说明](https://images.gitee.com/uploads/images/2018/0810/180247_e628d58f_470194.png "快速使用说明.png")

1. 打开名字为“覆盖文件”的文件夹，全选复制，粘贴到Cesium源码根目录下；
2. Cesium源码根目录运行cmd, 执行 npm run build 进行编译；
3. 执行npm start启动服务；
4. 打开浏览器输入以下网址：http://localhost:8080/Apps/Sandcastle/gallery/vtxf-flatenning.html，即可在网页中操作模型压平功能。

### 覆盖文件结构
目前需要覆盖的文件如下，实际上只有viewer.js会被覆盖，其他文件都是新增。  
![文件结构](https://images.gitee.com/uploads/images/2018/0810/180317_ea312175_470194.png "文件结构.png")

### Viewer.js中修改点
该文件一般情况下直接覆盖即可，但因版本升级造成其他异常，可以参照以下修改处自行更正一下。更正的地方很少。  
基本规律就是有Cesium3DTileset的地方，都加上一个Cesium3DTileset2即可。  
![输入图片说明](https://images.gitee.com/uploads/images/2018/0810/180353_ffb9ef2c_470194.png "屏幕截图.png")
![输入图片说明](https://images.gitee.com/uploads/images/2018/0810/180401_4760e505_470194.png "屏幕截图.png")
![输入图片说明](https://images.gitee.com/uploads/images/2018/0810/180407_10a425e9_470194.png "屏幕截图.png")
![输入图片说明](https://images.gitee.com/uploads/images/2018/0810/180414_d0ee9514_470194.png "屏幕截图.png")

### 压平示例代码
目前的处理方式是重新创建了一个Cesium3DTiles2这个类，凡事使用这个类创建的3dtiles数据，都可以进行模型压平操作。使用代码如下：  
```
///////////////////////////////////////////////////////////////////////////
// 创建压平多边形方法1：直接构件压平多边形
function createFlattenedPolygon() {
    // 1.1 给定压平多边形的地理坐标点
    var shapePoints = [
        [108.95909563341269, 34.21954069619409, 0],
        [108.95980684652642, 34.21953868188837, 0],
        [108.9598189099838, 34.22016413275479, 0],
        [108.95907674800323, 34.2201857401145, 0]
    ];

    // 1.2 装换成笛卡尔坐标
    var newPoints = shapePoints.map(function (point) {
        return Cesium.Cartesian3.fromDegrees(point[0], point[1], point[2]);
    });

    // 1.3 创建压平多边形
    var myPolygon = new Cesium.PolygonGeometry({
        polygonHierarchy : new Cesium.PolygonHierarchy(newPoints),
        //perPositionHeight : true,
        height: 0.0
    });

    // 1.4 加入到_flattenedPolygons中去，并置脏
    tileset._flattenedPolygons.push(myPolygon);
    tileset._flattenedPolygonDirty = true;
}
```
以上代码源于此文件：Apps\Sandcastle\gallery\vtxf-flatenning.html。该示例代码内部还提供了另外两种方法：
1 从geojson文件中加载压平多边形数据
2 定义鼠标操作，以交互模式创建压平多边形
可以查阅相关文件。
![输入图片说明](https://images.gitee.com/uploads/images/2018/0810/180443_f7728873_470194.png "屏幕截图.png")
![输入图片说明](https://images.gitee.com/uploads/images/2018/0810/180449_4a24844e_470194.png "屏幕截图.png")
