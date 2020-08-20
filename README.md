# 关于3D模型在线计算体积与尺寸

版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

本文链接： [https://blog.csdn.net/qq_34320912/article/details/93143792](https://blog.csdn.net/qq_34320912/article/details/93143792)
 
模型大多是不规则的物体，无法用传统数学公式去计算，这里采用Three.js（针对3D模型展示技术）。

本文的例子是一个3D模型展示的例子，在此基础上二次开发计算体积与尺寸。

当前three.js 版本为92。


体积与尺寸console.log出来了。实际尺寸取2x,2y,2z，对比信息可参照魔猴网打印页面
 [魔猴网](http://www.mohou.com/yundayin/yundayin.html)。




# Three.js模型几何体面积、体积计算


版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

本文链接： [https://blog.csdn.net/u014291990/article/details/91492447](https://blog.csdn.net/u014291990/article/details/91492447)
 
在工作中通过Three.js开发项目的时候，一些特定的情况下你可能需要计算一个三维模型的表面积或者体积，比如在3D打印的Web项目中，你需要计算一个三维模型的体积，然后通过体积计算打印一个三维模型所需要的3D打印材料费；比如开发的一个程序中，需要自动计算一个地面、墙面或某个零件的表面需要多少涂料，肯定需要先计算它的外表面面积是多少。

个人技术博客

## 思路——模型几何体表面积计算

如果是一个立方体、球体等规则几何体，计算它们的面积，可以直接代入公式，实际应用中，不一定就是规则几何体，下面的任务是探讨一个通用的算法，可以计算任意形状的几何体，任意自由形状的曲面表面积。

一个三维模型，可能包含多个网格模型 `Mesh`，如果一个三维模型有多个网格模型，你可以分别计算每一个网格模型的表面积，然后求和即可。这里先考虑如何计算一个网格模型`Mesh`的表面积,每个网格模型的几何体本质上都是多个三角形组成，计算一个网格模型的外表面积，只需要计算该网格模型几何体所有三角形面积就可以，如果想计算所有三角形的面积，肯定要先计算一个三角形的面积。

## 一个三角形面积

已知三角形的三个顶点`p1, p2, p3`的坐标值，利用三个顶点的坐标计算三角形的面积，每个顶点的坐标通过一个三维向量对象`Vector3`表示，具体代码见下方。

下面的计算主要是向量的计算，如果你对`Three.js`向量相关内容不了解，可以查看官方文档`Vector3`接口的介绍，更多向量`Vector3`相关的内容可以参考`Three.js`进阶视频教程数学部分。

```$xslt

//三角形面积计算
function AreaOfTriangle(p1, p2, p3){
  var v1 = new THREE.Vector3();
  var v2 = new THREE.Vector3();
  // 通过两个顶点坐标计算其中两条边构成的向量
  v1 = p1.clone().sub(p2);
  v2 = p1.clone().sub(p3);

  var v3 = new THREE.Vector3();
  // 三角形面积计算
  v3.crossVectors(v1,v2);
  var s = v3.length()/2;
  return s
}

```

## 计算几何体表面积

上面代码封装了一个三角形面积计算函数`AreaOfTriangle()`，只要以三维向量`Vector3`形式输入三角形三个顶点坐标就可以返回一个三角形面积值。一般来说加载外部`ply、stl、obj、fbx`等格式三维模型，模型的几何体都是`BufferGeometry`,可以通过`BufferGeometry`的顶点索引`.index`属性和`attributes.position`属性获得三角形顶点的位置坐标。

这里不加载外部模型，先以`Three.js`自带的`Geometry`类型API为例创建一个几何体，然后计算它的表面，也就是计算一个球体或立方体的表面积来验证上面函数`AreaOfTriangle()`是否正确，你可以通过公式先计算一个立方体或球体的表面积，然后再和通过调用`AreaOfTriangle()`函数计算的结果相比较，来验证`AreaOfTriangle()`函数的算法是否可行。

```
// var geometry = new THREE.SphereGeometry(10, 50, 50);
var geometry = new THREE.BoxGeometry(10, 10, 10);
// 声明一个变量表示几何体的表面积
var area = 0.0;
// 遍历一个几何体的全部三角形geometry.faces，所有三角形面积累积就是几何体的表面积
// 对于不规则曲面，细分程度越高，面积计算精度越高
for (var i = 0; i < geometry.faces.length; i++) {
  //三角形的对应顶点索引
  var a = geometry.faces[i].a;
  var b = geometry.faces[i].b;
  var c = geometry.faces[i].c;
  // 获得三角形对三个顶点的坐标
  var p1 = geometry.vertices[a];
  var p2 = geometry.vertices[b];
  var p3 = geometry.vertices[c];
  // 调用三角形面积计算函数AreaOfTriangle
  area += AreaOfTriangle(p1, p2, p3); //三角形Face3面积累计计算
}
// 查看面积计算结果
document.write("面积:" + area)

```

## 计算几何体体积

几何体的每一个三角形可以和顶点坐标构成一个四面体，计算一个几何体的体积，可以计算所有所有三角形和坐标原点构成的四面体体积，然后求和。

## 四面体体积

三角形三个顶点和坐标系原点构成一个四面体，已知三角形的三个顶点坐标`p1、 p2`和`p3`，计算该四面体的体积。

计算的结果可能是正可能是负，几何体所有的四面体累积后可以计算出正确的结果。

```
// 四面体体积计算公式
function vFun(p1, p2, p3) {
  //借助threejs的Vector3的叉乘、点乘方法进行计算
  return p1.clone().cross(p2).dot(p3) / 6; //p1叉乘p2点乘p3除以6
}
```

## 几何体体积计算
下面一个立方体`BoxGeometry`为例验证`vFun`函数封装的算法是否正确,实际应用的时候，如果是`BufferGeometry`类型几何体，只需要改变顶点的坐标获取方式就可以，具体查看`Three.js`官方文档，或者学习`Three.js`视频教程的第二章。

```
var box = new THREE.BoxGeometry(10, 10, 10);
box.translate(200, 200, 1000); //平移之后并不影响结果
// 声明一个变量表示几何体的体积
var V = 0.0;
// 几何体三角形索引
for (var i = 0; i < box.faces.length; i++) {
  // 几何体三角形索引
  var index0 = box.faces[i].a;
  var index1 = box.faces[i].b;
  var index2 = box.faces[i].c;
  // 通过索引访问三角形顶点坐标值
  var p0 = box.vertices[index0];
  var p1 = box.vertices[index1];
  var p2 = box.vertices[index2];
  //使用下面的函数，并不会改变p0, p1, p2引用指向的geo顶点坐标值
  //三角形与坐标原点构成的四面体体积累计计算
  V += vFun(p0, p1, p2);
}
//查看体积计算结果
document.write("体积:" + V);
console.log("体积", V);
```
