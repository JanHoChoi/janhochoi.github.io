---
layout: post
title: LayoutGroup组件介绍
date:  2021-05-21 02:24:00 +0800
description: 总结
img: 404.jpeg # Add image post (optional)
tags: [UGUI]
author: # Add name author (optional)
ugui: true
---

{{site.label1}} <a href="https://github.com/janhochoi/" target="\_blank">JanHoChoi</a> {{site.label2}}

# LayoutGroup

这次介绍分为两部分，一部分是介绍Unity的LayoutGroup组件，另一部分是介绍项目Lua中封装的UILayoutGroup类。

## LayoutGroup

### 参数介绍

![image.png](../../assets/img/layoutGroup_1.png)

Padding：控制子物体离容器四周的距离；

Spacing：控制子物体之间的间隙距离；

Child Alignment：控制子物体垂直方向（Upper Middle Lower）水平方向（Left Center Right）的对齐方式；

Control Child Size：是否控制子物体的size（包括width和height），若勾选，则子物体的大小会根据Auto Layout的规则去安排；不勾选则可以自行调整，layoutGroup仅控制位置；

Use Child Scale：布局是否考虑子物体的Scale；比如子物体的width是100，Scale是0.5，若勾选，则当作是widh=50的物体去布局；不勾选则依然考虑原始size作布局；

Child Force Expand：用的不多，勾选后，在获得子物体Size时，子物体的FlexibleSize为Mathf.Max(flexible, 1)，后续会讲到这个属性作用；
```
// HorizontalOrVerticalLayoutGroup.cs

private void GetChildSizes(..., bool childForceExpand, ...)
{
    ...
    if (childForceExpand)
        flexible = Mathf.Max(flexible, 1);
}
```

### 布局相关

首先Unity的布局基于两个概念：①Layout Element；②Layout Controller。

#### Layout Element

只要有RectTransform的gameObject都是Layout Element。Layout Element有如下的一些属性：
- Minimum width
- Minimum height
- Preferred width
- Preferred height
- Flexible width
- Flexible height

它们知道自己的size是多少，但是不会直接控制自己的size；

Text和Image的这些属性也有这些属性，只不过是已经默认写好了的。

![image.png](../../assets/img/layoutGroup_2.png)

如果想修改这些属性，需要手动在Image/Text下挂一个Layout Element组件。

![image.png](../../assets/img/layoutGroup_3.png)

#### Layout Controller

包括三种LayoutGroup（Horizontal、Vertical、Grid）以及Content Size Fitter，还有一个冷门的Aspect Ratio Fitter不过用的较少。

其中三种Layout Group控制的是子物体的Size（实现了ILayoutGroup接口），而Content Size Fitter控制的是本身的Size（实现了ILayoutSelfController接口）。

Layout Controller通过上述Layout Element的属性，去算各子物体的size。算子物体size的规则如下，以Width为例：

1. 先算Min Width，子物体最小宽度为Min Width的数值；
2. 如果Layout Group有多余的空间，则按比例分配给Preferred Width的子物体，比如多余了50像素，A物体Preferred Width - Min Width = 50，B物体Preferred Width - Min Width = 25，则A物体分配50 * 50 / 75 = 33.333像素，B物体分配16.667像素；
3. 如果Layout Group有多余的空间，则按比例分配给Flexible Width > 0的子物体，比如剩下100像素，A物体Flexible=3，B物体Flexible=2，则A分配到60像素，B分配到40像素；

## 应用

### 变长信息

![image.png](../../assets/img/layoutGroup_4.png)

如图所示，右边的【姓名】【消息内容】【时间戳】放在了pnl_content下，组件有Vertical Layout Group以及Content Size Fitter。

![image.png](../../assets/img/layoutGroup_5.png)

同时，因为【消息内容】不仅有文字，而且有一个聊天框也需要动态变化，所以用pnl_txt作为父节点控制文字和聊天框的层级以及尺寸大小。pnl_txt也设定了Vertical Layout Group组件。

![image.png](../../assets/img/layoutGroup_6.png)

当【消息内容】长度改变时，布局系统先自底向上（bottom-up）顺序计算每个Layout Element的各属性（Min、Preferred、Flexible）。再由顶向下（top-down）顺序去给每个Layout Element设值。

所以长度改变后，最顶层的Content Size Fitter首先修改的是最顶层pnl_content的尺寸，再一步步通过Preferred尺寸去分配每个子物体的大小。达到变长的效果。

## 参考资料

[Unity官方关于布局的文档Auto Layout](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/UIAutoLayout.html)