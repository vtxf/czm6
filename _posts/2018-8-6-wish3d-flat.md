---
layout: post
title: Wish3D的模型压平功能分析
---

### Wish3D的模型压平功能分析

分析一下wish3dhttp://www.wish3d.com/Examples/Flat.html

##### 节点绘制shader
以下shader来自于PageLODNodeVS.glsl，绘制每一个Wish3D的PageLOD数据时，vertexShader中会从某个深度纹理（实际上是颜色纹理）中获取到每一个顶点的高度值。
```
uniform mat4 czm_projection;
#line 0

#line 0
attribute vec3 position;
uniform mat4 u_modelViewMatrix;
uniform bool u_bFlatten;
uniform sampler2D u_polygonTexture;
uniform vec4 u_polygonBounds;
attribute vec2 textureCoordinates;
varying vec2 v_textureCoordinates;
bool isPointInBound(vec4 point, vec4 bounds) {
 return (point.x>bounds.x&&point.x<bounds.z&&point.y<bounds.y&&point.y>bounds.w);
}
float unpackDepth(const in vec4 rgba_depth) {
 const vec4 bitShifts = vec4(1.0, 1.0 / 255.0, 1.0 / (255.0 * 255.0), 1.0 / (255.0 * 255.0 * 255.0));
 float depth = dot(rgba_depth, bitShifts);
 return depth;
}
void main() {
 if(u_bFlatten) {
 vec4 viewPos = vec4(position.xyz, 1.0);
 if (isPointInBound(viewPos, u_polygonBounds)) {
 vec2 texCoord;
 texCoord.x = (viewPos.x-u_polygonBounds.x)/(u_polygonBounds.z-u_polygonBounds.x);
 texCoord.y = (viewPos.y-u_polygonBounds.w)/(u_polygonBounds.y-u_polygonBounds.w);
 float texelSize = 1.0/4096.0;
 float depth0 = unpackDepth(texture2D(u_polygonTexture, texCoord.xy));
 float depth1 = unpackDepth(texture2D(u_polygonTexture, texCoord.xy + vec2(-texelSize, 0.0)));
 float depth2 = unpackDepth(texture2D(u_polygonTexture, texCoord.xy + vec2(texelSize, 0.0)));
 float depth3 = unpackDepth(texture2D(u_polygonTexture, texCoord.xy + vec2(0.0, -texelSize)));
 float depth4 = unpackDepth(texture2D(u_polygonTexture, texCoord.xy + vec2(0.0, texelSize)));
 float minDepth = min( min( min( min( depth0, depth1), depth2), depth3), depth4);
 if(minDepth*5000.0 > 1.0) {
 float depth = max( max( max( max( depth0, depth1), depth2), depth3), depth4);
 viewPos.z = depth*5000.0;
            }
 gl_Position = czm_projection* u_modelViewMatrix* vec4(viewPos.xyz, 1.0);
        }
 else {
 gl_Position = czm_projection* u_modelViewMatrix* vec4(position.xyz, 1.0);
        }

    }
 else {
 gl_Position = czm_projection* u_modelViewMatrix* vec4(position.xyz, 1.0);
    }
 v_textureCoordinates = textureCoordinates;
```
这张纹理实际上来自于PageLOD数据下的一张纹理变量 _defaultTexture。G代表的是LSPageLODNode。

![](https://images.gitee.com/uploads/images/2018/0806/210245_ba2dee7d_470194.png "屏幕截图.png")

##### 多边形纹理绘制shader
Wish3D在LSPolygonDepth这个类中绘制多边形的纹理。其实的VS和FS分别如下：  
 **PolygonDepthVS**   
```
uniform mat4 czm_projection;
#line 0

#line 0
attribute vec3 position;
varying vec2 depth;
void main() {
 vec4 pos = vec4(position.xyz, 1.0);
 depth = pos.zw;
 pos.z = 0.0;
 gl_Position = czm_projection*pos;
}
```
 **PolygonDepthFS**   
```
#ifdef GL_FRAGMENT_PRECISION_HIGH
 precision highp float;
#else
 precision mediump float;
#endif

#line 0

#line 0
varying vec2 depth;
vec4 packDepth(float depth) {
 const vec4 bias = vec4(1.0 / 255.0, 1.0 / 255.0, 1.0 / 255.0, 0.0);
 float r = depth;
 float g = fract(r * 255.0);
 float b = fract(g * 255.0);
 float a = fract(b * 255.0);
 vec4 color = vec4(r, g, b, a);
 return color - (color.yzww * bias);
}
void main() {
 float fDepth = depth.x / 5000.0;
 gl_FragColor = packDepth(fDepth);
}
```

Wish3D始终在一个4096*4096大纹理上创建多边形，要是有多个多边形，就会等比缩小到这个大纹理能够同时承载多个polygon位置。

但是问题是，如果两个多边形之间距离过远时，纹理可能就很不准确了。

![输入图片说明](https://images.gitee.com/uploads/images/2018/0806/210512_8af25e6e_470194.png "屏幕截图.png")

![输入图片说明](https://images.gitee.com/uploads/images/2018/0806/210520_a2c15fca_470194.png "屏幕截图.png")

![输入图片说明](https://images.gitee.com/uploads/images/2018/0806/210526_b95165b2_470194.png "屏幕截图.png")

正常拍平后的效果是这样的：
![输入图片说明](https://images.gitee.com/uploads/images/2018/0806/210548_547fa6d2_470194.png "屏幕截图.png")

如果绘制了多个压平多边形，且距离过远时，就出现这样的情况了：
![输入图片说明](https://images.gitee.com/uploads/images/2018/0806/210602_b351ed94_470194.png "屏幕截图.png")

##### 我想到的有三种实现方式：
1. tile载入是直接修改顶点坐标，缺点是踏平区域变化时，需要刷新数据；
1. 在cesium自带的clipplane的基础上改改，缺点clipplane本身有潜在的bug，某些3dtiles数据显示有问题；
1. wish3d这种直接用纹理，优点是实时刷新，缺点是距离远了效果也有问题。
