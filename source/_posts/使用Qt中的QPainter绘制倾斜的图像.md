---
title: 使用Qt中的QPainter绘制倾斜的图像
date: 2016-09-12 20:29:02
tags: [Qt, OpenGL]
categories: Learn&Think
---

最近项目中需要做一个功能，把所有家装设计布局中的门窗，以标准图示的形式进行显示。

由此产生了一个问题，如何应对倾斜显示的墙上的门窗？

Qt中的QPainter可以绘制倾斜图像，在使用之前先了解下这个接口的绘制机制。

  1.我们先尝试不添加位移和旋转，直接设置大小绘制一个图像，伪代码如下：

```C
QRectF src(绘制图片的大小);
QRectF tgt(绘制目标区域的大小);
painter.drawImage(tgt, img, src);
```

我们发现绘制结果显示在整个屏幕的左上角，以tgt为大小绘制了img这个图像。
由此得出结论，绘制的区域坐标系为屏幕左上角为原点，向右为x正向，向下为y正向。
而绘制机制实际上是将图片的左上角对准绘制点，绘制制定区域。

  2.现在尝试加上位移，使用translate接口，伪代码如下：

```C
painter.translate(100, 200);
QRectF src(绘制图片的大小);
QRectF tgt(绘制目标区域的大小);
painter.drawImage(tgt, img, src);
```

可以发现图片的左上角对到了(100, 200)这个点。

  3.现在位移的基础上加上旋转，使用rotate接口：

```C
painter.rotate(30); // 顺时针旋转30度
painter.translate(100, 200);
QRectF src(绘制图片的大小);
QRectF tgt(绘制目标区域的大小);
painter.drawImage(tgt, img, src);
```

此时情况不太对劲啊，本来我的想法是先以图片左上角为中心点旋转30度，然后位移到(100, 200)这个点。
但是结果居然是先位移到(100, 200)，然后整体旋转了30度。

但是咱读书多，这种情况是可以解释的。

实际上QPainter使用OpenGL来进行绘制，那么drawImage实质上是绘制矩形区域，贴上图片绘制到帧缓冲。
这里就有个图形学的特色了，最终绘制结果的位置是由模型矩阵进行控制的，也即pvm矩阵中的m。
然而图形学中，矩阵的先后顺序和逻辑操作的先后顺序是反的，如果你要先进行旋转后进行位移，
那么旋转矩阵要乘在位移矩阵的右边，也就是后面。
所以，如果先调用rotate后调用translate，实际上是先进行了位移后进行旋转。顺序就反了。
让我调整下顺序看看：

```C
painter.translate(100, 200);
painter.rotate(30); // 顺时针旋转30度
QRectF src(绘制图片的大小);
QRectF tgt(绘制目标区域的大小);
painter.drawImage(tgt, img, src);
```

这次结果就正确了。我们现在可以将任意图片，用QPainter以任意角度，绘制到屏幕的任意区域了~~~
