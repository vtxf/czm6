---
layout: post  
title: 在Cesium的PlaneGeometry上播放视频
---

Cesium的sancastle示例中有一个在球体上创建视频的例子，不过大家可能更需要一个在平面上创建视频，好应用到实际工程中去。
<!-- more -->

![输入图片说明](https://images.gitee.com/uploads/images/2018/0812/171406_67e648c4_470194.png "屏幕截图.png")

代码比较简单，使用了Material Apperance，并且创建了一个包含video的Material。
需要注意的点是：new Cesium.Material的方式，把video作为构造函数参数传进去的做法不能用，Cesium有bug，内部没加对HtmlVideo的判断；

需要在html中增加如下代码：
```
<video id="trailer" style="display: none;" autoplay="" loop="" crossorigin="" controls="">
    <source src="https://cesiumjs.org/videos/Sandcastle/big-buck-bunny_trailer.webm" type="video/webm">
    <source src="https://cesiumjs.org/videos/Sandcastle/big-buck-bunny_trailer.mp4" type="video/mp4">
    <source src="https://cesiumjs.org/videos/Sandcastle/big-buck-bunny_trailer.mov" type="video/quicktime">
    Your browser does not support the <code>video</code> element.
</video>
```

然后js代码如下：
```
var viewer = new Cesium.Viewer('cesiumContainer');
var scene = viewer.scene;

var dimensions = new Cesium.Cartesian3(400000.0, 400000.0, 1.0);
var positionOnEllipsoid = Cesium.Cartesian3.fromDegrees(116.3912, 39.920);
var translateMatrix = Cesium.Transforms.eastNorthUpToFixedFrame(positionOnEllipsoid);
var rotationXMatrix = Cesium.Matrix4.fromRotationTranslation(Cesium.Matrix3.fromRotationX(Cesium.Math.toRadians(-90.0)));
var rotationYMatrix = Cesium.Matrix4.fromRotationTranslation(Cesium.Matrix3.fromRotationY(Cesium.Math.toRadians(90.0)));
var scaleMatrix = Cesium.Matrix4.fromScale(dimensions);

var planeZModelMatrix = new Cesium.Matrix4();
Cesium.Matrix4.multiply(translateMatrix, scaleMatrix, planeZModelMatrix);

var planeXModelMatrix = new Cesium.Matrix4();
Cesium.Matrix4.multiply(translateMatrix, rotationXMatrix, planeXModelMatrix);
Cesium.Matrix4.multiply(planeXModelMatrix, scaleMatrix, planeXModelMatrix);

var planeYModelMatrix = new Cesium.Matrix4();
Cesium.Matrix4.multiply(translateMatrix, rotationYMatrix, planeYModelMatrix);
Cesium.Matrix4.multiply(planeYModelMatrix, scaleMatrix, planeYModelMatrix);

var videoElement = document.getElementById('trailer');

createPlane(planeZModelMatrix, new Cesium.Color(0.0, 0.0, 1.0, 1.0));
//createPlane(planeXModelMatrix, new Cesium.Color(1.0, 0.0, 0.0, 1.0));
//createPlane(planeYModelMatrix, new Cesium.Color(0.0, 1.0, 0.0, 1.0));

function createPlane(planeModelMatrix) {
   // 创建平面
    var planeGeometry = new Cesium.PlaneGeometry({
        vertexFormat : Cesium.MaterialAppearance.VERTEX_FORMAT,
    });

    // 创建平面外轮廓
    var planeOutlineGeometry = new Cesium.PlaneOutlineGeometry({
    });

    var planeGeometryInstance = new Cesium.GeometryInstance({
        geometry : planeGeometry,
        modelMatrix : planeModelMatrix
    });
    
    var material = Cesium.Material.fromType('Image');
    material.uniforms.image = videoElement;
    
    /* Cesium 1.47 有bug，以下代码会报错
    var material = new Cesium.Material({
        fabric : {
            type : 'Image',
            uniforms : {
                image : videoElement
            }
        }
    });
    */

    scene.primitives.add(new Cesium.Primitive({
        geometryInstances : planeGeometryInstance,
        appearance : new Cesium.MaterialAppearance({
            closed: false,
            //translucent: false,
            material: material
        })
    }));

    var planeOutlineGeometryInstance = new Cesium.GeometryInstance({
        geometry : planeOutlineGeometry,
        modelMatrix : planeModelMatrix,
        attributes : {
            color : Cesium.ColorGeometryInstanceAttribute.fromColor(new Cesium.Color(1.0, 1.0, 1.0, 1.0))
        }
    });

    scene.primitives.add(new Cesium.Primitive({
        geometryInstances : planeOutlineGeometryInstance,
        appearance : new Cesium.PerInstanceColorAppearance({
            flat : true,
            renderState : {
                lineWidth : Math.min(2.0, scene.maximumAliasedLineWidth)
            }
        })
    })); 
}

viewer.camera.flyToBoundingSphere(new Cesium.BoundingSphere(positionOnEllipsoid, 300000));
```

点击[czm6.com](https://www.czm6.com)，查看更多内容。

