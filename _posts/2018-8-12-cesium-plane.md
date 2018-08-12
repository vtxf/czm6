---
layout: post  
title: 写一个Cesium Sandcastle中没有的PlaneGeometry创建的示例  
---

### Cesium Sandcastle中Entity API创建的Plane有点儿问题  
Cesium在前两个月的版本中新增了Plane这种Geometry类型，然后只给了一个Entity API下的示例。其中按照示例代码中的写法，这三个平面分别垂直于x、y、z三个轴。  
<!-- more -->
![输入图片说明](https://images.gitee.com/uploads/images/2018/0812/154606_9cf782a2_470194.png "屏幕截图.png")  
然而这个示例看着，总让人有点儿觉得哪儿有点儿不对的感觉。。三个轴的对应关系貌似有问题。。于是，我把他们的原点放在一起，结果就变成了下图这样。  
![输入图片说明](https://images.gitee.com/uploads/images/2018/0812/155125_2bdd5a9b_470194.png "屏幕截图.png")  
Cesium通过Entity API创建的Plane貌似不太好控制姿态。。

### 使用Primitive API创建一个Plane  
Cesium没有提供Primitive API的PlaneGeometry示例，于是我用Primitive API写了以下示例，这样通过ModelMatrix来控制姿态就会方便很多。
![输入图片说明](https://images.gitee.com/uploads/images/2018/0812/160515_a185fead_470194.png "屏幕截图.png")

```
var viewer = new Cesium.Viewer('cesiumContainer');
var scene = viewer.scene;
var dimensions = new Cesium.Cartesian3(400000.0, 300000.0, 1.0);
var positionOnEllipsoid = Cesium.Cartesian3.fromDegrees(116.3912, 39.920);
var translateMatrix = Cesium.Transforms.eastNorthUpToFixedFrame(positionOnEllipsoid);
var scaleMatrix = Cesium.Matrix4.fromScale(dimensions);

var planeModelMatrix = new Cesium.Matrix4();
Cesium.Matrix4.multiply(translateMatrix, scaleMatrix, planeModelMatrix);

// 创建平面
var planeGeometry = new Cesium.PlaneGeometry({
    vertexFormat : Cesium.PerInstanceColorAppearance.VERTEX_FORMAT,
});

// 创建平面外轮廓
var planeOutlineGeometry = new Cesium.PlaneOutlineGeometry({
});

var planeGeometryInstance = new Cesium.GeometryInstance({
    geometry : planeGeometry,
    modelMatrix : planeModelMatrix,
    attributes : {
        color : Cesium.ColorGeometryInstanceAttribute.fromColor(new Cesium.Color(1.0, 0.0, 0.0, 0.5))
    }
});

scene.primitives.add(new Cesium.Primitive({
    geometryInstances : planeGeometryInstance,
    appearance : new Cesium.PerInstanceColorAppearance({
        closed: true
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

viewer.camera.flyToBoundingSphere(new Cesium.BoundingSphere(positionOnEllipsoid, 300000));
```

### 接下来创建三个分别垂直x、y、z轴的平面
效果如下图所示，很显然，使用Primitive API来控制姿态还是靠谱一点！  
![输入图片说明](https://images.gitee.com/uploads/images/2018/0812/162917_0e9c58df_470194.png "屏幕截图.png")  
代码如下：
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

createPlane(planeZModelMatrix, new Cesium.Color(0.0, 0.0, 1.0, 1.0));
createPlane(planeXModelMatrix, new Cesium.Color(1.0, 0.0, 0.0, 1.0));
createPlane(planeYModelMatrix, new Cesium.Color(0.0, 1.0, 0.0, 1.0));

function createPlane(planeModelMatrix, color) {
   // 创建平面
    var planeGeometry = new Cesium.PlaneGeometry({
        vertexFormat : Cesium.PerInstanceColorAppearance.VERTEX_FORMAT,
    });

    // 创建平面外轮廓
    var planeOutlineGeometry = new Cesium.PlaneOutlineGeometry({
    });

    var planeGeometryInstance = new Cesium.GeometryInstance({
        geometry : planeGeometry,
        modelMatrix : planeModelMatrix,
        attributes : {
            color : Cesium.ColorGeometryInstanceAttribute.fromColor(color)
        }
    });

    scene.primitives.add(new Cesium.Primitive({
        geometryInstances : planeGeometryInstance,
        appearance : new Cesium.PerInstanceColorAppearance({
            closed: false,
            translucent: false
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


