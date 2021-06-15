---
layout: post
title: RectTransform组件介绍
date:  2021-02-21 09:45:00 +0800
description: 总结
img: 404.jpeg # Add image post (optional)
tags: [UGUI]
author: # Add name author (optional)
ugui: true
---

{{site.label1}} <a href="https://github.com/janhochoi/" target="\_blank">JanHoChoi</a> {{site.label2}}

# RectTransform

![image.png](../../assets/img/rectTransform_1.png)

RectTransform比较特殊的概念是Anchors和Pivot。

![image.png](../../assets/img/rectTransform_2.png)

Scene视图中蓝色圈圈是中心点Pivot，4个白色三角形是4个锚点Anchors。4个锚点可以通过鼠标拖动分别移动位置，也可以点中间拖动同时移动4个锚点。蓝色Pivot也可以选中随便拖。

## Pivot的意义

中心点Pivot，表示该UI的旋转Rotation和缩放Scale都以Pivot的点为中心。Pivot没有规定一定要在UI内，也可以在UI元素外，即它的xy取值可以>1或者<0。

## Anchor的意义

4个锚点Anchor决定的是该RectTransform怎么“锚”在它的父RectTransform上。

![image.png](../../assets/img/rectTransform_3.png)

minXY与maxXY四个值的物理意义如上图所示，决定了4个锚点在父对象的哪个位置。如果在Scene视图拖动时，4个值的取值范围是\[0,1]，但是在代码或是Inspector里面可以随意输入值。**无论父对象的大小怎么变，4个Anchor相对父对象的相对位置是不变的。**

- 当min X = max X且min Y = max Y时，即4个锚点在同一点时，

![image.png](../../assets/img/rectTransform_4.png)

此时Inspector面板显示的是这4个值。Pos X和Pos Y表示Pivot与锚点的距离。Width和Height表示该UI对象的宽度和高度。

- 当min X = max X时，即4个锚点连成一条垂直线时，

![image.png](../../assets/img/rectTransform_5.png)

此时Pos Y/Height被Top/Bottom代替。Pos X表示Pivot与锚点连线的距离。Width表示该UI对象宽度。Top和Bottom表示该UI对象上边与下边分别与上下锚点的距离（有点像写网页前端的那些padding的感觉）。这种情况下，该对象的高度会随着父对象高度变化而变化。Top和Bottom值不变下，父对象高度越大，子对象高度也越大。

- 当min Y = max Y时，即4个锚点连成一条水平线时，

![image.png](../../assets/img/rectTransform_6.png)

同理，此时Pos X/Width被Left/Right代替。Pos Y表示Pivot与锚点连线的距离。Height表示该UI对象高度。Left和Right表示该UI对象左边与右边分别与左右锚点的距离。这种情况下，该对象的宽度会随着父对象宽度变化而变化。Left和Right值不变下，父对象宽度越大，子对象宽度也越大。

- 当min X != max X且min Y != max Y，即4个锚点分开时，

![image.png](../../assets/img/rectTransform_7.png)

此时PosX/PosY/Width/Height被Left/Right/Top/Bottom代替。分别表示该UI四边距离锚点的四边的距离。这种情况下，该对象的宽度和高度都会随着父对象变化而变化。

---

此外，如果父物体有Layout Group组件，某些属性会被Layout Group组件控制，无法手动调整。

![image.png](../../assets/img/rectTransform_8.png)

## Anchor Preset

![image.png](../../assets/img/rectTransform_9.png)

Unity提供了一些Anchor点的预设（主要是4个锚点在同一点情况的不同位置，按住Shift可以同时设置锚点，按住Alt可以设置位置。主要是给UI同学用的。

## 相关属性介绍

### localPosition

子物体的局部空间位置。子物体的pivot点所在位置与父物体的pivot点所在位置的差值。

### anchoredPosition

anchoredPosition为Pivot在自身区域的比例，映射到Anchors上的点，再与Pivot的相对位置，就是anchoredPosition。如网图所示。

![image.png](../../assets/img/rectTransform_10.png)

当4个锚点在同一点时，anchoredPosition为Pivot位置与Anchor位置的距离。可以通过修改anchoredPosition达到移动UI的效果（只修改anchoredPosition不会影响UI大小）。

### anchorMin/anchorMax

即锚点的Min和Max值。修改该值时，UGUI会使anchoredPosition不变，以及Left/Right/Top/Bottom四个值不变，而因为Anchor变了，所以该UI的大小会变。

![anchorMax.gif](../../assets/img/rectTransform_11.gif)

### offsetMin/offsetMax

![image.png](../../assets/img/rectTransform_12.png)

offsetMin为左下锚点到UI左下角的向量；offsetMax为右上锚点到UI右上角的向量。与Rotate以及Scale无关。

### sizeDelta

```
sizeDelta = offsetMax - offsetMin
offsetMax = rectMaxPos - anchorMaxPos // 右上锚点到UI矩形右上角的向量
offsetMin = rectMinPos - anchorMinPos // 左下锚点到UI矩形左下角的向量
代入
sizeDelta = (rectMaxPos - rectMinPos) – (anchorMaxPos – anchorMinPos)
sizeDelta = rectSize - anchorSize
```
所以sizeDelta表示RecTransform的矩形长宽减去锚框长宽。当4个锚点都在一起时，sizeDelta=RectTransform的真实长宽。

## 相关方法介绍

### 移动RectTransform的位置

1. 修改localPosition属性，改后的结果与Anchor位置无关，与Pivot位置有关；

2. 修改anchoredPosition属性，改后的结果与Anchor位置有关，与Pivot位置也有关，关系见上面属性介绍；

### 修改RectTransform大小

1. 修改sizeDelta属性；

需要注意，修改后的尺寸根据上面sizeDelta的定义计算（只有4锚点在一起时sizeDelta才代表长宽），所以改后的结果与Anchor有关。同时又会根据Pivot点位置决定改后矩形的corner具体位置，所以与Pivot位置也有关。

2. 使用SetSizeWithCurrentAnchors函数；

根据当前锚点布局去改变水平或垂直方向的尺寸。如果父物体的size改变了，则anchor的矩形面积也变了，sizeDelta不变的情况下，该物体的rect也会相应改变。改变后结果同样与Pivot位置也有关。

3. SetInsetAndSizeFromParentEdge函数；

这个方法以父物体的四条边中的一条为参照，去设置该RectTransform的大小。这个方法不仅改了RectTransform大小，还会修改anchors、anchoredPosition和sizeDelta。用得不多，可以自己试试。

## 相关资料

[Rect Transform - Unity Official Tutorials视频讲解](https://www.youtube.com/watch?v=FeheZqu85WI)

[RectTransform成员属性的再认识(有很多推导)](https://zhuanlan.zhihu.com/p/194317677)