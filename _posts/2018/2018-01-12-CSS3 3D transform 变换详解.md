---
layout: post
title: CSS3 3D transform 变换详解
category: CSS
---

## 一. 相关CSS属性和转换方法

| 属性 | 说明 |
|---|---|
| transform：translate3d(x,y,z) | 平移变换 |
| perspective | 设置视点距离 |
| perspective-origin | 设置视点位置 |
| transform-style | 定义变换类型 |
| backface-visibility | 定义元素在不面对屏幕时是否可见 |

## 二.首先认识rotate3d(x,y,z)

rotate3d(x,y,z)可以看成rotateX()，rotateY()，rotateZ() 三个变换方法的合集，用来定义3D旋转，分别指绕X，Y，Z轴进行旋转。

其中X轴水平于显示器上下边，Y轴平行于显示器左右边，Z轴垂直于显示器平面。如下图：
![创建实例.png](https://geminate.github.io/assets/images/2018/3d-computer-300x259.jpg)

弄清了xyz轴之后，3D旋转就变得十分好理解，如下图
![创建实例.png](https://geminate.github.io/assets/images/2018/rotate.png)

效果见 3D旋转示例 。

## 三. 了解 perspective 视距 和 perspective-origin 视点中心

perspective 属性 用于设置 我们眼睛距离元素的垂直距离。这个距离越小，3D变换的效果就越明显。如果不设置 perspective 属性的话，和 perspective 设置为无穷大的效果是一样的。

这里也许比较难以理解，下面的例子可以帮助 理解 perspective 属性，可以点击这里查看不同的 perspective 下rotateY 的效果：不同Perspective值对变换的影响。

图中的方形宽度是200，绕Y轴旋转45度，当perspective设置为100px的时候，方形仿佛就在眼前旋转一样，因为视距近，因此导致3D效果强烈，而perspective值越大则变换的元素在视觉上离我们越远。

perspective-origin则类似于transform-origin，用于定义一个中心即视点中心，在默认不写该属性情况下，视点中心位于perspective属性所在元素的中心。修改该属性可改变视点中心位置，点击这里查看 视点中心发生变化后对变换造成的影响

图中三个元素均旋转45度，视距均为300px，但最终效果却有所不同，调整变换角度，发现在旋转角度达到70度左右时，视点在最左侧的图缩成了一条线，这是因为视线与旋转后的元素平行。通常视点在元素中心时，旋转90度会导致这一现象，之所以提前平行，正是因为视点发生了移动。（不好理解的话，拿一个本在眼前，当自己正对着本的中心时，本绕Y轴转90度会导致本变成一条竖线，但当平移自己的视点，让自己正对着本的一条竖边时，再旋转本，这时只需要旋转一个小于90的度数本就会变成一条线）

另外视点是垂直于其perspective属性所在的元素的，这意味着我们将perspective属性定义在变换元素的父级（即舞台元素）上时，舞台上的变换元素由于其在舞台上的位置不同，会导致变换效果也不同。可以点击这里查看效果：所在舞台位置不同导致变换效果不同

示例中元素均在灰色的舞台上Y旋转90度，只有位于舞台中心的元素变成一条线消失了，其余元素仍有一部分可以被看到，结合我们平时去电影院的经验不难理解。如果我们想让每个元素的变换效果都一样，我们可以为每个元素单独设置一个舞台元素，或者使用perspective()直接写在变换元素的transform属性里。

## 四. translate-z 调整元素的“远近”

了解了perspective属性后translate-z就不难理解，他控制的是元素的“远近”，设为正值时，元素会沿着Z轴走向我们，设为负值时，元素会沿着Z轴远离我们。直观的效果就是“近大远小”。

举个例子，我们设置舞台元素的perspective为200px，之后变换元素的translate-z为0px时,元素不会有变化，随着translate-z值逐渐变大，元素也会一步步变大，当translate-z为200px时，它会充满整个屏幕，这就是一叶障目的道理，当继续增加translate-z的值时，元素就消失了，因为这时候元素已经走到了我们的视点后面，眼睛自然看不到背后的东西了。

## 五.transform-style和backface-visibility

这两个属性很好理解，transform-style控制子元素是否一起跟随3D变换，backface-visibility则控制元素翻转后，我们能否在元素翻转后看到元素内容。